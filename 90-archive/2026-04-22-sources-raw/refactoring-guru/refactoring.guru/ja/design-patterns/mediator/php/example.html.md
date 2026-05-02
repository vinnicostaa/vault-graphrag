:::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} /
[デザインパターン](../../../design-patterns.html){.type} /
[Mediator](../../mediator.html){.type} / [PHP](../../php.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Mediator](../../../../images/patterns/cards/mediator-mini1561.png?id=a7e43ee8e17e4474737b1fcb3201d7ba){srcset="/images/patterns/cards/mediator-mini-2x.png?id=d288d7c73f5581ae09701d61200ae09e 2x"}
:::

# **Mediator** を PHP で {#mediator-を-php-で .pattern-example-title-block-title}
::::

:::::::::::::: pattern-example-body
::: pattern-example-brief
**Mediator** は[、]{.chpule2}[
]{.chpuri2}振る舞いに関するデザインパターンの一つで[、]{.chpule2}[
]{.chpuri2}プログラムのコンポーネント間の通信を特別なメディエーター・オブジェクトを通して行うことで[、]{.chpule2}[
]{.chpuri2}結合を疎にします[。]{.chpule2}[ ]{.chpuri2}

Mediator により[、]{.chpule2}[
]{.chpuri2}個々のコンポーネントは[、]{.chpule2}[
]{.chpuri2}何十ものクラスへの依存がなくなるため[、]{.chpule2}[
]{.chpuri2}変更[、]{.chpule2}[ ]{.chpuri2}拡張[、]{.chpule2}[
]{.chpuri2}再利用が容易になります[。]{.chpule2}[ ]{.chpuri2}
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Mediator の詳細 ](../../mediator.html){.btn .btn-lg .btn-primary}
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

**使用例[：]{.chpule2}[ ]{.chpuri2}**純粋な Mediator
パターンの実装は[、]{.chpule2}[ ]{.chpuri2}Java や C#
といった[、]{.chpule2}[ ]{.chpuri2}GUI 目的の言語に比べて[、]{.chpule2}[
]{.chpuri2}PHP ではあまりみかけません[。]{.chpule2}[ ]{.chpuri2}PHP
アプリケーションには[、]{.chpule2}[
]{.chpuri2}十数個のコンポーネントがあるかもしれませんが[、]{.chpule2}[
]{.chpuri2}それが一つのセッション中で直接通信することは稀です[。]{.chpule2}[
]{.chpuri2}

しかしそれでも[、]{.chpule2}[ ]{.chpuri2}多くの PHP
フレームワークのイベント・ディスパッチャーや[、]{.chpule2}[
]{.chpuri2}MVC コントローラーの実装のように[、]{.chpule2}[
]{.chpuri2}Mediator
パターンの使い道がないわけではありません[。]{.chpule2}[ ]{.chpuri2}
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

この例は[、]{.chpule2}[ ]{.chpuri2}**Mediator**
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

namespace RefactoringGuru\Mediator\Conceptual;

/**
 * The Mediator interface declares a method used by components to notify the
 * mediator about various events. The Mediator may react to these events and
 * pass the execution to other components.
 */
interface Mediator
{
    public function notify(object $sender, string $event): void;
}

/**
 * Concrete Mediators implement cooperative behavior by coordinating several
 * components.
 */
class ConcreteMediator implements Mediator
{
    private $component1;

    private $component2;

    public function __construct(Component1 $c1, Component2 $c2)
    {
        $this-&gt;component1 = $c1;
        $this-&gt;component1-&gt;setMediator($this);
        $this-&gt;component2 = $c2;
        $this-&gt;component2-&gt;setMediator($this);
    }

    public function notify(object $sender, string $event): void
    {
        if ($event == &quot;A&quot;) {
            echo &quot;Mediator reacts on A and triggers following operations:\n&quot;;
            $this-&gt;component2-&gt;doC();
        }

        if ($event == &quot;D&quot;) {
            echo &quot;Mediator reacts on D and triggers following operations:\n&quot;;
            $this-&gt;component1-&gt;doB();
            $this-&gt;component2-&gt;doC();
        }
    }
}

/**
 * The Base Component provides the basic functionality of storing a mediator&#39;s
 * instance inside component objects.
 */
class BaseComponent
{
    protected $mediator;

