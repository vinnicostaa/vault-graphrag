---
title: Visibilidade Videos (diagrama do cliente)
type: note
tags: [master-sindico, client-canvas, diagram, mermaid]
project: master-sindico
source: Clients/MasterSindico original .canvas (2026-03-10 a 2026-03-24) — convertido para MD+mermaid em 2026-04-24
created: 2026-04-24
---

# Visibilidade Videos

Diagrama original do cliente convertido de `.canvas` (Obsidian Canvas) para Mermaid. **Visão visual** dos fluxos/arquitetura; conteúdo canônico vive em [[../04-requirements/_moc]] + [[../02-architecture/_moc]].

## Diagrama

```mermaid
graph TD
  E["🏢 Empresa<br/>faz upload"]
  V["📹 Vídeo Institucional"]
  S["👔 Síndico"]
  M["🏠 Morador"]
  E -->|upload| V
  V -->|N1/N2: 25% preview| S
  V -->|N3/Plus/Pro: completo| S
  V -->|autorizar_exibicao = true<br/>+ votação ativa| M
  M -->|BLOQUEADO<br/>após votação| V
  classDef group fill:#eef,stroke:#66a,stroke-width:2px,color:#000
```

## Nodes (4)

- `E` — 🏢 Empresa · faz upload
- `V` — 📹 Vídeo Institucional
- `S` — 👔 Síndico
- `M` — 🏠 Morador

## Edges (5)

- `E` → `V` — _upload_
- `V` → `S` — _N1/N2: 25% preview_
- `V` → `S` — _N3/Plus/Pro: completo_
- `V` → `M` — _autorizar_exibicao = true
+ votação ativa_
- `M` → `V` — _BLOQUEADO
após votação_

## Links

- [[_moc]] — índice dos canvas do cliente
- [[../CLAUDE]] — contrato do projeto
- [[../02-architecture/_moc]]
- [[../04-requirements/_moc]]