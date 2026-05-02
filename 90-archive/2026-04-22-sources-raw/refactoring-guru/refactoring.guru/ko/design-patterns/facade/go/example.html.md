:::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [디자인
패턴들](../../../design-patterns.html){.type} /
[퍼사드](../../facade.html){.type} / [Go](../../go.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![퍼사드](../../../../images/patterns/cards/facade-mini14fd.png?id=71ad6fa98b168c11cb3a1a9517dedf78){srcset="/images/patterns/cards/facade-mini-2x.png?id=d4cc6a5d81a31143cc665f7ac1481ac8 2x"}
:::

# Go로 작성된 **퍼사드** {#go로-작성된-퍼사드 .pattern-example-title-block-title}
::::

::::::::::::::::: pattern-example-body
::: pattern-example-brief
**퍼사드**는 구조 디자인 패턴이며 간단한 (그러나 제한된 인터페이스를
클래스, 라이브러리 또는 프레임워크로 구성된 복잡한 시스템에 제공합니다.

퍼사드는 앱의 전반적인 복잡성을 줄이는 동시에 원치 않는 의존성들을 한
곳으로 옮기는 것을 돕습니다.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ 퍼사드에 대하여 더 자세히 알아보세요 ](../../facade.html){.btn .btn-lg
.btn-primary}
:::

:::::::::::::: {.sidebar-navigation .with-scroll}
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

## 개념적인 예시 {#example-0-title}

신용 카드를 사용하여 피자를 주문할 때 시스템 내부에서 일어나는
프로세스의 복잡성을 과소평가하기 쉽습니다. 이 프로세스 내에 작동하는
수십 개의 하위 시스템이 있으며 다음은 그중 일부입니다:

-   계정 확인
-   보안 PIN 확인
-   대변/차변 잔액
-   원장 입력
-   알림 보내기

이와 같은 복잡한 시스템을 탐색하기 쉽지 않으며 또 잘못된 작업을 수행하면
코드가 깨지기 쉽습니다. 이것이 클라이언트가 간단한 인터페이스를 사용하여
수십 개의 구성 요소와 작업할 수 있도록 하는 퍼사드 패턴의 개념이 있는
이유입니다. 클라이언트는 카드 정보, 보안 핀, 지불 금액 및 작업 유형만
입력하면 됩니다. 퍼사드는 클라이언트를 내부 복잡성에 노출하지 않고
다양한 구성 요소와의 추가 통신을 지시합니다.

#### []{#example-0--walletFacade-go .anchor} **walletFacade.go:** 퍼사드

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

#### []{#example-0--account-go .anchor} **account.go:** 복잡한 하위 시스템 부품들

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

#### []{#example-0--securityCode-go .anchor} **securityCode.go:** 복잡한 하위 시스템 부품들

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

#### []{#example-0--wallet-go .anchor} **wallet.go:** 복잡한 하위 시스템 부품들

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

#### []{#example-0--ledger-go .anchor} **ledger.go:** 복잡한 하위 시스템 부품들

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

#### []{#example-0--notification-go .anchor} **notification.go:** 복잡한 하위 시스템 부품들

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

#### []{#example-0--main-go .anchor} **main.go:** 클라이언트 코드

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

#### []{#example-0--output-txt .anchor} **output.txt:** 실행 결과

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
#### 다음을 보세요

[Go로 작성된 플라이웨이트 []{.fa
.fa-arrow-right}](../../flyweight/go/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### 돌아가기

[[]{.fa .fa-arrow-left} Go로 작성된
데코레이터](../../decorator/go/example.html){.btn .btn-default
rel="prev"}
:::

## 다른 언어로 작성된 **퍼사드**

[![C#으로 작성된
퍼사드](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "C#으로 작성된 퍼사드"){.prog-lang-link}
[![C++로 작성된
퍼사드](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "C++로 작성된 퍼사드"){.prog-lang-link}
[![자바로 작성된
퍼사드](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "자바로 작성된 퍼사드"){.prog-lang-link}
[![PHP로 작성된
퍼사드](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "PHP로 작성된 퍼사드"){.prog-lang-link}
[![파이썬으로 작성된
퍼사드](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "파이썬으로 작성된 퍼사드"){.prog-lang-link}
[![루비로 작성된
퍼사드](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "루비로 작성된 퍼사드"){.prog-lang-link}
[![러스트로 작성된
퍼사드](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "러스트로 작성된 퍼사드"){.prog-lang-link}
[![스위프트로 작성된
퍼사드](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "스위프트로 작성된 퍼사드"){.prog-lang-link}
[![타입스크립트로 작성된
퍼사드](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "타입스크립트로 작성된 퍼사드"){.prog-lang-link}
:::::::::::::::::::::::

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
:::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::
