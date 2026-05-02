---
title: MOC — 50-projects
type: moc
tags:
  - moc
  - projects
created: 2026-04-22T00:00:00.000Z
updated: '2026-04-24'
---

# 50-projects

Projetos vivos neste vault. Cada projeto tem scaffold 00-12 canônico (ver [[../40-templates/project-scaffold/_moc]]).

## Projetos ativos

- [[master-sindico/CLAUDE]] — SaaS governança condominial BR multi-tenant (6 BCs Go + SolidJS + Flutter). Migração Fase 6 concluída 2026-04-23. M1 2026-05-08/18 (decisão macro em curso).
  - Governança viva: [[master-sindico/STATE]] · [[master-sindico/AUDIT]] · [[master-sindico/SESSION_CHARTER]] · [[master-sindico/ROADMAP]]
  - Legado arquivado em `master-sindico/_archive/2026-04-23-pre-v2-migration/`
  - Research destilado em [[../60-sources/master-sindico-research/_toc]] *(a criar TOC)*
  - Material ingerido em [[../90-archive/from-master-sindico-v2-ingested/INDEX]] *(se existir)*
- [[websocket-chat/CLAUDE]] — Sistema de mensagens diretas via WebSocket em Go (CLI, inspirado WhatsApp/Signal). Fase 1 em andamento (UUIDv7 + SQLite + entrega offline). Criado 2026-04-24.
  - Governança viva: [[websocket-chat/STATE]] · [[websocket-chat/AUDIT]] · [[websocket-chat/SESSION_CHARTER]] · [[websocket-chat/ROADMAP]]

## Como adicionar novo projeto

1. Copiar [[../40-templates/project-scaffold/_moc]] → `50-projects/<slug>/`
2. Adaptar `CLAUDE.md`, `_moc.md`, `STATE.md` (vazio inicial), `AUDIT.md` (vazio inicial)
3. Linkar neste MOC
4. Usar [[../30-agents/playbooks/onboarding-novo-projeto]]

## Vizinhos

- [[../10-knowledge/_moc]] — knowledge atemporal (linkar, não duplicar)
- [[../20-stacks/_moc]] — stacks cross-projeto
- [[../30-agents/_moc]] — operação multi-agente
- [[../40-templates/_moc]] — templates replicáveis
- [[../CLAUDE]] — contrato raiz do vault
