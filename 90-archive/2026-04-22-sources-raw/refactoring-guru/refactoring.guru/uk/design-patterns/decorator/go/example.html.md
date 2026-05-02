:::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Патерни
проектування](../../../design-patterns.html){.type} /
[Декоратор](../../decorator.html){.type} / [Go](../../go.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Декоратор](../../../../images/patterns/cards/decorator-mini6adb.png?id=d30458908e315af195cb183bc52dbef9){srcset="/images/patterns/cards/decorator-mini-2x.png?id=3b58e540d7d28523080cad341ed9b2e9 2x"}
:::

# **Декоратор** на Go {#декоратор-на-go .pattern-example-title-block-title}
::::

::::::::::::::: pattern-example-body
::: pattern-example-brief
**Декоратор** --- це структурний патерн, який дозволяє додавати «на
льоту» нові поведінки об'єктам, розміщаючи їх в об'єктах-обгортках.

Декоратор дозволяє загортати об'єкти безліч разів завдяки тому, що і
обгортки, і реальні об'єкти, що загортаються, мають спільний інтерфейс.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Детальніше про Декоратор ](../../decorator.html){.btn .btn-lg
.btn-primary}
:::

:::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
Навігація
:::

::: en-intro
 [Інтро](#)
:::

::: en-intro
 [Концептуальний приклад](#example-0)
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

## Концептуальний приклад {#example-0-title}

#### []{#example-0--pizza-go .anchor} **pizza.go:** Інтерфейс компонента

<figure class="code">
<pre class="code" lang="go"><code>package main

type IPizza interface {
    getPrice() int
}</code></pre>
</figure>

#### []{#example-0--veggieMania-go .anchor} **veggieMania.go:** Конкретний компонент

<figure class="code">
<pre class="code" lang="go"><code>package main

type VeggieMania struct {
}

func (p *VeggieMania) getPrice() int {
    return 15
}</code></pre>
</figure>

#### []{#example-0--tomatoTopping-go .anchor} **tomatoTopping.go:** Конкретний декоратор

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

#### []{#example-0--cheeseTopping-go .anchor} **cheeseTopping.go:** Конкретний декоратор

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

#### []{#example-0--main-go .anchor} **main.go:** Клієнтський код

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

#### []{#example-0--output-txt .anchor} **output.txt:** Результат виконання

<figure class="code">
<pre class="code" lang="output"><code>Price of veggeMania with tomato and cheese topping is 32</code></pre>
</figure>

::: next
#### Читаємо далі

[Фасад на Go []{.fa .fa-arrow-right}](../../facade/go/example.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Повернутись назад

[[]{.fa .fa-arrow-left} Компонувальник на
Go](../../composite/go/example.html){.btn .btn-default rel="prev"}
:::

## **Декоратор** іншими мовами програмування

[![Декоратор на
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Декоратор на C#"){.prog-lang-link}
[![Декоратор на
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Декоратор на C++"){.prog-lang-link}
[![Декоратор на
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Декоратор на Java"){.prog-lang-link}
[![Декоратор на
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Декоратор на PHP"){.prog-lang-link}
[![Декоратор на
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Декоратор на Python"){.prog-lang-link}
[![Декоратор на
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Декоратор на Ruby"){.prog-lang-link}
[![Декоратор на
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Декоратор на Rust"){.prog-lang-link}
[![Декоратор на
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Декоратор на Swift"){.prog-lang-link}
[![Декоратор на
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Декоратор на TypeScript"){.prog-lang-link}
:::::::::::::::::::::

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
:::::::::::::::::::::::::::
::::::::::::::::::::::::::::
