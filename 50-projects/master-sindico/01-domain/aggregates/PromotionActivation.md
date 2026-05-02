---
title: PromotionActivation (aggregate stub)
type: aggregate
tags:
  - aggregate
  - master-sindico
  - stub
  - vizinhanca
bc: commercial
status: stub
created: '2026-04-27'
updated: '2026-04-27'
---

# PromotionActivation

> 🟡 **Stub** criado em 2026-04-27 para cobrir refs em ui-catalog/jornadas/. Entity filha de [[Promotion]].

Ativação de [[Promotion]] por morador — gera código/cupom único; rastreia uso no Partner; registra redenção quando aplicável.

**BC**: commercial.

**Invariantes esperadas**:
- Owner: Membership do morador ativador.
- 1 ativação por morador por Promotion (UNIQUE).
- Código único gerado server-side.
- Estado: `activated | redeemed | expired`.

**Eventos**: `PromotionActivated`, `PromotionRedeemed`.

## Links

- [[_moc]]
- [[Promotion]]
- [[Partner]]
- [[Membership]]
