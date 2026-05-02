---
title: Netflix Observability — Atlas (metrics) + Mantis (stream processing)
type: source
tags: [observability, atlas, mantis, time-series, metrics, anomaly-detection, research]
source: https://netflix.github.io/atlas-docs/overview/
created: 2026-04-23
updated: 2026-04-23
aliases: [netflix-atlas-mantis]
---

# Netflix Observability: Atlas + Mantis

> Fontes: Atlas docs, Mantis docs, Netflix TechBlog "Lessons from Building Observability Tools" (2023), InfoQ — destilado 2026-04-23.

## Atlas — time-series metrics platform

### Escala

- **2011**: 2M métricas → triggered reescrita (Epic saturated).
- **2014**: 1,2B métricas.
- **Atual**: bilhões, cresce continuamente.

### Arquitetura chave

- **Dimensional time-series** — métricas carregam tags (país, device, region, cluster) em vez de nomes colados.
- **In-memory storage** (heap Java) — latência < 1s mesmo agregando 900M datapoints pra resultado de 180.
- **Query language stack-based** — URL-friendly, deep-link share, stack de operações (query, aggregate, consolidate, visualize).
- **Retention em camadas**:
  - < 6h: tudo em memória.
  - < 4d: rollup (tira dimensão "node").
  - < 16d: rollup completo só critical metrics.
  - > 16d: S3 + Hadoop, on-demand.

### Lições aplicáveis

1. **Tagged metrics** (dimensional) > nomes hierárquicos — Prometheus, OpenTelemetry já adotam.
2. **Rollup automático** — agregar e descartar dimensões conforme dado envelhece; custo de storage controlado.
3. **Query linguagem simples** com URL sharing → favorece cultura de dashboards compartilhados.
4. **Fan-out regional** — queries globais quebradas em sub-queries por região; escala linear.

## Mantis — stream processing + observability

### O que é

Plataforma de **stream processing microservices**. Usada pra **anomaly detection, alerting, realtime dashboards**, operational insights. Trilhões de eventos/dia desde 2014.

### Use cases canônicos

- **Log streaming com filtro SQL-like** — só logs que matcham query ficam em memória.
- **Anomaly detection** — estatística sliding window sobre event stream.
- **Métricas derivadas** — jobs Mantis publicam métricas pro Atlas.

## Stack completo Netflix (como se encaixa)

```
Apps → Atlas Publish Client (metrics) → Atlas
     → Logs → Mantis (stream) → Atlas (derived metrics)
                              → Alerts
Apps → Traces → internal distributed tracing (Edgar)
```

## Ferramentas equivalentes open source 2026

| Netflix tool | Equivalente moderno |
|---|---|
| Atlas | **Prometheus + VictoriaMetrics** ou **Grafana Mimir** |
| Mantis | **Apache Flink** ou **Materialize** ou **Benthos/Bento** |
| Edgar (tracing) | **Grafana Tempo** ou **Jaeger** ou **Honeycomb** |
| Lumen (dashboards) | **Grafana** |

## Aplicabilidade ao Master Síndico

Stack canônica já definida no vault: **OpenTelemetry + Grafana stack + Sentry**.

### M1 (MVP)

- **OpenTelemetry** SDK no Go app — metrics + traces + logs unificados.
- **Grafana Cloud free tier** (10k series, 50GB logs/mês) ou self-host Grafana + Prometheus + Loki + Tempo.
- **Sentry** pra error tracking (já decidido).
- RED/USE metrics por endpoint (Rate, Errors, Duration / Utilization, Saturation, Errors).

### M2-M3

- Dashboards por bounded context (Governança, Conteúdo, Connect Me).
- SLOs definidos (ex: 99,9% availability API, p95 < 300ms, erro < 0,1%).
- Alerting baseado em SLO burn rate (não em threshold cru).
- Log aggregation Loki com tenant_id em toda linha.

### Lições Netflix aplicáveis (sem overkill)

- [ ] Dimensional tagging (tenant_id, route, status_code) em toda métrica.
- [ ] URL-shareable dashboards (Grafana já faz).
- [ ] Rollup pra controlar custo de storage (Prometheus recording rules).
- [ ] Correlacionar logs ↔ traces ↔ métricas via trace_id (OTel já faz).

### NÃO adotar

- Atlas/Mantis diretamente (não open-source + overkill).
- Custom observability platform — Grafana stack é suficiente por décadas.

## Links

- [[source-05]] — chaos depende de observability maduro
- [[source-09]] — CD com canary depende de metrics
- [[_destilado]]
