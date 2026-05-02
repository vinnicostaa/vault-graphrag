---
title: Sprint 10 — Finalização M1
type: spec
tags: [tasks, sprint, sprint-10, m1, master-sindico, v2]
source: 90-ingested/from-vault-50-projects-master-sindico/05-tasks + AUDIT A-020..A-034 + backlog.md
created: 2026-04-23
updated: 2026-04-23
---

# Sprint 10 — Finalização M1

Janela: **2026-04-22 → 2026-05-15** (24 dias úteis, buffer 2 pra deploy 2026-05-18 — M1 firme após D-106).

Meta: **fechar M1 (MVP contratual)** = Identity + Billing + Institutional base + frontend web + mobile minimal + AUDIT 🔴🟡 = 0 pra todos os 🔴; 🟡 que ficar deve ter plano documentado.

> Sprint **herdada do legado**: backend concluído Sprints 0-9. Finalização = frontend web, mobile shell, findings A-020..A-024, polish, deploy prep.

---

## Tasks

### Frontend web (SolidJS)

| ID | Título | Estimate | Dono | Status |
|---|---|---|---|---|
| T-001 | Onboarding síndico — 4 passos (plano, condomínio, unidades, convite moradores) | 3d | web | ⏳ em curso |
| T-001a | TrialBlocker component — bloqueia feature premium se trial não-aderente | 1d | web | ⏳ |
| T-001b | Dashboard condomínio — 13 cards (INT-025) | 2d | web | ⏳ |
| T-001c | Timeline view — append-only, filtro 7 tipos, adendo | 2d | web | ⏳ |
| T-001d | Plano Diretor view — 26 áreas × 7 status | 2d | web | ⏳ |
| T-001e | Seletor de contexto (síndico 2+ condos) persistido em cookie sessão | 0.5d | web | ⏳ |

### Mobile (Flutter)

| ID | Título | Estimate | Dono | Status |
|---|---|---|---|---|
| T-002 | Auth OIDC fluxo mobile Bearer | 2d | mobile | ⏳ |
| T-002a | Home shell morador (timeline leitura) | 2d | mobile | ⏳ |
| T-002b | Home shell síndico (timeline + dashboard minimal) | 1d | mobile | ⏳ |
| T-002c | Device fingerprint coletar + enviar | 0.5d | mobile | ⏳ |

### AUDIT findings abertos (resolver antes do deploy M1)

| ID | Título | Estimate | Dono | Status |
|---|---|---|---|---|
| T-003 | A-020 — Fix N+1 UPDATE em `master_plan_timeline_handler` (fetch em batch) | 0.5d | backend | ⏳ |
| T-004 | A-023 — Device change audit log com `old_fp` + `new_fp` + diff | 0.5d | backend | ⏳ |
| T-005 | A-021 — Documentar anti-pattern `SELECT *` em `07-quality/PATTERNS.md` + grep CI | 0.25d | backend | ⏳ |
| T-005a | A-021 — Substituir 12 `SELECT *` por colunas explícitas em sqlc | 1d | backend | ⏳ |
| T-005b | A-024 — Elevar `rawBodyReader` ao contracts/core | 0.5d | backend | ⏳ |

Nota: A-029..A-034 ficam para Sprints 11+ (M2/M3), pois são bugs em BCs comerciais/assembly cujo frontend entra pós-M1.

### LGPD + Compliance M1

| ID | Título | Estimate | Dono | Status |
|---|---|---|---|---|
| T-006 | Consent registry versionado + UI | 1d | backend+web | ⏳ |
| T-006a | `GET /me/export` (dados do user em JSON+ZIP) — SLA 15d | 1d | backend | ⏳ |
| T-006b | `DELETE /me` (soft-delete + audit + notify DPO) SLA 15d | 1d | backend | ⏳ |
| T-006c | DPO canal + runbook 72h breach notification | 0.5d | ops | ⏳ |

### Deploy prep