    public function __construct(Mediator $mediator = null)
    {
        $this-&gt;mediator = $mediator;
    }

    public function setMediator(Mediator $mediator): void
    {
        $this-&gt;mediator = $mediator;
    }
}

/**
 * Concrete Components implement various functionality. They don&#39;t depend on
 * other components. They also don&#39;t depend on any concrete mediator classes.
 */
class Component1 extends BaseComponent
{
    public function doA(): void
    {
        echo &quot;Component 1 does A.\n&quot;;
        $this-&gt;mediator-&gt;notify($this, &quot;A&quot;);
    }

    public function doB(): void
    {
        echo &quot;Component 1 does B.\n&quot;;
        $this-&gt;mediator-&gt;notify($this, &quot;B&quot;);
    }
}

class Component2 extends BaseComponent
{
    public function doC(): void
    {
        echo &quot;Component 2 does C.\n&quot;;
        $this-&gt;mediator-&gt;notify($this, &quot;C&quot;);
    }

    public function doD(): void
    {
        echo &quot;Component 2 does D.\n&quot;;
        $this-&gt;mediator-&gt;notify($this, &quot;D&quot;);
    }
}

/**
 * The client code.
 */
$c1 = new Component1();
$c2 = new Component2();
$mediator = new ConcreteMediator($c1, $c2);

echo &quot;Client triggers operation A.\n&quot;;
$c1-&gt;doA();

echo &quot;\n&quot;;
echo &quot;Client triggers operation D.\n&quot;;
$c2-&gt;doD();</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** 実行結果

<figure class="code">
<pre class="code" lang="output"><code>Client triggers operation A.
Component 1 does A.
Mediator reacts on A and triggers following operations:
Component 2 does C.

Client triggers operation D.
Component 2 does D.
Mediator reacts on D and triggers following operations:
Component 1 does B.
Component 2 does C.</code></pre>
</figure>
:::

