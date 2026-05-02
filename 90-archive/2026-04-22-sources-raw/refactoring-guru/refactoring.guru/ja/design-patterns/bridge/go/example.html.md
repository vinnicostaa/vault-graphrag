:::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} /
[デザインパターン](../../../design-patterns.html){.type} /
[Bridge](../../bridge.html){.type} / [Go](../../go.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Bridge](../../../../images/patterns/cards/bridge-mini9e60.png?id=b389101d8ee8e23ffa1b534c704d0774){srcset="/images/patterns/cards/bridge-mini-2x.png?id=2622384cf623ed150ee9c21a0812dd87 2x"}
:::

# **Bridge** を Go で {#bridge-を-go-で .pattern-example-title-block-title}
::::

::::::::::::::::: pattern-example-body
::: pattern-example-brief
**Bridge** は[、]{.chpule2}[
]{.chpuri2}構造に関するデザインパターンの一つで[、]{.chpule2}[
]{.chpuri2}ビジネス・ロジックや巨大なクラスを独立して開発可能なクラス階層に分割します[。]{.chpule2}[
]{.chpuri2}

階層の一つ[ ]{.chpule1}[（]{.chpuri1}抽象化と呼ばれる[）]{.chpule2}[
]{.chpuri2}は[、]{.chpule2}[ ]{.chpuri2}二つ目の階層[
]{.chpule1}[（]{.chpuri1}実装[）]{.chpule2}[
]{.chpuri2}のオブジェクトへの参照を持ちます[。]{.chpule2}[
]{.chpuri2}抽象化階層は[、]{.chpule2}[
]{.chpuri2}その呼び出しのいくつか[
]{.chpule1}[（]{.chpuri1}場合によっては大多数[）]{.chpule2}[
]{.chpuri2}を実装階層のオブジェクトに委任します[。]{.chpule2}[
]{.chpuri2}すべての実装は[、]{.chpule2}[
]{.chpuri2}共通のインターフェースを持っているので[、]{.chpule2}[
]{.chpuri2}抽象化の中で入れ替え可能です[。]{.chpule2}[ ]{.chpuri2}
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Bridge の詳細 ](../../bridge.html){.btn .btn-lg .btn-primary}
:::

:::::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
ナビゲーション
:::

