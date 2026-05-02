:::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Wzorce
projektowe](../../../design-patterns.html){.type} /
[Polecenie](../../command.html){.type} / [Java](../../java.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Polecenie](../../../../images/patterns/cards/command-mini8482.png?id=b149eda017c0583c1e92343b83cfb1eb){srcset="/images/patterns/cards/command-mini-2x.png?id=e5f6332e057f6d352a209da963a8fc54 2x"}
:::

# **Polecenie** w języku Java {#polecenie-w-języku-java .pattern-example-title-block-title}
::::

::::::::::::::::::::::::: pattern-example-body
::: pattern-example-brief
**Polecenie** to behawioralny wzorzec projektowy według którego żądania
lub proste działania są konwertowane na obiekty.

Wyżej wymieniona konwersja pozwala odkładać zadania w czasie,
uruchamiać je zdalnie, przechowywać ich historię, itp.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Dowiedz się więcej o Polecenie ](../../command.html){.btn .btn-lg
.btn-primary}
:::

:::::::::::::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
Nawigacja
:::

::: en-intro
 [Intro](#)
:::

::: en-intro
 [Polecenia edytora tekstu oraz funkcjonalność 'cofnij'](#example-0)
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

**Złożoność:** []{.fa-stars}

**Popularność:** []{.fa-stars}

**Przykłady użycia:** Wzorzec Polecenie jest dość powszechny w kodzie
napisanym w Javie jako alternatywa dla wywołania zwrotnego (ang.
callback) zakładającego przypisanie akcji elementom interfejsu
użytkownika w odpowiedzi na różne interakcje z tymi elementami. Jest
stosowany także w celu kolejkowania zadań, śledzenia historii wykonanych
działań, itp.

Oto parę przykładów użycia wzorca Polecenie w głównych bibliotekach
Java:

-   Wszystkie implementacje
    [`java.lang.Runnable`](http://docs.oracle.com/javase/8/docs/api/java/lang/Runnable.html)

-   Wszystkie implementacje
    [`javax.swing.Action`](http://docs.oracle.com/javase/8/docs/api/javax/swing/Action.html)

**Identyfikacja:** Jeśli widzisz zestaw powiązanych klas odpowiadających
konkretnym czynnościom (jak "Kopiuj", "Wytnij", "Wyślij", "Drukuj",
itd.) --- może to oznaczać użycie wzorca Polecenie. Takie klasy powinny
implementować ten sam interfejs/klasę abstrakcyjną. Polecenia mogą
samodzielnie implementować poszczególne czynności lub delegować zadania
osobnym obiektom --- zwanym odbiorcami. Ostatnim elementem układanki
jest znalezienie obiektu wywołującego --- będzie to klasa przyjmująca
obiekty-polecenia w charakterze parametrów swoich metod lub
konstruktora.
:::::::::::::::::::::::::

[]{#example-0}

## Polecenia edytora tekstu oraz funkcjonalność 'cofnij' {#example-0-title}

Edytor tekstu w poniższym przykładzie tworzy nowe obiekty-polecenia przy
okazji każdej interakcji użytkownika z programem. Po wykonaniu swoich
zadań, polecenie trafia na stos stanowiący historię poleceń.

Aby cofnąć czynność, aplikacja pobiera ostatnio uruchomione polecenie z
historii i albo wykonuje odwrotną czynność, albo przywraca wcześniejszy
stan edytora, zapisany przez to polecenie.

### []{#example-0--commands .anchor} **commands** {#commands .example-title}

#### []{#example-0--commands-Command-java .anchor} **commands/Command.java:** Abstrakcyjne bazowe polecenie

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

#### []{#example-0--commands-CopyCommand-java .anchor} **commands/CopyCommand.java:** Kopiuj zaznaczony tekst do schowka

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

#### []{#example-0--commands-PasteCommand-java .anchor} **commands/PasteCommand.java:** Wklej tekst ze schowka

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

#### []{#example-0--commands-CutCommand-java .anchor} **commands/CutCommand.java:** Wytnij tekst i umieść w schowku

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

#### []{#example-0--commands-CommandHistory-java .anchor} **commands/CommandHistory.java:** Historia poleceń

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

#### []{#example-0--editor-Editor-java .anchor} **editor/Editor.java:** GUI edytora tekstu

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

#### []{#example-0--Demo-java .anchor} **Demo.java:** Kod klienta

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

#### []{#example-0--OutputDemo-png .anchor} **OutputDemo.png:** Wynik działania

![](../../../../images/patterns/examples/java/command/OutputDemoefa1.png?id=f20c0baa8286ed8d9eff64c17ae9e18d)

::: next
#### Czytaj dalej

[Iterator w języku Java []{.fa
.fa-arrow-right}](../../iterator/java/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Wróć

[[]{.fa .fa-arrow-left} Łańcuch zobowiązań w języku
Java](../../chain-of-responsibility/java/example.html){.btn .btn-default
rel="prev"}
:::

## **Polecenie** w innych językach

[![Polecenie w języku
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Polecenie w języku C#"){.prog-lang-link}
[![Polecenie w języku
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Polecenie w języku C++"){.prog-lang-link}
[![Polecenie w języku
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Polecenie w języku Go"){.prog-lang-link}
[![Polecenie w języku
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Polecenie w języku PHP"){.prog-lang-link}
[![Polecenie w języku
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Polecenie w języku Python"){.prog-lang-link}
[![Polecenie w języku
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Polecenie w języku Ruby"){.prog-lang-link}
[![Polecenie w języku
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Polecenie w języku Rust"){.prog-lang-link}
[![Polecenie w języku
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Polecenie w języku Swift"){.prog-lang-link}
[![Polecenie w języku
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Polecenie w języku TypeScript"){.prog-lang-link}
:::::::::::::::::::::::::::::::

::::::: {.prom .banner-sidebar .banner .banner-striped .banner-examples .banner-removable .banner-removable-patterns data-id="DP: Examples-IDE" creative-id="examples-sidebar-pl" position="sidebar"}
:::::: {.banner-inner style="font-size: 14px; line-height: 21px;"}
::: banner-examples-img
[![](../../../../images/patterns/banners/examples-ide91ca.png?id=3115b4b548fb96b75974e2de8f4f49bc){width="215"
height="158" loading="lazy"
srcset="/images/patterns/banners/examples-ide-2x.png?id=93c007a6d157b97c28eb213f59d716ae 2x"}](../../book.html)
:::

::: banner-examples-fade
[ Archiwum\
przykładów kodu](../../book.html){.btn .btn-secondary .btn-block}
:::

::: {.banner-examples-text style="margin-top: 1rem;"}
Kupując eBook **Wzorce projektowe. Nowoczesny podręcznik** zyskujesz
dostęp do archiwum zawierającego mnóstwo szczegółowych przykładów, które
można od razu wkleić do swojego preferowanego zintegrowanego środowiska
programistycznego.

[ Dowiedz się więcej...](../../book.html){.btn .btn-secondary
.btn-block}
:::
::::::
:::::::
:::::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::::
