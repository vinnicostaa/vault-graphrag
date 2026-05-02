::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Patrones de
diseño](../../../design-patterns.html){.type} /
[Strategy](../../strategy.html){.type} / [Go](../../go.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Strategy](../../../../images/patterns/cards/strategy-mini28f6.png?id=d38abee4fb6f2aed909d262bdadca936){srcset="/images/patterns/cards/strategy-mini-2x.png?id=f4e6608561f8e5d18be6927d4620ad29 2x"}
:::

# **Strategy** en Go {#strategy-en-go .pattern-example-title-block-title}
::::

:::::::::::::::: pattern-example-body
::: pattern-example-brief
**Strategy** es un patrón de diseño de comportamiento que convierte un
grupo de comportamientos en objetos y los hace intercambiables dentro
del objeto de contexto original.

El objeto original, llamado contexto, contiene una referencia a un
objeto de estrategia y le delega la ejecución del comportamiento. Para
cambiar la forma en que el contexto realiza su trabajo, otros objetos
pueden sustituir el objeto de estrategia actualmente vinculado, por
otro.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Aprende más sobre el patrón Strategy ](../../strategy.html){.btn
.btn-lg .btn-primary}
:::

::::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
Navegación
:::

::: en-intro
 [Intro](#)
:::

::: en-intro
 [Ejemplo conceptual](#example-0)
:::

::: en-file
 [eviction­Algo](#example-0--evictionAlgo-go)
:::

::: en-file
 [fifo](#example-0--fifo-go)
:::

::: en-file
 [lru](#example-0--lru-go)
:::

::: en-file
 [lfu](#example-0--lfu-go)
:::

::: en-file
 [cache](#example-0--cache-go)
:::

::: en-file
 [main](#example-0--main-go)
:::

::: en-file
 [output](#example-0--output-txt)
:::
:::::::::::::
::::::::::::::::

[]{#example-0}

## Ejemplo conceptual {#example-0-title}

Supongamos que estás creando una memoria caché. Debido a que está en la
memoria, tiene un tamaño limitado. Cuando alcanza su tamaño máximo,
deben eliminarse algunas entradas para liberar espacio. Esto puede
suceder a través de varios algoritmos. Algunos de los algoritmos
populares son:

-   Menos usada recientemente (LRU, por sus siglas en inglés): elimina
    una entrada que se ha utilizado menos recientemente.
-   Primera en entrar, primera en salir (FIFO): elimina una entrada que
    se creó primero.
-   Menos frecuentemente usada (LFU): elimina una entrada que se ha
    utilizado menos frecuentemente.

El problema está en cómo desacoplar nuestras clases caché de estos
algoritmos para que podamos cambiar el algoritmo durante el tiempo de
ejecución. Además, la clase caché no debe cambiar cuando se añade un
nuevo algoritmo.

Aquí es donde el patrón Strategy entra en escena. Sugiere la creación de
una familia del algoritmo en la que cada algoritmo tiene su propia
clase. Cada una de estas clases sigue la misma interfaz, y esto hace que
el algoritmo sea intercambiable dentro de la familia. Digamos que el
nombre de la interfaz común es `evictionAlgo`.

Ahora nuestra clase caché principal integrará la interfaz
`evictionAlgo`. En lugar de implementar todos los tipos de algoritmos de
reemplazo en sí mismos, nuestra clase caché delegará todo ello a la
interfaz `evictionAlgo`. Ya que `evictionAlgo` es una interfaz, podemos
cambiar el algoritmo durante el tiempo de ejecución a LRU, FIFO, LFU sin
realizar cambios en la clase caché.

#### []{#example-0--evictionAlgo-go .anchor} **evictionAlgo.go:** Interfaz de estrategia

<figure class="code">
<pre class="code" lang="go"><code>package main

type EvictionAlgo interface {
    evict(c *Cache)
}</code></pre>
</figure>

#### []{#example-0--fifo-go .anchor} **fifo.go:** Concrete strategy

<figure class="code">
<pre class="code" lang="go"><code>package main

import &quot;fmt&quot;

type Fifo struct {
}

func (l *Fifo) evict(c *Cache) {
    fmt.Println(&quot;Evicting by fifo strtegy&quot;)
}</code></pre>
</figure>

#### []{#example-0--lru-go .anchor} **lru.go:** Estrategia concreta

<figure class="code">
<pre class="code" lang="go"><code>package main

import &quot;fmt&quot;

type Lru struct {
}

func (l *Lru) evict(c *Cache) {
    fmt.Println(&quot;Evicting by lru strtegy&quot;)
}</code></pre>
</figure>

#### []{#example-0--lfu-go .anchor} **lfu.go:** Estrategia concreta

<figure class="code">
<pre class="code" lang="go"><code>package main

import &quot;fmt&quot;

type Lfu struct {
}

func (l *Lfu) evict(c *Cache) {
    fmt.Println(&quot;Evicting by lfu strtegy&quot;)
}</code></pre>
</figure>

#### []{#example-0--cache-go .anchor} **cache.go:** Contexto

<figure class="code">
<pre class="code" lang="go"><code>package main

type Cache struct {
    storage      map[string]string
    evictionAlgo EvictionAlgo
    capacity     int
    maxCapacity  int
}

func initCache(e EvictionAlgo) *Cache {
    storage := make(map[string]string)
    return &amp;Cache{
        storage:      storage,
        evictionAlgo: e,
        capacity:     0,
        maxCapacity:  2,
    }
}

func (c *Cache) setEvictionAlgo(e EvictionAlgo) {
    c.evictionAlgo = e
}

func (c *Cache) add(key, value string) {
    if c.capacity == c.maxCapacity {
        c.evict()
    }
    c.capacity++
    c.storage[key] = value
}

func (c *Cache) get(key string) {
    delete(c.storage, key)
}

func (c *Cache) evict() {
    c.evictionAlgo.evict(c)
    c.capacity--
}</code></pre>
</figure>

#### []{#example-0--main-go .anchor} **main.go:** Código cliente

<figure class="code">
<pre class="code" lang="go"><code>package main

func main() {
    lfu := &amp;Lfu{}
    cache := initCache(lfu)

    cache.add(&quot;a&quot;, &quot;1&quot;)
    cache.add(&quot;b&quot;, &quot;2&quot;)

    cache.add(&quot;c&quot;, &quot;3&quot;)

    lru := &amp;Lru{}
    cache.setEvictionAlgo(lru)

    cache.add(&quot;d&quot;, &quot;4&quot;)

    fifo := &amp;Fifo{}
    cache.setEvictionAlgo(fifo)

    cache.add(&quot;e&quot;, &quot;5&quot;)

}</code></pre>
</figure>

#### []{#example-0--output-txt .anchor} **output.txt:** Resultado de la ejecución

<figure class="code">
<pre class="code" lang="output"><code>Evicting by lfu strtegy
Evicting by lru strtegy
Evicting by fifo strtegy</code></pre>
</figure>

::: next
#### Leer siguiente

[Template Method en Go []{.fa
.fa-arrow-right}](../../template-method/go/example.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Volver

[[]{.fa .fa-arrow-left} State en Go](../../state/go/example.html){.btn
.btn-default rel="prev"}
:::

## **Strategy** en otros lenguajes

[![Strategy en
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Strategy en C#"){.prog-lang-link}
[![Strategy en
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Strategy en C++"){.prog-lang-link}
[![Strategy en
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Strategy en Java"){.prog-lang-link}
[![Strategy en
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Strategy en PHP"){.prog-lang-link}
[![Strategy en
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Strategy en Python"){.prog-lang-link}
[![Strategy en
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Strategy en Ruby"){.prog-lang-link}
[![Strategy en
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Strategy en Rust"){.prog-lang-link}
[![Strategy en
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Strategy en Swift"){.prog-lang-link}
[![Strategy en
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Strategy en TypeScript"){.prog-lang-link}
::::::::::::::::::::::

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
::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::
