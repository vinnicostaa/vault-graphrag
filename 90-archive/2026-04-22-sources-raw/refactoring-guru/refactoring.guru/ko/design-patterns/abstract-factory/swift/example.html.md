:::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [디자인
패턴들](../../../design-patterns.html){.type} / [추상
팩토리](../../abstract-factory.html){.type} /
[스위프트](../../swift.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![추상
팩토리](../../../../images/patterns/cards/abstract-factory-minid1c5.png?id=4c3927c446313a38ce77dfee38111e27){srcset="/images/patterns/cards/abstract-factory-mini-2x.png?id=22236aaa65ff52cbde1c713216d52c1f 2x"}
:::

# 스위프트로 작성된 **추상 팩토리** {#스위프트로-작성된-추상-팩토리 .pattern-example-title-block-title}
::::

:::::::::::::: pattern-example-body
::: pattern-example-brief
**추상 팩토리**는 생성 디자인 패턴이며, 관련 객체들의 구상 클래스들을
지정하지 않고도 해당 객체들의 제품 패밀리들을 생성할 수 있도록 합니다.

추상 팩토리는 모든 고유한 제품들을 생성하기 위한 인터페이스를 정의하지만
실제 제품 생성은 구상 팩토리 클래스들에 맡깁니다. 또 각 팩토리 유형은
특정 제품군에 해당합니다.

클라이언트 코드는 생성자 호출​(`new` 연산자)​로 직접 제품들을 생성하는
대신 팩토리 객체의 생성 메서드들을 호출합니다. 팩토리는 단일 제품 변형에
해당하므로 해당 팩토리의 모든 제품이 호환될 것입니다.

클라이언트 코드는 추상 인터페이스를 통해서만 팩토리 및 제품과 함께
작동하며, 이렇게 하면 클라이언트 코드가 팩토리 객체에 의해 생성된 모든
제품 변형과 함께 작동할 수 있습니다. 새로운 구상 팩토리 클래스를 생성한
후 클라이언트 코드에 전달합니다.

> 다양한 팩토리 패턴들과 개념들의 차이점을 이해하지 못하셨다면 [팩토리
> 비교](../../factory-comparison.html)를 읽어보세요.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ 추상 팩토리에 대하여 더 자세히 알아보세요
](../../abstract-factory.html){.btn .btn-lg .btn-primary}
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

**사용 예시들:** 추상 팩토리 패턴은 스위프트 코드에 자주 사용됩니다.
많은 프레임워크들과 라이브러리들은 이 패턴을 표준 컴포넌트들을 확장 및
사용자 지정하기 위해 사용합니다.

**식별:** 패턴은 팩토리 객체를 반환하는 메서드들의 존재 여부로 쉽게
인식할 수 있습니다. 그 후 팩토리는 특정 하위 컴포넌트들을 만드는 데
사용됩니다.
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

이 예시는 **추상 팩토리** 디자인 패턴의 구조를 보여주고 다음 질문에
중점을 둡니다:

-   패턴은 어떤 클래스들로 구성되어 있나요?
-   이 클래스들은 어떤 역할을 하나요?
-   패턴의 요소들은 어떻게 서로 연관되어 있나요?

이 패턴의 구조를 배우면 실제 스위프트 사용 사례를 기반으로 하는 다음
예시를 더욱 쉽게 이해할 수 있을 것입니다.

#### []{#example-0--Example-swift .anchor} **Example.swift:** 개념적인 예시

<figure class="code">
<pre class="code" lang="swift"><code>import XCTest

/// The Abstract Factory protocol declares a set of methods that return
/// different abstract products. These products are called a family and are
/// related by a high-level theme or concept. Products of one family are usually
/// able to collaborate among themselves. A family of products may have several
/// variants, but the products of one variant are incompatible with products of
/// another.
protocol AbstractFactory {

    func createProductA() -&gt; AbstractProductA
    func createProductB() -&gt; AbstractProductB
}

/// Concrete Factories produce a family of products that belong to a single
/// variant. The factory guarantees that resulting products are compatible. Note
/// that signatures of the Concrete Factory&#39;s methods return an abstract
/// product, while inside the method a concrete product is instantiated.
class ConcreteFactory1: AbstractFactory {

    func createProductA() -&gt; AbstractProductA {
        return ConcreteProductA1()
    }

    func createProductB() -&gt; AbstractProductB {
        return ConcreteProductB1()
    }
}

/// Each Concrete Factory has a corresponding product variant.
class ConcreteFactory2: AbstractFactory {

    func createProductA() -&gt; AbstractProductA {
        return ConcreteProductA2()
    }

    func createProductB() -&gt; AbstractProductB {
        return ConcreteProductB2()
    }
}

/// Each distinct product of a product family should have a base protocol. All
/// variants of the product must implement this protocol.
protocol AbstractProductA {

    func usefulFunctionA() -&gt; String
}

/// Concrete Products are created by corresponding Concrete Factories.
class ConcreteProductA1: AbstractProductA {

    func usefulFunctionA() -&gt; String {
        return &quot;The result of the product A1.&quot;
    }
}

