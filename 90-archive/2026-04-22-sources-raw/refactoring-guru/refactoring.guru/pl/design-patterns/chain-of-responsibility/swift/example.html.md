::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Wzorce
projektowe](../../../design-patterns.html){.type} / [Łańcuch
zobowiązań](../../chain-of-responsibility.html){.type} /
[Swift](../../swift.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Łańcuch
zobowiązań](../../../../images/patterns/cards/chain-of-responsibility-mini6d42.png?id=36d85eba8d14986f053123de17aac7a7){srcset="/images/patterns/cards/chain-of-responsibility-mini-2x.png?id=8c81f7069e51259b2443801b91135f7f 2x"}
:::

# **Łańcuch zobowiązań** w języku Swift {#łańcuch-zobowiązań-w-języku-swift .pattern-example-title-block-title}
::::

:::::::::::::: pattern-example-body
::: pattern-example-brief
**Łańcuch zobowiązań** to behawioralny wzorzec projektowy pozwalający
przekazywać żądanie wzdłuż łańcucha potencjalnych obiektów
obsługujących aż zostanie obsłużone.

W łańcuchu zobowiązań wiele obiektów może obsłużyć żądanie bez
konieczności sprzęgania klas wysyłających je z konkretnymi klasami
odbierającymi. Łańcuch można układać dynamicznie w trakcie działania
programu z dowolnych obiektów obsługujących, wyposażonych w standardowy
interfejs obsługi żądań.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Dowiedz się więcej o Łańcuch zobowiązań
](../../chain-of-responsibility.html){.btn .btn-lg .btn-primary}
:::

::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
Nawigacja
:::

