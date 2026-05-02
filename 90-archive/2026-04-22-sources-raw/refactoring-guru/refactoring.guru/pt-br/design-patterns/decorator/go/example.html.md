:::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Padrões de
Projeto](../../../design-patterns.html){.type} /
[Decorator](../../decorator.html){.type} / [Go](../../go.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Decorator](../../../../images/patterns/cards/decorator-mini6adb.png?id=d30458908e315af195cb183bc52dbef9){srcset="/images/patterns/cards/decorator-mini-2x.png?id=3b58e540d7d28523080cad341ed9b2e9 2x"}
:::

# **Decorator** em Go {#decorator-em-go .pattern-example-title-block-title}
::::

::::::::::::::: pattern-example-body
::: pattern-example-brief
O **Decorator** é um padrão estrutural que permite adicionar novos
comportamentos aos objetos dinamicamente, colocando-os dentro de objetos
wrapper especiais.

Usando decoradores, você pode agrupar objetos inúmeras vezes, pois os
objetos de destino e os decoradores seguem a mesma interface. O objeto
resultante terá um comportamento de empilhamento de todos os wrappers.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Saiba mais sobre o Decorator ](../../decorator.html){.btn .btn-lg
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
 [Exemplo conceitual](#example-0)
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

## Exemplo conceitual {#example-0-title}

#### []{#example-0--pizza-go .anchor} **pizza.go:** Interface de componente

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

#### []{#example-0--output-txt .anchor} **output.txt:** Resultados da execução

<figure class="code">
<pre class="code" lang="output"><code>Price of veggeMania with tomato and cheese topping is 32</code></pre>
</figure>

::: next
#### Leia a seguir

[Facade em Go []{.fa
.fa-arrow-right}](../../facade/go/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Voltar

[[]{.fa .fa-arrow-left} Composite em
Go](../../composite/go/example.html){.btn .btn-default rel="prev"}
:::

## **Decorator** em outras linguagens

[![Decorator em
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Decorator em C#"){.prog-lang-link}
[![Decorator em
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Decorator em C++"){.prog-lang-link}
[![Decorator em
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Decorator em Java"){.prog-lang-link}
[![Decorator em
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Decorator em PHP"){.prog-lang-link}
[![Decorator em
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Decorator em Python"){.prog-lang-link}
[![Decorator em
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Decorator em Ruby"){.prog-lang-link}
[![Decorator em
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Decorator em Rust"){.prog-lang-link}
[![Decorator em
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Decorator em Swift"){.prog-lang-link}
[![Decorator em
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Decorator em TypeScript"){.prog-lang-link}
:::::::::::::::::::::

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
:::::::::::::::::::::::::::
::::::::::::::::::::::::::::
