---
title: Aplicações Netflix → Master Síndico — cruzamento + gaps + priorização
type: note
tags: [netflix, master-sindico, aplicacoes, architecture, resilience, chaos, saga, cdn, cache, personalization]
source: 13-research/netflix/_destilado.md cruzado com 02-architecture/{patterns,event-driven,observability,research-inspirations,adr}/ + 09-testing/test-strategy + 01-domain/invariants + 00-product/sub-produtos/04-conteudo-videos + STATE
created: 2026-04-23
updated: 2026-04-23
aliases: [netflix-aplicacoes, netflix-gaps-master-sindico]
---

# Aplicações Netflix → Master Síndico

> Cruzamento do [[_destilado]] (12 fontes Netflix) com a arquitetura remontada do Master Síndico v2. Objetivo: identificar o que já está incorporado, onde existem gaps, e priorizar ações até M1 (2026-05-08) e além. **Regra-ouro**: aproveitar princípios, rejeitar escala planetária.

---

## 1. Estado atual — o que já está incorporado

Já destilado e formalizado na v2:

| Tema Netflix | Artefato v2 | Status |
|---|---|---|
| Circuit Breaker (Hystrix-legacy) | [[../../02-architecture/patterns#12-circuit-breaker-sonygobreaker-em-providers-externos]] + [[../../02-architecture/adr/0022-circuit-breaker-gobreaker]] | ✅ M1 — `sony/gobreaker` por provider (Stripe/Mux/LiveKit/Zitadel) |
| Retry + backoff + jitter + DLQ | [[../../02-architecture/patterns#13-retry-exponential-backoff-jitter-dlq]] + [[../../02-architecture/event-driven#8-retry-exponential-backoff-jitter]] | ✅ M1 — `avast/retry-go` |
| Saga com compensação | [[../../02-architecture/patterns#7-saga-com-compensação-inter-bc-provider-externo]] + [[../../02-architecture/adr/0015-uow-intra-saga-inter]] + [[../../02-architecture/adr/0030-saga-orchestration-default]] | ✅ M1 — orchestration default; 5 sagas canônicas (Contract, VideoPublish, Assembly, TrialExpiry, AccountDeletion) |
| Observability OTel + Grafana + Sentry | [[../../02-architecture/observability]] + [[../../02-architecture/adr/0020-opentelemetry-grafana-sentry]] | ✅ M1 — RED/USE, burn-rate alerting, dimensional tags |
| SLI/SLO + error-budget policy | [[../../02-architecture/observability#3-sli-slo-por-bounded-context]] + [[../../02-architecture/adr/0025-sli-slo-error-budget]] | ✅ M1 — 99.5% baseline, rolling 4w |
| Monolito modular → extrair sob gatilho | [[../../02-architecture/research-inspirations#111-monolito-modular-extrair-sob-gatilho]] + [[../../02-architecture/adr/0001-clean-architecture-ddd-cqrs]] | ✅ M1 — ordem Content → Assembly → Commercial |
| CDN não-própria | [[../../02-architecture/research-inspirations#130-não-construir-cdn-própria-mux-cloudflare]] | ✅ M1 — Mux+Cloudflare |
| Polyglot persistence mínima | PG 18 + Redis + R2/MinIO + Mux | ✅ M1 |
| Recsys determinístico | [[../../02-architecture/adr/0023-recsys-deterministic-m1]] | ✅ M1 — Connect Me ranking weighted, ML diferido |
| Chaos roadmap | [[../../09-testing/test-strategy#10-chaos-engineering-m3]] | 🟡 M3 — só plano textual (Toxiproxy), sem runbook nem hipóteses |
| Idempotência webhook | [[../../02-architecture/patterns#11-idempotency]] + [[../../02-architecture/adr/0027-webhook-idempotency-db]] | ✅ M1 |
| Outbox default, NATS ≥3×3 | [[../../02-architecture/event-driven#2-critério-para-ligar-nats-jetstream]] + [[../../02-architecture/adr/0019-nats-jetstream-threshold]] | ✅ M1 outbox |

**Total**: ~85% do destilado Netflix absorvido antes desta passagem. Gaps abaixo são os 15% restantes.

---

## 2. Gaps identificados (cruzamento fino)

### G1 — Bulkhead pattern não está formalizado

[[../../02-architecture/patterns]] menciona Circuit Breaker e Retry+Backoff em detalhes (§12, §13) mas **não tem seção Bulkhead**. [[_destilado#2-resilience-patterns-em-providers-stripe-mux-livekit-zitadel]] explicitamente prescreve `golang.org/x/sync/semaphore` por provider (Stripe 20 concurrent, Mux upload 10, LiveKit 5, Zitadel 50-100) — e [[research-inspirations#17-resilience-stack]] lista bulkhead como canônico, mas **não há ADR dedicada nem pattern documentado**. Sem bulkhead, um pico de Mux uploads satura goroutines e derruba routes de billing/identity no mesmo processo Go — clássica cascading failure cross-BC.

### G2 — Fallback strategy fail-closed/fail-open não está codificado como invariante

[[_destilado#2-resilience-patterns-em-providers-stripe-mux-livekit-zitadel]] §"Regra-ouro" define: **fail-closed em auth/billing, fail-open em cosmético**. Isso aparece em prosa no ADR-0022 mas não está listado em [[../../01-domain/invariants]] como invariante operacional auditável. Sem invariante, revisões de PR/dual-check não têm critério concreto para aceitar ou rejeitar uma branch CB-open.

### G3 — Bulkhead cross-tenant não existe

Netflix isola por tenant/region via cluster sharding. No Master Síndico, um tenant "ruidoso" (um condomínio grande disparando uploads em massa ou webhooks em lote) pode consumir 100% da quota do provider (Stripe/Mux) e degradar os outros. Nada no [[../../02-architecture/patterns]] ou [[../../02-architecture/topology-multitenant]] (não revisado aqui) fala sobre **quota per-tenant por provider** nem **semáforos tenant-aware**.

### G4 — Chaos engineering tem plano, mas falta DoR

[[../../09-testing/test-strategy#10-chaos-engineering-m3]] lista pré-requisitos corretos (SLO, burn-rate, runbooks, rollback), mas **não tem lista de hipóteses a validar** (ex.: "hipótese: se Stripe cai 30s, fila de webhooks drena em < 2min sem perda"). Sem hipóteses, experimentos viram teatro. Falta também escopo: quais sagas são candidatas prioritárias? `AssemblySaga` provavelmente (LiveKit+Egress+Mux em cascata).

### G5 — Saga orchestration está definida mas sem engine/state-store concreto

[[../../02-architecture/adr/0030-saga-orchestration-default]] define orchestration como default e cita `saga_instances`. Netflix usa **Conductor** (engine declarativa com DSL JSON + workers stateless). No master-sindico, cada saga será escrita hand-coded em Go. Risco M1: duplicação de boilerplate (state machine, retry, timeout, compensation chain) entre 5 sagas. Gap: **não há decisão sobre coordinator genérico** (ex.: `internal/cross-domain/saga/coordinator.go` com interface `Step{Do, Compensate, Idempotent}`) vs N implementações ad-hoc.

### G6 — Hot-path cache: estratégia mencionada, política de invalidação/TTL dispersa

[[research-inspirations#129-two-tier-cache]] define write-through ABAC + write-behind feed/ranking. Mas **inventário consolidado de chaves quentes** (ABAC decision, rate limit counters, Connect Me ranking, reputation tier, Mux playback signed URL, Zitadel token) com **TTL + invalidation trigger** não existe em um único lugar. Sem inventário, cada BC reinventa, TTLs divergem, invalidação quebra.

### G7 — Mux como "Open Connect" — PoP BR + preço não estão avaliados

[[_destilado#4-cdn-strategy-o-que-aprender-de-open-connect]] diz "Mux faz CDN" e lista trigger de migração (> R$50k/mês). Mas **não existe avaliação explícita de latência/PoP BR** para Mux delivery — Master Síndico é 100% BR, se Mux entrega do US-East teremos 100+ ms adicionais comparado a Cloudflare Stream (PoP GRU). [[../../00-product/sub-produtos/04-conteudo-videos]] confia 100% Mux sem essa análise. Gap: **baseline QoE BR** + **trigger-B secundário** (latência p95 > 400ms em RJ/SP).

### G8 — Personalization: feed Vizinhança e matching Banco de Talentos não foram cruzados com Netflix offline/nearline/online

[[_destilado#5-recomendaçãopersonalização]] separa offline/nearline/online e recomenda LambdaMART/XGBoost M3+ se volume justificar. [[../../02-architecture/adr/0023-recsys-deterministic-m1]] cobre **Connect Me** mas não menciona **feed Vizinhança** (comunidade do condomínio) nem **matching Banco de Talentos** (curriculum ↔ oportunidades). Ambos são candidatos naturais a content-based filtering simples (tags + regiao + reputação). Gap: **ranking determinístico explícito** nesses dois sub-produtos, mesmo que seja cópia reduzida do Connect Me.

### G9 — Cache warming pós-deploy não é parte do CD runbook

Netflix popula cache antes de abrir tráfego. Nosso deploy Railway (M1) e futuro Argo (M2) **não tem hook de cache warming** para Redis quente (ABAC policies mais usadas, ranking Connect Me top-100 por região, token introspection). Primeiro request pós-deploy paga cold-cache em cima de cold-JIT-Go. Gap operacional pequeno, mas existe.

### G10 — Load testing + chaos convergem: falta k6 cenário "provider down"

[[../../09-testing/test-strategy#ferramentas]] lista k6 mas sem cenário explícito "Stripe latência 2s por 30s" ou "LiveKit down durante assembleia". São experimentos baratos que validam CB+fallback antes de staging real.

---

## 3. Top-10 priorizado

Ordem = (impacto em M1) × (custo baixo) × (desbloqueia outros).

| # | Ação | Marco | Custo | Unlocks |
|---|---|---|---|---|
| **1** | **ADR-0033 — Bulkhead pattern por provider** (semaphore weight, tenant-aware opcional M2) | M1 | 4h | G1, G3 parcial; cascading failure cross-BC |
| **2** | **Formalizar invariante operacional `INV-OP-001` fail-closed/fail-open** em [[../../01-domain/invariants]] | M1 | 1h | G2; critério dual-check CB-open |
| **3** | **`internal/cross-domain/saga/coordinator.go` spec + interface `SagaStep`** (Go hand-coded, não Conductor) | M1 | 6h spec + 2h dual-check | G5; reduz boilerplate em 5 sagas canônicas |
| **4** | **`02-architecture/cache-catalog.md`** consolidando todas chaves quentes (key, TTL, invalidation trigger, fallback) | M1 | 3h | G6, G9; baseline pra alertas |
| **5** | **ADR-0034 — Bulkhead por tenant em providers críticos** (Stripe/Mux) via semaphore `tenant_id`-keyed | M2 | 4h ADR + 6h prototype | G3; tenant ruidoso não afeta outros |
| **6** | **`09-testing/chaos-hypotheses.md`** com 8-10 hipóteses (formato: se/então/medida/limite aceitável) | M2 prep (M3 execução) | 3h | G4; experimentos deixam de ser teatro |
| **7** | **Ranking determinístico Vizinhança + Banco Talentos** — ADR estendendo 0023 | M2 | 4h | G8; diferencial + paridade com Connect Me |
| **8** | **Mux QoE baseline BR** — 1 script probe (GRU/POA/REC) em staging medindo p50/p95 segmentos HLS; trigger-B documentado | M1 | 2h script + 1h doc | G7; protege decisão Mux longo prazo |
| **9** | **k6 cenário `provider-degraded.js`** simulando latência 2s Stripe + 30% 5xx Mux, asserta CB-open + DLQ < threshold | M2 | 4h | G10; validação CI contra regressão CB |
| **10** | **Cache warming hook em deploy** — script Go `bin/cache-warmer` executado post-release | M2 | 4h | G9; reduz p95 pós-deploy |

Itens 1-4 e 8 são **M1 obrigatório** antes de 2026-05-08 (28h total — 3-4 dias de um dev). Demais seguem M2.

---

## 4. ADRs novas propostas

| ID | Título | Escopo | Sprint |
|---|---|---|---|
| **ADR-0033** | Bulkhead por provider via `golang.org/x/sync/semaphore` | Provider-aware concurrency limits (Stripe 20, Mux upload 10, LiveKit 5, Zitadel 100). Composição canônica `Retry(CB(Bulkhead(Timeout(call))))`. Fallback hook explícito. | M1 |
| **ADR-0034** | Bulkhead tenant-aware em providers críticos | Semaphore `tenant_id`-keyed com quota configurável por plano. Alert quando tenant ≥ 80% quota. | M2 |
| **ADR-0035** | Saga coordinator genérico `SagaStep` interface | Eliminar boilerplate em 5 sagas M1. Define `Do(ctx, state) (state, error)` + `Compensate(ctx, state) error` + `Idempotent() bool`. State em `saga_instances`. **Rejeita** Netflix Conductor (overkill). | M1 |
| **ADR-0036** | Ranking determinístico estendido (Vizinhança + Banco Talentos) | Pesos explícitos + invariante recalculabilidade LGPD-Art.20. | M2 |
| **ADR-0037** | Mux CDN — QoE baseline BR + trigger-B migração | Probe GRU/POA/REC semanal. Trigger-A custo R$50k/mês, trigger-B p95 > 400ms sustentado. | M1 (doc) / M4+ (migração potencial) |

---

## 5. Decisões (D-###) e Dívidas (DT-###)

**Decisões novas para [[../../STATE]]**:

- **D-083** — Adotar bulkhead por provider como invariante arquitetural (ADR-0033). Justificativa: [[_destilado#2-resilience-patterns-em-providers-stripe-mux-livekit-zitadel]] + G1; fecha item 1 do top-10.
- **D-084** — Fail-closed em auth/billing, fail-open em features cosméticas — invariante operacional `INV-OP-001`. Justificativa: G2; base pra revisão dual-check.
- **D-085** — Saga coordinator genérico hand-coded em Go; **rejeitar** engine Conductor. Justificativa: complexidade Netflix planetária vs nossa escala BR; ADR-0035.
- **D-086** — Mux mantido em M1; trigger-B latência BR documentado; probe semanal staging. ADR-0037.
- **D-087** — Ranking determinístico canonicalizado também em Vizinhança e Banco Talentos, não só Connect Me. ADR-0036.

**Dívidas técnicas novas**:

- **DT-004** — `cache-catalog.md` inexistente; chaves/TTL/invalidation dispersos por BC. Risco: inconsistência cross-BC. SLA resolver em M1 (item 4 top-10).
- **DT-005** — Chaos hypotheses file ausente; plano M3 textual sem critério DoR. Resolver até fim M2 (item 6 top-10).
- **DT-006** — Tenant-aware bulkhead ausente; tenant ruidoso pode sequestrar quota provider. Mitigação M1: rate limit global Stripe SDK; resolução M2 via ADR-0034.
- **DT-007** — k6 cenário `provider-degraded.js` não existe; regressão de CB só pega em prod. Resolver M2 (item 9 top-10).
- **DT-008** — Cache warming hook ausente no CD; p95 pós-deploy degradado. Resolver M2 (item 10 top-10).

---

## 6. Invariantes (adicionar em [[../../01-domain/invariants]])

Invariantes operacionais complementares aos de domínio:

- **INV-OP-001** — **Fail-closed em auth/billing**: qualquer falha em Zitadel introspect (fora da janela de cache 30s) OU Stripe create/update subscription → **negar operação**, nunca mockar sucesso. Teste: integration com CB forçadamente open.
- **INV-OP-002** — **Fail-open em features cosméticas**: thumbnail Mux, contadores de timeline, ranking cache miss → **servir estado degradado** (placeholder, stale data) em vez de 5xx.
- **INV-OP-003** — **Bulkhead obrigatório** em toda call externa: nenhuma chamada provider pode ser feita fora de um semaphore nomeado + CB + timeout. Linter/code review bloqueia.
- **INV-OP-004** — **Compensação obrigatória em saga step com side-effect externo**: todo step que chama provider idempotente OU não-idempotente deve declarar `Compensate()` explícito (no-op permitido mas documentado).
- **INV-OP-005** — **Ranking determinístico + recalculável**: qualquer ordenação exibida ao usuário (Connect Me, Vizinhança, Banco Talentos, feed Conteúdo) deve ser reproduzível offline a partir dos inputs persistidos (LGPD Art. 20 — direito à explicação).

---

## 7. Vizinhos

- [[_destilado]] — destilado Netflix 12 fontes
- [[_moc]] — MOC Netflix
- [[../_moc]] — MOC 13-research
- [[../../02-architecture/patterns]] §7, §11, §12, §13
- [[../../02-architecture/event-driven]]
- [[../../02-architecture/observability]]
- [[../../02-architecture/research-inspirations]] §1.7, §1.11, §1.30
- [[../../02-architecture/adr/_moc]]
- [[../../09-testing/test-strategy]] §10
- [[../../01-domain/invariants]]
- [[../../00-product/sub-produtos/04-conteudo-videos]]
- [[../../STATE]] — decisões D-083..D-087, dívidas DT-004..DT-008
