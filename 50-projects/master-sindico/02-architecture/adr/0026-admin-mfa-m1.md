---
title: ADR-0026 — Admin MFA obrigatório em M1 + step-up em endpoints críticos
type: adr
tags: [adr, security, mfa, admin, step-up, beyond-corp, master-sindico, v2]
status: accepted
date: 2026-04-23
created: 2026-04-23
updated: 2026-04-23
aliases: [admin-mfa-m1, step-up-mfa]
---

# ADR-0026 — Admin MFA obrigatório em M1 + step-up em endpoints críticos

## Status

accepted — 2026-04-23

## Contexto

Finding [[../../11-audit/audit-log/2026-04-23-pentest-specs#A-DC-PEN-007]] (pentester pass) reclassificou a spec original `IDN-007` ("admin MFA obrigatório a partir de M2") como 🔴 **M1 bloqueante**. Justificativa:

- Em M1, admin tem `admin_bypass` ([[../../01-domain/invariants#INV-088]]) que pula tenant check; uma credencial comprometida = plataforma inteira comprometida.
- [[../../08-security/BEYOND_CORP#Tenet 6]] já exige step-up em 6 ações críticas (`DELETE /me/data`, publish minutes, approve >R$10k, transferir síndico, mudar email/telefone, revogar device de outro device), mas a obrigatoriedade de MFA **continua** como S (M2) em [[../../04-requirements/functional/identity#IDN-007]] — inconsistente.
- Dual-check [[../../11-audit/dual-check-log/2026-04-23-security-gaps#Gap-SEC-002]] reclassificou parcialmente (M2→M1) os 3 endpoints LGPD/financeiro/legal.
- Zitadel suporta MFA TOTP nativo; custo ≈ 0.

Sem step-up, o findings A-DC-SEC-002 (IDOR `/me/devices/:id`), A-DC-PEN-019 (presigned Mux), A-DC-PEN-029 (`/me/export` como oracle) têm superfície explorável adicional.

## Decisão

**MFA obrigatório em M1** para:

1. **Todos os usuários com `role = admin`** (enforce via Zitadel project configuration `mfa_type = required`).
2. **Todos os `role_type ∈ {principal, administrator}` de empresas com plan_tier ∈ {pro, platinum, diamante}`** a partir do primeiro login pós-upgrade.

**Step-up MFA fresh (`≤ 5 min`) obrigatório em M1** para os seguintes endpoints, que emitem `401 step_up_required` se o header `X-Step-Up-Token` estiver ausente ou expirado:

| Endpoint | Justificativa |
|---|---|
| `DELETE /api/v1/me` | LGPD Art. 18 VI — auto-exploit trivial se sessão roubada |
| `DELETE /api/v1/me/devices/:id` | cobre [[../../11-audit/dual-check-log/2026-04-23-security-gaps#A-DC-SEC-001]] |
| `GET /api/v1/me/export` | cobre A-DC-PEN-029 (oracle LGPD) |
| `POST /api/v1/assemblies/:id/minutes/publish` | Lei 4.591/64 — nulidade de ata |
| `POST /api/v1/contracts/:id/approve` quando `value ≥ R$ 10.000` | fraude financeira |
| `POST /api/v1/condominiums/:id/transfer-mandate` | hijack institucional |
| `PATCH /api/v1/me` quando altera `email` ou `phone` | recovery hijack |
| `POST /admin/v1/*` (qualquer rota) | admin bypass é superpoder |

### Mecânica

- `POST /auth/step-up` emite JWT curto (5 min) com claim `amr=["mfa"]` + `acr=2` (OIDC padrão NIST SP 800-63-3 AAL2).
- Handler crítico exige `X-Step-Up-Token` + verify JWT (assinatura, expiração, `acr`) + verify denylist Redis `ms:stepup-denylist:<jti>`.
- Logout invalida step-up token ativo (INSERT em denylist).
- Rotação de sessão HTTP (ADR-0032 quando existir; por ora ver [[../../11-audit/dual-check-log/2026-04-23-security-gaps#A-DC-PEN-003]]) também revoga step-up.

### Reclassificação

- [[../../04-requirements/functional/identity#IDN-007]] atualizado: M (M1) para admin + empresa Pro/Platinum/Diamante; S (M2) para síndico N3 + morador opt-in.
- [[../../01-domain/invariants]]: INV-110 novo (endpoints críticos exigem step-up — ver seção `cross-domain (INV-101..INV-110)`).

## Consequências

### Positivas

- Fecha A-DC-PEN-007 (credential stuffing admin M1).
- Reduz superfície de A-DC-SEC-001 (IDOR device) + A-DC-PEN-019 (quota theft) com gate adicional.
- Alinha M1 com [[../../08-security/BEYOND_CORP#Tenet 6]].
- Zitadel nativo; zero custo extra.

### Negativas

- UX admin: mais fricção no login (TOTP obrigatório).
- Backup codes + recovery process precisam estar maduros pré-M1 (risco lockout operacional).
- Step-up handler adiciona latência (~50ms Redis denylist lookup).
- Precisa documentar flow `/auth/step-up` em [[../api-design]] §7 + OpenAPI.

## Regras

1. Admin sem MFA configurado **não pode** logar em M1 (Zitadel bloqueia no IdP).
2. Step-up token é válido **1 request** (burn após uso) OU TTL 5 min, o que vier primeiro.
3. Denylist Redis: `ms:stepup-denylist:<jti>` TTL = token_exp - now.
4. Audit trail ([[../../01-domain/invariants#INV-033]]) registra `step_up.emitted` + `step_up.consumed` + `step_up.rejected` para cada evento.
5. Rate limit específico em `/auth/step-up`: 5 rpm por `user_id` (mais restritivo que o default `/auth/*` 20 rpm).

## Alternativas consideradas

1. **Manter IDN-007 como M2** — rejeitado: custo zero de implementar M1 vs risco catastrófico.
2. **Passkeys WebAuthn em vez de TOTP** — ver [[../../11-audit/dual-check-log/2026-04-23-security-gaps#A-DC-SEC-014]]; adiado para M2 porque UX iOS Safari ainda inconsistente; TOTP cobre 95% dos casos.
3. **Step-up via SMS Twilio em vez de TOTP** — SMS é AAL1 (NIST SP 800-63B) — insuficiente para ações de alto valor.
4. **Aplicar step-up em TODAS as rotas mutating** — overkill; mata UX; não fecha mais attack surface do que a lista acima.

## Referências

- Finding: [[../../11-audit/audit-log/2026-04-23-pentest-specs#A-DC-PEN-007]] (M1 bloqueante admin MFA).
- Finding: [[../../11-audit/dual-check-log/2026-04-23-security-gaps#Gap-SEC-002]] (reclassificação partial M2→M1).
- Finding: [[../../11-audit/dual-check-log/2026-04-23-security-gaps#A-DC-SEC-001]] (IDOR `/me/devices`).
- [[../../08-security/BEYOND_CORP#1. Princípios — 7 tenets NIST SP 800-207 traduzidos]] — Tenet 6.
- [[../../04-requirements/functional/identity#IDN-007]] — requirement atualizado.
- [[../../01-domain/invariants#INV-110]] — novo invariante.
- [[0016-beyondcorp-security-posture]] — posture fundadora.
- [[0003-zitadel-oidc-provider]] — IdP que emite os tokens.
- NIST SP 800-63-3 AAL2: https://pages.nist.gov/800-63-3/sp800-63b.html#aal2
- Zitadel MFA docs: https://zitadel.com/docs/guides/solution-scenarios/multi-factor
