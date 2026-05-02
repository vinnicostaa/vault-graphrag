::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Patrones de
diseño](../../../design-patterns.html){.type} / [Factory
Method](../../factory-method.html){.type} /
[Swift](../../swift.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Factory
Method](../../../../images/patterns/cards/factory-method-minid7f6.png?id=72619e9527893374b98a5913779ac167){srcset="/images/patterns/cards/factory-method-mini-2x.png?id=fa9d4a8d61a67cc3822e52b9daf69dad 2x"}
:::

# **Factory Method** en Swift {#factory-method-en-swift .pattern-example-title-block-title}
::::

:::::::::::::: pattern-example-body
::: pattern-example-brief
**Factory method** es un patrón de diseño creacional que resuelve el
problema de crear objetos de producto sin especificar sus clases
concretas.

El patrón Factory Method define un método que debe utilizarse para crear
objetos, en lugar de una llamada directa al constructor (operador
`new`). Las subclases pueden sobrescribir este método para cambiar las
clases de los objetos que se crearán.

> Si no sabes la diferencia entre varios patrones y conceptos de la
> fábrica, lee nuestra [Comparación de
> fábricas](../../factory-comparison.html).
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Aprende más sobre el patrón Factory Method
](../../factory-method.html){.btn .btn-lg .btn-primary}
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

**Ejemplos de uso:** El patrón Factory Method se utiliza mucho en el
código Swift. Resulta muy útil cuando necesitas proporcionar un alto
nivel de flexibilidad a tu código.

**Identificación:** Los métodos fábrica pueden ser reconocidos por
métodos de creación, que crean objetos de clases concretas, pero los
devuelven como objetos del tipo abstracto o interfaz.
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

Este ejemplo ilustra la estructura del patrón de diseño **Factory
Method** y se centra en las siguientes preguntas:

-   ¿De qué clases se compone?
-   ¿Qué papeles juegan esas clases?
-   ¿De qué forma se relacionan los elementos del patrón?

Después de conocer la estructura del patrón, será más fácil comprender
el siguiente ejemplo basado en un caso de uso real de Swift.

#### []{#example-0--Example-swift .anchor} **Example.swift:** Ejemplo conceptual

<figure class="code">
<pre class="code" lang="swift"><code>import XCTest

/// The Creator protocol declares the factory method that&#39;s supposed to return a
/// new object of a Product class. The Creator&#39;s subclasses usually provide the
/// implementation of this method.
protocol Creator {

    /// Note that the Creator may also provide some default implementation of
    /// the factory method.
    func factoryMethod() -&gt; Product

    /// Also note that, despite its name, the Creator&#39;s primary responsibility
    /// is not creating products. Usually, it contains some core business logic
    /// that relies on Product objects, returned by the factory method.
    /// Subclasses can indirectly change that business logic by overriding the
    /// factory method and returning a different type of product from it.
    func someOperation() -&gt; String
}

/// This extension implements the default behavior of the Creator. This behavior
/// can be overridden in subclasses.
extension Creator {

    func someOperation() -&gt; String {
        // Call the factory method to create a Product object.
        let product = factoryMethod()

        // Now, use the product.
        return &quot;Creator: The same creator&#39;s code has just worked with &quot; + product.operation()
    }
}

/// Concrete Creators override the factory method in order to change the
/// resulting product&#39;s type.
class ConcreteCreator1: Creator {

    /// Note that the signature of the method still uses the abstract product
    /// type, even though the concrete product is actually returned from the
    /// method. This way the Creator can stay independent of concrete product
    /// classes.
    public func factoryMethod() -&gt; Product {
        return ConcreteProduct1()
    }
}

class ConcreteCreator2: Creator {

    public func factoryMethod() -&gt; Product {
        return ConcreteProduct2()
    }
}

/// The Product protocol declares the operations that all concrete products must
/// implement.
protocol Product {

    func operation() -&gt; String
}

/// Concrete Products provide various implementations of the Product protocol.
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


/// The client code works with an instance of a concrete creator, albeit through
/// its base protocol. As long as the client keeps working with the creator via
/// the base protocol, you can pass it any creator&#39;s subclass.
class Client {
    // ...
    static func someClientCode(creator: Creator) {
        print(&quot;Client: I&#39;m not aware of the creator&#39;s class, but it still works.\n&quot;
            + creator.someOperation())
    }
    // ...
}

/// Let&#39;s see how it all works together.
class FactoryMethodConceptual: XCTestCase {

    func testFactoryMethodConceptual() {

        /// The Application picks a creator&#39;s type depending on the
        /// configuration or environment.

        print(&quot;App: Launched with the ConcreteCreator1.&quot;)
        Client.someClientCode(creator: ConcreteCreator1())

        print(&quot;\nApp: Launched with the ConcreteCreator2.&quot;)
        Client.someClientCode(creator: ConcreteCreator2())
    }
}</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** Resultado de la ejecución

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
## Ejemplo del mundo real {#example-1-title}

#### []{#example-1--Example-swift .anchor} **Example.swift:** Ejemplo del mundo real

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

#### []{#example-1--Output-txt .anchor} **Output.txt:** Resultado de la ejecución

<figure class="code">
<pre class="code" lang="output"><code>Info is presented over Wifi: Very important info of the presentation
Info is presented over Bluetooth: Very important info of the presentation</code></pre>
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

[Prototype en Swift []{.fa
.fa-arrow-right}](../../prototype/swift/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Volver

[[]{.fa .fa-arrow-left} Builder en
Swift](../../builder/swift/example.html){.btn .btn-default rel="prev"}
:::

## **Factory Method** en otros lenguajes

[![Factory Method en
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Factory Method en C#"){.prog-lang-link}
[![Factory Method en
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Factory Method en C++"){.prog-lang-link}
[![Factory Method en
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Factory Method en Go"){.prog-lang-link}
[![Factory Method en
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Factory Method en Java"){.prog-lang-link}
[![Factory Method en
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Factory Method en PHP"){.prog-lang-link}
[![Factory Method en
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Factory Method en Python"){.prog-lang-link}
[![Factory Method en
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Factory Method en Ruby"){.prog-lang-link}
[![Factory Method en
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Factory Method en Rust"){.prog-lang-link}
[![Factory Method en
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Factory Method en TypeScript"){.prog-lang-link}
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
