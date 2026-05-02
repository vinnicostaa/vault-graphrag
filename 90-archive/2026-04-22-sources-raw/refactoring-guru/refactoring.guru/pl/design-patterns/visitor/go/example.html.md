::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Wzorce
projektowe](../../../design-patterns.html){.type} /
[Odwiedzający](../../visitor.html){.type} / [Go](../../go.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Odwiedzający](../../../../images/patterns/cards/visitor-mini0a1c.png?id=854a35a62963bec1d75eab996918989b){srcset="/images/patterns/cards/visitor-mini-2x.png?id=9b87f3f3b772f434b28a25876829b504 2x"}
:::

# **Odwiedzający** w języku Go {#odwiedzający-w-języku-go .pattern-example-title-block-title}
::::

:::::::::::::::::: pattern-example-body
::: pattern-example-brief
**Odwiedzający** to behawioralny wzorzec projektowy pozwalający dodawać
nowe zachowanie istniejącej hierarchii klas bez zmiany kodu jej klas.

> O tym, dlaczego Odwiedzającego nie można po prostu zastąpić
> przeciążaniem metod, przeczytasz w naszym artykule [Odwiedzający i
> podwójna dyspozycja](../../visitor-double-dispatch.html).
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Dowiedz się więcej o Odwiedzający ](../../visitor.html){.btn .btn-lg
.btn-primary}
:::

::::::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
Nawigacja
:::

