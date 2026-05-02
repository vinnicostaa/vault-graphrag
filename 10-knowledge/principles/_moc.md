---
title: MOC — Principles
type: moc
tags: [moc, principles]
created: 2026-04-22
updated: 2026-04-24
---

# Principles

Princípios atemporais de design, código e ergonomia. **Linguagem-agnósticos**. Implementações específicas por stack vivem em `20-stacks/<stack>/`.

## Notas

- [[solid]] — os 5 princípios (SRP, OCP, LSP, ISP, DIP) com exemplos Go
- [[clean-code]] — 17 princípios canônicos destilados de Uncle Bob (naming, funções pequenas, error handling, boundaries, testes)
- [[code-calisthenics]] — 9 regras Jeff Bay (indent único, sem else, wrap primitivos, first-class collections, Demeter, sem abreviação, entidades pequenas, sem getters/setters)
- [[do-dont-checklist]] — regras duras cross-linguagem consolidadas
- [[no-defensive-try-catch]] — capturar só com ação útil; caso padrão é propagar; wrap com contexto em Go/Rust/TS
- [[no-if-else-hell]] — aninhamento ≥ 3 níveis pede extração; early return, strategy, dispatch, polimorfismo
- [[no-premature-abstraction]] — YAGNI + Rule of Three; abstrair com 2+ implementações reais; speculative generality (AP-012)

## A criar (conforme necessidade)

- dry-kiss-yagni — princípios complementares curtos
- law-of-demeter — detalhamento com exemplos de acoplamento
- fail-fast — guard clauses, invariantes garantidas
- tell-dont-ask — o par de Demeter, encapsulamento comportamental

## Vizinhos

- [[../_moc]]
- [[../architecture/_moc]] — Clean Architecture, DDD, CQRS (princípios aplicados em arquitetura)
- [[../antipatterns/_moc]] — catálogo AP-### (o que viola estes princípios)
- [[../methodology/_moc]] — SDD, GSD, TDD (onde aplicar)
- [[../testing/_moc]] — testes como encarnação dos princípios (SRP, boundaries)
- [[../observability/_moc]] — logs estruturados refletem clean code
- [[../../20-stacks/_moc]] — exemplos concretos por linguagem
