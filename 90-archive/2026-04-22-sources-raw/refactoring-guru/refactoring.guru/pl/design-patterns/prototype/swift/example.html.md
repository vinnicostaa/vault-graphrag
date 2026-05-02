::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Wzorce
projektowe](../../../design-patterns.html){.type} /
[Prototyp](../../prototype.html){.type} /
[Swift](../../swift.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Prototyp](../../../../images/patterns/cards/prototype-mini6540.png?id=bc3046bb39ff36574c08d49839fd1c8e){srcset="/images/patterns/cards/prototype-mini-2x.png?id=b871f789a736e7efbb1cd082d2de6398 2x"}
:::

# **Prototyp** w języku Swift {#prototyp-w-języku-swift .pattern-example-title-block-title}
::::

:::::::::::::: pattern-example-body
::: pattern-example-brief
**Prototyp** To kreacyjny wzorzec projektowy pozwalający klonować
obiekty --- również te złożone --- bez konieczności sprzęgania z ich
klasami.

Wszystkie klasy prototyp powinny mieć wspólny interfejs który pozwoli
kopiować ich obiekty nawet gdy nie zna się ich konkretnych klas.
Obiekty-prototypy mogą tworzyć kompletne kopie, ponieważ pola prywatne
danej klasy są dostępne dla innych obiektów tej samej klasy.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Dowiedz się więcej o Prototyp ](../../prototype.html){.btn .btn-lg
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

**Przykłady użycia:** Wzorzec Prototyp jest dostępny w Swift od razu ---
dzięki interfejsowi `NSCopying`.

**Identyfikacja:** Prototyp można łatwo poznać dzięki obecności metod
`clone` lub `copy`, itp.
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

Poniższy przykład ilustruje strukturę wzorca **Prototyp** ze szczególnym
naciskiem na następujące kwestie:

-   Z jakich składa się klas?
-   Jakie role pełnią te klasy?
-   W jaki sposób elementy wzorca są ze sobą powiązane?

Poznawszy strukturę wzorca będzie ci łatwiej zrozumieć następujący
przykład, oparty na prawdziwym przypadku użycia Swift.

#### []{#example-0--Example-swift .anchor} **Example.swift:** Przykład koncepcyjny

<figure class="code">
<pre class="code" lang="swift"><code>import XCTest

/// Swift has built-in cloning support. To add cloning support to your class,
/// you need to implement the NSCopying protocol in that class and provide the
/// implementation for the `copy` method.
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

/// Subclasses can override the base `copy` method to copy their own data into
/// the resulting object. But you should always call the base method first.
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

/// The client code.
class Client {
    // ...
    static func someClientCode() {
        let original = SubClass(intValue: 2, stringValue: &quot;Value2&quot;)

        guard let copy = original.copy() as? SubClass else {
            XCTAssert(false)
            return
        }

        /// See implementation of `Equatable` protocol for more details.
        XCTAssert(copy == original)

        print(&quot;The original object is equal to the copied object!&quot;)
    }
    // ...
}

/// Let&#39;s see how it all works together.
class PrototypeConceptual: XCTestCase {

    func testPrototype_NSCopying() {
        Client.someClientCode()
    }
}</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** Wynik działania

<figure class="code">
<pre class="code" lang="output"><code>Values defined in BaseClass have been cloned!
Values defined in SubClass have been cloned!
The original object is equal to the copied object!</code></pre>
</figure>
:::

::: {#example-1 .tab-pane role="tabpanel" aria-labelledby="example-1-tab" style="padding-top: 24px"}
## Przykład z prawdziwego życia {#example-1-title}

#### []{#example-1--Example-swift .anchor} **Example.swift:** Przykład z prawdziwego życia

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

#### []{#example-1--Output-txt .anchor} **Output.txt:** Wynik działania

<figure class="code">
<pre class="code" lang="output"><code>Original title: My First Page
Copied title: Copy of &#39;My First Page&#39;
Count of pages: 2</code></pre>
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

[Singleton w języku Swift []{.fa
.fa-arrow-right}](../../singleton/swift/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Wróć

[[]{.fa .fa-arrow-left} Metoda wytwórcza w języku
Swift](../../factory-method/swift/example.html){.btn .btn-default
rel="prev"}
:::

## **Prototyp** w innych językach

[![Prototyp w języku
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Prototyp w języku C#"){.prog-lang-link}
[![Prototyp w języku
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Prototyp w języku C++"){.prog-lang-link}
[![Prototyp w języku
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Prototyp w języku Go"){.prog-lang-link}
[![Prototyp w języku
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Prototyp w języku Java"){.prog-lang-link}
[![Prototyp w języku
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Prototyp w języku PHP"){.prog-lang-link}
[![Prototyp w języku
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Prototyp w języku Python"){.prog-lang-link}
[![Prototyp w języku
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Prototyp w języku Ruby"){.prog-lang-link}
[![Prototyp w języku
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Prototyp w języku Rust"){.prog-lang-link}
[![Prototyp w języku
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Prototyp w języku TypeScript"){.prog-lang-link}
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
