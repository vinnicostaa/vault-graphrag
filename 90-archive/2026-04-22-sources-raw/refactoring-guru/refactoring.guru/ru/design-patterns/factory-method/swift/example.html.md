::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Паттерны
проектирования](../../../design-patterns.html){.type} / [Фабричный
метод](../../factory-method.html){.type} /
[Swift](../../swift.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Фабричный
метод](../../../../images/patterns/cards/factory-method-minid7f6.png?id=72619e9527893374b98a5913779ac167){srcset="/images/patterns/cards/factory-method-mini-2x.png?id=fa9d4a8d61a67cc3822e52b9daf69dad 2x"}
:::

# **Фабричный метод** на Swift {#фабричный-метод-на-swift .pattern-example-title-block-title}
::::

:::::::::::::: pattern-example-body
::: pattern-example-brief
**Фабричный метод** --- это порождающий паттерн проектирования, который
решает проблему создания различных продуктов, без указания конкретных
классов продуктов.

Фабричный метод задаёт метод, который следует использовать вместо вызова
оператора `new` для создания объектов-продуктов. Подклассы могут
переопределить этот метод, чтобы изменять тип создаваемых продуктов.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Подробней о паттерне Фабричный метод ](../../factory-method.html){.btn
.btn-lg .btn-primary}
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

**Применимость:** Паттерн можно часто встретить в любом Swift-коде, где
требуется гибкость при создании продуктов.

**Признаки применения паттерна:** Фабричный метод можно определить по
создающим методам, которые возвращают объекты продуктов через
абстрактные типы или интерфейсы. Это позволяет переопределять типы
создаваемых продуктов в подклассах.
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

Этот пример показывает структуру паттерна **Фабричный метод**, а
именно --- из каких классов он состоит, какие роли эти классы выполняют
и как они взаимодействуют друг с другом.

После ознакомления со структурой, вам будет легче воспринимать второй
пример, который рассматривает реальный случай использования паттерна в
мире Swift.

#### []{#example-0--Example-swift .anchor} **Example.swift:** Пример структуры паттерна

<figure class="code">
<pre class="code" lang="swift"><code>import XCTest

/// Класс Создатель объявляет фабричный метод, который должен возвращать объект
/// класса Продукт. Подклассы Создателя обычно предоставляют реализацию этого
/// метода.
protocol Creator {

    /// Обратите внимание, что Создатель может также обеспечить реализацию
    /// фабричного метода по умолчанию.
    func factoryMethod() -&gt; Product

    /// Также заметьте, что, несмотря на название, основная обязанность
    /// Создателя не заключается в создании продуктов. Обычно он содержит
    /// некоторую базовую бизнес-логику, которая основана на объектах Продуктов,
    /// возвращаемых фабричным методом. Подклассы могут косвенно изменять эту
    /// бизнес-логику, переопределяя фабричный метод и возвращая из него другой
    /// тип продукта.
    func someOperation() -&gt; String
}

/// Это расширение реализует базовое поведение Создателя. Оно может быть
/// переопределено в подклассах.
extension Creator {

    func someOperation() -&gt; String {
        // Вызываем фабричный метод, чтобы получить объект-продукт.
        let product = factoryMethod()

        // Далее, работаем с этим продуктом.
        return &quot;Creator: The same creator&#39;s code has just worked with &quot; + product.operation()
    }
}

/// Конкретные Создатели переопределяют фабричный метод для того, чтобы изменить
/// тип результирующего продукта.
class ConcreteCreator1: Creator {

    /// Обратите внимание, что сигнатура метода по-прежнему использует тип
    /// абстрактного продукта, хотя фактически из метода возвращается конкретный
    /// продукт. Таким образом, Создатель может оставаться независимым от
    /// конкретных классов продуктов.
    public func factoryMethod() -&gt; Product {
        return ConcreteProduct1()
    }
}

class ConcreteCreator2: Creator {

    public func factoryMethod() -&gt; Product {
        return ConcreteProduct2()
    }
}

/// Протокол Продукта объявляет операции, которые должны выполнять все
/// конкретные продукты.
protocol Product {

    func operation() -&gt; String
}

/// Конкретные Продукты предоставляют различные реализации протокола Продукта.
class ConcreteProduct1: Product {

    func operation() -&gt; String {
        return &quot;{Result of the ConcreteProduct1}&quot;
    }
}

class ConcreteProduct2: Product {

    func operation() -&gt; String {
        return &quot;{Result of the ConcreteProduct2}&quot;
    }
}


/// Клиентский код работает с экземпляром конкретного создателя, хотя и через
/// его базовый протокол. Пока клиент продолжает работать с создателем через
/// базовый протокол, вы можете передать ему любой подкласс создателя.
class Client {
    // ...
    static func someClientCode(creator: Creator) {
        print(&quot;Client: I&#39;m not aware of the creator&#39;s class, but it still works.\n&quot;
            + creator.someOperation())
    }
    // ...
}

/// Давайте посмотрим как всё это будет работать.
class FactoryMethodConceptual: XCTestCase {

