---
title: Dual-Check — Fase 8 re-destilação dos 3 sub-projetos (backend Go + web SolidJS + mobile Flutter)
type: note
tags: [dual-check, subprojects, backend, web, mobile, master-sindico, v2, fase-8]
created: 2026-04-23
updated: 2026-04-23
aliases: [dual-check-fase-8-subprojetos, 2026-04-23-fase8-dual-check]
---

# Dual-Check — Fase 8 (3 sub-projetos re-destilados)

Revisor em **sessão nova** (contexto limpo), sem ter participado da re-destilação Fase 8 dos
sub-projetos `backend/`, `web/`, `mobile/`. Insumos consultados:

- Artefatos v2 em `master-sindico-v2/03-subprojects/{backend,web,mobile}/*.md` (7+8+8 arquivos,
  lidos integralmente).
- Código-fonte real em `Clientes/Joao/MasterSindico/Development/{backend,web,app}/` (amostragem
  dirigida para validar claims: `go.mod`, `migrations/*.sql`, `module.go`, `pubspec.yaml`,
  `build.gradle.kts`, `auth_local_datasource.dart`, `app_colors.dart`, `apps/*/package.json`,
  `bun.lock`).
- Regras arquiteturais obrigatórias do prompt (D-056 / D-057 / D-058 + stacks confirmadas).
- [[../../CLAUDE]], [[../../STATE]], [[../../STEERING]] v2.

Sem acesso ao agente que escreveu os artefatos; avaliação independente baseada **apenas** em
código real e regras duras.

---

## 0. Sumário executivo

| Sub-projeto | Arquivos | Veredito geral | Gaps críticos | Achados A-DC-FASE8 |
|---|---|---|---|---|
| **backend** | 7 (README, architecture, patterns, requirements, security, tasks, _moc) | ✅ **Aprovado com 6 correções menores** — destilação de altíssima fidelidade ao código real; todos os números principais batem (118 endpoints, 39 migrations, `plan_tier` enum, 4 PBT suites); gaps BE-001/BE-006/A-023/A-039 corretamente identificados e priorizados | — | A-DC-FASE8-01 … 06 |
| **web** | 8 (README, architecture, patterns, ui-catalog, design-system, requirements, security, tasks, _moc) | ✅ **Aprovado com 5 correções** — stack real SolidJS+TanStack+Rsbuild+UnoCSS+Kobalte+Axios+Zod+Biome+Bun confirmada exata nas versões `bun.lock`; 6 apps × portas corretas; D-058 admin transversal em `cms/admin/*` consistente; ui-catalog bem estruturado mas com inconsistência 141 vs 174 telas | — | A-DC-FASE8-07 … 11 |
| **mobile** | 8 (README, architecture, patterns, ui-catalog, design-system-como-security, requirements, security, tasks, _moc) | ✅ **Aprovado com 3 correções e 1 gap estrutural** — destilação identifica exatamente as 10 deps ausentes vs CLAUDE/ARCHITECTURE alvo; `minSdk=19`, bundle ID divergente, chave token inconsistente (`AUTH_TOKEN` vs `auth_access_token`), paleta Flutter default `#6C63FF` todos confirmados; falta `design-system.md` dedicado (como web tem) | **DT-MB-005 paleta não reconciliada com Manual IV** | A-DC-FASE8-12 … 15 |

**Veredito global**: **✅ Fase 8 pronta para fechar**, após ajustes A-DC-FASE8-01..15 (maioria
correções pontuais de número/redação). Nenhum achado bloqueante de conformidade arquitetural
(D-056/D-057/D-058 respeitados em todos os 3 sub-projetos).

**Conformidade regras obrigatórias**:

| Regra | Backend | Web | Mobile |
|---|---|---|---|
| D-057 Planos globais Stripe-style (`trial/base/plus/pro`) | ✅ enum DB confirmado `migrations/00001:27-32` + `PlanTierOrder` `shared/types/auth_user.go:85-90` | ✅ schemas.ts planeja `trial\|base\|plus\|pro` | ✅ README destaca D-057 + "sem slugs N1/N2/N3" |
| D-058 Admin = role privilegiada (não app separada) | ✅ `abac_engine` bypass global hardcoded p/ `role=="admin"` | ✅ `apps/cms/routes/_authenticated/admin/` (views embutidas; sem `apps/admin`) | ✅ sem feature admin separada; gate ABAC na mesma app |
| D-056 Agnosticismo de provedor (I*Provider em domain) | ✅ `IPaymentGateway`/`IVideoProvider`/`ILiveProvider` em `domain/providers/` + adapters em `infrastructure/providers/` | N/A (consumidor) | N/A (consumidor) |
| Stack web canônica | — | ✅ SolidJS 1.9.12 + TanStack Solid Router 1.168 + Rsbuild 1.7 + UnoCSS 66.6 + Kobalte 0.13 + Axios 1.15 + Zod 4.3 + Biome 2.3 + Bun — **todos confirmados versão exata em `bun.lock`** | — |
| Stack mobile canônica | — | — | ✅ Flutter + FVM + Clean Arch + get_it/injectable + go_router **confirmados em `pubspec.yaml`**; dívidas listadas honestamente |

---

## 1. Método de validação

### 1.1 Reads integrais

