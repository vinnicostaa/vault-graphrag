---
inclusion: always
---

# Plugins Claude Code — marketplace oficial

Plugins complementam MCPs (que expõem ferramentas) oferecendo **skills, slash commands, hooks e agents** prontos. Marketplace oficial: `https://claude.com/plugins` · catálogo JSON: [anthropics/claude-plugins-official](https://github.com/anthropics/claude-plugins-official/blob/main/.claude-plugin/marketplace.json).

## Comando básico

```bash
/plugin                                 # abre painel
/plugin install <nome>@claude-plugins-official   # marketplace oficial (auto-registrado)
/plugin marketplace add <owner/repo>    # terceiros
/plugin install <nome>@<marketplace>
/reload-plugins                         # aplica mudanças
```

## Instalação por fase

### Dia 1 (obrigatórios — instalar antes de qualquer implementação)

**Status**: ✅ habilitados no `backend/.claude/settings.json` via `enabledPlugins`. Na primeira abertura do Claude Code neste diretório, os plugins são instalados automaticamente.

```bash
# Os plugins abaixo já estão em enabledPlugins — apenas referência
gopls-lsp@claude-plugins-official        # Go LSP: navegação + type errors em tempo real
typescript-lsp@claude-plugins-official   # TS LSP (quando frontend entrar)
security-guidance@claude-plugins-official # pre-tool hook detecta injection, eval, dangerouslySetInnerHTML
context7@claude-plugins-official         # docs oficiais de libs (Zitadel, pgx, sqlc, stripe-go, etc)
railway@claude-plugins-official          # deploy (Postgres, Redis, serviços)
```

**Deps**:
- `gopls-lsp` → `go install golang.org/x/tools/gopls@latest`
- `typescript-lsp` → `npm install -g typescript-language-server typescript`
- `railway` → `railway` CLI (`npm install -g @railway/cli`) + login (`railway login`)

### Semana 1 (workflow)

**Status**: ✅ todos habilitados em `enabledPlugins` + `extraKnownMarketplaces` (para superpowers) no `settings.json`.

```bash
# Já habilitados automaticamente:
feature-dev@claude-plugins-official            # agents SDD: Discuss (exploration) + Plan (design) + Verify (review)
code-review@claude-plugins-official            # revisão com scoring de confiança (filtra falsos positivos)
commit-commands@claude-plugins-official        # /commit, /push, /pr
claude-md-management@claude-plugins-official   # mantém CLAUDE.md dos 3 sub-projetos coerente
figma@claude-plugins-official                  # design→código; projeto oficial já mapeado em design-assets.md
superpowers@superpowers-marketplace            # TDD red/green, brainstorm, write-plan, execute-plan (third-party obra)
```

**Marketplace third-party** (superpowers): `obra/superpowers-marketplace` registrado em `extraKnownMarketplaces`. Auto-update **desligado** (padrão community) — atualizar manualmente com `/plugin marketplace update superpowers-marketplace`.

### Por módulo específico

| Quando | Instalar |
|---|---|
| Iniciar frontend SolidJS (Sprint 1.7+) | `/plugin install figma@claude-plugins-official` (design-to-code). **Projeto Figma oficial**: https://www.figma.com/design/pxJkmn8rzezKnCYXcA7lyL/Master-sindico-APP---UX · Plus `ui-ux-pro-max` via `/plugin marketplace add nextlevelbuilder/ui-ux-pro-max-skill` |
| Sprint 1.3 — Billing/Stripe | `/plugin install stripe@claude-plugins-official` (complementa o MCP Stripe já listado; o plugin traz `/explain-error`, `/test-cards`, best-practices) |
| Sprint 5+ — E2E frontend | `/plugin install playwright@claude-plugins-official` |

### Sprint 7 — Polish + observabilidade

```bash
/plugin install sentry@claude-plugins-official          # error monitoring + /seer agent
/reload-plugins
```

## Não instalar (baixo ROI)

| Plugin | Por quê |
|---|---|
| `github@claude-plugins-official` | Redundante — já temos MCP GitHub oficial em `.mcp.json` |
| `terraform@claude-plugins-official` | Só se migrar de Railway para AWS com IaC. Hoje overkill |
| `deploy-on-aws` / `aws-serverless` | Se ficar em Railway, não instalar |
| LSPs de outras linguagens | Projeto é Go + TS + Dart (Flutter tem seu próprio LSP via IDE) |
| `frontend-design` | Sobrepõe `figma` |

## Regras de segurança

1. **Publisher oficial vs community**: marketplace oficial (`claude-plugins-official`) tem auto-update. Third-party (`obra/superpowers-marketplace`) é auto-update **desligado** — atualize manualmente com `/plugin marketplace update <name>`.
2. **Revisar código antes de instalar** plugins de terceiros — todo plugin executa com seus privilégios.
3. **Credenciais em `.env`, não em settings**: Railway token, Figma API, Stripe key, etc. `.env` no `.gitignore`.
4. **Destrutivos pedem confirmação**: plugins como `railway` e `github` executam ações reais (delete service, close PR). Confirmação humana em cada uma.

## Como cada plugin entra no fluxo SDD (ciclo 5 fases)

| Fase | Plugins úteis |
|---|---|
| **Discuss** | `context7` (doc oficial), `feature-dev` (codebase exploration agent), `claude-md-management` |
| **Plan** | `superpowers` (`/brainstorm`, `/write-plan`), `feature-dev` (architecture design agent) |
| **Execute** | `gopls-lsp` + `typescript-lsp` (type errors em tempo real), `figma` (design-to-code), `stripe` (guidance) |
| **Verify** | `code-review` (scoring + false positive filter), `security-guidance` (pre-tool hook), `superpowers` (TDD red/green skills) |
| **Ship** | `commit-commands` (`/commit`, `/push`, `/pr`), `github` MCP, `railway` (deploy) |

## Interação com MCPs já configurados

Estado atual após consolidação:

| Ferramenta | MCP em `.mcp.json`? | Plugin em `settings.json`? | Por quê |
|---|---|---|---|
| **context7** | ❌ removido | ✅ `context7@claude-plugins-official` | Plugin é self-contained, sem duplicação |
| **github** | ✅ mantido (oficial GitHub MCP) | ❌ não | Plugin seria redundante — MCP já cobre |
| **postgres** | ✅ crystaldba restricted mode | ❌ não existe plugin equivalente | MCP é a única opção |
| **aws-docs** | ✅ awslabs read-only | ❌ não | MCP já é read-only e seguro |
| **obsidian** | ✅ (user-level em `~/.claude.json`) | ❌ não | Vault local, MCP é a entrada |
| **railway** | ❌ não | ✅ `railway@claude-plugins-official` | Plugin é vendor-official com CLI-first |
| **figma** | ❌ não | ✅ `figma@claude-plugins-official` | Plugin já tem MCP embutido + skills |
| **stripe** | ❌ comentado | ⏳ ativar Sprint 4 | Plugin traz skills `/explain-error`, `/test-cards`; MCP separado apenas se precisar de automação crua |
| **sentry** | ❌ | ⏳ ativar Sprint 7 | Plugin traz `/seer` agent |
| **playwright** | ❌ | ⏳ ativar Sprint 5+ | E2E frontend |

## Variáveis de ambiente necessárias

Em `backend/.env` (já no `.gitignore`):

```bash
# Railway
RAILWAY_TOKEN=...

# Figma (quando instalar)
FIGMA_ACCESS_TOKEN=figd_...

# Sentry (quando instalar, Sprint 7)
SENTRY_AUTH_TOKEN=...
SENTRY_ORG=master-sindico
SENTRY_PROJECT=backend

# Context7 (se usar via plugin; plugin pode pedir API key)
CONTEXT7_API_KEY=...
```

## Ponteiro para gerenciamento

- Lista viva de plugins instalados e status: `/plugin`
- Troubleshooting: ver `.claude/README.md` seção Troubleshooting
- Novos plugins descobertos: adicionar em `.kiro/STATE.md` com avaliação antes de instalar

## Plugins que **não existem oficialmente** (e alternativa)

| Caso de uso | Sem plugin | Alternativa |
|---|---|---|
| Zod (validação TS) | ✗ | Context7 MCP para docs + padrões no `steering/web-solidjs.md` |
| TanStack Router/Query/Form | ✗ | Context7 + steering manual |
| Zitadel/OIDC | ✗ | Context7 (trust 9.5) + skill custom em `.claude/skills/` quando precisar |
| pgx/sqlc/goose | ✗ | Context7 + `steering/database.md` + skills `migration-writer` e `sqlc-writer` |
| testcontainers-go | ✗ | Context7 + `steering/testing.md` |
| rapid (PBT) | ✗ | `steering/testing.md` |
| Docker Compose | ✗ | Bash tools nativos + `docker-compose.yml` do projeto |
| AWS CDK | ✗ | Context7 + se necessário |
| Flutter/Dart | ✗ | IDE-side (Android Studio/VS Code plugins tradicionais) |