    func testFactoryMethodConceptual() {

        /// Приложение выбирает тип создателя в зависимости от конфигурации или
        /// среды.

        print(&quot;App: Launched with the ConcreteCreator1.&quot;)
        Client.someClientCode(creator: ConcreteCreator1())

        print(&quot;\nApp: Launched with the ConcreteCreator2.&quot;)
        Client.someClientCode(creator: ConcreteCreator2())
    }
}</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** Результат выполнения

<figure class="code">
<pre class="code" lang="output"><code>App: Launched with the ConcreteCreator1.
Client: I&#39;m not aware of the creator&#39;s class, but it still works.
Creator: The same creator&#39;s code has just worked with {Result of the ConcreteProduct1}

App: Launched with the ConcreteCreator2.
Client: I&#39;m not aware of the creator&#39;s class, but it still works.
Creator: The same creator&#39;s code has just worked with {Result of the ConcreteProduct2}</code></pre>
</figure>
:::

::: {#example-1 .tab-pane role="tabpanel" aria-labelledby="example-1-tab" style="padding-top: 24px"}
## Пример из реальной жизни {#example-1-title}

#### []{#example-1--Example-swift .anchor} **Example.swift:** Пример из реальной жизни

<figure class="code">
<pre class="code" lang="swift"><code>import XCTest

class FactoryMethodRealWorld: XCTestCase {

    func testFactoryMethodRealWorld() {

        let info = &quot;Very important info of the presentation&quot;

        let clientCode = ClientCode()

        /// Present info over WiFi
        clientCode.present(info: info, with: WifiFactory())

        /// Present info over Bluetooth
        clientCode.present(info: info, with: BluetoothFactory())
    }
}

protocol ProjectorFactory {

    func createProjector() -&gt; Projector

    func syncedProjector(with projector: Projector) -&gt; Projector
}

extension ProjectorFactory {

    /// Base implementation of ProjectorFactory

    func syncedProjector(with projector: Projector) -&gt; Projector {

        /// Every instance creates an own projector
        let newProjector = createProjector()

        /// sync projectors
        newProjector.sync(with: projector)

        return newProjector
    }
}

class WifiFactory: ProjectorFactory {

    func createProjector() -&gt; Projector {
        return WifiProjector()
    }
}

class BluetoothFactory: ProjectorFactory {

    func createProjector() -&gt; Projector {
        return BluetoothProjector()
    }
}

protocol Projector {

    /// Abstract projector interface

    var currentPage: Int { get }

    func present(info: String)

    func sync(with projector: Projector)

    func update(with page: Int)
}

extension Projector {

    /// Base implementation of Projector methods

    func sync(with projector: Projector) {
        projector.update(with: currentPage)
    }
}

class WifiProjector: Projector {

    var currentPage = 0

    func present(info: String) {
        print(&quot;Info is presented over Wifi: \(info)&quot;)
    }

    func update(with page: Int) {
        /// ... scroll page via WiFi connection
        /// ...
        currentPage = page
    }
}

class BluetoothProjector: Projector {

    var currentPage = 0

    func present(info: String) {
        print(&quot;Info is presented over Bluetooth: \(info)&quot;)
    }

    func update(with page: Int) {
        /// ... scroll page via Bluetooth connection
        /// ...
        currentPage = page
    }
}

private class ClientCode {

    private var currentProjector: Projector?

    func present(info: String, with factory: ProjectorFactory) {

        /// Check whether the client code is already presenting something...

        guard let projector = currentProjector else {

            /// &#39;currentProjector&#39; variable is nil. Create a new projector and
            /// start presentation.

            let projector = factory.createProjector()
            projector.present(info: info)
            self.currentProjector = projector
            return
        }

        /// Client code already has a projector. Let&#39;s sync pages of the old
        /// projector with a new one.

        self.currentProjector = factory.syncedProjector(with: projector)
        self.currentProjector?.present(info: info)
    }
}</code></pre>
</figure>

#### []{#example-1--Output-txt .anchor} **Output.txt:** Результат выполнения

<figure class="code">
<pre class="code" lang="output"><code>Info is presented over Wifi: Very important info of the presentation
Info is presented over Bluetooth: Very important info of the presentation</code></pre>
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

[Прототип на Swift []{.fa
.fa-arrow-right}](../../prototype/swift/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Строитель на
Swift](../../builder/swift/example.html){.btn .btn-default rel="prev"}
:::

## **Фабричный метод** на других языках программирования

[![Фабричный метод на
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Фабричный метод на C#"){.prog-lang-link}
[![Фабричный метод на
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Фабричный метод на C++"){.prog-lang-link}
[![Фабричный метод на
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Фабричный метод на Go"){.prog-lang-link}
[![Фабричный метод на
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Фабричный метод на Java"){.prog-lang-link}
[![Фабричный метод на
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Фабричный метод на PHP"){.prog-lang-link}
[![Фабричный метод на
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Фабричный метод на Python"){.prog-lang-link}
[![Фабричный метод на
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Фабричный метод на Ruby"){.prog-lang-link}
[![Фабричный метод на
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Фабричный метод на Rust"){.prog-lang-link}
[![Фабричный метод на
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Фабричный метод на TypeScript"){.prog-lang-link}
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
