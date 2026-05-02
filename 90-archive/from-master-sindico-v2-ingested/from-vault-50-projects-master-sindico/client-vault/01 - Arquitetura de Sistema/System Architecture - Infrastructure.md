---
title: System Architecture — Infrastructure
parent: System Architecture
domain: CROSS-DOMAIN
status: Phase 0
tags:
  - mastersindico
  - architecture
  - infrastructure
  - nats
  - outbox
  - meilisearch
---

# Infrastructure Architecture

> **Status:** Fase 0 — primeira coisa a implementar
> **Escopo:** NATS JetStream, Transactional Outbox, Meilisearch, WebSocket, Storage, Audit
> **Source of Truth:** Blueprint v2.0 + Decisões técnicas confirmadas

> Veja o canvas visual: [[Infrastructure Architecture.canvas]]

---

## 1. NATS JetStream (Event Bus)

### Por que NATS JetStream?

| Critério | EventEmitter (atual) | NATS JetStream |
|----------|---------------------|----------------|
| Durabilidade | Nenhuma — perde eventos no crash | Persistente em disco |
| Replay | Impossível | Replay desde qualquer ponto |
| Multi-consumer | Single handler | Consumer groups com ack individual |
| Cross-process | Impossível (in-memory) | Via TCP, preparado para microserviços |
| Dead letter | Não existe | Built-in |
| Ordering | Garantido (sync) | Por subject/stream |

### Streams (1 por HIGH-DOMAIN)

| Stream | Subjects | Retenção | Max Age |
|--------|---------|----------|---------|
| **IDENTITY** | `identity.auth.>`, `identity.billing.>`, `identity.user.>`, `identity.profile.>` | Interest-based | 30 dias |
| **INSTITUTIONAL** | `institutional.condominium.>`, `institutional.governance.>`, `institutional.compliance.>`, `institutional.validation.>` | Interest-based | 90 dias |
| **COMMERCIAL** | `commercial.connect-me.>`, `commercial.proposal.>`, `commercial.voting.>`, `commercial.deal.>`, `commercial.execution.>`, `commercial.vizinhanca.>` | Interest-based | 90 dias |
| **CONTENT** | `content.video.>`, `content.academy.>`, `content.talent.>`, `content.interaction.>` | Interest-based | 30 dias |
| **ASSEMBLY** | `assembly.config.>`, `assembly.live.>`, `assembly.vote.>`, `assembly.speech.>`, `assembly.checkin.>` | Limits (replay) | 365 dias |
| **AUDIT** | `audit.>` | Limits | **5 anos** (LGPD) |

### Subject Naming Convention

```
{high-domain}.{module}.{event-type}

Exemplos:
  commercial.connect-me.request-created
  institutional.governance.timeline-published
  assembly.live.vote-cast
  identity.billing.subscription-expired
```

### Consumer Groups (durable, pull-based)

| Consumer | Streams | Filter | Propósito |
|----------|---------|--------|-----------|
| **timeline-projector** | COMMERCIAL, INSTITUTIONAL | `*.*.validated`, `*.deal.confirmed` | Auto-publica na Timeline |
| **master-plan-updater** | INSTITUTIONAL | `institutional.governance.timeline-published` | Recalcula status Master Plan |
| **notification-dispatcher** | ALL | `*.*.created`, `*.*.confirmed`, `*.*.expired` | Roteia → email/push/SMS |
| **meilisearch-syncer** | IDENTITY, COMMERCIAL, CONTENT | `*.profile.*`, `*.video.*`, `*.talent.*`, `*.partner.*` | Sync PostgreSQL → Meilisearch |
| **audit-logger** | ALL | `>` (tudo) | Grava audit_trail append-only |
| **assembly-broadcaster** | ASSEMBLY | `assembly.live.>` | NATS → Redis pub/sub → WebSocket |

---

## 2. Transactional Outbox Pattern

