---
title: Aplicações google-arch → Master Síndico v2 (cruzamento + gaps + recomendações)
type: note
tags: [research, google, applications, master-sindico, v2, sre, slo, multi-tenant, cell-based, saga, rls]
source: 13-research/google-arch/_destilado.md + 02-architecture/{observability,topology-multitenant,patterns,research-inspirations}.md + 01-domain/invariants.md + STATE.md + 02-architecture/adr/
created: 2026-04-23
updated: 2026-04-23
aliases: [google-arch-aplicacoes, google-arch-gaps-master-sindico]
---

# Aplicações google-arch → Master Síndico v2

Cruzamento nominativo do destilado [[_destilado]] com a arquitetura v2 vigente. Para cada princípio Google: **o que já está aplicado**, **gap identificado** e **recomendação priorizada** com arquivo-alvo nominal + marco. Filosofia: pragmático, MVP 2026-05-08, YAGNI em recomendações M3+.

> Leituras vizinhas: [[_destilado]], [[../../02-architecture/observability]], [[../../02-architecture/topology-multitenant]], [[../../02-architecture/patterns]], [[../../02-architecture/research-inspirations]], [[../../01-domain/invariants]], [[../../STATE]].

---

## §1. Estado atual — o que já está aplicado

Resumo do que o destilado propôs e que já foi absorvido em artefatos canônicos da v2 (nominativamente):

### 1.1 SRE framework (SLI/SLO/error budget/postmortem)

- **SLI/SLO por BC**: matriz M1 em [[../../02-architecture/observability]] §3.1 com 3 SLIs por BC (identity, billing, institutional, commercial, content, assembly, cross-domain). Baseline 99.5%, 4-week rolling window. Formaliza destilado §1.1.
- **Error budget policy**: [[../../02-architecture/observability]] §4 + ADR [[../../02-architecture/adr/0025-sli-slo-error-budget]]. Fórmula `1 - SLO`, trigger halt-releases, exceções de infra externa documentadas. Implementa destilado §1.2.
- **Postmortem blameless template**: [[../../02-architecture/observability]] §6 + pasta `11-audit/postmortems/` prevista. Triggers explícitos (downtime >5min, perda de dado, resolução >1h). Implementa destilado §1.3.
- **Burn rate multi-window alerts**: [[../../02-architecture/observability]] §5.1 (1h/14.4×, 6h/6×, 24h/1×, 72h/0.5×). Implementa destilado §4.2.

### 1.2 Observability stack Dapper-aligned

- OTel + Prometheus + Grafana + Sentry canônico em [[../../02-architecture/adr/0020-opentelemetry-grafana-sentry]].
- Sampling head-based 1% + 100% em 5xx em [[../../02-architecture/observability]] §2.3.1.
- Atributos mínimos padronizados (`tenant_id`, `user_id` opaco, `bc`, `saga_id`) em §2.3.
- Regra dura **zero PII em spans/logs** com CI linter. Alinha destilado §4.1.

### 1.3 Multi-tenancy row-based Spanner-inspired

- [[../../02-architecture/topology-multitenant]] §1 + ADR [[../../02-architecture/adr/0021-multi-tenant-row-based]] aplica:
  - `tenant_id BYTEA NOT NULL` em toda tabela de domínio.
  - PK composto `(tenant_id, ulid)` (herança Spanner interleaved, destilado §2.1).
  - Índices iniciando por `tenant_id` (locality B-tree).
  - RLS opcional em tabelas sensíveis + `SET LOCAL app.tenant_id` no middleware.
  - ULID/UUIDv7 como PK (destilado §2.3 evitar UUIDv4 hotspotting) — ADR [[../../02-architecture/adr/0009-uuidv7-ulid-pk]].

### 1.4 Idempotência + at-least-once + outbox

- Outbox pattern obrigatório em [[../../02-architecture/patterns]] §14.
- HMAC antes de parse + Redis SetNX 24h em webhooks (§11).
- INV-093 (HMAC before parse), INV-095 (events via outbox na mesma tx), INV-105 (DB UNIQUE `(provider, event_id)` como fonte de verdade) em [[../../01-domain/invariants]].
- Alinha destilado §5 (Pub/Sub ordering + idempotency).

### 1.5 Zero-trust + IAP-like ABAC

