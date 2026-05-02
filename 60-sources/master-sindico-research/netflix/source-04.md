---
title: Resilience4j vs Hystrix — patterns de resiliência modernos
type: source
tags: [resilience, circuit-breaker, bulkhead, retry, timeout, resilience4j, hystrix, research]
source: https://resilience4j.readme.io/docs/comparison-to-netflix-hystrix
created: 2026-04-23
updated: 2026-04-23
aliases: [resilience4j-comparison]
---

# Resilience4j vs Hystrix: patterns de resiliência

> Fonte: docs resilience4j + comparação Exoscale + Baeldung — síntese destilada em 2026-04-23.

## Design: o que mudou

| Aspecto | Hystrix | Resilience4j |
|---|---|---|
| Paradigma | OO — `HystrixCommand` wrapper | Funcional — decorators componíveis sobre lambda/function |
| Concorrência | Thread pool (padrão) → CPU/mem alto | Semáforo (padrão) → leve |
| Anotação | `@HystrixCommand` + properties | Fluent API + decorators |
| Half-open | Single execution determina fechamento | **N execuções configuráveis + threshold** |
| Reactive | Limitado | Operators nativos Reactor/RxJava |

## 5 patterns canônicos (aplicáveis a qualquer stack, inclusive Go)

1. **Circuit Breaker** — 3 estados (CLOSED/OPEN/HALF_OPEN). Abre após taxa de erro > threshold; rejeita calls por período; testa em half-open.
2. **Bulkhead** — isola recursos por pool/semáforo. Evita que call lenta a serviço X esgote threads globais.
3. **Retry** — reexecuta com backoff exponencial + jitter. Só aplicar em operações idempotentes.
4. **Timeout (TimeLimiter)** — corta chamadas que ultrapassam deadline. Inegociável em call externa.
5. **RateLimiter** — limita RPS pra proteger provider (ex: Stripe tem rate limit próprio).

## Ordem de composição (importante)

`Retry ( CircuitBreaker ( Bulkhead ( TimeLimiter ( call ) ) ) )`

- Timeout mais interno (abortar call travada primeiro).
- Bulkhead isola recurso.
- Circuit breaker decide se sequer tenta.
- Retry por último (sobre tudo, só se CB permite).

## Anti-padrões documentados

- Retry sem backoff → amplifica carga em provider degradado.
- Circuit breaker com threshold muito baixo → false positives; com muito alto → inútil.
- Timeout igual ao SLA do provider → sem margem pra TCP handshake/retry local.
- Fallback defensivo que esconde bug (retornar cache stale sem métrica).

## Aplicabilidade ao Master Síndico

Providers críticos e padrões mínimos:

| Provider | Circuit Breaker | Retry | Timeout | Observação |
|---|---|---|---|---|
| **Stripe** | Sim, global por endpoint | Sim, idempotency-key obrigatório | 10s | Webhooks: verify signature, retry do Stripe cobre |
| **Mux** | Sim, separado upload vs playback | Sim em upload | 30s upload, 5s metadata | Webhook → idempotente via asset_id |
| **Livekit** | Sim, por room | Não (não-idempotente) | 5s | Falha = sala não criada, user retenta |
| **Zitadel** | Sim, auth é crítico | Sim, token refresh | 3s | Cache local de tokens válidos |

## Libs Go recomendadas

- `github.com/sony/gobreaker` — CB idiomático, state machine clara.
- `github.com/avast/retry-go/v4` — retry com backoff + jitter.
- `context.WithTimeout` — timeout nativo, usar em toda chamada externa.
- Bulkhead: `golang.org/x/sync/semaphore` ou worker pool próprio.

## Links

- [[source-03]] — Netflix OSS deprecação
- [[source-05]] — chaos engineering pra validar patterns
- [[_destilado]]
