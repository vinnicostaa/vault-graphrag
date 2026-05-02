---
title: Memento
type: pattern
tags: [pattern, behavioral, gof, snapshot, undo]
source: https://refactoring.guru/design-patterns/memento
created: 2026-04-24
updated: 2026-04-24
aliases: [Snapshot]
---

# Memento

Padrão **comportamental** que permite salvar e restaurar o **estado anterior** de um objeto **sem expor os detalhes de sua implementação**.

## Problema que resolve

Implementar undo/redo ou rollback transacional exige capturar snapshots do estado de um objeto. A abordagem ingênua — percorrer os campos do objeto de fora e copiá-los — **quebra encapsulamento**: obriga tornar público o que deveria ser privado, e acopla o código de snapshot à estrutura interna.

Qualquer refactor na classe alvo (campo novo, campo renomeado, campo removido) exige mudar também o código de snapshot. O snapshot em si, se exposto publicamente, permite que qualquer outro objeto leia ou adultere o estado guardado, vazando os internos.

Fica a escolha ruim: expor tudo e ficar frágil, ou esconder tudo e não conseguir salvar.

## Solução

Delegar a criação do snapshot ao **próprio dono do estado** (o **originator**). Ele tem acesso a seus campos privados e sabe o que salvar. O snapshot vai para um objeto imutável chamado **memento**, que é opaco para o mundo externo.

O **caretaker** (quem pede para salvar, ex.: histórico de comandos) guarda os mementos em uma pilha, mas só os manipula pela interface estreita (metadata). Quando quer reverter, devolve o memento ao originator, que se restaura com base nele. O estado privado nunca sai do originator.

## Estrutura

- **Originator** — cria mementos (`createSnapshot()`) e restaura-se (`restore(memento)`).
- **Memento** — value object imutável com o estado salvo; interface ampla para o originator, estreita para os demais.
- **Caretaker** — guarda mementos (stack/list); decide quando salvar e quando restaurar; não inspeciona o conteúdo.

Três variantes comuns: (1) memento como **nested class** do originator; (2) interface intermediária com só metadata pública; (3) memento linkado ao originator, com `restore()` no próprio memento.

## Pseudocódigo

```
class Editor is
    private text, cursorX, cursorY

    method createSnapshot(): Snapshot is
        return new Snapshot(this, text, cursorX, cursorY)

    method restore(text, x, y) is
        this.text = text; this.cursorX = x; this.cursorY = y

class Snapshot is  // immutable
    private editor, text, curX, curY
    constructor(e, t, x, y) is ...
    method restore() is editor.restore(text, curX, curY)

class History is
    private stack: Snapshot[]
    method save(editor) is stack.push(editor.createSnapshot())
    method undo() is
        if (!stack.empty()) stack.pop().restore()
```

## Quando usar

- Implementar undo/redo sem violar encapsulamento.
- Transações que precisam de rollback em erro.
- Checkpoints de estado para recuperação de falha.
- Serialização parcial de estado para persistência de sessão.

## Quando evitar

- Objeto é simples e seus campos já são públicos sem consequência — um `clone()` basta.
- Estado é enorme e snapshots frequentes inviabilizam memória/RAM.
- Linguagens sem controle de acesso real (Python, JS) não conseguem garantir imutabilidade do memento sem convenção.

## Sinais de que você está reinventando este pattern

- Método `getState()` / `setState()` público só para permitir histórico.
- Código externo fazendo deep-copy de objeto alheio por reflexão.
- Struct "backup" que replica todos os campos do objeto original.

## Trade-offs

### Vantagens
- Snapshot sem quebrar encapsulamento.
- Caretaker simplificado — só guarda, não interpreta.
- Originator e caretaker têm responsabilidades claras (SRP).

### Desvantagens
- Consumo de memória alto com snapshots frequentes.
- Caretaker deve gerenciar ciclo de vida dos mementos obsoletos.
- Em linguagens dinâmicas, imutabilidade é convenção, não garantia.
- Pode exigir nested classes ou ginástica de visibilidade.

## Relações com outros patterns

- [[command]] — Command + Memento para undo: o command dispara a operação, o memento guarda o estado anterior para reversão.
- [[iterator]] — memento captura estado da iteração para rollback posterior.
- [[prototype]] — alternativa mais simples quando o objeto é linearizável e não tem ligações externas complicadas.

## Fontes

- [Refactoring Guru — Memento](https://refactoring.guru/design-patterns/memento)
- Gamma, Helm, Johnson, Vlissides (1994). *Design Patterns: Elements of Reusable Object-Oriented Software*. Addison-Wesley.

## Links

- [[_moc]]
- [[behavioral-patterns]]
- [[../principles/solid]] — SRP preservando encapsulamento
- [[../_moc]] — knowledge raiz
