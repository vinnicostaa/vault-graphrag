---
title: Factory Method
type: pattern
tags: [pattern, creational, gof, inheritance, extensibility]
source: https://refactoring.guru/design-patterns/factory-method
created: 2026-04-24
updated: 2026-04-24
aliases: [Virtual Constructor, Factory Method Pattern]
---

# Factory Method

Padrão **criacional** que fornece uma interface para **criar objetos em uma superclasse**, permitindo que **subclasses decidam o tipo concreto** do objeto produzido.

## Problema que resolve

Uma classe orquestra lógica de negócio que depende de um tipo concreto específico (`new Truck()` espalhado por várias ramificações). Quando surge a necessidade de outro tipo equivalente (`Ship`, `Train`), o código acaba poluído por condicionais e por acoplamento direto entre a lógica e as classes concretas.

Adicionar um novo tipo exige tocar em toda a codebase, e qualquer nova variante futura repete o mesmo esforço. É preciso isolar a **decisão de qual tipo instanciar** do código que **usa** o objeto.

## Solução

Substituir chamadas diretas a `new` por chamadas a um **método de fábrica** declarado na superclasse (*creator*). O método retorna um tipo abstrato (interface ou classe base); subclasses sobrescrevem o método para retornar uma implementação concreta diferente.

O cliente interage com o creator pela interface comum e nunca vê a classe concreta do produto — o polimorfismo no método de fábrica é o que permite trocar a variante sem alterar o cliente.

Restrição: todas as variantes de produto precisam compartilhar uma **interface/base comum**, e o tipo de retorno do método de fábrica tem de ser essa interface.

## Estrutura

- **Product** (interface) — declara operações comuns a todos os objetos que o creator produz.
- **Concrete Products** — implementações específicas da interface Product.
- **Creator** — classe que declara o método de fábrica retornando Product; contém a lógica de negócio que depende do produto. A criação não é sua responsabilidade primária.
- **Concrete Creators** — sobrescrevem o método de fábrica para retornar uma variante específica de produto.

## Pseudocódigo

```
interface Button is
    method render()
    method onClick(f)

class WindowsButton implements Button is ...
class HTMLButton    implements Button is ...

// Creator: contém lógica de negócio; delega criação ao método de fábrica.
abstract class Dialog is
    abstract method createButton(): Button

    method render() is
        Button ok = createButton()
        ok.onClick(closeDialog)
        ok.render()

class WindowsDialog extends Dialog is
    method createButton(): Button is return new WindowsButton()

class WebDialog extends Dialog is
    method createButton(): Button is return new HTMLButton()

// Cliente escolhe o creator no boot; resto do código usa só a interface.
dialog = (config.OS == "Windows") ? new WindowsDialog() : new WebDialog()
dialog.render()
```

## Quando usar

- Não se sabe de antemão os tipos concretos com os quais o código precisa trabalhar — quer deixar o ponto de extensão explícito.
- Você publica lib/framework e quer dar aos usuários um jeito padronizado de estender componentes internos sem forkar.
- Precisa economizar recursos reaproveitando objetos de um pool, cache ou outra fonte — o método de fábrica não obriga a retornar instância nova a cada chamada.

## Quando evitar

- Há só uma variante de produto e nenhuma previsão real de uma segunda — criar creator + hierarquia agrega complexidade sem ganho.
- A lógica do creator é trivial e consiste apenas em "retornar `new X`" — um simples construtor ou função-fábrica resolve sem hierarquia.
- Linguagens com funções de 1ª classe: uma função-fábrica passada como parâmetro pode substituir a hierarquia de creators.

## Sinais de que você está reinventando este pattern

- Classe com método `createXxx()` retornando um tipo abstrato, sobrescrito em subclasses para devolver implementação concreta.
- `switch`/`if` sobre `type` dentro do código que **também usa** o objeto criado — candidato a extrair para método sobrescrito em subclasses.
- Lib/framework com "hook" para o usuário prover objeto de tipo customizado.

## Trade-offs

### Vantagens
- Evita acoplamento forte entre creator e produtos concretos.
- **Single Responsibility Principle** — código de criação concentrado num ponto.
- **Open/Closed Principle** — novos tipos de produto entram com nova subclasse de creator, sem alterar clientes existentes.

### Desvantagens
- Aumenta número de classes (nova hierarquia paralela de creators). Melhor caso é aplicar em hierarquia de creators que já existe por outro motivo.

## Relações com outros patterns

- [[abstract-factory]] — frequentemente montada sobre um conjunto de Factory Methods; Abstract Factory cria **famílias** de produtos, Factory Method cria **um** tipo.
- [[prototype]] — alternativa baseada em cópia (não em herança); não requer hierarquia de creators, mas exige inicialização complexa do clone.
- [[builder]] — Builder foca construção **passo a passo** de objeto complexo; Factory Method retorna o produto em uma chamada.
- [[template-method]] — Factory Method é frequentemente uma especialização de Template Method; pode ser um passo dentro de um Template Method maior.
- [[iterator]] — Factory Method serve para subclasses de coleção retornarem iterators compatíveis com elas.

## Fontes

- [Refactoring Guru — Factory Method](https://refactoring.guru/design-patterns/factory-method)
- Gamma, Helm, Johnson, Vlissides (1994). *Design Patterns: Elements of Reusable Object-Oriented Software*. Addison-Wesley.

## Links

- [[_moc]]
- [[creational-patterns]]
- [[../principles/solid]] — OCP e SRP aplicados por este pattern
- [[../_moc]] — knowledge raiz
