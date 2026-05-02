---
title: gsd-code-reviewer
type: agent-contract
tags: [agent, gsd, executor, external-reference]
source: https://github.com/gsd-build/get-shit-done
source_raw: 60-sources/get-shit-done/raw/agents/gsd-code-reviewer.md
source_date: 2026-04-22
created: 2026-04-24
updated: 2026-04-24
aliases: [code-reviewer]
---

# gsd-code-reviewer

**Família**: [[family-executor]]. **Pipeline (nosso)**: Worker (produz artefato de review para o Reviewer/fixer consumirem).

## Intenção

Revisa arquivos-fonte com postura adversarial — hipótese de partida é que o código contém defeitos. Produz REVIEW.md com findings classificados por severidade (Critical/Warning/Info), sem validar "se o trabalho foi feito". Escopo de escrita: só o REVIEW.md — nunca modifica o código.

## Entrada

- Lista explícita de arquivos alterados (do workflow) — se ausente, falha fechada em vez de adivinhar escopo.
- `depth`: `quick` (só regex — 2min), `standard` (análise por arquivo, 5-15min), `deep` (call-chain cross-file, 15-30min).
- `CLAUDE.md` + skills do projeto (o que é anti-padrão aqui).

## Saída

- REVIEW.md com frontmatter (`phase`, `depth`, `files_reviewed_list`, contagem por severidade, `status: clean | issues_found | skipped`).
- Findings estruturados com `file:line`, `issue`, `fix` (snippet concreto para Critical/Warning).

## Ferramentas declaradas

Read, Write, Bash, Grep, Glob.

## Padrão-chave replicável

- **clean ≠ skipped**: `status: clean` significa "revisado e sem issues"; `status: skipped` significa "não havia arquivos revisáveis". A distinção importa downstream (quem consome sabe se o silêncio é confiança ou ausência de dado).
- **Severidade obrigatória**: todo finding carrega `BLOCKER | WARNING | INFO`. Performance (O(n²), memory leaks) é fora de escopo v1 — só entra se também for correctness.

## Fonte

- Raw: [[../../60-sources/get-shit-done/raw/agents/gsd-code-reviewer]]
- Repo: `gsd-build/get-shit-done`

## Links

- [[family-executor]]
- [[_moc]]
- [[pipeline-mapping]]
- [[../_moc]]
