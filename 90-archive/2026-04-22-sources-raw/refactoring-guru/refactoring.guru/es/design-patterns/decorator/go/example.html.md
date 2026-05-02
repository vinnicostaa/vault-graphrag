:::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Patrones de
diseño](../../../design-patterns.html){.type} /
[Decorator](../../decorator.html){.type} / [Go](../../go.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Decorator](../../../../images/patterns/cards/decorator-mini6adb.png?id=d30458908e315af195cb183bc52dbef9){srcset="/images/patterns/cards/decorator-mini-2x.png?id=3b58e540d7d28523080cad341ed9b2e9 2x"}
:::

# **Decorator** en Go {#decorator-en-go .pattern-example-title-block-title}
::::

::::::::::::::: pattern-example-body
::: pattern-example-brief
**Decorator** es un patrón de diseño estructural que permite añadir
dinámicamente nuevos comportamientos a objetos colocándolos dentro de
objetos especiales que los envuelven (\_wrappers\_).

Utilizando decoradores puedes envolver objetos innumerables veces, ya
que los objetos objetivo y los decoradores siguen la misma interfaz. El
objeto resultante obtendrá un comportamiento de apilado de todos los
*wrappers*.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Aprende más sobre el patrón Decorator ](../../decorator.html){.btn
.btn-lg .btn-primary}
:::

:::::::::::: {.sidebar-navigation .with-scroll}
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
 [pizza](#example-0--pizza-go)
:::

::: en-file
 [veggie­Mania](#example-0--veggieMania-go)
:::

::: en-file
 [tomato­Topping](#example-0--tomatoTopping-go)
:::

::: en-file
 [cheese­Topping](#example-0--cheeseTopping-go)
:::

::: en-file
 [main](#example-0--main-go)
:::

::: en-file
 [output](#example-0--output-txt)
:::
::::::::::::
:::::::::::::::

[]{#example-0}

## Ejemplo conceptual {#example-0-title}

#### []{#example-0--pizza-go .anchor} **pizza.go:** Interfaz componente

<figure class="code">
<pre class="code" lang="go"><code>package main

type IPizza interface {
    getPrice() int
}</code></pre>
</figure>

#### []{#example-0--veggieMania-go .anchor} **veggieMania.go:** Componente concreto

<figure class="code">
<pre class="code" lang="go"><code>package main

type VeggieMania struct {
}

func (p *VeggieMania) getPrice() int {
    return 15
}</code></pre>
</figure>

#### []{#example-0--tomatoTopping-go .anchor} **tomatoTopping.go:** Decorador concreto

<figure class="code">
<pre class="code" lang="go"><code>package main

type TomatoTopping struct {
    pizza IPizza
}

func (c *TomatoTopping) getPrice() int {
    pizzaPrice := c.pizza.getPrice()
    return pizzaPrice + 7
}</code></pre>
</figure>

#### []{#example-0--cheeseTopping-go .anchor} **cheeseTopping.go:** Decorador concreto

<figure class="code">
<pre class="code" lang="go"><code>package main

type CheeseTopping struct {
    pizza IPizza
}

func (c *CheeseTopping) getPrice() int {
    pizzaPrice := c.pizza.getPrice()
    return pizzaPrice + 10
}</code></pre>
</figure>

#### []{#example-0--main-go .anchor} **main.go:** Código cliente

<figure class="code">
<pre class="code" lang="go"><code>package main

import &quot;fmt&quot;

func main() {

    pizza := &amp;VeggieMania{}

    //Add cheese topping
    pizzaWithCheese := &amp;CheeseTopping{
        pizza: pizza,
    }

    //Add tomato topping
    pizzaWithCheeseAndTomato := &amp;TomatoTopping{
        pizza: pizzaWithCheese,
    }

    fmt.Printf(&quot;Price of veggeMania with tomato and cheese topping is %d\n&quot;, pizzaWithCheeseAndTomato.getPrice())
}</code></pre>
</figure>

#### []{#example-0--output-txt .anchor} **output.txt:** Resultado de la ejecución

<figure class="code">
<pre class="code" lang="output"><code>Price of veggeMania with tomato and cheese topping is 32</code></pre>
</figure>

::: next
#### Leer siguiente

[Facade en Go []{.fa
.fa-arrow-right}](../../facade/go/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Volver

[[]{.fa .fa-arrow-left} Composite en
Go](../../composite/go/example.html){.btn .btn-default rel="prev"}
:::

## **Decorator** en otros lenguajes

[![Decorator en
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Decorator en C#"){.prog-lang-link}
[![Decorator en
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Decorator en C++"){.prog-lang-link}
[![Decorator en
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Decorator en Java"){.prog-lang-link}
[![Decorator en
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Decorator en PHP"){.prog-lang-link}
[![Decorator en
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Decorator en Python"){.prog-lang-link}
[![Decorator en
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Decorator en Ruby"){.prog-lang-link}
[![Decorator en
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Decorator en Rust"){.prog-lang-link}
[![Decorator en
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Decorator en Swift"){.prog-lang-link}
[![Decorator en
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Decorator en TypeScript"){.prog-lang-link}
:::::::::::::::::::::

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
:::::::::::::::::::::::::::
::::::::::::::::::::::::::::
