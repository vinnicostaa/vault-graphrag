---
title: Adapter
type: pattern
tags: [pattern, structural, gof, interface-compat, legacy]
source: https://refactoring.guru/design-patterns/adapter
created: 2026-04-24
updated: 2026-04-24
aliases: [Adapter Pattern, Wrapper]
---

# Adapter

Padrão **estrutural** que permite objetos com **interfaces incompatíveis colaborarem**, traduzindo chamadas entre elas sem alterar o código original de nenhum dos lados.

## Problema que resolve

Você tem código que fala **formato A** (ex.: dados em XML) e precisa integrar uma lib/serviço que espera **formato B** (ex.: JSON). Alternativas ruins:

- Reescrever a lib — pode quebrar usuários existentes; frequentemente o código-fonte nem está acessível.
- Reescrever o código próprio para falar formato B — invasivo; piora quando houver múltiplas libs em formatos distintos.

É necessária uma camada tradutora que **não modifica** nenhum dos dois lados.

## Solução

Criar um **adapter** — objeto que implementa a interface que o cliente espera e internamente encapsula (ou herda de) o serviço incompatível. Cada chamada do cliente é traduzida pelo adapter para o formato que o serviço aceita.

Variantes:
- **Object adapter** (composição) — adapter contém uma referência ao serviço e delega. Suportado em qualquer linguagem OO.
- **Class adapter** (herança múltipla) — adapter herda de cliente **e** serviço. Só funciona em linguagens com herança múltipla (ex.: C++).

Adapters podem ser **bidirecionais**, traduzindo chamadas em ambos os sentidos.

## Estrutura

- **Client** — código que contém a lógica de negócio e espera uma interface específica.
- **Client Interface** — protocolo que o cliente conhece.
- **Service** — classe útil (frequentemente third-party ou legacy) com interface incompatível.
- **Adapter** — implementa a Client Interface e encapsula o Service; traduz cada chamada para o formato que o Service entende.

## Pseudocódigo

```
// Cliente espera RoundPeg; Service oferece SquarePeg (incompatível).
class RoundHole is
    method fits(peg: RoundPeg) is
        return this.getRadius() >= peg.getRadius()

class SquarePeg is
    method getWidth() is ...

class SquarePegAdapter extends RoundPeg is
    private field peg: SquarePeg

    constructor SquarePegAdapter(peg: SquarePeg) is
        this.peg = peg

    method getRadius() is
        // adapter "finge" ser um RoundPeg com raio equivalente
        return peg.getWidth() * sqrt(2) / 2

// Uso:
hole  = new RoundHole(5)
sqpeg = new SquarePeg(5)
hole.fits(new SquarePegAdapter(sqpeg))  // true
```

## Quando usar

- Você quer usar uma classe existente, mas a interface dela não combina com o resto do código.
- Você quer reaproveitar várias subclasses que faltam funcionalidade comum que não cabe na superclasse.
- Integração com SDK externo: cria-se um adapter em `infrastructure/providers/<provider>/` que implementa a interface de domínio — padrão recorrente em Clean Architecture.

## Quando evitar

- Se você **pode** alterar a classe de serviço para torná-la compatível, muitas vezes é menos código do que criar o adapter.
- Adapter genérico para "qualquer serviço" tende a escorregar para *anti-corruption layer* mal definida — prefira adapter específico por integração.

## Sinais de que você está reinventando este pattern

- Classe-ponte que recebe um objeto "legado" no construtor e expõe métodos que internamente chamam métodos do legado traduzindo argumentos.
- Funções "`fromLibXToMyFormat(input)`" espalhadas pelo código — sinal para consolidar num adapter.
- Conversões XML/JSON, snake_case/camelCase, millis/seconds repetidas em múltiplos pontos de entrada.

## Trade-offs

### Vantagens
- **Single Responsibility Principle** — separa conversão de interface da lógica de negócio.
- **Open/Closed Principle** — novos adapters podem ser adicionados sem alterar o código do cliente.
- Isola dependência de lib externa atrás da interface de domínio.

### Desvantagens
- Aumenta complexidade (novas classes e interfaces). Para casos simples, alterar a classe de serviço diretamente é menos código.

## Relações com outros patterns

- [[bridge]] — é desenhado antecipadamente para permitir desenvolvimento independente de partes; Adapter é aplicado depois, para fazer classes pré-existentes colaborarem.
- [[decorator]] — Decorator mantém ou estende a interface do objeto; Adapter altera para uma **diferente**. Decorator suporta composição recursiva; Adapter normalmente não.
- [[proxy]] — Proxy mantém a mesma interface; Adapter altera.
- [[facade]] — Facade cria interface nova sobre subsistema inteiro; Adapter normalmente encapsula **um** objeto.
- Estruturalmente similar a [[bridge]], [[state]], [[strategy]] — todos baseados em composição; a distinção é de **intenção**, não de forma.

## Fontes

- [Refactoring Guru — Adapter](https://refactoring.guru/design-patterns/adapter)
- Gamma, Helm, Johnson, Vlissides (1994). *Design Patterns: Elements of Reusable Object-Oriented Software*. Addison-Wesley.

## Links

- [[_moc]]
- [[structural-patterns]]
- [[../architecture/clean-architecture-layers]] — adapters na camada infrastructure
- [[../_moc]] — knowledge raiz
