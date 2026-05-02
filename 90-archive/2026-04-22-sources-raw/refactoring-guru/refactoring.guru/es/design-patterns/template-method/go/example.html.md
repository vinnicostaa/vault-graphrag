::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Patrones de
diseño](../../../design-patterns.html){.type} / [Template
Method](../../template-method.html){.type} / [Go](../../go.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Template
Method](../../../../images/patterns/cards/template-method-mini56e3.png?id=9f200248d88026d8e79d0f3dae411ab4){srcset="/images/patterns/cards/template-method-mini-2x.png?id=178bf56e39b3a1f548dd636076209c98 2x"}
:::

# **Template Method** en Go {#template-method-en-go .pattern-example-title-block-title}
::::

:::::::::::::: pattern-example-body
::: pattern-example-brief
**Template Method** es un patrón de diseño de comportamiento que te
permite definir el esqueleto de un algoritmo en una clase base y permite
a las subclases sobrescribir los pasos sin cambiar la estructura general
del algoritmo.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Aprende más sobre el patrón Template Method
](../../template-method.html){.btn .btn-lg .btn-primary}
:::

::::::::::: {.sidebar-navigation .with-scroll}
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

## Ejemplo conceptual {#example-0-title}

Consideremos el ejemplo de la funcionalidad de la contraseña de un solo
uso (OTP, por sus siglas en inglés). Hay varias formas de entregarle la
OTP a un usuario (SMS, correo electrónico, etc.). Pero,
independientemente de si se trata de un SMS o un correo electrónico, el
proceso OTP es el mismo:

1.  Generar un número de n dígitos al azar.
2.  Guardar el número en caché para su posterior verificación.
3.  Preparar el contenido.
4.  Enviar la notificación.

Todos los nuevos tipos de OTP que se introduzcan en el futuro,
probablemente pasen por todos estos pasos.

De modo que tenemos un escenario en el que los pasos de una operación
concreta son los mismos, pero la implementación de estos pasos puede
variar. Ésta es una situación adecuada para considerar utilizar el
patrón Template Method.

Primero, definimos un algoritmo plantilla base que consista en un número
fijo de métodos. Éste será nuestro método plantilla. Después,
implementaremos cada uno de los métodos de los pasos, pero dejaremos el
método plantilla sin variar.

#### []{#example-0--otp-go .anchor} **otp.go:** Método plantilla

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

#### []{#example-0--sms-go .anchor} **sms.go:** Implementación concreta

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

#### []{#example-0--email-go .anchor} **email.go:** Implementación concreta

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

#### []{#example-0--main-go .anchor} **main.go:** Código cliente

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

#### []{#example-0--output-txt .anchor} **output.txt:** Resultado de la ejecución

<figure class="code">
<pre class="code" lang="output"><code>SMS: generating random otp 1234
SMS: saving otp: 1234 to cache
SMS: sending sms: SMS OTP for login is 1234

EMAIL: generating random otp 1234
EMAIL: saving otp: 1234 to cache
EMAIL: sending email: EMAIL OTP for login is 1234</code></pre>
</figure>

::: next
#### Leer siguiente

[Visitor en Go []{.fa
.fa-arrow-right}](../../visitor/go/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Volver

[[]{.fa .fa-arrow-left} Strategy en
Go](../../strategy/go/example.html){.btn .btn-default rel="prev"}
:::

## **Template Method** en otros lenguajes

[![Template Method en
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Template Method en C#"){.prog-lang-link}
[![Template Method en
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Template Method en C++"){.prog-lang-link}
[![Template Method en
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Template Method en Java"){.prog-lang-link}
[![Template Method en
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Template Method en PHP"){.prog-lang-link}
[![Template Method en
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Template Method en Python"){.prog-lang-link}
[![Template Method en
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Template Method en Ruby"){.prog-lang-link}
[![Template Method en
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Template Method en Rust"){.prog-lang-link}
[![Template Method en
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Template Method en Swift"){.prog-lang-link}
[![Template Method en
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Template Method en TypeScript"){.prog-lang-link}
::::::::::::::::::::

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
::::::::::::::::::::::::::
:::::::::::::::::::::::::::
