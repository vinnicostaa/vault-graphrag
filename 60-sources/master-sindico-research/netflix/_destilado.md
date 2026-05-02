---
title: Destilado Netflix — síntese aplicável ao Master Síndico
type: note
tags: [netflix, destilado, master-sindico, architecture, research]
source: 12 fontes Netflix TechBlog / GitHub / InfoQ / Wikipedia
created: 2026-04-23
updated: 2026-04-23
aliases: [netflix-destilado]
---

# Destilado Netflix — síntese aplicável ao Master Síndico

> Síntese das 12 fontes (`source-01` a `source-12`) filtrada pelo prisma do Master Síndico: SaaS BR multi-tenant em Go, pré-M1 (2026-05-08), escala brasileira, não escala planetária. **Regra-ouro**: aproveitar os princípios, rejeitar a escala.

## Mapa de decisão — o que aplicar, o que NÃO

| Tema | Netflix faz | Nós aplicamos? | Quando |
|---|---|---|---|
| Monolito → microserviços | Fez em 7 anos | Modular monolith em Go agora; extrair aos poucos | M2+ (Conteúdo primeiro) |
| Circuit breaker | Hystrix (deprecado) → Istio | `sony/gobreaker` em calls externas | M1 (Stripe/Mux/Livekit/Zitadel) |
| Bulkhead | Thread pool | Semáforo + worker pool por provider | M1 |
| Retry + timeout | Resilience lib | `retry-go` + `context.WithTimeout` | M1 |
| Chaos engineering | Simian Army, Chaos Kong | Latency injection staging M3; Chaos Monkey blast=1 pod M4 | M3-M4 |
| CDN | Open Connect próprio | Mux pra vídeo + Cloudflare pra assets | M1 |
| Cache | EVCache / Dynomite | Redis single → Cluster M3 | M1 |
| Service discovery | Eureka (deprecado) → k8s | Kubernetes Service direto | quando for pra k8s (M2+) |
| API Gateway | Zuul → Istio Gateway | Nginx ingress OU Cloudflare + reverse proxy no app Go | M1 |
| CD pipeline | Spinnaker | GitHub Actions → Argo CD/Rollouts | M1 → M3 |
| Observability | Atlas + Mantis + Edgar | **OpenTelemetry + Grafana stack + Sentry** | M1 obrigatório |
| Polyglot persistence | Cassandra + ES + MySQL + Redis + Dynomite | **Postgres + Redis + S3 + Mux** | M1 — não crescer até ter problema |
| Recomendação | Deep learning + bandits + Hydra | Ordenação determinística; ML simples só se provar valor | M3+ se justificar |
| Search | Elasticsearch | Postgres FTS M1 → Meilisearch/Typesense M2+ | M1 com Postgres |

## 1. Quando faz sentido quebrar monolito em micro

### O que Netflix ensina
- Gatilho real foi **ponto-único-de-falha + deploy bottleneck** (DB corruption 2008).
- Transição durou **7 anos**.
- Começou extraindo o que não era core.
- Conway's law: reorganizar time ANTES de quebrar serviço.

### Pra Master Síndico
- **Hoje (M1)**: modular monolith em Go é a escolha certa. Bounded contexts bem separados (pacotes `internal/<context>`) dão 80% do benefício com 20% do custo operacional.
- **Gatilhos pra extrair** (ordem provável):
  1. **Conteúdo/Vídeos** — stack diferente (pipeline Mux pesado, long-running jobs), candidate a worker/service separado.
  2. **Assembleias Live** — picos de carga (Livekit burst), scaling independente faz sentido.
  3. **Connect Me** — marketplace pode ter features ML distintas, time dedicado.
- **Nunca chegar em federação GraphQL** — nosso horizonte 5 anos fica em gateway simples.
- **Anti-padrão evitar**: "vamos começar com microserviços pra ser cloud-native" → distributed monolith. Start mono.

Referência: `source-01`, `source-02`.

## 2. Resilience patterns em providers (Stripe, Mux, Livekit, Zitadel)

### Composição canônica
```
Retry( CircuitBreaker( Bulkhead( TimeLimiter( call_provider() ) ) ) )
```

### Configuração sugerida (inicial — medir e ajustar)

