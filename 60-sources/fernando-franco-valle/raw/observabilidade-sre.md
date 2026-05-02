[FFV Academy](../index.html)/Observabilidade & SRE

🔭

Blog

# Observabilidade & SRE

Para quem já sabe o básico e quer ir fundo. Aqui o assunto é como os modelos funcionam em produção: memória, roteamento, ferramentas, agentes. O lado técnico que pouca gente explica direito.

8artigos

635XP total

📄PDF da trilha

🖥️Apresentar trilha

[](../aprenda/observability-pilares/index.html)

01

## 🔍 Observability: os 3 pilares (logs, métricas, traces) e por que não basta

Os 3 pilares clássicos, por que "observability" \> monitoring, cardinalidade, events como 4º pilar, profiles (pprof, Pyroscope) como 5º, o que realmente medir.

⏱ 15 min·+75 XP

→[](../aprenda/metricas-red-use/index.html)

02

## 📉 Métricas RED e USE: os frameworks que cobrem 90% dos casos

RED (Rate, Errors, Duration) para serviços, USE (Utilization, Saturation, Errors) para recursos, Golden Signals do Google SRE, quando aplicar cada um.

⏱ 14 min·+70 XP

→[](../aprenda/opentelemetry-stack/index.html)

03

## 🛰️ OpenTelemetry end-to-end: instrumentação app → backend

OpenTelemetry SDK (auto vs manual), Collector (receivers, processors, exporters), context propagation, resource detection, pipelines reais em Node, Python e Go.

⏱ 18 min·+90 XP

→[](../aprenda/logs-estruturados/index.html)

04

## 📝 Logs Estruturados: JSON, correlation IDs e levels com propósito

JSON logs, trace_id/span_id correlation, log levels (o que DEBUG, INFO, WARN, ERROR realmente significam), structured logging libs, formatter cost.

⏱ 14 min·+70 XP

→[](../aprenda/distributed-tracing/index.html)

05

## 🧵 Distributed Tracing: spans, baggage e sampling strategies

Spans, parent-child, context propagation (W3C Trace Context), baggage, head vs tail sampling, probabilistic vs rule-based, Jaeger e Tempo comparados.

⏱ 16 min·+80 XP

→[](../aprenda/slos-error-budgets/index.html)

06

## 🎯 SLOs e Error Budgets: a contabilidade da confiabilidade

SLI → SLO → SLA, error budget, burn rate alerts multi-window/multi-burn, toil budget, política de freeze quando orçamento estoura.

⏱ 16 min·+80 XP

→[](../aprenda/incident-response-postmortem/index.html)

07

## 🚑 Incident Response: comando, comunicação e postmortem blameless

Incident Commander, Comms Lead, Ops Lead, timeline, postmortem blameless, 5 whys, action items com SLA, learning review mensal.

⏱ 16 min·+80 XP

→[](../aprenda/capstone-sre-slo-runbook/index.html)

08

## 🏁 Capstone: SLO + error budget + runbook reais

Projeto: definir SLIs (latency, availability) pra serviço real, calcular SLO (99.9%?), error budget mensal, alertas multi-burn-rate (fast/slow), runbook com ações por alerta, gameday simulado pra validar.

⏱ 20 min·+90 XP

→

[← Voltar à home](../index.html)
