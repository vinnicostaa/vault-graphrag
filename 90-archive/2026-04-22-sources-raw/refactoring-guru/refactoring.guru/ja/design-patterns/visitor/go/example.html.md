::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} /
[デザインパターン](../../../design-patterns.html){.type} /
[Visitor](../../visitor.html){.type} / [Go](../../go.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Visitor](../../../../images/patterns/cards/visitor-mini0a1c.png?id=854a35a62963bec1d75eab996918989b){srcset="/images/patterns/cards/visitor-mini-2x.png?id=9b87f3f3b772f434b28a25876829b504 2x"}
:::

# **Visitor** を Go で {#visitor-を-go-で .pattern-example-title-block-title}
::::

:::::::::::::::::: pattern-example-body
::: pattern-example-brief
**Visitor** は[、]{.chpule2}[
]{.chpuri2}振る舞いに関するデザインパターンの一つで[、]{.chpule2}[
]{.chpuri2}既存コードを変更することなく[、]{.chpule2}[
]{.chpuri2}既存のクラス階層に新しい振る舞いの追加を可能とします[。]{.chpule2}[
]{.chpuri2}

> Visitor の代わりに単純にメソッドの多重定義[
> ]{.chpule1}[（]{.chpuri1}overload[）]{.chpule2}[
> ]{.chpuri2}を使うことができない理由については[、]{.chpule2}[
> ]{.chpuri2}別の記事[
> ]{.chpule1}[『]{.chpuri1}[ビジターと二重ディスパッチ](../../visitor-double-dispatch.html)[』]{.chpule2}[
> ]{.chpuri2}を参照[。]{.chpule2}[ ]{.chpuri2}
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Visitor の詳細 ](../../visitor.html){.btn .btn-lg .btn-primary}
:::

::::::::::::::: {.sidebar-navigation .with-scroll}
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

## 概念的な例 {#example-0-title}

Visitor パターンを使うと[、]{.chpule2}[
]{.chpuri2}ある構造体への新しい振る舞いの追加を[、]{.chpule2}[
]{.chpuri2}構造体そのものを変更することなく行えます[。]{.chpule2}[
]{.chpuri2}以下のような図形を表現する構造体を含むライブラリーの保守をしているとしましょう[：]{.chpule2}[
]{.chpuri2}

-   Square[ ]{.chpule1}[（]{.chpuri1}正方形[）]{.chpule2}[ ]{.chpuri2}
-   Circle[ ]{.chpule1}[（]{.chpuri1}円[）]{.chpule2}[ ]{.chpuri2}
-   Triangle[ ]{.chpule1}[（]{.chpuri1}三角形[）]{.chpule2}[ ]{.chpuri2}

上記の図形構造体は[、]{.chpule2}[
]{.chpuri2}それぞれ共通の図形インターフェースを実装します[。]{.chpule2}[
]{.chpuri2}

会社の人たちが[、]{.chpule2}[
]{.chpuri2}あなたの素晴らしいライブラリーを使い始めるとすぐに[、]{.chpule2}[
]{.chpuri2}新機能リクエストが殺到しました[。]{.chpule2}[
]{.chpuri2}一番簡単なのを見てみましょう[：]{.chpule2}[
]{.chpuri2}あるチームからは[、]{.chpule2}[ ]{.chpuri2}shape 構造体に
`get­Area`[ ]{.chpule1}[（]{.chpuri1}面積を取得[）]{.chpule2}[
]{.chpuri2}機能を追加して欲しいというリクエストが来ました[。]{.chpule2}[
]{.chpuri2}

この問題を解決するには[、]{.chpule2}[
]{.chpuri2}いくつかの選択肢があります[。]{.chpule2}[ ]{.chpuri2}

