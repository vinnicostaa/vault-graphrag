---
title: Backend — Requirements (destilação Fase 8, código real)
type: requirement
tags: [requirements, backend, master-sindico, v2, functional, non-functional, destilacao-fase-8]
source: /home/vinni/Documentos/Projetos/Clientes/Joao/MasterSindico/Development/backend + .kiro/specs/master-sindico/requirements
created: 2026-04-23
updated: 2026-04-23
---

# Backend — Requirements funcionais + não-funcionais

Visão consolidada dos **requisitos vivos** que o backend Go atende, derivada
do código real + specs `backend/.kiro/specs/master-sindico/requirements/*.md`
(8 recortes com numeração Req 1-103). Este arquivo consolida para o vault v2,
mantendo rastreabilidade com arquivos-fonte.

---

## 1. Fontes consultadas (2026-04-23)

| Recorte (path) | Linhas | Escopo |
|---|---|---|
| `backend/.kiro/specs/master-sindico/requirements/billing.md:1-120` | 120 | Plans universais + trial + quota |
| `backend/.kiro/specs/master-sindico/requirements/identity.md` | — | Auth Zitadel + sessions + 1-device |
| `backend/.kiro/specs/master-sindico/requirements/institutional.md` | — | Condo, timeline, MP, polls, compliance |
| `backend/.kiro/specs/master-sindico/requirements/commercial.md` | — | Connect Me, deals, proposals, harassment |
| `backend/.kiro/specs/master-sindico/requirements/content.md` | — | Vídeos Mux + agência marketing |
| `backend/.kiro/specs/master-sindico/requirements/assembly.md` | — | Ciclo de assembleia + live |
| `backend/.kiro/specs/master-sindico/requirements/cross-domain.md` | — | Cross-module contracts |
| `backend/.kiro/specs/master-sindico/requirements/functional-matrix.md` | — | Matriz F × persona × plan |
| `backend/.kiro/specs/master-sindico/requirements/personas-and-plans.md:1-30` | 30+ | Matriz persona × plano |

Plus código real em `backend/internal/**` (38 entidades, 41 enums nomeados,
118 endpoints).

---

## 2. Plans universais (Stripe-style — ADR-0032)

**D-032 do STEERING** + confirmação no código: `plan_tier` no DB é enum de 4
valores fixos, usado para **todas** as roles.

```sql
-- migration 00001_create_enums.sql:26-31
CREATE TYPE plan_tier AS ENUM ('trial', 'base', 'plus', 'pro');
```

```go
// internal/shared/types/auth_user.go:85-90
var PlanTierOrder = map[string]int{
    "trial": 0,
    "base":  1,
    "plus":  2,
    "pro":   3,
}
```

- `HasMinimumPlanTier(userTier, minTier)` é o **único** comparador hierárquico.
- Rules ABAC usam `MinPlanTier: "base"` ou `"plus"` — **nunca slug composto**.
- Plans = `(role × tier)` via UNIQUE em tabela `plans`.

**Nenhum `N1/N2/N3` ou slug composto encontrado** no código backend Go:
grep `'N1\|N2\|N3'` em migrations/enums retornou zero matches. DT-004 do
legado (que exigia limpeza de slugs) não é aplicável aqui — provavelmente
resquício do full-stack TS legado.

### 2.1 Trial por persona

| Persona (`role`) | `tier` inicial | Duração trial (Req 4.1) | Gates típicos |
|---|---|---|---|
| Síndico | `trial` → `base`/`plus`/`pro` | 15 dias | Criar timeline, criar comunicados, exportar relatório |
| Empresa | — | 7 dias | Publicar vídeos, registrar execuções, gerenciar usuários |
| Parceiro vizinhança | — | 30 dias | Criar promoções, ver métricas |
| Morador Pagante | `base` → `plus` | — | Sem trial — pagamento imediato ou permanece Base |

`trial_started_at` + `trial_days` na tabela `subscriptions` → cálculo SQL.
Enforced em `Subscription.IsActive()` (backend/internal/modules/billing/domain/entities/subscription.go:132-142)
com fix **trial retroativo** (Sprint 7.6 bug 2).

### 2.2 Quotas por feature (`services/QuotaLimits`)

| Feature | Escopo de janela | Limite trial / base / plus / pro |
|---|---|---|
| `connect_me` | anual (UTC) | syndic: 2/2/4/∞ ; resident: —/2/4/— |
| `video_upload` | mensal (UTC) | syndic: —/4/8/12 ; enterprise: —/—/8/12 |

