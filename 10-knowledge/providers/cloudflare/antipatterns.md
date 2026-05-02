---
title: Cloudflare — Antipatterns (cross-produto)
type: antipattern
tags:
  - provider
  - cloudflare
  - antipatterns
  - edge
doc-consulted: 2026-04-24
created: 2026-04-24
updated: 2026-04-24
aliases:
  - CF antipatterns
  - Cloudflare antipatterns
---

# Cloudflare — Antipatterns (cross-produto)

Erros comuns que **atravessam 2+ produtos Cloudflare**. Gotchas específicos vão na nota do produto (R2 egress em `r2.md`, DO hotspot em `durable-objects.md`, etc.).

## 1. Confundir camadas de cache

**Sintoma**: dev bate cache via Cache Rules mas também instancia `caches.default` em Worker, e ainda cacheia em KV — sem entender como os 3 interagem.

**Três camadas reais**:

| Camada | Produto | Scope | TTL típico | Consistência |
|---|---|---|---|---|
| **Default CDN** | HTTP cache nativo (edge) | PoP | Seguindo headers `Cache-Control` | Por PoP |
| **Cache Rules / Cache API** | Workers `caches.default` | PoP (API explícita) | Configurável | Por PoP |
| **KV** | Workers KV | Global | TTL por key | Eventual (~60s) |

**Regras**:
- CDN default é para **resposta HTTP estática** (imagens, assets, API cacheável com headers).
- Cache API (`caches.default`) é para **programático** — você decide chave + TTL dentro do Worker.
- KV é para **dado pequeno que muda pouco e precisa global** (feature flag, config).

**Errado**: usar KV como cache de resposta HTTP (prefira Cache API — mais rápido, grátis).

**Errado**: usar Cache API para dado que precisa consistência global (prefira KV ou D1).

## 2. Vendor lock-in silencioso

**Sintoma**: código importa `@cloudflare/workers-types` em toda camada; handlers assumem `Request`/`Response` globais CF; DB queries usam `D1PreparedStatement`; saída do projeto vira impossível sem reescrita.

**Mitigação**:
- **Handler thin**: `export default { async fetch(req: Request, env: Env) { return app.fetch(req, env) } }` — toda lógica em `app` agnóstico (Hono, por exemplo).
- **Domain layer zero dependência de CF**: use cases aceitam interfaces (`IStorage`, `IKV`, `ICache`). Infra adapta CF bindings para essas interfaces.
- **Tests locais** não dependem de `wrangler dev --remote` — rodam em Node/Bun com mocks.

**Quando tolerar**: MVP com prazo e sem intenção de migrar em 18 meses. Documentar em `README` + ADR.

## 3. R2 egress misassumption

**Sintoma**: "R2 tem zero egress, então posso servir vídeo direto do bucket."

**Realidade**:
- Zero egress é para saída **pela internet quando cliente bate em endpoint R2 direto** (via signed URL ou Worker proxying).
- Class A operations (writes) e Class B operations (reads) **são cobradas** — $4.50 / 1M reads Classe B, $0.36 / 1M reads Classe A. 1M views de vídeo-10MB = $0.36 de reads + zero egress. Custo baixo mas **não zero**.
- Delivery via **Workers stream** consome CPU time do Worker (paid tier 30s / request).

**Melhor pattern**:
- **Estático frequentemente acessado**: R2 com Cache Rules HTTP → CDN serve o maior % → R2 só no cache miss.
- **Vídeo on-demand**: usar [[../mux/_moc|Mux]] ou [[_moc|Stream (lazy)]] (lazy — CF Stream) em vez de R2 + Worker.

## 4. Cold start pessimism em Workers

**Sintoma**: arquiteto ouviu "serverless tem cold start", presume 500ms+, evita Workers para UX crítica.

**Realidade**: V8 isolates cold start é **<50ms típico** — sub-10ms em isolates já "quentes" no PoP. Muito abaixo de Lambda/containers.

**Quando cold start importa**:
- Request com < 5 req/min no PoP específico — cada uma pode pagar 20-50ms.
- Mitigação: **paid tier** tem priority mais alta + "always warm" via scheduled ping (cron trigger chama `fetch` a cada 1 min — mantém isolate ativo).

**Quando não importa**: API que tem > 1 req/s agregado → nunca frio.

## 5. Durable Object hotspot

**Sintoma**: 1 DO para "global rate limiter" ou "global counter" — todas as requests do mundo convergem para 1 isolate.

