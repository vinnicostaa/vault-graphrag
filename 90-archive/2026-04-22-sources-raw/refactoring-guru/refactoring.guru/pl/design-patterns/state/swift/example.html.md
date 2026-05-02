::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Wzorce
projektowe](../../../design-patterns.html){.type} /
[Stan](../../state.html){.type} / [Swift](../../swift.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Stan](../../../../images/patterns/cards/state-mini63fb.png?id=f4018837e0641d1dade756b6678fd4ee){srcset="/images/patterns/cards/state-mini-2x.png?id=7e24398b27a43c7bd286fc0ea54d2a35 2x"}
:::

# **Stan** w języku Swift {#stan-w-języku-swift .pattern-example-title-block-title}
::::

:::::::::::::: pattern-example-body
::: pattern-example-brief
**Stan** to behawioralny wzorzec projektowy pozwalający zmieniać
zachowanie obiektu w odpowiedzi na zmianę jego wewnętrznego stanu.

Wzorzec Stan zakłada ekstrakcję zachowań odnoszących się do stanu
obiektu do osobnych klas odpowiadających jego poszczególnym stanom.
Pierwotny obiekt wówczas deleguje pracę instancjom tych klas, zamiast
wykonywać ją samodzielnie.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Dowiedz się więcej o Stan ](../../state.html){.btn .btn-lg
.btn-primary}
:::

