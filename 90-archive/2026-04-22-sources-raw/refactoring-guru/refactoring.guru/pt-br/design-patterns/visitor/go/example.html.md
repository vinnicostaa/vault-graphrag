::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Padrões de
Projeto](../../../design-patterns.html){.type} /
[Visitor](../../visitor.html){.type} / [Go](../../go.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Visitor](../../../../images/patterns/cards/visitor-mini0a1c.png?id=854a35a62963bec1d75eab996918989b){srcset="/images/patterns/cards/visitor-mini-2x.png?id=9b87f3f3b772f434b28a25876829b504 2x"}
:::

# **Visitor** em Go {#visitor-em-go .pattern-example-title-block-title}
::::

:::::::::::::::::: pattern-example-body
::: pattern-example-brief
O **Visitor** é um padrão de projeto comportamental que permite
adicionar novos comportamentos à hierarquia de classes existente sem
alterar nenhum código existente.

> Leia por que os Visitors não podem ser simplesmente substituídos pela
> sobrecarga de método em nosso artigo [Visitor e Double
> Dispatch](../../visitor-double-dispatch.html).
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Saiba mais sobre o Visitor ](../../visitor.html){.btn .btn-lg
.btn-primary}
:::

::::::::::::::: {.sidebar-navigation .with-scroll}
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

## Exemplo conceitual {#example-0-title}

O padrão Visitor permite adicionar comportamento a uma struct sem
realmente modificar a struct. Digamos que você seja o mantenedor de uma
biblioteca que tem structs de formatos diferentes, como:

-   Quadrado
-   Círculo
-   Triângulo

Cada uma das structs de forma acima implementa a interface de forma
comum.

Depois que as pessoas em sua empresa começaram a usar sua biblioteca
incrível, você foi inundado com solicitações de recursos. Vamos revisar
um dos mais simples: uma equipe solicitou que você adicionasse o
comportamento `getArea` às structs de forma.

Existem muitas opções para resolver este problema.

A primeira opção que vem à mente é adicionar o método `getArea`
diretamente na interface de forma e então implementá-lo em cada struct
de forma. Esta parece ser a solução certa, mas tem um custo. Como
mantenedor da biblioteca, você não quer arriscar quebrar seu precioso
código toda vez que alguém solicitar outro comportamento. Ainda assim,
você deseja que outras equipes estendam sua biblioteca de alguma forma.

A segunda opção é que a equipe que está solicitando o recurso pode
implementar o comportamento por conta própria. Porém, nem sempre isso é
possível, pois esse comportamento pode depender do código privado.

A terceira opção é resolver o problema acima usando o padrão Visitor.
Começamos definindo uma interface de visitante como esta:

<figure class="code">
<pre class="code"><code>type visitor interface {
    visitForSquare(square)
    visitForCircle(circle)
    visitForTriangle(triangle)
}</code></pre>
</figure>

As funções `visitForSquare (square)`, `visitForCircle (circle)`,
`visitForTriangle (triangle)` nos permitem adicionar funcionalidade a
quadrados, círculos e triângulos, respectivamente.

Está se perguntando por que não podemos ter um único método
`visit(shape)` na interface do visitante? O motivo é que a linguagem Go
não oferece suporte à sobrecarga de método, então você não pode ter
métodos com os mesmos nomes, mas com parâmetros diferentes.

Agora, a segunda parte importante é adicionar o método `accept` à
interface de forma.

<figure class="code">
<pre class="code"><code>func accept(v visitor)</code></pre>
</figure>

Todas as structs de forma precisam definir este método, de forma
semelhante a este:

<figure class="code">
<pre class="code"><code>func (obj *square) accept(v visitor){
    v.visitForSquare(obj)
}</code></pre>
</figure>

Espere um segundo, eu não mencionei que não queremos modificar nossas
structs de forma existentes? Infelizmente, sim, ao usar o padrão
Visitor, temos que alterar nossas structs de forma. Mas essa modificação
só será feita apenas uma vez.

No caso de adicionar qualquer outro comportamento, como `getNumSides`,
`getMiddleCoordinates`, usaremos a mesma função `accept(v visitor)` sem
quaisquer mudanças adicionais nas structs de forma.

No final, as structs de forma precisam ser modificadas apenas uma vez, e
todas as solicitações futuras para comportamentos diferentes podem ser
tratadas usando a mesma função de aceitação. Se a equipe solicitar o
comportamento `getArea`, podemos simplesmente definir a implementação
concreta da interface do visitante e escrever a lógica de cálculo da
área nessa implementação concreta.

#### []{#example-0--shape-go .anchor} **shape.go:** Elemento

<figure class="code">
<pre class="code" lang="go"><code>package main

type Shape interface {
    getType() string
    accept(Visitor)
}</code></pre>
</figure>

#### []{#example-0--square-go .anchor} **square.go:** Elemento concreto

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

#### []{#example-0--circle-go .anchor} **circle.go:** Elemento concreto

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

#### []{#example-0--rectangle-go .anchor} **rectangle.go:** Elemento concreto

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

#### []{#example-0--visitor-go .anchor} **visitor.go:** Visitor

<figure class="code">
<pre class="code" lang="go"><code>package main

type Visitor interface {
    visitForSquare(*Square)
    visitForCircle(*Circle)
    visitForrectangle(*Rectangle)
}</code></pre>
</figure>

#### []{#example-0--areaCalculator-go .anchor} **areaCalculator.go:** Visitante concreto

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

#### []{#example-0--middleCoordinates-go .anchor} **middleCoordinates.go:** Visitante concreto

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

#### []{#example-0--output-txt .anchor} **output.txt:** Resultados da execução

<figure class="code">
<pre class="code" lang="output"><code>Calculating area for square
Calculating area for circle
Calculating area for rectangle

Calculating middle point coordinates for square
Calculating middle point coordinates for circle
Calculating middle point coordinates for rectangle</code></pre>
</figure>

::: next
#### Leia a seguir

[Padrões de Projeto em Java []{.fa
.fa-arrow-right}](../../java.html){.btn .btn-primary rel="next"}
:::

::: prev
#### Voltar

[[]{.fa .fa-arrow-left} Template Method em
Go](../../template-method/go/example.html){.btn .btn-default rel="prev"}
:::

## **Visitor** em outras linguagens

[![Visitor em
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Visitor em C#"){.prog-lang-link}
[![Visitor em
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Visitor em C++"){.prog-lang-link}
[![Visitor em
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Visitor em Java"){.prog-lang-link}
[![Visitor em
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Visitor em PHP"){.prog-lang-link}
[![Visitor em
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Visitor em Python"){.prog-lang-link}
[![Visitor em
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Visitor em Ruby"){.prog-lang-link}
[![Visitor em
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Visitor em Rust"){.prog-lang-link}
[![Visitor em
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Visitor em Swift"){.prog-lang-link}
[![Visitor em
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Visitor em TypeScript"){.prog-lang-link}
::::::::::::::::::::::::

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
::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::
