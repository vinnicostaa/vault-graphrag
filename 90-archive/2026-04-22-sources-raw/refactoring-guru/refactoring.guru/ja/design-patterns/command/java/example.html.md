:::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} /
[デザインパターン](../../../design-patterns.html){.type} /
[Command](../../command.html){.type} / [Java](../../java.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Command](../../../../images/patterns/cards/command-mini8482.png?id=b149eda017c0583c1e92343b83cfb1eb){srcset="/images/patterns/cards/command-mini-2x.png?id=e5f6332e057f6d352a209da963a8fc54 2x"}
:::

# **Command** を Java で {#command-を-java-で .pattern-example-title-block-title}
::::

::::::::::::::::::::::::: pattern-example-body
::: pattern-example-brief
**Command** は[、]{.chpule2}[
]{.chpuri2}振る舞いに関するデザインパターンの一つで[、]{.chpule2}[
]{.chpuri2}リクエストや簡単な操作をオブジェクトに変換します[。]{.chpule2}[
]{.chpuri2}

変換により[、]{.chpule2}[
]{.chpuri2}コマンドの遅延実行や遠隔実行を可能にしたり[、]{.chpule2}[
]{.chpuri2}コマンドの履歴の保存を可能にしたりできます[。]{.chpule2}[
]{.chpuri2}
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Command の詳細 ](../../command.html){.btn .btn-lg .btn-primary}
:::

:::::::::::::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
ナビゲーション
:::

