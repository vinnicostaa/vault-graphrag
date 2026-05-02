::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Паттерны
проектирования](../../../design-patterns.html){.type} /
[Стратегия](../../strategy.html){.type} /
[Swift](../../swift.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Стратегия](../../../../images/patterns/cards/strategy-mini28f6.png?id=d38abee4fb6f2aed909d262bdadca936){srcset="/images/patterns/cards/strategy-mini-2x.png?id=f4e6608561f8e5d18be6927d4620ad29 2x"}
:::

# **Стратегия** на Swift {#стратегия-на-swift .pattern-example-title-block-title}
::::

:::::::::::::: pattern-example-body
::: pattern-example-brief
**Стратегия** --- это поведенческий паттерн, выносит набор алгоритмов в
собственные классы и делает их взаимозаменимыми.

Другие объекты содержат ссылку на объект-стратегию и делегируют ей
работу. Программа может подменить этот объект другим, если требуется
иной способ решения задачи.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Подробней о паттерне Стратегия ](../../strategy.html){.btn .btn-lg
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

**Применимость:** Стратегия часто используется в Swift-коде, особенно
там, где нужно подменять алгоритм во время выполнения программы. Многие
примеры стратегии можно заменить простыми lambda-выражениями.

**Признаки применения паттерна:** Класс делегирует выполнение вложенному
объекту абстрактного типа или интерфейса.
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

Этот пример показывает структуру паттерна **Стратегия**, а именно --- из
каких классов он состоит, какие роли эти классы выполняют и как они
взаимодействуют друг с другом.

После ознакомления со структурой, вам будет легче воспринимать второй
пример, который рассматривает реальный случай использования паттерна в
мире Swift.

#### []{#example-0--Example-swift .anchor} **Example.swift:** Пример структуры паттерна

<figure class="code">
<pre class="code" lang="swift"><code>import XCTest

/// Контекст определяет интерфейс, представляющий интерес для клиентов.
class Context {

    /// Контекст хранит ссылку на один из объектов Стратегии. Контекст не знает
    /// конкретного класса стратегии. Он должен работать со всеми стратегиями
    /// через интерфейс Стратегии.
    private var strategy: Strategy

    /// Обычно Контекст принимает стратегию через конструктор, а также
    /// предоставляет сеттер для её изменения во время выполнения.
    init(strategy: Strategy) {
        self.strategy = strategy
    }

    /// Обычно Контекст позволяет заменить объект Стратегии во время выполнения.
    func update(strategy: Strategy) {
        self.strategy = strategy
    }

    /// Вместо того, чтобы самостоятельно реализовывать множественные версии
    /// алгоритма, Контекст делегирует некоторую работу объекту Стратегии.
    func doSomeBusinessLogic() {
        print(&quot;Context: Sorting data using the strategy (not sure how it&#39;ll do it)\n&quot;)

        let result = strategy.doAlgorithm([&quot;a&quot;, &quot;b&quot;, &quot;c&quot;, &quot;d&quot;, &quot;e&quot;])
        print(result.joined(separator: &quot;,&quot;))
    }
}

/// Интерфейс Стратегии объявляет операции, общие для всех поддерживаемых версий
/// некоторого алгоритма.
///
/// Контекст использует этот интерфейс для вызова алгоритма, определённого
/// Конкретными Стратегиями.
protocol Strategy {

    func doAlgorithm&lt;T: Comparable&gt;(_ data: [T]) -&gt; [T]
}

/// Конкретные Стратегии реализуют алгоритм, следуя базовому интерфейсу
/// Стратегии. Этот интерфейс делает их взаимозаменяемыми в Контексте.
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

/// Давайте посмотрим как всё это будет работать.
class StrategyConceptual: XCTestCase {

    func test() {

        /// Клиентский код выбирает конкретную стратегию и передаёт её в
        /// контекст. Клиент должен знать о различиях между стратегиями, чтобы
        /// сделать правильный выбор.

        let context = Context(strategy: ConcreteStrategyA())
        print(&quot;Client: Strategy is set to normal sorting.\n&quot;)
        context.doSomeBusinessLogic()

        print(&quot;\nClient: Strategy is set to reverse sorting.\n&quot;)
        context.update(strategy: ConcreteStrategyB())
        context.doSomeBusinessLogic()
    }
}</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** Результат выполнения

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
## Пример из реальной жизни {#example-1-title}

#### []{#example-1--Example-swift .anchor} **Example.swift:** Пример из реальной жизни

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

#### []{#example-1--Output-txt .anchor} **Output.txt:** Результат выполнения

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
[Концептуальный пример](#example-0){#example-0-tab-bottom .nav-item
.nav-link .active toggle="tab" role="tab" aria-controls="example-0"
aria-selected="true"} [Пример из реальной
жизни](#example-1){#example-1-tab-bottom .nav-item .nav-link
toggle="tab" role="tab" aria-controls="example-1" aria-selected="true"}
:::

::: next
#### Читаем дальше

[Шаблонный метод на Swift []{.fa
.fa-arrow-right}](../../template-method/swift/example.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Состояние на
Swift](../../state/swift/example.html){.btn .btn-default rel="prev"}
:::

## **Стратегия** на других языках программирования

[![Стратегия на
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Стратегия на C#"){.prog-lang-link}
[![Стратегия на
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Стратегия на C++"){.prog-lang-link}
[![Стратегия на
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Стратегия на Go"){.prog-lang-link}
[![Стратегия на
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Стратегия на Java"){.prog-lang-link}
[![Стратегия на
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Стратегия на PHP"){.prog-lang-link}
[![Стратегия на
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Стратегия на Python"){.prog-lang-link}
[![Стратегия на
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Стратегия на Ruby"){.prog-lang-link}
[![Стратегия на
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Стратегия на Rust"){.prog-lang-link}
[![Стратегия на
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Стратегия на TypeScript"){.prog-lang-link}
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
