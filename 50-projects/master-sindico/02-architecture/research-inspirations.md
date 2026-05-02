---
title: Research Inspirations — princípios herdados do research web
type: guide
tags: [architecture, research, inspirations, master-sindico, v2]
source: 13-research/{beyond-corp,google-arch,netflix,youtube,google-meet,tiktok,instagram,linkedin}/_destilado.md
created: 2026-04-23
updated: 2026-04-23
aliases: [research-synthesis, inspirations-map]
---

# Research Inspirations — mapa dos princípios herdados

Síntese final do research web. Cada princípio: **o quê** → **de quem herdamos** → **como aplica no master-sindico** → **marco**.

> Origem: pesquisa em fontes primárias (engineering blogs, whitepapers, papers, docs oficiais). Destilados em [[../13-research/beyond-corp/_destilado]], [[../13-research/google-arch/_destilado]], [[../13-research/netflix/_destilado]], [[../13-research/youtube/_destilado]], [[../13-research/google-meet/_destilado]], [[../13-research/tiktok/_destilado]], [[../13-research/instagram/_destilado]], [[../13-research/linkedin/_destilado]].

---

## 1. Mapa: princípio → origem → aplicação → marco

### 1.1 Multi-tenancy row-based + PK composto

- **Origem**: Google Spanner docs (multi-tenancy patterns) — [[../13-research/google-arch/_destilado]] §2.1.
- **Aplicação**: `tenant_id BYTEA NOT NULL` + PK `(tenant_id, ulid)` + RLS opcional + todo índice começa por `tenant_id`.
- **Marco**: **M1**. ADR [[adr/0021-multi-tenant-row-based]].
- **Por quê**: "Row patterns eliminate per-tenant minimums" — custo fixo zero por tenant.

### 1.2 ULID / UUIDv7 como PK (evita hotspotting)

- **Origem**: Google Spanner docs — "avoid randomly generated UUIDs as primary keys in high-write tables" — [[../13-research/google-arch/_destilado]] §2.3.
- **Aplicação**: ULID (Crockford base32, time-ordered) em todas PKs. Prepara sharding futuro.
- **Marco**: **M1**. ADR [[adr/0009-uuidv7-ulid-pk]].

### 1.3 Identidade centralizada em IdP (Zitadel) com PKCE obrigatório

- **Origem**: BeyondCorp (Ward & Beyer, 2014) + NIST SP 800-207 — [[../13-research/beyond-corp/_destilado]] §1.3.
- **Aplicação**: Zitadel OIDC Authorization Code + PKCE, MFA para papéis sensíveis, Organizations = tenants.
- **Marco**: **M1**. ADR [[adr/0003-zitadel-oidc-provider]] + [[adr/0016-beyondcorp-security-posture]].

### 1.4 ABAC policy engine (admin_bypass → role → action → tenant → scope)

- **Origem**: BeyondCorp ACE (Access Control Engine) + NIST PDP/PEP — [[../13-research/beyond-corp/_destilado]] §2.2 Princípio 3.
- **Aplicação**: `ABACPolicyEvaluator` no domain do BC identity + cache Redis 5min com invalidação webhook.
- **Marco**: **M1**. ADR [[adr/0012-abac-redis-cache]].

### 1.5 1 device per account + device fingerprint

- **Origem**: BeyondCorp Device Inventory + Okta Device Assurance — [[../13-research/beyond-corp/_destilado]] §2.2 Princípio 2 + §3.1.
- **Aplicação**: tabela `devices`, fingerprint (IP + UA + canvas hash), Play Integrity / DeviceCheck em mobile. Assembleia live exige atestação.
- **Marco**: **M1** (base); M2+ atestação plataforma. ADR [[adr/0014-one-device-policy]].

### 1.6 Step-up MFA para ações críticas

- **Origem**: Google IAP context-aware + Okta device assurance — [[../13-research/beyond-corp/_destilado]] §2.2 Princípio 6 + §3.5.
- **Aplicação**: endpoints marcados `auth_level: step_up`: aprovar transação, encerrar votação, mudar síndico, admin cross-tenant.
- **Marco**: **M1**. Parte de [[adr/0016-beyondcorp-security-posture]].

