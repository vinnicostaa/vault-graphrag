---
title: Source 11 — SRE Book Postmortem Culture Learning from Failure
type: source
tags: [research, google, sre, postmortem, blameless, culture, incident]
source: https://sre.google/sre-book/postmortem-culture/
created: 2026-04-23
updated: 2026-04-23
aliases: [sre-postmortem-culture]
---

# Source 11 — SRE Book: Postmortem Culture — Learning from Failure (Ch.15)

- **URL primária**: https://sre.google/sre-book/postmortem-culture/
- **Livro**: "Site Reliability Engineering" — O'Reilly, 2016.
- **Data fetch**: 2026-04-23

## Trechos rastreáveis (WebFetch 2026-04-23)

### Filosofia core
- "The cost of failure is education".
- Um postmortem "must focus on identifying contributing causes without indicting individuals or teams for misconduct".

### Triggers objetivos pra postmortem
- "User-visible downtime or degradation beyond a certain threshold".
- "Data loss of any kind".
- On-call engineer intervention (rollbacks, traffic rerouting).
- Resolution time excedendo thresholds estabelecidos.
- Monitoring failures que exigiram descoberta manual.

### Blameless principles
- "Everyone involved in an incident had good intentions and did the right thing with the information they had".
- "You cannot fix people, but you can improve systems and processes".

### Google's practices
- **Estrutura**: docs colaborativos em tempo real, review por senior engineer, compartilhamento amplo na org.
- **Cultural reinforcement**: newsletters mensais com postmortems exemplares, reading clubs, role-playing ("Wheel of Misfortune"), recognition por leadership (peer bonuses).
- **Evolution**: trend analysis + ML pra prever weaknesses.

### On-call mention
- "At least eight people need to be part of the on-call team to avoid fatigue and allow sustainable staffing, preferably in two well-separated geographic locations to avoid nighttime pages".

## Aplicabilidade ao Master Síndico

- **Direta M1**: ALTA (mesmo solo).
  - Template em `11-audit/postmortems/YYYY-MM-DD-<slug>.md`.
  - Triggers: downtime > 5 min; qualquer perda de dado; rollback manual; >1h pra resolver; falha de monitoring.
  - **Blameless praticado solo**: mesmo sem outros pra culpar, escrever "o processo/sistema permitiu isso" e não "eu fui descuidado" — muda a remediação.
  - Linkar todo postmortem a A-### em AUDIT com remediação concreta.
- **On-call formal**: PENDENTE até M3 com 3+ devs (não faz sentido hoje).
- **Wheel of Misfortune / chaos**: viável mesmo solo no M2 — simular falha em staging, executar runbook.

## Links relacionados

- [[source-04]] SLO (define quando triggers objetivos atuam).
- [[source-10]] Error Budget Policy (postmortem obrigatório em >20% consumido).
- Destilado: [[_destilado#1.3 Postmortem]], [[_destilado#1.5 On-call]].
