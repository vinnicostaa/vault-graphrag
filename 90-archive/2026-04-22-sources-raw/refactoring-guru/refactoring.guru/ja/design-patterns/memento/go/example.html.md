::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} /
[デザインパターン](../../../design-patterns.html){.type} /
[Memento](../../memento.html){.type} / [Go](../../go.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Memento](../../../../images/patterns/cards/memento-mini723d.png?id=8b2ea4dc2c5d15775a654808cc9de099){srcset="/images/patterns/cards/memento-mini-2x.png?id=1d7cba189261dd84b11369a6838b1055 2x"}
:::

# **Memento** を Go で {#memento-を-go-で .pattern-example-title-block-title}
::::

:::::::::::::: pattern-example-body
::: pattern-example-brief
**Memento** は[、]{.chpule2}[
]{.chpuri2}振る舞いに関するデザインパターンの一つで[、]{.chpule2}[
]{.chpuri2}オブジェクトの状態のスナップショットを作成し[、]{.chpule2}[
]{.chpuri2}それを将来復元します[。]{.chpule2}[ ]{.chpuri2}

Memento は[、]{.chpule2}[
]{.chpuri2}その対象オブジェクトの内部構造やスナップショットの内部に保存されるデータの機密を守ります[。]{.chpule2}[
]{.chpuri2}
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Memento の詳細 ](../../memento.html){.btn .btn-lg .btn-primary}
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
 [originator](#example-0--originator-go)
:::

::: en-file
 [memento](#example-0--memento-go)
:::

::: en-file
 [caretaker](#example-0--caretaker-go)
:::

::: en-file
 [main](#example-0--main-go)
:::

::: en-file
 [output](#example-0--output-txt)
:::
:::::::::::
::::::::::::::

[]{#example-0}

## 概念的な例 {#example-0-title}

Memento パターンでは[、]{.chpule2}[
]{.chpuri2}オブジェクトの状態のスナップショットを保存できます[。]{.chpule2}[
]{.chpuri2}スナップショットを使って[、]{.chpule2}[
]{.chpuri2}オブジェクトを以前の状態に戻すことができます[。]{.chpule2}[
]{.chpuri2}オブジェクトの操作取り消しや[、]{.chpule2}[
]{.chpuri2}やり直しを実装する際に便利です[。]{.chpule2}[ ]{.chpuri2}

#### []{#example-0--originator-go .anchor} **originator.go:** オリジネーター

<figure class="code">
<pre class="code" lang="go"><code>package main

type Originator struct {
    state string
}

func (e *Originator) createMemento() *Memento {
    return &amp;Memento{state: e.state}
}

func (e *Originator) restoreMemento(m *Memento) {
    e.state = m.getSavedState()
}

func (e *Originator) setState(state string) {
    e.state = state
}

func (e *Originator) getState() string {
    return e.state
}</code></pre>
</figure>

#### []{#example-0--memento-go .anchor} **memento.go:** メメント

<figure class="code">
<pre class="code" lang="go"><code>package main

type Memento struct {
    state string
}

func (m *Memento) getSavedState() string {
    return m.state
}</code></pre>
</figure>

#### []{#example-0--caretaker-go .anchor} **caretaker.go:** ケアテーカー

<figure class="code">
<pre class="code" lang="go"><code>package main

type Caretaker struct {
    mementoArray []*Memento
}

func (c *Caretaker) addMemento(m *Memento) {
    c.mementoArray = append(c.mementoArray, m)
}

func (c *Caretaker) getMemento(index int) *Memento {
    return c.mementoArray[index]
}</code></pre>
</figure>

#### []{#example-0--main-go .anchor} **main.go:** クライアント・コード

<figure class="code">
<pre class="code" lang="go"><code>package main

import &quot;fmt&quot;

func main() {

    caretaker := &amp;Caretaker{
        mementoArray: make([]*Memento, 0),
    }

    originator := &amp;Originator{
        state: &quot;A&quot;,
    }

    fmt.Printf(&quot;Originator Current State: %s\n&quot;, originator.getState())
    caretaker.addMemento(originator.createMemento())

    originator.setState(&quot;B&quot;)
    fmt.Printf(&quot;Originator Current State: %s\n&quot;, originator.getState())
    caretaker.addMemento(originator.createMemento())

    originator.setState(&quot;C&quot;)
    fmt.Printf(&quot;Originator Current State: %s\n&quot;, originator.getState())
    caretaker.addMemento(originator.createMemento())

    originator.restoreMemento(caretaker.getMemento(1))
    fmt.Printf(&quot;Restored to State: %s\n&quot;, originator.getState())

    originator.restoreMemento(caretaker.getMemento(0))
    fmt.Printf(&quot;Restored to State: %s\n&quot;, originator.getState())

}</code></pre>
</figure>

#### []{#example-0--output-txt .anchor} **output.txt:** 実行結果

<figure class="code">
<pre class="code" lang="output"><code>originator Current State: A
originator Current State: B
originator Current State: C
Restored to State: B
Restored to State: A</code></pre>
</figure>

::: next
#### 次を読む

[Observer を Go で []{.fa
.fa-arrow-right}](../../observer/go/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### 戻る

[[]{.fa .fa-arrow-left} Mediator を Go
で](../../mediator/go/example.html){.btn .btn-default rel="prev"}
:::

## 他言語での **Memento**

[![Memento を C#
で](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Memento を C# で"){.prog-lang-link}
[![Memento を C++
で](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Memento を C++ で"){.prog-lang-link}
[![Memento を Java
で](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Memento を Java で"){.prog-lang-link}
[![Memento を PHP
で](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Memento を PHP で"){.prog-lang-link}
[![Memento を Python
で](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Memento を Python で"){.prog-lang-link}
[![Memento を Ruby
で](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Memento を Ruby で"){.prog-lang-link}
[![Memento を Rust
で](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Memento を Rust で"){.prog-lang-link}
[![Memento を Swift
で](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Memento を Swift で"){.prog-lang-link}
[![Memento を TypeScript
で](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Memento を TypeScript で"){.prog-lang-link}
::::::::::::::::::::

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
::::::::::::::::::::::::::
:::::::::::::::::::::::::::