最初に思い浮かぶ選択肢は[、]{.chpule2}[ ]{.chpuri2}shape
インターフェースに直接 `get­Area` メソッドを追加し[、]{.chpule2}[
]{.chpuri2}shape
を実装した構造体それぞれで実装することです[。]{.chpule2}[
]{.chpuri2}これは[、]{.chpule2}[
]{.chpuri2}まったくもっともで[、]{.chpule2}[
]{.chpuri2}すぐに実施すべきなように思えますが[、]{.chpule2}[
]{.chpuri2}タダというわけではありません[。]{.chpule2}[
]{.chpuri2}ライブラリーの保守担当者としては[、]{.chpule2}[
]{.chpuri2}誰かが新機能をリクエストするたびに[、]{.chpule2}[
]{.chpuri2}大切なコードを壊してしまうリスクを負いたくありません[。]{.chpule2}[
]{.chpuri2}それでも[、]{.chpule2}[
]{.chpuri2}他のチームにライブラリーの拡張ができるようにしてあげたいと思っています[。]{.chpule2}[
]{.chpuri2}

二つ目の選択肢は[、]{.chpule2}[
]{.chpuri2}新機能をリクエストしているチームが自分たちでそれを実装できるようにすることです[。]{.chpule2}[
]{.chpuri2}しかし[、]{.chpule2}[
]{.chpuri2}新規の振る舞いは非公開のコードに依存しているかもしれないので[、]{.chpule2}[
]{.chpuri2}これが常に可能とは限りません[。]{.chpule2}[ ]{.chpuri2}

三つ目の選択肢は[、]{.chpule2}[ ]{.chpuri2}Visitor
パターンを使ってこの問題を解くことです[。]{.chpule2}[
]{.chpuri2}まず[、]{.chpule2}[
]{.chpuri2}このようなビジター・インターフェースを定義します[：]{.chpule2}[
]{.chpuri2}

<figure class="code">
<pre class="code"><code>type visitor interface {
   visitForSquare(square)
   visitForCircle(circle)
   visitForTriangle(triangle)
}</code></pre>
</figure>

`visit­For­Square­(square)`[、]{.chpule2}[
]{.chpuri2}`visit­For­Circle­(circle)`[、]{.chpule2}[
]{.chpuri2}`visit­For­Triangle­(triangle)` といった関数は[、]{.chpule2}[
]{.chpuri2}それぞれ[、]{.chpule2}[ ]{.chpuri2}正方形[、]{.chpule2}[
]{.chpuri2}円[、]{.chpule2}[
]{.chpuri2}三角形に機能を追加します[。]{.chpule2}[ ]{.chpuri2}

何でビジター・インターフェースに `visit­(shape)`
というメソッドを一つだけ追加しないんだと疑問に思いますか[？]{.chpule2}[
]{.chpuri2}理由は[、]{.chpule2}[ ]{.chpuri2}Go
言語がメソッドの多重定義をサポートしていないからです[。]{.chpule2}[
]{.chpuri2}パラメーターだけ異なり名前が同じメソッドは許可されていません[。]{.chpule2}[
]{.chpuri2}

二つ目の重要な事項は[、]{.chpule2}[ ]{.chpuri2}shape
インターフェースに[、]{.chpule2}[ ]{.chpuri2}`accept`
メソッドを追加することです[。]{.chpule2}[ ]{.chpuri2}

<figure class="code">
<pre class="code"><code>func accept(v visitor)</code></pre>
</figure>

shape 構造体はすべて[、]{.chpule2}[
]{.chpuri2}このメソッドを定義する必要があります[。]{.chpule2}[
]{.chpuri2}例[：]{.chpule2}[ ]{.chpuri2}

<figure class="code">
<pre class="code"><code>func (obj *square) accept(v visitor){
    v.visitForSquare(obj)
}</code></pre>
</figure>

ちょっと待って[！]{.chpule2}[
]{.chpuri2}さっき既存の構造体は変更したくないと言わなかったですか[？]{.chpule2}[
]{.chpuri2}残念なことに[、]{.chpule2}[ ]{.chpuri2}Visitor
パターンを利用する時は[、]{.chpule2}[ ]{.chpuri2}shape
の構造体を変更する必要があります[。]{.chpule2}[
]{.chpuri2}しかし[、]{.chpule2}[
]{.chpuri2}この変更は一度だけです[。]{.chpule2}[ ]{.chpuri2}

