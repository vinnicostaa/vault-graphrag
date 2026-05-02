---
title: Railway — Integration Guide
type: note
tags: [provider, railway, integration-guide, deploy, cli]
source: https://docs.railway.com
source_date: 2026-04-24
created: 2026-04-24
updated: 2026-04-24
aliases: [Railway Integration, Railway Setup]
---

# Railway — Integration Guide

Passo-a-passo para subir um serviço do zero ao go-live.

## 1. Setup de conta + CLI

```bash
# instalar CLI (macOS/Linux via npm; outros via script install)
npm install -g @railway/cli

# login (abre browser)
railway login
```

## 2. Criar project + link ao repo

Via dashboard (railway.com/dashboard → "New Project" → "Deploy from GitHub"):
- Autorizar GitHub app.
- Selecionar repo.
- Railway detecta linguagem via Nixpacks.

Via CLI:
```bash
railway init                # cria project e linka ao cwd
railway link                # linka cwd a project existente
```

## 3. Env vars (Variables)

Dashboard → Service → Variables. Dois tipos:

- **Service-scoped**: específicas da service.
- **Shared (project-wide)**: disponíveis em todas services via reference `${{shared.VAR_NAME}}`.

Variables especiais auto-injetadas:
- `PORT` — port que o container deve escutar (dinâmico).
- `RAILWAY_ENVIRONMENT` — `production`/`staging`/…
- `RAILWAY_PROJECT_ID`, `RAILWAY_SERVICE_ID`
- `DATABASE_URL` (quando plugin Postgres linkado)
- `REDIS_URL` (quando plugin Redis linkado)

## 4. Deploy

### Via push (CD automático)

Default: cada push no branch configurado dispara deployment. Configurável via dashboard (`Settings → Service → Source → Branch`).

### Via CLI (deploy manual)

```bash
railway up                  # build + deploy do cwd
railway up --detach         # não streama logs
railway up --service <name> # se project tem múltiplos
```

### Dockerfile

Se `Dockerfile` existe na raiz, Railway usa ele em vez de Nixpacks. Útil para imagens custom (multi-stage, distroless).

## 5. Plugins (serviços gerenciados)

Adicionar Postgres:
```bash
# via CLI
railway add --plugin postgresql

# via dashboard: "+ New" → "Database" → "Postgres"
```

Após add, `DATABASE_URL` fica disponível como shared variable. Linka no service via:
```
DATABASE_URL=${{Postgres.DATABASE_URL}}
```

Redis:
```bash
railway add --plugin redis
# REDIS_URL → ${{Redis.REDIS_URL}}
```

## 6. Custom domain + SSL

Dashboard → Service → Settings → Domains → "Custom Domain". SSL via Let's Encrypt automático (ACME). TXT record de verificação + CNAME/ALIAS apontando para `<service>.up.railway.app`.

## 7. Cron jobs

Service type `cron` + cron expression. Exemplo via `railway.json`:

```json
{
  "deploy": {
    "cronSchedule": "0 3 * * *",
    "restartPolicyType": "NEVER"
  }
}
```

Executa 1×/dia às 03:00 UTC; exit 0 OK, exit ≠ 0 alerta.

## 8. Migrations antes do deploy

**Padrão recomendado**: rodar migrations como pré-deploy hook, não no startup da app (evita race entre múltiplas instâncias).

### Opção A — release command (via dashboard)

Settings → Deploy → Pre-deploy command: `goose -dir migrations postgres "$DATABASE_URL" up`.

### Opção B — CLI separado

```bash
railway run goose -dir migrations postgres "$DATABASE_URL" up
railway up
```

## 9. Go-live checklist

- [ ] Custom domain + SSL ativo.
- [ ] Env vars de production preenchidas (sem fallback para test).
- [ ] Healthcheck endpoint `/health` retornando 200 em < 5s.
- [ ] Graceful shutdown (SIGTERM handler).
- [ ] Migrations rodadas como pre-deploy.
- [ ] Plugin Postgres com backup habilitado (Settings → Backups).
- [ ] Sentry/error tracking conectado.
- [ ] Log export configurado se precisa agregador externo (Datadog/Grafana).
- [ ] Alerta de high-CPU/memory (via webhook integration).

## Links

- [[_moc]]
- [[../_moc]]
- [[../../../20-stacks/postgres/postgresql-conventions]]
- [[patterns]] — graceful shutdown, healthcheck, log aggregation
