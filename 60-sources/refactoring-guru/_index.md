---
title: refactoring.guru — índice local
type: source
tags: [source, patterns, refactoring, gof]
source_url: https://refactoring.guru
ingested: 2026-04-22
---

# refactoring.guru — mirror local

Mirror do site `refactoring.guru` (Alexander Shvets), referência canônica de **Design Patterns (GoF)** + **catálogo de refactorings** (Martin Fowler). 3513 páginas em markdown, originalmente clonado via httrack.

## Conteúdo principal

- **Design Patterns** (23 padrões GoF + variantes):
  - Creational: Factory Method, Abstract Factory, Builder, Prototype, Singleton
  - Structural: Adapter, Bridge, Composite, Decorator, Facade, Flyweight, Proxy
  - Behavioral: Chain of Responsibility, Command, Iterator, Mediator, Memento, Observer, State, Strategy, Template Method, Visitor
- **Catálogo de refactorings** (Martin Fowler):
  - Composing methods, moving features, organizing data, simplifying conditionals, simplifying method calls, dealing with generalization
- **Code smells** (mau cheiros de código) com refactoring correspondente
- **Exemplos em múltiplas linguagens**: Java, C#, PHP, Python, Ruby, Swift, TypeScript, Go

## Estrutura canônica (pós reorg 2026-04-22)

```
refactoring-guru/
├── _index.md
├── _toc.md           # índice curado por domínio (GoF + Fowler + Smells) ← começar aqui
├── raw/              # 376 MDs EN (sufixo .html.md limpo pra .md)
│   ├── design-patterns-index.md     # landing GoF
│   ├── refactoring-index.md         # landing Fowler
│   ├── design-patterns/             # 22 patterns + 10 langs cada
│   │   ├── strategy.md, strategy/{cpp,go,java,typescript,...}/...
│   │   └── ... (mesma estrutura pros demais)
│   ├── refactoring/                 # meta + grupos + smells/ + techniques/
│   ├── refactorings/                # 66 técnicas Fowler individuais
│   └── smells/                      # 23 code smells
└── _notes/           # staging pre-promoção
```

**Arquivados em** `90-archive/2026-04-22-sources-raw/refactoring-guru/`: idiomas não-EN (`es/`, `fr/`, `ja/`, `ko/`, `pl/`, `pt-br/`, `ru/`, `uk/` + respectivos `.html.md` indexes) + ruído httrack (terms, privacy, refunds, store, register, login*, sendy/, password/, site-about, favicon, cert, images, content-usage-policy, ebook-license, index.html.md root).

## Como o agente consulta

**Retrieval**:
1. Agente vê antipattern AP-013 (switch gigante) em PR.
2. Busca `type: source` + tag `pattern` + termo "strategy" no grafo.
3. Encontra este diretório + nota `design-patterns/strategy/strategy.html.md`.
4. Lê a descrição do padrão + exemplo na linguagem do projeto.
5. Referencia em review: "ver [[refactoring-guru/design-patterns/strategy/strategy.html]]".

**Destilação**: à medida que padrões forem aplicados em projetos, criar notas atômicas em `[[../../10-knowledge/patterns/_moc|10-knowledge/patterns]]` (`strategy-pattern.md`, `builder-pattern.md`, etc.) que **referenciam** este diretório como fonte canônica.

## Status de ingestão

- ✅ 3513 MDs originalmente copiados do mirror httrack
- ✅ 2026-04-22 reorg: 376 MDs em EN em `raw/` (−89% do volume) + ~3137 arquivados (i18n + admin mirror)
- 📋 A destilar: ver [[_toc#Prioridade de destilação]]
- ⏳ Imagens (1061 PNGs) **não** copiadas — referências em markdown ficam com path quebrado; resolver via `beforeLoad hook` ou renderizar do HTML original quando precisar visualizar

## Licença / atribuição

Site é comercial (livro pago). Este mirror é uso **pessoal** pra consulta offline. **Não republicar** conteúdo íntegro em documentação pública — apenas citar com atribuição.

## Links

- [[_toc]] — **índice curado (GoF + Fowler + Smells)**
- [[_notes/_moc]] — staging de destilados
- [[../_moc]] — sources
- [[../../10-knowledge/patterns/_moc]] — destino do destilado
- [[../../10-knowledge/antipatterns/_moc]] — smells como AP-XXX
- [[../../10-knowledge/methodology/_moc]] — destino do catálogo Fowler
- Site original: https://refactoring.guru
