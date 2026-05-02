---
title: Distributed Tracing — OpenTelemetry e correlação ponta-a-ponta
type: concept
tags: [observability, tracing, opentelemetry, otel, w3c]
source: https://opentelemetry.io/docs/concepts/signals/traces/
source_date: 2026-04-24
created: 2026-04-24
updated: 2026-04-24
---

# Distributed Tracing — OpenTelemetry e correlação ponta-a-ponta

Em sistema com 3+ serviços, uma request cruza rede, fila, DB, cache, provider externo. *Traces* respondem **"por onde passou essa request e quanto tempo ficou em cada lugar?"** — pergunta que log e metric sozinhos não respondem.

## Anatomia de um trace

- **Trace** — operação completa de ponta-a-ponta, identificada por `trace_id` (128 bits)
- **Span** — unidade de trabalho dentro do trace (1 HTTP call, 1 query SQL, 1 publish em fila)
- **Parent/child spans** — spans formam árvore; um span filho ocorre dentro de um pai
- **Attributes** — pares chave-valor (`http.method=POST`, `db.statement=SELECT ...`, `user.id=42`)
- **Events** — marcos temporais dentro do span (ex: `cache.miss`, `retry.attempt`)
- **Status** — `ok` / `error` + mensagem
- **Links** — referência a spans em outro trace (útil em batch / fan-in)

## OpenTelemetry (OTel) — padrão-ouro

CNCF standard que unifica traces + metrics + logs. Cobre:

- **SDK** por linguagem (Go, Rust, TS, Python, Java, .NET, Ruby...)
- **API** — instrumentação que não depende do backend
- **Collector** — agente que recebe, processa, exporta pra 1+ backends
- **Semantic conventions** — chaves padronizadas (`http.*`, `db.*`, `messaging.*`, `rpc.*`)

Valor: **vendor lock-in zero**. Trocar Jaeger por Honeycomb por Datadog sem tocar código — só reconfigurar o Collector.

## W3C Trace Context — propagação entre serviços

Trace só funciona se o `trace_id` **atravessa** a rede. Padrão W3C define 2 headers HTTP:

```
traceparent: 00-<trace_id:32hex>-<span_id:16hex>-<flags:2hex>
             │   └ 0af7651916cd43dd8448eb211c80319c
             │                                   └ b7ad6b7169203331
             │                                                     └ 01 (sampled)
tracestate:  vendor1=value1,vendor2=value2
```

- **traceparent** — obrigatório, identifica a trace + span pai
- **tracestate** — opcional, carga vendor-specific
- **baggage** (header separado) — propaga chave-valor semântica (`user.id=42`, `tenant=acme`)

Middleware HTTP faz extract → start_span → inject antes de outgoing call. Em fila: headers em Kafka, atributos em SQS/SNS, user_properties em MQTT.

## Instrumentação — automática vs manual

- **Automática** — SDK instrumenta bibliotecas conhecidas (http client, SQL driver, Kafka client) sem código do dev. Começar por aqui.
- **Manual** — spans custom para operações de negócio que o SDK não vê (ex: "processar_pagamento", "calcular_desconto"). Nome claro + attributes específicos do domínio.

```go
// Go — manual span com attributes de negócio
ctx, span := tracer.Start(ctx, "checkout.authorize",
    trace.WithAttributes(
        attribute.String("order.id", orderID),
        attribute.String("user.id", userID),
        attribute.Float64("amount", amount),
    ))
defer span.End()

if err := svc.authorize(ctx, orderID); err != nil {
    span.RecordError(err)
    span.SetStatus(codes.Error, err.Error())
    return err
}
```

## Sampling — head-based vs tail-based

Trace tem custo (rede + storage). Sampling decide quais traces manter.

### Head-based sampling

Decisão **na origem** (1º span) antes de saber resultado. Propagada via bit de `traceparent`.

- Simples, barato
- Perde traces de erro raro (se não foi amostrado, sumiu)
- Estratégias: `always_on`, `always_off`, `parent_based`, `trace_id_ratio(0.1)`

