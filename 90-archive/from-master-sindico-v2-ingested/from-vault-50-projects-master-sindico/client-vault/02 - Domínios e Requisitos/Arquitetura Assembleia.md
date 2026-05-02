---
title: "Arquitetura Assembleia (canvas)"
type: canvas
tags: [canvas, obsidian, converted]
source: 50-projects/master-sindico/client-vault/02 - Domínios e Requisitos/Arquitetura Assembleia.canvas
converted: 2026-04-22
---

# Arquitetura Assembleia

> Canvas Obsidian convertido. Estrutura original: JSON com nodes + edges.

## Nodes

- **group**: ⏳ Pré-Assembleia
- **text**: Configuração & Pauta / Lock de Edição
- **text**: Edital Automático / PDF
- **text**: Registro de Ciência / (Bloqueia App)
- **text**: Procurações & / Simulador Quórum
- **group**: 🔴 Ao Vivo
- **text**: Check-in / App/Terminal/QR
- **text**: Painel do Presidente / Controle de Fluxo
- **text**: Telão (Big Screen) / Apresentação + Operação
- **text**: Fila de Fala / Timer: 2/3 min
- **text**: Votação Tempo Real / Peso Fracionado
- **group**: 📜 Pós-Assembleia
- **text**: Homologação / Lock Definitivo
- **text**: Consolidação / Dados p/ Ata
- **text**: Trilha de Auditoria / Transparência

## Edges

- CONF → EDITAL 
- EDITAL → CIENCIA 
- PROX → g_live 
- CIENCIA → g_live 
- CHECK → PRES 
- PRES → VOTO 
- VOTO → TELAO 
- VOZ → TELAO 
- PRES → HOMOLOG 
- HOMOLOG → ATA 
- ATA → AUDIT 
