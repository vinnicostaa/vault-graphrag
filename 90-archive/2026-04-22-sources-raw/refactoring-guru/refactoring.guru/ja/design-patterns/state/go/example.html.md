:::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} /
[デザインパターン](../../../design-patterns.html){.type} /
[State](../../state.html){.type} / [Go](../../go.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![State](../../../../images/patterns/cards/state-mini63fb.png?id=f4018837e0641d1dade756b6678fd4ee){srcset="/images/patterns/cards/state-mini-2x.png?id=7e24398b27a43c7bd286fc0ea54d2a35 2x"}
:::

# **State** を Go で {#state-を-go-で .pattern-example-title-block-title}
::::

::::::::::::::::: pattern-example-body
::: pattern-example-brief
**State** は[、]{.chpule2}[
]{.chpuri2}振る舞いに関するデザインパターンの一つで[、]{.chpule2}[
]{.chpuri2}オブジェクトの内部状態が変化した時にその振る舞いを変更することを可能とします[。]{.chpule2}[
]{.chpuri2}

このパターンは[、]{.chpule2}[
]{.chpuri2}状態に関連した振る舞いを個別の状態のクラスへ抽出し[、]{.chpule2}[
]{.chpuri2}元のオブジェクトが作業を自分で行わず[、]{.chpule2}[
]{.chpuri2}これらのクラスのインスタンスに委任することを強制します[。]{.chpule2}[
]{.chpuri2}
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ State の詳細 ](../../state.html){.btn .btn-lg .btn-primary}
:::

