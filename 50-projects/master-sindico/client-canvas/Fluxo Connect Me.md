---
title: Fluxo Connect Me (diagrama do cliente)
type: note
tags: [master-sindico, client-canvas, diagram, mermaid]
project: master-sindico
source: Clients/MasterSindico original .canvas (2026-03-10 a 2026-03-24) — convertido para MD+mermaid em 2026-04-24
created: 2026-04-24
---

# Fluxo Connect Me

Diagrama original do cliente convertido de `.canvas` (Obsidian Canvas) para Mermaid. **Visão visual** dos fluxos/arquitetura; conteúdo canônico vive em [[../04-requirements/_moc]] + [[../02-architecture/_moc]].

## Diagrama

```mermaid
graph TD
  O1["Originador<br/>Cria formulário"]
  CM1["Connect Me<br/>Registra LGPD"]
  D1["Destinatário<br/>Recebe notificação"]
  D2["Destinatário<br/>'Tenho Interesse'"]
  CM2["Connect Me<br/>Revela dados cruzados"]
  O2["Originador<br/>Recebe contato"]
  D3["Destinatário<br/>'Não Atendo'"]
  CM3["Connect Me<br/>Notifica recusa"]
  O3["Originador<br/>Fim do fluxo"]
  O1 --> CM1
  CM1 --> D1
  D1 --> D2
  D2 --> CM2
  CM2 --> O2
  D1 --> D3
  D3 --> CM3
  CM3 --> O3
  classDef group fill:#eef,stroke:#66a,stroke-width:2px,color:#000
```

## Nodes (9)

- `O1` — Originador · Cria formulário
- `CM1` — Connect Me · Registra LGPD
- `D1` — Destinatário · Recebe notificação
- `D2` — Destinatário · 'Tenho Interesse'
- `CM2` — Connect Me · Revela dados cruzados
- `O2` — Originador · Recebe contato
- `D3` — Destinatário · 'Não Atendo'
- `CM3` — Connect Me · Notifica recusa
- `O3` — Originador · Fim do fluxo

## Edges (8)

- `O1` → `CM1`
- `CM1` → `D1`
- `D1` → `D2`
- `D2` → `CM2`
- `CM2` → `O2`
- `D1` → `D3`
- `D3` → `CM3`
- `CM3` → `O3`

## Links

- [[_moc]] — índice dos canvas do cliente
- [[../CLAUDE]] — contrato do projeto
- [[../02-architecture/_moc]]
- [[../04-requirements/_moc]]