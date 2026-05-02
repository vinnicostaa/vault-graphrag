---
title: Fluxo Ciclo Servico (diagrama do cliente)
type: note
tags: [master-sindico, client-canvas, diagram, mermaid]
project: master-sindico
source: Clients/MasterSindico original .canvas (2026-03-10 a 2026-03-24) — convertido para MD+mermaid em 2026-04-24
created: 2026-04-24
---

# Fluxo Ciclo Servico

Diagrama original do cliente convertido de `.canvas` (Obsidian Canvas) para Mermaid. **Visão visual** dos fluxos/arquitetura; conteúdo canônico vive em [[../04-requirements/_moc]] + [[../02-architecture/_moc]].

## Diagrama

```mermaid
graph TD
  CM["Connect Me<br/>Requisição"]
  PROP["Proposta<br/>Req 20"]
  VOTE["Votação Fornecedor<br/>Req 21"]
  DEAL["Negócio Fechado<br/>Req 22"]
  EXEC["Registro Execução<br/>Req 23"]
  TECH["Atividade Técnica<br/>Req 23.1"]
  COM["Comunicado Pós-Deal<br/>Req 24"]
  EVAL["Avaliação<br/>Req 26"]
  ADENDO["Adendo<br/>Req 25"]
  TL["Timeline"]
  CM --> PROP
  PROP --> VOTE
  VOTE --> DEAL
  DEAL --> EXEC
  DEAL --> TECH
  DEAL --> COM
  EXEC -->|validação síndico| TL
  TECH -->|validação síndico| TL
  COM -->|validação síndico| TL
  TL -->|imutável| ADENDO
  DEAL -->|concluído| EVAL
  classDef group fill:#eef,stroke:#66a,stroke-width:2px,color:#000
```

## Nodes (10)

- `CM` — Connect Me · Requisição
- `PROP` — Proposta · Req 20
- `VOTE` — Votação Fornecedor · Req 21
- `DEAL` — Negócio Fechado · Req 22
- `EXEC` — Registro Execução · Req 23
- `TECH` — Atividade Técnica · Req 23.1
- `COM` — Comunicado Pós-Deal · Req 24
- `EVAL` — Avaliação · Req 26
- `ADENDO` — Adendo · Req 25
- `TL` — Timeline

## Edges (11)

- `CM` → `PROP`
- `PROP` → `VOTE`
- `VOTE` → `DEAL`
- `DEAL` → `EXEC`
- `DEAL` → `TECH`
- `DEAL` → `COM`
- `EXEC` → `TL` — _validação síndico_
- `TECH` → `TL` — _validação síndico_
- `COM` → `TL` — _validação síndico_
- `TL` → `ADENDO` — _imutável_
- `DEAL` → `EVAL` — _concluído_

## Links

- [[_moc]] — índice dos canvas do cliente
- [[../CLAUDE]] — contrato do projeto
- [[../02-architecture/_moc]]
- [[../04-requirements/_moc]]