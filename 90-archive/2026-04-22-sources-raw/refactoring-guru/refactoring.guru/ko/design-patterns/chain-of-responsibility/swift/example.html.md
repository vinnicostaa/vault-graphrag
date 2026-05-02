:::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [디자인
패턴들](../../../design-patterns.html){.type} / [책임
연쇄](../../chain-of-responsibility.html){.type} /
[스위프트](../../swift.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![책임
연쇄](../../../../images/patterns/cards/chain-of-responsibility-mini6d42.png?id=36d85eba8d14986f053123de17aac7a7){srcset="/images/patterns/cards/chain-of-responsibility-mini-2x.png?id=8c81f7069e51259b2443801b91135f7f 2x"}
:::

# 스위프트로 작성된 **책임 연쇄** {#스위프트로-작성된-책임-연쇄 .pattern-example-title-block-title}
::::

:::::::::::::: pattern-example-body
::: pattern-example-brief
**책임 연쇄** 패턴은 핸들러 중 하나가 요청을 처리할 때까지 핸들러들의
체인​(사슬)​을 따라 요청을 전달할 수 있게 해주는 행동 디자인 패턴입니다.

이 패턴은 발신자 클래스를 수신자들의 구상 클래스들에 연결하지 않고도
여러 객체가 요청을 처리할 수 있도록 합니다. 체인은 표준 핸들러
인터페이스를 따르는 모든 핸들러와 런타임 때 동적으로 구성될 수 있습니다.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ 책임 연쇄에 대하여 더 자세히 알아보세요
](../../chain-of-responsibility.html){.btn .btn-lg .btn-primary}
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

**사용 예시들:** 책임 연쇄 패턴은 스위프트 코드에 매우 일반적이며,
당신의 코드가 필터, 이벤터 체인 등과 같은 객체 체인과 함께 작동할 때
특히 유용합니다.

**식별:** 패턴의 모든 객체는 공통 인터페이스를 따르며, 다른 객체들의
같은 메서드들을 간접적으로 호출하는 한 객체 그룹의 행동 메서드들이
있습니다.
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

이 예시는 **책임 연쇄** 패턴의 구조를 보여주고 다음 질문에 중점을
둡니다:

-   패턴은 어떤 클래스들로 구성되어 있나요?
-   이 클래스들은 어떤 역할을 하나요?
-   패턴의 요소들은 어떻게 서로 연관되어 있나요?

이 패턴의 구조를 배우면 실제 스위프트 사용 사례를 기반으로 하는 다음
예시를 더욱 쉽게 이해할 수 있을 것입니다.

#### []{#example-0--Example-swift .anchor} **Example.swift:** 개념적인 예시

<figure class="code">
<pre class="code" lang="swift"><code>import XCTest

/// The Handler interface declares a method for building the chain of handlers.
/// It also declares a method for executing a request.
protocol Handler: AnyObject {

    @discardableResult
    func setNext(handler: Handler) -&gt; Handler

    func handle(request: String) -&gt; String?

    var nextHandler: Handler? { get set }
}

extension Handler {

    func setNext(handler: Handler) -&gt; Handler {
        self.nextHandler = handler

        /// Returning a handler from here will let us link handlers in a
        /// convenient way like this:
        /// monkey.setNext(handler: squirrel).setNext(handler: dog)
        return handler
    }

    func handle(request: String) -&gt; String? {
        return nextHandler?.handle(request: request)
    }
}

/// All Concrete Handlers either handle a request or pass it to the next handler
/// in the chain.
class MonkeyHandler: Handler {

    var nextHandler: Handler?

    func handle(request: String) -&gt; String? {
        if (request == &quot;Banana&quot;) {
            return &quot;Monkey: I&#39;ll eat the &quot; + request + &quot;.\n&quot;
        } else {
            return nextHandler?.handle(request: request)
        }
    }
}

class SquirrelHandler: Handler {

    var nextHandler: Handler?

    func handle(request: String) -&gt; String? {

        if (request == &quot;Nut&quot;) {
            return &quot;Squirrel: I&#39;ll eat the &quot; + request + &quot;.\n&quot;
        } else {
            return nextHandler?.handle(request: request)
        }
    }
}

class DogHandler: Handler {

    var nextHandler: Handler?

    func handle(request: String) -&gt; String? {
        if (request == &quot;MeatBall&quot;) {
            return &quot;Dog: I&#39;ll eat the &quot; + request + &quot;.\n&quot;
        } else {
            return nextHandler?.handle(request: request)
        }
    }
}

/// The client code is usually suited to work with a single handler. In most
/// cases, it is not even aware that the handler is part of a chain.
class Client {
    // ...
    static func someClientCode(handler: Handler) {

        let food = [&quot;Nut&quot;, &quot;Banana&quot;, &quot;Cup of coffee&quot;]

        for item in food {

            print(&quot;Client: Who wants a &quot; + item + &quot;?\n&quot;)

            guard let result = handler.handle(request: item) else {
                print(&quot;  &quot; + item + &quot; was left untouched.\n&quot;)
                return
            }

            print(&quot;  &quot; + result)
        }
    }
    // ...
}