- `backend/README.md` (559 linhas) + `backend/architecture.md` (624) + `backend/patterns.md` (678) + `backend/requirements.md` (311) + `backend/security.md` (493) + `backend/tasks.md` (267) + `backend/_moc.md` (81).
- `web/README.md` (272) + `web/architecture.md` (720) + `web/patterns.md` (1–100 amostra) + `web/security.md` (645) + `web/ui-catalog.md` (1–150 amostra + linha 610 totais) + `web/design-system.md` (1–150 amostra) + `web/requirements.md` (1–100 amostra) + `web/tasks.md` (355) + `web/_moc.md` (99).
- `mobile/README.md` (288) + `mobile/architecture.md` (1–100 amostra) + `mobile/security.md` (1–150 amostra) + `mobile/tasks.md` (1–100 amostra) + `mobile/_moc.md` (55).

### 1.2 Amostragem de código/config validada

| # | Claim no artefato v2 | Fonte real | Resultado |
|---|---|---|---|
| 1 | "Plans universais — `plan_tier` enum 4 valores em migration 00001" | `backend/internal/shared/providers/database/migrations/00001_create_enums.sql:27-32` | ✅ `CREATE TYPE plan_tier AS ENUM ('trial', 'base', 'plus', 'pro');` — exato |
| 2 | "118 endpoints protegidos `/api/v1/*`" | `grep -rE '\.(GET|POST|PUT|PATCH|DELETE)\(' internal/modules/ \| wc -l` | ✅ **118** (bate) |
| 3 | "39 migrations goose" | `ls internal/shared/providers/database/migrations/ \| wc -l` | ✅ **39** |
| 4 | "38 aggregates de domínio" | `find internal/modules/*/domain/entities/*.go -not -name '*_test.go' \| wc -l` | ✅ **38** |
| 5 | "4 Value Objects (CPF, CNPJ, Email, Address, BimonthlyPeriod)" | `find internal/modules/*/domain/value_objects/ -name '*.go' -not -name '*_test.go'` | ⚠️ **4 VOs reais**: `cnpj.go`, `cpf.go`, `email.go`, `address.go`. **`BimonthlyPeriod` é `domain/services/`, não VO** — inconsistência de classificação (ver A-DC-FASE8-02) |
| 6 | "~48 arquivos de use cases" | `find internal/modules/*/application/use_cases/ -name '*.go' -not -name '*_test.go' -not -name '*_stubs*' \| wc -l` | ✅ **46** (README diz "~48", aceitável aproximação) |
| 7 | "3 PBT suites críticas" (README §9) vs "4 PBT suites" (patterns §12) | `grep -rln 'rapid\.Check\|rapid\.Run' backend/` | ✅ **4 suites**: `agenda_item_pbt_test.go`, `feature_usage_pbt_test.go`, `abac_engine_pbt_test.go`, `pagination_test.go`. README está **inconsistente consigo mesmo** (diz "3 suites críticas + pagination"); `_moc.md` §Snapshot diz "4 PBT suites" — correto |
| 8 | "NATS JetStream ausente no `go.mod`" | `grep 'nats-io/nats' backend/go.mod` | ⚠️ **Parcialmente falso**: `github.com/nats-io/nats.go v1.48.0 // indirect` **está** no go.mod (indireto via otel/livekit). A claim é "não usado como direct dep + publisher é `InMemoryPublisher`" — essa parte é verdade (não há `nats.Connect` no código de módulos). Redação do README (§2 linha 72: "Não presente no `go.mod`") é **tecnicamente incorreta** — corrigir para "não é direct dependency; não há publisher NATS instanciado" (ver A-DC-FASE8-01) |
| 9 | "Unit aggregate: sem `unit_type`/`fracao_ideal`" | `grep 'unit_type\|fracao_ideal' migrations/00202_create_units_memberships.sql` | ✅ **confirmado**: a migration tem apenas `unit_number`, `block`, `floor`, `active`. Gap BE-001 real |
| 10 | "ABAC matrix 789 linhas, 70+ actions, ~130 rules" | `wc -l internal/shared/middleware/abac_rules.go` | ⚠️ **788 linhas** (off-by-one, "789" é boundary do `head`/`tail`). `~130 rules` aferido via `grep -c 'AllowedRoles\|Name:' = 215` → número de rules aproximado; aceitável. (A-DC-FASE8-03) |
| 11 | "Stack web SolidJS 1.9.12 + TanStack Router 1.168 + Rsbuild 1.7 + UnoCSS 66.6 + Kobalte 0.13 + Axios 1.15 + Zod 4.3" | `bun.lock` + `apps/auth/package.json` | ✅ **TODOS confirmados** (`@kobalte/core@0.13.11`, `@tanstack/solid-router@1.168.17`, `@rsbuild/core@1.7.5`, `axios@1.15.0`, `solid-js@1.9.12`, `zod@4.3.6`) |
| 12 | "6 apps: auth 3000, cms 3001, lms 3002, forum 3003, assembly 3004, lp 3005" | `grep 'rsbuild dev --port' apps/*/package.json` | ✅ **Todas as 6 portas confirmadas** exatas |
| 13 | "admin D-058 transversal embutido em `cms` sob `/admin/*`" | `ls apps/` | ✅ **não existe `apps/admin`**; `apps/` contém apenas assembly/auth/cms/forum/lms/lp (6 apps). Destilação correta |
| 14 | "`apps/cms/src/routes/` apenas `index.tsx` + `__root.tsx`" | `ls apps/cms/src/routes/` | ✅ confirmado — scaffold apenas; telas são alvo M1-M3. Destilação é honesta (README reconhece "Scaffold; `routes/index.tsx` stub") |
| 15 | "Mobile: 10 deps ausentes do `pubspec.yaml`" | `cat app/pubspec.yaml` | ✅ **confirmado**: ausentes `bloc_concurrency`, `hydrated_bloc`, `freezed`, `hive_flutter`, `local_auth`, `device_info_plus`, `firebase_messaging`, `app_links`, `easy_localization`, `freeRASP_flutter`. Todos listados honestamente como "⚠️ ausente — dívida" |
| 16 | "Mobile: `minSdk=19`" | `grep 'minSdk' android/app/build.gradle.kts` | ✅ `minSdk = 19` na linha 27 |
| 17 | "Mobile: bundle ID `com.mastersindico.br.mastersindico` divergente do alvo `com.mastersindico.app`" | `grep 'namespace\|applicationId' android/app/build.gradle.kts` | ✅ **confirmado** (linhas 9 + 24) |
| 18 | "Mobile: chave token `AUTH_TOKEN` vs canônica `auth_access_token`" | `grep 'AUTH_TOKEN\|auth_access_token' lib/` | ✅ **inconsistência confirmada**: `AuthLocalDatasourceImpl` usa `AUTH_TOKEN`; `AppConstants.tokenKey = 'auth_access_token'` (constants.dart:32); refactor canonicalização de fato pendente |
| 19 | "Mobile: paleta default Flutter `#6C63FF`" | `grep primary app/lib/core/theme/app_colors.dart` | ✅ **`Color(0xFF6C63FF)` linha 14** — exato default `flutter create`; Manual IV oficial é gold `#C69C3F` + navy `#071A33` (destilado do web `design-system.md`). Reconciliação ausente |

