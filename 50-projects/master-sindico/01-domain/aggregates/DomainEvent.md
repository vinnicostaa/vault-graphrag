---
title: Aggregate — DomainEvent
type: spec
tags: [domain, ddd, aggregate, cross-domain, event-sourcing, outbox, master-sindico, v2]
bc: cross-domain
source: 01-domain/domain-events.md + 01-domain/invariants.md (INV-094, INV-095) + ADR 0015 (saga orchestration)
created: 2026-04-23
updated: 2026-04-23
---

# Aggregate — `DomainEvent`

**BC**: cross-domain · **Raiz**: ✅ (agregado transversal — interface canônica de evento de domínio)

Interface comum + metadata canônica de todo evento emitido por qualquer BC (identity, billing, institutional, commercial, content, assembly, cross-domain). Persistido no **outbox pattern** (`domain_events` table) dentro da mesma transação que originou o estado → garantia at-least-once via drain worker NATS (INV-095).

Criado como fechamento do finding **A-DC-QA-040** (cross-domain aggregates cross-BC ausentes).

## Entidade raiz

```go
type DomainEvent struct {
    id             EventID        // UUIDv7 (INV-094) — ordenável sem coordenação
    aggregateType  string         // "User" | "Subscription" | "Assembly" | ...
    aggregateID    string         // id do aggregate que emitiu
    eventType      string         // "UserCreated" | "SubscriptionActivated" | ... (nome canônico)
    eventVersion   int            // 1, 2, 3 — evoluções do schema do evento
    payload        []byte         // JSON serializado (contrato versioned em `domain-events.md`)
    metadata       EventMetadata  // VO — ver abaixo
    occurredAt     time.Time      // wall-clock (UTC) do evento
    publishedAt    *time.Time     // wall-clock quando drain worker publicou em NATS
    tenantID       *string        // condominium_id | company_id | NULL se cross-tenant
    correlationID  string         // propagado pra trace distribuído (OTel)
    causationID    *string        // id do evento que causou este (saga chain)
}
```

## Value Objects

```go
type EventMetadata struct {
    ActorUserID   *string  // user_id que disparou (NULL se system)
    ActorRole     *string  // role no momento do disparo
    IP            string   // audit/antifraud
    UserAgent     string   // idem
    RequestID     string   // trace_id do OTel
    SessionID     *string  // session que originou
    SourceService string   // "api" | "worker" | "cron" | "webhook"
}

type EventID string  // UUIDv7
```

## Invariantes

- **INV-094**: `DomainEvent.id` UUIDv7 (ordenável sem coordenação — DR-017)
- **INV-095**: Eventos publicados dentro da mesma tx via outbox pattern (tabela `domain_events`)
- Schema `event_type + event_version` versionado em [[../domain-events]] — evolução sem break backward compatibility
- `occurredAt` nunca alterado pós-insert (append-only)
- `publishedAt` só preenchido pelo drain worker (INV-095); retry idempotente via (provider='nats', event_id) em `webhook_events` ledger

## Factories

```go
// NewDomainEvent cria evento novo — chamado dentro de UoW do aggregate que emite
func NewDomainEvent(aggType, aggID, eventType string, version int, payload any, md EventMetadata, tenantID *string, correlationID string) (*DomainEvent, error) {
    // valida: eventType != "", version > 0, payload não-nil, correlationID UUID
}

// ReconstructDomainEvent — sem validação (vem do repo/outbox)
func ReconstructDomainEvent(id EventID, ...) *DomainEvent
```

## Métodos de domínio

- `MarkPublished(now time.Time)` — drain worker chama após publish NATS sucesso
- `MarkFailed(err error)` — incrementa retry count; DLQ após N tentativas
- `IsStale(now time.Time, staleAfter time.Duration) bool` — utility pra alerta observabilidade

## Repository interface (Go)

```go
// domain/shared/domain_event_repository.go
type IDomainEventRepository interface {
    Save(ctx context.Context, e *DomainEvent) error                     // INSERT (outbox, dentro da tx do aggregate)
    FindUnpublished(ctx context.Context, limit int) ([]*DomainEvent, error)  // drain worker polling
    MarkPublished(ctx context.Context, id EventID, at time.Time) error
    MarkFailed(ctx context.Context, id EventID, err string) error
    FindByCorrelationID(ctx context.Context, corID string) ([]*DomainEvent, error) // debugging / saga trace
}
```

**Uso crítico**: `Save` chamado **dentro** de `UnitOfWork.Run(ctx, fn)` junto com a mutação que originou o evento → atomicidade (INV-095; outbox pattern). Publicação NATS é async (drain worker); falha NATS não derruba a operação de domínio.

## Eventos relacionados (meta)

Este aggregate **não emite** eventos próprios (seria recursão); é a estrutura comum usada por todos os outros events. Ver lista canônica em [[../domain-events]] (E-001 `UserCreated` até E-### cobrindo ~80 eventos).

## Evolução de schema

- Nunca remover campos em versão N pós-produção; adicionar `event_version=N+1` com novo layout.
- Consumers validam versão antes de unmarshal.
- Drain worker não transforma evento — só publica bytes.

## ⚠️ Pendências

- **P-DE-001**: decidir se `causationID` é obrigatório em sagas (orchestration — ADR-0030 proposto) ou opcional (choreography).
- **P-DE-002**: retention do outbox — arquivar pós-publish após 30d? 7d? manter 5 anos pra audit (INV-033)?

## Links

- [[../bounded-contexts#7-cross-domain]]
- [[../domain-events]] — catálogo E-### completo
- [[../invariants#cross-domain-inv-086-a-inv-100]] (INV-094, INV-095)
- [[OutboxEntry]] — relacionado, registro de estado de publicação
- [[WebhookEvent]] — ledger de webhooks inbound (provider events)
- [[AuditEntry]] — superconjunto de audit (muitos eventos disparam AuditEntry também)
- [[../../02-architecture/patterns]] — outbox pattern, saga orchestration
