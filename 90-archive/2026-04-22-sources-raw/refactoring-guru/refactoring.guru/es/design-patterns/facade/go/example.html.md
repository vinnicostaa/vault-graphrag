:::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Patrones de
diseño](../../../design-patterns.html){.type} /
[Facade](../../facade.html){.type} / [Go](../../go.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Facade](../../../../images/patterns/cards/facade-mini14fd.png?id=71ad6fa98b168c11cb3a1a9517dedf78){srcset="/images/patterns/cards/facade-mini-2x.png?id=d4cc6a5d81a31143cc665f7ac1481ac8 2x"}
:::

# **Facade** en Go {#facade-en-go .pattern-example-title-block-title}
::::

::::::::::::::::: pattern-example-body
::: pattern-example-brief
**Facade** es un patrón de diseño estructural que proporciona una
interfaz simplificada (pero limitada) a un sistema complejo de clases,
bibliotecas o \_frameworks\_.

El patrón Facade disminuye la complejidad general de la aplicación, al
mismo tiempo que ayuda a mover dependencias no deseadas a un solo lugar.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Aprende más sobre el patrón Facade ](../../facade.html){.btn .btn-lg
.btn-primary}
:::

:::::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
Navegación
:::

::: en-intro
 [Intro](#)
:::

::: en-intro
 [Ejemplo conceptual](#example-0)
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

## Ejemplo conceptual {#example-0-title}

Resulta sencillo subestimar las complejidades que tienen lugar tras
bambalinas cuando pides una pizza con tu tarjeta de crédito. Decenas de
subsistemas actúan en este proceso. Aquí tienes una pequeña muestra de
ellos:

-   Comprobar la cuenta
-   Comprobar el PIN de seguridad
-   Balance de crédito/débito
-   Realizar la entrada del movimiento
-   Enviar la notificación

En un sistema complejo como éste, es fácil perderse y es fácil
descomponer algo si se hacen las cosas mal. Por eso existe el concepto
del patrón Facade: algo que permite al cliente trabajar con decenas de
componentes utilizando una interfaz simple. El cliente solo tiene que
introducir los datos de la tarjeta, el pin de seguridad, la cantidad a
pagar y el tipo de operación. El patrón Facade dirige las comunicaciones
con varios componentes sin exponer al cliente a las complejidades
internas.

#### []{#example-0--walletFacade-go .anchor} **walletFacade.go:** Fachada

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

#### []{#example-0--account-go .anchor} **account.go:** Partes del subsistema complejo

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

#### []{#example-0--securityCode-go .anchor} **securityCode.go:** Partes del subsistema complejo

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

#### []{#example-0--wallet-go .anchor} **wallet.go:** Partes del subsistema complejo

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

#### []{#example-0--ledger-go .anchor} **ledger.go:** Partes del subsistema complejo

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

#### []{#example-0--notification-go .anchor} **notification.go:** Partes del subsistema complejo

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

#### []{#example-0--main-go .anchor} **main.go:** Código cliente

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

#### []{#example-0--output-txt .anchor} **output.txt:** Resultado de la ejecución

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
#### Leer siguiente

[Flyweight en Go []{.fa
.fa-arrow-right}](../../flyweight/go/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Volver

[[]{.fa .fa-arrow-left} Decorator en
Go](../../decorator/go/example.html){.btn .btn-default rel="prev"}
:::

## **Facade** en otros lenguajes

[![Facade en
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Facade en C#"){.prog-lang-link}
[![Facade en
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Facade en C++"){.prog-lang-link}
[![Facade en
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Facade en Java"){.prog-lang-link}
[![Facade en
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Facade en PHP"){.prog-lang-link}
[![Facade en
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Facade en Python"){.prog-lang-link}
[![Facade en
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Facade en Ruby"){.prog-lang-link}
[![Facade en
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Facade en Rust"){.prog-lang-link}
[![Facade en
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Facade en Swift"){.prog-lang-link}
[![Facade en
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Facade en TypeScript"){.prog-lang-link}
:::::::::::::::::::::::

::::::: {.prom .banner-sidebar .banner .banner-striped .banner-examples .banner-removable .banner-removable-patterns data-id="DP: Examples-IDE" creative-id="examples-sidebar-es" position="sidebar"}
:::::: {.banner-inner style="font-size: 14px; line-height: 21px;"}
::: banner-examples-img
[![](../../../../images/patterns/banners/examples-ide91ca.png?id=3115b4b548fb96b75974e2de8f4f49bc){width="215"
height="158" loading="lazy"
srcset="/images/patterns/banners/examples-ide-2x.png?id=93c007a6d157b97c28eb213f59d716ae 2x"}](../../book.html)
:::

::: banner-examples-fade
[ Archivo con ejemplos de código](../../book.html){.btn .btn-secondary
.btn-block}
:::

::: {.banner-examples-text style="margin-top: 1rem;"}
Compra el libro electrónico **Sumérgete en los patrones de diseño** para
tener acceso al archivo con decenas de ejemplos detallados que puedes
abrir en tu IDE.

[ Saber más...](../../book.html){.btn .btn-secondary .btn-block}
:::
::::::
:::::::
:::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::
