---
title: IValidatable — interface cross-BC para hub de validações
type: pattern
tags: [master-sindico, domain, pattern, ddd, cross-bc, validation]
project: master-sindico
created: 2026-04-24
aliases: [IValidatable, ValidationItem interface, validation hub pattern]
---

# IValidatable — interface cross-BC para hub de validações

**Origem**: [[../../STATE#D-105 — ValidationItem como conceito (Opção B DDD puro)|D-105]] (2026-04-24).

## Problema

O hub "Validações Pendentes" (telas [[../../03-subprojects/web/ui-catalog/sindico/S21]], [[../../03-subprojects/web/ui-catalog/sindico/S22]], [[../../03-subprojects/web/ui-catalog/sindico/S26]]) agrega **5 tipos** de itens que o síndico valida:

1. **ServiceRecord** (commercial BC) — registro de execução de serviço pela empresa após Connect Me fechado
2. **TechnicalActivity** (commercial BC) — atividade técnica com classificação de risco/natureza
3. **PostDealAnnouncement** (commercial BC) — comunicado pós-fechamento de negócio
4. **Proposal** (commercial BC) — proposta comercial
5. **Adjustment** (commercial BC) — ajuste/adendo a item já publicado

Todos precisam da **mesma ação** (validar, aprovar, rejeitar, solicitar ajuste) feita pelo **mesmo ator** (síndico), com **mesmo audit trail** e **mesma notificação**, mas cada um tem **invariantes distintos** no domínio.

## Decisão de design

**Opção B (DDD puro)** — `ValidationItem` é **conceito/superset semântico**, não aggregate. Cada item validável é aggregate próprio no seu BC. Cada aggregate implementa a interface `IValidatable`.

Hub consome via:
1. **View materializada** `pending_validations` (UNION ALL das 5 tabelas) OU
2. **Endpoint agregador** `GET /api/v1/institutional/validation-items?status=pending&condominium_id=X` que retorna DTOs uniformes

## Interface

```go
// internal/shared/types/validatable.go
package types

import (
    "context"
    "time"
)

type ValidationType string

const (
    ValidationTypeServiceRecord       ValidationType = "service_record"
    ValidationTypeTechnicalActivity   ValidationType = "technical_activity"
    ValidationTypePostDealAnnouncement ValidationType = "post_deal_announcement"
    ValidationTypeProposal             ValidationType = "proposal"
    ValidationTypeAdjustment           ValidationType = "adjustment"
)

type ValidationStatus string

const (
    ValidationPending            ValidationStatus = "pending"
    ValidationApproved           ValidationStatus = "approved"
    ValidationRejected           ValidationStatus = "rejected"
    ValidationAdjustmentRequested ValidationStatus = "adjustment_requested"
)

type ValidateAction string

const (
    ValidateActionApprove            ValidateAction = "approve"
    ValidateActionReject             ValidateAction = "reject"
    ValidateActionRequestAdjustment  ValidateAction = "request_adjustment"
)

// IValidatable é implementada por todo aggregate que aparece no hub.
type IValidatable interface {
    // Identidade
    ValidationItemID() string          // ID globalmente único do item
    ValidationItemType() ValidationType
    CondominiumID() string
    
    // Submissão
    SubmittedByUserID() string
    SubmittedAt() time.Time
    
    // Estado
    ValidationStatus() ValidationStatus
    
    // Comportamento de domínio
    Validate(ctx context.Context, actor ValidatorActor, action ValidateAction, reason string) error
}

// ValidatorActor é o contexto de quem está validando (quase sempre Síndico do condomínio).
type ValidatorActor struct {
    UserID        string
    Role          string  // deve ser "syndic" ou "admin"
    CondominiumID string  // deve ser igual ao do item (tenant isolation)
}
```

## Contratos de `Validate`

- **Preconditions**:
  - `actor.Role ∈ {syndic, admin}` — senão `ErrForbidden`
  - `actor.CondominiumID == item.CondominiumID` — senão `ErrTenantMismatch` (tenant isolation)
  - `item.Status == Pending` — senão `ErrBusiness("already validated")`
  - `action == RequestAdjustment` implica `reason != ""` — senão `ErrValidation("reason required")`

- **Postconditions**:
  - `action == Approve` → `status = Approved`; publica `<Type>Approved` event (ex: `ServiceRecordApproved`)
  - `action == Reject` → `status = Rejected`; publica `<Type>Rejected` com reason
  - `action == RequestAdjustment` → `status = AdjustmentRequested`; publica `<Type>AdjustmentRequested` com reason
  - Append-only em `audit_log_entries` com `(actor.UserID, item_id, action, reason, timestamp)`
  - Se status transita para Approved: integrações específicas (ex: ServiceRecord approved → publica em timeline imutável)

- **Idempotência**: duas chamadas consecutivas com mesmo actor + item + action retornam erro na segunda (`ErrBusiness("already validated")`). Não é replay-safe; chamador usa `If-Match` header com versão.

## Implementadores (aggregates)

### Proposal (✅ existe)

Ver [[../aggregates/Proposal]]. Implementa `IValidatable.Validate(...)` com invariantes:
- Aprovação cria `ClosedDeal` candidato (outra aggregate)
- Rejeição marca proposal como encerrada; permite envio de nova

### ServiceRecord (🟡 stub)

Ver [[../aggregates/ServiceRecord]] (a criar). Implementa `Validate(...)` com:
- Aprovação publica `TimelineEntry` imutável (via `ITimelinePublisher`)
- Ajuste solicitado permite empresa reenviar com versão nova

### TechnicalActivity (🟡 stub)

Ver [[../aggregates/TechnicalActivity]] (a criar). Similar a ServiceRecord mas sem trigger de timeline (fica em histórico técnico da empresa).

### PostDealAnnouncement (🟡 a avaliar)

Ver [[../aggregates/Announcement]] (existente) — avaliar se cobre caso pós-deal ou se precisa aggregate separado. Spec em D-113 (a abrir).

### Adjustment (🟡 stub)

Ver [[../aggregates/Adjustment]] (a criar). É "emenda" a item publicado. Invariante crítico: adjustment nunca altera o original (append-only).

## View materializada (backend)

```sql
-- pending_validations: UNION ALL agregando os 5 tipos
CREATE MATERIALIZED VIEW pending_validations AS
SELECT
    sr.id AS item_id,
    'service_record' AS item_type,
    sr.condominium_id,
    sr.submitted_by_user_id,
    sr.submitted_at,
    sr.status,
    sr.title AS preview_title,
    sr.company_id AS preview_company_id
FROM service_records sr
WHERE sr.status = 'pending' AND sr.deleted_at IS NULL

UNION ALL

SELECT
    ta.id, 'technical_activity', ta.condominium_id, ta.submitted_by_user_id,
    ta.submitted_at, ta.status, ta.title, ta.company_id
FROM technical_activities ta
WHERE ta.status = 'pending' AND ta.deleted_at IS NULL

UNION ALL

-- idem para post_deal_announcements, proposals, adjustments
;

-- Refresh: via trigger em cada INSERT/UPDATE nas 5 tabelas OU REFRESH CONCURRENTLY via cron 1min
CREATE UNIQUE INDEX pending_validations_pk ON pending_validations (item_id, item_type);
CREATE INDEX pending_validations_condo ON pending_validations (condominium_id) WHERE status = 'pending';
```

**Alternativa à view materializada**: endpoint em Go faz 5 queries SELECT concorrentes + merge + paginação. Trade-off: view é mais rápida mas requer refresh; queries paralelas não têm staleness mas são 5 round-trips. Para M1, view materializada com refresh 1min é suficiente.

## Endpoint hub

```
GET /api/v1/institutional/validation-items
  ?condominium_id=<uuid>
  &status=pending|approved|rejected|adjustment_requested
  &type=service_record|technical_activity|...|all
  &limit=50&offset=0
  
Retorno:
{
  "items": [
    {
      "item_id": "...",
      "item_type": "service_record",
      "condominium_id": "...",
      "submitted_by": {"user_id": "...", "name": "..."},
      "submitted_at": "2026-04-24T10:00:00Z",
      "status": "pending",
      "preview": {
        "title": "Manutenção hidráulica ap 301",
        "company": {"id": "...", "name": "HidroCorp"},
        "...": "campos típicos do tipo"
      },
      "links": {
        "detail": "/api/v1/commercial/service-records/<id>"
      }
    }
  ],
  "total": 23,
  "page": 1
}
```

ABAC: só `syndic` OR `admin` do condomínio vê. Tenant isolation: `condominium_id` vem do claim do token; se diferir do query, 403.

## Ações do síndico (PATCH)

```
PATCH /api/v1/commercial/service-records/<id>
  body: { "action": "approve|reject|request_adjustment", "reason": "..." }
```

Ou ação genérica cross-type:
```
PATCH /api/v1/institutional/validation-items/<item_type>/<id>
  body: { "action": "...", "reason": "..." }
```

A variante cross-type é **conveniente para o hub** mas rota por tipo é mais RESTful. Preferir rota por tipo; hub sabe o tipo do item via `pending_validations.item_type` e monta URL correta.

## Trade-offs vs. Opção A (aggregate único polimórfico)

| Aspecto | Opção B (esta) | Opção A (aggregate único) |
|---|---|---|
| Fidelidade DDD | ✅ Aggregate SRP; invariantes encapsulados | ❌ Aggregate "inchado" com type discriminator |
| Teste | ✅ Cada aggregate testável isoladamente | 🟡 Tests precisam cobrir N variantes |
| UI hub | 🟡 View materializada OU 5 queries merged | ✅ 1 query simples |
| Código | 🟡 5 implementações da interface | ✅ 1 implementação |
| Evolução | ✅ Adicionar 6º tipo sem tocar os outros | 🟡 Adicionar coluna na tabela única |
| Performance | ✅ Queries scoped por tipo; indexes focados | 🟡 Tabela única cresce mais rápido |
| Manutenção | ✅ Mudanças isoladas | ❌ Mudança em 1 tipo pode quebrar outros |

**Decisão do usuário (D-105)**: Opção B "facilita manutenção e estabilidade".

## Quando mudar para Opção A

Gatilho: se aparecerem **2+ novos tipos validáveis em 1 sprint** (ex: validação de voto, validação de adendo de assembleia, validação de promoção) e o padrão estabilizar, considerar migração para aggregate único via [[../../02-architecture/adr/0038-validatable-interface-cross-bc]]. Não é urgente.

## ADR relacionada

[[../../02-architecture/adr/0038-validatable-interface-cross-bc]] (a escrever) formalizará a escolha Opção B com alternativas rejeitadas.

## Links

- [[_moc]] *(a criar)* — MOC de patterns
- [[../../STATE#D-105 — ValidationItem como conceito (Opção B DDD puro)|D-105]]
- [[../aggregates/Proposal]]
- [[../aggregates/Announcement]]
- [[../aggregates/_moc]]
- [[../../03-subprojects/web/ui-catalog/sindico/S21]]
- [[../../03-subprojects/web/ui-catalog/sindico/S22]]
- [[../../03-subprojects/web/ui-catalog/sindico/S26]]
- [[../../03-subprojects/backend/requirements/cross-domain]]
- [[../../11-audit/audit-log/2026-04-24-gap-analysis-dominio]]
