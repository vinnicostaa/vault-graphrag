---
title: MOC — 60-sources/_ingested
type: moc
tags: [moc, sources, ingestion, tracking]
created: 2026-04-22
---

# 60-sources/_ingested — rastreio de destilação

**Um arquivo por fonte** rastreando:
- Status atual (% destilado)
- Notas geradas e pra onde foram promovidas
- Dívidas pendentes (o que ainda falta destilar)

Os `_toc.md` locais de cada fonte têm a **fila de prioridade**; este `_ingested/` mantém o **contrato de entrega** — o que **efetivamente virou conhecimento** no vault.

## Status por fonte

| Fonte | Tracker | Prioritárias | Destiladas | Promovidas |
|---|---|---:|---:|---:|
| [[spec-kit]] | [[spec-kit]] | 10 | 0 | 0 |
| [[clean-coder]] | [[clean-coder]] | 6 | 0 | 0 |
| [[get-shit-done]] | [[get-shit-done]] | 4 | 0 | 0 |
| [[fernando-franco-valle]] | [[fernando-franco-valle]] | 2 | 0 | 0 |
| [[refactoring-guru]] | [[refactoring-guru]] | 2 | 0 | 0 |

**Total a destilar nas prioritárias**: 24 unidades de trabalho.

## Como atualizar

Quando uma nota é promovida de `<fonte>/_notes/` pra destino (`10-knowledge/`, `20-stacks/`, `30-agents/`, `40-templates/`):

1. Mover o arquivo (`mv`).
2. Atualizar linha em `_ingested/<fonte>.md` na tabela "Promovidas".
3. Decrementar contador no `_moc.md` (aqui) e no `_toc.md` da fonte.
4. Se a fonte ficar 100% destilada, marcar `✅ drenada` no `_moc.md` de 60-sources.

## Vizinhos

- [[../_moc]] — sources
- [[../../10-knowledge/_moc]], [[../../20-stacks/_moc]], [[../../30-agents/_moc]], [[../../40-templates/_moc]]