| Provider | Timeout | Retry (tentativas / backoff) | Circuit breaker (abre) | Bulkhead (concurrent) |
|---|---|---|---|---|
| **Stripe** | 10s | 3× exponencial base 200ms, jitter | > 50% erro em 20 req / 30s window | 20 concurrent |
| **Mux upload** | 30s | 3× exp, só se idempotency-key | > 30% erro / 30s | 10 |
| **Mux metadata** | 5s | 2× | idem | 50 |
| **Livekit room create** | 5s | **sem retry** (não idempotente) | > 50% erro | 5 |
| **Zitadel token refresh** | 3s | 3× rápido | > 30% erro | 50 (cache local token) |
| **Zitadel introspect** | 2s | 2× | idem | 100 |

### Lib Go obrigatória
- `github.com/sony/gobreaker` — circuit breaker.
- `github.com/avast/retry-go/v4` — retry + backoff + jitter.
- `context.WithTimeout` — nativo.
- `golang.org/x/sync/semaphore` — bulkhead leve.

### Fallback strategy (importante)

- **Stripe** indisponível → mostrar erro claro ao user, registrar intent, processar quando voltar. Nunca "mock success".
- **Mux** upload falhou → queue para retry background, UI mostra progress + retry automático.
- **Livekit** falhou → user retenta; log alerta se falhas > threshold.
- **Zitadel** introspect falhou → **SE token recém-validado (< 30s)** pode usar cache; caso contrário, negar auth (fail-closed).

### Regra-ouro

> **Fail-closed em auth/billing; fail-open em features cosméticas.**

Referência: `source-03`, `source-04`, `source-12`.

## 3. Chaos Engineering — quando introduzir

### Netflix ensina
- Pré-requisito: **observability madura** (métricas, alertas, runbooks).
- Começa blast radius = 1 instância, expande (AZ → região).
- Hipótese primeiro, experimento depois.

### Pra Master Síndico
- **M1 (2026-05-08) → M2 (≤ 2026 fim)**: **não introduzir**. Focar em testing tradicional (pirâmide 70/20/10, coverage thresholds 95/90/85/75 do vault).
- **M3 (início 2027)**: **latency injection em staging** — middleware que injeta 500ms-2s em X% de calls a Stripe/Mux/Livekit. Valida circuit breaker + timeout + fallback.
- **M4+**: Chaos Monkey em prod com blast = 1 pod, em janela de manutenção, com kill-switch.
- **Nunca**: Chaos Kong (não temos multi-região).

### Ferramenta sugerida
- **LitmusChaos** (CNCF, k8s-native) — quando estivermos em k8s.
- **Toxiproxy** — proxy TCP que injeta latência/dropped connections; leve, roda em docker-compose pra staging.
- **AWS Fault Injection Service** se formos AWS-heavy.

### Pré-requisitos (DoD pra introduzir chaos)

- [ ] SLO definido (ex: 99,9% API).
- [ ] Alerting em burn rate (não threshold cru).
- [ ] Runbooks publicados (por incidente comum).
- [ ] Rollback automatizado no CD.
- [ ] On-call rotation formalizado.

Referência: `source-05`, `source-10`.

## 4. CDN strategy — o que aprender de Open Connect

### Netflix construiu CDN própria. Nós **não vamos**.

### Lições aplicáveis sem construir CDN

1. **Proactive caching > reactive** — em deploy, popular cache antes de abrir tráfego (cache warming). Aplicável ao Redis de ranking Connect Me, biblioteca docs.
2. **Fill windows off-peak** — jobs pesados (reindex search, regenerate reports) rodar em horário de baixa (3-6h BRT).
3. **Prefetching no cliente** — app mobile Flutter prefetch próximos capítulos de curso quando user abre módulo. Reduz latência percebida.
4. **Hierarquia de cache** — cliente (PWA/app cache) → edge (Cloudflare) → origem (app Go + Redis + Postgres).

### Stack CDN real pra Master Síndico

