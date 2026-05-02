---
title: "Fluxo Governanca (canvas)"
type: canvas
tags: [canvas, obsidian, converted]
source: 50-projects/master-sindico/legacy-docs/MasterSindico/02 - Domínios e Requisitos/Fluxo Governanca.canvas
converted: 2026-04-22
---

# Fluxo Governanca

> Canvas Obsidian convertido. Estrutura original: JSON com nodes + edges.

## Nodes

- **text**: 👔 Síndico / Cria atividade da gestão
- **text**: 📜 Timeline / Recebe atividade
- **text**: 📋 Plano Diretor / Status atualizado auto
- **text**: 🏢 Empresa / Envia registro execução
- **text**: 👔 Síndico / Valida registro
- **text**: 📜 Timeline / Publica registro (imutável)
- **text**: 🏠 Morador / Avalia gestão
- **text**: 🏠 Morador / Visualiza Timeline

## Edges

- S1 → T1 
- T1 → PD1 
- E1 → S2 
- S2 → T2 
- T2 → M1 (modo leitura)
- S2 → M2 (avalia (3 perguntas))
