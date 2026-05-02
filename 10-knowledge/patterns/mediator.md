---
title: Mediator
type: pattern
tags: [pattern, behavioral, gof, decoupling, communication]
source: https://refactoring.guru/design-patterns/mediator
created: 2026-04-24
updated: 2026-04-24
aliases: [Intermediary, Controller]
---

# Mediator

Padrão **comportamental** que reduz dependências caóticas entre objetos, forçando-os a colaborar **indiretamente** através de um objeto mediador central.

## Problema que resolve

Em sistemas onde vários componentes interagem (formulários com campos que reagem uns aos outros, objetos de domínio que se notificam mutuamente), cada componente acaba referenciando todos os outros que lhe interessam. A matriz de dependências explode em `N*(N-1)` conexões.

Componentes ficam impossíveis de reusar isoladamente — um checkbox que controla a visibilidade de um campo de texto só funciona naquele formulário específico. Mudar um componente propaga ondas de alteração para os vizinhos.

Pior: testar um componente exige montar o grafo inteiro de colaboradores.

## Solução

Interromper toda comunicação direta entre componentes. Introduzir um **Mediator** que conhece os componentes e orquestra as interações. Quando algo importante acontece em um componente, ele só avisa o mediator; este decide o que fazer e notifica quem precisa reagir.

Componentes passam a depender apenas da interface do mediator, não uns dos outros. A teia de conexões some de dentro dos componentes e concentra-se no mediator concreto, onde pode ser mantida em um ponto só.

## Estrutura

- **Mediator** (interface) — declara método(s) de notificação que componentes usam.
- **Concrete Mediator** — implementa cooperação; guarda referências aos componentes e despacha.
- **Component** — classe de negócio com referência ao mediator; desconhece outros componentes.
- **Concrete Components** — só falam com o mediator via notificação.

## Pseudocódigo

```
interface Mediator is
    method notify(sender: Component, event: string)

class AuthDialog implements Mediator is
    private loginChk, okBtn, userField, passField

    method notify(sender, event) is
        if (sender == loginChk and event == "toggle")
            showLoginFields()
        if (sender == okBtn and event == "click")
            validateAndSubmit()

class Component is
    protected mediator: Mediator
    constructor(m) is this.mediator = m

class Button extends Component is
    method click() is mediator.notify(this, "click")

class Checkbox extends Component is
    method toggle() is mediator.notify(this, "toggle")
```

## Quando usar

- Mudar uma classe é difícil porque ela está amarrada a muitas outras.
- Quer reusar um componente em outros contextos, mas ele depende demais dos vizinhos.
- Cria-se subclasses demais de componentes só para pequenas variações de comportamento relacional.
- Lógica de coordenação entre componentes está espalhada e repetida.

## Quando evitar

- Interações simples, 1-para-1 — mediator adiciona indireção desnecessária.
- Sistemas pequenos onde o acoplamento direto não incomoda.
- Quando a coordenação é naturalmente hierárquica — composição explícita pode ser mais clara.

## Sinais de que você está reinventando este pattern

- Componentes UI com referências cruzadas diretas (o botão conhece todos os campos).
- Regra "se eu mudar X, atualize Y, Z, W" repetida em várias classes.
- Um controller/orquestrador que já conhece todos os colaboradores — provavelmente já é mediator.

## Trade-offs

### Vantagens
- **SRP** — centraliza comunicação em um lugar visível.
- **OCP** — novos mediators introduzem novos modos de cooperação sem mudar componentes.
- Reduz acoplamento; componentes viram reutilizáveis.
- Fluxo de interação fica documentado em um único arquivo.

### Desvantagens
- Mediator tende a crescer e pode virar **God Object** se não for particionado.
- Toda interação passa pelo mediator — pode se tornar gargalo conceitual.
- Debug: rastrear "quem disparou isto" exige seguir o mediator.

## Relações com outros patterns

- [[chain-of-responsibility]], [[command]], [[observer]] — todos tratam conexão emissor-receptor; Mediator elimina conexões diretas forçando comunicação indireta.
- [[facade]] — ambos organizam colaboração de subsistemas complexos; Facade **simplifica** interface sem mudar como o subsistema internamente opera; Mediator **centraliza** comunicação e componentes só falam com ele.
- [[observer]] — diferença sutil: Mediator elimina dependências mútuas via hub central; Observer estabelece conexões one-way dinâmicas. Mediator frequentemente é implementado via Observer (mediator = publisher).

## Fontes

- [Refactoring Guru — Mediator](https://refactoring.guru/design-patterns/mediator)
- Gamma, Helm, Johnson, Vlissides (1994). *Design Patterns: Elements of Reusable Object-Oriented Software*. Addison-Wesley.

## Links

- [[_moc]]
- [[behavioral-patterns]]
- [[../principles/solid]] — SRP/OCP aplicados por este pattern
- [[../_moc]] — knowledge raiz
