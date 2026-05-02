---
title: Template Method
type: pattern
tags: [pattern, behavioral, gof, inheritance, skeleton]
source: https://refactoring.guru/design-patterns/template-method
created: 2026-04-24
updated: 2026-04-24
aliases: [Template Method Pattern]
---

# Template Method

Padrão **comportamental** que define o **esqueleto de um algoritmo** em uma superclasse e permite que subclasses **sobrescrevam etapas específicas** sem alterar a estrutura do algoritmo.

## Problema que resolve

Classes que executam variantes do mesmo algoritmo acabam duplicando a estrutura comum e divergindo só em alguns passos. Exemplo clássico: parsers de documentos (PDF, DOC, CSV) — a parte de abrir/fechar arquivo e extrair dados é diferente, mas o fluxo de análise e geração de relatório é idêntico.

Deixar a duplicação cresce problemas em cascata: qualquer mudança no fluxo comum exige alterar todas as classes. Além disso, o cliente que consome essas classes polvilha `if/else` por tipo para escolher qual invocar — nada de polimorfismo real.

## Solução

Uma superclasse abstrata define o **template method**, que chama uma sequência de passos. Cada passo é um método separado que pode ser:

- **abstract** — subclasse é obrigada a implementar;
- **com implementação default** — subclasse pode sobrescrever ou não;
- **hook** — método vazio em pontos estratégicos, que subclasses podem plugar se quiserem.

O template method em si não muda. Subclasses fornecem só as partes variantes; a ordem e a estrutura do algoritmo estão trancadas na superclasse, evitando que variantes divirjam.

## Estrutura

- **Abstract Class** — declara o template method (idealmente `final`) e os passos (abstract, default, hooks).
- **Concrete Classes** — implementam os passos abstract e sobrescrevem os opcionais; **não** sobrescrevem o template method.

## Pseudocódigo

```
abstract class DataMiner is
    // Template method - não deve ser sobrescrito
    final method mine(path) is
        raw = openFile(path)
        data = extractData(raw)
        parsed = parseData(data)
        analysis = analyzeData(parsed)
        sendReport(analysis)
        closeFile(raw)

    // Steps abstract - subclasses devem implementar
    abstract method openFile(path)
    abstract method extractData(raw)
    abstract method parseData(data)
    abstract method closeFile(file)

    // Steps com default - subclasses podem sobrescrever
    method analyzeData(data) is return defaultAnalysis(data)
    method sendReport(r) is defaultReport(r)

class PDFMiner extends DataMiner is
    method openFile(p) is ...
    method extractData(r) is ...
    method parseData(d) is ...
    method closeFile(f) is ...

class CSVMiner extends DataMiner is
    method openFile(p) is ...
    // ...
```

## Quando usar

- Vários algoritmos têm a mesma estrutura mas variam em alguns passos.
- Quer permitir extensão pontual do algoritmo, não reestruturação total.
- Há duplicação gritante entre classes que fazem "quase a mesma coisa".
- Frameworks — quer oferecer "encaixes" para o usuário sem expor o fluxo interno.

## Quando evitar

- Passos variantes mudam em runtime — [[strategy]] (composição) é mais flexível.
- Sem relação hereditária natural — forçar herança só pelo pattern gera acoplamento frágil.
- Em linguagens sem herança (ou com ênfase em composição, como Go) — considerar delegation/functional options.
- Quando template method vira muitos steps (10+) — manutenção degrada rapidamente.

## Sinais de que você está reinventando este pattern

- Função que chama `step1()`, `step2()`, `step3()` e existe em múltiplas classes com corpos similares.
- Subclasses que copiam método do pai e só mudam uma chamada no meio.
- Callback chain onde a ordem de chamadas é sempre a mesma.

## Trade-offs

### Vantagens
- Elimina duplicação pulling comum para superclasse.
- Cliente controla só o que varia; estrutura permanece protegida.
- Hooks dão flexibilidade sem exigir herança em toda subclasse.

### Desvantagens
- Limita cliente ao esqueleto pré-definido — estrutura rígida.
- Risco de violar **Liskov Substitution Principle** se subclasse suprime step com semântica essencial.
- Quanto mais steps, mais difícil manter coerência.
- Depende de herança (acoplamento estático, class-level).

## Relações com outros patterns

- [[factory-method]] — especialização de Template Method aplicada a construção; um factory method pode ser um step dentro de um template method.
- [[strategy]] — ambos variam partes de comportamento. Template Method usa **herança** (estático, class-level); Strategy usa **composição** (dinâmico, object-level, trocável em runtime).

## Fontes

- [Refactoring Guru — Template Method](https://refactoring.guru/design-patterns/template-method)
- Gamma, Helm, Johnson, Vlissides (1994). *Design Patterns: Elements of Reusable Object-Oriented Software*. Addison-Wesley.

## Links

- [[_moc]]
- [[behavioral-patterns]]
- [[../principles/solid]] — OCP (extensão sem modificação) e cuidado com LSP
- [[../_moc]] — knowledge raiz
