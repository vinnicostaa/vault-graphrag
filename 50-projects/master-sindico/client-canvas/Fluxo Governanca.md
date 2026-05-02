---
title: Fluxo Governanca (diagrama do cliente)
type: note
tags: [master-sindico, client-canvas, diagram, mermaid]
project: master-sindico
source: Clients/MasterSindico original .canvas (2026-03-10 a 2026-03-24) — convertido para MD+mermaid em 2026-04-24
created: 2026-04-24
---

# Fluxo Governanca

Diagrama original do cliente convertido de `.canvas` (Obsidian Canvas) para Mermaid. **Visão visual** dos fluxos/arquitetura; conteúdo canônico vive em [[../04-requirements/_moc]] + [[../02-architecture/_moc]].

## Diagrama

```mermaid
graph TD
  S1["👔 Síndico<br/>Cria atividade da gestão"]
  T1["📜 Timeline<br/>Recebe atividade"]
  PD1["📋 Plano Diretor<br/>Status atualizado auto"]
  E1["🏢 Empresa<br/>Envia registro execução"]
  S2["👔 Síndico<br/>Valida registro"]
  T2["📜 Timeline<br/>Publica registro (imutável)"]
  M2["🏠 Morador<br/>Avalia gestão"]
  M1["🏠 Morador<br/>Visualiza Timeline"]
  S1 --> T1
  T1 --> PD1
  E1 --> S2
  S2 --> T2
  T2 -->|modo leitura| M1
  S2 -->|avalia (3 perguntas)| M2
  classDef group fill:#eef,stroke:#66a,stroke-width:2px,color:#000
```

## Nodes (8)

- `S1` — 👔 Síndico · Cria atividade da gestão
- `T1` — 📜 Timeline · Recebe atividade
- `PD1` — 📋 Plano Diretor · Status atualizado auto
- `E1` — 🏢 Empresa · Envia registro execução
- `S2` — 👔 Síndico · Valida registro
- `T2` — 📜 Timeline · Publica registro (imutável)
- `M2` — 🏠 Morador · Avalia gestão
- `M1` — 🏠 Morador · Visualiza Timeline

## Edges (6)

- `S1` → `T1`
- `T1` → `PD1`
- `E1` → `S2`
- `S2` → `T2`
- `T2` → `M1` — _modo leitura_
- `S2` → `M2` — _avalia (3 perguntas)_

## Links

- [[_moc]] — índice dos canvas do cliente
- [[../CLAUDE]] — contrato do projeto
- [[../02-architecture/_moc]]
- [[../04-requirements/_moc]]