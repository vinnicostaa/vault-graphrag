::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} /
[デザインパターン](../../../design-patterns.html){.type} / [Template
Method](../../template-method.html){.type} / [Go](../../go.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Template
Method](../../../../images/patterns/cards/template-method-mini56e3.png?id=9f200248d88026d8e79d0f3dae411ab4){srcset="/images/patterns/cards/template-method-mini-2x.png?id=178bf56e39b3a1f548dd636076209c98 2x"}
:::

# **Template Method** を Go で {#template-method-を-go-で .pattern-example-title-block-title}
::::

:::::::::::::: pattern-example-body
::: pattern-example-brief
**Template Method** は[、]{.chpule2}[
]{.chpuri2}振る舞いに関するデザインパターンの一つで[、]{.chpule2}[
]{.chpuri2}アルゴリズムの骨組みを基底クラスで定義し[、]{.chpule2}[
]{.chpuri2}サブクラスではアルゴリズムの全体的な構造は残したまま[、]{.chpule2}[
]{.chpuri2}ステップを上書きします[。]{.chpule2}[ ]{.chpuri2}
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Template Method の詳細 ](../../template-method.html){.btn .btn-lg
.btn-primary}
:::

::::::::::: {.sidebar-navigation .with-scroll}
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

## 概念的な例 {#example-0-title}

One Time Password[ ]{.chpule1}[（]{.chpuri1}OTP[）]{.chpule2}[
]{.chpuri2}機能を考えてみましょう[。]{.chpule2}[
]{.chpuri2}使い切りパスワードをユーザーに配布するには[、]{.chpule2}[
]{.chpuri2}SMS[、]{.chpule2}[
]{.chpuri2}電子メールなどいくつかの方法があります[。]{.chpule2}[
]{.chpuri2}しかし[、]{.chpule2}[ ]{.chpuri2}SMS
であろうと電子メールであろうと[、]{.chpule2}[ ]{.chpuri2}OTP
の全体的プロセスは一緒です[：]{.chpule2}[ ]{.chpuri2}

1.  n 桁の乱数を発生[。]{.chpule2}[ ]{.chpuri2}
2.  後の検証のため[、]{.chpule2}[
    ]{.chpuri2}この番号をキャッシュに保存[。]{.chpule2}[ ]{.chpuri2}
3.  通知内容を用意[。]{.chpule2}[ ]{.chpuri2}
4.  通知を送信[。]{.chpule2}[ ]{.chpuri2}

将来導入されるかも知れない新型 OTP
もおそらくこれらのステップを踏むものと思われます[。]{.chpule2}[
]{.chpuri2}

というわけで[、]{.chpule2}[ ]{.chpuri2}我々は[、]{.chpule2}[
]{.chpuri2}ある特定の作業のステップの大枠は同じだが[、]{.chpule2}[
]{.chpuri2}ステップの実装は異なる[、]{.chpule2}[
]{.chpuri2}という状況にあります[。]{.chpule2}[
]{.chpuri2}これは[、]{.chpule2}[ ]{.chpuri2}Template Method
パターンを適用するのに適した状況です[。]{.chpule2}[ ]{.chpuri2}

最初に[、]{.chpule2}[
]{.chpuri2}決まった数のメソッドからなる[、]{.chpule2}[
]{.chpuri2}基底の雛形のアルゴリズムを定義します[。]{.chpule2}[
]{.chpuri2}これが[、]{.chpule2}[
]{.chpuri2}我々のテンプレート・メソッドです[。]{.chpule2}[
]{.chpuri2}次にステップのメソッドを実装しますが[、]{.chpule2}[
]{.chpuri2}テンプレート・メソッドはそのままにします[。]{.chpule2}[
]{.chpuri2}

#### []{#example-0--otp-go .anchor} **otp.go:** テンプレート・メソッド

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

#### []{#example-0--sms-go .anchor} **sms.go:** 具象実装

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

#### []{#example-0--email-go .anchor} **email.go:** 具象実装

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

#### []{#example-0--main-go .anchor} **main.go:** クライアント・コード

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

#### []{#example-0--output-txt .anchor} **output.txt:** 実行結果

<figure class="code">
<pre class="code" lang="output"><code>SMS: generating random otp 1234
SMS: saving otp: 1234 to cache
SMS: sending sms: SMS OTP for login is 1234

EMAIL: generating random otp 1234
EMAIL: saving otp: 1234 to cache
EMAIL: sending email: EMAIL OTP for login is 1234</code></pre>
</figure>

::: next
#### 次を読む

[Visitor を Go で []{.fa
.fa-arrow-right}](../../visitor/go/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### 戻る

[[]{.fa .fa-arrow-left} Strategy を Go
で](../../strategy/go/example.html){.btn .btn-default rel="prev"}
:::

## 他言語での **Template Method**

[![Template Method を C#
で](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Template Method を C# で"){.prog-lang-link}
[![Template Method を C++
で](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Template Method を C++ で"){.prog-lang-link}
[![Template Method を Java
で](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Template Method を Java で"){.prog-lang-link}
[![Template Method を PHP
で](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Template Method を PHP で"){.prog-lang-link}
[![Template Method を Python
で](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Template Method を Python で"){.prog-lang-link}
[![Template Method を Ruby
で](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Template Method を Ruby で"){.prog-lang-link}
[![Template Method を Rust
で](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Template Method を Rust で"){.prog-lang-link}
[![Template Method を Swift
で](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Template Method を Swift で"){.prog-lang-link}
[![Template Method を TypeScript
で](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Template Method を TypeScript で"){.prog-lang-link}
::::::::::::::::::::

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
::::::::::::::::::::::::::
:::::::::::::::::::::::::::
