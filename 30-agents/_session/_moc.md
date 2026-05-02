---
title: MOC — _session
type: moc
tags: [moc, agents, session, stub]
created: 2026-04-24
updated: 2026-04-24
---

# _session — estado da sessão atual (stub)

Pasta operacional do pipeline multi-agente descrita em [[../_moc]]. **Atualmente stub** — estrutura declarada mas sem conteúdo materializado (ver [[../../00-meta/AUDIT|A-vault-001]]).

## Papel previsto

Armazenar **estado efêmero da sessão corrente** — some quando a sessão termina:

- `active-task.md` — task atualmente em execução + progresso
- `agent-state.md` — estado interno de cada subagente rodando
- `context-budget.md` — uso de contexto por fase

Diferente de [[../_memory/_moc|_memory]] que é durável cross-sessão.

## Status

⏳ Não instrumentado. Decisão de materializar ou podar pendente em [[../../00-meta/AUDIT|A-vault-001]].

## Vizinhos

- [[../_moc]]
- [[../_memory/_moc]] — contraparte durável
- [[../../00-meta/STATE]] · [[../../00-meta/AUDIT]]
