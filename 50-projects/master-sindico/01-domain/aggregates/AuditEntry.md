---
title: Aggregate — AuditEntry
type: spec
tags: [domain, ddd, aggregate, cross-domain, audit, lgpd, compliance, master-sindico, v2]
bc: cross-domain
source: 01-domain/invariants.md (INV-033, INV-088, INV-090, INV-091) + 04-requirements/compliance-lgpd.md + 04-requirements/functional/cross-domain.md (XD-010)
created: 2026-04-23
updated: 2026-04-23
---

# Aggregate — `AuditEntry`

**BC**: cross-domain · **Raiz**: ✅ (registro imutável de trilha LGPD/governança)

Entrada append-only da trilha de auditoria. Retenção 5 anos (LGPD), depois **hard-delete** (direito ao esquecimento — BR-010/INV-091). Usado por: admin bypass (INV-088), consentimento LGPD (Connect Me reveal INV-048), operações sensíveis (refund, impersonate, role change, delegation revoke AM5, ABAC decisões críticas).

Criado como fechamento do finding **A-DC-QA-040**.

## Entidade raiz

```go
type AuditEntry struct {
    id             AuditEntryID       // UUIDv7
    action         string             // canônico: "delegation.revoked", "admin.bypass", "lgpd.data_revealed", "billing.refund", "session.impersonate", ...
    actorUserID    *string            // user_id que executou (NULL se system/cron)
    actorRole      *string            // role no momento
    targetType     string             // tipo do alvo: "user" | "company" | "subscription" | "connect_me_request" | ...
    targetID       string             // id do alvo
    tenantID       *string            // condominium_id | company_id (escopo)
    outcome        string             // "success" | "failure" | "denied"
    reason         *string            // texto livre motivo (obrigatório em denials, admin_bypass, refund, revoke)
    metadata       AuditMetadata      // VO
    occurredAt     time.Time          // UTC (GR-033 — storage UTC, apresentação BRT)
    archivedAt     *time.Time         // quando movido pra cold storage (MinIO) após 1 ano (INV-033)
    expiresAt      time.Time          // occurredAt + 5 anos (retenção LGPD — INV-091)
}
```

## Value Objects

```go
type AuditMetadata struct {
    IP             string             // Masked pra cross-tenant logs (/24 só nos primeiros 3 octetos em telas públicas)
    UserAgent      string
    TraceID        string             // OTel
    RequestID      string
    PayloadBefore  map[string]any     // estado prévio (JSON) — sujeito a PII filter (INV-092)
    PayloadAfter   map[string]any     // estado novo (JSON) — idem
    ConsentVersion *string            // pra LGPD (versão do termo aceito)
    SourceService  string             // "api" | "webhook" | "cron" | "admin-console"
}

type AuditEntryID string  // UUIDv7
```

## Invariantes

- **INV-033**: Append-only — sem UPDATE, sem DELETE; arquivamento após 1 ano para MinIO (cold storage); schema sem `updated_at`
- **INV-088**: Toda decisão ABAC `admin_bypass` **sempre** cria AuditEntry
- **INV-090**: Trail append-only (cross-domain) — sem UPDATE/DELETE ao nível DB (trigger Postgres opcional; CI static check via grep)
- **INV-091**: Retenção 5 anos; post-expiry → hard-delete idempotente por job noturno (LGPD Art. 16 direito ao esquecimento)
- **INV-092**: PII masked em logs derivados (dashboard admin AD7 aplica masking); campos `payloadBefore`/`payloadAfter` podem ter PII em raw no DB, mas qualquer renderização pública pré-masking
- `actorUserID` NULL apenas quando `sourceService in {cron, webhook}` (ações de sistema)
- `reason` obrigatório quando `action in {admin.bypass, delegation.revoked, billing.refund, user.ban, content.video.remove.pre90d}`

## Factories

