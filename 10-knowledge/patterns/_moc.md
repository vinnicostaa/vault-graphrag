---
title: MOC — Patterns
type: moc
tags: [moc, patterns, gof]
created: 2026-04-22
updated: 2026-04-24
---

# Patterns

Padrões de projeto cross-linguagem. Catálogo **GoF completo** (22 padrões em 3 categorias) + padrões arquiteturais do vault (transaction, integration, concurrency).

## GoF — catálogo clássico (destilado em 2026-04-24, D-vault-001)

Ver tag `gof` para Dataview query: `WHERE contains(tags, "gof")`.

### Creational (5) — [[creational-patterns]]

- [[factory-method]] — interface para criar objetos; subclasses decidem tipo concreto
- [[abstract-factory]] — famílias de objetos relacionados sem acoplar ao tipo concreto
- [[builder]] — objetos complexos passo a passo; variantes via mesmo código de construção
- [[prototype]] — copia objetos existentes sem acoplar ao tipo concreto
- [[singleton]] — instância única + ponto global de acesso controlado

### Structural (7) — [[structural-patterns]]

- [[adapter]] — interfaces incompatíveis colaboram via tradutor
- [[bridge]] — separa em duas hierarquias (abstração + implementação) que evoluem independentes
- [[composite]] — árvore de objetos tratada uniformemente como objeto único
- [[decorator]] — anexa comportamentos envolvendo objetos em wrappers
- [[facade]] — interface simplificada para subsistema complexo
- [[flyweight]] — compartilha estado comum entre objetos para reduzir memória
- [[proxy]] — substituto que controla acesso ao objeto original

### Behavioral (10) — [[behavioral-patterns]]

- [[chain-of-responsibility]] — request passa por cadeia de handlers
- [[command]] — request vira objeto (fila, histórico, undo)
- [[iterator]] — percorre coleção sem expor representação interna
- [[mediator]] — reduz dependências caóticas via objeto mediador
- [[memento]] — salva/restaura estado anterior sem revelar implementação
- [[observer]] — assinatura pub-sub para notificar múltiplos objetos
- [[state]] — objeto altera comportamento quando estado interno muda
- [[strategy]] — família de algoritmos intercambiáveis em runtime
- [[template-method]] — esqueleto de algoritmo na superclasse; subclasses sobrescrevem partes
- [[visitor]] — separa algoritmos dos objetos em que operam

## Padrões arquiteturais do vault (não-GoF)

- [[transaction-patterns]] — ACID via UnitOfWork, Saga orchestrada/choreography, domain events, locks explícitos (`SELECT ... FOR UPDATE`), advisory locks

## Roadmap (a criar conforme necessidade)

- value-object — primitivo com invariantes
- module-pattern — `Register(router, container)` + `Bootstrap(container)`
- unit-of-work — detalhes de `Run(ctx, fn)` com tx
- domain-events — publicação + handlers idempotent
- dispatcher-pattern — webhook com `switch event.type`
- provider-adapter — interface no domínio, impl em infra (aplicação concreta de [[adapter]])
- circuit-breaker — fallback quando provider externo cai
- retry-with-backoff — exponential + DLQ

## Vizinhos

- [[../_moc]]
- [[../antipatterns/_moc]] — AP-### que cada pattern resolve
- [[../principles/_moc|principles]] — SOLID, DRY, YAGNI aplicados
- [[../../20-stacks/_moc]] — implementação por linguagem
- [[../../00-meta/STATE]] — D-vault-001 (template canônico + tag `gof`)