class ConcreteProductA2: AbstractProductA {

    func usefulFunctionA() -&gt; String {
        return &quot;The result of the product A2.&quot;
    }
}

/// The base protocol of another product. All products can interact with each
/// other, but proper interaction is possible only between products of the same
/// concrete variant.
protocol AbstractProductB {

    /// Product B is able to do its own thing...
    func usefulFunctionB() -&gt; String

    /// ...but it also can collaborate with the ProductA.
    ///
    /// The Abstract Factory makes sure that all products it creates are of the
    /// same variant and thus, compatible.
    func anotherUsefulFunctionB(collaborator: AbstractProductA) -&gt; String
}

/// Concrete Products are created by corresponding Concrete Factories.
class ConcreteProductB1: AbstractProductB {

    func usefulFunctionB() -&gt; String {
        return &quot;The result of the product B1.&quot;
    }

    /// This variant, Product B1, is only able to work correctly with the
    /// variant, Product A1. Nevertheless, it accepts any instance of
    /// AbstractProductA as an argument.
    func anotherUsefulFunctionB(collaborator: AbstractProductA) -&gt; String {
        let result = collaborator.usefulFunctionA()
        return &quot;The result of the B1 collaborating with the (\(result))&quot;
    }
}

class ConcreteProductB2: AbstractProductB {

    func usefulFunctionB() -&gt; String {
        return &quot;The result of the product B2.&quot;
    }

    /// This variant, Product B2, is only able to work correctly with the
    /// variant, Product A2. Nevertheless, it accepts any instance of
    /// AbstractProductA as an argument.
    func anotherUsefulFunctionB(collaborator: AbstractProductA) -&gt; String {
        let result = collaborator.usefulFunctionA()
        return &quot;The result of the B2 collaborating with the (\(result))&quot;
    }
}

/// The client code works with factories and products only through abstract
/// types: AbstractFactory and AbstractProduct. This lets you pass any factory
/// or product subclass to the client code without breaking it.
class Client {
    // ...
    static func someClientCode(factory: AbstractFactory) {
        let productA = factory.createProductA()
        let productB = factory.createProductB()

        print(productB.usefulFunctionB())
        print(productB.anotherUsefulFunctionB(collaborator: productA))
    }
    // ...
}

/// Let&#39;s see how it all works together.
class AbstractFactoryConceptual: XCTestCase {

    func testAbstractFactoryConceptual() {

        /// The client code can work with any concrete factory class.

        print(&quot;Client: Testing client code with the first factory type:&quot;)
        Client.someClientCode(factory: ConcreteFactory1())

        print(&quot;Client: Testing the same client code with the second factory type:&quot;)
        Client.someClientCode(factory: ConcreteFactory2())
    }
}</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** 실행 결과

<figure class="code">
<pre class="code" lang="output"><code>Client: Testing client code with the first factory type:
The result of the product B1.
The result of the B1 collaborating with the (The result of the product A1.)
Client: Testing the same client code with the second factory type:
The result of the product B2.
The result of the B2 collaborating with the (The result of the product A2.)</code></pre>
</figure>
:::

