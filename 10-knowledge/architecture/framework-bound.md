---
title: Framework Bound — frameworks são ferramentas, não modos de vida
type: concept
tags: [architecture, frameworks, coupling, boundaries, uncle-bob]
source: https://blog.cleancoder.com/uncle-bob/2014/05/11/FrameworkBound.html
source_date: 2014-05-11
created: 2026-04-24
updated: 2026-04-24
aliases: [Framework Bound, Framework Boundedness, Frameworks como ferramenta]
---

# Framework Bound

Destilado do post de Robert C. Martin (Uncle Bob), 2014. Tese central: **frameworks são ferramentas poderosas que cobramos preço alto quando tratamos como modo de vida.**

## O apelo das frameworks

Rails, Spring, JSF, Hibernate, Django, .NET. Imagine construir um sistema web sem elas — é desanimador. Detalhes infinitos para tratar manualmente. É como *"construir circuito mnemônico com facas de pedra e peles de urso"*.

Por isso nos acoplamos felizes às frameworks, antecipando seus benefícios. E cometemos o mesmo erro que incontáveis programadores antes de nós.

## O custo invisível do commitment

Ao aceitar um framework no seu código, você **entrega o controle** dos detalhes que o framework gerencia. Parece bom — geralmente é. Mas há uma armadilha:

1. Você se pega fazendo **atos anti-naturais**: herdar de base classes do framework, abrir mão de fluxo de controle, curvando-se cada vez mais às convenções, quirks e idiossincrasias do framework.
2. Apesar do commitment **enorme** que você fez, o framework **não se compromete com você**.

O framework é livre para evoluir na direção que *agrada seu autor*. Quando isso acontece, você **segue como cachorrinho fiel**.

## Por que isso não é surpresa

Frameworks são escritos por pessoas para resolver problemas **que elas têm**. Esses problemas podem ser **similares** aos seus — mas não são os seus.

- Onde os problemas se **sobrepõem** → framework ajuda enormemente.
- Onde os problemas **conflitam** → framework é impedimento enorme.

**TANSTAAFL** (There Ain't No Such Thing As A Free Lunch).

## Authors dos frameworks — agenda própria

- Um dos itens da agenda deles é fazer você **usar** o framework.
- Autoestima pessoal se tornou tecida à aceitação do framework. (Uncle Bob fala como ex-autor de frameworks.)
- Escrevem papers e dão talks. Fornecem exemplos que mostram **tight binding** ao framework. Demonstram benefícios do binding forte.
- São *true believers* — e é legítimo. Eles querem você na tribo.

**Mas assim que você está dentro, o interesse deles pode mudar.**

## A estratégia saudável: arm's length

> *Sim, frameworks podem ser extremamente úteis. Mas há custos — e às vezes altíssimos.*

A estratégia saudável: manter frameworks como Spring, Hibernate, Rails **atrás de fronteiras arquiteturais**.

- Você consegue **a maior parte** do benefício deles assim.
- Você pode tirar **vantagem agressiva** deles onde são úteis.
- Você **não** entrega autonomia a eles.
- Você **não** deixa os tentáculos do código deles entrelaçar-se com a policy de alto nível do seu sistema.
- Eles podem tocar subsistemas periféricos. **Não** podem tocar o core business logic.

**High-level policies nunca serão tocadas por frameworks.**

## Aplicação prática

Fronteiras arquiteturais (ver [[the-clean-architecture]]):

1. Framework vive na camada **externa** (Frameworks & Drivers).
2. Adapter traduz entre framework e application (camada **Interface Adapters**).
3. Use cases e entities **não sabem** que o framework existe.
4. Mudar de Rails para Django muda a camada externa + adapter — não o core.

## Sinais de que você está framework-bound

- Entity herdando de classe-base do framework (ex.: `ActiveRecord::Base`, `JpaEntity`).
- Business rule importando tipos do framework (ex.: `HttpRequest`, `Session`).
- Upgrade do framework obriga mudança em arquivos do domínio.
- Dificuldade de escrever teste unitário sem bootar o framework.
- Docs do framework são a principal referência do time — não docs do próprio domínio.

## Fontes

- [Robert C. Martin — Framework Bound (blog.cleancoder.com, 2014-05-11)](https://blog.cleancoder.com/uncle-bob/2014/05/11/FrameworkBound.html)
- Martin, Robert C. (2017). *Clean Architecture*, capítulo sobre frameworks.

## Links

- [[_moc]]
- [[the-clean-architecture]] — boundaries arquiteturais que mantêm frameworks a distância
- [[screaming-architecture]] — arquitetura grita domínio, não framework
- [[no-db]] — DB é framework disfarçado
- [[a-little-architecture]] — framework não é decisão importante do arquiteto
- [[../patterns/adapter]] — pattern usado para isolar framework
- [[clean-architecture-layers]] — onde encaixa operacionalmente
