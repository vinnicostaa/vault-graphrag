---
title: gsd-integration-checker
type: agent-contract
tags: [agent, gsd, executor, external-reference]
source: https://github.com/gsd-build/get-shit-done
source_raw: 60-sources/get-shit-done/raw/agents/gsd-integration-checker.md
source_date: 2026-04-22
created: 2026-04-24
updated: 2026-04-24
aliases: [integration-checker]
---

# gsd-integration-checker

**Família**: [[family-executor]]. **Pipeline (nosso)**: Worker (verificação cross-fase após milestone).

## Intenção

Verifica que fases **se conectam**, não apenas que cada fase individualmente parece completa. Existência ≠ integração. Um componente pode existir sem nunca ser importado; uma rota pode existir sem consumidor. O foco são as conexões end-to-end e os fluxos E2E do usuário.

## Entrada

- Lista de fases no escopo do milestone.
- SUMMARY.md de cada fase (exports e arquivos criados).
- Estrutura do código (`src/`, rotas de API, componentes).
- Lista de REQ-IDs do milestone (para mapa de integração por requirement).

## Saída

- Relatório estruturado com:
  - Wiring: `connected`, `orphaned` (exportado e nunca importado), `missing` (conexão esperada ausente).
  - APIs: consumidas vs. órfãs.
  - Auth protection: áreas sensíveis com/sem checagem.
  - E2E flows: completos vs. `broken_at: <step>` com `steps_missing`.
  - Requirements Integration Map (REQ-ID → path → WIRED/PARTIAL/UNWIRED).

## Ferramentas declaradas

Read, Bash, Grep, Glob.

## Padrão-chave replicável

- **Trace de caminho completo**: Component → API → DB → Response → Display — quebra em qualquer ponto é fluxo quebrado. Não basta "importa e exporta"; é preciso ver a *chamada* no ponto certo.
- **Findings acionáveis**: "Dashboard não funciona" é inútil; "Dashboard.tsx linha 45 faz fetch em `/api/users` mas não dá await na resposta" é acionável. BLOCKER quando a conexão está ausente/quebrada; WARNING quando existe mas é frágil.

## Fonte

- Raw: [[../../60-sources/get-shit-done/raw/agents/gsd-integration-checker]]
- Repo: `gsd-build/get-shit-done`

## Links

- [[family-executor]]
- [[_moc]]
- [[pipeline-mapping]]
- [[../_moc]]
