::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Wzorce
projektowe](../../../design-patterns.html){.type} /
[Budowniczy](../../builder.html){.type} /
[Swift](../../swift.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Budowniczy](../../../../images/patterns/cards/builder-mini6d24.png?id=19b95fd05e6469679752c0554b116815){srcset="/images/patterns/cards/builder-mini-2x.png?id=de6d0938678b86903a1426dddfdd13bf 2x"}
:::

# **Budowniczy** w języku Swift {#budowniczy-w-języku-swift .pattern-example-title-block-title}
::::

:::::::::::::: pattern-example-body
::: pattern-example-brief
**Budowniczy** to kreacyjny wzorzec projektowy umożliwiający tworzenie
złożonych obiektów krok po kroku.

W przeciwieństwie do innych wzorców kreacyjnych Budowniczy nie zakłada
definiowania wspólnego interfejsu dla produktów. Dzięki temu da się
wytwarzać różne produkty stosując ten sam proces konstrukcyjny.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Dowiedz się więcej o Budowniczy ](../../builder.html){.btn .btn-lg
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

**Przykłady użycia:** Wzorzec Budowniczy jest dobrze znany w świecie
Swift. Przydaje się szczególnie gdy istnieje potrzeba tworzenia
obiektów w wielu różnych możliwych konfiguracjach.

**Identyfikacja:** Wzorzec Budowniczy można poznać po klasie
posiadającej jedną metodę kreacyjną i wiele metod służących konfiguracji
tworzonego obiektu. Metody budowniczych można zwykle łańcuchować, na
przykład: `someBuilder.setValueA(1).setValueB(2).create()`.
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

Poniższy przykład ilustruje strukturę wzorca **Budowniczy** ze
szczególnym naciskiem na następujące zagadnienia:

-   Z jakich składa się klas?
-   Jakie role pełnią te klasy?
-   W jaki sposób elementy wzorca są ze sobą powiązane?

Poznawszy strukturę wzorca będzie ci łatwiej zrozumieć poniższy
przykład, oparty na prawdziwym przypadku użycia Swift.

#### []{#example-0--Example-swift .anchor} **Example.swift:** Przykład koncepcyjny

<figure class="code">
<pre class="code" lang="swift"><code>import XCTest

/// The Builder interface specifies methods for creating the different parts of
/// the Product objects.
protocol Builder {

    func producePartA()
    func producePartB()
    func producePartC()
}

/// The Concrete Builder classes follow the Builder interface and provide
/// specific implementations of the building steps. Your program may have
/// several variations of Builders, implemented differently.
class ConcreteBuilder1: Builder {

    /// A fresh builder instance should contain a blank product object, which is
    /// used in further assembly.
    private var product = Product1()

    func reset() {
        product = Product1()
    }

    /// All production steps work with the same product instance.
    func producePartA() {
        product.add(part: &quot;PartA1&quot;)
    }

    func producePartB() {
        product.add(part: &quot;PartB1&quot;)
    }

    func producePartC() {
        product.add(part: &quot;PartC1&quot;)
    }

    /// Concrete Builders are supposed to provide their own methods for
    /// retrieving results. That&#39;s because various types of builders may create
    /// entirely different products that don&#39;t follow the same interface.
    /// Therefore, such methods cannot be declared in the base Builder interface
    /// (at least in a statically typed programming language).
    ///
    /// Usually, after returning the end result to the client, a builder
    /// instance is expected to be ready to start producing another product.
    /// That&#39;s why it&#39;s a usual practice to call the reset method at the end of
    /// the `getProduct` method body. However, this behavior is not mandatory,
    /// and you can make your builders wait for an explicit reset call from the
    /// client code before disposing of the previous result.
    func retrieveProduct() -&gt; Product1 {
        let result = self.product
        reset()
        return result
    }
}

/// The Director is only responsible for executing the building steps in a
/// particular sequence. It is helpful when producing products according to a
/// specific order or configuration. Strictly speaking, the Director class is
/// optional, since the client can control builders directly.
class Director {

    private var builder: Builder?

    /// The Director works with any builder instance that the client code passes
    /// to it. This way, the client code may alter the final type of the newly
    /// assembled product.
    func update(builder: Builder) {
        self.builder = builder
    }

    /// The Director can construct several product variations using the same
    /// building steps.
    func buildMinimalViableProduct() {
        builder?.producePartA()
    }

    func buildFullFeaturedProduct() {
        builder?.producePartA()
        builder?.producePartB()
        builder?.producePartC()
    }
}

/// It makes sense to use the Builder pattern only when your products are quite
/// complex and require extensive configuration.
///
/// Unlike in other creational patterns, different concrete builders can produce
/// unrelated products. In other words, results of various builders may not
/// always follow the same interface.
class Product1 {

    private var parts = [String]()

    func add(part: String) {
        self.parts.append(part)
    }

    func listParts() -&gt; String {
        return &quot;Product parts: &quot; + parts.joined(separator: &quot;, &quot;) + &quot;\n&quot;
    }
}

/// The client code creates a builder object, passes it to the director and then
/// initiates the construction process. The end result is retrieved from the
/// builder object.
class Client {
    // ...
    static func someClientCode(director: Director) {
        let builder = ConcreteBuilder1()
        director.update(builder: builder)
        
        print(&quot;Standard basic product:&quot;)
        director.buildMinimalViableProduct()
        print(builder.retrieveProduct().listParts())

        print(&quot;Standard full featured product:&quot;)
        director.buildFullFeaturedProduct()
        print(builder.retrieveProduct().listParts())

        // Remember, the Builder pattern can be used without a Director class.
        print(&quot;Custom product:&quot;)
        builder.producePartA()
        builder.producePartC()
        print(builder.retrieveProduct().listParts())
    }
    // ...
}

/// Let&#39;s see how it all comes together.
class BuilderConceptual: XCTestCase {

    func testBuilderConceptual() {
        let director = Director()
        Client.someClientCode(director: director)
    }
}</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** Wynik działania

<figure class="code">
<pre class="code" lang="output"><code>Standard basic product:
Product parts: PartA1

Standard full featured product:
Product parts: PartA1, PartB1, PartC1

Custom product:
Product parts: PartA1, PartC1</code></pre>
</figure>
:::

::: {#example-1 .tab-pane role="tabpanel" aria-labelledby="example-1-tab" style="padding-top: 24px"}
## Przykład z prawdziwego życia {#example-1-title}

#### []{#example-1--Example-swift .anchor} **Example.swift:** Przykład z prawdziwego życia

<figure class="code">
<pre class="code" lang="swift"><code>import Foundation
import XCTest


class BaseQueryBuilder&lt;Model: DomainModel&gt; {

    typealias Predicate = (Model) -&gt; (Bool)

    func limit(_ limit: Int) -&gt; BaseQueryBuilder&lt;Model&gt; {
        return self
    }

    func filter(_ predicate: @escaping Predicate) -&gt; BaseQueryBuilder&lt;Model&gt; {
        return self
    }

    func fetch() -&gt; [Model] {
        preconditionFailure(&quot;Should be overridden in subclasses.&quot;)
    }
}

class RealmQueryBuilder&lt;Model: DomainModel&gt;: BaseQueryBuilder&lt;Model&gt; {

    enum Query {
        case filter(Predicate)
        case limit(Int)
        /// ...
    }

    fileprivate var operations = [Query]()

    @discardableResult
    override func limit(_ limit: Int) -&gt; RealmQueryBuilder&lt;Model&gt; {
        operations.append(Query.limit(limit))
        return self
    }

    @discardableResult
    override func filter(_ predicate: @escaping Predicate) -&gt; RealmQueryBuilder&lt;Model&gt; {
        operations.append(Query.filter(predicate))
        return self
    }

    override func fetch() -&gt; [Model] {
        print(&quot;RealmQueryBuilder: Initializing RealmDataProvider with \(operations.count) operations:&quot;)
        return RealmProvider().fetch(operations)
    }
}

class CoreDataQueryBuilder&lt;Model: DomainModel&gt;: BaseQueryBuilder&lt;Model&gt; {

    enum Query {
        case filter(Predicate)
        case limit(Int)
        case includesPropertyValues(Bool)
        /// ...
    }

    fileprivate var operations = [Query]()

    override func limit(_ limit: Int) -&gt; CoreDataQueryBuilder&lt;Model&gt; {
        operations.append(Query.limit(limit))
        return self
    }

    override func filter(_ predicate: @escaping Predicate) -&gt; CoreDataQueryBuilder&lt;Model&gt; {
        operations.append(Query.filter(predicate))
        return self
    }

    func includesPropertyValues(_ toggle: Bool) -&gt; CoreDataQueryBuilder&lt;Model&gt; {
        operations.append(Query.includesPropertyValues(toggle))
        return self
    }

    override func fetch() -&gt; [Model] {
        print(&quot;CoreDataQueryBuilder: Initializing CoreDataProvider with \(operations.count) operations.&quot;)
        return CoreDataProvider().fetch(operations)
    }
}


/// Data Providers contain a logic how to fetch models. Builders accumulate
/// operations and then update providers to fetch the data.

class RealmProvider {

    func fetch&lt;Model: DomainModel&gt;(_ operations: [RealmQueryBuilder&lt;Model&gt;.Query]) -&gt; [Model] {

        print(&quot;RealmProvider: Retrieving data from Realm...&quot;)

        for item in operations {
            switch item {
            case .filter(_):
                print(&quot;RealmProvider: executing the &#39;filter&#39; operation.&quot;)
                /// Use Realm instance to filter results.
                break
            case .limit(_):
                print(&quot;RealmProvider: executing the &#39;limit&#39; operation.&quot;)
                /// Use Realm instance to limit results.
                break
            }
        }

        /// Return results from Realm
        return []
    }
}

class CoreDataProvider {

    func fetch&lt;Model: DomainModel&gt;(_ operations: [CoreDataQueryBuilder&lt;Model&gt;.Query]) -&gt; [Model] {

        /// Create a NSFetchRequest

        print(&quot;CoreDataProvider: Retrieving data from CoreData...&quot;)

        for item in operations {
            switch item {
            case .filter(_):
                print(&quot;CoreDataProvider: executing the &#39;filter&#39; operation.&quot;)
                /// Set a &#39;predicate&#39; for a NSFetchRequest.
                break
            case .limit(_):
                print(&quot;CoreDataProvider: executing the &#39;limit&#39; operation.&quot;)
                /// Set a &#39;fetchLimit&#39; for a NSFetchRequest.
                break
            case .includesPropertyValues(_):
                print(&quot;CoreDataProvider: executing the &#39;includesPropertyValues&#39; operation.&quot;)
                /// Set an &#39;includesPropertyValues&#39; for a NSFetchRequest.
                break
            }
        }

        /// Execute a NSFetchRequest and return results.
        return []
    }
}


protocol DomainModel {
    /// The protocol groups domain models to the common interface
}

private struct User: DomainModel {
    let id: Int
    let age: Int
    let email: String
}


class BuilderRealWorld: XCTestCase {

    func testBuilderRealWorld() {
        print(&quot;Client: Start fetching data from Realm&quot;)
        clientCode(builder: RealmQueryBuilder&lt;User&gt;())

        print()

        print(&quot;Client: Start fetching data from CoreData&quot;)
        clientCode(builder: CoreDataQueryBuilder&lt;User&gt;())
    }

    fileprivate func clientCode(builder: BaseQueryBuilder&lt;User&gt;) {

        let results = builder.filter({ $0.age &lt; 20 })
            .limit(1)
            .fetch()

        print(&quot;Client: I have fetched: &quot; + String(results.count) + &quot; records.&quot;)
    }
}</code></pre>
</figure>

#### []{#example-1--Output-txt .anchor} **Output.txt:** Wynik działania

<figure class="code">
<pre class="code" lang="output"><code>Client: Start fetching data from Realm
RealmQueryBuilder: Initializing RealmDataProvider with 2 operations:
RealmProvider: Retrieving data from Realm...
RealmProvider: executing the &#39;filter&#39; operation.
RealmProvider: executing the &#39;limit&#39; operation.
Client: I have fetched: 0 records.

Client: Start fetching data from CoreData
CoreDataQueryBuilder: Initializing CoreDataProvider with 2 operations.
CoreDataProvider: Retrieving data from CoreData...
CoreDataProvider: executing the &#39;filter&#39; operation.
CoreDataProvider: executing the &#39;limit&#39; operation.
Client: I have fetched: 0 records.</code></pre>
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

[Metoda wytwórcza w języku Swift []{.fa
.fa-arrow-right}](../../factory-method/swift/example.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Wróć

[[]{.fa .fa-arrow-left} Fabryka abstrakcyjna w języku
Swift](../../abstract-factory/swift/example.html){.btn .btn-default
rel="prev"}
:::

## **Budowniczy** w innych językach

[![Budowniczy w języku
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Budowniczy w języku C#"){.prog-lang-link}
[![Budowniczy w języku
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Budowniczy w języku C++"){.prog-lang-link}
[![Budowniczy w języku
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Budowniczy w języku Go"){.prog-lang-link}
[![Budowniczy w języku
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Budowniczy w języku Java"){.prog-lang-link}
[![Budowniczy w języku
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Budowniczy w języku PHP"){.prog-lang-link}
[![Budowniczy w języku
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Budowniczy w języku Python"){.prog-lang-link}
[![Budowniczy w języku
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Budowniczy w języku Ruby"){.prog-lang-link}
[![Budowniczy w języku
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Budowniczy w języku Rust"){.prog-lang-link}
[![Budowniczy w języku
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Budowniczy w języku TypeScript"){.prog-lang-link}
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
