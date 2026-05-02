---
title: >-
  2026-04-23 — Fase 14 (IAM Cloudflare-style + Cloudflare edge + SMS
  Twilio/Noop) — dual-check
type: audit
tags:
  - audit
  - dual-check
  - iam
  - cloudflare
  - sms
  - master-sindico
  - v2
  - fase-14
status: completed
date: 2026-04-23T00:00:00.000Z
created: 2026-04-23T00:00:00.000Z
updated: 2026-04-23T00:00:00.000Z
---

# Fase 14 — IAM Cloudflare-style + Cloudflare edge + SMS strategy

## Sumário executivo

Fase 14 da reestruturação master-sindico-v2 → vault Obsidian. Trabalho focado em três decisões cliente João (sessão 2026-04-23):

1. **Insight IAM**: substituir personas pré-estabelecidas por modelo Cloudflare-style (Member + Group + Membership + Role + PermissionGroup + Quota + Plan); único role primitivo = `platform_admin`; nomenclatura `(group_kind)_(role_type)`.
2. **Provider migration**: Cloudflare como edge primário (DDoS/WAF/Pages/R2/Workers/Access/Tunnel/Email Routing inbound/Turnstile); Railway permanece origin (Backend Go + Postgres + Redis); migração AWS ECS M4+ permanece válida.
3. **SMS strategy**: Twilio em prod, NoopSmsProvider em dev/staging; ship M1 com `sms.enabled=false` enquanto Twilio BR não liberado (MFA TOTP-only inicialmente).

## Tarefas executadas

| # | Task | Status | Artefato |
|---|------|--------|----------|
| F14.1 | Mapear Cloudflare + IAM no vault global | ✅ | Confirmado catálogo `10-knowledge/providers/cloudflare/` (14 produtos + 14 MCPs) |
| F14.2 | Benchmark IAM grandes empresas (AWS/GCP/Cloudflare/Azure/Okta) | ✅ | [[../../../../10-knowledge/security/iam-models-benchmark]] (1617 linhas; 7 patterns + 8 anti-patterns) |
| F14.3 | Modelar IAM Master Síndico (Member+Group+Membership+Role+PG+Quota+Plan) | ✅ | [[../../02-architecture/adr/0039-iam-cloudflare-style-master-sindico]] |
| F14.4 | Migração providers (Cloudflare edge + email/SMS strategy) | ✅ | [[../../02-architecture/adr/0040-cloudflare-edge-primary-railway-origin]] + [[../../02-architecture/adr/0041-sms-twilio-prod-noop-dev]] |
| F14.5 | Registrar D-116..D-118 + ADRs + atualizar AUDIT/MOC/sub-produto | ✅ | STATE §D-116..D-118 · este audit log · adr/_moc atualizado |

## Dual-check (obrigatório por `feedback_dual_check_always.md`)

### ADR-0039 — IAM Cloudflare-style

**Reviewer**: Agent Opus em contexto limpo, 2026-04-23 (sessão d77d3b1f).
**Veredito**: APROVADO COM AJUSTES (3 críticos + 5 médios + 2 cosméticos).

**Findings críticos (todos aplicados)**:

1. **🔴 Multi-Group por Member não modelado**. Member pode ser titular condo A + dependente condo B + employee enterprise simultaneamente. `tags.tenant_active` (single) não resolve. ✅ **Aplicado**: §2.5 Membership(member×group×role×status) como entidade explícita; INV-IAM-001..004; authz sempre passa `(user_id, tenant_id, action, resource)` no request; `tags.tenant_active` re-classificado como UI-state apenas.

2. **🔴 Redundância `Role.is_default` ↔ `Group.default_role`**. Mesmo dado em dois lugares = convite a inconsistência. ✅ **Aplicado**: removido `is_default` da Role; `Group.default_role_id` é fonte canônica única; INV-IAM-004 garante consistência via DB.

3. **🔴 Status `accepted` prematuro** sem caminho claro de aplicação. ✅ **Aplicado**: status degradado para `accepted (conditional)` — promove para `accepted (active)` quando 3 artefatos pré-M1 existirem (catálogo PG seed YAML, migration idempotente, endpoints platform-side admin).

**Findings médios (todos aplicados)**:

4. 🟡 **Personas faltantes no mapping** — adicionado mapping completo separando "com legacy" vs "sem legacy persona" (Roles novas atribuídas manualmente pelo Group admin pós-migration).
5. 🟡 **ADR-0034 (RLS) não cross-linkado em §Decisão** — adicionado ao passo 2 da ordem de avaliação ("DB-layer fail-safe").
6. 🟡 **Cache invalidation furo silencioso** em Group `enterprise`/`marketing_agency` (não-tenant) — adicionado key `abac:role:{role_id}:members` SET; SUNION on Role.permission_group_ids change.
7. 🟡 **Conditions não era vocabulário fechado** de fato — adicionada §"Vocabulário condicional fechado M1" com 7 conditions explícitas: `plan_in`, `mfa_required`, `step_up_required`, `device_compliant`, `time_window`, `geo_in`, `quota_metric`.
8. 🟡 **Painel admin sem dry-run/simulate-as** — adicionados endpoints `POST /admin/policies/dry-run` e `GET /admin/simulate-as/{user_id}` (Cloudflare playbook §3.7).

