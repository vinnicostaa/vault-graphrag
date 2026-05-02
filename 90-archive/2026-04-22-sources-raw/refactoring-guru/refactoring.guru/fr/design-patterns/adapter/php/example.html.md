:::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Patrons de
conception](../../../design-patterns.html){.type} /
[Adaptateur](../../adapter.html){.type} / [PHP](../../php.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Adaptateur](../../../../images/patterns/cards/adapter-minid866.png?id=b2ee4f681fb589be5a0685b94692aebb){srcset="/images/patterns/cards/adapter-mini-2x.png?id=8274d99afbbe9c63bfbfd0d68ceeffc7 2x"}
:::

# **Adaptateur** en PHP {#adaptateur-en-php .pattern-example-title-block-title}
::::

:::::::::::::: pattern-example-body
::: pattern-example-brief
L'**Adaptateur** est un patron de conception structurel qui permet à des
objets incompatibles de collaborer.

L'adaptateur fait office d'emballeur entre les deux objets. Il récupère
les appels à un objet et les met dans un format et une interface
reconnaissables par le second objet.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ En savoir plus sur la patron Adaptateur ](../../adapter.html){.btn
.btn-lg .btn-primary}
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
 [index](#example-0--index-php)
:::

::: en-file
 [Output](#example-0--Output-txt)
:::

::: en-intro
 [Analogie du monde réel](#example-1)
:::

::: en-file
 [index](#example-1--index-php)
:::

::: en-file
 [Output](#example-1--Output-txt)
:::
:::::::::::

**Complexité :** []{.fa-stars}

**Popularité :** []{.fa-stars}

**Exemples d'utilisation :** L'adaptateur est très répandu en PHP. On le
retrouve souvent dans des systèmes basés sur du code hérité, dans
lesquels l'adaptateur fait fonctionner du code hérité avec des classes
modernes.

**Identification :** L'adaptateur peut être identifié grâce à son
constructeur qui prend une instance d'un type abstrait différent ou
d'une interface différente. Lorsque l'une des méthodes de l'adaptateur
est appelée, il traduit les paramètres dans un format approprié et
redirige l'appel vers une ou plusieurs méthodes de l'objet emballé.
::::::::::::::

::: {#nav-tab .nav .nav-tabs .nav-justified role="tablist"}
[Exemple conceptuel](#example-0){#example-0-tab .nav-item .nav-link
.active toggle="tab" role="tab" aria-controls="example-0"
aria-selected="true"} [Analogie du monde
réel](#example-1){#example-1-tab .nav-item .nav-link toggle="tab"
role="tab" aria-controls="example-1" aria-selected="true"}
:::

::::: {#nav-tabContent .pattern-example-tab-content .tab-content}
::: {#example-0 .tab-pane .show .active role="tabpanel" aria-labelledby="example-0-tab" style="padding-top: 24px"}
## Exemple conceptuel {#example-0-title}

Dans cet exemple, nous allons voir la structure de l'**Adaptateur** et
répondre aux questions suivantes :

-   Que contiennent les classes ?
-   Quels rôles jouent-elles ?
-   Comment les éléments du patron sont-ils reliés ?

Après avoir étudié la structure du patron, vous pourrez plus facilement
comprendre l'exemple suivant qui est basé sur un cas d'utilisation réel
en PHP.

#### []{#example-0--index-php .anchor} **index.php:** Exemple conceptuel

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

#### []{#example-0--Output-txt .anchor} **Output.txt:** Résultat de l'exécution

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
## Analogie du monde réel {#example-1-title}

Le patron **Adaptateur** vous permet d'utiliser des classes externes ou
héritées même si elles ne sont pas vraiment compatibles avec votre code.
Par exemple, plutôt que de réécrire l'interface de notification de votre
application pour pouvoir prendre en charge des services externes comme
Slack, Facebook, SMS ou bien d'autres, vous pouvez créer un ensemble
d'emballeurs qui vont envoyer les appels faits dans votre application
vers une interface dans le format attendu par la classe externe.

#### []{#example-1--index-php .anchor} **index.php:** Exemple du monde réel

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

#### []{#example-1--Output-txt .anchor} **Output.txt:** Résultat de l'exécution

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
[Exemple conceptuel](#example-0){#example-0-tab-bottom .nav-item
.nav-link .active toggle="tab" role="tab" aria-controls="example-0"
aria-selected="true"} [Analogie du monde
réel](#example-1){#example-1-tab-bottom .nav-item .nav-link toggle="tab"
role="tab" aria-controls="example-1" aria-selected="true"}
:::

::: next
#### Suivant

[Pont en PHP []{.fa
.fa-arrow-right}](../../bridge/php/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Retour

[[]{.fa .fa-arrow-left} Singleton en
PHP](../../singleton/php/example.html){.btn .btn-default rel="prev"}
:::

## **Adaptateur** dans les autres langues

[![Adaptateur en
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Adaptateur en C#"){.prog-lang-link}
[![Adaptateur en
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Adaptateur en C++"){.prog-lang-link}
[![Adaptateur en
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Adaptateur en Go"){.prog-lang-link}
[![Adaptateur en
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Adaptateur en Java"){.prog-lang-link}
[![Adaptateur en
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Adaptateur en Python"){.prog-lang-link}
[![Adaptateur en
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Adaptateur en Ruby"){.prog-lang-link}
[![Adaptateur en
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Adaptateur en Rust"){.prog-lang-link}
[![Adaptateur en
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Adaptateur en Swift"){.prog-lang-link}
[![Adaptateur en
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Adaptateur en TypeScript"){.prog-lang-link}
:::::::::::::::::::::::::

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
:::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::
