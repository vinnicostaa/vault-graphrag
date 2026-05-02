:::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [디자인
패턴들](../../../design-patterns.html){.type} /
[브리지](../../bridge.html){.type} / [Go](../../go.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![브리지](../../../../images/patterns/cards/bridge-mini9e60.png?id=b389101d8ee8e23ffa1b534c704d0774){srcset="/images/patterns/cards/bridge-mini-2x.png?id=2622384cf623ed150ee9c21a0812dd87 2x"}
:::

# Go로 작성된 **브리지** {#go로-작성된-브리지 .pattern-example-title-block-title}
::::

::::::::::::::::: pattern-example-body
::: pattern-example-brief
**브리지**는 구조 디자인 패턴입니다. 이 패턴은 비즈니스 로직 또는 거대한
클래스를 독립적으로 개발할 수 있는 별도의 클래스 계층구조들로 나눕니다.

종종 추상화라고 불리는 이러한 계층구조 중 하나는 두 번째 계층구조의
객체​(구현)​에 대한 참조를 얻습니다. 추상화의 호출들 일부​(때로는 대부분)​를
구현 객체에 위임할 수 있습니다. 모든 구현은 공통 인터페이스를 가지므로
추상화 내에서 상호 교환할 수 있습니다.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ 브리지에 대하여 더 자세히 알아보세요 ](../../bridge.html){.btn .btn-lg
.btn-primary}
:::

:::::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
내비게이션
:::

::: en-intro
 [소개](#)
:::

::: en-intro
 [개념적인 예시](#example-0)
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

## 개념적인 예시 {#example-0-title}

맥과 윈도우 유형의 두 컴퓨터 유형이 있다고 가정해 봅시다. 또한 엡손과 HP
유형의 두 프린터 유형이 있다고 가정해 봅시다. 컴퓨터와 프린터 유형들은
모든 조합으로 서로 작동해야 합니다. 클라이언트는 프린터와 컴퓨터를
연결하는 세부 사항에 대하여 걱정하고 싶어 하지 않습니다.

또 우리는 새 프린터를 도입할 때 코드가 기하급수적으로 증가하는 것을
원하지 않습니다. 따라서 2\*2 조합에 대해 4개의 구조체를 만드는 대신
우리는 두 개의 계층구조를 만듭니다.

-   추상화 계층구조: 이것은 우리의 컴퓨터가 될 것입니다.
-   구현 계층구조: 이것은 우리의 프린터가 될 것입니다.

이 두 계층구조는 브리지를 통해 서로 통신하며, 여기서 추상화​(컴퓨터)​에는
구현​(프린터)​에 대한 참조가 포함됩니다. 추상화와 구현은 서로 영향을
미치지 않고 독립적으로 개발될 수 있습니다.

#### []{#example-0--computer-go .anchor} **computer.go:** 추상화

<figure class="code">
<pre class="code" lang="go"><code>package main

type Computer interface {
    Print()
    SetPrinter(Printer)
}</code></pre>
</figure>

#### []{#example-0--mac-go .anchor} **mac.go:** 정제된 추상화

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

#### []{#example-0--windows-go .anchor} **windows.go:** 정제된 추상화

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

#### []{#example-0--printer-go .anchor} **printer.go:** 구현

<figure class="code">
<pre class="code" lang="go"><code>package main

type Printer interface {
    PrintFile()
}</code></pre>
</figure>

#### []{#example-0--epson-go .anchor} **epson.go:** 구상 구현

<figure class="code">
<pre class="code" lang="go"><code>package main

import &quot;fmt&quot;

type Epson struct {
}

func (p *Epson) PrintFile() {
    fmt.Println(&quot;Printing by a EPSON Printer&quot;)
}</code></pre>
</figure>

#### []{#example-0--hp-go .anchor} **hp.go:** 구상 구현

<figure class="code">
<pre class="code" lang="go"><code>package main

import &quot;fmt&quot;

type Hp struct {
}

func (p *Hp) PrintFile() {
    fmt.Println(&quot;Printing by a HP Printer&quot;)
}</code></pre>
</figure>

#### []{#example-0--main-go .anchor} **main.go:** 클라이언트 코드

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

#### []{#example-0--output-txt .anchor} **output.txt:** 실행 결과

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
#### 다음을 보세요

[Go로 작성된 복합체 []{.fa
.fa-arrow-right}](../../composite/go/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### 돌아가기

[[]{.fa .fa-arrow-left} Go로 작성된
어댑터](../../adapter/go/example.html){.btn .btn-default rel="prev"}
:::

## 다른 언어로 작성된 **브리지**

[![C#으로 작성된
브리지](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "C#으로 작성된 브리지"){.prog-lang-link}
[![C++로 작성된
브리지](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "C++로 작성된 브리지"){.prog-lang-link}
[![자바로 작성된
브리지](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "자바로 작성된 브리지"){.prog-lang-link}
[![PHP로 작성된
브리지](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "PHP로 작성된 브리지"){.prog-lang-link}
[![파이썬으로 작성된
브리지](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "파이썬으로 작성된 브리지"){.prog-lang-link}
[![루비로 작성된
브리지](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "루비로 작성된 브리지"){.prog-lang-link}
[![러스트로 작성된
브리지](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "러스트로 작성된 브리지"){.prog-lang-link}
[![스위프트로 작성된
브리지](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "스위프트로 작성된 브리지"){.prog-lang-link}
[![타입스크립트로 작성된
브리지](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "타입스크립트로 작성된 브리지"){.prog-lang-link}
:::::::::::::::::::::::

::::::: {.prom .banner-sidebar .banner .banner-striped .banner-examples .banner-removable .banner-removable-patterns data-id="DP: Examples-IDE" creative-id="examples-sidebar-ko" position="sidebar"}
:::::: {.banner-inner style="font-size: 14px; line-height: 21px;"}
::: banner-examples-img
[![](../../../../images/patterns/banners/examples-ide91ca.png?id=3115b4b548fb96b75974e2de8f4f49bc){width="215"
height="158" loading="lazy"
srcset="/images/patterns/banners/examples-ide-2x.png?id=93c007a6d157b97c28eb213f59d716ae 2x"}](../../book.html)
:::

::: banner-examples-fade
[ 예시들을 포함한 저장소](../../book.html){.btn .btn-secondary
.btn-block}
:::

::: {.banner-examples-text style="margin-top: 1rem;"}
전자책 **디자인 패턴에 뛰어들기**를 구매하면 IDE에서 바로 열 수 있는
수십 가지의 상세한 코드 예시들이 포함된 저장소를 사용할 수 있습니다.

[ 더 알아보기...](../../book.html){.btn .btn-secondary .btn-block}
:::
::::::
:::::::
:::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::
