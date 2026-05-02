::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} /
[デザインパターン](../../../design-patterns.html){.type} /
[Composite](../../composite.html){.type} /
[Swift](../../swift.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Composite](../../../../images/patterns/cards/composite-minib543.png?id=a369d98d18b417f255d04568fd0131b8){srcset="/images/patterns/cards/composite-mini-2x.png?id=3f7f811fefeb0b64f6774746eb42af09 2x"}
:::

# **Composite** を Swift で {#composite-を-swift-で .pattern-example-title-block-title}
::::

:::::::::::::: pattern-example-body
::: pattern-example-brief
**Composite** は[、]{.chpule2}[
]{.chpuri2}構造に関するデザインパターンの一つで[、]{.chpule2}[
]{.chpuri2}オブジェクトを木のような構造に構成し[、]{.chpule2}[
]{.chpuri2}あたかも単一のオブジェクトであるかのように扱えるようにします[。]{.chpule2}[
]{.chpuri2}

Composite は[、]{.chpule2}[
]{.chpuri2}ツリー構造の構築を必要とする問題の大部分の解決策として[、]{.chpule2}[
]{.chpuri2}かなりの人気を得るようになりました[。]{.chpule2}[
]{.chpuri2}Composite の大きな特徴は[、]{.chpule2}[
]{.chpuri2}ツリー構造全体でメソッドを再帰的に実行し[、]{.chpule2}[
]{.chpuri2}結果をまとめあげることです[。]{.chpule2}[ ]{.chpuri2}
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Composite の詳細 ](../../composite.html){.btn .btn-lg .btn-primary}
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

**使用例[：]{.chpule2}[ ]{.chpuri2}** Composite
パターンは[、]{.chpule2}[ ]{.chpuri2}Swift
コードではよく見かけます[。]{.chpule2}[
]{.chpuri2}ユーザー・インターフェースのコンポーネントの階層構造の表現や[、]{.chpule2}[
]{.chpuri2}グラフに対して動作するコードで多く使われます[。]{.chpule2}[
]{.chpuri2}

**見つけ方[：]{.chpule2}[ ]{.chpuri2}**
オブジェクト・ツリーがあって[、]{.chpule2}[
]{.chpuri2}ツリーのそれぞれのオブジェクトが同じクラス階層の部分であれば[、]{.chpule2}[
]{.chpuri2}十中八九[、]{.chpule2}[
]{.chpuri2}コンポジットです[。]{.chpule2}[
]{.chpuri2}もしこれらのクラス中のメソッドが[、]{.chpule2}[
]{.chpuri2}ツリーの子オブジェクトに仕事を委任し[、]{.chpule2}[
]{.chpuri2}それを階層の基底クラスやインターフェースを介して行うなら[、]{.chpule2}[
]{.chpuri2}これは間違いなくコンポジットです[。]{.chpule2}[ ]{.chpuri2}
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

この例は[、]{.chpule2}[ ]{.chpuri2}**Composite**
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

#### []{#example-0--Output-txt .anchor} **Output.txt:** 実行結果

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
## 現実的な例 {#example-1-title}

#### []{#example-1--Example-swift .anchor} **Example.swift:** 現実的な例

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

#### []{#example-1--Output-txt .anchor} **Output.txt:** 実行結果

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
[概念的な例](#example-0){#example-0-tab-bottom .nav-item .nav-link
.active toggle="tab" role="tab" aria-controls="example-0"
aria-selected="true"} [現実的な例](#example-1){#example-1-tab-bottom
.nav-item .nav-link toggle="tab" role="tab" aria-controls="example-1"
aria-selected="true"}
:::

::: next
#### 次を読む

[Decorator を Swift で []{.fa
.fa-arrow-right}](../../decorator/swift/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### 戻る

[[]{.fa .fa-arrow-left} Bridge を Swift
で](../../bridge/swift/example.html){.btn .btn-default rel="prev"}
:::

## 他言語での **Composite**

[![Composite を C#
で](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Composite を C# で"){.prog-lang-link}
[![Composite を C++
で](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Composite を C++ で"){.prog-lang-link}
[![Composite を Go
で](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Composite を Go で"){.prog-lang-link}
[![Composite を Java
で](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Composite を Java で"){.prog-lang-link}
[![Composite を PHP
で](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Composite を PHP で"){.prog-lang-link}
[![Composite を Python
で](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Composite を Python で"){.prog-lang-link}
[![Composite を Ruby
で](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Composite を Ruby で"){.prog-lang-link}
[![Composite を Rust
で](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Composite を Rust で"){.prog-lang-link}
[![Composite を TypeScript
で](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Composite を TypeScript で"){.prog-lang-link}
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
