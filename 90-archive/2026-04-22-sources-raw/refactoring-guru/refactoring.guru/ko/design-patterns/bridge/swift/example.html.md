:::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [디자인
패턴들](../../../design-patterns.html){.type} /
[브리지](../../bridge.html){.type} / [스위프트](../../swift.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![브리지](../../../../images/patterns/cards/bridge-mini9e60.png?id=b389101d8ee8e23ffa1b534c704d0774){srcset="/images/patterns/cards/bridge-mini-2x.png?id=2622384cf623ed150ee9c21a0812dd87 2x"}
:::

# 스위프트로 작성된 **브리지** {#스위프트로-작성된-브리지 .pattern-example-title-block-title}
::::

:::::::::::::: pattern-example-body
::: pattern-example-brief
**브리지**는 구조 디자인 패턴입니다. 이 패턴은 비즈니스 로직 또는 거대한
클래스를 독립적으로 개발할 수 있는 별도의 클래스 계층구조들로 나눕니다.

종종 추상화라고 불리는 이러한 계층구조 중 하나는 두 번째 계층구조의
객체​(구현)​에 대한 참조를 얻습니다. 추상화의 호출들 일부​(때로는 대부분)​를
구현 객체에 위임할 수 있습니다. 모든 구현은 공통 인터페이스를 가지므로
추상화 내에서 상호 교환할 수 있습니다.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ 브리지에 대하여 더 자세히 알아보세요 ](../../bridge.html){.btn .btn-lg
.btn-primary}
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

**사용 예시들:** 브리지 패턴은 크로스 플랫폼 앱들을 처리할 때, 여러
유형의 데이터베이스 서버를 지원할 때 또는 특정 종류의 여러 API
제공자​(예: 클라우드 플랫폼, 소셜 네트워크 등)​와 작업할 때 특히
유용합니다.

**식별법:** 브리지는 일부 제어 개체가 해당 개체가 의존하는 여러 다른
플랫폼들과 명확하게 구분됩니다.
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

이 예시는 **브리지** 디자인 패턴의 구조를 보여주고 다음 질문에 중점을
둡니다:

-   패턴은 어떤 클래스들로 구성되어 있나요?
-   이 클래스들은 어떤 역할을 하나요?
-   패턴의 요소들은 어떻게 서로 연관되어 있나요?

이 패턴의 구조를 배우면 실제 스위프트 사용 사례를 기반으로 하는 다음
예시를 더욱 쉽게 이해할 수 있을 것입니다.

#### []{#example-0--Example-swift .anchor} **Example.swift:** 개념적인 예시

<figure class="code">
<pre class="code" lang="swift"><code>import XCTest

/// The Abstraction defines the interface for the &quot;control&quot; part of the two
/// class hierarchies. It holds a reference to an object from the Implementation
/// hierarchy and delegates all of the real work to this object.
class Abstraction {

    fileprivate var implementation: Implementation

    init(_ implementation: Implementation) {
        self.implementation = implementation
    }

    func operation() -&gt; String {
        let operation = implementation.operationImplementation()
        return &quot;Abstraction: Base operation with:\n&quot; + operation
    }
}

/// You can extend the Abstraction without changing the Implementation classes.
class ExtendedAbstraction: Abstraction {

    override func operation() -&gt; String {
        let operation = implementation.operationImplementation()
        return &quot;ExtendedAbstraction: Extended operation with:\n&quot; + operation
    }
}

/// The Implementation defines the interface for all implementation classes. It
/// doesn&#39;t have to match the Abstraction&#39;s interface. In fact, the two
/// interfaces can be entirely different. Typically the Implementation interface
/// provides only primitive operations, while the Abstraction defines higher-
/// level operations based on those primitives.
protocol Implementation {

    func operationImplementation() -&gt; String
}

/// Each Concrete Implementation corresponds to a specific platform and
/// implements the Implementation interface using that platform&#39;s API.
class ConcreteImplementationA: Implementation {

    func operationImplementation() -&gt; String {
        return &quot;ConcreteImplementationA: Here&#39;s the result on the platform A.\n&quot;
    }
}

class ConcreteImplementationB: Implementation {

    func operationImplementation() -&gt; String {
        return &quot;ConcreteImplementationB: Here&#39;s the result on the platform B\n&quot;
    }
}

/// Except for the initialization phase, where an Abstraction object gets linked
/// with a specific Implementation object, the client code should only depend on
/// the Abstraction class. This way the client code can support any abstraction-
/// implementation combination.
class Client {
    // ...
    static func someClientCode(abstraction: Abstraction) {
        print(abstraction.operation())
    }
    // ...
}

/// Let&#39;s see how it all works together.
class BridgeConceptual: XCTestCase {

    func testBridgeConceptual() {
        // The client code should be able to work with any pre-configured
        // abstraction-implementation combination.
        let implementation = ConcreteImplementationA()
        Client.someClientCode(abstraction: Abstraction(implementation))

        let concreteImplementation = ConcreteImplementationB()
        Client.someClientCode(abstraction: ExtendedAbstraction(concreteImplementation))
    }
}</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** 실행 결과

