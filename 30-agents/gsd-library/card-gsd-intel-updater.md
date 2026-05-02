---
title: gsd-intel-updater
type: agent-contract
tags: [agent, gsd, planning, external-reference]
source: https://github.com/gsd-build/get-shit-done
source_raw: 60-sources/get-shit-done/raw/agents/gsd-intel-updater.md
source_date: 2026-04-22
created: 2026-04-24
updated: 2026-04-24
aliases: [intel-updater]
---

# gsd-intel-updater

**Família**: [[family-planning]]. **Pipeline (nosso)**: Orchestrator (produz intel que outros agents consultam em vez de re-escanear).

## Intenção

Lê o código do projeto e escreve intel estruturado em `.planning/intel/` (`stack.json`, `files.json`, `apis.json`, `deps.json`, `arch.md`). Cada claim é baseado em evidência do próprio arquivo — zero suposição a partir de nome de pasta.

## Entrada

- Diretiva do `/gsd-intel`: `full` (todos os 5 arquivos) ou `partial --files <paths>`.
- Layout do runtime detectado (`.claude` vs `.kilo`) — resolve paths canônicos.
- Intel anterior (quando existe) para merge em modo partial.

## Saída

- `stack.json` (languages, frameworks, tools, content_formats).
- `files.json` (exports/imports/type — só símbolos reais, nunca descrições).
- `apis.json` (rotas/endpoints com method/path/file).
- `deps.json` (runtime/dev/peer + `used_by` + `invocation`).
- `arch.md` (sumário humano de arquitetura).
- `.last-refresh.json` via `gsd-sdk query intel.snapshot`.
- Marker final: `## INTEL UPDATE COMPLETE` ou `## INTEL UPDATE FAILED`.

## Ferramentas declaradas

Read, Write, Bash, Glob, Grep.

## Padrão-chave replicável

- **Evidência sempre rastreável**: cada entrada em `files.json`/`apis.json` aponta para um path real verificado por Glob/Read — nunca inferido. Exports são identificadores reais, não frases descritivas.
- **Cross-platform**: Bash só pra `gsd-sdk query`; listagem/leitura de arquivos sempre via Glob/Read (evita quebrar em Windows).

## Fonte

- Raw: [[../../60-sources/get-shit-done/raw/agents/gsd-intel-updater]]
- Repo: `gsd-build/get-shit-done`

## Links

- [[family-planning]]
- [[_moc]]
- [[pipeline-mapping]]
- [[../_moc]]
