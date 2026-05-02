---
title: MOC — Onboarding Mobile
type: moc
tags: [moc, master-sindico, mobile, flutter, onboarding]
project: master-sindico
stack: mobile
created: 2026-04-24
---

# MOC — Onboarding Mobile

Wizards mobile por persona. Ver diagrama completo em [[../../onboarding-flow]].

## Morador (ONB-M)

- [[ONB-M2]] — Buscar Condomínio (CEP + número)
- [[ONB-M3]] — Identificação Unidade
- [[ONB-M4]] — Termo de Ciência
- [[ONB-M5]] — Status da Unidade
- [[ONB-MFIN]] — Tudo Pronto

## Síndico (ONB-S)

- [[ONB-S2]] — Selecionar Condomínio
- [[ONB-S3]] — Dados Pessoais
- [[ONB-S4]] — Upload Ata de Posse
- [[ONB-S5]] — Selfie com Documento
- [[ONB-S6]] — Termo + Assinatura
- [[ONB-SFIN]] — Análise em Curso

## Empresa (ONB-E) — M2+

- [[ONB-E2]] — Dados Empresa
- [[ONB-E3]] — CNPJ + Razão Social
- [[ONB-E4]] — Endereço
- [[ONB-E5]] — Área de Atuação
- [[ONB-E6]] — Documentos
- [[ONB-E7]] — Banco de Talentos Opt-in

## Parceiro (ONB-P) — M2+

- [[ONB-P2]] — Tipo de Parceiro
- [[ONB-P3]] — Dados Básicos
- [[ONB-P4]] — Termo de Comissão

## Final (FIN)

- [[ONB-FIN]] — Placeholder final genérico pós-persona

## Princípios mobile

- Auto-save debounce 2s.
- Retomada via `hydrated_bloc` + `GET /onboarding/sessions/me`.
- Offline: edição livre; submit enfileirado.
- Touch targets ≥ 48dp, labels visíveis, teclado adequado (CPF = number, email = emailAddress).

## Links

- [[../../../../_moc]]
- [[../../requirements/onboarding]]
- [[../../onboarding-flow]]
