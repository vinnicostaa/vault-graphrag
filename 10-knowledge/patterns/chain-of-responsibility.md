---
title: Chain of Responsibility
type: pattern
tags: [pattern, behavioral, gof, pipeline, handlers]
source: https://refactoring.guru/design-patterns/chain-of-responsibility
created: 2026-04-24
updated: 2026-04-24
aliases: [CoR, Chain of Command]
---

# Chain of Responsibility

Padrão **comportamental** que permite passar um request por uma **cadeia dinâmica de handlers**, onde cada handler decide se processa o request ou o passa adiante.

## Problema que resolve

Fluxos que exigem várias verificações/etapas em sequência (autenticação, sanitização, rate limiting, cache, autorização) tendem a acumular em uma única classe de entrada. Esse bloco monolítico infla a cada nova regra, mistura preocupações e torna mudança perigosa: alterar uma etapa afeta as outras.

Pior ainda quando outras partes do sistema precisam das mesmas verificações, mas em combinações diferentes. Sem desacoplamento, só resta duplicar trechos de código ou amarrar componentes a uma sequência fixa.

O resultado é uma classe que cresce sem parar, difícil de entender, testar e reusar.

## Solução

Extrair cada etapa para um **handler** com um único método de processamento. Cada handler guarda referência ao **próximo** da cadeia; depois de fazer seu trabalho (ou antes), decide se delega para o próximo ou encerra o processamento.

Todos os handlers implementam a mesma interface, então o cliente monta a cadeia em runtime combinando handlers como peças de Lego. O request pode parar no meio (variante "first match") ou percorrer toda a cadeia (variante "pipeline").

## Estrutura

- **Handler** (interface) — declara o método de processamento e, opcionalmente, setter para o próximo handler.
- **Base Handler** (opcional) — boilerplate compartilhado: campo `next`, delegação padrão para o próximo.
- **Concrete Handlers** — implementam a lógica real; decidem processar e/ou repassar.
- **Client** — monta a cadeia e dispara o request no primeiro handler (ou em qualquer ponto).

## Pseudocódigo

```
interface Handler is
    method setNext(h: Handler): Handler
    method handle(req): Result

abstract class BaseHandler implements Handler is
    private next: Handler
    method setNext(h) is this.next = h; return h
    method handle(req) is
        if (next != null) return next.handle(req)
        return null

class AuthHandler extends BaseHandler is
    method handle(req) is
        if (!authenticated(req)) return reject("auth")
        return super.handle(req)

class RateLimitHandler extends BaseHandler is
    method handle(req) is
        if (overLimit(req)) return reject("rate")
        return super.handle(req)

// Client:
chain = new AuthHandler()
chain.setNext(new RateLimitHandler()).setNext(new BusinessHandler())
result = chain.handle(request)
```

## Quando usar

- Múltiplas verificações sequenciais cuja ordem e composição variam.
- Conjunto de handlers e sua ordem podem mudar em runtime.
- Só um handler deve processar o request, mas quem é não é conhecido a priori (event bubbling em GUI).
- Quer desacoplar emissor do request dos possíveis receptores.

## Quando evitar

- Pipeline fixo com ordem imutável — um simples `switch` ou lista estática basta.
- Nenhum request pode ficar sem handler — o pattern permite request não-processado silenciosamente; use com cuidado.
- Complexidade de debugar fluxos que saltam entre handlers não compensa em fluxos simples.

## Sinais de que você está reinventando este pattern

- Cascata de `if/else` guardando pré-condições antes do trabalho principal.
- Lista ordenada de validators/middlewares chamada manualmente em loop.
- Middleware stacks em frameworks web (Express, Koa, ASP.NET pipeline) — já são CoR.

## Trade-offs

### Vantagens
- Controla a ordem de processamento explicitamente.
- **SRP** — separa invocação de execução; cada handler tem uma responsabilidade.
- **OCP** — novos handlers entram sem alterar o cliente.
- Chains dinâmicas montáveis conforme configuração.

### Desvantagens
- Request pode chegar ao fim sem ser tratado — precisa fallback explícito.
- Debug difícil quando a cadeia é longa ou montada indiretamente.
- Performance: cada passo é uma chamada indireta a mais.

## Relações com outros patterns

- [[command]] — handlers podem ser implementados como commands; alternativamente, o request em si pode ser um command percorrendo a cadeia.
- [[mediator]] — elimina conexões diretas entre emissor e receptor; CoR passa o request sequencialmente por receptores potenciais.
- [[observer]] — receptores se inscrevem/desinscrevem dinamicamente; CoR tem ordem fixa de tentativa.
- [[composite]] — CoR aparece naturalmente em árvores Composite (request sobe dos leaves até a raiz).
- [[decorator]] — estrutura parecida (composição recursiva), mas Decorator estende comportamento sem quebrar fluxo; CoR pode abortar.

## Fontes

- [Refactoring Guru — Chain of Responsibility](https://refactoring.guru/design-patterns/chain-of-responsibility)
- Gamma, Helm, Johnson, Vlissides (1994). *Design Patterns: Elements of Reusable Object-Oriented Software*. Addison-Wesley.

## Links

- [[_moc]]
- [[behavioral-patterns]]
- [[../principles/solid]] — SRP e OCP aplicados por este pattern
- [[../_moc]] — knowledge raiz
