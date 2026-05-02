:::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [디자인
패턴들](../../../design-patterns.html){.type} /
[플라이웨이트](../../flyweight.html){.type} / [Go](../../go.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![플라이웨이트](../../../../images/patterns/cards/flyweight-minie3d8.png?id=422ca8d2f90614dce810a8812c626698){srcset="/images/patterns/cards/flyweight-mini-2x.png?id=857ca676fff84317bb0dab76abfce08e 2x"}
:::

# Go로 작성된 **플라이웨이트** {#go로-작성된-플라이웨이트 .pattern-example-title-block-title}
::::

::::::::::::::::: pattern-example-body
::: pattern-example-brief
**플라이웨이트**는 구조 패턴이며 프로그램들이 객체들의 메모리 소비를
낮게 유지하여 방대한 양의 객체들을 지원할 수 있도록 합니다.

이 패턴은 여러 객체 사이의 객체 상태를 공유하여 위를 달성합니다. 다르게
설명하자면 플라이웨이트는 다른 객체들이 공통으로 사용하는 데이터를
캐싱하여 RAM을 절약합니다.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ 플라이웨이트에 대하여 더 자세히 알아보세요
](../../flyweight.html){.btn .btn-lg .btn-primary}
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

## 개념적인 예시 {#example-0-title}

카운터-스트라이크 게임에서 테러리스트와 대테러 요원은 각각 여러 유형의
복장​(dress)​을 가지고 있으나 예시의 단순화를 위해 테러리스트와 대테러
요원 모두 각각 하나의 드레스 유형을 가지고 있다고 가정해 보겠습니다.
드레스 객체는 아래와 같이 플레이어 객체에 내장되어 있습니다.

아래는 플레이어를 위한 구조체​(struct)​이며 드레스 객체가 플레이어
구조체에 포함되어 있음을 알 수 있습니다:

<figure class="code">
<pre class="code" lang="go"><code>type player struct {
    dress      dress
    playerType string // Can be T or CT
    lat        int
    long       int
}</code></pre>
</figure>

5명의 테러리스트와 5명의 대테러 요원들, 그러니까 총 10명의 플레이어가
있다고 가정해 봅시다. 이제 복장과 관련하여 두 가지 옵션이 있습니다.

1.  10명의 플레이어 객체들은 각각 다른 복장 객체를 만든 후 포함시킵니다.
    총 10개의 복장 객체들이 생성됩니다.

2.  우리는 두 개의 복장 객체를 만듭니다:

    -   단일 테러리스트 복장 객체: 5명의 테러리스트가 공유합니다.
    -   단일 대테러 요원 복장 객체: 5명의 대테러 요원들이 공유합니다.

첫 번째 접근법에서는 총 10개의 복장 객체들이 생성되나 두 번째
접근법에서는 두 개의 복장 객체만 생성됩니다. 플라이웨이트 패턴은 두 번째
접근법을 따릅니다. 우리가 만든 두 개의 복장 객체들을 플라이웨이트
객체라고 합니다.

플라이웨이트 패턴은 공통부분들을 제거하고 플라이웨이트 객체들을
생성합니다. 플라이웨이트 객체들​(복장)​은 이제 여러 객체​(플레이어) 간에
공유될 수 있습니다. 이렇게 하면 복장 객체들의 수가 크게 줄어들며,
플레이어들을 더 많이 생성하더라도 두 개의 복장 객체로 충분하다는
점입니다.

플라이웨이트 패턴에서 우리는 플라이웨이트 객체들을 맵 필드에 저장합니다.
플라이웨이트 객체들을 공유하는 다른 객체들이 생성될 때마다 플라이웨이트
객체들은 맵에서 가져와집니다.

이 구조의 어떤 부분이 고유한 상태와 공유한 상태가 되는지 봅시다.

-   *[고]{.cjk}[유]{.cjk}[한]{.cjk} [상]{.cjk}[태]{.cjk}*: 여러
    테러리스트 및 대테러 요원 객체에서 공유할 수 있으므로 복장은 고유한
    상태입니다.

-   *[공]{.cjk}[유]{.cjk}[한]{.cjk} [상]{.cjk}[태]{.cjk}*: 플레이어
    위치와 플레이어 무기는 객체마다 다르므로 공유한 상태입니다.

#### []{#example-0--dressFactory-go .anchor} **dressFactory.go:** 플라이웨이트 팩토리

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

#### []{#example-0--dress-go .anchor} **dress.go:** 플라이웨이트 인터페이스

<figure class="code">
<pre class="code" lang="go"><code>package main

type Dress interface {
    getColor() string
}</code></pre>
</figure>

#### []{#example-0--terroristDress-go .anchor} **terroristDress.go:** 구상 플라이웨이트 객체

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

#### []{#example-0--counterTerroristDress-go .anchor} **counterTerroristDress.go:** 구상 플라이웨이트 객체

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

#### []{#example-0--player-go .anchor} **player.go:** 콘텍스트

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

#### []{#example-0--game-go .anchor} **game.go:** 클라이언트 코드

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

#### []{#example-0--main-go .anchor} **main.go:** 클라이언트 코드

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

#### []{#example-0--output-txt .anchor} **output.txt:** 실행 결과

<figure class="code">
<pre class="code" lang="output"><code>DressColorType: ctDress
DressColor: green
DressColorType: tDress
DressColor: red</code></pre>
</figure>

::: next
#### 다음을 보세요

[Go로 작성된 프록시 []{.fa
.fa-arrow-right}](../../proxy/go/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### 돌아가기

[[]{.fa .fa-arrow-left} Go로 작성된
퍼사드](../../facade/go/example.html){.btn .btn-default rel="prev"}
:::

## 다른 언어로 작성된 **플라이웨이트**

[![C#으로 작성된
플라이웨이트](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "C#으로 작성된 플라이웨이트"){.prog-lang-link}
[![C++로 작성된
플라이웨이트](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "C++로 작성된 플라이웨이트"){.prog-lang-link}
[![자바로 작성된
플라이웨이트](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "자바로 작성된 플라이웨이트"){.prog-lang-link}
[![PHP로 작성된
플라이웨이트](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "PHP로 작성된 플라이웨이트"){.prog-lang-link}
[![파이썬으로 작성된
플라이웨이트](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "파이썬으로 작성된 플라이웨이트"){.prog-lang-link}
[![루비로 작성된
플라이웨이트](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "루비로 작성된 플라이웨이트"){.prog-lang-link}
[![러스트로 작성된
플라이웨이트](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "러스트로 작성된 플라이웨이트"){.prog-lang-link}
[![스위프트로 작성된
플라이웨이트](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "스위프트로 작성된 플라이웨이트"){.prog-lang-link}
[![타입스크립트로 작성된
플라이웨이트](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "타입스크립트로 작성된 플라이웨이트"){.prog-lang-link}
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
