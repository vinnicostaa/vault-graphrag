---
title: Behavioral Patterns (categoria GoF)
type: moc
tags:
  - moc
  - pattern
  - behavioral
  - gof
source: 'https://refactoring.guru/design-patterns/behavioral-patterns'
created: 2026-04-24T00:00:00.000Z
updated: 2026-04-24T00:00:00.000Z
aliases:
  - Behavioral Design Patterns
---

# Behavioral Patterns (categoria GoF)

Padrões comportamentais lidam com **algoritmos e atribuição de responsabilidades entre objetos**. Os 10 padrões clássicos da categoria GoF:

- [[chain-of-responsibility]] — passa request por cadeia de handlers; cada handler decide processar ou passar adiante.
- [[command]] — encapsula request como objeto com informação suficiente para passar, enfileirar, desfazer.
- [[iterator]] — percorre elementos de coleção sem expor representação interna.
- [[mediator]] — reduz dependências caóticas entre objetos forçando comunicação via objeto mediador.
- [[memento]] — salva e restaura estado anterior de objeto sem revelar implementação.
- [[observer]] — mecanismo de assinatura que notifica múltiplos objetos de eventos.
- [[state]] — objeto altera comportamento quando estado interno muda (parece trocar de classe).
- [[strategy]] — família de algoritmos intercambiáveis em runtime.
- [[template-method]] — esqueleto de algoritmo na superclasse; subclasses sobrescrevem partes específicas.
- [[visitor]] — separa algoritmos dos objetos em que operam.

## Fonte

- [Refactoring Guru — Behavioral Patterns](https://refactoring.guru/design-patterns/behavioral-patterns)
- Gamma, Helm, Johnson, Vlissides (1994). *Design Patterns*.

## Links

- [[_moc]]
- [[creational-patterns]]
- [[structural-patterns]]
- [[../_moc]] — knowledge raiz