::: en-intro
 [はじめに](#)
:::

::: en-intro
 [テキスト・エディターのコマンドと取り消し操作](#example-0)
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

**複雑度[：]{.chpule2}[ ]{.chpuri2}**[]{.fa-stars}

**人気度[：]{.chpule2}[ ]{.chpuri2}**[]{.fa-stars}

**使用例[：]{.chpule2}[ ]{.chpuri2}** Command パターンは[、]{.chpule2}[
]{.chpuri2}Java コードではよく見かけます[。]{.chpule2}[
]{.chpuri2}最もよく使われるのは[、]{.chpule2}[ ]{.chpuri2}UI
要素をアクションでパラメーター化する時のコールバックの代わりとしてです[。]{.chpule2}[
]{.chpuri2}また[、]{.chpule2}[
]{.chpuri2}タスクをキューに入れたり[、]{.chpule2}[
]{.chpuri2}操作履歴の管理などでも使われます[。]{.chpule2}[ ]{.chpuri2}

Java のコア・ライブラリーでの Command の使用例です[：]{.chpule2}[
]{.chpuri2}

-   [`java.lang.Runnable`](http://docs.oracle.com/javase/8/docs/api/java/lang/Runnable.html)
    の全実装

-   [`javax.swing.Action`](http://docs.oracle.com/javase/8/docs/api/javax/swing/Action.html)
    の全実装

**見つけ方[：]{.chpule2}[ ]{.chpuri2}**特定の操作[
]{.chpule1}[（]{.chpuri1}コピー[、]{.chpule2}[
]{.chpuri2}切り取り[、]{.chpule2}[ ]{.chpuri2}ペースト[、]{.chpule2}[
]{.chpuri2}送信[、]{.chpule2}[ ]{.chpuri2}印刷など[）]{.chpule2}[
]{.chpuri2}を表す[、]{.chpule2}[
]{.chpuri2}関連したクラスの集まりを見つけたら[、]{.chpule2}[
]{.chpuri2}Command パターンを使用しているかもしれません[。]{.chpule2}[
]{.chpuri2}これらのクラスは[、]{.chpule2}[
]{.chpuri2}同一のインターフェースか抽象クラスを実装しているはずです[。]{.chpule2}[
]{.chpuri2}コマンドは[、]{.chpule2}[
]{.chpuri2}関連する操作を独自に実装するかもしれませんし[、]{.chpule2}[
]{.chpuri2}別個のオブジェクト[[ ]{.chpule1}[（]{.chpuri1}]{.ls-05}[
]{.chpule1}[「]{.chpuri1}受け手[」]{.ls-05}[）]{.chpule2}[
]{.chpuri2}に作業を委任するかもしれません[。]{.chpule2}[
]{.chpuri2}このパターンの識別に必要な最後の段階は[、]{.chpule2}[
]{.chpuri2}インボーカー[ ]{.chpule1}[（]{.chpuri1}送り手[）]{.chpule2}[
]{.chpuri2}を特定することです[。]{.chpule2}[
]{.chpuri2}メソッドやコンストラクターのパラメーターにコマンド・オブジェクトを受け付けるようなクラスを探してみてください[。]{.chpule2}[
]{.chpuri2}
:::::::::::::::::::::::::

[]{#example-0}

## テキスト・エディターのコマンドと取り消し操作 {#example-0-title}

この例のテキスト・エディターは[、]{.chpule2}[
]{.chpuri2}ユーザーが何らかの操作をするたびに新しいコマンド・オブジェクトを作成します[。]{.chpule2}[
]{.chpuri2}コマンドは[、]{.chpule2}[
]{.chpuri2}そのアクションの実行後に[、]{.chpule2}[
]{.chpuri2}履歴スタックにプッシュされます[。]{.chpule2}[ ]{.chpuri2}

ここで[、]{.chpule2}[ ]{.chpuri2}取り消し操作を実行します[。]{.chpule2}[
]{.chpuri2}アプリケーションは[、]{.chpule2}[
]{.chpuri2}履歴から最後に実行されたコマンドを取り出し[、]{.chpule2}[
]{.chpuri2}逆のことを行うアクションを実行するか[、]{.chpule2}[
]{.chpuri2}そのコマンドによって保存されたエディターの過去の状態を復元します[。]{.chpule2}[
]{.chpuri2}

### []{#example-0--commands .anchor} **commands** {#commands .example-title}

#### []{#example-0--commands-Command-java .anchor} **commands/Command.java:** コマンドの抽象基底クラス

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

#### []{#example-0--commands-CopyCommand-java .anchor} **commands/CopyCommand.java:** 選択されたテキストをクリップボードへコピー

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

#### []{#example-0--commands-PasteCommand-java .anchor} **commands/PasteCommand.java:** クリップボードからテキストをペースト

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

#### []{#example-0--commands-CutCommand-java .anchor} **commands/CutCommand.java:** クリップボードへテキストを切り取り

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

#### []{#example-0--commands-CommandHistory-java .anchor} **commands/CommandHistory.java:** コマンド履歴

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

#### []{#example-0--editor-Editor-java .anchor} **editor/Editor.java:** テキスト・エディターの GUI

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

#### []{#example-0--Demo-java .anchor} **Demo.java:** クライアント・コード

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

#### []{#example-0--OutputDemo-png .anchor} **OutputDemo.png:** 実行結果

![](../../../../images/patterns/examples/java/command/OutputDemoefa1.png?id=f20c0baa8286ed8d9eff64c17ae9e18d)

::: next
#### 次を読む

[Iterator を Java で []{.fa
.fa-arrow-right}](../../iterator/java/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### 戻る

[[]{.fa .fa-arrow-left} Chain of Responsibility を Java
で](../../chain-of-responsibility/java/example.html){.btn .btn-default
rel="prev"}
:::

## 他言語での **Command**

[![Command を C#
で](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Command を C# で"){.prog-lang-link}
[![Command を C++
で](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Command を C++ で"){.prog-lang-link}
[![Command を Go
で](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Command を Go で"){.prog-lang-link}
[![Command を PHP
で](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Command を PHP で"){.prog-lang-link}
[![Command を Python
で](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Command を Python で"){.prog-lang-link}
[![Command を Ruby
で](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Command を Ruby で"){.prog-lang-link}
[![Command を Rust
で](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Command を Rust で"){.prog-lang-link}
[![Command を Swift
で](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Command を Swift で"){.prog-lang-link}
[![Command を TypeScript
で](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Command を TypeScript で"){.prog-lang-link}
:::::::::::::::::::::::::::::::

::::::: {.prom .banner-sidebar .banner .banner-striped .banner-examples .banner-removable .banner-removable-patterns data-id="DP: Examples-IDE" creative-id="examples-sidebar-ja" position="sidebar"}
:::::: {.banner-inner style="font-size: 14px; line-height: 21px;"}
::: banner-examples-img
[![](../../../../images/patterns/banners/examples-ide91ca.png?id=3115b4b548fb96b75974e2de8f4f49bc){width="215"
height="158" loading="lazy"
srcset="/images/patterns/banners/examples-ide-2x.png?id=93c007a6d157b97c28eb213f59d716ae 2x"}](../../book.html)
:::

::: banner-examples-fade
[ コード例の書庫](../../book.html){.btn .btn-secondary .btn-block}
:::

::: {.banner-examples-text style="margin-top: 1rem;"}
**直撃！デザインパターン**を電子書籍で購入し、IDE
で開くことができる数多くの詳細な例を含む書庫にアクセス。

[ 詳細](../../book.html){.btn .btn-secondary .btn-block}
:::
::::::
:::::::
:::::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::::
