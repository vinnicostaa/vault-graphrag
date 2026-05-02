---
title: Profile (aggregate stub)
type: aggregate
tags:
  - aggregate
  - master-sindico
  - stub
  - identity
bc: identity
status: stub
created: '2026-04-27'
updated: '2026-04-27'
dispute: User — pode virar VO de User (separation identidade vs apresentação)
---

# Profile

> 🟡 **Stub** criado em 2026-04-27 para cobrir refs em ui-catalog/jornadas/. Pode ser merged em [[User]] na destilação completa.

Perfil público de [[User]] — nome de exibição, avatar, bio, preferências de privacidade. Distinção vs `User` está em discussão (User carrega identidade canônica CPF/CNPJ; Profile é apresentação).

**BC**: identity.

**Invariantes esperadas**:
- 1:1 com User.
- Privacidade: tier (`public | members-only | private`).
- Avatar via [[../../02-architecture/adr/0010-mux-video-provider|Mux]] thumbnails ou storage simples.

## Links

- [[_moc]]
- [[User]]
- [[../../03-subprojects/web/ui-catalog/jornadas/onboarding]]
