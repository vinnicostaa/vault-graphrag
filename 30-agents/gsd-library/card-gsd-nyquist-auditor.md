---
title: gsd-nyquist-auditor
type: agent-contract
tags: [agent, gsd, audit, external-reference]
source: https://github.com/gsd-build/get-shit-done
source_raw: 60-sources/get-shit-done/raw/agents/gsd-nyquist-auditor.md
source_date: 2026-04-22
created: 2026-04-24
updated: 2026-04-24
aliases: [nyquist-auditor]
---

# gsd-nyquist-auditor

**Família**: [[family-audit]]. **Pipeline (nosso)**: Reviewer (foco em preencher gaps de cobertura de teste).

## Intenção

Para cada gap de validação declarado em VALIDATION.md: gera um teste comportamental real que pode falhar, roda, debuga se quebrar (máx. 3 iterações), reporta o que **acontece de fato** — não o que a implementação alega. Só mexe em arquivos de teste/fixtures; implementação é read-only. Bugs da implementação são escalados, não corrigidos.

## Entrada

- `<gaps>`: lista de task_id + requirement + tipo de gap (`no_test_file` / `test_fails` / `no_automated_command`).
- Arquivos de implementação (para entender exports, API pública, contratos de I/O — read-only).
- PLAN.md, SUMMARY.md, configuração de infraestrutura de teste (framework, runner, conventions).
- VALIDATION.md existente (mapa atual).

## Saída

- Arquivos de teste novos/atualizados seguindo convenções do projeto (pytest/jest/vitest/go test).
- Atualização da VALIDATION.md com `automated_command` por task_id.
- Retorno estruturado: `## GAPS FILLED`, `## PARTIAL`, ou `## ESCALATE` com detalhes das iterações de debug.

## Ferramentas declaradas

Read, Write, Edit, Bash, Glob, Grep.

## Padrão-chave replicável

- **Never weaken the assertion**: se um teste falha, a solução é escalar (implementation bug) ou corrigir o teste quando a expectativa estava errada — nunca afrouxar o assert para fazer passar. "Test file created" não é "gap filled" — só teste *rodando e passando* fecha o gap.
- **Escopo de edit estritamente limitado**: só arquivos de teste, fixtures e VALIDATION.md. Bug da impl é ESCALATE, não fix silencioso — separa limpo a responsabilidade de cobertura vs. correção.

## Fonte

- Raw: [[../../60-sources/get-shit-done/raw/agents/gsd-nyquist-auditor]]
- Repo: `gsd-build/get-shit-done`

## Links

- [[family-audit]]
- [[_moc]]
- [[pipeline-mapping]]
- [[../_moc]]
