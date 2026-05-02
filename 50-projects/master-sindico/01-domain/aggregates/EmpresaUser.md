---
title: EmpresaUser (aggregate stub)
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
dispute: >-
  Membership — provavelmente Membership com kind=empresa_user; não aggregate
  root
---

# EmpresaUser

> 🟡 **Stub** criado em 2026-04-27 para cobrir refs em ui-catalog/jornadas/. User vinculado a [[Company]] com role específica.

[[User]] vinculado a [[Company]] com role específica (admin, operador, comercial). Pode ser implementado via [[Membership]] com `kind=empresa_user` em vez de aggregate dedicado — verificar na destilação.

**BC**: commercial.

## Links

- [[_moc]]
- [[User]]
- [[Company]]
- [[Membership]]
