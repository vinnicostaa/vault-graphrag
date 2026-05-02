::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Patrons de
conception](../../../design-patterns.html){.type} /
[Monteur](../../builder.html){.type} / [Swift](../../swift.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Monteur](../../../../images/patterns/cards/builder-mini6d24.png?id=19b95fd05e6469679752c0554b116815){srcset="/images/patterns/cards/builder-mini-2x.png?id=de6d0938678b86903a1426dddfdd13bf 2x"}
:::

# **Monteur** en Swift {#monteur-en-swift .pattern-example-title-block-title}
::::

:::::::::::::: pattern-example-body
::: pattern-example-brief
Le **Monteur** est un patron de conception de création qui permet de
construire des objets complexes étape par étape.

Le monteur n'est pas comme les autres patrons de création : les produits
n'ont pas besoin d'avoir une interface commune. Il est ainsi possible de
créer différents produits en utilisant le même procédé de fabrication.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ En savoir plus sur la patron Monteur ](../../builder.html){.btn
.btn-lg .btn-primary}
:::

::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
Navigation
:::

::: en-intro
 [Intro](#)
:::

::: en-intro
 [Exemple conceptuel](#example-0)
:::

::: en-file
 [Example](#example-0--Example-swift)
:::

::: en-file
 [Output](#example-0--Output-txt)
:::

::: en-intro
 [Analogie du monde réel](#example-1)
:::

::: en-file
 [Example](#example-1--Example-swift)
:::

::: en-file
 [Output](#example-1--Output-txt)
:::
:::::::::::

**Complexité :** []{.fa-stars}

**Popularité :** []{.fa-stars}

**Exemples d'utilisation :** Le monteur est bien connu dans le monde du
Swift. Il se montre très utile lorsque vous devez créer un objet
possédant de nombreuses configurations possibles.

**Identification :** Le monteur peut être identifié à l'intérieur d'une
classe qui n'a qu'une seule méthode de création et plusieurs méthodes
permettant de configurer l'objet en résultant. Les méthodes du monteur
prennent souvent en charge le chaînage (par exemple,
`someBuilder.setValueA(1).setValueB(2).create()`).
::::::::::::::

::::: {style="padding: 1rem; border: solid 4px #e35e5e; border-radius: 1rem; text-align: center"}
::: {style="margin-bottom: 0.5rem;"}
Les exemples suivants sont disponibles sur le site de [Swift
Playgrounds](https://www.alemohamad.com/playgrounds){style="text-decoration: underline; display: inline-block"
rel="nofollow"}.
:::

::: {style="font-size: 13px;"}
Félicitations à [Alejandro
Mohamad](https://www.alemohamad.com/){style="text-decoration: underline;"
rel="nofollow"} pour avoir créé la version du Playground.
:::
:::::

::: {#nav-tab .nav .nav-tabs .nav-justified role="tablist"}
[Exemple conceptuel](#example-0){#example-0-tab .nav-item .nav-link
.active toggle="tab" role="tab" aria-controls="example-0"
aria-selected="true"} [Analogie du monde
réel](#example-1){#example-1-tab .nav-item .nav-link toggle="tab"
role="tab" aria-controls="example-1" aria-selected="true"}
:::

::::: {#nav-tabContent .pattern-example-tab-content .tab-content}
::: {#example-0 .tab-pane .show .active role="tabpanel" aria-labelledby="example-0-tab" style="padding-top: 24px"}
## Exemple conceptuel {#example-0-title}

Dans cet exemple, nous allons voir la structure du **Monteur** et
répondre aux questions suivantes :

-   Que contiennent les classes ?
-   Quels rôles jouent-elles ?
-   Comment les éléments du patron sont-ils reliés ?

Après avoir étudié la structure du patron, vous pourrez plus facilement
comprendre l'exemple suivant qui est basé sur un cas d'utilisation réel
en Swift.

#### []{#example-0--Example-swift .anchor} **Example.swift:** Exemple conceptuel

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

#### []{#example-0--Output-txt .anchor} **Output.txt:** Résultat de l'exécution

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
## Analogie du monde réel {#example-1-title}

#### []{#example-1--Example-swift .anchor} **Example.swift:** Analogie du monde réel

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

#### []{#example-1--Output-txt .anchor} **Output.txt:** Résultat de l'exécution

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
[Exemple conceptuel](#example-0){#example-0-tab-bottom .nav-item
.nav-link .active toggle="tab" role="tab" aria-controls="example-0"
aria-selected="true"} [Analogie du monde
réel](#example-1){#example-1-tab-bottom .nav-item .nav-link toggle="tab"
role="tab" aria-controls="example-1" aria-selected="true"}
:::

::: next
#### Suivant

[Fabrique en Swift []{.fa
.fa-arrow-right}](../../factory-method/swift/example.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Retour

[[]{.fa .fa-arrow-left} Fabrique abstraite en
Swift](../../abstract-factory/swift/example.html){.btn .btn-default
rel="prev"}
:::

## **Monteur** dans les autres langues

[![Monteur en
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Monteur en C#"){.prog-lang-link}
[![Monteur en
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Monteur en C++"){.prog-lang-link}
[![Monteur en
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Monteur en Go"){.prog-lang-link}
[![Monteur en
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Monteur en Java"){.prog-lang-link}
[![Monteur en
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Monteur en PHP"){.prog-lang-link}
[![Monteur en
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Monteur en Python"){.prog-lang-link}
[![Monteur en
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Monteur en Ruby"){.prog-lang-link}
[![Monteur en
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Monteur en Rust"){.prog-lang-link}
[![Monteur en
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Monteur en TypeScript"){.prog-lang-link}
::::::::::::::::::::::::::::

::::::: {.prom .banner-sidebar .banner .banner-striped .banner-examples .banner-removable .banner-removable-patterns data-id="DP: Examples-IDE" creative-id="examples-sidebar-fr" position="sidebar"}
:::::: {.banner-inner style="font-size: 14px; line-height: 21px;"}
::: banner-examples-img
[![](../../../../images/patterns/banners/examples-ide91ca.png?id=3115b4b548fb96b75974e2de8f4f49bc){width="215"
height="158" loading="lazy"
srcset="/images/patterns/banners/examples-ide-2x.png?id=93c007a6d157b97c28eb213f59d716ae 2x"}](../../book.html)
:::

::: banner-examples-fade
[ Archive avec exemples](../../book.html){.btn .btn-secondary
.btn-block}
:::

::: {.banner-examples-text style="margin-top: 1rem;"}
Achetez l'ebook **Plongée au cœur des patrons de conception** et obtenez
l'accès à une archive comprenant des dizaines d'exemples détaillés qui
peuvent être ouverts dans votre environnement de développement.

[ En savoir plus...](../../book.html){.btn .btn-secondary .btn-block}
:::
::::::
:::::::
::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::
