:::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Wzorce
projektowe](../../../design-patterns.html){.type} /
[Fasada](../../facade.html){.type} / [Go](../../go.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Fasada](../../../../images/patterns/cards/facade-mini14fd.png?id=71ad6fa98b168c11cb3a1a9517dedf78){srcset="/images/patterns/cards/facade-mini-2x.png?id=d4cc6a5d81a31143cc665f7ac1481ac8 2x"}
:::

# **Fasada** w języku Go {#fasada-w-języku-go .pattern-example-title-block-title}
::::

::::::::::::::::: pattern-example-body
::: pattern-example-brief
**Fasada** to strukturalny wzorzec projektowy wyposażający złożony
system klas, bibliotekę lub framework w uproszczony interfejs.

Fasada zmniejsza ogólną złożoność aplikacji oraz pomaga przenieść
niechciane zależności w jedno miejsce w programie.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Dowiedz się więcej o Fasada ](../../facade.html){.btn .btn-lg
.btn-primary}
:::

:::::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
Nawigacja
:::

::: en-intro
 [Intro](#)
:::

::: en-intro
 [Przykład koncepcyjny](#example-0)
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

## Przykład koncepcyjny {#example-0-title}

Łatwo można nie docenić złożoności czegoś tak pozornie prostego jak
zamawianie pizzy korzystając z karty kredytowej. W procesie tym za
kulisami bierze udział multum podsystemów:

-   Sprawdzanie konta
-   Sprawdzanie kodu PIN
-   Sprawdzanie salda konta
-   Księgowanie
-   Wysłanie powiadomienia

W tak skomplikowanym systemie łatwo jest się pogubić i coś zepsuć w
razie popełnienia błędu. Z myślą o tym stworzono koncepcję wzorca
Fasada: coś, co pozwoli klientowi korzystać z wielu komponentów za
pomocą prostego interfejsu. Klient musi jedynie znać szczegóły karty,
kod PIN, kwotę do zapłacenia i typ operacji. Fasada zaś pokieruje dalszą
komunikacją z poszczególnymi komponentami bez konieczności
wtajemniczania klienta w cały proces.

#### []{#example-0--walletFacade-go .anchor} **walletFacade.go:** Fasada

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

#### []{#example-0--account-go .anchor} **account.go:** Złożone części podsystemu

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

#### []{#example-0--securityCode-go .anchor} **securityCode.go:** Złożone części podsystemu

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

#### []{#example-0--wallet-go .anchor} **wallet.go:** Złożone części podsystemu

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

#### []{#example-0--ledger-go .anchor} **ledger.go:** Złożone części podsystemu

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

#### []{#example-0--notification-go .anchor} **notification.go:** Złożone części podsystemu

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

#### []{#example-0--main-go .anchor} **main.go:** Kod klienta

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

#### []{#example-0--output-txt .anchor} **output.txt:** Wynik działania

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
#### Czytaj dalej

[Pyłek w języku Go []{.fa
.fa-arrow-right}](../../flyweight/go/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Wróć

[[]{.fa .fa-arrow-left} Dekorator w języku
Go](../../decorator/go/example.html){.btn .btn-default rel="prev"}
:::

## **Fasada** w innych językach

[![Fasada w języku
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Fasada w języku C#"){.prog-lang-link}
[![Fasada w języku
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Fasada w języku C++"){.prog-lang-link}
[![Fasada w języku
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Fasada w języku Java"){.prog-lang-link}
[![Fasada w języku
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Fasada w języku PHP"){.prog-lang-link}
[![Fasada w języku
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Fasada w języku Python"){.prog-lang-link}
[![Fasada w języku
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Fasada w języku Ruby"){.prog-lang-link}
[![Fasada w języku
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Fasada w języku Rust"){.prog-lang-link}
[![Fasada w języku
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Fasada w języku Swift"){.prog-lang-link}
[![Fasada w języku
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Fasada w języku TypeScript"){.prog-lang-link}
:::::::::::::::::::::::

::::::: {.prom .banner-sidebar .banner .banner-striped .banner-examples .banner-removable .banner-removable-patterns data-id="DP: Examples-IDE" creative-id="examples-sidebar-pl" position="sidebar"}
:::::: {.banner-inner style="font-size: 14px; line-height: 21px;"}
::: banner-examples-img
[![](../../../../images/patterns/banners/examples-ide91ca.png?id=3115b4b548fb96b75974e2de8f4f49bc){width="215"
height="158" loading="lazy"
srcset="/images/patterns/banners/examples-ide-2x.png?id=93c007a6d157b97c28eb213f59d716ae 2x"}](../../book.html)
:::

::: banner-examples-fade
[ Archiwum\
przykładów kodu](../../book.html){.btn .btn-secondary .btn-block}
:::

::: {.banner-examples-text style="margin-top: 1rem;"}
Kupując eBook **Wzorce projektowe. Nowoczesny podręcznik** zyskujesz
dostęp do archiwum zawierającego mnóstwo szczegółowych przykładów, które
można od razu wkleić do swojego preferowanego zintegrowanego środowiska
programistycznego.

[ Dowiedz się więcej...](../../book.html){.btn .btn-secondary
.btn-block}
:::
::::::
:::::::
:::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::
