::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../index.html){.home} / [Design
Patterns](../../../design-patterns.html){.type} /
[Strategy](../../strategy.html){.type} /
[Swift](../../swift.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Strategy](../../../images/patterns/cards/strategy-mini28f6.png?id=d38abee4fb6f2aed909d262bdadca936){srcset="/images/patterns/cards/strategy-mini-2x.png?id=f4e6608561f8e5d18be6927d4620ad29 2x"}
:::

# **Strategy** in Swift {#strategy-in-swift .pattern-example-title-block-title}
::::

:::::::::::::: pattern-example-body
::: pattern-example-brief
**Strategy** is a behavioral design pattern that turns a set of
behaviors into objects and makes them interchangeable inside original
context object.

The original object, called context, holds a reference to a strategy
object. The context delegates executing the behavior to the linked
strategy object. In order to change the way the context performs its
work, other objects may replace the currently linked strategy object
with another one.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Learn more about Strategy ](../../strategy.html){.btn .btn-lg
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

**Usage examples:** The Strategy pattern is very common in Swift code.
It's often used in various frameworks to provide users a way to change
the behavior of a class without extending it.

**Identification:** Strategy pattern can be recognized by a method that
lets a nested object do the actual work, as well as a setter that allows
replacing that object with a different one.
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

This example illustrates the structure of the **Strategy** design
pattern and focuses on the following questions:

-   What classes does it consist of?
-   What roles do these classes play?
-   In what way the elements of the pattern are related?

After learning about the pattern's structure it'll be easier for you to
grasp the following example, based on a real-world Swift use case.

#### []{#example-0--Example-swift .anchor} **Example.swift:** Conceptual example

<figure class="code">
<pre class="code" lang="swift"><code>import XCTest

/// The Context defines the interface of interest to clients.
class Context {

    /// The Context maintains a reference to one of the Strategy objects. The
    /// Context does not know the concrete class of a strategy. It should work
    /// with all strategies via the Strategy interface.
    private var strategy: Strategy

    /// Usually, the Context accepts a strategy through the constructor, but
    /// also provides a setter to change it at runtime.
    init(strategy: Strategy) {
        self.strategy = strategy
    }

    /// Usually, the Context allows replacing a Strategy object at runtime.
    func update(strategy: Strategy) {
        self.strategy = strategy
    }

    /// The Context delegates some work to the Strategy object instead of
    /// implementing multiple versions of the algorithm on its own.
    func doSomeBusinessLogic() {
        print(&quot;Context: Sorting data using the strategy (not sure how it&#39;ll do it)\n&quot;)

        let result = strategy.doAlgorithm([&quot;a&quot;, &quot;b&quot;, &quot;c&quot;, &quot;d&quot;, &quot;e&quot;])
        print(result.joined(separator: &quot;,&quot;))
    }
}

/// The Strategy interface declares operations common to all supported versions
/// of some algorithm.
///
/// The Context uses this interface to call the algorithm defined by Concrete
/// Strategies.
protocol Strategy {

    func doAlgorithm&lt;T: Comparable&gt;(_ data: [T]) -&gt; [T]
}

/// Concrete Strategies implement the algorithm while following the base
/// Strategy interface. The interface makes them interchangeable in the Context.
class ConcreteStrategyA: Strategy {

    func doAlgorithm&lt;T: Comparable&gt;(_ data: [T]) -&gt; [T] {
        return data.sorted()
    }
}

class ConcreteStrategyB: Strategy {

    func doAlgorithm&lt;T: Comparable&gt;(_ data: [T]) -&gt; [T] {
        return data.sorted(by: &gt;)
    }
}

/// Let&#39;s see how it all works together.
class StrategyConceptual: XCTestCase {

    func test() {

        /// The client code picks a concrete strategy and passes it to the
        /// context. The client should be aware of the differences between
        /// strategies in order to make the right choice.

        let context = Context(strategy: ConcreteStrategyA())
        print(&quot;Client: Strategy is set to normal sorting.\n&quot;)
        context.doSomeBusinessLogic()

        print(&quot;\nClient: Strategy is set to reverse sorting.\n&quot;)
        context.update(strategy: ConcreteStrategyB())
        context.doSomeBusinessLogic()
    }
}</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** Execution result

<figure class="code">
<pre class="code" lang="output"><code>Client: Strategy is set to normal sorting.

Context: Sorting data using the strategy (not sure how it&#39;ll do it)

a,b,c,d,e

Client: Strategy is set to reverse sorting.

Context: Sorting data using the strategy (not sure how it&#39;ll do it)

e,d,c,b,a</code></pre>
</figure>
:::

::: {#example-1 .tab-pane role="tabpanel" aria-labelledby="example-1-tab" style="padding-top: 24px"}
## Real World Example {#example-1-title}

#### []{#example-1--Example-swift .anchor} **Example.swift:** Real world example

<figure class="code">
<pre class="code" lang="swift"><code>import XCTest

class StrategyRealWorld: XCTestCase {

    /// This example shows a simple implementation of a list controller that is
    /// able to display models from different data sources:
    ///
    /// (MemoryStorage, CoreDataStorage, RealmStorage)

    func test() {

        let controller = ListController()

        let memoryStorage = MemoryStorage&lt;User&gt;()
        memoryStorage.add(usersFromNetwork())

        clientCode(use: controller, with: memoryStorage)

        clientCode(use: controller, with: CoreDataStorage())

        clientCode(use: controller, with: RealmStorage())
    }

    func clientCode(use controller: ListController, with dataSource: DataSource) {

        controller.update(dataSource: dataSource)
        controller.displayModels()
    }

    private func usersFromNetwork() -&gt; [User] {
        let firstUser = User(id: 1, username: &quot;username1&quot;)
        let secondUser = User(id: 2, username: &quot;username2&quot;)
        return [firstUser, secondUser]
    }
}

class ListController {

    private var dataSource: DataSource?

    func update(dataSource: DataSource) {
        /// ... resest current states ...
        self.dataSource = dataSource
    }

    func displayModels() {

        guard let dataSource = dataSource else { return }
        let models = dataSource.loadModels() as [User]

        /// Bind models to cells of a list view...
        print(&quot;\nListController: Displaying models...&quot;)
        models.forEach({ print($0) })
    }
}

protocol DataSource {

    func loadModels&lt;T: DomainModel&gt;() -&gt; [T]
}

class MemoryStorage&lt;Model&gt;: DataSource {

    private lazy var items = [Model]()

    func add(_ items: [Model]) {
        self.items.append(contentsOf: items)
    }

    func loadModels&lt;T: DomainModel&gt;() -&gt; [T] {
        guard T.self == User.self else { return [] }
        return items as! [T]
    }
}

class CoreDataStorage: DataSource {

    func loadModels&lt;T: DomainModel&gt;() -&gt; [T] {
        guard T.self == User.self else { return [] }

        let firstUser = User(id: 3, username: &quot;username3&quot;)
        let secondUser = User(id: 4, username: &quot;username4&quot;)

        return [firstUser, secondUser] as! [T]
    }
}

class RealmStorage: DataSource {

    func loadModels&lt;T: DomainModel&gt;() -&gt; [T] {
        guard T.self == User.self else { return [] }

        let firstUser = User(id: 5, username: &quot;username5&quot;)
        let secondUser = User(id: 6, username: &quot;username6&quot;)

        return [firstUser, secondUser] as! [T]
    }
}

protocol DomainModel {

    var id: Int { get }
}

struct User: DomainModel {

    var id: Int
    var username: String
}</code></pre>
</figure>

#### []{#example-1--Output-txt .anchor} **Output.txt:** Execution result

<figure class="code">
<pre class="code" lang="output"><code>ListController: Displaying models...
User(id: 1, username: &quot;username1&quot;)
User(id: 2, username: &quot;username2&quot;)

ListController: Displaying models...
User(id: 3, username: &quot;username3&quot;)
User(id: 4, username: &quot;username4&quot;)

ListController: Displaying models...
User(id: 5, username: &quot;username5&quot;)
User(id: 6, username: &quot;username6&quot;)</code></pre>
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

[Template Method in Swift []{.fa
.fa-arrow-right}](../../template-method/swift/example.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Return

[[]{.fa .fa-arrow-left} State in
Swift](../../state/swift/example.html){.btn .btn-default rel="prev"}
:::

## **Strategy** in Other Languages

[![Strategy in
C#](../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Strategy in C#"){.prog-lang-link}
[![Strategy in
C++](../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Strategy in C++"){.prog-lang-link}
[![Strategy in
Go](../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Strategy in Go"){.prog-lang-link}
[![Strategy in
Java](../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Strategy in Java"){.prog-lang-link}
[![Strategy in
PHP](../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Strategy in PHP"){.prog-lang-link}
[![Strategy in
Python](../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Strategy in Python"){.prog-lang-link}
[![Strategy in
Ruby](../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Strategy in Ruby"){.prog-lang-link}
[![Strategy in
Rust](../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Strategy in Rust"){.prog-lang-link}
[![Strategy in
TypeScript](../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Strategy in TypeScript"){.prog-lang-link}
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
