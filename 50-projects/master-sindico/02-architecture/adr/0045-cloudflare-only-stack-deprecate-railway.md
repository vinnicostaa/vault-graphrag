---
title: ADR-0045 — Cloudflare como stack única de infra; Railway descontinuado
type: adr
tags:
  - adr
  - infra
  - cloudflare
  - railway-deprecated
  - master-sindico
  - v2
  - fase-15
status: accepted
date: '2026-04-27T00:00:00.000Z'
created: '2026-04-27T00:00:00.000Z'
updated: '2026-04-27T00:00:00.000Z'
supersedes:
  - 0017-railway-initial-aws-m4
  - 0040-cloudflare-edge-primary-railway-origin
related_decisions:
  - D-138
---

# ADR-0045 — Cloudflare-only stack; Railway descontinuado

## Status

`accepted` (clarificado 2026-04-27 19:45 com nova evidência) — Cliente João trouxe consulta direta com Cloudflare AI confirmando **Workers VPC** (beta público desde nov/2025 VPC Services, abr/2026 VPC Networks) como pattern Cloudflare-blessed para backend long-running externo via Tunnel. Isso resolve a tensão técnica identificada na revisão anterior.

**Histórico desta ADR (transparência)**:
1. 2026-04-27 ~19:00: criada como `accepted` aplicando ordem cliente — **erro Agente 1** (não completou Research-Loop §4 + Dual-check §5).
2. 2026-04-27 ~19:30: rebaixada para `proposed (pending dual-check)` ao detectar tensão (CF Containers beta + CF docs dizendo "long-running compute fora de CF").
3. 2026-04-27 ~19:45: cliente trouxe **nova evidência decisiva** (Cloudflare AI: **Workers VPC** é o pattern oficial). Promovida de volta a `accepted` com Opção C explicitamente escolhida.

