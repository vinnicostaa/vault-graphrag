---
title: RED, USE, SLI/SLO — métricas que importam
type: concept
tags: [observability, sli, slo, sre, metrics]
created: 2026-04-24
updated: 2026-04-24
aliases: [RED method, USE method, SLO, Error budget]
---

# RED, USE, SLI/SLO — métricas que importam

Observability sem foco gera dashboards que ninguém olha. RED, USE, Golden Signals e SLO/SLI são frameworks que respondem: **o que medir, quando alertar, quando agir**.

## RED Method (Tom Wilkie, Weaveworks)

Para cada serviço **request-driven** (HTTP handler, gRPC, consumer de fila), medir:

| Letra | Métrica | Pergunta |
|---|---|---|
| **R** | **Rate** — requests por segundo | Quanto tráfego está entrando? |
| **E** | **Errors** — taxa de requests que falham | Quantas falham? |
| **D** | **Duration** — latência (p50, p95, p99) | Quão rápido respondemos? |

Três instrumentos por endpoint/rota + label de status code resolvem 80% da detecção de incidentes request-bound. Cada serviço que recebe request: **mesmo trio**.

```
http_requests_total{route, method, status}              → Rate (deriv), Errors (filter 5xx)
http_request_duration_seconds{route, method}            → Duration (histograma)
```

## USE Method (Brendan Gregg)

Para cada **recurso compartilhado** (CPU, RAM, disco, rede, file descriptors, connection pool), medir:

| Letra | Métrica | Pergunta |
|---|---|---|
| **U** | **Utilization** — % do tempo ocupado | Recurso está sendo usado? |
| **S** | **Saturation** — trabalho enfileirado / stalled | Há fila? |
| **E** | **Errors** — erros do recurso | Está falhando? |

Diferente de RED, mira o **recurso**, não o request. CPU a 95% com fila de run-queue = saturação; CPU a 95% sem fila = ok. Saturação é o sinal antecipado de colapso.

## RED + USE são complementares

- **RED** responde: "como meu serviço está atendendo o cliente?"
- **USE** responde: "meu serviço tem recursos pra continuar atendendo?"

Incidente típico: latência RED sobe → investigar USE → encontra pool de conexão de DB saturado.

## 4 Golden Signals (Google SRE)

Do livro *Site Reliability Engineering* (Beyer et al., 2016), cap. 6:

1. **Latency** — tempo de resposta, separando sucesso de erro (erro 500 rápido é enganoso se medido junto)
2. **Traffic** — volume de demanda (req/s, msg/s, conexões concorrentes)
3. **Errors** — taxa de falhas explícitas (5xx, timeouts, exceções)
4. **Saturation** — quão "cheio" o serviço está (fila, memória, conexões livres)

Union de RED + USE com enfoque em **experiência do cliente**. É a recomendação SRE oficial do Google.

## SLI — Service Level Indicator

Métrica **quantitativa** específica da qualidade percebida:

```
SLI = boas_ocorrências / total_ocorrências
```

Exemplos:

- `% de requests com latência < 300ms`
- `% de pagamentos confirmados em < 5s`
- `% de uploads concluídos sem reintento`
- `availability: % de requests não-5xx`

### Como escolher SLI

1. **Customer-facing** antes de interno — o que o usuário sente, não o que a infra sente
2. **Específico por jornada** — `login`, `checkout`, `upload` têm SLIs próprios
3. **Razão entre eventos bons e válidos** — não medir coisas que o cliente não toca
4. **Observável** — ideal ser medido no edge (CDN/load balancer), não só no serviço

## SLO — Service Level Objective

Meta **interna** sobre o SLI ao longo de uma janela:

```
SLO: 99.9% dos requests /checkout respondem em < 400ms
     medido em janela deslizante de 28 dias
```

SLO vira **contrato interno** entre engenharia e produto. Define o que é "rápido o suficiente" sem overengineering de 99.999% desnecessário.

### Escolher o número certo

- **100% é errado** — impossível e caro
- **99.9%** (three nines) — 43min/mês de orçamento de erro
- **99.99%** (four nines) — 4min/mês
- **99.999%** (five nines) — 26s/mês — só pra infra crítica

Número vem do **impacto ao usuário**, não do desejo da engenharia. Verificar o que o usuário tolera (research, NPS, benchmark concorrência).

## Error Budget

```
error_budget = (1 - SLO) × janela
```

Ex: SLO 99.9% em 30 dias → orçamento de erro = 0.1% × 30d ≈ **43 minutos** de indisponibilidade tolerada.

### Error Budget Policy

Acordo do time sobre **o que fazer quando o orçamento acaba**:

- **Budget intacto** → liberdade para deploys arriscados, features novas, experimentos
- **Budget < 30%** → reduzir risco: canary mais longo, freeze de mudanças não-críticas
- **Budget esgotado** → **feature freeze**; time foca em confiabilidade até o orçamento voltar

Regra mais valiosa que o número em si: desloca a conversa "vamos fazer mais features?" para "temos orçamento?".

## Burn rate alerts

Alertar quando o orçamento está queimando rápido, **não** em cada erro isolado.

```
burn_rate = (erros na janela) / (orçamento alocado pra janela)
```

Padrão duplo recomendado pelo SRE Workbook:

- **Fast burn** — 14.4× em 1h (consome 2% do orçamento de 30d em 1h) → incidente urgente
- **Slow burn** — 6× em 6h (consome 10% do orçamento em 6h) → investigar no horário

Regra: alerta é **sobre queimar orçamento**, não sobre "alguma métrica subiu". Isso elimina pager fatigue.

## SLA vs SLO vs SLI

| Termo | Quem vê | Consequência |
|---|---|---|
| **SLI** | Engenharia | Métrica crua — "% pedidos em < 400ms" |
| **SLO** | Engenharia + produto | Meta interna — "99.9% em 28d" |
| **SLA** | Cliente externo | **Contrato legal** — "se < 99.5% no mês, crédito X%" |

SLO deve ser **mais rigoroso** que o SLA (folga para errar internamente sem violar cláusula). Ex: SLA 99%, SLO 99.9%.

## Antipatterns

- **Alertar em threshold fixo** (CPU > 80%) em vez de em burn rate → ruído
- **Dashboard de 50 métricas** sem hierarquia → ninguém olha
- **SLO copiado de outro time** sem considerar jornada do próprio usuário
- **Error budget** que ninguém respeita → regra de ouro: se o budget é desrespeitável, o SLO está errado ou a cultura não está pronta
- **Latency média** como SLI → p99 é o que o pior 1% sente

## Links

- [[_moc]]
- [[structured-logging]]
- [[../_moc]]
- [[../security/_moc]]
- [[../testing/_moc]]
