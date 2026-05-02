---
title: AgenciaLink (aggregate stub)
type: aggregate
tags:
  - aggregate
  - master-sindico
  - stub
  - commercial
  - agencia-marketing
bc: commercial
status: stub
created: '2026-04-27'
updated: '2026-04-27'
---

# AgenciaLink

> 🟡 **Stub** criado em 2026-04-27 para cobrir refs em ui-catalog/jornadas/. Conteúdo a destilar quando Agência de Marketing (sub-produto 10, D-073) entrar em sprint.

Vínculo de **delegação** entre [[Company]] e Agência de Marketing — empresa autoriza agência a publicar/responder Connect Me em seu nome com **escopo parametrizável** (revogável a qualquer momento).

**BC**: commercial.

**Invariantes esperadas**:
- Owner: Company (delegante).
- Permissões granulares (publicar vídeos, responder Connect Me, gerir perfil).
- Audit trail de toda ação da agência em nome da empresa.
- Revogação imediata + audit log.

**Eventos**: `AgenciaLinked`, `AgenciaPermissionUpdated`, `AgenciaRevoked`.

## Links

- [[_moc]]
- [[Company]]
- [[../../00-product/sub-produtos/10-marketing-agencias]]
- [[../../03-subprojects/web/ui-catalog/jornadas/agencia-marketing]]
- [[../../STATE]] §D-073
