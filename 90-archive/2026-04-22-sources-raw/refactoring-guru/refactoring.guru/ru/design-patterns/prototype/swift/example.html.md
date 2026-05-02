::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Паттерны
проектирования](../../../design-patterns.html){.type} /
[Прототип](../../prototype.html){.type} /
[Swift](../../swift.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Прототип](../../../../images/patterns/cards/prototype-mini6540.png?id=bc3046bb39ff36574c08d49839fd1c8e){srcset="/images/patterns/cards/prototype-mini-2x.png?id=b871f789a736e7efbb1cd082d2de6398 2x"}
:::

# **Прототип** на Swift {#прототип-на-swift .pattern-example-title-block-title}
::::

:::::::::::::: pattern-example-body
::: pattern-example-brief
**Прототип** --- это порождающий паттерн, который позволяет копировать
объекты любой сложности без привязки к их конкретным классам.

Все классы---Прототипы имеют общий интерфейс. Поэтому вы можете
копировать объекты, не обращая внимания на их конкретные типы и всегда
быть уверены, что получите точную копию. Клонирование совершается самим
объектом-прототипом, что позволяет ему скопировать значения всех полей,
даже приватных.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Подробней о паттерне Прототип ](../../prototype.html){.btn .btn-lg
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

**Применимость:** Паттерн Прототип реализован в базовой библиотеке Swift
посредством интерфейса `NSCopying`.

**Признаки применения паттерна:** Прототип легко определяется в коде по
наличию методов `clone`, `copy` и прочих.
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

Этот пример показывает структуру паттерна **Прототип**, а именно --- из
каких классов он состоит, какие роли эти классы выполняют и как они
взаимодействуют друг с другом.

После ознакомления со структурой, вам будет легче воспринимать второй
пример, который рассматривает реальный случай использования паттерна в
мире Swift.

#### []{#example-0--Example-swift .anchor} **Example.swift:** Пример структуры паттерна

<figure class="code">
<pre class="code" lang="swift"><code>import XCTest

/// Swift имеет встроенную поддержку клонирования. Чтобы сделать класс
/// клонируемым, вам нужно реализовать в нём протокол NSCopying, а именно метод
/// `copy`.
class BaseClass: NSCopying, Equatable {

    private var intValue = 1
    private var stringValue = &quot;Value&quot;

    required init(intValue: Int = 1, stringValue: String = &quot;Value&quot;) {

        self.intValue = intValue
        self.stringValue = stringValue
    }

    /// MARK: - NSCopying
    func copy(with zone: NSZone? = nil) -&gt; Any {
        let prototype = type(of: self).init()
        prototype.intValue = intValue
        prototype.stringValue = stringValue
        print(&quot;Values defined in BaseClass have been cloned!&quot;)
        return prototype
    }

    /// MARK: - Equatable
    static func == (lhs: BaseClass, rhs: BaseClass) -&gt; Bool {
        return lhs.intValue == rhs.intValue &amp;&amp; lhs.stringValue == rhs.stringValue
    }
}

/// Подклассы могет переопределять базовый метод `copy`, чтобы дополнительно
/// скопировать данные собственного класса. Но в этом случае всегда сперва
/// вызывайте родительскую реализацию метод копирования.
class SubClass: BaseClass {

    private var boolValue = true

    func copy() -&gt; Any {
        return copy(with: nil)
    }

    override func copy(with zone: NSZone?) -&gt; Any {
        guard let prototype = super.copy(with: zone) as? SubClass else {
            return SubClass() // oops
        }
        prototype.boolValue = boolValue
        print(&quot;Values defined in SubClass have been cloned!&quot;)
        return prototype
    }
}

/// Клиентский код.
class Client {
    // ...
    static func someClientCode() {
        let original = SubClass(intValue: 2, stringValue: &quot;Value2&quot;)

        guard let copy = original.copy() as? SubClass else {
            XCTAssert(false)
            return
        }

        /// См. реализацию протокола `Equatable`.
        XCTAssert(copy == original)

        print(&quot;The original object is equal to the copied object!&quot;)
    }
    // ...
}

/// Давайте посмотрим как всё это будет работать.
class PrototypeConceptual: XCTestCase {

    func testPrototype_NSCopying() {
        Client.someClientCode()
    }
}</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** Результат выполнения

<figure class="code">
<pre class="code" lang="output"><code>Values defined in BaseClass have been cloned!
Values defined in SubClass have been cloned!
The original object is equal to the copied object!</code></pre>
</figure>
:::

::: {#example-1 .tab-pane role="tabpanel" aria-labelledby="example-1-tab" style="padding-top: 24px"}
## Пример из реальной жизни {#example-1-title}

#### []{#example-1--Example-swift .anchor} **Example.swift:** Пример из реальной жизни

<figure class="code">
<pre class="code" lang="swift"><code>import XCTest

class PrototypeRealWorld: XCTestCase {

    func testPrototypeRealWorld() {

        let author = Author(id: 10, username: &quot;Ivan_83&quot;)
        let page = Page(title: &quot;My First Page&quot;, contents: &quot;Hello world!&quot;, author: author)

        page.add(comment: Comment(message: &quot;Keep it up!&quot;))

        /// Since NSCopying returns Any, the copied object should be unwrapped.
        guard let anotherPage = page.copy() as? Page else {
            XCTFail(&quot;Page was not copied&quot;)
            return
        }

        /// Comments should be empty as it is a new page.
        XCTAssert(anotherPage.comments.isEmpty)

        /// Note that the author is now referencing two objects.
        XCTAssert(author.pagesCount == 2)

        print(&quot;Original title: &quot; + page.title)
        print(&quot;Copied title: &quot; + anotherPage.title)
        print(&quot;Count of pages: &quot; + String(author.pagesCount))
    }
}

private class Author {

    private var id: Int
    private var username: String
    private var pages = [Page]()

    init(id: Int, username: String) {
        self.id = id
        self.username = username
    }

    func add(page: Page) {
        pages.append(page)
    }

    var pagesCount: Int {
        return pages.count
    }
}

private class Page: NSCopying {

    private(set) var title: String
    private(set) var contents: String
    private weak var author: Author?
    private(set) var comments = [Comment]()

    init(title: String, contents: String, author: Author?) {
        self.title = title
        self.contents = contents
        self.author = author
        author?.add(page: self)
    }

    func add(comment: Comment) {
        comments.append(comment)
    }

    /// MARK: - NSCopying

    func copy(with zone: NSZone? = nil) -&gt; Any {
        return Page(title: &quot;Copy of &#39;&quot; + title + &quot;&#39;&quot;, contents: contents, author: author)
    }
}

private struct Comment {

    let date = Date()
    let message: String
}</code></pre>
</figure>

#### []{#example-1--Output-txt .anchor} **Output.txt:** Результат выполнения

<figure class="code">
<pre class="code" lang="output"><code>Original title: My First Page
Copied title: Copy of &#39;My First Page&#39;
Count of pages: 2</code></pre>
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

[Одиночка на Swift []{.fa
.fa-arrow-right}](../../singleton/swift/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Фабричный метод на
Swift](../../factory-method/swift/example.html){.btn .btn-default
rel="prev"}
:::

## **Прототип** на других языках программирования

[![Прототип на
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Прототип на C#"){.prog-lang-link}
[![Прототип на
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Прототип на C++"){.prog-lang-link}
[![Прототип на
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Прототип на Go"){.prog-lang-link}
[![Прототип на
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Прототип на Java"){.prog-lang-link}
[![Прототип на
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Прототип на PHP"){.prog-lang-link}
[![Прототип на
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Прототип на Python"){.prog-lang-link}
[![Прототип на
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Прототип на Ruby"){.prog-lang-link}
[![Прототип на
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Прототип на Rust"){.prog-lang-link}
[![Прототип на
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Прототип на TypeScript"){.prog-lang-link}
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
