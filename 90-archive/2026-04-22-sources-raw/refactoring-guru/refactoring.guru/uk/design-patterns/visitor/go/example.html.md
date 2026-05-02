::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Патерни
проектування](../../../design-patterns.html){.type} /
[Відвідувач](../../visitor.html){.type} / [Go](../../go.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Відвідувач](../../../../images/patterns/cards/visitor-mini0a1c.png?id=854a35a62963bec1d75eab996918989b){srcset="/images/patterns/cards/visitor-mini-2x.png?id=9b87f3f3b772f434b28a25876829b504 2x"}
:::

# **Відвідувач** на Go {#відвідувач-на-go .pattern-example-title-block-title}
::::

:::::::::::::::::: pattern-example-body
::: pattern-example-brief
**Відвідувач** --- це поведінковий патерн, який дозволяє додати нову
операцію для цілої ієрархії класів, не змінюючи код цих класів.

> Детальніше про те, чому Відвідувач не можна замінити звичайним
> перевантаженням методів читайте в статті [Відвідувач і Double
> Dispatch](../../visitor-double-dispatch.html).
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Детальніше про Відвідувач ](../../visitor.html){.btn .btn-lg
.btn-primary}
:::

::::::::::::::: {.sidebar-navigation .with-scroll}
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

## Концептуальний приклад {#example-0-title}

Патерн Відвідувач дає змогу вам додавати поведінку в структуру без її
зміни. Уявімо, що ви розробник бібліотеки, яка містить структури різних
фігур:

-   Квадрат
-   Коло
-   Трикутник

Структури кожної з вищеназваних фігур реалізують спільний інтерфейс
фігури.

Як тільки співробітники у вашій компанії почали використовувати вашу
чудову бібліотеку, вас засипали проханнями додати той чи інший
функціонал. Розглянемо один з найпростіших варіантів: команда попросила
вас додати в структуру функцію `getArea`, що повертає площу фігури.

Є багато рішень цієї проблеми.

Перше, що спадає на думку --- додати метод `getArea` безпосередньо в
інтерфейс фігури, і потім реалізувати його у кожній структурі. Це
здається найбільш правильним рішенням, але у нього є свої мінуси. Як
розробник бібліотеки, ви ризикуєте зламати ваш дорогоцінний код щоразу,
коли додаєте нову поведінку. Незважаючи на це, ви хочете дати іншим
командам можливість якимось чином розширювати вашу бібліотеку.

Другий варіант --- команда, яка запитує додавання нової функції, може
сама реалізувати її в своєму проекті. Однак, це не завжди є можливим,
оскільки новий функціонал може покладатися на прихований код.

Третій можливий варіант --- вирішити вищезгадану проблему завдяки
використанню патерна Відвідувач. Спершу ми визначаємо інтерфейс
користувача в такий спосіб:

<figure class="code">
<pre class="code"><code>type visitor interface {
   visitForSquare(square)
   visitForCircle(circle)
   visitForTriangle(triangle)
}</code></pre>
</figure>

Функції `visitForSquare(square)`, `visitForCircle(circle)`,
`visitForTriangle(triangle)` дозволять нам додавати функціонал для
квадратів, кіл і трикутників відповідно.

Не розумієте, чому ми не можемо залишити лише один метод `visit(shape)`
в інтерфейсі користувача? Це неможливо через те, що мова Go не підтримує
перевантаження методів, тому ви не можете мати методи з однаковими
іменами, але різними параметрами.

Другий, не менш важливий, етап --- додавання методу `accept` в інтерфейс
фігури.

<figure class="code">
<pre class="code"><code>func accept(v visitor)</code></pre>
</figure>

Всі структури фігур мусять визначати цей метод схожим способом:

<figure class="code">
<pre class="code"><code>func (obj *square) accept(v visitor){
    v.visitForSquare(obj)
}</code></pre>
</figure>

Стривайте, хіба я трохи раніше не заявляв, що не хочу змінювати наявні
структури фігур? На жаль, під час використання патерну Відвідувача нам
доведеться вносити зміни в структури, але лише один раз.

У разі додавання іншого функціоналу, наприклад `getNumSides` або
`getMiddleCoordinates`, ми будемо використовувати все той же метод
`accept(v visitor)` без нових змін у структурах фігур.

В кінцевому підсумку, структури потрібно змінити лише один раз, і всі
майбутні запити нового функціоналу можна буде реалізувати завдяки
функції `accept(v visitor)`. Якщо команда дасть запит на поведінку
`getArea`, ми можемо просто визначити явну реалізацію інтерфейсу
користувача і прописати логіку обчислення площі в цій конкретній
імплементації.

#### []{#example-0--shape-go .anchor} **shape.go:** Елемент

<figure class="code">
<pre class="code" lang="go"><code>package main

type Shape interface {
    getType() string
    accept(Visitor)
}</code></pre>
</figure>

#### []{#example-0--square-go .anchor} **square.go:** Конкретний елемент

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

#### []{#example-0--circle-go .anchor} **circle.go:** Конкретний елемент

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

#### []{#example-0--rectangle-go .anchor} **rectangle.go:** Конкретний елемент

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

#### []{#example-0--visitor-go .anchor} **visitor.go:** Відвідувач

<figure class="code">
<pre class="code" lang="go"><code>package main

type Visitor interface {
    visitForSquare(*Square)
    visitForCircle(*Circle)
    visitForrectangle(*Rectangle)
}</code></pre>
</figure>

#### []{#example-0--areaCalculator-go .anchor} **areaCalculator.go:** Конкретний відвідувач

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

#### []{#example-0--middleCoordinates-go .anchor} **middleCoordinates.go:** Конкретний відвідувач

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

#### []{#example-0--main-go .anchor} **main.go:** Клієнтський код

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

#### []{#example-0--output-txt .anchor} **output.txt:** Результат виконання

<figure class="code">
<pre class="code" lang="output"><code>Calculating area for square
Calculating area for circle
Calculating area for rectangle

Calculating middle point coordinates for square
Calculating middle point coordinates for circle
Calculating middle point coordinates for rectangle</code></pre>
</figure>

::: next
#### Читаємо далі

[Патерни проектування на Java []{.fa
.fa-arrow-right}](../../java.html){.btn .btn-primary rel="next"}
:::

::: prev
#### Повернутись назад

[[]{.fa .fa-arrow-left} Шаблонний метод на
Go](../../template-method/go/example.html){.btn .btn-default rel="prev"}
:::

## **Відвідувач** іншими мовами програмування

[![Відвідувач на
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Відвідувач на C#"){.prog-lang-link}
[![Відвідувач на
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Відвідувач на C++"){.prog-lang-link}
[![Відвідувач на
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Відвідувач на Java"){.prog-lang-link}
[![Відвідувач на
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Відвідувач на PHP"){.prog-lang-link}
[![Відвідувач на
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Відвідувач на Python"){.prog-lang-link}
[![Відвідувач на
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Відвідувач на Ruby"){.prog-lang-link}
[![Відвідувач на
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Відвідувач на Rust"){.prog-lang-link}
[![Відвідувач на
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Відвідувач на Swift"){.prog-lang-link}
[![Відвідувач на
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Відвідувач на TypeScript"){.prog-lang-link}
::::::::::::::::::::::::

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
::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::
