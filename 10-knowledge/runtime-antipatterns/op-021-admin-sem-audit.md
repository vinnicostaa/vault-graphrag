---
title: OP-021 — Ação de admin sem audit trail
type: antipattern
tags:
  - antipattern
  - runtime
  - operational
  - observability
  - audit
  - compliance
id: OP-021
severity: High
created: 2026-04-24
updated: 2026-04-24
doc-consulted: 2026-04-24
aliases:
  - Admin bypass unlogged
---

# OP-021 — Ação de admin sem audit trail

**Severidade**: High (compliance: LGPD/GDPR/SOX)
**Categoria**: Observabilidade / Compliance

## Sintoma

Código bypassando permissão (`if role === 'admin' allow all`) sem gravar quem, o quê, quando, por quê. Sem trilha, não há responsabilização, não há investigação de incidente, não há relatório regulatório.

## Por que é problema

- **Compliance quebrado**: LGPD (art. 37), SOX, HIPAA, PCI-DSS exigem trilha de acesso/modificação a dados sensíveis.
- **Incident response cego**: "quem aprovou este reembolso?" / "quem deletou este usuário?" — resposta impossível.
- **Insider threat invisível**: admin malicioso age sem deixar rastro.

## Exemplo (anti-pattern)

```go
// ❌ ERRADO — bypass silencioso
func (h *UserHandler) Delete(ctx contracts.Context, userID string) error {
    actor := auth.Actor(ctx)
    if actor.Role == "admin" {
        return h.repo.HardDelete(ctx, userID)   // sem log, sem audit
    }
    // ...
}
```

## Fix preferido — toda bypass gera entrada em `audit_trail`

```sql
CREATE TABLE audit_trail (
    id            UUID PRIMARY KEY,
    actor_id      TEXT NOT NULL,
    actor_role    TEXT NOT NULL,
    action        TEXT NOT NULL,      -- 'user.hard_delete', 'billing.waive_fee'
    resource_type TEXT NOT NULL,
    resource_id   TEXT NOT NULL,
    tenant_id     TEXT,
    reason        TEXT,
    metadata      JSONB NOT NULL DEFAULT '{}',
    ip_address    INET,
    user_agent    TEXT,
    request_id    TEXT,
    occurred_at   TIMESTAMPTZ NOT NULL DEFAULT now()
);

-- Trigger bloqueia UPDATE/DELETE — tabela append-only
CREATE RULE audit_trail_no_update AS ON UPDATE TO audit_trail DO INSTEAD NOTHING;
CREATE RULE audit_trail_no_delete AS ON DELETE TO audit_trail DO INSTEAD NOTHING;
```

```go
func (h *UserHandler) Delete(ctx contracts.Context, userID string) error {
    actor := auth.Actor(ctx)
    if actor.Role == "admin" {
        if err := h.repo.HardDelete(ctx, userID); err != nil { return err }
        return h.audit.Record(ctx, AuditEntry{
            ActorID: actor.ID, ActorRole: actor.Role,
            Action:  "user.hard_delete",
            ResourceType: "user", ResourceID: userID,
            Reason:  ctx.GetHeader("X-Admin-Reason"),
            IP:      ctx.ClientIP(),
            RequestID: ctx.RequestID(),
        })
    }
    // ...
}
```

## Alternativa

- **Middleware de audit**: intercepta ações sensíveis automaticamente via annotation/decorator + tabela de ações auditáveis.
- **Domain Events** de ações críticas publicados em bus → worker grava em audit_trail + espelha em SIEM (Datadog, Splunk).
- **Obrigatoriedade de `reason`** em endpoints admin (header `X-Admin-Reason` ou campo no body).

## Quando tolerar

Ações triviais sem implicação regulatória (ex.: admin visualiza dashboard público). Critério: se a ação **muda estado** ou **revela PII**, auditar.

## Relações

- **Patterns**: [[../patterns/decorator]] (audit via middleware), Event Sourcing (audit trail intrínseco)
- **OPs relacionados**: [[op-008-autorizacao-apenas-cache-hit]] (decisão de autz), [[op-020-erro-engolido]], [[op-022-log-com-pii]]

## Fontes

- [[../../40-templates/AGENTS_SPEC]] §4 OP-021, §3.20 (Segurança)
- LGPD — [Lei 13.709/2018](https://www.planalto.gov.br/ccivil_03/_ato2015-2018/2018/lei/l13709.htm) (consultada 2026-04-24)
- NIST — [SP 800-92 Log Management](https://csrc.nist.gov/publications/detail/sp/800-92/final) (consultada 2026-04-24)

## Links

- [[_moc]]
- [[../observability/audit-trail]]
- [[../security/security-principles]]
- [[op-022-log-com-pii]]
