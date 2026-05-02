:::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} /
[デザインパターン](../../../design-patterns.html){.type} /
[Adapter](../../adapter.html){.type} / [PHP](../../php.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Adapter](../../../../images/patterns/cards/adapter-minid866.png?id=b2ee4f681fb589be5a0685b94692aebb){srcset="/images/patterns/cards/adapter-mini-2x.png?id=8274d99afbbe9c63bfbfd0d68ceeffc7 2x"}
:::

# **Adapter** を PHP で {#adapter-を-php-で .pattern-example-title-block-title}
::::

:::::::::::::: pattern-example-body
::: pattern-example-brief
**Adapter** は[、]{.chpule2}[
]{.chpuri2}構造に関するデザインパターンの一つで[、]{.chpule2}[
]{.chpuri2}非互換なオブジェクト同士の協働を可能とします[。]{.chpule2}[
]{.chpuri2}

アダプターは二つのオブジェクト間のラッパーとして機能します[。]{.chpule2}[
]{.chpuri2}一方のオブジェクトへの呼び出しを捕まえ[、]{.chpule2}[
]{.chpuri2}二つ目のオブジェクトが認識可能なデータ形式とインターフェースに変換します[。]{.chpule2}[
]{.chpuri2}
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Adapter の詳細 ](../../adapter.html){.btn .btn-lg .btn-primary}
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
 [index](#example-0--index-php)
:::

::: en-file
 [Output](#example-0--Output-txt)
:::

::: en-intro
 [現実的な例](#example-1)
:::

::: en-file
 [index](#example-1--index-php)
:::

::: en-file
 [Output](#example-1--Output-txt)
:::
:::::::::::

**複雑度[：]{.chpule2}[ ]{.chpuri2}**[]{.fa-stars}

**人気度[：]{.chpule2}[ ]{.chpuri2}**[]{.fa-stars}

**使用例[：]{.chpule2}[ ]{.chpuri2}** Adapter パターンは PHP
コードではよく見かけます[。]{.chpule2}[
]{.chpuri2}旧来のコードに基づいたシステムでよく使用されます[。]{.chpule2}[
]{.chpuri2}そのような場合[、]{.chpule2}[
]{.chpuri2}アダプターは旧来のコードを現代のクラスから使用可能にします[。]{.chpule2}[
]{.chpuri2}

**見つけ方[：]{.chpule2}[ ]{.chpuri2}**アダプターは[、]{.chpule2}[
]{.chpuri2}異なる抽象クラスまたはインターフェース型のインスタンスを取るコンストラクターの存在により認識できます[。]{.chpule2}[
]{.chpuri2}アダプターは[、]{.chpule2}[
]{.chpuri2}いずれかのメソッドへの呼び出しを受け取ると[、]{.chpule2}[
]{.chpuri2}パラメーターを適切な形式に変換し[、]{.chpule2}[
]{.chpuri2}ラップされたオブジェクトの一つまたは複数のメソッドを呼び出します[。]{.chpule2}[
]{.chpuri2}
::::::::::::::

::: {#nav-tab .nav .nav-tabs .nav-justified role="tablist"}
[概念的な例](#example-0){#example-0-tab .nav-item .nav-link .active
toggle="tab" role="tab" aria-controls="example-0" aria-selected="true"}
[現実的な例](#example-1){#example-1-tab .nav-item .nav-link toggle="tab"
role="tab" aria-controls="example-1" aria-selected="true"}
:::

::::: {#nav-tabContent .pattern-example-tab-content .tab-content}
::: {#example-0 .tab-pane .show .active role="tabpanel" aria-labelledby="example-0-tab" style="padding-top: 24px"}
## 概念的な例 {#example-0-title}

この例は[、]{.chpule2}[ ]{.chpuri2}**Adapter**
デザインパターンの構造を説明するためのものです[。]{.chpule2}[
]{.chpuri2}以下の質問に答えることを目的としています[：]{.chpule2}[
]{.chpuri2}

-   どういうクラスからできているか[？]{.chpule2}[ ]{.chpuri2}
-   それぞれのクラスの役割は[？]{.chpule2}[ ]{.chpuri2}
-   パターンの要素同士はどう関係しているのか[？]{.chpule2}[ ]{.chpuri2}

ここでパターンの構造を学んだ後だと[、]{.chpule2}[
]{.chpuri2}これに続く[、]{.chpule2}[ ]{.chpuri2}現実世界の PHP
でのユースケースが理解しやすくなります[。]{.chpule2}[ ]{.chpuri2}

#### []{#example-0--index-php .anchor} **index.php:** 概念的な例

<figure class="code">
<pre class="code" lang="php"><code>&lt;?php

namespace RefactoringGuru\Adapter\Conceptual;

/**
 * The Target defines the domain-specific interface used by the client code.
 */
class Target
{
    public function request(): string
    {
        return &quot;Target: The default target&#39;s behavior.&quot;;
    }
}

/**
 * The Adaptee contains some useful behavior, but its interface is incompatible
 * with the existing client code. The Adaptee needs some adaptation before the
 * client code can use it.
 */
class Adaptee
{
    public function specificRequest(): string
    {
        return &quot;.eetpadA eht fo roivaheb laicepS&quot;;
    }
}

/**
 * The Adapter makes the Adaptee&#39;s interface compatible with the Target&#39;s
 * interface.
 */
class Adapter extends Target
{
    private $adaptee;

    public function __construct(Adaptee $adaptee)
    {
        $this-&gt;adaptee = $adaptee;
    }

    public function request(): string
    {
        return &quot;Adapter: (TRANSLATED) &quot; . strrev($this-&gt;adaptee-&gt;specificRequest());
    }
}

/**
 * The client code supports all classes that follow the Target interface.
 */
function clientCode(Target $target)
{
    echo $target-&gt;request();
}

echo &quot;Client: I can work just fine with the Target objects:\n&quot;;
$target = new Target();
clientCode($target);
echo &quot;\n\n&quot;;

$adaptee = new Adaptee();
echo &quot;Client: The Adaptee class has a weird interface. See, I don&#39;t understand it:\n&quot;;
echo &quot;Adaptee: &quot; . $adaptee-&gt;specificRequest();
echo &quot;\n\n&quot;;

echo &quot;Client: But I can work with it via the Adapter:\n&quot;;
$adapter = new Adapter($adaptee);
clientCode($adapter);</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** 実行結果

<figure class="code">
<pre class="code" lang="output"><code>Client: I can work just fine with the Target objects:
Target: The default target&#39;s behavior.

Client: The Adaptee class has a weird interface. See, I don&#39;t understand it:
Adaptee: .eetpadA eht fo roivaheb laicepS

Client: But I can work with it via the Adapter:
Adapter: (TRANSLATED) Special behavior of the Adaptee.</code></pre>
</figure>
:::

::: {#example-1 .tab-pane role="tabpanel" aria-labelledby="example-1-tab" style="padding-top: 24px"}
## 現実的な例 {#example-1-title}

**Adapter** パターンでは[、]{.chpule2}[
]{.chpuri2}自分のコードの大部分と互換性がない他社作成のクラスや旧来のクラスを使えるようにします[。]{.chpule2}[
]{.chpuri2}たとえば[、]{.chpule2}[ ]{.chpuri2}Slack[、]{.chpule2}[
]{.chpuri2}Facebook[、]{.chpule2}[
]{.chpuri2}SMS[[、]{.ls-1}]{.chpule2}[ ]{.chpuri2}​[
]{.chpule1}[（]{.chpuri1}その他適宜[）]{.chpule2}[
]{.chpuri2}などの他社のサービスをサポートするために自分のアプリの通知インターフェースを書き直す代わりに[、]{.chpule2}[
]{.chpuri2}アプリからの呼び出しを[、]{.chpule2}[
]{.chpuri2}それぞれの他社作成クラスが要求するインターフェースとデータ形式に適合させる特別なラッパーを書くことができます[。]{.chpule2}[
]{.chpuri2}

#### []{#example-1--index-php .anchor} **index.php:** 現実的な例

<figure class="code">
<pre class="code" lang="php"><code>&lt;?php

namespace RefactoringGuru\Adapter\RealWorld;

/**
 * The Target interface represents the interface that your application&#39;s classes
 * already follow.
 */
interface Notification
{
    public function send(string $title, string $message);
}

/**
 * Here&#39;s an example of the existing class that follows the Target interface.
 *
 * The truth is that many real apps may not have this interface clearly defined.
 * If you&#39;re in that boat, your best bet would be to extend the Adapter from one
 * of your application&#39;s existing classes. If that&#39;s awkward (for instance,
 * SlackNotification doesn&#39;t feel like a subclass of EmailNotification), then
 * extracting an interface should be your first step.
 */
class EmailNotification implements Notification
{
    private $adminEmail;

    public function __construct(string $adminEmail)
    {
        $this-&gt;adminEmail = $adminEmail;
    }

    public function send(string $title, string $message): void
    {
        mail($this-&gt;adminEmail, $title, $message);
        echo &quot;Sent email with title &#39;$title&#39; to &#39;{$this-&gt;adminEmail}&#39; that says &#39;$message&#39;.&quot;;
    }
}

/**
 * The Adaptee is some useful class, incompatible with the Target interface. You
 * can&#39;t just go in and change the code of the class to follow the Target
 * interface, since the code might be provided by a 3rd-party library.
 */
class SlackApi
{
    private $login;
    private $apiKey;

    public function __construct(string $login, string $apiKey)
    {
        $this-&gt;login = $login;
        $this-&gt;apiKey = $apiKey;
    }

    public function logIn(): void
    {
        // Send authentication request to Slack web service.
        echo &quot;Logged in to a slack account &#39;{$this-&gt;login}&#39;.\n&quot;;
    }

    public function sendMessage(string $chatId, string $message): void
    {
        // Send message post request to Slack web service.
        echo &quot;Posted following message into the &#39;$chatId&#39; chat: &#39;$message&#39;.\n&quot;;
    }
}

/**
 * The Adapter is a class that links the Target interface and the Adaptee class.
 * In this case, it allows the application to send notifications using Slack
 * API.
 */
class SlackNotification implements Notification
{
    private $slack;
    private $chatId;

    public function __construct(SlackApi $slack, string $chatId)
    {
        $this-&gt;slack = $slack;
        $this-&gt;chatId = $chatId;
    }

    /**
     * An Adapter is not only capable of adapting interfaces, but it can also
     * convert incoming data to the format required by the Adaptee.
     */
    public function send(string $title, string $message): void
    {
        $slackMessage = &quot;#&quot; . $title . &quot;# &quot; . strip_tags($message);
        $this-&gt;slack-&gt;logIn();
        $this-&gt;slack-&gt;sendMessage($this-&gt;chatId, $slackMessage);
    }
}

/**
 * The client code can work with any class that follows the Target interface.
 */
function clientCode(Notification $notification)
{
    // ...

    echo $notification-&gt;send(
        &quot;Website is down!&quot;,
        &quot;&lt;strong style=&#39;color:red;font-size: 50px;&#39;&gt;Alert!&lt;/strong&gt; &quot; .
        &quot;Our website is not responding. Call admins and bring it up!&quot;
    );

    // ...
}

echo &quot;Client code is designed correctly and works with email notifications:\n&quot;;
$notification = new EmailNotification(&quot;developers@example.com&quot;);
clientCode($notification);
echo &quot;\n\n&quot;;


echo &quot;The same client code can work with other classes via adapter:\n&quot;;
$slackApi = new SlackApi(&quot;example.com&quot;, &quot;XXXXXXXX&quot;);
$notification = new SlackNotification($slackApi, &quot;Example.com Developers&quot;);
clientCode($notification);</code></pre>
</figure>

#### []{#example-1--Output-txt .anchor} **Output.txt:** 実行結果

<figure class="code">
<pre class="code" lang="output"><code>Client code is designed correctly and works with email notifications:
Sent email with title &#39;Website is down!&#39; to &#39;developers@example.com&#39; that says &#39;&lt;strong style=&#39;color:red;font-size: 50px;&#39;&gt;Alert!&lt;/strong&gt; Our website is not responding. Call admins and bring it up!&#39;.

The same client code can work with other classes via adapter:
Logged in to a slack account &#39;example.com&#39;.
Posted following message into the &#39;Example.com Developers&#39; chat: &#39;#Website is down!# Alert! Our website is not responding. Call admins and bring it up!&#39;.</code></pre>
</figure>
:::
:::::

::: {#nav-tab .nav .nav-tabs .nav-tabs-bottom .nav-justified role="tablist"}
[概念的な例](#example-0){#example-0-tab-bottom .nav-item .nav-link
.active toggle="tab" role="tab" aria-controls="example-0"
aria-selected="true"} [現実的な例](#example-1){#example-1-tab-bottom
.nav-item .nav-link toggle="tab" role="tab" aria-controls="example-1"
aria-selected="true"}
:::

::: next
#### 次を読む

[Bridge を PHP で []{.fa
.fa-arrow-right}](../../bridge/php/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### 戻る

[[]{.fa .fa-arrow-left} Singleton を PHP
で](../../singleton/php/example.html){.btn .btn-default rel="prev"}
:::

## 他言語での **Adapter**

[![Adapter を C#
で](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Adapter を C# で"){.prog-lang-link}
[![Adapter を C++
で](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Adapter を C++ で"){.prog-lang-link}
[![Adapter を Go
で](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Adapter を Go で"){.prog-lang-link}
[![Adapter を Java
で](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Adapter を Java で"){.prog-lang-link}
[![Adapter を Python
で](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Adapter を Python で"){.prog-lang-link}
[![Adapter を Ruby
で](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Adapter を Ruby で"){.prog-lang-link}
[![Adapter を Rust
で](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Adapter を Rust で"){.prog-lang-link}
[![Adapter を Swift
で](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Adapter を Swift で"){.prog-lang-link}
[![Adapter を TypeScript
で](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Adapter を TypeScript で"){.prog-lang-link}
:::::::::::::::::::::::::

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
:::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::
