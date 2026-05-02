:::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Паттерны
проектирования](../../../design-patterns.html){.type} /
[Мост](../../bridge.html){.type} / [Go](../../go.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Мост](../../../../images/patterns/cards/bridge-mini9e60.png?id=b389101d8ee8e23ffa1b534c704d0774){srcset="/images/patterns/cards/bridge-mini-2x.png?id=2622384cf623ed150ee9c21a0812dd87 2x"}
:::

# **Мост** на Go {#мост-на-go .pattern-example-title-block-title}
::::

::::::::::::::::: pattern-example-body
::: pattern-example-brief
**Мост** --- это структурный паттерн, который разделяет бизнес-логику
или большой класс на несколько отдельных иерархий, которые потом можно
развивать отдельно друг от друга.

Одна из этих иерархий (абстракция) получит ссылку на объекты другой
иерархии (реализация) и будет делегировать им основную работу. Благодаря
тому, что все реализации будут следовать общему интерфейсу, их можно
будет взаимозаменять внутри абстракции.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Подробней о паттерне Мост ](../../bridge.html){.btn .btn-lg
.btn-primary}
:::

:::::::::::::: {.sidebar-navigation .with-scroll}
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
 [computer](#example-0--computer-go)
:::

::: en-file
 [mac](#example-0--mac-go)
:::

::: en-file
 [windows](#example-0--windows-go)
:::

::: en-file
 [printer](#example-0--printer-go)
:::

::: en-file
 [epson](#example-0--epson-go)
:::

::: en-file
 [hp](#example-0--hp-go)
:::

::: en-file
 [main](#example-0--main-go)
:::

::: en-file
 [output](#example-0--output-txt)
:::
::::::::::::::
:::::::::::::::::

[]{#example-0}

## Концептуальный пример {#example-0-title}

Представим, что у вас есть два типа компьютеров: Mac и Windows, а также
два типа принтеров: Epson и HP. Компьютеры и принтеры должны работать
между собой в любых комбинациях. Клиент не хочет думать об особенностях
подключения принтеров к компьютерам.

Мы не хотим, чтобы при введении в эту систему новых принтеров количество
кода увеличивалось по экспоненте. Вместо создания четырех структур для
2\*2 комбинаций, мы создадим две иерархии:

-   Иерархия абстракции: сюда будут входить наши компьютеры
-   Иерархия реализации: сюда будут входить наши принтеры

Эти две иерархии общаются между собой посредством Моста, в котором
Абстракция (компьютер) содержит ссылку на Реализацию (принтер). И
абстракцию, и реализацию можно разрабатывать отдельно, не влияя друг на
друга.

#### []{#example-0--computer-go .anchor} **computer.go:** Абстракция

<figure class="code">
<pre class="code" lang="go"><code>package main

type Computer interface {
    Print()
    SetPrinter(Printer)
}</code></pre>
</figure>

#### []{#example-0--mac-go .anchor} **mac.go:** Расширенная абстракция

<figure class="code">
<pre class="code" lang="go"><code>package main

import &quot;fmt&quot;

type Mac struct {
    printer Printer
}

func (m *Mac) Print() {
    fmt.Println(&quot;Print request for mac&quot;)
    m.printer.PrintFile()
}

func (m *Mac) SetPrinter(p Printer) {
    m.printer = p
}</code></pre>
</figure>

#### []{#example-0--windows-go .anchor} **windows.go:** Расширенная абстракция

<figure class="code">
<pre class="code" lang="go"><code>package main

import &quot;fmt&quot;

type Windows struct {
    printer Printer
}

func (w *Windows) Print() {
    fmt.Println(&quot;Print request for windows&quot;)
    w.printer.PrintFile()
}

func (w *Windows) SetPrinter(p Printer) {
    w.printer = p
}</code></pre>
</figure>

#### []{#example-0--printer-go .anchor} **printer.go:** Реализация

<figure class="code">
<pre class="code" lang="go"><code>package main

type Printer interface {
    PrintFile()
}</code></pre>
</figure>

#### []{#example-0--epson-go .anchor} **epson.go:** Конкретная реализация

<figure class="code">
<pre class="code" lang="go"><code>package main

import &quot;fmt&quot;

type Epson struct {
}

func (p *Epson) PrintFile() {
    fmt.Println(&quot;Printing by a EPSON Printer&quot;)
}</code></pre>
</figure>

#### []{#example-0--hp-go .anchor} **hp.go:** Конкретная реализация

<figure class="code">
<pre class="code" lang="go"><code>package main

import &quot;fmt&quot;

type Hp struct {
}

func (p *Hp) PrintFile() {
    fmt.Println(&quot;Printing by a HP Printer&quot;)
}</code></pre>
</figure>

#### []{#example-0--main-go .anchor} **main.go:** Клиентский код

<figure class="code">
<pre class="code" lang="go"><code>package main

import &quot;fmt&quot;

func main() {

    hpPrinter := &amp;Hp{}
    epsonPrinter := &amp;Epson{}

    macComputer := &amp;Mac{}

    macComputer.SetPrinter(hpPrinter)
    macComputer.Print()
    fmt.Println()

    macComputer.SetPrinter(epsonPrinter)
    macComputer.Print()
    fmt.Println()

    winComputer := &amp;Windows{}

    winComputer.SetPrinter(hpPrinter)
    winComputer.Print()
    fmt.Println()

    winComputer.SetPrinter(epsonPrinter)
    winComputer.Print()
    fmt.Println()
}</code></pre>
</figure>

#### []{#example-0--output-txt .anchor} **output.txt:** Результат выполнения

<figure class="code">
<pre class="code" lang="output"><code>Print request for mac
Printing by a HP Printer

Print request for mac
Printing by a EPSON Printer

Print request for windows
Printing by a HP Printer

Print request for windows</code></pre>
</figure>

::: next
#### Читаем дальше

[Компоновщик на Go []{.fa
.fa-arrow-right}](../../composite/go/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Адаптер на
Go](../../adapter/go/example.html){.btn .btn-default rel="prev"}
:::

## **Мост** на других языках программирования

[![Мост на
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Мост на C#"){.prog-lang-link}
[![Мост на
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Мост на C++"){.prog-lang-link}
[![Мост на
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Мост на Java"){.prog-lang-link}
[![Мост на
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Мост на PHP"){.prog-lang-link}
[![Мост на
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Мост на Python"){.prog-lang-link}
[![Мост на
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Мост на Ruby"){.prog-lang-link}
[![Мост на
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Мост на Rust"){.prog-lang-link}
[![Мост на
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Мост на Swift"){.prog-lang-link}
[![Мост на
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Мост на TypeScript"){.prog-lang-link}
:::::::::::::::::::::::

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
:::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::
