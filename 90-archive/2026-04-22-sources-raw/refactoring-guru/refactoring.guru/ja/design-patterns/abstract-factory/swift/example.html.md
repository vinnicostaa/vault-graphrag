::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} /
[デザインパターン](../../../design-patterns.html){.type} / [Abstract
Factory](../../abstract-factory.html){.type} /
[Swift](../../swift.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Abstract
Factory](../../../../images/patterns/cards/abstract-factory-minid1c5.png?id=4c3927c446313a38ce77dfee38111e27){srcset="/images/patterns/cards/abstract-factory-mini-2x.png?id=22236aaa65ff52cbde1c713216d52c1f 2x"}
:::

# **Abstract Factory** を Swift で {#abstract-factory-を-swift-で .pattern-example-title-block-title}
::::

:::::::::::::: pattern-example-body
::: pattern-example-brief
**Abstract Factory** は[、]{.chpule2}[
]{.chpuri2}生成に関するデザインパターンのひとつで[、]{.chpule2}[
]{.chpuri2}具象クラスを指定することなく[、]{.chpule2}[
]{.chpuri2}プロダクト[ ]{.chpule1}[（]{.chpuri1}訳注[：]{.chpule2}[
]{.chpuri2}本パターンでは[、]{.chpule2}[
]{.chpuri2}生成されるモノのことを一般にプロダクトと呼びます[）]{.chpule2}[
]{.chpuri2}のファミリー全部を生成することを可能とします[。]{.chpule2}[
]{.chpuri2}

Abstract Factory は[、]{.chpule2}[
]{.chpuri2}個々のプロダクト全部を作成するためのインターフェースを定義しますが[、]{.chpule2}[
]{.chpuri2}実際のプロダクト作成の作業は[、]{.chpule2}[
]{.chpuri2}具象クラスに委ねられます[。]{.chpule2}[
]{.chpuri2}ファクトリーの型[
]{.chpule1}[（]{.chpuri1}クラス[）]{.chpule2}[
]{.chpuri2}それぞれは[、]{.chpule2}[
]{.chpuri2}特定のプロダクトの異種に対応します[。]{.chpule2}[ ]{.chpuri2}

クライアント・コードは[、]{.chpule2}[
]{.chpuri2}コンストラクター呼び出し[ ]{.chpule1}[（]{.chpuri1}`new`
演算子[）]{.chpule2}[
]{.chpuri2}で直接プロダクトを作成する代わりにファクトリー・オブジェクトの作成メソッドを呼び出します[。]{.chpule2}[
]{.chpuri2}
ファクトリーはプロダクトの特定の異種に対応しているため[、]{.chpule2}[
]{.chpuri2}すべてのプロダクトには互換性があります[。]{.chpule2}[
]{.chpuri2}

クライアント・コードは[、]{.chpule2}[
]{.chpuri2}その抽象インターフェイスを通じてのみファクトリーやプロダクトとやりとりします[。]{.chpule2}[
]{.chpuri2}このため[、]{.chpule2}[
]{.chpuri2}クライアント・コードはファクトリー・オブジェクトによって作成された任意のプロダクトの異種と動作します[。]{.chpule2}[
]{.chpuri2}プログラマーがやるべきことは[、]{.chpule2}[
]{.chpuri2}新しい具象ファクトリー・クラスを作成し[、]{.chpule2}[
]{.chpuri2}それをクライアント・コードに渡すことです[。]{.chpule2}[
]{.chpuri2}

> もし各種ファクトリー系のパターンやコンセプトの違いで迷った場合は[、]{.chpule2}[
> ]{.chpuri2}[ファクトリーの比較](../../factory-comparison.html)
> をご覧ください[。]{.chpule2}[ ]{.chpuri2}
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Abstract Factory の詳細 ](../../abstract-factory.html){.btn .btn-lg
.btn-primary}
:::

::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
ナビゲーション
:::

