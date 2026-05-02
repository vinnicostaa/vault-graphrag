::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Паттерны
проектирования](../../../design-patterns.html){.type} /
[Строитель](../../builder.html){.type} / [Go](../../go.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Строитель](../../../../images/patterns/cards/builder-mini6d24.png?id=19b95fd05e6469679752c0554b116815){srcset="/images/patterns/cards/builder-mini-2x.png?id=de6d0938678b86903a1426dddfdd13bf 2x"}
:::

# **Строитель** на Go {#строитель-на-go .pattern-example-title-block-title}
::::

:::::::::::::::: pattern-example-body
::: pattern-example-brief
**Строитель** --- это порождающий паттерн проектирования, который
позволяет создавать объекты пошагово.

В отличие от других порождающих паттернов, Строитель позволяет
производить различные продукты, используя один и тот же процесс
строительства.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Подробней о паттерне Строитель ](../../builder.html){.btn .btn-lg
.btn-primary}
:::

::::::::::::: {.sidebar-navigation .with-scroll}
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
 [i­Builder](#example-0--iBuilder-go)
:::

::: en-file
 [normal­Builder](#example-0--normalBuilder-go)
:::

::: en-file
 [igloo­Builder](#example-0--iglooBuilder-go)
:::

::: en-file
 [house](#example-0--house-go)
:::

::: en-file
 [director](#example-0--director-go)
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

## Концептуальный пример {#example-0-title}

Паттерн Строитель также используется, когда нужный продукт сложный и
требует нескольких шагов для построения. В таких случаях несколько
конструкторных методов подойдут лучше, чем один громадный конструктор.
При использовании пошагового построения объектов потенциальной проблемой
является выдача клиенту частично построенного нестабильного продукта.
Паттерн \"Строитель\" скрывает объект до тех пор, пока он не построен до
конца.

Нижеприведенный код использует разные типы домов (`igloo` и
`normalHouse`), которые конструируются с помощью строителей
`iglooBuilder` и `normalBuilder`. При создании каждого дома используются
одинаковые шаги. Для помощи в организации процесса можно использовать
директор, хоть это и необязательно.

#### []{#example-0--iBuilder-go .anchor} **iBuilder.go:** Интерфейс cтроителя

<figure class="code">
<pre class="code" lang="go"><code>package main

type IBuilder interface {
    setWindowType()
    setDoorType()
    setNumFloor()
    getHouse() House
}

func getBuilder(builderType string) IBuilder {
    if builderType == &quot;normal&quot; {
        return newNormalBuilder()
    }

    if builderType == &quot;igloo&quot; {
        return newIglooBuilder()
    }
    return nil
}</code></pre>
</figure>

#### []{#example-0--normalBuilder-go .anchor} **normalBuilder.go:** Конкретный строитель

<figure class="code">
<pre class="code" lang="go"><code>package main

type NormalBuilder struct {
    windowType string
    doorType   string
    floor      int
}

func newNormalBuilder() *NormalBuilder {
    return &amp;NormalBuilder{}
}

func (b *NormalBuilder) setWindowType() {
    b.windowType = &quot;Wooden Window&quot;
}

func (b *NormalBuilder) setDoorType() {
    b.doorType = &quot;Wooden Door&quot;
}

func (b *NormalBuilder) setNumFloor() {
    b.floor = 2
}

func (b *NormalBuilder) getHouse() House {
    return House{
        doorType:   b.doorType,
        windowType: b.windowType,
        floor:      b.floor,
    }
}</code></pre>
</figure>

#### []{#example-0--iglooBuilder-go .anchor} **iglooBuilder.go:** Конкретный строитель

<figure class="code">
<pre class="code" lang="go"><code>package main

type IglooBuilder struct {
    windowType string
    doorType   string
    floor      int
}

func newIglooBuilder() *IglooBuilder {
    return &amp;IglooBuilder{}
}

func (b *IglooBuilder) setWindowType() {
    b.windowType = &quot;Snow Window&quot;
}

func (b *IglooBuilder) setDoorType() {
    b.doorType = &quot;Snow Door&quot;
}

func (b *IglooBuilder) setNumFloor() {
    b.floor = 1
}

func (b *IglooBuilder) getHouse() House {
    return House{
        doorType:   b.doorType,
        windowType: b.windowType,
        floor:      b.floor,
    }
}</code></pre>
</figure>

#### []{#example-0--house-go .anchor} **house.go:** Продукт

<figure class="code">
<pre class="code" lang="go"><code>package main

type House struct {
    windowType string
    doorType   string
    floor      int
}</code></pre>
</figure>

#### []{#example-0--director-go .anchor} **director.go:** Директор

<figure class="code">
<pre class="code" lang="go"><code>package main

type Director struct {
    builder IBuilder
}

func newDirector(b IBuilder) *Director {
    return &amp;Director{
        builder: b,
    }
}

func (d *Director) setBuilder(b IBuilder) {
    d.builder = b
}

func (d *Director) buildHouse() House {
    d.builder.setDoorType()
    d.builder.setWindowType()
    d.builder.setNumFloor()
    return d.builder.getHouse()
}</code></pre>
</figure>

#### []{#example-0--main-go .anchor} **main.go:** Клиентский код

<figure class="code">
<pre class="code" lang="go"><code>package main

import &quot;fmt&quot;

func main() {
    normalBuilder := getBuilder(&quot;normal&quot;)
    iglooBuilder := getBuilder(&quot;igloo&quot;)

    director := newDirector(normalBuilder)
    normalHouse := director.buildHouse()

    fmt.Printf(&quot;Normal House Door Type: %s\n&quot;, normalHouse.doorType)
    fmt.Printf(&quot;Normal House Window Type: %s\n&quot;, normalHouse.windowType)
    fmt.Printf(&quot;Normal House Num Floor: %d\n&quot;, normalHouse.floor)

    director.setBuilder(iglooBuilder)
    iglooHouse := director.buildHouse()

    fmt.Printf(&quot;\nIgloo House Door Type: %s\n&quot;, iglooHouse.doorType)
    fmt.Printf(&quot;Igloo House Window Type: %s\n&quot;, iglooHouse.windowType)
    fmt.Printf(&quot;Igloo House Num Floor: %d\n&quot;, iglooHouse.floor)

}</code></pre>
</figure>

#### []{#example-0--output-txt .anchor} **output.txt:** Результат выполнения

<figure class="code">
<pre class="code" lang="output"><code>Normal House Door Type: Wooden Door
Normal House Window Type: Wooden Window
Normal House Num Floor: 2

Igloo House Door Type: Snow Door
Igloo House Window Type: Snow Window
Igloo House Num Floor: 1</code></pre>
</figure>

::: next
#### Читаем дальше

[Фабричный метод на Go []{.fa
.fa-arrow-right}](../../factory-method/go/example.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Абстрактная фабрика на
Go](../../abstract-factory/go/example.html){.btn .btn-default
rel="prev"}
:::

## **Строитель** на других языках программирования

[![Строитель на
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Строитель на C#"){.prog-lang-link}
[![Строитель на
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Строитель на C++"){.prog-lang-link}
[![Строитель на
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Строитель на Java"){.prog-lang-link}
[![Строитель на
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Строитель на PHP"){.prog-lang-link}
[![Строитель на
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Строитель на Python"){.prog-lang-link}
[![Строитель на
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Строитель на Ruby"){.prog-lang-link}
[![Строитель на
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Строитель на Rust"){.prog-lang-link}
[![Строитель на
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Строитель на Swift"){.prog-lang-link}
[![Строитель на
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Строитель на TypeScript"){.prog-lang-link}
::::::::::::::::::::::

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
::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::
