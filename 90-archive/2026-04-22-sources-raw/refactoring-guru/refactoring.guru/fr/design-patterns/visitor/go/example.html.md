::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Patrons de
conception](../../../design-patterns.html){.type} /
[Visiteur](../../visitor.html){.type} / [Go](../../go.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Visiteur](../../../../images/patterns/cards/visitor-mini0a1c.png?id=854a35a62963bec1d75eab996918989b){srcset="/images/patterns/cards/visitor-mini-2x.png?id=9b87f3f3b772f434b28a25876829b504 2x"}
:::

# **Visiteur** en Go {#visiteur-en-go .pattern-example-title-block-title}
::::

:::::::::::::::::: pattern-example-body
::: pattern-example-brief
Le **Visiteur** est un patron de conception comportemental qui permet
d'ajouter de nouveaux comportements à une hiérarchie de classes sans
modifier l'existant.

> Découvrez pourquoi les visiteurs ne peuvent pas être remplacés par la
> surcharge de méthodes dans notre article [Visiteur et double
> répartition](../../visitor-double-dispatch.html).
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ En savoir plus sur la patron Visiteur ](../../visitor.html){.btn
.btn-lg .btn-primary}
:::

::::::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
Navigation
:::

::: en-intro
 [Intro](#)
:::

::: en-intro
 [Exemple conceptuel](#example-0)
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

## Exemple conceptuel {#example-0-title}

Le patron de conception visiteur vous permet d'ajouter des comportements
à une struct sans vraiment modifier la structure. Disons que vous vous
occupez de maintenir une bibliothèque qui a différentes structs de
formes comme :

-   Carré
-   Cercle
-   Triangle

Chaque struct forme ci-dessus implémente l'interface commune Forme.

Dès que vos collègues ont commencé à utiliser votre super bibliothèque,
vous avez été enseveli sous les demandes de fonctionnalités.
Penchons-nous sur l'une des plus simples : une équipe vous a demandé de
rajouter le comportement `getArea` aux structs de formes.

Nous pouvons résoudre ce problème de plusieurs manières :

La première option qui vient à l'esprit est d'ajouter la méthode
`getArea` directement dans l'interface forme et de l'implémenter dans
chaque struct de forme. À première vue, cela semble être la meilleure
solution, mais elle a un certain coût. En tant que responsable de la
bibliothèque, vous ne voulez pas risquer d'endommager votre précieux
code chaque fois que quelqu'un réclame un nouveau comportement. Mais
vous voulez tout de même que les autres équipes aient la possibilité
d'étendre votre code.

La deuxième option est que l'équipe qui demande la fonctionnalité
l'implémente elle-même. Ce n'est pas toujours possible, car le
comportement peut dépendre de code privé.

La troisième option est de résoudre le problème en utilisant le patron
de conception visiteur. Nous commençons par définir une interface
Visiteur comme ci-dessous :

<figure class="code">
<pre class="code"><code>type visitor interface {
   visitForSquare(square)
   visitForCircle(circle)
   visitForTriangle(triangle)
}</code></pre>
</figure>

Les fonctions `visitForSquare(square)`, `visitForCircle(circle)`,
`visitForTriangle(triangle)` nous permettent d'ajouter des
fonctionnalités respectivement aux carrés, cercles et triangles.

Vous vous demandez pourquoi on ne peut pas avoir une seule méthode
`visit(shape)` dans l'interface visiteur ? La raison est tout simplement
que le Go ne gère pas la surcharge de méthodes. On ne peut donc pas
définir des méthodes avec un même nom et des paramètres différents.

La deuxième partie du travail consiste à ajouter la méthode `accept` à
l'interface Forme.

<figure class="code">
<pre class="code"><code>func accept(v visitor)</code></pre>
</figure>

Toutes les structs de formes doivent définir cette méthode comme ceci :

<figure class="code">
<pre class="code"><code>func (obj *square) accept(v visitor){
    v.visitForSquare(obj)
}</code></pre>
</figure>

Attendez une minute, n'ai-je pas dit que nous ne voulions pas modifier
nos structs de formes existantes ? Malheureusement, lorsque l'on utilise
le visiteur, nous devons forcément modifier nos structs de formes, mais
cette modification ne devra être faite qu'une seule fois.

Dans le cas où nous voulons ajouter d'autres comportements comme
`getNumSides` ou `getMiddleCoordinates`, nous utiliserons la même
fonction `accept(v visitor)` sans apporter d'autres modifications aux
structs de formes.

Au final, nous ne modifierons qu'une seule fois la struct forme, et
toutes les futures demandes pour les nouveaux comportements seront
gérées en utilisant cette même fonction accept. Si une équipe demande le
comportement `getArea`, nous pouvons simplement définir une
implémentation concrète de l'interface visiteur et y écrire la logique
de calcul de la surface.

#### []{#example-0--shape-go .anchor} **shape.go:** Élément

<figure class="code">
<pre class="code" lang="go"><code>package main

type Shape interface {
    getType() string
    accept(Visitor)
}</code></pre>
</figure>

#### []{#example-0--square-go .anchor} **square.go:** Élément concret

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

#### []{#example-0--circle-go .anchor} **circle.go:** Élément concret

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

#### []{#example-0--rectangle-go .anchor} **rectangle.go:** Élément concret

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

#### []{#example-0--visitor-go .anchor} **visitor.go:** Visiteur

<figure class="code">
<pre class="code" lang="go"><code>package main

type Visitor interface {
    visitForSquare(*Square)
    visitForCircle(*Circle)
    visitForrectangle(*Rectangle)
}</code></pre>
</figure>

#### []{#example-0--areaCalculator-go .anchor} **areaCalculator.go:** Visiteur concret

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

#### []{#example-0--middleCoordinates-go .anchor} **middleCoordinates.go:** Visiteur concret

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

#### []{#example-0--main-go .anchor} **main.go:** Code client

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

#### []{#example-0--output-txt .anchor} **output.txt:** Résultat de l'exécution

<figure class="code">
<pre class="code" lang="output"><code>Calculating area for square
Calculating area for circle
Calculating area for rectangle

Calculating middle point coordinates for square
Calculating middle point coordinates for circle
Calculating middle point coordinates for rectangle</code></pre>
</figure>

::: next
#### Suivant

[Les patrons de conception en Java []{.fa
.fa-arrow-right}](../../java.html){.btn .btn-primary rel="next"}
:::

::: prev
#### Retour

[[]{.fa .fa-arrow-left} Patron de méthode en
Go](../../template-method/go/example.html){.btn .btn-default rel="prev"}
:::

## **Visiteur** dans les autres langues

[![Visiteur en
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Visiteur en C#"){.prog-lang-link}
[![Visiteur en
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Visiteur en C++"){.prog-lang-link}
[![Visiteur en
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Visiteur en Java"){.prog-lang-link}
[![Visiteur en
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Visiteur en PHP"){.prog-lang-link}
[![Visiteur en
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Visiteur en Python"){.prog-lang-link}
[![Visiteur en
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Visiteur en Ruby"){.prog-lang-link}
[![Visiteur en
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Visiteur en Rust"){.prog-lang-link}
[![Visiteur en
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Visiteur en Swift"){.prog-lang-link}
[![Visiteur en
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Visiteur en TypeScript"){.prog-lang-link}
::::::::::::::::::::::::

::::::: {.prom .banner-sidebar .banner .banner-striped .banner-examples .banner-removable .banner-removable-patterns data-id="DP: Examples-IDE" creative-id="examples-sidebar-fr" position="sidebar"}
:::::: {.banner-inner style="font-size: 14px; line-height: 21px;"}
::: banner-examples-img
[![](../../../../images/patterns/banners/examples-ide91ca.png?id=3115b4b548fb96b75974e2de8f4f49bc){width="215"
height="158" loading="lazy"
srcset="/images/patterns/banners/examples-ide-2x.png?id=93c007a6d157b97c28eb213f59d716ae 2x"}](../../book.html)
:::

::: banner-examples-fade
[ Archive avec exemples](../../book.html){.btn .btn-secondary
.btn-block}
:::

::: {.banner-examples-text style="margin-top: 1rem;"}
Achetez l'ebook **Plongée au cœur des patrons de conception** et obtenez
l'accès à une archive comprenant des dizaines d'exemples détaillés qui
peuvent être ouverts dans votre environnement de développement.

[ En savoir plus...](../../book.html){.btn .btn-secondary .btn-block}
:::
::::::
:::::::
::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::