::: en-intro
 [はじめに](#)
:::

::: en-intro
 [概念的な例](#example-0)
:::

::: en-file
 [Example](#example-0--Example-swift)
:::

::: en-file
 [Output](#example-0--Output-txt)
:::

::: en-intro
 [現実的な例](#example-1)
:::

::: en-file
 [Example](#example-1--Example-swift)
:::

::: en-file
 [Output](#example-1--Output-txt)
:::
:::::::::::

**複雑度[：]{.chpule2}[ ]{.chpuri2}**[]{.fa-stars}

**人気度[：]{.chpule2}[ ]{.chpuri2}**[]{.fa-stars}

**使用例[：]{.chpule2}[ ]{.chpuri2}** Abstract Factory
パターンは[、]{.chpule2}[ ]{.chpuri2}Swift
コードではよく見かけます[。]{.chpule2}[
]{.chpuri2}多くのフレームワークやライブラリーが[、]{.chpule2}[
]{.chpuri2}その標準コンポーネントを拡張したりカスタマイズするためにこのパターンを使います[。]{.chpule2}[
]{.chpuri2}

**見つけ方[：]{.chpule2}[ ]{.chpuri2}**このパターンは[、]{.chpule2}[
]{.chpuri2}ファクトリー・オブジェクトを返すメソッドに注目すれば[、]{.chpule2}[
]{.chpuri2}簡単に見つけられます[。]{.chpule2}[
]{.chpuri2}そしてファクトリーを使ってサブコンポーネントが作成されます[。]{.chpule2}[
]{.chpuri2}
::::::::::::::

::::: {style="padding: 1rem; border: solid 4px #e35e5e; border-radius: 1rem; text-align: center"}
::: {style="margin-bottom: 0.5rem;"}
以下の例は [Swift
Playgrounds](https://www.alemohamad.com/playgrounds){style="text-decoration: underline; display: inline-block"
rel="nofollow"}で利用できます。
:::

::: {style="font-size: 13px;"}
Playgroundバージョンを作成してくれた [Alejandro
Mohamad](https://www.alemohamad.com/){style="text-decoration: underline;"
rel="nofollow"}に感謝します。
:::
:::::

::: {#nav-tab .nav .nav-tabs .nav-justified role="tablist"}
[概念的な例](#example-0){#example-0-tab .nav-item .nav-link .active
toggle="tab" role="tab" aria-controls="example-0" aria-selected="true"}
[現実的な例](#example-1){#example-1-tab .nav-item .nav-link toggle="tab"
role="tab" aria-controls="example-1" aria-selected="true"}
:::

::::: {#nav-tabContent .pattern-example-tab-content .tab-content}
::: {#example-0 .tab-pane .show .active role="tabpanel" aria-labelledby="example-0-tab" style="padding-top: 24px"}
## 概念的な例 {#example-0-title}

この例は[、]{.chpule2}[ ]{.chpuri2}**Abstract Factory**
デザインパターンの構造を説明するためのものです[。]{.chpule2}[
]{.chpuri2}以下の質問に答えることを目的としています[：]{.chpule2}[
]{.chpuri2}

-   どういうクラスからできているか[？]{.chpule2}[ ]{.chpuri2}
-   それぞれのクラスの役割は[？]{.chpule2}[ ]{.chpuri2}
-   パターンの要素同士はどう関係しているのか[？]{.chpule2}[ ]{.chpuri2}

ここでパターンの構造を学んだ後だと[、]{.chpule2}[
]{.chpuri2}これに続く[、]{.chpule2}[ ]{.chpuri2}現実世界の Swift
でのユースケースが理解しやすくなります[。]{.chpule2}[ ]{.chpuri2}

#### []{#example-0--Example-swift .anchor} **Example.swift:** 概念的な例

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

#### []{#example-0--Output-txt .anchor} **Output.txt:** 実行結果

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
## 現実的な例 {#example-1-title}

#### []{#example-1--Example-swift .anchor} **Example.swift:** 現実的な例

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

#### []{#example-1--Output-txt .anchor} **Output.txt:** 実行結果

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
[概念的な例](#example-0){#example-0-tab-bottom .nav-item .nav-link
.active toggle="tab" role="tab" aria-controls="example-0"
aria-selected="true"} [現実的な例](#example-1){#example-1-tab-bottom
.nav-item .nav-link toggle="tab" role="tab" aria-controls="example-1"
aria-selected="true"}
:::

::: next
#### 次を読む

[Builder を Swift で []{.fa
.fa-arrow-right}](../../builder/swift/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### 戻る

[[]{.fa .fa-arrow-left} デザインパターンを Swift
で](../../swift.html){.btn .btn-default rel="prev"}
:::

## 他言語での **Abstract Factory**

[![Abstract Factory を C#
で](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Abstract Factory を C# で"){.prog-lang-link}
[![Abstract Factory を C++
で](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Abstract Factory を C++ で"){.prog-lang-link}
[![Abstract Factory を Go
で](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Abstract Factory を Go で"){.prog-lang-link}
[![Abstract Factory を Java
で](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Abstract Factory を Java で"){.prog-lang-link}
[![Abstract Factory を PHP
で](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Abstract Factory を PHP で"){.prog-lang-link}
[![Abstract Factory を Python
で](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Abstract Factory を Python で"){.prog-lang-link}
[![Abstract Factory を Ruby
で](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Abstract Factory を Ruby で"){.prog-lang-link}
[![Abstract Factory を Rust
で](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Abstract Factory を Rust で"){.prog-lang-link}
[![Abstract Factory を TypeScript
で](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Abstract Factory を TypeScript で"){.prog-lang-link}
::::::::::::::::::::::::::::

::::::: {.prom .banner-sidebar .banner .banner-striped .banner-examples .banner-removable .banner-removable-patterns data-id="DP: Examples-IDE" creative-id="examples-sidebar-ja" position="sidebar"}
:::::: {.banner-inner style="font-size: 14px; line-height: 21px;"}
::: banner-examples-img
[![](../../../../images/patterns/banners/examples-ide91ca.png?id=3115b4b548fb96b75974e2de8f4f49bc){width="215"
height="158" loading="lazy"
srcset="/images/patterns/banners/examples-ide-2x.png?id=93c007a6d157b97c28eb213f59d716ae 2x"}](../../book.html)
:::

::: banner-examples-fade
[ コード例の書庫](../../book.html){.btn .btn-secondary .btn-block}
:::

::: {.banner-examples-text style="margin-top: 1rem;"}
**直撃！デザインパターン**を電子書籍で購入し、IDE
で開くことができる数多くの詳細な例を含む書庫にアクセス。

[ 詳細](../../book.html){.btn .btn-secondary .btn-block}
:::
::::::
:::::::
::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::