| ID | Título | Estimate | Dono | Status |
|---|---|---|---|---|
| T-700 | OTel traces cobrindo 80% endpoints críticos | 1d | backend | ⏳ (Sprint 7 ✅ base) |
| T-701 | Sentry prod em 3 sub-projetos | 0.5d | ops | ⏳ |
| T-701a | Smoke test Playwright (5 fluxos críticos) | 1d | web | ⏳ |
| T-702 | Runbook rollback-deploy escrito + dry-run | 0.5d | ops | ⏳ |
| T-702a | Runbook incident-lgpd-breach escrito | 0.5d | ops | ⏳ |
| T-702b | Runbook webhook-reprocessing (Stripe/Mux) escrito | 0.5d | ops | ⏳ |
| T-708 | `Config.Validate()` bloqueia CORS wildcard em staging/prod | 0.25d | backend | ⏳ |

### Infraestrutura M1

| ID | Título | Estimate | Dono | Status |
|---|---|---|---|---|
| T-042 | (ex-Kiro 9.19) Zitadel Railway + DNS produção (`auth.mastersindico.com.br`) | 2d | ops | ⏳ |
| T-043 | Stripe live mode + webhook prod configurado | 0.5d | ops | ⏳ |
| T-044 | Sessão `.mastersindico.com.br` wildcard cookie validado em prod | 0.25d | ops+backend | ⏳ |

### Hardening pré-M1 (D-119..D-125 — systematic review 2026-04-24)

| ID | Título | Estimate | Dono | Status | Origem |
|---|---|---|---|---|---|
| T-OUTBOX-01 | Tabela `domain_events` (outbox pattern) + worker poller a cada 1s; substitui `InMemoryPublisher` em billing+institutional+commercial | 2d | backend | ⏳ | Systematic P0-A1 |
| T-WEBHOOK-01 | Tabela `webhook_events` UNIQUE `(provider, event_id)` + idempotency runtime | 1d | backend | ⏳ | ADR-0027 / P0-A4 |
| T-WEBHOOK-02 | Anti-replay HMAC: reduzir janela 5min → 2min (Stripe + Mux + LiveKit + Twilio) | 0.5d | backend | ⏳ | P0-A5 / A-DC-PEN-017 |
| T-COOKIE-01 | Cookie session `Domain=app.mastersindico.com.br` (sem ponto inicial — wildcard subdomain proibido) | 0.5d | backend | ⏳ | P0-A6 / A-DC-SEC-003 |
| T-UNIT-01 (BE-001) | Migration 00211: `Unit.unit_type` enum + `Unit.fracao_ideal NUMERIC(10,4) CHECK BETWEEN 0 AND 100` + recálculo quórum | 1d | backend | ⏳ | P0-D3 / CC art. 1.352 |
| T-RLS-01 | Habilitar RLS default-on em todas as tabelas com `tenant_id`; opt-out explícito tabelas globais | 1.5d | backend | ⏳ | ADR-0034 / INV-121 |
| T-GUARD-01 | `TenantBoundaryGuard` middleware: enforce path/header `tenant_id` coerência com `AuthUser.CondominiumIDs` | 1d | backend | ⏳ | ADR-0034 Parte B / INV-122 / P0-A2 |
| T-MFA-01 | MFA admin obrigatório (TOTP + backup codes) — Zitadel `mfa_type=required` para admin role + emergency lockout runbook | 1d | backend+ops | ⏳ | ADR-0026 / IDN-007 |
| T-MFA-02 | Step-up MFA endpoint `/auth/step-up` + denylist Redis + 8 endpoints críticos protegidos | 1d | backend | ⏳ | IDN-036 |
| T-SEARCH-01 | `ISearchProvider` real M1 + adapter inicial (OpenSearch ou Meilisearch — dual-check ADR-0042 antes) + indexar `users`, `companies`, `talents`, `condominiums`, `minutes` | 3d | backend | ⏳ | D-120 / ADR-0042 |
| T-MIGRATE-01 | Migration alterar DB CHECK `talents_links.count BETWEEN 1 AND 3` → `BETWEEN 1 AND 5` | 0.25d | backend | ⏳ | D-122 |
| T-SPEC-01 | Atualizar BIL-035 (live_admin_only → admin + Empresa Pro + agência delegada) + cross-domain.md Fase 12 D-097 | 0.5d | backend | ⏳ | D-123 |

