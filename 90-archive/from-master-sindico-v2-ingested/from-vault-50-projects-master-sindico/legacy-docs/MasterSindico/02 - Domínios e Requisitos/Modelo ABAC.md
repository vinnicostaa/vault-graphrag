---
title: "Modelo ABAC (canvas)"
type: canvas
tags: [canvas, obsidian, converted]
source: 50-projects/master-sindico/legacy-docs/MasterSindico/02 - Domínios e Requisitos/Modelo ABAC.canvas
converted: 2026-04-22
---

# Modelo ABAC

> Canvas Obsidian convertido. Estrutura original: JSON com nodes + edges.

## Nodes

- **text**: Usuário / userId · role · plan
- **text**: Tenant / tenantId · type
- **text**: Ação / action · resource
- **text**: Contexto / quota · time
- **text**: Policy Engine / Oso
- **text**: Resultado / allowed · features

## Edges

- U → P 
- T → P 
- A → P 
- C → P 
- P → R 
