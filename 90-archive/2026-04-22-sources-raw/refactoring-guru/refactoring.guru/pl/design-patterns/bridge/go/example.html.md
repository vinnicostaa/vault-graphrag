:::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Wzorce
projektowe](../../../design-patterns.html){.type} /
[Most](../../bridge.html){.type} / [Go](../../go.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Most](../../../../images/patterns/cards/bridge-mini9e60.png?id=b389101d8ee8e23ffa1b534c704d0774){srcset="/images/patterns/cards/bridge-mini-2x.png?id=2622384cf623ed150ee9c21a0812dd87 2x"}
:::

# **Most** w języku Go {#most-w-języku-go .pattern-example-title-block-title}
::::

::::::::::::::::: pattern-example-body
::: pattern-example-brief
**Most** jest strukturalnym wzorcem projektowym zakładającym podział
logiki biznesowej lub dużej klasy na osobne hierarchie klas które
następnie można rozwijać niezależnie od siebie.

Jedna z takich hierarchii (zwana często Abstrakcją) posiada
referencję do obiektu drugiej hierarchii (zwanej Implementacją) i
deleguje mu część (czasem większość) wywołań. Ponieważ wszystkie
implementacje mają wspólny interfejs, z punktu widzenia abstrakcji są
wymienialne.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Dowiedz się więcej o Most ](../../bridge.html){.btn .btn-lg
.btn-primary}
:::

:::::::::::::: {.sidebar-navigation .with-scroll}
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

## Przykład koncepcyjny {#example-0-title}

Przypuśćmy, że mamy dwa rodzaje komputerów: Mac i Windows, a do tego dwa
rodzaje drukarek: Epson i HP. Każdy z komputerów powinien móc
współdziałać z każdą z drukarek w dowolnej kombinacji. Klient nie chce
martwić się szczegółami łączenia ze sobą komputerów i drukarek.

Nie chcemy aby kod rósł wykładniczo gdy przyjdzie pora kupić nowe
drukarki. Zamiast tworzyć cztery struktury z myślą o tej kombinacji
2\*2, tworzymy dwie hierarchie:

-   Hierarchia abstrakcyjna: nasze komputery,
-   Hierarchia implementacyjna: nasze drukarki.

Obie hierarchie będą komunikować się ze sobą za pośrednictwem Mostu,
gdzie Abstrakcja (komputer) będzie posiadała odniesienie do
Implementacji (drukarka). Zarówno abstrakcję, jak i implementację można
wówczas rozwijać niezależnie.

#### []{#example-0--computer-go .anchor} **computer.go:** Abstrakcja

<figure class="code">
<pre class="code" lang="go"><code>package main

type Computer interface {
    Print()
    SetPrinter(Printer)
}</code></pre>
</figure>

#### []{#example-0--mac-go .anchor} **mac.go:** Wzbogacona abstrakcja

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

#### []{#example-0--windows-go .anchor} **windows.go:** Wzbogacona abstrakcja

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

#### []{#example-0--printer-go .anchor} **printer.go:** Implementacja

<figure class="code">
<pre class="code" lang="go"><code>package main

type Printer interface {
    PrintFile()
}</code></pre>
</figure>

#### []{#example-0--epson-go .anchor} **epson.go:** Konkretna implementacja

<figure class="code">
<pre class="code" lang="go"><code>package main

import &quot;fmt&quot;

type Epson struct {
}

func (p *Epson) PrintFile() {
    fmt.Println(&quot;Printing by a EPSON Printer&quot;)
}</code></pre>
</figure>

#### []{#example-0--hp-go .anchor} **hp.go:** Konkretna implementacja

<figure class="code">
<pre class="code" lang="go"><code>package main

import &quot;fmt&quot;

type Hp struct {
}

func (p *Hp) PrintFile() {
    fmt.Println(&quot;Printing by a HP Printer&quot;)
}</code></pre>
</figure>

#### []{#example-0--main-go .anchor} **main.go:** Kod klienta

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

#### []{#example-0--output-txt .anchor} **output.txt:** Wynik działania

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
#### Czytaj dalej

[Kompozyt w języku Go []{.fa
.fa-arrow-right}](../../composite/go/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Wróć

[[]{.fa .fa-arrow-left} Adapter w języku
Go](../../adapter/go/example.html){.btn .btn-default rel="prev"}
:::

## **Most** w innych językach

[![Most w języku
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Most w języku C#"){.prog-lang-link}
[![Most w języku
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Most w języku C++"){.prog-lang-link}
[![Most w języku
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Most w języku Java"){.prog-lang-link}
[![Most w języku
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Most w języku PHP"){.prog-lang-link}
[![Most w języku
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Most w języku Python"){.prog-lang-link}
[![Most w języku
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Most w języku Ruby"){.prog-lang-link}
[![Most w języku
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Most w języku Rust"){.prog-lang-link}
[![Most w języku
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Most w języku Swift"){.prog-lang-link}
[![Most w języku
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Most w języku TypeScript"){.prog-lang-link}
:::::::::::::::::::::::

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
:::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::
