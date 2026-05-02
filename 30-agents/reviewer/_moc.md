---
title: MOC — Reviewer (pipeline operacional stub)
type: moc
tags: [moc, agents, reviewer, pipeline, dual-check, stub]
created: 2026-04-24
updated: 2026-04-24
---

# Reviewer — pipeline operacional (stub)

Pasta operacional do pipeline multi-agente. **Atualmente stub** (A-vault-001) — decisão de materialização será tomada em D3 (Passada 4).

## Papel previsto

Agente que **valida em contexto limpo** o que o Worker produziu:

- Lê execução de `_inbox/reviewer/<task-id>.md`
- Reviewer A e B rodam **em paralelo**, **independentes** (não veem review um do outro)
- Re-pesquisa fontes do plan
- Produz `reviews/<task-id>-A.md` + `reviews/<task-id>-B.md`
- Veredicto: `approved | revise | rejected`

Segue protocolo [[../playbooks/dual-check]] obrigatoriamente.

## Referência externa

[[../gsd-library/family-audit]] — biblioteca GSD com 8 agents desta família (verifier, eval-auditor, eval-planner, nyquist-auditor, security-auditor, ui-auditor, ui-checker, ui-researcher). Nosso Reviewer pode compor variantes temáticas.

## Padrões reutilizáveis

- **Postura adversarial** — parte de "falhou até provar contrário".
- **Evidência concreta > claim** — `file:line` obrigatório.
- **Severidade obrigatória** — BLOCKER/WARNING/INFO.
- **Read-only sobre código** — proibido modificar implementação.

## Status

⏳ **Não instrumentado**. A-vault-001 pendente.

## Vizinhos

- [[../_moc]]
- [[../_inbox/_moc]]
- [[../orchestrator/_moc]] — consume veredicto
- [[../playbooks/dual-check]]
- [[../gsd-library/family-audit]]
