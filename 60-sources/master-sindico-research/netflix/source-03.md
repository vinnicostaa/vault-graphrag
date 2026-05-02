---
title: Netflix OSS deprecado (Hystrix, Eureka, Zuul, Ribbon) → service mesh
type: source
tags: [netflix, oss, hystrix, eureka, zuul, istio, service-mesh, resilience, research]
source: https://oneuptime.com/blog/post/2026-02-24-how-to-migrate-netflix-oss-stack-to-istio/view
created: 2026-04-23
updated: 2026-04-23
aliases: [netflix-oss-deprecation]
---

# Netflix OSS deprecado → service mesh (2026)

> Fontes: OneUptime (fev/2026), Medium "Eureka in 2025", Netflix Hystrix wiki — síntese destilada em 2026-04-23.

## Status de deprecação (confirmado)

| Componente | Função original | Status 2026 |
|---|---|---|
| **Hystrix** | Circuit breaker | Maintenance mode desde 2018. Netflix recomenda migrar. |
| **Eureka** | Service discovery | Netflix migrou silenciosamente 2016-2018 para tools internas. Spring Cloud parou de manter Netflix integrations em 2023. |
| **Ribbon** | Client-side load balancing | Substituído por infra (Envoy sidecar). |
| **Zuul** | API gateway | Zuul 2 existe mas raramente é primeira escolha; preferem Envoy/Istio Gateway. |
| **Turbine** | Metrics aggregation | Obsoleto — Prometheus/Grafana + OpenTelemetry. |

## Equivalências modernas

| Legado | Alternativa 2026 | Benefício |
|---|---|---|
| Eureka | Kubernetes DNS + Istio registry | Infra-level, sem lib no cliente |
| Ribbon | Envoy sidecar | Transparente no proxy |
| Hystrix | Istio `DestinationRule.outlierDetection` ou **resilience4j/gobreaker na aplicação** | Mais algoritmos, menos código |
| Zuul | Istio Gateway + VirtualService | Ingress unificado |
| Turbine | Prometheus + Grafana + OTel | Coleta automática sem lib |

## Insight-chave

Shift fundamental: **networking concerns saem da aplicação e vão pro mesh**. Serviços viram mais simples; circuit breaker, retry, timeout, mTLS, observability passam pro sidecar.

## Quando service mesh ainda é overkill

- < 10 serviços, 1 região, 1 cluster: Istio/Linkerd adicionam complexidade operacional que não compensa.
- Em monolito com módulos: circuit breaker é lib in-process (gobreaker, sony/gobreaker). Não precisa de mesh.

## Aplicabilidade ao Master Síndico

- M1-M2 (monolito Go + poucos workers): **NÃO adotar service mesh**. Usar `sony/gobreaker` pra calls externas (Stripe, Mux, Livekit, Zitadel) e timeout/retry via lib (hashicorp/go-retryablehttp).
- M3+ quando quebrar em micro: avaliar Linkerd (mais leve que Istio) ou direto Kubernetes Gateway API + Envoy, sem dependência Netflix OSS.
- **Não reimplementar Eureka** — Kubernetes Service já faz discovery.
- **Não copiar Hystrix com Thread pool** — gobreaker usa semáforo/state machine, leve e idiomático Go.

## Referência pra Go

- `github.com/sony/gobreaker` — circuit breaker idiomático
- `github.com/cenkalti/backoff` — exponential backoff
- `github.com/hashicorp/go-retryablehttp` — http client com retry

## Links

- [[source-04]] — resilience4j vs Hystrix
- [[source-01]] — evolução API
- [[_destilado]]
