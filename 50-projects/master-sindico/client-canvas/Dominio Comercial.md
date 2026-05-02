---
title: Dominio Comercial (diagrama do cliente)
type: note
tags: [master-sindico, client-canvas, diagram, mermaid]
project: master-sindico
source: Clients/MasterSindico original .canvas (2026-03-10 a 2026-03-24) — convertido para MD+mermaid em 2026-04-24
created: 2026-04-24
---

# Dominio Comercial

Diagrama original do cliente convertido de `.canvas` (Obsidian Canvas) para Mermaid. **Visão visual** dos fluxos/arquitetura; conteúdo canônico vive em [[../04-requirements/_moc]] + [[../02-architecture/_moc]].

## Diagrama

```mermaid
graph TD
  g_com["📦 💼 COMERCIAL"]:::group
  REG["Registro Empresa<br/>CNPJ · Perfil"]
  CM["Connect Me<br/>4 direções"]
  PROP["Propostas<br/>Envio · Validação"]
  DEAL["Negócio Fechado<br/>Dupla confirmação"]
  EXEC["Execução de Serviço<br/>Registro · Validação"]
  EVAL["Avaliação<br/>5 critérios"]
  VIZ["Vizinhança<br/>Parceiros · Promoções"]
  TIMELINE["📜 Timeline"]
  CM -->|oportunidade aceita| PROP
  PROP -->|proposta aceita| DEAL
  DEAL -->|serviço prestado| EXEC
  EXEC -->|validado pelo síndico| TIMELINE
  DEAL -->|concluído| EVAL
  REG -->|perfil visível| CM
  classDef group fill:#eef,stroke:#66a,stroke-width:2px,color:#000
```

## Nodes (9)

- **[GROUP]** `g_com` — 💼 COMERCIAL
- `REG` — Registro Empresa · CNPJ · Perfil
- `CM` — Connect Me · 4 direções
- `PROP` — Propostas · Envio · Validação
- `DEAL` — Negócio Fechado · Dupla confirmação
- `EXEC` — Execução de Serviço · Registro · Validação
- `EVAL` — Avaliação · 5 critérios
- `VIZ` — Vizinhança · Parceiros · Promoções
- `TIMELINE` — 📜 Timeline

## Edges (6)

- `CM` → `PROP` — _oportunidade aceita_
- `PROP` → `DEAL` — _proposta aceita_
- `DEAL` → `EXEC` — _serviço prestado_
- `EXEC` → `TIMELINE` — _validado pelo síndico_
- `DEAL` → `EVAL` — _concluído_
- `REG` → `CM` — _perfil visível_

## Links

- [[_moc]] — índice dos canvas do cliente
- [[../CLAUDE]] — contrato do projeto
- [[../02-architecture/_moc]]
- [[../04-requirements/_moc]]