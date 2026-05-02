::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Padrões de
Projeto](../../../design-patterns.html){.type} /
[Mediator](../../mediator.html){.type} /
[Swift](../../swift.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Mediator](../../../../images/patterns/cards/mediator-mini1561.png?id=a7e43ee8e17e4474737b1fcb3201d7ba){srcset="/images/patterns/cards/mediator-mini-2x.png?id=d288d7c73f5581ae09701d61200ae09e 2x"}
:::

# **Mediator** em Swift {#mediator-em-swift .pattern-example-title-block-title}
::::

:::::::::::::: pattern-example-body
::: pattern-example-brief
O **Mediator** é um padrão de projeto comportamental que reduz o
acoplamento entre os componentes de um programa, fazendo-os se comunicar
indiretamente, por meio de um objeto mediador especial.

O Mediator facilita a modificação, a extensão e a reutilização de
componentes individuais porque eles não são mais dependentes de dezenas
de outras classes.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Saiba mais sobre o Mediator ](../../mediator.html){.btn .btn-lg
.btn-primary}
:::

::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
Navegação
:::

::: en-intro
 [Introdução](#)
:::

::: en-intro
 [Exemplo conceitual](#example-0)
:::

::: en-file
 [Example](#example-0--Example-swift)
:::

::: en-file
 [Output](#example-0--Output-txt)
:::

::: en-intro
 [Exemplo do mundo real](#example-1)
:::

::: en-file
 [Example](#example-1--Example-swift)
:::

::: en-file
 [Output](#example-1--Output-txt)
:::
:::::::::::

**Complexidade:** []{.fa-stars}

**Popularidade:** []{.fa-stars}

**Exemplos de uso:** O uso mais popular do padrão Mediator no código
Swift é facilitar a comunicação entre os componentes de interface do
usuário de uma aplicação. O sinônimo do Mediator é a parte do
Controlador do padrão MVC.
::::::::::::::

::::: {style="padding: 1rem; border: solid 4px #e35e5e; border-radius: 1rem; text-align: center"}
::: {style="margin-bottom: 0.5rem;"}
The following examples are available on [Swift
Playgrounds](https://www.alemohamad.com/playgrounds){style="text-decoration: underline; display: inline-block"
rel="nofollow"}.
:::

::: {style="font-size: 13px;"}
Kudos to [Alejandro
Mohamad](https://www.alemohamad.com/){style="text-decoration: underline;"
rel="nofollow"} for creating the Playground version.
:::
:::::

::: {#nav-tab .nav .nav-tabs .nav-justified role="tablist"}
[Exemplo conceitual](#example-0){#example-0-tab .nav-item .nav-link
.active toggle="tab" role="tab" aria-controls="example-0"
aria-selected="true"} [Exemplo do mundo real](#example-1){#example-1-tab
.nav-item .nav-link toggle="tab" role="tab" aria-controls="example-1"
aria-selected="true"}
:::

::::: {#nav-tabContent .pattern-example-tab-content .tab-content}
::: {#example-0 .tab-pane .show .active role="tabpanel" aria-labelledby="example-0-tab" style="padding-top: 24px"}
## Exemplo conceitual {#example-0-title}

Este exemplo ilustra a estrutura do padrão de projeto **Mediator**. Ele
se concentra em responder a estas perguntas:

-   De quais classes ele consiste?
-   Quais papéis essas classes desempenham?
-   De que maneira os elementos do padrão estão relacionados?

Depois de aprender sobre a estrutura do padrão, será mais fácil entender
o exemplo a seguir, com base em um caso de uso Swift do mundo real.

#### []{#example-0--Example-swift .anchor} **Example.swift:** Exemplo conceitual

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

#### []{#example-0--Output-txt .anchor} **Output.txt:** Resultados da execução

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
## Exemplo do mundo real {#example-1-title}

#### []{#example-1--Example-swift .anchor} **Example.swift:** Exemplo do mundo real

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

#### []{#example-1--Output-txt .anchor} **Output.txt:** Resultados da execução

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
[Exemplo conceitual](#example-0){#example-0-tab-bottom .nav-item
.nav-link .active toggle="tab" role="tab" aria-controls="example-0"
aria-selected="true"} [Exemplo do mundo
real](#example-1){#example-1-tab-bottom .nav-item .nav-link toggle="tab"
role="tab" aria-controls="example-1" aria-selected="true"}
:::

::: next
#### Leia a seguir

[Memento em Swift []{.fa
.fa-arrow-right}](../../memento/swift/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Voltar

[[]{.fa .fa-arrow-left} Iterator em
Swift](../../iterator/swift/example.html){.btn .btn-default rel="prev"}
:::

## **Mediator** em outras linguagens

[![Mediator em
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Mediator em C#"){.prog-lang-link}
[![Mediator em
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Mediator em C++"){.prog-lang-link}
[![Mediator em
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Mediator em Go"){.prog-lang-link}
[![Mediator em
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Mediator em Java"){.prog-lang-link}
[![Mediator em
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Mediator em PHP"){.prog-lang-link}
[![Mediator em
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Mediator em Python"){.prog-lang-link}
[![Mediator em
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Mediator em Ruby"){.prog-lang-link}
[![Mediator em
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Mediator em Rust"){.prog-lang-link}
[![Mediator em
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Mediator em TypeScript"){.prog-lang-link}
::::::::::::::::::::::::::::

::::::: {.prom .banner-sidebar .banner .banner-striped .banner-examples .banner-removable .banner-removable-patterns data-id="DP: Examples-IDE" creative-id="examples-sidebar-pt-br" position="sidebar"}
:::::: {.banner-inner style="font-size: 14px; line-height: 21px;"}
::: banner-examples-img
[![](../../../../images/patterns/banners/examples-ide91ca.png?id=3115b4b548fb96b75974e2de8f4f49bc){width="215"
height="158" loading="lazy"
srcset="/images/patterns/banners/examples-ide-2x.png?id=93c007a6d157b97c28eb213f59d716ae 2x"}](../../book.html)
:::

::: banner-examples-fade
[ Arquivo com exemplos de código](../../book.html){.btn .btn-secondary
.btn-block}
:::

::: {.banner-examples-text style="margin-top: 1rem;"}
Compre o eBook **Mergulho nos Padrões de Projeto** e tenha acesso ao
arquivo com dúzias de exemplos detalhados que podem ser abertos
diretamente em seu IDE.

[ Saiba mais...](../../book.html){.btn .btn-secondary .btn-block}
:::
::::::
:::::::
::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::
