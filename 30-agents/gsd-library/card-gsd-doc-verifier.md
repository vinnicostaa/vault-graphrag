---
title: gsd-doc-verifier
type: agent-contract
tags: [agent, gsd, docs, external-reference]
source: https://github.com/gsd-build/get-shit-done
source_raw: 60-sources/get-shit-done/raw/agents/gsd-doc-verifier.md
source_date: 2026-04-22
created: 2026-04-24
updated: 2026-04-24
aliases: [doc-verifier]
---

# gsd-doc-verifier

**Família**: [[family-docs]]. **Pipeline (nosso)**: Dual Reviewer (específico para drift doc ↔ código).

## Intenção
Verificar **factualidade** de claims gerados em docs contra o codebase vivo, sob **stance adversarial**: assumir que toda afirmação está errada até evidência de filesystem provar o contrário. Saída: JSON estruturado por doc com PASS / FAIL / UNVERIFIABLE.

## Entrada
- `<verify_assignment>` com `doc_path` + `project_root`
- O próprio arquivo de doc (leitura obrigatória)
- `package.json` (cache parseado para verificação de scripts/deps)

## Saída
- `.planning/tmp/verify-{doc_filename}.json` com `doc_path`, `claims_checked`, `claims_passed`, `claims_failed`, array `failures[{line, claim, expected, actual}]`
- Confirmação de 1-2 linhas ao orchestrator

## Ferramentas declaradas
`Read, Write, Bash, Grep, Glob`

## Padrão-chave replicável
1. **Adversarial stance > naive trust**: lista explícita de "failure modes de verificadores moles" (aceitar path existe sem checar conteúdo, parar no primeiro PASS, marcar UNCERTAIN quando grep resolveria). Doc recém-escrita não presume-se correta.
2. **5 categorias taxonomizadas**: file paths (por extensão), commands (npm/yarn/pnpm scripts), API endpoints (`(GET|POST|PUT|DELETE|PATCH) /path`), function/export claims (`<id>(`), dependencies (context phrases como "uses", "depends on"). Extração regex-driven, verificação filesystem-only.
3. **Skip rules pré-extração**: VERIFY markers, quoted prose de terceiros, example prefixes (`e.g.`, `like:`), placeholders (`your-`, `<name>`), blocks `diff|example|template` — evita falsos FAILs em conteúdo que **é** exemplo.
4. **Read-only discipline**: nunca executa comandos do doc, nunca modifica doc file. `claims_failed === failures.length` validado antes de escrever.

## Fonte
- Raw: [[../../60-sources/get-shit-done/raw/agents/gsd-doc-verifier]]
- Repo: `gsd-build/get-shit-done`

## Links
- [[family-docs]]
- [[_moc]]
- [[pipeline-mapping]]
- [[../_moc]]
