---
title: Clean Architecture — as 4 camadas
type: concept
tags: [architecture, clean-architecture, ddd, cqrs, layers]
source: .kiro/steering/sdd-layers.md
created: 2026-04-22
aliases: [Clean Arch, Camadas Clean Architecture]
---

# Clean Architecture — as 4 camadas

Cada feature começa com spec antes de código. **Spec define contrato, código implementa.** Nunca o contrário.

Dependências apontam **pra dentro**: `infrastructure → application → domain`. `domain/` não importa nada das outras camadas. Ver [[../principles/do-dont-checklist]] pra regra dura.

## Camada 1: Domain

**O que vai aqui:** regras de negócio puras, sem I/O, sem dependências externas.

### Entities

Conceitos com **identidade** e **ciclo de vida**.

Regras:
- Estado **privado**, getters explícitos
- Métodos de domínio expressam intenção de negócio (`Activate()`, `Cancel(reason)` — não `SetStatus`)
- Factory function `NewX()` (criação) ou `ReconstructX()` (hidratação do banco)
- **Nunca** importa de `infrastructure/` ou `application/`

Exemplos completos em [[../../20-stacks/go/go-patterns]].

### Value Objects

Sem identidade. Imutáveis após criação.

Regras:
- Validação **na factory**, nunca depois
- Sem setters
- Comparação **por valor**, não por referência
- Wrapping de primitivos com invariantes (`Email`, `CPF`, `Money`, `Phone`)

Ver [[../patterns/value-object]] *(a criar)*.

### Repository Interfaces

Contrato de persistência — **sem implementação**.

Regras:
- **Apenas interface** no domínio
- Métodos retornam **entidades de domínio**, nunca tipos do banco
- `context.Context` sempre primeiro parâmetro

```go
type ISubscriptionRepository interface {
    FindByID(ctx context.Context, id string) (*Subscription, error)
    FindActiveByUserID(ctx context.Context, userID string) (*Subscription, error)
    Save(ctx context.Context, subscription *Subscription) error
    Delete(ctx context.Context, id string) error
}
```

### Domain Events

Comunicação **entre bounded contexts**.

Regras:
- **Imutável** após criação
- Nome **no passado**: `SubscriptionActivated`, não `ActivateSubscription`
- Contém **apenas** dados necessários pros handlers

```go
type SubscriptionActivated struct {
    SubscriptionID string
    UserID         string
    PlanID         string
    occurredAt     time.Time
}

func (e SubscriptionActivated) EventName() string     { return "subscription.activated" }
func (e SubscriptionActivated) OccurredAt() time.Time { return e.occurredAt }
```

---

## Camada 2: Application

**O que vai aqui:** orquestração de casos de uso. Coordena domain + infrastructure via interfaces.

### Use Cases (CQRS)

Regras:
- **Um arquivo** por use case
- **Commands (escrita)** e **Queries (leitura)** são structs **separadas**
- Único método `Execute`
- Dependências via construtor (DI)
- Sem acesso direto a banco, sem HTTP, sem SDK externo