| Camada | Uso |
|---|---|
| **Mux** | Vídeo (storage + ABR + DRM + delivery próprio) — SaaS, não operamos |
| **Cloudflare CDN** | Assets estáticos (logos, PDFs de atas, JS/CSS do dashboard) |
| **Cloudflare R2** | Object storage (S3-compatible, sem egress fee) — uploads de documentos |
| **Redis** | Hot data (ranking, session, rate limit) |
| **Postgres** | Source of truth |

### Quando reavaliar

- Mux custo > R$ 50k/mês sustentado (migrar pra Bunny.net ou Cloudflare Stream).
- Cloudflare não tender BR bem (improvável, têm PoP em SP).

Referência: `source-06`, `source-07`.

## 5. Recomendação/personalização — aplicável ao Master Síndico?

### Netflix ensina (mas em escala planetária)
- 3 camadas: offline (batch ML) / nearline (stream features) / online (serving).
- Contextual bandit pra artwork personalization.
- Multi-task learning (Hydra) consolida N modelos.
- > 80% consumo via recomendação.

### Connect Me (marketplace de empresas)

**Sim, aplicável em versão simplificada:**

- **M1**: ranking determinístico = função de (relevância textual da query, distância geo, reputação média, nº de contratos, data cadastro). Sem ML. Postgres query + window function.
- **M2**: adicionar **signals por tenant** — condomínio X costuma contratar qual tipo? boost. Cache em Redis, invalidar em evento.
- **M3+**: se volume justificar, **LambdaMART ou XGBoost** com features explícitas. Rodar como batch job diário, resultado em Redis. **Não deep learning**, **não bandits**.

### Feed de vídeos (Conteúdo)

- **M1**: curadoria manual + filtro por persona (síndico, porteiro, conselheiro).
- **M2**: "continue watching" (estado user) + "próximos no seu curso" (sequência do módulo).
- **M3+**: se biblioteca > 1000 vídeos, content-based simples (tags + embeddings título/descrição com modelo pequeno, ex: MiniLM). Cache de similarity matrix no Redis.

### O que NÃO aplicar

- Artwork personalization (não temos volume).
- Deep neural nets em produção (custo > benefício).
- Feature store dedicado (Postgres + Redis basta até termos > 1M users).
- Hydra / multi-task (um modelo por use case é mais simples e manutenível).

### Princípio aplicável desde já

**Separar offline / nearline / online mentalmente** mesmo sem ML:
- Offline: relatórios semanais de reputação, stats de consumo.
- Nearline: Kafka / NATS / Postgres LISTEN → atualiza contadores em tempo real.
- Online: request → lê pré-computado, não recalcula.

Referência: `source-11`.

## 6. Observability — stack canônica

### Do Atlas/Mantis pro nosso contexto

Stack canônica master-sindico (já decidida): **OpenTelemetry + Grafana stack + Sentry**.

### Lições Netflix aplicáveis diretamente

- **Dimensional tagging** — toda métrica com tags `tenant_id, route, status_code, method, product` (governanca/conteudo/connect_me/...).
- **RED + USE metrics** — Rate/Errors/Duration nos handlers; Utilization/Saturation/Errors nos pools e DB.
- **Rollup retention** — Prometheus recording rules agregam métricas de 15s → 1min → 1h conforme envelhece.
- **URL-shareable dashboards** — Grafana já faz; cultivar cultura de compartilhar links em PRs/ADRs.
- **Correlação trace ↔ log ↔ metric** via `trace_id` (OTel nativo).
- **SLO burn rate alerting** — alert quando queima 10% do error budget em 1h, não "threshold > 500ms".

### Ferramentas concretas

| Camada | Ferramenta |
|---|---|
| SDK | OpenTelemetry Go |
| Metrics backend | Prometheus → longo prazo VictoriaMetrics ou Grafana Mimir |
| Logs backend | Grafana Loki (multi-tenant nativo) |
| Traces backend | Grafana Tempo |
| Error tracking | Sentry (já decidido) |
| Dashboards + alerting | Grafana |
| Stream processing (se precisar) | Benthos/Bento — equivalente leve de Mantis |

### Anti-padrões

- Dashboard sem owner → vira lixo.
- Alert sem runbook → alert fatigue.
- Logging PII → **vault proíbe** (BC: no-PII-logs).
- Log sem `tenant_id` → debugging multi-tenant impossível.

