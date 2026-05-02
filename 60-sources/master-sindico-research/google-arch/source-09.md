---
title: Source 09 — SRE Workbook Implementing SLOs
type: source
tags: [research, google, sre, slo, workbook, error-budget, burn-rate]
source: https://sre.google/workbook/implementing-slos/
created: 2026-04-23
updated: 2026-04-23
aliases: [sre-workbook-slo]
---

# Source 09 — SRE Workbook: Implementing SLOs

- **URL primária**: https://sre.google/workbook/implementing-slos/
- **Livro**: "The Site Reliability Workbook" — O'Reilly, 2018.
- **Editores**: Beyer, Murphy, Rensin, Kawahara, Thorne
- **Data fetch**: 2026-04-23

## Trechos rastreáveis (WebFetch 2026-04-23)

### Escolher SLIs
- "treat SLIs as the ratio of two numbers: the number of good events divided by the total number of events".
- Tipos: availability, latency (requests completing within threshold / total), freshness, correctness, durability, quality, coverage.
- Para primeiros SLIs: "choose something measurable with minimal engineering overhead".

### Setting SLO targets
- **Evitar 100%**: sistemas têm múltiplos pontos de falha; usuários experienciam falhas em cascata; updates exigem algum downtime.
- "Round down measured performance to manageable numbers"— ex: 97.1% medido → SLO 97%.

### Time windows
- **Rolling windows** (recomendado 4 semanas): "align with user experience; use integral weeks to control for weekday/weekend variance".
- Calendar windows melhores pra planejamento business.
- Windows curtos = decisões táticas; longos = estratégicas.

### Stakeholder buy-in
Aprovação exige 3 partes:
1. Product managers (threshold garante user satisfaction).
2. Dev teams (reliability não trava velocity).
3. SREs (objetivos defensáveis sem toil excessivo).

### Error Budget Policies
- Documentar ações quando budget exaure: "prioritizing reliability bugs, pausing feature releases, declaring production freezes".
- "Clear escalation paths prevent disputes over whether policy enforcement is warranted".

## Aplicabilidade ao Master Síndico

- **Direta M1**: adotar o framework conforme source-04. Este source adiciona o **como** implementar:
  - Rolling window **4 semanas**.
  - Round-down do baseline observado (se API atinge 99.6% naturalmente, SLO = 99.5%).
  - Alertar em **burn rate multi-window multi-burn-rate** (2% budget em 1h → page; 5% em 6h → ticket) — padrão do workbook.
- **Decisão**: pular o "stakeholder buy-in" formal (solo-dev), mas documentar mesmo SLOs candidatos em ADR pra forçar o exercício de análise.

## Links relacionados

- [[source-04]] SRE book SLO chapter (fundamentos).
- [[source-10]] Error Budget Policy.
- Destilado: [[_destilado#1.1 SLI]], [[_destilado#1.2 Error budget]].
