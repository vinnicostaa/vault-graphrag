---
title: Facade
type: pattern
tags: [pattern, structural, gof, encapsulation, subsystem]
source: https://refactoring.guru/design-patterns/facade
created: 2026-04-24
updated: 2026-04-24
aliases: [Facade Pattern]
---

# Facade

Padrão **estrutural** que oferece uma **interface simplificada** para uma biblioteca, framework ou subsistema complexo, escondendo seu ciclo de vida e suas dependências internas.

## Problema que resolve

Integrar com uma lib ou subsistema sofisticado tipicamente exige inicializar vários objetos na ordem certa, gerenciar dependências, conhecer detalhes de configuração e sequência de chamadas. Se o cliente faz isso diretamente, a lógica de negócio fica **acoplada aos detalhes do subsistema** — código difícil de entender, difícil de manter e impossível de trocar sem reescrever tudo.

A dor aumenta quando o subsistema evolui: cada mudança de API ou upgrade de versão obriga a mexer em vários pontos do cliente.

## Solução

Criar uma classe **Facade** que expõe só o subconjunto de operações que o cliente realmente precisa, escondendo o resto. Internamente a facade instancia, orquestra e finaliza os objetos do subsistema na ordem correta. O cliente passa a falar só com a facade.

É um trade-off deliberado: menos poder, mais simplicidade. Se o cliente precisa de algo fora do subconjunto, cai no subsistema direto — mas 80% dos casos ficam atendidos por uma superfície pequena e estável.

Quando a facade cresce demais, extrai-se **facades adicionais** por área funcional.

## Estrutura

- **Facade** — ponto de entrada simplificado; conhece as classes do subsistema e sabe orquestrá-las.
- **Additional Facade** (opcional) — facade secundária criada para evitar que a principal vire *god object*; pode ser usada tanto por clientes quanto pela facade principal.
- **Complex Subsystem** — conjunto de classes reais; **não** conhece a facade.
- **Client** — chama apenas métodos da facade.

## Pseudocódigo

```
// Classes do subsistema complexo (third-party, muitas peças).
class VideoFile is ...
class CodecFactory is ...
class BitrateReader is ...
class AudioMixer is ...
class MPEG4Codec is ...
class OggCodec is ...

// Facade: interface única e simples.
class VideoConverter is
    method convert(filename, format): File is
        file   = new VideoFile(filename)
        source = CodecFactory.extract(file)
        dest   = (format == "mp4") ? new MPEG4Codec() : new OggCodec()
        buffer = BitrateReader.read(filename, source)
        result = BitrateReader.convert(buffer, dest)
        result = new AudioMixer().fix(result)
        return new File(result)

// Cliente não conhece as dezenas de classes internas.
class Application is
    method main() is
        converter = new VideoConverter()
        mp4 = converter.convert("funny-cats.ogg", "mp4")
        mp4.save()
```

## Quando usar

- Você precisa de uma **interface simples** e estável sobre um subsistema complexo.
- Quer **estruturar em camadas**: cada camada expõe uma facade, camadas se comunicam só via facades, reduzindo acoplamento.
- Quer isolar o resto do código de uma lib terceira — trocar/atualizar a lib passa a ser trocar a implementação da facade.
- Só alguns poucos casos do subsistema são usados pelo resto do app — vale encapsular.

## Quando evitar

- O subsistema já é simples — facade só adiciona mais uma camada.
- Clientes precisam de quase todas as operações do subsistema — facade não economiza superfície.
- Facade está constantemente crescendo para expor novos métodos — pode virar *god object*; considere dividir em facades menores ou repensar a fronteira.

## Sinais de que você está reinventando este pattern

- Classe utilitária estática `MinhaLibHelper` que orquestra vários objetos de uma lib.
- Função `initEverything()` no bootstrap que instancia 15 objetos numa ordem frágil.
- Clientes importando muitos tipos de uma mesma biblioteca só para chamar um fluxo único.

## Trade-offs

### Vantagens
- Isola o cliente da complexidade do subsistema.
- Reduz acoplamento entre camadas da aplicação.
- Facilita upgrades/substituições de libs — só a facade muda.

### Desvantagens
- Facade pode virar *god object* se acumular responsabilidades de todo o subsistema.
- Clientes que precisam de algo fora do subconjunto exposto acabam furando a facade e falando direto com o subsistema, enfraquecendo o isolamento.

## Relações com outros patterns

- [[adapter]] — Adapter faz uma interface existente **usável** por um cliente com expectativas diferentes (geralmente um único objeto); Facade define **interface nova** sobre um subsistema inteiro.
- [[abstract-factory]] — pode substituir Facade quando o único objetivo é esconder **como** os objetos são criados.
- [[mediator]] — centraliza a comunicação entre componentes que **sabem do mediator**; Facade é usada pelo cliente e o subsistema **ignora** sua existência.
- [[flyweight]] — Flyweight mostra como ter **muitos objetos pequenos**; Facade mostra como representar um subsistema por **um único objeto**.
- [[singleton]] — Facade frequentemente é transformada em Singleton, pois uma instância costuma bastar.
- [[proxy]] — similar por também encapsular entidade complexa com inicialização própria; diferença: Proxy **preserva** a interface do serviço e é intercambiável com ele; Facade define interface nova.

## Fontes

- [Refactoring Guru — Facade](https://refactoring.guru/design-patterns/facade)
- Gamma, Helm, Johnson, Vlissides (1994). *Design Patterns: Elements of Reusable Object-Oriented Software*. Addison-Wesley.

## Links

- [[_moc]]
- [[structural-patterns]]
- [[../architecture/clean-architecture-layers]] — facades como fronteira entre camadas
- [[../_moc]] — knowledge raiz