::: en-intro
 [Intro](#)
:::

::: en-intro
 [Przykład koncepcyjny](#example-0)
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

## Przykład koncepcyjny {#example-0-title}

Wzorzec Odwiedzający pozwala dodawać nowe zachowania do struktury bez
konieczności modyfikowania jej. Powiedzmy, że naszym zadaniem jest
utrzymanie biblioteki zawierającej różne struktury figur geometrycznych:

-   Kwadrat
-   Okrąg
-   Trójkąt

Każda z powyższych struktur figur implementuje wspólny interfejs figury
geometrycznej.

Jak tylko twoi współpracownicy zaczęli korzystać z biblioteki, napłynęła
masa próśb o dodatkową funkcjonalność. Skupmy się na najprostszych:
powiedzmy, że zespół poprosił o dodanie funkcjonalności `getArea`
(pobierz pole) do struktur.

Jest wiele sposobów na rozwiązanie tego problemu.

Pierwszą możliwością, jaka przychodzi do głowy, jest dodanie metody
`getArea` bezpośrednio do interfejsu figury geometrycznej, a następnie
zaimplementowanie jej dla każdej z figur. Jest to najpopularniejsze
podejście w przypadku tego typu sytuacji, ale wiąże się z pewnym
kosztem. Jako osoba odpowiedzialna za utrzymanie biblioteki nie chcesz
ryzykować zepsucia kodu za każdym razem gdy ktoś poprosi o jakieś nowe
funkcje. Z drugiej strony, chcesz pozwolić innym zespołom rozszerzać
twoją bibliotekę.

Drugą opcją jest umożliwienie zespołom samodzielnej implementacji
potrzebnego zachowania. Nie jest to jednak zawsze możliwe, gdyż
potrzebne zachowanie może być zależne od kodu prywatnego.

Trzecie rozwiązanie sugeruje zastosowanie wzorca Odwiedzający w
połączeniu z powyższym rozwiązaniem. Zaczniemy od zdefiniowania
interfejsu odwiedzającego:

<figure class="code">
<pre class="code"><code>type visitor interface {
   visitForSquare(square)
   visitForCircle(circle)
   visitForTriangle(triangle)
}</code></pre>
</figure>

Funkcje `visitForSquare(square)`, `visitForCircle(circle)`,
`visitForTriangle(triangle)` pozwolą dodać funkcjonalność do ---
odpowiednio --- kwadratów, okręgów i trójkątów.

Pewnie zastanawiasz się, dlaczego nie możemy mieć tylko jednej metody
`visit(shape)` dla wszystkich figur. Powodem jest brak możliwości
przeciążania metod w języku Go, więc nie możemy mieć metod o takiej
samej nazwie, różniących się tylko parametrem.

Kolejnym istotnym krokiem jest dodanie metody `accept` (przyjmij) do
interfejsu figury.

<figure class="code">
<pre class="code"><code>func accept(v visitor)</code></pre>
</figure>

Wszystkie struktury figur geometrycznych muszą definiować tę metodę w
następujący sposób:

<figure class="code">
<pre class="code"><code>func (obj *square) accept(v visitor){
    v.visitForSquare(obj)
}</code></pre>
</figure>

Ale zaraz, przecież nie chcieliśmy zmieniać istniejących struktur figur!
Niestety --- zastosowanie wzorca Odwiedzający nakłada na nas taki
obowiązek. Na szczęście modyfikację trzeba wykonać tylko jednorazowo.

W przypadku dodawania innych funkcjonalności, jak `getNumSides` lub
`getMiddleCoordinates` będziemy mogli stosować tę samą funkcję
przyjmowania `accept(v visitor)` bez konieczności ponownego zmieniania
struktur reprezentujących figury.

Podsumowując, struktury odpowiadające poszczególnym figurom trzeba
zmodyfikować tylko raz, a wszystkie nowe funkcjonalności będzie można
obsłużyć tą samą funkcją przyjmującą. Jeśli zespół poprosi o
funkcjonalność `getArea`, możemy po prostu zdefiniować konkretną
implementację interfejsu odwiedzającego i zaprogramować logikę
obliczania w tej konkretnej implementacji.

#### []{#example-0--shape-go .anchor} **shape.go:** Element

<figure class="code">
<pre class="code" lang="go"><code>package main

type Shape interface {
    getType() string
    accept(Visitor)
}</code></pre>
</figure>

#### []{#example-0--square-go .anchor} **square.go:** Konkretny element

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

#### []{#example-0--circle-go .anchor} **circle.go:** Konkretny element

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

#### []{#example-0--rectangle-go .anchor} **rectangle.go:** Konkretny element

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

#### []{#example-0--visitor-go .anchor} **visitor.go:** Odwiedzający

<figure class="code">
<pre class="code" lang="go"><code>package main

type Visitor interface {
    visitForSquare(*Square)
    visitForCircle(*Circle)
    visitForrectangle(*Rectangle)
}</code></pre>
</figure>

#### []{#example-0--areaCalculator-go .anchor} **areaCalculator.go:** Konkretny odwiedzający

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

#### []{#example-0--middleCoordinates-go .anchor} **middleCoordinates.go:** Konkretny odwiedzający

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

#### []{#example-0--main-go .anchor} **main.go:** Kod klienta

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

#### []{#example-0--output-txt .anchor} **output.txt:** Wynik działania

<figure class="code">
<pre class="code" lang="output"><code>Calculating area for square
Calculating area for circle
Calculating area for rectangle

Calculating middle point coordinates for square
Calculating middle point coordinates for circle
Calculating middle point coordinates for rectangle</code></pre>
</figure>

::: next
#### Czytaj dalej

[Wzorce projektowe w Java []{.fa .fa-arrow-right}](../../java.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Wróć

[[]{.fa .fa-arrow-left} Metoda szablonowa w języku
Go](../../template-method/go/example.html){.btn .btn-default rel="prev"}
:::

## **Odwiedzający** w innych językach

[![Odwiedzający w języku
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Odwiedzający w języku C#"){.prog-lang-link}
[![Odwiedzający w języku
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Odwiedzający w języku C++"){.prog-lang-link}
[![Odwiedzający w języku
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Odwiedzający w języku Java"){.prog-lang-link}
[![Odwiedzający w języku
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Odwiedzający w języku PHP"){.prog-lang-link}
[![Odwiedzający w języku
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Odwiedzający w języku Python"){.prog-lang-link}
[![Odwiedzający w języku
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Odwiedzający w języku Ruby"){.prog-lang-link}
[![Odwiedzający w języku
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Odwiedzający w języku Rust"){.prog-lang-link}
[![Odwiedzający w języku
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Odwiedzający w języku Swift"){.prog-lang-link}
[![Odwiedzający w języku
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Odwiedzający w języku TypeScript"){.prog-lang-link}
::::::::::::::::::::::::

::::::: {.prom .banner-sidebar .banner .banner-striped .banner-examples .banner-removable .banner-removable-patterns data-id="DP: Examples-IDE" creative-id="examples-sidebar-pl" position="sidebar"}
:::::: {.banner-inner style="font-size: 14px; line-height: 21px;"}
::: banner-examples-img
[![](../../../../images/patterns/banners/examples-ide91ca.png?id=3115b4b548fb96b75974e2de8f4f49bc){width="215"
height="158" loading="lazy"
srcset="/images/patterns/banners/examples-ide-2x.png?id=93c007a6d157b97c28eb213f59d716ae 2x"}](../../book.html)
:::

::: banner-examples-fade
[ Archiwum\
przykładów kodu](../../book.html){.btn .btn-secondary .btn-block}
:::

::: {.banner-examples-text style="margin-top: 1rem;"}
Kupując eBook **Wzorce projektowe. Nowoczesny podręcznik** zyskujesz
dostęp do archiwum zawierającego mnóstwo szczegółowych przykładów, które
można od razu wkleić do swojego preferowanego zintegrowanego środowiska
programistycznego.

[ Dowiedz się więcej...](../../book.html){.btn .btn-secondary
.btn-block}
:::
::::::
:::::::
::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::
