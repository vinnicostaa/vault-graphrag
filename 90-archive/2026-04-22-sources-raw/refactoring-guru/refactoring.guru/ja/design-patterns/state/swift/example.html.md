::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} /
[デザインパターン](../../../design-patterns.html){.type} /
[State](../../state.html){.type} / [Swift](../../swift.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![State](../../../../images/patterns/cards/state-mini63fb.png?id=f4018837e0641d1dade756b6678fd4ee){srcset="/images/patterns/cards/state-mini-2x.png?id=7e24398b27a43c7bd286fc0ea54d2a35 2x"}
:::

# **State** を Swift で {#state-を-swift-で .pattern-example-title-block-title}
::::

:::::::::::::: pattern-example-body
::: pattern-example-brief
**State** は[、]{.chpule2}[
]{.chpuri2}振る舞いに関するデザインパターンの一つで[、]{.chpule2}[
]{.chpuri2}オブジェクトの内部状態が変化した時にその振る舞いを変更することを可能とします[。]{.chpule2}[
]{.chpuri2}

このパターンは[、]{.chpule2}[
]{.chpuri2}状態に関連した振る舞いを個別の状態のクラスへ抽出し[、]{.chpule2}[
]{.chpuri2}元のオブジェクトが作業を自分で行わず[、]{.chpule2}[
]{.chpuri2}これらのクラスのインスタンスに委任することを強制します[。]{.chpule2}[
]{.chpuri2}
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ State の詳細 ](../../state.html){.btn .btn-lg .btn-primary}
:::

::::::::::: {.sidebar-navigation .with-scroll}
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
 [Example](#example-0--Example-swift)
:::

::: en-file
 [Output](#example-0--Output-txt)
:::

::: en-intro
 [現実的な例](#example-1)
:::

::: en-file
 [Example](#example-1--Example-swift)
:::

::: en-file
 [Output](#example-1--Output-txt)
:::
:::::::::::

**複雑度[：]{.chpule2}[ ]{.chpuri2}**[]{.fa-stars}

**人気度[：]{.chpule2}[ ]{.chpuri2}**[]{.fa-stars}

**使用例[：]{.chpule2}[ ]{.chpuri2}**Swift では[、]{.chpule2}[
]{.chpuri2}State パターンは[、]{.chpule2}[ ]{.chpuri2}膨大な数の
`switch`
文に基づく状態機械をオブジェクトに変換する時によく使われます[。]{.chpule2}[
]{.chpuri2}

**見つけ方[：]{.chpule2}[ ]{.chpuri2}**オブジェクトが[、]{.chpule2}[
]{.chpuri2}その外的に制御される状態によって振る舞いを変えるようなメソッドを持っていたら[、]{.chpule2}[
]{.chpuri2}State パターンを識別できます[。]{.chpule2}[ ]{.chpuri2}
::::::::::::::

::::: {style="padding: 1rem; border: solid 4px #e35e5e; border-radius: 1rem; text-align: center"}
::: {style="margin-bottom: 0.5rem;"}
以下の例は [Swift
Playgrounds](https://www.alemohamad.com/playgrounds){style="text-decoration: underline; display: inline-block"
rel="nofollow"}で利用できます。
:::

::: {style="font-size: 13px;"}
Playgroundバージョンを作成してくれた [Alejandro
Mohamad](https://www.alemohamad.com/){style="text-decoration: underline;"
rel="nofollow"}に感謝します。
:::
:::::

::: {#nav-tab .nav .nav-tabs .nav-justified role="tablist"}
[概念的な例](#example-0){#example-0-tab .nav-item .nav-link .active
toggle="tab" role="tab" aria-controls="example-0" aria-selected="true"}
[現実的な例](#example-1){#example-1-tab .nav-item .nav-link toggle="tab"
role="tab" aria-controls="example-1" aria-selected="true"}
:::

::::: {#nav-tabContent .pattern-example-tab-content .tab-content}
::: {#example-0 .tab-pane .show .active role="tabpanel" aria-labelledby="example-0-tab" style="padding-top: 24px"}
## 概念的な例 {#example-0-title}

この例は[、]{.chpule2}[ ]{.chpuri2}**State**
デザインパターンの構造を説明するためのものです[。]{.chpule2}[
]{.chpuri2}以下の質問に答えることを目的としています[：]{.chpule2}[
]{.chpuri2}

-   どういうクラスからできているか[？]{.chpule2}[ ]{.chpuri2}
-   それぞれのクラスの役割は[？]{.chpule2}[ ]{.chpuri2}
-   パターンの要素同士はどう関係しているのか[？]{.chpule2}[ ]{.chpuri2}

ここでパターンの構造を学んだ後だと[、]{.chpule2}[
]{.chpuri2}これに続く[、]{.chpule2}[ ]{.chpuri2}現実世界の Swift
でのユースケースが理解しやすくなります[。]{.chpule2}[ ]{.chpuri2}

#### []{#example-0--Example-swift .anchor} **Example.swift:** 概念的な例

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

#### []{#example-0--Output-txt .anchor} **Output.txt:** 実行結果

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
## 現実的な例 {#example-1-title}

#### []{#example-1--Example-swift .anchor} **Example.swift:** 現実的な例

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

#### []{#example-1--Output-txt .anchor} **Output.txt:** 実行結果

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
[概念的な例](#example-0){#example-0-tab-bottom .nav-item .nav-link
.active toggle="tab" role="tab" aria-controls="example-0"
aria-selected="true"} [現実的な例](#example-1){#example-1-tab-bottom
.nav-item .nav-link toggle="tab" role="tab" aria-controls="example-1"
aria-selected="true"}
:::

::: next
#### 次を読む

[Strategy を Swift で []{.fa
.fa-arrow-right}](../../strategy/swift/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### 戻る

[[]{.fa .fa-arrow-left} Observer を Swift
で](../../observer/swift/example.html){.btn .btn-default rel="prev"}
:::

## 他言語での **State**

[![State を C#
で](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "State を C# で"){.prog-lang-link}
[![State を C++
で](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "State を C++ で"){.prog-lang-link}
[![State を Go
で](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "State を Go で"){.prog-lang-link}
[![State を Java
で](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "State を Java で"){.prog-lang-link}
[![State を PHP
で](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "State を PHP で"){.prog-lang-link}
[![State を Python
で](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "State を Python で"){.prog-lang-link}
[![State を Ruby
で](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "State を Ruby で"){.prog-lang-link}
[![State を Rust
で](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "State を Rust で"){.prog-lang-link}
[![State を TypeScript
で](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "State を TypeScript で"){.prog-lang-link}
::::::::::::::::::::::::::::

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
::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::
