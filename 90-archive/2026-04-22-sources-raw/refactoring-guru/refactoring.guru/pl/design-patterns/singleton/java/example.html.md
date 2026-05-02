::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Wzorce
projektowe](../../../design-patterns.html){.type} /
[Singleton](../../singleton.html){.type} /
[Java](../../java.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Singleton](../../../../images/patterns/cards/singleton-minif559.png?id=914e1565dfdf15f240e766163bd303ec){srcset="/images/patterns/cards/singleton-mini-2x.png?id=4a7348083d7719d02709a5c9ff878b9f 2x"}
:::

# **Singleton** w języku Java {#singleton-w-języku-java .pattern-example-title-block-title}
::::

::::::::::::::::::::: pattern-example-body
::: pattern-example-brief
**Singleton** to kreacyjny wzorzec projektowy gwarantujący istnienie
tylko jednego obiektu danego rodzaju. Udostępnia też pojedynczy punkt
dostępowy do takiego obiektu z dowolnego miejsca w programie.

Singleton charakteryzuje się prawie takimi samymi zaletami i wadami jak
zmienne globalne i chociaż jest bardzo poręczny, to jednak psuje
modularność kodu.

Nie można przenieść klasy zależnej od Singletona i użyć jej w innym
kontekście bez równoczesnego przeniesienia tego drugiego. To
ograniczenie zazwyczaj ujawnia się na etapie tworzenia testów
jednostkowych.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Dowiedz się więcej o Singleton ](../../singleton.html){.btn .btn-lg
.btn-primary}
:::

:::::::::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
Nawigacja
:::

