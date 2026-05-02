---
title: Cloudflare — Overview (modelo mental)
type: note
tags:
  - provider
  - cloudflare
  - edge
  - overview
  - architecture
doc-consulted: 2026-04-24
created: 2026-04-24
updated: 2026-04-24
aliases:
  - Cloudflare overview
  - CF overview
---

# Cloudflare — Overview

Cloudflare é uma **edge cloud platform** — compute + storage + network + security executados em ~330 cidades, no equivalente topológico do PoP/ISP edge (camada mais próxima possível do usuário final, chamada "**eyeball network**" em CDN). Este arquivo cobre o modelo mental necessário antes de tocar em qualquer produto específico.

## 1. O que diferencia Cloudflare

Três atributos arquiteturais distintos de AWS/GCP/Azure tradicional:

1. **V8 isolates em vez de containers** (Workers). Compartilhamento leve, cold start <50ms, milhares de tenants num mesmo processo sem violar segurança.
2. **Global-by-default**. Você não escolhe região; o código roda onde há tráfego. Data locality existe mas com nuances (Data Localization Suite, R2 bucket jurisdiction).
3. **Zero egress fees em R2** (storage). Disrupção direta ao modelo S3 que cobra saída — bucket com 10TB e 50TB/mês de leitura custa $150 em R2 vs ~$4500 em S3.

## 2. V8 Isolates vs Containers vs Lambda

Entender isso é pré-requisito para não fazer analogia errada.

| Dimensão | Workers (V8 isolate) | Lambda (Firecracker microVM) | Container (ECS/K8s) |
|---|---|---|---|
| Isolamento | Process-level via V8 | microVM completa | Namespace Linux + cgroups |
| Cold start | <50ms (single isolate) | 100ms-1s | 1-10s |
| Memória | 128MB por isolate | 128MB-10GB | 0.5-N GB |
| CPU | 10ms free / 30s paid | 0-15min | Ilimitado |
| Concorrência | Milhares de isolates / processo | 1 invocação / microVM | N processos / container |
| Idioma | JS/TS + WASM (Rust/Go via wasi) | Qualquer (com runtime) | Qualquer |
| Custo mental | "Função serverless global" | "Função serverless regional" | "Servidor sempre ligado" |

**Implicações**:
- Workers é ótimo para **request/response rápido** (<30s) e **paralelismo alto**.
- **Não** é substituto para job de horas, pipeline de ML, ou compute long-running — use Railway/ECS/GKE.
- **Estado** em Workers vai em R2/D1/KV/Durable Objects (não local) — isolate morre depois da request.

## 3. Eyeball Network

Cloudflare construiu a rede **peering-first** nos últimos 15 anos: 12k+ ASs peered, 330+ cidades, ~95% população mundial a <50ms. O efeito:

- Request de usuário → chega no PoP Cloudflare mais próximo (não AWS us-east-1).
- Se resposta cacheável → servida no PoP (não volta ao origin).
- Se precisa compute → Worker roda no PoP.
- Se precisa storage → R2/D1/KV/DO procura camada mais próxima (R2 tem "automatic migration", DOs tem "jurisdictional").

**Contraste**: AWS CloudFront é CDN-sobre-AWS. Cloudflare começou como CDN → expandiu para compute/storage, preservando topologia peering.

## 4. Mental model por produto (visão rápida)

```
                    ┌────────────────────────────┐
                    │         User (eyeball)      │
                    └────────────┬───────────────┘
                                 │  DNS (CF DNS)
                                 ▼
                    ┌────────────────────────────┐
                    │    PoP Cloudflare (330+)    │
                    │  ┌──────────────────────┐  │
                    │  │  Rate limit / WAF    │  │
                    │  │  Turnstile / Access  │  │
                    │  └──────────┬───────────┘  │
                    │             │              │
                    │  ┌──────────▼──────────┐   │
                    │  │  Cache (CDN default)│   │
                    │  │  ├─ hit? serve       │   │
                    │  │  └─ miss ▼           │   │
                    │  ├──────────────────────┤  │
                    │  │  Workers (V8 isolate)│  │
                    │  │  │   bindings: R2,   │  │
                    │  │  │    KV, D1, DO,    │  │
                    │  │  │    Queues, AI     │  │
                    │  │  ▼                    │  │
                    │  │  Origin (opcional):   │  │
                    │  │  Pages / Tunnel /     │  │
                    │  │  AWS / Railway        │  │
                    │  └──────────────────────┘  │
                    └────────────────────────────┘
```

