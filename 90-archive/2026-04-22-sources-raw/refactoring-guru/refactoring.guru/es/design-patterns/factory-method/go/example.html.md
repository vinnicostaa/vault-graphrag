::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Patrones de
diseño](../../../design-patterns.html){.type} / [Factory
Method](../../factory-method.html){.type} / [Go](../../go.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Factory
Method](../../../../images/patterns/cards/factory-method-minid7f6.png?id=72619e9527893374b98a5913779ac167){srcset="/images/patterns/cards/factory-method-mini-2x.png?id=fa9d4a8d61a67cc3822e52b9daf69dad 2x"}
:::

# **Factory Method** en Go {#factory-method-en-go .pattern-example-title-block-title}
::::

:::::::::::::::: pattern-example-body
::: pattern-example-brief
**Factory method** es un patrón de diseño creacional que resuelve el
problema de crear objetos de producto sin especificar sus clases
concretas.

El patrón Factory Method define un método que debe utilizarse para crear
objetos, en lugar de una llamada directa al constructor (operador
`new`). Las subclases pueden sobrescribir este método para cambiar las
clases de los objetos que se crearán.

> Si no sabes la diferencia entre varios patrones y conceptos de la
> fábrica, lee nuestra [Comparación de
> fábricas](../../factory-comparison.html).
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Aprende más sobre el patrón Factory Method
](../../factory-method.html){.btn .btn-lg .btn-primary}
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
 [i­Gun](#example-0--iGun-go)
:::

::: en-file
 [gun](#example-0--gun-go)
:::

::: en-file
 [ak47](#example-0--ak47-go)
:::

::: en-file
 [musket](#example-0--musket-go)
:::

::: en-file
 [gun­Factory](#example-0--gunFactory-go)
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

Resulta imposible implementar el clásico patrón Factory Method en Go
debido a la falta de funciones POO, como las clases y la herencia. Sin
embargo, podemos implementar la versión básica del patrón, el Simple
Factory.

En este ejemplo, vamos a crear varios tipos de armas utilizando una
estructura de fábrica.

Primero, creamos la interfaz `iGun`, que define todos los métodos con
los que debería contar un arma. Hay un tipo de estructura `gun` que
implementa la interfaz iGun. Dos armas concretas (`ak47` y `musket`),
ambas integran la estructura del arma e, indirectamente, implementan
todos los métodos `iGun`.

La estructura `gunFactory` sirve como fábrica, que crea armas del tipo
deseado en base a un argumento entrante. *main.go* actúa como cliente.
En lugar de interactuar directamente con `ak47` o `musket`, se basa en
`gunFactory` para crear instancias de varias armas, utilizando
únicamente parámetros de cadenas para controlar la producción.

#### []{#example-0--iGun-go .anchor} **iGun.go:** Interfaz de producto

<figure class="code">
<pre class="code" lang="go"><code>package main

type IGun interface {
    setName(name string)
    setPower(power int)
    getName() string
    getPower() int
}</code></pre>
</figure>

#### []{#example-0--gun-go .anchor} **gun.go:** Producto concreto

<figure class="code">
<pre class="code" lang="go"><code>package main

type Gun struct {
    name  string
    power int
}

func (g *Gun) setName(name string) {
    g.name = name
}

func (g *Gun) getName() string {
    return g.name
}

func (g *Gun) setPower(power int) {
    g.power = power
}

func (g *Gun) getPower() int {
    return g.power
}</code></pre>
</figure>

#### []{#example-0--ak47-go .anchor} **ak47.go:** Producto concreto

<figure class="code">
<pre class="code" lang="go"><code>package main

type Ak47 struct {
    Gun
}

func newAk47() IGun {
    return &amp;Ak47{
        Gun: Gun{
            name:  &quot;AK47 gun&quot;,
            power: 4,
        },
    }
}</code></pre>
</figure>

#### []{#example-0--musket-go .anchor} **musket.go:** Producto concreto

<figure class="code">
<pre class="code" lang="go"><code>package main

type musket struct {
    Gun
}

func newMusket() IGun {
    return &amp;musket{
        Gun: Gun{
            name:  &quot;Musket gun&quot;,
            power: 1,
        },
    }
}</code></pre>
</figure>

#### []{#example-0--gunFactory-go .anchor} **gunFactory.go:** Fábrica

<figure class="code">
<pre class="code" lang="go"><code>package main

import &quot;fmt&quot;

func getGun(gunType string) (IGun, error) {
    if gunType == &quot;ak47&quot; {
        return newAk47(), nil
    }
    if gunType == &quot;musket&quot; {
        return newMusket(), nil
    }
    return nil, fmt.Errorf(&quot;Wrong gun type passed&quot;)
}</code></pre>
</figure>

#### []{#example-0--main-go .anchor} **main.go:** Código cliente

<figure class="code">
<pre class="code" lang="go"><code>package main

import &quot;fmt&quot;

func main() {
    ak47, _ := getGun(&quot;ak47&quot;)
    musket, _ := getGun(&quot;musket&quot;)

    printDetails(ak47)
    printDetails(musket)
}

func printDetails(g IGun) {
    fmt.Printf(&quot;Gun: %s&quot;, g.getName())
    fmt.Println()
    fmt.Printf(&quot;Power: %d&quot;, g.getPower())
    fmt.Println()
}</code></pre>
</figure>

#### []{#example-0--output-txt .anchor} **output.txt:** Resultado de la ejecución

<figure class="code">
<pre class="code" lang="output"><code>Gun: AK47 gun
Power: 4
Gun: Musket gun
Power: 1</code></pre>
</figure>

::: next
#### Leer siguiente

[Prototype en Go []{.fa
.fa-arrow-right}](../../prototype/go/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Volver

[[]{.fa .fa-arrow-left} Builder en
Go](../../builder/go/example.html){.btn .btn-default rel="prev"}
:::

## **Factory Method** en otros lenguajes

[![Factory Method en
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Factory Method en C#"){.prog-lang-link}
[![Factory Method en
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Factory Method en C++"){.prog-lang-link}
[![Factory Method en
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Factory Method en Java"){.prog-lang-link}
[![Factory Method en
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Factory Method en PHP"){.prog-lang-link}
[![Factory Method en
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Factory Method en Python"){.prog-lang-link}
[![Factory Method en
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Factory Method en Ruby"){.prog-lang-link}
[![Factory Method en
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Factory Method en Rust"){.prog-lang-link}
[![Factory Method en
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Factory Method en Swift"){.prog-lang-link}
[![Factory Method en
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Factory Method en TypeScript"){.prog-lang-link}
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
