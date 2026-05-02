---
title: Global Requirements — Master Síndico
type: project
tags: [master-sindico, requirements, global, cross-cutting]
project: master-sindico
created: 2026-04-22
---

# Global Requirements — Master Síndico

Requisitos **transversais** — aplicam-se a **todos** os módulos. Requisitos específicos por módulo em [[functional/_moc]] (*a criar*).

---

## GR-001 — Autenticação OIDC via Zitadel

Toda request autenticada usa OIDC Authorization Code + PKCE via Zitadel.

- Cookie httpOnly `session` para web (domain `.mastersindico.com.br`)
- Authorization: Bearer <token> para mobile
- Token introspection em middleware `Authenticate`
- Refresh automático em middleware antes de expirar

## GR-002 — Autorização via ABAC

Toda rota autenticada passa por middleware ABAC antes do handler.

- Decisão em ordem: admin_bypass → role_check → action_check → tenant_check → scope_check
- Cache Redis 5min por `{user_id, tenant_id}`
- Invalidação via webhook Stripe `customer.subscription.*`

## GR-003 — Tenant Isolation

Toda query em recurso escopado a condomínio inclui `WHERE condominium_id = $tenant_id`.

## GR-004 — 1-device policy

User tem no máximo `MAX_ACTIVE_SESSIONS` sessões ativas (default 1).

## GR-005 — Request ID propagation

Toda request tem `X-Request-ID` (gerado se ausente). Propagado em logs, traces e response headers.

## GR-006 — Rate limiting

- `/auth/*`: 20 rpm, burst 5
- `/api/v1/*`: 60 rpm, burst 10
- `/webhooks/*`: sem rate limit interno

## GR-007 — PII nunca em logs

CPF, CNPJ, email, token, password, PAN: proibido. `Masked()` ou `user_id`.

## GR-008 — Webhook signature antes de parse

Stripe, Mux, Livekit: HMAC verify antes de `json.Unmarshal`.

## GR-009 — Idempotência de webhook

`event.id` único em idempotency ledger; processamento defensivo em reentrega.

## GR-010 — Internacionalização (i18n)

Pt-BR é língua primária. Frontend preparado para EN (M3+).

- Backend: mensagens de erro em pt-BR (user-facing); logs em inglês
- Frontend: i18n keys com fallback pt-BR

## GR-011 — Observability

- Logs estruturados com `request_id`, `tenant_id`, `user_id`
- Traces OpenTelemetry (spans em cada use case + query)
- Metrics: counters de domain events + histograms de latência por endpoint
- Errors: Sentry (frontend + backend + mobile)

## GR-012 — LGPD compliance

- Consentimento explícito no cadastro
- Direito de exclusão (SLA 15d)
- Direito de portabilidade (export JSON)
- DPO registrado no footer
- Audit trail em operações sensíveis (DT-010)

## GR-013 — Accessibility (WCAG AA)

Frontend (web e mobile): WCAG 2.1 AA mínimo.

## GR-014 — API contract versioning

- `/api/v1` é o contrato canônico
- Breaking change → `/api/v2` com deprecation em `/api/v1` por 6 meses
- Non-breaking: adicionar campos optional; nunca remover

## GR-015 — Error responses consistentes

```json
{
  "code": "validation_error",
  "message": "starts_at obrigatório",
  "fields": [{"path": "starts_at", "rule": "required"}]
}
```

Códigos: `validation_error` (422), `not_found` (404), `conflict` (409), `forbidden` (403), `unauthorized` (401), `payment_required` (402), `quota_exhausted` (403), `internal_error` (500).

## GR-016 — Pagination cursor-based

Toda listagem suporta `?limit=20&cursor=<opaque>`; retorna `{items[], next_cursor?}`.

## GR-017 — Timezone

- Backend armazena UTC
- Frontend exibe em `America/Sao_Paulo`
- Datas em API: ISO 8601 com timezone

## GR-018 — File size limits

- Body request: 10MB (exceto upload direto Mux)
- Avatar: 5MB
- PDF edital/ata: 20MB

## GR-019 — Upload direto para provider externo

Uploads grandes (vídeo, PDF ata) usam presigned URL (Mux direct upload, S3 presigned). Backend **nunca** proxy.

## GR-020 — CORS strict

Allowlist em `CORS_ALLOWED_ORIGINS`. Wildcard `*` bloqueado em staging/prod.

---

## Fontes

- `backend/.kiro/specs/master-sindico/requirements/cross-domain.md`
- `backend/.kiro/specs/master-sindico/requirements/functional-matrix.md`
- [[../08-security/BEYOND_CORP]]
- [[../CLAUDE]]

## Links

- [[_moc]]
- [[functional/_moc]]
- [[non-functional/_moc]]
- [[compliance]]
- [[../01-domain/business-rules]]
- [[../08-security/_moc]]
