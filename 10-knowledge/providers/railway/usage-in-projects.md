---
title: Railway — Usage Across Projects
type: note
tags: [provider, railway, cross-project]
created: 2026-04-24
updated: 2026-04-24
---

# Railway — Usage Across Projects

Tabela de projetos do vault que usam Railway + link para a aplicação concreta.

| Projeto | Uso | Aplicado em |
|---|---|---|
| [[../../../50-projects/master-sindico/CLAUDE\|Master Síndico]] | Deploy backend Go + Postgres 18 + Redis 7 + cron jobs (scheduler noturno). Substituiu AWS ECS (D-067 do legado). Private networking para reduzir egress entre api + worker. | [[../../../50-projects/master-sindico/08-security/BEYOND_CORP]] (deploy + compliance) · [[../../../50-projects/master-sindico/STATE]] (D-067 reconciliação stack) |

## Padrão de integração observado

- Backend Go via Nixpacks (Dockerfile não necessário para Go padrão).
- Plugin Postgres 18 com backup diário habilitado.
- Plugin Redis 7 para cache ABAC + rate limit.
- Cron job service separado para scheduler noturno (marcar `connect_me_request` expired, `master_plan_activity` atrasada, fechar polls).
- Private networking: api ↔ worker via `worker.railway.internal`.
- Preview deploy habilitado por PR.
- Logs JSON via `zerolog` → exportado para Grafana Cloud via webhook integration.

## Quando criar novo projeto com Railway

1. Ler [[_moc]] primeiro (visão geral + preço).
2. Ler [[integration-guide]] (setup end-to-end).
3. Ler [[patterns]] antes de escrever healthcheck/shutdown.
4. Ler [[antipatterns]] para evitar armadilhas comuns.

## Links

- [[_moc]]
- [[../_moc]]
- [[integration-guide]]
- [[patterns]]
- [[antipatterns]]
