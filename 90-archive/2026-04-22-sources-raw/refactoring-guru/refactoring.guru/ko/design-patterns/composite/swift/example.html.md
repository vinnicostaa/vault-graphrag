:::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [디자인
패턴들](../../../design-patterns.html){.type} /
[복합체](../../composite.html){.type} /
[스위프트](../../swift.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![복합체](../../../../images/patterns/cards/composite-minib543.png?id=a369d98d18b417f255d04568fd0131b8){srcset="/images/patterns/cards/composite-mini-2x.png?id=3f7f811fefeb0b64f6774746eb42af09 2x"}
:::

# 스위프트로 작성된 **복합체** {#스위프트로-작성된-복합체 .pattern-example-title-block-title}
::::

:::::::::::::: pattern-example-body
::: pattern-example-brief
**복합체** 패턴은 객체들을 트리 구조들로 구성한 후, 이러한 구조들을 개별
객체들처럼 다룰 수 있도록 하는 구조 패턴입니다.

복합체는 트리 구조를 생성해야 하는 대부분 문제에 대해 인기 있는
해결책입니다. 전체 트리 구조에 대해 재귀적으로 메서드들을 실행하고
결과를 요약하는 기능은 복합체의 훌륭한 기능 중 하나입니다.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ 복합체에 대하여 더 자세히 알아보세요 ](../../composite.html){.btn
.btn-lg .btn-primary}
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

**사용 예시들:** 복합체 패턴은 스위프트 코드에서 매우 일반적입니다.
사용자 인터페이스 컴포넌트의 계층구조나 그래프와 함께 작동하는 코드를
나타내는 데 자주 사용됩니다.

**식별:** 만약 코드에 객체 트리가 있고 트리의 각 객체가 같은 클래스
계층구조의 일부면 이는 복합체 패턴일 가능성이 큽니다. 이러한 클래스들의
메서드들이 작업을 트리의 자식 객체에 위임하고 이러한 위임을 계층구조의
기초 클래스/인터페이스를 통해 수행하면 이는 확실히 복합체입니다.
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

이 예시는 **복합체** 패턴의 구조를 보여주고 다음 질문에 중점을 둡니다:

-   패턴은 어떤 클래스들로 구성되어 있나요?
-   이 클래스들은 어떤 역할을 하나요?
-   패턴의 요소들은 어떻게 서로 연관되어 있나요?

이 패턴의 구조를 배우면 실제 스위프트 사용 사례를 기반으로 하는 다음
예시를 더욱 쉽게 이해할 수 있을 것입니다.

#### []{#example-0--Example-swift .anchor} **Example.swift:** 개념적인 예시

<figure class="code">
<pre class="code" lang="swift"><code>import XCTest

/// The base Component class declares common operations for both simple and
/// complex objects of a composition.
protocol Component {

    /// The base Component may optionally declare methods for setting and
    /// accessing a parent of the component in a tree structure. It can also
    /// provide some default implementation for these methods.
    var parent: Component? { get set }

    /// In some cases, it would be beneficial to define the child-management
    /// operations right in the base Component class. This way, you won&#39;t need
    /// to expose any concrete component classes to the client code, even during
    /// the object tree assembly. The downside is that these methods will be
    /// empty for the leaf-level components.
    func add(component: Component)
    func remove(component: Component)

    /// You can provide a method that lets the client code figure out whether a
    /// component can bear children.
    func isComposite() -&gt; Bool

    /// The base Component may implement some default behavior or leave it to
    /// concrete classes.
    func operation() -&gt; String
}

extension Component {

    func add(component: Component) {}
    func remove(component: Component) {}
    func isComposite() -&gt; Bool {
        return false
    }
}

/// The Leaf class represents the end objects of a composition. A leaf can&#39;t
/// have any children.
///
/// Usually, it&#39;s the Leaf objects that do the actual work, whereas Composite
/// objects only delegate to their sub-components.
class Leaf: Component {

    var parent: Component?

    func operation() -&gt; String {
        return &quot;Leaf&quot;
    }
}

/// The Composite class represents the complex components that may have
/// children. Usually, the Composite objects delegate the actual work to their
/// children and then &quot;sum-up&quot; the result.
class Composite: Component {

    var parent: Component?

    /// This fields contains the conponent subtree.
    private var children = [Component]()

    /// A composite object can add or remove other components (both simple or
    /// complex) to or from its child list.
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

    /// The Composite executes its primary logic in a particular way. It
    /// traverses recursively through all its children, collecting and summing
    /// their results. Since the composite&#39;s children pass these calls to their
    /// children and so forth, the whole object tree is traversed as a result.
    func operation() -&gt; String {
        let result = children.map({ $0.operation() })
        return &quot;Branch(&quot; + result.joined(separator: &quot; &quot;) + &quot;)&quot;
    }
}

class Client {

    /// The client code works with all of the components via the base interface.
    static func someClientCode(component: Component) {
        print(&quot;Result: &quot; + component.operation())
    }

    /// Thanks to the fact that the child-management operations are also
    /// declared in the base Component class, the client code can work with both
    /// simple or complex components.
    static func moreComplexClientCode(leftComponent: Component, rightComponent: Component) {
        if leftComponent.isComposite() {
            leftComponent.add(component: rightComponent)
        }
        print(&quot;Result: &quot; + leftComponent.operation())
    }
}

/// Let&#39;s see how it all comes together.
class CompositeConceptual: XCTestCase {

    func testCompositeConceptual() {

        /// This way the client code can support the simple leaf components...
        print(&quot;Client: I&#39;ve got a simple component:&quot;)
        Client.someClientCode(component: Leaf())

        /// ...as well as the complex composites.
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

#### []{#example-0--Output-txt .anchor} **Output.txt:** 실행 결과

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
## 실제 사례 예시 {#example-1-title}

#### []{#example-1--Example-swift .anchor} **Example.swift:** 실제 사례 예시

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

#### []{#example-1--Output-txt .anchor} **Output.txt:** 실행 결과

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
[개념적인 예시](#example-0){#example-0-tab-bottom .nav-item .nav-link
.active toggle="tab" role="tab" aria-controls="example-0"
aria-selected="true"} [실제 사례 예시](#example-1){#example-1-tab-bottom
.nav-item .nav-link toggle="tab" role="tab" aria-controls="example-1"
aria-selected="true"}
:::

::: next
#### 다음을 보세요

[스위프트로 작성된 데코레이터 []{.fa
.fa-arrow-right}](../../decorator/swift/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### 돌아가기

[[]{.fa .fa-arrow-left} 스위프트로 작성된
브리지](../../bridge/swift/example.html){.btn .btn-default rel="prev"}
:::

## 다른 언어로 작성된 **복합체**

[![C#으로 작성된
복합체](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "C#으로 작성된 복합체"){.prog-lang-link}
[![C++로 작성된
복합체](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "C++로 작성된 복합체"){.prog-lang-link}
[![Go로 작성된
복합체](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Go로 작성된 복합체"){.prog-lang-link}
[![자바로 작성된
복합체](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "자바로 작성된 복합체"){.prog-lang-link}
[![PHP로 작성된
복합체](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "PHP로 작성된 복합체"){.prog-lang-link}
[![파이썬으로 작성된
복합체](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "파이썬으로 작성된 복합체"){.prog-lang-link}
[![루비로 작성된
복합체](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "루비로 작성된 복합체"){.prog-lang-link}
[![러스트로 작성된
복합체](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "러스트로 작성된 복합체"){.prog-lang-link}
[![타입스크립트로 작성된
복합체](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "타입스크립트로 작성된 복합체"){.prog-lang-link}
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
