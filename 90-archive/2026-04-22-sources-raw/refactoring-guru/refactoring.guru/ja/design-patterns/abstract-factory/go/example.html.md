::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} /
[デザインパターン](../../../design-patterns.html){.type} / [Abstract
Factory](../../abstract-factory.html){.type} /
[Go](../../go.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Abstract
Factory](../../../../images/patterns/cards/abstract-factory-minid1c5.png?id=4c3927c446313a38ce77dfee38111e27){srcset="/images/patterns/cards/abstract-factory-mini-2x.png?id=22236aaa65ff52cbde1c713216d52c1f 2x"}
:::

# **Abstract Factory** を Go で {#abstract-factory-を-go-で .pattern-example-title-block-title}
::::

:::::::::::::::::::: pattern-example-body
::: pattern-example-brief
**Abstract Factory** は[、]{.chpule2}[
]{.chpuri2}生成に関するデザインパターンのひとつで[、]{.chpule2}[
]{.chpuri2}具象クラスを指定することなく[、]{.chpule2}[
]{.chpuri2}プロダクト[ ]{.chpule1}[（]{.chpuri1}訳注[：]{.chpule2}[
]{.chpuri2}本パターンでは[、]{.chpule2}[
]{.chpuri2}生成されるモノのことを一般にプロダクトと呼びます[）]{.chpule2}[
]{.chpuri2}のファミリー全部を生成することを可能とします[。]{.chpule2}[
]{.chpuri2}

Abstract Factory は[、]{.chpule2}[
]{.chpuri2}個々のプロダクト全部を作成するためのインターフェースを定義しますが[、]{.chpule2}[
]{.chpuri2}実際のプロダクト作成の作業は[、]{.chpule2}[
]{.chpuri2}具象クラスに委ねられます[。]{.chpule2}[
]{.chpuri2}ファクトリーの型[
]{.chpule1}[（]{.chpuri1}クラス[）]{.chpule2}[
]{.chpuri2}それぞれは[、]{.chpule2}[
]{.chpuri2}特定のプロダクトの異種に対応します[。]{.chpule2}[ ]{.chpuri2}

クライアント・コードは[、]{.chpule2}[
]{.chpuri2}コンストラクター呼び出し[ ]{.chpule1}[（]{.chpuri1}`new`
演算子[）]{.chpule2}[
]{.chpuri2}で直接プロダクトを作成する代わりにファクトリー・オブジェクトの作成メソッドを呼び出します[。]{.chpule2}[
]{.chpuri2}
ファクトリーはプロダクトの特定の異種に対応しているため[、]{.chpule2}[
]{.chpuri2}すべてのプロダクトには互換性があります[。]{.chpule2}[
]{.chpuri2}

クライアント・コードは[、]{.chpule2}[
]{.chpuri2}その抽象インターフェイスを通じてのみファクトリーやプロダクトとやりとりします[。]{.chpule2}[
]{.chpuri2}このため[、]{.chpule2}[
]{.chpuri2}クライアント・コードはファクトリー・オブジェクトによって作成された任意のプロダクトの異種と動作します[。]{.chpule2}[
]{.chpuri2}プログラマーがやるべきことは[、]{.chpule2}[
]{.chpuri2}新しい具象ファクトリー・クラスを作成し[、]{.chpule2}[
]{.chpuri2}それをクライアント・コードに渡すことです[。]{.chpule2}[
]{.chpuri2}

> もし各種ファクトリー系のパターンやコンセプトの違いで迷った場合は[、]{.chpule2}[
> ]{.chpuri2}[ファクトリーの比較](../../factory-comparison.html)
> をご覧ください[。]{.chpule2}[ ]{.chpuri2}
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Abstract Factory の詳細 ](../../abstract-factory.html){.btn .btn-lg
.btn-primary}
:::

