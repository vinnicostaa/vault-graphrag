---
title: Railway — Overview
type: note
tags: [provider, railway, paas, overview, deploy]
source: https://docs.railway.com
source_date: 2026-04-24
created: 2026-04-24
updated: 2026-04-24
aliases: [Railway Overview]
---

# Railway — Overview

Railway é uma **PaaS** (Platform-as-a-Service) para deploy de containers + serviços gerenciados (Postgres, Redis, MySQL, Mongo). Origem: 2021, como alternativa ao Heroku pós-removal do free tier. Foco: **ergonomia de deploy** para startups e times pequenos.

## Hierarquia de recursos

```
Project (raiz)
├── Environments (production / staging / preview / …)
│   ├── Services (app containers ou plugins)
│   │   ├── Deployments (versões)
│   │   └── Variables (env vars)
│   └── Shared Variables (project-wide)
```

- **Project**: raiz. Tem múltiplos environments e services.
- **Environment**: isolamento lógico; `production` é default. Cada env tem suas próprias variables e deployments.
- **Service**: unidade deployável. Pode ser *app service* (seu código) ou *plugin* (serviço gerenciado).
- **Deployment**: build + start de um service; cada push gera nova deployment.
- **Plugin**: serviço gerenciado (`Postgres`, `Redis`, `MySQL`, `MongoDB`). Expõe connection string via shared variable (ex.: `DATABASE_URL`).

## Build system — Nixpacks (padrão) + Dockerfile

- **Nixpacks** (default): detecta linguagem/framework automaticamente, constrói imagem. Suporta Go, Node, Python, Ruby, Rust, Java, PHP, Elixir, Deno.
- **Dockerfile**: se presente na raiz, sobrescreve Nixpacks. Recomendado para controle fino.

## Private networking

Services do mesmo project comunicam via `<service-name>.railway.internal` (IPv6 privado). Zero egress billing — traffic permanece em rede interna. Crítico para reduzir custo em arquiteturas multi-service.

## Preview deploys

Cada PR no GitHub pode disparar **environment ephemeral** (PR → preview env). URL pública temporária; destruída ao merge. Gate natural de CI visual.

## Modelo de preço

Pay-per-resource:
- RAM: ~$0.000231/GB-hora
- CPU: ~$0.000463/vCPU-hora
- Egress: $0.10/GB (público)
- Plugins: ajustável conforme tamanho

Sem free tier permanente (há trial credit de $5 no signup). Trade-off: previsível mas requer atenção a recursos ociosos.

## Concorrentes — quando cada vence

| Concorrente | Quando vence |
|---|---|
| **Fly.io** | Latência global crítica (edge deploy em múltiplas regiões) |
| **Render** | Similar a Railway; Render tem tier free mais generoso para estáticos |
| **Heroku** | Ecossistema maduro, 1-click addons de 3rd party |
| **AWS ECS/EKS** | Escala > $5k/mês, compliance fino, controle IAM granular |
| **Vercel** | Frontend Next.js-centric com ISR/ edge functions |

## Quando **usar** Railway

1. Deploy rápido com Postgres/Redis gerenciados (startup/MVP).
2. Equipe sem DevOps dedicado.
3. Preview deploys automáticos por PR.
4. Migração de Heroku sem explodir custo.
5. Arquitetura multi-service com private networking.

## Quando **não** usar

- Latência global < 50ms em múltiplos continentes.
- Compliance strict (LGPD/HIPAA em região específica — Railway roda em us-west por padrão).
- Escala > $5k/mês sem tuning.
- Postgres > 100GB ou alto RPS (RDS/Aiven/Crunchy vencem).

## Links

- [[_moc]]
- [[../_moc]]
- [[../../../20-stacks/postgres/postgresql-conventions]] — Postgres idioms (Railway gerencia, mas queries são universais)
