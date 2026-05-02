::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Патерни
проектування](../../../design-patterns.html){.type} /
[Одинак](../../singleton.html){.type} / [Swift](../../swift.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Одинак](../../../../images/patterns/cards/singleton-minif559.png?id=914e1565dfdf15f240e766163bd303ec){srcset="/images/patterns/cards/singleton-mini-2x.png?id=4a7348083d7719d02709a5c9ff878b9f 2x"}
:::

# **Одинак** на Swift {#одинак-на-swift .pattern-example-title-block-title}
::::

:::::::::::::: pattern-example-body
::: pattern-example-brief
**Одинак** --- це породжуючий патерн, який гарантує існування тільки
одного об'єкта певного класу, а також дозволяє дістатися цього об'єкта з
будь-якого місця програми.

Одинак має такі ж переваги та недоліки, що і глобальні змінні. Його
неймовірно зручно використовувати, але він порушує модульність вашого
коду.

Ви не зможете просто взяти і використовувати клас, залежний від одинака,
в іншій програмі. Для цього доведеться емулювати там присутність
одинака. Найчастіше ця проблема проявляється при написанні юніт-тестів.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Детальніше про Одинак ](../../singleton.html){.btn .btn-lg
.btn-primary}
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

**Застосування:** Багато програмістів вважають Одинака антипатерном,
тому його все рідше і рідше можна зустріти в Swift-коді.

**Ознаки застосування патерна:** Одинака можна визначити за статичним
створюючим методом, який повертає один і той же об'єкт.
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

Цей приклад показує структуру патерна **Одинак**, а саме --- з яких
класів він складається, які ролі ці класи виконують і як вони
взаємодіють один з одним.

Після ознайомлення зі структурою, вам буде легше сприймати наступний
приклад, що розглядає реальний випадок використання патерна в світі
Swift.

#### []{#example-0--Example-swift .anchor} **Example.swift:** Приклад структури патерна

<figure class="code">
<pre class="code" lang="swift"><code>import XCTest

/// The Singleton class defines the `shared` field that lets clients access the
/// unique singleton instance.
class Singleton {

    /// The static field that controls the access to the singleton instance.
    ///
    /// This implementation let you extend the Singleton class while keeping
    /// just one instance of each subclass around.
    static var shared: Singleton = {
        let instance = Singleton()
        // ... configure the instance
        // ...
        return instance
    }()

    /// The Singleton&#39;s initializer should always be private to prevent direct
    /// construction calls with the `new` operator.
    private init() {}

    /// Finally, any singleton should define some business logic, which can be
    /// executed on its instance.
    func someBusinessLogic() -&gt; String {
        // ...
        return &quot;Result of the &#39;someBusinessLogic&#39; call&quot;
    }
}

/// Singletons should not be cloneable.
extension Singleton: NSCopying {

    func copy(with zone: NSZone? = nil) -&gt; Any {
        return self
    }
}

/// The client code.
class Client {
    // ...
    static func someClientCode() {
        let instance1 = Singleton.shared
        let instance2 = Singleton.shared

        if (instance1 === instance2) {
            print(&quot;Singleton works, both variables contain the same instance.&quot;)
        } else {
            print(&quot;Singleton failed, variables contain different instances.&quot;)
        }
    }
    // ...
}

/// Let&#39;s see how it all works together.
class SingletonConceptual: XCTestCase {

    func testSingletonConceptual() {
        Client.someClientCode()
    }
}</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** Результат виконання

<figure class="code">
<pre class="code" lang="output"><code>Singleton works, both variables contain the same instance.</code></pre>
</figure>
:::

