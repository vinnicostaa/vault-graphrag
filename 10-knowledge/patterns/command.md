---
title: Command
type: pattern
tags: [pattern, behavioral, gof, undo, queue]
source: https://refactoring.guru/design-patterns/command
created: 2026-04-24
updated: 2026-04-24
aliases: [Action, Transaction]
---

# Command

Padrão **comportamental** que transforma um request em um **objeto autônomo** contendo todas as informações sobre a requisição, permitindo parametrizar, enfileirar, logar e desfazer operações.

## Problema que resolve

Quando GUI, atalhos, menus e APIs precisam disparar a mesma operação, colocar a lógica dentro do componente visual acopla apresentação a domínio e gera duplicação: `CopyButton`, `CopyMenuItem`, `CtrlCShortcut` acabariam replicando o mesmo código.

Pior: quando surge a necessidade de **undo/redo**, **fila de operações**, **execução deferida** ou **log de ações**, não há ponto único para aplicar essa infra — cada invocação tem seu próprio caminho.

Subclassificar o componente por operação explode em hierarquias frágeis; chamar diretamente a camada de negócio amarra o emissor ao receptor.

## Solução

Extrair a requisição — método, receptor, parâmetros — para um objeto **Command** com interface uniforme (tipicamente `execute()`). O emissor (invoker) dispara o command sem conhecer o receptor; o cliente monta o command com seu contexto e o injeta no emissor.

Como o command é um objeto, ele pode ser **serializado**, **enfileirado**, **armazenado em histórico**, **combinado em macros** ou **enviado pela rede**. Undo vira uma operação oposta ou restauração via [[memento]].

## Estrutura

- **Sender / Invoker** — guarda o command e o aciona; não conhece o receptor.
- **Command** (interface) — declara `execute()` (e opcionalmente `undo()`).
- **Concrete Commands** — guardam receptor + parâmetros; delegam o trabalho real ao receptor.
- **Receiver** — contém a lógica de negócio.
- **Client** — cria o concrete command, liga ao receptor, associa ao sender.

## Pseudocódigo

```
interface Command is
    method execute(): bool
    method undo()

class CopyCommand implements Command is
    private editor: Editor
    private backup: string

    constructor(editor) is this.editor = editor

    method execute() is
        backup = editor.getText()
        editor.copySelection()
        return true

    method undo() is editor.setText(backup)

class Button is
    private cmd: Command
    method setCommand(c) is this.cmd = c
    method click() is
        if (cmd.execute()) history.push(cmd)

// Client:
editor = new Editor()
copyBtn.setCommand(new CopyCommand(editor))
shortcut.on("Ctrl+C", new CopyCommand(editor))
```

## Quando usar

- Parametrizar objetos com operações (GUI buttons, menus, atalhos).
- Enfileirar, agendar, logar ou executar requisições remotamente.
- Implementar undo/redo, macros, replay de ações.
- Separar camada de UI da camada de domínio com middle layer explícito.

## Quando evitar

- Apenas um emissor chamando um receptor, sem necessidade de histórico/deferred — uma chamada direta basta.
- Linguagem tem funções de 1ª classe e não há estado a carregar — closure pode substituir Command.
- Explodir tudo em commands para operações triviais vira cerimônia sem ganho.

## Sinais de que você está reinventando este pattern

- Hierarquia de subclasses de um mesmo widget só para variar o que acontece no clique.
- Struct `{action: string, payload: any}` + `switch` gigante para dispatch.
- Event bus onde cada evento carrega referência ao método e argumentos — já é Command anêmico.
- Fila de "jobs" com campo `type` e payload — Command serializado.

## Trade-offs

### Vantagens
- **SRP** — separa quem invoca de quem executa.
- **OCP** — novos commands entram sem alterar o invoker.
- Undo/redo, macros, deferred execution, logging, remote execution.
- Commands são combináveis — composto de commands vira macro.

### Desvantagens
- Camada extra entre sender e receiver aumenta número de classes.
- Pode parecer exagerado para ações simples e únicas.
- Undo por backup consome memória; undo por operação inversa pode ser impossível.

## Relações com outros patterns

- [[chain-of-responsibility]] — handlers da CoR podem ser implementados como Commands; ou o request percorrendo a cadeia pode ser um Command.
- [[memento]] — Command + Memento para undo: command executa, memento guarda estado anterior.
- [[strategy]] — ambos parametrizam objeto com ação; Strategy descreve **formas de fazer a mesma coisa**, Command converte **uma operação específica** em objeto.
- [[prototype]] — clona commands para salvar no histórico sem mutar o original.
- [[visitor]] — visto como Command poderoso que opera sobre objetos de classes diferentes.

## Fontes

- [Refactoring Guru — Command](https://refactoring.guru/design-patterns/command)
- Gamma, Helm, Johnson, Vlissides (1994). *Design Patterns: Elements of Reusable Object-Oriented Software*. Addison-Wesley.

## Links

- [[_moc]]
- [[behavioral-patterns]]
- [[../principles/solid]] — SRP/OCP aplicados por este pattern
- [[../_moc]] — knowledge raiz
