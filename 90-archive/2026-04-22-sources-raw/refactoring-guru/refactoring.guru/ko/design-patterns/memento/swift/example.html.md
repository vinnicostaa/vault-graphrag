:::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [디자인
패턴들](../../../design-patterns.html){.type} /
[메멘토](../../memento.html){.type} /
[스위프트](../../swift.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![메멘토](../../../../images/patterns/cards/memento-mini723d.png?id=8b2ea4dc2c5d15775a654808cc9de099){srcset="/images/patterns/cards/memento-mini-2x.png?id=1d7cba189261dd84b11369a6838b1055 2x"}
:::

# 스위프트로 작성된 **메멘토** {#스위프트로-작성된-메멘토 .pattern-example-title-block-title}
::::

:::::::::::::: pattern-example-body
::: pattern-example-brief
**메멘토** 패턴은 행동 디자인 패턴입니다. 이 패턴은 객체 상태의 스냅숏을
만든 후 나중에 복원할 수 있도록 합니다.

메멘토는 함께 작동하는 객체의 내부 구조와 스냅숏들 내부에 보관된
데이터를 손상하지 않습니다.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ 메멘토에 대하여 더 자세히 알아보세요 ](../../memento.html){.btn
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

**사용 사례들:** 메멘토의 원칙은 직렬화를 사용하여 달성할 수 있으며,
이는 스위프트 코드에서 매우 일반적입니다. 직렬화는 객체 상태의 스냅숏을
만드는 유일한 또는 가장 효율적인 방법은 아니나 다른 객체로부터
오리지네이터의 구조를 보호하면서 상태 백업을 저장할 수 있도록 합니다.
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

이 예시는 **메멘토** 패턴의 구조를 보여주고 다음 질문에 중점을 둡니다:

-   패턴은 어떤 클래스들로 구성되어 있나요?
-   이 클래스들은 어떤 역할을 하나요?
-   패턴의 요소들은 어떻게 서로 연관되어 있나요?

이 패턴의 구조를 배우면 실제 스위프트 사용 사례를 기반으로 하는 다음
예시를 더욱 쉽게 이해할 수 있을 것입니다.

#### []{#example-0--Example-swift .anchor} **Example.swift:** 개념적인 예시

<figure class="code">
<pre class="code" lang="swift"><code>import XCTest

/// The Originator holds some important state that may change over time. It also
/// defines a method for saving the state inside a memento and another method
/// for restoring the state from it.
class Originator {

    /// For the sake of simplicity, the originator&#39;s state is stored inside a
    /// single variable.
    private var state: String

    init(state: String) {
        self.state = state
        print(&quot;Originator: My initial state is: \(state)&quot;)
    }

    /// The Originator&#39;s business logic may affect its internal state.
    /// Therefore, the client should backup the state before launching methods
    /// of the business logic via the save() method.
    func doSomething() {
        print(&quot;Originator: I&#39;m doing something important.&quot;)
        state = generateRandomString()
        print(&quot;Originator: and my state has changed to: \(state)&quot;)
    }

    private func generateRandomString() -&gt; String {
        return String(UUID().uuidString.suffix(4))
    }

    /// Saves the current state inside a memento.
    func save() -&gt; Memento {
        return ConcreteMemento(state: state)
    }

    /// Restores the Originator&#39;s state from a memento object.
    func restore(memento: Memento) {
        guard let memento = memento as? ConcreteMemento else { return }
        self.state = memento.state
        print(&quot;Originator: My state has changed to: \(state)&quot;)
    }
}

/// The Memento interface provides a way to retrieve the memento&#39;s metadata,
/// such as creation date or name. However, it doesn&#39;t expose the Originator&#39;s
/// state.
protocol Memento {

    var name: String { get }
    var date: Date { get }
}

/// The Concrete Memento contains the infrastructure for storing the
/// Originator&#39;s state.
class ConcreteMemento: Memento {

    /// The Originator uses this method when restoring its state.
    private(set) var state: String
    private(set) var date: Date

    init(state: String) {
        self.state = state
        self.date = Date()
    }

    /// The rest of the methods are used by the Caretaker to display metadata.
    var name: String { return state + &quot; &quot; + date.description.suffix(14).prefix(8) }
}

/// The Caretaker doesn&#39;t depend on the Concrete Memento class. Therefore, it
/// doesn&#39;t have access to the originator&#39;s state, stored inside the memento. It
/// works with all mementos via the base Memento interface.
class Caretaker {

    private lazy var mementos = [Memento]()
    private var originator: Originator

    init(originator: Originator) {
        self.originator = originator
    }

    func backup() {
        print(&quot;\nCaretaker: Saving Originator&#39;s state...\n&quot;)
        mementos.append(originator.save())
    }

    func undo() {

        guard !mementos.isEmpty else { return }
        let removedMemento = mementos.removeLast()

        print(&quot;Caretaker: Restoring state to: &quot; + removedMemento.name)
        originator.restore(memento: removedMemento)
    }

    func showHistory() {
        print(&quot;Caretaker: Here&#39;s the list of mementos:\n&quot;)
        mementos.forEach({ print($0.name) })
    }
}

/// Let&#39;s see how it all works together.
class MementoConceptual: XCTestCase {

    func testMementoConceptual() {

        let originator = Originator(state: &quot;Super-duper-super-puper-super.&quot;)
        let caretaker = Caretaker(originator: originator)

        caretaker.backup()
        originator.doSomething()

        caretaker.backup()
        originator.doSomething()

        caretaker.backup()
        originator.doSomething()

        print(&quot;\n&quot;)
        caretaker.showHistory()

        print(&quot;\nClient: Now, let&#39;s rollback!\n\n&quot;)
        caretaker.undo()

        print(&quot;\nClient: Once more!\n\n&quot;)
        caretaker.undo()
    }
}</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** 실행 결과

<figure class="code">
<pre class="code" lang="output"><code>Originator: My initial state is: Super-duper-super-puper-super.

Caretaker: Saving Originator&#39;s state...

Originator: I&#39;m doing something important.
Originator: and my state has changed to: 1923

Caretaker: Saving Originator&#39;s state...

Originator: I&#39;m doing something important.
Originator: and my state has changed to: 74FB

Caretaker: Saving Originator&#39;s state...

Originator: I&#39;m doing something important.
Originator: and my state has changed to: 3681


Caretaker: Here&#39;s the list of mementos:

Super-duper-super-puper-super. 11:45:44
1923 11:45:44
74FB 11:45:44

Client: Now, let&#39;s rollback!


Caretaker: Restoring state to: 74FB 11:45:44
Originator: My state has changed to: 74FB

Client: Once more!


Caretaker: Restoring state to: 1923 11:45:44
Originator: My state has changed to: 1923</code></pre>
</figure>
:::

::: {#example-1 .tab-pane role="tabpanel" aria-labelledby="example-1-tab" style="padding-top: 24px"}
## 실제 사례 예시 {#example-1-title}

#### []{#example-1--Example-swift .anchor} **Example.swift:** 실제 사례 예시

<figure class="code">
<pre class="code" lang="swift"><code>import XCTest

class MementoRealWorld: XCTestCase {

    /// State and Command are often used together when the previous state of the
    /// object should be restored in case of failure of some operation.
    ///
    /// Note: UndoManager can be used as an alternative.

    func test() {

        let textView = UITextView()
        let undoStack = UndoStack(textView)

        textView.text = &quot;First Change&quot;
        undoStack.save()

        textView.text = &quot;Second Change&quot;
        undoStack.save()

        textView.text = (textView.text ?? &quot;&quot;) + &quot; &amp; Third Change&quot;
        textView.textColor = .red
        undoStack.save()

        print(undoStack)

        print(&quot;Client: Perform Undo operation 2 times\n&quot;)
        undoStack.undo()
        undoStack.undo()

        print(undoStack)
    }
}

class UndoStack: CustomStringConvertible {

    private lazy var mementos = [Memento]()
    private let textView: UITextView

    init(_ textView: UITextView) {
        self.textView = textView
    }

    func save() {
        mementos.append(textView.memento)
    }

    func undo() {
        guard !mementos.isEmpty else { return }
        textView.restore(with: mementos.removeLast())
    }

    var description: String {
        return mementos.reduce(&quot;&quot;, { $0 + $1.description })
    }
}

protocol Memento: CustomStringConvertible {

    var text: String { get }
    var date: Date { get }
}

extension UITextView {

    var memento: Memento {
        return TextViewMemento(text: text,
                               textColor: textColor,
                               selectedRange: selectedRange)
    }

    func restore(with memento: Memento) {
        guard let textViewMemento = memento as? TextViewMemento else { return }

        text = textViewMemento.text
        textColor = textViewMemento.textColor
        selectedRange = textViewMemento.selectedRange
    }

    struct TextViewMemento: Memento {

        let text: String
        let date = Date()

        let textColor: UIColor?
        let selectedRange: NSRange

        var description: String {
            let time = Calendar.current.dateComponents([.hour, .minute, .second, .nanosecond],
                                                       from: date)
            let color = String(describing: textColor)
            return &quot;Text: \(text)\n&quot; + &quot;Date: \(time.description)\n&quot;
                + &quot;Color: \(color)\n&quot; + &quot;Range: \(selectedRange)\n\n&quot;
        }
    }
}</code></pre>
</figure>

#### []{#example-1--Output-txt .anchor} **Output.txt:** 실행 결과

<figure class="code">
<pre class="code" lang="output"><code>Text: First Change
Date: hour: 12 minute: 21 second: 50 nanosecond: 821737051 isLeapMonth: false
Color: nil
Range: {12, 0}

Text: Second Change
Date: hour: 12 minute: 21 second: 50 nanosecond: 826483011 isLeapMonth: false
Color: nil
Range: {13, 0}

Text: Second Change &amp; Third Change
Date: hour: 12 minute: 21 second: 50 nanosecond: 829187035 isLeapMonth: false
Color: Optional(UIExtendedSRGBColorSpace 1 0 0 1)
Range: {28, 0}


Client: Perform Undo operation 2 times

Text: First Change
Date: hour: 12 minute: 21 second: 50 nanosecond: 821737051 isLeapMonth: false
Color: nil
Range: {12, 0}</code></pre>
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

[스위프트로 작성된 옵서버 []{.fa
.fa-arrow-right}](../../observer/swift/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### 돌아가기

[[]{.fa .fa-arrow-left} 스위프트로 작성된
중재자](../../mediator/swift/example.html){.btn .btn-default rel="prev"}
:::

## 다른 언어로 작성된 **메멘토**

[![C#으로 작성된
메멘토](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "C#으로 작성된 메멘토"){.prog-lang-link}
[![C++로 작성된
메멘토](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "C++로 작성된 메멘토"){.prog-lang-link}
[![Go로 작성된
메멘토](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Go로 작성된 메멘토"){.prog-lang-link}
[![자바로 작성된
메멘토](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "자바로 작성된 메멘토"){.prog-lang-link}
[![PHP로 작성된
메멘토](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "PHP로 작성된 메멘토"){.prog-lang-link}
[![파이썬으로 작성된
메멘토](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "파이썬으로 작성된 메멘토"){.prog-lang-link}
[![루비로 작성된
메멘토](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "루비로 작성된 메멘토"){.prog-lang-link}
[![러스트로 작성된
메멘토](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "러스트로 작성된 메멘토"){.prog-lang-link}
[![타입스크립트로 작성된
메멘토](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "타입스크립트로 작성된 메멘토"){.prog-lang-link}
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
