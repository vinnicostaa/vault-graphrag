:::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} /
[デザインパターン](../../../design-patterns.html){.type} /
[Facade](../../facade.html){.type} / [Go](../../go.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Facade](../../../../images/patterns/cards/facade-mini14fd.png?id=71ad6fa98b168c11cb3a1a9517dedf78){srcset="/images/patterns/cards/facade-mini-2x.png?id=d4cc6a5d81a31143cc665f7ac1481ac8 2x"}
:::

# **Facade** を Go で {#facade-を-go-で .pattern-example-title-block-title}
::::

::::::::::::::::: pattern-example-body
::: pattern-example-brief
**Facade** は[、]{.chpule2}[
]{.chpuri2}構造に関するデザインパターンの一つで[、]{.chpule2}[
]{.chpuri2}複雑なクラスのシステム[、]{.chpule2}[
]{.chpuri2}ライブラリー[、]{.chpule2}[
]{.chpuri2}またはフレームワークに対して単純な[
]{.chpule1}[（]{.chpuri1}しかし限定された[）]{.chpule2}[
]{.chpuri2}インターフェースを提供します[。]{.chpule2}[ ]{.chpuri2}

Facade は[、]{.chpule2}[
]{.chpuri2}アプリケーションの全体としての複雑さを軽減しますが[、]{.chpule2}[
]{.chpuri2}それと同時に望ましくない依存性を一箇所に集めるのにも役立ちます[。]{.chpule2}[
]{.chpuri2}
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Facade の詳細 ](../../facade.html){.btn .btn-lg .btn-primary}
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
 [wallet­Facade](#example-0--walletFacade-go)
:::

::: en-file
 [account](#example-0--account-go)
:::

::: en-file
 [security­Code](#example-0--securityCode-go)
:::

::: en-file
 [wallet](#example-0--wallet-go)
:::

::: en-file
 [ledger](#example-0--ledger-go)
:::

::: en-file
 [notification](#example-0--notification-go)
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

クレジットカードを使ってピザを注文した時[、]{.chpule2}[
]{.chpuri2}その裏で何が起きているか[、]{.chpule2}[
]{.chpuri2}我々はその複雑性を過小評価しがちです[。]{.chpule2}[
]{.chpuri2}この工程には[、]{.chpule2}[
]{.chpuri2}いくつものサブシステムが存在します[。]{.chpule2}[
]{.chpuri2}主要なものだけ挙げると[：]{.chpule2}[ ]{.chpuri2}

-   口座の確認
-   口座の暗証番号の確認
-   クレジットまたはデビットカードの残高照会
-   注文台帳に項目作成
-   通知の送信

このような複雑なシステムは[、]{.chpule2}[
]{.chpuri2}途中で理解できなくなったりしがちです[。]{.chpule2}[
]{.chpuri2}またちょっでも間違ったことをすると[、]{.chpule2}[
]{.chpuri2}簡単に異常を起こしてしまいます[。]{.chpule2}[
]{.chpuri2}Facade パターンの概念は[、]{.chpule2}[
]{.chpuri2}そのためにあります[：]{.chpule2}[
]{.chpuri2}単純なインターフェースを使ってクライアントがいくつものコンポーネントと作業できるようにする[。]{.chpule2}[
]{.chpuri2}クライアントは[、]{.chpule2}[
]{.chpuri2}カードの詳細[、]{.chpule2}[
]{.chpuri2}暗唱番号[、]{.chpule2}[
]{.chpuri2}支払額そして手続きの種類だけを入力します[。]{.chpule2}[
]{.chpuri2}ファサードは[、]{.chpule2}[
]{.chpuri2}クライアントから内部の複雑さを隠蔽し[、]{.chpule2}[
]{.chpuri2}それ以降の種々のコンポーネントとのやりとりを行います[。]{.chpule2}[
]{.chpuri2}

#### []{#example-0--walletFacade-go .anchor} **walletFacade.go:** ファサード

<figure class="code">
<pre class="code" lang="go"><code>package main

import &quot;fmt&quot;

type WalletFacade struct {
    account      *Account
    wallet       *Wallet
    securityCode *SecurityCode
    notification *Notification
    ledger       *Ledger
}

func newWalletFacade(accountID string, code int) *WalletFacade {
    fmt.Println(&quot;Starting create account&quot;)
    walletFacacde := &amp;WalletFacade{
        account:      newAccount(accountID),
        securityCode: newSecurityCode(code),
        wallet:       newWallet(),
        notification: &amp;Notification{},
        ledger:       &amp;Ledger{},
    }
    fmt.Println(&quot;Account created&quot;)
    return walletFacacde
}

func (w *WalletFacade) addMoneyToWallet(accountID string, securityCode int, amount int) error {
    fmt.Println(&quot;Starting add money to wallet&quot;)
    err := w.account.checkAccount(accountID)
    if err != nil {
        return err
    }
    err = w.securityCode.checkCode(securityCode)
    if err != nil {
        return err
    }
    w.wallet.creditBalance(amount)
    w.notification.sendWalletCreditNotification()
    w.ledger.makeEntry(accountID, &quot;credit&quot;, amount)
    return nil
}

func (w *WalletFacade) deductMoneyFromWallet(accountID string, securityCode int, amount int) error {
    fmt.Println(&quot;Starting debit money from wallet&quot;)
    err := w.account.checkAccount(accountID)
    if err != nil {
        return err
    }

    err = w.securityCode.checkCode(securityCode)
    if err != nil {
        return err
    }
    err = w.wallet.debitBalance(amount)
    if err != nil {
        return err
    }
    w.notification.sendWalletDebitNotification()
    w.ledger.makeEntry(accountID, &quot;debit&quot;, amount)
    return nil
}</code></pre>
</figure>

#### []{#example-0--account-go .anchor} **account.go:** 複雑なサブシステム[ ]{.chpule1}[（]{.chpuri1}部分[）]{.chpule2}[ ]{.chpuri2}

<figure class="code">
<pre class="code" lang="go"><code>package main

import &quot;fmt&quot;

type Account struct {
    name string
}

func newAccount(accountName string) *Account {
    return &amp;Account{
        name: accountName,
    }
}

func (a *Account) checkAccount(accountName string) error {
    if a.name != accountName {
        return fmt.Errorf(&quot;Account Name is incorrect&quot;)
    }
    fmt.Println(&quot;Account Verified&quot;)
    return nil
}</code></pre>
</figure>

#### []{#example-0--securityCode-go .anchor} **securityCode.go:** 複雑なサブシステム[ ]{.chpule1}[（]{.chpuri1}部分[）]{.chpule2}[ ]{.chpuri2}

<figure class="code">
<pre class="code" lang="go"><code>package main

import &quot;fmt&quot;

type SecurityCode struct {
    code int
}

func newSecurityCode(code int) *SecurityCode {
    return &amp;SecurityCode{
        code: code,
    }
}

func (s *SecurityCode) checkCode(incomingCode int) error {
    if s.code != incomingCode {
        return fmt.Errorf(&quot;Security Code is incorrect&quot;)
    }
    fmt.Println(&quot;SecurityCode Verified&quot;)
    return nil
}</code></pre>
</figure>

#### []{#example-0--wallet-go .anchor} **wallet.go:** 複雑なサブシステム[ ]{.chpule1}[（]{.chpuri1}部分[）]{.chpule2}[ ]{.chpuri2}

<figure class="code">
<pre class="code" lang="go"><code>package main

import &quot;fmt&quot;

type Wallet struct {
    balance int
}

func newWallet() *Wallet {
    return &amp;Wallet{
        balance: 0,
    }
}

func (w *Wallet) creditBalance(amount int) {
    w.balance += amount
    fmt.Println(&quot;Wallet balance added successfully&quot;)
    return
}

func (w *Wallet) debitBalance(amount int) error {
    if w.balance &lt; amount {
        return fmt.Errorf(&quot;Balance is not sufficient&quot;)
    }
    fmt.Println(&quot;Wallet balance is Sufficient&quot;)
    w.balance = w.balance - amount
    return nil
}</code></pre>
</figure>

#### []{#example-0--ledger-go .anchor} **ledger.go:** 複雑なサブシステム[ ]{.chpule1}[（]{.chpuri1}部分[）]{.chpule2}[ ]{.chpuri2}

<figure class="code">
<pre class="code" lang="go"><code>package main

import &quot;fmt&quot;

type Ledger struct {
}

func (s *Ledger) makeEntry(accountID, txnType string, amount int) {
    fmt.Printf(&quot;Make ledger entry for accountId %s with txnType %s for amount %d\n&quot;, accountID, txnType, amount)
    return
}</code></pre>
</figure>

#### []{#example-0--notification-go .anchor} **notification.go:** 複雑なサブシステム[ ]{.chpule1}[（]{.chpuri1}部分[）]{.chpule2}[ ]{.chpuri2}

<figure class="code">
<pre class="code" lang="go"><code>package main

import &quot;fmt&quot;

type Notification struct {
}

func (n *Notification) sendWalletCreditNotification() {
    fmt.Println(&quot;Sending wallet credit notification&quot;)
}

func (n *Notification) sendWalletDebitNotification() {
    fmt.Println(&quot;Sending wallet debit notification&quot;)
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
    fmt.Println()
    walletFacade := newWalletFacade(&quot;abc&quot;, 1234)
    fmt.Println()

    err := walletFacade.addMoneyToWallet(&quot;abc&quot;, 1234, 10)
    if err != nil {
        log.Fatalf(&quot;Error: %s\n&quot;, err.Error())
    }

    fmt.Println()
    err = walletFacade.deductMoneyFromWallet(&quot;abc&quot;, 1234, 5)
    if err != nil {
        log.Fatalf(&quot;Error: %s\n&quot;, err.Error())
    }
}</code></pre>
</figure>

#### []{#example-0--output-txt .anchor} **output.txt:** 実行結果

<figure class="code">
<pre class="code" lang="output"><code>Starting create account
Account created

Starting add money to wallet
Account Verified
SecurityCode Verified
Wallet balance added successfully
Sending wallet credit notification
Make ledger entry for accountId abc with txnType credit for amount 10

Starting debit money from wallet
Account Verified
SecurityCode Verified
Wallet balance is Sufficient
Sending wallet debit notification
Make ledger entry for accountId abc with txnType debit for amount 5</code></pre>
</figure>

::: next
#### 次を読む

[Flyweight を Go で []{.fa
.fa-arrow-right}](../../flyweight/go/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### 戻る

[[]{.fa .fa-arrow-left} Decorator を Go
で](../../decorator/go/example.html){.btn .btn-default rel="prev"}
:::

## 他言語での **Facade**

[![Facade を C#
で](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Facade を C# で"){.prog-lang-link}
[![Facade を C++
で](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Facade を C++ で"){.prog-lang-link}
[![Facade を Java
で](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Facade を Java で"){.prog-lang-link}
[![Facade を PHP
で](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Facade を PHP で"){.prog-lang-link}
[![Facade を Python
で](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Facade を Python で"){.prog-lang-link}
[![Facade を Ruby
で](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Facade を Ruby で"){.prog-lang-link}
[![Facade を Rust
で](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Facade を Rust で"){.prog-lang-link}
[![Facade を Swift
で](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Facade を Swift で"){.prog-lang-link}
[![Facade を TypeScript
で](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Facade を TypeScript で"){.prog-lang-link}
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
