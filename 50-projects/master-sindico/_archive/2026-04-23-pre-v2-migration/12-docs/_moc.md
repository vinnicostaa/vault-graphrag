---
title: MOC — 12-docs
type: moc
tags: [moc, master-sindico, docs]
project: master-sindico
created: 2026-04-22
---

# 12-docs — Documentação navegável

Docs voltadas pra **leitor humano** (dev, stakeholder, suporte). README + how-to + runbooks + ADR index + changelog.

## Notas canônicas

- [[README]] — entry point; links pra CLAUDE, ROADMAP, sub-projetos
- [[how-to/_moc]] — guias "como fazer X"
- [[runbooks/_moc]] — runbooks operacionais ("quando Y acontece, faça Z")
- [[adr-index]] — índice dos ADRs em [[../02-architecture/adr/_moc]]
- [[changelog]] — histórico de releases do projeto
- [[glossary]] — glossário resumido (link pra [[../01-domain/ubiquitous-language]])
- [[api-docs]] — como consumir a API (link pra [[../02-architecture/openapi/_moc]])
- [[contributing]] — como contribuir (link pra [[../CLAUDE]] e [[../DoR-DoD]])
- [[support]] — como pedir suporte, SLA, canais

## How-tos sugeridos (a criar)

- `how-to/setup-dev-env.md` — rodar docker-compose + .env + hook stripe CLI
- `how-to/criar-modulo.md` — scaffold de novo bounded context
- `how-to/criar-use-case.md` — CQRS command/query por domínio
- `how-to/adicionar-webhook.md` — Stripe/Mux/integração externa
- `how-to/fazer-migration.md` — goose + expand-contract
- `how-to/rodar-testes-local.md` — unit + integration + coverage
- `how-to/deploy-staging.md` — Railway staging
- `how-to/rollback-producao.md` — [[../06-execution/playbooks/rollback]]

## Runbooks sugeridos (a criar)

- `runbooks/db-down.md` — Postgres indisponível
- `runbooks/redis-down.md` — cache indisponível (graceful degradation)
- `runbooks/zitadel-down.md` — auth indisponível
- `runbooks/stripe-webhook-fail.md` — idempotência + replay
- `runbooks/mux-webhook-fail.md` — retry + reconciliação
- `runbooks/livekit-incident.md` — falha em assembleia ao vivo
- `runbooks/cve-critical.md` — bypass do fluxo normal pra hotfix

## Vizinhos

- [[../_moc]]
- [[../CLAUDE]]
- [[../ROADMAP]]
- [[../06-execution/_moc]]
- [[../02-architecture/adr/_moc]]
