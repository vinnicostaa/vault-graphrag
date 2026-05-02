---
title: MOC — Observability
type: moc
tags: [moc, observability]
created: 2026-04-24
updated: 2026-04-24
---

# Observability

Os 3 pilares — **logs, metrics, traces** — e as disciplinas que os tornam úteis: RED/USE/Golden Signals para saber o que medir, SLI/SLO/error budget para saber quando agir, structured logging para virar dado consultável, OpenTelemetry como padrão de coleta, audit trail para compliance.

## Notas

- [[red-use-sli-slo]] — RED (Rate/Errors/Duration), USE (Utilization/Saturation/Errors), 4 Golden Signals do Google SRE, SLI/SLO/SLA, error budget policy, burn rate alerts
- [[structured-logging]] — JSON lines vs unstructured, níveis, context propagation (trace_id/tenant_id), PII safety, sampling, logs vs metrics vs traces, retenção
- [[distributed-tracing]] — OpenTelemetry como padrão, W3C Trace Context, head vs tail sampling, backends (Jaeger, Tempo, Honeycomb, Datadog), correlação log↔trace↔metric
- [[audit-trail]] — log imutável para compliance (LGPD 5y, GDPR 6y, SOX 7y), append-only + hash chain, WORM storage, outbox pattern, campos obrigatórios
- [[alerting]] — SLO-based burn rate, fast burn vs slow burn, multi-window alerting, severity tiers, runbooks, pager fatigue, Prometheus Alertmanager + PagerDuty

## A criar

- opentelemetry-basics — spec W3C, traces/metrics/logs unificados, SDK vs Collector
- incident-response — severity, post-mortem blameless, correção de processo
- metrics-cardinality — explosão de labels, custo em Prometheus, estratégias de agregação

## Vizinhos

- [[../_moc]]
- [[../security/_moc]] — audit trail, logs sem PII, detecção de anomalia
- [[../testing/_moc]] — synthetic monitoring, smoke em prod
- [[../architecture/_moc]] — observability é preocupação arquitetural (não add-on)
- [[../../30-agents/mcp-integration]] — MCPs podem expor métricas de uso
