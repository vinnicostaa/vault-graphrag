:::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} /
[デザインパターン](../../../design-patterns.html){.type} /
[Observer](../../observer.html){.type} / [Go](../../go.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Observer](../../../../images/patterns/cards/observer-minifa2e.png?id=fd2081ab1cff29c60b499bcf6a62786a){srcset="/images/patterns/cards/observer-mini-2x.png?id=f205b0655572ac8e4636691c0e0debfd 2x"}
:::

# **Observer** を Go で {#observer-を-go-で .pattern-example-title-block-title}
::::

::::::::::::::: pattern-example-body
::: pattern-example-brief
**Observer** は[、]{.chpule2}[
]{.chpuri2}振る舞いに関するデザインパターンの一つで[、]{.chpule2}[
]{.chpuri2}オブジェクトが別のオブジェクトに状態の変化を通知できるようにします[。]{.chpule2}[
]{.chpuri2}

Observer パターンは[、]{.chpule2}[
]{.chpuri2}サブスクライバー・インターフェースを実装するいかなるオブジェクトのイベント通知の申し込みと停止をする方法を提供します[。]{.chpule2}[
]{.chpuri2}
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Observer の詳細 ](../../observer.html){.btn .btn-lg .btn-primary}
:::

:::::::::::: {.sidebar-navigation .with-scroll}
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
 [subject](#example-0--subject-go)
:::

::: en-file
 [item](#example-0--item-go)
:::

::: en-file
 [observer](#example-0--observer-go)
:::

::: en-file
 [customer](#example-0--customer-go)
:::

::: en-file
 [main](#example-0--main-go)
:::

::: en-file
 [output](#example-0--output-txt)
:::
::::::::::::
:::::::::::::::

[]{#example-0}

## 概念的な例 {#example-0-title}

電子商取引のウェブサイトでは[、]{.chpule2}[
]{.chpuri2}商品が時々在庫切れになります[。]{.chpule2}[
]{.chpuri2}在庫切れとなった商品に興味を持っていた顧客にどう対処するかという問題には三つの解決策があります[：]{.chpule2}[
]{.chpuri2}

1.  顧客に入荷したかどうかを定期的にチェックしてもらう[。]{.chpule2}[
    ]{.chpuri2}
2.  電子商取引サイトが[、]{.chpule2}[
    ]{.chpuri2}新商品入荷のメールを多数送りつける[。]{.chpule2}[
    ]{.chpuri2}
3.  顧客は興味のある特定商品のみに関して[、]{.chpule2}[
    ]{.chpuri2}入荷したら通知を受けるよう登録する[。]{.chpule2}[
    ]{.chpuri2}複数の顧客が同一商品に通知登録することも可能[。]{.chpule2}[
    ]{.chpuri2}

三つ目の解決策が一番妥当で[、]{.chpule2}[ ]{.chpuri2}これこそが Observer
パターンです[。]{.chpule2}[ ]{.chpuri2}Observer
パターンの主要構成要素は[：]{.chpule2}[ ]{.chpuri2}

-   サブジェクトは[、]{.chpule2}[
    ]{.chpuri2}何かが起きた時にイベントを発行するインスタンスです[。]{.chpule2}[
    ]{.chpuri2}
-   オブザーバーは[、]{.chpule2}[
    ]{.chpuri2}サブジェクトのイベントの通知を申し込み[、]{.chpule2}[
    ]{.chpuri2}イベントが起きると通知されます[。]{.chpule2}[ ]{.chpuri2}

#### []{#example-0--subject-go .anchor} **subject.go:** サブジェクト

<figure class="code">
<pre class="code" lang="go"><code>package main

type Subject interface {
    register(observer Observer)
    deregister(observer Observer)
    notifyAll()
}</code></pre>
</figure>

#### []{#example-0--item-go .anchor} **item.go:** 具象サブジェクト

<figure class="code">
<pre class="code" lang="go"><code>package main

import &quot;fmt&quot;

type Item struct {
    observerList []Observer
    name         string
    inStock      bool
}

func newItem(name string) *Item {
    return &amp;Item{
        name: name,
    }
}
func (i *Item) updateAvailability() {
    fmt.Printf(&quot;Item %s is now in stock\n&quot;, i.name)
    i.inStock = true
    i.notifyAll()
}
func (i *Item) register(o Observer) {
    i.observerList = append(i.observerList, o)
}

func (i *Item) deregister(o Observer) {
    i.observerList = removeFromslice(i.observerList, o)
}

func (i *Item) notifyAll() {
    for _, observer := range i.observerList {
        observer.update(i.name)
    }
}

func removeFromslice(observerList []Observer, observerToRemove Observer) []Observer {
    observerListLength := len(observerList)
    for i, observer := range observerList {
        if observerToRemove.getID() == observer.getID() {
            observerList[observerListLength-1], observerList[i] = observerList[i], observerList[observerListLength-1]
            return observerList[:observerListLength-1]
        }
    }
    return observerList
}</code></pre>
</figure>

#### []{#example-0--observer-go .anchor} **observer.go:** オブザーバー

<figure class="code">
<pre class="code" lang="go"><code>package main

type Observer interface {
    update(string)
    getID() string
}</code></pre>
</figure>

#### []{#example-0--customer-go .anchor} **customer.go:** 具象オブザーバー

<figure class="code">
<pre class="code" lang="go"><code>package main

import &quot;fmt&quot;

type Customer struct {
    id string
}

func (c *Customer) update(itemName string) {
    fmt.Printf(&quot;Sending email to customer %s for item %s\n&quot;, c.id, itemName)
}

func (c *Customer) getID() string {
    return c.id
}</code></pre>
</figure>

#### []{#example-0--main-go .anchor} **main.go:** クライアント・コード

<figure class="code">
<pre class="code" lang="go"><code>package main

func main() {

    shirtItem := newItem(&quot;Nike Shirt&quot;)

    observerFirst := &amp;Customer{id: &quot;abc@gmail.com&quot;}
    observerSecond := &amp;Customer{id: &quot;xyz@gmail.com&quot;}

    shirtItem.register(observerFirst)
    shirtItem.register(observerSecond)

    shirtItem.updateAvailability()
}</code></pre>
</figure>

#### []{#example-0--output-txt .anchor} **output.txt:** 実行結果

<figure class="code">
<pre class="code" lang="output"><code>Item Nike Shirt is now in stock
Sending email to customer abc@gmail.com for item Nike Shirt
Sending email to customer xyz@gmail.com for item Nike Shirt</code></pre>
</figure>

::: next
#### 次を読む

[State を Go で []{.fa
.fa-arrow-right}](../../state/go/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### 戻る

[[]{.fa .fa-arrow-left} Memento を Go
で](../../memento/go/example.html){.btn .btn-default rel="prev"}
:::

## 他言語での **Observer**

[![Observer を C#
で](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Observer を C# で"){.prog-lang-link}
[![Observer を C++
で](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Observer を C++ で"){.prog-lang-link}
[![Observer を Java
で](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Observer を Java で"){.prog-lang-link}
[![Observer を PHP
で](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Observer を PHP で"){.prog-lang-link}
[![Observer を Python
で](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Observer を Python で"){.prog-lang-link}
[![Observer を Ruby
で](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Observer を Ruby で"){.prog-lang-link}
[![Observer を Rust
で](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Observer を Rust で"){.prog-lang-link}
[![Observer を Swift
で](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Observer を Swift で"){.prog-lang-link}
[![Observer を TypeScript
で](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Observer を TypeScript で"){.prog-lang-link}
:::::::::::::::::::::

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
:::::::::::::::::::::::::::
::::::::::::::::::::::::::::
