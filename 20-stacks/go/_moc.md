---
title: MOC — Go
type: moc
tags: [moc, go, stack]
created: 2026-04-22
---

# Go — stack

Aplicação concreta de [[../../10-knowledge/_moc|padrões atemporais]] em Go 1.26+. Escolha canônica do backend do projeto automation-agents e do cliente Master Síndico.

## Notas

- [[go-patterns]] — regras duras, estrutura de entidade/VO/use case/handler/provider, error handling, compile-time checks

## A criar (conforme necessidade)

- go-testing — Testify + rapid (PBT) + testcontainers-go
- go-concurrency — goroutines, channels, context cancellation
- go-idioms — quirks específicos (ctx pgx, SetGlobalLevel, interface checks)
- go-middleware-stack — implementação concreta do BeyondCorp stack

## Vizinhos

- [[../_moc]] — stacks
- [[../postgres/_moc]] *(não criado ainda, equivalente em `../postgres/_moc`)* → [[../postgres/postgresql-conventions]]
- [[../../10-knowledge/principles/code-calisthenics]] — 9 regras com exemplos Go
- [[../../10-knowledge/patterns/transaction-patterns]] — UoW com pgx.BeginTxFunc
- [[../../10-knowledge/architecture/clean-architecture-layers]] — as 4 camadas aplicadas em Go
