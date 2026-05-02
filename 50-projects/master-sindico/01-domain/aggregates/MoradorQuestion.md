---
title: MoradorQuestion (aggregate stub)
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

# MoradorQuestion

> 🟡 **Stub** criado em 2026-04-27 para cobrir refs em ui-catalog/jornadas/. Pergunta não-vinculante do síndico aos moradores — distinta de [[Poll]] (que pode ter vinculação).

Pergunta feita pelo síndico aos moradores — coleta opinião, **não-vinculante**. Resultado consolidado vai para [[TimelineEntry]] após encerramento.

**BC**: institutional.

**Invariantes esperadas**:
- Não-vinculante (não substitui assembleia).
- Janela temporal definida (`opens_at`, `closes_at`).
- Resposta anônima ou nominal conforme configuração.
- Resultado agregado publicado em Timeline.

**Eventos**: `QuestionPublished`, `QuestionAnswered`, `QuestionClosed`.

## Links

- [[_moc]]
- [[Poll]]
- [[TimelineEntry]]
- [[../../04-requirements/functional/governance]] §REQ-GOV-PERGUNTA-MORADOR
