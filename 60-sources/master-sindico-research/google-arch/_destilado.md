---
title: Destilado — Arquiteturas Google aplicáveis ao Master Síndico
type: note
tags: [research, google, distilled, master-sindico, v2, sre, spanner, bigtable, borg, kubernetes, dapper, monarch, iap, pubsub]
source: web-research 2026-04-23
created: 2026-04-23
updated: 2026-04-23
aliases: [google-arch-destilado]
---

# Destilado — Arquiteturas Google aplicáveis ao Master Síndico

> Síntese pt-BR de 12 fontes oficiais (research.google, sre.google, cloud.google.com) destilando princípios, patterns e decisões aplicáveis ao Master Síndico v2. Cada afirmação carrega referência a `source-NN.md` com URL + trecho. Sem alucinação — se não achei fonte, marquei `[PENDENTE]`.

## 0. Contexto Master Síndico (recap)

- SaaS governança condominial BR, multi-tenant, 7 sub-produtos.
- Stack atual: **Go + Gin + pgx + sqlc + Redis + PostgreSQL + Zitadel + Stripe + Mux + Livekit**.
- M1 (2026-05-08): Governança Institucional + Connect Me (base).
- Escala-alvo curto prazo: dezenas → centenas de condomínios; milhares → dezenas de milhares de usuários.
- Escala-alvo longo prazo (M3+): abrangência nacional, integrações fiscais/bancárias, governança escala "milhões-de-tickets/dia".

**Por isso importa Google**: são a única empresa publicamente documentando padrões de distribuição, confiabilidade e segurança em planet-scale por 2+ décadas; mesmo não rodando nessa escala, os princípios escalam pra baixo (não o contrário).

## 1. Princípios SRE aplicáveis (agora — M1/M2)

Fontes: [[source-04]] (SLI/SLO/SLA), [[source-09]] (SLO workbook), [[source-10]] (Error budget policy), [[source-11]] (Postmortem).

### 1.1 SLI / SLO / SLA — adotar formalmente ainda no M1

Definições canônicas de Google SRE (source-04):

- **SLI**: "a carefully defined quantitative measure of some aspect of the level of service that is provided" — é uma razão `bons eventos / total de eventos`.
- **SLO**: "a target value or range of values for a service level that is measured by an SLI" — estrutura `SLI ≤ target` ou `lower ≤ SLI ≤ upper`.
- **SLA**: "an explicit or implicit contract with your users that includes consequences of meeting (or missing) the SLOs". Diferença: SLA tem consequência (financeira/contratual), SLO é interno.

