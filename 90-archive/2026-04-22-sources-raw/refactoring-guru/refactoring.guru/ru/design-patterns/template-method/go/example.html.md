::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Паттерны
проектирования](../../../design-patterns.html){.type} / [Шаблонный
метод](../../template-method.html){.type} / [Go](../../go.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Шаблонный
метод](../../../../images/patterns/cards/template-method-mini56e3.png?id=9f200248d88026d8e79d0f3dae411ab4){srcset="/images/patterns/cards/template-method-mini-2x.png?id=178bf56e39b3a1f548dd636076209c98 2x"}
:::

# **Шаблонный метод** на Go {#шаблонный-метод-на-go .pattern-example-title-block-title}
::::

:::::::::::::: pattern-example-body
::: pattern-example-brief
**Шаблонный метод** --- это поведенческий паттерн, задающий скелет
алгоритма в суперклассе и заставляющий подклассы реализовать конкретные
шаги этого алгоритма.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Подробней о паттерне Шаблонный метод
](../../template-method.html){.btn .btn-lg .btn-primary}
:::

::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
Навигация
:::

::: en-intro
 [Интро](#)
:::

::: en-intro
 [Концептуальный пример](#example-0)
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

## Концептуальный пример {#example-0-title}

Давайте разберем пример функционала одноразового пароля (OTP -- One Time
Password). Он может быть доставлен пользователю разными путями (СМС,
электронная почта и т. д.), но независимо от способа доставки, сам
процесс OTP один и тот же:

1.  Создать случайное число с n-ым количеством цифр.
2.  Сохранить этот номер в кэш для дальнейшей верификации.
3.  Подготовить содержимое.
4.  Отправить оповещение.

Возможные OTP, которые будут представлены в будущем, скорее всего также
будут использовать процедуру выше.

В таком случае шаги конкретной операции одинаковы, но их реализация
может отличаться. Такая ситуация подходит для использования паттерна
Шаблонный метод.

Сперва мы определим базовый шаблонный алгоритм, который состоит из
фиксированного количества методов. Это и будет нашим шаблонным методом.
После этого мы реализуем методы для каждого шага, но шаблонный метод при
этом трогать не будем.

#### []{#example-0--otp-go .anchor} **otp.go:** Шаблонный метод

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

#### []{#example-0--sms-go .anchor} **sms.go:** Конкретная реализация

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

#### []{#example-0--email-go .anchor} **email.go:** Конкретная реализация

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

#### []{#example-0--main-go .anchor} **main.go:** Клиентский код

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

#### []{#example-0--output-txt .anchor} **output.txt:** Результат выполнения

<figure class="code">
<pre class="code" lang="output"><code>SMS: generating random otp 1234
SMS: saving otp: 1234 to cache
SMS: sending sms: SMS OTP for login is 1234

EMAIL: generating random otp 1234
EMAIL: saving otp: 1234 to cache
EMAIL: sending email: EMAIL OTP for login is 1234</code></pre>
</figure>

::: next
#### Читаем дальше

[Посетитель на Go []{.fa
.fa-arrow-right}](../../visitor/go/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Стратегия на
Go](../../strategy/go/example.html){.btn .btn-default rel="prev"}
:::

## **Шаблонный метод** на других языках программирования

[![Шаблонный метод на
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Шаблонный метод на C#"){.prog-lang-link}
[![Шаблонный метод на
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Шаблонный метод на C++"){.prog-lang-link}
[![Шаблонный метод на
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Шаблонный метод на Java"){.prog-lang-link}
[![Шаблонный метод на
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Шаблонный метод на PHP"){.prog-lang-link}
[![Шаблонный метод на
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Шаблонный метод на Python"){.prog-lang-link}
[![Шаблонный метод на
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Шаблонный метод на Ruby"){.prog-lang-link}
[![Шаблонный метод на
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Шаблонный метод на Rust"){.prog-lang-link}
[![Шаблонный метод на
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Шаблонный метод на Swift"){.prog-lang-link}
[![Шаблонный метод на
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Шаблонный метод на TypeScript"){.prog-lang-link}
::::::::::::::::::::

::::::: {.prom .banner-sidebar .banner .banner-striped .banner-examples .banner-removable .banner-removable-patterns data-id="DP: Examples-IDE" creative-id="examples-sidebar-ru" position="sidebar"}
:::::: {.banner-inner style="font-size: 14px; line-height: 21px;"}
::: banner-examples-img
[![](../../../../images/patterns/banners/examples-ide91ca.png?id=3115b4b548fb96b75974e2de8f4f49bc){width="215"
height="158" loading="lazy"
srcset="/images/patterns/banners/examples-ide-2x.png?id=93c007a6d157b97c28eb213f59d716ae 2x"}](../../book.html)
:::

::: banner-examples-fade
[ Архив с примерами](../../book.html){.btn .btn-secondary .btn-block}
:::

::: {.banner-examples-text style="margin-top: 1rem;"}
Купи книгу **Погружение в Паттерны** и получи архив с десятками
детальных примеров, которые можно открывать прямо в IDE.

[ Узнать больше...](../../book.html){.btn .btn-secondary .btn-block}
:::
::::::
:::::::
::::::::::::::::::::::::::
:::::::::::::::::::::::::::
