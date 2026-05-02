::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} /
[デザインパターン](../../../design-patterns.html){.type} /
[Builder](../../builder.html){.type} / [Go](../../go.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Builder](../../../../images/patterns/cards/builder-mini6d24.png?id=19b95fd05e6469679752c0554b116815){srcset="/images/patterns/cards/builder-mini-2x.png?id=de6d0938678b86903a1426dddfdd13bf 2x"}
:::

# **Builder** を Go で {#builder-を-go-で .pattern-example-title-block-title}
::::

:::::::::::::::: pattern-example-body
::: pattern-example-brief
**Builder** は生成に関するデザインパターンの一つで[、]{.chpule2}[
]{.chpuri2}複雑なオブジェクトを段階的に構築することができます[。]{.chpule2}[
]{.chpuri2}

他の生成に関するパターンとは異なり[、]{.chpule2}[ ]{.chpuri2}Builder
ではプロダクト[ ]{.chpule1}[（]{.chpuri1}訳注[：]{.chpule2}[
]{.chpuri2}本パターンでは[、]{.chpule2}[
]{.chpuri2}生成されるモノのことを一般にプロダクトと呼びます[）]{.chpule2}[
]{.chpuri2}が共通のインターフェースを持つ必要はありません[。]{.chpule2}[
]{.chpuri2}このため[、]{.chpule2}[
]{.chpuri2}同じ構築の手続きを経て[、]{.chpule2}[
]{.chpuri2}異なるプロダクトを作成することができます[。]{.chpule2}[
]{.chpuri2}
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Builder の詳細 ](../../builder.html){.btn .btn-lg .btn-primary}
:::

::::::::::::: {.sidebar-navigation .with-scroll}
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

## 概念的な例 {#example-0-title}

Builder パターンは[、]{.chpule2}[
]{.chpuri2}目的とするプロダクトが複雑で[、]{.chpule2}[
]{.chpuri2}完成まで多くのステップが必要な時に使われます[。]{.chpule2}[
]{.chpuri2}この場合[、]{.chpule2}[
]{.chpuri2}単一の超巨大なコンストラクターを持つよりも[、]{.chpule2}[
]{.chpuri2}いくつかの構築メソッドを持つ方がシンプルです[。]{.chpule2}[
]{.chpuri2}多段階での構築方法では[、]{.chpule2}[
]{.chpuri2}未完成の[、]{.chpule2}[
]{.chpuri2}安全性に欠けるプロダクトがクライアントから利用可能になってしまうという潜在的問題がありますが[、]{.chpule2}[
]{.chpuri2}Builder パターンを使うと[、]{.chpule2}[
]{.chpuri2}構築が完了するまで[、]{.chpule2}[
]{.chpuri2}プロダクトは外部に非公開の状態が保たれます[。]{.chpule2}[
]{.chpuri2}

以下のコードでは[、]{.chpule2}[ ]{.chpuri2}異なる種類の家[
]{.chpule1}[（]{.chpuri1}`igloo` と `normal­House`[）]{.chpule2}[
]{.chpuri2}が[、]{.chpule2}[ ]{.chpuri2}`igloo­Builder` と
`normal­Builder` によって構築されています[。]{.chpule2}[
]{.chpuri2}どちらの種類の家も[、]{.chpule2}[
]{.chpuri2}同じ構築ステップを使います[。]{.chpule2}[
]{.chpuri2}ディレクター構造体は[、]{.chpule2}[
]{.chpuri2}構築過程を組織化する役に立ちます[。]{.chpule2}[ ]{.chpuri2}

#### []{#example-0--iBuilder-go .anchor} **iBuilder.go:** ビルダーのインターフェース

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

#### []{#example-0--normalBuilder-go .anchor} **normalBuilder.go:** 具象ビルダー

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

#### []{#example-0--iglooBuilder-go .anchor} **iglooBuilder.go:** 具象ビルダー

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

#### []{#example-0--house-go .anchor} **house.go:** 製品

<figure class="code">
<pre class="code" lang="go"><code>package main

type House struct {
    windowType string
    doorType   string
    floor      int
}</code></pre>
</figure>

#### []{#example-0--director-go .anchor} **director.go:** ディレクター

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

#### []{#example-0--main-go .anchor} **main.go:** クライアント・コード

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

#### []{#example-0--output-txt .anchor} **output.txt:** 実行結果

<figure class="code">
<pre class="code" lang="output"><code>Normal House Door Type: Wooden Door
Normal House Window Type: Wooden Window
Normal House Num Floor: 2

Igloo House Door Type: Snow Door
Igloo House Window Type: Snow Window
Igloo House Num Floor: 1</code></pre>
</figure>

::: next
#### 次を読む

[Factory Method を Go で []{.fa
.fa-arrow-right}](../../factory-method/go/example.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### 戻る

[[]{.fa .fa-arrow-left} Abstract Factory を Go
で](../../abstract-factory/go/example.html){.btn .btn-default
rel="prev"}
:::

## 他言語での **Builder**

[![Builder を C#
で](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Builder を C# で"){.prog-lang-link}
[![Builder を C++
で](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Builder を C++ で"){.prog-lang-link}
[![Builder を Java
で](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Builder を Java で"){.prog-lang-link}
[![Builder を PHP
で](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Builder を PHP で"){.prog-lang-link}
[![Builder を Python
で](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Builder を Python で"){.prog-lang-link}
[![Builder を Ruby
で](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Builder を Ruby で"){.prog-lang-link}
[![Builder を Rust
で](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Builder を Rust で"){.prog-lang-link}
[![Builder を Swift
で](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Builder を Swift で"){.prog-lang-link}
[![Builder を TypeScript
で](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Builder を TypeScript で"){.prog-lang-link}
::::::::::::::::::::::

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
::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::
