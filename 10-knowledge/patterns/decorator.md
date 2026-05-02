---
title: Decorator
type: pattern
tags: [pattern, structural, gof, composition, wrapper]
source: https://refactoring.guru/design-patterns/decorator
created: 2026-04-24
updated: 2026-04-24
aliases: [Decorator Pattern, Wrapper]
---

# Decorator

Padrão **estrutural** que anexa novos comportamentos a um objeto **envolvendo-o em wrappers** que compartilham sua interface — tudo em tempo de execução, sem herança.

## Problema que resolve

Estender comportamento via herança tem limites concretos: é **estática** (não dá pra trocar o comportamento de um objeto já existente) e, na maioria das linguagens, só uma superclasse é permitida. Se você precisa combinar várias responsabilidades opcionais (ex.: notificação por email + SMS + Slack; leitura de arquivo + compressão + criptografia), criar uma subclasse pra cada combinação produz **explosão combinatória**.

Além disso, classes marcadas como `final`/`sealed` não podem ser estendidas — a única saída é envolvê-las.

## Solução

Usar **composição** em vez de herança. O wrapper implementa a mesma interface do objeto alvo e mantém uma referência a ele. Cada chamada é delegada ao alvo, mas o wrapper pode executar algo **antes ou depois** — adicionando compressão, log, cache, criptografia, tradução, etc.

Como wrapper e alvo implementam a mesma interface, wrappers podem envolver outros wrappers, formando uma **pilha** configurada em runtime. O cliente interage com o topo da pilha sem saber quantas camadas existem.

## Estrutura

- **Component** — interface comum para o objeto envolvido e seus wrappers.
- **Concrete Component** — implementação básica; o objeto que será decorado.
- **Base Decorator** — guarda referência a um `Component` e delega todas as operações; facilita a escrita de decorators concretos.
- **Concrete Decorator** — sobrescreve métodos do Base Decorator para executar comportamento antes/depois da chamada ao objeto envolto.
- **Client** — monta a pilha (ordem importa) e só fala com a interface `Component`.

## Pseudocódigo

```
interface DataSource is
    method writeData(data)
    method readData(): data

class FileDataSource implements DataSource is
    constructor(filename) { ... }
    method writeData(data) is /* escrever em arquivo */
    method readData() is /* ler do arquivo */

class DataSourceDecorator implements DataSource is
    protected field wrappee: DataSource
    constructor(source) is this.wrappee = source
    method writeData(data) is wrappee.writeData(data)
    method readData() is return wrappee.readData()

class EncryptionDecorator extends DataSourceDecorator is
    method writeData(data) is
        encrypted = encrypt(data)
        wrappee.writeData(encrypted)
    method readData() is
        return decrypt(wrappee.readData())

class CompressionDecorator extends DataSourceDecorator is
    method writeData(data) is wrappee.writeData(compress(data))
    method readData() is return decompress(wrappee.readData())

// Montagem: Encryption(Compression(File))
source = new EncryptionDecorator(
            new CompressionDecorator(
                new FileDataSource("salary.dat")))
source.writeData(records)
```

## Quando usar

- Você precisa atribuir comportamentos extras a objetos **em runtime**, sem quebrar o código que já usa esses objetos.
- A combinação de comportamentos é grande, mas cada comportamento é opcional (logging, cache, métricas, retry, rate-limit).
- Extender a classe via herança é inviável (classe `final`, terceiros, linguagem sem herança múltipla).
- Middlewares de HTTP/RPC são um exemplo prático de Decorator aplicado a handlers.

## Quando evitar

- A ordem dos decorators é relevante mas difícil de garantir na prática — tende a gerar bugs sutis.
- Só existe uma ou duas variantes de comportamento fixas — herança simples ou [[strategy]] bastam.
- A pilha fica profunda demais e dificulta debugging (stack traces enormes sem contexto).

## Sinais de que você está reinventando este pattern

- `if (logging) then ...; if (cache) then ...` repetidos dentro de um serviço que deveria ser agnóstico.
- Classe base com flags booleanas que ligam/desligam funcionalidades.
- Subclasses com nomes combinatórios (`LoggedCachedSecuredService`).

## Trade-offs

### Vantagens
- Estende comportamento sem criar subclasses.
- Adiciona/remove responsabilidades em runtime.
- Combina múltiplos comportamentos empilhando wrappers.
- *Single Responsibility Principle* — cada decorator faz uma coisa.

### Desvantagens
- Remover um wrapper específico do meio da pilha é difícil.
- Comportamento pode depender da **ordem** dos decorators — difícil de fazer comutativo.
- Código de configuração inicial da pilha pode ficar feio.
- Debug de pilha profunda é custoso.

## Relações com outros patterns

- [[adapter]] — altera a **interface** de um objeto; Decorator **mantém ou estende** a mesma interface. Decorator suporta composição recursiva; Adapter geralmente não.
- [[proxy]] — estrutura similar, **intenção diferente**: Proxy controla ciclo de vida/acesso do serviço (tipicamente criado pelo próprio proxy); Decorator é sempre composto pelo cliente e adiciona responsabilidade.
- [[composite]] — mesma forma recursiva; Decorator tem **um** filho e adiciona comportamento, Composite tem **vários** e agrega. Podem cooperar: decorar um nó específico da árvore.
- [[chain-of-responsibility]] — estrutura parecida (composição recursiva), mas handlers de CoR podem **interromper** a requisição, decorators não.
- [[strategy]] — Decorator troca a "pele" do objeto; Strategy troca o "miolo".
- [[prototype]] — útil para clonar pilhas de decorators complexas.

## Fontes

- [Refactoring Guru — Decorator](https://refactoring.guru/design-patterns/decorator)
- Gamma, Helm, Johnson, Vlissides (1994). *Design Patterns: Elements of Reusable Object-Oriented Software*. Addison-Wesley.

## Links

- [[_moc]]
- [[structural-patterns]]
- [[proxy]] — estrutura similar, intenção distinta
- [[../_moc]] — knowledge raiz
