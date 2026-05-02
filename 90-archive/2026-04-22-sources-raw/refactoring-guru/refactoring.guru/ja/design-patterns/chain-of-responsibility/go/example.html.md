:::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} /
[デザインパターン](../../../design-patterns.html){.type} / [Chain of
Responsibility](../../chain-of-responsibility.html){.type} /
[Go](../../go.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Chain of
Responsibility](../../../../images/patterns/cards/chain-of-responsibility-mini6d42.png?id=36d85eba8d14986f053123de17aac7a7){srcset="/images/patterns/cards/chain-of-responsibility-mini-2x.png?id=8c81f7069e51259b2443801b91135f7f 2x"}
:::

# **Chain of Responsibility** を Go で {#chain-of-responsibility-を-go-で .pattern-example-title-block-title}
::::

::::::::::::::::: pattern-example-body
::: pattern-example-brief
**Chain of Responsibility** は[、]{.chpule2}[
]{.chpuri2}振る舞いに関するデザインパターンの一つで[、]{.chpule2}[
]{.chpuri2}潜在的なハンドラーの連鎖の上を[、]{.chpule2}[
]{.chpuri2}ハンドラーのどれかが処理するまで[、]{.chpule2}[
]{.chpuri2}リクエストを回していきます[。]{.chpule2}[ ]{.chpuri2}

