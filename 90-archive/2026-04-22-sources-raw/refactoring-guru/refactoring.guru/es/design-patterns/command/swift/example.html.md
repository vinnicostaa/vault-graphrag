::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Patrones de
diseño](../../../design-patterns.html){.type} /
[Command](../../command.html){.type} / [Swift](../../swift.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Command](../../../../images/patterns/cards/command-mini8482.png?id=b149eda017c0583c1e92343b83cfb1eb){srcset="/images/patterns/cards/command-mini-2x.png?id=e5f6332e057f6d352a209da963a8fc54 2x"}
:::

# **Command** en Swift {#command-en-swift .pattern-example-title-block-title}
::::

:::::::::::::: pattern-example-body
::: pattern-example-brief
**Command** es un patrón de diseño de comportamiento que convierte
solicitudes u operaciones simples en objetos.

La conversión permite la ejecución diferida de comandos, el
almacenamiento del historial de comandos, etc.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Aprende más sobre el patrón Command ](../../command.html){.btn .btn-lg
.btn-primary}
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

**Ejemplos de uso:** El patrón Command es muy común en el código Swift.
La mayoría de las veces se utiliza como alternativa a las retrollamadas
(*callbacks*) para parametrizar elementos UI con acciones. También se
utiliza para poner tareas en cola, realizar el seguimiento del historial
de operaciones, etc.

**Identificación:** El patrón Command es reconocible por los métodos de
comportamiento en un tipo de clase abstracta/interfaz (emisora) que
invoca un método en una implementación de un tipo de clase
abstracta/interfaz diferente (receptora) que la implementación del
comando ha implementado durante su creación. Las clases de comando se
limitan normalmente a acciones específicas.
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

Este ejemplo ilustra la estructura del patrón de diseño **Command** y se
centra en las siguientes preguntas:

-   ¿De qué clases se compone?
-   ¿Qué papeles juegan esas clases?
-   ¿De qué forma se relacionan los elementos del patrón?

Después de conocer la estructura del patrón, será más fácil comprender
el siguiente ejemplo basado en un caso de uso real de Swift.

#### []{#example-0--Example-swift .anchor} **Example.swift:** Ejemplo conceptual

<figure class="code">
<pre class="code" lang="swift"><code>import XCTest

/// The Command interface declares a method for executing a command.
protocol Command {

    func execute()
}

/// Some commands can implement simple operations on their own.
class SimpleCommand: Command {

    private var payload: String

    init(_ payload: String) {
        self.payload = payload
    }

    func execute() {
        print(&quot;SimpleCommand: See, I can do simple things like printing (&quot; + payload + &quot;)&quot;)
    }
}

/// However, some commands can delegate more complex operations to other
/// objects, called &quot;receivers.&quot;
class ComplexCommand: Command {

    private var receiver: Receiver

    /// Context data, required for launching the receiver&#39;s methods.
    private var a: String
    private var b: String

    /// Complex commands can accept one or several receiver objects along with
    /// any context data via the constructor.
    init(_ receiver: Receiver, _ a: String, _ b: String) {
        self.receiver = receiver
        self.a = a
        self.b = b
    }

    /// Commands can delegate to any methods of a receiver.
    func execute() {
        print(&quot;ComplexCommand: Complex stuff should be done by a receiver object.\n&quot;)
        receiver.doSomething(a)
        receiver.doSomethingElse(b)
    }
}

/// The Receiver classes contain some important business logic. They know how to
/// perform all kinds of operations, associated with carrying out a request. In
/// fact, any class may serve as a Receiver.
class Receiver {

    func doSomething(_ a: String) {
        print(&quot;Receiver: Working on (&quot; + a + &quot;)\n&quot;)
    }

    func doSomethingElse(_ b: String) {
        print(&quot;Receiver: Also working on (&quot; + b + &quot;)\n&quot;)
    }
}

/// The Invoker is associated with one or several commands. It sends a request
/// to the command.
class Invoker {

