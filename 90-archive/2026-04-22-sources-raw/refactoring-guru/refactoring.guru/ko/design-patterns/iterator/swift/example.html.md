:::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [디자인
패턴들](../../../design-patterns.html){.type} /
[반복자](../../iterator.html){.type} /
[스위프트](../../swift.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![반복자](../../../../images/patterns/cards/iterator-minie2c2.png?id=76c28bb48f997b36965983dd2b41f02e){srcset="/images/patterns/cards/iterator-mini-2x.png?id=7d28f66a487066774a743cfceaa40ea4 2x"}
:::

# 스위프트로 작성된 **반복자** {#스위프트로-작성된-반복자 .pattern-example-title-block-title}
::::

:::::::::::::: pattern-example-body
::: pattern-example-brief
**반복자**는 복잡한 데이터 구조의 내부 세부 정보를 노출하지 않고 해당
구조를 차례대로 순회할 수 있도록 하는 행동 디자인 패턴입니다.

반복자 덕분에 클라이언트들은 단일 반복기 인터페이스를 사용하여 유사한
방식으로 다른 컬렉션들의 요소들을 탐색할 수 있습니다.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ 반복자에 대하여 더 자세히 알아보세요 ](../../iterator.html){.btn
.btn-lg .btn-primary}
:::

::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
내비게이션
:::

::: en-intro
 [소개](#)
:::

::: en-intro
 [개념적인 예시](#example-0)
:::

::: en-file
 [Example](#example-0--Example-swift)
:::

::: en-file
 [Output](#example-0--Output-txt)
:::

::: en-intro
 [실제 사례 예시](#example-1)
:::

::: en-file
 [Example](#example-1--Example-swift)
:::

::: en-file
 [Output](#example-1--Output-txt)
:::
:::::::::::

**복잡도:** []{.fa-stars}

**인기도:** []{.fa-stars}

**사용 예시들:** 이 패턴은 스위프트 코드에 자주 사용됩니다. 많은
프레임워크들과 라이브러리들은 이 패턴을 컬렉션을 순회하는 표준 방법을
제공하기 위해 사용합니다.

**식별법:** 반복자는 그의 탐색 메서드들​(예: `next`, `previous` 등)​로
쉽게 인식할 수 있습니다. 또 반복자를 사용하는 클라이언트 코드는 반복자가
순회하는 컬렉션을 직접 접근하지 못할 수도 있습니다.
::::::::::::::

::: {#nav-tab .nav .nav-tabs .nav-justified role="tablist"}
[개념적인 예시](#example-0){#example-0-tab .nav-item .nav-link .active
toggle="tab" role="tab" aria-controls="example-0" aria-selected="true"}
[실제 사례 예시](#example-1){#example-1-tab .nav-item .nav-link
toggle="tab" role="tab" aria-controls="example-1" aria-selected="true"}
:::

::::: {#nav-tabContent .pattern-example-tab-content .tab-content}
::: {#example-0 .tab-pane .show .active role="tabpanel" aria-labelledby="example-0-tab" style="padding-top: 24px"}
## 개념적인 예시 {#example-0-title}

이 예시는 **반복자** 디자인 패턴의 구조를 보여주고 다음 질문에 중점을
둡니다:

-   패턴은 어떤 클래스들로 구성되어 있나요?
-   이 클래스들은 어떤 역할을 하나요?
-   패턴의 요소들은 어떻게 서로 연관되어 있나요?

이 패턴의 구조를 배우면 실제 스위프트 사용 사례를 기반으로 하는 다음
예시를 더욱 쉽게 이해할 수 있을 것입니다.

#### []{#example-0--Example-swift .anchor} **Example.swift:** 개념적인 예시

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

#### []{#example-0--Output-txt .anchor} **Output.txt:** 실행 결과

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
## 실제 사례 예시 {#example-1-title}

#### []{#example-1--Example-swift .anchor} **Example.swift:** 실제 사례 예시

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

#### []{#example-1--Output-txt .anchor} **Output.txt:** 실행 결과

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
[개념적인 예시](#example-0){#example-0-tab-bottom .nav-item .nav-link
.active toggle="tab" role="tab" aria-controls="example-0"
aria-selected="true"} [실제 사례 예시](#example-1){#example-1-tab-bottom
.nav-item .nav-link toggle="tab" role="tab" aria-controls="example-1"
aria-selected="true"}
:::

::: next
#### 다음을 보세요

[스위프트로 작성된 중재자 []{.fa
.fa-arrow-right}](../../mediator/swift/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### 돌아가기

[[]{.fa .fa-arrow-left} 스위프트로 작성된
커맨드](../../command/swift/example.html){.btn .btn-default rel="prev"}
:::

## 다른 언어로 작성된 **반복자**

[![C#으로 작성된
반복자](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "C#으로 작성된 반복자"){.prog-lang-link}
[![C++로 작성된
반복자](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "C++로 작성된 반복자"){.prog-lang-link}
[![Go로 작성된
반복자](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Go로 작성된 반복자"){.prog-lang-link}
[![자바로 작성된
반복자](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "자바로 작성된 반복자"){.prog-lang-link}
[![PHP로 작성된
반복자](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "PHP로 작성된 반복자"){.prog-lang-link}
[![파이썬으로 작성된
반복자](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "파이썬으로 작성된 반복자"){.prog-lang-link}
[![루비로 작성된
반복자](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "루비로 작성된 반복자"){.prog-lang-link}
[![러스트로 작성된
반복자](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "러스트로 작성된 반복자"){.prog-lang-link}
[![타입스크립트로 작성된
반복자](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "타입스크립트로 작성된 반복자"){.prog-lang-link}
:::::::::::::::::::::::::

::::::: {.prom .banner-sidebar .banner .banner-striped .banner-examples .banner-removable .banner-removable-patterns data-id="DP: Examples-IDE" creative-id="examples-sidebar-ko" position="sidebar"}
:::::: {.banner-inner style="font-size: 14px; line-height: 21px;"}
::: banner-examples-img
[![](../../../../images/patterns/banners/examples-ide91ca.png?id=3115b4b548fb96b75974e2de8f4f49bc){width="215"
height="158" loading="lazy"
srcset="/images/patterns/banners/examples-ide-2x.png?id=93c007a6d157b97c28eb213f59d716ae 2x"}](../../book.html)
:::

::: banner-examples-fade
[ 예시들을 포함한 저장소](../../book.html){.btn .btn-secondary
.btn-block}
:::

::: {.banner-examples-text style="margin-top: 1rem;"}
전자책 **디자인 패턴에 뛰어들기**를 구매하면 IDE에서 바로 열 수 있는
수십 가지의 상세한 코드 예시들이 포함된 저장소를 사용할 수 있습니다.

[ 더 알아보기...](../../book.html){.btn .btn-secondary .btn-block}
:::
::::::
:::::::
:::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::
