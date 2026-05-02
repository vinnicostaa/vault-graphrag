---
title: Arquitetura Assembleia (diagrama do cliente)
type: note
tags: [master-sindico, client-canvas, diagram, mermaid]
project: master-sindico
source: Clients/MasterSindico original .canvas (2026-03-10 a 2026-03-24) — convertido para MD+mermaid em 2026-04-24
created: 2026-04-24
---

# Arquitetura Assembleia

Diagrama original do cliente convertido de `.canvas` (Obsidian Canvas) para Mermaid. **Visão visual** dos fluxos/arquitetura; conteúdo canônico vive em [[../04-requirements/_moc]] + [[../02-architecture/_moc]].

## Diagrama

```mermaid
graph TD
  g_pre["📦 ⏳ Pré-Assembleia"]:::group
  CONF["Configuração & Pauta<br/>Lock de Edição"]
  EDITAL["Edital Automático<br/>PDF"]
  CIENCIA["Registro de Ciência<br/>(Bloqueia App)"]
  PROX["Procurações &<br/>Simulador Quórum"]
  g_live["📦 🔴 Ao Vivo"]:::group
  CHECK["Check-in<br/>App/Terminal/QR"]
  PRES["Painel do Presidente<br/>Controle de Fluxo"]
  TELAO["Telão (Big Screen)<br/>Apresentação + Operação"]
  VOZ["Fila de Fala<br/>Timer: 2/3 min"]
  VOTO["Votação Tempo Real<br/>Peso Fracionado"]
  g_post["📦 📜 Pós-Assembleia"]:::group
  HOMOLOG["Homologação<br/>Lock Definitivo"]
  ATA["Consolidação<br/>Dados p/ Ata"]
  AUDIT["Trilha de Auditoria<br/>Transparência"]
  CONF --> EDITAL
  EDITAL --> CIENCIA
  PROX --> g_live
  CIENCIA --> g_live
  CHECK --> PRES
  PRES --> VOTO
  VOTO --> TELAO
  VOZ --> TELAO
  PRES --> HOMOLOG
  HOMOLOG --> ATA
  ATA --> AUDIT
  classDef group fill:#eef,stroke:#66a,stroke-width:2px,color:#000
```

## Nodes (15)

- **[GROUP]** `g_pre` — ⏳ Pré-Assembleia
- `CONF` — Configuração & Pauta · Lock de Edição
- `EDITAL` — Edital Automático · PDF
- `CIENCIA` — Registro de Ciência · (Bloqueia App)
- `PROX` — Procurações & · Simulador Quórum
- **[GROUP]** `g_live` — 🔴 Ao Vivo
- `CHECK` — Check-in · App/Terminal/QR
- `PRES` — Painel do Presidente · Controle de Fluxo
- `TELAO` — Telão (Big Screen) · Apresentação + Operação
- `VOZ` — Fila de Fala · Timer: 2/3 min
- `VOTO` — Votação Tempo Real · Peso Fracionado
- **[GROUP]** `g_post` — 📜 Pós-Assembleia
- `HOMOLOG` — Homologação · Lock Definitivo
- `ATA` — Consolidação · Dados p/ Ata
- `AUDIT` — Trilha de Auditoria · Transparência

## Edges (11)

- `CONF` → `EDITAL`
- `EDITAL` → `CIENCIA`
- `PROX` → `g_live`
- `CIENCIA` → `g_live`
- `CHECK` → `PRES`
- `PRES` → `VOTO`
- `VOTO` → `TELAO`
- `VOZ` → `TELAO`
- `PRES` → `HOMOLOG`
- `HOMOLOG` → `ATA`
- `ATA` → `AUDIT`

## Links

- [[_moc]] — índice dos canvas do cliente
- [[../CLAUDE]] — contrato do projeto
- [[../02-architecture/_moc]]
- [[../04-requirements/_moc]]