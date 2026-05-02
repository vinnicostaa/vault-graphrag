---
title: MOC — _memory
type: moc
tags: [moc, agents, memory, stub]
created: 2026-04-24
updated: 2026-04-24
---

# _memory — memória persistente cross-sessão (stub)

Pasta operacional do pipeline multi-agente descrita em [[../_moc]]. **Atualmente stub** — estrutura declarada mas sem conteúdo materializado (ver [[../../00-meta/AUDIT|A-vault-001]]).

## Papel previsto

Armazenar memória durável que **agentes escrevem** (não apenas leem):

- `patterns.md` — padrões que funcionaram e devem ser reusados
- `failures.md` — erros recorrentes e como evitá-los
- `decisions.md` — decisões cross-sessão de baixo peso (abaixo de D-vault-###)

Grafo se auto-expande: agente hoje aprende de agente ontem.

## Status

⏳ Não instrumentado. Decisão de materializar ou podar pendente em [[../../00-meta/AUDIT|A-vault-001]].

## Vizinhos

- [[../_moc]] — pipeline operacional
- [[../autonomous-execution]] — modo autônomo que consumiria memory
- [[../../00-meta/STATE]] · [[../../00-meta/AUDIT]]