**Realidade**: DO é **single-threaded por instance**. 100k req/s em 1 DO = throughput-killer.

**Mitigação**:
- **Shard by hash**: 32 DOs `rate-limiter-{0..31}`, request vai para `hash(userId) % 32`. 32× throughput.
- **Per-user DO**: `env.RATE_LIMITER.idFromName(userId)` — cada usuário tem seu DO; carga natural balanceada.
- **Usar KV ou D1 com UPSERT atômico** quando não precisa de coordenação por chave (preferir [[../../runtime-antipatterns/op-001-race-condition-contador|OP-001]] fix atomic-increment).

## 6. `wrangler dev` local como prod-like

**Sintoma**: testa em `wrangler dev` (sem `--remote`); bindings são simulados (D1 em SQLite local, KV em memória). Passa teste; deploy em prod; comportamento diferente (WAF bloqueia, Access rejeita, rate limit ativa).

**Realidade**: local mode é útil para velocidade; **não** valida comportamento da edge (WAF, Access, rate limit, Turnstile, cache real).

**Mitigação**:
- **`wrangler dev --remote`** para validar integração edge real.
- **Staging env** dedicado com route `staging.example.com/*` — testes E2E lá antes de prod.
- **CI roda `--env staging`** em toda PR.

## 7. API token "All Resources"

**Sintoma**: token com escopo amplo commitado em CI ou env de dev.

**Realidade**: token vaza em log, breach, ou é reusado malicioso — atacante tem acesso a conta inteira (todos os Workers, R2, D1, DNS, etc.).

**Mitigação**:
- **Token com scope mínimo**: `Workers Scripts:Edit`, `Account:Read`, + produtos específicos usados.
- **Token diferente por env**: `CLOUDFLARE_API_TOKEN_STAGING` e `_PROD`.
- **Rotação periódica** via painel; revogar imediato quando CI compromete.
- **Audit logs** via [MCP auditlogs.mcp.cloudflare.com](https://auditlogs.mcp.cloudflare.com/mcp) para ver uso anormal.

## 8. Observability desligada em prod

**Sintoma**: `wrangler.toml` sem `[observability]`; ou `enabled = false`. Em prod, não há logs investigáveis — só dashboard de alto-nível com contagem de req/error.

**Realidade**: Workers tier free tem observability básica; tier paid ($5/mo Worker) dá **Workers Logs** com 7 dias, query SQL-like. Vale o custo mesmo em app pequeno.

**Mitigação**:
- **Sempre `[observability] enabled = true`** em prod.
- **Sampling rate baixo** (0.1) em high-traffic para controlar custo.
- **Alertas** em `errors > threshold` via Cloudflare notifications ou export para SIEM.

## 9. Tratar Workers AI como "ChatGPT barato"

**Sintoma**: substituir OpenAI por Workers AI no backend sem avaliar: modelo menor, latência diferente, features ausentes.

**Realidade** (mesmo excluído do escopo atual, registro aqui para futura destilação):
- Workers AI: modelos menores (Llama 3.1 8B, Mistral, etc.) — **não** paridade com GPT-4.
- Latência na edge é ótima para inference leve; pesado foge do free tier rápido.
- AI Gateway faz **proxy** — você ainda paga OpenAI; ganho é observabilidade + cache.

**Mitigação**: avaliar caso real; benchmark lado-a-lado antes de trocar.

## Fontes

- [Cache API vs Cache Rules vs KV](https://developers.cloudflare.com/workers/reference/how-the-cache-works/) (consultada 2026-04-24)
- [R2 Pricing](https://developers.cloudflare.com/r2/pricing/) (consultada 2026-04-24)
- [Workers cold start](https://blog.cloudflare.com/eliminating-cold-starts-with-cloudflare-workers/) (consultada 2026-04-24)
- [Durable Objects best practices](https://developers.cloudflare.com/durable-objects/best-practices/) (consultada 2026-04-24)
- [API Token scopes](https://developers.cloudflare.com/fundamentals/api/reference/permissions/) (consultada 2026-04-24)
- [Workers Observability](https://developers.cloudflare.com/workers/observability/) (consultada 2026-04-24)

## Links

- [[_moc]]
- [[patterns]]
- [[overview]]
- [[../../runtime-antipatterns/op-001-race-condition-contador]]
- [[../../runtime-antipatterns/op-005-cache-sem-versionamento]]