::::::::::::::::: {.sidebar-navigation .with-scroll}
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
 [i­Sports­Factory](#example-0--iSportsFactory-go)
:::

::: en-file
 [adidas](#example-0--adidas-go)
:::

::: en-file
 [nike](#example-0--nike-go)
:::

::: en-file
 [i­Shoe](#example-0--iShoe-go)
:::

::: en-file
 [adidas­Shoe](#example-0--adidasShoe-go)
:::

::: en-file
 [nike­Shoe](#example-0--nikeShoe-go)
:::

::: en-file
 [i­Shirt](#example-0--iShirt-go)
:::

::: en-file
 [adidas­Shirt](#example-0--adidasShirt-go)
:::

::: en-file
 [nike­Shirt](#example-0--nikeShirt-go)
:::

::: en-file
 [main](#example-0--main-go)
:::

::: en-file
 [output](#example-0--output-txt)
:::
:::::::::::::::::
::::::::::::::::::::

[]{#example-0}

## 概念的な例 {#example-0-title}

靴 1 足とシャツ 1
枚の二つの異なった製品の組み合わせであるスポーツ・キットを買う必要があるとします[。]{.chpule2}[
]{.chpuri2}あなたは[、]{.chpule2}[
]{.chpuri2}同じブランドのスポーツ・キットを購入したいと思っています[。]{.chpule2}[
]{.chpuri2}

もしこれをコードに転換したければ[、]{.chpule2}[ ]{.chpuri2}Abstract
Factory が[、]{.chpule2}[
]{.chpuri2}常に互いにマッチするプロダクトの組を作成するのに役立ちます[。]{.chpule2}[
]{.chpuri2}

#### []{#example-0--iSportsFactory-go .anchor} **iSportsFactory.go:** Abstract Factory インターフェース

<figure class="code">
<pre class="code" lang="go"><code>package main

import &quot;fmt&quot;

type ISportsFactory interface {
    makeShoe() IShoe
    makeShirt() IShirt
}

func GetSportsFactory(brand string) (ISportsFactory, error) {
    if brand == &quot;adidas&quot; {
        return &amp;Adidas{}, nil
    }

    if brand == &quot;nike&quot; {
        return &amp;Nike{}, nil
    }

    return nil, fmt.Errorf(&quot;Wrong brand type passed&quot;)
}</code></pre>
</figure>

#### []{#example-0--adidas-go .anchor} **adidas.go:** 具象ファクトリー

<figure class="code">
<pre class="code" lang="go"><code>package main

type Adidas struct {
}

func (a *Adidas) makeShoe() IShoe {
    return &amp;AdidasShoe{
        Shoe: Shoe{
            logo: &quot;adidas&quot;,
            size: 14,
        },
    }
}

func (a *Adidas) makeShirt() IShirt {
    return &amp;AdidasShirt{
        Shirt: Shirt{
            logo: &quot;adidas&quot;,
            size: 14,
        },
    }
}</code></pre>
</figure>

#### []{#example-0--nike-go .anchor} **nike.go:** 具象ファクトリー

<figure class="code">
<pre class="code" lang="go"><code>package main

type Nike struct {
}

func (n *Nike) makeShoe() IShoe {
    return &amp;NikeShoe{
        Shoe: Shoe{
            logo: &quot;nike&quot;,
            size: 14,
        },
    }
}

func (n *Nike) makeShirt() IShirt {
    return &amp;NikeShirt{
        Shirt: Shirt{
            logo: &quot;nike&quot;,
            size: 14,
        },
    }
}</code></pre>
</figure>

#### []{#example-0--iShoe-go .anchor} **iShoe.go:** 抽象プロダクト

<figure class="code">
<pre class="code" lang="go"><code>package main

type IShoe interface {
    setLogo(logo string)
    setSize(size int)
    getLogo() string
    getSize() int
}

type Shoe struct {
    logo string
    size int
}

func (s *Shoe) setLogo(logo string) {
    s.logo = logo
}

func (s *Shoe) getLogo() string {
    return s.logo
}

func (s *Shoe) setSize(size int) {
    s.size = size
}

func (s *Shoe) getSize() int {
    return s.size
}</code></pre>
</figure>

#### []{#example-0--adidasShoe-go .anchor} **adidasShoe.go:** 具象プロダクト

<figure class="code">
<pre class="code" lang="go"><code>package main

type AdidasShoe struct {
    Shoe
}</code></pre>
</figure>

#### []{#example-0--nikeShoe-go .anchor} **nikeShoe.go:** 具象プロダクト

<figure class="code">
<pre class="code" lang="go"><code>package main

type NikeShoe struct {
    Shoe
}</code></pre>
</figure>

#### []{#example-0--iShirt-go .anchor} **iShirt.go:** 抽象プロダクト

<figure class="code">
<pre class="code" lang="go"><code>package main

type IShirt interface {
    setLogo(logo string)
    setSize(size int)
    getLogo() string
    getSize() int
}

type Shirt struct {
    logo string
    size int
}

func (s *Shirt) setLogo(logo string) {
    s.logo = logo
}

func (s *Shirt) getLogo() string {
    return s.logo
}

func (s *Shirt) setSize(size int) {
    s.size = size
}

func (s *Shirt) getSize() int {
    return s.size
}</code></pre>
</figure>

#### []{#example-0--adidasShirt-go .anchor} **adidasShirt.go:** 具象プロダクト

<figure class="code">
<pre class="code" lang="go"><code>package main

type AdidasShirt struct {
    Shirt
}</code></pre>
</figure>

#### []{#example-0--nikeShirt-go .anchor} **nikeShirt.go:** 具象プロダクト

<figure class="code">
<pre class="code" lang="go"><code>package main

type NikeShirt struct {
    Shirt
}</code></pre>
</figure>

#### []{#example-0--main-go .anchor} **main.go:** クライアント・コード

<figure class="code">
<pre class="code" lang="go"><code>package main

import &quot;fmt&quot;

func main() {
    adidasFactory, _ := GetSportsFactory(&quot;adidas&quot;)
    nikeFactory, _ := GetSportsFactory(&quot;nike&quot;)

    nikeShoe := nikeFactory.makeShoe()
    nikeShirt := nikeFactory.makeShirt()

    adidasShoe := adidasFactory.makeShoe()
    adidasShirt := adidasFactory.makeShirt()

    printShoeDetails(nikeShoe)
    printShirtDetails(nikeShirt)

    printShoeDetails(adidasShoe)
    printShirtDetails(adidasShirt)
}

func printShoeDetails(s IShoe) {
    fmt.Printf(&quot;Logo: %s&quot;, s.getLogo())
    fmt.Println()
    fmt.Printf(&quot;Size: %d&quot;, s.getSize())
    fmt.Println()
}

func printShirtDetails(s IShirt) {
    fmt.Printf(&quot;Logo: %s&quot;, s.getLogo())
    fmt.Println()
    fmt.Printf(&quot;Size: %d&quot;, s.getSize())
    fmt.Println()
}</code></pre>
</figure>

#### []{#example-0--output-txt .anchor} **output.txt:** 実行結果

<figure class="code">
<pre class="code" lang="output"><code>Logo: nike
Size: 14
Logo: nike
Size: 14
Logo: adidas
Size: 14
Logo: adidas
Size: 14</code></pre>
</figure>

::: next
#### 次を読む

[Builder を Go で []{.fa
.fa-arrow-right}](../../builder/go/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### 戻る

[[]{.fa .fa-arrow-left} デザインパターンを Go で](../../go.html){.btn
.btn-default rel="prev"}
:::

## 他言語での **Abstract Factory**

[![Abstract Factory を C#
で](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Abstract Factory を C# で"){.prog-lang-link}
[![Abstract Factory を C++
で](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Abstract Factory を C++ で"){.prog-lang-link}
[![Abstract Factory を Java
で](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Abstract Factory を Java で"){.prog-lang-link}
[![Abstract Factory を PHP
で](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Abstract Factory を PHP で"){.prog-lang-link}
[![Abstract Factory を Python
で](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Abstract Factory を Python で"){.prog-lang-link}
[![Abstract Factory を Ruby
で](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Abstract Factory を Ruby で"){.prog-lang-link}
[![Abstract Factory を Rust
で](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Abstract Factory を Rust で"){.prog-lang-link}
[![Abstract Factory を Swift
で](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Abstract Factory を Swift で"){.prog-lang-link}
[![Abstract Factory を TypeScript
で](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Abstract Factory を TypeScript で"){.prog-lang-link}
::::::::::::::::::::::::::

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
::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::