:::::::::::::: {.sidebar-navigation .with-scroll}
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
 [vending­Machine](#example-0--vendingMachine-go)
:::

::: en-file
 [state](#example-0--state-go)
:::

::: en-file
 [no­Item­State](#example-0--noItemState-go)
:::

::: en-file
 [has­Item­State](#example-0--hasItemState-go)
:::

::: en-file
 [item­Requested­State](#example-0--itemRequestedState-go)
:::

::: en-file
 [has­Money­State](#example-0--hasMoneyState-go)
:::

::: en-file
 [main](#example-0--main-go)
:::

::: en-file
 [output](#example-0--output-txt)
:::
::::::::::::::
:::::::::::::::::

[]{#example-0}

## 概念的な例 {#example-0-title}

State デザインパターンを[、]{.chpule2}[
]{.chpuri2}自動販売機に適用してみましょう[。]{.chpule2}[
]{.chpuri2}話を簡単にするため[、]{.chpule2}[ ]{.chpuri2}自動販売機は 1
種類の商品しか扱わないこととします[。]{.chpule2}[
]{.chpuri2}また[、]{.chpule2}[ ]{.chpuri2}自動販売機は[、]{.chpule2}[
]{.chpuri2}以下の四つの状態のいずれかにあるものと仮定します[：]{.chpule2}[
]{.chpuri2}

-   hasItem[ ]{.chpule1}[（]{.chpuri1}在庫あり[）]{.chpule2}[
    ]{.chpuri2}
-   noItem[ ]{.chpule1}[（]{.chpuri1}品切れ[）]{.chpule2}[ ]{.chpuri2}
-   itemRequested[ ]{.chpule1}[（]{.chpuri1}商品選択済み[）]{.chpule2}[
    ]{.chpuri2}
-   hasMoney[ ]{.chpule1}[（]{.chpuri1}入金あり[）]{.chpule2}[
    ]{.chpuri2}

自動販売機は[、]{.chpule2}[
]{.chpuri2}異なるアクションがあります[。]{.chpule2}[
]{.chpuri2}簡略化のため[、]{.chpule2}[
]{.chpuri2}以下の四つのアクションしかないものとします[：]{.chpule2}[
]{.chpuri2}

-   商品を選ぶ
-   商品を追加
-   お金を入れる
-   商品を出す

オブジェクトが多くの異なる状態を取ることができ[、]{.chpule2}[
]{.chpuri2}入ってくるリクエストに応じてオブジェクトが現在の状態を変更する必要がある場合[、]{.chpule2}[
]{.chpuri2}State デザインパターンを使うべきです[。]{.chpule2}[
]{.chpuri2}

この例では[、]{.chpule2}[ ]{.chpuri2}自動販売機は[、]{.chpule2}[
]{.chpuri2}多くの異なる状態を取ることができ[、]{.chpule2}[
]{.chpuri2}状態は常に切り替わります[。]{.chpule2}[
]{.chpuri2}自動販売機が[、]{.chpule2}[ ]{.chpuri2}`item­Requested`
状態だったとします[[。]{.ls-1}]{.chpule2}[ ]{.chpuri2}​[
]{.chpule1}[「]{.chpuri1}お金を入れる[」]{.chpule2}[
]{.chpuri2}アクションが一旦起きると[、]{.chpule2}[
]{.chpuri2}自動販売機は `has­Money` 状態に移動します[。]{.chpule2}[
]{.chpuri2}

現在の状態に応じて[、]{.chpule2}[
]{.chpuri2}自動販売機は同じリクエストに対して違う振る舞いをします[。]{.chpule2}[
]{.chpuri2}たとえば[、]{.chpule2}[
]{.chpuri2}もしユーザーが商品を購入したいとして[、]{.chpule2}[
]{.chpuri2}もし `has­Item­State` 状態だったら[、]{.chpule2}[
]{.chpuri2}そのまま先に進みますが[、]{.chpule2}[ ]{.chpuri2}もし
`no­Item­State` 状態だった場合は[、]{.chpule2}[
]{.chpuri2}拒絶します[。]{.chpule2}[ ]{.chpuri2}

この自動販売機のコードは[、]{.chpule2}[
]{.chpuri2}ロジックで汚染されていません[。]{.chpule2}[
]{.chpuri2}状態依存のコードはすべて[、]{.chpule2}[
]{.chpuri2}対応する状態の実装の中にあります[。]{.chpule2}[ ]{.chpuri2}

#### []{#example-0--vendingMachine-go .anchor} **vendingMachine.go:** コンテキスト

<figure class="code">
<pre class="code" lang="go"><code>package main

import &quot;fmt&quot;

type VendingMachine struct {
    hasItem       State
    itemRequested State
    hasMoney      State
    noItem        State

    currentState State

    itemCount int
    itemPrice int
}

func newVendingMachine(itemCount, itemPrice int) *VendingMachine {
    v := &amp;VendingMachine{
        itemCount: itemCount,
        itemPrice: itemPrice,
    }
    hasItemState := &amp;HasItemState{
        vendingMachine: v,
    }
    itemRequestedState := &amp;ItemRequestedState{
        vendingMachine: v,
    }
    hasMoneyState := &amp;HasMoneyState{
        vendingMachine: v,
    }
    noItemState := &amp;NoItemState{
        vendingMachine: v,
    }

    v.setState(hasItemState)
    v.hasItem = hasItemState
    v.itemRequested = itemRequestedState
    v.hasMoney = hasMoneyState
    v.noItem = noItemState
    return v
}

func (v *VendingMachine) requestItem() error {
    return v.currentState.requestItem()
}

func (v *VendingMachine) addItem(count int) error {
    return v.currentState.addItem(count)
}

func (v *VendingMachine) insertMoney(money int) error {
    return v.currentState.insertMoney(money)
}

func (v *VendingMachine) dispenseItem() error {
    return v.currentState.dispenseItem()
}

func (v *VendingMachine) setState(s State) {
    v.currentState = s
}

func (v *VendingMachine) incrementItemCount(count int) {
    fmt.Printf(&quot;Adding %d items\n&quot;, count)
    v.itemCount = v.itemCount + count
}</code></pre>
</figure>

#### []{#example-0--state-go .anchor} **state.go:** 状態インターフェース

<figure class="code">
<pre class="code" lang="go"><code>package main

type State interface {
    addItem(int) error
    requestItem() error
    insertMoney(money int) error
    dispenseItem() error
}</code></pre>
</figure>

#### []{#example-0--noItemState-go .anchor} **noItemState.go:** 具象ステート

<figure class="code">
<pre class="code" lang="go"><code>package main

import &quot;fmt&quot;

type NoItemState struct {
    vendingMachine *VendingMachine
}

func (i *NoItemState) requestItem() error {
    return fmt.Errorf(&quot;Item out of stock&quot;)
}

func (i *NoItemState) addItem(count int) error {
    i.vendingMachine.incrementItemCount(count)
    i.vendingMachine.setState(i.vendingMachine.hasItem)
    return nil
}

func (i *NoItemState) insertMoney(money int) error {
    return fmt.Errorf(&quot;Item out of stock&quot;)
}
func (i *NoItemState) dispenseItem() error {
    return fmt.Errorf(&quot;Item out of stock&quot;)
}</code></pre>
</figure>

#### []{#example-0--hasItemState-go .anchor} **hasItemState.go:** 具象ステート

<figure class="code">
<pre class="code" lang="go"><code>package main

import &quot;fmt&quot;

type HasItemState struct {
    vendingMachine *VendingMachine
}

func (i *HasItemState) requestItem() error {
    if i.vendingMachine.itemCount == 0 {
        i.vendingMachine.setState(i.vendingMachine.noItem)
        return fmt.Errorf(&quot;No item present&quot;)
    }
    fmt.Printf(&quot;Item requestd\n&quot;)
    i.vendingMachine.setState(i.vendingMachine.itemRequested)
    return nil
}

func (i *HasItemState) addItem(count int) error {
    fmt.Printf(&quot;%d items added\n&quot;, count)
    i.vendingMachine.incrementItemCount(count)
    return nil
}

func (i *HasItemState) insertMoney(money int) error {
    return fmt.Errorf(&quot;Please select item first&quot;)
}
func (i *HasItemState) dispenseItem() error {
    return fmt.Errorf(&quot;Please select item first&quot;)
}</code></pre>
</figure>

#### []{#example-0--itemRequestedState-go .anchor} **itemRequestedState.go:** 具象ステート

<figure class="code">
<pre class="code" lang="go"><code>package main

import &quot;fmt&quot;

type ItemRequestedState struct {
    vendingMachine *VendingMachine
}

func (i *ItemRequestedState) requestItem() error {
    return fmt.Errorf(&quot;Item already requested&quot;)
}

func (i *ItemRequestedState) addItem(count int) error {
    return fmt.Errorf(&quot;Item Dispense in progress&quot;)
}

func (i *ItemRequestedState) insertMoney(money int) error {
    if money &lt; i.vendingMachine.itemPrice {
        return fmt.Errorf(&quot;Inserted money is less. Please insert %d&quot;, i.vendingMachine.itemPrice)
    }
    fmt.Println(&quot;Money entered is ok&quot;)
    i.vendingMachine.setState(i.vendingMachine.hasMoney)
    return nil
}
func (i *ItemRequestedState) dispenseItem() error {
    return fmt.Errorf(&quot;Please insert money first&quot;)
}</code></pre>
</figure>

#### []{#example-0--hasMoneyState-go .anchor} **hasMoneyState.go:** 具象ステート

<figure class="code">
<pre class="code" lang="go"><code>package main

import &quot;fmt&quot;

type HasMoneyState struct {
    vendingMachine *VendingMachine
}

func (i *HasMoneyState) requestItem() error {
    return fmt.Errorf(&quot;Item dispense in progress&quot;)
}

func (i *HasMoneyState) addItem(count int) error {
    return fmt.Errorf(&quot;Item dispense in progress&quot;)
}

func (i *HasMoneyState) insertMoney(money int) error {
    return fmt.Errorf(&quot;Item out of stock&quot;)
}
func (i *HasMoneyState) dispenseItem() error {
    fmt.Println(&quot;Dispensing Item&quot;)
    i.vendingMachine.itemCount = i.vendingMachine.itemCount - 1
    if i.vendingMachine.itemCount == 0 {
        i.vendingMachine.setState(i.vendingMachine.noItem)
    } else {
        i.vendingMachine.setState(i.vendingMachine.hasItem)
    }
    return nil
}</code></pre>
</figure>

#### []{#example-0--main-go .anchor} **main.go:** クライアント・コード

<figure class="code">
<pre class="code" lang="go"><code>package main

import (
    &quot;fmt&quot;
    &quot;log&quot;
)

func main() {
    vendingMachine := newVendingMachine(1, 10)

    err := vendingMachine.requestItem()
    if err != nil {
        log.Fatalf(err.Error())
    }

    err = vendingMachine.insertMoney(10)
    if err != nil {
        log.Fatalf(err.Error())
    }

    err = vendingMachine.dispenseItem()
    if err != nil {
        log.Fatalf(err.Error())
    }

    fmt.Println()

    err = vendingMachine.addItem(2)
    if err != nil {
        log.Fatalf(err.Error())
    }

    fmt.Println()

    err = vendingMachine.requestItem()
    if err != nil {
        log.Fatalf(err.Error())
    }

    err = vendingMachine.insertMoney(10)
    if err != nil {
        log.Fatalf(err.Error())
    }

    err = vendingMachine.dispenseItem()
    if err != nil {
        log.Fatalf(err.Error())
    }
}</code></pre>
</figure>

#### []{#example-0--output-txt .anchor} **output.txt:** 実行結果

<figure class="code">
<pre class="code" lang="output"><code>Item requestd
Money entered is ok
Dispensing Item

Adding 2 items

Item requestd
Money entered is ok
Dispensing Item</code></pre>
</figure>

::: next
#### 次を読む

[Strategy を Go で []{.fa
.fa-arrow-right}](../../strategy/go/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### 戻る

[[]{.fa .fa-arrow-left} Observer を Go
で](../../observer/go/example.html){.btn .btn-default rel="prev"}
:::

## 他言語での **State**

[![State を C#
で](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "State を C# で"){.prog-lang-link}
[![State を C++
で](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "State を C++ で"){.prog-lang-link}
[![State を Java
で](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "State を Java で"){.prog-lang-link}
[![State を PHP
で](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "State を PHP で"){.prog-lang-link}
[![State を Python
で](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "State を Python で"){.prog-lang-link}
[![State を Ruby
で](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "State を Ruby で"){.prog-lang-link}
[![State を Rust
で](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "State を Rust で"){.prog-lang-link}
[![State を Swift
で](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "State を Swift で"){.prog-lang-link}
[![State を TypeScript
で](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "State を TypeScript で"){.prog-lang-link}
:::::::::::::::::::::::

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
:::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::
