::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Паттерны
проектирования](../../../design-patterns.html){.type} /
[Заместитель](../../proxy.html){.type} /
[Swift](../../swift.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Заместитель](../../../../images/patterns/cards/proxy-minid8d7.png?id=25890b11e7dc5af29625ccd0678b63a8){srcset="/images/patterns/cards/proxy-mini-2x.png?id=8638fac9dc08c992852492f9cb29d9c6 2x"}
:::

# **Заместитель** на Swift {#заместитель-на-swift .pattern-example-title-block-title}
::::

:::::::::::::: pattern-example-body
::: pattern-example-brief
**Заместитель** --- это объект, который выступает прослойкой между
клиентом и реальным сервисным объектом. Заместитель получает вызовы от
клиента, выполняет свою функцию (контроль доступа, кеширование,
изменение запроса и прочее), а затем передаёт вызов сервисному объекту.

Заместитель имеет тот же интерфейс, что и реальный объект, поэтому для
клиента нет разницы --- работать через заместителя или напрямую.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Подробней о паттерне Заместитель ](../../proxy.html){.btn .btn-lg
.btn-primary}
:::

::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
Навигация
:::

::: en-intro
 [Интро](#)
:::

::: en-intro
 [Концептуальный пример](#example-0)
:::

::: en-file
 [Example](#example-0--Example-swift)
:::

::: en-file
 [Output](#example-0--Output-txt)
:::

::: en-intro
 [Пример из реальной жизни](#example-1)
:::

::: en-file
 [Example](#example-1--Example-swift)
:::

::: en-file
 [Output](#example-1--Output-txt)
:::
:::::::::::

**Сложность:** []{.fa-stars}

**Популярность:** []{.fa-stars}

**Применимость:** Паттерн Заместитель применяется в Swift коде тогда,
когда надо заменить настоящий объект его суррогатом, причём незаметно
для клиентов настоящего объекта. Это позволит выполнить какие-то
добавочные поведения до или после основного поведения настоящего
объекта.

**Признаки применения паттерна:** Класс заместителя чаще всего
делегирует всю настоящую работу своему реальному объекту. Заместители
часто сами следят за жизненным циклом своего реального объекта.
::::::::::::::

::::: {style="padding: 1rem; border: solid 4px #e35e5e; border-radius: 1rem; text-align: center"}
::: {style="margin-bottom: 0.5rem;"}
Следующие примеры доступны на [Swift
Playgrounds](https://www.alemohamad.com/playgrounds){style="text-decoration: underline; display: inline-block"
rel="nofollow"}.
:::

::: {style="font-size: 13px;"}
Благодарность [Alejandro
Mohamad](https://www.alemohamad.com/){style="text-decoration: underline;"
rel="nofollow"} за создание версии Playground.
:::
:::::

::: {#nav-tab .nav .nav-tabs .nav-justified role="tablist"}
[Концептуальный пример](#example-0){#example-0-tab .nav-item .nav-link
.active toggle="tab" role="tab" aria-controls="example-0"
aria-selected="true"} [Пример из реальной
жизни](#example-1){#example-1-tab .nav-item .nav-link toggle="tab"
role="tab" aria-controls="example-1" aria-selected="true"}
:::

::::: {#nav-tabContent .pattern-example-tab-content .tab-content}
::: {#example-0 .tab-pane .show .active role="tabpanel" aria-labelledby="example-0-tab" style="padding-top: 24px"}
## Концептуальный пример {#example-0-title}

Этот пример показывает структуру паттерна **Заместитель**, а именно ---
из каких классов он состоит, какие роли эти классы выполняют и как они
взаимодействуют друг с другом.

После ознакомления со структурой, вам будет легче воспринимать второй
пример, который рассматривает реальный случай использования паттерна в
мире Swift.

#### []{#example-0--Example-swift .anchor} **Example.swift:** Пример структуры паттерна

<figure class="code">
<pre class="code" lang="swift"><code>import XCTest

/// Интерфейс Субъекта объявляет общие операции как для Реального Субъекта, так
/// и для Заместителя. Пока клиент работает с Реальным Субъектом, используя этот
/// интерфейс,  вы сможете передать ему заместителя вместо реального субъекта.
protocol Subject {

    func request()
}

/// Реальный Субъект содержит некоторую базовую бизнес-логику. Как правило,
/// Реальные Субъекты способны выполнять некоторую полезную работу, которая к
/// тому же может быть очень медленной или точной – например, коррекция входных
/// данных. Заместитель может решить эти задачи без каких-либо изменений в коде
/// Реального Субъекта.
class RealSubject: Subject {

    func request() {
        print(&quot;RealSubject: Handling request.&quot;)
    }
}

/// Интерфейс Заместителя идентичен интерфейсу Реального Субъекта.
class Proxy: Subject {

    private var realSubject: RealSubject

    /// Заместитель хранит ссылку на объект класса РеальныйСубъект. Клиент может
    /// либо лениво загрузить его, либо передать Заместителю.
    init(_ realSubject: RealSubject) {
        self.realSubject = realSubject
    }

    /// Наиболее распространёнными областями применения паттерна Заместитель
    /// являются ленивая загрузка, кэширование, контроль доступа, ведение
    /// журнала и т.д. Заместитель может выполнить одну из этих задач, а затем,
    /// в зависимости от результата, передать выполнение одноимённому методу в
    /// связанном объекте класса РеальныйСубъект.
    func request() {

        if (checkAccess()) {
            realSubject.request()
            logAccess()
        }
    }

    private func checkAccess() -&gt; Bool {

        /// Некоторые реальные проверки должны проходить здесь.

        print(&quot;Proxy: Checking access prior to firing a real request.&quot;)

        return true
    }

    private func logAccess() {
        print(&quot;Proxy: Logging the time of request.&quot;)
    }
}

/// Клиентский код должен работать со всеми объектами (как с реальными, так и
/// заместителями) через интерфейс Субъекта, чтобы поддерживать как реальные
/// субъекты, так и заместителей. В реальной жизни, однако, клиенты в основном
/// работают с реальными субъектами напрямую. В этом случае, для более простой
/// реализации паттерна, можно расширить заместителя из класса реального
/// субъекта.
class Client {
    // ...
    static func clientCode(subject: Subject) {
        // ...
        subject.request()
        // ...
    }
    // ...
}

/// Давайте посмотрим как всё это будет работать.
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

#### []{#example-0--Output-txt .anchor} **Output.txt:** Результат выполнения

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
## Пример из реальной жизни {#example-1-title}

#### []{#example-1--Example-swift .anchor} **Example.swift:** Пример из реальной жизни

<figure class="code">
<pre class="code" lang="swift"><code>import XCTest

class ProxyRealWorld: XCTestCase {

    /// Паттерн Заместитель
    ///
    /// Назначение: Позволяет подставлять вместо реальных объектов специальные
    /// объекты-заменители. Эти объекты перехватывают вызовы к оригинальному
    /// объекту, позволяя сделать что-то до или после передачи вызова оригиналу.
    ///
    /// Пример: Существует бесчисленное множество направлений, где могут быть
    /// использованы заместители: кэширование, логирование, контроль доступа,
    /// отложенная инициализация и т.д.

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

#### []{#example-1--Output-txt .anchor} **Output.txt:** Результат выполнения

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
[Концептуальный пример](#example-0){#example-0-tab-bottom .nav-item
.nav-link .active toggle="tab" role="tab" aria-controls="example-0"
aria-selected="true"} [Пример из реальной
жизни](#example-1){#example-1-tab-bottom .nav-item .nav-link
toggle="tab" role="tab" aria-controls="example-1" aria-selected="true"}
:::

::: next
#### Читаем дальше

[Цепочка обязанностей на Swift []{.fa
.fa-arrow-right}](../../chain-of-responsibility/swift/example.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Легковес на
Swift](../../flyweight/swift/example.html){.btn .btn-default rel="prev"}
:::

## **Заместитель** на других языках программирования

[![Заместитель на
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Заместитель на C#"){.prog-lang-link}
[![Заместитель на
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Заместитель на C++"){.prog-lang-link}
[![Заместитель на
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Заместитель на Go"){.prog-lang-link}
[![Заместитель на
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Заместитель на Java"){.prog-lang-link}
[![Заместитель на
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Заместитель на PHP"){.prog-lang-link}
[![Заместитель на
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Заместитель на Python"){.prog-lang-link}
[![Заместитель на
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Заместитель на Ruby"){.prog-lang-link}
[![Заместитель на
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Заместитель на Rust"){.prog-lang-link}
[![Заместитель на
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Заместитель на TypeScript"){.prog-lang-link}
::::::::::::::::::::::::::::

::::::: {.prom .banner-sidebar .banner .banner-striped .banner-examples .banner-removable .banner-removable-patterns data-id="DP: Examples-IDE" creative-id="examples-sidebar-ru" position="sidebar"}
:::::: {.banner-inner style="font-size: 14px; line-height: 21px;"}
::: banner-examples-img
[![](../../../../images/patterns/banners/examples-ide91ca.png?id=3115b4b548fb96b75974e2de8f4f49bc){width="215"
height="158" loading="lazy"
srcset="/images/patterns/banners/examples-ide-2x.png?id=93c007a6d157b97c28eb213f59d716ae 2x"}](../../book.html)
:::

::: banner-examples-fade
[ Архив с примерами](../../book.html){.btn .btn-secondary .btn-block}
:::

::: {.banner-examples-text style="margin-top: 1rem;"}
Купи книгу **Погружение в Паттерны** и получи архив с десятками
детальных примеров, которые можно открывать прямо в IDE.

[ Узнать больше...](../../book.html){.btn .btn-secondary .btn-block}
:::
::::::
:::::::
::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::