- `QuotaUnlimited = -1` sentinel no domínio.
- `FeatureUsage` persiste `used` com UNIQUE `(user_id, feature_key, period_start)` — janela corretamente arquivada.
- **PBT**: 300 casos em `feature_usage_pbt_test.go`.

### 2.3 Paywall / soft block (Decision 4)

- **Sem grace period** — `SubscriptionExpired` event → acesso premium bloqueia imediato.
- **Dados permanecem** — nenhum DELETE; `CheckFeatureAccessUseCase` retorna 402 `feature_not_available`.
- Mensagem padrão (copy pt-BR): _"Essa funcionalidade está disponível nos planos ativos da plataforma Master Síndico."_

---

## 3. Requisitos não-funcionais (gates duros)

### 3.1 Coverage threshold (steering/testing.md)

| Camada | Mínimo | Aferido |
|---|---|---|
| `internal/modules/*/domain/entities` | 95% | institutional 98.7%, commercial 96.1%, assembly 100% |
| `internal/modules/*/application/use_cases` | 90% | institutional 91.0%, commercial 92.8%, content 100%, assembly 91.5% |
| `internal/modules/*/infrastructure/database` | 85% | coberto por integration tests |
| `internal/modules/*/infrastructure/http` | 75% | handlers table-driven |
| `internal/shared/middleware` | 90% | unit + ABAC matrix (100+) + PBT 400 |
| `internal/core` | 95% | |
| `pkg/*` | 90% | pagination PBT 1000, telemetry 4 tests |
| Global | ≥85% | |

### 3.2 Security gates

```bash
gosec -severity=high -exclude-dir=generated -exclude-dir=cmd/zitadel-setup ./...
# saída esperada: 0 Issues em 314 files

govulncheck ./...
# saída esperada: "No vulnerabilities found"
```

Snapshot 2026-04-22: ✅ 0 findings, 0 CVEs.

### 3.3 Performance / Pagination

- `LIMIT + OFFSET` em todas as queries de lista paginadas (Sprint 7.7).
- Safety caps em assembly SQL direto (AgendaItem 100, Attendance 1000, Votes 1000, Proxies 500, Science 1000).
- Pool Postgres configurável: `ConnMaxIdleTime` (default 30 min) via `DB_CONN_MAX_IDLE_TIME`.

### 3.4 Observabilidade (OTel) — Sprint 7.9 entregue

- **Spans**:
  - HTTP via `otelgin` — `request_id`, `http.method`, `http.route`, `http.status_code`.
  - DB via `pgx_tracer.go` — `db.query`, `db.batch`, `db.copy_from` semconv v1.39.
  - Application layer: uso planejado em use cases críticos (billing webhook, assembly live).
- **Métricas**: Prometheus bridge; `/metrics` ativado quando `OTEL_PROMETHEUS_ENABLED=true`.
- **Logs**: zerolog enriquece com `trace_id` + `span_id` quando span válido.
- **Sample rate**: 1.0 (dev/staging), 0.1 (prod).

### 3.5 Rate limiting

| Grupo | RPM padrão | Burst |
|---|---|---|
| `/auth/*` | 20 | 5 |
| `/api/v1/*` | 60 | 10 |

Config via env: `AUTH_RATE_LIMIT_RPM`, `AUTH_RATE_LIMIT_BURST`,
`RATE_LIMIT_RPM`, `RATE_LIMIT_BURST`.

**A-039 aberto**: spec exige **per-userID** adicional (300 rpm por user).
Implementação atual só por IP — risco em NAT corporativo.

### 3.6 Idempotency

- **Webhooks Stripe**: `evt_XXX` ID é estável; `HandleStripeWebhookUseCase` suporta receber o mesmo evento 2x sem efeito colateral (mudança de status idempotente + UNIQUE em stripe_subscription_id).
- **Webhooks Mux**: `video.asset.ready` — video_id UUIDv7 + `UpdateAssetFromWebhook` idempotente (mesmo asset OK).
- **Stripe CreateCustomer/Subscription**: `IdempotencyKey` obrigatório (`customer:{userID}`, `subscription:{userID}:{planID}:{interval}`).

### 3.7 Tenant isolation (multi-tenancy row-based)