::: en-intro
 [Intro](#)
:::

::: en-intro
 [Przykład koncepcyjny](#example-0)
:::

::: en-file
 [Example](#example-0--Example-swift)
:::

::: en-file
 [Output](#example-0--Output-txt)
:::

::: en-intro
 [Przykład z prawdziwego życia](#example-1)
:::

::: en-file
 [Example](#example-1--Example-swift)
:::

::: en-file
 [Output](#example-1--Output-txt)
:::
:::::::::::

**Złożoność:** []{.fa-stars}

**Popularność:** []{.fa-stars}

**Przykłady użycia:** Łańcuch zobowiązań jest rzadkim rozwiązaniem w
programach napisanych w Swift, gdyż ma sens tylko w tym kodzie, w którym
występują łańcuchy obiektów.

**Identyfikacja:** Wzorzec można rozpoznać na podstawie obecności
behawioralnych metod jednej grupy obiektów pośrednio wywołujących
analogiczne metody w innych obiektach za pośrednictwem wspólnego
interfejsu.
::::::::::::::

::::: {style="padding: 1rem; border: solid 4px #e35e5e; border-radius: 1rem; text-align: center"}
::: {style="margin-bottom: 0.5rem;"}
Następujące przykłady są dostępne na [Swift
Playgrounds](https://www.alemohamad.com/playgrounds){style="text-decoration: underline; display: inline-block"
rel="nofollow"}.
:::

::: {style="font-size: 13px;"}
Gratulacje dla [Alejandro
Mohamad](https://www.alemohamad.com/){style="text-decoration: underline;"
rel="nofollow"} za stworzenie wersji Playground.
:::
:::::

::: {#nav-tab .nav .nav-tabs .nav-justified role="tablist"}
[Przykład koncepcyjny](#example-0){#example-0-tab .nav-item .nav-link
.active toggle="tab" role="tab" aria-controls="example-0"
aria-selected="true"} [Przykład z prawdziwego
życia](#example-1){#example-1-tab .nav-item .nav-link toggle="tab"
role="tab" aria-controls="example-1" aria-selected="true"}
:::

::::: {#nav-tabContent .pattern-example-tab-content .tab-content}
::: {#example-0 .tab-pane .show .active role="tabpanel" aria-labelledby="example-0-tab" style="padding-top: 24px"}
## Przykład koncepcyjny {#example-0-title}

Poniższy przykład ilustruje strukturę wzorca **Łańcuch zobowiązań** ze
szczególnym naciskiem na następujące kwestie:

-   Z jakich składa się klas?
-   Jakie role pełnią te klasy?
-   W jaki sposób elementy wzorca są ze sobą powiązane?

Poznawszy strukturę wzorca będzie ci łatwiej zrozumieć następujący
przykład, oparty na prawdziwym przypadku użycia Swift.

#### []{#example-0--Example-swift .anchor} **Example.swift:** Przykład koncepcyjny

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

#### []{#example-0--Output-txt .anchor} **Output.txt:** Wynik działania

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
## Przykład z prawdziwego życia {#example-1-title}

#### []{#example-1--Example-swift .anchor} **Example.swift:** Przykład z prawdziwego życia

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

#### []{#example-1--Output-txt .anchor} **Output.txt:** Wynik działania

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
[Przykład koncepcyjny](#example-0){#example-0-tab-bottom .nav-item
.nav-link .active toggle="tab" role="tab" aria-controls="example-0"
aria-selected="true"} [Przykład z prawdziwego
życia](#example-1){#example-1-tab-bottom .nav-item .nav-link
toggle="tab" role="tab" aria-controls="example-1" aria-selected="true"}
:::

::: next
#### Czytaj dalej

[Polecenie w języku Swift []{.fa
.fa-arrow-right}](../../command/swift/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Wróć

[[]{.fa .fa-arrow-left} Pełnomocnik w języku
Swift](../../proxy/swift/example.html){.btn .btn-default rel="prev"}
:::

## **Łańcuch zobowiązań** w innych językach

[![Łańcuch zobowiązań w języku
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Łańcuch zobowiązań w języku C#"){.prog-lang-link}
[![Łańcuch zobowiązań w języku
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Łańcuch zobowiązań w języku C++"){.prog-lang-link}
[![Łańcuch zobowiązań w języku
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Łańcuch zobowiązań w języku Go"){.prog-lang-link}
[![Łańcuch zobowiązań w języku
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Łańcuch zobowiązań w języku Java"){.prog-lang-link}
[![Łańcuch zobowiązań w języku
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Łańcuch zobowiązań w języku PHP"){.prog-lang-link}
[![Łańcuch zobowiązań w języku
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Łańcuch zobowiązań w języku Python"){.prog-lang-link}
[![Łańcuch zobowiązań w języku
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Łańcuch zobowiązań w języku Ruby"){.prog-lang-link}
[![Łańcuch zobowiązań w języku
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Łańcuch zobowiązań w języku Rust"){.prog-lang-link}
[![Łańcuch zobowiązań w języku
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Łańcuch zobowiązań w języku TypeScript"){.prog-lang-link}
::::::::::::::::::::::::::::

::::::: {.prom .banner-sidebar .banner .banner-striped .banner-examples .banner-removable .banner-removable-patterns data-id="DP: Examples-IDE" creative-id="examples-sidebar-pl" position="sidebar"}
:::::: {.banner-inner style="font-size: 14px; line-height: 21px;"}
::: banner-examples-img
[![](../../../../images/patterns/banners/examples-ide91ca.png?id=3115b4b548fb96b75974e2de8f4f49bc){width="215"
height="158" loading="lazy"
srcset="/images/patterns/banners/examples-ide-2x.png?id=93c007a6d157b97c28eb213f59d716ae 2x"}](../../book.html)
:::

::: banner-examples-fade
[ Archiwum\
przykładów kodu](../../book.html){.btn .btn-secondary .btn-block}
:::

::: {.banner-examples-text style="margin-top: 1rem;"}
Kupując eBook **Wzorce projektowe. Nowoczesny podręcznik** zyskujesz
dostęp do archiwum zawierającego mnóstwo szczegółowych przykładów, które
można od razu wkleić do swojego preferowanego zintegrowanego środowiska
programistycznego.

[ Dowiedz się więcej...](../../book.html){.btn .btn-secondary
.btn-block}
:::
::::::
:::::::
::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::
