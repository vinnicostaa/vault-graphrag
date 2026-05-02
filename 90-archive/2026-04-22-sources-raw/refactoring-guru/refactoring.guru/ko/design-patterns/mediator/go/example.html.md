::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [디자인
패턴들](../../../design-patterns.html){.type} /
[중재자](../../mediator.html){.type} / [Go](../../go.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![중재자](../../../../images/patterns/cards/mediator-mini1561.png?id=a7e43ee8e17e4474737b1fcb3201d7ba){srcset="/images/patterns/cards/mediator-mini-2x.png?id=d288d7c73f5581ae09701d61200ae09e 2x"}
:::

# Go로 작성된 **중재자** {#go로-작성된-중재자 .pattern-example-title-block-title}
::::

:::::::::::::::: pattern-example-body
::: pattern-example-brief
**중재자**는 행동 디자인 패턴이며 프로그램의 컴포넌트들이 특수 중재자
객체를 통하여 간접적으로 소통하게 함으로써 해당 컴포넌트 간의 결합도를
줄입니다.

중재자는 개별 컴포넌트들을 편집, 확장 및 재사용하는 것을 쉽게 만드는데,
그 이유는 이들이 더 이상 수십 개의 다른 클래스들에 의존하지 않기
때문입니다.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ 중재자에 대하여 더 자세히 알아보세요 ](../../mediator.html){.btn
.btn-lg .btn-primary}
:::

