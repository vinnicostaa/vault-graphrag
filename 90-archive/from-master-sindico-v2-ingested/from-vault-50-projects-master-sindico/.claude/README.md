# .claude/ — Configuração do Claude Code para o Backend

Este diretório contém a configuração local do Claude Code para o backend Master Síndico.

## Estrutura

```
.claude/
├── settings.json       ← permissões + hooks + modelo (versionado)
├── skills/             ← slash commands customizados (versionados)
│   ├── task-executor.md
│   ├── go-review.md
│   ├── migration-writer.md
│   └── sqlc-writer.md
└── README.md           ← este arquivo
```

**MCPs ficam em `../.mcp.json`** (raiz do backend), não aqui. Separação:
- `.claude/settings.json` → comportamento do agente (permissões, hooks, modelo)
- `.mcp.json` → quais MCPs estão disponíveis (servidores externos)

## settings.json — principais blocos

### `model`
- `claude-opus-4-7` default (capacidade máxima)
- `claude-sonnet-4-6` fallback (equilíbrio quando Opus não está disponível)

### `hooks`
Hook `PostToolUse` dispara `go build ./... 2>&1 | head -20` após cada `Write`/`Edit`. Sinal rápido de quebra de compilação.

**Testes, vet, coverage, gosec, govulncheck NÃO rodam no hook** — ficam no pre-commit e CI. Hook precisa ser barato para não travar o fluxo.

### `permissions`
Política de três tiers:
- **`allow`** (auto-execute): ferramentas reversíveis (Read, Grep, Glob, WebFetch) + MCPs read-only + comandos Bash específicos (build, test, docker compose, etc)
- **`ask`** (confirmar): ferramentas com efeito (Write, Edit) + MCPs que alteram estado em sandbox (Stripe, GitHub create/update) + git add/commit
- **`deny`** (bloqueado): destrutivos (`rm -rf`, `git push --force`, `git reset --hard`, `DROP TABLE`, `docker volume rm`, AWS services de escrita)

Ajuste aqui quando adicionar nova ferramenta ou descobrir nova necessidade.

## skills/ — slash commands

Invocar com `/{nome}` no Claude Code interativo.

### `/task-executor`
Executa uma task do `.kiro/specs/master-sindico/tasks/` seguindo o protocolo SDD de 5 fases (Discuss → Plan → Execute → Verify → Ship). Ver `.kiro/steering/sdd-workflow.md`.

### `/go-review`
Revisa código Go por prioridade (Dependency Rule → Error Handling → Padrões Go → DDD → Segurança). Não avança se houver problemas críticos.

### `/migration-writer`
Cria migration goose com:
- Numeração por módulo (identity 001-099, billing 100-199, etc)
- CHECK constraints em enums/invariantes
- Índices padrão (soft delete, tenant, FK, status)

### `/sqlc-writer`
Escreve query sqlc com:
- Colunas listadas explicitamente (nunca `SELECT *`)
- `deleted_at IS NULL` em tabelas com soft delete
- Tenant isolation (`WHERE condominium_id = $N`)

## MCPs — ver `../.mcp.json` + `.kiro/steering/mcp-integration.md`

Obrigatórios desde Sprint 0:
- **Context7** — doc oficial de libs (Zitadel trust 9.5, pgx, sqlc, stripe-go, SolidJS)
- **GitHub MCP oficial** — PR/issue/review
- **Postgres MCP Pro** (`--access-mode=restricted`) — schema, explain_plan, analyze_workload
- **AWS Docs MCP** — read-only, sempre seguro

A partir do Sprint 4: **Stripe MCP** (com `sk_test_*`). Descomentar bloco em `.mcp.json`.

A partir do Sprint 5: **Playwright MCP** (E2E frontend), **ui-ux-pro-max skill** (design system).

A partir do Sprint 7: **Sentry MCP**, AWS servers específicos (ECS, CloudWatch, IaC).

## Variáveis de ambiente necessárias

Em `backend/.env` (no `.gitignore` — **nunca commitar**):

```bash
# MCPs
CONTEXT7_API_KEY=                                                    # opcional
GITHUB_PAT=ghp_...                                                   # fine-grained, 30d, escopo repo
DATABASE_URI_LOCAL=postgresql://postgres:postgres@localhost:5432/master_sindico

# Stripe (descomentar em .mcp.json no Sprint 4)
STRIPE_KEY_PRIV=sk_test_...                                          # NUNCA sk_live_* em dev
STRIPE_WEBHOOK_SECRET=whsec_...

# Zitadel
ZITADEL_DOMAIN=...
ZITADEL_KEY_PATH=./zitadel-key.json                                  # service account key
ZITADEL_PROJECT_ID=...
ZITADEL_WEB_CLIENT_ID=...
ZITADEL_WEB_ENCRYPTION_KEY=                                          # openssl rand -hex 16
ZITADEL_WEB_HASH_KEY=                                                # openssl rand -hex 16

# Banco e cache
DB_HOST=localhost
DB_PORT=5432
DB_NAME=master_sindico
DB_USER=postgres
DB_PASSWORD=postgres
REDIS_HOST=localhost
REDIS_PORT=6379
REDIS_PASSWORD=redis_password
```

Ver `.env.example` para template completo quando criado.

## Fluxo recomendado ao trabalhar em nova task

1. Abra a task em `.kiro/specs/master-sindico/tasks/{sprint}.md`.
2. Rode `/task-executor task-id` — o agente segue o protocolo SDD de 5 fases.
3. Ao fim da fase Execute, antes de Ship: rode `/go-review` nos arquivos modificados.
4. Corrija o que o review apontar.
5. Valide localmente:
   ```bash
   go build ./... && go vet ./... && go test -race -count=1 ./...
   go test -tags=integration -count=1 ./...
   go-test-coverage --profile=coverage.out --threshold-file=.coverage.yml
   gosec -severity high ./... && govulncheck ./...
   ```
6. Abra PR (via GitHub MCP ou `gh pr create`). Fluxo completo em `.kiro/steering/sdd-workflow.md`.

## Fontes de contexto carregadas automaticamente

O Claude Code carrega na abertura de sessão:
- `CLAUDE.md` (raiz do backend) — orientação enxuta
- `.kiro/steering/*.md` com `inclusion: always` — regras duras
- `.kiro/STATE.md` — bloqueadores e decisões vivas

Quando o contexto cresce, o Claude Code faz auto-compact preservando: objetivo do usuário, arquivos tocados, comandos executados, decisões de design. Sessões de horas não estouram a janela.

## Troubleshooting

**MCP não conecta** → verifique env vars em `.env` e se o binário do MCP está instalado (ex: `docker pull ghcr.io/github/github-mcp-server`).

**Hook `go build` muito lento** → não é. Se estiver, revise se há imports circulares ou caches corrompidos (`go clean -cache`).

**Agente usa API desatualizada de SDK** → reforce no prompt: "consulte Context7 antes de codar stripe-go". Ver `.kiro/steering/mcp-integration.md`.

**Agente quer fazer algo destrutivo** → `permissions.deny` bloqueia. Se precisa fazer intencional (ex: drop table em dev), faça você manualmente no terminal, não pelo agente.
