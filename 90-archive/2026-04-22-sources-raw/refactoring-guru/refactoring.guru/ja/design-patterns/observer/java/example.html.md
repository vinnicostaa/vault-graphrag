::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} /
[デザインパターン](../../../design-patterns.html){.type} /
[Observer](../../observer.html){.type} / [Java](../../java.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Observer](../../../../images/patterns/cards/observer-minifa2e.png?id=fd2081ab1cff29c60b499bcf6a62786a){srcset="/images/patterns/cards/observer-mini-2x.png?id=f205b0655572ac8e4636691c0e0debfd 2x"}
:::

# **Observer** を Java で {#observer-を-java-で .pattern-example-title-block-title}
::::

:::::::::::::::::::::::: pattern-example-body
::: pattern-example-brief
**Observer** は[、]{.chpule2}[
]{.chpuri2}振る舞いに関するデザインパターンの一つで[、]{.chpule2}[
]{.chpuri2}オブジェクトが別のオブジェクトに状態の変化を通知できるようにします[。]{.chpule2}[
]{.chpuri2}

Observer パターンは[、]{.chpule2}[
]{.chpuri2}サブスクライバー・インターフェースを実装するいかなるオブジェクトのイベント通知の申し込みと停止をする方法を提供します[。]{.chpule2}[
]{.chpuri2}
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Observer の詳細 ](../../observer.html){.btn .btn-lg .btn-primary}
:::

::::::::::::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
ナビゲーション
:::

