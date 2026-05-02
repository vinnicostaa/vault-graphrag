---
title: gsd-verifier
type: agent-contract
tags: [agent, gsd, audit, external-reference]
source: https://github.com/gsd-build/get-shit-done
source_raw: 60-sources/get-shit-done/raw/agents/gsd-verifier.md
source_date: 2026-04-22
created: 2026-04-24
updated: 2026-04-24
aliases: [verifier]
---

# gsd-verifier

**Família**: [[family-audit]]. **Pipeline (nosso)**: Reviewer.

## Intenção

Verifica, depois da execução, se o **goal** da fase foi realmente alcançado no código — não confia no SUMMARY.md. Goal-backward: parte do que a fase *deveria* entregar e traça até o arquivo real (existe, substancial, conectado, com dado fluindo). Implementa o padrão Escalation Gate: gaps irresolvíveis são escalados, não varridos.

## Entrada

- PLAN.md (frontmatter `must_haves`), SUMMARY.md (reivindicações a serem falsificadas).
- ROADMAP.md (success_criteria da fase — contrato não-negociável).
- REQUIREMENTS.md (cobertura REQ-ID), VERIFICATION.md anterior (modo re-verify).
- `CLAUDE.md` + skills do projeto.

## Saída

- `.planning/phases/<fase>/<phase>-VERIFICATION.md` com status (`passed | gaps_found | human_needed`), score `N/M`, `gaps` e `deferred` em frontmatter YAML, `overrides` quando aceitos.
- Resultado estruturado ao orchestrator (nunca commita — o orchestrator agrupa).

## Ferramentas declaradas

Read, Write, Bash, Grep, Glob.

## Padrão-chave replicável

- **4 níveis de artefato**: existe → substancial (não-stub) → wired (importado E usado) → data flows (fonte produz dado real, não `[]` hardcoded). Passar os 3 primeiros mas falhar no 4º é `HOLLOW` — componente renderiza placeholder.
- **clean ≠ passed**: se há itens de verificação humana (visual, fluxo real-time, external service), status é `human_needed` mesmo com score N/N — `passed` só quando a seção humana está vazia.

## Fonte

- Raw: [[../../60-sources/get-shit-done/raw/agents/gsd-verifier]]
- Repo: `gsd-build/get-shit-done`

## Links

- [[family-audit]]
- [[_moc]]
- [[pipeline-mapping]]
- [[../_moc]]
