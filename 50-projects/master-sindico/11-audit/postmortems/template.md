---
title: Template — Postmortem Blameless
type: template
tags: [template, postmortem, blameless, master-sindico, v2]
source: 06-execution/incident.md + SRE industry practices
created: 2026-04-23
updated: 2026-04-23
---

# Template — Postmortem Blameless

Copiar este arquivo pra `YYYY-MM-DD-<slug>.md` (ex: `2026-06-05-stripe-webhook-outage.md`) após qualquer incident P0/P1 (ou P2 com aprendizado relevante).

**Regra de ouro**: blameless. Foco em sistema + processo. Ninguém é demitido por incident bem-investigado.

---

```markdown
---
title: Postmortem — <título>
type: audit
tags: [postmortem, incident, <sev>, <bc-afetado>, master-sindico, v2]
severity: P0 | P1 | P2
date: YYYY-MM-DD
duration: "HH:MM (Xmin total)"
users_affected: <count + %>
feature_blocked: <feature name>
author: <quem escreveu>
reviewers: <quem revisou>
status: draft | final
---

# Postmortem — <título>

## TL;DR

<2-3 linhas: o quê, quando, quanto durou, quantos afetados, o quê mitigou>

Exemplo:
> Em 2026-06-05 entre 14:32 e 15:01 UTC (29min), Stripe webhooks pararam de ser processados por erro em validação HMAC após rotação de secret. 340 pagamentos (8% do dia) ficaram stuck em `pending`. Rollback do deploy restaurou serviço. Reprocessing completo em T+2h.

---

## Timeline

| Tempo | Evento |
|---|---|
| **T-Xd** | <contexto pré-incident; mudanças recentes> |
| **T+0** (HH:MM UTC) | <detecção> |
| T+Xmin | <acknowledge> |
| T+Ymin | <triage> |
| T+Zmin | <decisão de mitigação> |
| T+Wmin | <mitigação executada> |
| T+Vmin | <verificação> |
| T+Rmin | **RESOLVED** |
| T+postmortem | Postmortem draft |

---

## Impacto

### Usuários afetados

- **Count**: N usuários
- **% MAU**: X%
- **Severidade**: <bloqueio total / degradação / cosmético>

### Features bloqueadas

- <feature 1>
- <feature 2>

### SLO impact

- Error budget consumido: X% (anual)
- Uptime impact: 99.5% → <Y%> no período
- MTTR observado vs alvo: Xmin vs 30min

### Financeiro

- Transações afetadas: N (Stripe)
- Valor em risco: R$ X
- Refund se aplicável: sim/não
- Perda de receita estimada (indisponibilidade): R$ Y

### LGPD (se aplicável)

- Dados pessoais expostos? <sim/não>
- Titulares afetados: <count>
- ANPD notificada? <data + hora>
- Titulares notificados? <data + hora>

---

## Root cause (5 whys)

### Por que isso aconteceu?

1. **Por que**: <superficial> — <resposta>
2. **Por que**: <menos superficial> — <resposta>
3. **Por que**: <processo> — <resposta>
4. **Por que**: <sistema> — <resposta>
5. **Por que** (root): <causa raiz> — <resposta>

### Por que passou pelos gates?

- CI passou porque...
- Review passou porque...
- Staging 48h passou porque...
- Não foi detectado pelos alertas porque...

### Qual sistema/processo falhou?

<explanation; sistêmico, não pessoal>

---

## Lições aprendidas

### O que foi bem

- <item>
- <item>

### O que foi ruim

- <item>
- <item>

### Mudanças necessárias

**No sistema**:
- <alerta novo>
- <teste novo>
- <circuit breaker>
- <rate limit>

**No processo**:
- <runbook atualizado>
- <checklist pré-deploy>
- <dual-check em X tipo de decisão>

**Nas docs**:
- <documentar Y>
- <ADR nova>

---

## Ações de prevenção (tracking T-###)

Cada ação vira task no backlog:

| ID | Título | Dono | Sprint | Status |
|---|---|---|---|---|
| T-### | <ação 1> | <quem> | <sprint-N> | ⏳ |
| T-### | <ação 2> | <quem> | <sprint-N> | ⏳ |

---

## Comms

### Interno

- Canal #incident: <link>
- Stand-up da sprint seguinte: apresentar postmortem

### Externo

- Status page: <status>
- Email a clientes afetados: <enviado em YYYY-MM-DD HH:MM>
- Redes sociais (se público): <não / sim + link>

### LGPD (se aplicável)

- ANPD notification: <enviada em>
- Titulares: <notificados em, canal>

---

## Metadata

- **Detecção**: <source — Sentry / PagerDuty / cliente / monitor>
- **TTA** (Time to Acknowledge): Xmin
- **TTM** (Time to Mitigate): Ymin
- **TTR** (Time to Resolution): Zmin
- **Comparison com SLA TTR**: <bateu ou extrapolou>
- **Ferramentas usadas na investigação**: <Grafana / Sentry / pgBadger / pprof / etc.>
- **Dev(s) envolvido(s)**: <nomes (não pra culpar; pra aprender juntos)>
- **Stakeholders notificados**: <PO / DPO / cliente / etc.>

---

## Links

- [[_moc]]
- Incident thread: <link canal>
- [[../AUDIT]] (se gerou A-### novo)
- [[../../05-tasks/backlog]] (se T-### foi criado)
- [[../../06-execution/incident]]
- [[../../06-execution/rollback]]
- Runbook relevante: [[../../12-docs/runbooks/<name>]]
```

---

## Notas de uso

- Nome do arquivo: `YYYY-MM-DD-<slug>.md`.
- Status evolui: `draft` (48h pós-resolução) → `final` (após review).
- Circular no time: postar no #announcement quando `final`.
- Arquivar permanentemente (imutável pós-final — como ata).

## Links

- [[_moc]]
- [[../AUDIT]]
- [[../../06-execution/incident]]
