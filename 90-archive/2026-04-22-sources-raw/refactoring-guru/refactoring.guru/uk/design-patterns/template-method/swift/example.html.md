::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Патерни
проектування](../../../design-patterns.html){.type} / [Шаблонний
метод](../../template-method.html){.type} /
[Swift](../../swift.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Шаблонний
метод](../../../../images/patterns/cards/template-method-mini56e3.png?id=9f200248d88026d8e79d0f3dae411ab4){srcset="/images/patterns/cards/template-method-mini-2x.png?id=178bf56e39b3a1f548dd636076209c98 2x"}
:::

# **Шаблонний метод** на Swift {#шаблонний-метод-на-swift .pattern-example-title-block-title}
::::

:::::::::::::: pattern-example-body
::: pattern-example-brief
**Шаблонний метод** --- це поведінковий патерн, який визначає кістяк
алгоритму в суперкласі та змушує підкласи реалізувати конкретні кроки
цього алгоритму.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Детальніше про Шаблонний метод ](../../template-method.html){.btn
.btn-lg .btn-primary}
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

**Застосування:** Шаблонні методи можна зустріти в багатьох бібліотечних
класах Swift. Розробники створюють їх, щоб дозволити клієнтам легко та
швидко розширювати стандартний код за допомогою спадкування.

**Ознаки застосування патерна:** Клас змушує своїх нащадків реалізувати
методи-кроки, але самостійно реалізовує структуру алгоритму.
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

Цей приклад показує структуру патерна **Шаблонний метод**, а саме --- з
яких класів він складається, які ролі ці класи виконують і як вони
взаємодіють один з одним.

Після ознайомлення зі структурою, вам буде легше сприймати наступний
приклад, що розглядає реальний випадок використання патерна в світі
Swift.

#### []{#example-0--Example-swift .anchor} **Example.swift:** Приклад структури патерна

<figure class="code">
<pre class="code" lang="swift"><code>import XCTest


/// The Abstract Protocol and its extension defines a template method that
/// contains a skeleton of some algorithm, composed of calls to (usually)
/// abstract primitive operations.
///
/// Concrete subclasses should implement these operations, but leave the
/// template method itself intact.
protocol AbstractProtocol {

    /// The template method defines the skeleton of an algorithm.
    func templateMethod()

    /// These operations already have implementations.
    func baseOperation1()

    func baseOperation2()

    func baseOperation3()

    /// These operations have to be implemented in subclasses.
    func requiredOperations1()
    func requiredOperation2()

    /// These are &quot;hooks.&quot; Subclasses may override them, but it&#39;s not mandatory
    /// since the hooks already have default (but empty) implementation. Hooks
    /// provide additional extension points in some crucial places of the
    /// algorithm.
    func hook1()
    func hook2()
}

extension AbstractProtocol {

    func templateMethod() {
        baseOperation1()
        requiredOperations1()
        baseOperation2()
        hook1()
        requiredOperation2()
        baseOperation3()
        hook2()
    }

    /// These operations already have implementations.
    func baseOperation1() {
        print(&quot;AbstractProtocol says: I am doing the bulk of the work\n&quot;)
    }

    func baseOperation2() {
        print(&quot;AbstractProtocol says: But I let subclasses override some operations\n&quot;)
    }

    func baseOperation3() {
        print(&quot;AbstractProtocol says: But I am doing the bulk of the work anyway\n&quot;)
    }

    func hook1() {}
    func hook2() {}
}

/// Concrete classes have to implement all abstract operations of the base
/// class. They can also override some operations with a default implementation.
class ConcreteClass1: AbstractProtocol {

    func requiredOperations1() {
        print(&quot;ConcreteClass1 says: Implemented Operation1\n&quot;)
    }

    func requiredOperation2() {
        print(&quot;ConcreteClass1 says: Implemented Operation2\n&quot;)
    }

    func hook2() {
        print(&quot;ConcreteClass1 says: Overridden Hook2\n&quot;)
    }
}

/// Usually, concrete classes override only a fraction of base class&#39;
/// operations.
class ConcreteClass2: AbstractProtocol {

    func requiredOperations1() {
        print(&quot;ConcreteClass2 says: Implemented Operation1\n&quot;)
    }

    func requiredOperation2() {
        print(&quot;ConcreteClass2 says: Implemented Operation2\n&quot;)
    }

    func hook1() {
        print(&quot;ConcreteClass2 says: Overridden Hook1\n&quot;)
    }
}

/// The client code calls the template method to execute the algorithm. Client
/// code does not have to know the concrete class of an object it works with, as
/// long as it works with objects through the interface of their base class.
class Client {
    // ...
    static func clientCode(use object: AbstractProtocol) {
        // ...
        object.templateMethod()
        // ...
    }
    // ...
}


/// Let&#39;s see how it all works together.
class TemplateMethodConceptual: XCTestCase {

    func test() {

        print(&quot;Same client code can work with different subclasses:\n&quot;)
        Client.clientCode(use: ConcreteClass1())

        print(&quot;\nSame client code can work with different subclasses:\n&quot;)
        Client.clientCode(use: ConcreteClass2())
    }
}</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** Результат виконання

<figure class="code">
<pre class="code" lang="output"><code>Same client code can work with different subclasses:

AbstractProtocol says: I am doing the bulk of the work

ConcreteClass1 says: Implemented Operation1

AbstractProtocol says: But I let subclasses override some operations

