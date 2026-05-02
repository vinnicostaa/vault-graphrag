---
title: gsd-code-fixer
type: agent-contract
tags: [agent, gsd, executor, external-reference]
source: https://github.com/gsd-build/get-shit-done
source_raw: 60-sources/get-shit-done/raw/agents/gsd-code-fixer.md
source_date: 2026-04-22
created: 2026-04-24
updated: 2026-04-24
aliases: [code-fixer]
---

# gsd-code-fixer

**Família**: [[family-executor]]. **Pipeline (nosso)**: Worker (modo corretivo pós-review).

## Intenção

Aplica fixes de achados em REVIEW.md um a um, de forma atômica e inteligente — a sugestão do reviewer é *guidance*, não patch cego. Lê o estado atual do arquivo, adapta o fix, verifica em 3 tiers, commita por finding e produz REVIEW-FIX.md com contagem `fixed / skipped / status`.

## Entrada

- REVIEW.md (findings classificados CR-#/WR-#/IN-# com file/line/issue/fix).
- `fix_scope`: `critical_warning` (default) ou `all`.
- `CLAUDE.md` do projeto + skills.
- Código-fonte atual (pode ter drifted desde o review).

## Saída

- Um commit por finding (`fix({padded_phase}): {finding_id} {short_description}`) com todos os arquivos afetados.
- `REVIEW-FIX.md` em `phase_dir` com frontmatter (`findings_in_scope`, `fixed`, `skipped`, `status`) + seções "Fixed Issues" e "Skipped Issues".
- Findings logicamente corrigidos mas que exigem verificação humana ficam marcados `"fixed: requires human verification"`.

## Ferramentas declaradas

Read, Edit, Write, Bash, Grep, Glob.

## Padrão-chave replicável

- **Verificação em 3 tiers**: Tier 1 (reler o arquivo modificado — sempre); Tier 2 (syntax check — `node -c`, `tsc --noEmit`, `python ast.parse`); Tier 3 (fallback aceita Tier 1). Erros pré-existentes em outros arquivos não quebram o commit — só erros *introduzidos* no arquivo editado.
- **Rollback atômico**: `git checkout -- {file}` (commit ainda não aconteceu); nunca usar Write para desfazer — escrita parcial corrompe o arquivo sem caminho de recuperação.

## Fonte

- Raw: [[../../60-sources/get-shit-done/raw/agents/gsd-code-fixer]]
- Repo: `gsd-build/get-shit-done`

## Links

- [[family-executor]]
- [[_moc]]
- [[pipeline-mapping]]
- [[../_moc]]
