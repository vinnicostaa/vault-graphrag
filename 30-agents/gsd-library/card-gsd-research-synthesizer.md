---
title: gsd-research-synthesizer
type: agent-contract
tags: [agent, gsd, research, external-reference]
source: https://github.com/gsd-build/get-shit-done
source_raw: 60-sources/get-shit-done/raw/agents/gsd-research-synthesizer.md
source_date: 2026-04-22
created: 2026-04-24
updated: 2026-04-24
aliases: [research-synthesizer]
---

# gsd-research-synthesizer

**Família**: [[family-research]]. **Pipeline (nosso)**: Orchestrator.

## Intenção

Lê os 4 outputs dos researchers paralelos (STACK, FEATURES, ARCHITECTURE, PITFALLS) e sintetiza em `SUMMARY.md` coeso que alimenta o `gsd-roadmapper`. Papel de reducer: transforma N outputs em 1 narrativa.

## Entrada

- `.planning/research/STACK.md`, `FEATURES.md`, `ARCHITECTURE.md`, `PITFALLS.md` (escritos pelos parallel researchers)

## Saída

`.planning/research/SUMMARY.md` com:
- Executive Summary (2-3 parágrafos sintéticos)
- Key Findings por research file
- **Implications for Roadmap** (sugestão de estrutura de fases com rationale)
- Research Flags (quais fases precisam `/gsd-research-phase` depois)
- Confidence assessment + gaps

Commita todos os 4 research files + SUMMARY.md juntos (researchers não commitam).

## Ferramentas declaradas

`Read`, `Write`, `Bash`. **Sem web/Context7** — sintetiza o que outros já pesquisaram.

## Padrão-chave replicável

- **Reducer puro no fan-out/fan-in** — N researchers paralelos (cada um em contexto isolado) → 1 synthesizer que vê tudo. Evita que researchers precisem se coordenar entre si.
- **Synthesis ≠ concatenation** — achados são *integrados* (ex: features cruzam com pitfalls para virar flags de fase), não só empilhados. Roadmapper recebe narrativa, não 4 docs crus.
- **Synthesizer comita pelo grupo** — solução clássica para atomicidade: parallel workers escrevem mas não commitam; o reducer fecha o batch com um único commit.

## Fonte

- Raw: [[../../60-sources/get-shit-done/raw/agents/gsd-research-synthesizer]]
- Repo: `gsd-build/get-shit-done`

## Links

- [[family-research]]
- [[_moc]]
- [[pipeline-mapping]]
- [[../_moc]]
