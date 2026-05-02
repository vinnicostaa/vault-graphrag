---
title: "Fluxo Ciclo Servico (canvas)"
type: canvas
tags: [canvas, obsidian, converted]
source: 50-projects/master-sindico/legacy-docs/MasterSindico/02 - Domínios e Requisitos/Fluxo Ciclo Servico.canvas
converted: 2026-04-22
---

# Fluxo Ciclo Servico

> Canvas Obsidian convertido. Estrutura original: JSON com nodes + edges.

## Nodes

- **text**: Connect Me / Requisição
- **text**: Proposta / Req 20
- **text**: Votação Fornecedor / Req 21
- **text**: Negócio Fechado / Req 22
- **text**: Registro Execução / Req 23
- **text**: Atividade Técnica / Req 23.1
- **text**: Comunicado Pós-Deal / Req 24
- **text**: Avaliação / Req 26
- **text**: Adendo / Req 25
- **text**: Timeline

## Edges

- CM → PROP 
- PROP → VOTE 
- VOTE → DEAL 
- DEAL → EXEC 
- DEAL → TECH 
- DEAL → COM 
- EXEC → TL (validação síndico)
- TECH → TL (validação síndico)
- COM → TL (validação síndico)
- TL → ADENDO (imutável)
- DEAL → EVAL (concluído)