::::::::::::: {.sidebar-navigation .with-scroll}
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
 [train](#example-0--train-go)
:::

::: en-file
 [passenger­Train](#example-0--passengerTrain-go)
:::

::: en-file
 [freight­Train](#example-0--freightTrain-go)
:::

::: en-file
 [mediator](#example-0--mediator-go)
:::

::: en-file
 [station­Manager](#example-0--stationManager-go)
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

## 개념적인 예시 {#example-0-title}

중재자 패턴의 좋은 예시는 기차역 교통 시스템입니다. 두 열차는 플랫폼의
가용성을 위해 서로 통신하지 않습니다. `station­Manager`​(기차역 관리자)​가
중재자 역할을 하며, 그는 나머지 열차는 대기열에 유지하면서 도착하는 열차
중 하나만 플랫폼을 사용할 수 있도록 합니다. 출발하는 열차가 역에 출발
사실을 알리면 대기열에 있는 다음 열차가 도착할 수 있습니다.

#### []{#example-0--train-go .anchor} **train.go:** 컴포넌트

<figure class="code">
<pre class="code" lang="go"><code>package main

type Train interface {
    arrive()
    depart()
    permitArrival()
}</code></pre>
</figure>

#### []{#example-0--passengerTrain-go .anchor} **passengerTrain.go:** 구상 컴포넌트

<figure class="code">
<pre class="code" lang="go"><code>package main

import &quot;fmt&quot;

type PassengerTrain struct {
    mediator Mediator
}

func (g *PassengerTrain) arrive() {
    if !g.mediator.canArrive(g) {
        fmt.Println(&quot;PassengerTrain: Arrival blocked, waiting&quot;)
        return
    }
    fmt.Println(&quot;PassengerTrain: Arrived&quot;)
}

func (g *PassengerTrain) depart() {
    fmt.Println(&quot;PassengerTrain: Leaving&quot;)
    g.mediator.notifyAboutDeparture()
}

func (g *PassengerTrain) permitArrival() {
    fmt.Println(&quot;PassengerTrain: Arrival permitted, arriving&quot;)
    g.arrive()
}</code></pre>
</figure>

#### []{#example-0--freightTrain-go .anchor} **freightTrain.go:** 구상 컴포넌트

<figure class="code">
<pre class="code" lang="go"><code>package main

import &quot;fmt&quot;

type FreightTrain struct {
    mediator Mediator
}

func (g *FreightTrain) arrive() {
    if !g.mediator.canArrive(g) {
        fmt.Println(&quot;FreightTrain: Arrival blocked, waiting&quot;)
        return
    }
    fmt.Println(&quot;FreightTrain: Arrived&quot;)
}

func (g *FreightTrain) depart() {
    fmt.Println(&quot;FreightTrain: Leaving&quot;)
    g.mediator.notifyAboutDeparture()
}

func (g *FreightTrain) permitArrival() {
    fmt.Println(&quot;FreightTrain: Arrival permitted&quot;)
    g.arrive()
}</code></pre>
</figure>

#### []{#example-0--mediator-go .anchor} **mediator.go:** 중재자 인터페이스

<figure class="code">
<pre class="code" lang="go"><code>package main

type Mediator interface {
    canArrive(Train) bool
    notifyAboutDeparture()
}</code></pre>
</figure>

#### []{#example-0--stationManager-go .anchor} **stationManager.go:** 구상 중재자

<figure class="code">
<pre class="code" lang="go"><code>package main

type StationManager struct {
    isPlatformFree bool
    trainQueue     []Train
}

func newStationManger() *StationManager {
    return &amp;StationManager{
        isPlatformFree: true,
    }
}

func (s *StationManager) canArrive(t Train) bool {
    if s.isPlatformFree {
        s.isPlatformFree = false
        return true
    }
    s.trainQueue = append(s.trainQueue, t)
    return false
}

func (s *StationManager) notifyAboutDeparture() {
    if !s.isPlatformFree {
        s.isPlatformFree = true
    }
    if len(s.trainQueue) &gt; 0 {
        firstTrainInQueue := s.trainQueue[0]
        s.trainQueue = s.trainQueue[1:]
        firstTrainInQueue.permitArrival()
    }
}</code></pre>
</figure>

#### []{#example-0--main-go .anchor} **main.go:** 클라이언트 코드

<figure class="code">
<pre class="code" lang="go"><code>package main

func main() {
    stationManager := newStationManger()

    passengerTrain := &amp;PassengerTrain{
        mediator: stationManager,
    }
    freightTrain := &amp;FreightTrain{
        mediator: stationManager,
    }

    passengerTrain.arrive()
    freightTrain.arrive()
    passengerTrain.depart()
}</code></pre>
</figure>

#### []{#example-0--output-txt .anchor} **output.txt:** 실행 결과

<figure class="code">
<pre class="code" lang="output"><code>PassengerTrain: Arrived
FreightTrain: Arrival blocked, waiting
PassengerTrain: Leaving
FreightTrain: Arrival permitted
FreightTrain: Arrived</code></pre>
</figure>

::: next
#### 다음을 보세요

[Go로 작성된 메멘토 []{.fa
.fa-arrow-right}](../../memento/go/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### 돌아가기

[[]{.fa .fa-arrow-left} Go로 작성된
반복자](../../iterator/go/example.html){.btn .btn-default rel="prev"}
:::

## 다른 언어로 작성된 **중재자**

[![C#으로 작성된
중재자](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "C#으로 작성된 중재자"){.prog-lang-link}
[![C++로 작성된
중재자](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "C++로 작성된 중재자"){.prog-lang-link}
[![자바로 작성된
중재자](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "자바로 작성된 중재자"){.prog-lang-link}
[![PHP로 작성된
중재자](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "PHP로 작성된 중재자"){.prog-lang-link}
[![파이썬으로 작성된
중재자](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "파이썬으로 작성된 중재자"){.prog-lang-link}
[![루비로 작성된
중재자](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "루비로 작성된 중재자"){.prog-lang-link}
[![러스트로 작성된
중재자](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "러스트로 작성된 중재자"){.prog-lang-link}
[![스위프트로 작성된
중재자](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "스위프트로 작성된 중재자"){.prog-lang-link}
[![타입스크립트로 작성된
중재자](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "타입스크립트로 작성된 중재자"){.prog-lang-link}
::::::::::::::::::::::

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
::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::
