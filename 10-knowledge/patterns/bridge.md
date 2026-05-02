---
title: Bridge
type: pattern
tags: [pattern, structural, gof, abstraction, composition]
source: https://refactoring.guru/design-patterns/bridge
created: 2026-04-24
updated: 2026-04-24
aliases: [Bridge Pattern, Handle/Body]
---

# Bridge

Padrão **estrutural** que divide uma classe grande (ou um conjunto de classes fortemente ligadas) em **duas hierarquias independentes** — abstração e implementação — que podem evoluir separadamente.

## Problema que resolve

Quando uma classe precisa variar em **duas dimensões ortogonais** (ex.: `Shape` × `Color`, `GUI` × `OS`, `Report` × `Storage`), herança sozinha gera explosão combinatória: cada nova subclasse exige criar cópias para cada valor da outra dimensão.

O código vira inerentemente frágil: mudar qualquer dimensão obriga a tocar dezenas de subclasses, e adicionar uma terceira dimensão multiplica a dor. Uma única classe monolítica com `if/else` por plataforma sofre do mesmo problema, só que escondido.

## Solução

Extrair **uma das dimensões** em uma hierarquia própria (*implementation*) e fazer a hierarquia original (*abstraction*) conter uma referência a um objeto dela. A abstração delega todo o trabalho específico da dimensão extraída ao objeto referenciado — a **ponte** entre as duas hierarquias.

O termo "abstraction" aqui não é `abstract class` da linguagem: é a **camada de alto nível** (controle). "Implementation" é a **camada de baixo nível** (plataforma). As duas hierarquias podem evoluir e ser compostas em runtime; trocar a implementação é atribuir outro objeto ao campo.

## Estrutura

- **Abstraction** — lógica de alto nível; mantém referência a um `Implementation` e delega o trabalho concreto.
- **Refined Abstraction** — variantes de controle, estendendo `Abstraction`.
- **Implementation** — interface da camada de baixo nível; pode ser bem diferente da abstração, tipicamente com operações primitivas.
- **Concrete Implementation** — código específico da plataforma/variante.
- **Client** — liga uma `Abstraction` a uma `Implementation` concreta; daí em diante só fala com a abstração.

## Pseudocódigo

```
// Implementation (baixo nível)
interface Device is
    method enable(); method disable()
    method setVolume(percent); method getVolume()

class Tv implements Device is ...
class Radio implements Device is ...

// Abstraction (alto nível) — tem referência a Device
class RemoteControl is
    protected field device: Device
    constructor(device) is this.device = device
    method togglePower() is
        if device.isEnabled() then device.disable()
        else device.enable()
    method volumeUp() is device.setVolume(device.getVolume() + 10)

// Refined Abstraction
class AdvancedRemote extends RemoteControl is
    method mute() is device.setVolume(0)

// Client
remote = new AdvancedRemote(new Tv())
remote.togglePower()
```

## Quando usar

- A classe tem **2+ dimensões ortogonais** de variação (tipo × plataforma, formato × destino, driver × canal).
- Você precisa **trocar a implementação em runtime** (mesmo remoto controlando TV ou rádio).
- Dividir um monólito grande com várias variantes de funcionalidade — por exemplo, código que fala com múltiplos servidores de banco via mesma API de domínio.
- Em Clean Architecture, separar um *use case* (abstraction) de suas portas de saída (implementation) é uma aplicação do espírito do Bridge.

## Quando evitar

- Aplicado a uma classe altamente coesa, o pattern **adiciona complexidade sem payoff** real.
- Só existe **uma dimensão** hoje e não há sinais concretos de uma segunda — abstração prematura (YAGNI).

## Sinais de que você está reinventando este pattern

- Classe que recebe um "driver"/"backend"/"provider" no construtor e delega cada método pra ele.
- Hierarquia com nomes compostos (`WindowsPdfReport`, `LinuxPdfReport`, `WindowsXmlReport`...) crescendo em tabuada.
- Camada de GUI cheia de `switch(platform)` para chamar APIs diferentes.

## Trade-offs

### Vantagens
- Permite classes e apps **independentes de plataforma**.
- Cliente fala só com abstrações de alto nível, isolado de detalhes.
- *Open/Closed Principle* — adicionar abstrações e implementações **independentemente** umas das outras.
- *Single Responsibility Principle* — abstração foca em controle, implementação em plataforma.

### Desvantagens
- Pode complicar código já coeso desnecessariamente.
- Exige desenho antecipado — retrofit é mais caro.

## Relações com outros patterns

- [[adapter]] — aplicado **depois**, pra fazer classes pré-existentes cooperarem; Bridge é desenhado **antes**, pra permitir desenvolvimento independente das partes.
- [[state]], [[strategy]] — compartilham estrutura (composição/delegação); diferem pela **intenção**. Bridge frequentemente é confundido com Strategy quando a implementação é trocada em runtime.
- [[abstract-factory]] — pode encapsular combinações válidas de Abstraction × Implementation, escondendo a ligação do cliente.
- [[builder]] — o director pode atuar como abstração, diferentes builders como implementações.

## Fontes

- [Refactoring Guru — Bridge](https://refactoring.guru/design-patterns/bridge)
- Gamma, Helm, Johnson, Vlissides (1994). *Design Patterns: Elements of Reusable Object-Oriented Software*. Addison-Wesley.

## Links

- [[_moc]]
- [[structural-patterns]]
- [[../architecture/clean-architecture-layers]] — abstração/implementação em camadas
- [[../_moc]] — knowledge raiz
