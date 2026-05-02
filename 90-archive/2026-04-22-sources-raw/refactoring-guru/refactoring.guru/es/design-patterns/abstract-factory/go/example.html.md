::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Patrones de
diseño](../../../design-patterns.html){.type} / [Abstract
Factory](../../abstract-factory.html){.type} /
[Go](../../go.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Abstract
Factory](../../../../images/patterns/cards/abstract-factory-minid1c5.png?id=4c3927c446313a38ce77dfee38111e27){srcset="/images/patterns/cards/abstract-factory-mini-2x.png?id=22236aaa65ff52cbde1c713216d52c1f 2x"}
:::

# **Abstract Factory** en Go {#abstract-factory-en-go .pattern-example-title-block-title}
::::

:::::::::::::::::::: pattern-example-body
::: pattern-example-brief
**Abstract Factory** es un patrón de diseño creacional que resuelve el
problema de crear familias enteras de productos sin especificar sus
clases concretas.

El patrón Abstract Factory define una interfaz para crear todos los
productos, pero deja la propia creación de productos para las clases de
fábrica concretas. Cada tipo de fábrica se corresponde con cierta
variedad de producto.

El código cliente invoca los métodos de creación de un objeto de fábrica
en lugar de crear los productos directamente con una llamada al
constructor (operador `new`). Como una fábrica se corresponde con una
única variante de producto, todos sus productos serán compatibles.

El código cliente trabaja con fábricas y productos únicamente a través
de sus interfaces abstractas. Esto permite al mismo código cliente
trabajar con productos diferentes. Simplemente, creas una nueva clase
fábrica concreta y la pasas al código cliente.

> Si no sabes la diferencia entre los distintos patrones de fábrica y
> sus conceptos, lee nuestra [Comparación de
> fábricas](../../factory-comparison.html).
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Aprende más sobre el patrón Abstract Factory
](../../abstract-factory.html){.btn .btn-lg .btn-primary}
:::

