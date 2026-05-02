::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} /
[デザインパターン](../../../design-patterns.html){.type} /
[Mediator](../../mediator.html){.type} /
[Swift](../../swift.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Mediator](../../../../images/patterns/cards/mediator-mini1561.png?id=a7e43ee8e17e4474737b1fcb3201d7ba){srcset="/images/patterns/cards/mediator-mini-2x.png?id=d288d7c73f5581ae09701d61200ae09e 2x"}
:::

# **Mediator** を Swift で {#mediator-を-swift-で .pattern-example-title-block-title}
::::

:::::::::::::: pattern-example-body
::: pattern-example-brief
**Mediator** は[、]{.chpule2}[
]{.chpuri2}振る舞いに関するデザインパターンの一つで[、]{.chpule2}[
]{.chpuri2}プログラムのコンポーネント間の通信を特別なメディエーター・オブジェクトを通して行うことで[、]{.chpule2}[
]{.chpuri2}結合を疎にします[。]{.chpule2}[ ]{.chpuri2}

Mediator により[、]{.chpule2}[
]{.chpuri2}個々のコンポーネントは[、]{.chpule2}[
]{.chpuri2}何十ものクラスへの依存がなくなるため[、]{.chpule2}[
]{.chpuri2}変更[、]{.chpule2}[ ]{.chpuri2}拡張[、]{.chpule2}[
]{.chpuri2}再利用が容易になります[。]{.chpule2}[ ]{.chpuri2}
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Mediator の詳細 ](../../mediator.html){.btn .btn-lg .btn-primary}
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

**使用例[：]{.chpule2}[ ]{.chpuri2}** Swift コードで Mediator
パターンがよく使われるのは[、]{.chpule2}[ ]{.chpuri2}アプリの GUI
コンポーネント間の通信のやりとりです[。]{.chpule2}[ ]{.chpuri2}Mediator
は[、]{.chpule2}[ ]{.chpuri2}MVC パターンの C の部分[、]{.chpule2}[
]{.chpuri2}Controller と同義語です[。]{.chpule2}[ ]{.chpuri2}
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

この例は[、]{.chpule2}[ ]{.chpuri2}**Mediator**
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

/// The Mediator interface declares a method used by components to notify the
/// mediator about various events. The Mediator may react to these events and
/// pass the execution to other components.
protocol Mediator: AnyObject {

    func notify(sender: BaseComponent, event: String)
}

/// Concrete Mediators implement cooperative behavior by coordinating several
/// components.
class ConcreteMediator: Mediator {

    private var component1: Component1
    private var component2: Component2

    init(_ component1: Component1, _ component2: Component2) {
        self.component1 = component1
        self.component2 = component2

        component1.update(mediator: self)
        component2.update(mediator: self)
    }

    func notify(sender: BaseComponent, event: String) {
        if event == &quot;A&quot; {
            print(&quot;Mediator reacts on A and triggers following operations:&quot;)
            self.component2.doC()
        }
        else if (event == &quot;D&quot;) {
            print(&quot;Mediator reacts on D and triggers following operations:&quot;)
            self.component1.doB()
            self.component2.doC()
        }
    }
}

/// The Base Component provides the basic functionality of storing a mediator&#39;s
/// instance inside component objects.
class BaseComponent {

    fileprivate weak var mediator: Mediator?

    init(mediator: Mediator? = nil) {
        self.mediator = mediator
    }

    func update(mediator: Mediator) {
        self.mediator = mediator
    }
}

/// Concrete Components implement various functionality. They don&#39;t depend on
/// other components. They also don&#39;t depend on any concrete mediator classes.
class Component1: BaseComponent {

    func doA() {
        print(&quot;Component 1 does A.&quot;)
        mediator?.notify(sender: self, event: &quot;A&quot;)
    }

    func doB() {
        print(&quot;Component 1 does B.\n&quot;)
        mediator?.notify(sender: self, event: &quot;B&quot;)
    }
}