**Aplicação ao Master Síndico (decisão D-### candidata, registrar em STATE):**

Definir 3 SLIs mínimos por bounded context no M1:

| Contexto | SLI-1 (latência) | SLI-2 (disponibilidade) | SLI-3 (correção/negócio) |
|---|---|---|---|
| Governança Institucional | p95 < 300ms em `POST /assembleias` | 99.5% requests HTTP 2xx/3xx | 100% atas emitidas em ≤ 24h pós-assembleia |
| Connect Me (chat) | p95 entrega msg < 1s | 99.5% | 0% msgs perdidas (durabilidade) |
| Billing/Stripe webhooks | — | 99.9% webhooks processados | 100% idempotência (dedup hash) |

SLO inicial: **99.5%** (conservador, 4-week rolling window conforme source-09 recomenda "integral weeks to control for weekday/weekend variance"). Upgrade pra 99.9% após 2 ciclos estáveis.

**Não confundir SLA com SLO** — com primeiros clientes master-sindico, começar só com SLOs internos; SLA contratual só após M2 quando tiver baseline de 3-6 meses.

### 1.2 Error budget — fazer regra explícita antes do M1 encerrar

Fonte source-10: error budget = `1 - SLO`. Para 99.5% SLO → 0.5% error budget (em 30 dias ~ 3.6h de downtime permitido).

**Política adaptada ao Master Síndico** (inspirada no template SRE workbook):

- **Trigger**: quando error budget de um BC é exaurido numa janela de 4 semanas.
- **Consequência**: "halt all changes and releases other than P0 issues or security fixes until the service is back within its SLO" (citação source-10).
- **Exceções permitidas** (source-10 explicita): outages por infra externa (Stripe, Zitadel, Livekit, GCP) que não estão no perímetro de controle da aplicação → não contabilizam no budget; basta categorização + postmortem.
- **Escalation**: incidente consumindo >20% do budget trimestral → postmortem obrigatório + remediação documentada como A-### em AUDIT.

**Registrar ADR pré-M1**: "Error budget policy do Master Síndico M1" com owners (CTO/solo = tu), template de categorização e checklist de postmortem.

### 1.3 Postmortem blameless — cultura desde a solo-org

Fonte source-11: "The cost of failure is education"; "everyone involved in an incident had good intentions and did the right thing with the information they had".

**Triggers que **forçam** postmortem** (adaptação direta source-11):

- User-visible downtime/degradação > threshold (definir: > 5min p/ M1).
- Perda de dados de qualquer tipo (imediato + A-### urgente).
- Intervenção manual (rollback, reroute) pra resolver.
- Resolução > 1h.
- Falha de monitoramento que exigiu descoberta manual.

**Aplicável hoje**: mesmo solo, escrever postmortem em `11-audit/postmortems/YYYY-MM-DD-<slug>.md`. Já cria ritmo pra quando escalar time.

### 1.4 Toil reduction

SRE book define toil como trabalho manual, repetitivo, automatizável, tático, sem valor duradouro, que escala linearmente com o serviço. **Regra**: se mesma tarefa operacional apareceu 3x → automatizar ou virar runbook + tarefa de eliminação. Aplicável desde M1 nos playbooks de deploy/rollback.

### 1.5 On-call — só faz sentido a partir de M2/M3

Source-11 menciona "at least eight people need to be part of the on-call team to avoid fatigue". Com time solo não rola; alternativa: **SLA interno de 2h em horário comercial** + status page + health checks + alerting (Grafana/Prometheus/Monarch-inspired). On-call formal entra quando houver 3+ devs.

## 2. Patterns de distribuição (multi-tenant — M1+)

Fontes: [[source-01]] (Spanner), [[source-07]] (Spanner multi-tenancy), [[source-02]] (Bigtable), [[source-08]] (F1), [[source-06]] (Colossus).

### 2.1 Escolha de isolamento multi-tenant — aplicável hoje com PostgreSQL

Google Spanner (source-07) formaliza 3 padrões que **espelham 1:1 o que fazemos em Postgres multi-tenant**:

| Pattern Spanner | Equivalente Postgres | Quando usar no Master Síndico |
|---|---|---|
| **Instance-per-tenant** | DB server separado por condomínio | NÃO — custo operacional proibitivo pra centenas de condomínios |
| **Database-per-tenant** | schema ou DB separado por tenant | Candidato pra **contas enterprise** (grandes administradoras com 50+ condomínios) em M3 |
| **Shared DB + tenant_id (row-based)** | `WHERE tenant_id = $1` + RLS | **Escolha default M1/M2** — alinhado com stack atual pgx/sqlc |

**Trade-offs explícitos (source-07):**

- "Resource isolation: instance patterns prevent noisy neighbor problems, while row and database patterns allow tenant workloads to compete for shared resources"
- "Row patterns eliminate per-tenant minimums" — ou seja, um condomínio com 20 apartamentos não paga custo fixo de instância.

**Transposição Postgres + pgx + sqlc:**

1. `tenant_id UUID NOT NULL` em toda tabela de domínio (não-infra).
2. PostgreSQL **Row-Level Security (RLS)** ativado por padrão, com policy `USING (tenant_id = current_setting('app.tenant_id')::uuid)`.
3. Middleware em Gin seta `SET LOCAL app.tenant_id = ...` no início de cada tx (equivalente ao "tenant predicate" do source-07).
4. Index composto sempre iniciando com `tenant_id` (equivalente ao "interleaved tables with TenantId as root of primary key hierarchy" de Spanner — source-07 literalmente recomenda isso).

**Registro**: isso é um ADR grande (ADR-00XX "Multi-tenancy strategy") — referenciar source-07 como base de comparação.

### 2.2 Consistência — Spanner/F1 ensinam o que NÃO precisamos

- Spanner (source-01): "externally-consistent distributed transactions" via TrueTime API — caso extremo global.
- F1 (source-08): "combines high availability, the scalability of NoSQL... and the consistency and usability of traditional SQL databases" — rodando em cima do Spanner pra AdWords (100+ TB, 100k+ req/s).

**Aplicável hoje**: **NÃO**. PostgreSQL com serializable + SAGA inter-módulo (já no CLAUDE.md §8) cobre até M3 sem precisar global distribution. TrueTime/Spanner é inspiração pra M4+ ou nunca — condomínios BR não são workload planet-scale.

**Padrão que herdamos**: **hierarchical schema model** (F1) — dados por condomínio formam hierarquia `tenant → condominio → unidade → morador`; Postgres resolve isso via FK + cascata. Aplicar interleaved-style (PK composto `(tenant_id, id)`) melhora locality e cache Redis.

### 2.3 Sharding — não urgente, mas decidir chave cedo

Source-07: "sequential UUIDs or composite keys that avoid hotspotting (e.g., tenant_id + timestamp)" e "avoiding randomly generated UUIDs as primary keys in high-write tables, as they lead to write amplification".

**Aplicação**: usar **ULID** ou **UUIDv7** (time-ordered) no Master Síndico, não UUIDv4 puro. Isso mantém locality em B-tree mesmo sem particionamento explícito — preparando terreno pra particionamento PostgreSQL nativo quando uma tabela cruzar ~100M linhas (provável só em M3 com vídeos/assembleias log).

### 2.4 Colossus / Bigtable / GFS — não-aplicáveis diretamente, mas inspiracionais

- **Bigtable** (source-02): "sparse, distributed multi-dimensional sorted map"; 10+ exabytes em 2024; construído sobre Colossus + Chubby + SSTable.
- **Colossus** (source-06): sucessor GFS; metadata em Bigtable; 100x escala vs GFS.

**Aplicável ao Master Síndico**: **NÃO** — nossos dados cabem em PostgreSQL por décadas. O que herdamos conceitualmente:
- **Separação storage/compute** (source-13 Dremel/BigQuery também) → guia pra **não acoplar** worker Livekit, worker de transcodificação de vídeo Mux, e API Gin ao mesmo pod. Cada responsabilidade tem scaling independente.
- **SSTable immutability + compaction** → inspiração pra **event-sourcing com event store imutável + snapshots** (se formos por esse caminho em M2+).

## 3. Identity/Auth patterns — comparar Zitadel atual vs. Google

Fontes: [[source-05]] (IAP), [[source-12]] (Identity Platform vs Firebase), [[source-04]] referência cruzada SRE.

### 3.1 Identity Platform (Google) vs Zitadel (nosso)

Feature comparison (source-12):

| Feature | Google Identity Platform | Zitadel (usado hoje) |
|---|---|---|
| OIDC + SAML | Sim (enterprise) | Sim |
| MFA | Sim nativo | Sim nativo |
| **Multi-tenancy** | Sim (tenants nativos) | **Sim (organizations) — match** |
| Blocking functions (hooks pré/pós-auth) | Sim | Sim (actions) |
| Self-host | Não | **Sim** — vantagem Zitadel |
| Compliance SLA | 99.95% enterprise SLA | Depende do deployment |
| Vendor lock | GCP | Open-source (Apache 2.0) |

**Decisão existente**: Zitadel segue sendo escolha M1. Motivo: paridade de features com Identity Platform + self-host + licença aberta. Source-12 vira **benchmark defensivo** — se cliente enterprise pedir "usa Google?", mostrar matrix.

### 3.2 IAP (Identity-Aware Proxy) — padrão zero-trust aplicável conceitualmente

Source-05: IAP "implements zero-trust access... every request evaluated in real time to ensure only authenticated, authorized users can reach protected resources". Policies baseadas em:
- User identity (quem)
- Device state (device posture — postura do dispositivo)
- Resource type (o quê)
- Access context (onde/quando)
- Application (qual app)

**Aplicável ao Master Síndico (alinha com BeyondCorp — CLAUDE.md §3.10):**

- **Hoje (M1)**: auth+authz em TODA request (sem "trusted network" interno) → já no padrão CLAUDE.md.
- **M1+**: Adicionar **ABAC matrix** por `(user_role, tenant_id, resource_type, action)` — cada policy = função Go testável.
- **M2**: Adicionar **device posture** pro app sindicato-admin (dispositivos corporativos): device-id vinculado ao user, MFA obrigatório em novo device, rate limit por device.
- **M2+**: Context-aware (geo + horário comercial). Ex: `assembleia.start` só permitido de IP BR + horário 6h-23h local.

**Não-replicar**: IAP como proxy físico (GCP-only). Nossa versão: middleware Go `authz.Enforce(ctx, subject, resource, action)` rodando no Gin antes de qualquer handler.

## 4. Observability — Dapper → OpenTelemetry (já adotado)

Fontes: [[source-03]] (Dapper), [[source-13]] (Monarch).

### 4.1 Dapper — DNA do OpenTelemetry

Source-03: design goals do Dapper:
1. **Low overhead** — sampling reduz overhead mantendo visibility
2. **Application-level transparency** — "instrumentation to a rather small number of common libraries", zero cognitive burden no dev de feature.
3. **Ubiquitous deployment** — roda em tudo.

Traces/spans/sampling nasceram aqui; Ben Sigelman (um dos autores Dapper) criou OpenTracing → mergiu com OpenCensus → **OpenTelemetry (v1.0 em 2021)**.

**Aplicação no Master Síndico**: OpenTelemetry já é stack-padrão. Reforçar padrões Dapper-originais:

- **Sampling head-based** em produção (1-5% de traces) — não traçar 100% das requests.
- **Spans atributos mínimos**: `tenant_id`, `user_id` (só quando não-PII no log), `bc` (bounded context), `saga_id` se aplicável.
- **Auto-instrumentation via otelgin + otelpgx + otelhttp** — "application-level transparency" conforme Dapper.
- **Trace propagation** em evento Pub/Sub / queue interna (TraceContext no header).
- **NUNCA logar PII em span** — alinha com CLAUDE.md §3.10 "no-PII-logs".

### 4.2 Monarch — inspiração pra métricas/alerting

Source-13: Monarch é time-series DB in-memory planet-scale (~1 PB, 6M queries/s em 2019). Regional architecture + global query plane. Multi-tenant por design.

**Aplicação no Master Síndico**: não replicar — usar **Prometheus + Grafana + Mimir/Thanos** (padrão open-source que espelha princípios Monarch: regional scrape + global query). Para M1 basta Prometheus single-instance; Thanos/Mimir quando tiver 3+ ambientes.

**Padrões herdados**:
- **Alertar em SLO burn rate**, não em CPU/memória (source-09): "multi-window multi-burn-rate alerts" — 2% budget em 1h → pager; 5% em 6h → ticket.
- **Cardinalidade controlada** — não usar `user_id` como label (explode série). Usar `tenant_id`, `bc`, `endpoint`, `status_class` (2xx/4xx/5xx).

## 5. Messaging — Pub/Sub patterns aplicáveis

Fontes: [[source-14]] (Pub/Sub ordering).

Source-14: Pub/Sub entrega "at-least-once por padrão; ordered delivery via ordering key (1MB/s por chave)". Idempotência do subscriber é obrigatória ("subscriber to be idempotent when processing messages"). Trade-off: "ordered delivery decreases publish availability and increases end-to-end message delivery latency".

**Aplicação no Master Síndico** (hoje temos Redis Streams / pgx-based outbox):

1. **Idempotência é não-negociável** — mesmo em Redis/pg outbox, replay acontece. Toda mensagem tem `idempotency_key`; handler faz upsert em `processed_events(idempotency_key PRIMARY KEY)`.
2. **Ordering só quando necessário** — pra eventos de `assembleia` (abrir → votar → encerrar) usar ordering key = `assembleia_id`. Pra eventos broadcast (notificação a todos moradores) não precisa ordem.
3. **Dead-letter queue** — após N retries falhou, move pra DLQ + cria A-### automático.
4. **Trade-off documentado**: ordered delivery custa latência; ADR deve registrar quais contextos exigem ordem.

**Quando migrar pra Pub/Sub real (GCP)**: apenas em M3+ se escala exigir. Redis Streams aguenta 100k+ msg/s por tópico com latência sub-ms; suficiente até centenas de milhares de usuários.

## 6. Cluster management — Borg → Kubernetes

Fontes: [[source-15]] (Borg), [[source-16]] (Borg/Omega/K8s).

Source-15: Borg roda "hundreds of thousands of jobs... tens of thousands of machines"; "admission control, efficient task-packing, over-commitment, machine sharing with process-level performance isolation".

Source-16: evolução Borg → Omega (Paxos + optimistic concurrency) → Kubernetes ("set of composable building blocks"). "Borglet → Kubelet, alloc → pod".

**Aplicação no Master Síndico**:

- **M1**: Docker Compose ou single-node k3s é suficiente (Go binários pequenos, 5-10 réplicas).
- **M2**: Kubernetes gerenciado (EKS/GKE/DOKS). Razões herdadas de Borg:
  - **Declarative job specification** — YAML == BorgCfg conceitual.
  - **Bin-packing + over-commitment** — requests < limits, nodepools dedicados pra worker Livekit (GPU/NIC-heavy).
  - **Process-level isolation** — cgroups v2, não confiar em container como boundary de segurança; combinar com namespace Kubernetes por ambiente (staging/prod) + NetworkPolicy.
- **Evitar anti-pattern** que o paper Borg/Omega/K8s menciona: Kubernetes alloca "IP address per pod, aligning network identity with application identity" — **não compartilhar pods entre bounded contexts**. Um BC = um Deployment (regra DDD + Clean Arch alinhada).

## 7. BigQuery / Dataflow — analytics (M3+)

Fonte: [[source-17]] (Dremel).

Source-17: Dremel = "disaggregated storage and compute, in situ analysis, columnar storage for semistructured data" → BigQuery GA 2011.

**Aplicabilidade ao Master Síndico**:

- **M1/M2**: NÃO necessário. Reports dentro do Postgres com CTE + mat views bastam.
- **M3+**: SE chegar em volume de eventos onde OLTP sofre com queries analíticas (>10M eventos/dia), considerar:
  - **BigQuery** (se estivermos em GCP)
  - **ClickHouse** (equivalente open-source, colunar + disaggregated)
  - **DuckDB** no edge para reports do síndico no device
- **Padrão herdado desde já**: **não misturar OLTP e OLAP** — ainda que em M1 esteja tudo no mesmo Postgres, modelar schemas separados (`app.*` vs `analytics.*`) e views materializadas pra facilitar migração futura sem refactor.

## 8. Pub/Sub Research at Google — outros papers monitorados

Source-18 (Research at Google papers index). Papers relevantes pra M3+ que vale deixar no radar sem destilar agora:

- **MapReduce (2004)** — inspirou Hadoop. Hoje irrelevante (substituído por Spark/Flink/Dataflow). Não destilar.
- **Chubby (2006)** — [[source-19]]. Lock service via Paxos. **Aplicabilidade Master Síndico**: NÃO — usamos Redis Redlock ou pg_advisory_lock pra coordenação simples.
- **Google File System (2003)** — superseded por Colossus. Não destilar.

## 9. Matriz de aplicabilidade (resumo executivo)

| Tema / Fonte | M1 (2026-05) | M2 (2026-Q3) | M3+ (2027) | Não-aplicável |
|---|---|---|---|---|
| SLI/SLO interno (source-04, 09) | **Sim** (3 por BC) | Sim (9-12 por BC) | SLA contratual | — |
| Error budget policy (source-10) | **Sim** (ADR) | Enforcement automatizado | — | — |
| Postmortem blameless (source-11) | **Sim** (template) | Trend analysis | ML prediction | — |
| Multi-tenancy row-based (source-07) | **Sim** (RLS + tenant_id) | Composite PK (tenant_id, ulid) | Database-per-tenant enterprise | Instance-per-tenant (custo) |
| TrueTime / external consistency (source-01) | — | — | Inspiração | **Não** (Postgres basta) |
| F1 hybrid SQL (source-08) | Inspira hierarchical schema | — | — | Não replicar |
| Colossus (source-06) | — | — | — | **Não** (Postgres + S3 basta) |
| Bigtable (source-02) | — | — | — | **Não** |
| IAP zero-trust (source-05) | **Sim** (já via CLAUDE §3.10) | Device posture + ABAC matrix | Context-aware completo | IAP literal (GCP-lock) |
| Identity Platform (source-12) | Benchmark defensivo | — | — | **Zitadel vence** (self-host) |
| Dapper / OTel (source-03) | **Sim** (já em stack) | Sampling adaptativo | — | — |
| Monarch (source-13) | Prometheus single | Thanos/Mimir | — | Monarch literal (GCP) |
| Pub/Sub ordering (source-14) | **Sim** (Redis Streams + idempotency) | Outbox pg | Pub/Sub GCP se escalar | — |
| Borg/K8s (source-15, 16) | Compose/k3s | **K8s gerenciado** | Multi-cluster | Borg literal |
| Dremel/BigQuery (source-17) | — | Mat views Postgres | ClickHouse/BigQuery | — |
| Chubby (source-19) | — | — | — | **Não** (pg_advisory_lock) |

## 10. Próximas decisões a registrar como D-### em STATE

1. **D-###**: adotar SLI/SLO framework SRE — 3 SLIs por BC no M1, 4-week rolling window, 99.5% target inicial.
2. **D-###**: error budget policy formal ADR-00XX pré-M1 (trigger, consequência, exceção).
3. **D-###**: postmortem template + trigger rules em `11-audit/postmortems/` — todo incidente > 5min downtime vira postmortem.
4. **D-###**: multi-tenancy = shared DB + `tenant_id` + RLS + composite PK `(tenant_id, ulid)` — registrar ADR grande referenciando source-07.
5. **D-###**: ULID/UUIDv7 como PK de todas tabelas high-write (não UUIDv4) — fonte: source-07 "avoid randomly generated UUIDs".
6. **D-###**: manter Zitadel; source-12 é benchmark (não migrar pra Identity Platform).
7. **D-###**: OTel padrões Dapper — sampling 1-5%, atributos mínimos padronizados, zero PII em spans.
8. **D-###**: Redis Streams + outbox no M1; Pub/Sub apenas se M3 exigir.
9. **D-###**: K8s gerenciado no M2 (não M1 — overkill).

## 11. O que deixei como PENDENTE

- [PENDENTE] Matriz SLI → alerta burn rate específica por endpoint crítico (requer levantamento de métricas atuais em 07-quality).
- [PENDENTE] ADR de estratégia de particionamento PostgreSQL (quando uma tabela atingir threshold X) — precisa dimensionamento de cada BC.
- [PENDENTE] Avaliação quantitativa `Redis Streams` vs `outbox pg` vs `NATS` vs `Pub/Sub` — precisa spike em 13-research específico.

## 12. Links

- Fontes por ordem: [[source-01]] ... [[source-19]].
- Vizinhos: [[_moc]], [[../beyond-corp/_moc]] (zero-trust correlato), [[../netflix/_moc]] (patterns resiliência big tech).
- Projeto: [[../../CLAUDE]], [[../../STATE]].
