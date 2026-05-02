---
title: Deploy Config — Railway + AWS
version: 1.0
status: stable
type: design
domain: cross-domain
tags: [deploy, infra, railway, aws, env]
---

# Deploy Config — GitHub + Railway + AWS (futuro)

Estratégia de deploy do Master Síndico. **SCM**: GitHub (org `mastersindico`). **Deploy primário**: Railway (dev, staging, prod inicial). **Deploy futuro**: AWS ECS Fargate + RDS + ElastiCache (scale).

## GitHub — organização + repositórios

- **Org**: https://github.com/mastersindico
- **backend** (Go): https://github.com/mastersindico/backend — este repositório
- **web** (SolidJS monorepo): https://github.com/mastersindico/web
- **app** (Flutter): https://github.com/mastersindico/app
- **platform** (legacy TS/Fastify): https://github.com/mastersindico/platform — **arquivado**, apenas referência histórica

Cada repo é **independente** — não é monorepo. Pull request, CI, release separados. O diretório local `Development/` agrega os 3 ativos + 1 legacy como workspace, mas cada sub-projeto tem seu próprio `.git/`, issue tracker, e lifecycle de release.

### Convenções por repo

| Repo | Branch default | Strategy | CI |
|---|---|---|---|
| `backend` | `main` | Feature branch + PR + squash merge | Sprint 6 (CI completo: build, vet, test, coverage, gosec, govulncheck) |
| `web` | `main` | Feature branch + PR | Sprint 6 (typecheck, biome, rsbuild build) |
| `app` | `main` | Feature branch + PR | Sprint 6 (flutter analyze + test) |

PAT atual (em `backend/.env`) tem acesso aos 3 repos via `GITHUB_OWNER=mastersindico`. MCP GitHub do `.mcp.json` opera no contexto do repo atual (detecta via `git remote get-url origin`).

## Railway — Projeto oficial

- **Nome**: `mastersindico`
- **Ambiente inicial**: `plataforma`
- **URL do projeto**: https://railway.com/project/efe664b7-5352-48e7-ae09-78d2f286e0d5
- **Project ID**: `efe664b7-5352-48e7-ae09-78d2f286e0d5`

CLI local já autenticado via `railway login`. Token projeto fica em `backend/.env` como `RAILWAY_TOKEN` (opcional em uso interativo).

### Serviços no Railway

| Serviço | Status |
|---|---|
| API Go (container) | A configurar — Dockerfile multi-stage |
| PostgreSQL 18 | Plug-in Railway |
| Redis 7 | Plug-in Railway |
| NATS JetStream | A avaliar (Railway suporta via image custom) |

### Comandos (via plugin Railway ou CLI)

```bash
railway status                              # estado do projeto
railway logs --tail                         # logs em tempo real
railway variables                           # env vars configuradas
railway run -- go run ./cmd/migrate up      # rodar migrations contra DB remoto
railway up                                  # deploy (ask em settings.json)
```

Via plugin Claude Code: permissions já configuradas — read ops em `allow`, write ops em `ask`, delete em `deny`.

### Env vars em produção (Railway Variables)

Mapeamento das env vars locais → Railway:

| Local (`.env`) | Railway | Fonte do valor |
|---|---|---|
| `APP_ENV=production` | `APP_ENV=production` | fixo |
| `DB_HOST`, `DB_PORT`, etc | vem dos plug-ins PostgreSQL/Redis via Reference Variables | Railway injeta automaticamente |
| `ZITADEL_DOMAIN` | Var manual | Instância Zitadel prod (criar separada de dev) |
| `ZITADEL_KEY_PATH` | **Conteúdo** em `ZITADEL_KEY_JSON` + app lê da env | Service account key criada em Zitadel prod |
| `STRIPE_SECRET_KEY` | `sk_live_*` (Stripe Live Mode) | **Apenas em Railway**, nunca em dev |
| `STRIPE_WEBHOOK_SECRET` | whsec_* do endpoint Railway | Criar endpoint no Stripe Dashboard apontando para URL Railway |

**Regra dura**: `sk_live_*` **existe apenas no Railway Variables** da prod. Nunca em `.env` local, nunca em plano, nunca em log.

### Health checks

Railway pinga `GET /health` (retorna 200 sempre) e `GET /health/ready` (verifica DB + Redis).

### Domínio customizado (quando prod atingir)

- `api.mastersindico.com.br` → Railway custom domain
- ACM cert ou Let's Encrypt via Railway
- Coordenar com `auth.mastersindico.com.br` (Zitadel) para cookie compartilhado no domínio raiz `.mastersindico.com.br`

## AWS — Destino futuro

**Não implementado**. Plano de migração quando Railway atingir limite de scale ou quando compliance/regulatório exigir (LGPD + região específica):

| Serviço | AWS | Notas |
|---|---|---|
| Compute | ECS Fargate | Container image do Railway reusável |
| DB | RDS PostgreSQL 18 Multi-AZ | Import de snapshot do Railway |
| Cache | ElastiCache Redis 7 | |
| Storage | S3 + CloudFront | Assets públicos, signed URLs |
| Email | SES | |
| CDN | CloudFront | |
| DNS | Route53 | Transferência do domínio |
| Logs | CloudWatch | |
| Obs | CloudWatch + X-Ray (ou OTel → Jaeger) | Sprint 7 |

Migração é trabalho de **pós-Marco 3** — não planejado para o ciclo atual.

## CI/CD (Sprint 6)

Pipeline proposto:

```
PR aberto → CI (go build + vet + test + gosec + govulncheck + coverage)
Merge em main → deploy staging no Railway (branch deploy)
Tag v*.*.* → deploy prod no Railway (manual approval)
```

GitHub Actions workflow files em `.github/workflows/` — criar no Sprint 6 junto com AWS Foundation.

## Backup strategy

- **PostgreSQL**: snapshot diário automático do Railway (retention 7d em plan atual)
- **Config/secrets**: export mensal de `railway variables` → armazenar cifrado offline
- **Disaster recovery**: RTO 4h (recriar projeto Railway + import snapshot + deploy imagem Docker)

## Referências

- `steering/plugins.md` — plugin Railway + permissions
- `steering/mcp-integration.md` — MCPs de produção (apenas read-only)
- `design-assets.md` — Figma do frontend
- `.env.example` — template de env vars
