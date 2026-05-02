::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Patrons de
conception](../../../design-patterns.html){.type} / [Patron de
méthode](../../template-method.html){.type} / [Go](../../go.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Patron de
méthode](../../../../images/patterns/cards/template-method-mini56e3.png?id=9f200248d88026d8e79d0f3dae411ab4){srcset="/images/patterns/cards/template-method-mini-2x.png?id=178bf56e39b3a1f548dd636076209c98 2x"}
:::

# **Patron de méthode** en Go {#patron-de-méthode-en-go .pattern-example-title-block-title}
::::

:::::::::::::: pattern-example-body
::: pattern-example-brief
Le **Patron de méthode** est un patron de conception comportemental qui
permet de définir le squelette d'un algorithme dans la classe de base,
et laisse les sous-classes redéfinir les étapes sans modifier la
structure globale de l'algorithme.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ En savoir plus sur la patron Patron de méthode
](../../template-method.html){.btn .btn-lg .btn-primary}
:::

::::::::::: {.sidebar-navigation .with-scroll}
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
 [otp](#example-0--otp-go)
:::

::: en-file
 [sms](#example-0--sms-go)
:::

::: en-file
 [email](#example-0--email-go)
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

## Exemple conceptuel {#example-0-title}

Imaginons un exemple de fonctionnalité de mot de passe à usage unique.
Nous disposons de plusieurs possibilités pour envoyer le mot de passe à
un utilisateur (SMS, e-mail, etc.). Quelle que soit la façon de le
communiquer, le processus complet pour le mot de passe à usage unique
reste le même :

1.  Générer un nombre aléatoire avec n chiffres.
2.  Sauvegarder ce nombre dans le cache pour vérification ultérieure.
3.  Préparer le contenu.
4.  Envoyer la notification.

Tout nouveau type de mot de passe à usage unique ajouté dans le futur
suivra probablement les mêmes étapes.

Nous nous retrouvons donc dans un scénario où les étapes d'un traitement
particulier resteront les mêmes, seule leur implémentation pourra
différer. C'est la situation idéale pour envisager d'utiliser le patron
de méthode.

Tout d'abord, nous allons définir un modèle de base pour l'algorithme
qui va être composé d'un nombre fixe de méthodes (notre patron de
méthode). Ensuite nous implémenterons toutes les méthodes des étapes,
sans toucher au patron de méthode.

#### []{#example-0--otp-go .anchor} **otp.go:** Patron de méthode

<figure class="code">
<pre class="code" lang="go"><code>package main

type IOtp interface {
    genRandomOTP(int) string
    saveOTPCache(string)
    getMessage(string) string
    sendNotification(string) error
}

// type otp struct {
// }

// func (o *otp) genAndSendOTP(iOtp iOtp, otpLength int) error {
//  otp := iOtp.genRandomOTP(otpLength)
//  iOtp.saveOTPCache(otp)
//  message := iOtp.getMessage(otp)
//  err := iOtp.sendNotification(message)
//  if err != nil {
//      return err
//  }
//  return nil
// }

type Otp struct {
    iOtp IOtp
}

func (o *Otp) genAndSendOTP(otpLength int) error {
    otp := o.iOtp.genRandomOTP(otpLength)
    o.iOtp.saveOTPCache(otp)
    message := o.iOtp.getMessage(otp)
    err := o.iOtp.sendNotification(message)
    if err != nil {
        return err
    }
    return nil
}</code></pre>
</figure>

#### []{#example-0--sms-go .anchor} **sms.go:** Implémentation concrète

<figure class="code">
<pre class="code" lang="go"><code>package main

import &quot;fmt&quot;

type Sms struct {
    Otp
}

func (s *Sms) genRandomOTP(len int) string {
    randomOTP := &quot;1234&quot;
    fmt.Printf(&quot;SMS: generating random otp %s\n&quot;, randomOTP)
    return randomOTP
}

func (s *Sms) saveOTPCache(otp string) {
    fmt.Printf(&quot;SMS: saving otp: %s to cache\n&quot;, otp)
}

func (s *Sms) getMessage(otp string) string {
    return &quot;SMS OTP for login is &quot; + otp
}

func (s *Sms) sendNotification(message string) error {
    fmt.Printf(&quot;SMS: sending sms: %s\n&quot;, message)
    return nil
}</code></pre>
</figure>

#### []{#example-0--email-go .anchor} **email.go:** Implémentation concrète

<figure class="code">
<pre class="code" lang="go"><code>package main

import &quot;fmt&quot;

type Email struct {
    Otp
}

func (s *Email) genRandomOTP(len int) string {
    randomOTP := &quot;1234&quot;
    fmt.Printf(&quot;EMAIL: generating random otp %s\n&quot;, randomOTP)
    return randomOTP
}

func (s *Email) saveOTPCache(otp string) {
    fmt.Printf(&quot;EMAIL: saving otp: %s to cache\n&quot;, otp)
}

func (s *Email) getMessage(otp string) string {
    return &quot;EMAIL OTP for login is &quot; + otp
}

func (s *Email) sendNotification(message string) error {
    fmt.Printf(&quot;EMAIL: sending email: %s\n&quot;, message)
    return nil
}</code></pre>
</figure>

#### []{#example-0--main-go .anchor} **main.go:** Code client

<figure class="code">
<pre class="code" lang="go"><code>package main

import &quot;fmt&quot;

func main() {
    // otp := otp{}

    // smsOTP := &amp;sms{
    //  otp: otp,
    // }

    // smsOTP.genAndSendOTP(smsOTP, 4)

    // emailOTP := &amp;email{
    //  otp: otp,
    // }
    // emailOTP.genAndSendOTP(emailOTP, 4)
    // fmt.Scanln()
    smsOTP := &amp;Sms{}
    o := Otp{
        iOtp: smsOTP,
    }
    o.genAndSendOTP(4)

    fmt.Println(&quot;&quot;)
    emailOTP := &amp;Email{}
    o = Otp{
        iOtp: emailOTP,
    }
    o.genAndSendOTP(4)

}</code></pre>
</figure>

#### []{#example-0--output-txt .anchor} **output.txt:** Résultat de l'exécution

<figure class="code">
<pre class="code" lang="output"><code>SMS: generating random otp 1234
SMS: saving otp: 1234 to cache
SMS: sending sms: SMS OTP for login is 1234

EMAIL: generating random otp 1234
EMAIL: saving otp: 1234 to cache
EMAIL: sending email: EMAIL OTP for login is 1234</code></pre>
</figure>

::: next
#### Suivant

[Visiteur en Go []{.fa
.fa-arrow-right}](../../visitor/go/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Retour

[[]{.fa .fa-arrow-left} Stratégie en
Go](../../strategy/go/example.html){.btn .btn-default rel="prev"}
:::

## **Patron de méthode** dans les autres langues

[![Patron de méthode en
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Patron de méthode en C#"){.prog-lang-link}
[![Patron de méthode en
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Patron de méthode en C++"){.prog-lang-link}
[![Patron de méthode en
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Patron de méthode en Java"){.prog-lang-link}
[![Patron de méthode en
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Patron de méthode en PHP"){.prog-lang-link}
[![Patron de méthode en
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Patron de méthode en Python"){.prog-lang-link}
[![Patron de méthode en
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Patron de méthode en Ruby"){.prog-lang-link}
[![Patron de méthode en
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Patron de méthode en Rust"){.prog-lang-link}
[![Patron de méthode en
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Patron de méthode en Swift"){.prog-lang-link}
[![Patron de méthode en
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Patron de méthode en TypeScript"){.prog-lang-link}
::::::::::::::::::::

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
::::::::::::::::::::::::::
:::::::::::::::::::::::::::
