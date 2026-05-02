::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../index.html){.home} / [Design
Patterns](../../../design-patterns.html){.type} /
[Flyweight](../../flyweight.html){.type} /
[Swift](../../swift.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Flyweight](../../../images/patterns/cards/flyweight-minie3d8.png?id=422ca8d2f90614dce810a8812c626698){srcset="/images/patterns/cards/flyweight-mini-2x.png?id=857ca676fff84317bb0dab76abfce08e 2x"}
:::

# **Flyweight** in Swift {#flyweight-in-swift .pattern-example-title-block-title}
::::

:::::::::::::: pattern-example-body
::: pattern-example-brief
**Flyweight** is a structural design pattern that allows programs to
support vast quantities of objects by keeping their memory consumption
low.

The pattern achieves it by sharing parts of object state between
multiple objects. In other words, the Flyweight saves RAM by caching the
same data used by different objects.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Learn more about Flyweight ](../../flyweight.html){.btn .btn-lg
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

**Usage examples:** The Flyweight pattern has a single purpose:
minimizing memory intake. If your program doesn't struggle with a
shortage of RAM, then you might just ignore this pattern for a while.

**Identification:** Flyweight can be recognized by a creation method
that returns cached objects instead of creating new.
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

This example illustrates the structure of the **Flyweight** design
pattern and focuses on the following questions:

-   What classes does it consist of?
-   What roles do these classes play?
-   In what way the elements of the pattern are related?

After learning about the pattern's structure it'll be easier for you to
grasp the following example, based on a real-world Swift use case.

#### []{#example-0--Example-swift .anchor} **Example.swift:** Conceptual example

<figure class="code">
<pre class="code" lang="swift"><code>import XCTest

/// The Flyweight stores a common portion of the state (also called intrinsic
/// state) that belongs to multiple real business entities. The Flyweight
/// accepts the rest of the state (extrinsic state, unique for each entity) via
/// its method parameters.
class Flyweight {

    private let sharedState: [String]

    init(sharedState: [String]) {
        self.sharedState = sharedState
    }

    func operation(uniqueState: [String]) {
        print(&quot;Flyweight: Displaying shared (\(sharedState)) and unique (\(uniqueState)) state.\n&quot;)
    }
}

/// The Flyweight Factory creates and manages the Flyweight objects. It ensures
/// that flyweights are shared correctly. When the client requests a flyweight,
/// the factory either returns an existing instance or creates a new one, if it
/// doesn&#39;t exist yet.
class FlyweightFactory {

    private var flyweights: [String: Flyweight]

    init(states: [[String]]) {

        var flyweights = [String: Flyweight]()

        for state in states {
            flyweights[state.key] = Flyweight(sharedState: state)
        }

        self.flyweights = flyweights
    }

    /// Returns an existing Flyweight with a given state or creates a new one.
    func flyweight(for state: [String]) -&gt; Flyweight {

        let key = state.key

        guard let foundFlyweight = flyweights[key] else {

            print(&quot;FlyweightFactory: Can&#39;t find a flyweight, creating new one.\n&quot;)
            let flyweight = Flyweight(sharedState: state)
            flyweights.updateValue(flyweight, forKey: key)
            return flyweight
        }
        print(&quot;FlyweightFactory: Reusing existing flyweight.\n&quot;)
        return foundFlyweight
    }

    func printFlyweights() {
        print(&quot;FlyweightFactory: I have \(flyweights.count) flyweights:\n&quot;)
        for item in flyweights {
            print(item.key)
        }
    }
}

extension Array where Element == String {

    /// Returns a Flyweight&#39;s string hash for a given state.
    var key: String {
        return self.joined()
    }
}


class FlyweightConceptual: XCTestCase {

    func testFlyweight() {

        /// The client code usually creates a bunch of pre-populated flyweights
        /// in the initialization stage of the application.

        let factory = FlyweightFactory(states:
        [
            [&quot;Chevrolet&quot;, &quot;Camaro2018&quot;, &quot;pink&quot;],
            [&quot;Mercedes Benz&quot;, &quot;C300&quot;, &quot;black&quot;],
            [&quot;Mercedes Benz&quot;, &quot;C500&quot;, &quot;red&quot;],
            [&quot;BMW&quot;, &quot;M5&quot;, &quot;red&quot;],
            [&quot;BMW&quot;, &quot;X6&quot;, &quot;white&quot;]
        ])

        factory.printFlyweights()

        /// ...

        addCarToPoliceDatabase(factory,
                &quot;CL234IR&quot;,
                &quot;James Doe&quot;,
                &quot;BMW&quot;,
                &quot;M5&quot;,
                &quot;red&quot;)

        addCarToPoliceDatabase(factory,
                &quot;CL234IR&quot;,
                &quot;James Doe&quot;,
                &quot;BMW&quot;,
                &quot;X1&quot;,
                &quot;red&quot;)

        factory.printFlyweights()
    }

    func addCarToPoliceDatabase(
            _ factory: FlyweightFactory,
            _ plates: String,
            _ owner: String,
            _ brand: String,
            _ model: String,
            _ color: String) {

        print(&quot;Client: Adding a car to database.\n&quot;)

        let flyweight = factory.flyweight(for: [brand, model, color])

        /// The client code either stores or calculates extrinsic state and
        /// passes it to the flyweight&#39;s methods.
        flyweight.operation(uniqueState: [plates, owner])
    }
}</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** Execution result

<figure class="code">
<pre class="code" lang="output"><code>FlyweightFactory: I have 5 flyweights:

Mercedes BenzC500red
ChevroletCamaro2018pink
Mercedes BenzC300black
BMWX6white
BMWM5red
Client: Adding a car to database.

FlyweightFactory: Reusing existing flyweight.

Flyweight: Displaying shared ([&quot;BMW&quot;, &quot;M5&quot;, &quot;red&quot;]) and unique ([&quot;CL234IR&quot;, &quot;James Doe&quot;] state.

Client: Adding a car to database.

FlyweightFactory: Can&#39;t find a flyweight, creating new one.

Flyweight: Displaying shared ([&quot;BMW&quot;, &quot;X1&quot;, &quot;red&quot;]) and unique ([&quot;CL234IR&quot;, &quot;James Doe&quot;] state.

FlyweightFactory: I have 6 flyweights:

Mercedes BenzC500red
BMWX1red
ChevroletCamaro2018pink
Mercedes BenzC300black
BMWX6white
BMWM5red</code></pre>
</figure>
:::

::: {#example-1 .tab-pane role="tabpanel" aria-labelledby="example-1-tab" style="padding-top: 24px"}
## Real World Example {#example-1-title}

#### []{#example-1--Example-swift .anchor} **Example.swift:** Real world example

<figure class="code">
<pre class="code" lang="swift"><code>import XCTest
import UIKit

class FlyweightRealWorld: XCTestCase {

    func testFlyweightRealWorld() {

        let maineCoon = Animal(name: &quot;Maine Coon&quot;,
                               country: &quot;USA&quot;,
                               type: .cat)

        let sphynx = Animal(name: &quot;Sphynx&quot;,
                            country: &quot;Egypt&quot;,
                            type: .cat)

        let bulldog = Animal(name: &quot;Bulldog&quot;,
                             country: &quot;England&quot;,
                             type: .dog)

        print(&quot;Client: I created a number of objects to display&quot;)

        /// Displaying objects for the 1-st time.

        print(&quot;Client: Let&#39;s show animals for the 1st time\n&quot;)
        display(animals: [maineCoon, sphynx, bulldog])


        /// Displaying objects for the 2-nd time.
        ///
        /// Note: The cached appearance object will be reused this time.

        print(&quot;\nClient: I have a new dog, let&#39;s show it the same way!\n&quot;)

        let germanShepherd = Animal(name: &quot;German Shepherd&quot;,
                              country: &quot;Germany&quot;,
                              type: .dog)

        display(animals: [germanShepherd])
    }
}

extension FlyweightRealWorld {

    func display(animals: [Animal]) {

        let cells = loadCells(count: animals.count)

        for index in 0..&lt;animals.count {
            cells[index].update(with: animals[index])
        }

        /// Using cells...
    }

    func loadCells(count: Int) -&gt; [Cell] {
        /// Emulates behavior of a table/collection view.
        return Array(repeating: Cell(), count: count)
    }
}

enum Type: String {
    case cat
    case dog
}

class Cell {

    private var animal: Animal?

    func update(with animal: Animal) {
        self.animal = animal
        let type = animal.type.rawValue
        let photos = &quot;photos \(animal.appearance.photos.count)&quot;
        print(&quot;Cell: Updating an appearance of a \(type)-cell: \(photos)\n&quot;)
    }
}

struct Animal: Equatable {

    /// This is an external context that contains specific values and an object
    /// with a common state.
    ///
    /// Note: The object of appearance will be lazily created when it is needed

    let name: String
    let country: String
    let type: Type

    var appearance: Appearance {
        return AppearanceFactory.appearance(for: type)
    }
}

struct Appearance: Equatable {

    /// This object contains the predefined appearance for each cell.

    let photos: [UIImage]
    let backgroundColor: UIColor
}

extension Animal: CustomStringConvertible {

    var description: String {
        return &quot;\(name), \(country), \(type.rawValue) + \(appearance.description)&quot;
    }
}

extension Appearance: CustomStringConvertible {

    var description: String {
        return &quot;photos: \(photos.count), \(backgroundColor)&quot;
    }
}

class AppearanceFactory {

    private static var cache = [Type: Appearance]()

    static func appearance(for key: Type) -&gt; Appearance {

        guard cache[key] == nil else {
            print(&quot;AppearanceFactory: Reusing an existing \(key.rawValue)-appearance.&quot;)
            return cache[key]!
        }

        print(&quot;AppearanceFactory: Can&#39;t find a cached \(key.rawValue)-object, creating a new one.&quot;)

        switch key {
        case .cat:
            cache[key] = catInfo
        case .dog:
            cache[key] = dogInfo
        }

        return cache[key]!
    }
}

extension AppearanceFactory {

    private static var catInfo: Appearance {
        return Appearance(photos: [UIImage()], backgroundColor: .red)
    }

    private static var dogInfo: Appearance {
        return Appearance(photos: [UIImage(), UIImage()], backgroundColor: .blue)
    }
}</code></pre>
</figure>

#### []{#example-1--Output-txt .anchor} **Output.txt:** Execution result

<figure class="code">
<pre class="code" lang="output"><code>Client: I created a number of objects to display
Client: Let&#39;s show animals for the 1st time

AppearanceFactory: Can&#39;t find a cached cat-object, creating a new one.
Cell: Updating an appearance of a cat-cell: photos 1

AppearanceFactory: Reusing an existing cat-appearance.
Cell: Updating an appearance of a cat-cell: photos 1

AppearanceFactory: Can&#39;t find a cached dog-object, creating a new one.
Cell: Updating an appearance of a dog-cell: photos 2


Client: I have a new dog, let&#39;s show it the same way!

AppearanceFactory: Reusing an existing dog-appearance.
Cell: Updating an appearance of a dog-cell: photos 2</code></pre>
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

[Proxy in Swift []{.fa
.fa-arrow-right}](../../proxy/swift/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Return

[[]{.fa .fa-arrow-left} Facade in
Swift](../../facade/swift/example.html){.btn .btn-default rel="prev"}
:::

## **Flyweight** in Other Languages

[![Flyweight in
C#](../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Flyweight in C#"){.prog-lang-link}
[![Flyweight in
C++](../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Flyweight in C++"){.prog-lang-link}
[![Flyweight in
Go](../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Flyweight in Go"){.prog-lang-link}
[![Flyweight in
Java](../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Flyweight in Java"){.prog-lang-link}
[![Flyweight in
PHP](../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Flyweight in PHP"){.prog-lang-link}
[![Flyweight in
Python](../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Flyweight in Python"){.prog-lang-link}
[![Flyweight in
Ruby](../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Flyweight in Ruby"){.prog-lang-link}
[![Flyweight in
Rust](../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Flyweight in Rust"){.prog-lang-link}
[![Flyweight in
TypeScript](../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Flyweight in TypeScript"){.prog-lang-link}
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
