---
title: "Modelo de Tenants (canvas)"
type: canvas
tags: [canvas, obsidian, converted]
source: 50-projects/master-sindico/client-vault/00 - Visão Geral/Modelo de Tenants.canvas
converted: 2026-04-22
---

# Modelo de Tenants

> Canvas Obsidian convertido. Estrutura original: JSON com nodes + edges.

## Nodes

- **group**: Tenant: Condomínio
- **text**: 👔 Síndico / até 15 condos
- **text**: 🏠 Morador / titular + 2 dep.
- **text**: 🏢 Empresa Síndica / delegação parametrizável
- **group**: Tenant: Empresa
- **text**: 👤 Administrador
- **text**: 🔧 Operador Técnico
- **group**: Actor Delegado
- **text**: 📸 Agência Marketing / não é tenant
- **group**: Entidade Independente
- **text**: 🏪 Parceiro Vizinhança / login próprio · CPF

## Edges

- S → M (gere)
- ES → S (delegação)
- EA → AG (convida)
- S → EA (Connect Me)
- M → S (Connect Me)
- PV → M (promoções)