### Dual-checks pendentes (STEERING §32)

| ID | Título | Estimate | Dono | Status |
|---|---|---|---|---|
| T-DC-01 | Dual-check ADR-0042 (OpenSearch vs Meilisearch) — fechar antes de implementar T-SEARCH-01 | 1d | arquiteto | ⏳ |
| T-DC-02 | Dual-check D-103 (governance score 1-10 + compliance 0-100) — antes de propagar para INS-032 + ASM-028 + cross-domain.md Fase 12 | 0.5d | arquiteto + product | ⏳ |
| T-DC-03 | Dual-check retroativo D-094..D-118 (13 decisões Fase 13/14 sem dual-check registrado) — registrar em `11-audit/dual-check-log/` | 1d | arquiteto | ⏳ |
| T-DC-04 | Dual-check D-MT-2 (tenant kinds 4 vs 2 vs ADR-0039 redefine) — pré-Sprint 10 crítico (mexer schema depois é caro) | 1d | arquiteto + product | ⏳ |
| T-DC-05 | Dual-check D-MT-8 (tenant resolution flow oficial — header → JWT → path → membership) | 0.5d | arquiteto | ⏳ |

### Multi-tenant consolidation (D-MT-1..D-MT-9 — pré-Sprint 10)

| ID | Título | Estimate | Dono | Status |
|---|---|---|---|---|
| T-MT-01 | Editar `topology-multitenant.md`: §1.2 Tenant Kinds (após D-MT-2) + §5 Resolution Chain (após D-MT-8) + §6.5 Admin Bypass (escolher 1 mecanismo) + §7.5 Observability + §10 Backup/restore + §11 Deletion LGPD | 2d | arquiteto | ⏳ |
| T-MT-02 | Criar `01-domain/patterns/tenant-isolation-row-based.md` (pattern Go canônico repo+sqlc+UoW+TenantBoundaryGuard) | 1d | tech lead backend | ⏳ |
| T-MT-03 | Criar `06-execution/playbooks/rls-debug.md` (suporte sem bypass audit) | 0.5d | SRE | ⏳ |
| T-MT-04 | Criar `08-security/cross-tenant-roles-matrix.md` (matriz role × kind × action) | 1d | arquiteto + pentester | ⏳ |
| T-MT-05 | ADR-0021 §Decisão regra 4 + Schema template — atualizar texto stale "RLS opcional" → "RLS default-on (ver ADR-0034)" | 0.25d | arquiteto | ⏳ |
| T-MT-06 | `12-docs/adr-index.md` — corrigir entrada "ADR-0004 — Multi-tenant" (vestígio renumeração) | 0.1d | docs | ⏳ |
| T-MT-07 | `01-domain/patterns/_moc.md` — substituir "A criar — tenant-isolation-row-based.md" por entrada ativa após T-MT-02 | 0.1d | docs | ⏳ |

### Documentação operacional (P0-DOC4..7)

| ID | Título | Estimate | Dono | Status |
|---|---|---|---|---|
| T-QUALITY-01 | Reescrever 4 stubs em `07-quality/`: CLEAN_CODE, CLEAN_ARCH, SOLID, CODE_CALISTHENICS — regras Go concretas + exemplos | 0.5d | tech lead | ⏳ (subagente em curso) |
| T-PLAYBOOK-01 | Criar 5 playbooks operacionais: stripe-down, zitadel-down, mux-down, livekit-down-assembly, cross-tenant-leak | 0.5d | SRE | ⏳ (subagente em curso) |
| T-DR-01 | Executar **DR drill pré-M1** em 2026-04-30 (não esperar cadência mensal pós-M1); documentar em `11-audit/audit-log/2026-04-30-dr-drill-pre-m1.md` | 1d | SRE | ⏳ |
| T-LOAD-01 | Smoke load test em staging M1 (500 VUs em `/auth/callback`, `/api/v1/connect-me`, webhook Stripe) — antes do go-live | 1d | SRE | ⏳ |
| T-ANALISE-01 | Reescrever `analise-crua.md` removendo N1/N2/N3 (substituir por trial/base/plus/pro D-081/D-096) | 0.5d | docs | ⏳ (subagente em curso) |