::::::::::::::::: {.sidebar-navigation .with-scroll}
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
 [i­Sports­Factory](#example-0--iSportsFactory-go)
:::

::: en-file
 [adidas](#example-0--adidas-go)
:::

::: en-file
 [nike](#example-0--nike-go)
:::

::: en-file
 [i­Shoe](#example-0--iShoe-go)
:::

::: en-file
 [adidas­Shoe](#example-0--adidasShoe-go)
:::

::: en-file
 [nike­Shoe](#example-0--nikeShoe-go)
:::

::: en-file
 [i­Shirt](#example-0--iShirt-go)
:::

::: en-file
 [adidas­Shirt](#example-0--adidasShirt-go)
:::

::: en-file
 [nike­Shirt](#example-0--nikeShirt-go)
:::

::: en-file
 [main](#example-0--main-go)
:::

::: en-file
 [output](#example-0--output-txt)
:::
:::::::::::::::::
::::::::::::::::::::

[]{#example-0}

## Ejemplo conceptual {#example-0-title}

Digamos que tienes que comprar dos productos diferentes de equipamiento
deportivo: un par de zapatillas y una camiseta. Te gustaría comprar todo
el equipamiento de la misma marca, para que los artículos combinen.

Si intentamos traducir esto en código, la fábrica abstracta nos ayudará
a crear grupos de productos para que siempre coincidan entre sí.

#### []{#example-0--iSportsFactory-go .anchor} **iSportsFactory.go:** Interfaz de la fábrica abstracta

<figure class="code">
<pre class="code" lang="go"><code>package main

import &quot;fmt&quot;

type ISportsFactory interface {
    makeShoe() IShoe
    makeShirt() IShirt
}

func GetSportsFactory(brand string) (ISportsFactory, error) {
    if brand == &quot;adidas&quot; {
        return &amp;Adidas{}, nil
    }

    if brand == &quot;nike&quot; {
        return &amp;Nike{}, nil
    }

    return nil, fmt.Errorf(&quot;Wrong brand type passed&quot;)
}</code></pre>
</figure>

#### []{#example-0--adidas-go .anchor} **adidas.go:** Fábrica concreta

<figure class="code">
<pre class="code" lang="go"><code>package main

type Adidas struct {
}

func (a *Adidas) makeShoe() IShoe {
    return &amp;AdidasShoe{
        Shoe: Shoe{
            logo: &quot;adidas&quot;,
            size: 14,
        },
    }
}

func (a *Adidas) makeShirt() IShirt {
    return &amp;AdidasShirt{
        Shirt: Shirt{
            logo: &quot;adidas&quot;,
            size: 14,
        },
    }
}</code></pre>
</figure>

#### []{#example-0--nike-go .anchor} **nike.go:** Fábrica concreta

<figure class="code">
<pre class="code" lang="go"><code>package main

type Nike struct {
}

func (n *Nike) makeShoe() IShoe {
    return &amp;NikeShoe{
        Shoe: Shoe{
            logo: &quot;nike&quot;,
            size: 14,
        },
    }
}

func (n *Nike) makeShirt() IShirt {
    return &amp;NikeShirt{
        Shirt: Shirt{
            logo: &quot;nike&quot;,
            size: 14,
        },
    }
}</code></pre>
</figure>

#### []{#example-0--iShoe-go .anchor} **iShoe.go:** Producto abstracto

<figure class="code">
<pre class="code" lang="go"><code>package main

type IShoe interface {
    setLogo(logo string)
    setSize(size int)
    getLogo() string
    getSize() int
}

type Shoe struct {
    logo string
    size int
}

func (s *Shoe) setLogo(logo string) {
    s.logo = logo
}

func (s *Shoe) getLogo() string {
    return s.logo
}

func (s *Shoe) setSize(size int) {
    s.size = size
}

func (s *Shoe) getSize() int {
    return s.size
}</code></pre>
</figure>

#### []{#example-0--adidasShoe-go .anchor} **adidasShoe.go:** Producto concreto

<figure class="code">
<pre class="code" lang="go"><code>package main

type AdidasShoe struct {
    Shoe
}</code></pre>
</figure>

#### []{#example-0--nikeShoe-go .anchor} **nikeShoe.go:** Producto concreto

<figure class="code">
<pre class="code" lang="go"><code>package main

type NikeShoe struct {
    Shoe
}</code></pre>
</figure>

#### []{#example-0--iShirt-go .anchor} **iShirt.go:** Producto abstracto

<figure class="code">
<pre class="code" lang="go"><code>package main

type IShirt interface {
    setLogo(logo string)
    setSize(size int)
    getLogo() string
    getSize() int
}

type Shirt struct {
    logo string
    size int
}

func (s *Shirt) setLogo(logo string) {
    s.logo = logo
}

func (s *Shirt) getLogo() string {
    return s.logo
}

func (s *Shirt) setSize(size int) {
    s.size = size
}

func (s *Shirt) getSize() int {
    return s.size
}</code></pre>
</figure>

#### []{#example-0--adidasShirt-go .anchor} **adidasShirt.go:** Producto concreto

<figure class="code">
<pre class="code" lang="go"><code>package main

type AdidasShirt struct {
    Shirt
}</code></pre>
</figure>

#### []{#example-0--nikeShirt-go .anchor} **nikeShirt.go:** Producto concreto

<figure class="code">
<pre class="code" lang="go"><code>package main

type NikeShirt struct {
    Shirt
}</code></pre>
</figure>

#### []{#example-0--main-go .anchor} **main.go:** Código cliente

<figure class="code">
<pre class="code" lang="go"><code>package main

import &quot;fmt&quot;

func main() {
    adidasFactory, _ := GetSportsFactory(&quot;adidas&quot;)
    nikeFactory, _ := GetSportsFactory(&quot;nike&quot;)

    nikeShoe := nikeFactory.makeShoe()
    nikeShirt := nikeFactory.makeShirt()

    adidasShoe := adidasFactory.makeShoe()
    adidasShirt := adidasFactory.makeShirt()

    printShoeDetails(nikeShoe)
    printShirtDetails(nikeShirt)

    printShoeDetails(adidasShoe)
    printShirtDetails(adidasShirt)
}

func printShoeDetails(s IShoe) {
    fmt.Printf(&quot;Logo: %s&quot;, s.getLogo())
    fmt.Println()
    fmt.Printf(&quot;Size: %d&quot;, s.getSize())
    fmt.Println()
}

func printShirtDetails(s IShirt) {
    fmt.Printf(&quot;Logo: %s&quot;, s.getLogo())
    fmt.Println()
    fmt.Printf(&quot;Size: %d&quot;, s.getSize())
    fmt.Println()
}</code></pre>
</figure>

#### []{#example-0--output-txt .anchor} **output.txt:** Resultado de la ejecución

<figure class="code">
<pre class="code" lang="output"><code>Logo: nike
Size: 14
Logo: nike
Size: 14
Logo: adidas
Size: 14
Logo: adidas
Size: 14</code></pre>
</figure>

::: next
#### Leer siguiente

[Builder en Go []{.fa
.fa-arrow-right}](../../builder/go/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Volver

[[]{.fa .fa-arrow-left} Patrones de diseño en Go](../../go.html){.btn
.btn-default rel="prev"}
:::

## **Abstract Factory** en otros lenguajes

[![Abstract Factory en
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Abstract Factory en C#"){.prog-lang-link}
[![Abstract Factory en
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Abstract Factory en C++"){.prog-lang-link}
[![Abstract Factory en
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Abstract Factory en Java"){.prog-lang-link}
[![Abstract Factory en
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Abstract Factory en PHP"){.prog-lang-link}
[![Abstract Factory en
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Abstract Factory en Python"){.prog-lang-link}
[![Abstract Factory en
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Abstract Factory en Ruby"){.prog-lang-link}
[![Abstract Factory en
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Abstract Factory en Rust"){.prog-lang-link}
[![Abstract Factory en
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Abstract Factory en Swift"){.prog-lang-link}
[![Abstract Factory en
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Abstract Factory en TypeScript"){.prog-lang-link}
::::::::::::::::::::::::::

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
::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::