::::::::::: {.sidebar-navigation .with-scroll}
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
 [Example](#example-0--Example-swift)
:::

::: en-file
 [Output](#example-0--Output-txt)
:::

::: en-intro
 [Przykład z prawdziwego życia](#example-1)
:::

::: en-file
 [Example](#example-1--Example-swift)
:::

::: en-file
 [Output](#example-1--Output-txt)
:::
:::::::::::

**Złożoność:** []{.fa-stars}

**Popularność:** []{.fa-stars}

**Przykłady użycia:** Wzorzec Stan jest powszechnie stosowany w kodzie
Swift w celu konwersji obszernych maszyn stanów opartych na instrukcji
`switch` w obiekty.

**Identyfikacja:** Wzorzec Stan można poznać po obecności metod
zmieniających swoje zachowanie zależnie od stanu obiektu, sterowanego z
zewnątrz.
::::::::::::::

::::: {style="padding: 1rem; border: solid 4px #e35e5e; border-radius: 1rem; text-align: center"}
::: {style="margin-bottom: 0.5rem;"}
Następujące przykłady są dostępne na [Swift
Playgrounds](https://www.alemohamad.com/playgrounds){style="text-decoration: underline; display: inline-block"
rel="nofollow"}.
:::

::: {style="font-size: 13px;"}
Gratulacje dla [Alejandro
Mohamad](https://www.alemohamad.com/){style="text-decoration: underline;"
rel="nofollow"} za stworzenie wersji Playground.
:::
:::::

::: {#nav-tab .nav .nav-tabs .nav-justified role="tablist"}
[Przykład koncepcyjny](#example-0){#example-0-tab .nav-item .nav-link
.active toggle="tab" role="tab" aria-controls="example-0"
aria-selected="true"} [Przykład z prawdziwego
życia](#example-1){#example-1-tab .nav-item .nav-link toggle="tab"
role="tab" aria-controls="example-1" aria-selected="true"}
:::

::::: {#nav-tabContent .pattern-example-tab-content .tab-content}
::: {#example-0 .tab-pane .show .active role="tabpanel" aria-labelledby="example-0-tab" style="padding-top: 24px"}
## Przykład koncepcyjny {#example-0-title}

Poniższy przykład ilustruje strukturę wzorca **Stan** ze szczególnym
naciskiem na następujące kwestie:

-   Z jakich składa się klas?
-   Jakie role pełnią te klasy?
-   W jaki sposób elementy wzorca są ze sobą powiązane?

Poznawszy strukturę wzorca będzie ci łatwiej zrozumieć następujący
przykład, oparty na prawdziwym przypadku użycia Swift.

#### []{#example-0--Example-swift .anchor} **Example.swift:** Przykład koncepcyjny

<figure class="code">
<pre class="code" lang="swift"><code>import XCTest

/// The Context defines the interface of interest to clients. It also maintains
/// a reference to an instance of a State subclass, which represents the current
/// state of the Context.
class Context {

    /// A reference to the current state of the Context.
    private var state: State

    init(_ state: State) {
        self.state = state
        transitionTo(state: state)
    }

    /// The Context allows changing the State object at runtime.
    func transitionTo(state: State) {
        print(&quot;Context: Transition to &quot; + String(describing: state))
        self.state = state
        self.state.update(context: self)
    }

    /// The Context delegates part of its behavior to the current State object.
    func request1() {
        state.handle1()
    }

    func request2() {
        state.handle2()
    }
}

/// The base State class declares methods that all Concrete State should
/// implement and also provides a backreference to the Context object,
/// associated with the State. This backreference can be used by States to
/// transition the Context to another State.
protocol State: AnyObject {

    func update(context: Context)

    func handle1()
    func handle2()
}

class BaseState: State {

    private(set) weak var context: Context?

    func update(context: Context) {
        self.context = context
    }

    func handle1() {}
    func handle2() {}
}

/// Concrete States implement various behaviors, associated with a state of the
/// Context.
class ConcreteStateA: BaseState {

    override func handle1() {
        print(&quot;ConcreteStateA handles request1.&quot;)
        print(&quot;ConcreteStateA wants to change the state of the context.\n&quot;)
        context?.transitionTo(state: ConcreteStateB())
    }

    override func handle2() {
        print(&quot;ConcreteStateA handles request2.\n&quot;)
    }
}

class ConcreteStateB: BaseState {

    override func handle1() {
        print(&quot;ConcreteStateB handles request1.\n&quot;)
    }

    override func handle2() {
        print(&quot;ConcreteStateB handles request2.&quot;)
        print(&quot;ConcreteStateB wants to change the state of the context.\n&quot;)
        context?.transitionTo(state: ConcreteStateA())
    }
}

/// Let&#39;s see how it all works together.
class StateConceptual: XCTestCase {

    func test() {
        let context = Context(ConcreteStateA())
        context.request1()
        context.request2()
    }
}</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** Wynik działania

<figure class="code">
<pre class="code" lang="output"><code>Context: Transition to StateConceptual.ConcreteStateA
ConcreteStateA handles request1.
ConcreteStateA wants to change the state of the context.

Context: Transition to StateConceptual.ConcreteStateB
ConcreteStateB handles request2.
ConcreteStateB wants to change the state of the context.

Context: Transition to StateConceptual.ConcreteStateA</code></pre>
</figure>
:::

::: {#example-1 .tab-pane role="tabpanel" aria-labelledby="example-1-tab" style="padding-top: 24px"}
## Przykład z prawdziwego życia {#example-1-title}

#### []{#example-1--Example-swift .anchor} **Example.swift:** Przykład z prawdziwego życia

<figure class="code">
<pre class="code" lang="swift"><code>import XCTest

class StateRealWorld: XCTestCase {

    func test() {

        print(&quot;Client: I&#39;m starting working with a location tracker&quot;)
        let tracker = LocationTracker()

        print()
        tracker.startTracking()

        print()
        tracker.pauseTracking(for: 2)

        print()
        tracker.makeCheckIn()

        print()
        tracker.findMyChildren()

        print()
        tracker.stopTracking()
    }
}

class LocationTracker {

    /// Location tracking is enabled by default
    private lazy var trackingState: TrackingState = EnabledTrackingState(tracker: self)

    func startTracking() {
        trackingState.startTracking()
    }

    func stopTracking() {
        trackingState.stopTracking()
    }

    func pauseTracking(for time: TimeInterval) {
        trackingState.pauseTracking(for: time)
    }

    func makeCheckIn() {
        trackingState.makeCheckIn()
    }

    func findMyChildren() {
        trackingState.findMyChildren()
    }

    func update(state: TrackingState) {
        trackingState = state
    }
}

protocol TrackingState {

    func startTracking()
    func stopTracking()
    func pauseTracking(for time: TimeInterval)

    func makeCheckIn()
    func findMyChildren()
}

class EnabledTrackingState: TrackingState {

    private weak var tracker: LocationTracker?

    init(tracker: LocationTracker?) {
        self.tracker = tracker
    }

    func startTracking() {
        print(&quot;EnabledTrackingState: startTracking is invoked&quot;)
        print(&quot;EnabledTrackingState: tracking location....1&quot;)
        print(&quot;EnabledTrackingState: tracking location....2&quot;)
        print(&quot;EnabledTrackingState: tracking location....3&quot;)
    }

    func stopTracking() {
        print(&quot;EnabledTrackingState: Received &#39;stop tracking&#39;&quot;)
        print(&quot;EnabledTrackingState: Changing state to &#39;disabled&#39;...&quot;)
        tracker?.update(state: DisabledTrackingState(tracker: tracker))
        tracker?.stopTracking()
    }

    func pauseTracking(for time: TimeInterval) {
        print(&quot;EnabledTrackingState: Received &#39;pause tracking&#39; for \(time) seconds&quot;)
        print(&quot;EnabledTrackingState: Changing state to &#39;disabled&#39;...&quot;)
        tracker?.update(state: DisabledTrackingState(tracker: tracker))
        tracker?.pauseTracking(for: time)
    }

    func makeCheckIn() {
        print(&quot;EnabledTrackingState: performing check-in at the current location&quot;)
    }

    func findMyChildren() {
        print(&quot;EnabledTrackingState: searching for children...&quot;)
    }
}

class DisabledTrackingState: TrackingState {

    private weak var tracker: LocationTracker?

    init(tracker: LocationTracker?) {
        self.tracker = tracker
    }

    func startTracking() {
        print(&quot;DisabledTrackingState: Received &#39;start tracking&#39;&quot;)
        print(&quot;DisabledTrackingState: Changing state to &#39;enabled&#39;...&quot;)
        tracker?.update(state: EnabledTrackingState(tracker: tracker))
    }

    func pauseTracking(for time: TimeInterval) {
        print(&quot;DisabledTrackingState: Pause tracking for \(time) seconds&quot;)

        for i in 0...Int(time) {
            print(&quot;DisabledTrackingState: pause...\(i)&quot;)
        }

        print(&quot;DisabledTrackingState: Time is over&quot;)
        print(&quot;DisabledTrackingState: Returing to &#39;enabled state&#39;...\n&quot;)
        self.tracker?.update(state: EnabledTrackingState(tracker: self.tracker))
        self.tracker?.startTracking()
    }

    func stopTracking() {
        print(&quot;DisabledTrackingState: Received &#39;stop tracking&#39;&quot;)
        print(&quot;DisabledTrackingState: Do nothing...&quot;)
    }

    func makeCheckIn() {
        print(&quot;DisabledTrackingState: Received &#39;make check-in&#39;&quot;)
        print(&quot;DisabledTrackingState: Changing state to &#39;enabled&#39;...&quot;)
        tracker?.update(state: EnabledTrackingState(tracker: tracker))
        tracker?.makeCheckIn()
    }

    func findMyChildren() {
        print(&quot;DisabledTrackingState: Received &#39;find my children&#39;&quot;)
        print(&quot;DisabledTrackingState: Changing state to &#39;enabled&#39;...&quot;)
        tracker?.update(state: EnabledTrackingState(tracker: tracker))
        tracker?.findMyChildren()
    }
}</code></pre>
</figure>

#### []{#example-1--Output-txt .anchor} **Output.txt:** Wynik działania

<figure class="code">
<pre class="code" lang="output"><code>Client: I&#39;m starting working with a location tracker

EnabledTrackingState: startTracking is invoked
EnabledTrackingState: tracking location....1
EnabledTrackingState: tracking location....2
EnabledTrackingState: tracking location....3

EnabledTrackingState: Received &#39;pause tracking&#39; for 2.0 seconds
EnabledTrackingState: Changing state to &#39;disabled&#39;...
DisabledTrackingState: Pause tracking for 2.0 seconds
DisabledTrackingState: pause...0
DisabledTrackingState: pause...1
DisabledTrackingState: pause...2
DisabledTrackingState: Time is over
DisabledTrackingState: Returing to &#39;enabled state&#39;...

EnabledTrackingState: startTracking is invoked
EnabledTrackingState: tracking location....1
EnabledTrackingState: tracking location....2
EnabledTrackingState: tracking location....3

EnabledTrackingState: performing check-in at the current location

EnabledTrackingState: searching for children...

EnabledTrackingState: Received &#39;stop tracking&#39;
EnabledTrackingState: Changing state to &#39;disabled&#39;...
DisabledTrackingState: Received &#39;stop tracking&#39;
DisabledTrackingState: Do nothing...</code></pre>
</figure>
:::
:::::

::: {#nav-tab .nav .nav-tabs .nav-tabs-bottom .nav-justified role="tablist"}
[Przykład koncepcyjny](#example-0){#example-0-tab-bottom .nav-item
.nav-link .active toggle="tab" role="tab" aria-controls="example-0"
aria-selected="true"} [Przykład z prawdziwego
życia](#example-1){#example-1-tab-bottom .nav-item .nav-link
toggle="tab" role="tab" aria-controls="example-1" aria-selected="true"}
:::

::: next
#### Czytaj dalej

[Strategia w języku Swift []{.fa
.fa-arrow-right}](../../strategy/swift/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Wróć

[[]{.fa .fa-arrow-left} Obserwator w języku
Swift](../../observer/swift/example.html){.btn .btn-default rel="prev"}
:::

## **Stan** w innych językach

[![Stan w języku
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Stan w języku C#"){.prog-lang-link}
[![Stan w języku
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Stan w języku C++"){.prog-lang-link}
[![Stan w języku
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Stan w języku Go"){.prog-lang-link}
[![Stan w języku
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Stan w języku Java"){.prog-lang-link}
[![Stan w języku
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Stan w języku PHP"){.prog-lang-link}
[![Stan w języku
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Stan w języku Python"){.prog-lang-link}
[![Stan w języku
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Stan w języku Ruby"){.prog-lang-link}
[![Stan w języku
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Stan w języku Rust"){.prog-lang-link}
[![Stan w języku
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Stan w języku TypeScript"){.prog-lang-link}
::::::::::::::::::::::::::::

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
::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::
