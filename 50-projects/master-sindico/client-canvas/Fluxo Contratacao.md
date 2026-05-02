---
title: Fluxo Contratacao (diagrama do cliente)
type: note
tags: [master-sindico, client-canvas, diagram, mermaid]
project: master-sindico
source: Clients/MasterSindico original .canvas (2026-03-10 a 2026-03-24) — convertido para MD+mermaid em 2026-04-24
created: 2026-04-24
---

# Fluxo Contratacao

Diagrama original do cliente convertido de `.canvas` (Obsidian Canvas) para Mermaid. **Visão visual** dos fluxos/arquitetura; conteúdo canônico vive em [[../04-requirements/_moc]] + [[../02-architecture/_moc]].

## Diagrama

```mermaid
graph TD
  S1["👔 Síndico<br/>Cria Connect Me"]
  E1["🏢 Empresa<br/>Tenho interesse"]
  N1["🤝 Negociação<br/>Fora da plataforma"]
  S2["✅ Validation Hub<br/>Registra 'Negócio Fechado'"]
  E2["🏢 Empresa<br/>Envia Registro de Execução"]
  S3["✅ Validation Hub<br/>Valida Registro"]
  T1["📜 Timeline<br/>Publica (Imutável)"]
  E3["🏢 Empresa<br/>Envia Comunicado Técnico"]
  S4["✅ Validation Hub<br/>Valida e Publica"]
  S1 --> E1
  E1 --> N1
  N1 --> S2
  S2 --> E2
  E2 --> S3
  S3 --> T1
  E2 --> E3
  E3 --> S4
  classDef group fill:#eef,stroke:#66a,stroke-width:2px,color:#000
```

## Nodes (9)

- `S1` — 👔 Síndico · Cria Connect Me
- `E1` — 🏢 Empresa · Tenho interesse
- `N1` — 🤝 Negociação · Fora da plataforma
- `S2` — ✅ Validation Hub · Registra 'Negócio Fechado'
- `E2` — 🏢 Empresa · Envia Registro de Execução
- `S3` — ✅ Validation Hub · Valida Registro
- `T1` — 📜 Timeline · Publica (Imutável)
- `E3` — 🏢 Empresa · Envia Comunicado Técnico
- `S4` — ✅ Validation Hub · Valida e Publica

## Edges (8)

- `S1` → `E1`
- `E1` → `N1`
- `N1` → `S2`
- `S2` → `E2`
- `E2` → `S3`
- `S3` → `T1`
- `E2` → `E3`
- `E3` → `S4`

## Links

- [[_moc]] — índice dos canvas do cliente
- [[../CLAUDE]] — contrato do projeto
- [[../02-architecture/_moc]]
- [[../04-requirements/_moc]]