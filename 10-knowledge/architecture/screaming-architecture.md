---
title: Screaming Architecture
type: concept
tags: [architecture, use-cases, domain, uncle-bob, naming]
source: https://blog.cleancoder.com/uncle-bob/2011/09/30/Screaming-Architecture.html
source_date: 2011-09-30
created: 2026-04-24
updated: 2026-04-24
aliases: [Screaming Architecture, Arquitetura Gritante]
---

# Screaming Architecture

Destilado do post de Robert C. Martin (Uncle Bob), 2011. Tese central: **a arquitetura de uma aplicação deve gritar o domínio, não o framework**.

## A analogia da planta arquitetônica

Quando você olha a planta de uma **residência**, a arquitetura grita "casa": entrada, sala, cozinha, quartos. Quando olha a planta de uma **biblioteca**, grita "biblioteca": balcão de empréstimo, galerias, salas de leitura.

Mas quando olhamos o top-level de um repositório de software, o que ele grita?

- "Rails", "Spring/Hibernate", "ASP.NET"? → arquitetura está gritando **framework**.
- "Sistema de Saúde", "Sistema Contábil", "Sistema de Inventário"? → arquitetura está gritando **domínio**. É o que ela deveria fazer.

## O tema de uma arquitetura

Retomando Ivar Jacobson (*Object Oriented Software Engineering: A Use Case Driven Approach*):

> Arquiteturas de software são estruturas que suportam os **use cases** do sistema.

A planta grita os use cases da construção (morar, emprestar livros). A arquitetura de software deveria gritar os use cases do produto.

**Arquiteturas não são frameworks**. Frameworks são ferramentas **usadas**, não arquiteturas às quais o sistema **se conforma**. Se a arquitetura é baseada em framework, ela **não pode** ser baseada em use cases.

## Por que use cases no centro

O arquiteto projeta a casa primeiro para ser **habitável** — só depois decide se é de tijolo, madeira ou pedra. Software é igual: a boa arquitetura permite diferir decisões sobre framework, DB, web server, e outros detalhes ambientais.

- Boa arquitetura **não obriga** a decidir Rails, Spring, Hibernate, Tomcat ou MySQL até muito tarde no projeto.
- Boa arquitetura **facilita mudar de ideia** sobre essas decisões.
- Boa arquitetura **enfatiza use cases** e desacopla de concerns periféricos.

## E a web?

A web é uma **delivery mechanism**, não arquitetura. O fato de o sistema ser entregue via web é um **detalhe** que deve ser **diferível**. O sistema deveria ser entregável como:

- console app
- web app
- thick client
- serviço REST

...sem complicação nem mudança fundamental na arquitetura.

## Frameworks — céticos, não devotos

Autores de framework acreditam em sua criação. Escrevem exemplos com o ponto de vista de **true believer**. Mostram como acoplar fortemente ao framework — extraindo todo o benefício prometido.

Leia-os com **olhar cético**:

- Framework pode ajudar, mas a que custo?
- Como usá-lo e **proteger-se** dele?
- Como preservar a ênfase use-case da arquitetura?
- Como impedir que o framework **tome conta** da arquitetura?

## Arquiteturas testáveis

Se a arquitetura é sobre use cases e frameworks ficam a braços de distância, então:

- Testes rodam **sem** web server.
- Testes rodam **sem** database connection.
- Business objects são *plain old objects*, sem dependência de framework.
- Use cases coordenam business objects, todos testáveis in-situ.

## Conclusão

> Um programador novo olhando o repositório deve dizer: *"Oh, isto é um sistema de saúde"* — e aprender todos os use cases sem saber ainda como o sistema é entregue.
>
> Se ele pergunta *"onde estão as views e os controllers?"* a resposta é: *"São detalhes que não precisam te preocupar agora; mostraremos mais tarde."*

## Fontes

- [Robert C. Martin — Screaming Architecture (blog.cleancoder.com, 2011-09-30)](https://blog.cleancoder.com/uncle-bob/2011/09/30/Screaming-Architecture.html)
- Jacobson, Ivar (1992). *Object Oriented Software Engineering: A Use Case Driven Approach*. Addison-Wesley.

## Links

- [[_moc]]
- [[the-clean-architecture]] — Dependency Rule que torna isto possível
- [[framework-bound]] — complementa: como não cair na armadilha
- [[no-db]] — DB também é detalhe diferível
- [[a-little-architecture]] — diálogo sobre decisões importantes vs irrelevantes
- [[ddd-strategic-tactical]] — ubiquitous language alimenta o "screaming"