::: {#example-1 .tab-pane role="tabpanel" aria-labelledby="example-1-tab" style="padding-top: 24px"}
## Життєвий приклад {#example-1-title}

#### []{#example-1--Example-swift .anchor} **Example.swift:** Життєвий приклад

<figure class="code">
<pre class="code" lang="swift"><code>import XCTest

/// Singleton Design Pattern
///
/// Intent: Ensure that class has a single instance, and provide a global point
/// of access to it.

class SingletonRealWorld: XCTestCase {

    func testSingletonRealWorld() {

        /// There are two view controllers.
        ///
        /// MessagesListVC displays a list of last messages from a user&#39;s chats.
        /// ChatVC displays a chat with a friend.
        ///
        /// FriendsChatService fetches messages from a server and provides all
        /// subscribers (view controllers in our example) with new and removed
        /// messages.
        ///
        /// FriendsChatService is used by both view controllers. It can be
        /// implemented as an instance of a class as well as a global variable.
        ///
        /// In this example, it is important to have only one instance that
        /// performs resource-intensive work.

        let listVC = MessagesListVC()
        let chatVC = ChatVC()

        listVC.startReceiveMessages()
        chatVC.startReceiveMessages()

        /// ... add view controllers to the navigation stack ...
    }
}


class BaseVC: UIViewController, MessageSubscriber {

    func accept(new messages: [Message]) {
        /// Handles new messages in the base class
    }

    func accept(removed messages: [Message]) {
        /// Hanldes removed messages in the base class
    }

    func startReceiveMessages() {

        /// The singleton can be injected as a dependency. However, from an
        /// informational perspective, this example calls FriendsChatService
        /// directly to illustrate the intent of the pattern, which is: &quot;...to
        /// provide the global point of access to the instance...&quot;

        FriendsChatService.shared.add(subscriber: self)
    }
}

class MessagesListVC: BaseVC {

    override func accept(new messages: [Message]) {
        print(&quot;MessagesListVC accepted &#39;new messages&#39;&quot;)
        /// Handles new messages in the child class
    }

    override func accept(removed messages: [Message]) {
        print(&quot;MessagesListVC accepted &#39;removed messages&#39;&quot;)
        /// Handles removed messages in the child class
    }

    override func startReceiveMessages() {
        print(&quot;MessagesListVC starts receive messages&quot;)
        super.startReceiveMessages()
    }
}

class ChatVC: BaseVC {

    override func accept(new messages: [Message]) {
        print(&quot;ChatVC accepted &#39;new messages&#39;&quot;)
        /// Handles new messages in the child class
    }

    override func accept(removed messages: [Message]) {
        print(&quot;ChatVC accepted &#39;removed messages&#39;&quot;)
        /// Handles removed messages in the child class
    }

    override func startReceiveMessages() {
        print(&quot;ChatVC starts receive messages&quot;)
        super.startReceiveMessages()
    }
}

/// Protocol for call-back events

protocol MessageSubscriber {

    func accept(new messages: [Message])
    func accept(removed messages: [Message])
}

/// Protocol for communication with a message service

protocol MessageService {

    func add(subscriber: MessageSubscriber)
}

/// Message domain model

struct Message {

    let id: Int
    let text: String
}


class FriendsChatService: MessageService {

    static let shared = FriendsChatService()

    private var subscribers = [MessageSubscriber]()

    func add(subscriber: MessageSubscriber) {

        /// In this example, fetching starts again by adding a new subscriber
        subscribers.append(subscriber)

        /// Please note, the first subscriber will receive messages again when
        /// the second subscriber is added
        startFetching()
    }

    func startFetching() {

        /// Set up the network stack, establish a connection...
        /// ...and retrieve data from a server

        let newMessages = [Message(id: 0, text: &quot;Text0&quot;),
                           Message(id: 5, text: &quot;Text5&quot;),
                           Message(id: 10, text: &quot;Text10&quot;)]

        let removedMessages = [Message(id: 1, text: &quot;Text0&quot;)]

        /// Send updated data to subscribers
        receivedNew(messages: newMessages)
        receivedRemoved(messages: removedMessages)
    }
}

private extension FriendsChatService {

    func receivedNew(messages: [Message]) {

        subscribers.forEach { item in
            item.accept(new: messages)
        }
    }

    func receivedRemoved(messages: [Message]) {

        subscribers.forEach { item in
            item.accept(removed: messages)
        }
    }
}</code></pre>
</figure>

#### []{#example-1--Output-txt .anchor} **Output.txt:** Результат виконання

<figure class="code">
<pre class="code" lang="output"><code>MessagesListVC starts receive messages
MessagesListVC accepted &#39;new messages&#39;
MessagesListVC accepted &#39;removed messages&#39;
======== At this point, the second subscriber is added ======
ChatVC starts receive messages
MessagesListVC accepted &#39;new messages&#39;
ChatVC accepted &#39;new messages&#39;
MessagesListVC accepted &#39;removed messages&#39;
ChatVC accepted &#39;removed messages&#39;</code></pre>
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

[Адаптер на Swift []{.fa
.fa-arrow-right}](../../adapter/swift/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Повернутись назад

[[]{.fa .fa-arrow-left} Прототип на
Swift](../../prototype/swift/example.html){.btn .btn-default rel="prev"}
:::

## **Одинак** іншими мовами програмування

[![Одинак на
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Одинак на C#"){.prog-lang-link}
[![Одинак на
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Одинак на C++"){.prog-lang-link}
[![Одинак на
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Одинак на Go"){.prog-lang-link}
[![Одинак на
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Одинак на Java"){.prog-lang-link}
[![Одинак на
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Одинак на PHP"){.prog-lang-link}
[![Одинак на
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Одинак на Python"){.prog-lang-link}
[![Одинак на
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Одинак на Ruby"){.prog-lang-link}
[![Одинак на
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Одинак на Rust"){.prog-lang-link}
[![Одинак на
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Одинак на TypeScript"){.prog-lang-link}
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
