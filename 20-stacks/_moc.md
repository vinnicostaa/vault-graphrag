---
title: MOC — 20-stacks
type: moc
tags: [moc, stacks]
created: 2026-04-22
---

# 20-stacks — Templates por stack

Aplicação concreta do conhecimento de [[../10-knowledge/_moc]] em cada stack tecnológico. Cada sub-pasta é auto-contida.

## Stacks disponíveis

- [[go/_moc]] — Go 1.26+, tipagem forte, Clean Arch backend, sqlc, pgx, OTel
- [[typescript/_moc]] — TypeScript strict, Node 20+, Fastify/NestJS, Zod, Drizzle/Prisma
- [[rust/_moc]] — Rust (futuro — para quando migrar daemons críticos)
- [[flutter/_moc]] — Flutter/Dart mobile, Clean Arch + Feature First, BLoC, get_it+injectable
- [[postgres/_moc]] — PostgreSQL 18+, schemas, migrations, indexes, tipos corretos

## Template de novo stack

Ao adicionar stack novo (ex: Kotlin, Python, Swift):

1. Criar `20-stacks/<nome>/_moc.md` com seção "Convenções do stack"
2. Criar notas específicas: `<nome>/patterns.md`, `<nome>/testing.md`, `<nome>/error-handling.md`
3. Linkar pros equivalentes em [[../10-knowledge/patterns/_moc]] (ex: `Strategy em Go` → `Strategy genérico`)

## Vizinhos no grafo

- [[../_index]] — raiz
- [[../10-knowledge/_moc]] — teoria que informa os templates daqui
- [[../40-templates/_moc]] — templates de projeto completo (CLAUDE/AGENTS/README_SPEC) cruzam com este
