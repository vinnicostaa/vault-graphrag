---
title: NO DB — o banco é detalhe, não centro
type: concept
tags: [architecture, database, use-cases, uncle-bob, deferral]
source: https://blog.cleancoder.com/uncle-bob/2012/05/15/NODB.html
source_date: 2012-05-15
created: 2026-04-24
updated: 2026-04-24
aliases: [NO DB, No-DB, DB como detalhe]
---

# NO DB

Destilado do post de Robert C. Martin (Uncle Bob), 2012. Tese central: **o centro da aplicação não é o database nem qualquer framework — são os use cases.**

## A história dos anos 80-90

Nos anos 80, um punhado de empresas gigantes de RDBMS dominou o mercado via FUD (*fear, uncertainty, doubt*) junto a gerentes e marketing. "Relational" virou sinônimo de "bom". Outros mecanismos de armazenamento foram **proibidos** de fato.

Times passaram a pôr o **database no centro** do sistema. Convenceram-se, após ondas de hype de marketing, que o data model era **o** aspecto mais importante da arquitetura e o DB era coração/alma do design.

## Os sintomas do veneno

- **DBA** virou função separada. *"Mere programmers cannot be trusted with data."* — a mensagem de marketing que foi absorvida.
- SQL infiltrou-se em toda frestra: vazou para UI, business rules foram movidas para stored procedures, mail-merge escrito em SQL.
- Tabelas e rows permearam o código-fonte. O schema virou **The Blob**, consumindo tudo à vista.

## A renovação — e o mesmo erro

Anos 2000: surge o movimento **NoSQL**. BigTable, Mongo, CouchDB — várias alternativas. *"O beer voltou."*

**Mas**: Uncle Bob observa com horror que muitos times com NoSQL **ainda projetam o sistema ao redor do database**. A hype relacional envenenada continua ecoando — apenas com DB diferente no centro.

Uma **onda de frameworks** ascende embalando o DB, lutando para ocupar o centro das aplicações. Alegam "masterizar e domar" o DB. Frameworks clamam: *"Depend on me, and I'll set you free."* — e continuam o mesmo erro.

## A tese em 1 linha

> **The center of your application are the use cases of your application.**

O DB é um detalhe que **você não precisa decidir agora**.

## Sintomas de aplicação DB-centric

Uncle Bob enlouquece quando vê um dev descrever o sistema assim:

> *"É um sistema Tomcat usando Spring e Hibernate com Oracle."*

A própria frase **coloca frameworks e DB no centro**. Imagine a arquitetura:

- Business objects looking suspeitamente like database rows.
- Schema e frameworks poluindo tudo.
- Código arranjado para caber **dentro** do pattern das frameworks.
- Use cases quase invisíveis.

## Como deve parecer uma aplicação saudável

- **Use cases no topo** do diagrama — a entidade mais alta e visível arquiteturalmente.
- **Use cases no centro** — sempre.
- **Databases e frameworks como detalhes** nas bordas.
- Você **não decide** sobre eles no início. Decide **muito depois**, quando os use cases e business rules estão escritos **e testados**.

## Qual é o melhor momento para decidir o data model?

**Quando você sabe**:

1. Quais são as data entities.
2. Como elas se relacionam.
3. Como são usadas.

**Quando você sabe disso?** Quando os use cases e business rules estão escritos **e testados**. Nessa hora você identifica as queries, os relacionamentos, os data elements — e constrói um data model que se encaixa nicamente em um DB.

## Isso muda se for NoSQL?

> *"Of course not!"*

Você ainda foca em use cases funcionando e testados **antes** de pensar no DB — não importa qual tipo.

## Como evitar que o DB conquiste o centro

- **Não** inicie decisões de DB cedo. Use flat file, in-memory, mock — qualquer coisa que destrave você.
- **Adiar** a decisão real até que as queries reais sejam conhecidas (depois de use cases testados).
- **Say "No"** à tentação de "fazer o DB funcionar primeiro".

Se você envolve o DB cedo, ele **deturpa o design**. Vai lutar para controlar o centro — e, como *scruffy terrier*, não solta.

## Implicações práticas

- Business rules testáveis **sem** DB conectado.
- Repository patterns são **domínio-owned** (interface em `domain/`, impl em `infrastructure/`).
- Migrations e schemas são **consequência** das business rules, não premissa.
- Mudança de DB é mudança de **um adapter**, não do core.

## Fontes

- [Robert C. Martin — NO DB (blog.cleancoder.com, 2012-05-15)](https://blog.cleancoder.com/uncle-bob/2012/05/15/NODB.html)
- Martin, Robert C. (2017). *Clean Architecture*.

## Links

- [[_moc]]
- [[the-clean-architecture]] — DB na camada externa; core o ignora
- [[screaming-architecture]] — arquitetura grita domínio, não DB
- [[a-little-architecture]] — decisão de DB é irrelevante pro arquiteto
- [[framework-bound]] — frameworks embalando DB são o mesmo erro disfarçado
- [[../patterns/adapter]] — DB via adapter em infrastructure
- [[../database/database-patterns]] — quando chegar a hora de decidir, aqui são os padrões
- [[../patterns/transaction-patterns]] — UoW + Saga como alternativas ao DB-centric
