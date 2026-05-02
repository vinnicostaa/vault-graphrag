---
title: Obsidian Vault Mapping
version: 1.0
status: stable
type: reference
---

# Obsidian Vault — mapa de notas úteis

O João mantém um vault Obsidian em `~/Documentos/Obsidian Vault/Clients/MasterSindico/` com notas de discovery, arquitetura, roadmap e decisões. Essas notas são **contexto de arquiteto** (reflexão do João), **não** spec executável.

Esta tabela existe para **localizar** contexto relevante rapidamente. **O MCP Obsidian** (ver `.kiro/steering/mcp-integration.md`) permite o agente ler/pesquisar notas em runtime.

## Estrutura do vault

```
Clients/MasterSindico/
├── 00 - Visão Geral/
│   ├── 00 - Índice Master Síndico.md
│   ├── Definition of Done.md
│   ├── Roadmap 2026.md
│   ├── 📂 Plano de Reorganização do Vault.md
│   └── 🗺️ Mapa de Artefatos Visuais.md
├── 01 - Arquitetura de Sistema/
│   ├── System Architecture.md          (Go Edition, marca de tempo Mar/2026)
│   ├── System Architecture.canvas      (visão visual)
│   ├── System Architecture - API Design.md
│   ├── System Architecture - Data Model.md
│   ├── System Architecture - Identity.md
│   ├── System Architecture - Institutional.md
│   ├── System Architecture - Commercial.md
│   ├── Decisões e Pendências.md
│   ├── Status do Documento.md
│   ├── Arquitetura do Ecossistema.md
│   └── Tldraw 2026-03-11 9.13PM.md     (visual)
├── 02 - Domínios e Requisitos/
│   ├── Domínio - Identidade.md
│   ├── Domínio - Comercial.md
│   ├── Domínio - Conteúdo.md
│   ├── Domínio - Assembleia.md
│   └── Domínio - Cross-Domain e Integrações.md
├── 03 - Modelagem e Dados/
│   └── Enums e Listas Mestre.md
├── 04 - Execution/
│   └── Sprint 1 Identity.md
└── 05 - Entregas/
    ├── 08-05-2026.md   (Marco 1)
    └── 20-06-2026.md   (Marco 2)
```

## Status de atualização (o que é confiável hoje)

| Nota | Atualização | Status |
|---|---|---|
| `00 - Índice Master Síndico.md` | 2026-03 | ✅ confiável — navegação |
| `Roadmap 2026.md` | 2026-03 | ⚠️ **datas diferentes do atual** — MS atual tem Marcos 08/05 e 20/06; Obsidian aponta 20/07 e 08/08. O MS atual prevalece. |
| `Definition of Done.md` | 2026-03 | ⚠️ menciona Ent ORM e Casbin — **desatualizado**. MS atual usa sqlc + ABAC próprio. |
| `01 - System Architecture.md` | 2026-03 | ⚠️ **Go Edition v4** mas cita Ent, Casbin, Uber-FX — desatualizado. MS atual usa pgx+sqlc, ABAC manual, DI manual. |
| `01 - System Architecture - Data Model.md` | 2026-03 | 🟡 útil como visão geral; schema real em `design/data-model.md` ou nas migrations `internal/shared/providers/database/migrations/` |
| `01 - System Architecture - API Design.md` | 2026-03 | 🟡 útil como skeleton; contratos reais em `design/api-design.md` + annotations Go |
| `02 - Domínio - *` | 2026-03 | 🟡 úteis como visão de negócio; detalhes técnicos em `requirements/*.md` |
| `04 - Sprint 1 Identity.md` | 2026-04 | ⚠️ menciona Uber-FX, Casbin — desatualizado tecnicamente. Escopo funcional ainda relevante. |
| `05 - Entregas/08-05-2026.md` | 2026-03 | ✅ marco correto |

## Quando consultar o Obsidian

- **Sim**: para entender contexto de negócio que não esteja em `requirements/`, visualizações (canvas, tldraw), decisões históricas do João com cliente
- **Não**: para decisões técnicas (stack, arquitetura Go, dependências) — backend é fonte de verdade
- **Não**: para roadmap operacional — `tasks/` é fonte de verdade

## Como usar o MCP Obsidian

```
mcp__obsidian__search_notes(query="Connect Me")          # buscar por termo
mcp__obsidian__read_note(path="Clients/MasterSindico/02 - Domínios e Requisitos/Domínio - Comercial.md")
mcp__obsidian__read_multiple_notes(paths=[...])          # até 10 notas em batch
mcp__obsidian__list_directory(path="Clients/MasterSindico/01 - Arquitetura de Sistema")
```

Permissões: **read-only** no `settings.json` atual — aceita `mcp__obsidian__search_notes`, `mcp__obsidian__read_*`, `mcp__obsidian__list_directory`. Tools de escrita (`write_note`, `delete_note`, `update_frontmatter`) requerem `ask`.

## Se divergência é encontrada

Quando o Obsidian diz X e o backend diz Y:

1. **Backend vence** (regra dura)
2. Abrir issue/nota: "Obsidian note X divergente do backend requirements Y"
3. Sugerir ao João atualizar a nota — não editar por conta própria (nota é dele)
4. Se a divergência é nome/rótulo, documentar em `.kiro/STATE.md`

## Notas deprecadas (ignorar)

- Qualquer menção a **Ent ORM**, **Casbin**, **Uber-FX**, **Drizzle** — stack descartado
- Datas de sprint do Roadmap 2026 — MS atual tem datas diferentes
- Menções a "Fastify" e "TypeScript backend" — legacy

## Fontes de atualidade

Se quiser saber o **estado real** do projeto:

1. **Código**: `internal/modules/` — é isso que está rodando
2. **Specs atuais**: `requirements/`, `design/`, `tasks/` (esta pasta é filha de)
3. **Bloqueadores**: `.kiro/STATE.md`
4. **Config local**: `.claude/settings.json`, `.mcp.json`, `.env`, `go.mod`
