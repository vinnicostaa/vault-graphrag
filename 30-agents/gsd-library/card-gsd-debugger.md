---
title: gsd-debugger
type: agent-contract
tags: [agent, gsd, debug, external-reference]
source: https://github.com/gsd-build/get-shit-done
source_raw: 60-sources/get-shit-done/raw/agents/gsd-debugger.md
source_date: 2026-04-22
created: 2026-04-24
updated: 2026-04-24
aliases: [debugger]
---

# gsd-debugger

**Família**: [[family-debug]]. **Pipeline (nosso)**: Worker (investigativo) + Support (persistência de sessão).

## Intenção
Investigar bugs pelo **método científico** (hipótese → teste → evidência) mantendo estado persistente em arquivo que sobrevive a resets de contexto. Encontra root cause, opcionalmente aplica fix e verifica.

## Entrada
- Trigger verbatim do usuário (ou UAT parallel diagnosis)
- Flags de modo: `symptoms_prefilled`, `goal: find_root_cause_only | find_and_fix`, `tdd_mode`
- Knowledge base histórica em `.planning/debug/knowledge-base.md`

## Saída
- Debug file em `.planning/debug/{slug}.md` com Current Focus / Symptoms (imutável) / Eliminated (append) / Evidence (append) / Resolution
- Retorno estruturado: `ROOT CAUSE FOUND` | `DEBUG COMPLETE` | `CHECKPOINT REACHED` | `TDD CHECKPOINT` | `INVESTIGATION INCONCLUSIVE`
- `specialist_hint` derivado de extensões/frameworks dos arquivos envolvidos

## Ferramentas declaradas
`Read, Write, Edit, Bash, Grep, Glob, WebSearch`

## Padrão-chave replicável
1. **Hipóteses falsificáveis obrigatórias**: antes de tocar em código, `reasoning_checkpoint` com hypothesis + confirming_evidence + falsification_test + fix_rationale + blind_spots. Se qualquer campo fica vago → não confirmou root cause, volta a investigar.
2. **Debug file é o cérebro**: atualização **antes** da ação (não depois), `next_action` sempre concreto e acionável. Após `/clear`, qualquer agent retoma do mesmo ponto lendo o arquivo.
3. **Knowledge base append-only**: sessões resolvidas viram entradas com `Error patterns`; investigação futura faz match por keyword overlap (2+ tokens) e testa padrão conhecido **primeiro** (como hipótese, não certeza).
4. **Técnicas combináveis**: binary search + observability first + differential debugging + minimal reproduction + working backwards — tabela de seleção por situação.

## Fonte
- Raw: [[../../60-sources/get-shit-done/raw/agents/gsd-debugger]]
- Repo: `gsd-build/get-shit-done`

## Links
- [[family-debug]]
- [[_moc]]
- [[pipeline-mapping]]
- [[../_moc]]
