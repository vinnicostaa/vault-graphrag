:::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Патерни
проектування](../../../design-patterns.html){.type} /
[Посередник](../../mediator.html){.type} / [PHP](../../php.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Посередник](../../../../images/patterns/cards/mediator-mini1561.png?id=a7e43ee8e17e4474737b1fcb3201d7ba){srcset="/images/patterns/cards/mediator-mini-2x.png?id=d288d7c73f5581ae09701d61200ae09e 2x"}
:::

# **Посередник** на PHP {#посередник-на-php .pattern-example-title-block-title}
::::

:::::::::::::: pattern-example-body
::: pattern-example-brief
**Посередник** --- це поведінковий патерн, який спрощує комунікацію між
компонентами системи.

Посередник прибирає прямі зв'язки між окремими компонентами, змушуючи їх
спілкуватися один з одним через себе.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Детальніше про Посередник ](../../mediator.html){.btn .btn-lg
.btn-primary}
:::

::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
Навігація
:::

::: en-intro
 [Інтро](#)
:::

::: en-intro
 [Концептуальний приклад](#example-0)
:::

::: en-file
 [index](#example-0--index-php)
:::

::: en-file
 [Output](#example-0--Output-txt)
:::

::: en-intro
 [Життєвий приклад](#example-1)
:::

::: en-file
 [index](#example-1--index-php)
:::

::: en-file
 [Output](#example-1--Output-txt)
:::
:::::::::::

**Складність:** []{.fa-stars}

**Популярність:** []{.fa-stars}

**Застосування:** Посередник не настільки актуальний у PHP, як в інших
мовах типу Java чи C#. У PHP-програмі різні компоненти не часто
спілкуються один з одним в межах однієї сесії скрипта. Тому вони частіше
за все і так бувають достатньо розділеними.

Проте прикладами патерна можуть служити EventDispatcher-и багатьох
фреймворків, а також деякі реалізації контролерів в MVC фреймворк.
::::::::::::::

::: {#nav-tab .nav .nav-tabs .nav-justified role="tablist"}
[Концептуальний приклад](#example-0){#example-0-tab .nav-item .nav-link
.active toggle="tab" role="tab" aria-controls="example-0"
aria-selected="true"} [Життєвий приклад](#example-1){#example-1-tab
.nav-item .nav-link toggle="tab" role="tab" aria-controls="example-1"
aria-selected="true"}
:::

::::: {#nav-tabContent .pattern-example-tab-content .tab-content}
::: {#example-0 .tab-pane .show .active role="tabpanel" aria-labelledby="example-0-tab" style="padding-top: 24px"}
## Концептуальний приклад {#example-0-title}

Цей приклад показує структуру патерна **Посередник**, а саме --- з яких
класів він складається, які ролі ці класи виконують і як вони
взаємодіють один з одним.

Після ознайомлення зі структурою, вам буде легше сприймати наступний
приклад, що розглядає реальний випадок використання патерна в світі PHP.

#### []{#example-0--index-php .anchor} **index.php:** Приклад структури патерна

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

#### []{#example-0--Output-txt .anchor} **Output.txt:** Результат виконання

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
## Життєвий приклад {#example-1-title}

#### []{#example-1--index-php .anchor} **index.php:** Приклад з реального світу

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

#### []{#example-1--Output-txt .anchor} **Output.txt:** Результат виконання

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
[Концептуальний приклад](#example-0){#example-0-tab-bottom .nav-item
.nav-link .active toggle="tab" role="tab" aria-controls="example-0"
aria-selected="true"} [Життєвий
приклад](#example-1){#example-1-tab-bottom .nav-item .nav-link
toggle="tab" role="tab" aria-controls="example-1" aria-selected="true"}
:::

::: next
#### Читаємо далі

[Знімок на PHP []{.fa
.fa-arrow-right}](../../memento/php/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Повернутись назад

[[]{.fa .fa-arrow-left} Ітератор на
PHP](../../iterator/php/example.html){.btn .btn-default rel="prev"}
:::

## **Посередник** іншими мовами програмування

[![Посередник на
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Посередник на C#"){.prog-lang-link}
[![Посередник на
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Посередник на C++"){.prog-lang-link}
[![Посередник на
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Посередник на Go"){.prog-lang-link}
[![Посередник на
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Посередник на Java"){.prog-lang-link}
[![Посередник на
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Посередник на Python"){.prog-lang-link}
[![Посередник на
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Посередник на Ruby"){.prog-lang-link}
[![Посередник на
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Посередник на Rust"){.prog-lang-link}
[![Посередник на
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Посередник на Swift"){.prog-lang-link}
[![Посередник на
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Посередник на TypeScript"){.prog-lang-link}
:::::::::::::::::::::::::

::::::: {.prom .banner-sidebar .banner .banner-striped .banner-examples .banner-removable .banner-removable-patterns data-id="DP: Examples-IDE" creative-id="examples-sidebar-uk" position="sidebar"}
:::::: {.banner-inner style="font-size: 14px; line-height: 21px;"}
::: banner-examples-img
[![](../../../../images/patterns/banners/examples-ide91ca.png?id=3115b4b548fb96b75974e2de8f4f49bc){width="215"
height="158" loading="lazy"
srcset="/images/patterns/banners/examples-ide-2x.png?id=93c007a6d157b97c28eb213f59d716ae 2x"}](../../book.html)
:::

::: banner-examples-fade
[ Архів з прикладами](../../book.html){.btn .btn-secondary .btn-block}
:::

::: {.banner-examples-text style="margin-top: 1rem;"}
Придбай книгу **Занурення в Патерни** та отримай архів з десятками
детальних прикладів, які можна відкривати прямо в IDE.

[ Дізнатися більше...](../../book.html){.btn .btn-secondary .btn-block}
:::
::::::
:::::::
:::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::
