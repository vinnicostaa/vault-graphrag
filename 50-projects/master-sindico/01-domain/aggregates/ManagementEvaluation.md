---
title: ManagementEvaluation (aggregate stub)
type: aggregate
tags:
  - aggregate
  - master-sindico
  - stub
  - institutional
bc: institutional
status: stub
created: '2026-04-27'
updated: '2026-04-27'
---

# ManagementEvaluation

> 🟡 **Stub** criado em 2026-04-27 para cobrir refs em ui-catalog/jornadas/. Avaliação anônima da gestão pelo morador.

Avaliação anônima da gestão pelo morador (3 perguntas escala 1-10, ciclo bimestral). Síndico vê **só média + breakdown** — nunca individuais. Componente do `governance_score` (25% no fim do mandato).

**BC**: institutional.

**Invariantes esperadas**:
- Anonimato hard: `evaluator_id` é nunca exposto ao síndico (pseudonimização HMAC keyed — ADR-0028).
- Janela: 60-91 dias após disponibilização.
- Atraso 30+ dias bloqueia ações estratégicas do síndico.

## Links

- [[_moc]]
- [[Membership]]
- [[../../04-requirements/functional/governance]] §REQ-GOV-AVALIACAO-GESTAO
- [[../../02-architecture/adr/0028-lgpd-keyed-hmac]]
