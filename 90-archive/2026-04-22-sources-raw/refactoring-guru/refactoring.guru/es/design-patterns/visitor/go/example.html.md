::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Patrones de
diseño](../../../design-patterns.html){.type} /
[Visitor](../../visitor.html){.type} / [Go](../../go.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Visitor](../../../../images/patterns/cards/visitor-mini0a1c.png?id=854a35a62963bec1d75eab996918989b){srcset="/images/patterns/cards/visitor-mini-2x.png?id=9b87f3f3b772f434b28a25876829b504 2x"}
:::

# **Visitor** en Go {#visitor-en-go .pattern-example-title-block-title}
::::

:::::::::::::::::: pattern-example-body
::: pattern-example-brief
**Visitor** es un patrón de diseño de comportamiento que permite añadir
nuevos comportamientos a una jerarquía de clases existente sin alterar
el código.

> Lee por qué el patrón Visitor no puede simplemente sustituirse por la
> sobrecarga de métodos, en nuestro artículo [Visitor y envío
> doble](../../visitor-double-dispatch.html).
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Aprende más sobre el patrón Visitor ](../../visitor.html){.btn .btn-lg
.btn-primary}
:::

::::::::::::::: {.sidebar-navigation .with-scroll}
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
 [shape](#example-0--shape-go)
:::

::: en-file
 [square](#example-0--square-go)
:::

::: en-file
 [circle](#example-0--circle-go)
:::

::: en-file
 [rectangle](#example-0--rectangle-go)
:::

::: en-file
 [visitor](#example-0--visitor-go)
:::

::: en-file
 [area­Calculator](#example-0--areaCalculator-go)
:::

::: en-file
 [middle­Coordinates](#example-0--middleCoordinates-go)
:::

::: en-file
 [main](#example-0--main-go)
:::

::: en-file
 [output](#example-0--output-txt)
:::
:::::::::::::::
::::::::::::::::::

[]{#example-0}

## Ejemplo conceptual {#example-0-title}

El patrón Visitor permite añadir comportamientos a una estructura sin
modificar dicha estructura. Digamos que tienes encargado mantener una
biblioteca que tiene distintas estructuras de forma, como:

-   Cuadrado
-   Círculo
-   Triángulo

Cada una de estas estructuras de forma implementa la interfaz común de
forma.

Una vez que la gente de tu empresa empieza a utilizar tu estupenda
biblioteca, te llueven las solicitudes de funciones. Vamos a revisar
algunas de las más simples: un equipo te pide que añadas el
comportamiento `getArea` a las estructuras de forma.

Existen muchas opciones para resolver este problema.

La primera opción que viene a la mente es añadir el método `getArea`
directamente a la interfaz de forma e implementarla después en cada
estructura de forma. Ésta parece una solución a la que recurrir, pero
tiene un precio. Como encargado de mantener la biblioteca, no puedes
arriesgarte a descomponer tu precioso código cada vez que alguien pide
un nuevo comportamiento. Sin embargo, quieres que otros equipos amplíen
también tu biblioteca.

La segunda opción es que el equipo que solicita la función pueda
implementar el comportamiento por sí mismo. No obstante, esto no siempre
es posible, ya que el comportamiento dependerá del código privado.

La tercera opción es resolver el problema anterior utilizando el patrón
Visitor. Comenzamos definiendo una interfaz visitante, así:

<figure class="code">
<pre class="code"><code>type visitor interface {
   visitForSquare(square)
   visitForCircle(circle)
   visitForTriangle(triangle)
}</code></pre>
</figure>

Las funciones `visitForSquare(square)`, `visitForCircle(circle)`,
`visitForTriangle(triangle)` nos permitirán añadir funcionalidades a
cuadrados, círculos y triángulos respectivamente.

¿Te preguntas por qué no podemos tener un único método `visit(shape)` en
la interfaz visitante? La razón es que el lenguaje Go no soporta la
sobrecarga de métodos, por lo que no puedes tener métodos con el mismo
nombre pero distintos parámetros.

Ahora, la segunda parte importante es añadir el método `accept` a la
interfaz de forma.

<figure class="code">
<pre class="code"><code>func accept(v visitor)</code></pre>
</figure>

Toda la estructura de forma tiene que definir este método, de forma
parecida a ésta:

<figure class="code">
<pre class="code"><code>func (obj *square) accept(v visitor){
    v.visitForSquare(obj)
}</code></pre>
</figure>

Espera un momento, ¿no acabo de decir que no queremos modificar nuestras
estructuras de forma existentes? Lamentablemente, sí, cuando utilizamos
el patrón Visitor, tenemos que alterar nuestras estructuras de forma.
Pero esta modificación se realizará solo una vez.

En caso de añadir otros comportamientos, como `getNumSides`,
`getMiddleCoordinates`, utilizaremos la misma función
`accept(v visitor)` sin realizar más cambios en las estructuras de
forma.

Al final, las estructuras de forma solo tienen que modificarse una vez,
y todas las futuras solicitudes de comportamientos podrán gestionarse
utilizando la misma función accept. Si el equipo solicita el
comportamiento `getArea`, podemos limitarnos a definir la implementación
concreta de la interfaz visitante y escribir la lógica de cálculo del
área en esa implementación concreta.

#### []{#example-0--shape-go .anchor} **shape.go:** Elemento

<figure class="code">
<pre class="code" lang="go"><code>package main

type Shape interface {
    getType() string
    accept(Visitor)
}</code></pre>
</figure>

#### []{#example-0--square-go .anchor} **square.go:** Elemento concreta

<figure class="code">
<pre class="code" lang="go"><code>package main

type Square struct {
    side int
}

func (s *Square) accept(v Visitor) {
    v.visitForSquare(s)
}

func (s *Square) getType() string {
    return &quot;Square&quot;
}</code></pre>
</figure>

#### []{#example-0--circle-go .anchor} **circle.go:** Elemento concreta

<figure class="code">
<pre class="code" lang="go"><code>package main

type Circle struct {
    radius int
}

func (c *Circle) accept(v Visitor) {
    v.visitForCircle(c)
}

func (c *Circle) getType() string {
    return &quot;Circle&quot;
}</code></pre>
</figure>

#### []{#example-0--rectangle-go .anchor} **rectangle.go:** Elemento concreta

<figure class="code">
<pre class="code" lang="go"><code>package main

type Rectangle struct {
    l int
    b int
}

func (t *Rectangle) accept(v Visitor) {
    v.visitForrectangle(t)
}

func (t *Rectangle) getType() string {
    return &quot;rectangle&quot;
}</code></pre>
</figure>

#### []{#example-0--visitor-go .anchor} **visitor.go:** Visitante

<figure class="code">
<pre class="code" lang="go"><code>package main

type Visitor interface {
    visitForSquare(*Square)
    visitForCircle(*Circle)
    visitForrectangle(*Rectangle)
}</code></pre>
</figure>

#### []{#example-0--areaCalculator-go .anchor} **areaCalculator.go:** Visitante concreta

<figure class="code">
<pre class="code" lang="go"><code>package main

import (
    &quot;fmt&quot;
)

type AreaCalculator struct {
    area int
}

func (a *AreaCalculator) visitForSquare(s *Square) {
    // Calculate area for square.
    // Then assign in to the area instance variable.
    fmt.Println(&quot;Calculating area for square&quot;)
}

func (a *AreaCalculator) visitForCircle(s *Circle) {
    fmt.Println(&quot;Calculating area for circle&quot;)
}
func (a *AreaCalculator) visitForrectangle(s *Rectangle) {
    fmt.Println(&quot;Calculating area for rectangle&quot;)
}</code></pre>
</figure>

#### []{#example-0--middleCoordinates-go .anchor} **middleCoordinates.go:** Visitante concreta

<figure class="code">
<pre class="code" lang="go"><code>package main

import &quot;fmt&quot;

type MiddleCoordinates struct {
    x int
    y int
}

func (a *MiddleCoordinates) visitForSquare(s *Square) {
    // Calculate middle point coordinates for square.
    // Then assign in to the x and y instance variable.
    fmt.Println(&quot;Calculating middle point coordinates for square&quot;)
}

func (a *MiddleCoordinates) visitForCircle(c *Circle) {
    fmt.Println(&quot;Calculating middle point coordinates for circle&quot;)
}
func (a *MiddleCoordinates) visitForrectangle(t *Rectangle) {
    fmt.Println(&quot;Calculating middle point coordinates for rectangle&quot;)
}</code></pre>
</figure>

#### []{#example-0--main-go .anchor} **main.go:** Código cliente

<figure class="code">
<pre class="code" lang="go"><code>package main

import &quot;fmt&quot;

func main() {
    square := &amp;Square{side: 2}
    circle := &amp;Circle{radius: 3}
    rectangle := &amp;Rectangle{l: 2, b: 3}

    areaCalculator := &amp;AreaCalculator{}

    square.accept(areaCalculator)
    circle.accept(areaCalculator)
    rectangle.accept(areaCalculator)

    fmt.Println()
    middleCoordinates := &amp;MiddleCoordinates{}
    square.accept(middleCoordinates)
    circle.accept(middleCoordinates)
    rectangle.accept(middleCoordinates)
}</code></pre>
</figure>

#### []{#example-0--output-txt .anchor} **output.txt:** Resultado de la ejecución

<figure class="code">
<pre class="code" lang="output"><code>Calculating area for square
Calculating area for circle
Calculating area for rectangle

Calculating middle point coordinates for square
Calculating middle point coordinates for circle
Calculating middle point coordinates for rectangle</code></pre>
</figure>

::: next
#### Leer siguiente

[Patrones de diseño en Java []{.fa
.fa-arrow-right}](../../java.html){.btn .btn-primary rel="next"}
:::

::: prev
#### Volver

[[]{.fa .fa-arrow-left} Template Method en
Go](../../template-method/go/example.html){.btn .btn-default rel="prev"}
:::

## **Visitor** en otros lenguajes

[![Visitor en
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Visitor en C#"){.prog-lang-link}
[![Visitor en
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Visitor en C++"){.prog-lang-link}
[![Visitor en
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Visitor en Java"){.prog-lang-link}
[![Visitor en
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Visitor en PHP"){.prog-lang-link}
[![Visitor en
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Visitor en Python"){.prog-lang-link}
[![Visitor en
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Visitor en Ruby"){.prog-lang-link}
[![Visitor en
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Visitor en Rust"){.prog-lang-link}
[![Visitor en
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Visitor en Swift"){.prog-lang-link}
[![Visitor en
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Visitor en TypeScript"){.prog-lang-link}
::::::::::::::::::::::::

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
::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::
