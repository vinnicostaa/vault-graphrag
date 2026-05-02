::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} /
[デザインパターン](../../../design-patterns.html){.type} /
[Singleton](../../singleton.html){.type} /
[Java](../../java.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Singleton](../../../../images/patterns/cards/singleton-minif559.png?id=914e1565dfdf15f240e766163bd303ec){srcset="/images/patterns/cards/singleton-mini-2x.png?id=4a7348083d7719d02709a5c9ff878b9f 2x"}
:::

# **Singleton** を Java で {#singleton-を-java-で .pattern-example-title-block-title}
::::

::::::::::::::::::::: pattern-example-body
::: pattern-example-brief
**Singleton** は[、]{.chpule2}[
]{.chpuri2}生成に関するデザインパターンの一つで[、]{.chpule2}[
]{.chpuri2}この種類のオブジェクトがただ一つだけ存在することを保証し[、]{.chpule2}[
]{.chpuri2}他のコードに対して唯一のアクセス・ポイントを提供します[。]{.chpule2}[
]{.chpuri2}

Singleton には[、]{.chpule2}[
]{.chpuri2}大域変数とほぼ同じ長所と短所があります[。]{.chpule2}[
]{.chpuri2}両方とも随分と便利ですが[、]{.chpule2}[
]{.chpuri2}コードのモジュール性を犠牲にしています[。]{.chpule2}[
]{.chpuri2}

シングルトンのクラスに依存しているあるクラスを使う場合[、]{.chpule2}[
]{.chpuri2}シングルトンのクラスも一緒に使う必要があります[。]{.chpule2}[
]{.chpuri2}ほとんどの場合[、]{.chpule2}[
]{.chpuri2}この制限は[、]{.chpule2}[
]{.chpuri2}ユニット・テストの作成で問題となります[。]{.chpule2}[
]{.chpuri2}
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Singleton の詳細 ](../../singleton.html){.btn .btn-lg .btn-primary}
:::

:::::::::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
ナビゲーション
:::

