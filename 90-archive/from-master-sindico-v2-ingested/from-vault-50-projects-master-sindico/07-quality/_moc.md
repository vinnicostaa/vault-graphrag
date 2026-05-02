---
title: MOC — 07-quality
type: moc
tags: [moc, master-sindico, quality, clean-code, patterns]
project: master-sindico
created: 2026-04-22
---

# 07-quality — Qualidade de código, padrões aplicados

**Como** escrevemos código neste projeto. Aplica universais de [[../../../10-knowledge/_moc]] ao contexto Master Síndico.

## Notas canônicas

### Princípios
- [[CLEAN_CODE]] — princípios aplicados (naming, funções, comentários, classes, SRP) por stack (Go/TS/Dart)
- [[CLEAN_ARCH]] — Clean Architecture neste projeto: como cada camada fica no código Go real
- [[CODE_CALISTHENICS]] — 9 regras adaptadas a Go; exceções registradas com ratio
- [[PATTERNS]] — patterns aplicados (DDD, CQRS, SAGA, UoW, ABAC, Strategy, Factory, Specification) com exemplos do repo

### Checklists e guias
- [[code-review-checklist]] — o que conferir em PR review (além de tests)
- [[naming-guide]] — convenções específicas (prefixos, abreviações proibidas, ubiquitous language)
- [[logging-guide]] — como estruturar log (level, request_id, tenant_id, fields); PII proibido
- [[error-handling-guide]] — AppError, sentinels, ValidationError; quando não capturar
- [[api-design-guide]] — paths, status codes, payload shapes, headers, paginação cursor
- [[commit-pr-guide]] — mensagem imperativa, prefixo de domínio, Release Notes

### Anti-padrões
- [[antipatterns-applied]] — AP-001..AP-022 do catálogo global aplicados a este projeto; onde e como detectar

## Fontes

- [[../.claude/skills/go-review]]
- [[../specs/reference/antipatterns-to-avoid]]
- [[../../../10-knowledge/principles/code-calisthenics]]
- [[../../../10-knowledge/principles/do-dont-checklist]]
- [[../../../10-knowledge/architecture/clean-architecture-layers]]
- [[../../../10-knowledge/antipatterns/_moc]]
- [[../../../10-knowledge/patterns/_moc]]

## Vizinhos

- [[../_moc]]
- [[../02-architecture/_moc]]
- [[../08-security/_moc]]
- [[../09-testing/_moc]]
