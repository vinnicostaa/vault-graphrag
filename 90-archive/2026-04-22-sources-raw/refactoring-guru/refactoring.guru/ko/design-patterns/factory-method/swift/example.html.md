:::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [디자인
패턴들](../../../design-patterns.html){.type} / [팩토리
메서드](../../factory-method.html){.type} /
[스위프트](../../swift.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![팩토리
메서드](../../../../images/patterns/cards/factory-method-minid7f6.png?id=72619e9527893374b98a5913779ac167){srcset="/images/patterns/cards/factory-method-mini-2x.png?id=fa9d4a8d61a67cc3822e52b9daf69dad 2x"}
:::

# 스위프트로 작성된 **팩토리 메서드** {#스위프트로-작성된-팩토리-메서드 .pattern-example-title-block-title}
::::

:::::::::::::: pattern-example-body
::: pattern-example-brief
**팩토리 메서드**는 제품 객체들의 구상 클래스들을 지정하지 않고 해당
제품 객체들을 생성할 수 있도록 하는 생성 디자인 패턴입니다.

팩토리 메서드는 메서드를 정의하며, 이 메서드는 직접 생성자 호출​(`new`
연산자)​을 사용하여 객체를 생성하는 대신 객체 생성에 사용되여야 합니다.
자식 클래스들은 이 메서드를 오버라이드하여 생성될 객체들의 클래스를
변경할 수 있습니다.

> 다양한 팩토리 패턴들과 개념들의 차이점을 이해하지 못하셨다면 [팩토리
> 비교](../../factory-comparison.html)를 읽어보세요.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ 팩토리 메서드에 대하여 더 자세히 알아보세요
](../../factory-method.html){.btn .btn-lg .btn-primary}
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

**사용 사례들:** 팩토리 메서드 패턴은 스위프트 코드에서 널리 사용되며
코드에 높은 수준의 유연성을 제공해야 할 때 매우 유용합니다.

**식별:** 팩토리 메서드는 구상 클래스들로부터 객체들을 생성하는 생성
메서드들로 인식될 수 있습니다. 구상 클래스들은 객체 생성 중에 사용되지만
팩토리 메서드들의 반환 유형은 일반적으로 추상 클래스 또는 인터페이스로
선언됩니다.
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

이 예시는 **팩토리 메서드**의 구조를 보여주고 다음 질문에 중점을 둡니다:

-   패턴은 어떤 클래스들로 구성되어 있나요?
-   이 클래스들은 어떤 역할을 하나요?
-   패턴의 요소들은 어떻게 서로 연관되어 있나요?

이 패턴의 구조를 배우면 실제 스위프트 언어 사용 사례를 기반으로 하는
다음 예시를 더욱 쉽게 이해할 수 있을 것입니다.

#### []{#example-0--Example-swift .anchor} **Example.swift:** 개념적인 예시

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

#### []{#example-0--Output-txt .anchor} **Output.txt:** 실행 결과

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
## 실제 사례 예시 {#example-1-title}

#### []{#example-1--Example-swift .anchor} **Example.swift:** 실제 사례 예시

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

#### []{#example-1--Output-txt .anchor} **Output.txt:** 실행 결과

<figure class="code">
<pre class="code" lang="output"><code>Info is presented over Wifi: Very important info of the presentation
Info is presented over Bluetooth: Very important info of the presentation</code></pre>
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

[스위프트로 작성된 프로토타입 []{.fa
.fa-arrow-right}](../../prototype/swift/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### 돌아가기

[[]{.fa .fa-arrow-left} 스위프트로 작성된
빌더](../../builder/swift/example.html){.btn .btn-default rel="prev"}
:::

## 다른 언어로 작성된 **팩토리 메서드**

[![C#으로 작성된 팩토리
메서드](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "C#으로 작성된 팩토리 메서드"){.prog-lang-link}
[![C++로 작성된 팩토리
메서드](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "C++로 작성된 팩토리 메서드"){.prog-lang-link}
[![Go로 작성된 팩토리
메서드](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Go로 작성된 팩토리 메서드"){.prog-lang-link}
[![자바로 작성된 팩토리
메서드](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "자바로 작성된 팩토리 메서드"){.prog-lang-link}
[![PHP로 작성된 팩토리
메서드](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "PHP로 작성된 팩토리 메서드"){.prog-lang-link}
[![파이썬으로 작성된 팩토리
메서드](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "파이썬으로 작성된 팩토리 메서드"){.prog-lang-link}
[![루비로 작성된 팩토리
메서드](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "루비로 작성된 팩토리 메서드"){.prog-lang-link}
[![러스트로 작성된 팩토리
메서드](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "러스트로 작성된 팩토리 메서드"){.prog-lang-link}
[![타입스크립트로 작성된 팩토리
메서드](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "타입스크립트로 작성된 팩토리 메서드"){.prog-lang-link}
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