::: en-intro
 [はじめに](#)
:::

::: en-intro
 [素朴なシングルトン（シングル・スレッド）](#example-0)
:::

::: en-file
 [Singleton](#example-0--Singleton-java)
:::

::: en-file
 [Demo­Single­Thread](#example-0--DemoSingleThread-java)
:::

::: en-file
 [Output­Demo­Single­Thread](#example-0--OutputDemoSingleThread-txt)
:::

::: en-intro
 [素朴なシングルトン（マルチ・スレッド）](#example-1)
:::

::: en-file
 [Singleton](#example-1--Singleton-java)
:::

::: en-file
 [Demo­Multi­Thread](#example-1--DemoMultiThread-java)
:::

::: en-file
 [Output­Demo­Multi­Thread](#example-1--OutputDemoMultiThread-txt)
:::

::: en-intro
 [遅延ロード付き、スレッド・セーフなシングルトン](#example-2)
:::

::: en-file
 [Singleton](#example-2--Singleton-java)
:::

::: en-file
 [Demo­Multi­Thread](#example-2--DemoMultiThread-java)
:::

::: en-file
 [Output­Demo­Multi­Thread](#example-2--OutputDemoMultiThread-txt)
:::

::: en-intro
 [もっと知りたいですか？](#example-3)
:::
::::::::::::::::::

**複雑度[：]{.chpule2}[ ]{.chpuri2}**[]{.fa-stars}

**人気度[：]{.chpule2}[ ]{.chpuri2}**[]{.fa-stars}

**使用例[：]{.chpule2}[ ]{.chpuri2}**多くの開発者は[、]{.chpule2}[
]{.chpuri2}Singleton をアンチ・パターンと見なしています[。]{.chpule2}[
]{.chpuri2}Java コードでの使用が減少したのはこのためです[。]{.chpule2}[
]{.chpuri2}

にもかかわらず[、]{.chpule2}[ ]{.chpuri2}Java
のコア・ライブラリーには[、]{.chpule2}[ ]{.chpuri2}Singleton
利用の例が結構たくさんあります[：]{.chpule2}[ ]{.chpuri2}

-   [`java.lang.Runtime#getRuntime()`](http://docs.oracle.com/javase/8/docs/api/java/lang/Runtime.html#getRuntime--)
-   [`java.awt.Desktop#getDesktop()`](http://docs.oracle.com/javase/8/docs/api/java/awt/Desktop.html#getDesktop--)
-   [`java.lang.System#getSecurityManager()`](http://docs.oracle.com/javase/8/docs/api/java/lang/System.html#getSecurityManager--)

**見つけ方[：]{.chpule2}[ ]{.chpuri2}** Singleton は[、]{.chpule2}[
]{.chpuri2}キャッシュされた同一オブジェクトを返す静的生成メソッドで識別できます[。]{.chpule2}[
]{.chpuri2}
:::::::::::::::::::::

::: {#nav-tab .nav .nav-tabs .nav-justified role="tablist"}
[素朴なシングルトン（シングル・スレッド）](#example-0){#example-0-tab
.nav-item .nav-link .active toggle="tab" role="tab"
aria-controls="example-0" aria-selected="true"}
[素朴なシングルトン（マルチ・スレッド）](#example-1){#example-1-tab
.nav-item .nav-link toggle="tab" role="tab" aria-controls="example-1"
aria-selected="true"}
[遅延ロード付き、スレッド・セーフなシングルトン](#example-2){#example-2-tab
.nav-item .nav-link toggle="tab" role="tab" aria-controls="example-2"
aria-selected="true"}
[もっと知りたいですか？](#example-3){#example-3-tab .nav-item .nav-link
toggle="tab" role="tab" aria-controls="example-3" aria-selected="true"}
:::

::::::: {#nav-tabContent .pattern-example-tab-content .tab-content}
::: {#example-0 .tab-pane .show .active role="tabpanel" aria-labelledby="example-0-tab" style="padding-top: 24px"}
## 素朴なシングルトン（シングル・スレッド） {#example-0-title}

いい加減なシングルトンの実装は[、]{.chpule2}[
]{.chpuri2}超簡単です[。]{.chpule2}[
]{.chpuri2}コンストラクターを隠して[、]{.chpule2}[
]{.chpuri2}静的生成メソッドを一つ書くだけです[。]{.chpule2}[ ]{.chpuri2}

#### []{#example-0--Singleton-java .anchor} **Singleton.java:** シングルトン

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.singleton.example.non_thread_safe;

public final class Singleton {
    private static Singleton instance;
    public String value;

    private Singleton(String value) {
        // The following code emulates slow initialization.
        try {
            Thread.sleep(1000);
        } catch (InterruptedException ex) {
            ex.printStackTrace();
        }
        this.value = value;
    }

    public static Singleton getInstance(String value) {
        if (instance == null) {
            instance = new Singleton(value);
        }
        return instance;
    }
}</code></pre>
</figure>

#### []{#example-0--DemoSingleThread-java .anchor} **DemoSingleThread.java:** クライアント・コード

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.singleton.example.non_thread_safe;

public class DemoSingleThread {
    public static void main(String[] args) {
        System.out.println(&quot;If you see the same value, then singleton was reused (yay!)&quot; + &quot;\n&quot; +
                &quot;If you see different values, then 2 singletons were created (booo!!)&quot; + &quot;\n\n&quot; +
                &quot;RESULT:&quot; + &quot;\n&quot;);
        Singleton singleton = Singleton.getInstance(&quot;FOO&quot;);
        Singleton anotherSingleton = Singleton.getInstance(&quot;BAR&quot;);
        System.out.println(singleton.value);
        System.out.println(anotherSingleton.value);
    }
}</code></pre>
</figure>

#### []{#example-0--OutputDemoSingleThread-txt .anchor} **OutputDemoSingleThread.txt:** 実行結果

<figure class="code">
<pre class="code" lang="output"><code>If you see the same value, then singleton was reused (yay!)
If you see different values, then 2 singletons were created (booo!!)

RESULT:

FOO
FOO</code></pre>
</figure>
:::

::: {#example-1 .tab-pane role="tabpanel" aria-labelledby="example-1-tab" style="padding-top: 24px"}
## 素朴なシングルトン（マルチ・スレッド） {#example-1-title}

同じクラスは[、]{.chpule2}[
]{.chpuri2}マルチ・スレッド環境下では正しく動作しません[。]{.chpule2}[
]{.chpuri2}複数のスレッドが[、]{.chpule2}[
]{.chpuri2}生成メソッドを同時に呼ぶことができ[、]{.chpule2}[
]{.chpuri2}シングルトン・クラスの複数のインスタンスができてしまいます[。]{.chpule2}[
]{.chpuri2}

#### []{#example-1--Singleton-java .anchor} **Singleton.java:** シングルトン

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.singleton.example.non_thread_safe;

public final class Singleton {
    private static Singleton instance;
    public String value;

    private Singleton(String value) {
        // The following code emulates slow initialization.
        try {
            Thread.sleep(1000);
        } catch (InterruptedException ex) {
            ex.printStackTrace();
        }
        this.value = value;
    }

    public static Singleton getInstance(String value) {
        if (instance == null) {
            instance = new Singleton(value);
        }
        return instance;
    }
}</code></pre>
</figure>

#### []{#example-1--DemoMultiThread-java .anchor} **DemoMultiThread.java:** クライアント・コード

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.singleton.example.non_thread_safe;

public class DemoMultiThread {
    public static void main(String[] args) {
        System.out.println(&quot;If you see the same value, then singleton was reused (yay!)&quot; + &quot;\n&quot; +
                &quot;If you see different values, then 2 singletons were created (booo!!)&quot; + &quot;\n\n&quot; +
                &quot;RESULT:&quot; + &quot;\n&quot;);
        Thread threadFoo = new Thread(new ThreadFoo());
        Thread threadBar = new Thread(new ThreadBar());
        threadFoo.start();
        threadBar.start();
    }

    static class ThreadFoo implements Runnable {
        @Override
        public void run() {
            Singleton singleton = Singleton.getInstance(&quot;FOO&quot;);
            System.out.println(singleton.value);
        }
    }

    static class ThreadBar implements Runnable {
        @Override
        public void run() {
            Singleton singleton = Singleton.getInstance(&quot;BAR&quot;);
            System.out.println(singleton.value);
        }
    }
}</code></pre>
</figure>

#### []{#example-1--OutputDemoMultiThread-txt .anchor} **OutputDemoMultiThread.txt:** 実行結果

<figure class="code">
<pre class="code" lang="output"><code>If you see the same value, then singleton was reused (yay!)
If you see different values, then 2 singletons were created (booo!!)

RESULT:

FOO
BAR</code></pre>
</figure>
:::

::: {#example-2 .tab-pane role="tabpanel" aria-labelledby="example-2-tab" style="padding-top: 24px"}
## 遅延ロード付き、スレッド・セーフなシングルトン {#example-2-title}

問題を可決するには[、]{.chpule2}[
]{.chpuri2}最初のシングルトン・オブジェクトの生成の間[、]{.chpule2}[
]{.chpuri2}スレッドを同期する必要があります[。]{.chpule2}[ ]{.chpuri2}

#### []{#example-2--Singleton-java .anchor} **Singleton.java:** シングルトン

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.singleton.example.thread_safe;

public final class Singleton {
    // The field must be declared volatile so that double check lock would work
    // correctly.
    private static volatile Singleton instance;

    public String value;

    private Singleton(String value) {
        this.value = value;
    }

    public static Singleton getInstance(String value) {
        // The approach taken here is called double-checked locking (DCL). It
        // exists to prevent race condition between multiple threads that may
        // attempt to get singleton instance at the same time, creating separate
        // instances as a result.
        //
        // It may seem that having the `result` variable here is completely
        // pointless. There is, however, a very important caveat when
        // implementing double-checked locking in Java, which is solved by
        // introducing this local variable.
        //
        // You can read more info DCL issues in Java here:
        // https://refactoring.guru/java-dcl-issue
        Singleton result = instance;
        if (result != null) {
            return result;
        }
        synchronized(Singleton.class) {
            if (instance == null) {
                instance = new Singleton(value);
            }
            return instance;
        }
    }
}</code></pre>
</figure>

#### []{#example-2--DemoMultiThread-java .anchor} **DemoMultiThread.java:** クライアント・コード

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.singleton.example.thread_safe;

public class DemoMultiThread {
    public static void main(String[] args) {
        System.out.println(&quot;If you see the same value, then singleton was reused (yay!)&quot; + &quot;\n&quot; +
                &quot;If you see different values, then 2 singletons were created (booo!!)&quot; + &quot;\n\n&quot; +
                &quot;RESULT:&quot; + &quot;\n&quot;);
        Thread threadFoo = new Thread(new ThreadFoo());
        Thread threadBar = new Thread(new ThreadBar());
        threadFoo.start();
        threadBar.start();
    }

    static class ThreadFoo implements Runnable {
        @Override
        public void run() {
            Singleton singleton = Singleton.getInstance(&quot;FOO&quot;);
            System.out.println(singleton.value);
        }
    }

    static class ThreadBar implements Runnable {
        @Override
        public void run() {
            Singleton singleton = Singleton.getInstance(&quot;BAR&quot;);
            System.out.println(singleton.value);
        }
    }
}</code></pre>
</figure>

#### []{#example-2--OutputDemoMultiThread-txt .anchor} **OutputDemoMultiThread.txt:** 実行結果

<figure class="code">
<pre class="code" lang="output"><code>If you see the same value, then singleton was reused (yay!)
If you see different values, then 2 singletons were created (booo!!)

RESULT:

BAR
BAR</code></pre>
</figure>
:::

::: {#example-3 .tab-pane role="tabpanel" aria-labelledby="example-3-tab" style="padding-top: 24px"}
## もっと知りたいですか？ {#example-3-title}

Java には[、]{.chpule2}[ ]{.chpuri2}もっと特殊な Singleton
パターンの変わり種があります[。]{.chpule2}[
]{.chpuri2}詳しくは[、]{.chpule2}[
]{.chpuri2}下記の記事をご覧ください[：]{.chpule2}[ ]{.chpuri2}

[Java Singleton Design Pattern Best Practices with
Examples](https://www.journaldev.com/1377/java-singleton-design-pattern-best-practices-examples)[
]{.chpule1}[（]{.chpuri1}例題付き[、]{.chpule2}[ ]{.chpuri2}Java の
Singleton デザインパターンのベスト・プラクティス[）]{.chpule2}[
]{.chpuri2}
:::
:::::::

::: {#nav-tab .nav .nav-tabs .nav-tabs-bottom .nav-justified role="tablist"}
[素朴なシングルトン（シングル・スレッド）](#example-0){#example-0-tab-bottom
.nav-item .nav-link .active toggle="tab" role="tab"
aria-controls="example-0" aria-selected="true"}
[素朴なシングルトン（マルチ・スレッド）](#example-1){#example-1-tab-bottom
.nav-item .nav-link toggle="tab" role="tab" aria-controls="example-1"
aria-selected="true"}
[遅延ロード付き、スレッド・セーフなシングルトン](#example-2){#example-2-tab-bottom
.nav-item .nav-link toggle="tab" role="tab" aria-controls="example-2"
aria-selected="true"}
[もっと知りたいですか？](#example-3){#example-3-tab-bottom .nav-item
.nav-link toggle="tab" role="tab" aria-controls="example-3"
aria-selected="true"}
:::

::: next
#### 次を読む

[Adapter を Java で []{.fa
.fa-arrow-right}](../../adapter/java/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### 戻る

[[]{.fa .fa-arrow-left} Prototype を Java
で](../../prototype/java/example.html){.btn .btn-default rel="prev"}
:::

## 他言語での **Singleton**

[![Singleton を C#
で](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Singleton を C# で"){.prog-lang-link}
[![Singleton を C++
で](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Singleton を C++ で"){.prog-lang-link}
[![Singleton を Go
で](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Singleton を Go で"){.prog-lang-link}
[![Singleton を PHP
で](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Singleton を PHP で"){.prog-lang-link}
[![Singleton を Python
で](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Singleton を Python で"){.prog-lang-link}
[![Singleton を Ruby
で](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Singleton を Ruby で"){.prog-lang-link}
[![Singleton を Rust
で](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Singleton を Rust で"){.prog-lang-link}
[![Singleton を Swift
で](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Singleton を Swift で"){.prog-lang-link}
[![Singleton を TypeScript
で](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Singleton を TypeScript で"){.prog-lang-link}
::::::::::::::::::::::::::::::::::

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
::::::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::::::
