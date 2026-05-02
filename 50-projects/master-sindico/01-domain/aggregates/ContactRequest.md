---
title: ContactRequest (aggregate stub)
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
dispute: ConnectMeRequest — pode ser specialização ou kind separado
---

# ContactRequest

> 🟡 **Stub** criado em 2026-04-27 para cobrir refs em ui-catalog/jornadas/. Pode ser specialização de [[ConnectMeRequest]].

Solicitação de contato direto — empresa solicita ver currículo do morador (Banco de Talentos) ou morador solicita contato com empresa fora do fluxo Connect Me. LGPD: requer aceite explícito antes da revelação.

**BC**: commercial.

**Invariantes esperadas**:
- LGPD obrigatório: aceite [[LegalAcceptance]] kind=`contact_revelation` antes de revelar dados.
- Estado: `pending | accepted | rejected | expired`.
- Audit obrigatório.

## Links

- [[_moc]]
- [[ConnectMeRequest]]
- [[Curriculum]]
- [[LegalAcceptance]]
- [[../../08-security/lgpd]]
