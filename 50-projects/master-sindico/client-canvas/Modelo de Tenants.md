---
title: Modelo de Tenants (diagrama do cliente)
type: note
tags: [master-sindico, client-canvas, diagram, mermaid]
project: master-sindico
source: Clients/MasterSindico original .canvas (2026-03-10 a 2026-03-24) — convertido para MD+mermaid em 2026-04-24
created: 2026-04-24
---

# Modelo de Tenants

Diagrama original do cliente convertido de `.canvas` (Obsidian Canvas) para Mermaid. **Visão visual** dos fluxos/arquitetura; conteúdo canônico vive em [[../04-requirements/_moc]] + [[../02-architecture/_moc]].

## Diagrama

```mermaid
graph TD
  g_tc["📦 Tenant: Condomínio"]:::group
  S["👔 Síndico<br/>até 15 condos"]
  M["🏠 Morador<br/>titular + 2 dep."]
  ES["🏢 Empresa Síndica<br/>delegação parametrizável"]
  g_te["📦 Tenant: Empresa"]:::group
  EA["👤 Administrador"]
  EO["🔧 Operador Técnico"]
  g_ad["📦 Actor Delegado"]:::group
  AG["📸 Agência Marketing<br/>não é tenant"]
  g_ei["📦 Entidade Independente"]:::group
  PV["🏪 Parceiro Vizinhança<br/>login próprio · CPF"]
  S -->|gere| M
  ES -->|delegação| S
  EA -->|convida| AG
  S -->|Connect Me| EA
  M -->|Connect Me| S
  PV -->|promoções| M
  classDef group fill:#eef,stroke:#66a,stroke-width:2px,color:#000
```

## Nodes (11)

- **[GROUP]** `g_tc` — Tenant: Condomínio
- `S` — 👔 Síndico · até 15 condos
- `M` — 🏠 Morador · titular + 2 dep.
- `ES` — 🏢 Empresa Síndica · delegação parametrizável
- **[GROUP]** `g_te` — Tenant: Empresa
- `EA` — 👤 Administrador
- `EO` — 🔧 Operador Técnico
- **[GROUP]** `g_ad` — Actor Delegado
- `AG` — 📸 Agência Marketing · não é tenant
- **[GROUP]** `g_ei` — Entidade Independente
- `PV` — 🏪 Parceiro Vizinhança · login próprio · CPF

## Edges (6)

- `S` → `M` — _gere_
- `ES` → `S` — _delegação_
- `EA` → `AG` — _convida_
- `S` → `EA` — _Connect Me_
- `M` → `S` — _Connect Me_
- `PV` → `M` — _promoções_

## Links

- [[_moc]] — índice dos canvas do cliente
- [[../CLAUDE]] — contrato do projeto
- [[../02-architecture/_moc]]
- [[../04-requirements/_moc]]