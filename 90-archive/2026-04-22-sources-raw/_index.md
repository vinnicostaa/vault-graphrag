---
title: 2026-04-22 — Sources raw archive (i18n + admin)
type: note
tags: [archive, sources, i18n, admin]
archived: 2026-04-22
archived_reason: limpeza da reorganização de 60-sources — ruído admin (README i18n, CHANGELOG, CODE_OF_CONDUCT, terms/privacy) e duplicação i18n (ES/FR/JA/KO/PL/PT-BR/RU/UK em refactoring.guru; ja-JP/ko-KR/zh-CN em GSD) fora do caminho quente
created: 2026-04-22
---

# 2026-04-22 — Sources raw archive

Material movido de `60-sources/` durante a reorganização canônica. **Não deletar** — trilha de auditoria.

## Política

- **EN é a fonte canônica** em todas as sources externas. Traduções (pt-BR, ja-JP, etc.) movidas pra cá.
- **Ruído administrativo** de repos OSS (CHANGELOG, CONTRIBUTING, CODE_OF_CONDUCT, SECURITY, SUPPORT, VERSIONING, PULL_REQUEST_TEMPLATE, AGENTS.md dos repos, DEVELOPMENT.md) não agrega conhecimento técnico — arquivado.
- **Ruído de mirror httrack** (index.html.md raiz, terms/privacy por idioma) — arquivado.
- **JS content extraído** de bundles Next.js (fernando-franco-valle/_js-content/) — fragmentos sem contexto semântico, arquivado.

## O que há aqui

| Subpasta | Origem | Motivo |
|---|---|---|
| [[spec-kit/]] | `60-sources/spec-kit/` | Ruído admin de repo OSS |
| [[clean-coder/]] | `60-sources/clean-coder/` | Reservado — nada movido (estrutura limpa) |
| [[get-shit-done/]] | `60-sources/get-shit-done/` | README i18n + admin + docs i18n não-EN |
| [[fernando-franco-valle/]] | `60-sources/fernando-franco-valle/` | _js-content + páginas de news/simulados/progresso + assets Next.js |
| [[refactoring-guru/]] | `60-sources/refactoring-guru/` | i18n não-EN (~2900 MDs) + terms/privacy + admin de mirror |

## Como reverter

`mv 90-archive/2026-04-22-sources-raw/<fonte>/<pasta> 60-sources/<fonte>/raw/<pasta>` — estrutura original preservada intacta.

## Vizinhos no grafo

- [[../_moc]]
- [[../../60-sources/_moc]]
