---
title: Plugins Claude Code — guia genérico
type: guide
tags: [claude-code, plugins, methodology]
source: .kiro/steering/plugins.md
created: 2026-04-22
---

# Plugins Claude Code — guia genérico

Plugins complementam MCPs (que expõem ferramentas externas) oferecendo **skills, slash commands, hooks e sub-agents** prontos. Este doc descreve o **padrão de uso** independente de projeto. Instalação específica por projeto vive em `50-projects/<proj>/plugins.md`.

Marketplace oficial: `https://claude.com/plugins` · catálogo: [`anthropics/claude-plugins-official`](https://github.com/anthropics/claude-plugins-official).

## Comandos básicos

```
/plugin                                      abre painel
/plugin install <nome>@claude-plugins-official  marketplace oficial (auto-registrado)
/plugin marketplace add <owner/repo>         terceiros
/plugin install <nome>@<marketplace>
/reload-plugins                              aplica mudanças
```

## Plugins úteis por fase SDD

Ver [[../10-knowledge/methodology/sdd-workflow]] *(a criar)* pro ciclo completo. Plugins mapeados por fase:

| Fase | Plugins úteis |
|---|---|
| **Discuss** | `context7` (doc oficial de libs), `feature-dev` (codebase exploration), `claude-md-management` |
| **Plan** | `superpowers` (`/brainstorm`, `/write-plan`), `feature-dev` (architecture design) |
| **Execute** | LSP do stack (`gopls-lsp`, `typescript-lsp`, etc.) — type errors em tempo real; `figma` (design-to-code); SDK-specific (`stripe`, etc.) |
| **Verify** | `code-review` (scoring + false positive filter), `security-guidance` (pre-tool hook), `superpowers` (TDD red/green) |
| **Ship** | `commit-commands` (`/commit`, `/push`, `/pr`), MCP do repo host (GitHub/GitLab), plataforma de deploy (`railway`, etc.) |

## Regras de segurança

1. **Publisher oficial vs community**: marketplace oficial (`claude-plugins-official`) tem auto-update. Third-party tem auto-update **desligado** — atualizar manualmente via `/plugin marketplace update <name>`.
2. **Revisar código antes de instalar** plugins de terceiros — plugin executa com seus privilégios.
3. **Credenciais em `.env`, não em settings**: tokens (Railway, Figma, Stripe, Sentry) vão em `.env` no `.gitignore`.
4. **Destrutivos pedem confirmação**: plugins como `railway` e `github` executam ações reais (delete service, close PR). Confirmar humano-in-the-loop.

## Interação com MCPs

Muitos plugins **substituem** um MCP que você teria configurado em `.mcp.json`. Evitar duplicação:

- Se o plugin é vendor-official e self-contained → usar plugin, remover MCP correspondente.
- Se MCP oferece funcionalidade mais ampla (ex: GitHub MCP oficial vs plugin), **manter MCP**.
- Documentar a decisão no `plugins.md` do projeto específico (tabela "MCP em `.mcp.json`? / Plugin habilitado?").

## Plugins que comumente **não** existem (e alternativas)

Muita lib popular não tem plugin oficial. Alternativas:

| Caso | Solução |
|---|---|
| Lib sem plugin mas muito usada | Context7 MCP/plugin + steering manual (`.kiro/steering/<lib>.md`) |
| Pattern específico de framework | Skill customizada em `.claude/skills/` |
| Código gerado (sqlc, drizzle-kit, build_runner) | Skill customizada (`migration-writer`, `sqlc-writer`) |
| Testing library | Steering doc + templates |
| Docker Compose / IaC | Bash tools nativos + arquivos YAML do repo |

## Como escolher plugins ao iniciar um projeto

Heurística:

1. **LSP do stack principal** — sempre (type errors em tempo real economizam ciclos).
2. **`context7`** — sempre (docs oficiais de libs; maior trust reduz alucinação).
3. **`security-guidance`** — sempre (pre-tool hook barra `eval`, `innerHTML` cru, etc.).
4. **`claude-md-management`** — sempre que houver multi-CLAUDE.md (monorepo).
5. **`code-review`** — sempre (scoring filtra false positive de lint).
6. **`commit-commands`** — sempre (padroniza mensagens, integração /commit /pr).
7. **Plataforma de deploy** — quando entrar em deploy real.
8. **SDK-specific** (Stripe, Sentry) — sob demanda, no sprint que o SDK entra.
9. **Figma** — se há design em Figma + implementação de telas.
10. **Superpowers** (third-party) — workflow avançado de TDD e brainstorm.

## Links

- [[_moc]] — MOC de agentes
- [[../10-knowledge/methodology/sdd-workflow]] *(a criar)* — ciclo SDD 5 fases
- Aplicação em projeto: [[../50-projects/master-sindico/plugins]]
