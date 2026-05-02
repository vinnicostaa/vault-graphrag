---
title: Master Síndico — Plugins Claude Code habilitados
type: project
tags: [master-sindico, claude-code, plugins]
source: .kiro/steering/plugins.md
created: 2026-04-22
project: master-sindico
---

# Plugins Claude Code — Master Síndico

Plugins ativados no `backend/.claude/settings.json` via `enabledPlugins`. Explicação genérica do conceito de plugins está em [[../../30-agents/claude-code-plugins]].

## Dia 1 (obrigatórios)

Status: ✅ habilitados em `enabledPlugins`. Primeira abertura do Claude Code no repo instala tudo automaticamente.

```
gopls-lsp@claude-plugins-official
typescript-lsp@claude-plugins-official
security-guidance@claude-plugins-official
context7@claude-plugins-official
railway@claude-plugins-official
```

**Dependências**:
- `gopls-lsp` → `go install golang.org/x/tools/gopls@latest`
- `typescript-lsp` → `npm install -g typescript-language-server typescript`
- `railway` → `railway` CLI (`npm install -g @railway/cli`) + `railway login`

## Semana 1 (workflow SDD)

Status: ✅ todos habilitados + `extraKnownMarketplaces` (superpowers) no `settings.json`.

```
feature-dev@claude-plugins-official
code-review@claude-plugins-official
commit-commands@claude-plugins-official
claude-md-management@claude-plugins-official
figma@claude-plugins-official
superpowers@superpowers-marketplace
```

**Marketplace third-party** (superpowers): `obra/superpowers-marketplace`. Auto-update **desligado** — atualizar manualmente com `/plugin marketplace update superpowers-marketplace`.

## Por módulo / sprint

| Quando | Instalar |
|---|---|
| Sprint 1.7+ (frontend SolidJS) | `figma@claude-plugins-official` já ativado. Projeto Figma: https://www.figma.com/design/pxJkmn8rzezKnCYXcA7lyL/Master-sindico-APP---UX . Plus `ui-ux-pro-max` via `/plugin marketplace add nextlevelbuilder/ui-ux-pro-max-skill` |
| Sprint 1.3 (Billing/Stripe) | `/plugin install stripe@claude-plugins-official` — traz `/explain-error`, `/test-cards` |
| Sprint 5+ (E2E frontend) | `/plugin install playwright@claude-plugins-official` |
| Sprint 7 (Polish + observabilidade) | `/plugin install sentry@claude-plugins-official` — traz `/seer` agent |

## Não instalar (baixo ROI para este projeto)

| Plugin | Motivo |
|---|---|
| `github@claude-plugins-official` | Redundante — MCP GitHub oficial já em `.mcp.json` |
| `terraform@claude-plugins-official` | Só se migrar Railway → AWS com IaC. Hoje overkill. |
| `deploy-on-aws` / `aws-serverless` | Se ficar em Railway, não instalar |
| LSPs de outras linguagens | Projeto é Go + TS + Dart (Flutter tem LSP nativo via IDE) |
| `frontend-design` | Sobrepõe `figma` |

## MCPs em `.mcp.json`

| Ferramenta | MCP ativo? | Por quê |
|---|---|---|
| context7 | ❌ removido | Plugin `context7@official` é self-contained |
| github | ✅ MCP oficial | Plugin seria redundante |
| postgres | ✅ `crystaldba` restricted mode | MCP é a única opção |
| aws-docs | ✅ `awslabs` read-only | MCP já seguro |
| obsidian | ✅ (user-level em `~/.claude.json`) | Vault local, MCP é a entrada |
| railway | ❌ não | Plugin vendor-official com CLI-first |
| figma | ❌ não | Plugin já tem MCP embutido + skills |
| stripe | ⏳ Sprint 4 | Plugin traz skills; MCP separado se precisar automação crua |
| sentry | ⏳ Sprint 7 | Plugin traz `/seer` |
| playwright | ⏳ Sprint 5+ | E2E frontend |

## Variáveis de ambiente

Em `backend/.env` (gitignored):

```bash
RAILWAY_TOKEN=...
FIGMA_ACCESS_TOKEN=figd_...
SENTRY_AUTH_TOKEN=...
SENTRY_ORG=master-sindico
SENTRY_PROJECT=backend
CONTEXT7_API_KEY=...
```

## Links

- [[_moc]] — MOC do projeto
- [[overview]] — stack geral do projeto
- [[../../30-agents/claude-code-plugins]] — padrão genérico de uso de plugins no SDD
