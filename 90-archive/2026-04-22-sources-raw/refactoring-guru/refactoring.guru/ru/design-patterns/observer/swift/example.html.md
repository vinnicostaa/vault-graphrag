::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Паттерны
проектирования](../../../design-patterns.html){.type} /
[Наблюдатель](../../observer.html){.type} /
[Swift](../../swift.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Наблюдатель](../../../../images/patterns/cards/observer-minifa2e.png?id=fd2081ab1cff29c60b499bcf6a62786a){srcset="/images/patterns/cards/observer-mini-2x.png?id=f205b0655572ac8e4636691c0e0debfd 2x"}
:::

# **Наблюдатель** на Swift {#наблюдатель-на-swift .pattern-example-title-block-title}
::::

:::::::::::::: pattern-example-body
::: pattern-example-brief
**Наблюдатель** --- это поведенческий паттерн, который позволяет
объектам оповещать другие объекты об изменениях своего состояния.

При этом наблюдатели могут свободно подписываться и отписываться от этих
оповещений.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Подробней о паттерне Наблюдатель ](../../observer.html){.btn .btn-lg
.btn-primary}
:::

::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
Навигация
:::

::: en-intro
 [Интро](#)
:::

::: en-intro
 [Концептуальный пример](#example-0)
:::

::: en-file
 [Example](#example-0--Example-swift)
:::

::: en-file
 [Output](#example-0--Output-txt)
:::

::: en-intro
 [Пример из реальной жизни](#example-1)
:::

::: en-file
 [Example](#example-1--Example-swift)
:::

::: en-file
 [Output](#example-1--Output-txt)
:::
:::::::::::

**Сложность:** []{.fa-stars}

**Популярность:** []{.fa-stars}

**Применимость:** Наблюдатель можно часто встретить в Swift коде,
особенно там, где применяется событийная модель отношений между
компонентами. Наблюдатель позволяет отдельным компонентам реагировать на
события, происходящие в других компонентах.

**Признаки применения паттерна:** Наблюдатель можно определить по
механизму подписки и методам оповещения, которые вызывают компоненты
программы.
::::::::::::::

::::: {style="padding: 1rem; border: solid 4px #e35e5e; border-radius: 1rem; text-align: center"}
::: {style="margin-bottom: 0.5rem;"}
Следующие примеры доступны на [Swift
Playgrounds](https://www.alemohamad.com/playgrounds){style="text-decoration: underline; display: inline-block"
rel="nofollow"}.
:::

::: {style="font-size: 13px;"}
Благодарность [Alejandro
Mohamad](https://www.alemohamad.com/){style="text-decoration: underline;"
rel="nofollow"} за создание версии Playground.
:::
:::::

::: {#nav-tab .nav .nav-tabs .nav-justified role="tablist"}
[Концептуальный пример](#example-0){#example-0-tab .nav-item .nav-link
.active toggle="tab" role="tab" aria-controls="example-0"
aria-selected="true"} [Пример из реальной
жизни](#example-1){#example-1-tab .nav-item .nav-link toggle="tab"
role="tab" aria-controls="example-1" aria-selected="true"}
:::

::::: {#nav-tabContent .pattern-example-tab-content .tab-content}
::: {#example-0 .tab-pane .show .active role="tabpanel" aria-labelledby="example-0-tab" style="padding-top: 24px"}
## Концептуальный пример {#example-0-title}

Этот пример показывает структуру паттерна **Наблюдатель**, а именно ---
из каких классов он состоит, какие роли эти классы выполняют и как они
взаимодействуют друг с другом.

После ознакомления со структурой, вам будет легче воспринимать второй
пример, который рассматривает реальный случай использования паттерна в
мире Swift.

#### []{#example-0--Example-swift .anchor} **Example.swift:** Пример структуры паттерна

<figure class="code">
<pre class="code" lang="swift"><code>import XCTest

/// Издатель владеет некоторым важным состоянием и оповещает наблюдателей о его
/// изменениях.
class Subject {

    /// Для удобства в этой переменной хранится состояние Издателя, необходимое
    /// всем подписчикам.
    var state: Int = { return Int(arc4random_uniform(10)) }()

    /// @var array Список подписчиков. В реальной жизни список подписчиков может
    /// храниться в более подробном виде (классифицируется по типу события и
    /// т.д.)
    private lazy var observers = [Observer]()

    /// Методы управления подпиской.
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

    /// Запуск обновления в каждом подписчике.
    func notify() {
        print(&quot;Subject: Notifying observers...\n&quot;)
        observers.forEach({ $0.update(subject: self)})
    }

    /// Обычно логика подписки – только часть того, что делает Издатель.
    /// Издатели часто содержат некоторую важную бизнес-логику, которая
    /// запускает метод уведомления всякий раз, когда должно произойти что-то
    /// важное (или после этого).
    func someBusinessLogic() {
        print(&quot;\nSubject: I&#39;m doing something important.\n&quot;)
        state = Int(arc4random_uniform(10))
        print(&quot;Subject: My state has just changed to: \(state)\n&quot;)
        notify()
    }
}

/// Наблюдатель объявляет метод уведомления, который используют издатели для
/// оповещения.
protocol Observer: AnyObject {

    func update(subject: Subject)
}

/// Конкретные Наблюдатели реагируют на обновления, выпущенные Издателем, к
/// которому они прикреплены.
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

/// Давайте посмотрим как всё это будет работать.
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

#### []{#example-0--Output-txt .anchor} **Output.txt:** Результат выполнения

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
## Пример из реальной жизни {#example-1-title}

#### []{#example-1--Example-swift .anchor} **Example.swift:** Пример из реальной жизни

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

#### []{#example-1--Output-txt .anchor} **Output.txt:** Результат выполнения

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
[Концептуальный пример](#example-0){#example-0-tab-bottom .nav-item
.nav-link .active toggle="tab" role="tab" aria-controls="example-0"
aria-selected="true"} [Пример из реальной
жизни](#example-1){#example-1-tab-bottom .nav-item .nav-link
toggle="tab" role="tab" aria-controls="example-1" aria-selected="true"}
:::

::: next
#### Читаем дальше

[Состояние на Swift []{.fa
.fa-arrow-right}](../../state/swift/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Снимок на
Swift](../../memento/swift/example.html){.btn .btn-default rel="prev"}
:::

## **Наблюдатель** на других языках программирования

[![Наблюдатель на
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Наблюдатель на C#"){.prog-lang-link}
[![Наблюдатель на
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Наблюдатель на C++"){.prog-lang-link}
[![Наблюдатель на
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Наблюдатель на Go"){.prog-lang-link}
[![Наблюдатель на
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Наблюдатель на Java"){.prog-lang-link}
[![Наблюдатель на
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Наблюдатель на PHP"){.prog-lang-link}
[![Наблюдатель на
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Наблюдатель на Python"){.prog-lang-link}
[![Наблюдатель на
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Наблюдатель на Ruby"){.prog-lang-link}
[![Наблюдатель на
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Наблюдатель на Rust"){.prog-lang-link}
[![Наблюдатель на
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Наблюдатель на TypeScript"){.prog-lang-link}
::::::::::::::::::::::::::::

::::::: {.prom .banner-sidebar .banner .banner-striped .banner-examples .banner-removable .banner-removable-patterns data-id="DP: Examples-IDE" creative-id="examples-sidebar-ru" position="sidebar"}
:::::: {.banner-inner style="font-size: 14px; line-height: 21px;"}
::: banner-examples-img
[![](../../../../images/patterns/banners/examples-ide91ca.png?id=3115b4b548fb96b75974e2de8f4f49bc){width="215"
height="158" loading="lazy"
srcset="/images/patterns/banners/examples-ide-2x.png?id=93c007a6d157b97c28eb213f59d716ae 2x"}](../../book.html)
:::

::: banner-examples-fade
[ Архив с примерами](../../book.html){.btn .btn-secondary .btn-block}
:::

::: {.banner-examples-text style="margin-top: 1rem;"}
Купи книгу **Погружение в Паттерны** и получи архив с десятками
детальных примеров, которые можно открывать прямо в IDE.

[ Узнать больше...](../../book.html){.btn .btn-secondary .btn-block}
:::
::::::
:::::::
::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::
