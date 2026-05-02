::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Patrones de
diseño](../../../design-patterns.html){.type} /
[Singleton](../../singleton.html){.type} /
[Java](../../java.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Singleton](../../../../images/patterns/cards/singleton-minif559.png?id=914e1565dfdf15f240e766163bd303ec){srcset="/images/patterns/cards/singleton-mini-2x.png?id=4a7348083d7719d02709a5c9ff878b9f 2x"}
:::

# **Singleton** en Java {#singleton-en-java .pattern-example-title-block-title}
::::

::::::::::::::::::::: pattern-example-body
::: pattern-example-brief
**Singleton** es un patrón de diseño creacional que garantiza que tan
solo exista un objeto de su tipo y proporciona un único punto de acceso
a él para cualquier otro código.

El patrón tiene prácticamente los mismos pros y contras que las
variables globales. Aunque son muy útiles, rompen la modularidad de tu
código.

No se puede utilizar una clase que dependa del Singleton en otro
contexto. Tendrás que llevar también la clase Singleton. La mayoría de
las veces, esta limitación aparece durante la creación de pruebas de
unidad.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Aprende más sobre el patrón Singleton ](../../singleton.html){.btn
.btn-lg .btn-primary}
:::

:::::::::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
Navegación
:::

::: en-intro
 [Intro](#)
:::

::: en-intro
 [Singleton ingenuo (hilo único)](#example-0)
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
 [Singleton ingenuo (multihilo)](#example-1)
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
 [Singleton con seguridad en los hilos y carga diferida](#example-2)
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
 [¿Quieres más?](#example-3)
:::
::::::::::::::::::

**Complejidad:** []{.fa-stars}

**Popularidad:** []{.fa-stars}

**Ejemplos de uso:** Muchos desarrolladores consideran el patrón
Singleton un antipatrón. Por este motivo, su uso está en declive en el
código Java.

A pesar de ello, existen muchos ejemplos del patrón Singleton en las
principales bibliotecas de Java:

-   [`java.lang.Runtime#getRuntime()`](http://docs.oracle.com/javase/8/docs/api/java/lang/Runtime.html#getRuntime--)
-   [`java.awt.Desktop#getDesktop()`](http://docs.oracle.com/javase/8/docs/api/java/awt/Desktop.html#getDesktop--)
-   [`java.lang.System#getSecurityManager()`](http://docs.oracle.com/javase/8/docs/api/java/lang/System.html#getSecurityManager--)

**Identificación:** El patrón Singleton se puede reconocer por un método
de creación estático, que devuelve el mismo objeto guardado en caché.
:::::::::::::::::::::

::: {#nav-tab .nav .nav-tabs .nav-justified role="tablist"}
[Singleton ingenuo (hilo único)](#example-0){#example-0-tab .nav-item
.nav-link .active toggle="tab" role="tab" aria-controls="example-0"
aria-selected="true"} [Singleton ingenuo
(multihilo)](#example-1){#example-1-tab .nav-item .nav-link toggle="tab"
role="tab" aria-controls="example-1" aria-selected="true"} [Singleton
con seguridad en los hilos y carga diferida](#example-2){#example-2-tab
.nav-item .nav-link toggle="tab" role="tab" aria-controls="example-2"
aria-selected="true"} [¿Quieres más?](#example-3){#example-3-tab
.nav-item .nav-link toggle="tab" role="tab" aria-controls="example-3"
aria-selected="true"}
:::

::::::: {#nav-tabContent .pattern-example-tab-content .tab-content}
::: {#example-0 .tab-pane .show .active role="tabpanel" aria-labelledby="example-0-tab" style="padding-top: 24px"}
## Singleton ingenuo (hilo único) {#example-0-title}

Es muy fácil implementar un Singleton descuidado. Tan solo necesitas
esconder el constructor e implementar un método de creación estático.

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

#### []{#example-0--DemoSingleThread-java .anchor} **DemoSingleThread.java:** Código cliente

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

#### []{#example-0--OutputDemoSingleThread-txt .anchor} **OutputDemoSingleThread.txt:** Resultados de la ejecución

<figure class="code">
<pre class="code" lang="output"><code>If you see the same value, then singleton was reused (yay!)
If you see different values, then 2 singletons were created (booo!!)

RESULT:

FOO
FOO</code></pre>
</figure>
:::

::: {#example-1 .tab-pane role="tabpanel" aria-labelledby="example-1-tab" style="padding-top: 24px"}
## Singleton ingenuo (multihilo) {#example-1-title}

La misma clase se comporta de forma incorrecta en un entorno de
múltiples hilos. Los múltiples hilos pueden llamar al método de creación
de forma simultánea y obtener varias instancias de la clase Singleton.

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

#### []{#example-1--DemoMultiThread-java .anchor} **DemoMultiThread.java:** Código cliente

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

#### []{#example-1--OutputDemoMultiThread-txt .anchor} **OutputDemoMultiThread.txt:** Resultados de la ejecución

<figure class="code">
<pre class="code" lang="output"><code>If you see the same value, then singleton was reused (yay!)
If you see different values, then 2 singletons were created (booo!!)

RESULT:

FOO
BAR</code></pre>
</figure>
:::

::: {#example-2 .tab-pane role="tabpanel" aria-labelledby="example-2-tab" style="padding-top: 24px"}
## Singleton con seguridad en los hilos y carga diferida {#example-2-title}

Para arreglar el problema, debes sincronizar hilos durante la primera
creación del objeto Singleton.

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

#### []{#example-2--DemoMultiThread-java .anchor} **DemoMultiThread.java:** Código cliente

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

#### []{#example-2--OutputDemoMultiThread-txt .anchor} **OutputDemoMultiThread.txt:** Resultados de la ejecución

<figure class="code">
<pre class="code" lang="output"><code>If you see the same value, then singleton was reused (yay!)
If you see different values, then 2 singletons were created (booo!!)

RESULT:

BAR
BAR</code></pre>
</figure>
:::

::: {#example-3 .tab-pane role="tabpanel" aria-labelledby="example-3-tab" style="padding-top: 24px"}
## ¿Quieres más? {#example-3-title}

Existen variedades aún más especiales del patrón Singleton en Java.
Consulta este artículo para saber más:

[Buenas prácticas del patrón de diseño Singleton en Java con
ejemplos](https://www.journaldev.com/1377/java-singleton-design-pattern-best-practices-examples)
:::
:::::::

::: {#nav-tab .nav .nav-tabs .nav-tabs-bottom .nav-justified role="tablist"}
[Singleton ingenuo (hilo único)](#example-0){#example-0-tab-bottom
.nav-item .nav-link .active toggle="tab" role="tab"
aria-controls="example-0" aria-selected="true"} [Singleton ingenuo
(multihilo)](#example-1){#example-1-tab-bottom .nav-item .nav-link
toggle="tab" role="tab" aria-controls="example-1" aria-selected="true"}
[Singleton con seguridad en los hilos y carga
diferida](#example-2){#example-2-tab-bottom .nav-item .nav-link
toggle="tab" role="tab" aria-controls="example-2" aria-selected="true"}
[¿Quieres más?](#example-3){#example-3-tab-bottom .nav-item .nav-link
toggle="tab" role="tab" aria-controls="example-3" aria-selected="true"}
:::

::: next
#### Leer siguiente

[Adapter en Java []{.fa
.fa-arrow-right}](../../adapter/java/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Volver

[[]{.fa .fa-arrow-left} Prototype en
Java](../../prototype/java/example.html){.btn .btn-default rel="prev"}
:::

## **Singleton** en otros lenguajes

[![Singleton en
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Singleton en C#"){.prog-lang-link}
[![Singleton en
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Singleton en C++"){.prog-lang-link}
[![Singleton en
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Singleton en Go"){.prog-lang-link}
[![Singleton en
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Singleton en PHP"){.prog-lang-link}
[![Singleton en
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Singleton en Python"){.prog-lang-link}
[![Singleton en
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Singleton en Ruby"){.prog-lang-link}
[![Singleton en
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Singleton en Rust"){.prog-lang-link}
[![Singleton en
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Singleton en Swift"){.prog-lang-link}
[![Singleton en
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Singleton en TypeScript"){.prog-lang-link}
::::::::::::::::::::::::::::::::::

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
::::::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::::::
