---
title: "Modelo Planos (canvas)"
type: canvas
tags: [canvas, obsidian, converted]
source: 50-projects/master-sindico/client-vault/02 - Domínios e Requisitos/Modelo Planos.canvas
converted: 2026-04-22
---

# Modelo Planos

> Canvas Obsidian convertido. Estrutura original: JSON com nodes + edges.

## Nodes

- **text**: Base / Gratuito
- **text**: Pro / Mensal/Anual
- **text**: Trial / 15d/7d/30d
- **text**: Plus / Mensal/Anual

## Edges

- T → B (expira sem pagamento)
- T → PLUS (paga)
- T → PRO (paga)
- B → PLUS (upgrade)
- B → PRO (upgrade)
- PLUS → PRO (upgrade)
- PRO → PLUS (downgrade)
- PLUS → B (downgrade)
- PRO → B (expira)
- PLUS → B (expira)
