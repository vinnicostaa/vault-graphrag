---
title: Source 03 — Dapper Large-Scale Distributed Systems Tracing Infrastructure
type: source
tags: [research, google, dapper, tracing, opentelemetry, observability]
source: https://research.google/pubs/dapper-a-large-scale-distributed-systems-tracing-infrastructure/
created: 2026-04-23
updated: 2026-04-23
aliases: [dapper-paper]
---

# Source 03 — Dapper: A Large-Scale Distributed Systems Tracing Infrastructure

- **URL primária**: https://research.google/pubs/dapper-a-large-scale-distributed-systems-tracing-infrastructure/
- **PDF canônico**: https://research.google.com/archive/papers/dapper-2010-1.pdf
- **Ano**: 2010 (após 2 anos de uso interno pré-publicação)
- **Autores**: Sigelman, Barroso, Burrows, Stephenson, Plakal, Beaver, Jaspan, Shanbhag
- **Data fetch**: 2026-04-23

## Trechos rastreáveis (WebFetch + WebSearch 2026-04-23)

- Design goals: **low overhead, application-level transparency, ubiquitous deployment**.
- "restrict the instrumentation to a rather small number of common libraries" — instrumentação em libs compartilhadas, não em cada app.
- Sampling: "rather than tracing every request, Dapper uses sampling to reduce overhead while maintaining visibility".
- Conceitos canônicos (traces, spans, sampling) introduzidos aqui, hoje standard.
- **Origem OpenTelemetry**: Ben Sigelman (co-autor Dapper) criou OpenTracing (2016) → mergiu com OpenCensus → **OpenTelemetry (2019)**, v1.0 em 2021 (confirmado: Tracetest blog e CNCF announcements).

## Aplicabilidade ao Master Síndico

- **Direta M1**: ALTA. OpenTelemetry é herdeiro legítimo de Dapper — já é stack-padrão. Reforçar princípios:
  1. **Sampling head-based 1-5%** em prod (nunca 100%).
  2. **Zero code-change nos features** via `otelgin`, `otelpgx`, `otelhttp`.
  3. **Spans atributos mínimos**: `tenant_id`, `bc`, `saga_id`; nunca PII.
  4. **Context propagation** via W3C TraceContext em HTTP e em headers de eventos.
- **Por quê crucial**: sem tracing distribuído, debug em microserviços (mesmo os 7 sub-produtos do Master) vira impossível. Dapper prova que low-overhead tracing é viável em prod de escala Google; qualquer dúvida sobre "vai performar?" está resolvida pela literatura.

## Links relacionados

- [[source-13]] Monarch — complementa (métricas) o que Dapper faz (traces).
- Destilado: [[_destilado#4.1 Dapper]].
