:::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [디자인
패턴들](../../../design-patterns.html){.type} /
[커맨드](../../command.html){.type} /
[스위프트](../../swift.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![커맨드](../../../../images/patterns/cards/command-mini8482.png?id=b149eda017c0583c1e92343b83cfb1eb){srcset="/images/patterns/cards/command-mini-2x.png?id=e5f6332e057f6d352a209da963a8fc54 2x"}
:::

# 스위프트로 작성된 **커맨드** {#스위프트로-작성된-커맨드 .pattern-example-title-block-title}
::::

:::::::::::::: pattern-example-body
::: pattern-example-brief
**커맨드**는 요청 또는 간단한 작업을 객체로 변환하는 행동 디자인
패턴입니다.

이러한 변환은 명령의 지연 또는 원격 실행, 명령 기록 저장 등을
허용합니다.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ 커맨드에 대하여 더 자세히 알아보세요 ](../../command.html){.btn
.btn-lg .btn-primary}
:::

::::::::::: {.sidebar-navigation .with-scroll}
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
 [Example](#example-0--Example-swift)
:::

::: en-file
 [Output](#example-0--Output-txt)
:::

::: en-intro
 [실제 사례 예시](#example-1)
:::

::: en-file
 [Example](#example-1--Example-swift)
:::

::: en-file
 [Output](#example-1--Output-txt)
:::
:::::::::::

**복잡도:** []{.fa-stars}

**인기도:** []{.fa-stars}

**사용 예시들:** 커맨드 패턴은 스위프트 코드에서 매우 일반적입니다.
대부분의 경우 작업으로 UI 요소를 매개 변수화하기 위한 콜백의 대안으로
사용되며 작업 대기, 작업 기록 추적 등에도 사용됩니다.

**식별:** 커맨드 패턴은 다음과 같은 특징이 있습니다. 추상/인터페이스
유형​(발신자)​의 행동 메서드들이 있으며 이러한 메서드들은 다른
추상/인터페이스 유형​(수신자)​의 구현에서 메서드를 호출하며 이 메서드는
생성되는 동안 커맨드 구현으로 캡슐화되었습니다. 또 커맨드 클래스는
일반적으로 특정 작업만 수행할 수 있습니다.
::::::::::::::

::: {#nav-tab .nav .nav-tabs .nav-justified role="tablist"}
[개념적인 예시](#example-0){#example-0-tab .nav-item .nav-link .active
toggle="tab" role="tab" aria-controls="example-0" aria-selected="true"}
[실제 사례 예시](#example-1){#example-1-tab .nav-item .nav-link
toggle="tab" role="tab" aria-controls="example-1" aria-selected="true"}
:::

::::: {#nav-tabContent .pattern-example-tab-content .tab-content}
::: {#example-0 .tab-pane .show .active role="tabpanel" aria-labelledby="example-0-tab" style="padding-top: 24px"}
## 개념적인 예시 {#example-0-title}

이 예시는 **커맨드** 디자인 패턴의 구조를 보여주고 다음 질문에 중점을
둡니다:

-   패턴은 어떤 클래스들로 구성되어 있나요?
-   이 클래스들은 어떤 역할을 하나요?
-   패턴의 요소들은 어떻게 서로 연관되어 있나요?

이 패턴의 구조를 배우면 실제 스위프트 사용 사례를 기반으로 하는 다음
예시를 더욱 쉽게 이해할 수 있을 것입니다.

#### []{#example-0--Example-swift .anchor} **Example.swift:** 개념적인 예시

<figure class="code">
<pre class="code" lang="swift"><code>import XCTest

/// The Command interface declares a method for executing a command.
protocol Command {

    func execute()
}

/// Some commands can implement simple operations on their own.
class SimpleCommand: Command {

    private var payload: String

    init(_ payload: String) {
        self.payload = payload
    }

    func execute() {
        print(&quot;SimpleCommand: See, I can do simple things like printing (&quot; + payload + &quot;)&quot;)
    }
}

/// However, some commands can delegate more complex operations to other
/// objects, called &quot;receivers.&quot;
class ComplexCommand: Command {

    private var receiver: Receiver

    /// Context data, required for launching the receiver&#39;s methods.
    private var a: String
    private var b: String

    /// Complex commands can accept one or several receiver objects along with
    /// any context data via the constructor.
    init(_ receiver: Receiver, _ a: String, _ b: String) {
        self.receiver = receiver
        self.a = a
        self.b = b
    }

    /// Commands can delegate to any methods of a receiver.
    func execute() {
        print(&quot;ComplexCommand: Complex stuff should be done by a receiver object.\n&quot;)
        receiver.doSomething(a)
        receiver.doSomethingElse(b)
    }
}

/// The Receiver classes contain some important business logic. They know how to
/// perform all kinds of operations, associated with carrying out a request. In
/// fact, any class may serve as a Receiver.
class Receiver {

    func doSomething(_ a: String) {
        print(&quot;Receiver: Working on (&quot; + a + &quot;)\n&quot;)
    }

    func doSomethingElse(_ b: String) {
        print(&quot;Receiver: Also working on (&quot; + b + &quot;)\n&quot;)
    }
}

/// The Invoker is associated with one or several commands. It sends a request
/// to the command.
class Invoker {

    private var onStart: Command?

    private var onFinish: Command?

    /// Initialize commands.
    func setOnStart(_ command: Command) {
        onStart = command
    }

    func setOnFinish(_ command: Command) {
        onFinish = command
    }

    /// The Invoker does not depend on concrete command or receiver classes. The
    /// Invoker passes a request to a receiver indirectly, by executing a
    /// command.
    func doSomethingImportant() {

        print(&quot;Invoker: Does anybody want something done before I begin?&quot;)

        onStart?.execute()

        print(&quot;Invoker: ...doing something really important...&quot;)
        print(&quot;Invoker: Does anybody want something done after I finish?&quot;)

        onFinish?.execute()
    }
}

/// Let&#39;s see how it all comes together.
class CommandConceptual: XCTestCase {

    func test() {
        /// The client code can parameterize an invoker with any commands.

        let invoker = Invoker()
        invoker.setOnStart(SimpleCommand(&quot;Say Hi!&quot;))

        let receiver = Receiver()
        invoker.setOnFinish(ComplexCommand(receiver, &quot;Send email&quot;, &quot;Save report&quot;))
        invoker.doSomethingImportant()
    }
}</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** 실행 결과

<figure class="code">
<pre class="code" lang="output"><code>Invoker: Does anybody want something done before I begin?
SimpleCommand: See, I can do simple things like printing (Say Hi!)
Invoker: ...doing something really important...
Invoker: Does anybody want something done after I finish?
ComplexCommand: Complex stuff should be done by a receiver object.

Receiver: Working on (Send email)

Receiver: Also working on (Save report)</code></pre>
</figure>
:::

::: {#example-1 .tab-pane role="tabpanel" aria-labelledby="example-1-tab" style="padding-top: 24px"}
## 실제 사례 예시 {#example-1-title}

#### []{#example-1--Example-swift .anchor} **Example.swift:** 실제 사례 예시

<figure class="code">
<pre class="code" lang="swift"><code>import Foundation
import XCTest


class DelayedOperation: Operation, @unchecked Sendable {

    private var delay: TimeInterval

    init(_ delay: TimeInterval = 0) {
        self.delay = delay
    }

    override var isExecuting : Bool {
        get { return _executing }
        set {
            willChangeValue(forKey: &quot;isExecuting&quot;)
            _executing = newValue
            didChangeValue(forKey: &quot;isExecuting&quot;)
        }
    }
    private var _executing : Bool = false

    override var isFinished : Bool {
        get { return _finished }
        set {
            willChangeValue(forKey: &quot;isFinished&quot;)
            _finished = newValue
            didChangeValue(forKey: &quot;isFinished&quot;)
        }
    }
    private var _finished : Bool = false

    override func start() {

        guard delay &gt; 0 else {
            _start()
            return
        }

        let deadline = DispatchTime.now() + delay
        DispatchQueue(label: &quot;&quot;).asyncAfter(deadline: deadline) {
            self._start()
        }
    }

    private func _start() {

        guard !self.isCancelled else {
            print(&quot;\(self): operation is canceled&quot;)
            self.isFinished = true
            return
        }

        self.isExecuting = true
        self.main()
        self.isExecuting = false
        self.isFinished = true
    }
}

class WindowOperation: DelayedOperation, @unchecked Sendable {

    override func main() {
        print(&quot;\(self): Windows are closed via HomeKit.&quot;)
    }

    override var description: String { return &quot;WindowOperation&quot; }
}

class DoorOperation: DelayedOperation, @unchecked Sendable {

    override func main() {
        print(&quot;\(self): Doors are closed via HomeKit.&quot;)
    }

    override var description: String { return &quot;DoorOperation&quot; }
}

class TaxiOperation: DelayedOperation, @unchecked Sendable {

    override func main() {
        print(&quot;\(self): Taxi is ordered via Uber&quot;)
    }

    override var description: String { return &quot;TaxiOperation&quot; }
}



class CommandRealWorld: XCTestCase {

    func testCommandRealWorld() {
        prepareTestEnvironment {

            let siri = SiriShortcuts.shared

            print(&quot;User: Hey Siri, I am leaving my home&quot;)
            siri.perform(.leaveHome)

            print(&quot;User: Hey Siri, I am leaving my work in 3 minutes&quot;)
            siri.perform(.leaveWork, delay: 3) /// for simplicity, we use seconds

            print(&quot;User: Hey Siri, I am still working&quot;)
            siri.cancel(.leaveWork)
        }
    }
}

extension CommandRealWorld {

    struct ExecutionTime {
        static let max: TimeInterval = 5
        static let waiting: TimeInterval = 4
    }

    func prepareTestEnvironment(_ execution: () -&gt; ()) {

        /// This method tells Xcode to wait for async operations. Otherwise the
        /// main test is done immediately.

        let expectation = self.expectation(description: &quot;Expectation for async operations&quot;)

        let deadline = DispatchTime.now() + ExecutionTime.waiting
        DispatchQueue.main.asyncAfter(deadline: deadline) { expectation.fulfill() }

        execution()

        wait(for: [expectation], timeout: ExecutionTime.max)
    }
}

class SiriShortcuts {

    static let shared = SiriShortcuts()
    private lazy var queue = OperationQueue()

    private init() {}

    enum Action: String {
        case leaveHome
        case leaveWork
    }

    func perform(_ action: Action, delay: TimeInterval = 0) {
        print(&quot;Siri: performing \(action)-action\n&quot;)
        switch action {
        case .leaveHome:
            add(operation: WindowOperation(delay))
            add(operation: DoorOperation(delay))
        case .leaveWork:
            add(operation: TaxiOperation(delay))
        }
    }

    func cancel(_ action: Action) {
        print(&quot;Siri: canceling \(action)-action\n&quot;)
        switch action {
        case .leaveHome:
            cancelOperation(with: WindowOperation.self)
            cancelOperation(with: DoorOperation.self)
        case .leaveWork:
            cancelOperation(with: TaxiOperation.self)
        }
    }

    private func cancelOperation(with operationType: Operation.Type) {
        queue.operations.filter { operation in
            return type(of: operation) == operationType
        }.forEach({ $0.cancel() })
    }

    private func add(operation: Operation) {
        queue.addOperation(operation)
    }
}</code></pre>
</figure>

#### []{#example-1--Output-txt .anchor} **Output.txt:** 실행 결과

<figure class="code">
<pre class="code" lang="output"><code>User: Hey Siri, I am leaving my home
Siri: performing leaveHome-action

User: Hey Siri, I am leaving my work in 3 minutes
Siri: performing leaveWork-action

User: Hey Siri, I am still working
Siri: canceling leaveWork-action

DoorOperation: Doors are closed via HomeKit.
WindowOperation: Windows are closed via HomeKit.
TaxiOperation: operation is canceled</code></pre>
</figure>
:::
:::::

::: {#nav-tab .nav .nav-tabs .nav-tabs-bottom .nav-justified role="tablist"}
[개념적인 예시](#example-0){#example-0-tab-bottom .nav-item .nav-link
.active toggle="tab" role="tab" aria-controls="example-0"
aria-selected="true"} [실제 사례 예시](#example-1){#example-1-tab-bottom
.nav-item .nav-link toggle="tab" role="tab" aria-controls="example-1"
aria-selected="true"}
:::

::: next
#### 다음을 보세요

[스위프트로 작성된 반복자 []{.fa
.fa-arrow-right}](../../iterator/swift/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### 돌아가기

[[]{.fa .fa-arrow-left} 스위프트로 작성된 책임
연쇄](../../chain-of-responsibility/swift/example.html){.btn
.btn-default rel="prev"}
:::

## 다른 언어로 작성된 **커맨드**

[![C#으로 작성된
커맨드](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "C#으로 작성된 커맨드"){.prog-lang-link}
[![C++로 작성된
커맨드](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "C++로 작성된 커맨드"){.prog-lang-link}
[![Go로 작성된
커맨드](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Go로 작성된 커맨드"){.prog-lang-link}
[![자바로 작성된
커맨드](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "자바로 작성된 커맨드"){.prog-lang-link}
[![PHP로 작성된
커맨드](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "PHP로 작성된 커맨드"){.prog-lang-link}
[![파이썬으로 작성된
커맨드](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "파이썬으로 작성된 커맨드"){.prog-lang-link}
[![루비로 작성된
커맨드](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "루비로 작성된 커맨드"){.prog-lang-link}
[![러스트로 작성된
커맨드](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "러스트로 작성된 커맨드"){.prog-lang-link}
[![타입스크립트로 작성된
커맨드](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "타입스크립트로 작성된 커맨드"){.prog-lang-link}
:::::::::::::::::::::::::

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
:::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::
