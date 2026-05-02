:::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [디자인
패턴들](../../../design-patterns.html){.type} /
[프록시](../../proxy.html){.type} / [스위프트](../../swift.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![프록시](../../../../images/patterns/cards/proxy-minid8d7.png?id=25890b11e7dc5af29625ccd0678b63a8){srcset="/images/patterns/cards/proxy-mini-2x.png?id=8638fac9dc08c992852492f9cb29d9c6 2x"}
:::

# 스위프트로 작성된 **프록시** {#스위프트로-작성된-프록시 .pattern-example-title-block-title}
::::

:::::::::::::: pattern-example-body
::: pattern-example-brief
**프록시**는 클라이언트가 사용하는 실제 서비스 객체를 대신하는 객체를
제공하는 구조 디자인 패턴입니다. 프록시는 클라이언트 요청을 수신하고,
일부 작업​(접근 제어, 캐싱 등)​을 수행한 다음 요청을 서비스 객체에
전달합니다.

프록시 객체는 서비스 객체와 같은 인터페이스를 가지기 때문에 클라이언트에
전달되면 실제 객체와 상호교환이 가능합니다.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ 프록시에 대하여 더 자세히 알아보세요 ](../../proxy.html){.btn .btn-lg
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

**사용 사례들:** 프록시 패턴은 대부분의 스위프트 앱에서 일반적으로
발견되지 않습니다. 그러나 일부 특별한 경우에는 여전히 매우 유용할 수
있습니다. 클라이언트 코드를 변경하지 않고 기존 클래스의 객체에 몇 가지
추가 행동들을 추가해야 할 때 매우 유용합니다.

**식별:** 프록시들은 모든 실제 작업을 다른 객체에 위임합니다. 각 프록시
메서드는 프록시가 서비스 객체의 자식 클래스가 아닌 이상 최종적으로
서비스 객체를 참조해야 합니다.
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

이 예시는 **프록시** 디자인 패턴의 구조를 보여주고 다음 질문에 중점을
둡니다:

-   패턴은 어떤 클래스들로 구성되어 있나요?
-   이 클래스들은 어떤 역할을 하나요?
-   패턴의 요소들은 어떻게 서로 연관되어 있나요?

이 패턴의 구조를 배우면 실제 스위프트 사용 사례를 기반으로 하는 다음
예시를 더욱 쉽게 이해할 수 있을 것입니다.

#### []{#example-0--Example-swift .anchor} **Example.swift:** 개념적인 예시

<figure class="code">
<pre class="code" lang="swift"><code>import XCTest

/// The Subject interface declares common operations for both RealSubject and
/// the Proxy. As long as the client works with RealSubject using this
/// interface, you&#39;ll be able to pass it a proxy instead of a real subject.
protocol Subject {

    func request()
}

/// The RealSubject contains some core business logic. Usually, RealSubjects are
/// capable of doing some useful work which may also be very slow or sensitive -
/// e.g. correcting input data. A Proxy can solve these issues without any
/// changes to the RealSubject&#39;s code.
class RealSubject: Subject {

    func request() {
        print(&quot;RealSubject: Handling request.&quot;)
    }
}

/// The Proxy has an interface identical to the RealSubject.
class Proxy: Subject {

    private var realSubject: RealSubject

    /// The Proxy maintains a reference to an object of the RealSubject class.
    /// It can be either lazy-loaded or passed to the Proxy by the client.
    init(_ realSubject: RealSubject) {
        self.realSubject = realSubject
    }

    /// The most common applications of the Proxy pattern are lazy loading,
    /// caching, controlling the access, logging, etc. A Proxy can perform one
    /// of these things and then, depending on the result, pass the execution to
    /// the same method in a linked RealSubject object.
    func request() {

        if (checkAccess()) {
            realSubject.request()
            logAccess()
        }
    }

    private func checkAccess() -&gt; Bool {

        /// Some real checks should go here.

        print(&quot;Proxy: Checking access prior to firing a real request.&quot;)

        return true
    }

    private func logAccess() {
        print(&quot;Proxy: Logging the time of request.&quot;)
    }
}

/// The client code is supposed to work with all objects (both subjects and
/// proxies) via the Subject interface in order to support both real subjects
/// and proxies. In real life, however, clients mostly work with their real
/// subjects directly. In this case, to implement the pattern more easily, you
/// can extend your proxy from the real subject&#39;s class.
class Client {
    // ...
    static func clientCode(subject: Subject) {
        // ...
        subject.request()
        // ...
    }
    // ...
}

/// Let&#39;s see how it all works together.
class ProxyConceptual: XCTestCase {

    func test() {
        print(&quot;Client: Executing the client code with a real subject:&quot;)
        let realSubject = RealSubject()
        Client.clientCode(subject: realSubject)

        print(&quot;\nClient: Executing the same client code with a proxy:&quot;)
        let proxy = Proxy(realSubject)
        Client.clientCode(subject: proxy)
    }
}</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** 실행 결과

<figure class="code">
<pre class="code" lang="output"><code>Client: Executing the client code with a real subject:
RealSubject: Handling request.

Client: Executing the same client code with a proxy:
Proxy: Checking access prior to firing a real request.
RealSubject: Handling request.
Proxy: Logging the time of request.</code></pre>
</figure>
:::

::: {#example-1 .tab-pane role="tabpanel" aria-labelledby="example-1-tab" style="padding-top: 24px"}
## 실제 사례 예시 {#example-1-title}

#### []{#example-1--Example-swift .anchor} **Example.swift:** 실제 사례 예시

<figure class="code">
<pre class="code" lang="swift"><code>import XCTest

class ProxyRealWorld: XCTestCase {

    /// Proxy Design Pattern
    ///
    /// Intent: Provide a surrogate or placeholder for another object to control
    /// access to the original object or to add other responsibilities.
    ///
    /// Example: There are countless ways proxies can be used: caching, logging,
    /// access control, delayed initialization, etc.

    func testProxyRealWorld() {

        print(&quot;Client: Loading a profile WITHOUT proxy&quot;)
        loadBasicProfile(with: Keychain())
        loadProfileWithBankAccount(with: Keychain())

        print(&quot;\nClient: Let&#39;s load a profile WITH proxy&quot;)
        loadBasicProfile(with: ProfileProxy())
        loadProfileWithBankAccount(with: ProfileProxy())
    }

    func loadBasicProfile(with service: ProfileService) {

        service.loadProfile(with: [.basic], success: { profile in
            print(&quot;Client: Basic profile is loaded&quot;)
        }) { error in
            print(&quot;Client: Cannot load a basic profile&quot;)
            print(&quot;Client: Error: &quot; + error.localizedSummary)
        }
    }

    func loadProfileWithBankAccount(with service: ProfileService) {

        service.loadProfile(with: [.basic, .bankAccount], success: { profile in
            print(&quot;Client: Basic profile with a bank account is loaded&quot;)
        }) { error in
            print(&quot;Client: Cannot load a profile with a bank account&quot;)
            print(&quot;Client: Error: &quot; + error.localizedSummary)
        }
    }
}

enum AccessField {

    case basic
    case bankAccount
}

protocol ProfileService {

    typealias Success = (Profile) -&gt; ()
    typealias Failure = (LocalizedError) -&gt; ()

    func loadProfile(with fields: [AccessField], success: Success, failure: Failure)
}

class ProfileProxy: ProfileService {

    private let keychain = Keychain()

    func loadProfile(with fields: [AccessField], success: Success, failure: Failure) {

        if let error = checkAccess(for: fields) {
            failure(error)
        } else {
            /// Note:
            /// At this point, the `success` and `failure` closures can be
            /// passed directly to the original service (as it is now) or
            /// expanded here to handle a result (for example, to cache).

            keychain.loadProfile(with: fields, success: success, failure: failure)
        }
    }

    private func checkAccess(for fields: [AccessField]) -&gt; LocalizedError? {
        if fields.contains(.bankAccount) {
            switch BiometricsService.checkAccess() {
            case .authorized: return nil
            case .denied: return ProfileError.accessDenied
            }
        }
        return nil
    }
}

class Keychain: ProfileService {

    func loadProfile(with fields: [AccessField], success: Success, failure: Failure) {

        var profile = Profile()

        for item in fields {
            switch item {
            case .basic:
                let info = loadBasicProfile()
                profile.firstName = info[Profile.Keys.firstName.raw]
                profile.lastName = info[Profile.Keys.lastName.raw]
                profile.email = info[Profile.Keys.email.raw]
            case .bankAccount:
                profile.bankAccount = loadBankAccount()
            }
        }

        success(profile)
    }

    private func loadBasicProfile() -&gt; [String : String] {
        /// Gets these fields from a secure storage.
        return [Profile.Keys.firstName.raw : &quot;Vasya&quot;,
                Profile.Keys.lastName.raw : &quot;Pupkin&quot;,
                Profile.Keys.email.raw : &quot;vasya.pupkin@gmail.com&quot;]
    }

    private func loadBankAccount() -&gt; BankAccount {
        /// Gets these fields from a secure storage.
        return BankAccount(id: 12345, amount: 999)
    }
}

class BiometricsService {

    enum Access {
        case authorized
        case denied
    }

    static func checkAccess() -&gt; Access {
        /// The service uses Face ID, Touch ID or a plain old password to
        /// determine whether the current user is an owner of the device.

        /// Let&#39;s assume that in our example a user forgot a password :)
        return .denied
    }
}

struct Profile {

    enum Keys: String {
        case firstName
        case lastName
        case email
    }

    var firstName: String?
    var lastName: String?
    var email: String?

    var bankAccount: BankAccount?
}

struct BankAccount {

    var id: Int
    var amount: Double
}

enum ProfileError: LocalizedError {

    case accessDenied

    var errorDescription: String? {
        switch self {
        case .accessDenied:
            return &quot;Access is denied. Please enter a valid password&quot;
        }
    }
}

extension RawRepresentable {

    var raw: Self.RawValue {
        return rawValue
    }
}

extension LocalizedError {

    var localizedSummary: String {
        return errorDescription ?? &quot;&quot;
    }
}</code></pre>
</figure>

#### []{#example-1--Output-txt .anchor} **Output.txt:** 실행 결과

<figure class="code">
<pre class="code" lang="output"><code>Client: Loading a profile WITHOUT proxy
Client: Basic profile is loaded
Client: Basic profile with a bank account is loaded

Client: Let&#39;s load a profile WITH proxy
Client: Basic profile is loaded
Client: Cannot load a profile with a bank account
Client: Error: Access is denied. Please enter a valid password</code></pre>
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

[스위프트로 작성된 책임 연쇄 []{.fa
.fa-arrow-right}](../../chain-of-responsibility/swift/example.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### 돌아가기

[[]{.fa .fa-arrow-left} 스위프트로 작성된
플라이웨이트](../../flyweight/swift/example.html){.btn .btn-default
rel="prev"}
:::

## 다른 언어로 작성된 **프록시**

[![C#으로 작성된
프록시](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "C#으로 작성된 프록시"){.prog-lang-link}
[![C++로 작성된
프록시](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "C++로 작성된 프록시"){.prog-lang-link}
[![Go로 작성된
프록시](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Go로 작성된 프록시"){.prog-lang-link}
[![자바로 작성된
프록시](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "자바로 작성된 프록시"){.prog-lang-link}
[![PHP로 작성된
프록시](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "PHP로 작성된 프록시"){.prog-lang-link}
[![파이썬으로 작성된
프록시](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "파이썬으로 작성된 프록시"){.prog-lang-link}
[![루비로 작성된
프록시](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "루비로 작성된 프록시"){.prog-lang-link}
[![러스트로 작성된
프록시](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "러스트로 작성된 프록시"){.prog-lang-link}
[![타입스크립트로 작성된
프록시](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "타입스크립트로 작성된 프록시"){.prog-lang-link}
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
