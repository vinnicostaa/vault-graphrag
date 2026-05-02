---
title: Promotion (aggregate stub)
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

# Promotion

> 🟡 **Stub** criado em 2026-04-27 para cobrir refs em ui-catalog/jornadas/. Conteúdo a destilar quando Sistema de Cupons (D-064/D-078) entrar em sprint.

Aggregate root do sub-produto **Vizinhança** (08). Promoção criada por [[Partner]] (parceiro local) e ativada por moradores. Pode ser **aberta** (qualquer condomínio) ou **exclusiva** (escopo definido pelo síndico).

**BC**: commercial.

**Invariantes esperadas**:
- Owner: Partner (parceiro de bairro).
- Escopo: `aberta | exclusiva` — exclusiva exige `condominium_ids[]`.
- Validade temporal (`starts_at`, `ends_at`); auto-encerra após `ends_at`.
- Quota máxima de ativações (cupom limitado).
- Métricas: views, activations, redemptions.

**Eventos**: `PromotionPublished`, `PromotionActivated`, `PromotionRedeemed`, `PromotionExpired`.

## Links

- [[_moc]]
- [[Partner]]
- [[PromotionActivation]]
- [[../../00-product/sub-produtos/08-vizinhanca]]
- [[../../03-subprojects/web/ui-catalog/jornadas/parceiro-vizinhanca]]
