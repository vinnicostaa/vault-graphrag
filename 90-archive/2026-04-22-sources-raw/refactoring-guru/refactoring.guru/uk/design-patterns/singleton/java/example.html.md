::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Патерни
проектування](../../../design-patterns.html){.type} /
[Одинак](../../singleton.html){.type} / [Java](../../java.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Одинак](../../../../images/patterns/cards/singleton-minif559.png?id=914e1565dfdf15f240e766163bd303ec){srcset="/images/patterns/cards/singleton-mini-2x.png?id=4a7348083d7719d02709a5c9ff878b9f 2x"}
:::

# **Одинак** на Java {#одинак-на-java .pattern-example-title-block-title}
::::

::::::::::::::::::::: pattern-example-body
::: pattern-example-brief
**Одинак** --- це породжуючий патерн, який гарантує існування тільки
одного об'єкта певного класу, а також дозволяє дістатися цього об'єкта з
будь-якого місця програми.

Одинак має такі ж переваги та недоліки, що і глобальні змінні. Його
неймовірно зручно використовувати, але він порушує модульність вашого
коду.

Ви не зможете просто взяти і використовувати клас, залежний від одинака,
в іншій програмі. Для цього доведеться емулювати там присутність
одинака. Найчастіше ця проблема проявляється при написанні юніт-тестів.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Детальніше про Одинак ](../../singleton.html){.btn .btn-lg
.btn-primary}
:::

:::::::::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
Навігація
:::