### 1.7 Resilience stack: Circuit Breaker + Retry + Timeout + Bulkhead + Fallback

- **Origem**: Netflix Hystrix → resilience4j → padrões modernos — [[../13-research/netflix/_destilado]] §2.
- **Aplicação**: `sony/gobreaker` + `avast/retry-go` + `context.WithTimeout` + `golang.org/x/sync/semaphore` em **toda** call externa. Fallback fail-closed (auth/billing) vs fail-open (cosmético).
- **Marco**: **M1**. ADR [[adr/0022-circuit-breaker-gobreaker]].

### 1.8 SLI / SLO formais por BC + error budget policy

- **Origem**: Google SRE book + SLO workbook — [[../13-research/google-arch/_destilado]] §1.
- **Aplicação**: 3 SLIs por BC, 4-week rolling window, 99.5% target M1, error budget policy com halt automático.
- **Marco**: **M1**. ADR [[adr/0025-sli-slo-error-budget]].

### 1.9 Postmortem blameless

- **Origem**: Google SRE book "The cost of failure is education" — [[../13-research/google-arch/_destilado]] §1.3.
- **Aplicação**: template obrigatório em `11-audit/postmortems/` quando triggers (downtime > 5min, perda de dado, resolução > 1h, intervenção manual).
- **Marco**: **M1**. Parte de [[adr/0025-sli-slo-error-budget]].

### 1.10 Observability OpenTelemetry padrão Dapper

- **Origem**: Google Dapper (2010) → OpenTelemetry — [[../13-research/google-arch/_destilado]] §4.1.
- **Aplicação**: otelgin + otelpgx + otelhttp; sampling 1% head-based; atributos mínimos padronizados (tenant_id, user_id opaco, bc, saga_id); **zero PII em spans**.
- **Marco**: **M1**. ADR [[adr/0020-opentelemetry-grafana-sentry]].

### 1.11 Monolito modular → extrair sob gatilho

- **Origem**: Netflix (7 anos pra quebrar) + Instagram Django monolito milhões LOC — [[../13-research/netflix/_destilado]] §1 + [[../13-research/instagram/_destilado]] §9.
- **Aplicação**: Go monolito modular em M1 com BCs bem separados. Ordem provável de extração: Content → Assembly → Commercial.
- **Marco**: M1 monolito; extração M3+. ADR [[adr/0001-clean-architecture-ddd-cqrs]].

### 1.12 Mux-as-YouTube — terceirizar 100% pipeline vídeo

- **Origem**: YouTube VCU + Google Media CDN (operação cara) — [[../13-research/youtube/_destilado]] TL;DR.
- **Aplicação**: Mux resolve upload tus.io + transcode ABR + HLS + CDN + signed URL + QoE. Master Síndico só emite webhook handler.
- **Marco**: **já feito**. ADR [[adr/0010-mux-video-provider]].

### 1.13 Moderação cascade: heurística → LLM → humano

- **Origem**: TikTok MLLM + LinkedIn Sigma + YouTube Content ID (decisão: **não** copiar Content ID) — [[../13-research/tiktok/_destilado]] §3, [[../13-research/linkedin/_destilado]] §4, [[../13-research/youtube/_destilado]] §4.
- **Aplicação**: M1 manual 48h SLA; M2+ AWS Rekognition Video + OpenAI Moderation + fila humana quando score médio. M3+ custom classifier se volume justificar.
- **Marco**: M1 manual; M2+ cascade. ADR [[adr/0024-moderation-cascade]].

### 1.14 Feed retrieval ≠ ranking (2-stage)

- **Origem**: Instagram Explore, YouTube DNN recsys, TikTok Monolith — [[../13-research/instagram/_destilado]] §1 + [[../13-research/youtube/_destilado]] §5 + [[../13-research/tiktok/_destilado]] §1.
- **Aplicação**: Connect Me busca — retrieval OpenSearch (filtros categoria + região + disponibilidade) + re-rank determinístico Go-side (score composto ponderado).
- **Marco**: **M1 determinístico**. ADR [[adr/0023-recsys-deterministic-m1]].

