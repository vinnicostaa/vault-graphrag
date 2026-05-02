::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Wzorce
projektowe](../../../design-patterns.html){.type} /
[Obserwator](../../observer.html){.type} /
[Swift](../../swift.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Obserwator](../../../../images/patterns/cards/observer-minifa2e.png?id=fd2081ab1cff29c60b499bcf6a62786a){srcset="/images/patterns/cards/observer-mini-2x.png?id=f205b0655572ac8e4636691c0e0debfd 2x"}
:::

# **Obserwator** w języku Swift {#obserwator-w-języku-swift .pattern-example-title-block-title}
::::

:::::::::::::: pattern-example-body
::: pattern-example-brief
**Obserwator** to behawioralny wzorzec projektowy pozwalający obiektom
powiadamiać inne obiekty o zmianach swojego stanu.

Obserwator daje możliwość subskrypcji lub zrezygnowania z subskrypcji
zdarzeń dowolnego obiektu implementującego interfejs subskrybenta.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Dowiedz się więcej o Obserwator ](../../observer.html){.btn .btn-lg
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

**Przykłady użycia:** Wzorzec Obserwator jest dość powszechny w kodzie
Swift, szczególnie w komponentach graficznego interfejsu użytkownika.
Pozwala reagować na zdarzenia dotyczące innych obiektów bez konieczności
sprzęgania z klasami tych obiektów.

**Identyfikacja:** Wzorzec Obserwator można poznać po obecności metod
służących subskrypcji, które przechowują obiekty w strukturze listy i po
wywołaniach metod aktualizacji obiektów z tej listy.
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

Poniższy przykład ilustruje strukturę wzorca **Obserwator** ze
szczególnym naciskiem na następujące kwestie:

-   Z jakich składa się klas?
-   Jakie role pełnią te klasy?
-   W jaki sposób elementy wzorca są ze sobą powiązane?

Poznawszy strukturę wzorca będzie ci łatwiej zrozumieć następujący
przykład, oparty na prawdziwym przypadku użycia Swift.

#### []{#example-0--Example-swift .anchor} **Example.swift:** Przykład koncepcyjny

<figure class="code">
<pre class="code" lang="swift"><code>import XCTest

/// The Subject owns some important state and notifies observers when the state
/// changes.
class Subject {

    /// For the sake of simplicity, the Subject&#39;s state, essential to all
    /// subscribers, is stored in this variable.
    var state: Int = { return Int(arc4random_uniform(10)) }()

    /// @var array List of subscribers. In real life, the list of subscribers
    /// can be stored more comprehensively (categorized by event type, etc.).
    private lazy var observers = [Observer]()

    /// The subscription management methods.
    func attach(_ observer: Observer) {
        print(&quot;Subject: Attached an observer.\n&quot;)
        observers.append(observer)
    }

    func detach(_ observer: Observer) {
        if let idx = observers.firstIndex(where: { $0 === observer }) {
            observers.remove(at: idx)
            print(&quot;Subject: Detached an observer.\n&quot;)
        }
    }

    /// Trigger an update in each subscriber.
    func notify() {
        print(&quot;Subject: Notifying observers...\n&quot;)
        observers.forEach({ $0.update(subject: self)})
    }

    /// Usually, the subscription logic is only a fraction of what a Subject can
    /// really do. Subjects commonly hold some important business logic, that
    /// triggers a notification method whenever something important is about to
    /// happen (or after it).
    func someBusinessLogic() {
        print(&quot;\nSubject: I&#39;m doing something important.\n&quot;)
        state = Int(arc4random_uniform(10))
        print(&quot;Subject: My state has just changed to: \(state)\n&quot;)
        notify()
    }
}

/// The Observer protocol declares the update method, used by subjects.
protocol Observer: AnyObject {

    func update(subject: Subject)
}

/// Concrete Observers react to the updates issued by the Subject they had been
/// attached to.
class ConcreteObserverA: Observer {

    func update(subject: Subject) {

        if subject.state &lt; 3 {
            print(&quot;ConcreteObserverA: Reacted to the event.\n&quot;)
        }
    }
}

class ConcreteObserverB: Observer {

    func update(subject: Subject) {

        if subject.state &gt;= 3 {
            print(&quot;ConcreteObserverB: Reacted to the event.\n&quot;)
        }
    }
}

/// Let&#39;s see how it all works together.
class ObserverConceptual: XCTestCase {

    func testObserverConceptual() {

        let subject = Subject()

        let observer1 = ConcreteObserverA()
        let observer2 = ConcreteObserverB()

        subject.attach(observer1)
        subject.attach(observer2)

        subject.someBusinessLogic()
        subject.someBusinessLogic()
        subject.detach(observer2)
        subject.someBusinessLogic()
    }
}</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** Wynik działania

<figure class="code">
<pre class="code" lang="output"><code>Subject: Attached an observer.

Subject: Attached an observer.


Subject: I&#39;m doing something important.

Subject: My state has just changed to: 4

Subject: Notifying observers...

ConcreteObserverB: Reacted to the event.


Subject: I&#39;m doing something important.

Subject: My state has just changed to: 2

Subject: Notifying observers...

ConcreteObserverA: Reacted to the event.

Subject: Detached an observer.


Subject: I&#39;m doing something important.

Subject: My state has just changed to: 8

Subject: Notifying observers...</code></pre>
</figure>
:::

::: {#example-1 .tab-pane role="tabpanel" aria-labelledby="example-1-tab" style="padding-top: 24px"}
## Przykład z prawdziwego życia {#example-1-title}

#### []{#example-1--Example-swift .anchor} **Example.swift:** Przykład z prawdziwego życia

<figure class="code">
<pre class="code" lang="swift"><code>import XCTest

class ObserverRealWorld: XCTestCase {

    func test() {

        let cartManager = CartManager()

        let navigationBar = UINavigationBar()
        let cartVC = CartViewController()

        cartManager.add(subscriber: navigationBar)
        cartManager.add(subscriber: cartVC)

        let apple = Food(id: 111, name: &quot;Apple&quot;, price: 10, calories: 20)
        cartManager.add(product: apple)

        let tShirt = Clothes(id: 222, name: &quot;T-shirt&quot;, price: 200, size: &quot;L&quot;)
        cartManager.add(product: tShirt)

        cartManager.remove(product: apple)
    }
}

protocol CartSubscriber: CustomStringConvertible {

    func accept(changed cart: [Product])
}

protocol Product {

    var id: Int { get }
    var name: String { get }
    var price: Double { get }

    func isEqual(to product: Product) -&gt; Bool
}

extension Product {

    func isEqual(to product: Product) -&gt; Bool {
        return id == product.id
    }
}

struct Food: Product {

    var id: Int
    var name: String
    var price: Double

    /// Food-specific properties
    var calories: Int
}

struct Clothes: Product {

    var id: Int
    var name: String
    var price: Double

    /// Clothes-specific properties
    var size: String
}

class CartManager {

    private lazy var cart = [Product]()
    private lazy var subscribers = [CartSubscriber]()

    func add(subscriber: CartSubscriber) {
        print(&quot;CartManager: I&#39;am adding a new subscriber: \(subscriber.description)&quot;)
        subscribers.append(subscriber)
    }

    func add(product: Product) {
        print(&quot;\nCartManager: I&#39;am adding a new product: \(product.name)&quot;)
        cart.append(product)
        notifySubscribers()
    }

    func remove(subscriber filter: (CartSubscriber) -&gt; (Bool)) {
        guard let index = subscribers.firstIndex(where: filter) else { return }
        subscribers.remove(at: index)
    }

    func remove(product: Product) {
        guard let index = cart.firstIndex(where: { $0.isEqual(to: product) }) else { return }
        print(&quot;\nCartManager: Product &#39;\(product.name)&#39; is removed from a cart&quot;)
        cart.remove(at: index)
        notifySubscribers()
    }

    private func notifySubscribers() {
        subscribers.forEach({ $0.accept(changed: cart) })
    }
}

extension UINavigationBar: CartSubscriber {

    func accept(changed cart: [Product]) {
        print(&quot;UINavigationBar: Updating an appearance of navigation items&quot;)
    }

    open override var description: String { return &quot;UINavigationBar&quot; }
}

class CartViewController: UIViewController, CartSubscriber {

    func accept(changed cart: [Product]) {
        print(&quot;CartViewController: Updating an appearance of a list view with products&quot;)
    }

    open override var description: String { return &quot;CartViewController&quot; }
}</code></pre>
</figure>

#### []{#example-1--Output-txt .anchor} **Output.txt:** Wynik działania

<figure class="code">
<pre class="code" lang="output"><code>CartManager: I&#39;am adding a new subscriber: UINavigationBar
CartManager: I&#39;am adding a new subscriber: CartViewController

CartManager: I&#39;am adding a new product: Apple
UINavigationBar: Updating an appearance of navigation items
CartViewController: Updating an appearance of a list view with products

CartManager: I&#39;am adding a new product: T-shirt
UINavigationBar: Updating an appearance of navigation items
CartViewController: Updating an appearance of a list view with products

CartManager: Product &#39;Apple&#39; is removed from a cart
UINavigationBar: Updating an appearance of navigation items
CartViewController: Updating an appearance of a list view with products</code></pre>
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

[Stan w języku Swift []{.fa
.fa-arrow-right}](../../state/swift/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Wróć

[[]{.fa .fa-arrow-left} Pamiątka w języku
Swift](../../memento/swift/example.html){.btn .btn-default rel="prev"}
:::

## **Obserwator** w innych językach

[![Obserwator w języku
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Obserwator w języku C#"){.prog-lang-link}
[![Obserwator w języku
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Obserwator w języku C++"){.prog-lang-link}
[![Obserwator w języku
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Obserwator w języku Go"){.prog-lang-link}
[![Obserwator w języku
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Obserwator w języku Java"){.prog-lang-link}
[![Obserwator w języku
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Obserwator w języku PHP"){.prog-lang-link}
[![Obserwator w języku
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Obserwator w języku Python"){.prog-lang-link}
[![Obserwator w języku
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Obserwator w języku Ruby"){.prog-lang-link}
[![Obserwator w języku
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Obserwator w języku Rust"){.prog-lang-link}
[![Obserwator w języku
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Obserwator w języku TypeScript"){.prog-lang-link}
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
