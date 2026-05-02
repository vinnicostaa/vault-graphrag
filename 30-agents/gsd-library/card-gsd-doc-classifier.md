---
title: gsd-doc-classifier
type: agent-contract
tags: [agent, gsd, docs, external-reference]
source: https://github.com/gsd-build/get-shit-done
source_raw: 60-sources/get-shit-done/raw/agents/gsd-doc-classifier.md
source_date: 2026-04-22
created: 2026-04-24
updated: 2026-04-24
aliases: [doc-classifier]
---

# gsd-doc-classifier

**Família**: [[family-docs]]. **Pipeline (nosso)**: Planner (sub-etapa: ingestão de material pré-existente).

## Intenção
Classificar **um único** documento de planejamento como ADR / PRD / SPEC / DOC / UNKNOWN, extraindo título, scope e cross-refs. Roda em **paralelo** (um spawn por arquivo) sob `/gsd-ingest-docs`. A fidelidade dessa classificação é load-bearing: tagging errado corrompe todo o pipeline downstream.

## Entrada
- `FILEPATH` (doc a classificar), `OUTPUT_DIR` para o JSON
- `MANIFEST_TYPE` + `MANIFEST_PRECEDENCE` opcionais (se usuário declarou via manifest, são autoritativos)

## Saída
- Um JSON por doc em `{OUTPUT_DIR}/{slug}-{source_hash}.json` (hash SHA-256 do path evita colisão entre múltiplos `README.md`)
- Campos: `source_path`, `type`, `confidence (high/medium/low)`, `manifest_override`, `title`, `summary`, `scope[]`, `cross_refs[]`, `locked`, `precedence`, `notes`
- Linha única de confirmação ao orchestrator

## Ferramentas declaradas
`Read, Write, Grep, Glob`

## Padrão-chave replicável
1. **Heurística rápida antes de leitura**: filename/path match (`**/adr/**`, `0001-*.md`) dá sinal forte antes de abrir o arquivo — economia de contexto em ingest grande.
2. **Três camadas de sinal**: manifest (autoritativo) > frontmatter YAML > conteúdo; empate resolve por precedência (ADR > SPEC > PRD > DOC) e registra ambiguidade em `notes`.
3. **Locked ≠ existir**: só ADR com status `Accepted` vira `locked: true`. Proposed/Draft nunca — evita que decisões não-finalizadas virem barreiras rígidas downstream.
4. **UNKNOWN é status legítimo**: não baixar confidence silenciosamente; se sinais são finos, explicita `UNKNOWN` com observed signals para o synthesizer bloquear.

## Fonte
- Raw: [[../../60-sources/get-shit-done/raw/agents/gsd-doc-classifier]]
- Repo: `gsd-build/get-shit-done`

## Links
- [[family-docs]]
- [[_moc]]
- [[pipeline-mapping]]
- [[../_moc]]
