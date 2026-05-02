---
title: Proxy
type: pattern
tags: [pattern, structural, gof, access-control, lazy]
source: https://refactoring.guru/design-patterns/proxy
created: 2026-04-24
updated: 2026-04-24
aliases: [Proxy Pattern, Surrogate]
---

# Proxy

Padrão **estrutural** que fornece um **substituto ou intermediário** para outro objeto, preservando sua interface e controlando o acesso — antes, durante ou depois de encaminhar a requisição ao alvo real.

## Problema que resolve

Às vezes o objeto real é caro (conexão de banco, download remoto, cálculo pesado), sensível (deve ser acessado só por certos clientes), instável (remoto, pode falhar) ou vem de lib terceira que você não pode modificar. Replicar lazy-init, cache, log, controle de acesso em **cada ponto** que usa o objeto causa duplicação e fragilidade.

Seria ideal colocar essa lógica dentro da classe do serviço, mas ou a classe não pode ser alterada, ou misturar isso lá viola SRP.

## Solução

Criar uma classe Proxy com a **mesma interface** do serviço real. O cliente continua recebendo algo que cumpre essa interface e não nota a diferença. O proxy guarda uma referência ao serviço e decide quando/como delegar:

- cria o serviço só na primeira chamada (lazy),
- valida credenciais antes de encaminhar (protection),
- armazena resultados anteriores (cache),
- registra cada chamada (logging),
- despacha para um servidor remoto (remote),
- monitora uso e libera recursos quando ninguém mais precisa (smart reference).

Como interface e intercambiabilidade são preservadas, o proxy pode ser injetado em qualquer lugar que esperava o serviço.

## Estrutura

- **Service Interface** — contrato comum entre serviço e proxy.
- **Service** — classe real com a lógica de negócio.
- **Proxy** — implementa a mesma interface; mantém referência ao `Service` (muitas vezes criando-o e gerenciando seu ciclo de vida); executa trabalho adicional antes/depois de delegar.
- **Client** — fala só com `Service Interface`; recebe ora um serviço real, ora um proxy.

## Pseudocódigo

```
interface ThirdPartyYouTubeLib is
    method listVideos()
    method getVideoInfo(id)
    method downloadVideo(id)

class ThirdPartyYouTubeClass implements ThirdPartyYouTubeLib is
    method listVideos() is /* chamada HTTP cara */
    method getVideoInfo(id) is /* chamada HTTP cara */
    method downloadVideo(id) is /* download pesado */

class CachedYouTubeClass implements ThirdPartyYouTubeLib is
    private field service: ThirdPartyYouTubeLib
    private field listCache, videoCache

    constructor(service) is this.service = service

    method listVideos() is
        if listCache == null then listCache = service.listVideos()
        return listCache

    method getVideoInfo(id) is
        if videoCache[id] == null then videoCache[id] = service.getVideoInfo(id)
        return videoCache[id]

    method downloadVideo(id) is
        if not downloadExists(id) then service.downloadVideo(id)

// Cliente não muda: recebe algo que implementa a interface.
real  = new ThirdPartyYouTubeClass()
proxy = new CachedYouTubeClass(real)
manager = new YouTubeManager(proxy)
```

## Quando usar

- **Virtual proxy** — adiar criação de um objeto caro até que seja realmente usado.
- **Protection proxy** — checar credenciais/políticas antes de permitir acesso.
- **Remote proxy** — representar localmente um objeto que vive em outro processo/host (RPC stubs).
- **Logging proxy** — registrar histórico de chamadas sem poluir o serviço.
- **Caching proxy** — memoizar resultados custosos com chave nos parâmetros.
- **Smart reference** — contar referências, liberar quando ninguém usa, detectar mutação.

## Quando evitar

- Não há comportamento transversal para interceptar — proxy vira boilerplate.
- A classe real poderia simplesmente ganhar a lógica (lazy, cache) sem violar SRP — o wrapper é exagero.
- Latência extra introduzida é inaceitável para o caso de uso.

## Sinais de que você está reinventando este pattern

- Decorador `*WithCache`, `*WithLogging`, `*WithAuth` envolvendo um serviço cuja interface é preservada.
- Stubs de RPC gerados que parecem o serviço remoto, mas empacotam a chamada pela rede.
- Hibernate/ORM entregando "entity proxies" que só carregam dados ao primeiro acesso — Proxy em ação.
- Service worker/CDN intermediando requisições HTTP transparentemente.

## Trade-offs

### Vantagens
- Controla o serviço sem que o cliente saiba.
- Gerencia ciclo de vida quando o cliente não quer saber dele.
- Funciona mesmo se o serviço real não está pronto/disponível (lazy, fallback).
- *Open/Closed Principle* — novos proxies entram sem alterar serviço ou cliente.

### Desvantagens
- Mais classes para manter.
- Resposta pode ficar mais lenta (overhead do proxy).
- Pilhas longas (cliente → proxy → proxy → serviço) dificultam debugging.

## Relações com outros patterns

- [[adapter]] — Adapter **altera** a interface; Proxy **preserva** a interface; [[decorator]] **estende** a interface.
- [[facade]] — ambos encapsulam entidade complexa e podem inicializá-la; diferença: Proxy tem a **mesma interface** do serviço (intercambiável), Facade define **interface nova**.
- [[decorator]] — estrutura quase idêntica, intenções diferentes: Decorator adiciona comportamento e a pilha é montada pelo **cliente**; Proxy tipicamente **gerencia sozinho** o ciclo de vida do serviço.
- [[singleton]] — proxy virtual que sempre retorna a mesma instância se aproxima de Singleton; use com critério.

## Fontes

- [Refactoring Guru — Proxy](https://refactoring.guru/design-patterns/proxy)
- Gamma, Helm, Johnson, Vlissides (1994). *Design Patterns: Elements of Reusable Object-Oriented Software*. Addison-Wesley.

## Links

- [[_moc]]
- [[structural-patterns]]
- [[decorator]] — estrutura similar, intenção distinta
- [[../_moc]] — knowledge raiz