### Tail-based sampling

Decisão **no fim do trace** (no Collector), depois de ver todos os spans.

- Mantém 100% de traces com erro ou lentos
- Sampling baixo (ex: 1%) pra traces ok
- Requer buffer em memória (5-30s) — custo maior no coletor
- Preserva anomalias — recomendação forte para produção

```yaml
# OTel Collector — exemplo tail_sampling processor
processors:
  tail_sampling:
    decision_wait: 10s
    policies:
      - name: errors
        type: status_code
        status_code: { status_codes: [ERROR] }
      - name: slow
        type: latency
        latency: { threshold_ms: 1000 }
      - name: healthy
        type: probabilistic
        probabilistic: { sampling_percentage: 1 }
```

## Backends comuns

| Backend | Modelo | Quando |
|---|---|---|
| **Jaeger** | OSS, self-hosted | Stack controlada; baixo volume |
| **Grafana Tempo** | OSS, object storage | Integra Grafana + Loki + Mimir (stack unificada) |
| **Honeycomb** | SaaS, high-cardinality analytics | Exploração interativa, BubbleUp em alta-card |
| **Datadog APM** | SaaS, full-platform | Times que já usam DD para metrics + logs |
| **AWS X-Ray** | Managed | Stack 100% AWS |
| **New Relic / Dynatrace** | SaaS enterprise | Empresas grandes com stack mista |
| **ClickHouse + otel** | OSS, custom | Alto volume, custo baixo, DIY |

Todos aceitam OTLP (protocolo OTel) — escolha não é sobre instrumentação, é sobre analytics/custo/retenção.

## Correlação log ↔ trace ↔ metric

O valor real do tracing surge quando os 3 pilares se **correlacionam**:

- **Log** sempre carrega `trace_id` + `span_id` → clicar num log salta pro trace
- **Trace** carrega span events + attributes com IDs de negócio → encontrar request pelo user_id
- **Metric exemplars** — Prometheus 2.26+ suporta anexar trace_id a amostra de metric → clicar num outlier de p99 salta pro trace daquele request

Ver [[structured-logging]] sobre incluir `trace_id` em logs; ver [[red-use-sli-slo]] sobre metrics + exemplars.

## Quando usar (e quando evitar)

**Use tracing quando:**

- Sistema tem 3+ serviços (mais de um hop de rede)
- Problema de latência "some no meio" entre serviços
- Times precisam ver dependências cross-service
- Async/event-driven — entender fluxo por eventos

**Pense duas vezes quando:**

- Monólito simples — logs estruturados + metrics resolvem 95% dos casos
- Volume imenso sem orçamento — tracing custa; considere head sampling agressivo + tail apenas em edge
- Bibliotecas sem SDK — custo de instrumentação manual pode não compensar

**Não substitua** logs e metrics — os 3 são complementares. Alertar em metric, diagnosticar em trace, confirmar em log.

## Antipatterns

- **Trace sem attributes de negócio** — só URLs e status codes; não dá pra filtrar por user/order
- **Span de 1 nível** (sem profundidade) — não mostra onde tempo foi
- **Cardinality explosion** em attributes — `user.id` como attribute em metric derivativa mata Prometheus
- **Sampling agressivo sem tail-based** — perde os erros raros que importavam
- **`trace_id` ausente em logs** — quebra correlação, trace vira silo
- **Instrumentar só HTTP e ignorar DB/queue** — 70% do tempo some em camada invisível

## Links

- [[_moc]]
- [[red-use-sli-slo]]
- [[structured-logging]]
- [[../_moc]]
- [[../architecture/_moc]]

## Fontes

- OpenTelemetry — https://opentelemetry.io/docs/concepts/signals/traces/
- W3C Trace Context — https://www.w3.org/TR/trace-context/
- Beyer, B. et al. (2016). *Site Reliability Engineering*. Google / O'Reilly, cap. 6.
- Majors, C. et al. (2022). *Observability Engineering*. O'Reilly.
- Sridharan, C. (2018). *Distributed Systems Observability*. O'Reilly.
