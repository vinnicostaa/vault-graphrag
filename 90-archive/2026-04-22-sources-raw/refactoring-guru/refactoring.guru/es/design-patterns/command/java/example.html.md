:::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Patrones de
diseño](../../../design-patterns.html){.type} /
[Command](../../command.html){.type} / [Java](../../java.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Command](../../../../images/patterns/cards/command-mini8482.png?id=b149eda017c0583c1e92343b83cfb1eb){srcset="/images/patterns/cards/command-mini-2x.png?id=e5f6332e057f6d352a209da963a8fc54 2x"}
:::

# **Command** en Java {#command-en-java .pattern-example-title-block-title}
::::

::::::::::::::::::::::::: pattern-example-body
::: pattern-example-brief
**Command** es un patrón de diseño de comportamiento que convierte
solicitudes u operaciones simples en objetos.

La conversión permite la ejecución diferida de comandos, el
almacenamiento del historial de comandos, etc.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Aprende más sobre el patrón Command ](../../command.html){.btn .btn-lg
.btn-primary}
:::

:::::::::::::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
Navegación
:::

::: en-intro
 [Intro](#)
:::

::: en-intro
 [Comandos de editor de texto y deshacer](#example-0)
:::

::: en-file
 commands
:::

:::: en-inside
::: en-file
  [Command](#example-0--commands-Command-java)
:::
::::

:::: en-inside
::: en-file
  [Copy­Command](#example-0--commands-CopyCommand-java)
:::
::::

:::: en-inside
::: en-file
  [Paste­Command](#example-0--commands-PasteCommand-java)
:::
::::

:::: en-inside
::: en-file
  [Cut­Command](#example-0--commands-CutCommand-java)
:::
::::

:::: en-inside
::: en-file
  [Command­History](#example-0--commands-CommandHistory-java)
:::
::::

::: en-file
 editor
:::

:::: en-inside
::: en-file
  [Editor](#example-0--editor-Editor-java)
:::
::::

::: en-file
 [Demo](#example-0--Demo-java)
:::

::: en-file
 [Output­Demo](#example-0--OutputDemo-png)
:::
::::::::::::::::::::::

**Complejidad:** []{.fa-stars}

**Popularidad:** []{.fa-stars}

**Ejemplos de uso:** El patrón Command es muy común en el código Java.
La mayoría de las veces se utiliza como alternativa a las retrollamadas
(*callbacks*) para parametrizar elementos UI con acciones. También se
utiliza para poner tareas en cola, realizar el seguimiento del historial
de operaciones, etc.

Aquí tienes algunos ejemplos del patrón Command en las principales
bibliotecas Java:

-   Todas las implementaciones de
    [`java.lang.Runnable`](http://docs.oracle.com/javase/8/docs/api/java/lang/Runnable.html)

-   Todas las implementaciones de
    [`javax.swing.Action`](http://docs.oracle.com/javase/8/docs/api/javax/swing/Action.html)

**Identificación:** Si ves un grupo de clases relacionadas que
representan acciones específicas (como "Copiar", "Cortar", "Enviar",
"Imprimir", etc.), puede que se trate de un patrón Command. Estas clases
deben implementar la misma interfaz/clase abstracta. Los comandos pueden
implementar las acciones relevantes por su cuenta, o delegar el trabajo
a objetos separados, que serían los receptores. La última pieza del
rompecabezas es identificar una invocadora: busca una clase que acepte
los objetos de comando de los parámetros de sus métodos o constructor.
:::::::::::::::::::::::::

[]{#example-0}

## Comandos de editor de texto y deshacer {#example-0-title}

El editor de texto de este ejemplo crea nuevos objetos de comando cada
vez que un usuario interactúa con él. Tras ejecutar sus acciones, un
comando es empujado a la pila del historial.

Ahora, para realizar la operación deshacer (undo), la aplicación toma el
último comando ejecutado del historial y, o bien realiza una acción
inversa, o bien restaura el pasado estado del editor guardado por ese
comando.

### []{#example-0--commands .anchor} **commands** {#commands .example-title}

#### []{#example-0--commands-Command-java .anchor} **commands/Command.java:** Comando base abstracto

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.command.example.commands;

import refactoring_guru.command.example.editor.Editor;

public abstract class Command {
    public Editor editor;
    private String backup;

    Command(Editor editor) {
        this.editor = editor;
    }

    void backup() {
        backup = editor.textField.getText();
    }

    public void undo() {
        editor.textField.setText(backup);
    }

    public abstract boolean execute();
}</code></pre>
</figure>

#### []{#example-0--commands-CopyCommand-java .anchor} **commands/CopyCommand.java:** Copiar el texto seleccionado en el portapapeles

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.command.example.commands;

import refactoring_guru.command.example.editor.Editor;

public class CopyCommand extends Command {

    public CopyCommand(Editor editor) {
        super(editor);
    }

    @Override
    public boolean execute() {
        editor.clipboard = editor.textField.getSelectedText();
        return false;
    }
}</code></pre>
</figure>

#### []{#example-0--commands-PasteCommand-java .anchor} **commands/PasteCommand.java:** Pegar texto desde el portapapeles

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.command.example.commands;

import refactoring_guru.command.example.editor.Editor;

public class PasteCommand extends Command {

    public PasteCommand(Editor editor) {
        super(editor);
    }

    @Override
    public boolean execute() {
        if (editor.clipboard == null || editor.clipboard.isEmpty()) return false;

        backup();
        editor.textField.insert(editor.clipboard, editor.textField.getCaretPosition());
        return true;
    }
}</code></pre>
</figure>

#### []{#example-0--commands-CutCommand-java .anchor} **commands/CutCommand.java:** Cortar texto al portapapeles

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.command.example.commands;

import refactoring_guru.command.example.editor.Editor;

public class CutCommand extends Command {

    public CutCommand(Editor editor) {
        super(editor);
    }

    @Override
    public boolean execute() {
        if (editor.textField.getSelectedText().isEmpty()) return false;

        backup();
        String source = editor.textField.getText();
        editor.clipboard = editor.textField.getSelectedText();
        editor.textField.setText(cutString(source));
        return true;
    }

    private String cutString(String source) {
        String start = source.substring(0, editor.textField.getSelectionStart());
        String end = source.substring(editor.textField.getSelectionEnd());
        return start + end;
    }
}</code></pre>
</figure>

#### []{#example-0--commands-CommandHistory-java .anchor} **commands/CommandHistory.java:** historial del comando

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.command.example.commands;

import java.util.Stack;

public class CommandHistory {
    private Stack&lt;Command&gt; history = new Stack&lt;&gt;();

    public void push(Command c) {
        history.push(c);
    }

    public Command pop() {
        return history.pop();
    }

    public boolean isEmpty() { return history.isEmpty(); }
}</code></pre>
</figure>

### []{#example-0--editor .anchor} **editor** {#editor .example-title}

#### []{#example-0--editor-Editor-java .anchor} **editor/Editor.java:** GUI del editor de texto

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.command.example.editor;

import refactoring_guru.command.example.commands.*;

import javax.swing.*;
import java.awt.*;
import java.awt.event.ActionEvent;
import java.awt.event.ActionListener;

public class Editor {
    public JTextArea textField;
    public String clipboard;
    private CommandHistory history = new CommandHistory();

    public void init() {
        JFrame frame = new JFrame(&quot;Text editor (type &amp; use buttons, Luke!)&quot;);
        JPanel content = new JPanel();
        frame.setContentPane(content);
        frame.setDefaultCloseOperation(WindowConstants.EXIT_ON_CLOSE);
        content.setLayout(new BoxLayout(content, BoxLayout.Y_AXIS));
        textField = new JTextArea();
        textField.setLineWrap(true);
        content.add(textField);
        JPanel buttons = new JPanel(new FlowLayout(FlowLayout.CENTER));
        JButton ctrlC = new JButton(&quot;Ctrl+C&quot;);
        JButton ctrlX = new JButton(&quot;Ctrl+X&quot;);
        JButton ctrlV = new JButton(&quot;Ctrl+V&quot;);
        JButton ctrlZ = new JButton(&quot;Ctrl+Z&quot;);
        Editor editor = this;
        ctrlC.addActionListener(new ActionListener() {
            @Override
            public void actionPerformed(ActionEvent e) {
                executeCommand(new CopyCommand(editor));
            }
        });
        ctrlX.addActionListener(new ActionListener() {
            @Override
            public void actionPerformed(ActionEvent e) {
                executeCommand(new CutCommand(editor));
            }
        });
        ctrlV.addActionListener(new ActionListener() {
            @Override
            public void actionPerformed(ActionEvent e) {
                executeCommand(new PasteCommand(editor));
            }
        });
        ctrlZ.addActionListener(new ActionListener() {
            @Override
            public void actionPerformed(ActionEvent e) {
                undo();
            }
        });
        buttons.add(ctrlC);
        buttons.add(ctrlX);
        buttons.add(ctrlV);
        buttons.add(ctrlZ);
        content.add(buttons);
        frame.setSize(450, 200);
        frame.setLocationRelativeTo(null);
        frame.setVisible(true);
    }

    private void executeCommand(Command command) {
        if (command.execute()) {
            history.push(command);
        }
    }

    private void undo() {
        if (history.isEmpty()) return;

        Command command = history.pop();
        if (command != null) {
            command.undo();
        }
    }
}</code></pre>
</figure>

#### []{#example-0--Demo-java .anchor} **Demo.java:** Código cliente

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.command.example;

import refactoring_guru.command.example.editor.Editor;

public class Demo {
    public static void main(String[] args) {
        Editor editor = new Editor();
        editor.init();
    }
}</code></pre>
</figure>

#### []{#example-0--OutputDemo-png .anchor} **OutputDemo.png:** Resultado de la ejecución

![](../../../../images/patterns/examples/java/command/OutputDemoefa1.png?id=f20c0baa8286ed8d9eff64c17ae9e18d)

::: next
#### Leer siguiente

[Iterator en Java []{.fa
.fa-arrow-right}](../../iterator/java/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Volver

[[]{.fa .fa-arrow-left} Chain of Responsibility en
Java](../../chain-of-responsibility/java/example.html){.btn .btn-default
rel="prev"}
:::

## **Command** en otros lenguajes

[![Command en
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Command en C#"){.prog-lang-link}
[![Command en
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Command en C++"){.prog-lang-link}
[![Command en
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Command en Go"){.prog-lang-link}
[![Command en
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Command en PHP"){.prog-lang-link}
[![Command en
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Command en Python"){.prog-lang-link}
[![Command en
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Command en Ruby"){.prog-lang-link}
[![Command en
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Command en Rust"){.prog-lang-link}
[![Command en
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Command en Swift"){.prog-lang-link}
[![Command en
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Command en TypeScript"){.prog-lang-link}
:::::::::::::::::::::::::::::::

::::::: {.prom .banner-sidebar .banner .banner-striped .banner-examples .banner-removable .banner-removable-patterns data-id="DP: Examples-IDE" creative-id="examples-sidebar-es" position="sidebar"}
:::::: {.banner-inner style="font-size: 14px; line-height: 21px;"}
::: banner-examples-img
[![](../../../../images/patterns/banners/examples-ide91ca.png?id=3115b4b548fb96b75974e2de8f4f49bc){width="215"
height="158" loading="lazy"
srcset="/images/patterns/banners/examples-ide-2x.png?id=93c007a6d157b97c28eb213f59d716ae 2x"}](../../book.html)
:::

::: banner-examples-fade
[ Archivo con ejemplos de código](../../book.html){.btn .btn-secondary
.btn-block}
:::

::: {.banner-examples-text style="margin-top: 1rem;"}
Compra el libro electrónico **Sumérgete en los patrones de diseño** para
tener acceso al archivo con decenas de ejemplos detallados que puedes
abrir en tu IDE.

[ Saber más...](../../book.html){.btn .btn-secondary .btn-block}
:::
::::::
:::::::
:::::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::::