Referência: `source-10`.

## 7. O que Netflix faz e NÓS NÃO DEVEMOS fazer

Lista de cristal pra evitar cargo-cult:

1. **Construir CDN própria** — Mux + Cloudflare. Fim.
2. **Adotar Cassandra / Elasticsearch prematuramente** — Postgres cobre 95% dos use cases no nosso horizonte.
3. **Spinnaker** — GitHub Actions + Argo CD é muito mais simples pra nossa escala.
4. **Service mesh (Istio) em M1** — monolito Go com gobreaker interno resolve.
5. **Deep learning pra recomendação** — LambdaMART/XGBoost em batch serve.
6. **Chaos Kong (multi-região failover)** — não somos global.
7. **Federated GraphQL gateway** — REST + OpenAPI 3.1 (vault padrão) cobre sobejamente.
8. **Custom observability platform** — Grafana stack é commodity resolvida.
9. **Feature store dedicado (Feast etc)** — Postgres + Redis até > 1M users.
10. **Reimplementar Hystrix/Eureka** — ambos deprecados; usar primitives Go + k8s.

## 8. Priorização concreta (M1 / M2 / M3)

### M1 (até 2026-05-08)
- [ ] Monolito modular Go com bounded contexts separados.
- [ ] `sony/gobreaker` + `retry-go` + `context.WithTimeout` em **toda** call external (Stripe, Mux, Livekit, Zitadel).
- [ ] OpenTelemetry SDK integrado; Grafana Cloud free tier ou self-host.
- [ ] Postgres + Redis + S3/R2 + Mux — stack persistence.
- [ ] GitHub Actions CI/CD; immutable container images (tag = commit SHA).
- [ ] SLOs definidos (99,9% API, p95 < 300ms).

### M2 (≤ 2026 fim)
- [ ] Canary deployment automatizado (Argo Rollouts se k8s).
- [ ] Cache warming post-deploy.
- [ ] Search dedicado (Meilisearch) se Postgres FTS ficar lento.
- [ ] Ranking Connect Me com signals por tenant.
- [ ] Dashboard por bounded context.

### M3 (2027 Q1-Q2)
- [ ] Latency injection em staging (Toxiproxy).
- [ ] Extrair primeiro microserviço (Conteúdo/Vídeos) se justificar.
- [ ] LambdaMART ranking se volume justificar.
- [ ] Postgres read replicas se contenção em reads.

### M4+ (2027 H2+)
- [ ] Chaos Monkey blast=1 pod em prod.
- [ ] Multi-cluster k8s se SLA justificar.

## 9. Links cruzados

- [[source-01]] — evolução API (monolito → federated)
- [[source-02]] — monolito → micro pitfalls
- [[source-03]] — Netflix OSS deprecação
- [[source-04]] — resilience patterns (CB, bulkhead, retry, timeout)
- [[source-05]] — chaos engineering
- [[source-06]] — Open Connect CDN
- [[source-07]] — EVCache/Dynomite
- [[source-08]] — polyglot persistence
- [[source-09]] — Spinnaker CD
- [[source-10]] — Atlas/Mantis observability
- [[source-11]] — recommendation system
- [[source-12]] — lições Netflix OSS
- [[_moc]] — MOC da pasta
- [[../_moc]] — MOC raiz 13-research
- [[../../02-architecture/_moc]] — destino: patterns arquiteturais
- [[../../CLAUDE]] — contrato da v2

## 10. Próximos passos (pós-destilação)

1. Abrir ADR em `02-architecture/adr/` pra cada decisão grande destilada:
   - ADR: "Modular monolith Go em M1, extrair micro sob gatilho"
   - ADR: "Stack resilience (gobreaker + retry-go + timeout)"
   - ADR: "Observability OTel + Grafana + Sentry"
   - ADR: "Persistence Postgres + Redis + S3 + Mux (polyglot mínimo)"
   - ADR: "CDN via Mux + Cloudflare; não construir"
   - ADR: "Chaos engineering introduzido em M3; staging primeiro"
2. Registrar decisões em `STATE.md` como D-### com referência ao `_destilado.md`.
3. Mapear `02-architecture/patterns.md` com os 5 patterns de resiliência canônicos.
