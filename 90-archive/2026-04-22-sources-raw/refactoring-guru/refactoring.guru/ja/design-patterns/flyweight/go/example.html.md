:::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} /
[デザインパターン](../../../design-patterns.html){.type} /
[Flyweight](../../flyweight.html){.type} / [Go](../../go.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Flyweight](../../../../images/patterns/cards/flyweight-minie3d8.png?id=422ca8d2f90614dce810a8812c626698){srcset="/images/patterns/cards/flyweight-mini-2x.png?id=857ca676fff84317bb0dab76abfce08e 2x"}
:::

# **Flyweight** を Go で {#flyweight-を-go-で .pattern-example-title-block-title}
::::

::::::::::::::::: pattern-example-body
::: pattern-example-brief
**Flyweight** は[、]{.chpule2}[
]{.chpuri2}構造に関するデザインパターンの一つで[、]{.chpule2}[
]{.chpuri2}メモリー消費量を低く抑えることで[、]{.chpule2}[
]{.chpuri2}プログラムが膨大な数のオブジェクトを支えることができるようにします[。]{.chpule2}[
]{.chpuri2}

複数のオブジェクト間でオブジェクトの状態の一部を共有することにより[、]{.chpule2}[
]{.chpuri2}これを実現します[。]{.chpule2}[
]{.chpuri2}つまり[、]{.chpule2}[ ]{.chpuri2}Flyweight は[、]{.chpule2}[
]{.chpuri2}異なるオブジェクトによって使われる同じデータをキャッシュすることにより[、]{.chpule2}[
]{.chpuri2}RAM を節約します[。]{.chpule2}[ ]{.chpuri2}
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Flyweight の詳細 ](../../flyweight.html){.btn .btn-lg .btn-primary}
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
 [dress­Factory](#example-0--dressFactory-go)
:::

::: en-file
 [dress](#example-0--dress-go)
:::

::: en-file
 [terrorist­Dress](#example-0--terroristDress-go)
:::

::: en-file
 [counter­Terrorist­Dress](#example-0--counterTerroristDress-go)
:::

::: en-file
 [player](#example-0--player-go)
:::

::: en-file
 [game](#example-0--game-go)
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

Counter-Strike[ ]{.chpule1}[（]{.chpuri1}反撃[）]{.chpule2}[
]{.chpuri2}というゲームでは[、]{.chpule2}[
]{.chpuri2}テロリストと反テロリストは[、]{.chpule2}[
]{.chpuri2}それぞれ違う種類の服を着ています[。]{.chpule2}[
]{.chpuri2}話を簡単にするため[、]{.chpule2}[
]{.chpuri2}テロリストと反テロリストには[、]{.chpule2}[
]{.chpuri2}それぞれ 1 種類の服しかないとしましょう[。]{.chpule2}[
]{.chpuri2}以下に示すように[、]{.chpule2}[
]{.chpuri2}服オブジェクトは[、]{.chpule2}[
]{.chpuri2}プレーヤー・オブジェクトに埋め込まれています[。]{.chpule2}[
]{.chpuri2}

プレーヤーの構造体は[、]{.chpule2}[
]{.chpuri2}以下の通りです[。]{.chpule2}[ ]{.chpuri2}服[
]{.chpule1}[（]{.chpuri1}dress[）]{.chpule2}[
]{.chpuri2}オブジェクトは[、]{.chpule2}[
]{.chpuri2}プレーヤーの構造体に埋め込まれています[：]{.chpule2}[
]{.chpuri2}

<figure class="code">
<pre class="code" lang="go"><code>type player struct {
    dress      dress
    playerType string // T（テロリスト） か CT（反テロリスト）
    lat        int
    long       int
}</code></pre>
</figure>

テロリストが 5 人[、]{.chpule2}[ ]{.chpuri2}反テロリストが 5
人[、]{.chpule2}[ ]{.chpuri2}全員でプレーヤーが 10
人いるとします[。]{.chpule2}[ ]{.chpuri2}服に関しては[、]{.chpule2}[
]{.chpuri2}二つの扱い方があります[。]{.chpule2}[ ]{.chpuri2}

1.  10 人のプレーヤーのそれぞれが 1
    個ずつ異なる服オブジェクトを作成し[、]{.chpule2}[
    ]{.chpuri2}それを埋め込みます[。]{.chpule2}[ ]{.chpuri2}全部で 10
    個の服オブジェクトが作成されます[。]{.chpule2}[ ]{.chpuri2}

2.  服オブジェクトを 2 個作成[：]{.chpule2}[ ]{.chpuri2}

    -   テロリストの服オブジェクト一つ[：]{.chpule2}[
        ]{.chpuri2}これを[、]{.chpule2}[ ]{.chpuri2}テロリスト 5
        人で共有します[。]{.chpule2}[ ]{.chpuri2}
    -   反テロリストの服オブジェクト一つ[：]{.chpule2}[
        ]{.chpuri2}これを[、]{.chpule2}[ ]{.chpuri2}反テロリスト 5
        人で共有します[。]{.chpule2}[ ]{.chpuri2}

一つ目の方法では[、]{.chpule2}[ ]{.chpuri2}全部で 10
個の服オブジェクトが作成されるのに対し[、]{.chpule2}[
]{.chpuri2}二つ目の方法では[、]{.chpule2}[ ]{.chpuri2}たった 2
個の服オブジェクトが作成されます[。]{.chpule2}[
]{.chpuri2}二つ目の方法が[、]{.chpule2}[ ]{.chpuri2}Flyweight
デザインパターンです[。]{.chpule2}[ ]{.chpuri2}作成された 2
個の服オブジェクトのことを[、]{.chpule2}[
]{.chpuri2}フライウェイト・オブジェクトと呼びます[。]{.chpule2}[
]{.chpuri2}

Flyweight パターンでは[、]{.chpule2}[
]{.chpuri2}共通の部分を取り出し[、]{.chpule2}[
]{.chpuri2}フライウェイト・オブジェクトを作成します[。]{.chpule2}[
]{.chpuri2}これらのフライウェイト・オブジェクト[
]{.chpule1}[（]{.chpuri1}服[）]{.chpule2}[ ]{.chpuri2}は[、]{.chpule2}[
]{.chpuri2}複数のオブジェクト[
]{.chpule1}[（]{.chpuri1}プレーヤー[）]{.chpule2}[
]{.chpuri2}で共有します[。]{.chpule2}[ ]{.chpuri2}これは[、]{.chpule2}[
]{.chpuri2}ドレス・オブジェクトの数を大幅に削減します[。]{.chpule2}[
]{.chpuri2}この利点は[、]{.chpule2}[
]{.chpuri2}もしもっと多数のプレーヤーを作成したとしても[、]{.chpule2}[
]{.chpuri2}たった 2
個の服オブジェクトだけで十分だということです[。]{.chpule2}[ ]{.chpuri2}

Flyweight パターンでは[、]{.chpule2}[
]{.chpuri2}フライウェイト・オブジェクトは[、]{.chpule2}[
]{.chpuri2}マップ・フィールドに保存します[。]{.chpule2}[
]{.chpuri2}フライウェイト・オブジェクトを共有するオブジェクトが作成されると[、]{.chpule2}[
]{.chpuri2}フライウェイト・オブジェクトはマップから取得します[。]{.chpule2}[
]{.chpuri2}

このお膳立てのどの部分が内因的状態で[、]{.chpule2}[
]{.chpuri2}どれが外因的状態かを見てみましょう[：]{.chpule2}[ ]{.chpuri2}

-   *[内]{.cjk}[因]{.cjk}[的]{.cjk}[状]{.cjk}[態]{.cjk}*[：]{.chpule2}[
    ]{.chpuri2}服は複数のテロリスト・オブジェクトと反テトリスト・オブジェクトで共有されるため[、]{.chpule2}[
    ]{.chpuri2}内因的状態[。]{.chpule2}[ ]{.chpuri2}

-   *[外]{.cjk}[因]{.cjk}[的]{.cjk}[状]{.cjk}[態]{.cjk}*[：]{.chpule2}[
    ]{.chpuri2}プレーヤーの位置と武器は[、]{.chpule2}[
    ]{.chpuri2}オブジェクトごとに異なるので[、]{.chpule2}[
    ]{.chpuri2}外因的状態[。]{.chpule2}[ ]{.chpuri2}

#### []{#example-0--dressFactory-go .anchor} **dressFactory.go:** フライウェイト・ファクトリー

<figure class="code">
<pre class="code" lang="go"><code>package main

import &quot;fmt&quot;

const (
    //TerroristDressType terrorist dress type
    TerroristDressType = &quot;tDress&quot;
    //CounterTerrroristDressType terrorist dress type
    CounterTerrroristDressType = &quot;ctDress&quot;
)

var (
    dressFactorySingleInstance = &amp;DressFactory{
        dressMap: make(map[string]Dress),
    }
)

type DressFactory struct {
    dressMap map[string]Dress
}

func (d *DressFactory) getDressByType(dressType string) (Dress, error) {
    if d.dressMap[dressType] != nil {
        return d.dressMap[dressType], nil
    }

    if dressType == TerroristDressType {
        d.dressMap[dressType] = newTerroristDress()
        return d.dressMap[dressType], nil
    }
    if dressType == CounterTerrroristDressType {
        d.dressMap[dressType] = newCounterTerroristDress()
        return d.dressMap[dressType], nil
    }

    return nil, fmt.Errorf(&quot;Wrong dress type passed&quot;)
}

func getDressFactorySingleInstance() *DressFactory {
    return dressFactorySingleInstance
}</code></pre>
</figure>

#### []{#example-0--dress-go .anchor} **dress.go:** フライウェイト・インターフェース

<figure class="code">
<pre class="code" lang="go"><code>package main

type Dress interface {
    getColor() string
}</code></pre>
</figure>

#### []{#example-0--terroristDress-go .anchor} **terroristDress.go:** 具象フライウェイト・オブジェクト

<figure class="code">
<pre class="code" lang="go"><code>package main

type TerroristDress struct {
    color string
}

func (t *TerroristDress) getColor() string {
    return t.color
}

func newTerroristDress() *TerroristDress {
    return &amp;TerroristDress{color: &quot;red&quot;}
}</code></pre>
</figure>

#### []{#example-0--counterTerroristDress-go .anchor} **counterTerroristDress.go:** 具象フライウェイト・オブジェクト

<figure class="code">
<pre class="code" lang="go"><code>package main

type CounterTerroristDress struct {
    color string
}

func (c *CounterTerroristDress) getColor() string {
    return c.color
}

func newCounterTerroristDress() *CounterTerroristDress {
    return &amp;CounterTerroristDress{color: &quot;green&quot;}
}</code></pre>
</figure>

#### []{#example-0--player-go .anchor} **player.go:** コンテキスト

<figure class="code">
<pre class="code" lang="go"><code>package main

type Player struct {
    dress      Dress
    playerType string
    lat        int
    long       int
}

func newPlayer(playerType, dressType string) *Player {
    dress, _ := getDressFactorySingleInstance().getDressByType(dressType)
    return &amp;Player{
        playerType: playerType,
        dress:      dress,
    }
}

func (p *Player) newLocation(lat, long int) {
    p.lat = lat
    p.long = long
}</code></pre>
</figure>

#### []{#example-0--game-go .anchor} **game.go:** クライアント・コード

<figure class="code">
<pre class="code" lang="go"><code>package main

type game struct {
    terrorists        []*Player
    counterTerrorists []*Player
}

func newGame() *game {
    return &amp;game{
        terrorists:        make([]*Player, 1),
        counterTerrorists: make([]*Player, 1),
    }
}

func (c *game) addTerrorist(dressType string) {
    player := newPlayer(&quot;T&quot;, dressType)
    c.terrorists = append(c.terrorists, player)
    return
}

func (c *game) addCounterTerrorist(dressType string) {
    player := newPlayer(&quot;CT&quot;, dressType)
    c.counterTerrorists = append(c.counterTerrorists, player)
    return
}</code></pre>
</figure>

#### []{#example-0--main-go .anchor} **main.go:** クライアント・コード

<figure class="code">
<pre class="code" lang="go"><code>package main

import &quot;fmt&quot;

func main() {
    game := newGame()

    //Add Terrorist
    game.addTerrorist(TerroristDressType)
    game.addTerrorist(TerroristDressType)
    game.addTerrorist(TerroristDressType)
    game.addTerrorist(TerroristDressType)

    //Add CounterTerrorist
    game.addCounterTerrorist(CounterTerrroristDressType)
    game.addCounterTerrorist(CounterTerrroristDressType)
    game.addCounterTerrorist(CounterTerrroristDressType)

    dressFactoryInstance := getDressFactorySingleInstance()

    for dressType, dress := range dressFactoryInstance.dressMap {
        fmt.Printf(&quot;DressColorType: %s\nDressColor: %s\n&quot;, dressType, dress.getColor())
    }
}</code></pre>
</figure>

#### []{#example-0--output-txt .anchor} **output.txt:** 実行結果

<figure class="code">
<pre class="code" lang="output"><code>DressColorType: ctDress
DressColor: green
DressColorType: tDress
DressColor: red</code></pre>
</figure>

::: next
#### 次を読む

[Proxy を Go で []{.fa
.fa-arrow-right}](../../proxy/go/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### 戻る

[[]{.fa .fa-arrow-left} Facade を Go
で](../../facade/go/example.html){.btn .btn-default rel="prev"}
:::

## 他言語での **Flyweight**

[![Flyweight を C#
で](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Flyweight を C# で"){.prog-lang-link}
[![Flyweight を C++
で](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Flyweight を C++ で"){.prog-lang-link}
[![Flyweight を Java
で](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Flyweight を Java で"){.prog-lang-link}
[![Flyweight を PHP
で](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Flyweight を PHP で"){.prog-lang-link}
[![Flyweight を Python
で](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Flyweight を Python で"){.prog-lang-link}
[![Flyweight を Ruby
で](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Flyweight を Ruby で"){.prog-lang-link}
[![Flyweight を Rust
で](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Flyweight を Rust で"){.prog-lang-link}
[![Flyweight を Swift
で](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Flyweight を Swift で"){.prog-lang-link}
[![Flyweight を TypeScript
で](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Flyweight を TypeScript で"){.prog-lang-link}
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