### Problema que resolve
Sem outbox, há risco de: dado persistido mas evento perdido, ou evento publicado mas dado não commitado. O Outbox garante atomicidade.

### Integração com DrizzleUnitOfWork

O `DrizzleUnitOfWork` existente (em `infrastructure/database/drizzle/unit-of-work/`) usa `AsyncLocalStorage` para propagar transações. O Outbox se integra assim:

```
Use Case (dentro do UnitOfWork):
┌──────────────────────────────────────────────────┐
│  await unitOfWork.run(async () => {              │
│    // 1. Persiste dados na transação             │
│    await repo.create(entity);                    │
│                                                  │
│    // 2. Registra evento na MESMA transação      │
│    await outbox.publish({                        │
│      eventType: 'commercial.deal.confirmed',     │
│      aggregateId: deal.id,                       │
│      payload: { dealId, condominiumId }          │
│    });                                           │
│  });                                             │
│  // TX commit = dados + evento atômicos          │
└──────────────────────────────────────────────────┘
```

### Outbox Publisher (background worker)

```
loop (100ms):
  events = SELECT * FROM outbox_events
           WHERE status = 'pending'
           ORDER BY created_at
           LIMIT 100

  for event in events:
    nats.publish(event.event_type, payload)
    UPDATE outbox_events
      SET status = 'published',
          published_at = now()
      WHERE id = event.id

  on failure:
    retry_count++
    backoff exponencial (100ms, 200ms, 400ms, ...)
    dead-letter após 10 falhas
```

### Schema: outbox_events

```
outbox_events:
  id              UUID (PK, UUIDv7)
  aggregate_type  TEXT (e.g., 'closed_deal', 'timeline_entry')
  aggregate_id    UUID
  event_type      TEXT (e.g., 'commercial.deal.confirmed')
  payload         JSONB
  metadata        JSONB (user_id, tenant_id, correlation_id, ip)
  status          ENUM('pending', 'published', 'failed')
  published_at    TIMESTAMP (nullable)
  retry_count     INTEGER DEFAULT 0
  created_at      TIMESTAMP
```

### Alteração na interface IUnitOfWork

- Adicionar `publishEvent(event)` na interface `IUnitOfWork`
- Dentro do `run()`, eventos são coletados e escritos na `outbox_events` na mesma transação
- Novo serviço `OutboxPublisher` (singleton, registrado no Awilix) faz poll + publish

---

## 3. Meilisearch

### Por que Meilisearch (e não PostgreSQL tsvector)?

| Critério | tsvector (atual) | Meilisearch |
|----------|-----------------|-------------|
| Typo tolerance | Nenhuma | Built-in |
| Faceted search | Manual com JOINs | Built-in |
| Relevância | Básica (ts_rank) | Configurable ranking |
| Multi-index | Uma tabela por vez | Múltiplos índices |
| Performance | Degrada com volume | Sub-50ms consistente |
| Filtros combinados | Queries complexas | Declarativo |

### Índices

| Índice | Searchable | Filterable | Sortable | Visibilidade |
|--------|-----------|------------|----------|--------------|
| **companies** | name, fantasy_name, description, categories, city | category, city, state, plan_type, rating_avg, status | rating_avg, created_at, name | Plan-based |
| **sindicos** | name, city, state, sindico_type, governance_markers | city, state, sindico_type, plan_type | — | — |
| **videos** | title, description, video_type | video_type, company_id, status, created_at | views_count, created_at | — |
| **partners** | name, category, neighborhood, city, description | category, neighborhood, city, has_active_promotion | — | — |
| **curricula** | areas_of_interest, availability, contract_types, nr_courses | areas_of_interest, cnh_type, availability, contract_type, commute_time, salary_range | created_at, updated_at | Companies Plus/Pro |
| **courses** | title, description, category | category, level, type (free/paid), target_audience | — | — |
| **forum_topics** | title, content, category, tags | category, status, author_type | — | — |

