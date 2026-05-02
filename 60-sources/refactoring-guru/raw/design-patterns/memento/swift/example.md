::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../index.html){.home} / [Design
Patterns](../../../design-patterns.html){.type} /
[Memento](../../memento.html){.type} / [Swift](../../swift.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Memento](../../../images/patterns/cards/memento-mini723d.png?id=8b2ea4dc2c5d15775a654808cc9de099){srcset="/images/patterns/cards/memento-mini-2x.png?id=1d7cba189261dd84b11369a6838b1055 2x"}
:::

# **Memento** in Swift {#memento-in-swift .pattern-example-title-block-title}
::::

:::::::::::::: pattern-example-body
::: pattern-example-brief
**Memento** is a behavioral design pattern that allows making snapshots
of an object's state and restoring it in future.

The Memento doesn't compromise the internal structure of the object it
works with, as well as data kept inside the snapshots.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Learn more about Memento ](../../memento.html){.btn .btn-lg
.btn-primary}
:::

::::::::::: {.sidebar-navigation .with-scroll}
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
 [Example](#example-0--Example-swift)
:::

::: en-file
 [Output](#example-0--Output-txt)
:::

::: en-intro
 [Real World Example](#example-1)
:::

::: en-file
 [Example](#example-1--Example-swift)
:::

::: en-file
 [Output](#example-1--Output-txt)
:::
:::::::::::

**Complexity:** []{.fa-stars}

**Popularity:** []{.fa-stars}

**Usage examples:** The Memento's principle can be achieved using
serialization, which is quite common in Swift. While it's not the only
and the most efficient way to make snapshots of an object's state, it
still allows storing state backups while protecting the originator's
structure from other objects.
::::::::::::::

::::: {style="padding: 1rem; border: solid 4px #e35e5e; border-radius: 1rem; text-align: center"}
::: {style="margin-bottom: 0.5rem;"}
The following examples are available on [Swift
Playgrounds](https://www.alemohamad.com/playgrounds){style="text-decoration: underline; display: inline-block"
rel="nofollow"}.
:::

::: {style="font-size: 13px;"}
Kudos to [Alejandro
Mohamad](https://www.alemohamad.com/){style="text-decoration: underline;"
rel="nofollow"} for creating the Playground version.
:::
:::::

::: {#nav-tab .nav .nav-tabs .nav-justified role="tablist"}
[Conceptual Example](#example-0){#example-0-tab .nav-item .nav-link
.active toggle="tab" role="tab" aria-controls="example-0"
aria-selected="true"} [Real World Example](#example-1){#example-1-tab
.nav-item .nav-link toggle="tab" role="tab" aria-controls="example-1"
aria-selected="true"}
:::

::::: {#nav-tabContent .pattern-example-tab-content .tab-content}
::: {#example-0 .tab-pane .show .active role="tabpanel" aria-labelledby="example-0-tab" style="padding-top: 24px"}
## Conceptual Example {#example-0-title}

This example illustrates the structure of the **Memento** design pattern
and focuses on the following questions:

-   What classes does it consist of?
-   What roles do these classes play?
-   In what way the elements of the pattern are related?

After learning about the pattern's structure it'll be easier for you to
grasp the following example, based on a real-world Swift use case.

#### []{#example-0--Example-swift .anchor} **Example.swift:** Conceptual example

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

#### []{#example-0--Output-txt .anchor} **Output.txt:** Execution result

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
## Real World Example {#example-1-title}

#### []{#example-1--Example-swift .anchor} **Example.swift:** Real world example

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

#### []{#example-1--Output-txt .anchor} **Output.txt:** Execution result

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
[Conceptual Example](#example-0){#example-0-tab-bottom .nav-item
.nav-link .active toggle="tab" role="tab" aria-controls="example-0"
aria-selected="true"} [Real World
Example](#example-1){#example-1-tab-bottom .nav-item .nav-link
toggle="tab" role="tab" aria-controls="example-1" aria-selected="true"}
:::

::: next
#### Read next

[Observer in Swift []{.fa
.fa-arrow-right}](../../observer/swift/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Return

[[]{.fa .fa-arrow-left} Mediator in
Swift](../../mediator/swift/example.html){.btn .btn-default rel="prev"}
:::

## **Memento** in Other Languages

[![Memento in
C#](../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Memento in C#"){.prog-lang-link}
[![Memento in
C++](../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Memento in C++"){.prog-lang-link}
[![Memento in
Go](../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Memento in Go"){.prog-lang-link}
[![Memento in
Java](../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Memento in Java"){.prog-lang-link}
[![Memento in
PHP](../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Memento in PHP"){.prog-lang-link}
[![Memento in
Python](../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Memento in Python"){.prog-lang-link}
[![Memento in
Ruby](../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Memento in Ruby"){.prog-lang-link}
[![Memento in
Rust](../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Memento in Rust"){.prog-lang-link}
[![Memento in
TypeScript](../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Memento in TypeScript"){.prog-lang-link}
::::::::::::::::::::::::::::

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
::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::