ConcreteClass1 says: Implemented Operation2

AbstractProtocol says: But I am doing the bulk of the work anyway

ConcreteClass1 says: Overridden Hook2


Same client code can work with different subclasses:

AbstractProtocol says: I am doing the bulk of the work

ConcreteClass2 says: Implemented Operation1

AbstractProtocol says: But I let subclasses override some operations

ConcreteClass2 says: Overridden Hook1

ConcreteClass2 says: Implemented Operation2

AbstractProtocol says: But I am doing the bulk of the work anyway</code></pre>
</figure>
:::

::: {#example-1 .tab-pane role="tabpanel" aria-labelledby="example-1-tab" style="padding-top: 24px"}
## Життєвий приклад {#example-1-title}

#### []{#example-1--Example-swift .anchor} **Example.swift:** Життєвий приклад

<figure class="code">
<pre class="code" lang="swift"><code>import XCTest
import AVFoundation
import CoreLocation
import Photos

class TemplateMethodRealWorld: XCTestCase {

    /// A good example of Template Method is a life cycle of UIViewController

    func testTemplateMethodReal() {

        let accessors = [CameraAccessor(), MicrophoneAccessor(), PhotoLibraryAccessor()]

        accessors.forEach { item in
            item.requestAccessIfNeeded({ status in
                let message = status ? &quot;You have access to &quot; : &quot;You do not have access to &quot;
                print(message + item.description + &quot;\n&quot;)
            })
        }
    }
}

class PermissionAccessor: CustomStringConvertible {

    typealias Completion = (Bool) -&gt; ()

    func requestAccessIfNeeded(_ completion: @escaping Completion) {

        guard !hasAccess() else { completion(true); return }

        willReceiveAccess()

        requestAccess { status in
            status ? self.didReceiveAccess() : self.didRejectAccess()

            completion(status)
        }
    }

    func requestAccess(_ completion: @escaping Completion) {
        fatalError(&quot;Should be overridden&quot;)
    }

    func hasAccess() -&gt; Bool {
        fatalError(&quot;Should be overridden&quot;)
    }

    var description: String { return &quot;PermissionAccessor&quot; }

    /// Hooks
    func willReceiveAccess() {}

    func didReceiveAccess() {}

    func didRejectAccess() {}
}

class CameraAccessor: PermissionAccessor {

    override func requestAccess(_ completion: @escaping Completion) {
        AVCaptureDevice.requestAccess(for: .video) { status in
            return completion(status)
        }
    }

    override func hasAccess() -&gt; Bool {
        return AVCaptureDevice.authorizationStatus(for: .video) == .authorized
    }

    override var description: String { return &quot;Camera&quot; }
}

class MicrophoneAccessor: PermissionAccessor {

    override func requestAccess(_ completion: @escaping Completion) {
        AVAudioSession.sharedInstance().requestRecordPermission { status in
            completion(status)
        }
    }

    override func hasAccess() -&gt; Bool {
        return AVAudioSession.sharedInstance().recordPermission == .granted
    }

    override var description: String { return &quot;Microphone&quot; }
}

class PhotoLibraryAccessor: PermissionAccessor {

    override func requestAccess(_ completion: @escaping Completion) {
        PHPhotoLibrary.requestAuthorization { status in
            completion(status == .authorized)
        }
    }

    override func hasAccess() -&gt; Bool {
        return PHPhotoLibrary.authorizationStatus() == .authorized
    }

    override var description: String { return &quot;PhotoLibrary&quot; }

    override func didReceiveAccess() {
        /// We want to track how many people give access to the PhotoLibrary.
        print(&quot;PhotoLibrary Accessor: Receive access. Updating analytics...&quot;)
    }

    override func didRejectAccess() {
        /// ... and also we want to track how many people rejected access.
        print(&quot;PhotoLibrary Accessor: Rejected with access. Updating analytics...&quot;)
    }
}</code></pre>
</figure>

#### []{#example-1--Output-txt .anchor} **Output.txt:** Результат виконання

<figure class="code">
<pre class="code" lang="output"><code>You have access to Camera

You have access to Microphone

PhotoLibrary Accessor: Rejected with access. Updating analytics...
You do not have access to PhotoLibrary</code></pre>
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

[Відвідувач на Swift []{.fa
.fa-arrow-right}](../../visitor/swift/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Повернутись назад

[[]{.fa .fa-arrow-left} Стратегія на
Swift](../../strategy/swift/example.html){.btn .btn-default rel="prev"}
:::

## **Шаблонний метод** іншими мовами програмування

[![Шаблонний метод на
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Шаблонний метод на C#"){.prog-lang-link}
[![Шаблонний метод на
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Шаблонний метод на C++"){.prog-lang-link}
[![Шаблонний метод на
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Шаблонний метод на Go"){.prog-lang-link}
[![Шаблонний метод на
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Шаблонний метод на Java"){.prog-lang-link}
[![Шаблонний метод на
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Шаблонний метод на PHP"){.prog-lang-link}
[![Шаблонний метод на
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Шаблонний метод на Python"){.prog-lang-link}
[![Шаблонний метод на
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Шаблонний метод на Ruby"){.prog-lang-link}
[![Шаблонний метод на
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Шаблонний метод на Rust"){.prog-lang-link}
[![Шаблонний метод на
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Шаблонний метод на TypeScript"){.prog-lang-link}
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