/// Let&#39;s see how it all works together.
class ChainOfResponsibilityConceptual: XCTestCase {
 
    func test() {

        /// The other part of the client code constructs the actual chain.

        let monkey = MonkeyHandler()
        let squirrel = SquirrelHandler()
        let dog = DogHandler()
        monkey.setNext(handler: squirrel).setNext(handler: dog)

        /// The client should be able to send a request to any handler, not just
        /// the first one in the chain.

        print(&quot;Chain: Monkey &gt; Squirrel &gt; Dog\n\n&quot;)
        Client.someClientCode(handler: monkey)
        print()
        print(&quot;Subchain: Squirrel &gt; Dog\n\n&quot;)
        Client.someClientCode(handler: squirrel)
    }
}</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** 실행 결과

<figure class="code">
<pre class="code" lang="output"><code>Chain: Monkey &gt; Squirrel &gt; Dog


Client: Who wants a Nut?

Squirrel: I&#39;ll eat the Nut.

Client: Who wants a Banana?

Monkey: I&#39;ll eat the Banana.

Client: Who wants a Cup of coffee?

Cup of coffee was left untouched.


Subchain: Squirrel &gt; Dog


Client: Who wants a Nut?

Squirrel: I&#39;ll eat the Nut.

Client: Who wants a Banana?

Banana was left untouched.</code></pre>
</figure>
:::

