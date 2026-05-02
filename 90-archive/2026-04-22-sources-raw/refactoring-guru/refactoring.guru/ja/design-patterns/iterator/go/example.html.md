::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} /
[デザインパターン](../../../design-patterns.html){.type} /
[Iterator](../../iterator.html){.type} / [Go](../../go.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Iterator](../../../../images/patterns/cards/iterator-minie2c2.png?id=76c28bb48f997b36965983dd2b41f02e){srcset="/images/patterns/cards/iterator-mini-2x.png?id=7d28f66a487066774a743cfceaa40ea4 2x"}
:::

# **Iterator** を Go で {#iterator-を-go-で .pattern-example-title-block-title}
::::

:::::::::::::::: pattern-example-body
::: pattern-example-brief
**Iterator** は[、]{.chpule2}[
]{.chpuri2}振る舞いに関するデザインパターンの一つで[、]{.chpule2}[
]{.chpuri2}複雑なデータ構造の内部の詳細を公開することなく[、]{.chpule2}[
]{.chpuri2}順次横断的に探索することを可能とします[。]{.chpule2}[
]{.chpuri2}

Iterator のおかげで[、]{.chpule2}[
]{.chpuri2}クライアントは[、]{.chpule2}[
]{.chpuri2}異なるコレクション上の要素の探索を[、]{.chpule2}[
]{.chpuri2}単一のイテレーター・インターフェースを使用して同様の方法で行えます[。]{.chpule2}[
]{.chpuri2}
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Iterator の詳細 ](../../iterator.html){.btn .btn-lg .btn-primary}
:::

::::::::::::: {.sidebar-navigation .with-scroll}
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
 [collection](#example-0--collection-go)
:::

::: en-file
 [user­Collection](#example-0--userCollection-go)
:::

::: en-file
 [iterator](#example-0--iterator-go)
:::

::: en-file
 [user­Iterator](#example-0--userIterator-go)
:::

::: en-file
 [user](#example-0--user-go)
:::

::: en-file
 [main](#example-0--main-go)
:::

::: en-file
 [output](#example-0--output-txt)
:::
:::::::::::::
::::::::::::::::

[]{#example-0}

## 概念的な例 {#example-0-title}

Iterator パターンの基本的な考え方は[、]{.chpule2}[
]{.chpuri2}コレクションの反復ロジックをイテレーターと呼ばれる別のオブジェクトに抽出することです[。]{.chpule2}[
]{.chpuri2}このイテレーターは[、]{.chpule2}[
]{.chpuri2}型から独立した[、]{.chpule2}[
]{.chpuri2}コレクション上の反復を行う一般的メソッドを提供します[。]{.chpule2}[
]{.chpuri2}

#### []{#example-0--collection-go .anchor} **collection.go:** コレクション

<figure class="code">
<pre class="code" lang="go"><code>package main

type Collection interface {
    createIterator() Iterator
}</code></pre>
</figure>

#### []{#example-0--userCollection-go .anchor} **userCollection.go:** 具象コレクション

<figure class="code">
<pre class="code" lang="go"><code>package main

type UserCollection struct {
    users []*User
}

func (u *UserCollection) createIterator() Iterator {
    return &amp;UserIterator{
        users: u.users,
    }
}</code></pre>
</figure>

#### []{#example-0--iterator-go .anchor} **iterator.go:** イテレーター

<figure class="code">
<pre class="code" lang="go"><code>package main

type Iterator interface {
    hasNext() bool
    getNext() *User
}</code></pre>
</figure>

#### []{#example-0--userIterator-go .anchor} **userIterator.go:** 具象イテレーター

<figure class="code">
<pre class="code" lang="go"><code>package main

type UserIterator struct {
    index int
    users []*User
}

func (u *UserIterator) hasNext() bool {
    if u.index &lt; len(u.users) {
        return true
    }
    return false

}
func (u *UserIterator) getNext() *User {
    if u.hasNext() {
        user := u.users[u.index]
        u.index++
        return user
    }
    return nil
}</code></pre>
</figure>

#### []{#example-0--user-go .anchor} **user.go:** クライアント・コード

<figure class="code">
<pre class="code" lang="go"><code>package main

type User struct {
    name string
    age  int
}</code></pre>
</figure>

#### []{#example-0--main-go .anchor} **main.go:** クライアント・コード

<figure class="code">
<pre class="code" lang="go"><code>package main

import &quot;fmt&quot;

func main() {

    user1 := &amp;User{
        name: &quot;a&quot;,
        age:  30,
    }
    user2 := &amp;User{
        name: &quot;b&quot;,
        age:  20,
    }

    userCollection := &amp;UserCollection{
        users: []*User{user1, user2},
    }

    iterator := userCollection.createIterator()

    for iterator.hasNext() {
        user := iterator.getNext()
        fmt.Printf(&quot;User is %+v\n&quot;, user)
    }
}</code></pre>
</figure>

#### []{#example-0--output-txt .anchor} **output.txt:** 実行結果

<figure class="code">
<pre class="code" lang="output"><code>User is &amp;{name:a age:30}
User is &amp;{name:b age:20}</code></pre>
</figure>

::: next
#### 次を読む

[Mediator を Go で []{.fa
.fa-arrow-right}](../../mediator/go/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### 戻る

[[]{.fa .fa-arrow-left} Command を Go
で](../../command/go/example.html){.btn .btn-default rel="prev"}
:::

## 他言語での **Iterator**

[![Iterator を C#
で](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Iterator を C# で"){.prog-lang-link}
[![Iterator を C++
で](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Iterator を C++ で"){.prog-lang-link}
[![Iterator を Java
で](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Iterator を Java で"){.prog-lang-link}
[![Iterator を PHP
で](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Iterator を PHP で"){.prog-lang-link}
[![Iterator を Python
で](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Iterator を Python で"){.prog-lang-link}
[![Iterator を Ruby
で](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Iterator を Ruby で"){.prog-lang-link}
[![Iterator を Rust
で](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Iterator を Rust で"){.prog-lang-link}
[![Iterator を Swift
で](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Iterator を Swift で"){.prog-lang-link}
[![Iterator を TypeScript
で](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Iterator を TypeScript で"){.prog-lang-link}
::::::::::::::::::::::

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
::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::
