---
title: MOC — 02-architecture
type: moc
tags: [moc, master-sindico, architecture]
project: master-sindico
created: 2026-04-22
---

# 02-architecture — Arquitetura de sistema

**Como** o produto é construído. Clean Architecture + DDD + CQRS + BeyondCorp adaptado.

## Notas canônicas

### Visão
- [[system-overview]] — diagrama de alto nível + fluxo de request
- [[deployment-topology]] — Railway atual → ECS plano
- [[data-flow]] — sequência request→auth→abac→use case→db
- [[tech-stack-matrix]] — detalhado por camada

### Camadas e padrões
- [[clean-arch-mapping]] — como `infrastructure → application → domain` aterrissa no código Go
- [[cqrs-layout]] — commands vs queries; arquivo por use case
- [[modules-layout]] — estrutura `modules/<ctx>/{domain,application,infrastructure}`
- [[shared-providers]] — config, middleware, auth, cache, database, types
- [[adapter-layer]] — `contracts.HTTPRouter` isola Gin; `IPaymentGateway`, `IVideoProvider`, etc.

### Contratos
- [[openapi/_moc]] — OpenAPI 3.1 por módulo
- [[event-contracts]] — domain events publicados e consumidos
- [[webhook-contracts]] — Stripe, Mux
- [[error-contracts]] — AppError, ValidationError, códigos HTTP canônicos

### Infra
- [[infrastructure]] — Railway, PG18, Redis7, Zitadel, NATS, S3, CloudFront
- [[database-design]] — schema overview, migrations per module range
- [[cache-strategy]] — Redis TTLs, invalidação (ABAC 5min via webhook Stripe)
- [[observability-stack]] — OTel + Grafana + Sentry

### ADRs
- [[adr/_moc]] — Architecture Decision Records (NNNN-slug.md)

## Fontes

- [[../client-vault/01 - Arquitetura de Sistema/System Architecture]] — v4.0 (Mar/2026, parcialmente desatualizada — Ent/Casbin/Uber-FX **não** adotados)
- [[../client-vault/01 - Arquitetura de Sistema/System Architecture - API Design]]
- [[../client-vault/01 - Arquitetura de Sistema/System Architecture - Data Model]]
- [[../client-vault/01 - Arquitetura de Sistema/System Architecture - Infrastructure]]
- [[../client-vault/01 - Arquitetura de Sistema/System Architecture - Identity]] · Institutional · Commercial · Content · Assembly
- [[../client-vault/01 - Arquitetura de Sistema/Arquitetura do Ecossistema]]
- [[../client-vault/01 - Arquitetura de Sistema/Decisões e Pendências]]
- [[../specs/design]] + [[../specs/design/README]]
- [[../plans/architecture-refined]]
- [[../plans/master-sindico-implementation-plan]]

## Vizinhos

- [[../_moc]]
- [[../01-domain/_moc]]
- [[../03-subprojects/_moc]]
- [[../06-execution/_moc]]
- [[../07-quality/_moc]]
- [[../08-security/_moc]]
- [[../../../10-knowledge/architecture/_moc]]
