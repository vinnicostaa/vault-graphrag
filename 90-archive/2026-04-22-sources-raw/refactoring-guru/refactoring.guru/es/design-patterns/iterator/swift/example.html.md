::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Patrones de
diseño](../../../design-patterns.html){.type} /
[Iterator](../../iterator.html){.type} /
[Swift](../../swift.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Iterator](../../../../images/patterns/cards/iterator-minie2c2.png?id=76c28bb48f997b36965983dd2b41f02e){srcset="/images/patterns/cards/iterator-mini-2x.png?id=7d28f66a487066774a743cfceaa40ea4 2x"}
:::

# **Iterator** en Swift {#iterator-en-swift .pattern-example-title-block-title}
::::

:::::::::::::: pattern-example-body
::: pattern-example-brief
**Iterator** es un patrón de diseño de comportamiento que permite el
recorrido secuencial por una estructura de datos compleja sin exponer
sus detalles internos.

Gracias al patrón Iterator, los clientes pueden recorrer elementos de
colecciones diferentes de un modo similar, utilizando una única interfaz
iteradora.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Aprende más sobre el patrón Iterator ](../../iterator.html){.btn
.btn-lg .btn-primary}
:::

::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
Navegación
:::

::: en-intro
 [Intro](#)
:::

::: en-intro
 [Ejemplo conceptual](#example-0)
:::

::: en-file
 [Example](#example-0--Example-swift)
:::

::: en-file
 [Output](#example-0--Output-txt)
:::

::: en-intro
 [Ejemplo del mundo real](#example-1)
:::

::: en-file
 [Example](#example-1--Example-swift)
:::

::: en-file
 [Output](#example-1--Output-txt)
:::
:::::::::::

**Complejidad:** []{.fa-stars}

**Popularidad:** []{.fa-stars}

**Ejemplos de uso:** El patrón es muy común en el código Swift. Muchos
*frameworks* y bibliotecas lo utilizan para proporcionar una forma
estandarizada de recorrer sus colecciones.

**Identificación:** El patrón Iterator es fácil de reconocer por sus
métodos de navegación (como `next`, `previous` y otros). El código
cliente que utiliza iteradores puede no tener acceso directo a la
colección recorrida.
::::::::::::::

::::: {style="padding: 1rem; border: solid 4px #e35e5e; border-radius: 1rem; text-align: center"}
::: {style="margin-bottom: 0.5rem;"}
Los siguientes ejemplos están disponibles en [Swift
Playgrounds](https://www.alemohamad.com/playgrounds){style="text-decoration: underline; display: inline-block"
rel="nofollow"}.
:::

::: {style="font-size: 13px;"}
Kudos a [Alejandro
Mohamad](https://www.alemohamad.com/){style="text-decoration: underline;"
rel="nofollow"} por crear la versión de Playground.
:::
:::::

::: {#nav-tab .nav .nav-tabs .nav-justified role="tablist"}
[Ejemplo conceptual](#example-0){#example-0-tab .nav-item .nav-link
.active toggle="tab" role="tab" aria-controls="example-0"
aria-selected="true"} [Ejemplo del mundo
real](#example-1){#example-1-tab .nav-item .nav-link toggle="tab"
role="tab" aria-controls="example-1" aria-selected="true"}
:::

::::: {#nav-tabContent .pattern-example-tab-content .tab-content}
::: {#example-0 .tab-pane .show .active role="tabpanel" aria-labelledby="example-0-tab" style="padding-top: 24px"}
## Ejemplo conceptual {#example-0-title}

Este ejemplo ilustra la estructura del patrón de diseño **Iterator** y
se centra en las siguientes preguntas:

-   ¿De qué clases se compone?
-   ¿Qué papeles juegan esas clases?
-   ¿De qué forma se relacionan los elementos del patrón?

Después de conocer la estructura del patrón, será más fácil comprender
el siguiente ejemplo basado en un caso de uso real de Swift.

#### []{#example-0--Example-swift .anchor} **Example.swift:** Ejemplo conceptual

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

#### []{#example-0--Output-txt .anchor} **Output.txt:** Resultado de la ejecución

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
## Ejemplo del mundo real {#example-1-title}

#### []{#example-1--Example-swift .anchor} **Example.swift:** Ejemplo del mundo real

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

#### []{#example-1--Output-txt .anchor} **Output.txt:** Resultado de la ejecución

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
[Ejemplo conceptual](#example-0){#example-0-tab-bottom .nav-item
.nav-link .active toggle="tab" role="tab" aria-controls="example-0"
aria-selected="true"} [Ejemplo del mundo
real](#example-1){#example-1-tab-bottom .nav-item .nav-link toggle="tab"
role="tab" aria-controls="example-1" aria-selected="true"}
:::

::: next
#### Leer siguiente

[Mediator en Swift []{.fa
.fa-arrow-right}](../../mediator/swift/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Volver

[[]{.fa .fa-arrow-left} Command en
Swift](../../command/swift/example.html){.btn .btn-default rel="prev"}
:::

## **Iterator** en otros lenguajes

[![Iterator en
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Iterator en C#"){.prog-lang-link}
[![Iterator en
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Iterator en C++"){.prog-lang-link}
[![Iterator en
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Iterator en Go"){.prog-lang-link}
[![Iterator en
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Iterator en Java"){.prog-lang-link}
[![Iterator en
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Iterator en PHP"){.prog-lang-link}
[![Iterator en
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Iterator en Python"){.prog-lang-link}
[![Iterator en
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Iterator en Ruby"){.prog-lang-link}
[![Iterator en
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Iterator en Rust"){.prog-lang-link}
[![Iterator en
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Iterator en TypeScript"){.prog-lang-link}
::::::::::::::::::::::::::::

::::::: {.prom .banner-sidebar .banner .banner-striped .banner-examples .banner-removable .banner-removable-patterns data-id="DP: Examples-IDE" creative-id="examples-sidebar-es" position="sidebar"}
:::::: {.banner-inner style="font-size: 14px; line-height: 21px;"}
::: banner-examples-img
[![](../../../../images/patterns/banners/examples-ide91ca.png?id=3115b4b548fb96b75974e2de8f4f49bc){width="215"
height="158" loading="lazy"
srcset="/images/patterns/banners/examples-ide-2x.png?id=93c007a6d157b97c28eb213f59d716ae 2x"}](../../book.html)
:::

::: banner-examples-fade
[ Archivo con ejemplos de código](../../book.html){.btn .btn-secondary
.btn-block}
:::

::: {.banner-examples-text style="margin-top: 1rem;"}
Compra el libro electrónico **Sumérgete en los patrones de diseño** para
tener acceso al archivo con decenas de ejemplos detallados que puedes
abrir en tu IDE.

[ Saber más...](../../book.html){.btn .btn-secondary .btn-block}
:::
::::::
:::::::
::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::