::: en-intro
 [Intro](#)
:::

::: en-intro
 [Implementacja naiwna (jednowątkowa)](#example-0)
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
 [Implementacja naiwna (wielowątkowa)](#example-1)
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
 [Singleton z bezpieczeństwem wątków i z lazy loading](#example-2)
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
 [Chcesz wiedzieć więcej?](#example-3)
:::
::::::::::::::::::

**Złożoność:** []{.fa-stars}

**Popularność:** []{.fa-stars}

**Przykłady użycia:** Wielu twórców oprogramowania uważa Singleton za
antywzorzec, przez co jego użycie w kodzie Java stopniowo maleje.

Pomimo tego, w głównych bibliotekach Java można znaleźć mnóstwo
przykładów zastosowania tego wzorca:

-   [`java.lang.Runtime#getRuntime()`](http://docs.oracle.com/javase/8/docs/api/java/lang/Runtime.html#getRuntime--)
-   [`java.awt.Desktop#getDesktop()`](http://docs.oracle.com/javase/8/docs/api/java/awt/Desktop.html#getDesktop--)
-   [`java.lang.System#getSecurityManager()`](http://docs.oracle.com/javase/8/docs/api/java/lang/System.html#getSecurityManager--)

**Identyfikacja:** Singleton można rozpoznać po statycznej metodzie
kreacyjnej zwracającej jakiś obiekt którego instancja jest
przechowywana w pamięci podręcznej.
:::::::::::::::::::::

::: {#nav-tab .nav .nav-tabs .nav-justified role="tablist"}
[Implementacja naiwna (jednowątkowa)](#example-0){#example-0-tab
.nav-item .nav-link .active toggle="tab" role="tab"
aria-controls="example-0" aria-selected="true"} [Implementacja naiwna
(wielowątkowa)](#example-1){#example-1-tab .nav-item .nav-link
toggle="tab" role="tab" aria-controls="example-1" aria-selected="true"}
[Singleton z bezpieczeństwem wątków i z lazy
loading](#example-2){#example-2-tab .nav-item .nav-link toggle="tab"
role="tab" aria-controls="example-2" aria-selected="true"} [Chcesz
wiedzieć więcej?](#example-3){#example-3-tab .nav-item .nav-link
toggle="tab" role="tab" aria-controls="example-3" aria-selected="true"}
:::

::::::: {#nav-tabContent .pattern-example-tab-content .tab-content}
::: {#example-0 .tab-pane .show .active role="tabpanel" aria-labelledby="example-0-tab" style="padding-top: 24px"}
## Implementacja naiwna (jednowątkowa) {#example-0-title}

Łatwo jest zaimplementować wzorzec Singleton niechlujnie --- wystarczy
ukryć konstruktor i zaimplementować statyczną metodę kreacyjną.

#### []{#example-0--Singleton-java .anchor} **Singleton.java:** Singleton

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

#### []{#example-0--DemoSingleThread-java .anchor} **DemoSingleThread.java:** Kod klienta

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

#### []{#example-0--OutputDemoSingleThread-txt .anchor} **OutputDemoSingleThread.txt:** Wynik działania

<figure class="code">
<pre class="code" lang="output"><code>If you see the same value, then singleton was reused (yay!)
If you see different values, then 2 singletons were created (booo!!)

RESULT:

FOO
FOO</code></pre>
</figure>
:::

::: {#example-1 .tab-pane role="tabpanel" aria-labelledby="example-1-tab" style="padding-top: 24px"}
## Implementacja naiwna (wielowątkowa) {#example-1-title}

Ta sama klasa będzie działać nieprawidłowo w środowisku
wielowątkowym --- różne wątki mogą wywołać metodę kreacyjną w tym samym
momencie, otrzymując wiele instancji klasy Singleton.

#### []{#example-1--Singleton-java .anchor} **Singleton.java:** Singleton

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

#### []{#example-1--DemoMultiThread-java .anchor} **DemoMultiThread.java:** Kod klienta

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

#### []{#example-1--OutputDemoMultiThread-txt .anchor} **OutputDemoMultiThread.txt:** Wynik działania

<figure class="code">
<pre class="code" lang="output"><code>If you see the same value, then singleton was reused (yay!)
If you see different values, then 2 singletons were created (booo!!)

RESULT:

FOO
BAR</code></pre>
</figure>
:::

::: {#example-2 .tab-pane role="tabpanel" aria-labelledby="example-2-tab" style="padding-top: 24px"}
## Singleton z bezpieczeństwem wątków i z lazy loading {#example-2-title}

Aby pozbyć się wyżej wymienionego problemu, trzeba zsynchronizować
wątki w momencie pierwszego tworzenia obiektu Singleton.

#### []{#example-2--Singleton-java .anchor} **Singleton.java:** Singleton

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

#### []{#example-2--DemoMultiThread-java .anchor} **DemoMultiThread.java:** Kod klienta

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

#### []{#example-2--OutputDemoMultiThread-txt .anchor} **OutputDemoMultiThread.txt:** Wynik działania

<figure class="code">
<pre class="code" lang="output"><code>If you see the same value, then singleton was reused (yay!)
If you see different values, then 2 singletons were created (booo!!)

RESULT:

BAR
BAR</code></pre>
</figure>
:::

::: {#example-3 .tab-pane role="tabpanel" aria-labelledby="example-3-tab" style="padding-top: 24px"}
## Chcesz wiedzieć więcej? {#example-3-title}

Jest więcej specjalnych rodzajów wzorca Singleton w Javie i opisuje je
poniższy artykuł:

[Wzorzec projektowy Singleton w Java --- najlepsze praktyki i
przykłady](https://www.journaldev.com/1377/java-singleton-design-pattern-best-practices-examples)
:::
:::::::

::: {#nav-tab .nav .nav-tabs .nav-tabs-bottom .nav-justified role="tablist"}
[Implementacja naiwna (jednowątkowa)](#example-0){#example-0-tab-bottom
.nav-item .nav-link .active toggle="tab" role="tab"
aria-controls="example-0" aria-selected="true"} [Implementacja naiwna
(wielowątkowa)](#example-1){#example-1-tab-bottom .nav-item .nav-link
toggle="tab" role="tab" aria-controls="example-1" aria-selected="true"}
[Singleton z bezpieczeństwem wątków i z lazy
loading](#example-2){#example-2-tab-bottom .nav-item .nav-link
toggle="tab" role="tab" aria-controls="example-2" aria-selected="true"}
[Chcesz wiedzieć więcej?](#example-3){#example-3-tab-bottom .nav-item
.nav-link toggle="tab" role="tab" aria-controls="example-3"
aria-selected="true"}
:::

::: next
#### Czytaj dalej

[Adapter w języku Java []{.fa
.fa-arrow-right}](../../adapter/java/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Wróć

[[]{.fa .fa-arrow-left} Prototyp w języku
Java](../../prototype/java/example.html){.btn .btn-default rel="prev"}
:::

## **Singleton** w innych językach

[![Singleton w języku
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Singleton w języku C#"){.prog-lang-link}
[![Singleton w języku
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Singleton w języku C++"){.prog-lang-link}
[![Singleton w języku
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Singleton w języku Go"){.prog-lang-link}
[![Singleton w języku
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Singleton w języku PHP"){.prog-lang-link}
[![Singleton w języku
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Singleton w języku Python"){.prog-lang-link}
[![Singleton w języku
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Singleton w języku Ruby"){.prog-lang-link}
[![Singleton w języku
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Singleton w języku Rust"){.prog-lang-link}
[![Singleton w języku
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Singleton w języku Swift"){.prog-lang-link}
[![Singleton w języku
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Singleton w języku TypeScript"){.prog-lang-link}
::::::::::::::::::::::::::::::::::

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
::::::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::::::