### 1.15 Recsys determinístico M1, reavaliar ML M3+

- **Origem**: Netflix + TikTok + LinkedIn convergentes — ML só com volume de labels — [[../13-research/netflix/_destilado]] §5 + [[../13-research/tiktok/_destilado]] §1 + [[../13-research/linkedin/_destilado]] §5.
- **Aplicação**: Connect Me ranking M1 = `w1·rating + w2·recency + w3·1/distance + w4·success_rate + w5·tier_multiplier`. Sem ML até ≥10k interações/mês.
- **Marco**: **M1 deterministic**. ADR [[adr/0023-recsys-deterministic-m1]].

### 1.16 Reputação determinística (nunca ML caixa-preta)

- **Origem**: LinkedIn endorsements evoluíram mas master-sindico diverge pra **explicabilidade LGPD Art. 20** — [[../13-research/linkedin/_destilado]] §5.
- **Aplicação**: ReputationCalculator puro Go, função dos inputs (contratos fechados, reviews, condomínios atendidos, vídeos publicados). Publicada + auditável + explicável.
- **Marco**: **M1** e por toda vida do produto. Parte de [[../STEERING]] §39.

### 1.17 SFU WebRTC (LiveKit) para assembleias — não Meet, não Mux Live

- **Origem**: Google Meet SFU + Jitsi + Zoom convergência — [[../13-research/google-meet/_destilado]] §2.
- **Aplicação**: LiveKit Cloud M1; Room Composite Egress → MP4 S3 para ata legal. E2EE **off** (quebra gravação).
- **Marco**: **M3** (assembleia live entra aqui). ADR [[adr/0011-livekit-sfu]].

### 1.18 Votação RT em DataStream + fonte de verdade em HTTP

- **Origem**: Google Meet DataChannel patterns — [[../13-research/google-meet/_destilado]] §4.
- **Aplicação**: UI otimista via LiveKit DataStream `votacao:<id>`; persistência em Postgres via POST HTTP é fonte oficial. Voto = documento legal.
- **Marco**: **M3**.

### 1.19 Outbox pre-messaging (Postgres antes de NATS)

- **Origem**: LinkedIn "The Log" + critério 3×3 — [[../13-research/linkedin/_destilado]] §1.
- **Aplicação**: tabela `outbox_events` + worker poll M1. NATS JetStream só quando ≥3 produtores × 3 consumidores OU replay histórico.
- **Marco**: **M1 outbox**; NATS M2+. ADR [[adr/0019-nats-jetstream-threshold]].

### 1.20 Idempotência obrigatória em webhook + consumer

- **Origem**: Google Pub/Sub ("subscriber to be idempotent") + Stripe docs — [[../13-research/google-arch/_destilado]] §5.
- **Aplicação**: HMAC antes de parse + Redis `SET NX` 24h em webhooks inbound; tabela `processed_events` em consumers.
- **Marco**: **M1**.

### 1.21 Lock 90d em vídeos (diferenciador, não copiado)

- **Origem**: **decisão nossa** — YouTube/TikTok/IG não fazem. Requisito regulatório/governança BR — [[../13-research/youtube/_destilado]] §8.
- **Aplicação**: `Video` aggregate com `LockPeriod` VO; moderação pré-publish obrigatória; correção vira novo asset com redirect.
- **Marco**: **M2**. Parte de [[../STEERING]] §40.

### 1.22 Timeline imutável (diferenciador)

- **Origem**: **decisão nossa** — contraposição explícita a feed "freshness" Instagram/TikTok. Inspiração em event-sourcing clássico.
- **Aplicação**: `timeline_entries` INSERT-only, sem UPDATE/DELETE, sem `deleted_at`. Correção vira nova entry com `correction_of`.
- **Marco**: **M1**. Parte de [[../STEERING]] §41.

### 1.23 Vote único por (agenda_item, voter) — UNIQUE + ErrConflict

