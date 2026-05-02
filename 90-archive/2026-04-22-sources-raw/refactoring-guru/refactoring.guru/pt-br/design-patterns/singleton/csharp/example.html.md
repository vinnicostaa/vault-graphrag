:::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Padrões de
Projeto](../../../design-patterns.html){.type} /
[Singleton](../../singleton.html){.type} /
[C#](../../csharp.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Singleton](../../../../images/patterns/cards/singleton-minif559.png?id=914e1565dfdf15f240e766163bd303ec){srcset="/images/patterns/cards/singleton-mini-2x.png?id=4a7348083d7719d02709a5c9ff878b9f 2x"}
:::

# **Singleton** em C# {#singleton-em-c .pattern-example-title-block-title}
::::

::::::::::::::: pattern-example-body
::: pattern-example-brief
O **Singleton** é um padrão de projeto criacional, que garante que
apenas um objeto desse tipo exista e forneça um único ponto de acesso a
ele para qualquer outro código.

O Singleton tem quase os mesmos prós e contras que as variáveis globais.
Embora sejam super úteis, eles quebram a modularidade do seu código.

Você pode usar classes que dependem de singletons em algumas outras
situações. Você terá que levar a classe singleton também. Na maioria das
vezes, essa limitação surge durante a criação de testes de unidade.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Saiba mais sobre o Singleton ](../../singleton.html){.btn .btn-lg
.btn-primary}
:::

:::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
Navegação
:::

::: en-intro
 [Introdução](#)
:::

::: en-intro
 [Singleton ingênuo](#example-0)
:::

::: en-file
 [Program](#example-0--Program-cs)
:::

::: en-file
 [Output](#example-0--Output-txt)
:::

::: en-intro
 [Singleton seguro para threads](#example-1)
:::

::: en-file
 [Program](#example-1--Program-cs)
:::

::: en-file
 [Output](#example-1--Output-txt)
:::

::: en-intro
 [Quer mais?](#example-2)
:::
::::::::::::

**Complexidade:** []{.fa-stars}

**Popularidade:** []{.fa-stars}

**Exemplos de uso:** Muitos desenvolvedores consideram o padrão
Singleton um antipadrão. É por isso que seu uso está diminuindo no
código C#.

**Identificação:** O Singleton pode ser reconhecido por um método de
criação estático, que retorna o mesmo objeto em cache.
:::::::::::::::

::: {#nav-tab .nav .nav-tabs .nav-justified role="tablist"}
[Singleton ingênuo](#example-0){#example-0-tab .nav-item .nav-link
.active toggle="tab" role="tab" aria-controls="example-0"
aria-selected="true"} [Singleton seguro para
threads](#example-1){#example-1-tab .nav-item .nav-link toggle="tab"
role="tab" aria-controls="example-1" aria-selected="true"} [Quer
mais?](#example-2){#example-2-tab .nav-item .nav-link toggle="tab"
role="tab" aria-controls="example-2" aria-selected="true"}
:::

:::::: {#nav-tabContent .pattern-example-tab-content .tab-content}
::: {#example-0 .tab-pane .show .active role="tabpanel" aria-labelledby="example-0-tab" style="padding-top: 24px"}
## Singleton ingênuo {#example-0-title}

É muito fácil implementar um Singleton desleixado. Você só precisa
ocultar o construtor e implementar um método de criação estático.

A mesma classe se comporta incorretamente em um ambiente multithread.
Vários threads podem chamar o método de criação simultaneamente e obter
várias instâncias da classe Singleton.

#### []{#example-0--Program-cs .anchor} **Program.cs:** Exemplo conceitual

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

#### []{#example-0--Output-txt .anchor} **Output.txt:** Resultados da execução

<figure class="code">
<pre class="code" lang="output"><code>Singleton works, both variables contain the same instance.</code></pre>
</figure>
:::

::: {#example-1 .tab-pane role="tabpanel" aria-labelledby="example-1-tab" style="padding-top: 24px"}
## Singleton seguro para threads {#example-1-title}

Para corrigir o problema, você deve sincronizar os threads durante a
primeira criação do objeto Singleton.

#### []{#example-1--Program-cs .anchor} **Program.cs:** Exemplo conceitual

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

#### []{#example-1--Output-txt .anchor} **Output.txt:** Resultados da execução

<figure class="code">
<pre class="code" lang="output"><code>FOO
FOO</code></pre>
</figure>
:::

::: {#example-2 .tab-pane role="tabpanel" aria-labelledby="example-2-tab" style="padding-top: 24px"}
## Quer mais? {#example-2-title}

Existem ainda mais sabores especiais do padrão Singleton em C#. Dê uma
olhada neste artigo para descobrir mais:

[C# a Fundo: Implementando o
Singleton](https://csharpindepth.com/Articles/Singleton)
:::
::::::

::: {#nav-tab .nav .nav-tabs .nav-tabs-bottom .nav-justified role="tablist"}
[Singleton ingênuo](#example-0){#example-0-tab-bottom .nav-item
.nav-link .active toggle="tab" role="tab" aria-controls="example-0"
aria-selected="true"} [Singleton seguro para
threads](#example-1){#example-1-tab-bottom .nav-item .nav-link
toggle="tab" role="tab" aria-controls="example-1" aria-selected="true"}
[Quer mais?](#example-2){#example-2-tab-bottom .nav-item .nav-link
toggle="tab" role="tab" aria-controls="example-2" aria-selected="true"}
:::

::: next
#### Leia a seguir

[Adapter em C# []{.fa
.fa-arrow-right}](../../adapter/csharp/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Voltar

[[]{.fa .fa-arrow-left} Prototype em
C#](../../prototype/csharp/example.html){.btn .btn-default rel="prev"}
:::

## **Singleton** em outras linguagens

[![Singleton em
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Singleton em C++"){.prog-lang-link}
[![Singleton em
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Singleton em Go"){.prog-lang-link}
[![Singleton em
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Singleton em Java"){.prog-lang-link}
[![Singleton em
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Singleton em PHP"){.prog-lang-link}
[![Singleton em
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Singleton em Python"){.prog-lang-link}
[![Singleton em
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Singleton em Ruby"){.prog-lang-link}
[![Singleton em
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Singleton em Rust"){.prog-lang-link}
[![Singleton em
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Singleton em Swift"){.prog-lang-link}
[![Singleton em
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Singleton em TypeScript"){.prog-lang-link}
:::::::::::::::::::::::::::

::::::: {.prom .banner-sidebar .banner .banner-striped .banner-examples .banner-removable .banner-removable-patterns data-id="DP: Examples-IDE" creative-id="examples-sidebar-pt-br" position="sidebar"}
:::::: {.banner-inner style="font-size: 14px; line-height: 21px;"}
::: banner-examples-img
[![](../../../../images/patterns/banners/examples-ide91ca.png?id=3115b4b548fb96b75974e2de8f4f49bc){width="215"
height="158" loading="lazy"
srcset="/images/patterns/banners/examples-ide-2x.png?id=93c007a6d157b97c28eb213f59d716ae 2x"}](../../book.html)
:::

::: banner-examples-fade
[ Arquivo com exemplos de código](../../book.html){.btn .btn-secondary
.btn-block}
:::

::: {.banner-examples-text style="margin-top: 1rem;"}
Compre o eBook **Mergulho nos Padrões de Projeto** e tenha acesso ao
arquivo com dúzias de exemplos detalhados que podem ser abertos
diretamente em seu IDE.

[ Saiba mais...](../../book.html){.btn .btn-secondary .btn-block}
:::
::::::
:::::::
:::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::
