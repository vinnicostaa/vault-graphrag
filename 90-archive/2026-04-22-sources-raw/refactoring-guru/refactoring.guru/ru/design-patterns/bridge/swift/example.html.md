::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Паттерны
проектирования](../../../design-patterns.html){.type} /
[Мост](../../bridge.html){.type} / [Swift](../../swift.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Мост](../../../../images/patterns/cards/bridge-mini9e60.png?id=b389101d8ee8e23ffa1b534c704d0774){srcset="/images/patterns/cards/bridge-mini-2x.png?id=2622384cf623ed150ee9c21a0812dd87 2x"}
:::

# **Мост** на Swift {#мост-на-swift .pattern-example-title-block-title}
::::

:::::::::::::: pattern-example-body
::: pattern-example-brief
**Мост** --- это структурный паттерн, который разделяет бизнес-логику
или большой класс на несколько отдельных иерархий, которые потом можно
развивать отдельно друг от друга.

Одна из этих иерархий (абстракция) получит ссылку на объекты другой
иерархии (реализация) и будет делегировать им основную работу. Благодаря
тому, что все реализации будут следовать общему интерфейсу, их можно
будет взаимозаменять внутри абстракции.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Подробней о паттерне Мост ](../../bridge.html){.btn .btn-lg
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

**Применимость:** Паттерн Мост особенно полезен когда вам приходится
делать кросс-платформенные приложения, поддерживать несколько типов баз
данных или работать с разными поставщиками похожего API (например,
cloud-сервисы, социальные сети и т. д.)

**Признаки применения паттерна:** Если в программе чётко выделены классы
«управления» и несколько видов классов «платформ», причём управляющие
объекты делегируют выполнение платформам, то можно сказать, что у вас
используется Мост.
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

Этот пример показывает структуру паттерна **Мост**, а именно --- из
каких классов он состоит, какие роли эти классы выполняют и как они
взаимодействуют друг с другом.

После ознакомления со структурой, вам будет легче воспринимать второй
пример, который рассматривает реальный случай использования паттерна в
мире Swift.

#### []{#example-0--Example-swift .anchor} **Example.swift:** Пример структуры паттерна

<figure class="code">
<pre class="code" lang="swift"><code>import XCTest

/// Абстракция устанавливает интерфейс для «управляющей» части двух иерархий
/// классов. Она содержит ссылку на объект из иерархии Реализации и делегирует
/// ему всю настоящую работу.
class Abstraction {

    fileprivate var implementation: Implementation

    init(_ implementation: Implementation) {
        self.implementation = implementation
    }

    func operation() -&gt; String {
        let operation = implementation.operationImplementation()
        return &quot;Abstraction: Base operation with:\n&quot; + operation
    }
}

/// Можно расширить Абстракцию без изменения классов Реализации.
class ExtendedAbstraction: Abstraction {

    override func operation() -&gt; String {
        let operation = implementation.operationImplementation()
        return &quot;ExtendedAbstraction: Extended operation with:\n&quot; + operation
    }
}

/// Реализация устанавливает интерфейс для всех классов реализации. Он не должен
/// соответствовать интерфейсу Абстракции. На практике оба интерфейса могут быть
/// совершенно разными. Как правило, интерфейс Реализации предоставляет только
/// примитивные операции, в то время как Абстракция определяет операции более
/// высокого уровня, основанные на этих примитивах.
protocol Implementation {

    func operationImplementation() -&gt; String
}

/// Каждая Конкретная Реализация соответствует определённой платформе и
/// реализует интерфейс Реализации с использованием API этой платформы.
class ConcreteImplementationA: Implementation {

    func operationImplementation() -&gt; String {
        return &quot;ConcreteImplementationA: Here&#39;s the result on the platform A.\n&quot;
    }
}

class ConcreteImplementationB: Implementation {

    func operationImplementation() -&gt; String {
        return &quot;ConcreteImplementationB: Here&#39;s the result on the platform B\n&quot;
    }
}

/// За исключением этапа инициализации, когда объект Абстракции связывается с
/// определённым объектом Реализации, клиентский код должен зависеть только от
/// класса Абстракции. Таким образом, клиентский код может поддерживать любую
/// комбинацию абстракции и реализации.
class Client {
    // ...
    static func someClientCode(abstraction: Abstraction) {
        print(abstraction.operation())
    }
    // ...
}

/// Давайте посмотрим как всё это будет работать.
class BridgeConceptual: XCTestCase {

    func testBridgeConceptual() {
        // Клиентский код должен работать с любой предварительно
        // сконфигурированной комбинацией абстракции и реализации.
        let implementation = ConcreteImplementationA()
        Client.someClientCode(abstraction: Abstraction(implementation))

        let concreteImplementation = ConcreteImplementationB()
        Client.someClientCode(abstraction: ExtendedAbstraction(concreteImplementation))
    }
}</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** Результат выполнения

<figure class="code">
<pre class="code" lang="output"><code>Abstraction: Base operation with:
ConcreteImplementationA: Here&#39;s the result on the platform A

ExtendedAbstraction: Extended operation with:
ConcreteImplementationB: Here&#39;s the result on the platform B</code></pre>
</figure>
:::

::: {#example-1 .tab-pane role="tabpanel" aria-labelledby="example-1-tab" style="padding-top: 24px"}
## Пример из реальной жизни {#example-1-title}

#### []{#example-1--Example-swift .anchor} **Example.swift:** Пример из реальной жизни

<figure class="code">
<pre class="code" lang="swift"><code>import XCTest

private class BridgeRealWorld: XCTestCase {

    func testBridgeRealWorld() {

        print(&quot;Client: Pushing Photo View Controller...&quot;)
        push(PhotoViewController())

        print()

        print(&quot;Client: Pushing Feed View Controller...&quot;)
        push(FeedViewController())
    }

    func push(_ container: SharingSupportable) {

        let instagram = InstagramSharingService()
        let facebook = FaceBookSharingService()

        container.accept(service: instagram)
        container.update(content: foodModel)

        container.accept(service: facebook)
        container.update(content: foodModel)
    }

    var foodModel: Content {
        return FoodDomainModel(title: &quot;This food is so various and delicious!&quot;,
                               images: [UIImage(), UIImage()],
                               calories: 47)
    }
}

private protocol SharingSupportable {

    /// Abstraction
    func accept(service: SharingService)

    func update(content: Content)
}

class BaseViewController: UIViewController, SharingSupportable {

    fileprivate var shareService: SharingService?

    func update(content: Content) {
        /// ...updating UI and showing a content...
        /// ...
        /// ... then, a user will choose a content and trigger an event
        print(&quot;\(description): User selected a \(content) to share&quot;)
        /// ...
        shareService?.share(content: content)
    }

    func accept(service: SharingService) {
        shareService = service
    }
}

class PhotoViewController: BaseViewController {

    /// Custom UI and features

    override var description: String {
        return &quot;PhotoViewController&quot;
    }
}

class FeedViewController: BaseViewController {

    /// Custom UI and features

    override var description: String {
        return &quot;FeedViewController&quot;
    }
}

protocol SharingService {

    /// Implementation
    func share(content: Content)
}

class FaceBookSharingService: SharingService {

    func share(content: Content) {

        /// Use FaceBook API to share a content
        print(&quot;Service: \(content) was posted to the Facebook&quot;)
    }
}

class InstagramSharingService: SharingService {

    func share(content: Content) {

        /// Use Instagram API to share a content
        print(&quot;Service: \(content) was posted to the Instagram&quot;, terminator: &quot;\n\n&quot;)
    }
}

protocol Content: CustomStringConvertible {

    var title: String { get }
    var images: [UIImage] { get }
}

struct FoodDomainModel: Content {

    var title: String
    var images: [UIImage]
    var calories: Int

    var description: String {
        return &quot;Food Model&quot;
    }
}</code></pre>
</figure>

#### []{#example-1--Output-txt .anchor} **Output.txt:** Результат выполнения

<figure class="code">
<pre class="code" lang="output"><code>Client: Pushing Photo View Controller...
PhotoViewController: User selected a Food Model to share
Service: Food Model was posted to the Instagram

PhotoViewController: User selected a Food Model to share
Service: Food Model was posted to the Facebook

Client: Pushing Feed View Controller...
FeedViewController: User selected a Food Model to share
Service: Food Model was posted to the Instagram

FeedViewController: User selected a Food Model to share
Service: Food Model was posted to the Facebook</code></pre>
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

[Компоновщик на Swift []{.fa
.fa-arrow-right}](../../composite/swift/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Адаптер на
Swift](../../adapter/swift/example.html){.btn .btn-default rel="prev"}
:::

## **Мост** на других языках программирования

[![Мост на
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Мост на C#"){.prog-lang-link}
[![Мост на
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Мост на C++"){.prog-lang-link}
[![Мост на
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Мост на Go"){.prog-lang-link}
[![Мост на
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Мост на Java"){.prog-lang-link}
[![Мост на
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Мост на PHP"){.prog-lang-link}
[![Мост на
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Мост на Python"){.prog-lang-link}
[![Мост на
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Мост на Ruby"){.prog-lang-link}
[![Мост на
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Мост на Rust"){.prog-lang-link}
[![Мост на
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Мост на TypeScript"){.prog-lang-link}
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
