::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Паттерны
проектирования](../../../design-patterns.html){.type} /
[Компоновщик](../../composite.html){.type} /
[Swift](../../swift.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Компоновщик](../../../../images/patterns/cards/composite-minib543.png?id=a369d98d18b417f255d04568fd0131b8){srcset="/images/patterns/cards/composite-mini-2x.png?id=3f7f811fefeb0b64f6774746eb42af09 2x"}
:::

# **Компоновщик** на Swift {#компоновщик-на-swift .pattern-example-title-block-title}
::::

:::::::::::::: pattern-example-body
::: pattern-example-brief
**Компоновщик** --- это структурный паттерн, который позволяет создавать
дерево объектов и работать с ним так же, как и с единичным объектом.

Компоновщик давно стал синонимом всех задач, связанных с построением
дерева объектов. Все операции компоновщика основаны на рекурсии и
«суммировании» результатов на ветвях дерева.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Подробней о паттерне Компоновщик ](../../composite.html){.btn .btn-lg
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

**Применимость:** Паттерн Компоновщик встречается в любых задачах,
которые связаны с построением дерева. Самый простой пример --- составные
элементы GUI, которые тоже можно рассматривать как дерево.

**Признаки применения паттерна:** Если из объектов строится древовидная
структура, и со всеми объектами дерева, как и с самим деревом работают
через общий интерфейс.
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

Этот пример показывает структуру паттерна **Компоновщик**, а именно ---
из каких классов он состоит, какие роли эти классы выполняют и как они
взаимодействуют друг с другом.

После ознакомления со структурой, вам будет легче воспринимать второй
пример, который рассматривает реальный случай использования паттерна в
мире Swift.

#### []{#example-0--Example-swift .anchor} **Example.swift:** Пример структуры паттерна

<figure class="code">
<pre class="code" lang="swift"><code>import XCTest

/// Базовый класс Компонент объявляет общие операции как для простых, так и для
/// сложных объектов структуры.
protocol Component {

    /// При необходимости базовый Компонент может объявить интерфейс для
    /// установки и получения родителя компонента в древовидной структуре. Он
    /// также может предоставить  некоторую реализацию по умолчанию для этих
    /// методов.
    var parent: Component? { get set }

    /// В некоторых случаях целесообразно определить операции управления
    /// потомками прямо в базовом классе Компонент. Таким образом, вам не нужно
    /// будет предоставлять  конкретные классы компонентов клиентскому коду,
    /// даже во время сборки дерева объектов. Недостаток такого подхода в том,
    /// что эти методы будут пустыми для компонентов уровня листа.
    func add(component: Component)
    func remove(component: Component)

    /// Вы можете предоставить метод, который позволит клиентскому коду понять,
    /// может ли компонент иметь вложенные объекты.
    func isComposite() -&gt; Bool

    /// Базовый Компонент может сам реализовать некоторое поведение по умолчанию
    /// или поручить это конкретным классам.
    func operation() -&gt; String
}

extension Component {

    func add(component: Component) {}
    func remove(component: Component) {}
    func isComposite() -&gt; Bool {
        return false
    }
}

/// Класс Лист представляет собой конечные объекты структуры.  Лист не может
/// иметь вложенных компонентов.
///
/// Обычно объекты Листьев выполняют фактическую работу, тогда как объекты
/// Контейнера лишь делегируют работу своим подкомпонентам.
class Leaf: Component {

    var parent: Component?

    func operation() -&gt; String {
        return &quot;Leaf&quot;
    }
}

/// Класс Контейнер содержит сложные компоненты, которые могут иметь вложенные
/// компоненты. Обычно объекты Контейнеры делегируют фактическую работу своим
/// детям, а затем «суммируют» результат.
class Composite: Component {

    var parent: Component?

    /// Это поле содержит поддерево компонентов.
    private var children = [Component]()

    /// Объект контейнера может как добавлять компоненты в свой список вложенных
    /// компонентов, так и удалять их, как простые, так и сложные.
    func add(component: Component) {
        var item = component
        item.parent = self
        children.append(item)
    }

    func remove(component: Component) {
        // ...
    }

    func isComposite() -&gt; Bool {
        return true
    }

    /// Контейнер выполняет свою основную логику особым образом. Он проходит
    /// рекурсивно через всех своих детей, собирая и суммируя их результаты.
    /// Поскольку потомки контейнера передают эти вызовы своим потомкам и так
    /// далее,  в результате обходится всё дерево объектов.
    func operation() -&gt; String {
        let result = children.map({ $0.operation() })
        return &quot;Branch(&quot; + result.joined(separator: &quot; &quot;) + &quot;)&quot;
    }
}

class Client {

    /// Клиентский код работает со всеми компонентами через базовый интерфейс.
    static func someClientCode(component: Component) {
        print(&quot;Result: &quot; + component.operation())
    }

    /// Благодаря тому, что операции управления потомками объявлены в базовом
    /// классе Компонента, клиентский код может работать как с простыми, так и
    /// со сложными компонентами.
    static func moreComplexClientCode(leftComponent: Component, rightComponent: Component) {
        if leftComponent.isComposite() {
            leftComponent.add(component: rightComponent)
        }
        print(&quot;Result: &quot; + leftComponent.operation())
    }
}

/// Давайте посмотрим как всё это будет работать.
class CompositeConceptual: XCTestCase {

    func testCompositeConceptual() {

        /// Таким образом, клиентский код может поддерживать простые компоненты-
        /// листья...
        print(&quot;Client: I&#39;ve got a simple component:&quot;)
        Client.someClientCode(component: Leaf())

        /// ...а также сложные контейнеры.
        let tree = Composite()

        let branch1 = Composite()
        branch1.add(component: Leaf())
        branch1.add(component: Leaf())

        let branch2 = Composite()
        branch2.add(component: Leaf())
        branch2.add(component: Leaf())

        tree.add(component: branch1)
        tree.add(component: branch2)

        print(&quot;\nClient: Now I&#39;ve got a composite tree:&quot;)
        Client.someClientCode(component: tree)

        print(&quot;\nClient: I don&#39;t need to check the components classes even when managing the tree:&quot;)
        Client.moreComplexClientCode(leftComponent: tree, rightComponent: Leaf())
    }
}</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** Результат выполнения

<figure class="code">
<pre class="code" lang="output"><code>Client: I&#39;ve got a simple component:
Result: Leaf

Client: Now I&#39;ve got a composite tree:
Result: Branch(Branch(Leaf Leaf) Branch(Leaf Leaf))

Client: I don&#39;t need to check the components classes even when managing the tree:
Result: Branch(Branch(Leaf Leaf) Branch(Leaf Leaf) Leaf)</code></pre>
</figure>
:::

::: {#example-1 .tab-pane role="tabpanel" aria-labelledby="example-1-tab" style="padding-top: 24px"}
## Пример из реальной жизни {#example-1-title}

#### []{#example-1--Example-swift .anchor} **Example.swift:** Пример из реальной жизни

<figure class="code">
<pre class="code" lang="swift"><code>import UIKit
import XCTest

protocol Component {

    func accept&lt;T: Theme&gt;(theme: T)
}

extension Component where Self: UIViewController {

    func accept&lt;T: Theme&gt;(theme: T) {
        view.accept(theme: theme)
        view.subviews.forEach({ $0.accept(theme: theme) })
    }
}

extension UIView: Component {}
extension UIViewController: Component {}

extension Component where Self: UIView {

    func accept&lt;T: Theme&gt;(theme: T) {

        print(&quot;\t\(description): has applied \(theme.description)&quot;)

        backgroundColor = theme.backgroundColor
    }
}

extension Component where Self: UILabel {

    func accept&lt;T: LabelTheme&gt;(theme: T) {

        print(&quot;\t\(description): has applied \(theme.description)&quot;)

        backgroundColor = theme.backgroundColor
        textColor = theme.textColor
    }
}

extension Component where Self: UIButton {

    func accept&lt;T: ButtonTheme&gt;(theme: T) {

        print(&quot;\t\(description): has applied \(theme.description)&quot;)

        backgroundColor = theme.backgroundColor
        setTitleColor(theme.textColor, for: .normal)
        setTitleColor(theme.highlightedColor, for: .highlighted)
    }
}


protocol Theme: CustomStringConvertible {

    var backgroundColor: UIColor { get }
}

protocol ButtonTheme: Theme {

    var textColor: UIColor { get }

    var highlightedColor: UIColor { get }

    /// other properties
}

protocol LabelTheme: Theme {

    var textColor: UIColor { get }

    /// other properties
}

/// Button Themes

struct DefaultButtonTheme: ButtonTheme {

    var textColor = UIColor.red

    var highlightedColor = UIColor.white

    var backgroundColor = UIColor.orange

    var description: String { return &quot;Default Buttom Theme&quot; }
}

struct NightButtonTheme: ButtonTheme {

    var textColor = UIColor.white

    var highlightedColor = UIColor.red

    var backgroundColor = UIColor.black

    var description: String { return &quot;Night Buttom Theme&quot; }
}

/// Label Themes

struct DefaultLabelTheme: LabelTheme {

    var textColor = UIColor.red

    var backgroundColor = UIColor.black

    var description: String { return &quot;Default Label Theme&quot; }
}

struct NightLabelTheme: LabelTheme {

    var textColor = UIColor.white

    var backgroundColor = UIColor.black

    var description: String { return &quot;Night Label Theme&quot; }
}



class CompositeRealWorld: XCTestCase {

    func testCompositeRealWorld() {

        print(&quot;\nClient: Applying &#39;default&#39; theme for &#39;UIButton&#39;&quot;)
        apply(theme: DefaultButtonTheme(), for: UIButton())

        print(&quot;\nClient: Applying &#39;night&#39; theme for &#39;UIButton&#39;&quot;)
        apply(theme: NightButtonTheme(), for: UIButton())


        print(&quot;\nClient: Let&#39;s use View Controller as a composite!&quot;)

        /// Night theme
        print(&quot;\nClient: Applying &#39;night button&#39; theme for &#39;WelcomeViewController&#39;...&quot;)
        apply(theme: NightButtonTheme(), for: WelcomeViewController())
        print()

        print(&quot;\nClient: Applying &#39;night label&#39; theme for &#39;WelcomeViewController&#39;...&quot;)
        apply(theme: NightLabelTheme(), for: WelcomeViewController())
        print()

        /// Default Theme
        print(&quot;\nClient: Applying &#39;default button&#39; theme for &#39;WelcomeViewController&#39;...&quot;)
        apply(theme: DefaultButtonTheme(), for: WelcomeViewController())
        print()

        print(&quot;\nClient: Applying &#39;default label&#39; theme for &#39;WelcomeViewController&#39;...&quot;)
        apply(theme: DefaultLabelTheme(), for: WelcomeViewController())
        print()
    }

    func apply&lt;T: Theme&gt;(theme: T, for component: Component) {
        component.accept(theme: theme)
    }
}

class WelcomeViewController: UIViewController {

    class ContentView: UIView {

        var titleLabel = UILabel()
        var actionButton = UIButton()

        override init(frame: CGRect) {
            super.init(frame: frame)
            setup()
        }

        required init?(coder decoder: NSCoder) {
            super.init(coder: decoder)
            setup()
        }

        func setup() {
            addSubview(titleLabel)
            addSubview(actionButton)
        }
    }

    override func loadView() {
        view = ContentView()
    }
}

/// Let&#39;s override a description property for the better output

extension WelcomeViewController {

    open override var description: String { return &quot;WelcomeViewController&quot; }
}

extension WelcomeViewController.ContentView {

    override var description: String { return &quot;ContentView&quot; }
}

extension UIButton {

    open override var description: String { return &quot;UIButton&quot; }
}

extension UILabel {

    open override var description: String { return &quot;UILabel&quot; }
}</code></pre>
</figure>

#### []{#example-1--Output-txt .anchor} **Output.txt:** Результат выполнения

<figure class="code">
<pre class="code" lang="output"><code>Client: Applying &#39;default&#39; theme for &#39;UIButton&#39;
UIButton: has applied Default Buttom Theme

Client: Applying &#39;night&#39; theme for &#39;UIButton&#39;
UIButton: has applied Night Buttom Theme

Client: Let&#39;s use View Controller as a composite!

Client: Applying &#39;night button&#39; theme for &#39;WelcomeViewController&#39;...
ContentView: has applied Night Buttom Theme
UILabel: has applied Night Buttom Theme
UIButton: has applied Night Buttom Theme</code></pre>
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

[Декоратор на Swift []{.fa
.fa-arrow-right}](../../decorator/swift/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Мост на
Swift](../../bridge/swift/example.html){.btn .btn-default rel="prev"}
:::

## **Компоновщик** на других языках программирования

[![Компоновщик на
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Компоновщик на C#"){.prog-lang-link}
[![Компоновщик на
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Компоновщик на C++"){.prog-lang-link}
[![Компоновщик на
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Компоновщик на Go"){.prog-lang-link}
[![Компоновщик на
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Компоновщик на Java"){.prog-lang-link}
[![Компоновщик на
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Компоновщик на PHP"){.prog-lang-link}
[![Компоновщик на
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Компоновщик на Python"){.prog-lang-link}
[![Компоновщик на
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Компоновщик на Ruby"){.prog-lang-link}
[![Компоновщик на
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Компоновщик на Rust"){.prog-lang-link}
[![Компоновщик на
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Компоновщик на TypeScript"){.prog-lang-link}
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
