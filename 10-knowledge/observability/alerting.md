---
title: Alerting — SLO-based, burn rate, multi-window, sem fatigue
type: concept
tags: [observability, alerting, slo, sre, pagerduty]
source: https://sre.google/workbook/alerting-on-slos/
source_date: 2026-04-24
created: 2026-04-24
updated: 2026-04-24
---

# Alerting — SLO-based, burn rate, multi-window, sem fatigue

Alerta bom **desperta humano quando ação humana é necessária**. Alerta ruim desperta por ruído, erode confiança, leva a pager fatigue. A disciplina moderna (SRE Workbook) centra alertas em **error budget** e **burn rate**, não em thresholds estáticos de infra.

## Princípios

1. **Page on symptoms, not causes** — alertar no que o usuário sente (SLO violado), não em CPU alta
2. **Every page is actionable** — se não requer ação agora, não é page (é ticket/email)
3. **Every alert has a runbook** — link direto para procedimento, não "boa sorte"
4. **Precision e recall balanceados** — minimizar falso-positivo sem perder eventos reais
5. **Severity mapeada a impacto de usuário** — não a origem técnica

## SLO-based alerting — a mudança de paradigma

**Abordagem antiga (ruim):**

```
alert: HighErrorRate
expr: rate(http_errors[5m]) > 10
```

Problemas: threshold arbitrário (por que 10?), não relaciona a impacto, dispara em flutuação normal, não dispara em degradação lenta que consome SLO.

**Abordagem SLO-based:**

Alerta quando o **error budget** está queimando rápido demais pra durar a janela do SLO.

```
SLO: 99.9% em 30d → erro aceitável = 0.1% = 43 min/mês de indisponibilidade
burn_rate = (taxa_atual_de_erros) / (taxa_orçamentada_de_erros)
```

- `burn_rate = 1` → queima exatamente no ritmo do orçamento (ok)
- `burn_rate = 14.4` → queima 14× mais rápido — se não parar, orçamento de 30d some em ~2h

Ver [[red-use-sli-slo]] para fundamentos de SLI/SLO/error budget.

## Fast burn vs slow burn — duplo alerta

Um só alerta não detecta ambos: incidentes agudos curtos **e** degradação crônica lenta. SRE Workbook recomenda **dois alertas por SLO**:

| Alerta | Burn rate | Janela curta | Janela longa | % orçamento 30d consumido se dura |
|---|---|---|---|---|
| **Fast burn (page)** | 14.4× | 5 min | 1 h | 2% em 1h |
| **Slow burn (ticket)** | 6× | 30 min | 6 h | 10% em 6h |

Alerta dispara **somente se ambas as janelas** passam o threshold — elimina falso-positivo de pico de 1 minuto.

```yaml
# Prometheus Alertmanager — exemplo multi-window, multi-burn
- alert: SLOFastBurnCheckout
  expr: |
    (
      error_rate_5m{service="checkout"}  > (14.4 * 0.001)   # 0.001 = 1 - 99.9%
      and
      error_rate_1h{service="checkout"}  > (14.4 * 0.001)
    )
  for: 2m
  labels:
    severity: page
  annotations:
    summary: "Checkout consuming error budget at 14.4× rate"
    runbook: "https://runbooks/checkout-slo-burn"
    slo: "checkout-availability-99.9"

- alert: SLOSlowBurnCheckout
  expr: |
    (
      error_rate_30m{service="checkout"} > (6 * 0.001)
      and
      error_rate_6h{service="checkout"}  > (6 * 0.001)
    )
  for: 15m
  labels:
    severity: ticket
  annotations:
    runbook: "https://runbooks/checkout-slo-burn-slow"
```

Combinado: **fast burn** pega incidente agudo em ~5min; **slow burn** pega degradação lenta sem acordar on-call de madrugada.

## Severity tiers — mapeado a impacto

| Tier | Canal | SLA de resposta | Gatilho |
|---|---|---|---|
| **P1 / page** | PagerDuty → SMS/call | Acordar on-call 24×7, ack em 5 min | Fast burn SLO crítico, down total, data loss iminente |
| **P2 / ticket** | PagerDuty low-urgency / Jira | Próximo dia útil | Slow burn, degradação parcial, SLO secundário |
| **P3 / email** | email / Slack notify | Best-effort | Tendência que pode virar P2 se ignorada (disk 70% crescendo) |
| **P4 / dashboard** | Apenas visível | — | Observação passiva |

Regra: **page só para o que interrompe usuário agora ou em ≤ 1h**. Tudo mais é ticket. Cada page deve ser justificável em post-mortem.

## Alert fatigue — causas e mitigações

Pager fatigue = on-call perde precisão/moral porque recebe muito alerta ruim. Custo altíssimo (burnout, desligamento, descredibilização).

### Causas comuns

