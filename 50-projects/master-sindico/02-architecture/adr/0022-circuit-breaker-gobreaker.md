---
title: ADR-0022 — Circuit breaker sony/gobreaker em providers externos
type: adr
tags: [adr, resilience, circuit-breaker, gobreaker, master-sindico, v2]
status: accepted
date: 2026-04-23
created: 2026-04-23
updated: 2026-04-23
---

# ADR-0022 — Circuit breaker sony/gobreaker em providers externos

## Status

accepted — 2026-04-23

## Contexto

Toda call a provider externo (Stripe, Mux, LiveKit, Zitadel, Twilio, SES, FCM, OpenAI Moderation, Rekognition) pode falhar, ficar lenta ou cair. Sem circuit breaker: cascading failure + thundering herd + latência ruim. Netflix Hystrix foi referência ([[../../13-research/netflix/_destilado]] §2) — moderno: `sony/gobreaker`.

## Decisão

**`github.com/sony/gobreaker`** envolvendo **toda** call a provider externo. Um CB por provider (não por endpoint — explode cardinalidade).

### Stack resilience completa

Composição canônica:

```
Retry( CircuitBreaker( Bulkhead( TimeLimiter( call_provider() ) ) ) )
```

- **CircuitBreaker**: `sony/gobreaker` — abre em erros consecutivos.
- **Retry**: `avast/retry-go/v4` — 3× exp backoff + jitter; só para erros transientes.
- **Bulkhead**: `golang.org/x/sync/semaphore` — limite concurrent.
- **TimeLimiter**: `context.WithTimeout` nativo.

### Configuração por provider

Herdado [[../../13-research/netflix/_destilado]] §2.

| Provider | Timeout | Retry | CB abre | Bulkhead concurrent |
|---|---|---|---|---|
| **Stripe write** | 10s | 3× exp 200ms jitter (com idempotency-key) | >50% erro / 20 req / 30s | 20 |
| **Stripe read** | 5s | 3× exp | idem | 50 |
| **Mux upload** | 30s | 3× exp (idempotency-key) | >30% erro / 30s | 10 |
| **Mux metadata** | 5s | 2× | idem | 50 |
| **LiveKit room create** | 5s | **sem retry** (não idempotente) | >50% erro / 30s | 5 |
| **Zitadel token refresh** | 3s | 3× rápido | >30% erro / 30s | 50 (cache token local) |
| **Zitadel introspect** | 2s | 2× | idem | 100 |
| **Twilio SMS** | 10s | 2× exp | >30% erro / 30s | 20 |
| **SES/Resend** | 10s | 3× exp | >30% erro / 30s | 30 |
| **FCM** | 5s | 2× | >30% erro / 30s | 50 |
| **OpenAI Moderation** | 10s | 2× | >30% erro / 30s | 20 |
| **AWS Rekognition** | 15s | 2× | >30% erro / 30s | 10 |

### Fallback strategy

| Provider | Falha |
|---|---|
| **Stripe** | erro claro ao user; registra intent; processa quando voltar. Nunca "mock success". |
| **Mux upload** | queue para retry; UI mostra progress + retry auto. |
| **LiveKit** | user retenta; log alerta se falhas > threshold. |
| **Zitadel introspect** | cache local se token recém-validado (<30s); caso contrário deny (fail-closed). |
| **Twilio/SES/FCM** | outbox retry + DLQ; usuário recebe notificação por canal alternativo (email se SMS falhou). |
| **OpenAI / Rekognition** | moderação vai para fila humana (degrade graceful). |

**Regra-ouro**: fail-closed em auth/billing; fail-open em features cosméticas.

## Consequências

### Positivas

- Cascading failure prevenido.
- Latência previsível (circuit aberto retorna imediato).
- Debug trivial: métrica CB state visível em Grafana.

### Negativas

- Configuração inicial é chute educado; precisa medir e ajustar.
- CB mal configurado pode fechar prematuramente (over-sensitive) ou tarde demais.
- Gobreaker é lib simples — sem features avançadas (half-open probe customizável etc).

## Métricas emitidas

- `provider_circuit_state{provider}` — closed / half-open / open.
- `provider_requests_total{provider, outcome}` — success / failure / open.
- `provider_duration_seconds{provider}` histograma.

Dashboard Grafana "Providers" centraliza — ver [[../observability]] §7.

## Alternativas consideradas

1. **Hystrix-go** — deprecated.
2. **go-resiliency** — lib alt; similar sony/gobreaker em capacidade.
3. **Istio/service mesh** — resolve mesh-level em M2+ mas M1 monolito não precisa.
4. **Sem CB** — cascading failure quando provider cai.

## Referências

- [[../../13-research/netflix/_destilado]] §2 (composição canônica + tabela config)
- [[../patterns]] §12 (Circuit Breaker), §13 (Retry + DLQ), §10 (ACL)
- [[../observability]] §7 (dashboard providers)
- https://github.com/sony/gobreaker
