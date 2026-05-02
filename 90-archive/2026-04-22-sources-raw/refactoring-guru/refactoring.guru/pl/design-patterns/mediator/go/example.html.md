::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Wzorce
projektowe](../../../design-patterns.html){.type} /
[Mediator](../../mediator.html){.type} / [Go](../../go.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Mediator](../../../../images/patterns/cards/mediator-mini1561.png?id=a7e43ee8e17e4474737b1fcb3201d7ba){srcset="/images/patterns/cards/mediator-mini-2x.png?id=d288d7c73f5581ae09701d61200ae09e 2x"}
:::

# **Mediator** w języku Go {#mediator-w-języku-go .pattern-example-title-block-title}
::::

:::::::::::::::: pattern-example-body
::: pattern-example-brief
**Mediator** to behawioralny wzorzec projektowy pozwalający zredukować
sprzężenie pomiędzy komponentami programu poprzez zmuszenie ich do
komunikacji za pośrednictwem obiektu zwanego mediatorem.

Mediator ułatwia modyfikację, rozszerzanie i ponowne wykorzystanie
komponentów gdyż z jego pomocą nie są one zależne od wielu innych klas.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Dowiedz się więcej o Mediator ](../../mediator.html){.btn .btn-lg
.btn-primary}
:::

::::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
Nawigacja
:::

::: en-intro
 [Intro](#)
:::

::: en-intro
 [Przykład koncepcyjny](#example-0)
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

## Przykład koncepcyjny {#example-0-title}

Świetnym przykładem użycia wzorca Mediator jest stacja kierowania ruchem
kolejowym. Dwa pociągi nie muszą się ze sobą bezpośrednio komunikować
odnośnie zajętości peronu. `stationManager` pełni rolę mediatora
pomiędzy pociągami i pilnuje, aby tylko jeden z nich mógł wjechać na
peron, zaś drugi czekał. Odjeżdżając z peronu pociąg informuje o tym
fakcie stację, która z kolei udziela pozwolenia na wjazd oczekującemu
pociągowi.

#### []{#example-0--train-go .anchor} **train.go:** Komponent

<figure class="code">
<pre class="code" lang="go"><code>package main

type Train interface {
    arrive()
    depart()
    permitArrival()
}</code></pre>
</figure>

#### []{#example-0--passengerTrain-go .anchor} **passengerTrain.go:** Konkretny komponent

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

#### []{#example-0--freightTrain-go .anchor} **freightTrain.go:** Konkretny komponent

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

#### []{#example-0--mediator-go .anchor} **mediator.go:** Interfejs mediatora

<figure class="code">
<pre class="code" lang="go"><code>package main

type Mediator interface {
    canArrive(Train) bool
    notifyAboutDeparture()
}</code></pre>
</figure>

#### []{#example-0--stationManager-go .anchor} **stationManager.go:** Konkretny mediator

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

#### []{#example-0--main-go .anchor} **main.go:** Kod klienta

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

#### []{#example-0--output-txt .anchor} **output.txt:** Wynik działania

<figure class="code">
<pre class="code" lang="output"><code>PassengerTrain: Arrived
FreightTrain: Arrival blocked, waiting
PassengerTrain: Leaving
FreightTrain: Arrival permitted
FreightTrain: Arrived</code></pre>
</figure>

::: next
#### Czytaj dalej

[Pamiątka w języku Go []{.fa
.fa-arrow-right}](../../memento/go/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Wróć

[[]{.fa .fa-arrow-left} Iterator w języku
Go](../../iterator/go/example.html){.btn .btn-default rel="prev"}
:::

## **Mediator** w innych językach

[![Mediator w języku
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Mediator w języku C#"){.prog-lang-link}
[![Mediator w języku
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Mediator w języku C++"){.prog-lang-link}
[![Mediator w języku
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Mediator w języku Java"){.prog-lang-link}
[![Mediator w języku
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Mediator w języku PHP"){.prog-lang-link}
[![Mediator w języku
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Mediator w języku Python"){.prog-lang-link}
[![Mediator w języku
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Mediator w języku Ruby"){.prog-lang-link}
[![Mediator w języku
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Mediator w języku Rust"){.prog-lang-link}
[![Mediator w języku
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Mediator w języku Swift"){.prog-lang-link}
[![Mediator w języku
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Mediator w języku TypeScript"){.prog-lang-link}
::::::::::::::::::::::

::::::: {.prom .banner-sidebar .banner .banner-striped .banner-examples .banner-removable .banner-removable-patterns data-id="DP: Examples-IDE" creative-id="examples-sidebar-pl" position="sidebar"}
:::::: {.banner-inner style="font-size: 14px; line-height: 21px;"}
::: banner-examples-img
[![](../../../../images/patterns/banners/examples-ide91ca.png?id=3115b4b548fb96b75974e2de8f4f49bc){width="215"
height="158" loading="lazy"
srcset="/images/patterns/banners/examples-ide-2x.png?id=93c007a6d157b97c28eb213f59d716ae 2x"}](../../book.html)
:::

::: banner-examples-fade
[ Archiwum\
przykładów kodu](../../book.html){.btn .btn-secondary .btn-block}
:::

::: {.banner-examples-text style="margin-top: 1rem;"}
Kupując eBook **Wzorce projektowe. Nowoczesny podręcznik** zyskujesz
dostęp do archiwum zawierającego mnóstwo szczegółowych przykładów, które
można od razu wkleić do swojego preferowanego zintegrowanego środowiska
programistycznego.

[ Dowiedz się więcej...](../../book.html){.btn .btn-secondary
.btn-block}
:::
::::::
:::::::
::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::