::: {#example-1 .tab-pane role="tabpanel" aria-labelledby="example-1-tab" style="padding-top: 24px"}
## 실제 사례 예시 {#example-1-title}

#### []{#example-1--Example-swift .anchor} **Example.swift:** 실제 사례 예시

<figure class="code">
<pre class="code" lang="swift"><code>import Foundation
import UIKit
import XCTest

enum AuthType {
    case login
    case signUp
}


protocol AuthViewFactory {

    static func authView(for type: AuthType) -&gt; AuthView
    static func authController(for type: AuthType) -&gt; AuthViewController
}

class StudentAuthViewFactory: AuthViewFactory {

    static func authView(for type: AuthType) -&gt; AuthView {
        print(&quot;Student View has been created&quot;)
        switch type {
            case .login: return StudentLoginView()
            case .signUp: return StudentSignUpView()
        }
    }

    static func authController(for type: AuthType) -&gt; AuthViewController {
        let controller = StudentAuthViewController(contentView: authView(for: type))
        print(&quot;Student View Controller has been created&quot;)
        return controller
    }
}

class TeacherAuthViewFactory: AuthViewFactory {

    static func authView(for type: AuthType) -&gt; AuthView {
        print(&quot;Teacher View has been created&quot;)
        switch type {
            case .login: return TeacherLoginView()
            case .signUp: return TeacherSignUpView()
        }
    }

    static func authController(for type: AuthType) -&gt; AuthViewController {
        let controller = TeacherAuthViewController(contentView: authView(for: type))
        print(&quot;Teacher View Controller has been created&quot;)
        return controller
    }
}



protocol AuthView {

    typealias AuthAction = (AuthType) -&gt; ()

    var contentView: UIView { get }
    var authHandler: AuthAction? { get set }

    var description: String { get }
}

class StudentSignUpView: UIView, AuthView {

    private class StudentSignUpContentView: UIView {

        /// This view contains a number of features available only during a
        /// STUDENT authorization.
    }

    var contentView: UIView = StudentSignUpContentView()

    /// The handler will be connected for actions of buttons of this view.
    var authHandler: AuthView.AuthAction?

    override var description: String {
        return &quot;Student-SignUp-View&quot;
    }
}

class StudentLoginView: UIView, AuthView {

    private let emailField = UITextField()
    private let passwordField = UITextField()
    private let signUpButton = UIButton()

    var contentView: UIView {
        return self
    }

    /// The handler will be connected for actions of buttons of this view.
    var authHandler: AuthView.AuthAction?

    override var description: String {
        return &quot;Student-Login-View&quot;
    }
}



class TeacherSignUpView: UIView, AuthView {

    class TeacherSignUpContentView: UIView {

        /// This view contains a number of features available only during a
        /// TEACHER authorization.
    }

    var contentView: UIView = TeacherSignUpContentView()

    /// The handler will be connected for actions of buttons of this view.
    var authHandler: AuthView.AuthAction?

    override var description: String {
        return &quot;Teacher-SignUp-View&quot;
    }
}

class TeacherLoginView: UIView, AuthView {

    private let emailField = UITextField()
    private let passwordField = UITextField()
    private let loginButton = UIButton()
    private let forgotPasswordButton = UIButton()

    var contentView: UIView {
        return self
    }

    /// The handler will be connected for actions of buttons of this view.
    var authHandler: AuthView.AuthAction?

    override var description: String {
        return &quot;Teacher-Login-View&quot;
    }
}



class AuthViewController: UIViewController {

    fileprivate var contentView: AuthView

    init(contentView: AuthView) {
        self.contentView = contentView
        super.init(nibName: nil, bundle: nil)
    }

    required convenience init?(coder aDecoder: NSCoder) {
        return nil
    }
}

class StudentAuthViewController: AuthViewController {

    /// Student-oriented features
}

class TeacherAuthViewController: AuthViewController {

    /// Teacher-oriented features
}


private class ClientCode {

    private var currentController: AuthViewController?

    private lazy var navigationController: UINavigationController = {
        guard let vc = currentController else { return UINavigationController() }
        return UINavigationController(rootViewController: vc)
    }()

    private let factoryType: AuthViewFactory.Type

    init(factoryType: AuthViewFactory.Type) {
        self.factoryType = factoryType
    }

    /// MARK: - Presentation

    func presentLogin() {
        let controller = factoryType.authController(for: .login)
        navigationController.pushViewController(controller, animated: true)
    }

    func presentSignUp() {
        let controller = factoryType.authController(for: .signUp)
        navigationController.pushViewController(controller, animated: true)
    }

    /// Other methods...
}


class AbstractFactoryRealWorld: XCTestCase {

    func testFactoryMethodRealWorld() {

        #if teacherMode
            let clientCode = ClientCode(factoryType: TeacherAuthViewFactory.self)
        #else
            let clientCode = ClientCode(factoryType: StudentAuthViewFactory.self)
        #endif

        /// Present LogIn flow
        clientCode.presentLogin()
        print(&quot;Login screen has been presented&quot;)

        /// Present SignUp flow
        clientCode.presentSignUp()
        print(&quot;Sign up screen has been presented&quot;)
    }
}</code></pre>
</figure>

#### []{#example-1--Output-txt .anchor} **Output.txt:** 실행 결과

<figure class="code">
<pre class="code" lang="output"><code>Teacher View has been created
Teacher View Controller has been created
Login screen has been presented
Teacher View has been created
Teacher View Controller has been created
Sign up screen has been presented</code></pre>
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

[스위프트로 작성된 빌더 []{.fa
.fa-arrow-right}](../../builder/swift/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### 돌아가기

[[]{.fa .fa-arrow-left} 스위프트로 작성된 디자인
패턴들](../../swift.html){.btn .btn-default rel="prev"}
:::

## 다른 언어로 작성된 **추상 팩토리**

[![C#으로 작성된 추상
팩토리](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "C#으로 작성된 추상 팩토리"){.prog-lang-link}
[![C++로 작성된 추상
팩토리](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "C++로 작성된 추상 팩토리"){.prog-lang-link}
[![Go로 작성된 추상
팩토리](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Go로 작성된 추상 팩토리"){.prog-lang-link}
[![자바로 작성된 추상
팩토리](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "자바로 작성된 추상 팩토리"){.prog-lang-link}
[![PHP로 작성된 추상
팩토리](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "PHP로 작성된 추상 팩토리"){.prog-lang-link}
[![파이썬으로 작성된 추상
팩토리](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "파이썬으로 작성된 추상 팩토리"){.prog-lang-link}
[![루비로 작성된 추상
팩토리](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "루비로 작성된 추상 팩토리"){.prog-lang-link}
[![러스트로 작성된 추상
팩토리](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "러스트로 작성된 추상 팩토리"){.prog-lang-link}
[![타입스크립트로 작성된 추상
팩토리](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "타입스크립트로 작성된 추상 팩토리"){.prog-lang-link}
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