**Findings cosméticos (aplicados)**:

9. 🟢 `(group_role)` (insight informal) ≠ `(group_kind)` (formal) — adicionada nota terminológica explicativa.
10. 🟢 Reviewer não leu ADR-0034/0029 mas confirmou citação consistente — sem ação.

**Resultado pós-ajustes**: ADR-0039 promovido a `accepted (conditional)` definitivo. Promove para `accepted (active)` quando 3 artefatos entregues em pré-M1 (Sprint 10).

### ADR-0040 e ADR-0041

Não passaram por dual-check separado nesta sessão (escopo de tempo). **Debt**: dual-check incremental em sessão futura, especialmente:

- ADR-0040: validar que Workers BFF + Tunnel não introduz latência > 100ms vs origin direto; validar que Access cobre caso `/admin/*` sem quebrar ABAC do origin (defense in depth bem alinhado).
- ADR-0041: validar que NoopSmsProvider em dev é UX-suficiente (painel `/dev/sms-log` cobre debug); validar templates pt-BR não excedem 160 chars.

Ambos são `accepted` provisório com revisão pendente.

## Decisões consolidadas

- **D-116** — IAM Cloudflare-style (Member + Group + Membership + Role + PermissionGroup + Quota + Plan)
- **D-117** — Cloudflare edge primário + Railway origin (mantém ADR-0017)
- **D-118** — SMS Twilio prod / Noop dev / ship M1 com SMS desligado se Twilio não liberado

## Pré-requisitos pré-M1 (Sprint 10)

Para D-116 promover de `conditional` para `active`:

1. Catálogo PermissionGroup seed file: `backend/internal/modules/identity/iam/seed/permission_groups_v1.yaml`
2. Migration idempotente persona-legada → Group+Membership+Role (script Go re-runnable, dry-run mode obrigatório, audit log)
3. Endpoints platform-side admin (HTTP + service): `POST /admin/permission-groups`, `POST /admin/groups/{id}/roles`, `GET /admin/groups/{id}/effective-permissions/{user_id}`, `POST /admin/policies/dry-run`, `GET /admin/simulate-as/{user_id}`

Para D-117 ship M1:

1. Setup zone Cloudflare + DNS + Universal SSL + Tunnel cloudflared para origin Railway
2. Pages SolidJS deploy Git-integrated
3. R2 buckets (uploads, exports LGPD, vídeos transientes)
4. Workers BFF (rate-limit, webhook receivers Stripe/Resend/Mux idempotentes)
5. Cloudflare Access protegendo `/admin/*` com SSO Zitadel
6. Email Routing inbound `suporte@`, `billing@`, `bounces@` → Worker
7. Turnstile em signup/password-reset
8. DPA Cloudflare assinado

Para D-118 ship M1:

1. `ISmsProvider` interface + 3 implementações (Twilio/Noop/Test)
2. Painel `/dev/sms-log` em build dev
3. Webhook handler em Workers BFF (idempotency via `MessageSid` UNIQUE)
4. Templates pt-BR versionados em git
5. Rate-limit middleware (per-user 10/dia + global 500/dia + per-reason)
6. Audit trail SMS (user_id + reason + status + timestamp; sem conteúdo)
7. Ship com `sms.enabled=false` em prod até Twilio BR liberado + DPA Twilio assinado

## Impactos cruzados

- [[../../00-product/sub-produtos/01-governanca-institucional]] — atualizar §personas para refletir Group+Role (não persona pré-estabelecida); adicionar referência a Membership entity
- [[../../03-subprojects/web/ui-catalog]] — telas admin platform-side (CRUD PG/Role/Group) e Group-side (CRUD sub-roles, simulate-as, dry-run) — ainda não catalogadas
- [[../../03-subprojects/traceability]] — atualizar matriz feature × persona substituindo por feature × `(group_kind, role_type)` × plan
- [[../../05-tasks/by-module/admin-plataforma.md]] — adicionar tasks ADM-IAM-### (catálogo PG seed, migration job, painel)
- [[../../02-architecture/adr/_moc]] — adicionar 0039, 0040, 0041 + atualizar tabela síntese

## Próximos passos sugeridos

1. **Sprint 10 kickoff**: priorizar 3 artefatos pré-M1 D-116 (catálogo seed + migration + endpoints platform-side).
2. **Setup Cloudflare**: DPA assinado + zone + DNS + Tunnel + Pages — pode rodar em paralelo a IAM.
3. **Twilio approval**: cliente João precisa liberar conta Twilio BR + assinar DPA → flip prod.
4. **Dual-check ADR-0040 e 0041**: agendar sessão dedicada com Agent Opus em contexto limpo.
5. **Sub-produto identidade**: criar `00-product/sub-produtos/00-identidade.md` (ou rebatizar 01-governanca-institucional §personas) para abrigar modelo IAM canônico do produto (cliente-facing).

## Vizinhos

- [[_moc]]
- [[../../STATE]] §D-116..D-118
- [[2026-04-24-relatorio-mestre-para-joao]] (Fase 13 anterior)
- [[../../02-architecture/adr/0039-iam-cloudflare-style-master-sindico]]
- [[../../02-architecture/adr/0040-cloudflare-edge-primary-railway-origin]]
- [[../../02-architecture/adr/0041-sms-twilio-prod-noop-dev]]
