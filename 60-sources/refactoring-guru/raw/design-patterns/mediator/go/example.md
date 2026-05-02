::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../index.html){.home} / [Design
Patterns](../../../design-patterns.html){.type} /
[Mediator](../../mediator.html){.type} / [Go](../../go.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Mediator](../../../images/patterns/cards/mediator-mini1561.png?id=a7e43ee8e17e4474737b1fcb3201d7ba){srcset="/images/patterns/cards/mediator-mini-2x.png?id=d288d7c73f5581ae09701d61200ae09e 2x"}
:::

# **Mediator** in Go {#mediator-in-go .pattern-example-title-block-title}
::::

:::::::::::::::: pattern-example-body
::: pattern-example-brief
**Mediator** is a behavioral design pattern that reduces coupling
between components of a program by making them communicate indirectly,
through a special mediator object.

The Mediator makes it easy to modify, extend and reuse individual
components because they're no longer dependent on the dozens of other
classes.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Learn more about Mediator ](../../mediator.html){.btn .btn-lg
.btn-primary}
:::

::::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
Navigation
:::

::: en-intro
 [Intro](#)
:::

::: en-intro
 [Conceptual Example](#example-0)
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

## Conceptual Example {#example-0-title}

An excellent example of the Mediator pattern is a railway station
traffic system. Two trains never communicate between themselves for the
availability of the platform. The `stationManager` acts as a mediator
and makes the platform available to only one of the arriving trains
while keeping the rest in a queue. A departing train notifies the
stations, which lets the next train in the queue to arrive.

#### []{#example-0--train-go .anchor} **train.go:** Component

<figure class="code">
<pre class="code" lang="go"><code>package main

type Train interface {
    arrive()
    depart()
    permitArrival()
}</code></pre>
</figure>

#### []{#example-0--passengerTrain-go .anchor} **passengerTrain.go:** Concrete component

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

#### []{#example-0--freightTrain-go .anchor} **freightTrain.go:** Concrete component

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

#### []{#example-0--mediator-go .anchor} **mediator.go:** Mediator interface

<figure class="code">
<pre class="code" lang="go"><code>package main

type Mediator interface {
    canArrive(Train) bool
    notifyAboutDeparture()
}</code></pre>
</figure>

#### []{#example-0--stationManager-go .anchor} **stationManager.go:** Concrete mediator

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

#### []{#example-0--main-go .anchor} **main.go:** Client code

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

#### []{#example-0--output-txt .anchor} **output.txt:** Execution result

<figure class="code">
<pre class="code" lang="output"><code>PassengerTrain: Arrived
FreightTrain: Arrival blocked, waiting
PassengerTrain: Leaving
FreightTrain: Arrival permitted
FreightTrain: Arrived</code></pre>
</figure>

::: next
#### Read next

[Memento in Go []{.fa
.fa-arrow-right}](../../memento/go/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Return

[[]{.fa .fa-arrow-left} Iterator in
Go](../../iterator/go/example.html){.btn .btn-default rel="prev"}
:::

## **Mediator** in Other Languages

[![Mediator in
C#](../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Mediator in C#"){.prog-lang-link}
[![Mediator in
C++](../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Mediator in C++"){.prog-lang-link}
[![Mediator in
Java](../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Mediator in Java"){.prog-lang-link}
[![Mediator in
PHP](../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Mediator in PHP"){.prog-lang-link}
[![Mediator in
Python](../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Mediator in Python"){.prog-lang-link}
[![Mediator in
Ruby](../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Mediator in Ruby"){.prog-lang-link}
[![Mediator in
Rust](../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Mediator in Rust"){.prog-lang-link}
[![Mediator in
Swift](../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Mediator in Swift"){.prog-lang-link}
[![Mediator in
TypeScript](../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Mediator in TypeScript"){.prog-lang-link}
::::::::::::::::::::::

::::::: {.prom .banner-sidebar .banner .banner-striped .banner-examples .banner-removable .banner-removable-patterns data-id="DP: Examples-IDE" creative-id="examples-sidebar-en" position="sidebar"}
:::::: {.banner-inner style="font-size: 14px; line-height: 21px;"}
::: banner-examples-img
[![](../../../images/patterns/banners/examples-ide91ca.png?id=3115b4b548fb96b75974e2de8f4f49bc){width="215"
height="158" loading="lazy"
srcset="/images/patterns/banners/examples-ide-2x.png?id=93c007a6d157b97c28eb213f59d716ae 2x"}](../../book.html)
:::

::: banner-examples-fade
[ Archive with examples](../../book.html){.btn .btn-secondary
.btn-block}
:::

::: {.banner-examples-text style="margin-top: 1rem;"}
Buy the eBook **Dive Into Design Patterns** and get the access to
archive with dozens of detailed examples that can be opened right in
your IDE.

[ Learn more...](../../book.html){.btn .btn-secondary .btn-block}
:::
::::::
:::::::
::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::
