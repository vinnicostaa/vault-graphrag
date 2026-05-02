---
title: MOC — 01-domain/enums
type: moc
tags:
  - moc
  - master-sindico
  - domain
  - enum
created: '2026-04-27'
updated: '2026-04-27'
---

# MOC — 01-domain/enums

Enums de domínio do Master Síndico — listas canônicas de valores discretos referenciados em jornadas/aggregates. Cada enum vive como nota atômica para permitir destilação independente com cliente quando necessário.

## Enums catalogados

### Banco de Talentos (sub-produto 07)
- [[curriculum-areas]] — 18 áreas profissionais
- [[curriculum-modalidades]] — 9 modalidades de trabalho (CLT, PJ, autônomo, etc.)

### Commercial (Connect Me + Empresa)
- [[categorias-empresa]] — categorias de atuação de [[../aggregates/Company]]

### Vizinhança (sub-produto 08)
- [[promotion-types]] — tipos de promoção (desconto-percentual, brinde, etc.)
- [[vizinhanca-categorias-subcategorias]] — taxonomia comercial de bairro

### Institutional / governance
- [[areas-impactadas]] — áreas físicas/funcionais do condomínio (área comum, piscina, fachada, etc.)

### Risk & evaluation
- [[risk-categories]] — categorias de risco em avaliações de gestão

## Convenções

- **Listas preliminares**: stubs indicam valores prováveis; **lista canônica final exige validação com cliente**.
- **Backend**: enums viram `CHECK constraint` no Postgres ou `text` com lookup table conforme tamanho.
- **Frontend**: dropdown/multi-select via Kobalte; copy em pt-BR.

## Vizinhos

- [[../_moc]] — MOC 01-domain
- [[../aggregates/_moc]] — aggregates que consomem enums
- [[../ubiquitous-language]] — termos canônicos do domínio
