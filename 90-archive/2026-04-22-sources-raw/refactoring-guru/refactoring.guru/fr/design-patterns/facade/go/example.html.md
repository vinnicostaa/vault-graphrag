:::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Patrons de
conception](../../../design-patterns.html){.type} /
[Façade](../../facade.html){.type} / [Go](../../go.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Façade](../../../../images/patterns/cards/facade-mini14fd.png?id=71ad6fa98b168c11cb3a1a9517dedf78){srcset="/images/patterns/cards/facade-mini-2x.png?id=d4cc6a5d81a31143cc665f7ac1481ac8 2x"}
:::

# **Façade** en Go {#façade-en-go .pattern-example-title-block-title}
::::

::::::::::::::::: pattern-example-body
::: pattern-example-brief
La **Façade** est un patron de conception structurel qui fournit une
interface simplifiée (mais limitée) à un système complexe de classes,
bibliothèques ou frameworks.

La façade permet non seulement de diminuer la complexité générale d'une
application, mais elle permet également de rassembler les dépendances
indésirables au même endroit.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ En savoir plus sur la patron Façade ](../../facade.html){.btn .btn-lg
.btn-primary}
:::

:::::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
Navigation
:::

::: en-intro
 [Intro](#)
:::

::: en-intro
 [Exemple conceptuel](#example-0)
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

## Exemple conceptuel {#example-0-title}

On a tendance à sous-estimer ce qui se passe dans l'envers du décor
lorsque l'on commande une pizza avec une carte de crédit. Une dizaine de
sous-systèmes sont impliqués dans ce processus. En voici une liste non
exhaustive :

-   Vérifier le compte
-   Vérifier le code de sécurité
-   Solde du compte
-   Faire une entrée comptable
-   Envoyer une notification

Il est facile de s'y perdre et de tout casser dans un système aussi
complexe. Mais pour cela, nous avons le patron de conception façade : un
truc qui permet au client d'utiliser des dizaines de composants en
passant par une interface toute simple. Le client n'a besoin que de
saisir les informations de la carte, le code de sécurité, le montant à
payer et le type de l'opération. La façade s'occupe de la communication
avec les divers composants sans exposer le client aux complexités
internes.

#### []{#example-0--walletFacade-go .anchor} **walletFacade.go:** Façade

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

#### []{#example-0--account-go .anchor} **account.go:** Parties du sous-système complexe

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

#### []{#example-0--securityCode-go .anchor} **securityCode.go:** Parties du sous-système complexe

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

#### []{#example-0--wallet-go .anchor} **wallet.go:** Parties du sous-système complexe

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

#### []{#example-0--ledger-go .anchor} **ledger.go:** Parties du sous-système complexe

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

#### []{#example-0--notification-go .anchor} **notification.go:** Parties du sous-système complexe

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

#### []{#example-0--main-go .anchor} **main.go:** Code client

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

#### []{#example-0--output-txt .anchor} **output.txt:** Résultat de l'exécution

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
#### Suivant

[Poids mouche en Go []{.fa
.fa-arrow-right}](../../flyweight/go/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Retour

[[]{.fa .fa-arrow-left} Décorateur en
Go](../../decorator/go/example.html){.btn .btn-default rel="prev"}
:::

## **Façade** dans les autres langues

[![Façade en
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Façade en C#"){.prog-lang-link}
[![Façade en
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Façade en C++"){.prog-lang-link}
[![Façade en
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Façade en Java"){.prog-lang-link}
[![Façade en
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Façade en PHP"){.prog-lang-link}
[![Façade en
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Façade en Python"){.prog-lang-link}
[![Façade en
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Façade en Ruby"){.prog-lang-link}
[![Façade en
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Façade en Rust"){.prog-lang-link}
[![Façade en
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Façade en Swift"){.prog-lang-link}
[![Façade en
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Façade en TypeScript"){.prog-lang-link}
:::::::::::::::::::::::

::::::: {.prom .banner-sidebar .banner .banner-striped .banner-examples .banner-removable .banner-removable-patterns data-id="DP: Examples-IDE" creative-id="examples-sidebar-fr" position="sidebar"}
:::::: {.banner-inner style="font-size: 14px; line-height: 21px;"}
::: banner-examples-img
[![](../../../../images/patterns/banners/examples-ide91ca.png?id=3115b4b548fb96b75974e2de8f4f49bc){width="215"
height="158" loading="lazy"
srcset="/images/patterns/banners/examples-ide-2x.png?id=93c007a6d157b97c28eb213f59d716ae 2x"}](../../book.html)
:::

::: banner-examples-fade
[ Archive avec exemples](../../book.html){.btn .btn-secondary
.btn-block}
:::

::: {.banner-examples-text style="margin-top: 1rem;"}
Achetez l'ebook **Plongée au cœur des patrons de conception** et obtenez
l'accès à une archive comprenant des dizaines d'exemples détaillés qui
peuvent être ouverts dans votre environnement de développement.

[ En savoir plus...](../../book.html){.btn .btn-secondary .btn-block}
:::
::::::
:::::::
:::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::