::: en-intro
 [はじめに](#)
:::

::: en-intro
 [イベント・サブスクリプション](#example-0)
:::

::: en-file
 publisher
:::

:::: en-inside
::: en-file
  [Event­Manager](#example-0--publisher-EventManager-java)
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
 listeners
:::

:::: en-inside
::: en-file
  [Event­Listener](#example-0--listeners-EventListener-java)
:::
::::

:::: en-inside
::: en-file
  [Email­Notification­Listener](#example-0--listeners-EmailNotificationListener-java)
:::
::::

:::: en-inside
::: en-file
  [Log­Open­Listener](#example-0--listeners-LogOpenListener-java)
:::
::::

::: en-file
 [Demo](#example-0--Demo-java)
:::

::: en-file
 [Output­Demo](#example-0--OutputDemo-txt)
:::
:::::::::::::::::::::

**複雑度[：]{.chpule2}[ ]{.chpuri2}**[]{.fa-stars}

**人気度[：]{.chpule2}[ ]{.chpuri2}**[]{.fa-stars}

**使用例[：]{.chpule2}[ ]{.chpuri2}** Observer パターンは[、]{.chpule2}[
]{.chpuri2}Java コードではよく見かけます[
]{.chpule1}[（]{.chpuri1}特に[、]{.chpule2}[ ]{.chpuri2}GUI
コンポーネントで[）]{.ls-05}[。]{.chpule2}[
]{.chpuri2}他のオブジェクトで起きるイベントに反応することを[、]{.chpule2}[
]{.chpuri2}そのクラスに結合せずに可能とします[。]{.chpule2}[ ]{.chpuri2}

Java のコア・ライブラリーでのこのパターンの使用例です[：]{.chpule2}[
]{.chpuri2}

-   [`java.util.Observer`](http://docs.oracle.com/javase/8/docs/api/java/util/Observer.html)/[`java.util.Observable`](http://docs.oracle.com/javase/8/docs/api/java/util/Observable.html)[
    ]{.chpule1}[（]{.chpuri1}実際の使用は稀[）]{.chpule2}[ ]{.chpuri2}
-   [`java.util.EventListener`](http://docs.oracle.com/javase/8/docs/api/java/util/EventListener.html)
    の全実装[ ]{.chpule1}[（]{.chpuri1}Swing
    コンポーネントいたるところ[）]{.chpule2}[ ]{.chpuri2}
-   [`javax.servlet.http.HttpSessionBindingListener`](http://docs.oracle.com/javaee/7/api/javax/servlet/http/HttpSessionBindingListener.html)
-   [`javax.servlet.http.HttpSessionAttributeListener`](http://docs.oracle.com/javaee/7/api/javax/servlet/http/HttpSessionAttributeListener.html)
-   [`javax.faces.event.PhaseListener`](http://docs.oracle.com/javaee/7/api/javax/faces/event/PhaseListener.html)

**見つけ方[：]{.chpule2}[ ]{.chpuri2}**このパターンは[、]{.chpule2}[
]{.chpuri2}オブジェクトをリストに保存するサブスクリプション・メソッドと[、]{.chpule2}[
]{.chpuri2}そのリスト中のオブジェクトの更新メソッドの呼び出しにより[、]{.chpule2}[
]{.chpuri2}識別できます[。]{.chpule2}[ ]{.chpuri2}
::::::::::::::::::::::::

[]{#example-0}

## イベント・サブスクリプション {#example-0-title}

この例では[、]{.chpule2}[ ]{.chpuri2}Observer
パターンを使い[、]{.chpule2}[
]{.chpuri2}あるテキスト・エディターのオブジェクト間の間接的共同作業を確立します[。]{.chpule2}[
]{.chpuri2}`Editor` オブジェクトに変化があるたびに[、]{.chpule2}[
]{.chpuri2}変化をサブスクライバーに通知します[。]{.chpule2}[
]{.chpuri2}`Email­Notification­Listener` と `Log­Open­Listener`
は[、]{.chpule2}[ ]{.chpuri2}その主たる振る舞いを実行して[、]{.chpule2}[
]{.chpuri2}通知に反応します[。]{.chpule2}[ ]{.chpuri2}

サブスクライバー・クラスはエディターのクラスとは結合されておらず[、]{.chpule2}[
]{.chpuri2}必要なら他のアプリでも使えます[。]{.chpule2}[
]{.chpuri2}`Editor` クラスは[、]{.chpule2}[
]{.chpuri2}抽象サブスクライバー・インターフェースにのみ依存しています[。]{.chpule2}[
]{.chpuri2}このため[、]{.chpule2}[
]{.chpuri2}エディターのコードを変更することなく[、]{.chpule2}[
]{.chpuri2}新しいサブスクライバーを追加できます[。]{.chpule2}[
]{.chpuri2}

### []{#example-0--publisher .anchor} **publisher** {#publisher .example-title}

#### []{#example-0--publisher-EventManager-java .anchor} **publisher/EventManager.java:** 基本的なパブリッシャー

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.observer.example.publisher;

import refactoring_guru.observer.example.listeners.EventListener;

import java.io.File;
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

public class EventManager {
    Map&lt;String, List&lt;EventListener&gt;&gt; listeners = new HashMap&lt;&gt;();

    public EventManager(String... operations) {
        for (String operation : operations) {
            this.listeners.put(operation, new ArrayList&lt;&gt;());
        }
    }

    public void subscribe(String eventType, EventListener listener) {
        List&lt;EventListener&gt; users = listeners.get(eventType);
        users.add(listener);
    }

    public void unsubscribe(String eventType, EventListener listener) {
        List&lt;EventListener&gt; users = listeners.get(eventType);
        users.remove(listener);
    }

    public void notify(String eventType, File file) {
        List&lt;EventListener&gt; users = listeners.get(eventType);
        for (EventListener listener : users) {
            listener.update(eventType, file);
        }
    }
}</code></pre>
</figure>

### []{#example-0--editor .anchor} **editor** {#editor .example-title}

#### []{#example-0--editor-Editor-java .anchor} **editor/Editor.java:** 他のオブジェクトから追跡される具象パブリッシャー

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.observer.example.editor;

import refactoring_guru.observer.example.publisher.EventManager;

import java.io.File;

public class Editor {
    public EventManager events;
    private File file;

    public Editor() {
        this.events = new EventManager(&quot;open&quot;, &quot;save&quot;);
    }

    public void openFile(String filePath) {
        this.file = new File(filePath);
        events.notify(&quot;open&quot;, file);
    }

    public void saveFile() throws Exception {
        if (this.file != null) {
            events.notify(&quot;save&quot;, file);
        } else {
            throw new Exception(&quot;Please open a file first.&quot;);
        }
    }
}</code></pre>
</figure>

### []{#example-0--listeners .anchor} **listeners** {#listeners .example-title}

#### []{#example-0--listeners-EventListener-java .anchor} **listeners/EventListener.java:** 共通のオブザーバー・インターフェース

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.observer.example.listeners;

import java.io.File;

public interface EventListener {
    void update(String eventType, File file);
}</code></pre>
</figure>

#### []{#example-0--listeners-EmailNotificationListener-java .anchor} **listeners/EmailNotificationListener.java:** 通知受け付け後[、]{.chpule2}[ ]{.chpuri2}メールを送信

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.observer.example.listeners;

import java.io.File;

public class EmailNotificationListener implements EventListener {
    private String email;

    public EmailNotificationListener(String email) {
        this.email = email;
    }

    @Override
    public void update(String eventType, File file) {
        System.out.println(&quot;Email to &quot; + email + &quot;: Someone has performed &quot; + eventType + &quot; operation with the following file: &quot; + file.getName());
    }
}</code></pre>
</figure>

#### []{#example-0--listeners-LogOpenListener-java .anchor} **listeners/LogOpenListener.java:** 通知受け付け後[、]{.chpule2}[ ]{.chpuri2}ログにメッセージを書き込む

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.observer.example.listeners;

import java.io.File;

public class LogOpenListener implements EventListener {
    private File log;

    public LogOpenListener(String fileName) {
        this.log = new File(fileName);
    }

    @Override
    public void update(String eventType, File file) {
        System.out.println(&quot;Save to log &quot; + log + &quot;: Someone has performed &quot; + eventType + &quot; operation with the following file: &quot; + file.getName());
    }
}</code></pre>
</figure>

#### []{#example-0--Demo-java .anchor} **Demo.java:** 初期化コード

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.observer.example;

import refactoring_guru.observer.example.editor.Editor;
import refactoring_guru.observer.example.listeners.EmailNotificationListener;
import refactoring_guru.observer.example.listeners.LogOpenListener;

public class Demo {
    public static void main(String[] args) {
        Editor editor = new Editor();
        editor.events.subscribe(&quot;open&quot;, new LogOpenListener(&quot;/path/to/log/file.txt&quot;));
        editor.events.subscribe(&quot;save&quot;, new EmailNotificationListener(&quot;admin@example.com&quot;));

        try {
            editor.openFile(&quot;test.txt&quot;);
            editor.saveFile();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}</code></pre>
</figure>

#### []{#example-0--OutputDemo-txt .anchor} **OutputDemo.txt:** 実行結果

<figure class="code">
<pre class="code" lang="output"><code>Save to log \path\to\log\file.txt: Someone has performed open operation with the following file: test.txt
Email to admin@example.com: Someone has performed save operation with the following file: test.txt</code></pre>
</figure>

::: next
#### 次を読む

[State を Java で []{.fa
.fa-arrow-right}](../../state/java/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### 戻る

[[]{.fa .fa-arrow-left} Memento を Java
で](../../memento/java/example.html){.btn .btn-default rel="prev"}
:::

## 他言語での **Observer**

[![Observer を C#
で](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Observer を C# で"){.prog-lang-link}
[![Observer を C++
で](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Observer を C++ で"){.prog-lang-link}
[![Observer を Go
で](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Observer を Go で"){.prog-lang-link}
[![Observer を PHP
で](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Observer を PHP で"){.prog-lang-link}
[![Observer を Python
で](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Observer を Python で"){.prog-lang-link}
[![Observer を Ruby
で](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Observer を Ruby で"){.prog-lang-link}
[![Observer を Rust
で](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Observer を Rust で"){.prog-lang-link}
[![Observer を Swift
で](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Observer を Swift で"){.prog-lang-link}
[![Observer を TypeScript
で](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Observer を TypeScript で"){.prog-lang-link}
::::::::::::::::::::::::::::::

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
::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::
