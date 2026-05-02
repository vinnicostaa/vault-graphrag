---
title: MOC — Architecture
type: moc
tags: [moc, architecture]
created: 2026-04-22
updated: 2026-04-24
---

# Architecture

Padrões arquiteturais cross-linguagem: Clean Architecture, DDD, CQRS, Contract-First, Vertical Slices, Bounded Contexts.

## Notas

- [[clean-architecture-layers]] — as 4 camadas (Domain → Application → Infrastructure → Routes) com ordem canônica de implementação
- [[ddd-strategic-tactical]] — DDD estratégico (bounded context, context map, subdomain, ubiquitous language) + tático (entity, VO, aggregate, repository, domain event, factory, specification)
- [[cqrs]] — Command Query Responsibility Segregation (CQS → CQRS leve → read model materializado → Event Sourcing)
- [[openapi-contract-first]] — API contract-first com OpenAPI 3.1; versioning `/api/v1`; deprecation 6m; tooling (Spectral, Redocly, oapi-codegen) (D-vault-009)

## Canonical Five — Uncle Bob (D-vault-005, 2026-04-24)

Destilados do [The Clean Coder blog](https://blog.cleancoder.com/):

- [[the-clean-architecture]] — manifesto 2012: 4 círculos + Dependency Rule + independência de frameworks/DB/UI
- [[screaming-architecture]] — arquitetura deve gritar **domínio**, não framework
- [[a-little-architecture]] — diálogo socrático: o arquiteto decide o que permite **deferir** DB/framework/webserver; ISP + DIP
- [[framework-bound]] — frameworks são ferramentas, não modos de vida; mantê-los a braços de distância
- [[no-db]] — DB é detalhe; use cases no centro; data model decidido depois de use cases testados

## A criar (conforme necessidade)

- hexagonal-vs-clean — diferenças e quando escolher cada
- vertical-slices — slice por feature vs slice por camada
- dependency-rule — regra de dependência e como testar violação (pode ser consolidado dentro de [[the-clean-architecture]])
- event-driven-architecture — pub/sub, choreography vs orchestration, eventual consistency
- microservices-vs-modular-monolith — trade-offs, quando dividir

## Vizinhos

- [[../_moc]]
- [[../patterns/_moc]] — patterns concretos (22 GoF + transaction-patterns); [[../patterns/adapter]] é central ao Clean Architecture
- [[../principles/_moc]] — SOLID (motor do DIP na Clean Arch), clean-code, code-calisthenics
- [[../../20-stacks/_moc]] — implementação por linguagem
- [[../methodology/sdd-workflow]] — processo que produz estas arquiteturas
- [[../../00-meta/STATE]] — D-vault-005 (Canonical Five) + D-vault-009 (openapi-contract-first)
