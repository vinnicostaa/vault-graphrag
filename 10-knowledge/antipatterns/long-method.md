---
title: Long Method (AP-010)
type: antipattern
tags: [antipattern, code-smell, bloater, readability]
source: "https://refactoring.guru/smells/long-method"
source_date: 2026-04-24
created: 2026-04-24
updated: 2026-04-24
aliases: [long-function, giant-method]
---

# AP-010 — Long Method

Método/função com dezenas (centenas) de linhas que faz muitas coisas. Mais letal que *God Object* porque esconde a falta de estrutura dentro de um bloco só.

## Sintomas

- Comentários agrupando "seções" dentro da função (`// validate`, `// compute`, `// save`)
- Precisa scrollar ou abrir mapa do editor para entender fluxo
- Aninhamento de 3+ níveis (if dentro de for dentro de try)
- Variáveis temporárias em escopo amplo com nome genérico (`tmp`, `result`, `aux`)
- Difícil descrever o que faz em uma frase sem "e"

## Por que é problema

**Compreensão quadrática**: entender o método exige carregar todo o contexto mentalmente. Cada novo leitor paga o custo completo.

**Testes caros**: cobrir todos os caminhos exige N fixtures; mockar dependências exige recompor a função inteira.

**Bugs escondidos**: early return no meio, side-effects no final — fácil introduzir regressão sem perceber.

**Impede reuso**: lógica boa fica presa; copy-paste vira a opção.

## Exemplo

```python
# ❌ 60 linhas em 1 função
def process_order(order):
    # validate
    if not order.items: raise ...
    if order.total <= 0: raise ...
    # apply discounts
    for item in order.items:
        if item.category == "sale": ...
        elif item.category == "clearance": ...
    # charge
    try: payment = gateway.charge(...)
    except ...: ...
    # send email
    html = render_template(...)
    smtp.send(...)
    # audit
    audit.log(...)
    return receipt

# ✅ funções pequenas nomeadas
def process_order(order):
    validate(order)
    apply_discounts(order)
    payment = charge(order)
    notify(order, payment)
    audit(order, payment)
    return make_receipt(order, payment)
```

## Como refatorar

- **Extract Method**: cada "seção comentada" vira função nomeada
- **Replace Temp with Query**: variáveis temporárias viram chamadas
- **Introduce Parameter Object** quando a extração exige passar muitos params
- **Replace Method with Method Object** para métodos muito longos com estado compartilhado
- **Guard clauses / early return**: achatar aninhamento ([[../principles/code-calisthenics]])

## Quando tolerar

- **Pipelines lineares claros** sem ramificação (10-20 linhas descrevendo fluxo óbvio)
- **Algoritmos matemáticos** onde partir destruiria a coesão da expressão
- **Métodos gerados** por tool (parsers, lexers)
- **DSLs internas** onde a sequência é o conteúdo

## Patterns relacionados (que resolvem)

- [[../patterns/template-method]] — esqueleto + hooks
- [[../patterns/strategy]] — extrair ramificações
- [[../patterns/chain-of-responsibility]] — pipeline de handlers
- [[../principles/code-calisthenics]] — "um nível de indent por método"
- [[../principles/clean-code]] — "funções pequenas, nomeadas"

## Fontes

- Refactoring Guru — [Long Method](https://refactoring.guru/smells/long-method)
- Fowler, M. (2018). *Refactoring: Improving the Design of Existing Code*. 2nd ed. Addison-Wesley. — cap. "Extract Method" é o canônico.
- Martin, R. C. (2008). *Clean Code*. Cap. 3 (Functions) — "functions should be small; smaller than that".

## Links

- [[_moc]]
- [[../_moc]]
- [[../patterns/_moc]]
- [[../principles/code-calisthenics]]
- [[../principles/clean-code]]
