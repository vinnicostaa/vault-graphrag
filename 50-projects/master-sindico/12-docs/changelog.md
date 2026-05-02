---
title: Changelog — Master Síndico v2
type: guide
tags: [changelog, releases, master-sindico, v2]
source: seed 2026-04-23 (remontagem)
created: 2026-04-23
updated: 2026-04-23
---

# Changelog

Todas as mudanças notáveis de release do Master Síndico são documentadas aqui.

Formato: [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).

**Status**: seed — arquivo nasce zerado na remontagem v2; populado por deploy (M1, M2, M3, hotfixes) conforme acontecem.

---

## [Unreleased]

### Added
- Remontagem arquitetural v2 em curso (pasta `master-sindico-v2/`)
- 7 sub-produtos catalogados em [[../00-product/portfolio-de-produtos]]
- 6 bounded contexts canonizados em [[../01-domain/bounded-contexts]]
- Backlog consolidado com MoSCoW + T-### estáveis em [[../05-tasks/backlog]]
- Execução formalizada: STEPS + dev + qa + deploy + rollback + incident + playbooks em [[../06-execution/_moc]]
- Cronograma macro com marcos firmes M1/M2/M3 em [[../10-schedule/cronograma-macro]]
- 11-audit espelho canônico + audit-log + dual-check-log + postmortems
- 12-docs (este): README + adr-index + changelog (seed) + how-to + runbooks

### Pending
- M1 deploy 2026-05-08 (Sprint 10 finalizando)
- Fechar D-006..D-015 via research-loop + dual-check
- Formalizar ADR-0001..ADR-0027

---

## [1.0.0] — 2026-05-08 (M1 — MVP Contratual, **PLANEJADO**)

> Placeholder — será preenchido quando o deploy M1 for executado.

### Added (escopo planejado)

- **Identity** (BC `identity`):
  - Autenticação via Zitadel OIDC Authorization Code + PKCE
  - Sessões com 1-device policy + device fingerprint
  - Cookie httpOnly em `.mastersindico.com.br` (wildcard subdomínios)
  - Fallback Bearer mobile (RFC 6750)
  - ABAC engine com matriz (admin_bypass → role → action → tenant)
  - Anti-fraude: rate limit `/auth/*` 20rpm burst 5; bloqueio múltiplos IPs (10 signups/1h → 24h ban)
- **Billing** (BC `billing`):
  - 4 tiers: Trial / Base / Plus / Pro
  - Trial persona-aware (síndico 15d, empresa 7d, parceiro 30d, morador —)
  - Stripe recorrência live mode
  - Quotas anuais (Connect Me) e mensais (vídeos) por plano
  - Soft-block sem grace period
  - Webhooks idempotentes (`event.id` unique ledger)
  - Customer Portal Stripe
- **Institutional** (BC `institutional`):
  - Condomínio (tenant primário) com `public_id` 8 chars + QR
  - Unidades com fração ideal (NUMERIC 4 casas, Σ ≤ 100%)
  - Memberships (user ↔ condomínio ↔ unit ↔ role)
  - Timeline imutável (append-only, sem UPDATE/DELETE)
  - Plano Diretor base (26 áreas × 7 status via eventos)
  - Limite 15 condomínios por síndico
  - Compliance termo + audit trail 5 anos
- **LGPD base**:
  - Consent registry versionado
  - `GET /me/export` (portabilidade SLA 15d)
  - `DELETE /me` (direito de exclusão SLA 15d)
  - Runbook 72h breach notification → ANPD
- **Web** (SolidJS): onboarding síndico + dashboard + timeline + plano diretor
- **Mobile** (Flutter): auth OIDC + home síndico/morador
- **Observability**: OTel traces + Sentry prod + Grafana dashboards
- **Segurança**: CORS allowlist strict (Config.Validate bloqueia wildcard em prod), PII masking em logs

### Fixed (findings resolvidos pré-M1)

- A-020: N+1 UPDATE em `master_plan_timeline_handler`
- A-021: 12 `SELECT *` em queries commercial (substituídos por colunas explícitas)
- A-023: 1-device change com audit log comparativo (`old_fp` + `new_fp` + diff)
- A-024: `rawBodyReader` elevado ao contracts/core (se Sprint 10 chegar)

---

## [2.0.0] — 2026-06-20 (M2 — Connect Me + Conteúdo, **PLANEJADO**)

> Placeholder.

Escopo em [[../10-schedule/milestones#m2-connect-me--conteúdo]].

---

## [3.0.0] — 2026-07-20 (M3 — Assembleia + LMS + Polish, **PLANEJADO**)

> Placeholder.

Escopo em [[../10-schedule/milestones#m3-assembleia-live--lms--polish]].

---

## Versionamento

- **Major** (N.0.0): marco contratual (M1 → 1.0, M2 → 2.0, M3 → 3.0)
- **Minor** (N.M.0): feature release entre marcos
- **Patch** (N.M.P): bug fix / security patch

---

## Links

- [[../ROADMAP]]
- [[../10-schedule/milestones]]
- [[../05-tasks/backlog]]
- [[README]]
- [[../11-audit/AUDIT]]