    private var onStart: Command?

    private var onFinish: Command?

    /// Initialize commands.
    func setOnStart(_ command: Command) {
        onStart = command
    }

    func setOnFinish(_ command: Command) {
        onFinish = command
    }

    /// The Invoker does not depend on concrete command or receiver classes. The
    /// Invoker passes a request to a receiver indirectly, by executing a
    /// command.
    func doSomethingImportant() {

        print(&quot;Invoker: Does anybody want something done before I begin?&quot;)

        onStart?.execute()

        print(&quot;Invoker: ...doing something really important...&quot;)
        print(&quot;Invoker: Does anybody want something done after I finish?&quot;)

        onFinish?.execute()
    }
}

/// Let&#39;s see how it all comes together.
class CommandConceptual: XCTestCase {

    func test() {
        /// The client code can parameterize an invoker with any commands.

        let invoker = Invoker()
        invoker.setOnStart(SimpleCommand(&quot;Say Hi!&quot;))

        let receiver = Receiver()
        invoker.setOnFinish(ComplexCommand(receiver, &quot;Send email&quot;, &quot;Save report&quot;))
        invoker.doSomethingImportant()
    }
}</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** Resultado de la ejecución

<figure class="code">
<pre class="code" lang="output"><code>Invoker: Does anybody want something done before I begin?
SimpleCommand: See, I can do simple things like printing (Say Hi!)
Invoker: ...doing something really important...
Invoker: Does anybody want something done after I finish?
ComplexCommand: Complex stuff should be done by a receiver object.

Receiver: Working on (Send email)

Receiver: Also working on (Save report)</code></pre>
</figure>
:::

