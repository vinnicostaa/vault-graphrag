:::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Паттерны
проектирования](../../../design-patterns.html){.type} / [Шаблонный
метод](../../template-method.html){.type} / [PHP](../../php.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Шаблонный
метод](../../../../images/patterns/cards/template-method-mini56e3.png?id=9f200248d88026d8e79d0f3dae411ab4){srcset="/images/patterns/cards/template-method-mini-2x.png?id=178bf56e39b3a1f548dd636076209c98 2x"}
:::

# **Шаблонный метод** на PHP {#шаблонный-метод-на-php .pattern-example-title-block-title}
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
 [index](#example-0--index-php)
:::

::: en-file
 [Output](#example-0--Output-txt)
:::

::: en-intro
 [Пример из реальной жизни](#example-1)
:::

::: en-file
 [index](#example-1--index-php)
:::

::: en-file
 [Output](#example-1--Output-txt)
:::
:::::::::::

**Сложность:** []{.fa-stars}

**Популярность:** []{.fa-stars}

**Применимость:** Шаблонные методы можно встретить во многих
PHP-фреймворках. Разработчики создают такие методы, чтобы позволить
клиентам легко и быстро расширять стандартный код при помощи
наследования.

**Признаки применения паттерна:** Класс заставляет своих потомков
реализовать методы-шаги, но самостоятельно реализует структуру
алгоритма.
::::::::::::::

::: {#nav-tab .nav .nav-tabs .nav-justified role="tablist"}
[Концептуальный пример](#example-0){#example-0-tab .nav-item .nav-link
.active toggle="tab" role="tab" aria-controls="example-0"
aria-selected="true"} [Пример из реальной
жизни](#example-1){#example-1-tab .nav-item .nav-link toggle="tab"
role="tab" aria-controls="example-1" aria-selected="true"}
:::

::::: {#nav-tabContent .pattern-example-tab-content .tab-content}
::: {#example-0 .tab-pane .show .active role="tabpanel" aria-labelledby="example-0-tab" style="padding-top: 24px"}
## Концептуальный пример {#example-0-title}

Этот пример показывает структуру паттерна **Шаблонный метод**, а
именно --- из каких классов он состоит, какие роли эти классы выполняют
и как они взаимодействуют друг с другом.

После ознакомления со структурой, вам будет легче воспринимать второй
пример, который рассматривает реальный случай использования паттерна в
мире PHP.

#### []{#example-0--index-php .anchor} **index.php:** Пример структуры паттерна

<figure class="code">
<pre class="code" lang="php"><code>&lt;?php

namespace RefactoringGuru\TemplateMethod\Conceptual;

/**
 * Абстрактный Класс определяет шаблонный метод, содержащий скелет некоторого
 * алгоритма, состоящего из вызовов (обычно) абстрактных примитивных операций.
 *
 * Конкретные подклассы должны реализовать эти операции, но оставить сам
 * шаблонный метод без изменений.
 */
abstract class AbstractClass
{
    /**
     * Шаблонный метод определяет скелет алгоритма.
     */
    final public function templateMethod(): void
    {
        $this-&gt;baseOperation1();
        $this-&gt;requiredOperations1();
        $this-&gt;baseOperation2();
        $this-&gt;hook1();
        $this-&gt;requiredOperation2();
        $this-&gt;baseOperation3();
        $this-&gt;hook2();
    }

    /**
     * Эти операции уже имеют реализации.
     */
    protected function baseOperation1(): void
    {
        echo &quot;AbstractClass says: I am doing the bulk of the work\n&quot;;
    }

    protected function baseOperation2(): void
    {
        echo &quot;AbstractClass says: But I let subclasses override some operations\n&quot;;
    }

    protected function baseOperation3(): void
    {
        echo &quot;AbstractClass says: But I am doing the bulk of the work anyway\n&quot;;
    }

    /**
     * А эти операции должны быть реализованы в подклассах.
     */
    abstract protected function requiredOperations1(): void;

    abstract protected function requiredOperation2(): void;

    /**
     * Это «хуки». Подклассы могут переопределять их, но это не обязательно,
     * поскольку у хуков уже есть стандартная (но пустая) реализация. Хуки
     * предоставляют дополнительные точки расширения в некоторых критических
     * местах алгоритма.
     */
    protected function hook1(): void
    {
    }

    protected function hook2(): void
    {
    }
}

/**
 * Конкретные классы должны реализовать все абстрактные операции базового
 * класса. Они также могут переопределить некоторые операции с реализацией по
 * умолчанию.
 */
class ConcreteClass1 extends AbstractClass
{
    protected function requiredOperations1(): void
    {
        echo &quot;ConcreteClass1 says: Implemented Operation1\n&quot;;
    }

    protected function requiredOperation2(): void
    {
        echo &quot;ConcreteClass1 says: Implemented Operation2\n&quot;;
    }
}

/**
 * Обычно конкретные классы переопределяют только часть операций базового
 * класса.
 */
class ConcreteClass2 extends AbstractClass
{
    protected function requiredOperations1(): void
    {
        echo &quot;ConcreteClass2 says: Implemented Operation1\n&quot;;
    }

    protected function requiredOperation2(): void
    {
        echo &quot;ConcreteClass2 says: Implemented Operation2\n&quot;;
    }

    protected function hook1(): void
    {
        echo &quot;ConcreteClass2 says: Overridden Hook1\n&quot;;
    }
}

/**
 * Клиентский код вызывает шаблонный метод для выполнения алгоритма. Клиентский
 * код не должен знать конкретный класс объекта, с которым работает, при
 * условии, что он работает с объектами через интерфейс их базового класса.
 */
function clientCode(AbstractClass $class)
{
    // ...
    $class-&gt;templateMethod();
    // ...
}

echo &quot;Same client code can work with different subclasses:\n&quot;;
clientCode(new ConcreteClass1());
echo &quot;\n&quot;;

echo &quot;Same client code can work with different subclasses:\n&quot;;
clientCode(new ConcreteClass2());</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** Результат выполнения

<figure class="code">
<pre class="code" lang="output"><code>Same client code can work with different subclasses:
AbstractClass says: I am doing bulk of the work
ConcreteClass1 says: Implemented Operation1
AbstractClass says: But I let subclasses to override some operations
ConcreteClass1 says: Implemented Operation2
AbstractClass says: But I am doing bulk of the work anyway

Same client code can work with different subclasses:
AbstractClass says: I am doing bulk of the work
ConcreteClass2 says: Implemented Operation1
AbstractClass says: But I let subclasses to override some operations
ConcreteClass2 says: Overridden Hook1
ConcreteClass2 says: Implemented Operation2
AbstractClass says: But I am doing bulk of the work anyway</code></pre>
</figure>
:::

::: {#example-1 .tab-pane role="tabpanel" aria-labelledby="example-1-tab" style="padding-top: 24px"}
## Пример из реальной жизни {#example-1-title}

В этом примере **Шаблонный метод** определяет общую схему алгоритма
отправки сообщений в социальных сетях. Каждый подкласс представляет
отдельную социальную сеть и реализует все шаги по-разному, но повторно
использует базовый алгоритм.

#### []{#example-1--index-php .anchor} **index.php:** Пример из реальной жизни

<figure class="code">
<pre class="code" lang="php"><code>&lt;?php

namespace RefactoringGuru\TemplateMethod\RealWorld;

/**
 * Абстрактный Класс определяет метод шаблона и объявляет все его шаги.
 */
abstract class SocialNetwork
{
    protected $username;

    protected $password;

    public function __construct(string $username, string $password)
    {
        $this-&gt;username = $username;
        $this-&gt;password = $password;
    }

    /**
     * Фактический метод шаблона вызывает абстрактные шаги в определённом
     * порядке. Подкласс может реализовать все шаги, позволяя этому методу
     * реально публиковать что-то в социальной сети.
     */
    public function post(string $message): bool
    {
        // Проверка подлинности перед публикацией. Каждая сеть использует свой
        // метод авторизации.
        if ($this-&gt;logIn($this-&gt;username, $this-&gt;password)) {
            // Отправляем почтовые данные. Все сети имеют разные API.
            $result = $this-&gt;sendData($message);
            // ...
            $this-&gt;logOut();

            return $result;
        }

        return false;
    }

    /**
     * Шаги объявлены абстрактными, чтобы заставить подклассы реализовать их
     * полностью.
     */
    abstract public function logIn(string $userName, string $password): bool;

    abstract public function sendData(string $message): bool;

    abstract public function logOut(): void;
}

/**
 * Этот Конкретный Класс реализует API Facebook (ладно, он пытается).
 */
class Facebook extends SocialNetwork
{
    public function logIn(string $userName, string $password): bool
    {
        echo &quot;\nChecking user&#39;s credentials...\n&quot;;
        echo &quot;Name: &quot; . $this-&gt;username . &quot;\n&quot;;
        echo &quot;Password: &quot; . str_repeat(&quot;*&quot;, strlen($this-&gt;password)) . &quot;\n&quot;;

        simulateNetworkLatency();

        echo &quot;\n\nFacebook: &#39;&quot; . $this-&gt;username . &quot;&#39; has logged in successfully.\n&quot;;

        return true;
    }

    public function sendData(string $message): bool
    {
        echo &quot;Facebook: &#39;&quot; . $this-&gt;username . &quot;&#39; has posted &#39;&quot; . $message . &quot;&#39;.\n&quot;;

        return true;
    }

    public function logOut(): void
    {
        echo &quot;Facebook: &#39;&quot; . $this-&gt;username . &quot;&#39; has been logged out.\n&quot;;
    }
}

/**
 * Этот Конкретный Класс реализует API Twitter.
 */
class Twitter extends SocialNetwork
{
    public function logIn(string $userName, string $password): bool
    {
        echo &quot;\nChecking user&#39;s credentials...\n&quot;;
        echo &quot;Name: &quot; . $this-&gt;username . &quot;\n&quot;;
        echo &quot;Password: &quot; . str_repeat(&quot;*&quot;, strlen($this-&gt;password)) . &quot;\n&quot;;

        simulateNetworkLatency();

        echo &quot;\n\nTwitter: &#39;&quot; . $this-&gt;username . &quot;&#39; has logged in successfully.\n&quot;;

        return true;
    }

    public function sendData(string $message): bool
    {
        echo &quot;Twitter: &#39;&quot; . $this-&gt;username . &quot;&#39; has posted &#39;&quot; . $message . &quot;&#39;.\n&quot;;

        return true;
    }

    public function logOut(): void
    {
        echo &quot;Twitter: &#39;&quot; . $this-&gt;username . &quot;&#39; has been logged out.\n&quot;;
    }
}

/**
 * Небольшая вспомогательная функция, которая делает время ожидания похожим на
 * реальность.
 */
function simulateNetworkLatency()
{
    $i = 0;
    while ($i &lt; 5) {
        echo &quot;.&quot;;
        sleep(1);
        $i++;
    }
}

/**
 * Клиентский код.
 */
echo &quot;Username: \n&quot;;
$username = readline();
echo &quot;Password: \n&quot;;
$password = readline();
echo &quot;Message: \n&quot;;
$message = readline();

echo &quot;\nChoose the social network to post the message:\n&quot; .
    &quot;1 - Facebook\n&quot; .
    &quot;2 - Twitter\n&quot;;
$choice = readline();

// Теперь давайте создадим правильный объект социальной сети и отправим
// сообщение.
if ($choice == 1) {
    $network = new Facebook($username, $password);
} elseif ($choice == 2) {
    $network = new Twitter($username, $password);
} else {
    die(&quot;Sorry, I&#39;m not sure what you mean by that.\n&quot;);
}
$network-&gt;post($message);</code></pre>
</figure>

#### []{#example-1--Output-txt .anchor} **Output.txt:** Результат выполнения

<figure class="code">
<pre class="code" lang="output"><code>Username:
&gt; neo
Password:
&gt; 123123
Message:
&gt; What is the Matrix?

Choose the social network to post the message:
1 - Facebook
2 - Twitter
&gt; 1

Checking user&#39;s credentials...
Name: neo
Password: ******
.....

Facebook: &#39;neo&#39; has logged in successfully.
Facebook: &#39;neo&#39; has posted &#39;What is the Matrix?&#39;.
Facebook: &#39;neo&#39; has been logged out.</code></pre>
</figure>
:::
:::::

::: {#nav-tab .nav .nav-tabs .nav-tabs-bottom .nav-justified role="tablist"}
[Концептуальный пример](#example-0){#example-0-tab-bottom .nav-item
.nav-link .active toggle="tab" role="tab" aria-controls="example-0"
aria-selected="true"} [Пример из реальной
жизни](#example-1){#example-1-tab-bottom .nav-item .nav-link
toggle="tab" role="tab" aria-controls="example-1" aria-selected="true"}
:::

::: next
#### Читаем дальше

[Посетитель на PHP []{.fa
.fa-arrow-right}](../../visitor/php/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Стратегия на
PHP](../../strategy/php/example.html){.btn .btn-default rel="prev"}
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
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Шаблонный метод на Go"){.prog-lang-link}
[![Шаблонный метод на
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Шаблонный метод на Java"){.prog-lang-link}
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
:::::::::::::::::::::::::

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
:::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::