### Sync Strategy

1. **Real-time:** Consumer NATS `meilisearch-syncer` escuta eventos de criação/atualização
2. **On event:** Fetcha dado atualizado do PostgreSQL → upsert no Meilisearch
3. **Bulk recovery:** Job agendado (toad-scheduler) para full re-sync
4. **Latência:** Eventual consistency, tipicamente < 500ms
5. **Adapter:** `MeilisearchSearchProvider` implementa interface `ISearchProvider` existente

---

## 4. WebSocket Architecture (Assembly)

### Stack
- **Server:** `@fastify/websocket`
- **Auth:** JWT no query param ou cookie
- **Rooms:** Scoped por `assembly_id`
- **Fan-out:** Redis pub/sub para multi-instance

### Channels

```
assembly:{id}:telao     → presença, quorum, item atual, fila de fala
assembly:{id}:voting    → contagem votos, status votação
assembly:{id}:speech    → fila de fala, timer, extensões
assembly:{id}:checkin   → presença em tempo real
assembly:{id}:control   → ações do presidente
```

### Fluxo

```
1. Voto registrado → PostgreSQL + outbox (TX atômica)
2. Outbox publisher → NATS assembly.live.vote-cast
3. Consumer assembly-broadcaster → Redis pub/sub channel
4. Todas instâncias Fastify → push para WebSocket clients
```

---

## 5. Storage (Cloudflare R2)

| Bucket | Conteúdo | Acesso |
|--------|----------|--------|
| **documents** | Propostas, edital, relatórios, anexos | Signed URLs (15min TTL) |
| **media** | Thumbnails, logos, avatares | Public CDN |
| **videos** | Raw uploads (pré-Mux processing) | Internal only |
| **exports** | PDFs gerados, CSVs | Signed URLs (1h TTL) |

**Adapter:** `IStorageProvider` → `R2StorageProvider`

---

## 6. Audit Trail

```
audit_trail:
  id              UUID (PK, UUIDv7)
  actor_id        UUID (who)
  action          TEXT (what)
  resource_type   TEXT (on what)
  resource_id     UUID
  tenant_id       UUID (nullable)
  metadata        JSONB (details, ip, user_agent)
  created_at      TIMESTAMP

  -- Append-only, NEVER updated or deleted
  -- Retenção: 5 anos (LGPD + governança)
  -- Populado pelo consumer audit-logger via NATS
```

---

## 7. Notification Pipeline

Evolução do `CommunicationModule` existente para um sistema cross-domain.

### Canais

| Canal | Provider | Adapter |
|-------|----------|---------|
| Email | Resend (primário), Gmail SMTP (fallback) | `IEmailProvider` |
| Push | Firebase Cloud Messaging | `IPushProvider` |
| SMS | Twilio ou Zenvia (pendente Decision 14) | `ISMSProvider` |

### Regras de Roteamento

O `notification-dispatcher` consumer avalia cada evento e decide quais canais usar baseado em:
1. Tipo de evento (urgente vs informativo)
2. Preferências do usuário (opt-in/opt-out)
3. Tipo de tenant e plano
4. Horário (silêncio noturno para push/SMS)

---

## 8. Módulos de Infraestrutura

| Módulo | Tipo | DI Lifetime |
|--------|------|-------------|
| **OutboxModule** | Serviço singleton + background worker | SINGLETON |
| **NotificationModule** | Consumer NATS + dispatchers | SINGLETON |
| **StorageModule** | Adapter R2 | SINGLETON |
| **AuditModule** | Consumer NATS + writer | SINGLETON |
| **NATSModule** | Client NATS + stream/consumer setup | SINGLETON |
| **MeilisearchModule** | Client Meilisearch + sync service | SINGLETON |

---

## Navegação

← [[System Architecture - Assembly]] | [[System Architecture]] | [[System Architecture - Data Model]] →