::: en-intro
 [はじめに](#)
:::

::: en-intro
 [概念的な例](#example-0)
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

## 概念的な例 {#example-0-title}

ここに 2 種類のコンピューターがあるとします[。]{.chpule2}[
]{.chpuri2}Mac と Windows です[。]{.chpule2}[
]{.chpuri2}そして[、]{.chpule2}[ ]{.chpuri2}2
種類のプリンターがあるとします[。]{.chpule2}[ ]{.chpuri2}Epson と HP
です[。]{.chpule2}[
]{.chpuri2}両方のコンピューターとプリンターは[、]{.chpule2}[
]{.chpuri2}任意の組み合わせで動作する必要があります[。]{.chpule2}[
]{.chpuri2}クライアントは[、]{.chpule2}[
]{.chpuri2}プリンターをコンピューターに接続する詳細について気を使いたくありません[。]{.chpule2}[
]{.chpuri2}

新しいプリンターを導入するとして[、]{.chpule2}[
]{.chpuri2}コードが指数関数的に増大するのは避けたいものです[。]{.chpule2}[
]{.chpuri2}2 × 2 の組み合わせのために 4
個の構造体を作成する代わりに[、]{.chpule2}[
]{.chpuri2}二つの階層を作成します[：]{.chpule2}[ ]{.chpuri2}

-   抽象化階層[：]{.chpule2}[ ]{.chpuri2}コンピュータに対応します
-   実装階層[：]{.chpule2}[ ]{.chpuri2}プリンターに対応します

この二つの階層は[、]{.chpule2}[
]{.chpuri2}ブリッジを介して通信します[。]{.chpule2}[
]{.chpuri2}ここでは[、]{.chpule2}[ ]{.chpuri2}抽象化[
]{.chpule1}[（]{.chpuri1}コンピューター[）]{.chpule2}[
]{.chpuri2}は[、]{.chpule2}[ ]{.chpuri2}実装[
]{.chpule1}[（]{.chpuri1}プリンター[）]{.chpule2}[
]{.chpuri2}への参照を保持します[。]{.chpule2}[
]{.chpuri2}抽象化階層と実装階層は[、]{.chpule2}[
]{.chpuri2}互いに悪影響を与えることなく[、]{.chpule2}[
]{.chpuri2}独立して開発できます[。]{.chpule2}[ ]{.chpuri2}

#### []{#example-0--computer-go .anchor} **computer.go:** 抽象化

<figure class="code">
<pre class="code" lang="go"><code>package main

type Computer interface {
    Print()
    SetPrinter(Printer)
}</code></pre>
</figure>

#### []{#example-0--mac-go .anchor} **mac.go:** 特化した抽象化

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

#### []{#example-0--windows-go .anchor} **windows.go:** 特化した抽象化

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

#### []{#example-0--printer-go .anchor} **printer.go:** 実装

<figure class="code">
<pre class="code" lang="go"><code>package main

type Printer interface {
    PrintFile()
}</code></pre>
</figure>

#### []{#example-0--epson-go .anchor} **epson.go:** 具象実装

<figure class="code">
<pre class="code" lang="go"><code>package main

import &quot;fmt&quot;

type Epson struct {
}

func (p *Epson) PrintFile() {
    fmt.Println(&quot;Printing by a EPSON Printer&quot;)
}</code></pre>
</figure>

#### []{#example-0--hp-go .anchor} **hp.go:** 具象実装

<figure class="code">
<pre class="code" lang="go"><code>package main

import &quot;fmt&quot;

type Hp struct {
}

func (p *Hp) PrintFile() {
    fmt.Println(&quot;Printing by a HP Printer&quot;)
}</code></pre>
</figure>

#### []{#example-0--main-go .anchor} **main.go:** クライアント・コード

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

#### []{#example-0--output-txt .anchor} **output.txt:** 実行結果

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
#### 次を読む

[Composite を Go で []{.fa
.fa-arrow-right}](../../composite/go/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### 戻る

[[]{.fa .fa-arrow-left} Adapter を Go
で](../../adapter/go/example.html){.btn .btn-default rel="prev"}
:::

## 他言語での **Bridge**

[![Bridge を C#
で](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Bridge を C# で"){.prog-lang-link}
[![Bridge を C++
で](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Bridge を C++ で"){.prog-lang-link}
[![Bridge を Java
で](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Bridge を Java で"){.prog-lang-link}
[![Bridge を PHP
で](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Bridge を PHP で"){.prog-lang-link}
[![Bridge を Python
で](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Bridge を Python で"){.prog-lang-link}
[![Bridge を Ruby
で](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Bridge を Ruby で"){.prog-lang-link}
[![Bridge を Rust
で](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Bridge を Rust で"){.prog-lang-link}
[![Bridge を Swift
で](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Bridge を Swift で"){.prog-lang-link}
[![Bridge を TypeScript
で](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Bridge を TypeScript で"){.prog-lang-link}
:::::::::::::::::::::::

::::::: {.prom .banner-sidebar .banner .banner-striped .banner-examples .banner-removable .banner-removable-patterns data-id="DP: Examples-IDE" creative-id="examples-sidebar-ja" position="sidebar"}
:::::: {.banner-inner style="font-size: 14px; line-height: 21px;"}
::: banner-examples-img
[![](../../../../images/patterns/banners/examples-ide91ca.png?id=3115b4b548fb96b75974e2de8f4f49bc){width="215"
height="158" loading="lazy"
srcset="/images/patterns/banners/examples-ide-2x.png?id=93c007a6d157b97c28eb213f59d716ae 2x"}](../../book.html)
:::

::: banner-examples-fade
[ コード例の書庫](../../book.html){.btn .btn-secondary .btn-block}
:::

::: {.banner-examples-text style="margin-top: 1rem;"}
**直撃！デザインパターン**を電子書籍で購入し、IDE
で開くことができる数多くの詳細な例を含む書庫にアクセス。

[ 詳細](../../book.html){.btn .btn-secondary .btn-block}
:::
::::::
:::::::
:::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::
