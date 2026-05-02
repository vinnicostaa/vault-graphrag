---
title: Tasks — BC identity
type: spec
tags: [tasks, by-module, identity, master-sindico, v2]
source: 04-requirements/functional/identity.md + AUDIT A-023 + backlog.md
created: 2026-04-23
updated: 2026-04-23
---

# Tasks — BC `identity`

Tasks do bounded context **identity** (Zitadel OIDC + PKCE, 1-device, ABAC claims, sessões, mirror pattern user local↔Zitadel).

Alinhamento: [[../../04-requirements/functional/identity]] (IDN-001..IDN-013).

---

## M1 — MVP contratual (2026-05-08)

### Finalização frontend + mobile

| ID | Título | Sprint | Status |
|---|---|---|---|
| T-001 | Onboarding síndico — 4 passos (plano, condo, unidades, convites) | 10 | ⏳ |
| T-002 | Mobile auth OIDC Bearer + device fingerprint coleta | 10 | ⏳ |
| T-002c | Mobile device fingerprint enviar no signup/login | 10 | ⏳ |

### AUDIT findings

| ID | Título | Sprint | Status |
|---|---|---|---|
| T-004 | A-023 — 1-device change com audit log comparativo (`old_fp`+`new_fp`+diff) | 10 | ⏳ |

### Anti-fraude

| ID | Título | Sprint | Status |
|---|---|---|---|
| T-600 | IDN-012 — Anti-fraude múltiplos IPs (10 cadastros/1h bloqueia 24h) | 10 | ⏳ |
| T-601 | IDN-013 — Audit log tentativas auth (invalid_creds/mfa_failed/device_mismatch) | 10 | ⏳ |
| T-602 | IDN-011 — Rate limit `/auth/*` (20rpm burst 5) validado em prod | 10 | ⏳ |

## M2+ evolução

| ID | Título | Sprint | Status |
|---|---|---|---|
| T-650 | Cache Redis introspection TTL 5min com invalidação via webhook Zitadel (fechar D-001 legado) | 11-12 | ⏳ |
| T-651 | Role-type sub-contexto (`principal|auxiliary|assistant`) UI gestão usuários condomínio | 14 | ⏳ |

---

## Invariantes cobertos

I-001..I-005 de [[../../01-domain/bounded-contexts#identity]]:
- I-001 `users.zitadel_id` UNIQUE
- I-002 1 CPF = 1 User; 1 CNPJ = 1 User
- I-003 ≤ 1 Session ativa
- I-004 role nunca string vazia (guard clause `NO_ROLE_ASSIGNED`)
- I-005 `sessions.zitadel_session_id` não reutilizada após revogação

---

## Links

- [[_moc]]
- [[../../04-requirements/functional/identity]]
- [[../../01-domain/bounded-contexts#identity]]
- [[../../08-security/BEYOND_CORP]]
- [[../backlog]]
