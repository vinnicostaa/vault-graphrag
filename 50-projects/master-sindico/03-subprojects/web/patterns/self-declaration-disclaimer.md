---
title: Pattern — Self-Declaration Disclaimer
type: pattern
tags:
  - pattern
  - ui
  - web
  - master-sindico
  - lgpd
  - legal
created: '2026-04-27'
updated: '2026-04-27'
---

# Pattern — Self-Declaration Disclaimer

Disclaimer + auto-declaração explícita quando o user declara um fato sob sua responsabilidade — currículo (Banco de Talentos), formação acadêmica, certificações, dados de empresa, declaração de não-conflito-de-interesses.

## Estrutura

- **Disclaimer fixo no topo do form** (texto curto): "Os dados abaixo são **auto-declarados** e **não são verificados pelo Master Síndico**. Empresas/condomínios contratantes podem solicitar comprovação."
- **Checkbox final** "Declaro sob responsabilidade que as informações são verdadeiras e estou ciente que dados falsos sujeitam-me a sanções legais (Lei 14.063 / Código Penal art. 299)."
- **Persist**: `LegalAcceptance` aggregate com `kind: 'self_declaration'`, `subject_id`, `terms_version`, `accepted_at`.
- **Visual em telas que consumem dado**: badge "auto-declarado" ao lado do dado para informar leitor.

## Quando usar

- Currículo Banco de Talentos.
- Cadastro de Empresa (CNPJ, áreas de atuação, certificações sem upload).
- Resposta de Connect Me com escopo declarado.

## Quando evitar

- Quando há documento comprobatório obrigatório — usa upload + verificação manual ou OCR.
- Quando falsidade tem consequência financeira direta (ex.: status de inadimplência) — exige verificação automatizada.

## Cross-refs

- [[legal-terms-checkbox]] — base do mecanismo
- [[forms]] — host
- [[../../../08-security/lgpd]]
- [[../../../01-domain/aggregates/Curriculum]] *(quando criado)*

## Links

- [[_moc]]
