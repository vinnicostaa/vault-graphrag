:::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Патерни
проектування](../../../design-patterns.html){.type} /
[Міст](../../bridge.html){.type} / [Go](../../go.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Міст](../../../../images/patterns/cards/bridge-mini9e60.png?id=b389101d8ee8e23ffa1b534c704d0774){srcset="/images/patterns/cards/bridge-mini-2x.png?id=2622384cf623ed150ee9c21a0812dd87 2x"}
:::

# **Міст** на Go {#міст-на-go .pattern-example-title-block-title}
::::

::::::::::::::::: pattern-example-body
::: pattern-example-brief
**Міст** --- це структурний патерн, який розділяє бізнес-логіку або
великий клас на кілька окремих ієрархій, які можна розвивати далі окремо
одну від одної.

Одна з цих ієрархій (абстракція) отримає посилання на об'єкти іншої
ієрархії (реалізація) і буде делегувати їм основну роботу. Завдяки тому,
що всі реалізації будуть дотримуватись спільного інтерфейсу, їх можна
буде взаємозамінювати всередині абстракції.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Детальніше про Міст ](../../bridge.html){.btn .btn-lg .btn-primary}
:::

:::::::::::::: {.sidebar-navigation .with-scroll}
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

## Концептуальний приклад {#example-0-title}

Уявімо, що у вас є два типи комп'ютерів: Mac і Windows, а також два типи
принтерів: Epson і HP. Комп'ютери та принтери мусять працювати між собою
у будь-яких комбінаціях. Клієнт не хоче думати про особливості
підключення принтерів до комп'ютерів.

Ми не хочемо, щоб при введенні в цю систему нових принтерів кількість
коду збільшувалася по експоненті. Замість створення чотирьох структур
для 2 × 2 комбінацій, ми створимо дві ієрархії:

-   Ієрархія абстракції: сюди будуть входити наші комп'ютери
-   Ієрархія реалізації: сюди будуть входити наші принтери

Ці дві ієрархії спілкуються між собою за допомогою Моста, в якому
Абстракція (комп'ютер) містить посилання на Реалізацію (принтер). І
абстракцію, і реалізацію можна розробляти окремо, не впливаючи одне на
одного.

#### []{#example-0--computer-go .anchor} **computer.go:** Абстракція

<figure class="code">
<pre class="code" lang="go"><code>package main

type Computer interface {
    Print()
    SetPrinter(Printer)
}</code></pre>
</figure>

#### []{#example-0--mac-go .anchor} **mac.go:** Розширена абстракція

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

#### []{#example-0--windows-go .anchor} **windows.go:** Розширена абстракція

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

#### []{#example-0--printer-go .anchor} **printer.go:** Реалізація

<figure class="code">
<pre class="code" lang="go"><code>package main

type Printer interface {
    PrintFile()
}</code></pre>
</figure>

#### []{#example-0--epson-go .anchor} **epson.go:** Конкретна реалізація

<figure class="code">
<pre class="code" lang="go"><code>package main

import &quot;fmt&quot;

type Epson struct {
}

func (p *Epson) PrintFile() {
    fmt.Println(&quot;Printing by a EPSON Printer&quot;)
}</code></pre>
</figure>

#### []{#example-0--hp-go .anchor} **hp.go:** Конкретна реалізація

<figure class="code">
<pre class="code" lang="go"><code>package main

import &quot;fmt&quot;

type Hp struct {
}

func (p *Hp) PrintFile() {
    fmt.Println(&quot;Printing by a HP Printer&quot;)
}</code></pre>
</figure>

#### []{#example-0--main-go .anchor} **main.go:** Клієнтський код

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

#### []{#example-0--output-txt .anchor} **output.txt:** Результат виконання

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
#### Читаємо далі

[Компонувальник на Go []{.fa
.fa-arrow-right}](../../composite/go/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Повернутись назад

[[]{.fa .fa-arrow-left} Адаптер на
Go](../../adapter/go/example.html){.btn .btn-default rel="prev"}
:::

## **Міст** іншими мовами програмування

[![Міст на
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Міст на C#"){.prog-lang-link}
[![Міст на
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Міст на C++"){.prog-lang-link}
[![Міст на
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Міст на Java"){.prog-lang-link}
[![Міст на
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Міст на PHP"){.prog-lang-link}
[![Міст на
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Міст на Python"){.prog-lang-link}
[![Міст на
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Міст на Ruby"){.prog-lang-link}
[![Міст на
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Міст на Rust"){.prog-lang-link}
[![Міст на
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Міст на Swift"){.prog-lang-link}
[![Міст на
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Міст на TypeScript"){.prog-lang-link}
:::::::::::::::::::::::

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
:::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::