- ABAC engine canônica em [[../../02-architecture/adr/0012-abac-redis-cache]] + INV-086/087/088.
- Ordem estrita `admin_bypass → role → action → tenant → scope` + default-deny + audit log do bypass.
- Step-up MFA em endpoints críticos (INV-110, ADR [[../../02-architecture/adr/0026-admin-mfa-m1]]).
- 1-device policy + fingerprint (ADR [[../../02-architecture/adr/0014-one-device-policy]]). Alinha destilado §3.2.

### 1.6 Resilience stack (não-google mas convergente)

- Circuit breaker por provider em [[../../02-architecture/patterns]] §12 + ADR [[../../02-architecture/adr/0022-circuit-breaker-gobreaker]].
- Retry + backoff + jitter + DLQ em §13. Tabela de thresholds por provider.

### 1.7 Saga inter-BC com compensação

- [[../../02-architecture/patterns]] §7 + ADR [[../../02-architecture/adr/0015-uow-intra-saga-inter]] + ADR [[../../02-architecture/adr/0030-saga-orchestration-default]].
- UoW intra-BC, Saga inter-BC ou provider externo. INV-065 (UploadVideo compensar), INV-084 (StartLive compensar), INV-104 (AccountDeletion saga LGPD).

### 1.8 Cluster management — decisão temperada

- D-### prevê Docker Compose/k3s M1 → K8s gerenciado M2+ (alinhado destilado §6).
- Railway inicial M1–M2 → AWS ECS M3+ (ADR [[../../02-architecture/adr/0017-railway-initial-aws-m4]]).

---

## §2. Gaps — práticas Google ainda não aplicadas

### 2.1 Error budget **quantitativo automatizado**

- Hoje: policy escrita, trigger textual ("halt releases").
- Gap: **nenhum cálculo automatizado do budget consumido em runtime**. Não há exporter Prometheus que emita `slo_budget_remaining{bc}` nem alerta "budget exhausted". Sem isso, policy é teórica.
- Origem: destilado §1.2 (error budget = `1 - SLO`, decisão humana) + SRE workbook burn rate automatizado.

### 2.2 Load shedding + graceful degradation

- Hoje: circuit breaker em provider **externo**. Sem load shedding em sobrecarga interna (API própria).
- Gap: sem `MaxInflightRequests` por endpoint crítico, sem **priorização de requests** (usuário premium vs free, assembleia ativa vs read casual), sem **503 Retry-After** preventivo quando pool Postgres > 80%.
- Origem: destilado §1.4 (toil) + Borg admission control (§6 source-15).

### 2.3 Cell-based architecture (failure domain isolation)

- Hoje: 1 cluster, 1 Postgres, 1 Redis — **blast radius = plataforma inteira**.
- Gap: sem conceito de "cell" (grupo de tenants com infra isolada). Um bug em query pesada de tenant grande afeta todos.
- Origem: Google (Spanner instance-per-set, Borg cells) + AWS Well-Architected Cells. Não está no destilado mas é extensão natural de §2.1.

### 2.4 IAM scoping profundo — tokens de escopo mínimo em workers/providers

- Hoje: ABAC no HTTP é granular. **Workers internos** (outbox dispatcher, cron jobs, saga orchestrator) tipicamente rodam com credencial "service account" ampla.
- Gap: sem escopo por worker (ex: `outbox-dispatcher` só lê `outbox_events` + publica NATS; não pode ler `users`). IAM "least privilege" de Google IAM/Workload Identity não tem paralelo interno.
- Origem: destilado §3 (IAP + IAM implícito) + Borg Workload Identity.

### 2.5 PostgreSQL RLS — opt-in, não default

- Hoje: RLS "opcional em tabelas sensíveis" ([[../../02-architecture/topology-multitenant]] §1).
- Gap: **RLS não é default**. Spanner faz auth-per-row nativamente (source-07). Nossa RLS cobre ~5 tabelas; se dev esquecer `WHERE tenant_id` em query nova em tabela sem RLS, vaza cross-tenant. O linter sqlc grep é 1 barreira; falta a segunda (RLS) em 90% das tabelas.

### 2.6 Saga **orchestration** formal vs choreography ad-hoc

- Hoje: Sagas existem (UploadVideo, StartLive, AccountDeletion) mas **cada uma é hand-rolled** — sem state machine declarativa, sem visualização, sem replay de step.
- Gap: Google Cloud Workflows / Temporal pattern não aplicado. ADR [[../../02-architecture/adr/0030-saga-orchestration-default]] decidiu orchestration mas falta **primitivo canônico** (lib ou padrão de código) reutilizável.
- Origem: inferência sobre destilado §2 + §5 (Pub/Sub) + prática Google Workflows.

