---
title: gsd-debug-session-manager
type: agent-contract
tags: [agent, gsd, debug, external-reference]
source: https://github.com/gsd-build/get-shit-done
source_raw: 60-sources/get-shit-done/raw/agents/gsd-debug-session-manager.md
source_date: 2026-04-22
created: 2026-04-24
updated: 2026-04-24
aliases: [debug-session-manager]
---

# gsd-debug-session-manager

**Família**: [[family-debug]]. **Pipeline (nosso)**: Orchestrator (de uma sub-feature: loop de debug isolado).

## Intenção
Rodar o **loop multi-ciclo de debug em contexto isolado**, protegendo o orchestrator principal (`/gsd-debug`) de poluição por dezenas de evidências e checkpoints. Spawna `gsd-debugger`, media checkpoints via `AskUserQuestion`, dispara specialist skills, aplica fixes e devolve summary compacto (≤ 2K tokens).

## Entrada
- `slug`, `debug_file_path`, `symptoms_prefilled`, `tdd_mode`, `goal`, `specialist_dispatch_enabled`
- Debug file existente (leitura obrigatória antes de qualquer ação)

## Saída
- Debug file atualizado ciclo a ciclo, opcionalmente arquivado em `resolved/`
- Summary compacto: `## DEBUG SESSION COMPLETE` com Session / Root Cause / Fix / Cycles / TDD / Specialist review
- Variante `ABANDONED` se usuário para a sessão

## Ferramentas declaradas
`Read, Write, Bash, Grep, Glob, Task, AskUserQuestion`

## Padrão-chave replicável
1. **Context budget isolation**: manager só gerencia estado do loop — passa **caminhos de arquivo** para sub-agents, nunca inline de conteúdo. Codebase inteiro nunca entra no contexto dele.
2. **Security boundary via DATA_START/DATA_END**: toda resposta de usuário (via `AskUserQuestion` ou checkpoint) é envelopada como dado antes de ir para continuation agent — mitigação canônica de prompt injection.
3. **Specialist dispatch por hint**: `specialist_hint` (typescript/swift/python/ios/...) mapeia para skill correspondente que revisa a direção do fix antes de aplicar — camada extra de dual-check específica por stack.
4. **TDD gate explícito**: em `tdd_mode`, exige teste vermelho confirmado pelo usuário antes de liberar a fase green.

## Fonte
- Raw: [[../../60-sources/get-shit-done/raw/agents/gsd-debug-session-manager]]
- Repo: `gsd-build/get-shit-done`

## Links
- [[family-debug]]
- [[_moc]]
- [[pipeline-mapping]]
- [[../_moc]]
