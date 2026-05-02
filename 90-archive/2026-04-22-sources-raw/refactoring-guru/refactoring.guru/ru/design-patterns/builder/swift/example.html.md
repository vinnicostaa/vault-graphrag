::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Паттерны
проектирования](../../../design-patterns.html){.type} /
[Строитель](../../builder.html){.type} /
[Swift](../../swift.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Строитель](../../../../images/patterns/cards/builder-mini6d24.png?id=19b95fd05e6469679752c0554b116815){srcset="/images/patterns/cards/builder-mini-2x.png?id=de6d0938678b86903a1426dddfdd13bf 2x"}
:::

# **Строитель** на Swift {#строитель-на-swift .pattern-example-title-block-title}
::::

:::::::::::::: pattern-example-body
::: pattern-example-brief
**Строитель** --- это порождающий паттерн проектирования, который
позволяет создавать объекты пошагово.

В отличие от других порождающих паттернов, Строитель позволяет
производить различные продукты, используя один и тот же процесс
строительства.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Подробней о паттерне Строитель ](../../builder.html){.btn .btn-lg
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

**Применимость:** Паттерн можно часто встретить в Swift-коде, особенно
там, где требуется пошаговое создание продуктов или конфигурация сложных
объектов.

**Признаки применения паттерна:** Строителя можно узнать в классе,
который имеет один создающий метод и несколько методов настройки
создаваемого продукта. Обычно, методы настройки вызывают для удобства
цепочкой (например, `someBuilder.setValueA(1).setValueB(2).create()`).
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

Этот пример показывает структуру паттерна **Строитель**, а именно --- из
каких классов он состоит, какие роли эти классы выполняют и как они
взаимодействуют друг с другом.

После ознакомления со структурой, вам будет легче воспринимать второй
пример, который рассматривает реальный случай использования паттерна в
мире Swift.

#### []{#example-0--Example-swift .anchor} **Example.swift:** Пример структуры паттерна

<figure class="code">
<pre class="code" lang="swift"><code>import XCTest

/// Интерфейс Строителя объявляет создающие методы для различных частей объектов
/// Продуктов.
protocol Builder {

    func producePartA()
    func producePartB()
    func producePartC()
}

/// Классы Конкретного Строителя следуют интерфейсу Строителя и предоставляют
/// конкретные реализации шагов построения. Ваша программа может иметь несколько
/// вариантов Строителей, реализованных по-разному.
class ConcreteBuilder1: Builder {

    /// Новый экземпляр строителя должен содержать пустой объект продукта,
    /// который используется в дальнейшей сборке.
    private var product = Product1()

    func reset() {
        product = Product1()
    }

    /// Все этапы производства работают с одним и тем же экземпляром продукта.
    func producePartA() {
        product.add(part: &quot;PartA1&quot;)
    }

    func producePartB() {
        product.add(part: &quot;PartB1&quot;)
    }

    func producePartC() {
        product.add(part: &quot;PartC1&quot;)
    }

    /// Конкретные Строители должны предоставить свои собственные методы
    /// получения результатов. Это связано с тем, что различные типы строителей
    /// могут создавать совершенно разные продукты с разными интерфейсами.
    /// Поэтому такие методы не могут быть объявлены в базовом интерфейсе
    /// Строителя (по крайней мере, в статически типизированном языке
    /// программирования).
    ///
    /// Как правило, после возвращения конечного результата клиенту, экземпляр
    /// строителя должен быть готов к началу производства следующего продукта.
    /// Поэтому обычной практикой является вызов метода сброса в конце тела
    /// метода getProduct. Однако такое поведение не является обязательным, вы
    /// можете заставить своих строителей ждать явного запроса на сброс из кода
    /// клиента, прежде чем избавиться от предыдущего результата.
    func retrieveProduct() -&gt; Product1 {
        let result = self.product
        reset()
        return result
    }
}

/// Директор отвечает только за выполнение шагов построения в определённой
/// последовательности. Это полезно при производстве продуктов в определённом
/// порядке или особой конфигурации. Строго говоря, класс Директор необязателен,
/// так как клиент может напрямую управлять строителями.
class Director {

    private var builder: Builder?

    /// Директор работает с любым экземпляром строителя, который передаётся ему
    /// клиентским кодом. Таким образом, клиентский код может изменить конечный
    /// тип вновь собираемого продукта.
    func update(builder: Builder) {
        self.builder = builder
    }

    /// Директор может строить несколько вариаций продукта, используя одинаковые
    /// шаги построения.
    func buildMinimalViableProduct() {
        builder?.producePartA()
    }

    func buildFullFeaturedProduct() {
        builder?.producePartA()
        builder?.producePartB()
        builder?.producePartC()
    }
}

/// Имеет смысл использовать паттерн Строитель только тогда, когда ваши продукты
/// достаточно сложны и требуют обширной конфигурации.
///
/// В отличие от других порождающих паттернов, различные конкретные строители
/// могут производить несвязанные продукты. Другими словами, результаты
/// различных строителей могут не всегда следовать одному и тому же интерфейсу.
class Product1 {

    private var parts = [String]()

    func add(part: String) {
        self.parts.append(part)
    }

    func listParts() -&gt; String {
        return &quot;Product parts: &quot; + parts.joined(separator: &quot;, &quot;) + &quot;\n&quot;
    }
}

/// Клиентский код создаёт объект-строитель, передаёт его директору, а затем
/// инициирует процесс построения. Конечный результат извлекается из объекта-
/// строителя.
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

        // Помните, что паттерн Строитель можно использовать без класса
        // Директор.
        print(&quot;Custom product:&quot;)
        builder.producePartA()
        builder.producePartC()
        print(builder.retrieveProduct().listParts())
    }
    // ...
}

/// Давайте посмотрим как всё это будет работать.
class BuilderConceptual: XCTestCase {

    func testBuilderConceptual() {
        let director = Director()
        Client.someClientCode(director: director)
    }
}</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** Результат выполнения

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
## Пример из реальной жизни {#example-1-title}

#### []{#example-1--Example-swift .anchor} **Example.swift:** Пример из реальной жизни

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

#### []{#example-1--Output-txt .anchor} **Output.txt:** Результат выполнения

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
[Концептуальный пример](#example-0){#example-0-tab-bottom .nav-item
.nav-link .active toggle="tab" role="tab" aria-controls="example-0"
aria-selected="true"} [Пример из реальной
жизни](#example-1){#example-1-tab-bottom .nav-item .nav-link
toggle="tab" role="tab" aria-controls="example-1" aria-selected="true"}
:::

::: next
#### Читаем дальше

[Фабричный метод на Swift []{.fa
.fa-arrow-right}](../../factory-method/swift/example.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Абстрактная фабрика на
Swift](../../abstract-factory/swift/example.html){.btn .btn-default
rel="prev"}
:::

## **Строитель** на других языках программирования

[![Строитель на
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Строитель на C#"){.prog-lang-link}
[![Строитель на
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Строитель на C++"){.prog-lang-link}
[![Строитель на
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Строитель на Go"){.prog-lang-link}
[![Строитель на
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Строитель на Java"){.prog-lang-link}
[![Строитель на
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Строитель на PHP"){.prog-lang-link}
[![Строитель на
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Строитель на Python"){.prog-lang-link}
[![Строитель на
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Строитель на Ruby"){.prog-lang-link}
[![Строитель на
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Строитель на Rust"){.prog-lang-link}
[![Строитель на
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Строитель на TypeScript"){.prog-lang-link}
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