- Coluna `condominium_id` presente em todas as tabelas institucionais + commercial Connect Me/Deals/Proposals + assembly tables.
- `AuthUser.CondominiumIDs []string` populado no sync (via `institutional.CondominiumIDsProvider` JOIN syndic_mandates + memberships).
- ABAC `RequireTenantMatch: true` ⇒ engine compara `resourceTenantID` com esse slice.
- Bug do Sprint 7.6 (resolver `TenantID = OrgZitadelID ≠ CondominiumID`) **corrigido** — todos os 14+ ABAC rules com `RequireTenantMatch: true` passam.

### 3.8 1-device per account

- `X-Device-Fingerprint` header lido em `Authenticate`, propagado ao `SyncUserUseCase`.
- `sessionRepository.InvalidatePreviousByUserID(userID, currentSessionID)` chamado sempre no sync.
- Gap **A-023**: sem comparação de fingerprints + audit log `device_changed`.

### 3.9 PKCE + cookies seguros

- OIDC PKCE com `gorilla/securecookie` (HMAC + AES).
- Cookies: `HttpOnly=true`, `Secure` (forçado em non-dev), `SameSite=Lax`.
- `oidc_handler.go:110-112,166-168,181-183,200-202` — 4 pontos de `SetCookie`, todos consistentes.

### 3.10 Secrets management

- `.env` em dev; env vars no Railway em prod.
- `zitadel-key*.json`, `/migrate`, `/zitadel-setup` no `.gitignore` (A-018 fechado).
- Gosec **G101** (hardcoded credentials): 0 findings.
- Nenhum `InsecureSkipVerify` no código (F14 auditoria de backdoors concluída).

---

## 4. Requisitos funcionais por módulo (resumo)

### 4.1 Identity (Req 1-3)

- **Req 1** — Cadastro + Login via Zitadel (OIDC Auth Code + PKCE).
- **Req 2** — Session storage + 1-device + TTL configurável (`SessionTTL` em AppConfig).
- **Req 3** — `GET /auth/me` retorna `AuthUser` DTO.

### 4.2 Billing (Req 4 + 5 + 4.1)

- **Req 4** — Stripe via `IPaymentGateway`; CRUD de subscription + checkout + customer portal + webhook.
- **Req 4.1** — Trial por persona (15 / 7 / 30 / — dias).
- **Req 4.2** — Quotas feature × role × tier — 2 features em produção (connect_me, video_upload).
- **Req 5** — Onboarding wizard (frontend; backend responde `PATCH /onboarding/progress` — parte do identity).

### 4.3 Institutional (Req 9-22 + Req 12 + Req 13 + Req 14)

- **Req 9** — Unit + Membership (1 titular + 2 dependentes max; UNIQUE parcial em titular ativo).
- **Req 11** — Timeline append-only (7 tipos de entry, adendo sem modificar pai).
- **Req 12** — Master Plan com status automático (7 fases reagindo a `EventTimelinePublished`).
- **Req 13** — Eventos (13 tipos + resposta ciente/confirmado).
- **Req 14** — Comunicados (8 tipos × 4 prioridades + read tracking idempotente).
- **Req 14.1** — Polls (3 tipos: yes_no_dk, multiple_choice, scale_1_to_5).
- **Req 15** — Management Assessment bimestral obrigatória (3 perguntas, escala 1-10, anônima).
- **Req 16** — Compliance: termo de responsabilidade (IP + timestamp) + declaração anual + audit trail 5 anos LGPD.
- **Req 17** — Membership end com 4 motivos enum.
- **Req 18** — Mandate transfer + end.
- **Req 22** — Dashboard síndico (13 indicadores + filtros).

### 4.4 Commercial (Req 4.1 Connect Me + Req 4.4 Proposals + Req 19 Harassment)

- **Req 4.1 (Connect Me)** — 4 fluxos: syndic→enterprise, resident→syndic cold, enterprise→enterprise, syndic→local_company.
- **Req 4.3** — Execution records com ciclo draft → submitted → validated/rejected/adjustment.
- **Req 4.4** — Proposals + Supplier Vote Sessions (quórum simple/absolute/qualified).
- **Req 4.5** — Closed Deal com dupla assinatura empresa → síndico; imutável após `signed`.
- **Req 4.6** — Company profile + state machine + 2 roles (admin/operator).
- **Req 4.7** — Empresa Síndica (invite token UUIDv7 + 5 permissions booleanas + revoke).
- **Req 6.7** — Company evaluation (UNIQUE company + evaluator + condominium; A-029 com UoW+UNIQUE).
- **Req 6.8** — Post-deal comunicado (síndico valida).
- **Req 19** — Harassment report (admin warning/suspension/ban).

