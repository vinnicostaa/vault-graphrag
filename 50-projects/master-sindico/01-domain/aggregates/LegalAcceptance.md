---
title: LegalAcceptance (aggregate stub)
type: aggregate
tags:
  - aggregate
  - master-sindico
  - stub
  - lgpd
  - compliance
bc: cross-domain
status: stub
created: '2026-04-27'
updated: '2026-04-27'
---

# LegalAcceptance

> 🟡 **Stub** criado em 2026-04-27 para cobrir refs em ui-catalog/jornadas/. Aggregate transversal a múltiplos BCs — registra todo aceite legal explícito.

Aggregate cross-domain — registra **todo aceite legal explícito** dado por user (LGPD, termo de uso, aceite de mandato, declaração de responsabilidade técnica, auto-declaração de currículo).

**BC**: cross-domain (consumido por identity, commercial, institutional).

**Invariantes esperadas**:
- Append-only (cada aceite é immutable).
- Carrega `terms_version` (versão do termo aceito) — termo nunca é "reaproveitado", só re-aceito.
- Audit obrigatório: `accepted_at`, `ip`, `user_agent`, `terms_version`, `subject_id`.
- Tipos: `lgpd`, `tos`, `mandate`, `tech_responsibility`, `self_declaration`.

**Eventos**: `LegalTermAccepted`, `LegalTermSuperseded`.

## Links

- [[_moc]]
- [[../../03-subprojects/web/patterns/legal-terms-checkbox]]
- [[../../08-security/lgpd]]
- [[../../04-requirements/functional/compliance]]
