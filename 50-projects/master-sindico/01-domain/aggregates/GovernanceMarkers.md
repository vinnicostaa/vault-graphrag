---
title: GovernanceMarkers (aggregate stub)
type: aggregate
tags:
  - aggregate
  - master-sindico
  - stub
  - institutional
  - compliance
bc: institutional
status: stub
created: '2026-04-27'
updated: '2026-04-27'
dispute: VO de Condominium — entity filha
---

# GovernanceMarkers

> 🟡 **Stub** criado em 2026-04-27 para cobrir refs em ui-catalog/jornadas/. Entity filha de [[Condominium]] / vinculada à `governance_score` do mandato corrente.

Conjunto de **marcadores declaratórios** do condomínio — checkbox list de certificações, programas, selos: `membro_abracs`, `seguro_responsabilidade_civil`, `assessoria_juridica`, `assessoria_contabil`, `conformidade_lgpd`, `conformidade_nr1`, `programa_compliance`, `selo_reclame_aqui`, etc.

**BC**: institutional (com cross-ref para compliance).

**Invariantes esperadas**:
- Auto-declaração (não-verificada). Ver [[../../03-subprojects/web/patterns/self-declaration-disclaimer]].
- Histórico de mudanças preserva audit (LGPD).
- Componente do `governance_score` (D-103).

## Links

- [[_moc]]
- [[Condominium]]
- [[../../04-requirements/functional/institutional]] §Governance markers
- [[../../04-requirements/functional/compliance]]