**Substitui**: [[0017-railway-initial-aws-m4]], [[0040-cloudflare-edge-primary-railway-origin]]. **Implementa**: [[../../STATE#D-138]].

## Decisão técnica fechada: Opção C — Workers VPC

A análise inicial listou 3 opções (A: CF Containers, B: rewrite Workers TS, C: híbrida). Nova evidência cliente (Cloudflare AI 2026-04-27 19:45) consolida **Opção C como escolhida**:

### Padrão Cloudflare Workers VPC

```
[Browser/Mobile]
        │
        ▼
[Cloudflare Edge / PoP] ──┬── Pages (web SPAs, 7 apps)
                          ├── Workers (BFF, rate-limit, auth, cache, webhooks)
                          ├── R2 / KV / Durable Objects (storage CF-nativo)
                          └── Workers VPC binding ──┐
                                                   │ (private tunnel via cloudflared)
                                                   ▼
                              [VPC privada externa]
                              ├── Backend Go (Gin) — full long-running
                              ├── Postgres 18 (RLS, partitioning, sqlc)
                              ├── Redis 7 (ABAC cache)
                              ├── NATS JetStream (eventos cross-BC)
                              └── Worker async (jobs, sagas)
```

### Por que WASM não foi escolhido como substituto

Avaliado 2026-04-27 (research-loop completo via Context7 docs.cloudflare.com + workers.md vault + CF AI). Compilar backend Go atual para WASM Workers **não é viável**:

| Stack atual | Quebra em WASM Workers |
|---|---|
| Gin / net/http | sem syscalls TCP socket; WASI restrito |
| pgx Postgres driver | TCP raw proibido |
| sqlc generated | depende pgx — quebra junto |
| goose migrations | TCP — quebra |
| NATS JetStream client | TCP — quebra |
| Worker async (jobs) | goroutines completas restritas em Go→WASM |
| Qualquer dep CGO | proibido em WASM Workers |
| Binário Go padrão ~10-15MB | estoura Workers paid 10MB limit (TinyGo reduz para 1-2MB mas restringe stdlib drasticamente) |
| Cold start | binário grande → maior cold start → perde benefício edge |

Doc oficial Cloudflare Workers WASM (consultado via Context7 2026-04-27): "Compiling to WebAssembly often requires including additional runtime dependencies, making Wasm Workers typically larger than equivalent JavaScript Workers. A larger Worker may take longer to start."

Cloudflare AI (consultado pelo cliente 2026-04-27): "sem rede direta, sem goroutines completas, sem net/http tradicional, binários podem ficar grandes. Ideal para lógica computacional, parsing — **não para servidores HTTP tradicionais**".

Substituir backend Go por WASM seria equivalente à **Opção B (rewrite)** já descartada — só que pior por overhead WASM↔JS marshaling em I/O.

### WASM como complemento pontual (M2+ caso a caso)

WASM Go pode ser útil em casos específicos como **complemento** da Opção C, não substituto:

| Caso | Justificativa |
|---|---|
| Validação HMAC de webhook (Stripe/Mux/LiveKit) | crypto puro; vault security §5 exige HMAC antes de parse — Worker WASM faz na edge antes de chegar ao backend |
| Geração de QR code (ID condomínio 8 chars + QR) | pura função; zero I/O |
| Fórmula Reputação Bronze→Diamante ([[0023-recsys-deterministic-m1]]) | determinística; pode rodar na edge para cache |
| Parsing customizado | sem I/O dependency |

**Para nada disso WASM Go é estritamente melhor que TS direto** (Hono + Zod + jose + bcrypt já cobrem). WASM Go entra **só se houver lógica Go canônica que valha reusar sem rewrite TS**. Avaliação caso a caso M2+, fora do escopo M1.

### Configuração Worker → Backend Go via VPC

`wrangler.jsonc`:
```jsonc
{
  "name": "mastersindico-api",
  "main": "src/index.ts",
  "vpc_services": [
    {
      "binding": "GO_BACKEND",
      "service_id": "<provisionado por wrangler vpc service create>",
      "remote": true
    }
  ]
}
```

`Worker src/index.ts` (BFF):
```ts
export default {
  async fetch(request, env, ctx) {
    // edge: rate limit, auth perimeter, cache
    const response = await env.GO_BACKEND.fetch(`http://backend.internal:8080${new URL(request.url).pathname}`);
    return response;
  }
}
```

### Provedor de origin externo: AWS sa-east-1 (D-138-E FECHADO 2026-04-27 20:30)

Cliente confirmou conta AWS criada. **AWS sa-east-1** ressuscita parte da ADR-0017 (que previa AWS ECS M4+) — agora aplicada desde M1, pulando Railway.

#### Matriz de responsabilidades (CF + AWS + Twilio + Resend)

**🟧 Cloudflare**: Pages, Workers BFF, Workers VPC binding, R2, KV, Durable Objects, Queues, Cron, Access (admin SSO+MFA), Email Routing inbound, Turnstile, cloudflared Tunnel, DNS+WAF+DDoS+CDN.

**🟦 AWS sa-east-1**: ECS Fargate (Backend Go), RDS PG18 Multi-AZ (RLS+partitioning), ElastiCache Redis 7 (ABAC TTL), OpenSearch Service (search canônico → fecha ADR-0042 Opção A), ECR, CloudWatch, Secrets Manager, IAM, VPC+SG+KMS.

NÃO usar AWS para: S3 (R2 vence) · Route53 (CF) · CloudFront (CF) · WAF (CF) · API Gateway/Lambda (Workers) · Cognito (Zitadel) · SES (Resend) · SNS (Queues+Twilio).

**🟪 Twilio**: ADR-0041 mantida. SMS prod (OTP, reset, notificações).

**🟩 Resend**: ADR-0031 mantida. Email outbound dev/staging/prod. Bounces → CF Email Routing.

**🟨 SaaS externos mantidos**: Stripe, Mux, LiveKit, Zitadel, Sentry.

#### Decisões derivadas fechadas

- **D-138-B (Postgres)**: ✅ RDS sa-east-1 Multi-AZ (não Neon/Supabase)
- **D-138-D (search)**: ✅ OpenSearch sa-east-1 — ADR-0042 promove Opção A `accepted`
- **D-138-A (CF Containers)**: REVOGADO (Workers VPC + AWS suficiente)
- **D-138-C (custo)**: ~$305-345/mês M1 (validar via AWS Pricing Calculator)

#### Histórico (provedor escolhido)

Análise inicial listou Fly.io GRU, Hetzner FSN, AWS sa-east-1, Render. Cliente confirmou AWS já criada → escolhido por: maturidade, multi-AZ nativo, RDS managed, OpenSearch managed, IAM granular, DPA LGPD, sa-east-1 Brasil.

### LEGACY — análise inicial pré-decisão (mantida para referência histórica)

Vault canônico Cloudflare overview §6 elenca as alternativas: AWS, Railway, Fly.io. Critérios:

| Critério | AWS ECS | Fly.io | Hetzner | Render |
|---|---|---|---|---|
| Multi-AZ nativo | ✅ | ✅ (regions) | ⚠️ DC único típico | ✅ |
| Custo M1 baseline | $$ | $ | $ | $ |
| DPA LGPD | ✅ | ✅ | ✅ | ✅ |
| Region BR / latência | ✅ sa-east-1 | ✅ gru | ⚠️ FSN/HEL | ⚠️ Oregon |
| DX deploy | médio | bom | médio | bom |
| Postgres managed integrado | RDS | Fly Postgres | external | Render Postgres |

**Recomendação Agente 1 para o João decidir D-138-E**: avaliar Fly.io GRU (Brasil, Postgres managed embed, custo médio, DX wrangler-similar) ou Hetzner FSN (custo mínimo, mais ops). Se enterprise scale visa M2+ → AWS ECS sa-east-1 já documentado em ADR-0017 supersedida (dá pra ressuscitar partes).

### Postgres provider (D-138-B atualizado)

Duas configurações possíveis dado opção C:
- **C1**: Postgres do mesmo provider de Go (RDS se AWS, Fly Postgres se Fly, ...) — menor latência app↔DB
- **C2**: Postgres SaaS externo (Neon/Supabase) via Hyperdrive — write replicas globais; bom para escalabilidade futura

C1 simplifica M1; C2 prepara escala. Decisão derivada de D-138-E.

**Substitui (quando aceita)**: [[0017-railway-initial-aws-m4]], [[0040-cloudflare-edge-primary-railway-origin]]. **Implementa**: [[../../STATE#D-138]].

## ⚠️ Tensão técnica detectada (vault canônico vs ordem cliente)

Leitura sistemática de `10-knowledge/providers/cloudflare/{_moc,overview,workers,d1,usage-in-projects}.md` revelou conflitos entre ordem cliente "Cloudflare-only" e best practices canonizadas no vault:

1. **CF `_moc.md` §"Quando NÃO usar Cloudflare"**: explicitamente afirma "Compute long-running (>30s CPU) — usar Railway/AWS ECS/Fly.io". Backend Go do Master Síndico (Gin + sqlc + worker async) é exatamente long-running.

2. **CF `workers.md` §"Quando NÃO usar"**: "CPU > 30s por request → Railway/ECS/Lambda". E Workers só roda JS/TS/WASM, não Go nativo.

3. **CF `d1.md` §"Quando NÃO usar"**: "OLTP complexo (joins multi-tabela pesados, ACID cross-region forte, > 100 GB) → Postgres externo via Railway ou RDS". Backend MS usa RLS, partitioning, sqlc, goose — todas complexas. D1 limit 10GB/DB hard.

4. **CF Containers**: `_moc.md` lista "Container MCP" mas como "Sandbox envs" — produto compute Containers (lançado 2024+ beta) **não destilado no vault**. Status GA não confirmado em pesquisa Context7.

5. **CF `usage-in-projects.md`**: literalmente "🔴 Não instrumentado — zero projetos no vault usam Cloudflare ainda (2026-04-24)". Master Síndico será o **primeiro adopter**, sem pattern catalogado de migração.

### Implicações honestas

A ordem cliente "Cloudflare-only" tem 3 caminhos viáveis, todos com riscos:

- **A. Cloudflare Containers (beta?)** para backend Go — pendente verificar GA pre-M1; risco se ainda beta
- **B. Rewrite Go → Workers TS (Hono)** — viola escopo M1 (21 dias); abandona toda stack atual (Gin/sqlc/goose/zerolog)
- **C. Híbrida**: Frontend Pages + APIs Workers BFF + backend Go EM OUTRO provider de origin (Hetzner/Fly.io/Render) + Hyperdrive→Postgres externo. Tecnicamente "Cloudflare-first" mas não "Cloudflare-only".

**Recomendação Agente 1**: validar com cliente se a ordem é (a) literal "100% Cloudflare" — implica A ou B (alto risco M1) — ou (b) "Cloudflare como camada principal e provider único onde possível" — habilita C.

A 2ª interpretação combina ordem cliente com viabilidade técnica + best practices canonizadas. A 1ª exige aceitar risco M1 OU postergar M1.

## Contexto

Em 2026-04-27 o cliente João determinou:

> *"nao iremos usar a railway, vamos usar a cloudflare"*

Decisão revoga toda arquitetura híbrida documentada em ADR-0040 (Cloudflare edge + Railway origin) e ADR-0017 (Railway M1 → AWS ECS M4+). **Railway é descontinuado completamente**; Cloudflare cobre **toda** a infra (edge + origin + storage + database + cache + auth perimeter + email perimeter).

A motivação cliente combina:
- Vendor consolidation (1 fornecedor, 1 DPA LGPD, 1 fatura, 1 status page)
- Zero-egress fees em R2 vs custos transferência outbound Railway
- Defense in depth nativo (DDoS, WAF, Access)
- Custo total esperado menor em escala M2+

## Decisão

**Cloudflare é a stack única de infraestrutura do Master Síndico**.

### Mapa atualizado de responsabilidades

```
                          ┌──────────────────────────────────────┐
                          │       CLOUDFLARE (única infra)       │
   browser/mobile  ───▶   │                                      │
                          │  Pages: SolidJS web (7 apps)         │
                          │  Workers: BFF + APIs (substitui Go)  │
                          │  Containers: backend Go quando GA    │
                          │  D1 ou Hyperdrive+PG ext (DB)        │
                          │  KV ou Durable Objects (cache/state) │
                          │  R2 (uploads, vídeos, exports LGPD)  │
                          │  Access (SSO+MFA admin)              │
                          │  Email Routing (inbound)             │
                          │  Turnstile (CAPTCHA)                 │
                          │  Logpush (observability)             │
                          └──────────────────────────────────────┘
