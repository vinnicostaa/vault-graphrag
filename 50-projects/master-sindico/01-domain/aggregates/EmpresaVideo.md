---
title: EmpresaVideo (aggregate stub)
type: aggregate
tags:
  - aggregate
  - master-sindico
  - stub
  - content
bc: content
status: stub
created: '2026-04-27'
updated: '2026-04-27'
dispute: Video — kind=empresa_institutional
---

# EmpresaVideo

> 🟡 **Stub** criado em 2026-04-27 para cobrir refs em ui-catalog/jornadas/. Pode ser specialização de [[Video]] (kind=`empresa_institutional`).

Vídeo institucional publicado por [[Company]] — apresentação, portfolio, depoimento. Storage via Mux (ADR-0010), trava 90d pós-publish (regra de negócio).

**BC**: content.

**Invariantes esperadas**:
- Owner: Company.
- Lock 90d pós-publish (não pode deletar/editar para evitar manipulação reputacional).
- Privacidade: `public | members-only | empresa-only`.
- Métricas: views, completion rate (privadas para owner).

## Links

- [[_moc]]
- [[Video]]
- [[Company]]
- [[../../02-architecture/adr/0010-mux-video-provider]]