### 2.7 Postmortem culture sem blame — só tem template

- Hoje: template em [[../../02-architecture/observability]] §6.2. Pasta `11-audit/postmortems/` prevista.
- Gap: **sem processo** (quem conduz, quando, action items obrigatórios, trend analysis quinzenal). Solo-org hoje; mas culturalmente já deveria ter checklist operacional.
- Origem: destilado §1.3.

### 2.8 Tail-based sampling + trace-based debugging

- Hoje: head-based 1%.
- Gap: sem amostragem seletiva de traces "interessantes" (>1s latência, 5xx, saga com compensação disparada). Dapper moderno faz tail-based em Cloud Trace.
- Marco correto: M3+ (custo operacional), mas **decisão formal ainda ausente** em ADR.

### 2.9 Cardinalidade de métricas — regra escrita, não enforced

- Hoje: regra em [[../../02-architecture/observability]] §2.2.1.
- Gap: **sem CI linter** que detecte `prometheus.Counter(Name: "x", Labels: {"user_id": ...})` e bloqueie PR. Regra depende de review humano.

### 2.10 Adaptive concurrency + backpressure no Postgres pool

- Hoje: pgx pool com `MaxConns` fixo.
- Gap: sem **adaptive throttling** (source-09 SRE workbook) quando latência Postgres sobe. Sem priorização (ex: assembleia ativa prioriza sobre jobs de analytics). Gatilho natural M2.

### 2.11 Chaos engineering (destilado §1.5 on-call + §9 matriz M3)

- Hoje: nada. Marco M3+ correto, mas ausência de **ADR** ou placeholder.

### 2.12 Dremel/BigQuery OLAP split preparation

- Hoje: [[../../02-architecture/research-inspirations]] §1.11 menciona monolito modular, mas **schemas `app.*` vs `analytics.*` separados** (destilado §7) não são prática vigente. Reports ainda misturam.

---

## §3. Recomendações priorizadas (top-10)

Formato: título / origem destilado / arquivo-alvo / marco / esforço (P/M/G) / justificativa.