- **Origem**: **decisão nossa** — mas padrão herdado de sistemas de voto formais.
- **Aplicação**: UNIQUE DB constraint `(agenda_item_id, voter_id)`; use case trata `ErrConflict` (A-025 resolvido legado).
- **Marco**: **M3**. Parte de [[../STEERING]] §42.

### 1.24 Connect Me é unidirecional, não chat

- **Origem**: Meta Messenger E2EE custou anos, moderação em E2EE impossível, LGPD complica — [[../13-research/instagram/_destilado]] §7.
- **Aplicação**: formulário RFP estruturado, proposta estruturada, sem thread. Cotas anuais.
- **Marco**: **M1**. Parte de [[../STEERING]] §38 + ADR [[adr/0023-recsys-deterministic-m1]].

### 1.25 REST + OpenAPI 3.1, não GraphQL M1/M2

- **Origem**: Instagram usa REST (GraphQL nasceu em Meta mas Instagram não adota) — [[../13-research/instagram/_destilado]] §8.
- **Aplicação**: OpenAPI 3.1 contrato-first; oapi-codegen; versioning /v1. GraphQL reavaliação só M3+ com 3+ clientes heterogêneos.
- **Marco**: **M1**. Ver [[api-design]].

### 1.26 Unified content filtering library

- **Origem**: LinkedIn Trust & Safety — [[../13-research/linkedin/_destilado]] §4.
- **Aplicação**: `internal/safety/content` biblioteca interna usada por harassment report, assembly comment, Connect Me message, post body.
- **Marco**: **M2**.

### 1.27 Anti-fake empresa: device fingerprint + IP velocity + behavioral signals

- **Origem**: LinkedIn anti-fake (CNPJ/CPF sozinho não basta) — [[../13-research/linkedin/_destilado]] §4.
- **Aplicação**: Connect Me onboarding + CNPJ Receita + device fingerprint + IP velocity (>3 cadastros/hora = captcha + revisão); behavioral post-signup (>50 propostas em 1 dia = shadow-ban).
- **Marco**: **M2**.

### 1.28 Avaliações dupla-cegas (padrão Airbnb) para evitar retaliação

- **Origem**: LinkedIn endorsements + adaptação Airbnb — [[../13-research/linkedin/_destilado]] §7.
- **Aplicação**: avaliação bilateral síndico↔empresa só revela depois de ambas submeterem ou 14 dias.
- **Marco**: **M2**.

### 1.29 Two-tier cache: write-through ABAC + write-behind feed/ranking

- **Origem**: Meta TAO — [[../13-research/instagram/_destilado]] §3.
- **Aplicação**: ABAC cache write-through (segurança não tolera lag); ranking Connect Me write-behind (5-15min lag OK).
- **Marco**: **M1**.

### 1.30 Não construir CDN própria — Mux + Cloudflare

- **Origem**: Netflix Open Connect (construir CDN custa anos) — [[../13-research/netflix/_destilado]] §4.
- **Aplicação**: Mux faz vídeo (inclui CDN), Cloudflare para assets estáticos. Nunca construir.
- **Marco**: **M1**.

### 1.31 Não construir Content ID de copyright

- **Origem**: YouTube investiu $100M+ — [[../13-research/youtube/_destilado]] §4.
- **Aplicação**: cláusula contratual de termo de uso + moderação manual + AudD/ACRCloud fingerprint SaaS só se problema emergir.
- **Marco**: **M1**.

### 1.32 Stories ephemeral: **rejeitar** explicitamente

- **Origem**: Meta Stories (contra valor core) — [[../13-research/instagram/_destilado]] §2.
- **Aplicação**: Master Síndico valoriza memória institucional permanente. Timeline imutável é diferencial. Transitoriedade contra valor.
- **Marco**: **nunca**. Parte de [[../STEERING]] §41.

---

## 2. Matriz de decisão rápida — quando aplicar cada princípio