::: {#example-1 .tab-pane role="tabpanel" aria-labelledby="example-1-tab" style="padding-top: 24px"}
## 실제 사례 예시 {#example-1-title}

#### []{#example-1--Example-swift .anchor} **Example.swift:** 실제 사례 예시

<figure class="code">
<pre class="code" lang="swift"><code>import Foundation
import UIKit
import XCTest


protocol Handler {

    var next: Handler? { get }

    func handle(_ request: Request) -&gt; LocalizedError?
}

class BaseHandler: Handler {

    var next: Handler?

    init(with handler: Handler? = nil) {
        self.next = handler
    }

    func handle(_ request: Request) -&gt; LocalizedError? {
        return next?.handle(request)
    }
}

class LoginHandler: BaseHandler {

    override func handle(_ request: Request) -&gt; LocalizedError? {

        guard request.email?.isEmpty == false else {
            return AuthError.emptyEmail
        }

        guard request.password?.isEmpty == false else {
            return AuthError.emptyPassword
        }

        return next?.handle(request)
    }
}

class SignUpHandler: BaseHandler {

    private struct Limit {
        static let passwordLength = 8
    }

    override func handle(_ request: Request) -&gt; LocalizedError? {

        guard request.email?.contains(&quot;@&quot;) == true else {
            return AuthError.invalidEmail
        }

        guard (request.password?.count ?? 0) &gt;= Limit.passwordLength else {
            return AuthError.invalidPassword
        }

        guard request.password == request.repeatedPassword else {
            return AuthError.differentPasswords
        }

        return next?.handle(request)
    }
}

class LocationHandler: BaseHandler {

    override func handle(_ request: Request) -&gt; LocalizedError? {
        guard isLocationEnabled() else {
            return AuthError.locationDisabled
        }
        return next?.handle(request)
    }

    func isLocationEnabled() -&gt; Bool {
        return true /// Calls special method
    }
}

class NotificationHandler: BaseHandler {

    override func handle(_ request: Request) -&gt; LocalizedError? {
        guard isNotificationsEnabled() else {
            return AuthError.notificationsDisabled
        }
        return next?.handle(request)
    }

    func isNotificationsEnabled() -&gt; Bool {
        return false /// Calls special method
    }
}

enum AuthError: LocalizedError {

    case emptyFirstName
    case emptyLastName

    case emptyEmail
    case emptyPassword

    case invalidEmail
    case invalidPassword
    case differentPasswords

    case locationDisabled
    case notificationsDisabled

    var errorDescription: String? {
        switch self {
        case .emptyFirstName:
            return &quot;First name is empty&quot;
        case .emptyLastName:
            return &quot;Last name is empty&quot;
        case .emptyEmail:
            return &quot;Email is empty&quot;
        case .emptyPassword:
            return &quot;Password is empty&quot;
        case .invalidEmail:
            return &quot;Email is invalid&quot;
        case .invalidPassword:
            return &quot;Password is invalid&quot;
        case .differentPasswords:
            return &quot;Password and repeated password should be equal&quot;
        case .locationDisabled:
            return &quot;Please turn location services on&quot;
        case .notificationsDisabled:
            return &quot;Please turn notifications on&quot;
        }
    }
}


protocol Request {

    var firstName: String? { get }
    var lastName: String? { get }

    var email: String? { get }
    var password: String? { get }
    var repeatedPassword: String? { get }
}

extension Request {

    /// Default implementations

    var firstName: String? { return nil }
    var lastName: String? { return nil }

    var email: String? { return nil }
    var password: String? { return nil }
    var repeatedPassword: String? { return nil }
}

struct SignUpRequest: Request {

    var firstName: String?
    var lastName: String?

    var email: String?
    var password: String?
    var repeatedPassword: String?
}

struct LoginRequest: Request {

    var email: String?
    var password: String?
}



protocol AuthHandlerSupportable: AnyObject {

    var handler: Handler? { get set }
}

class BaseAuthViewController: UIViewController, AuthHandlerSupportable {

    /// Base class or extensions can be used to implement a base behavior
    var handler: Handler?

    init(handler: Handler) {
        self.handler = handler
        super.init(nibName: nil, bundle: nil)
    }

    required init?(coder aDecoder: NSCoder) {
        super.init(coder: aDecoder)
    }
}

class LoginViewController: BaseAuthViewController {

    func loginButtonSelected() {
        print(&quot;Login View Controller: User selected Login button&quot;)

        let request = LoginRequest(email: &quot;smth@gmail.com&quot;, password: &quot;123HardPass&quot;)

        if let error = handler?.handle(request) {
            print(&quot;Login View Controller: something went wrong&quot;)
            print(&quot;Login View Controller: Error -&gt; &quot; + (error.errorDescription ?? &quot;&quot;))
        } else {
            print(&quot;Login View Controller: Preconditions are successfully validated&quot;)
        }
    }
}

class SignUpViewController: BaseAuthViewController {

    func signUpButtonSelected() {
        print(&quot;SignUp View Controller: User selected SignUp button&quot;)

        let request = SignUpRequest(firstName: &quot;Vasya&quot;,
                                    lastName: &quot;Pupkin&quot;,
                                    email: &quot;vasya.pupkin@gmail.com&quot;,
                                    password: &quot;123HardPass&quot;,
                                    repeatedPassword: &quot;123HardPass&quot;)

        if let error = handler?.handle(request) {
            print(&quot;SignUp View Controller: something went wrong&quot;)
            print(&quot;SignUp View Controller: Error -&gt; &quot; + (error.errorDescription ?? &quot;&quot;))
        } else {
            print(&quot;SignUp View Controller: Preconditions are successfully validated&quot;)
        }
    }
}



class ChainOfResponsibilityRealWorld: XCTestCase {

    func testChainOfResponsibilityRealWorld() {

        print(&quot;Client: Let&#39;s test Login flow!&quot;)

        let loginHandler = LoginHandler(with: LocationHandler())
        let loginController = LoginViewController(handler: loginHandler)

        loginController.loginButtonSelected()

        print(&quot;\nClient: Let&#39;s test SignUp flow!&quot;)

        let signUpHandler = SignUpHandler(with: LocationHandler(with: NotificationHandler()))
        let signUpController = SignUpViewController(handler: signUpHandler)

        signUpController.signUpButtonSelected()
    }
}</code></pre>
</figure>

#### []{#example-1--Output-txt .anchor} **Output.txt:** 실행 결과

<figure class="code">
<pre class="code" lang="output"><code>Client: Let&#39;s test Login flow!
Login View Controller: User selected Login button
Login View Controller: Preconditions are successfully validated

Client: Let&#39;s test SignUp flow!
SignUp View Controller: User selected SignUp button
SignUp View Controller: something went wrong
SignUp View Controller: Error -&gt; Please turn notifications on</code></pre>
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

[스위프트로 작성된 커맨드 []{.fa
.fa-arrow-right}](../../command/swift/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### 돌아가기

[[]{.fa .fa-arrow-left} 스위프트로 작성된
프록시](../../proxy/swift/example.html){.btn .btn-default rel="prev"}
:::

## 다른 언어로 작성된 **책임 연쇄**

[![C#으로 작성된 책임
연쇄](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "C#으로 작성된 책임 연쇄"){.prog-lang-link}
[![C++로 작성된 책임
연쇄](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "C++로 작성된 책임 연쇄"){.prog-lang-link}
[![Go로 작성된 책임
연쇄](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Go로 작성된 책임 연쇄"){.prog-lang-link}
[![자바로 작성된 책임
연쇄](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "자바로 작성된 책임 연쇄"){.prog-lang-link}
[![PHP로 작성된 책임
연쇄](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "PHP로 작성된 책임 연쇄"){.prog-lang-link}
[![파이썬으로 작성된 책임
연쇄](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "파이썬으로 작성된 책임 연쇄"){.prog-lang-link}
[![루비로 작성된 책임
연쇄](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "루비로 작성된 책임 연쇄"){.prog-lang-link}
[![러스트로 작성된 책임
연쇄](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "러스트로 작성된 책임 연쇄"){.prog-lang-link}
[![타입스크립트로 작성된 책임
연쇄](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "타입스크립트로 작성된 책임 연쇄"){.prog-lang-link}
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