::: {#example-1 .tab-pane role="tabpanel" aria-labelledby="example-1-tab" style="padding-top: 24px"}
## Ejemplo del mundo real {#example-1-title}

#### []{#example-1--Example-swift .anchor} **Example.swift:** Ejemplo del mundo real

<figure class="code">
<pre class="code" lang="swift"><code>import Foundation
import XCTest


class DelayedOperation: Operation, @unchecked Sendable {

    private var delay: TimeInterval

    init(_ delay: TimeInterval = 0) {
        self.delay = delay
    }

    override var isExecuting : Bool {
        get { return _executing }
        set {
            willChangeValue(forKey: &quot;isExecuting&quot;)
            _executing = newValue
            didChangeValue(forKey: &quot;isExecuting&quot;)
        }
    }
    private var _executing : Bool = false

    override var isFinished : Bool {
        get { return _finished }
        set {
            willChangeValue(forKey: &quot;isFinished&quot;)
            _finished = newValue
            didChangeValue(forKey: &quot;isFinished&quot;)
        }
    }
    private var _finished : Bool = false

    override func start() {

        guard delay &gt; 0 else {
            _start()
            return
        }

        let deadline = DispatchTime.now() + delay
        DispatchQueue(label: &quot;&quot;).asyncAfter(deadline: deadline) {
            self._start()
        }
    }

    private func _start() {

        guard !self.isCancelled else {
            print(&quot;\(self): operation is canceled&quot;)
            self.isFinished = true
            return
        }

        self.isExecuting = true
        self.main()
        self.isExecuting = false
        self.isFinished = true
    }
}

class WindowOperation: DelayedOperation, @unchecked Sendable {

    override func main() {
        print(&quot;\(self): Windows are closed via HomeKit.&quot;)
    }

    override var description: String { return &quot;WindowOperation&quot; }
}

class DoorOperation: DelayedOperation, @unchecked Sendable {

    override func main() {
        print(&quot;\(self): Doors are closed via HomeKit.&quot;)
    }

    override var description: String { return &quot;DoorOperation&quot; }
}

class TaxiOperation: DelayedOperation, @unchecked Sendable {

    override func main() {
        print(&quot;\(self): Taxi is ordered via Uber&quot;)
    }

    override var description: String { return &quot;TaxiOperation&quot; }
}



class CommandRealWorld: XCTestCase {

    func testCommandRealWorld() {
        prepareTestEnvironment {

            let siri = SiriShortcuts.shared

            print(&quot;User: Hey Siri, I am leaving my home&quot;)
            siri.perform(.leaveHome)

            print(&quot;User: Hey Siri, I am leaving my work in 3 minutes&quot;)
            siri.perform(.leaveWork, delay: 3) /// for simplicity, we use seconds

            print(&quot;User: Hey Siri, I am still working&quot;)
            siri.cancel(.leaveWork)
        }
    }
}

extension CommandRealWorld {

    struct ExecutionTime {
        static let max: TimeInterval = 5
        static let waiting: TimeInterval = 4
    }

    func prepareTestEnvironment(_ execution: () -&gt; ()) {

        /// This method tells Xcode to wait for async operations. Otherwise the
        /// main test is done immediately.

        let expectation = self.expectation(description: &quot;Expectation for async operations&quot;)

        let deadline = DispatchTime.now() + ExecutionTime.waiting
        DispatchQueue.main.asyncAfter(deadline: deadline) { expectation.fulfill() }

        execution()

        wait(for: [expectation], timeout: ExecutionTime.max)
    }
}

class SiriShortcuts {

    static let shared = SiriShortcuts()
    private lazy var queue = OperationQueue()

    private init() {}

    enum Action: String {
        case leaveHome
        case leaveWork
    }

    func perform(_ action: Action, delay: TimeInterval = 0) {
        print(&quot;Siri: performing \(action)-action\n&quot;)
        switch action {
        case .leaveHome:
            add(operation: WindowOperation(delay))
            add(operation: DoorOperation(delay))
        case .leaveWork:
            add(operation: TaxiOperation(delay))
        }
    }

    func cancel(_ action: Action) {
        print(&quot;Siri: canceling \(action)-action\n&quot;)
        switch action {
        case .leaveHome:
            cancelOperation(with: WindowOperation.self)
            cancelOperation(with: DoorOperation.self)
        case .leaveWork:
            cancelOperation(with: TaxiOperation.self)
        }
    }

    private func cancelOperation(with operationType: Operation.Type) {
        queue.operations.filter { operation in
            return type(of: operation) == operationType
        }.forEach({ $0.cancel() })
    }

    private func add(operation: Operation) {
        queue.addOperation(operation)
    }
}</code></pre>
</figure>

#### []{#example-1--Output-txt .anchor} **Output.txt:** Resultado de la ejecución

<figure class="code">
<pre class="code" lang="output"><code>User: Hey Siri, I am leaving my home
Siri: performing leaveHome-action

User: Hey Siri, I am leaving my work in 3 minutes
Siri: performing leaveWork-action

User: Hey Siri, I am still working
Siri: canceling leaveWork-action

DoorOperation: Doors are closed via HomeKit.
WindowOperation: Windows are closed via HomeKit.
TaxiOperation: operation is canceled</code></pre>
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

[Iterator en Swift []{.fa
.fa-arrow-right}](../../iterator/swift/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Volver

[[]{.fa .fa-arrow-left} Chain of Responsibility en
Swift](../../chain-of-responsibility/swift/example.html){.btn
.btn-default rel="prev"}
:::

## **Command** en otros lenguajes

[![Command en
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Command en C#"){.prog-lang-link}
[![Command en
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Command en C++"){.prog-lang-link}
[![Command en
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Command en Go"){.prog-lang-link}
[![Command en
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Command en Java"){.prog-lang-link}
[![Command en
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Command en PHP"){.prog-lang-link}
[![Command en
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Command en Python"){.prog-lang-link}
[![Command en
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Command en Ruby"){.prog-lang-link}
[![Command en
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Command en Rust"){.prog-lang-link}
[![Command en
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Command en TypeScript"){.prog-lang-link}
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
