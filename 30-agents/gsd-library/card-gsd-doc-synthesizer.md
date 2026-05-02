---
title: gsd-doc-synthesizer
type: agent-contract
tags: [agent, gsd, docs, external-reference]
source: https://github.com/gsd-build/get-shit-done
source_raw: 60-sources/get-shit-done/raw/agents/gsd-doc-synthesizer.md
source_date: 2026-04-22
created: 2026-04-24
updated: 2026-04-24
aliases: [doc-synthesizer]
---

# gsd-doc-synthesizer

**Família**: [[family-docs]]. **Pipeline (nosso)**: Dual Reviewer (aplicando precedência) + Planner (consolidação de intel).

## Intenção
Consumir classificações JSON produzidas por N `gsd-doc-classifier` paralelos, aplicar **regras de precedência** (ADR > SPEC > PRD > DOC), detectar ciclos de cross-ref, enforcar hard-blocks em LOCKED-vs-LOCKED e produzir um relatório de conflitos em 3 buckets. Não prompta o usuário; não escreve PROJECT.md/REQUIREMENTS.md/ROADMAP.md (isso é do `gsd-roadmapper`).

## Entrada
- `CLASSIFICATIONS_DIR` (JSONs), `INTEL_DIR` (saída), `CONFLICTS_PATH`
- `MODE: new | merge` (merge checa contra CONTEXT.md / ROADMAP.md existentes)
- `PRECEDENCE` override (opcional, default `[ADR, SPEC, PRD, DOC]`)

## Saída
- Intel por tipo: `decisions.md` (ADRs), `requirements.md` (PRDs com IDs `REQ-{slug}`), `constraints.md` (SPECs), `context.md` (DOCs)
- `INGEST-CONFLICTS.md` com 3 buckets: `unresolved-blockers` (BLOCKER), `competing-variants` (WARNING), `auto-resolved` (INFO)
- `SYNTHESIS.md` como entry point pro roadmapper
- Confirmação ≤ 10 linhas com status READY / BLOCKED / AWAITING USER

## Ferramentas declaradas
`Read, Write, Grep, Glob, Bash`

## Padrão-chave replicável
1. **Surface conflict, never pick silently**: dois LOCKED ADRs que contradizem → BLOCKER sempre; dois PRDs com acceptance divergente → **preserva ambos** como `competing-variants` em vez de merge "combinado".
2. **Cycle detection antes da síntese**: DFS 3-color sobre o grafo de `cross_refs`, cap em profundidade 50 — loop em synthesis produz garbage; docs fora do ciclo ainda processam.
3. **Provenance obrigatória**: todo entry carrega `source: {path}` para rastreabilidade downstream — tracing de "de onde veio esse requirement" é sempre possível.
4. **Conflict report é plain text, 3 buckets**: viola contrato do `doc-conflict-engine` se usar tabela markdown; severidade é explícita (BLOCKER/WARNING/INFO).

## Fonte
- Raw: [[../../60-sources/get-shit-done/raw/agents/gsd-doc-synthesizer]]
- Repo: `gsd-build/get-shit-done`

## Links
- [[family-docs]]
- [[_moc]]
- [[pipeline-mapping]]
- [[../_moc]]