Ver [[../../20-stacks/go/go-patterns#Use Case (CQRS)]] pro exemplo completo em Go.

### Mappers

Entidade → DTO. Funções puras, sem estado.

Regras:
- Entidade → DTO (**nunca** o contrário no mapper de application)
- **Nunca** expõe tipos internos do banco

```go
func ToSubscriptionDTO(s *entities.Subscription) *SubscriptionDTO {
    return &SubscriptionDTO{
        ID:        s.ID(),
        PlanID:    s.PlanID(),
        Status:    string(s.Status()),
        ExpiresAt: s.ExpiresAt().Format(time.RFC3339),
    }
}
```

---

## Camada 3: Infrastructure

**O que vai aqui:** implementações concretas. Banco, HTTP, providers externos.

### Repository Implementation

Regras:
- **Implementa** interface do domain
- Usa codegen (sqlc, Prisma, Drizzle) pra queries tipadas
- **Mapeia** tipos do banco → entidades de domínio (nunca expõe tipos gerados)
- Usa `DBFromContext` (ou equivalente) pra suporte a transações

```go
type SubscriptionRepository struct { pool *pgxpool.Pool }

var _ domain.ISubscriptionRepository = (*SubscriptionRepository)(nil) // compile-time check

func (r *SubscriptionRepository) FindByID(ctx context.Context, id string) (*entities.Subscription, error) {
    q := billingdb.New(database.DBFromContext(ctx, r.pool))
    row, err := q.GetSubscriptionByID(ctx, id)
    if err != nil {
        if errors.Is(err, pgx.ErrNoRows) {
            return nil, apperrors.ErrNotFound.WithMessage("assinatura não encontrada")
        }
        return nil, apperrors.NewInternal(err)
    }
    return mapRowToSubscription(row), nil
}

// Mapper banco → domínio (privado)
func mapRowToSubscription(row billingdb.Subscription) *entities.Subscription {
    return entities.ReconstructSubscription(
        row.ID, row.UserID, row.PlanID,
        entities.SubscriptionStatus(row.Status),
        row.ExpiresAt.Time,
    )
}
```

### HTTP Handler

Regras:
- Responsabilidade única: **parse → use case → serialize**
- Sem lógica de negócio
- Propaga erros via `ctx.SetError(err)` — **nunca** escreve resposta de erro diretamente
- Usa contratos abstratos (`HTTPRouter`, `HandlerFunc`), **não** framework cru

Ver [[../../20-stacks/go/go-patterns#Handler]] pro exemplo completo.

### Provider Externo

Regras:
- **Implementa** interface do domain
- **Isola o SDK externo** — domínio nunca conhece `stripe-go`, `mux-go`, etc.
- **Mapeia** tipos do SDK → tipos do domínio (ex: `stripe.Error` → `apperrors.ErrValidation`)

Ver [[../patterns/transaction-patterns#Provider externo]] e [[../../20-stacks/go/go-patterns#Provider externo]].

---

## Camada 4: Routes (HTTP)

**O que vai aqui:** registro de rotas, agrupamento, middleware específico do módulo.

Regras:
- Registra rotas no grupo do módulo
- Aplica middleware específico (ex: `authorize` pra ABAC)
- **Não instancia** use cases — recebe via DI
- Webhook **não passa** pelo middleware de auth (tem validação própria de HMAC)

```go
func RegisterSubscriptionRoutes(router *gin.RouterGroup, handler *SubscriptionHandler) {
    subscriptions := router.Group("/subscriptions")
    {
        subscriptions.POST("/activate", handler.Activate)
        subscriptions.GET("/me", handler.GetMySubscription)
        subscriptions.DELETE("/cancel", handler.Cancel)
    }
    // Webhook fora do auth
    router.POST("/webhooks/stripe", handler.HandleStripeWebhook)
}
```

---

## Ordem de implementação por feature

```
1. Migration SQL + CHECK constraints
   └── Numeração particionada por módulo

2. Query tipada / codegen (sqlc, Prisma, drizzle-kit)

3. Domain entity / value object

4. Repository interface (domínio)

5. Domain event (se cruzar bounded contexts)

6. Use case (CQRS: command ≠ query, arquivo separado)
   └── Compile-time interface assertion

7. Mapper (entidade → DTO)

8. Repository implementation (infraestrutura)
   └── Suporte a transação via context

9. Provider externo (SDK isolado)
   └── Interface no domain/providers, impl em infrastructure/providers

10. Handler + rota (contratos abstratos)

11. Wire-up no module + main

12. Testes (unidade + integração)
```

**Nunca** fora dessa ordem. Pular migration antes do código é o erro mais caro.

## Módulo como unidade de deploy

Cada bounded context é um **módulo autocontido**:

```
modules/<ctx>/
  domain/         entidades privadas, VOs, interfaces, eventos
  application/    use cases CQRS, mappers
  infrastructure/ database (codegen), http/routes, providers, jobs
  module.go       implementa IModule, registra rotas + use cases + jobs
```

**`main.go` só conhece `IModule` e `IUserSyncer`** — não conhece internals dos módulos.

Ver [[../patterns/_moc]] pro template de Module Pattern.

## Tenant Isolation (cross-cutting)

- Toda query multi-tenant inclui `WHERE tenant_id = $ctx.tenantID`
- ABAC rejeita request onde `tenant_id` do token ≠ recurso solicitado
- Cross-tenant **apenas** via grants explícitos (Connect Me, validações)

Ver [[../security/security-principles]] e [[../database/database-patterns]].

## Imutabilidade (regras críticas)

Tabelas imutáveis por design:
- `timeline_entries` — sem `deleted_at`, sem UPDATE após publicação. Adendo = novo registro.
- `audit_trail` — append-only. Retenção 5 anos (LGPD).
- Votos após homologação — imutáveis.
- Contratos fechados após dupla assinatura — `is_immutable=true`.
- Estado derivado — atualizado **apenas** via evento (ex: "Plan Status" só via `TimelinePublished`).

## Links

- [[_moc]]
- [[../patterns/transaction-patterns]] — ACID, Saga, locks
- [[../database/database-patterns]] — schema, indexação, soft delete
- [[../principles/do-dont-checklist]] — regras duras
- [[../principles/code-calisthenics]] — 9 regras
- [[../../20-stacks/go/go-patterns]] — exemplos em Go
- [[../../10-knowledge/methodology/sdd-workflow]] — ciclo 5 fases que produz essas camadas
