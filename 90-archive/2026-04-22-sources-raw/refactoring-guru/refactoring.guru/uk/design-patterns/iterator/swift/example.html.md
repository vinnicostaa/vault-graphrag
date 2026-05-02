::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Патерни
проектування](../../../design-patterns.html){.type} /
[Ітератор](../../iterator.html){.type} /
[Swift](../../swift.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Ітератор](../../../../images/patterns/cards/iterator-minie2c2.png?id=76c28bb48f997b36965983dd2b41f02e){srcset="/images/patterns/cards/iterator-mini-2x.png?id=7d28f66a487066774a743cfceaa40ea4 2x"}
:::

# **Ітератор** на Swift {#ітератор-на-swift .pattern-example-title-block-title}
::::

:::::::::::::: pattern-example-body
::: pattern-example-brief
**Ітератор** --- це поведінковий патерн, що дозволяє послідовно обходити
складну колекцію, не розкриваючи деталі її реалізації.

Завдяки Ітераторові, клієнт може обходити різні колекції в один і той же
спосіб, використовуючи єдиний інтерфейс ітераторів.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Детальніше про Ітератор ](../../iterator.html){.btn .btn-lg
.btn-primary}
:::

::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
Навігація
:::

::: en-intro
 [Інтро](#)
:::

::: en-intro
 [Концептуальний приклад](#example-0)
:::

::: en-file
 [Example](#example-0--Example-swift)
:::

::: en-file
 [Output](#example-0--Output-txt)
:::

::: en-intro
 [Життєвий приклад](#example-1)
:::

::: en-file
 [Example](#example-1--Example-swift)
:::

::: en-file
 [Output](#example-1--Output-txt)
:::
:::::::::::

**Складність:** []{.fa-stars}

**Популярність:** []{.fa-stars}

**Застосування:** Патерн можна часто зустріти в Swift-коді, особливо в
програмах, що працюють з різними типами колекцій, коли потрібно виконати
обхід різних сутностей.

**Ознаки застосування патерна:** Ітератор легко визначити за методами
навігації (наприклад, отримання наступного/попереднього елементу і т.
д.). Код, який використовує ітератор, часто взагалі не має посилань на
колекцію, з якою працює ітератор. Ітератор або приймає колекцію в
параметрах конструктора під час створення, або повертається до самої
колекцією.
::::::::::::::

::::: {style="padding: 1rem; border: solid 4px #e35e5e; border-radius: 1rem; text-align: center"}
::: {style="margin-bottom: 0.5rem;"}
Наступні приклади доступні на [Swift
Playgrounds](https://www.alemohamad.com/playgrounds){style="text-decoration: underline; display: inline-block"
rel="nofollow"}.
:::

::: {style="font-size: 13px;"}
Вдячність [Alejandro
Mohamad](https://www.alemohamad.com/){style="text-decoration: underline;"
rel="nofollow"} за створення версії Playground.
:::
:::::

::: {#nav-tab .nav .nav-tabs .nav-justified role="tablist"}
[Концептуальний приклад](#example-0){#example-0-tab .nav-item .nav-link
.active toggle="tab" role="tab" aria-controls="example-0"
aria-selected="true"} [Життєвий приклад](#example-1){#example-1-tab
.nav-item .nav-link toggle="tab" role="tab" aria-controls="example-1"
aria-selected="true"}
:::

::::: {#nav-tabContent .pattern-example-tab-content .tab-content}
::: {#example-0 .tab-pane .show .active role="tabpanel" aria-labelledby="example-0-tab" style="padding-top: 24px"}
## Концептуальний приклад {#example-0-title}

Цей приклад показує структуру патерна **Ітератор**, а саме --- з яких
класів він складається, які ролі ці класи виконують і як вони
взаємодіють один з одним.

Після ознайомлення зі структурою, вам буде легше сприймати наступний
приклад, що розглядає реальний випадок використання патерна в світі
Swift.

#### []{#example-0--Example-swift .anchor} **Example.swift:** Приклад структури патерна

<figure class="code">
<pre class="code" lang="swift"><code>import XCTest

/// This is a collection that we&#39;re going to iterate through using an iterator
/// that conforms to IteratorProtocol.
class WordsCollection {

    fileprivate lazy var items = [String]()

    func append(_ item: String) {
        self.items.append(item)
    }
}

extension WordsCollection: Sequence {

    func makeIterator() -&gt; WordsIterator {
        return WordsIterator(self)
    }
}

/// Concrete Iterators implement various traversal algorithms. These classes
/// store the current traversal position at all times.
class WordsIterator: IteratorProtocol {

    private let collection: WordsCollection
    private var index = 0

    init(_ collection: WordsCollection) {
        self.collection = collection
    }

    func next() -&gt; String? {
        defer { index += 1 }
        return index &lt; collection.items.count ? collection.items[index] : nil
    }
}


/// This is another collection that we&#39;ll provide AnyIterator for traversing its
/// items.
class NumbersCollection {

    fileprivate lazy var items = [Int]()

    func append(_ item: Int) {
        self.items.append(item)
    }
}

extension NumbersCollection: Sequence {

    func makeIterator() -&gt; AnyIterator&lt;Int&gt; {
        var index = self.items.count - 1

        return AnyIterator {
            defer { index -= 1 }
            return index &gt;= 0 ? self.items[index] : nil
        }
    }
}

/// Client does not know the internal representation of a given sequence.
class Client {
    // ...
    static func clientCode&lt;S: Sequence&gt;(sequence: S) {
        for item in sequence {
            print(item)
        }
    }
    // ...
}

/// Let&#39;s see how it all works together.
class IteratorConceptual: XCTestCase {

    func testIteratorProtocol() {

        let words = WordsCollection()
        words.append(&quot;First&quot;)
        words.append(&quot;Second&quot;)
        words.append(&quot;Third&quot;)

        print(&quot;Straight traversal using IteratorProtocol:&quot;)
        Client.clientCode(sequence: words)
    }

    func testAnyIterator() {

        let numbers = NumbersCollection()
        numbers.append(1)
        numbers.append(2)
        numbers.append(3)

        print(&quot;\nReverse traversal using AnyIterator:&quot;)
        Client.clientCode(sequence: numbers)
    }
}</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** Результат виконання

<figure class="code">
<pre class="code" lang="output"><code>Straight traversal using IteratorProtocol:
First
Second
Third

Reverse traversal using AnyIterator:
3
2
1</code></pre>
</figure>
:::

::: {#example-1 .tab-pane role="tabpanel" aria-labelledby="example-1-tab" style="padding-top: 24px"}
## Життєвий приклад {#example-1-title}

#### []{#example-1--Example-swift .anchor} **Example.swift:** Життєвий приклад

<figure class="code">
<pre class="code" lang="swift"><code>import XCTest

class IteratorRealWorld: XCTestCase {

    func test() {

        let tree = Tree(1)
        tree.left = Tree(2)
        tree.right = Tree(3)

        print(&quot;Tree traversal: Inorder&quot;)
        clientCode(iterator: tree.iterator(.inOrder))

        print(&quot;\nTree traversal: Preorder&quot;)
        clientCode(iterator: tree.iterator(.preOrder))

        print(&quot;\nTree traversal: Postorder&quot;)
        clientCode(iterator: tree.iterator(.postOrder))
    }

    func clientCode&lt;T&gt;(iterator: AnyIterator&lt;T&gt;) {
        while case let item? = iterator.next() {
            print(item)
        }
    }
}

class Tree&lt;T&gt; {

    var value: T
    var left: Tree&lt;T&gt;?
    var right: Tree&lt;T&gt;?

    init(_ value: T) {
        self.value = value
    }

    typealias Block = (T) -&gt; ()

    enum IterationType {
        case inOrder
        case preOrder
        case postOrder
    }

    func iterator(_ type: IterationType) -&gt; AnyIterator&lt;T&gt; {
        var items = [T]()
        switch type {
        case .inOrder:
            inOrder { items.append($0) }
        case .preOrder:
            preOrder { items.append($0) }
        case .postOrder:
            postOrder { items.append($0) }
        }

        /// Note:
        /// AnyIterator is used to hide the type signature of an internal
        /// iterator.
        return AnyIterator(items.makeIterator())
    }

    private func inOrder(_ body: Block) {
        left?.inOrder(body)
        body(value)
        right?.inOrder(body)
    }

    private func preOrder(_ body: Block) {
        body(value)
        left?.preOrder(body)
        right?.preOrder(body)
    }

    private func postOrder(_ body: Block) {
        left?.postOrder(body)
        right?.postOrder(body)
        body(value)
    }
}</code></pre>
</figure>

#### []{#example-1--Output-txt .anchor} **Output.txt:** Результат виконання

<figure class="code">
<pre class="code" lang="output"><code>Tree traversal: Inorder
2
1
3

Tree traversal: Preorder
1
2
3

Tree traversal: Postorder
2
3
1</code></pre>
</figure>
:::
:::::

::: {#nav-tab .nav .nav-tabs .nav-tabs-bottom .nav-justified role="tablist"}
[Концептуальний приклад](#example-0){#example-0-tab-bottom .nav-item
.nav-link .active toggle="tab" role="tab" aria-controls="example-0"
aria-selected="true"} [Життєвий
приклад](#example-1){#example-1-tab-bottom .nav-item .nav-link
toggle="tab" role="tab" aria-controls="example-1" aria-selected="true"}
:::

::: next
#### Читаємо далі

[Посередник на Swift []{.fa
.fa-arrow-right}](../../mediator/swift/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Повернутись назад

[[]{.fa .fa-arrow-left} Команда на
Swift](../../command/swift/example.html){.btn .btn-default rel="prev"}
:::

## **Ітератор** іншими мовами програмування

[![Ітератор на
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Ітератор на C#"){.prog-lang-link}
[![Ітератор на
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Ітератор на C++"){.prog-lang-link}
[![Ітератор на
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Ітератор на Go"){.prog-lang-link}
[![Ітератор на
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Ітератор на Java"){.prog-lang-link}
[![Ітератор на
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Ітератор на PHP"){.prog-lang-link}
[![Ітератор на
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Ітератор на Python"){.prog-lang-link}
[![Ітератор на
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Ітератор на Ruby"){.prog-lang-link}
[![Ітератор на
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Ітератор на Rust"){.prog-lang-link}
[![Ітератор на
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Ітератор на TypeScript"){.prog-lang-link}
::::::::::::::::::::::::::::

::::::: {.prom .banner-sidebar .banner .banner-striped .banner-examples .banner-removable .banner-removable-patterns data-id="DP: Examples-IDE" creative-id="examples-sidebar-uk" position="sidebar"}
:::::: {.banner-inner style="font-size: 14px; line-height: 21px;"}
::: banner-examples-img
[![](../../../../images/patterns/banners/examples-ide91ca.png?id=3115b4b548fb96b75974e2de8f4f49bc){width="215"
height="158" loading="lazy"
srcset="/images/patterns/banners/examples-ide-2x.png?id=93c007a6d157b97c28eb213f59d716ae 2x"}](../../book.html)
:::

::: banner-examples-fade
[ Архів з прикладами](../../book.html){.btn .btn-secondary .btn-block}
:::

::: {.banner-examples-text style="margin-top: 1rem;"}
Придбай книгу **Занурення в Патерни** та отримай архів з десятками
детальних прикладів, які можна відкривати прямо в IDE.

[ Дізнатися більше...](../../book.html){.btn .btn-secondary .btn-block}
:::
::::::
:::::::
::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::
