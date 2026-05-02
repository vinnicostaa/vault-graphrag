---
title: The Clean Architecture
type: concept
tags: [architecture, clean-architecture, dependency-rule, use-cases, uncle-bob]
source: https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html
source_date: 2012-08-13
created: 2026-04-24
updated: 2026-04-24
aliases: [Clean Architecture, The Clean Architecture 2012]
---

# The Clean Architecture

Destilado do manifesto original de Robert C. Martin (Uncle Bob), 2012. Integra em um único diagrama acionável as ideias de **Hexagonal Architecture** (Cockburn), **Onion Architecture** (Palermo), **Screaming Architecture**, **DCI** (Coplien/Reenskaug) e **BCE** (Jacobson). Todas convergem para o mesmo objetivo: **separação de responsabilidades via camadas**.

Ver aplicação concreta em [[clean-architecture-layers]].

## Os 4 círculos concêntricos (schemáticos, não rígidos)

Do mais externo para o mais interno:

1. **Frameworks & Drivers** — DB, web framework, UI toolkit. Código de cola, detalhes "externos".
2. **Interface Adapters** — apresentadores, controladores, views (MVC); conversores DB ↔ use cases; gateways de serviços externos.
3. **Use Cases** — regras de negócio *application-specific*; orquestram entidades para atingir um goal do use case.
4. **Entities** — regras de negócio *enterprise-wide* (ou business objects da aplicação se não há enterprise). Nível mais alto, menos volátil.

Não é obrigatório ter exatamente 4 — você pode precisar mais. O que **não** muda é a regra de dependência.

## The Dependency Rule

> *Source code dependencies* can only point **inwards**.

- Nada em um círculo interno menciona nada de um círculo externo.
- Data formats do círculo externo **não** aparecem no interno.
- Nomes declarados externamente não entram no código interno — nem funções, nem classes, nem variáveis, nem qualquer entidade nomeada.

Consequência: o domínio **não sabe** que existe framework, DB, web. As camadas externas podem ser trocadas sem tocar o núcleo.

## Propriedades garantidas

Quando a Dependency Rule é respeitada, o sistema se torna:

1. **Independente de frameworks** — framework vira ferramenta, não restrição.
2. **Testável** — business rules testadas sem UI, DB, web server, nenhuma dependência externa.
3. **Independente de UI** — trocar web UI por console não muda o núcleo.
4. **Independente de DB** — Oracle para Mongo, CouchDB, flat files — business rules não dependem.
5. **Independente de qualquer agência externa** — o núcleo simplesmente não conhece o mundo de fora.

## Cruzando fronteiras (crossing boundaries)

O caso clássico: o use case precisa chamar o presenter, mas a Dependency Rule proíbe mencionar o presenter.

**Solução — Dependency Inversion Principle**:

- Use case declara uma interface (ex.: `UseCaseOutputPort`) **no seu próprio círculo**.
- Presenter, no círculo externo, **implementa** essa interface.
- Em runtime, o use case chama `outputPort.present(...)` — fluxo de controle **sai** do círculo interno.
- Em compile-time, a fonte do presenter depende da fonte do use case — dependência **entra** (conforme regra).

Polimorfismo dinâmico faz source-code-dependencies se oporem ao flow-of-control.

## Dados que cruzam fronteiras

**Structs simples** ou DTOs. Nunca:

- Entities cruzando para fora de seu círculo.
- Linhas de banco (RowStructures) entrando no core.
- Tipos de ORM vazando para use cases.

Passe sempre no formato **mais conveniente para o círculo interno** — mesmo que seja só um map ou argumentos soltos em uma função.

## Conclusão em 1 linha

> *Ao separar software em camadas concêntricas e respeitar a Dependency Rule, você obtém um sistema intrinsicamente testável, cujos detalhes periféricos (DB, framework, UI) são substituíveis com mínimo esforço.*

## Fontes

- [Robert C. Martin — The Clean Architecture (blog.cleancoder.com, 2012-08-13)](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html)
- Martin, Robert C. (2017). *Clean Architecture: A Craftsman's Guide to Software Structure and Design*. Prentice Hall.

## Links

- [[_moc]]
- [[clean-architecture-layers]] — aplicação concreta com camadas operacionais
- [[screaming-architecture]] — arquitetura grita o domínio
- [[a-little-architecture]] — diálogo sobre o que é uma decisão arquitetural
- [[framework-bound]] — por que manter frameworks a braços de distância
- [[no-db]] — DB é detalhe, não centro
- [[ddd-strategic-tactical]] — DDD tático cabe em Entities/Use Cases
- [[../principles/solid]] — DIP é o motor do cruzamento de fronteiras
