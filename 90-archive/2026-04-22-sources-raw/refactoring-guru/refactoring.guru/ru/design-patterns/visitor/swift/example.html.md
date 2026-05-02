::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Паттерны
проектирования](../../../design-patterns.html){.type} /
[Посетитель](../../visitor.html){.type} /
[Swift](../../swift.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Посетитель](../../../../images/patterns/cards/visitor-mini0a1c.png?id=854a35a62963bec1d75eab996918989b){srcset="/images/patterns/cards/visitor-mini-2x.png?id=9b87f3f3b772f434b28a25876829b504 2x"}
:::

# **Посетитель** на Swift {#посетитель-на-swift .pattern-example-title-block-title}
::::

:::::::::::::: pattern-example-body
::: pattern-example-brief
**Посетитель** --- это поведенческий паттерн, который позволяет добавить
новую операцию для целой иерархии классов, не изменяя код этих классов.

> Подробней о том, почему Посетитель нельзя заменить простой перегрузкой
> методов читайте в статье [Посетитель и Double
> Dispatch](../../visitor-double-dispatch.html).
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Подробней о паттерне Посетитель ](../../visitor.html){.btn .btn-lg
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

**Применимость:** Посетитель нечасто встречается в Swift-коде из-за
своей сложности и нюансов реализазации.
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

Этот пример показывает структуру паттерна **Посетитель**, а именно ---
из каких классов он состоит, какие роли эти классы выполняют и как они
взаимодействуют друг с другом.

После ознакомления со структурой, вам будет легче воспринимать второй
пример, который рассматривает реальный случай использования паттерна в
мире Swift.

#### []{#example-0--Example-swift .anchor} **Example.swift:** Пример структуры паттерна

<figure class="code">
<pre class="code" lang="swift"><code>import XCTest

/// Интерфейс Компонента объявляет метод принятия, который в качестве аргумента
/// может получать любой объект, реализующий интерфейс посетителя.
protocol Component {

    func accept(_ visitor: Visitor)
}

/// Каждый Конкретный Компонент должен реализовать метод принятия таким образом,
/// чтобы он вызывал метод посетителя, соотвествующий классу компонента.
class ConcreteComponentA: Component {

    /// Обратите внимание, мы вызываем visitConcreteComponentA, что
    /// соответствует названию текущего класса. Таким образом мы позволяем
    /// посетителю узнать, с каким классом компонента он работает.
    func accept(_ visitor: Visitor) {
        visitor.visitConcreteComponentA(element: self)
    }

    /// Конкретные Компоненты могут иметь особые методы, не объявленные в их
    /// базовом классе или интерфейсе. Посетитель всё же может использовать эти
    /// методы, поскольку он знает о конкретном классе компонента.
    func exclusiveMethodOfConcreteComponentA() -&gt; String {
        return &quot;A&quot;
    }
}

class ConcreteComponentB: Component {

    /// То же самое здесь: visitConcreteComponentB =&gt; ConcreteComponentB
    func accept(_ visitor: Visitor) {
        visitor.visitConcreteComponentB(element: self)
    }

    func specialMethodOfConcreteComponentB() -&gt; String {
        return &quot;B&quot;
    }
}

/// Интерфейс Посетителя объявляет набор методов посещения, соответствующих
/// классам компонентов. Сигнатура метода посещения позволяет посетителю
/// определить конкретный класс компонента, с которым он имеет дело.
protocol Visitor {

    func visitConcreteComponentA(element: ConcreteComponentA)
    func visitConcreteComponentB(element: ConcreteComponentB)
}

/// Конкретные Посетители реализуют несколько версий одного и того же алгоритма,
/// которые могут работать со всеми классами конкретных компонентов.
///
/// Максимальную выгоду от паттерна Посетитель вы почувствуете, используя его со
/// сложной структурой объектов, такой как дерево Компоновщика. В этом случае
/// было бы полезно хранить некоторое промежуточное состояние алгоритма при
/// выполнении методов посетителя над различными объектами структуры.
class ConcreteVisitor1: Visitor {

    func visitConcreteComponentA(element: ConcreteComponentA) {
        print(element.exclusiveMethodOfConcreteComponentA() + &quot; + ConcreteVisitor1\n&quot;)
    }

    func visitConcreteComponentB(element: ConcreteComponentB) {
        print(element.specialMethodOfConcreteComponentB() + &quot; + ConcreteVisitor1\n&quot;)
    }
}

class ConcreteVisitor2: Visitor {

    func visitConcreteComponentA(element: ConcreteComponentA) {
        print(element.exclusiveMethodOfConcreteComponentA() + &quot; + ConcreteVisitor2\n&quot;)
    }

    func visitConcreteComponentB(element: ConcreteComponentB) {
        print(element.specialMethodOfConcreteComponentB() + &quot; + ConcreteVisitor2\n&quot;)
    }
}

/// Клиентский код может выполнять операции посетителя над любым набором
/// элементов, не выясняя их конкретных классов. Операция принятия направляет
/// вызов к соответствующей операции в объекте посетителя.
class Client {
    // ...
    static func clientCode(components: [Component], visitor: Visitor) {
        // ...
        components.forEach({ $0.accept(visitor) })
        // ...
    }
    // ...
}

/// Давайте посмотрим как всё это будет работать.
class VisitorConceptual: XCTestCase {

    func test() {
        let components: [Component] = [ConcreteComponentA(), ConcreteComponentB()]

        print(&quot;The client code works with all visitors via the base Visitor interface:\n&quot;)
        let visitor1 = ConcreteVisitor1()
        Client.clientCode(components: components, visitor: visitor1)

        print(&quot;\nIt allows the same client code to work with different types of visitors:\n&quot;)
        let visitor2 = ConcreteVisitor2()
        Client.clientCode(components: components, visitor: visitor2)
    }
}</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** Результат выполнения

<figure class="code">
<pre class="code" lang="output"><code>The client code works with all visitors via the base Visitor interface:

A + ConcreteVisitor1

B + ConcreteVisitor1


It allows the same client code to work with different types of visitors:

A + ConcreteVisitor2

B + ConcreteVisitor2</code></pre>
</figure>
:::

::: {#example-1 .tab-pane role="tabpanel" aria-labelledby="example-1-tab" style="padding-top: 24px"}
## Пример из реальной жизни {#example-1-title}

#### []{#example-1--Example-swift .anchor} **Example.swift:** Пример из реальной жизни

<figure class="code">
<pre class="code" lang="swift"><code>import Foundation
import XCTest


protocol Notification: CustomStringConvertible {

    func accept(visitor: NotificationPolicy) -&gt; Bool
}

struct Email {

    let emailOfSender: String

    var description: String { return &quot;Email&quot; }
}

struct SMS {

    let phoneNumberOfSender: String

    var description: String { return &quot;SMS&quot; }
}

struct Push {

    let usernameOfSender: String

    var description: String { return &quot;Push&quot; }
}

extension Email: Notification {

    func accept(visitor: NotificationPolicy) -&gt; Bool {
        return visitor.isTurnedOn(for: self)
    }
}

extension SMS: Notification {

    func accept(visitor: NotificationPolicy) -&gt; Bool {
        return visitor.isTurnedOn(for: self)
    }
}

extension Push: Notification {

    func accept(visitor: NotificationPolicy) -&gt; Bool {
        return visitor.isTurnedOn(for: self)
    }
}


protocol NotificationPolicy: CustomStringConvertible {

    func isTurnedOn(for email: Email) -&gt; Bool

    func isTurnedOn(for sms: SMS) -&gt; Bool

    func isTurnedOn(for push: Push) -&gt; Bool
}

class NightPolicyVisitor: NotificationPolicy {

    func isTurnedOn(for email: Email) -&gt; Bool {
        return false
    }

    func isTurnedOn(for sms: SMS) -&gt; Bool {
        return true
    }

    func isTurnedOn(for push: Push) -&gt; Bool {
        return false
    }

    var description: String { return &quot;Night Policy Visitor&quot; }
}

class DefaultPolicyVisitor: NotificationPolicy {

    func isTurnedOn(for email: Email) -&gt; Bool {
        return true
    }

    func isTurnedOn(for sms: SMS) -&gt; Bool {
        return true
    }

    func isTurnedOn(for push: Push) -&gt; Bool {
        return true
    }

    var description: String { return &quot;Default Policy Visitor&quot; }
}

class BlackListVisitor: NotificationPolicy {

    private var bannedEmails = [String]()
    private var bannedPhones = [String]()
    private var bannedUsernames = [String]()

    init(emails: [String], phones: [String], usernames: [String]) {
        self.bannedEmails = emails
        self.bannedPhones = phones
        self.bannedUsernames = usernames
    }

    func isTurnedOn(for email: Email) -&gt; Bool {
        return bannedEmails.contains(email.emailOfSender)
    }

    func isTurnedOn(for sms: SMS) -&gt; Bool {
        return bannedPhones.contains(sms.phoneNumberOfSender)
    }

    func isTurnedOn(for push: Push) -&gt; Bool {
        return bannedUsernames.contains(push.usernameOfSender)
    }

    var description: String { return &quot;Black List Visitor&quot; }
}



class VisitorRealWorld: XCTestCase {

    func testVisitorRealWorld() {

        let email = Email(emailOfSender: &quot;some@email.com&quot;)
        let sms = SMS(phoneNumberOfSender: &quot;+3806700000&quot;)
        let push = Push(usernameOfSender: &quot;Spammer&quot;)

        let notifications: [Notification] = [email, sms, push]

        clientCode(handle: notifications, with: DefaultPolicyVisitor())

        clientCode(handle: notifications, with: NightPolicyVisitor())
    }
}

extension VisitorRealWorld {

    /// Client code traverses notifications with visitors and checks whether a
    /// notification is in a blacklist and should be shown in accordance with a
    /// current SilencePolicy

    func clientCode(handle notifications: [Notification], with policy: NotificationPolicy) {

        let blackList = createBlackList()

        print(&quot;\nClient: Using \(policy.description) and \(blackList.description)&quot;)

        notifications.forEach { item in

            guard !item.accept(visitor: blackList) else {
                print(&quot;\tWARNING: &quot; + item.description + &quot; is in a black list&quot;)
                return
            }

            if item.accept(visitor: policy) {
                print(&quot;\t&quot; + item.description + &quot; notification will be shown&quot;)
            } else {
                print(&quot;\t&quot; + item.description + &quot; notification will be silenced&quot;)
            }
        }
    }

    private func createBlackList() -&gt; BlackListVisitor {
        return BlackListVisitor(emails: [&quot;banned@email.com&quot;],
                                phones: [&quot;000000000&quot;, &quot;1234325232&quot;],
                                usernames: [&quot;Spammer&quot;])
    }
}</code></pre>
</figure>

#### []{#example-1--Output-txt .anchor} **Output.txt:** Результат выполнения

<figure class="code">
<pre class="code" lang="output"><code>Client: Using Default Policy Visitor and Black List Visitor
    Email notification will be shown
    SMS notification will be shown
    WARNING: Push is in a black list

Client: Using Night Policy Visitor and Black List Visitor
    Email notification will be silenced
    SMS notification will be shown
    WARNING: Push is in a black list</code></pre>
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

[Паттерны проектирования на TypeScript []{.fa
.fa-arrow-right}](../../typescript.html){.btn .btn-primary rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Шаблонный метод на
Swift](../../template-method/swift/example.html){.btn .btn-default
rel="prev"}
:::

## **Посетитель** на других языках программирования

[![Посетитель на
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Посетитель на C#"){.prog-lang-link}
[![Посетитель на
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Посетитель на C++"){.prog-lang-link}
[![Посетитель на
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Посетитель на Go"){.prog-lang-link}
[![Посетитель на
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Посетитель на Java"){.prog-lang-link}
[![Посетитель на
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Посетитель на PHP"){.prog-lang-link}
[![Посетитель на
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Посетитель на Python"){.prog-lang-link}
[![Посетитель на
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Посетитель на Ruby"){.prog-lang-link}
[![Посетитель на
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Посетитель на Rust"){.prog-lang-link}
[![Посетитель на
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Посетитель на TypeScript"){.prog-lang-link}
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