| Princípio | M1 | M2 | M3+ | Rejeitado |
|---|---|---|---|---|
| 1.1 Multi-tenant row-based | ✅ | — | avaliar DB-per-tenant enterprise | instance-per-tenant |
| 1.2 ULID PK | ✅ | — | — | UUIDv4 |
| 1.3 Zitadel OIDC + PKCE | ✅ | — | avaliar Cognito/Auth0 se enterprise pedir | senha local |
| 1.4 ABAC engine | ✅ | — | OPA/Cedar avaliar | RBAC puro |
| 1.5 1-device policy | ✅ | atestação plataforma | continuous risk scoring | sessão ilimitada |
| 1.6 Step-up MFA | ✅ | context-aware (geo/hora) | risk-based automático | MFA universal (friction) |
| 1.7 Resilience stack | ✅ | — | chaos engineering | sem CB |
| 1.8 SLI/SLO | ✅ 99.5% | 99.9% | SLA contratual | — |
| 1.9 Postmortem | ✅ template | trend analysis | ML prediction | — |
| 1.10 OTel Dapper | ✅ | sampling adaptativo | tail-based | sampling 100% |
| 1.11 Monolito modular | ✅ | — | extrair Content/Assembly | microservices dia 1 |
| 1.12 Mux video | ✅ | — | — | CDN próprio |
| 1.13 Moderação cascade | manual | Rekognition+LLM | custom classifier | autopublish |
| 1.14 Feed 2-stage | ✅ retrieval+rank | signals por tenant | ML ranker | 1-stage |
| 1.15 Recsys determinístico | ✅ | — | ML avaliar | ML dia 1 |
| 1.16 Reputação determinística | ✅ | — | **sempre** | ML caixa-preta |
| 1.17 LiveKit SFU | — | — | ✅ M3 | Meet/Zoom reimpl |
| 1.18 Voto RT + fonte HTTP | — | — | ✅ M3 | voto em WebSocket only |
| 1.19 Outbox → NATS | ✅ outbox | NATS se 3x3 | — | Kafka dia 1 |
| 1.20 Idempotência | ✅ | — | — | at-most-once |
| 1.21 Lock 90d | — | ✅ | — | — |
| 1.22 Timeline imutável | ✅ | — | — | soft-delete |
| 1.23 Vote único UNIQUE | — | — | ✅ M3 | contagem imprecisa |
| 1.24 Connect Me unidirecional | ✅ | — | — | chat livre |
| 1.25 REST OpenAPI 3.1 | ✅ | — | GraphQL BFF avaliar | GraphQL dia 1 |
| 1.26 Content filter library | — | ✅ | — | filtro ad-hoc |
| 1.27 Anti-fake | CNPJ Receita | +device+velocity+behavior | ML fraud | CNPJ sozinho |
| 1.28 Avaliação dupla-cega | — | ✅ | — | revela imediato |
| 1.29 Write-through ABAC | ✅ | — | — | eventual consistency em auth |
| 1.30 CDN não própria | ✅ | — | — | construir CDN |
| 1.31 Content ID não construir | ✅ | — | AudD se problema | construir fingerprint |
| 1.32 Stories ephemeral | ❌ | ❌ | ❌ | **banido** |

---

## 3. Leituras obrigatórias

Antes de PR em architecture/security:

1. [[../13-research/beyond-corp/_destilado]]
2. [[../13-research/google-arch/_destilado]] §1 (SRE) + §2 (multi-tenant) + §4 (OTel)
3. [[../13-research/netflix/_destilado]] §1 (monolito) + §2 (resilience) + §6 (obs)
4. [[../13-research/youtube/_destilado]] TL;DR + §1 + §3
5. [[../13-research/google-meet/_destilado]] TL;DR + §4
6. [[../13-research/tiktok/_destilado]] §1 + §3
7. [[../13-research/instagram/_destilado]] §1 + §3 + §7 + §8
8. [[../13-research/linkedin/_destilado]] §1 + §4 + §5 + §7

---

## 4. Vizinhos

- [[_moc]]
- [[c4-context]]
- [[c4-containers]]
- [[c4-components]]
- [[clean-arch-mapping]]
- [[patterns]]
- [[topology-multitenant]]
- [[api-design]]
- [[event-driven]]
- [[observability]]
- [[adr/_moc]]