| # | Recomendação | Origem | Arquivo-alvo (nominal) | Marco | Esforço | Justificativa |
|---|---|---|---|---|---|---|
| **R-01** | **Exporter Prometheus de error budget remaining + alerta `budget_exhausted` automático** | §1.2 | [[../../02-architecture/observability]] §4 (novo §4.4) + `12-docs/runbooks/budget-exhausted.md` | **M1** | M | Sem número automatizado, policy vira letra morta. Burn rate alerts já existem (§5.1) — falta só métrica stateful `slo_budget_remaining{bc,window}` derivada de Prometheus recording rules. |
| **R-02** | **RLS como default (opt-out)** — toda nova tabela de domínio já criada com RLS ativa; exceções explicitadas na migration | §2.1 (Spanner auth-per-row) | [[../../02-architecture/topology-multitenant]] §4 + nova migration template + CI gate | **M1** | M | Segunda barreira de defesa em profundidade. Linter sqlc sozinho é frágil. Hoje só ~5 tabelas têm RLS; invertendo o default elimina classe inteira de bugs cross-tenant. Nova ADR. |
| **R-03** | **Primitivo canônico de Saga orchestration** (lib interna `pkg/saga` com state machine declarativa, persistence em `saga_instances`, replay, visualização) | §2 + §5 (Workflows-inspired) | [[../../02-architecture/patterns]] §7 (reforçar) + novo `02-architecture/saga-primitives.md` + ADR nova | **M1** (lib mínima) → **M2** (tooling) | G | Hoje cada saga é hand-rolled (UploadVideo, StartLive, AccountDeletion, billing sync). Crescerá para 10+ sagas até M2. Sem primitivo: bugs de compensação reaparecem. |
| **R-04** | **Processo blameless formal** (checklist operacional, rito quinzenal de trend analysis, action items obrigatórios linkados a A-### em AUDIT) | §1.3 | Novo `07-quality/postmortem-process.md` + link em [[../../02-architecture/observability]] §6 | **M1** | P | Template sem processo não previne blame. Baixo esforço, alto retorno cultural (e quando time crescer, já tá pronto). Alternativa: merge em `06-execution/rituals.md` se existir. |
| **R-05** | **CI linter de cardinalidade de métricas + PII em spans** (AST scanner Go bloqueia labels high-cardinality + atributos com `cpf`/`email`/`phone` em `span.SetAttributes`) | §4.1 + §4.2 | `09-testing/ci-gates.md` + [[../../02-architecture/observability]] §10 | **M1** | M | Regras escritas dependem de review humano (falha). Existe gate similar pra log PII (grep regex); estender pra span.SetAttributes + prometheus labels é extensão natural. |
| **R-06** | **Load shedding middleware** (admission control: `MaxInflight` por BC, 503 Retry-After preventivo quando pool pg > 80% ou CPU > 85%) | §6 (Borg admission) + §1.4 | Novo `02-architecture/load-shedding.md` + middleware `internal/http/middleware/shed.go` (spec) + ADR nova | **M2** | M | Durante assembleia live (M3) + billing peak, sobrecarga interna explode. Circuit breaker só cobre provider externo. Shedding preventivo evita cascading failure. |
| **R-07** | **Cell-based architecture** (tenant-grouping por shard; 1 cell = N tenants isolados em cluster separado; roteamento por `tenant_id → cell_id` lookup) | §2.1 (instance patterns) + AWS Cells | [[../../02-architecture/topology-multitenant]] §8 (roadmap) + ADR nova M3 | **M3** (planejamento); **M4** (implementação) | G | Blast radius hoje = plataforma inteira. Com 500+ tenants M3, 1 bug = 500 condomínios caem. Cells isolam a ~50-100 tenants cada. PK `(tenant_id, ulid)` já prepara (§2.3 destilado). |
| **R-08** | **IAM scoping de workers internos** (service account por worker com permissão mínima — `outbox-dispatcher`, `moderation-worker`, `billing-cron`, `livekit-egress-handler`) | §3 (IAP + least privilege) | Novo `08-security/service-accounts.md` + extensão [[../../02-architecture/adr/0016-beyondcorp-security-posture]] | **M2** | M | Hoje workers usam credencial ampla. Se `moderation-worker` é comprometido, vaza tudo. ABAC humana é granular; ABAC máquina (M2M) não é. Alinha BeyondCorp least-privilege para workload identity. |
| **R-09** | **Schemas `app.*` vs `analytics.*` separados desde M1** + views materializadas com owner dedicado, prep pra ClickHouse/DuckDB/BigQuery M3+ | §7 (Dremel storage/compute split) | [[../../02-architecture/topology-multitenant]] (novo §5.3) + migration convention em `07-quality` | **M1** | P | Custo zero agora (só convenção de schema na migration). Evita refactor grande em M3 quando queries analíticas ficarem pesadas. |
| **R-10** | **Adaptive concurrency pg pool + priorização por BC** (pool dedicado para `assembly` durante assembleia ativa; throttle `content.moderation-worker` se pool > 70%) | §1.4 (toil) + §6 (admission) | Novo ADR `bc-pool-isolation` + [[../../02-architecture/observability]] dashboard "Database" §7.3 | **M2** | M | Noisy-neighbor entre BCs já previsto em [[../../02-architecture/topology-multitenant]] §8 como risco. Esperar até M3 é tarde — assembleia live (M3) é crítica, precisa pool garantido. |

---

## §4. ADRs novas (sugestão)

| Proposta | Contexto (1 frase) |
|---|---|
| **ADR-0033 — Error budget automated exporter** | Formalizar exporter Prometheus + recording rules para `slo_budget_remaining{bc,window}` + alerta `budget_exhausted` com halt automático (R-01). |
| **ADR-0034 — RLS default-on em multi-tenant** | Toda tabela de domínio nasce com RLS ativa; exceções globais (plans, companies_global_registry, moderation_rules) listadas explicitamente em migration + CI gate (R-02). |
| **ADR-0035 — Saga orchestration primitives** | Lib interna `pkg/saga` com state machine declarativa + persistence + replay como primitivo canônico; substitui sagas hand-rolled (R-03). |
| **ADR-0036 — Load shedding + admission control** | Middleware de admission por BC com `MaxInflight`, 503 Retry-After preventivo baseado em saturação pg pool / CPU (R-06). |
| **ADR-0037 — Cell-based architecture M3+** | Plano de particionamento físico em cells de ~50–100 tenants; roteamento `tenant_id → cell_id`; migração incremental (R-07). |
| **ADR-0038 — Service accounts por worker (M2M least privilege)** | Cada worker interno (outbox-dispatcher, moderation-worker, billing-cron, livekit-egress-handler) recebe credencial escopada mínima (R-08). |
| **ADR-0039 — Schema split `app.*` / `analytics.*`** | Separação física de schemas desde M1 para preparar OLAP split futuro sem refactor (R-09). |
| **ADR-0040 — BC-aware pg pool isolation** | Pool Postgres por BC ou por prioridade (assembleia > resto) com adaptive throttle baseado em latência (R-10). |

---

## §5. D-### e DT-### novas

### Decisões (a registrar em `STATE.md`)

| ID proposto | Decisão |
|---|---|
| **D-087** | Adotar R-01: exporter Prometheus + recording rules `slo_budget_remaining` M1; alerta `budget_exhausted` com halt automático (supera texto-apenas de ADR-0025). |
| **D-088** | Adotar R-02: RLS default-on em novas tabelas de domínio; opt-out explícito em migration comentário + CI gate. |
| **D-089** | Adotar R-03: primitivo canônico `pkg/saga` com state machine declarativa M1 (escopo mínimo: 3 sagas vigentes). |
| **D-090** | Adotar R-04: processo blameless formal em `07-quality/postmortem-process.md` M1, checklist operacional + rito quinzenal. |
| **D-091** | Adotar R-05: CI linter AST-scanner Go para cardinalidade de métricas + PII em spans M1. |
| **D-092** | Adotar R-09: schemas `app.*` vs `analytics.*` separados por convenção desde M1 (custo zero). |
| **D-093** | Adiar R-06/R-10 (load shedding + pg pool isolation) para M2 formalmente; registrar em ROADMAP. |
| **D-094** | Adiar R-07 (cell-based) para M3+ com ADR-0037 como placeholder; critério de gatilho: >300 tenants ou >1 incidente platform-wide. |
| **D-095** | Adiar R-08 (service accounts por worker) para M2; decidir mecanismo (JWT interno vs Zitadel M2M) em ADR-0038. |

### Dívidas técnicas (a registrar em `STATE.md`)

| ID proposto | Dívida |
|---|---|
| **DT-012** | Migrar as ~5 tabelas com RLS opcional hoje para RLS default-on pattern (quando ADR-0034 estabilizar). |
| **DT-013** | Refatorar sagas hand-rolled (UploadVideo, StartLive, AccountDeletion) para `pkg/saga` primitivo quando R-03 ship. |
| **DT-014** | Renomear schemas atuais para prefixo `app.*` após D-092 (impacto: todas migrations). |
| **DT-015** | Definir e ADR-zar estratégia tail-based sampling (§2.8) pra M3+; placeholder até lá. |
| **DT-016** | ADR e placeholder de chaos engineering pra M3 (destilado §1.5 on-call + fault injection). |
| **DT-017** | Mapear e documentar trend analysis quinzenal dos postmortems (output esperado: `11-audit/postmortems/_trend-YYYY-Q#.md`). |

---

## §6. Invariantes novos

Novos `INV-###` candidatos derivados das recomendações, a adicionar em [[../../01-domain/invariants]] seção **cross-domain** (INV-111+) ou **security hardening** (INV-111+ estendendo série 101-110).

| ID | Invariante | Owner | Enforcement | Origem |
|---|---|---|---|---|
| **INV-111** | `slo_budget_remaining{bc,window="4w"}` emitido para Prometheus a cada scrape; alerta `budget_exhausted` dispara halt de releases (CD pipeline consulta métrica antes de deploy). | cross-domain / ops | Prometheus recording rule + deploy guard script | R-01 / ADR-0033 |
| **INV-112** | Toda tabela com `tenant_id` nasce com `ALTER TABLE ... ENABLE ROW LEVEL SECURITY` + policy `tenant_isolation`; exceções (tabelas globais) declaradas em header da migration. | cross-domain | CI scanner migrations + teste de inventário RLS | R-02 / ADR-0034 |
| **INV-113** | Métrica Prometheus não tem label `user_id`, `request_id`, `session_id`, `email`, `cpf`, `cnpj`, `phone` (lista negra). | cross-domain / ops | CI AST-scanner Go rejeita `Labels: {...}` com high-cardinality + PII keys | R-05 / ADR-0033 |
| **INV-114** | `span.SetAttributes(...)` não contém chaves em lista negra PII (mesma de INV-113 + `full_name`). | cross-domain / ops | CI AST-scanner Go | R-05 |
| **INV-115** | Worker interno (outbox-dispatcher, moderation-worker, billing-cron, livekit-egress-handler) autentica com service account escopado; token tem `aud=internal-worker` + `scope` mínimo; não reutiliza credencial de user. | cross-domain / security | Middleware `RequireWorkerScope(scope)` em handlers internos + inventário em `08-security/service-accounts.md` | R-08 / ADR-0038 |
| **INV-116** | Saga com compensação tem persistence em `saga_instances(saga_id, bc, step, state, payload_snapshot, compensation_log)`; replay idempotente em startup (worker reprocessa sagas em estado não-terminal). | cross-domain | `pkg/saga` primitivo + teste integration com crash simulation | R-03 / ADR-0035 |
| **INV-117** | Todo schema de domínio está em `app.*`; tabelas analíticas / views materializadas em `analytics.*`; migration que cria tabela em schema não-listado falha CI. | cross-domain | CI migration linter | R-09 / ADR-0039 |

---

## 7. Cruzamento direto (foco especial da tarefa)

Respondendo aos pontos explícitos do prompt:

- **Error budget + SLO burn rate alerts → adicionar em `observability.md`?** → **Sim, parcialmente já existe (§5.1 burn rate)**. Gap concreto: falta recording rule + métrica stateful `slo_budget_remaining` e alerta `budget_exhausted`. Recomendação R-01 / ADR-0033. Seção nova §4.4 em [[../../02-architecture/observability]].
- **PostgreSQL RLS vs ABAC middleware?** → **Complementares, não substitutos**. ABAC é autz de aplicação (ordem, admin bypass, plan tier); RLS é a segunda barreira física no DB. Hoje RLS é opt-in em ~5 tabelas. Recomendação R-02 / ADR-0034: **RLS default-on**.
- **Saga orchestration vs choreography → billing/connect-me?** → **Orchestration já decidida (ADR-0030)**. Gap é ausência de primitivo canônico. Billing/Connect Me vão crescer sagas (subscription lifecycle, proposal → interest → reveal → timeline entry). Recomendação R-03 / ADR-0035: lib `pkg/saga` M1 mínima, tooling M2.
- **Cell-based architecture pra M3?** → **Sim, M3 como plano, M4 como execução**. ADR-0037 placeholder agora; gatilho >300 tenants ou >1 incidente platform-wide. PK composto `(tenant_id, ulid)` já prepara sharding físico sem refactor (destilado §2.3).
- **Postmortem process + sem blame em `07-quality/` ou `06-execution/`?** → **Recomendação: `07-quality/postmortem-process.md`**. Razão: quality owns testing/coverage/rituals de qualidade; postmortem é feedback loop de qualidade operacional. `06-execution` fica com Ship/Deploy rituals. Se preferir acoplar aos rituals de deploy, alternativa em `06-execution/rituals.md`.

---

## 8. O que deixei de fora intencionalmente (YAGNI M1)

- **Spanner TrueTime / external consistency** (§2.2 destilado): Postgres basta até M4+.
- **Chubby lock service** (§8): `pg_advisory_lock` + Redis Redlock resolvem.
- **Monarch literal** (§4.2): Prometheus + Mimir já é padrão open-source equivalente.
- **BigQuery/ClickHouse** (§7): só quando queries OLAP doerem — M3+.
- **K8s gerenciado M1**: overkill; Railway + Docker Compose/k3s suficientes até M2.
- **Pub/Sub GCP literal**: Redis Streams + outbox cobrem M1–M2; NATS JetStream quando ≥ 3×3 produtores/consumidores (ADR [[../../02-architecture/adr/0019-nats-jetstream-threshold]]).

---

## 9. Links

- Fonte primária: [[_destilado]] (305 linhas, 12 fontes oficiais Google).
- Observability vigente: [[../../02-architecture/observability]].
- Multi-tenancy vigente: [[../../02-architecture/topology-multitenant]].
- Patterns vigentes: [[../../02-architecture/patterns]].
- Síntese research: [[../../02-architecture/research-inspirations]].
- Invariantes: [[../../01-domain/invariants]].
- ADRs: [[../../02-architecture/adr/_moc]].
- Decisões vivas: [[../../STATE]].
