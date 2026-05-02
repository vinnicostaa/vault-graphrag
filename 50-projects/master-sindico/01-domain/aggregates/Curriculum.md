---
title: Curriculum (aggregate stub)
type: aggregate
tags:
  - aggregate
  - master-sindico
  - stub
  - banco-talentos
bc: commercial
status: stub
created: '2026-04-27'
updated: '2026-04-27'
---

# Curriculum

> 🟡 **Stub** criado em 2026-04-27 para cobrir refs em ui-catalog/jornadas/. Conteúdo a destilar quando Banco de Talentos (sub-produto 07) entrar em sprint dedicado.

Aggregate root do **Banco de Talentos** (sub-produto 07). Currículo do morador para vinculação com ofertas de Connect Me — auto-declarado (ver [[../../03-subprojects/web/patterns/self-declaration-disclaimer]]) com badges "auto-declarado".

**BC**: commercial.

**Invariantes esperadas** (a destilar):
- Owner: Membership do morador.
- Imutabilidade de versões publicadas; edições geram nova versão.
- Áreas/modalidades vêm de enums (`curriculum-areas`, `curriculum-modalidades`).
- Visibilidade controlada por flags (público / só empresas Pro / privado).

**Eventos esperados**: `CurriculumPublished`, `CurriculumUpdated`, `CurriculumViewedByCompany`.

## Links

- [[_moc]]
- [[../../00-product/sub-produtos/07-banco-de-talentos]]
- [[../../03-subprojects/web/ui-catalog/jornadas/curriculo-cadastro]]
- [[../../03-subprojects/web/ui-catalog/jornadas/curriculo-empresa-view]]
- [[../../STATE]]
