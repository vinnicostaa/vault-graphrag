---
title: DPIA Template (LGPD Data Protection Impact Assessment)
type: template
tags:
  - template
  - master-sindico
  - lgpd
  - compliance
  - stub
status: stub
created: '2026-04-27'
updated: '2026-04-27'
---

# DPIA Template — Data Protection Impact Assessment

> 🟡 **Stub** criado em 2026-04-27. Template DPIA exigido pela LGPD (art. 38) para tratamentos de dados pessoais com risco alto. Conteúdo a expandir junto com responsável LGPD designado.

## Estrutura DPIA (LGPD)

1. **Identificação do tratamento** — finalidade, escopo, tipos de dado, volumes
2. **Bases legais** (art. 7º LGPD) — consentimento, contrato, obrigação legal, etc.
3. **Necessidade e proporcionalidade** — por que esses dados, por que esse tempo
4. **Riscos identificados** — vazamento, uso indevido, re-identificação, discriminação
5. **Medidas de mitigação** — técnicas (HMAC keyed, encryption-at-rest, audit) + organizacionais (treinamento, acessos)
6. **Risco residual** — score após mitigação
7. **Aprovação** — DPO + responsável legal + data
8. **Revisão** — ciclo (mín. anual) e gatilhos de re-DPIA (mudança de escopo, incidente)

## Links

- [[../08-security/lgpd]]
- [[../02-architecture/adr/0028-lgpd-keyed-hmac]]
- [[../02-architecture/adr/0037-soft-delete-universal-pseudonymize]]
- [[../04-requirements/functional/compliance]]
