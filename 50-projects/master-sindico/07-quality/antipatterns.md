> **Ver knowledge global atemporal** — catálogo geral de antipatterns cross-projeto: [[../../../10-knowledge/antipatterns/_moc]].
>
> Este arquivo lista **antipatterns descobertos no Master Síndico** (AP-### específicos do projeto, com referências a código/ADRs/findings). Complementa o catálogo global.

---

title: Antipatterns catálogo AP-### — Master Síndico
type: antipattern
tags: [antipatterns, master-sindico, v2, quality, legado]
source: 90-ingested/from-archive-specs-consolidation/ANALISE_COMPLETA.md + vault/10-knowledge/antipatterns + 90-ingested/from-vault-50-projects-master-sindico/11-audit/AUDIT.md
created: 2026-04-23
updated: 2026-04-23
aliases: [master-sindico-antipatterns]
---

# Antipatterns catálogo AP-### — Master Síndico v2

Catálogo dos antipatterns detectados no legado TypeScript (Fastify + Drizzle) do Master Síndico e antipatterns universais aplicáveis ao rewrite em Go. Cada AP-### tem: **ID**, **título**, **o que é**, **onde ocorreu** (TS legado documentado quando há), **por que é ruim**, **mitigação** (pattern Go correto), **detecção** (lint/grep/review).

> Fontes: `90-ingested/from-archive-specs-consolidation/ANALISE_COMPLETA.md` (30-fev-2026, 47 problemas identificados), findings legados A-### em [[../11-audit/AUDIT]], catálogo universal [[../../vault/10-knowledge/antipatterns/_moc]]. IDs AP-### aqui são **específicos do projeto** (podem sobrepor numeração universal). Pattern canônico de fix em [[PATTERNS]].

---

## AP-001 — God Object (Subscription 450 linhas)

- **O que é**: aggregate/classe com responsabilidades múltiplas, tamanho > 200 linhas, múltiplas dependências externas.
- **Onde ocorria**: legado `full-stack/apps/api/src/modules/billing/domain/entities/subscription.ts` (450 linhas) agregando trial, ciclo, quotas, integração Stripe e lógica de plano.
- **Por que é ruim**: viola SRP; testes ficam frágeis; mudança em feature nova impacta 5 comportamentos; impossível reusar partes.
- **Mitigação Go**: aggregate `Subscription` ≤ 200 linhas; extrair `Plan` (VO), `BillingCycle` (VO), `TrialPolicy` (strategy), `Quota` (VO). Stripe via `IPaymentGateway` port.
- **Detecção**:
  - `gocyclo -over 15 ./internal/modules/billing/domain/...`
  - `wc -l domain/entities/*.go` em CI; fail se > 200.
  - Code review — "este aggregate fala com Stripe?" → blocker.

---

## AP-002 — Invoice 550 linhas (mesmo padrão)

- **O que é**: variante de AP-001; entidade `Invoice` mistura geração PDF, cálculo impostos, envio email, persistência.
- **Onde ocorria**: legado `full-stack/apps/api/src/modules/billing/domain/entities/invoice.ts` (550 linhas).
- **Por que é ruim**: PDF/email/cálculo são concerns de infra/application — poluíram domain.
- **Mitigação Go**: `Invoice` domain aggregate enxuto (state + transitions). PDF via `IInvoicePDFGenerator` port; email via `IEmailProvider`; cálculo em `TaxCalculatorService` (domain service puro).
- **Detecção**: mesma de AP-001 + grep por `import.*pdf` em `domain/`.

---

## AP-003 — Vazamento de ORM

- **O que é**: repo devolve tipo gerado pelo ORM (`pgtype.UUID`, `sqlc.GetUserRow`, `*drizzle.User`) pra camadas superiores.
- **Onde ocorria**: legado Drizzle retornando `typeof users.$inferSelect` direto para use cases.
- **Por que é ruim**: trocar ORM vira refactor massivo; tipos ORM têm nullable patterns do driver que poluem domínio.
- **Mitigação Go**: **mapper explícito** em `infrastructure/database/mappers.go`. Repo retorna `*domain.User`, nunca `*sqlcgen.User`.
- **Detecção**: grep CI `func.*Repo.*\) [A-Z].*sqlcgen\.` em `repositories/` → fail; `*pgtype\.` em retorno de repo → fail.

---

## AP-004 — N+1 queries

- **O que é**: loop que faz 1 query por item quando 1 query com `IN (...)` resolveria.
- **Onde ocorria**: legado `getPublicProfileUseCase` busca em 5 repos sempre (P1.3 do ANALISE_COMPLETA), mesmo quando só 1 role se aplica.
- **Por que é ruim**: latência linear com tamanho da lista; DB fica CPU-bound; p95 degrada sob carga.
- **Mitigação Go**:
  - Usar `WHERE id = ANY($1)` com array.
  - Coluna discriminadora + 1 query com `CASE`/join.
  - `sqlc` batch query se repetitivo.
- **Detecção**: APM traces (OpenTelemetry) — alerta quando `query_count / request > 10`. Code review manual.

---

## AP-005 — Cache sem versionamento

- **O que é**: key de cache não inclui versão do DTO/permissões/schema; cache stale permite decisão com dados velhos.
- **Onde ocorria**: legado `my_profile:${userId}` sem versão (P1.1), `permission_cache` sem hash do plano (P2.2).
- **Por que é ruim**: deploy muda shape do DTO → cache antigo quebra deserialização; admin muda permissão → user fica com ABAC velho por 5min.
- **Mitigação Go**:
  - `const CacheVer = "v3"` na key: `fmt.Sprintf("profile:%s:%s", CacheVer, userID)`.
  - Hash permissões/plano na key: `profile:v3:%s:%s` (userID, sha256(permissions)).
  - Invalidação via webhook Stripe (`customer.subscription.updated` → del keys do user).
- **Detecção**: code review; grep por key literal sem const `Ver`.

---

## AP-006 — Rate limit sem cleanup (memory leak)

- **O que é**: token bucket `sync.Map` acumula entry por IP/user sem expirar entries inativas.
- **Onde ocorria**: legado middleware Fastify e early Go backend (A-032 AUDIT).
- **Por que é ruim**: memória cresce O(unique IPs); OOMKill inevitável em horizonte de semanas.
- **Mitigação Go**: goroutine cleanup com `time.Ticker(5*time.Minute)` que remove entries com `lastSeen < now - 10min`. Teste unit simulando N IPs únicos e validando GC.
- **Detecção**: load test k6 1h de tráfego → `process_resident_memory_bytes` deve estabilizar; teste unit com FakeClock avançando 1h → `Size() == 0` se zero tráfego.

---

## AP-007 — Logs com PII (legado A-003/A-004)

- **O que é**: CPF/CNPJ/email/phone/token/password em log.
- **Onde ocorria**: legado logging completo do body da request (`fmt.Printf("user: %+v", u)`).
- **Por que é ruim**: LGPD §14 viola (retenção indevida); forensics cross-leak; logs em observability SaaS saem da infra confiável.
- **Mitigação Go**: VOs têm `Masked() string` (ex: `c***@d.com`, `***.***.***-99`); structured logger (`zerolog`) usa campos (`.Str("user_id", ...)` em vez de `%+v`).
- **Detecção**:
  - Regex CI: `\b\d{3}\.\d{3}\.\d{3}-\d{2}\b` ou variantes em `*.go` não-teste.
  - Regex: `logger.*%\+v` em `*.go`.
  - Custom analyzer `go vet` com rule "PII in log args".

---

## AP-008 — Inconsistência estrutural entre módulos

- **O que é**: módulo A tem `services/`, módulo B tem `helpers/`, módulo C tem ambos — onboarding dev fica lento.
- **Onde ocorria**: legado misturado TS/entidades/services/helpers sem convenção rígida.
- **Por que é ruim**: cada BC inventa organização; grep cross-module quebra; code review pede redundantemente a mesma coisa.
- **Mitigação Go**: **layout canônico obrigatório** (ver [[CLEAN_ARCH]] §2); template `40-templates/project-scaffold/` força layout; novo BC é cópia do template.
- **Detecção**: CI valida existência das pastas `domain/`, `application/`, `infrastructure/` em cada `internal/modules/*/`.

---

## AP-009 — Ausência de testes automatizados

- **O que é**: código entrou em prod sem cobertura mínima.
- **Onde ocorria**: legado TS com coverage variável (algumas áreas < 20%).
- **Por que é ruim**: refactor arriscado; bugs silenciosos (race conditions, TOCTOU) só aparecem em prod.
- **Mitigação Go**: thresholds duros em CI (ver [[../09-testing/coverage-thresholds]]); TDD obrigatório ([[../CLAUDE]] §8.5); PBT em invariantes críticos ([[../09-testing/pbt]]).
- **Detecção**: `go tool cover -func=coverage.out` + parser YAML em CI bloqueia merge.

---

## AP-010 — Billing sem use cases (lógica no handler/serviço de infra)

- **O que é**: regra de negócio billing vive em handler HTTP ou serviço de infra.
- **Onde ocorria**: legado `quota.service.ts` tinha ownership rules hardcoded (P2.1, P2.2).
- **Por que é ruim**: impossível testar sem subir HTTP; regras espalhadas; ABAC fica inconsistente.
- **Mitigação Go**: toda regra em `application/use_cases/*` com CQRS separado; domain services puros para cálculos; ABAC centralizado em `cross-domain/abac/engine.go`.
- **Detecção**: code review — handler com > 5 linhas de lógica condicional = blocker.

---

## AP-011 — If/else hell 5+ cases (Strategy missing)

- **O que é**: `switch` com 5+ cases, cada um chamando método privado similar — Open/Closed violado.
- **Onde ocorria**: legado `UpdateProfileUseCase.execute()` com switch de 5 roles + 4 métodos duplicados.
- **Por que é ruim**: adicionar role novo requer modificar switch + criar método novo idêntico aos outros.
- **Mitigação Go**: Strategy registry:
  ```go
  var updaterRegistry = map[ProfileType]IProfileUpdater{
      ProfileTypeResident: ResidentUpdater{},
      ProfileTypeSindico:  SindicoUpdater{},
      ...
  }
  ```
  Ver [[PATTERNS]] §8.
- **Detecção**: `gocyclo -over 15`; `revive` rule `cognitive-complexity`; code review marca switch 5+ como blocker.

---

## AP-012 — Duplicação de autorização (authorize() em 4 métodos)

- **O que é**: `authorize()` chamado em 4 métodos privados — lógica ABAC espalhada; mudança precisa editar N lugares.
- **Onde ocorria**: legado `UpdateProfileUseCase` chamava `this.authorize("update", profile)` em 4 métodos.
- **Por que é ruim**: DRY violado; risco de esquecer em método novo; inconsistência de política.
- **Mitigação Go**: helper privado `validateAndAuthorize(ctx, action, resource)` OU middleware `ABAC(action)` aplicado na rota; use case nunca chama `authorize()` — confia que middleware já decidiu.
- **Detecção**: grep `authorize(` > 1 no mesmo use case → code review.

---

## AP-013 — Inconsistência no fluxo de authorize()

- **O que é**: alguns métodos autorizam **antes** de buscar entidade, outros **depois**; bugs de segurança latentes (IDOR potencial).
- **Onde ocorria**: legado `updateResident` autorizava na linha 133, `updateEnterprise` na linha 157 — posições diferentes no fluxo.
- **Por que é ruim**: mix de autorização baseada em input vs autorização baseada em recurso; IDOR se o recurso carregado não bate com quem foi autorizado.
- **Mitigação Go**: sequência canônica **imutável**:
  ```
  Fetch → Validate → Authorize(onResource) → Mutate → Save → PublishEvents
  ```
  Middleware + use case mantêm ordem idêntica. Ver nota atômica [[../../../10-knowledge/security/security-principles|authorization-flow-order]].
- **Detecção**: code review; pattern check via linter customizado.

---

## AP-014 — Logging inconsistente (contexto pobre em alguns pontos)

- **O que é**: alguns logs têm `user_id`, outros `profile_id`, outros só mensagem — dificulta trace.
- **Onde ocorria**: legado `UpdateProfileUseCase` usava `userId` em uma linha, `profileId` em outra.
- **Por que é ruim**: correlation impossível; dashboards Grafana precisam fallback múltiplo.
- **Mitigação Go**: logger estruturado com campos canônicos em middleware (`request_id`, `user_id`, `tenant_id`, `bc`) injetado via `context.Context`; use case nunca loga sem `ctx`.
- **Detecção**: linter custom que valida log call tem `.Ctx(ctx)` ou `.Str("request_id", ...)`.

---

## AP-015 — ABAC cache sem tenant hash

- **O que é**: cache ABAC key `abac:{userID}:{action}` sem tenant — cross-tenant se user tem múltiplos memberships.
- **Onde ocorria**: legado `permission.service.ts` (P2.2) — cache `permission:${userId}:${action}`.
- **Por que é ruim**: user muda tenant → decisão ABAC stale; em cenários multi-tenant, permite IDOR.
- **Mitigação Go**: key `abac:v1:{userID}:{tenantID}:{action}`; invalidação ao trocar tenant ativo + TTL 5min; ver [[../02-architecture/topology-multitenant]].
- **Detecção**: code review; teste integration cross-tenant (100+ casos, ver [[../09-testing/integration]]) valida.

---

## AP-016 — SELECT \* em queries (legado A-021)

- **O que é**: `SELECT * FROM users WHERE id = $1` em sqlc.
- **Onde ocorria**: 12 queries do legado usavam `*` (A-021).
- **Por que é ruim**: add coluna nova (ex: `password_hash`) vaza pra mapper; perf degrada com colunas blobs; refactor arriscado.
- **Mitigação Go**: **colunas explícitas** em todas as queries sqlc:
  ```sql
  -- name: GetUserByID :one
  SELECT id, cpf, email, role, created_at FROM users WHERE id = $1;
  ```
- **Detecção**: `grep -n 'SELECT \*' internal/modules/*/infrastructure/database/queries/*.sql` — fail em CI; exceção única: `SELECT COUNT(*)`.

---

## AP-017 — Mutação de estado em hooks/lifecycle

- **O que é**: lifecycle hook (ex: `onBeforeSave`) muta estado que deveria ser imutável pós-construção.
- **Onde ocorria**: legado hooks Drizzle modificando `updatedAt` em contextos onde o campo deveria ficar como estava.
- **Por que é ruim**: bug silencioso; invariantes violados sem trace.
- **Mitigação Go**: sem hooks implícitos; mutação só via métodos do aggregate; campos derivados são `computed` em getter, não armazenados.
- **Detecção**: code review — evitar bibliotecas com magic hooks; se usar, documentar em [[../02-architecture/adr/_moc]].

---

## AP-018 — Secrets em `.env` versionado (legado A-018)

- **O que é**: `.env` com keys reais commitado no git.
- **Onde ocorria**: legado tinha `zitadel-key*.json` e `.env` versionados em algumas branches (A-018).
- **Por que é ruim**: secrets expostos; rotate mandatório após exposure.
- **Mitigação Go**:
  - `.env` e `zitadel-key*.json` no `.gitignore` (hard rule).
  - CI com secret scanner (GitHub secret scanning / trufflehog).
  - `.env.example` com placeholders.
  - Prod usa Railway/AWS Secrets Manager, não arquivo.
- **Detecção**: `git log --all -p -- .env zitadel-key*.json` fail em CI; secret scanner bloqueia push.

---

## AP-019 — `_ = fn()` em I/O crítica sem log (legado A-001..A-008)

- **O que é**: descartar erro de I/O sem tratamento nem log.
- **Onde ocorria**: legado `_ = searchEngine.index(user)` ignora falha no SearchEngine (A-001..A-008, A-015, A-019).
- **Por que é ruim**: estado divergente (DB tem, search não tem); nenhum monitoring detecta; bug misterioso meses depois.
- **Mitigação Go**:
  - **Bloqueia merge**: nunca `_ = fn(ctx, ...)` para I/O.
  - Outbox pattern ([[PATTERNS]] §14) — publica evento `UserUpdated`, consumer indexa.
  - Se fire-and-forget for intencional: `if err := fn(...); err != nil { logger.Warn().Err(err)... }` + outbox retry.
- **Detecção**: grep CI `^\s*_\s*=\s*\w+\.(Index|Send|Publish|Save|Insert|Update|Delete)` → fail.

---

## AP-020 — Validação só no frontend

- **O que é**: regra de negócio validada no JS/Dart, backend confia.
- **Onde ocorria**: legado tinha validações Zod no web que não eram refletidas na API.
- **Por que é ruim**: API é verdade única ([[../STEERING]] §72, BR-072); cliente malicioso bypassa frontend trivialmente.
- **Mitigação Go**: toda regra em VO + use case. Frontend valida pra UX; backend valida pra segurança — dupla-layer.
- **Detecção**: contract test (ver [[../09-testing/e2e]]); code review — regra crítica sem test unit do use case = blocker.

---

## AP-021 — SELECT \* em sqlc (duplicata com AP-016 — manter por rastreabilidade)

- **O que é**: mesma natureza de AP-016, dedicado a sqlc-generated.
- **Mitigação Go**: idem AP-016; sqlc força schema explícito via `sqlc.yaml` `emit_exact_table_names: true` + queries `.sql` revisadas.
- **Detecção**: idem AP-016.

---

## AP-022 — PII em logs sem mascaramento (mesmo root de AP-007, variante sqlc)

- **O que é**: log de query parameters sem mascarar (`sqlc` debug mode logando CPF).
- **Onde ocorria**: `sqlc` com flag `debug` habilitada em staging logava parâmetros sem redigir.
- **Por que é ruim**: LGPD; retenção indevida via logs SaaS.
- **Mitigação Go**: desabilitar sqlc debug em prod; logger structured; VOs com `Masked()` antes de logar.
- **Detecção**: linter custom; grep de `LogLevel=debug` em config prod.

---

## AP-023 — Webhook sem HMAC (INV-018, INV-093)

- **O que é**: webhook inbound aceita body sem verificar assinatura HMAC do provider.
- **Onde ocorria**: legado inicial aceitava webhook Stripe/Mux sem verify (fixado antes do v2 mas documentado).
- **Por que é ruim**: MITM injeta eventos fake; "Subscription paid" falsificada ativa features premium; DoS via JSON gigante.
- **Mitigação Go**: middleware `VerifyStripeSignature(secret)` / `VerifyMuxSignature(secret)` **antes** de parse body:
  ```go
  func (h *StripeWebhook) Handle(c contracts.Context) error {
      body, _ := io.ReadAll(c.Request().Body)
      if !verifyStripeHMAC(body, c.Header("Stripe-Signature"), h.secret) {
          return c.Status(400)
      }
      // só agora json.Unmarshal
  }
  ```
  Ver [[PATTERNS]] §11.
- **Detecção**: contract test (fake request sem signature → expect 400); pentest checklist [[../08-security/owasp-top10]] A1 Broken Access Control.

---

## AP-024 — Moderação bypass

- **O que é**: vídeo atinge `published` sem passar por estágio `moderation`.
- **Onde ocorria**: legado tinha path pra admin publicar direto sem moderação (INV-058 exige sempre).
- **Por que é ruim**: conteúdo não moderado vaza; violação dos BRs [[../01-domain/business-rules]] BR-034.
- **Mitigação Go**: state machine no aggregate `Video` que rejeita transição `moderation → published`; só permite `moderation → approved → published`.
- **Detecção**: unit test valida todas as transições de state; INV-058 enforcement.

---

## AP-025 — Missing tenant isolation (cross-tenant leak)

- **O que é**: query sem `WHERE tenant_id = $1` — tenant X lê/muta dados de tenant Y.
- **Onde ocorria**: queries herdadas do legado multi-tenant sem escopo (A-021 / A-089 pendentes).
- **Por que é ruim**: IDOR multi-tenant; LGPD crítico; blocker absoluto.
- **Mitigação Go**: **convenção sqlc** — toda query de tabela de domínio recebe `tenant_id` como primeiro param; 100+ tests cross-tenant ([[../09-testing/integration]]) validam sistematicamente; PBT ABAC 400 casos ([[../09-testing/pbt]]).
- **Detecção**: grep CI em queries `WHERE` sem `tenant_id` em tabelas marcadas `tenant_scoped: true` em sqlc.yaml; integration test `cross_tenant_isolation_test.go` obrigatório por repo.

---

## AP-026 — Race condition em UoW (TOCTOU em vote/evaluation)

- **O que é**: check-then-act não atômico: `HasVoted(x)` + `CreateVote(x)` fora de tx; 2 requests simultâneas criam 2 votes.
- **Onde ocorria**: legado `CastVoteUseCase` (A-025), `CreateCompanyEvaluationUseCase` (A-029), `SupplierVoteUseCase` (A-026/A-030).
- **Por que é ruim**: invariante "1 voto por (agenda_item, voter)" violado (INV-071); dupla contabilidade.
- **Mitigação Go**:
  - UNIQUE constraint no DB (`UNIQUE(agenda_item_id, voter_id)`).
  - `INSERT ... ON CONFLICT DO NOTHING` ou capturar violação → `apperrors.ErrConflict`.
  - `SELECT FOR UPDATE` em contador (supplier_vote).
  - UoW com isolation `SERIALIZABLE` onde necessário.
- **Detecção**: PBT TOCTOU ([[../09-testing/pbt]]) — 2 goroutines simulam race; integration test com 100 goroutines concorrentes valida UNIQUE.

---

## Mapa AP → pattern de fix

| AP | Pattern correto | Arquivo |
|---|---|---|
| AP-001, AP-002 | Aggregate pequeno + VO extraction + domain service | [[PATTERNS]] §1, §4 |
| AP-003, AP-021 | Mapper explícito | [[CLEAN_ARCH]] §5 |
| AP-004, AP-010 | Query tipada sqlc + use case CQRS | [[PATTERNS]] §5 |
| AP-005, AP-015 | Cache com versão + hash contexto + invalidação webhook | [[PATTERNS]] §11 |
| AP-006 | Cleanup goroutine + teste memory-leak | [[../09-testing/integration]] |
| AP-007, AP-022 | VO.Masked() + structured logger | [[CLEAN_CODE]] §4 |
| AP-008 | Template scaffold canônico | [[CLEAN_ARCH]] §2 |
| AP-009 | TDD + PBT + coverage thresholds | [[../09-testing/test-strategy]] |
| AP-011 | Strategy registry | [[PATTERNS]] §8 |
| AP-012, AP-013 | ABAC middleware central + fluxo canônico | [[../08-security/BEYOND_CORP]] |
| AP-014 | Logger structured com context fields | [[../02-architecture/observability]] |
| AP-016 | Colunas explícitas sqlc | [[CLEAN_ARCH]] §9 |
| AP-017 | Sem hooks; mutação via método | [[CODE_CALISTHENICS]] §8 |
| AP-018 | .gitignore + secret scanner CI | [[../08-security/BEYOND_CORP]] |
| AP-019 | Outbox + log structured | [[PATTERNS]] §14 |
| AP-020 | Backend como source of truth | [[../STEERING]] §72 |
| AP-023 | HMAC antes de parse | [[PATTERNS]] §11 |
| AP-024 | State machine no aggregate | [[PATTERNS]] §1 |
| AP-025 | Multi-tenant scoping + 100+ cross-tenant tests | [[../02-architecture/topology-multitenant]] |
| AP-026 | UNIQUE constraint + ON CONFLICT + PBT TOCTOU | [[../09-testing/pbt]] |

---

## Detecção em CI (resumo)

| Categoria | Ferramenta |
|---|---|
| Dependency rule (domain ←-/→ infra) | grep CI |
| `else` block | grep CI |
| `_ = fn()` em I/O | grep CI |
| `SELECT *` | grep CI |
| PII em log | regex CI |
| Secrets versionados | trufflehog / GitHub secret scanning |
| SAST | `gosec -severity=high` |
| Vulnerability | `govulncheck` |
| Coverage threshold | parser coverage.out vs `.coverage.yml` |
| Race condition | `go test -race -count=1 ./...` |
| Multi-tenant isolation | integration test obrigatório por repo |

---

## Histórico (findings legados A-### → AP correlato)

| A-### legado | AP-### v2 | Descrição |
|---|---|---|
| A-001..A-008, A-015, A-019 | AP-019 | `_ = fn()` em I/O crítica |
| A-003, A-004 | AP-007 | PII em log (CNPJ, invite_token plain) |
| A-005 | AP-018 (variante CORS) | CORS `*` em staging/prod |
| A-006, A-032 | AP-006 | RateLimiter sem cleanup |
| A-018 | AP-018 | Secrets em `.env` versionado |
| A-021 | AP-016, AP-025 | `SELECT *` + tenant missing |
| A-022 | AP-023 (variante rota) | Webhook Mux em `/api/v1` bloqueado por Authenticate |
| A-023 | AP-026 | Race session invalidation |
| A-025 | AP-026 | Vote TOCTOU |
| A-026, A-030 | AP-026 | SupplierVote race |
| A-027 | (Saga sem compensação) | UploadVideo Mux sem compensator |
| A-028 | (Saga sem compensação) | StartLive sem EndRoom |
| A-029 | AP-026 | CompanyEvaluation TOCTOU |

---

## Links

- [[_moc]]
- [[CLEAN_CODE]]
- [[CLEAN_ARCH]]
- [[CODE_CALISTHENICS]]
- [[PATTERNS]]
- [[../01-domain/invariants]]
- [[../01-domain/business-rules]]
- [[../08-security/BEYOND_CORP]]
- [[../08-security/owasp-top10]]
- [[../11-audit/AUDIT]]
- [[../09-testing/test-strategy]]
- [[../09-testing/pbt]]
- [[../09-testing/integration]]
- [[../../vault/10-knowledge/antipatterns/_moc]]