<figure class="code">
<pre class="code" lang="output"><code>Abstraction: Base operation with:
ConcreteImplementationA: Here&#39;s the result on the platform A

ExtendedAbstraction: Extended operation with:
ConcreteImplementationB: Here&#39;s the result on the platform B</code></pre>
</figure>
:::

::: {#example-1 .tab-pane role="tabpanel" aria-labelledby="example-1-tab" style="padding-top: 24px"}
## 실제 사례 예시 {#example-1-title}

#### []{#example-1--Example-swift .anchor} **Example.swift:** 실제 사례 예시

<figure class="code">
<pre class="code" lang="swift"><code>import XCTest

private class BridgeRealWorld: XCTestCase {

    func testBridgeRealWorld() {

        print(&quot;Client: Pushing Photo View Controller...&quot;)
        push(PhotoViewController())

        print()

        print(&quot;Client: Pushing Feed View Controller...&quot;)
        push(FeedViewController())
    }

    func push(_ container: SharingSupportable) {

        let instagram = InstagramSharingService()
        let facebook = FaceBookSharingService()

        container.accept(service: instagram)
        container.update(content: foodModel)

        container.accept(service: facebook)
        container.update(content: foodModel)
    }

    var foodModel: Content {
        return FoodDomainModel(title: &quot;This food is so various and delicious!&quot;,
                               images: [UIImage(), UIImage()],
                               calories: 47)
    }
}

private protocol SharingSupportable {

    /// Abstraction
    func accept(service: SharingService)

    func update(content: Content)
}

class BaseViewController: UIViewController, SharingSupportable {

    fileprivate var shareService: SharingService?

    func update(content: Content) {
        /// ...updating UI and showing a content...
        /// ...
        /// ... then, a user will choose a content and trigger an event
        print(&quot;\(description): User selected a \(content) to share&quot;)
        /// ...
        shareService?.share(content: content)
    }

    func accept(service: SharingService) {
        shareService = service
    }
}

class PhotoViewController: BaseViewController {

    /// Custom UI and features

    override var description: String {
        return &quot;PhotoViewController&quot;
    }
}

class FeedViewController: BaseViewController {

    /// Custom UI and features

    override var description: String {
        return &quot;FeedViewController&quot;
    }
}

protocol SharingService {

    /// Implementation
    func share(content: Content)
}

class FaceBookSharingService: SharingService {

    func share(content: Content) {

        /// Use FaceBook API to share a content
        print(&quot;Service: \(content) was posted to the Facebook&quot;)
    }
}

class InstagramSharingService: SharingService {

    func share(content: Content) {

        /// Use Instagram API to share a content
        print(&quot;Service: \(content) was posted to the Instagram&quot;, terminator: &quot;\n\n&quot;)
    }
}

protocol Content: CustomStringConvertible {

    var title: String { get }
    var images: [UIImage] { get }
}

struct FoodDomainModel: Content {

    var title: String
    var images: [UIImage]
    var calories: Int

    var description: String {
        return &quot;Food Model&quot;
    }
}</code></pre>
</figure>

#### []{#example-1--Output-txt .anchor} **Output.txt:** 실행 결과

<figure class="code">
<pre class="code" lang="output"><code>Client: Pushing Photo View Controller...
PhotoViewController: User selected a Food Model to share
Service: Food Model was posted to the Instagram

PhotoViewController: User selected a Food Model to share
Service: Food Model was posted to the Facebook

Client: Pushing Feed View Controller...
FeedViewController: User selected a Food Model to share
Service: Food Model was posted to the Instagram

FeedViewController: User selected a Food Model to share
Service: Food Model was posted to the Facebook</code></pre>
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

[스위프트로 작성된 복합체 []{.fa
.fa-arrow-right}](../../composite/swift/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### 돌아가기

[[]{.fa .fa-arrow-left} 스위프트로 작성된
어댑터](../../adapter/swift/example.html){.btn .btn-default rel="prev"}
:::

## 다른 언어로 작성된 **브리지**

[![C#으로 작성된
브리지](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "C#으로 작성된 브리지"){.prog-lang-link}
[![C++로 작성된
브리지](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "C++로 작성된 브리지"){.prog-lang-link}
[![Go로 작성된
브리지](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Go로 작성된 브리지"){.prog-lang-link}
[![자바로 작성된
브리지](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "자바로 작성된 브리지"){.prog-lang-link}
[![PHP로 작성된
브리지](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "PHP로 작성된 브리지"){.prog-lang-link}
[![파이썬으로 작성된
브리지](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "파이썬으로 작성된 브리지"){.prog-lang-link}
[![루비로 작성된
브리지](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "루비로 작성된 브리지"){.prog-lang-link}
[![러스트로 작성된
브리지](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "러스트로 작성된 브리지"){.prog-lang-link}
[![타입스크립트로 작성된
브리지](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "타입스크립트로 작성된 브리지"){.prog-lang-link}
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
