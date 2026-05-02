---
title: EmpresaProfile (aggregate stub)
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
dispute: Company — perfil público pode ser merged como VO de Company
---

# EmpresaProfile

> 🟡 **Stub** criado em 2026-04-27 para cobrir refs em ui-catalog/jornadas/. Pode ser merged em [[Company]] na destilação completa.

Perfil público da [[Company]] visível para síndicos e moradores — dados de apresentação, áreas de atuação, certificações auto-declaradas, vídeos institucionais, score de reputação Bronze→Diamante.

**BC**: commercial.

**Relação com [[Company]]**: distinção pendente — `Company` carrega CNPJ/dados-cadastrais; `EmpresaProfile` carrega apresentação + reputação. Pode virar VO ou entity de Company conforme destilação.

**Invariantes esperadas**:
- Tier de reputação calculado (D-051 fórmula determinística).
- Áreas de atuação: enum `categorias-empresa`.
- Certificações auto-declaradas (ver [[../../03-subprojects/web/patterns/self-declaration-disclaimer]]).
- Vídeos institucionais via Mux (ver [[Video]]).

## Links

- [[_moc]]
- [[Company]]
- [[CompanyEvaluation]]
- [[../../03-subprojects/web/ui-catalog/jornadas/empresa]]