::: en-intro
 [Інтро](#)
:::

::: en-intro
 [Наївний Одинак (один потік)](#example-0)
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
 [Наївний Одинак (багато потоків)](#example-1)
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
 [Багатопоточний Одинак](#example-2)
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
 [Бажаєте ще?](#example-3)
:::
::::::::::::::::::

**Складність:** []{.fa-stars}

**Популярність:** []{.fa-stars}

**Застосування:** Багато програмістів вважають Одинака антипатерном,
тому його все рідше і рідше можна зустріти в Java-коді.

Тим не менш, Одинакові знайшлося застосування в стандартних бібліотеках
Java:

-   [`java.lang.Runtime#getRuntime()`](http://docs.oracle.com/javase/8/docs/api/java/lang/Runtime.html#getRuntime--)

-   [`java.awt.Desktop#getDesktop()`](http://docs.oracle.com/javase/8/docs/api/java/awt/Desktop.html#getDesktop--)

-   [`java.lang.System#getSecurityManager()`](http://docs.oracle.com/javase/8/docs/api/java/lang/System.html#getSecurityManager--)

**Ознаки застосування патерна:** Одинака можна визначити за статичним
створюючим методом, який повертає один і той же об'єкт.
:::::::::::::::::::::

::: {#nav-tab .nav .nav-tabs .nav-justified role="tablist"}
[Наївний Одинак (один потік)](#example-0){#example-0-tab .nav-item
.nav-link .active toggle="tab" role="tab" aria-controls="example-0"
aria-selected="true"} [Наївний Одинак (багато
потоків)](#example-1){#example-1-tab .nav-item .nav-link toggle="tab"
role="tab" aria-controls="example-1" aria-selected="true"}
[Багатопоточний Одинак](#example-2){#example-2-tab .nav-item .nav-link
toggle="tab" role="tab" aria-controls="example-2" aria-selected="true"}
[Бажаєте ще?](#example-3){#example-3-tab .nav-item .nav-link
toggle="tab" role="tab" aria-controls="example-3" aria-selected="true"}
:::

::::::: {#nav-tabContent .pattern-example-tab-content .tab-content}
::: {#example-0 .tab-pane .show .active role="tabpanel" aria-labelledby="example-0-tab" style="padding-top: 24px"}
## Наївний Одинак (один потік) {#example-0-title}

Незграбно реалізувати Одинака дуже просто --- достатньо приховати
конструктор і надати створюючий статичний метод.

#### []{#example-0--Singleton-java .anchor} **Singleton.java:** Одинак

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

#### []{#example-0--DemoSingleThread-java .anchor} **DemoSingleThread.java:** Клієнтський код

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

#### []{#example-0--OutputDemoSingleThread-txt .anchor} **OutputDemoSingleThread.txt:** Результати виконання

<figure class="code">
<pre class="code" lang="output"><code>If you see the same value, then singleton was reused (yay!)
If you see different values, then 2 singletons were created (booo!!)

RESULT:

FOO
FOO</code></pre>
</figure>
:::

::: {#example-1 .tab-pane role="tabpanel" aria-labelledby="example-1-tab" style="padding-top: 24px"}
## Наївний Одинак (багато потоків) {#example-1-title}

Той же клас веде себе неправильно в багатопотоковому середовищі.
Декілька потоків можуть одночасно викликати метод отримання Одинака та
створити відразу кілька екземплярів об'єкта.

#### []{#example-1--Singleton-java .anchor} **Singleton.java:** Одинак

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

#### []{#example-1--DemoMultiThread-java .anchor} **DemoMultiThread.java:** Клієнтський код

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

#### []{#example-1--OutputDemoMultiThread-txt .anchor} **OutputDemoMultiThread.txt:** Результати виконання

<figure class="code">
<pre class="code" lang="output"><code>If you see the same value, then singleton was reused (yay!)
If you see different values, then 2 singletons were created (booo!!)

RESULT:

FOO
BAR</code></pre>
</figure>
:::

::: {#example-2 .tab-pane role="tabpanel" aria-labelledby="example-2-tab" style="padding-top: 24px"}
## Багатопоточний Одинак {#example-2-title}

Щоб виправити проблему, потрібно синхронізувати потоки при створенні
об'єкта-Одинака.

#### []{#example-2--Singleton-java .anchor} **Singleton.java:** Одинак

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

#### []{#example-2--DemoMultiThread-java .anchor} **DemoMultiThread.java:** Клієнтський код

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

#### []{#example-2--OutputDemoMultiThread-txt .anchor} **OutputDemoMultiThread.txt:** Результати виконання

<figure class="code">
<pre class="code" lang="output"><code>If you see the same value, then singleton was reused (yay!)
If you see different values, then 2 singletons were created (booo!!)

RESULT:

BAR
BAR</code></pre>
</figure>
:::

::: {#example-3 .tab-pane role="tabpanel" aria-labelledby="example-3-tab" style="padding-top: 24px"}
## Бажаєте ще? {#example-3-title}

Існує ще з півдюжини способів реалізації Одинака в Java. Якщо цікаво,
можете ознайомитися з ними тут:

[Java Singleton Design Pattern Best Practices with
Examples](https://www.journaldev.com/1377/java-singleton-design-pattern-best-practices-examples)
:::
:::::::

::: {#nav-tab .nav .nav-tabs .nav-tabs-bottom .nav-justified role="tablist"}
[Наївний Одинак (один потік)](#example-0){#example-0-tab-bottom
.nav-item .nav-link .active toggle="tab" role="tab"
aria-controls="example-0" aria-selected="true"} [Наївний Одинак (багато
потоків)](#example-1){#example-1-tab-bottom .nav-item .nav-link
toggle="tab" role="tab" aria-controls="example-1" aria-selected="true"}
[Багатопоточний Одинак](#example-2){#example-2-tab-bottom .nav-item
.nav-link toggle="tab" role="tab" aria-controls="example-2"
aria-selected="true"} [Бажаєте ще?](#example-3){#example-3-tab-bottom
.nav-item .nav-link toggle="tab" role="tab" aria-controls="example-3"
aria-selected="true"}
:::

::: next
#### Читаємо далі

[Адаптер на Java []{.fa
.fa-arrow-right}](../../adapter/java/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Повернутись назад

[[]{.fa .fa-arrow-left} Прототип на
Java](../../prototype/java/example.html){.btn .btn-default rel="prev"}
:::

## **Одинак** іншими мовами програмування

[![Одинак на
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Одинак на C#"){.prog-lang-link}
[![Одинак на
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Одинак на C++"){.prog-lang-link}
[![Одинак на
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Одинак на Go"){.prog-lang-link}
[![Одинак на
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Одинак на PHP"){.prog-lang-link}
[![Одинак на
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Одинак на Python"){.prog-lang-link}
[![Одинак на
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Одинак на Ruby"){.prog-lang-link}
[![Одинак на
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Одинак на Rust"){.prog-lang-link}
[![Одинак на
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Одинак на Swift"){.prog-lang-link}
[![Одинак на
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Одинак на TypeScript"){.prog-lang-link}
::::::::::::::::::::::::::::::::::

::::::: {.prom .banner-sidebar .banner .banner-striped .banner-examples .banner-removable .banner-removable-patterns data-id="DP: Examples-IDE" creative-id="examples-sidebar-uk" position="sidebar"}
:::::: {.banner-inner style="font-size: 14px; line-height: 21px;"}
::: banner-examples-img
[![](../../../../images/patterns/banners/examples-ide91ca.png?id=3115b4b548fb96b75974e2de8f4f49bc){width="215"
height="158" loading="lazy"
srcset="/images/patterns/banners/examples-ide-2x.png?id=93c007a6d157b97c28eb213f59d716ae 2x"}](../../book.html)
:::

::: banner-examples-fade
[ Архів з прикладами](../../book.html){.btn .btn-secondary .btn-block}
:::

::: {.banner-examples-text style="margin-top: 1rem;"}
Придбай книгу **Занурення в Патерни** та отримай архів з десятками
детальних прикладів, які можна відкривати прямо в IDE.

[ Дізнатися більше...](../../book.html){.btn .btn-secondary .btn-block}
:::
::::::
:::::::
::::::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::::::
