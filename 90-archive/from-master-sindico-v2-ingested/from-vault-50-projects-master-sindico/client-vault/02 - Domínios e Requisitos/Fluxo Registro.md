---
title: "Fluxo Registro (canvas)"
type: canvas
tags: [canvas, obsidian, converted]
source: 50-projects/master-sindico/client-vault/02 - Domínios e Requisitos/Fluxo Registro.canvas
converted: 2026-04-22
---

# Fluxo Registro

> Canvas Obsidian convertido. Estrutura original: JSON com nodes + edges.

## Nodes

- **text**: Usuário / POST /v1/auth/register
- **text**: User Registry / Valida/Hash/Cria User
- **text**: Email Service / Envia verificação
- **text**: Usuário / Acessa link email
- **text**: User Registry / Ativa conta (Active)
- **text**: Onboarding / Fluxo por persona

## Edges

- U1 → R1 
- R1 → E1 
- E1 → U2 
- U2 → R2 
- R2 → O1 
