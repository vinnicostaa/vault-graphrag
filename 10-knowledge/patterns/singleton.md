---
title: Singleton
type: pattern
tags: [pattern, creational, gof, thread-safety]
source: https://refactoring.guru/design-patterns/singleton
created: 2026-04-24
updated: 2026-04-24
aliases: [Singleton Pattern]
---

# Singleton

Padrão **criacional** que garante que uma classe tem **uma única instância** e fornece um **ponto global de acesso** controlado a ela.

## Problema que resolve

Dois problemas comuns aparecem juntos:

1. **Controlar acesso a recurso compartilhado** — conexão de banco, arquivo de configuração, cache global. Construtor normal sempre retorna objeto novo; não serve quando a instância precisa ser única.
2. **Fornecer ponto global de acesso** que não possa ser sobrescrito por código cliente. Variável global crua aceita qualquer reatribuição, o que torna o estado imprevisível.

O pattern endereça os dois ao mesmo tempo — combinação útil na prática, mas fonte de uma das principais críticas (ver Trade-offs).

## Solução

- Construtor da classe é `private` (ou equivalente na linguagem).
- Método estático `getInstance()` cria a instância na primeira chamada e retorna a mesma em chamadas subsequentes (*lazy initialization*).
- Em ambiente multi-thread, tratamento adicional é necessário para evitar duas instâncias criadas em corrida — técnica comum é *double-checked locking*.

## Estrutura

- **Singleton** — classe com campo estático privado `instance` + método estático público `getInstance()` + construtor privado.
- **Client** — nunca usa `new Singleton()`; sempre chama `Singleton.getInstance()`.

## Pseudocódigo

```
class Database is
    private static field instance: Database

    private constructor Database() is
        // inicialização, conexão ao DB

    public static method getInstance() is
        if (Database.instance == null) then
            acquireThreadLock() and then
                // double-checked locking — evita corrida entre threads
                if (Database.instance == null) then
                    Database.instance = new Database()
        return Database.instance

    public method query(sql) is
        // operação de domínio
```

## Quando usar

- Programa precisa **exatamente uma** instância de uma classe — ex.: conexão DB compartilhada, logger, registry de configuração.
- Precisa controle mais estrito sobre variável global — Singleton garante não-sobrescrita por outro código.

## Quando evitar

- **Testes ficam difíceis** — frameworks de mock dependem de herança; construtor privado + método estático bloqueia. Alternativa: injeção de dependência com interface + factory.
- **Estado mutável global** — efeito colateral implícito, difícil de rastrear em debugging.
- **Mascarar alto acoplamento** — vários componentes acessando o mesmo Singleton costuma revelar dependência escondida que deveria ser injetada explicitamente.

## Sinais de que você está reinventando este pattern

- Classe com campo estático guardando "a única X" + método que retorna essa instância (criando-a se não existir).
- Módulo com variável *top-level* que "começa `nil`/`null`" e é preenchida na primeira chamada.
- "Container" de configuração que é instância única espalhada por toda a aplicação.

## Trade-offs

### Vantagens
- Garantia de instância única com ponto de acesso controlado.
- Lazy initialization — instância criada só quando necessária.

### Desvantagens
- **Viola Single Responsibility Principle** — o pattern controla **ciclo de vida** da classe **e** a sua lógica de domínio.
- Mocking em testes exige truques (construtor privado + método estático).
- Requer tratamento especial em ambiente multi-thread.
- Pode mascarar alto acoplamento entre componentes.

## Relações com outros patterns

- [[facade]] — uma Facade com estado único frequentemente vira Singleton (um objeto de fachada basta).
- [[flyweight]] — parece Singleton, mas: (1) Flyweight permite múltiplas instâncias com estados intrínsecos distintos; (2) Flyweight é imutável, Singleton pode ser mutável.
- [[abstract-factory]], [[builder]], [[prototype]] — os três podem ser implementados como Singleton.

## Fontes

- [Refactoring Guru — Singleton](https://refactoring.guru/design-patterns/singleton)
- Gamma, Helm, Johnson, Vlissides (1994). *Design Patterns: Elements of Reusable Object-Oriented Software*. Addison-Wesley.

## Links

- [[_moc]]
- [[creational-patterns]]
- [[../principles/solid]] — SRP violado por este pattern
- [[../_moc]] — knowledge raiz