::: {#example-1 .tab-pane role="tabpanel" aria-labelledby="example-1-tab" style="padding-top: 24px"}
## 現実的な例 {#example-1-title}

この例では[、]{.chpule2}[ ]{.chpuri2}**Mediator**
パターンが[、]{.chpule2}[
]{.chpuri2}集中型イベント・ディスパッチャーによって[、]{.chpule2}[
]{.chpuri2}Observer パターンのアイディアを拡張しています[。]{.chpule2}[
]{.chpuri2}どんなオブジェクトでも[、]{.chpule2}[
]{.chpuri2}他のオブジェクト内で[、]{.chpule2}[
]{.chpuri2}クラスに依存することなく[、]{.chpule2}[
]{.chpuri2}イベントを追跡[、]{.chpule2}[
]{.chpuri2}トリガーできます[。]{.chpule2}[ ]{.chpuri2}

#### []{#example-1--index-php .anchor} **index.php:** 現実的な例

<figure class="code">
<pre class="code" lang="php"><code>&lt;?php

namespace RefactoringGuru\Mediator\RealWorld;

/**
 * The Event Dispatcher class acts as a Mediator and contains the subscription
 * and notification logic. While a classic Mediator often depends on concrete
 * component classes, this one is only tied to their abstract interfaces.
 *
 * We are able to achieve this level of indirection thanks to the way the
 * connections between components are established. The components themselves may
 * subscribe to specific events that they are interested in via the Mediator&#39;s
 * subscription interface.
 *
 * Note, we can&#39;t use the PHP&#39;s built-in Subject/Observer interfaces here
 * because we&#39;ll be stretching them too far from what they were designed for.
 */
class EventDispatcher
{
    /**
     * @var array
     */
    private $observers = [];

    public function __construct()
    {
        // The special event group for observers that want to listen to all
        // events.
        $this-&gt;observers[&quot;*&quot;] = [];
    }

    private function initEventGroup(string &amp;$event = &quot;*&quot;): void
    {
        if (!isset($this-&gt;observers[$event])) {
            $this-&gt;observers[$event] = [];
        }
    }

    private function getEventObservers(string $event = &quot;*&quot;): array
    {
        $this-&gt;initEventGroup($event);
        $group = $this-&gt;observers[$event];
        $all = $this-&gt;observers[&quot;*&quot;];

        return array_merge($group, $all);
    }

    public function attach(Observer $observer, string $event = &quot;*&quot;): void
    {
        $this-&gt;initEventGroup($event);

        $this-&gt;observers[$event][] = $observer;
    }

    public function detach(Observer $observer, string $event = &quot;*&quot;): void
    {
        foreach ($this-&gt;getEventObservers($event) as $key =&gt; $s) {
            if ($s === $observer) {
                unset($this-&gt;observers[$event][$key]);
            }
        }
    }

    public function trigger(string $event, object $emitter, $data = null): void
    {
        echo &quot;EventDispatcher: Broadcasting the &#39;$event&#39; event.\n&quot;;
        foreach ($this-&gt;getEventObservers($event) as $observer) {
            $observer-&gt;update($event, $emitter, $data);
        }
    }
}

/**
 * A simple helper function to provide global access to the event dispatcher.
 */
function events(): EventDispatcher
{
    static $eventDispatcher;
    if (!$eventDispatcher) {
        $eventDispatcher = new EventDispatcher();
    }

    return $eventDispatcher;
}

/**
 * The Observer interface defines how components receive the event
 * notifications.
 */
interface Observer
{
    public function update(string $event, object $emitter, $data = null);
}

/**
 * Unlike our Observer pattern example, this example makes the UserRepository
 * act as a regular component that doesn&#39;t have any special event-related
 * methods. Like any other component, this class relies on the EventDispatcher
 * to broadcast its events and listen for the other ones.
 *
 * @see \RefactoringGuru\Observer\RealWorld\UserRepository
 */
class UserRepository implements Observer
{
    /**
     * @var array List of application&#39;s users.
     */
    private $users = [];

    /**
     * Components can subscribe to events by themselves or by client code.
     */
    public function __construct()
    {
        events()-&gt;attach($this, &quot;users:deleted&quot;);
    }

    /**
     * Components can decide whether they&#39;d like to process an event using its
     * name, emitter or any contextual data passed along with the event.
     */
    public function update(string $event, object $emitter, $data = null): void
    {
        switch ($event) {
            case &quot;users:deleted&quot;:
                if ($emitter === $this) {
                    return;
                }
                $this-&gt;deleteUser($data, true);
                break;
        }
    }

    // These methods represent the business logic of the class.

    public function initialize(string $filename): void
    {
        echo &quot;UserRepository: Loading user records from a file.\n&quot;;
        // ...
        events()-&gt;trigger(&quot;users:init&quot;, $this, $filename);
    }

    public function createUser(array $data, bool $silent = false): User
    {
        echo &quot;UserRepository: Creating a user.\n&quot;;

        $user = new User();
        $user-&gt;update($data);

        $id = bin2hex(openssl_random_pseudo_bytes(16));
        $user-&gt;update([&quot;id&quot; =&gt; $id]);
        $this-&gt;users[$id] = $user;

        if (!$silent) {
            events()-&gt;trigger(&quot;users:created&quot;, $this, $user);
        }

        return $user;
    }

    public function updateUser(User $user, array $data, bool $silent = false): ?User
    {
        echo &quot;UserRepository: Updating a user.\n&quot;;

        $id = $user-&gt;attributes[&quot;id&quot;];
        if (!isset($this-&gt;users[$id])) {
            return null;
        }

        $user = $this-&gt;users[$id];
        $user-&gt;update($data);

        if (!$silent) {
            events()-&gt;trigger(&quot;users:updated&quot;, $this, $user);
        }

        return $user;
    }

    public function deleteUser(User $user, bool $silent = false): void
    {
        echo &quot;UserRepository: Deleting a user.\n&quot;;

        $id = $user-&gt;attributes[&quot;id&quot;];
        if (!isset($this-&gt;users[$id])) {
            return;
        }

        unset($this-&gt;users[$id]);

        if (!$silent) {
            events()-&gt;trigger(&quot;users:deleted&quot;, $this, $user);
        }
    }
}

/**
 * Let&#39;s keep the User class trivial since it&#39;s not the focus of our example.
 */
class User
{
    public $attributes = [];

    public function update($data): void
    {
        $this-&gt;attributes = array_merge($this-&gt;attributes, $data);
    }

    /**
     * All objects can trigger events.
     */
    public function delete(): void
    {
        echo &quot;User: I can now delete myself without worrying about the repository.\n&quot;;
        events()-&gt;trigger(&quot;users:deleted&quot;, $this, $this);
    }
}

/**
 * This Concrete Component logs any events it&#39;s subscribed to.
 */
class Logger implements Observer
{
    private $filename;

    public function __construct($filename)
    {
        $this-&gt;filename = $filename;
        if (file_exists($this-&gt;filename)) {
            unlink($this-&gt;filename);
        }
    }

    public function update(string $event, object $emitter, $data = null)
    {
        $entry = date(&quot;Y-m-d H:i:s&quot;) . &quot;: &#39;$event&#39; with data &#39;&quot; . json_encode($data) . &quot;&#39;\n&quot;;
        file_put_contents($this-&gt;filename, $entry, FILE_APPEND);

        echo &quot;Logger: I&#39;ve written &#39;$event&#39; entry to the log.\n&quot;;
    }
}

/**
 * This Concrete Component sends initial instructions to new users. The client
 * is responsible for attaching this component to a proper user creation event.
 */
class OnboardingNotification implements Observer
{
    private $adminEmail;

    public function __construct(string $adminEmail)
    {
        $this-&gt;adminEmail = $adminEmail;
    }

    public function update(string $event, object $emitter, $data = null): void
    {
        // mail($this-&gt;adminEmail,
        //     &quot;Onboarding required&quot;,
        //     &quot;We have a new user. Here&#39;s his info: &quot; .json_encode($data));

        echo &quot;OnboardingNotification: The notification has been emailed!\n&quot;;
    }
}

/**
 * The client code.
 */

$repository = new UserRepository();
events()-&gt;attach($repository, &quot;facebook:update&quot;);

$logger = new Logger(__DIR__ . &quot;/log.txt&quot;);
events()-&gt;attach($logger, &quot;*&quot;);

$onboarding = new OnboardingNotification(&quot;1@example.com&quot;);
events()-&gt;attach($onboarding, &quot;users:created&quot;);

// ...

$repository-&gt;initialize(__DIR__ . &quot;users.csv&quot;);

// ...

$user = $repository-&gt;createUser([
    &quot;name&quot; =&gt; &quot;John Smith&quot;,
    &quot;email&quot; =&gt; &quot;john99@example.com&quot;,
]);

// ...

$user-&gt;delete();</code></pre>
</figure>

#### []{#example-1--Output-txt .anchor} **Output.txt:** 実行結果

<figure class="code">
<pre class="code" lang="output"><code>UserRepository: Loading user records from a file.
EventDispatcher: Broadcasting the &#39;users:init&#39; event.
Logger: I&#39;ve written &#39;users:init&#39; entry to the log.
UserRepository: Creating a user.
EventDispatcher: Broadcasting the &#39;users:created&#39; event.
OnboardingNotification: The notification has been emailed!
Logger: I&#39;ve written &#39;users:created&#39; entry to the log.
User: I can now delete myself without worrying about the repository.
EventDispatcher: Broadcasting the &#39;users:deleted&#39; event.
UserRepository: Deleting a user.
Logger: I&#39;ve written &#39;users:deleted&#39; entry to the log.</code></pre>
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

[Memento を PHP で []{.fa
.fa-arrow-right}](../../memento/php/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### 戻る

[[]{.fa .fa-arrow-left} Iterator を PHP
で](../../iterator/php/example.html){.btn .btn-default rel="prev"}
:::

## 他言語での **Mediator**

[![Mediator を C#
で](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Mediator を C# で"){.prog-lang-link}
[![Mediator を C++
で](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Mediator を C++ で"){.prog-lang-link}
[![Mediator を Go
で](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Mediator を Go で"){.prog-lang-link}
[![Mediator を Java
で](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Mediator を Java で"){.prog-lang-link}
[![Mediator を Python
で](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Mediator を Python で"){.prog-lang-link}
[![Mediator を Ruby
で](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Mediator を Ruby で"){.prog-lang-link}
[![Mediator を Rust
で](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Mediator を Rust で"){.prog-lang-link}
[![Mediator を Swift
で](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Mediator を Swift で"){.prog-lang-link}
[![Mediator を TypeScript
で](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Mediator を TypeScript で"){.prog-lang-link}
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
