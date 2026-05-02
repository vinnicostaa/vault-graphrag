::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} /
[デザインパターン](../../../design-patterns.html){.type} /
[Proxy](../../proxy.html){.type} / [Swift](../../swift.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Proxy](../../../../images/patterns/cards/proxy-minid8d7.png?id=25890b11e7dc5af29625ccd0678b63a8){srcset="/images/patterns/cards/proxy-mini-2x.png?id=8638fac9dc08c992852492f9cb29d9c6 2x"}
:::

# **Proxy** を Swift で {#proxy-を-swift-で .pattern-example-title-block-title}
::::

:::::::::::::: pattern-example-body
::: pattern-example-brief
**Proxy** は[、]{.chpule2}[
]{.chpuri2}構造に関するデザインパターンの一つで[、]{.chpule2}[
]{.chpuri2}クライアントが使う本物のサービス・オブジェクトの代理として機能するオブジェクト[
]{.chpule1}[（]{.chpuri1}プロキシー[）]{.chpule2}[
]{.chpuri2}を提供します[。]{.chpule2}[
]{.chpuri2}プロキシーは[、]{.chpule2}[
]{.chpuri2}アクセス制御[、]{.chpule2}[
]{.chpuri2}キャッシングなど[、]{.chpule2}[
]{.chpuri2}何らかの作業を行なった後[、]{.chpule2}[
]{.chpuri2}リクエストをサービス・オブジェクトに渡します[。]{.chpule2}[
]{.chpuri2}

プロキシー・オブジェクトはサービスと同じインターフェースを持ち[、]{.chpule2}[
]{.chpuri2}クライアントにとっては[、]{.chpule2}[
]{.chpuri2}本物のオブジェクトと交換可能です[。]{.chpule2}[ ]{.chpuri2}
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Proxy の詳細 ](../../proxy.html){.btn .btn-lg .btn-primary}
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

**使用例[：]{.chpule2}[ ]{.chpuri2}** Proxy パターンは[、]{.chpule2}[
]{.chpuri2}ほとんどの Swift アプリケーションにおいては[、]{.chpule2}[
]{.chpuri2}あまり見かけませんが[、]{.chpule2}[
]{.chpuri2}いくつかの特殊なケースでは便利です[。]{.chpule2}[
]{.chpuri2}何らかの既存クラスのオブジェクトに何らかの振る舞いを追加したいがクライアント・コードには手を加えたくない時[、]{.chpule2}[
]{.chpuri2}かけがえのないものです[。]{.chpule2}[ ]{.chpuri2}

**見つけ方[：]{.chpule2}[ ]{.chpuri2}**プロキシーは[、]{.chpule2}[
]{.chpuri2}実際の作業はすべて他のオブジェクトに委任します[。]{.chpule2}[
]{.chpuri2}プロキシーがサービスのサブクラスである場合を除き[、]{.chpule2}[
]{.chpuri2}プロキシーのメソッドのそれぞれは[、]{.chpule2}[
]{.chpuri2}最終的にはサービス・オブジェクトを参照するはずです[。]{.chpule2}[
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

この例は[、]{.chpule2}[ ]{.chpuri2}**Proxy**
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

#### []{#example-0--Output-txt .anchor} **Output.txt:** 実行結果

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
## 現実的な例 {#example-1-title}

#### []{#example-1--Example-swift .anchor} **Example.swift:** 現実的な例

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

#### []{#example-1--Output-txt .anchor} **Output.txt:** 実行結果

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
[概念的な例](#example-0){#example-0-tab-bottom .nav-item .nav-link
.active toggle="tab" role="tab" aria-controls="example-0"
aria-selected="true"} [現実的な例](#example-1){#example-1-tab-bottom
.nav-item .nav-link toggle="tab" role="tab" aria-controls="example-1"
aria-selected="true"}
:::

::: next
#### 次を読む

[Chain of Responsibility を Swift で []{.fa
.fa-arrow-right}](../../chain-of-responsibility/swift/example.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### 戻る

[[]{.fa .fa-arrow-left} Flyweight を Swift
で](../../flyweight/swift/example.html){.btn .btn-default rel="prev"}
:::

## 他言語での **Proxy**

[![Proxy を C#
で](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Proxy を C# で"){.prog-lang-link}
[![Proxy を C++
で](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Proxy を C++ で"){.prog-lang-link}
[![Proxy を Go
で](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Proxy を Go で"){.prog-lang-link}
[![Proxy を Java
で](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Proxy を Java で"){.prog-lang-link}
[![Proxy を PHP
で](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Proxy を PHP で"){.prog-lang-link}
[![Proxy を Python
で](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Proxy を Python で"){.prog-lang-link}
[![Proxy を Ruby
で](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Proxy を Ruby で"){.prog-lang-link}
[![Proxy を Rust
で](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Proxy を Rust で"){.prog-lang-link}
[![Proxy を TypeScript
で](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Proxy を TypeScript で"){.prog-lang-link}
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
