::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Patrones de
diseño](../../../design-patterns.html){.type} /
[Facade](../../facade.html){.type} / [Rust](../../rust.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Facade](../../../../images/patterns/cards/facade-mini14fd.png?id=71ad6fa98b168c11cb3a1a9517dedf78){srcset="/images/patterns/cards/facade-mini-2x.png?id=d4cc6a5d81a31143cc665f7ac1481ac8 2x"}
:::

# **Facade** en Rust {#facade-en-rust .pattern-example-title-block-title}
::::

:::::::::::::::: pattern-example-body
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

::::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
Navegación
:::

::: en-intro
 [Intro](#)
:::

::: en-intro
 [Conceptual Example](#example-0)
:::

::: en-file
 [wallet_facade](#example-0--wallet_facade-rs)
:::

::: en-file
 [wallet](#example-0--wallet-rs)
:::

::: en-file
 [account](#example-0--account-rs)
:::

::: en-file
 [ledger](#example-0--ledger-rs)
:::

::: en-file
 [notification](#example-0--notification-rs)
:::

::: en-file
 [security_code](#example-0--security_code-rs)
:::

::: en-file
 [main](#example-0--main-rs)
:::
:::::::::::::
::::::::::::::::

[]{#example-0}

## Conceptual Example {#example-0-title}

`pub struct WalletFacade` hides a complex logic behind its API. A single
method `add_money_to_wallet` interacts with the account, code, wallet,
notification and ledger behind the scenes.

#### []{#example-0--wallet_facade-rs .anchor} **wallet_facade.rs**

<figure class="code">
<pre class="code" lang="rust"><code>use crate::{
    account::Account, ledger::Ledger, notification::Notification, security_code::SecurityCode,
    wallet::Wallet,
};

/// Facade hides a complex logic behind the API.
pub struct WalletFacade {
    account: Account,
    wallet: Wallet,
    code: SecurityCode,
    notification: Notification,
    ledger: Ledger,
}

impl WalletFacade {
    pub fn new(account_id: String, code: u32) -&gt; Self {
        println!(&quot;Starting create account&quot;);

        let this = Self {
            account: Account::new(account_id),
            wallet: Wallet::new(),
            code: SecurityCode::new(code),
            notification: Notification,
            ledger: Ledger,
        };

        println!(&quot;Account created&quot;);
        this
    }

    pub fn add_money_to_wallet(
        &amp;mut self,
        account_id: &amp;String,
        security_code: u32,
        amount: u32,
    ) -&gt; Result&lt;(), String&gt; {
        println!(&quot;Starting add money to wallet&quot;);
        self.account.check(account_id)?;
        self.code.check(security_code)?;
        self.wallet.credit_balance(amount);
        self.notification.send_wallet_credit_notification();
        self.ledger.make_entry(account_id, &quot;credit&quot;.into(), amount);
        Ok(())
    }

    pub fn deduct_money_from_wallet(
        &amp;mut self,
        account_id: &amp;String,
        security_code: u32,
        amount: u32,
    ) -&gt; Result&lt;(), String&gt; {
        println!(&quot;Starting debit money from wallet&quot;);
        self.account.check(account_id)?;
        self.code.check(security_code)?;
        self.wallet.debit_balance(amount);
        self.notification.send_wallet_debit_notification();
        self.ledger.make_entry(account_id, &quot;debit&quot;.into(), amount);
        Ok(())
    }
}</code></pre>
</figure>

#### []{#example-0--wallet-rs .anchor} **wallet.rs**

<figure class="code">
<pre class="code" lang="rust"><code>pub struct Wallet {
    balance: u32,
}

impl Wallet {
    pub fn new() -&gt; Self {
        Self { balance: 0 }
    }

    pub fn credit_balance(&amp;mut self, amount: u32) {
        self.balance += amount;
    }

    pub fn debit_balance(&amp;mut self, amount: u32) {
        self.balance
            .checked_sub(amount)
            .expect(&quot;Balance is not sufficient&quot;);
    }
}</code></pre>
</figure>

#### []{#example-0--account-rs .anchor} **account.rs**

<figure class="code">
<pre class="code" lang="rust"><code>pub struct Account {
    name: String,
}

impl Account {
    pub fn new(name: String) -&gt; Self {
        Self { name }
    }

    pub fn check(&amp;self, name: &amp;String) -&gt; Result&lt;(), String&gt; {
        if &amp;self.name != name {
            return Err(&quot;Account name is incorrect&quot;.into());
        }

        println!(&quot;Account verified&quot;);
        Ok(())
    }
}</code></pre>
</figure>

#### []{#example-0--ledger-rs .anchor} **ledger.rs**

<figure class="code">
<pre class="code" lang="rust"><code>pub struct Ledger;

impl Ledger {
    pub fn make_entry(&amp;mut self, account_id: &amp;String, txn_type: String, amount: u32) {
        println!(
            &quot;Make ledger entry for accountId {} with transaction type {} for amount {}&quot;,
            account_id, txn_type, amount
        );
    }
}</code></pre>
</figure>

#### []{#example-0--notification-rs .anchor} **notification.rs**

<figure class="code">
<pre class="code" lang="rust"><code>pub struct Notification;

impl Notification {
    pub fn send_wallet_credit_notification(&amp;self) {
        println!(&quot;Sending wallet credit notification&quot;);
    }

    pub fn send_wallet_debit_notification(&amp;self) {
        println!(&quot;Sending wallet debit notification&quot;);
    }
}</code></pre>
</figure>

#### []{#example-0--security_code-rs .anchor} **security_code.rs**

<figure class="code">
<pre class="code" lang="rust"><code>pub struct SecurityCode {
    code: u32,
}

impl SecurityCode {
    pub fn new(code: u32) -&gt; Self {
        Self { code }
    }

    pub fn check(&amp;self, code: u32) -&gt; Result&lt;(), String&gt; {
        if self.code != code {
            return Err(&quot;Security code is incorrect&quot;.into());
        }

        println!(&quot;Security code verified&quot;);
        Ok(())
    }
}</code></pre>
</figure>

#### []{#example-0--main-rs .anchor} **main.rs**

<figure class="code">
<pre class="code" lang="rust"><code>mod account;
mod ledger;
mod notification;
mod security_code;
mod wallet;
mod wallet_facade;

use wallet_facade::WalletFacade;

fn main() -&gt; Result&lt;(), String&gt; {
    let mut wallet = WalletFacade::new(&quot;abc&quot;.into(), 1234);
    println!();

    // Wallet Facade interacts with the account, code, wallet, notification and
    // ledger behind the scenes.
    wallet.add_money_to_wallet(&amp;&quot;abc&quot;.into(), 1234, 10)?;
    println!();

    wallet.deduct_money_from_wallet(&amp;&quot;abc&quot;.into(), 1234, 5)
}</code></pre>
</figure>

### Output

<figure class="code">
<pre class="code"><code>Starting create account
Account created

Starting add money to wallet
Account verified
Security code verified
Sending wallet credit notification
Make ledger entry for accountId abc with transaction type credit for amount 10

Starting debit money from wallet
Account verified
Security code verified
Sending wallet debit notification
Make ledger entry for accountId abc with transaction type debit for amount 5</code></pre>
</figure>

::: next
#### Leer siguiente

[Flyweight en Rust []{.fa
.fa-arrow-right}](../../flyweight/rust/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Volver

[[]{.fa .fa-arrow-left} Decorator en
Rust](../../decorator/rust/example.html){.btn .btn-default rel="prev"}
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
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Facade en Go"){.prog-lang-link}
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
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Facade en Swift"){.prog-lang-link}
[![Facade en
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Facade en TypeScript"){.prog-lang-link}
::::::::::::::::::::::

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
::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::
