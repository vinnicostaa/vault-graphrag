::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Паттерны
проектирования](../../../design-patterns.html){.type} /
[Посетитель](../../visitor.html){.type} / [Go](../../go.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Посетитель](../../../../images/patterns/cards/visitor-mini0a1c.png?id=854a35a62963bec1d75eab996918989b){srcset="/images/patterns/cards/visitor-mini-2x.png?id=9b87f3f3b772f434b28a25876829b504 2x"}
:::

# **Посетитель** на Go {#посетитель-на-go .pattern-example-title-block-title}
::::

:::::::::::::::::: pattern-example-body
::: pattern-example-brief
**Посетитель** --- это поведенческий паттерн, который позволяет добавить
новую операцию для целой иерархии классов, не изменяя код этих классов.

> Подробней о том, почему Посетитель нельзя заменить простой перегрузкой
> методов читайте в статье [Посетитель и Double
> Dispatch](../../visitor-double-dispatch.html).
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Подробней о паттерне Посетитель ](../../visitor.html){.btn .btn-lg
.btn-primary}
:::

::::::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
Навигация
:::

::: en-intro
 [Интро](#)
:::

::: en-intro
 [Концептуальный пример](#example-0)
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

## Концептуальный пример {#example-0-title}

Паттерн Посетитель позволяет вам добавлять поведение в структуру без ее
изменения. Представим, что вы разработчик библиотеки, которая содержит
структуры разных фигур:

-   Квадрат
-   Круг
-   Треугольник

Структуры каждой из вышеназванных фигур реализуют общий интерфейс
фигуры.

Как только сотрудники в вашей компании начали использовать вашу
замечательную библиотеку, вас засыпали просьбами добавить тот или иной
функционал. Рассмотрим один из простейших вариантов: команда попросила
вас добавить в структуру функцию `getArea`, возвращающую площадь фигуры.

Существует много решений этой проблемы.

Первое, что приходит в голову -- добавить метод `getArea` напрямую в
интерфейс фигуры, и затем реализовать его в каждой структуре. Это
кажется самым правильным решением, но у него есть свои минусы. Как
разработчик библиотеки, вы рискуете сломать ваш драгоценный код каждый
раз, когда добавляете новое поведение. Несмотря на это, вы хотите дать
другим командам возможность каким-то образом расширять вашу библиотеку.

Второй вариант -- команда, которая запрашивает добавление новой функции,
может сама реализовать ее в своем проекте. Однако, это не всегда
является возможным, так как новый функционал может полагаться на скрытый
код.

Третий возможный вариант --- решить вышеуказанную проблему благодаря
использованию паттерна Посетитель. Сперва мы определяем интерфейс
посетителя следующим способом:

<figure class="code">
<pre class="code"><code>type visitor interface {
   visitForSquare(square)
   visitForCircle(circle)
   visitForTriangle(triangle)
}</code></pre>
</figure>

Функции `visitForSquare(square)`, `visitForCircle(circle)`,
`visitForTriangle(triangle)` позволят нам добавлять функционал для
квадратов, кругов и треугольников соответственно.

Не понимаете, почему мы не можем оставить только один метод
`visit(shape)` в интерфейсе посетителя? Это невозможно из-за того, что
язык Go не поддерживает перегрузку методов, поэтому вы не можете иметь
методы с одинаковыми именами, но разными параметрами.

Второй, не менее важный, этап -- добавление метода `accept` в интерфейс
фигуры.

<figure class="code">
<pre class="code"><code>func accept(v visitor)</code></pre>
</figure>

Все структуры фигур должны определять этот метод похожим способом:

<figure class="code">
<pre class="code"><code>func (obj *square) accept(v visitor){
    v.visitForSquare(obj)
}</code></pre>
</figure>

Погодите, разве я не заявлял чуть ранее, что не хочу менять существующие
структуры фигур? К сожалению, во время использования паттерна Посетителя
нам придется вносить изменения в структуры, но лишь единожды.

В случае добавления другого функционала, например `getNumSides` или
`getMiddleCoordinates`, мы будем использовать все тот же метод
`accept(v visitor)` без новых изменений структур фигур.

В конечном итоге, структуры нужно изменить лишь единожды, и все будущие
запросы нового функционала можно будет реализовать с помощью функции
`accept(v visitor)`. Если команда запросит поведение `getArea`, мы можем
просто определить явную реализацию интерфейса посетителя и прописать
логику вычисления площади в этой конкретной имплементации.

#### []{#example-0--shape-go .anchor} **shape.go:** Элемент

<figure class="code">
<pre class="code" lang="go"><code>package main

type Shape interface {
    getType() string
    accept(Visitor)
}</code></pre>
</figure>

#### []{#example-0--square-go .anchor} **square.go:** Конкретный элемент

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

#### []{#example-0--circle-go .anchor} **circle.go:** Конкретный элемент

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

#### []{#example-0--rectangle-go .anchor} **rectangle.go:** Конкретный элемент

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

#### []{#example-0--visitor-go .anchor} **visitor.go:** Посетитель

<figure class="code">
<pre class="code" lang="go"><code>package main

type Visitor interface {
    visitForSquare(*Square)
    visitForCircle(*Circle)
    visitForrectangle(*Rectangle)
}</code></pre>
</figure>

#### []{#example-0--areaCalculator-go .anchor} **areaCalculator.go:** Конкретный посетитель

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

#### []{#example-0--middleCoordinates-go .anchor} **middleCoordinates.go:** Конкретный посетитель

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

#### []{#example-0--main-go .anchor} **main.go:** Клиентский код

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

#### []{#example-0--output-txt .anchor} **output.txt:** Результат выполнения

<figure class="code">
<pre class="code" lang="output"><code>Calculating area for square
Calculating area for circle
Calculating area for rectangle

Calculating middle point coordinates for square
Calculating middle point coordinates for circle
Calculating middle point coordinates for rectangle</code></pre>
</figure>

::: next
#### Читаем дальше

[Паттерны проектирования на Java []{.fa
.fa-arrow-right}](../../java.html){.btn .btn-primary rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Шаблонный метод на
Go](../../template-method/go/example.html){.btn .btn-default rel="prev"}
:::

## **Посетитель** на других языках программирования

[![Посетитель на
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Посетитель на C#"){.prog-lang-link}
[![Посетитель на
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Посетитель на C++"){.prog-lang-link}
[![Посетитель на
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Посетитель на Java"){.prog-lang-link}
[![Посетитель на
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Посетитель на PHP"){.prog-lang-link}
[![Посетитель на
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Посетитель на Python"){.prog-lang-link}
[![Посетитель на
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Посетитель на Ruby"){.prog-lang-link}
[![Посетитель на
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Посетитель на Rust"){.prog-lang-link}
[![Посетитель на
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Посетитель на Swift"){.prog-lang-link}
[![Посетитель на
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Посетитель на TypeScript"){.prog-lang-link}
::::::::::::::::::::::::

::::::: {.prom .banner-sidebar .banner .banner-striped .banner-examples .banner-removable .banner-removable-patterns data-id="DP: Examples-IDE" creative-id="examples-sidebar-ru" position="sidebar"}
:::::: {.banner-inner style="font-size: 14px; line-height: 21px;"}
::: banner-examples-img
[![](../../../../images/patterns/banners/examples-ide91ca.png?id=3115b4b548fb96b75974e2de8f4f49bc){width="215"
height="158" loading="lazy"
srcset="/images/patterns/banners/examples-ide-2x.png?id=93c007a6d157b97c28eb213f59d716ae 2x"}](../../book.html)
:::

::: banner-examples-fade
[ Архив с примерами](../../book.html){.btn .btn-secondary .btn-block}
:::

::: {.banner-examples-text style="margin-top: 1rem;"}
Купи книгу **Погружение в Паттерны** и получи архив с десятками
детальных примеров, которые можно открывать прямо в IDE.

[ Узнать больше...](../../book.html){.btn .btn-secondary .btn-block}
:::
::::::
:::::::
::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::