class Component2: BaseComponent {

    func doC() {
        print(&quot;Component 2 does C.&quot;)
        mediator?.notify(sender: self, event: &quot;C&quot;)
    }

    func doD() {
        print(&quot;Component 2 does D.&quot;)
        mediator?.notify(sender: self, event: &quot;D&quot;)
    }
}

/// Let&#39;s see how it all works together.
class MediatorConceptual: XCTestCase {

    func testMediatorConceptual() {

        let component1 = Component1()
        let component2 = Component2()

        let mediator = ConcreteMediator(component1, component2)
        print(&quot;Client triggers operation A.&quot;)
        component1.doA()

        print(&quot;\nClient triggers operation D.&quot;)
        component2.doD()

        print(mediator)
    }
}</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** 実行結果

<figure class="code">
<pre class="code" lang="output"><code>Client triggers operation A.
Component 1 does A.
Mediator reacts on A and triggers following operations:
Component 2 does C.

Client triggers operation D.
Component 2 does D.
Mediator reacts on D and triggers following operations:
Component 1 does B.

Component 2 does C.</code></pre>
</figure>
:::

::: {#example-1 .tab-pane role="tabpanel" aria-labelledby="example-1-tab" style="padding-top: 24px"}
## 現実的な例 {#example-1-title}

#### []{#example-1--Example-swift .anchor} **Example.swift:** 現実的な例

<figure class="code">
<pre class="code" lang="swift"><code>import XCTest

class MediatorRealWorld: XCTestCase {

    func test() {

        let newsArray = [News(id: 1, title: &quot;News1&quot;, likesCount: 1),
                         News(id: 2, title: &quot;News2&quot;, likesCount: 2)]

        let numberOfGivenLikes = newsArray.reduce(0, { $0 + $1.likesCount })

        let mediator = ScreenMediator()

        let feedVC = NewsFeedViewController(mediator, newsArray)
        let newsDetailVC = NewsDetailViewController(mediator, newsArray.first!)
        let profileVC = ProfileViewController(mediator, numberOfGivenLikes)

        mediator.update([feedVC, newsDetailVC, profileVC])

        feedVC.userLikedAllNews()
        feedVC.userDislikedAllNews()
    }
}

class NewsFeedViewController: ScreenUpdatable {

    private var newsArray: [News]
    private weak var mediator: ScreenUpdatable?

    init(_ mediator: ScreenUpdatable?, _ newsArray: [News]) {
        self.newsArray = newsArray
        self.mediator = mediator
    }

    func likeAdded(to news: News) {

        print(&quot;News Feed: Received a liked news model with id \(news.id)&quot;)

        for var item in newsArray {
            if item == news {
                item.likesCount += 1
            }
        }
    }

    func likeRemoved(from news: News) {

        print(&quot;News Feed: Received a disliked news model with id \(news.id)&quot;)

        for var item in newsArray {
            if item == news {
                item.likesCount -= 1
            }
        }
    }

    func userLikedAllNews() {
        print(&quot;\n\nNews Feed: User LIKED all news models&quot;)
        print(&quot;News Feed: I am telling to mediator about it...\n&quot;)
        newsArray.forEach({ mediator?.likeAdded(to: $0) })
    }

    func userDislikedAllNews() {
        print(&quot;\n\nNews Feed: User DISLIKED all news models&quot;)
        print(&quot;News Feed: I am telling to mediator about it...\n&quot;)
        newsArray.forEach({ mediator?.likeRemoved(from: $0) })
    }
}

class NewsDetailViewController: ScreenUpdatable {

    private var news: News
    private weak var mediator: ScreenUpdatable?

    init(_ mediator: ScreenUpdatable?, _ news: News) {
        self.news = news
        self.mediator = mediator
    }

    func likeAdded(to news: News) {
        print(&quot;News Detail: Received a liked news model with id \(news.id)&quot;)
        if self.news == news {
            self.news.likesCount += 1
        }
    }

    func likeRemoved(from news: News) {
        print(&quot;News Detail: Received a disliked news model with id \(news.id)&quot;)
        if self.news == news {
            self.news.likesCount -= 1
        }
    }
}

class ProfileViewController: ScreenUpdatable {

    private var numberOfGivenLikes: Int
    private weak var mediator: ScreenUpdatable?

    init(_ mediator: ScreenUpdatable?, _ numberOfGivenLikes: Int) {
        self.numberOfGivenLikes = numberOfGivenLikes
        self.mediator = mediator
    }

    func likeAdded(to news: News) {
        print(&quot;Profile: Received a liked news model with id \(news.id)&quot;)
        numberOfGivenLikes += 1
    }

    func likeRemoved(from news: News) {
        print(&quot;Profile: Received a disliked news model with id \(news.id)&quot;)
        numberOfGivenLikes -= 1
    }
}

protocol ScreenUpdatable: class {

    func likeAdded(to news: News)

    func likeRemoved(from news: News)
}

class ScreenMediator: ScreenUpdatable {

    private var screens: [ScreenUpdatable]?

    func update(_ screens: [ScreenUpdatable]) {
        self.screens = screens
    }

    func likeAdded(to news: News) {
        print(&quot;Screen Mediator: Received a liked news model with id \(news.id)&quot;)
        screens?.forEach({ $0.likeAdded(to: news) })
    }

    func likeRemoved(from news: News) {
        print(&quot;ScreenMediator: Received a disliked news model with id \(news.id)&quot;)
        screens?.forEach({ $0.likeRemoved(from: news) })
    }
}

struct News: Equatable {

    let id: Int

    let title: String

    var likesCount: Int

    /// Other properties

    static func == (left: News, right: News) -&gt; Bool {
        return left.id == right.id
    }
}</code></pre>
</figure>

#### []{#example-1--Output-txt .anchor} **Output.txt:** 実行結果

<figure class="code">
<pre class="code" lang="output"><code>News Feed: User LIKED all news models
News Feed: I am telling to mediator about it...

Screen Mediator: Received a liked news model with id 1
News Feed: Received a liked news model with id 1
News Detail: Received a liked news model with id 1
Profile: Received a liked news model with id 1
Screen Mediator: Received a liked news model with id 2
News Feed: Received a liked news model with id 2
News Detail: Received a liked news model with id 2
Profile: Received a liked news model with id 2


News Feed: User DISLIKED all news models
News Feed: I am telling to mediator about it...

ScreenMediator: Received a disliked news model with id 1
News Feed: Received a disliked news model with id 1
News Detail: Received a disliked news model with id 1
Profile: Received a disliked news model with id 1
ScreenMediator: Received a disliked news model with id 2
News Feed: Received a disliked news model with id 2
News Detail: Received a disliked news model with id 2
Profile: Received a disliked news model with id 2</code></pre>
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

[Memento を Swift で []{.fa
.fa-arrow-right}](../../memento/swift/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### 戻る

[[]{.fa .fa-arrow-left} Iterator を Swift
で](../../iterator/swift/example.html){.btn .btn-default rel="prev"}
:::

## 他言語での **Mediator**

[![Mediator を C#
で](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Mediator を C# で"){.prog-lang-link}
[![Mediator を C++
で](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Mediator を C++ で"){.prog-lang-link}
[![Mediator を Go
で](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Mediator を Go で"){.prog-lang-link}
[![Mediator を Java
で](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Mediator を Java で"){.prog-lang-link}
[![Mediator を PHP
で](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Mediator を PHP で"){.prog-lang-link}
[![Mediator を Python
で](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Mediator を Python で"){.prog-lang-link}
[![Mediator を Ruby
で](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Mediator を Ruby で"){.prog-lang-link}
[![Mediator を Rust
で](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Mediator を Rust で"){.prog-lang-link}
[![Mediator を TypeScript
で](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Mediator を TypeScript で"){.prog-lang-link}
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
