:::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../index.html){.home} / [Design
Patterns](../../../design-patterns.html){.type} /
[Observer](../../observer.html){.type} / [PHP](../../php.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Observer](../../../images/patterns/cards/observer-minifa2e.png?id=fd2081ab1cff29c60b499bcf6a62786a){srcset="/images/patterns/cards/observer-mini-2x.png?id=f205b0655572ac8e4636691c0e0debfd 2x"}
:::

# **Observer** in PHP {#observer-in-php .pattern-example-title-block-title}
::::

:::::::::::::: pattern-example-body
::: pattern-example-brief
**Observer** is a behavioral design pattern that allows some objects to
notify other objects about changes in their state.

The Observer pattern provides a way to subscribe and unsubscribe to and
from these events for any object that implements a subscriber interface.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Learn more about Observer ](../../observer.html){.btn .btn-lg
.btn-primary}
:::

::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
Navigation
:::

::: en-intro
 [Intro](#)
:::

::: en-intro
 [Conceptual Example](#example-0)
:::

::: en-file
 [index](#example-0--index-php)
:::

::: en-file
 [Output](#example-0--Output-txt)
:::

::: en-intro
 [Real World Example](#example-1)
:::

::: en-file
 [index](#example-1--index-php)
:::

::: en-file
 [Output](#example-1--Output-txt)
:::
:::::::::::

**Complexity:** []{.fa-stars}

**Popularity:** []{.fa-stars}

**Usage examples:** PHP has several built-in interfaces
([SplSubject](http://php.net/manual/en/class.splsubject.php),
[SplObserver](http://php.net/manual/en/class.splobserver.php)) that can
be used to make your implementations of the Observer pattern compatible
with the rest of the PHP code.

**Identification:** The pattern can be recognized by subscription
methods, that store objects in a list and by calls to the update method
issued to objects in that list.
::::::::::::::

::: {#nav-tab .nav .nav-tabs .nav-justified role="tablist"}
[Conceptual Example](#example-0){#example-0-tab .nav-item .nav-link
.active toggle="tab" role="tab" aria-controls="example-0"
aria-selected="true"} [Real World Example](#example-1){#example-1-tab
.nav-item .nav-link toggle="tab" role="tab" aria-controls="example-1"
aria-selected="true"}
:::

::::: {#nav-tabContent .pattern-example-tab-content .tab-content}
::: {#example-0 .tab-pane .show .active role="tabpanel" aria-labelledby="example-0-tab" style="padding-top: 24px"}
## Conceptual Example {#example-0-title}

This example illustrates the structure of the **Observer** design
pattern and focuses on the following questions:

-   What classes does it consist of?
-   What roles do these classes play?
-   In what way the elements of the pattern are related?

After learning about the pattern's structure it'll be easier for you to
grasp the following example, based on a real-world PHP use case.

#### []{#example-0--index-php .anchor} **index.php:** Conceptual example

<figure class="code">
<pre class="code" lang="php"><code>&lt;?php

namespace RefactoringGuru\Observer\Conceptual;

/**
 * PHP has a couple of built-in interfaces related to the Observer pattern.
 *
 * Here&#39;s what the Subject interface looks like:
 *
 * @link http://php.net/manual/en/class.splsubject.php
 *
 *     interface SplSubject
 *     {
 *         // Attach an observer to the subject.
 *         public function attach(SplObserver $observer);
 *
 *         // Detach an observer from the subject.
 *         public function detach(SplObserver $observer);
 *
 *         // Notify all observers about an event.
 *         public function notify();
 *     }
 *
 * There&#39;s also a built-in interface for Observers:
 *
 * @link http://php.net/manual/en/class.splobserver.php
 *
 *     interface SplObserver
 *     {
 *         public function update(SplSubject $subject);
 *     }
 */

/**
 * The Subject owns some important state and notifies observers when the state
 * changes.
 */
class Subject implements \SplSubject
{
    /**
     * @var int For the sake of simplicity, the Subject&#39;s state, essential to
     * all subscribers, is stored in this variable.
     */
    public $state;

    /**
     * @var \SplObjectStorage List of subscribers. In real life, the list of
     * subscribers can be stored more comprehensively (categorized by event
     * type, etc.).
     */
    private $observers;

    public function __construct()
    {
        $this-&gt;observers = new \SplObjectStorage();
    }

    /**
     * The subscription management methods.
     */
    public function attach(\SplObserver $observer): void
    {
        echo &quot;Subject: Attached an observer.\n&quot;;
        $this-&gt;observers-&gt;attach($observer);
    }

    public function detach(\SplObserver $observer): void
    {
        $this-&gt;observers-&gt;detach($observer);
        echo &quot;Subject: Detached an observer.\n&quot;;
    }

    /**
     * Trigger an update in each subscriber.
     */
    public function notify(): void
    {
        echo &quot;Subject: Notifying observers...\n&quot;;
        foreach ($this-&gt;observers as $observer) {
            $observer-&gt;update($this);
        }
    }

    /**
     * Usually, the subscription logic is only a fraction of what a Subject can
     * really do. Subjects commonly hold some important business logic, that
     * triggers a notification method whenever something important is about to
     * happen (or after it).
     */
    public function someBusinessLogic(): void
    {
        echo &quot;\nSubject: I&#39;m doing something important.\n&quot;;
        $this-&gt;state = rand(0, 10);

        echo &quot;Subject: My state has just changed to: {$this-&gt;state}\n&quot;;
        $this-&gt;notify();
    }
}

/**
 * Concrete Observers react to the updates issued by the Subject they had been
 * attached to.
 */
class ConcreteObserverA implements \SplObserver
{
    public function update(\SplSubject $subject): void
    {
        if ($subject-&gt;state &lt; 3) {
            echo &quot;ConcreteObserverA: Reacted to the event.\n&quot;;
        }
    }
}

class ConcreteObserverB implements \SplObserver
{
    public function update(\SplSubject $subject): void
    {
        if ($subject-&gt;state == 0 || $subject-&gt;state &gt;= 2) {
            echo &quot;ConcreteObserverB: Reacted to the event.\n&quot;;
        }
    }
}

/**
 * The client code.
 */

$subject = new Subject();

$o1 = new ConcreteObserverA();
$subject-&gt;attach($o1);

$o2 = new ConcreteObserverB();
$subject-&gt;attach($o2);

$subject-&gt;someBusinessLogic();
$subject-&gt;someBusinessLogic();

$subject-&gt;detach($o2);

$subject-&gt;someBusinessLogic();</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** Execution result

<figure class="code">
<pre class="code" lang="output"><code>Subject: Attached an observer.
Subject: Attached an observer.

Subject: I&#39;m doing something important.
Subject: My state has just changed to: 2
Subject: Notifying observers...
ConcreteObserverA: Reacted to the event.
ConcreteObserverB: Reacted to the event.

Subject: I&#39;m doing something important.
Subject: My state has just changed to: 4
Subject: Notifying observers...
ConcreteObserverB: Reacted to the event.
Subject: Detached an observer.

Subject: I&#39;m doing something important.
Subject: My state has just changed to: 1
Subject: Notifying observers...
ConcreteObserverA: Reacted to the event.</code></pre>
</figure>
:::

::: {#example-1 .tab-pane role="tabpanel" aria-labelledby="example-1-tab" style="padding-top: 24px"}
## Real World Example {#example-1-title}

In this example the **Observer** pattern allows various objects to
observe events that are happening inside a user repository of an app.

The repository emits various types of events and allows observers to
listen to all of them, as well as only individual ones.

#### []{#example-1--index-php .anchor} **index.php:** Real world example

<figure class="code">
<pre class="code" lang="php"><code>&lt;?php

namespace RefactoringGuru\Observer\RealWorld;

/**
 * The UserRepository represents a Subject. Various objects are interested in
 * tracking its internal state, whether it&#39;s adding a new user or removing one.
 */
class UserRepository implements \SplSubject
{
    /**
     * @var array The list of users.
     */
    private $users = [];

    // Here goes the actual Observer management infrastructure. Note that it&#39;s
    // not everything that our class is responsible for. Its primary business
    // logic is listed below these methods.

    /**
     * @var array
     */
    private $observers = [];

    public function __construct()
    {
        // A special event group for observers that want to listen to all
        // events.
        $this-&gt;observers[&quot;*&quot;] = [];
    }

    private function initEventGroup(string $event = &quot;*&quot;): void
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

    public function attach(\SplObserver $observer, string $event = &quot;*&quot;): void
    {
        $this-&gt;initEventGroup($event);

        $this-&gt;observers[$event][] = $observer;
    }

    public function detach(\SplObserver $observer, string $event = &quot;*&quot;): void
    {
        foreach ($this-&gt;getEventObservers($event) as $key =&gt; $s) {
            if ($s === $observer) {
                unset($this-&gt;observers[$event][$key]);
            }
        }
    }

    public function notify(string $event = &quot;*&quot;, $data = null): void
    {
        echo &quot;UserRepository: Broadcasting the &#39;$event&#39; event.\n&quot;;
        foreach ($this-&gt;getEventObservers($event) as $observer) {
            $observer-&gt;update($this, $event, $data);
        }
    }

    // Here are the methods representing the business logic of the class.

    public function initialize($filename): void
    {
        echo &quot;UserRepository: Loading user records from a file.\n&quot;;
        // ...
        $this-&gt;notify(&quot;users:init&quot;, $filename);
    }

    public function createUser(array $data): User
    {
        echo &quot;UserRepository: Creating a user.\n&quot;;

        $user = new User();
        $user-&gt;update($data);

        $id = bin2hex(openssl_random_pseudo_bytes(16));
        $user-&gt;update([&quot;id&quot; =&gt; $id]);
        $this-&gt;users[$id] = $user;

        $this-&gt;notify(&quot;users:created&quot;, $user);

        return $user;
    }

    public function updateUser(User $user, array $data): User
    {
        echo &quot;UserRepository: Updating a user.\n&quot;;

        $id = $user-&gt;attributes[&quot;id&quot;];
        if (!isset($this-&gt;users[$id])) {
            return null;
        }

        $user = $this-&gt;users[$id];
        $user-&gt;update($data);

        $this-&gt;notify(&quot;users:updated&quot;, $user);

        return $user;
    }

    public function deleteUser(User $user): void
    {
        echo &quot;UserRepository: Deleting a user.\n&quot;;

        $id = $user-&gt;attributes[&quot;id&quot;];
        if (!isset($this-&gt;users[$id])) {
            return;
        }

        unset($this-&gt;users[$id]);

        $this-&gt;notify(&quot;users:deleted&quot;, $user);
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
}

/**
 * This Concrete Component logs any events it&#39;s subscribed to.
 */
class Logger implements \SplObserver
{
    private $filename;

    public function __construct($filename)
    {
        $this-&gt;filename = $filename;
        if (file_exists($this-&gt;filename)) {
            unlink($this-&gt;filename);
        }
    }

    public function update(\SplSubject $repository, string $event = null, $data = null): void
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
class OnboardingNotification implements \SplObserver
{
    private $adminEmail;

    public function __construct($adminEmail)
    {
        $this-&gt;adminEmail = $adminEmail;
    }

    public function update(\SplSubject $repository, string $event = null, $data = null): void
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
$repository-&gt;attach(new Logger(__DIR__ . &quot;/log.txt&quot;), &quot;*&quot;);
$repository-&gt;attach(new OnboardingNotification(&quot;1@example.com&quot;), &quot;users:created&quot;);

$repository-&gt;initialize(__DIR__ . &quot;/users.csv&quot;);

// ...

$user = $repository-&gt;createUser([
    &quot;name&quot; =&gt; &quot;John Smith&quot;,
    &quot;email&quot; =&gt; &quot;john99@example.com&quot;,
]);

// ...

$repository-&gt;deleteUser($user);</code></pre>
</figure>

#### []{#example-1--Output-txt .anchor} **Output.txt:** Execution result

<figure class="code">
<pre class="code" lang="output"><code>UserRepository: Loading user records from a file.
UserRepository: Broadcasting the &#39;users:init&#39; event.
Logger: I&#39;ve written &#39;users:init&#39; entry to the log.
UserRepository: Creating a user.
UserRepository: Broadcasting the &#39;users:created&#39; event.
OnboardingNotification: The notification has been emailed!
Logger: I&#39;ve written &#39;users:created&#39; entry to the log.
UserRepository: Deleting a user.
UserRepository: Broadcasting the &#39;users:deleted&#39; event.
Logger: I&#39;ve written &#39;users:deleted&#39; entry to the log.</code></pre>
</figure>
:::
:::::

::: {#nav-tab .nav .nav-tabs .nav-tabs-bottom .nav-justified role="tablist"}
[Conceptual Example](#example-0){#example-0-tab-bottom .nav-item
.nav-link .active toggle="tab" role="tab" aria-controls="example-0"
aria-selected="true"} [Real World
Example](#example-1){#example-1-tab-bottom .nav-item .nav-link
toggle="tab" role="tab" aria-controls="example-1" aria-selected="true"}
:::

::: next
#### Read next

[State in PHP []{.fa
.fa-arrow-right}](../../state/php/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Return

[[]{.fa .fa-arrow-left} Memento in
PHP](../../memento/php/example.html){.btn .btn-default rel="prev"}
:::

## **Observer** in Other Languages

[![Observer in
C#](../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Observer in C#"){.prog-lang-link}
[![Observer in
C++](../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Observer in C++"){.prog-lang-link}
[![Observer in
Go](../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Observer in Go"){.prog-lang-link}
[![Observer in
Java](../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Observer in Java"){.prog-lang-link}
[![Observer in
Python](../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Observer in Python"){.prog-lang-link}
[![Observer in
Ruby](../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Observer in Ruby"){.prog-lang-link}
[![Observer in
Rust](../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Observer in Rust"){.prog-lang-link}
[![Observer in
Swift](../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Observer in Swift"){.prog-lang-link}
[![Observer in
TypeScript](../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Observer in TypeScript"){.prog-lang-link}
:::::::::::::::::::::::::

::::::: {.prom .banner-sidebar .banner .banner-striped .banner-examples .banner-removable .banner-removable-patterns data-id="DP: Examples-IDE" creative-id="examples-sidebar-en" position="sidebar"}
:::::: {.banner-inner style="font-size: 14px; line-height: 21px;"}
::: banner-examples-img
[![](../../../images/patterns/banners/examples-ide91ca.png?id=3115b4b548fb96b75974e2de8f4f49bc){width="215"
height="158" loading="lazy"
srcset="/images/patterns/banners/examples-ide-2x.png?id=93c007a6d157b97c28eb213f59d716ae 2x"}](../../book.html)
:::

::: banner-examples-fade
[ Archive with examples](../../book.html){.btn .btn-secondary
.btn-block}
:::

::: {.banner-examples-text style="margin-top: 1rem;"}
Buy the eBook **Dive Into Design Patterns** and get the access to
archive with dozens of detailed examples that can be opened right in
your IDE.

[ Learn more...](../../book.html){.btn .btn-secondary .btn-block}
:::
::::::
:::::::
:::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::