### 4.5 Content (Req 5.1 + 5.5)

- **Req 5.1** — Upload de vídeo via Mux direct upload URL.
- **Req 5.2** — Metadata: tipo (12 pt-BR), voting auth flag.
- **Req 5.5** — Agência de marketing invite/accept/update permissions/revoke.

### 4.6 Assembly (Req 7)

- **Req 7.1** — Configure + rascunho.
- **Req 7.2** — Publish edital (obrigatório antes de Start).
- **Req 7.3** — Start / End + guards (não inicia sem edital).
- **Req 7.4** — Agenda items com fração ideal (0-100%) + lock após publicação.
- **Req 7.5** — Votes: UNIQUE(agenda_item_id, voter_id) — anti-double-vote.
- **Req 7.6** — Attendance + proxies + science.
- **Req 7.7** — Live session via Livekit (retry + reconciliação NotFound).
- **Req 7.8** — Minutes auto-gerada.

---

## 5. Contratos externos

### 5.1 OpenAPI 3.0

- Gerado via `swaggo/swag` a partir de annotations godoc.
- Servido em `GET /docs` com Scalar UI.
- Atualizado no ciclo Ship de cada task.

### 5.2 Webhooks de terceiros (público — sem Authenticate)

| Path | Verificação | Evento relevante |
|---|---|---|
| `POST /webhooks/stripe` | HMAC-SHA256 via `gateway.VerifyWebhookSignature` antes de parse | `customer.subscription.*`, `invoice.*` |
| `POST /webhooks/mux` | HMAC-SHA256 `verifyMuxSignature` | `video.asset.ready`, `video.asset.errored` |

### 5.3 Frontend contracts

- Todas rotas `/api/v1/*` retornam JSON com schema estável.
- Erros via `AppError` serializado: `{code, message, status}`.
- Paginação: `?limit=50&offset=0`; resposta envolve lista em `{data: [...]}` + mais alguns casos `{data, pagination}`.
- Cookies cross-subdomain em prod: `Domain=.mastersindico.com.br`, `Secure`, `HttpOnly`, `SameSite=Lax`.

---

## 6. LGPD / Compliance

- Audit trail append-only em `audit_log_entries` (tabela) — atualmente só compliance usa (DT-010 a expandir).
- Retenção mínima 5 anos (Req 16).
- PII never logged (enforcement por grep, seção 17 de [[patterns]]).
- CNPJ/CPF com `Masked()` em logs.
- Usuário pode sair e pedir exclusão — backend marca `deleted` (soft-delete). Hard delete requer job explícito Sprint 10+.

---

## 7. Gaps e débitos ativos

Cruzando STATE.md + AUDIT.md (2026-04-22):

| ID | Tipo | Sprint | Descrição |
|---|---|---|---|
| **Unit sem `unit_type`/`fracao_ideal`** | gap spec | 10 (crítico) | Corner-case master-sindico; impossibilita quórum ponderado real |
| DT-003 | dívida | 10 | Sem circuit breaker em Zitadel introspect |
| DT-004 | dívida | já tarde | Sem `context.WithTimeout` em `UnitOfWork.Run` |
| DT-010 | dívida | 10 | `IAuditLogger` cross-módulo (billing + commercial + identity) |
| A-023 | finding | 10 | 1-device change sem audit log comparativo |
| A-024 | finding | 10 | `rawBodyReader` interface privada duplicada |
| A-039 | finding | 10 | Rate limit só por IP, spec pede per-userID |
| NATS/outbox | gap STEERING | 10-11 | Eventos só in-memory — sem durabilidade cross-process |
| SMS/Push/PDF providers | gap impl | 10-11 | Interfaces prontas, adapters são stubs |
| Sprint 9.10 | task aberta | 9 | 2 templates email faltantes (welcomes extensivo, billing-alerts) |
| Sprint 9.19 | task aberta | 9 | Zitadel managed em Railway + DNS custom |

---

## 8. Referências cruzadas

- Recortes canônicos: `backend/.kiro/specs/master-sindico/requirements/*.md`.
- Arquitetura: [[architecture]]
- Padrões: [[patterns]]
- Segurança: [[security]]
- Tasks: [[tasks]]
- Functional matrix completa: 
  `backend/.kiro/specs/master-sindico/requirements/functional-matrix.md`.