```go
// NewAuditEntry — ativamente emite a partir de qualquer handler/use case
func NewAuditEntry(action string, actor *string, target AuditTarget, outcome string, reason *string, md AuditMetadata) (*AuditEntry, error) {
    // valida: action != ""; target.Type/ID não vazios; reason obrigatório pras ações acima
}

func ReconstructAuditEntry(...) *AuditEntry
```

## Métodos de domínio

- `RequiresReason() bool` — retorna true se `action` está na lista que exige motivo
- `IsExpired(now time.Time) bool` — `NOW() >= expiresAt`
- `Archive(at time.Time)` — seta `archivedAt` (invocado pelo job de cold storage)
- `Masked() AuditEntry` — retorna cópia com PII masked (pra renderização em telas pública/AD7)

## Repository interface (Go)

```go
// domain/shared/audit_entry_repository.go
type IAuditEntryRepository interface {
    Save(ctx context.Context, e *AuditEntry) error                  // INSERT only
    FindByActorUserID(ctx context.Context, uid string, page PageArgs) ([]*AuditEntry, error)
    FindByTarget(ctx context.Context, targetType, targetID string, page PageArgs) ([]*AuditEntry, error)
    FindByAction(ctx context.Context, action string, page PageArgs) ([]*AuditEntry, error)
    FindExpired(ctx context.Context, limit int) ([]*AuditEntry, error)  // pro job de hard-delete LGPD
    DeleteExpired(ctx context.Context, ids []AuditEntryID) error        // único DELETE legal — só pós-5y (INV-091)
    Archive(ctx context.Context, cutoff time.Time) (archivedCount int, err error) // move pra MinIO
}
```

## Consumidores

Este aggregate é emitido por:
- [[AbacDecision]] quando `RequiresAudit() == true`
- [[Session]] (impersonate, 1-device revoke)
- [[../../03-subprojects/web/ui-catalog#am5]] (agency delegation revoke)
- [[Subscription]] (refund — BIL-027)
- [[HarassmentReport]] (status changes — INV-055)
- [[Video]] (remoção pre-90d — INV-057)
- [[User]] (ban/unban — IDN-024)
- Webhook handlers Stripe/Zitadel/Mux/Livekit (operações sensíveis)

## Eventos emitidos

Este aggregate **não emite** eventos próprios (observação: evita recursão infinita audit-of-audit). Mas `Save()` com sucesso pode disparar métricas Prometheus `audit_entries_total{action, outcome}`.

## Retenção e privacidade

- **Hot DB**: primeiro ano em Postgres (consultas rápidas AD7).
- **Cold storage**: 1-5 anos em MinIO (S3-compatible); buscável pelo admin via job async.
- **Hard-delete**: após 5 anos (LGPD); job noturno com SLO 15d (GR-030).
- **Consent**: LGPD logs de `lgpd.data_revealed` incluem `consentVersion` (versão do termo na hora do reveal — rastreabilidade R10).

## ⚠️ Pendências

- **P-AE-001**: confirmar se `payloadBefore`/`payloadAfter` é obrigatório ou opcional — legado sugere opcional em actions "triviais" mas legal-dept pede sempre pra admin actions.
- **P-AE-002**: definir lista canônica de `action` enum (aberta pro dev? fechada no código?).
- **P-AE-003**: LGPD Art. 16 — re-identificação pós-hard-delete via `payload` hash determinístico (A-DC-PEN-038): aplicar HMAC keyed rotating (ADR-0028 proposto).

## Links

- [[../bounded-contexts#7-cross-domain]]
- [[../invariants#cross-domain-inv-086-a-inv-100]] (INV-090, INV-091, INV-092)
- [[../../04-requirements/compliance-lgpd]]
- [[../../04-requirements/functional/cross-domain]] (XD-010, XD-011)
- [[AbacDecision]] — consumidor direto
- [[OutboxEntry]] — padrão análogo de durabilidade
- [[../../03-subprojects/web/ui-catalog#7-admin-master-síndico]] AD7 LGPD console