Leia de cima para baixo:
1. User faz request → CF DNS resolve → vai pro PoP mais próximo.
2. **Segurança-first**: WAF + rate limit + Turnstile + Access (se protegido) antes de qualquer lógica.
3. **Cache-first**: CDN default + Cache Rules + Workers Cache API — tiers.
4. **Compute**: Worker roda se caminho exige; consome bindings (R2, KV, D1, DO, Queues).
5. **Origin** (opcional): se Worker precisa buscar dados externos — Pages, Tunnel para servidor interno, ou AWS/Railway via Hyperdrive (Postgres) ou HTTP.

## 5. Bindings — como Workers consome outros produtos

Cloudflare não usa SDKs REST para ligar Workers a R2/KV/D1/DO/Queues. Usa **bindings**: referência tipada declarada em `wrangler.toml` e injetada no ambiente do Worker.

```toml
# wrangler.toml
name = "my-worker"
main = "src/index.ts"

[[r2_buckets]]
binding = "BUCKET"
bucket_name = "my-app-uploads"

[[kv_namespaces]]
binding = "CONFIG"
id = "abc123..."

[[d1_databases]]
binding = "DB"
database_name = "my-app"
database_id = "xyz789..."
```

```ts
// src/index.ts
interface Env {
  BUCKET: R2Bucket
  CONFIG: KVNamespace
  DB: D1Database
}

export default {
  async fetch(req: Request, env: Env): Promise<Response> {
    const obj = await env.BUCKET.get('key')              // R2
    const config = await env.CONFIG.get('feature-flag')  // KV
    const rows = await env.DB.prepare('SELECT ...').all()// D1
    return new Response(`hello`)
  }
}
```

**Vantagens**:
- Zero auth/network overhead — bindings são canais locais no PoP.
- Tipagem gerada (`@cloudflare/workers-types`) — compilador pega erros.
- Rotação de credenciais não afeta código do Worker.

## 6. Comparação com outras plataformas

| Task | Cloudflare | AWS | Railway | Vercel |
|---|---|---|---|---|
| Static site | Pages | S3 + CloudFront | — | Vercel |
| Serverless function | Workers | Lambda | — | Edge Functions |
| Object storage | R2 | S3 | — | Blob |
| SQL DB | D1 (SQLite) / Hyperdrive→PG | RDS | Postgres | — |
| Key-Value | KV / DO | DynamoDB / ElastiCache | Redis | — |
| Realtime/WS | Durable Objects | API Gateway WS | — | — |
| Queue | Queues | SQS | — | — |
| DNS | DNS | Route53 | — | — |
| CDN | CDN default | CloudFront | — | Incluso |
| WAF | WAF | WAF | — | — |
| Auth apps internos (BeyondCorp) | Access + Tunnel | IAM Identity Center | — | — |
| Edge compute pricing | Req-based baratíssimo | Lambda caro + egress | N/A (sempre ligado) | Edge includes |

## 7. Quando Cloudflare **não** é a resposta

Registrado no `_moc.md`; resumindo aqui os 5 critérios:

- Compute >30s CPU → Railway/AWS.
- GPU training → AWS/GCP.
- Postgres complexo → Postgres externo + Hyperdrive (cache connections from Workers) OU direto no Railway.
- Compliance de region pinning estrita → Cloudflare Data Localization Suite existe, mas AWS/GCP têm maturidade maior em DPA regional.
- SLO sub-10ms determinístico → Workers tem cold start <50ms mas não determinístico em free tier; considerar "always warm" via paid + scheduled pings.

## 8. Como o agente usa este nó

1. **Antes de integrar CF num projeto**: ler este arquivo + [[integration-guide]] + o arquivo do produto específico.
2. **Em dúvida sobre produto certo**: consultar tabela §6 e §7 deste arquivo.
3. **Consulta dinâmica de docs**: preferir [MCP docs.mcp.cloudflare.com](https://docs.mcp.cloudflare.com/mcp) sobre WebFetch.
4. **Debug de produção**: [MCP observability.mcp.cloudflare.com](https://observability.mcp.cloudflare.com/mcp).

## Fontes

- [Cloudflare Developer Platform](https://developers.cloudflare.com) (consultada 2026-04-24)
- [V8 Isolates vs Containers (Cloudflare blog — Kenton Varda)](https://blog.cloudflare.com/cloud-computing-without-containers/) (consultada 2026-04-24)
- [Cloudflare Network Map](https://www.cloudflare.com/network/) (consultada 2026-04-24)
- [Workers Limits](https://developers.cloudflare.com/workers/platform/limits/) (consultada 2026-04-24)

## Links

- [[_moc]]
- [[integration-guide]]
- [[patterns]]
- [[../../security/beyond-corp]]
- [[../railway/_moc]]