- Thresholds estáticos em métrica de infra (CPU > 80%) que não mapeiam a impacto
- Alerta que dispara em flutuação normal (5xx isolado em retry health check)
- Duplicação: 10 alertas diferentes disparando pelo mesmo root cause
- "Informativo" enviado a pager em vez de email/dashboard
- Sem ack + auto-resolve — alerta ecoa mesmo quando já resolvido
- Runbook ausente — on-call não sabe o que fazer

### Mitigações

- **SLO-based** em vez de threshold — elimina "CPU alta sem impacto"
- **Multi-window multi-burn-rate** — reduz falso-positivo
- **Inhibition rules** (Alertmanager) — alerta raiz silencia dependentes (`service_down` silencia `high_latency`)
- **Grouping / routing** — agrupar por serviço/time pra não floodar
- **On-call rotation justa** — max 2 pages/noite como sinal de saúde; se exceder, **reduzir alertas** não "aguentar"
- **Post-mortem em toda page** — a page era justificada? O runbook serviu? Ajustar thresholds / deletar alerta se não
- **Review trimestral** do catálogo de alertas — deletar os que nunca disparam ou sempre são ignorados

## Runbook — contrato de cada alerta

Alerta sem runbook é trap pra on-call. Runbook deve conter:

```markdown
# Runbook: SLOFastBurnCheckout

## O que significa
Serviço checkout está consumindo error budget em 14.4× o ritmo orçado.
Se persistir 2h, orçamento de 30d ~ esgota.

## Impacto no usuário
Checkout falha para N% dos compradores. Perda de receita ~$X/min.

## Diagnóstico rápido (5min)
1. Dashboard [checkout-slo](https://...) — qual rota/status code domina?
2. Traces recentes com erro — https://honeycomb/...
3. Logs 5xx últimos 15min — https://logs/...
4. Mudança recente (deploys, config) — https://gitops/...

## Mitigação
- Rollback do último deploy (se correlaciona): `kubectl rollout undo deploy/checkout`
- Circuit breaker manual em dependência lenta: toggle `feature.checkout.retry=false`
- Escalonar para on-call de platform se origem é infra

## Escalation
Se não resolvido em 15min → escalar para líder de checkout + SRE senior.

## Post-mortem
Template: [blameless-postmortem.md](...)
```

Link do runbook **no próprio alerta** (`annotation.runbook`). On-call nunca deveria procurar em Wiki no meio do incidente.

## Ferramentas

| Ferramenta | Papel | Quando |
|---|---|---|
| **Prometheus Alertmanager** | Roteamento, grouping, inhibition, silence | Stack Prometheus-native |
| **Grafana Alerting** | Unificado com dashboards, multi-datasource | Quem já usa Grafana como frontend |
| **PagerDuty** | Rotation, escalation, notification multi-canal | Padrão corporativo; integra com tudo |
| **OpsGenie (Atlassian)** | Similar PagerDuty | Stacks Atlassian |
| **VictorOps / Splunk On-Call** | Alternativa | — |
| **Datadog Monitors** | Alerting integrado ao backend de observability | Stack 100% Datadog |
| **incident.io** | Incident management + post-mortem workflow | Times maduros em SRE |

Arquitetura comum: métricas em Prometheus → regras de alert → **Alertmanager** dedupe/route → **PagerDuty** wake-up → incident.io pra coordenar.

## Antipatterns

- **Alertar em CPU/RAM/disk** sem mapear a user impact — "CPU 90% ok se latência p99 está ok"
- **Threshold estático** arbitrário em métrica de negócio — "orders/min < 50" em 3am do domingo
- **Um alerta por serviço down** e mais 20 dependentes disparando junto — sem inhibition rules
- **Page para warning** — "disk at 70%" não é P1
- **Sem auto-resolve** — alerta ecoa mesmo após recuperação
- **Runbook wiki com link quebrado** — "TODO: write runbook" há 2 anos
- **Alertar em flap** — métrica oscila entre 79% e 81%; alerta dispara/resolve em loop — falta histerese ou `for: Xm`
- **Alerta sem dono** — ninguém sabe quem mantém; ninguém reage → deletar
- **Teste em prod** — desativar alertas pra fazer deploy e esquecer de reativar

## Links

- [[_moc]]
- [[red-use-sli-slo]]
- [[structured-logging]]
- [[../_moc]]

## Fontes

- Google SRE Workbook — [Alerting on SLOs](https://sre.google/workbook/alerting-on-slos/)
- Beyer, B. et al. (2016). *Site Reliability Engineering*. Google / O'Reilly, cap. 6.
- Beyer, B. et al. (2018). *The Site Reliability Workbook*. Google / O'Reilly, cap. 5.
- Prometheus — [Alerting rules](https://prometheus.io/docs/prometheus/latest/configuration/alerting_rules/)
- Alertmanager — https://prometheus.io/docs/alerting/latest/alertmanager/
- Majors, C. et al. (2022). *Observability Engineering*. O'Reilly.
