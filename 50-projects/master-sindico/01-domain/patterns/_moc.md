---
title: MOC — 01-domain/patterns
type: moc
tags: [moc, master-sindico, domain, patterns, ddd]
created: 2026-04-24
---

# MOC — 01-domain/patterns

Patterns específicos do domínio Master Síndico (aplicações concretas de DDD + interfaces cross-BC). Para patterns atemporais genéricos, ver [[../../../../10-knowledge/patterns/_moc]] do vault global.

## Patterns

- [[validatable-interface]] — IValidatable cross-BC (Opção B D-105). 5 aggregates commercial implementam; hub institutional consome via view materializada ou endpoint agregador.

## A criar (conforme demanda)

- `identity-mirror-pattern.md` — padrão Zitadel ↔ DB user (referenciado em backend/.kiro — migrar de lá)
- `event-driven-integration.md` — NATS JetStream + InMemoryPublisher threshold
- `tenant-isolation-row-based.md` — RLS + TenantBoundaryGuard

## Vizinhos

- [[../aggregates/_moc]]
- [[../invariants]]
- [[../_moc]]
- [[../../../02-architecture/_moc]]
