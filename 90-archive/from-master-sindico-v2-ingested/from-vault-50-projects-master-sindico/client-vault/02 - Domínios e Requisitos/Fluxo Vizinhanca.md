---
title: "Fluxo Vizinhanca (canvas)"
type: canvas
tags: [canvas, obsidian, converted]
source: 50-projects/master-sindico/client-vault/02 - Domínios e Requisitos/Fluxo Vizinhanca.canvas
converted: 2026-04-22
---

# Fluxo Vizinhanca

> Canvas Obsidian convertido. Estrutura original: JSON com nodes + edges.

## Nodes

- **text**: 🏪 Parceiro / Cria promoção
- **text**: 🏘️ Vizinhança / Define escopo
- **text**: 🏠 Morador (Aberta) / Vê no feed
- **text**: 🏠 Morador (Exclusiva) / Vê se do condomínio
- **text**: 🏠 Morador / Ativa promoção
- **text**: 🏪 Parceiro / Atualiza métricas
- **text**: 👔 Síndico / Indica/Cria benefício

## Edges

- PV1 → VZ1 
- VZ1 → M1 
- VZ1 → M2 
- M2 → M3 
- M1 → M3 
- M3 → PV2 
- PV2 → S1 
