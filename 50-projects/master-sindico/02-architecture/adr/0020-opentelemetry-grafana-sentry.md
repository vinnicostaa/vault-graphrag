---
title: ADR-0020 — OpenTelemetry + Grafana + Sentry
type: adr
tags: [adr, observability, otel, grafana, sentry, master-sindico, v2]
status: accepted
date: 2026-04-23
created: 2026-04-23
updated: 2026-04-23
---

# ADR-0020 — OpenTelemetry + Grafana + Sentry

## Status

accepted — 2026-04-23 (herdado Sprint 7 legado)

## Contexto

Observabilidade é pré-requisito para SRE / SLO / chaos engineering / debugging em produção. Stack fragmentada (Datadog para logs, Lightstep para traces, Sentry para errors) é cara e leak informação entre ferramentas.

## Decisão

**Stack canônica única: OpenTelemetry (SDK) + Grafana (dashboards + Loki/Tempo/Mimir) + Sentry (errors)**.

### Componentes

- **SDK**: OpenTelemetry Go (`otelgin`, `otelpgx`, `otelhttp`, `otelredis`).
- **Logs**: zerolog → stdout → Grafana Loki.
- **Metrics**: Prometheus exposition → Grafana (Mimir/Thanos em M3+).
- **Traces**: OTLP gRPC → Grafana Tempo.
- **Errors**: Sentry Go SDK + SolidJS + Flutter.
- **Dashboards + alerting**: Grafana.
- **Status page**: BetterStack (externa).

### Sampling

- Head-based 1% em prod.
- 100% dos 5xx.
- 100% com `X-Debug-Trace` header (dev/ops).

### Atributos canônicos (spans e logs)

- `request_id`, `tenant_id`, `user_id` (opaco), `bc`, `endpoint`, `method`, `status`, `saga_id` (se aplicável).

### PII

- **Zero PII** em spans, logs ou Sentry stacktraces. Linter CI bloqueia.
- Sentry scrub lista: CPF, CNPJ, email, telefone, token, password.

## Consequências

### Positivas

- Stack open-source em M1 (Grafana Cloud free tier + Sentry free tier).
- Portável: OTel é vendor-neutral; se migrar para Datadog/Honeycomb no futuro, SDK fica.
- Correlação trace ↔ log ↔ metric via `trace_id` nativo.

### Negativas

- Grafana Cloud free tier tem limites (50GB logs/mês) — upgrade em M2-M3.
- Sentry free tier também limitado; plan pago ~$29/mês adequado M1.

## Roadmap maturidade

| Fase | O que entra |
|---|---|
| M1 | SDK integrado, Prometheus single, Loki, Tempo, Sentry, dashboards base, burn rate alerts, postmortem template |
| M2 | Mimir/Thanos se volume, tail-based sampling avaliação, multi-region traces, DLQ alertas auto-ticketing |
| M3 | SLO 99.9% upgrade, on-call formal, chaos latency injection staging |

Ver [[../observability]] (canônico).

## Alternativas consideradas

1. **Datadog** — stack completa; $$$. Lock-in.
2. **New Relic** — similar Datadog; lock-in.
3. **ELK stack** — self-host pesado (Elasticsearch + Logstash + Kibana).
4. **Honeycomb** — ótimo para traces; $$; Grafana Tempo alcança 80% valor free.
5. **Sem traces (só logs+metrics)** — debug de saga/microservice impossível.

## Referências

- [[../observability]] (canônico)
- [[../../13-research/google-arch/_destilado]] §4 (Dapper → OTel)
- [[../../13-research/netflix/_destilado]] §6 (Atlas/Mantis)
- [[0025-sli-slo-error-budget]]