`get­Num­Sides` とか `get­Middle­Coordinates`
のような振る舞いを追加する場合でも[、]{.chpule2}[ ]{.chpuri2}同じ
`accept­(v visitor)` 関数が使えるので[、]{.chpule2}[ ]{.chpuri2}shape
構造体への更なる変更は不要です[。]{.chpule2}[ ]{.chpuri2}

つまり[、]{.chpule2}[ ]{.chpuri2}shape
構造体は一度だけ変更する必要があり[、]{.chpule2}[
]{.chpuri2}将来のすべての機能追加リクエストは[、]{.chpule2}[
]{.chpuri2}同一の accept 関数で対処できます[。]{.chpule2}[
]{.chpuri2}チームが[、]{.chpule2}[ ]{.chpuri2} `get­Area`
機能の追加をリクエストした場合[、]{.chpule2}[
]{.chpuri2}単にビジター・インターフェースの具象実装を定義し[、]{.chpule2}[
]{.chpuri2}そこに面積計算のロジックを書くだけですみます[。]{.chpule2}[
]{.chpuri2}

#### []{#example-0--shape-go .anchor} **shape.go:** 要素

<figure class="code">
<pre class="code" lang="go"><code>package main

type Shape interface {
    getType() string
    accept(Visitor)
}</code></pre>
</figure>

#### []{#example-0--square-go .anchor} **square.go:** 具象要素

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

#### []{#example-0--circle-go .anchor} **circle.go:** 具象要素

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

#### []{#example-0--rectangle-go .anchor} **rectangle.go:** 具象要素

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

#### []{#example-0--visitor-go .anchor} **visitor.go:** ビジター

<figure class="code">
<pre class="code" lang="go"><code>package main

type Visitor interface {
    visitForSquare(*Square)
    visitForCircle(*Circle)
    visitForrectangle(*Rectangle)
}</code></pre>
</figure>

#### []{#example-0--areaCalculator-go .anchor} **areaCalculator.go:** 具象ビジター

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

#### []{#example-0--middleCoordinates-go .anchor} **middleCoordinates.go:** 具象ビジター

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

#### []{#example-0--main-go .anchor} **main.go:** クライアント・コード

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

#### []{#example-0--output-txt .anchor} **output.txt:** 実行結果

<figure class="code">
<pre class="code" lang="output"><code>Calculating area for square
Calculating area for circle
Calculating area for rectangle

Calculating middle point coordinates for square
Calculating middle point coordinates for circle
Calculating middle point coordinates for rectangle</code></pre>
</figure>

::: next
#### 次を読む

[デザインパターンを Java で []{.fa
.fa-arrow-right}](../../java.html){.btn .btn-primary rel="next"}
:::

::: prev
#### 戻る

[[]{.fa .fa-arrow-left} Template Method を Go
で](../../template-method/go/example.html){.btn .btn-default rel="prev"}
:::

## 他言語での **Visitor**

[![Visitor を C#
で](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Visitor を C# で"){.prog-lang-link}
[![Visitor を C++
で](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Visitor を C++ で"){.prog-lang-link}
[![Visitor を Java
で](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Visitor を Java で"){.prog-lang-link}
[![Visitor を PHP
で](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Visitor を PHP で"){.prog-lang-link}
[![Visitor を Python
で](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Visitor を Python で"){.prog-lang-link}
[![Visitor を Ruby
で](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Visitor を Ruby で"){.prog-lang-link}
[![Visitor を Rust
で](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Visitor を Rust で"){.prog-lang-link}
[![Visitor を Swift
で](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Visitor を Swift で"){.prog-lang-link}
[![Visitor を TypeScript
で](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Visitor を TypeScript で"){.prog-lang-link}
::::::::::::::::::::::::

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
::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::
