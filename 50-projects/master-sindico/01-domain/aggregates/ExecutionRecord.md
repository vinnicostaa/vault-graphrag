---
title: ExecutionRecord (aggregate stub)
type: aggregate
tags:
  - aggregate
  - master-sindico
  - stub
  - commercial
bc: commercial
status: stub
created: '2026-04-27'
updated: '2026-04-27'
dispute: ServiceRecord — verificar duplicação com aggregate existente
---

# ExecutionRecord

> 🟡 **Stub** criado em 2026-04-27 para cobrir refs em ui-catalog/jornadas/. Conteúdo a destilar junto com Connect Me ciclo completo.

Registro de **execução de serviço** após [[ClosedDeal]] — empresa envia evidência de execução; síndico (ou empresa síndica delegada) valida; gera [[TimelineEntry]] após validação. Aggregate equivalente conceitual a [[ServiceRecord]] (verificar duplicação na destilação).

**BC**: commercial (raiz) / institutional (timeline target).

**Invariantes esperadas**:
- Origem: ClosedDeal aprovado.
- Estados: state machine `submitted | under_review | validated | rejected | adjusted` (ver [[../state-machines#execution-record-states]]).
- Validação gera TimelineEntry imutável.
- Adjustments criam novo ExecutionRecord vinculado.

**Eventos**: `ExecutionRecordSubmitted`, `ExecutionRecordValidated`, `ExecutionRecordRejected`.

## Links

- [[_moc]]
- [[ClosedDeal]]
- [[ServiceRecord]]
- [[TimelineEntry]]
- [[../state-machines]]