このパターンを利用すると[、]{.chpule2}[
]{.chpuri2}送り手のクラスと受け手の具象クラスとを結合することなく[、]{.chpule2}[
]{.chpuri2}複数のオブジェクトにリクエストを処理する機会を与えることができます[。]{.chpule2}[
]{.chpuri2}連鎖は実行時に[、]{.chpule2}[
]{.chpuri2}標準のハンドラー・インターフェースに従うハンドラーから動的に構成されます[。]{.chpule2}[
]{.chpuri2}
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Chain of Responsibility の詳細
](../../chain-of-responsibility.html){.btn .btn-lg .btn-primary}
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
 [department](#example-0--department-go)
:::

::: en-file
 [reception](#example-0--reception-go)
:::

::: en-file
 [doctor](#example-0--doctor-go)
:::

::: en-file
 [medical](#example-0--medical-go)
:::

::: en-file
 [cashier](#example-0--cashier-go)
:::

::: en-file
 [patient](#example-0--patient-go)
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

Chain of Responsibility パターンを[、]{.chpule2}[
]{.chpuri2}病院アプリを例に見ていきましょう[。]{.chpule2}[
]{.chpuri2}病院には以下のような複数の場所や役割があります[：]{.chpule2}[
]{.chpuri2}

-   受付
-   医者
-   薬局
-   支払い

病人が到着すると[、]{.chpule2}[ ]{.chpuri2}まず受付に行き[、]{.chpule2}[
]{.chpuri2}医者に会い[、]{.chpule2}[
]{.chpuri2}薬局に行き[、]{.chpule2}[
]{.chpuri2}そして支払い窓口などへ行きます[。]{.chpule2}[
]{.chpuri2}患者は[、]{.chpule2}[
]{.chpuri2}部局の連鎖の中を回され[、]{.chpule2}[
]{.chpuri2}各部局はその機能を終えると[、]{.chpule2}[
]{.chpuri2}患者を次の部局へ送ります[。]{.chpule2}[ ]{.chpuri2}

このパターンは[、]{.chpule2}[
]{.chpuri2}同一リクエストを処理する複数の候補がある時[、]{.chpule2}[
]{.chpuri2}適用可能です[。]{.chpule2}[
]{.chpuri2}リクエストを処理できる複数のオブジェクトがあり[、]{.chpule2}[
]{.chpuri2}クライアントに受け手の選択をさせたくない場合も[、]{.chpule2}[
]{.chpuri2}このパターンが役に立ちます[。]{.chpule2}[
]{.chpuri2}もう一つの役に立つケースとしては[、]{.chpule2}[
]{.chpuri2}クライアントを受け手から分離したい場合です[。]{.chpule2}[
]{.chpuri2}クライアントは[、]{.chpule2}[
]{.chpuri2}連鎖の最初の要素だけ知っておく必要があります[。]{.chpule2}[
]{.chpuri2}

病院の例のように[、]{.chpule2}[
]{.chpuri2}患者は最初に受け付けに行きます[。]{.chpule2}[
]{.chpuri2}その後[、]{.chpule2}[
]{.chpuri2}患者の現在の状態に基づいて[、]{.chpule2}[
]{.chpuri2}受け付けは連鎖内の次のハンドラーに送り出します[。]{.chpule2}[
]{.chpuri2}

#### []{#example-0--department-go .anchor} **department.go:** ハンドラー・インターフェース

<figure class="code">
<pre class="code" lang="go"><code>package main

type Department interface {
    execute(*Patient)
    setNext(Department)
}</code></pre>
</figure>

#### []{#example-0--reception-go .anchor} **reception.go:** 具象ハンドラー

<figure class="code">
<pre class="code" lang="go"><code>package main

import &quot;fmt&quot;

type Reception struct {
    next Department
}

func (r *Reception) execute(p *Patient) {
    if p.registrationDone {
        fmt.Println(&quot;Patient registration already done&quot;)
        r.next.execute(p)
        return
    }
    fmt.Println(&quot;Reception registering patient&quot;)
    p.registrationDone = true
    r.next.execute(p)
}

func (r *Reception) setNext(next Department) {
    r.next = next
}</code></pre>
</figure>

#### []{#example-0--doctor-go .anchor} **doctor.go:** 具象ハンドラー

<figure class="code">
<pre class="code" lang="go"><code>package main

import &quot;fmt&quot;

type Doctor struct {
    next Department
}

func (d *Doctor) execute(p *Patient) {
    if p.doctorCheckUpDone {
        fmt.Println(&quot;Doctor checkup already done&quot;)
        d.next.execute(p)
        return
    }
    fmt.Println(&quot;Doctor checking patient&quot;)
    p.doctorCheckUpDone = true
    d.next.execute(p)
}

func (d *Doctor) setNext(next Department) {
    d.next = next
}</code></pre>
</figure>

#### []{#example-0--medical-go .anchor} **medical.go:** 具象ハンドラー

<figure class="code">
<pre class="code" lang="go"><code>package main

import &quot;fmt&quot;

type Medical struct {
    next Department
}

func (m *Medical) execute(p *Patient) {
    if p.medicineDone {
        fmt.Println(&quot;Medicine already given to patient&quot;)
        m.next.execute(p)
        return
    }
    fmt.Println(&quot;Medical giving medicine to patient&quot;)
    p.medicineDone = true
    m.next.execute(p)
}

func (m *Medical) setNext(next Department) {
    m.next = next
}</code></pre>
</figure>

#### []{#example-0--cashier-go .anchor} **cashier.go:** 具象ハンドラー

<figure class="code">
<pre class="code" lang="go"><code>package main

import &quot;fmt&quot;

type Cashier struct {
    next Department
}

func (c *Cashier) execute(p *Patient) {
    if p.paymentDone {
        fmt.Println(&quot;Payment Done&quot;)
    }
    fmt.Println(&quot;Cashier getting money from patient patient&quot;)
}

func (c *Cashier) setNext(next Department) {
    c.next = next
}</code></pre>
</figure>

#### []{#example-0--patient-go .anchor} **patient.go**

<figure class="code">
<pre class="code" lang="go"><code>package main

type Patient struct {
    name              string
    registrationDone  bool
    doctorCheckUpDone bool
    medicineDone      bool
    paymentDone       bool
}</code></pre>
</figure>

#### []{#example-0--main-go .anchor} **main.go:** クライアント・コード

<figure class="code">
<pre class="code" lang="go"><code>package main

func main() {

    cashier := &amp;Cashier{}

    //Set next for medical department
    medical := &amp;Medical{}
    medical.setNext(cashier)

    //Set next for doctor department
    doctor := &amp;Doctor{}
    doctor.setNext(medical)

    //Set next for reception department
    reception := &amp;Reception{}
    reception.setNext(doctor)

    patient := &amp;Patient{name: &quot;abc&quot;}
    //Patient visiting
    reception.execute(patient)
}</code></pre>
</figure>

#### []{#example-0--output-txt .anchor} **output.txt:** 実行結果

<figure class="code">
<pre class="code" lang="output"><code>Reception registering patient
Doctor checking patient
Medical giving medicine to patient
Cashier getting money from patient patient</code></pre>
</figure>

::: next
#### 次を読む

[Command を Go で []{.fa
.fa-arrow-right}](../../command/go/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### 戻る

[[]{.fa .fa-arrow-left} Proxy を Go
で](../../proxy/go/example.html){.btn .btn-default rel="prev"}
:::

## 他言語での **Chain of Responsibility**

[![Chain of Responsibility を C#
で](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Chain of Responsibility を C# で"){.prog-lang-link}
[![Chain of Responsibility を C++
で](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Chain of Responsibility を C++ で"){.prog-lang-link}
[![Chain of Responsibility を Java
で](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Chain of Responsibility を Java で"){.prog-lang-link}
[![Chain of Responsibility を PHP
で](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Chain of Responsibility を PHP で"){.prog-lang-link}
[![Chain of Responsibility を Python
で](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Chain of Responsibility を Python で"){.prog-lang-link}
[![Chain of Responsibility を Ruby
で](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Chain of Responsibility を Ruby で"){.prog-lang-link}
[![Chain of Responsibility を Rust
で](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Chain of Responsibility を Rust で"){.prog-lang-link}
[![Chain of Responsibility を Swift
で](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Chain of Responsibility を Swift で"){.prog-lang-link}
[![Chain of Responsibility を TypeScript
で](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Chain of Responsibility を TypeScript で"){.prog-lang-link}
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
