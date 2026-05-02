---
title: MOC — 02-architecture
type: moc
tags: [moc, master-sindico, v2, architecture]
created: 2026-04-23
updated: 2026-04-23
---

# MOC — 02-architecture

Arquitetura canônica do Master Síndico v2. C4 (contexts, containers, components), Clean Arch mapping, patterns, topologia multi-tenant, API design, event-driven, observability, ADRs e research-inspirations.

> Conteúdo escrito na Fase 3 do plano em `/home/vinni/.claude/plans/squishy-drifting-avalanche.md` com herança direta dos destilados [[../13-research/**/_destilado]] e do legado em `90-ingested/from-vault-50-projects-master-sindico/02-architecture` *(arquivado)*.

---

## Arquivos

### Visão macro (C4)

- [[c4-context]] — Nível 1 — atores + sistema + integrações externas
- [[c4-containers]] — Nível 2 — backend API, worker, web, mobile, PG, Redis, OpenSearch, NATS
- [[c4-components]] — Nível 3 — componentes por BC (identity, billing, institutional, commercial, content, assembly, cross-domain)

### Princípios estruturantes

- [[clean-arch-mapping]] — Clean Architecture aplicada: camadas, regras de dependência, exemplos
- [[patterns]] — padrões aplicados: DDD, Repository, Factory, VO, CQRS, UoW, Saga, Strategy, Specification, ACL, Idempotency, Circuit Breaker, Retry+DLQ, Outbox, Observer
- [[topology-multitenant]] — multi-tenant row-based + PK composto + RLS + subdomain routing

### Interfaces

- [[api-design]] — REST + OpenAPI 3.1, versioning /v1, envelope {data, meta} + {error}, cursor pagination, WebSocket, webhook inbound HMAC
- [[event-driven]] — Outbox pattern, NATS JetStream critério 3x3, schema canônico, idempotência, DLQ, Saga orchestration

### Operação

- [[observability]] — OTel + Grafana + Sentry, SLI/SLO 99.5% M1, error budget, postmortem blameless, burn rate alerting

### Síntese do research

- [[research-inspirations]] — mapa princípio → origem → aplicação → marco (BeyondCorp, Google SRE, Netflix, YouTube, Meet, TikTok, Instagram, LinkedIn)

### ADRs

- [[adr/_moc]] — índice das 25 ADRs (0001–0025)

---

## ADRs por categoria (atalho)

- **Fundamentos**: [[adr/0001-clean-architecture-ddd-cqrs]] · [[adr/0002-gin-abstracted-router]] · [[adr/0015-uow-intra-saga-inter]]
- **Persistência**: [[adr/0005-sqlc-typed-queries]] · [[adr/0006-pgx-v5]] · [[adr/0007-goose-migrations-partitioned]] · [[adr/0009-uuidv7-ulid-pk]] · [[adr/0018-opensearch-tsvector-dev]] · [[adr/0021-multi-tenant-row-based]]
- **Segurança**: [[adr/0003-zitadel-oidc-provider]] · [[adr/0012-abac-redis-cache]] · [[adr/0014-one-device-policy]] · [[adr/0016-beyondcorp-security-posture]]
- **Providers**: [[adr/0004-stripe-payment-gateway]] · [[adr/0010-mux-video-provider]] · [[adr/0011-livekit-sfu]] · [[adr/0022-circuit-breaker-gobreaker]]
- **Domínio**: [[adr/0013-timeline-immutable]] · [[adr/0023-recsys-deterministic-m1]] · [[adr/0024-moderation-cascade]]
- **Mensageria**: [[adr/0019-nats-jetstream-threshold]]
- **Infra & Obs**: [[adr/0008-zerolog-structured-logs]] · [[adr/0017-railway-initial-aws-m4]] · [[adr/0020-opentelemetry-grafana-sentry]] · [[adr/0025-sli-slo-error-budget]]

---

## Vizinhos

- [[../_moc]] — MOC raiz da v2
- [[../CLAUDE]] — contrato do agente
- [[../STATE]] — decisões vivas (D-###)
- [[../AUDIT]] — findings (A-###)
- [[../STEERING]] — não-negociáveis
- [[../01-domain/_moc]] — bounded contexts, ubiquitous language
- [[../13-research/_moc]] — research destilado
- [[../90-ingested/_moc]] — material legado
