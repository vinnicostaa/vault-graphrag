:::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} /
[デザインパターン](../../../design-patterns.html){.type} /
[Singleton](../../singleton.html){.type} /
[C#](../../csharp.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Singleton](../../../../images/patterns/cards/singleton-minif559.png?id=914e1565dfdf15f240e766163bd303ec){srcset="/images/patterns/cards/singleton-mini-2x.png?id=4a7348083d7719d02709a5c9ff878b9f 2x"}
:::

# **Singleton** を C# で {#singleton-を-c-で .pattern-example-title-block-title}
::::

::::::::::::::: pattern-example-body
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

:::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
ナビゲーション
:::

::: en-intro
 [はじめに](#)
:::

::: en-intro
 [素朴なシングルトン](#example-0)
:::

::: en-file
 [Program](#example-0--Program-cs)
:::

::: en-file
 [Output](#example-0--Output-txt)
:::

::: en-intro
 [スレッド・セーフなシングルトン](#example-1)
:::

::: en-file
 [Program](#example-1--Program-cs)
:::

::: en-file
 [Output](#example-1--Output-txt)
:::

::: en-intro
 [もっと知りたいですか？](#example-2)
:::
::::::::::::

**複雑度[：]{.chpule2}[ ]{.chpuri2}**[]{.fa-stars}

**人気度[：]{.chpule2}[ ]{.chpuri2}**[]{.fa-stars}

**使用例[：]{.chpule2}[ ]{.chpuri2}**多くの開発者は[、]{.chpule2}[
]{.chpuri2}Singleton をアンチ・パターンと見なしています[。]{.chpule2}[
]{.chpuri2}C# コードでの使用が減少したのはこのためです[。]{.chpule2}[
]{.chpuri2}

**見つけ方[：]{.chpule2}[ ]{.chpuri2}** Singleton は[、]{.chpule2}[
]{.chpuri2}キャッシュされた同一オブジェクトを返す静的生成メソッドで識別できます[。]{.chpule2}[
]{.chpuri2}
:::::::::::::::

::: {#nav-tab .nav .nav-tabs .nav-justified role="tablist"}
[素朴なシングルトン](#example-0){#example-0-tab .nav-item .nav-link
.active toggle="tab" role="tab" aria-controls="example-0"
aria-selected="true"}
[スレッド・セーフなシングルトン](#example-1){#example-1-tab .nav-item
.nav-link toggle="tab" role="tab" aria-controls="example-1"
aria-selected="true"}
[もっと知りたいですか？](#example-2){#example-2-tab .nav-item .nav-link
toggle="tab" role="tab" aria-controls="example-2" aria-selected="true"}
:::

:::::: {#nav-tabContent .pattern-example-tab-content .tab-content}
::: {#example-0 .tab-pane .show .active role="tabpanel" aria-labelledby="example-0-tab" style="padding-top: 24px"}
## 素朴なシングルトン {#example-0-title}

いい加減なシングルトンの実装は[、]{.chpule2}[
]{.chpuri2}超簡単です[。]{.chpule2}[
]{.chpuri2}コンストラクターを隠して[、]{.chpule2}[
]{.chpuri2}静的生成メソッドを一つ書くだけです[。]{.chpule2}[ ]{.chpuri2}

同じクラスは[、]{.chpule2}[
]{.chpuri2}マルチ・スレッド環境下では正しく動作しません[。]{.chpule2}[
]{.chpuri2}複数のスレッドが[、]{.chpule2}[
]{.chpuri2}生成メソッドを同時に呼ぶことができ[、]{.chpule2}[
]{.chpuri2}シングルトン・クラスの複数のインスタンスができてしまいます[。]{.chpule2}[
]{.chpuri2}

#### []{#example-0--Program-cs .anchor} **Program.cs:** 概念的な例

<figure class="code">
<pre class="code" lang="csharp"><code>using System;

namespace RefactoringGuru.DesignPatterns.Singleton.Conceptual.NonThreadSafe
{
    // The Singleton class defines the `GetInstance` method that serves as an
    // alternative to constructor and lets clients access the same instance of
    // this class over and over.

    // EN : The Singleton should always be a &#39;sealed&#39; class to prevent class
    // inheritance through external classes and also through nested classes.
    public sealed class Singleton
    {
        // The Singleton&#39;s constructor should always be private to prevent
        // direct construction calls with the `new` operator.
        private Singleton() { }

        // The Singleton&#39;s instance is stored in a static field. There there are
        // multiple ways to initialize this field, all of them have various pros
        // and cons. In this example we&#39;ll show the simplest of these ways,
        // which, however, doesn&#39;t work really well in multithreaded program.
        private static Singleton _instance;

        // This is the static method that controls the access to the singleton
        // instance. On the first run, it creates a singleton object and places
        // it into the static field. On subsequent runs, it returns the client
        // existing object stored in the static field.
        public static Singleton GetInstance()
        {
            if (_instance == null)
            {
                _instance = new Singleton();
            }
            return _instance;
        }

        // Finally, any singleton should define some business logic, which can
        // be executed on its instance.
        public void someBusinessLogic()
        {
            // ...
        }
    }

    class Program
    {
        static void Main(string[] args)
        {
            // The client code.
            Singleton s1 = Singleton.GetInstance();
            Singleton s2 = Singleton.GetInstance();

            if (s1 == s2)
            {
                Console.WriteLine(&quot;Singleton works, both variables contain the same instance.&quot;);
            }
            else
            {
                Console.WriteLine(&quot;Singleton failed, variables contain different instances.&quot;);
            }
        }
    }
}</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** 実行結果

<figure class="code">
<pre class="code" lang="output"><code>Singleton works, both variables contain the same instance.</code></pre>
</figure>
:::

::: {#example-1 .tab-pane role="tabpanel" aria-labelledby="example-1-tab" style="padding-top: 24px"}
## スレッド・セーフなシングルトン {#example-1-title}

問題を可決するには[、]{.chpule2}[
]{.chpuri2}最初のシングルトン・オブジェクトの生成の間[、]{.chpule2}[
]{.chpuri2}スレッドを同期する必要があります[。]{.chpule2}[ ]{.chpuri2}

#### []{#example-1--Program-cs .anchor} **Program.cs:** 概念的な例

<figure class="code">
<pre class="code" lang="csharp"><code>using System;
using System.Threading;

namespace Singleton
{
    // This Singleton implementation is called &quot;double check lock&quot;. It is safe
    // in multithreaded environment and provides lazy initialization for the
    // Singleton object.
    class Singleton
    {
        private Singleton() { }

        private static Singleton _instance;

        // We now have a lock object that will be used to synchronize threads
        // during first access to the Singleton.
        private static readonly object _lock = new object();

        public static Singleton GetInstance(string value)
        {
            // This conditional is needed to prevent threads stumbling over the
            // lock once the instance is ready.
            if (_instance == null)
            {
                // Now, imagine that the program has just been launched. Since
                // there&#39;s no Singleton instance yet, multiple threads can
                // simultaneously pass the previous conditional and reach this
                // point almost at the same time. The first of them will acquire
                // lock and will proceed further, while the rest will wait here.
                lock (_lock)
                {
                    // The first thread to acquire the lock, reaches this
                    // conditional, goes inside and creates the Singleton
                    // instance. Once it leaves the lock block, a thread that
                    // might have been waiting for the lock release may then
                    // enter this section. But since the Singleton field is
                    // already initialized, the thread won&#39;t create a new
                    // object.
                    if (_instance == null)
                    {
                        _instance = new Singleton();
                        _instance.Value = value;
                    }
                }
            }
            return _instance;
        }

        // We&#39;ll use this property to prove that our Singleton really works.
        public string Value { get; set; }
    }

    class Program
    {
        static void Main(string[] args)
        {
            // The client code.
            
            Console.WriteLine(
                &quot;{0}\n{1}\n\n{2}\n&quot;,
                &quot;If you see the same value, then singleton was reused (yay!)&quot;,
                &quot;If you see different values, then 2 singletons were created (booo!!)&quot;,
                &quot;RESULT:&quot;
            );
            
            Thread process1 = new Thread(() =&gt;
            {
                TestSingleton(&quot;FOO&quot;);
            });
            Thread process2 = new Thread(() =&gt;
            {
                TestSingleton(&quot;BAR&quot;);
            });
            
            process1.Start();
            process2.Start();
            
            process1.Join();
            process2.Join();
        }
        
        public static void TestSingleton(string value)
        {
            Singleton singleton = Singleton.GetInstance(value);
            Console.WriteLine(singleton.Value);
        } 
    }
}</code></pre>
</figure>

#### []{#example-1--Output-txt .anchor} **Output.txt:** 実行結果

<figure class="code">
<pre class="code" lang="output"><code>FOO
FOO</code></pre>
</figure>
:::

::: {#example-2 .tab-pane role="tabpanel" aria-labelledby="example-2-tab" style="padding-top: 24px"}
## もっと知りたいですか？ {#example-2-title}

C# には[、]{.chpule2}[ ]{.chpuri2}もっと特殊な Singleton
パターンの変わり種があります[。]{.chpule2}[
]{.chpuri2}詳しくは[、]{.chpule2}[
]{.chpuri2}下記の記事をご覧ください[：]{.chpule2}[ ]{.chpuri2}

[C# in Depth: Implementing
Singleton](https://csharpindepth.com/Articles/Singleton)[
]{.chpule1}[（]{.chpuri1}ディープ C#[：]{.chpule2}[
]{.chpuri2}シングルトンの実装[）]{.chpule2}[ ]{.chpuri2}
:::
::::::

::: {#nav-tab .nav .nav-tabs .nav-tabs-bottom .nav-justified role="tablist"}
[素朴なシングルトン](#example-0){#example-0-tab-bottom .nav-item
.nav-link .active toggle="tab" role="tab" aria-controls="example-0"
aria-selected="true"}
[スレッド・セーフなシングルトン](#example-1){#example-1-tab-bottom
.nav-item .nav-link toggle="tab" role="tab" aria-controls="example-1"
aria-selected="true"}
[もっと知りたいですか？](#example-2){#example-2-tab-bottom .nav-item
.nav-link toggle="tab" role="tab" aria-controls="example-2"
aria-selected="true"}
:::

::: next
#### 次を読む

[Adapter を C# で []{.fa
.fa-arrow-right}](../../adapter/csharp/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### 戻る

[[]{.fa .fa-arrow-left} Prototype を C#
で](../../prototype/csharp/example.html){.btn .btn-default rel="prev"}
:::

## 他言語での **Singleton**

[![Singleton を C++
で](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Singleton を C++ で"){.prog-lang-link}
[![Singleton を Go
で](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Singleton を Go で"){.prog-lang-link}
[![Singleton を Java
で](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Singleton を Java で"){.prog-lang-link}
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
:::::::::::::::::::::::::::

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
:::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::
