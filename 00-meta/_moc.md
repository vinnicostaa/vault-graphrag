---
title: MOC — 00-meta
type: moc
tags: [moc, meta, governance]
created: 2026-04-22
updated: 2026-04-24
---

# 00-meta — Governança do vault

Estado vivo **do vault em si** (não dos projetos). Canal de decisões estruturais e findings estruturais que afetam knowledge global, topologia, taxonomia, templates.

## Notas

- [[vault-guide]] — como operar este vault (ler antes de criar notas)
- [[STATE]] — **decisões vivas do vault global** (D-vault-###) — criado 2026-04-24 com D-vault-001..006
- [[AUDIT]] — **findings estruturais do vault global** (A-vault-###) — criado 2026-04-24 durante T6

## Distinção entre vault global e projetos

| Escopo | Decisões | Findings | Briefing de sessão |
|---|---|---|---|
| **Vault global** (knowledge atemporal, templates, pipeline) | [[STATE]] (D-vault-###) | [[AUDIT]] (A-vault-###) | N/A — sessão opera por projeto |
| **Projeto** (ex.: master-sindico, websocket-chat) | `50-projects/<slug>/STATE.md` (D-###) | `50-projects/<slug>/11-audit/AUDIT.md` (A-###) | `50-projects/<slug>/SESSION_CHARTER.md` |

**Regra**: mudar taxonomia/template/pipeline → `00-meta/STATE.md`. Mudar regra de negócio de produto → `50-projects/<slug>/STATE.md`.

## Vizinhos no grafo

- [[../_index]] — MOC raiz
- [[../CLAUDE]] — contrato raiz (§9 prescreve fluxo de leitura; §13 deve linkar aqui — ver A-vault-008)
- [[../40-templates/_moc]] — templates de STATE/AUDIT/CHARTER replicáveis pra novo projeto
- [[../30-agents/_moc]] — agentes consomem STATE/AUDIT no bootstrap; `30-agents/gsd-library/` implementado nesta governança (D-vault-004)
- [[../10-knowledge/methodology/graph-rag]] — topologia que o STATE+AUDIT suportam

## Dataview — últimas atualizações em 00-meta

```dataview
TABLE type, updated
FROM "00-meta"
SORT file.mtime DESC
LIMIT 10
```