### LGPD bloqueadores (D-107 transferidos ao cliente — adicionar SLA duro)

| ID | Título | Estimate | Dono | Status |
|---|---|---|---|---|
| T-LGPD-M1-001 | DPO nomeado + canal público | hand-off cliente | João + DPO | 🔴 sem deadline |
| T-LGPD-M1-002 | DPA Mux assinado | hand-off cliente | João + Mux | 🔴 |
| T-LGPD-M1-003 | DPA Zitadel assinado | hand-off cliente | João + Zitadel | 🔴 |
| T-LGPD-M1-004 | DPA Twilio assinado | hand-off cliente | João + Twilio | 🔴 |
| T-LGPD-M1-005 | DPIA-001 (Avaliação de Impacto) | hand-off cliente | João + DPO | 🔴 |
| T-LGPD-M1-006 | Política de privacidade publicada | hand-off cliente | João + jurídico | 🔴 |
| T-LGPD-M1-007 | DPA Cloudflare (D-117 + D-119 search se hospedado) | hand-off cliente | João + Cloudflare | 🔴 |
| T-LGPD-M1-008 | DPA Resend assinado | hand-off cliente | João + Resend | 🔴 |
| T-LGPD-M1-009 | DPA Railway assinado | hand-off cliente | João + Railway | 🔴 |
| T-AUDIT-01 | DT-010 IAuditLogger cross-BC (Billing/deal/empresa-síndica/membership) — LGPD Art. 46 | 1d | backend | ⏳ |

---

## Gates de saída (M1 deploy pré-check)

- [ ] `AUDIT.md` 🔴 = 0 (A-020, A-021, A-023, A-024 fixados ou documentados como 🟡 com plano)
- [ ] Coverage global ≥ 85%; domain ≥ 95%; application ≥ 90%
- [ ] `go build / vet / test -race / test -tags=integration / gosec -sev high / govulncheck` verdes
- [ ] Staging 48h sem regressão (error rate < 0.5%, p95 < 500ms)
- [ ] Smoke test Playwright passando em staging
- [ ] Sentry prod captura erros em dry-run
- [ ] Zitadel Railway + DNS prod ok
- [ ] Stripe live mode webhook recebendo events
- [ ] Runbooks (rollback, LGPD breach, webhook reprocess) escritos
- [ ] `CHANGELOG` M1 preparado
- [ ] DPO notificado + ciente do SLA LGPD
- [ ] Code freeze 48h antes do deploy (2026-05-06)

---

## Ritual

- **Seg 2026-04-28**: sprint planning (mid-sprint) — revisar progresso + replanejar sobras.
- **Qui 2026-05-01**: holiday BR (Dia do Trabalho); buffer.
- **Sex 2026-05-02 → Ter 2026-05-12**: dev intensivo + DR drill (T-DR-01) + dual-checks (T-DC-01..05) + ISearchProvider (T-SEARCH-01).
- **Qua 2026-05-13**: smoke load test em staging (T-LOAD-01) + sprint review.
- **Qui 2026-05-14**: polish + staging final.
- **Sex 2026-05-15**: **sprint-auditor** + freeze prep.
- **Sáb 2026-05-16 → Dom 2026-05-17**: freeze; só fix crítico.
- **Seg 2026-05-18**: deploy staging → prod (promote Railway/Cloudflare). **🟢 M1 ao vivo**. Monitor 48h.

---

## Links

- [[_moc]]
- [[../backlog#must-m1]]
- [[../../10-schedule/cronograma-macro]]
- [[../../11-audit/AUDIT]]
- [[../../ROADMAP#m1-mvp-contratual]]
- [[sprint-11]]