```

### Mapeamento componente ⇒ produto Cloudflare

| Componente atual / planejado | Antes (ADR-0040) | Agora (ADR-0045) |
|---|---|---|
| Web SolidJS (7 apps) | Railway static + edge CF Pages | **Cloudflare Pages** (1 projeto Pages por app) |
| API Backend Go (Gin) | Railway compute (Tunnel ↔ CF) | **Cloudflare Containers** (target M1; preview hoje) OU **rewrite incremental para Workers TS** se Containers não atingir GA pre-M1 |
| Postgres 18 (RLS, partitioning, sqlc) | Railway managed | **Hyperdrive + Postgres externo** (Neon ou Supabase managed) — D1 não cobre RLS/partitioning |
| Redis 7 (ABAC cache, sessions) | Railway managed | **Cloudflare KV** (sessions snapshot, conteúdo público) + **Durable Objects** (ABAC determinístico, contadores quota) |
| Worker (jobs, sagas, NATS JetStream) | Railway worker process | **Cloudflare Queues** (jobs assíncronos) + **Workers Cron Triggers** (scheduled) |
| Object storage (uploads, exports, vídeos) | R2 (já era CF) | **R2** (sem mudança) |
| Outbound email (transacional) | Resend SaaS externo | **Resend** mantido (sem alternativa CF outbound) |
| Inbound email | Cloudflare Email Routing | mantido |
| OIDC Zitadel | SaaS externo | mantido (SaaS) |
| Vídeo VOD/Live | Mux + LiveKit | mantidos (SaaS) |
| Stripe billing | SaaS externo | mantido |
| OpenSearch/Meilisearch | Railway docker | **Workers AI Vectorize** (avaliação M2) OU **Meilisearch Cloud externo** OU **Hyperdrive + Postgres pg_trgm** (M1 fallback) |
| OpenTelemetry → Grafana | Railway Grafana | **Cloudflare Logpush + Workers Analytics Engine** OU **Grafana Cloud externo** (mais comum) |
| Sentry | SaaS externo | mantido |

### Configuração de domínio (sem mudança vs ADR-0040)

- `mastersindico.com.br` → Pages (LP `lp`)
- `app.mastersindico.com.br` → Pages (`cms`)
- `auth.mastersindico.com.br` → Pages (`auth`)
- `academia.mastersindico.com.br` → Pages (`lms`)
- `vizinhanca.mastersindico.com.br` → Pages (`forum`)
- `assembleia.mastersindico.com.br` → Pages (`assembly`) + Workers WS proxy para LiveKit
- `admin.mastersindico.com.br` → Pages (`admin`) atrás de **Cloudflare Access SSO+MFA**
- `api.mastersindico.com.br` → **Workers** (BFF) → **Containers** (backend Go) ou direto Workers (rewrite)
- `cdn.mastersindico.com.br` → R2 público
- `webhooks.mastersindico.com.br` → Workers idempotentes (Stripe, Mux, Resend, LiveKit)
- `mail.mastersindico.com.br` → Resend outbound; MX = Email Routing inbound

### Estratégia dev / staging / prod

| Env | Edge + compute | Database | Storage |
|---|---|---|---|
| **dev (local)** | `wrangler dev` (Pages + Workers) + `wrangler containers run` | docker-compose Postgres OR Neon dev branch | R2 dev bucket |
| **staging** | Zone CF `staging.mastersindico.com.br` | Neon staging branch (ou Supabase staging project) | R2 staging buckets |
| **prod** | Zone CF `mastersindico.com.br` | Neon prod (Multi-AZ) ou Supabase prod | R2 prod buckets |

## Decisão técnica em aberto: Backend Go origin

Cloudflare Workers V8 isolates **não** rodam Go nativamente. Opções:

### Opção A — Cloudflare Containers (preferida)
- Status (verificar pre-M1): GA ou ainda beta
- Vantagem: sem rewrite; mantém Gin + sqlc + goose + zerolog atuais
- Risco: se ainda beta em 2026-05-18 (M1), pode não ser viável produtivo
- **Action:** dual-check status Containers via Context7 / Cloudflare docs **antes de codificar M1**

### Opção B — Rewrite incremental para Workers TypeScript
- Stack: Hono framework (Workers-native), drizzle-orm via Hyperdrive, zod
- Vantagem: 100% Cloudflare-native; latência sub-50ms global; Workers Cron + Queues nativo
- Custo: rewrite ~70% do backend Go (3-5 sprints adicionais)
- Risco: M1 em 21 dias úteis — rewrite inviável a tempo

### Opção C — Híbrida (M1 stopgap, M2 cleanup)
- Backend Go continua em provider externo (Hetzner/Fly.io/Render) durante M1
- Workers atuam como BFF + edge cache + auth perimeter
- M2: migração para Containers quando GA OU rewrite gradual
- Vantagem: respeita "Cloudflare-only" no edge + futuro 100% CF
- Desvantagem: cliente disse "não Railway" — é razoável interpretar como "não mais provider de origin tradicional"

**Recomendação Agente 1:** validar com o João qual opção (A/B/C) antes de iniciar implementação backend M1. Para o frontend web (este sub-projeto), nada bloqueia: Pages + Workers BFF é Cloudflare puro.

## Consequências

### Positivas
- **Vendor único** — 1 DPA, 1 console, 1 fatura, 1 status, 1 SLA
- **Zero-egress fees** R2 mantido
- **DDoS/WAF nativo** mantido
- **Edge global** mantido
- **Cloudflare Access** protege admin sem precisar provisionar infra extra
- **Custo M1 reduzido** vs Railway+CF (estimativa: -30 a -50% se Containers GA)
- **Migração futura** dentro do ecossistema CF é trivial (Pages → Workers, Workers → Containers, etc.)

### Negativas
- **Backend Go = bloqueio técnico** (ver §"Decisão técnica em aberto")
- **Postgres não é nativo CF** — depende de provider externo (Neon, Supabase) via Hyperdrive
- **Redis não é nativo CF** — KV é eventual; ABAC determinístico precisa Durable Objects (curva nova)
- **OpenSearch/Meilisearch não é nativo CF** — alternativas: Vectorize (limitado a vetores), pg_trgm (limitado), provider externo
- **Workers Containers em beta** = risco de timing M1
- **Capacitação time** em Wrangler/Workers/Containers/Durable Objects/Queues
- **DPAs adicionais**: Cloudflare DPA (já necessária) + provider Postgres (Neon/Supabase)

### Neutras
- **Mobile Flutter** continua usando `api.mastersindico.com.br` — abstração mantida
- **Zitadel/Mux/LiveKit/Stripe/Resend** permanecem SaaS externos (sem alternativa CF razoável)

## Migration plan (alto nível)

1. **Sprint atual (Sprint 10, M1)**:
   - Web: substituir 7 `railway.json` por `wrangler.toml` (Pages config) — **DT-WEB-001 + NEW-001 resolvidos juntos**
   - DNS: zone CF criada, subdomínios apontando para Pages preview
   - DPA Cloudflare assinada
   - Decidir A/B/C para backend Go (action item urgente)

2. **Sprint 10-11**:
   - Implementar opção escolhida (A/B/C) no backend
   - Migrar Postgres para Neon ou Supabase + configurar Hyperdrive
   - Migrar Redis usage para KV (sessions snapshot) + Durable Objects (ABAC)
   - Workers BFF para `/api/v1/*`
   - Cloudflare Access em `/admin/*`

3. **Sprint 11-12 (M2)**:
   - Logpush → Grafana Cloud
   - Cloudflare Queues para jobs assíncronos
   - Workers Cron para scheduled jobs
   - Removê NATS JetStream se Queues cobrir

4. **Pós-M1**:
   - Avaliar D1 para edge-cached reads (timeline público)
   - Vectorize para search se Workers AI for viável
   - Multi-region quando necessário

## Alternativas consideradas (e rejeitadas)

1. **Manter ADR-0040 híbrido** — rejeitada por decisão direta cliente
2. **Cloudflare full sem provider externo de Postgres** — rejeitada por D1 não cobrir RLS/partitioning críticos
3. **Volta para AWS ECS** — rejeitada por D-067 + decisão Cloudflare-only

## Pontos abertos / debts

- **D-138-A**: validar status Cloudflare Containers (GA?) — bloqueador M1 backend
- **D-138-B**: escolher provider Postgres externo (Neon vs Supabase vs PlanetScale) — dual-check ADR
- **D-138-C**: validar custo total CF projetado vs Railway+CF anterior
- **D-138-D**: avaliar se OpenSearch/Meilisearch é necessário em M1 (search vital — D-120) ou se pg_trgm cobre

## Referências

- [[../../STATE#D-138]] — decisão guarda-chuva
- [[../../../../10-knowledge/providers/cloudflare/_moc]] — catálogo CF (knowledge atemporal)
- [[../../../../10-knowledge/providers/cloudflare/access]]
- [[../../../../10-knowledge/providers/cloudflare/r2]]
- [[../../../../10-knowledge/providers/cloudflare/pages]]
- [[../../../../10-knowledge/providers/cloudflare/workers]]
- [[../../../../10-knowledge/providers/cloudflare/tunnel]]
- [[0017-railway-initial-aws-m4]] — **superseded** por esta
- [[0040-cloudflare-edge-primary-railway-origin]] — **superseded** por esta
- [[0026-admin-mfa-m1]] — MFA reforçado por Access
- [[0031-email-provider-resend-m1]] — outbound Resend mantido
- [[0042-search-provider]] — search provider precisa revisão à luz desta ADR
