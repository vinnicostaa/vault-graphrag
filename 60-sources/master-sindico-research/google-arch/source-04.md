---
title: Source 04 — Google SRE Book Service Level Objectives SLI SLO SLA
type: source
tags: [research, google, sre, sli, slo, sla, error-budget, reliability]
source: https://sre.google/sre-book/service-level-objectives/
created: 2026-04-23
updated: 2026-04-23
aliases: [sre-slo-chapter]
---

# Source 04 — Google SRE Book: Service Level Objectives (Ch.4)

- **URL primária**: https://sre.google/sre-book/service-level-objectives/
- **Livro**: "Site Reliability Engineering" — O'Reilly, 2016. Disponível integral em sre.google.
- **Editores**: Beyer, Jones, Petoff, Murphy
- **Data fetch**: 2026-04-23

## Trechos rastreáveis (WebFetch 2026-04-23)

- **SLI**: "a carefully defined quantitative measure of some aspect of the level of service that is provided". Common examples: request latency, error rate, system throughput.
- **SLO**: "a target value or range of values for a service level that is measured by an SLI". Estrutura `SLI ≤ target` ou `lower ≤ SLI ≤ upper`.
- **SLA**: "an explicit or implicit contract with your users that includes consequences of meeting (or missing) the SLOs". Diferença-chave: SLA tem consequências (multa, créditos); SLO é interno.
- **Error budget**: "It's both unrealistic and undesirable to insist that SLOs will be met 100% of the time". Budget = `1 - SLO`, tracked "on a daily or weekly basis", usado como input pro processo de decisão sobre releases.
- SLI típico: razão `good events / total events`.

## Aplicabilidade ao Master Síndico

- **Direta M1**: ALTA. Adotar framework SLI/SLO antes de M1 encerrar.
- Decisão candidata (ADR + D-### em STATE):
  - **3 SLIs por BC** (latência, disponibilidade, correção de negócio).
  - **Target inicial 99.5%** (4-week rolling window).
  - **SLA formal apenas em M2+** após baseline de 3-6 meses.
- **Por quê**: sem SLI/SLO explícito, cada decisão de "melhor confiabilidade ou mais features?" vira discussão subjetiva. Com error budget vira decisão data-driven.

## Links relacionados

- [[source-09]] SRE Workbook — implementação prática dos SLOs.
- [[source-10]] Error Budget Policy — política formal.
- [[source-11]] Postmortem Culture.
- Destilado: [[_destilado#1.1 SLI / SLO / SLA]].
