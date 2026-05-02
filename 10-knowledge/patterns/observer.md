---
title: Observer
type: pattern
tags: [pattern, behavioral, gof, events, pub-sub]
source: https://refactoring.guru/design-patterns/observer
created: 2026-04-24
updated: 2026-04-24
aliases: [Event-Subscriber, Listener, Publish-Subscribe]
---

# Observer

Padrão **comportamental** que define um **mecanismo de assinatura** para notificar múltiplos objetos sobre eventos que ocorrem no objeto que observam.

## Problema que resolve

Quando o estado de um objeto muda e outros precisam reagir, duas abordagens ingênuas falham:

1. **Polling** — os interessados verificam periodicamente (desperdício massivo de CPU e latência alta).
2. **Push forçado** — o objeto notifica todos, sempre, espalhando spam para quem não se interessa e acoplando o emissor ao conjunto fixo de receptores.

Pior ainda: o conjunto de interessados pode mudar em runtime (interessados entram e saem), e o emissor não deveria conhecer as classes concretas de quem o observa — senão muda toda vez que um novo observador aparece.

## Solução

O objeto observado (**publisher / subject**) mantém uma lista de **subscribers** e expõe métodos para `subscribe()` / `unsubscribe()`. Quando um evento ocorre, o publisher percorre a lista e chama `update()` em cada subscriber.

Subscribers implementam uma **interface comum** — o publisher os conhece só por ela. Assim pode-se ter N tipos de observadores reagindo diferentemente ao mesmo evento, sem que o publisher saiba disso. A dependência é invertida: subscribers se conectam ao publisher, não o contrário.

## Estrutura

- **Publisher / Subject** — mantém lista de subscribers; expõe `subscribe`, `unsubscribe`, `notify`.
- **Subscriber / Observer** (interface) — declara `update(context)`.
- **Concrete Subscribers** — reagem à notificação; podem ter lógica independente entre si.
- **Client** — instancia publisher + subscribers e liga-os.

## Pseudocódigo

```
interface Subscriber is
    method update(event, data)

class EventManager is
    private listeners: map<event, list<Subscriber>>

    method subscribe(event, s) is listeners[event].add(s)
    method unsubscribe(event, s) is listeners[event].remove(s)
    method notify(event, data) is
        foreach (s in listeners[event]) s.update(event, data)

class Editor is
    public events = new EventManager()
    method save() is
        writeFile()
        events.notify("save", filename)

class LoggingListener implements Subscriber is
    method update(event, data) is log.write(event + ": " + data)

// Client:
editor.events.subscribe("save", new LoggingListener())
editor.events.subscribe("save", new EmailAlertListener())
```

## Quando usar

- Mudança em um objeto pode disparar ações em N outros, e o conjunto muda em runtime.
- Emissor não deve (ou não pode) conhecer concretamente os receptores.
- Relações temporárias: alguns observam só durante certas fases.
- Modelar eventos de domínio, hooks, callbacks genéricos.

## Quando evitar

- Apenas um receptor conhecido — uma chamada direta é mais clara.
- Ordem e garantia de entrega importam muito — Observer é fire-and-forget sem ordem fixa.
- Fluxo síncrono crítico onde falhas precisam propagar — exceções em subscribers podem ser engolidas ou truncar a cadeia.

## Sinais de que você está reinventando este pattern

- Função chamando manualmente `logA()`, `logB()`, `notifyC()` toda vez que algo acontece.
- Classe com lista de callbacks e método `fire()` — já é Observer.
- Event bus global com `on()` / `emit()` — Observer em escala.

## Trade-offs

### Vantagens
- **OCP** — novos subscribers sem alterar o publisher.
- Relações estabelecidas em runtime, não em compile-time.
- Desacopla produtor de consumidores — publisher conhece só a interface.

### Desvantagens
- Subscribers notificados em ordem **não garantida**.
- Memory leak se subscribers não derregistram (listener zombie).
- Cadeias longas de notificação ficam difíceis de rastrear (debug hell).
- Propagação síncrona de exceções é delicada.

## Relações com outros patterns

- [[chain-of-responsibility]], [[command]], [[mediator]] — todos tratam conexão emissor-receptor; Observer permite **subscrição dinâmica** one-to-many.
- [[mediator]] — pode ser implementado sobre Observer (mediator vira publisher; componentes se inscrevem nos eventos dele). Diferença: Mediator elimina dependências mútuas via hub; Observer estabelece canais dinâmicos one-way.

## Fontes

- [Refactoring Guru — Observer](https://refactoring.guru/design-patterns/observer)
- Gamma, Helm, Johnson, Vlissides (1994). *Design Patterns: Elements of Reusable Object-Oriented Software*. Addison-Wesley.

## Links

- [[_moc]]
- [[behavioral-patterns]]
- [[../principles/solid]] — OCP e DIP aplicados por este pattern
- [[../_moc]] — knowledge raiz
