---
title: MOC — Railway (PaaS deploy + Postgres + Redis)
type: moc
tags: [moc, provider, railway, paas, deploy, infra, postgres, redis]
category: paas
mcp: null
sdk-official-cli: "@railway/cli"
doc-oficial: https://docs.railway.com
doc-consulted: 2026-04-23
created: 2026-04-22
updated: 2026-04-23
---

# Railway

**Categoria**: PaaS (Platform-as-a-Service) · deploy container + serviços gerenciados (Postgres, Redis, MySQL, Mongo).

**MCP disponível**: ❌ — usar WebFetch em https://docs.railway.com ou Context7 `resolve-library-id "railway"`.

**CLI oficial**: `@railway/cli` (`railway login`, `railway up`, `railway logs`).

**Doc oficial**: https://docs.railway.com — consultada em 2026-04-23.

**Modelo de preço**: pay-per-resource (RAM, CPU, egress). Sem free tier permanente; trial credit.

---

## Quando usar Railway

1. **Deploy rápido com Postgres/Redis gerenciados** (zero ops, bom fit Go/Node)
2. **Preview deploys por branch** (automático em PR)
3. **Migration de Heroku refugees** (ergonomia similar, sem custo absurdo)
4. **Pequena/média startup** — trade-off: ergonomia > controle fino
5. **Time sem DevOps dedicado**

## Quando NÃO usar Railway

- **Latência global crítica** → Fly.io (edge deploy) vence
- **Compliance strict (LGPD/HIPAA em região específica)** → AWS/GCP com ACL fino
- **Escala > $5k/mês** → AWS ECS/EKS é mais barato
- **Postgres > 100GB ou alto RPS** → RDS/Aiven/Crunchy são melhores

---

## Sub-docs do Railway

- [[overview]] — projects, services, environments, plugins
- [[integration-guide]] — deploy Go/Node, migrations, env vars, domain custom
- [[patterns]] — healthcheck correto, graceful shutdown, log aggregation, cron jobs
- [[antipatterns]] — ex: secrets no git; usar `PORT` hardcoded; containers sem HEALTHCHECK
- [[usage-in-projects]] — quais projetos

## Glossário interno do Railway

- **Project** — raiz; tem environments (default: `production`) e services.
- **Service** — aplicação (container) ou plugin (postgres, redis, etc.).
- **Environment** — ambiente (production/staging); vars e services podem diferir.
- **Deployment** — build + start de um service; cada push gera uma deployment.
- **Plugin** — serviço gerenciado (`Postgres`, `Redis`, `MySQL`, `MongoDB`); dá DATABASE_URL etc. via shared vars.
- **Private networking** — services do mesmo project se comunicam via `service-name.railway.internal` (sem egress billing).

## Fontes brutas (pendente destilar)

> **Estado**: destilação completa em [[overview]], [[integration-guide]], [[patterns]], [[antipatterns]] (D-vault-009, recuperação A-vault-042). Uso por projeto (incluindo referência a infraestrutura específica) foi movido para [[usage-in-projects]] conforme §11 do [[../../../CLAUDE]] (conteúdo específico de cliente não pertence ao _moc atemporal).

## Vizinhos no grafo

- [[../_moc]]
- [[../../../20-stacks/postgres/_moc]] — Postgres idioms (Railway gerencia, mas query patterns são universais)
- [[../../observability/_moc]] — Railway tem logs nativos; integração com Grafana/Datadog manual