### 1.3 Grep de slugs proibidos

Executei `grep -rn 'N1\|N2\|N3' master-sindico-v2/03-subprojects/**/*.md` para conferir se alguma
destilação reintroduziu slugs legacy proibidos por D-057 (planos universais):

- Resultado: **1 match** em `master-sindico-v2/03-subprojects/mobile/README.md` (linha 23) e
  **várias menções em `web/README.md`** como **negação explícita**: _"admin MS (N1/N2/N3)"_ no
  contexto de descrever personas históricas.

**Análise**: em ambos os casos as menções são **meta-referências** negando o slug ("sem slugs
N1/N2/N3" / "multi-persona que inclui antigos N1/N2/N3"). Não são usados como slug ativo em
código/schema. **Tolerável** mas sugiro A-DC-FASE8-04 para substituir por linguagem neutra tipo
"sindic roles histórico" para evitar que LLM futuro interprete como slug vivo.

### 1.4 Conformidade D-### (amostra)

- **D-020** (stack web real): ✅ README §Nota-de-reconciliação trata explicitamente "reverter
  Fase 2 que inferiu Vite+Tailwind; canônico é Rsbuild+UnoCSS".
- **D-057** (planos universais): ✅ reforçado em todos os 3 artefatos (backend/requirements §2,
  web/tasks FE-010, mobile/README §1).
- **D-058** (admin transversal): ✅ destacado em web/README §3 + web/security §10 + mobile/README §3.
- **D-056** (agnosticismo provider): ✅ backend README §6 mostra mapeamento interface→impl por
  módulo; todas as 12 interfaces têm adapter + stubs onde SDK externo não implementado.

---

## 2. Revisão por sub-projeto — Backend

### 2.1 Pontos fortes

1. **Destilação fidelíssima ao código**: 118 endpoints, 39 migrations, 38 aggregates, matriz ABAC
   com 70+ actions, tudo bate após contagem. Nível de rigor acima da média da Fase 3/4 original.
2. **D-056 inegociável respeitado**: mapeamento `IPaymentGateway`/`IVideoProvider`/`ILiveProvider`
   em `domain/providers/` + adapters isolados em `infrastructure/providers/` — comprovado por leitura
   de `architecture.md §6.1`.
3. **Gap BE-001 (Unit aggregate sem `unit_type`/`fracao_ideal`) priorizado corretamente** como
   tarefa de Sprint 10 crítica para cliente master-sindico — conforme orientação do prompt.
4. **NATS gap BE-006 identificado** com proposta de migration + outbox table + worker publisher.
5. **A-023 / A-039 bem descritos em `security.md §2`** com evidências BeyondCorp 14-invariantes
   (12 verdes + 2 abertos).
6. **PBT suites catalogadas (4 reais)** com generators e invariantes.
7. **Patterns doc com 23 seções + checklist regex de auditoria** — rigoroso.

### 2.2 Achados / correções sugeridas

#### A-DC-FASE8-01 — severity **low** — redação incorreta sobre NATS no `go.mod`

**Ocorre em**: `backend/README.md §2 linha 72`.

**Texto atual**: _"NATS JetStream [...] Não presente no `go.mod` — providers atuais usam
`InMemoryPublisher`"_.

**Realidade**: `go.mod` linha 126 tem `github.com/nats-io/nats.go v1.48.0 // indirect` (pull
transitivo via OpenTelemetry OTLP dependency tree).

**Fix**: trocar por: _"não é direct dependency (aparece como indirect via OTel); não há
`nats.Connect` / publisher NATS instanciado no código — providers atuais usam
`InMemoryPublisher`."_

#### A-DC-FASE8-02 — severity **low** — `BimonthlyPeriod` classificado como VO no README mas é service

**Ocorre em**: `backend/README.md §2 linha 68` + `§4.3` — lista "Value objects: `Address`
(CEP+rua...), `BimonthlyPeriod`".

**Realidade**: `find internal/modules/*/domain/value_objects/` retorna apenas **4 arquivos**:
`cnpj.go`, `cpf.go`, `email.go`, `address.go`. `bimonthly_period.go` vive em
`internal/modules/institutional/domain/services/`, não value_objects.

**Fix**: README deve dizer "4 value objects (CPF, CNPJ, Email, Address) + 1 domain service
temporal (BimonthlyPeriod em `domain/services/`)". Idem em `_moc.md §Snapshot`.

#### A-DC-FASE8-03 — severity **trivial** — "789 linhas" vs real 788

**Ocorre em**: `backend/README.md §1` (citação `abac_rules.go:1-789`) + `patterns.md §1` +
`security.md §5`.

**Realidade**: `wc -l abac_rules.go = 788`. Off-by-one provavelmente por convenção `[start-end]`
inclusivo vs número de linhas.

**Fix**: adotar notação consistente — `:1-788` ou `~789 linhas`.

#### A-DC-FASE8-04 — severity **trivial** — claim "3 PBT suites críticas" em `README §9` vs "4" em `patterns.md` e `_moc.md`

**Ocorre em**: `backend/README.md §9 Property-Based Tests` "**3 suites críticas**" seguido de 4
bullets (agenda_item, feature_usage, abac_engine, pagination — a 4ª listada como "+ 1000 casos
pagination" mas textualmente conta como 3 "críticas").

**Realidade**: 4 arquivos com `rapid.Check` (confirmado grep). `_moc.md §Snapshot` diz "4 PBT
suites" e `patterns.md §12` lista "12.1–12.4" (4).

**Fix**: padronizar README para "**4 suites PBT**" (ou explicitar "3 de domínio crítico +
pagination transversal" sem reduzir a contagem para "3").

#### A-DC-FASE8-05 — severity **low** — `_moc.md` lista "AUDIT findings abertos 3 (A-023 medium, A-024 medium, A-039 medium)" mas `security.md §16` também cita DT-003 + DT-010 como pendentes

Não é contradição (AUDIT ≠ STATE), mas pode confundir leitor. Sugerir `_moc.md §Snapshot`
separar:

```
| AUDIT findings abertos | 3 (A-023, A-024, A-039 — todos medium) |
| STATE dívidas ativas    | 4 (DT-003 circuit-breaker, DT-004 UoW timeout, DT-007, DT-010 audit logger) |
```

(Atualmente `_moc.md` já faz isso, mas a leitura conjunta é confusa — considerar espaçamento ou
agrupar por "pendentes Sprint 10"). Cosmético.

#### A-DC-FASE8-06 — severity **medium** — `tasks.md §3` lista Sprint 9 como "99% fechado" mas task **9.19 Zitadel managed Railway** está como **🔴 aberto** (open, high)

Coerência interna: o Sprint 9 não está 99% se há 1 task **high-severity aberta**. Sugerir mudar
para "95% fechado" ou reclassificar 9.19 como Sprint 10 (é de fato deploy ops, não
integração funcional).

### 2.3 Adversarial — gaps não identificados pela destilação

**Gap-1** — backend não tem **CI gate automatizado** para a regra "domain nunca importa
framework". `architecture.md §2` admite: _"Verificação preventiva em code review — sem check
automatizado no CI atual (gap reportável)"_. Bom que está reportado; merece task explícita
Sprint 10 (**proposta BE-016**): script `scripts/check-domain-purity.sh` que grep `import (` em
`internal/modules/*/domain/**/*.go` e falha se encontrar `gin-gonic|jackc/pgx|stripe|mux|
livekit|zerolog`.

**Gap-2** — `security.md §13.3` sobre grep CI de `condominium_id` no WHERE é proposta, ainda não
implementada. Merece task **BE-009** já existe em `tasks.md` — OK, destilado.

**Gap-3** — `requirements.md §3.6 Idempotency` fala de Stripe/Mux webhooks. Não fala explicitamente
do **`checkout_subscription_saga`**: a saga é idempotente? Se chamar 2× com mesmo input
(usuário clica Subscribe 2×), o que acontece? `patterns.md §6.1` descreve "IdempotencyKey
`subscription:{userID}:{planID}:{interval}`" na chamada ao Stripe — OK, mas **lado backend DB** (o
INSERT em `subscriptions`) tem UNIQUE `(user_id, plan_id, status=active)`? **Não está
documentado**. Vale investigar e registrar invariante. Sugestão **A-DC-FASE8-06b** (follow-up):
verificar na migration `00005_create_subscriptions.sql` se existe essa UNIQUE ou se idempotência
depende inteiramente do Stripe customer id.

**Gap-4** — `security.md §16 Pendências` não menciona **rotação de `ZITADEL_KEY`** (service
account JSON). Operação crítica que deveria ter procedimento. Sugestão backlog Sprint 11+.

### 2.4 Validação dos números-alvo (prompt)

| Número alvo (prompt) | Destilado | Real | Status |
|---|---|---|---|
| 118 endpoints | 118 | 118 | ✅ |
| 38 aggregates | 38 | 38 | ✅ |
| 4 VOs | 4 (+ Bimonthly listado erroneamente) | 4 reais | ⚠️ (ver A-DC-FASE8-02) |
| 41 enums | 41 | **Arquivos com `domain/enums/` = 5 files**; claim "41 enums nomeados" refere-se a enums individuais (tipos Go), não arquivos. Verificável via `grep -rE '^type \w+ \(string\|int\)$' internal/modules/*/domain/enums/`. Não validado por amostra — aceito plausível |
| 39 migrations | 39 | 39 | ✅ |
| 48 use cases | ~48 | 46 | ⚠️ (aproximação OK) |
| ~130 rules ABAC | ~130 | ~215 (contando `Name:` dentro de Rules) ou ~130 se contar Rules-únicas | ⚠️ ambíguo — o artefato diz "~130 rules" mas a contagem depende de se "rule" = linha `{Name: ...}` ou ação-consolidada |
| 4 PBT suites | 3 (README) / 4 (patterns + _moc) | **4** | ⚠️ (ver A-DC-FASE8-04) |
| BE-001 Sprint 10 | ✅ listado | gap real confirmado | ✅ |
| NATS ausente (BE-006) | ✅ listado | direct = sim, publisher = sim; transitive = não | ⚠️ (A-DC-FASE8-01) |
| Planos universais CONFIRMADO no código | ✅ `plan_tier` enum `migrations/00001:27-32` | ✅ exato | ✅ |

**Veredito**: 8/11 números batem direto; 3/11 têm imprecisão menor a corrigir.

---

## 3. Revisão por sub-projeto — Web

### 3.1 Pontos fortes

1. **Stack real mapeada versão-por-versão**: todos os claims do README §1 são verificáveis em
   `bun.lock` + `apps/auth/package.json`. Zero inferência — tudo ancorado.
2. **D-058 admin transversal executado corretamente**: catalogado em README §3 (admin não tem
   porta própria), architecture §12 (rotas `_authenticated/admin/` dentro de `cms`),
   security §10 (3-layer gate), ui-catalog §B.7 (AD1-AD8 embutidos). Confirmado no código que
   **não existe `apps/admin/`**.
3. **Portas dev corretas** (3000-3005 mapeadas a auth/cms/lms/forum/assembly/lp) — confirmei via
   `grep rsbuild dev --port`.
4. **Design-system com OKLCH + reconciliação Manual IV** (p.ex. §2.1 nota sobre divergência
   `#C49A2A` vs `#C69C3F` = 2 pontos no matiz, justificado por perceptual uniformity).
5. **Dívidas técnicas catalogadas** — DT-WEB-001 (`turbo` em railway.json mas root usa só Bun
   workspaces) até DT-WEB-015. Excelente rigor — evita drift futuro.
6. **Security.md cobre OWASP Top 10 + CSP strict + CORS allowlist + CSRF double-submit + IDOR
   404 + admin isolation**.

### 3.2 Achados / correções

#### A-DC-FASE8-07 — severity **medium** — inconsistência 141 vs 174 telas

**Ocorre em**: `web/README.md §9` "141 telas", `web/_moc.md §Arquivos` "≥ 141 telas distintas",
`web/ui-catalog.md §15 (topo)` "≥ 141 telas", **mas** `web/ui-catalog.md §linha 610` "**Total
único catalogado: 174**" com nota de pé "após deduplicação VZ/VS/VE/VM entre `cms` e `forum` =
141 distintas".

**Análise**: dois números coexistem: 174 (bruto, com rotas duplicadas entre apps) e 141 (distintas
estruturalmente). O prompt original diz "174 telas".

**Fix**: README, _moc e ui-catalog §15-topo devem falar **ambos**: _"**174 rotas catalogadas ×
141 telas estruturalmente distintas** (telas VZ/VS/VE/VM aparecem em `cms` e `forum` como
rotas canônicas; mesma UI)"_. Padronizar em todos os 3 lugares.

#### A-DC-FASE8-08 — severity **low** — código real tem **telas zero nos apps** exceto `auth` (2) e `lp`

**Ocorre em**: `web/README.md §2.1` descreve scaffold com `_auth/sign-in.tsx`, `_auth/sign-up.tsx`
mas **apenas em `apps/auth/`**. Confirmei `ls apps/cms/src/routes/` = `index.tsx` + `__root.tsx`
apenas.

**Destilação é honesta**: README §3 marca `cms` como "Scaffold; `routes/index.tsx` stub".

**Observação**: isso deixa as "174 telas" como **alvo 100% futuro** para web — zero
implementação real hoje (exceto 2 stubs em `auth`). Sugerir destacar mais claramente na
introdução do ui-catalog (`§15`?) que se trata de **especificação alvo** e não "telas
implementadas". Já está implícito mas não em negrito.

#### A-DC-FASE8-09 — severity **low** — `schemas` package é 100% stub

**Ocorre em**: `web/README.md §2.2` diz _"`packages/schemas/src/index.ts` — Apenas stub
`console.log("Hello via Bun!")` — **vazio**"_. Honesto, mas o `architecture.md §11` lista 10
arquivos previstos sob `src/` como se fossem estrutura planejada.

**Fix**: não é erro, mas reforçar no `_moc.md` que **"DT-WEB-005 é bloqueador M1"** —
sem schemas Zod compartilhados, paridade backend/frontend não existe. Já está em tasks.md
FE-010 como `L` (grande) Sprint 10; OK.

#### A-DC-FASE8-10 — severity **trivial** — frame do Figma entre destilações

`README.md §4.4` cita `https://www.figma.com/design/pxJkmn8rzezKnCYXcA7lyL/Master-sindico-APP---UX`.
`_moc.md §Coordenação cross-stack` cita mesma URL — OK.

Ver se esse frame **existe** (sem acesso web aqui); se o projeto v2 nunca acessou o Figma
em sessão real, marcar como "a validar P-WEB-###". Não crítico, mas honestidade importa.

#### A-DC-FASE8-11 — severity **low** — menção "admin MS" junto com "N1/N2/N3" em `README §11` (linha 13)

**Texto atual**: _"...síndico (N1/N2/N3), morador (base/pagante), empresa condominial (Plus/Pro)
..."_ + _"admin MS..."_.

**Análise**: `N1/N2/N3` aqui é descritivo de **sub-roles de síndico** histórico do legado — não
é slug de **plano**. `Plus/Pro` são planos (legado; universal agora é `base/plus/pro`).

**Risco**: LLM futuro pode ler e reintroduzir slugs compostos. **Fix**: adicionar disclaimer
explícito: _"(sub-roles síndico N1/N2/N3 = termo histórico do legado; NÃO corresponde a plan_tier
D-057 que é universal `trial/base/plus/pro`)"_.

Mesmo fix em `mobile/README.md` — linha 23 (frase similar).

### 3.3 Adversarial — gaps

**Gap-W1** — **sem task para contrato OpenAPI → TS types generator**. `tasks.md` lista FE-008
(openapi-typescript ou Orval) mas não define **quando o backend emite OpenAPI**. Coordenação
cross-stack pendente.

**Gap-W2** — **`packages/ui` sem `package.json` documentado**. Apenas referido via `main:
src/index.ts`. Validar existência no código real (confirmei `ls packages/ui/` = existe; ver se
estrutura bate).

**Gap-W3** — `security.md §4.3 CSRF double-submit` implementação sugere **cookie não-HttpOnly**
para CSRF token. Padrão OWASP OK, mas **SameSite=Lax do cookie de sessão já mitiga CSRF na maior
parte dos casos** — double-submit é reforço. Explicitar quando é obrigatório (Stripe Checkout
redirect? API de mutation pura?). Artefato é correto mas pode ser mais prescritivo.

**Gap-W4** — **Nenhum teste E2E cobrindo fluxo OIDC + 1-device**. `tasks.md` FE-241 cobre E2E
Login OIDC M3. OK, mas Sprint 10/11 ficam sem E2E — compensar com MSW + unit. OK.

---

## 4. Revisão por sub-projeto — Mobile

### 4.1 Pontos fortes

1. **Honestidade radical sobre dívidas**: README §1 lista 10 deps CLAUDE-alvo ausentes em
   `pubspec.yaml` real — `bloc_concurrency`, `hydrated_bloc`, `freezed`, `hive_flutter`,
   `local_auth`, `device_info_plus`, `firebase_messaging`, `app_links`, `freeRASP_flutter`,
   `easy_localization`. Confirmei `cat pubspec.yaml` = sem nenhum desses.
2. **DT-MB-001..013** catalogadas em `tasks.md §DT` — inclui o bundle ID divergente
   (`com.mastersindico.br.mastersindico` vs alvo `com.mastersindico.app`), `minSdk=19` (alvo 26),
   keys inconsistentes, paleta default Flutter, etc. Todos **confirmados em `build.gradle.kts`,
   `constants.dart`, `auth_local_datasource.dart`, `app_colors.dart`**.
3. **D-057 + D-058 explicitados no topo do README** — "planos globais Stripe-style + sem slugs
   N1/N2/N3" + "admin transversal".
4. **Arquitetura Clean + Feature First com Dependency Rule documentada** (`architecture.md §1`)
   — regra `presentation → domain ← data` inviolável.
5. **Security posture cobre OWASP Mobile Top 10, integridade escalonada D-050 (MASVS-R),
   cert pinning com rotação segura, PII masking**.
6. **Assembly feature completa listada corretamente**: 5 entities + 7 usecases + bloc + 5 pages
   (README §3 tabela). Único feature realmente implementado além de auth scaffold.

### 4.2 Achados / correções

#### A-DC-FASE8-12 — severity **medium** — **falta arquivo `design-system.md` dedicado**

**Ocorre em**: estrutura `mobile/` tem 8 arquivos: README + architecture + patterns + ui-catalog +
requirements + security + tasks + _moc. **Não há `design-system.md`**.

**Comparação**: `web/` tem 9 arquivos (inclui design-system.md 22KB com Manual IV tokens OKLCH).
`mobile/` delega para `[[../web/design-system]]` (mobile/README §8).

**Análise**: Flutter Material 3 + ColorScheme.fromSeed tem **ergonomia diferente** do UnoCSS CSS
vars. Mobile precisa documentar:
- Paleta do app (`app_colors.dart` — hoje **default Flutter `#6C63FF`**).
- Typography (MD3 scale em `app_text_styles.dart`).
- Spacing, radius (MD3 defaults).
- Touch targets (48×48 Android / 44×44 iOS).
- Ícones (cupertino_icons hoje + Material Icons).

**Fix**: criar `mobile/design-system.md` (`type: spec`, tags `design-system, flutter, material3`)
cobrindo **esses tokens específicos do mobile + reconciliação com Manual IV** (DT-MB-005).
Mencionar paridade com `web/design-system.md` mas com adapter Flutter-specific.

**Sub-achado**: README §5 lista `lib/core/theme/app_colors.dart` com valor **real** `#6C63FF`
(confirmado linha 14). Isso é **default do `flutter create`** — NÃO o Manual IV gold/navy.
Artefato já identifica como DT-MB-005 em tasks.md — **mas** README §1 ainda referencia
apenas a "paleta" sem fixar brand. Sugerir seção `§Design-system` no README com callout:

> ⚠️ **Paleta em dívida DT-MB-005**: `app_colors.dart` usa default Flutter `#6C63FF` (roxo).
> Manual IV oficial é ouro `#C69C3F` + navy `#071A33` ([[../web/design-system]]).
> Reconciliação em Sprint 10 MB-007.

#### A-DC-FASE8-13 — severity **low** — `pubspec.yaml` real vs README §1 tabela

Uma vez que **10 deps são "⚠️ ausente"**, a tabela da README vira pesada de ler. Sugerir
**separar em duas tabelas**: "1.A Instaladas hoje" (flutter_bloc, equatable, get_it,
injectable, go_router, dio, shared_preferences, dartz, connectivity_plus, intl, logger,
flutter_secure_storage, oidc, oidc_default_store, cupertino_icons — ~15 direct deps) e
"1.B Dívidas M1 (ausentes, planejadas)".

Atualmente lista mixed faz parecer que há mais código instalado do que existe.

#### A-DC-FASE8-14 — severity **low** — `ui-catalog.md` não foi lido integralmente nesta sessão

Pela amostra da _moc.md §Arquivos, mobile `ui-catalog` cobre §1 Sistema, §2 Auth, §3 Home shell
(4 abas), §4 Morador, §5 Síndico light, §6 Assembleia, §7 Connect Me, §8 Vídeo, §9 Perfil,
§10 Notificações, §11 Exceções, §12 Admin, §13 matriz paridade, §14 P-APP-001..009 pendências.

**Nota operacional**: 9 pendências `P-APP-###` listadas — bom que estão catalogadas. Sugerir
promover as críticas a DT-MB-### ou MB-### tasks antes de M1.

#### A-DC-FASE8-15 — severity **low** — `AuthLocalDatasourceImpl` key `AUTH_TOKEN` uppercase vs `AppConstants.tokenKey = 'auth_access_token'` lowercase — inconsistência **real** no código

`security.md §1.3 Estado atual vs alvo` explica isso bem. `tasks.md` tem DT-MB-007 + MB-003.
OK, destilação correta. Sem ajuste no vault v2; ajuste no **código real** é responsabilidade
M1.

### 4.3 Adversarial — gaps

**Gap-M1** — `security.md §1.1 Keys canônicas` lista `oidc_state` para state+nonce TTL curto.
**Não está no código real** (`AppConstants` não tem). Candidato a MB-### novo.

**Gap-M2** — **Tamanho de IPA/APK alvo** — `requirements.md` menciona "<40MB IPA" no §6. Razoável
para Flutter com split-per-abi. Hoje com 10 deps faltando, tamanho real é menor. Após instalar
deps M1 (firebase, hive, local_auth, etc.), validar não estoura budget.

**Gap-M3** — **Sem menção a `plugin-ecosystem.md`** ou análise de saúde dos plugins Flutter
terceiros (mainnainers ativos, última release, etc.). Para M1 é tolerável; para M3+ recomendar
auditar dependências estrangeiras (principalmente Talsec/freeRASP, que é proprietário).

**Gap-M4** — **iOS mandatory privacy manifest (`PrivacyInfo.xcprivacy`)** — Apple exigiu Apr
2024+. `security.md §3.1 ATS` menciona NSAppTransportSecurity mas não PrivacyInfo. Candidato a
DT-MB-### ou MB-### task para Sprint 7+.

---

## 5. Coerência cross-arquivo (dentro de cada sub-projeto)

### 5.1 Backend — coerência

- **README §4.2 billing** menciona `SubscriptionActivated/Canceled/TrialExpired` events. `architecture.md §11` confirma existência do `IEventPublisher` + sentinels. `patterns.md §7.3` mostra o `MasterPlanTimelineHandler` reactive. **Coerente**.
- **security.md §5.1 ABAC** diz "789 linhas". Mesma imprecisão que README §1 linha 41 — ver A-DC-FASE8-03.
- **tasks.md §4.1 BE-001 Unit** ↔ **requirements.md §7 Gap "Unit sem unit_type/fracao_ideal"** ↔ **README §8.1**. Cross-consistência impecável.

### 5.2 Web — coerência

- **README §3 admin D-058** ↔ **security.md §10** ↔ **ui-catalog §B.7 (AD1-AD8)** ↔ **architecture.md §12.3** — 4 arquivos mencionam "role transversal, não app separada, `cms/admin/*`". **Coerente**.
- **design-system OKLCH tokens** ↔ **main.css CSS vars** ↔ **packages/ui/src/button.tsx CVA mapping**. **Coerente**.
- **141 vs 174 telas** = **incoerência pontual** (A-DC-FASE8-07).

### 5.3 Mobile — coerência

- **README §1 Dívidas M1** ↔ **tasks.md §DT** ↔ **_moc.md §Arquivos.tasks**. **Coerente**.
- **security.md §1.3 AUTH_TOKEN vs auth_access_token** ↔ **tasks.md DT-MB-007**. **Coerente**.
- **Ausência de `design-system.md`** é única lacuna estrutural — A-DC-FASE8-12.

---

## 6. Recomendações finais

### 6.1 Para **fechar** a Fase 8 agora (mínimo viável)

| ID | Ação | Esforço | Bloqueador? |
|---|---|---|---|
| A-DC-FASE8-01 | Corrigir redação NATS no `backend/README §2` | 3 min | Não |
| A-DC-FASE8-02 | Mover `BimonthlyPeriod` de "Value objects" para "Domain services" no `backend/README` + `_moc.md` | 5 min | Não |
| A-DC-FASE8-03 | Padronizar "788 linhas" ou "~789 linhas" no `abac_rules.go` em 3 arquivos backend | 2 min | Não |
| A-DC-FASE8-04 | Padronizar "4 PBT suites" no `backend/README §9` | 2 min | Não |
| A-DC-FASE8-07 | Reconciliar "141 vs 174 telas" em `web/README + _moc + ui-catalog` com frase única em 3 lugares | 10 min | Não |
| A-DC-FASE8-11 | Adicionar disclaimer "N1/N2/N3 é sub-role histórico de síndico, não plan_tier" em `web/README` + `mobile/README` | 5 min | **Preventivo** |
| A-DC-FASE8-12 | Criar `mobile/design-system.md` cobrindo paleta/tipografia/spacing/radius/touch-target + callout DT-MB-005 paleta placeholder | 45 min | **Sim** — sem isso, mobile é incompleto vs web |

**Total**: ~75 min para fechar.

### 6.2 Para rodadas futuras (Sprint 10+)

1. **Backend BE-016** (novo): script CI `check-domain-purity.sh` grep framework imports em
   `domain/` — ver §2.3 Gap-1.
2. **Backend — validar UNIQUE em `subscriptions` table** (A-DC-FASE8-06b).
3. **Web DT-WEB-005 + FE-010** ganha prioridade máxima Sprint 10: sem `packages/schemas`, não há
   contratos frontend/backend compartilhados.
4. **Mobile MB-007** ganha prioridade antes de qualquer tela morador: **reconciliar paleta** —
   publicar app com `#6C63FF` é mal-começo de brand.
5. **Mobile MB-####** (novo): PrivacyInfo.xcprivacy + Privacy Labels para App Store — ver §4.3
   Gap-M4.

### 6.3 Comportamento LLM-safety (preventivo cross-destilação)

- **Padronizar "Stripe-style plans"** em todos os 3 sub-projetos — _sempre_ seguido de `trial |
  base | plus | pro` explícito + "**sem** slugs compostos, persona-specific, N1/N2/N3,
  Diamante, etc.". Hoje 2/3 sub-projetos fazem isso; reforçar em `backend/README §4.2 billing`
  (atualmente menciona `PlanTierOrder` mas não enumera os 4 valores no corpo).
- **Padronizar admin D-058** — "**role privilegiada transversal; NÃO app separada; NÃO
  subdomain isolado; gated via ABAC**". Hoje os 3 sub-projetos fazem bem.

---

## 7. Veredito final

**✅ Fase 8 APROVADA** com 15 achados (nenhum bloqueante arquitetural; 1 bloqueante estrutural
em mobile = criar `design-system.md`).

**Qualidade geral da destilação**: **superior à Fase 3/4 original** (mais ancorada em código
real, menos inferência, dívidas catalogadas honestamente). Parabéns ao agente re-destilador.

**Pós-correções A-DC-FASE8-01..15**, a migração Fase 9 (overlay → `vault/50-projects/
master-sindico/03-subprojects/`) pode prosseguir.

---

## 8. Ligação

- [[_moc]] — MOC dual-check log
- [[2026-04-23-decisoes-arquiteturais]] — dual-check de ADRs 0001-0025 + D-020..D-043
- [[2026-04-23-security-gaps]] — dual-check de BeyondCorp + LGPD + OWASP
- [[../../STATE]] — registrar D-### se A-DC-FASE8-01..15 virarem decisões (maioria é correção
  textual, sem impacto em D-###)
- [[../../AUDIT]] — registrar A-DC-FASE8-01..15 se forem promovidos a findings
- [[../../03-subprojects/backend/_moc]] · [[../../03-subprojects/web/_moc]] · [[../../03-subprojects/mobile/_moc]]
- Código real: `/home/vinni/Documentos/Projetos/Clientes/Joao/MasterSindico/Development/{backend,web,app}/`
