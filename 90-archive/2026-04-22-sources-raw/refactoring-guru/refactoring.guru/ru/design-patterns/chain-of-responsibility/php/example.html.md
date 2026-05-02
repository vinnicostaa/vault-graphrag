:::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Паттерны
проектирования](../../../design-patterns.html){.type} / [Цепочка
обязанностей](../../chain-of-responsibility.html){.type} /
[PHP](../../php.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Цепочка
обязанностей](../../../../images/patterns/cards/chain-of-responsibility-mini6d42.png?id=36d85eba8d14986f053123de17aac7a7){srcset="/images/patterns/cards/chain-of-responsibility-mini-2x.png?id=8c81f7069e51259b2443801b91135f7f 2x"}
:::

# **Цепочка обязанностей** на PHP {#цепочка-обязанностей-на-php .pattern-example-title-block-title}
::::

:::::::::::::: pattern-example-body
::: pattern-example-brief
**Цепочка обязанностей** --- это поведенческий паттерн, позволяющий
передавать запрос по цепочке потенциальных обработчиков, пока один из
них не обработает запрос.

Избавляет от жёсткой привязки отправителя запроса к его получателю,
позволяя выстраивать цепь из различных обработчиков динамически.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Подробней о паттерне Цепочка обязанностей
](../../chain-of-responsibility.html){.btn .btn-lg .btn-primary}
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

**Применимость:** Паттерн встречается в PHP не так уж часто, так как для
его применения нужно, чтобы в программе были цепи объектов. Пожалуй,
самым известным примером использования этого паттерна в PHP является
концепция [HTTP Request Middleware, описанная в
PSR-15](https://www.php-fig.org/psr/psr-15/). Это обработчики запросов,
которые программа запускает перед тем, как выполнить основной обработчик
запроса. Если их собрать в одну цепь (что чаще всего и происходит в
реальных приложениях), то получится конструкция, очень схожая с
паттерном Цепочка Обязанностей.

**Признаки применения паттерна:** Цепочку обязанностей можно определить
по спискам обработчиков или проверок, через которые пропускаются
запросы. Особенно если порядок следования обработчиков важен.
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

Этот пример показывает структуру паттерна **Цепочка обязанностей**, а
именно --- из каких классов он состоит, какие роли эти классы выполняют
и как они взаимодействуют друг с другом.

После ознакомления со структурой, вам будет легче воспринимать второй
пример, который рассматривает реальный случай использования паттерна в
мире PHP.

#### []{#example-0--index-php .anchor} **index.php:** Пример структуры паттерна

<figure class="code">
<pre class="code" lang="php"><code>&lt;?php

namespace RefactoringGuru\ChainOfResponsibility\Conceptual;

/**
 * Интерфейс Обработчика объявляет метод построения цепочки обработчиков. Он
 * также объявляет метод для выполнения запроса.
 */
interface Handler
{
    public function setNext(Handler $handler): Handler;

    public function handle(string $request): ?string;
}

/**
 * Поведение цепочки по умолчанию может быть реализовано внутри базового класса
 * обработчика.
 */
abstract class AbstractHandler implements Handler
{
    /**
     * @var Handler
     */
    private $nextHandler;

    public function setNext(Handler $handler): Handler
    {
        $this-&gt;nextHandler = $handler;
        // Возврат обработчика отсюда позволит связать обработчики простым
        // способом, вот так:
        // $monkey-&gt;setNext($squirrel)-&gt;setNext($dog);
        return $handler;
    }

    public function handle(string $request): ?string
    {
        if ($this-&gt;nextHandler) {
            return $this-&gt;nextHandler-&gt;handle($request);
        }

        return null;
    }
}

/**
 * Все Конкретные Обработчики либо обрабатывают запрос, либо передают его
 * следующему обработчику в цепочке.
 */
class MonkeyHandler extends AbstractHandler
{
    public function handle(string $request): ?string
    {
        if ($request === &quot;Banana&quot;) {
            return &quot;Monkey: I&#39;ll eat the &quot; . $request . &quot;.\n&quot;;
        } else {
            return parent::handle($request);
        }
    }
}

class SquirrelHandler extends AbstractHandler
{
    public function handle(string $request): ?string
    {
        if ($request === &quot;Nut&quot;) {
            return &quot;Squirrel: I&#39;ll eat the &quot; . $request . &quot;.\n&quot;;
        } else {
            return parent::handle($request);
        }
    }
}

class DogHandler extends AbstractHandler
{
    public function handle(string $request): ?string
    {
        if ($request === &quot;MeatBall&quot;) {
            return &quot;Dog: I&#39;ll eat the &quot; . $request . &quot;.\n&quot;;
        } else {
            return parent::handle($request);
        }
    }
}

/**
 * Обычно клиентский код приспособлен для работы с единственным обработчиком. В
 * большинстве случаев клиенту даже неизвестно, что этот обработчик является
 * частью цепочки.
 */
function clientCode(Handler $handler)
{
    foreach ([&quot;Nut&quot;, &quot;Banana&quot;, &quot;Cup of coffee&quot;] as $food) {
        echo &quot;Client: Who wants a &quot; . $food . &quot;?\n&quot;;
        $result = $handler-&gt;handle($food);
        if ($result) {
            echo &quot;  &quot; . $result;
        } else {
            echo &quot;  &quot; . $food . &quot; was left untouched.\n&quot;;
        }
    }
}

/**
 * Другая часть клиентского кода создает саму цепочку.
 */
$monkey = new MonkeyHandler();
$squirrel = new SquirrelHandler();
$dog = new DogHandler();

$monkey-&gt;setNext($squirrel)-&gt;setNext($dog);

/**
 * Клиент должен иметь возможность отправлять запрос любому обработчику, а не
 * только первому в цепочке.
 */
echo &quot;Chain: Monkey &gt; Squirrel &gt; Dog\n\n&quot;;
clientCode($monkey);
echo &quot;\n&quot;;

echo &quot;Subchain: Squirrel &gt; Dog\n\n&quot;;
clientCode($squirrel);</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** Результат выполнения

<figure class="code">
<pre class="code" lang="output"><code>Chain: Monkey &gt; Squirrel &gt; Dog

Client: Who wants a Nut?
  Squirrel: I&#39;ll eat the Nut.
Client: Who wants a Banana?
  Monkey: I&#39;ll eat the Banana.
Client: Who wants a Cup of coffee?
  Cup of coffee was left untouched.

Subchain: Squirrel &gt; Dog

Client: Who wants a Nut?
  Squirrel: I&#39;ll eat the Nut.
Client: Who wants a Banana?
  Banana was left untouched.
Client: Who wants a Cup of coffee?
  Cup of coffee was left untouched.</code></pre>
</figure>
:::

::: {#example-1 .tab-pane role="tabpanel" aria-labelledby="example-1-tab" style="padding-top: 24px"}
## Пример из реальной жизни {#example-1-title}

Позволяет избежать привязки отправителя запроса к его получателю,
предоставляя возможность обработать запрос нескольким объектам.
Связывает в цепочку объекты-получатели, а затем передаёт запрос по
цепочке, пока некий получатель не обработает его.

Пример: Пожалуй, самым известным применением паттерна Цепочка
обязанностей (CoR) в мире PHP являются промежуточные обработчики
HTTP-запросов, называемые **middleware**. Они стали настолько
популярными, что были реализованы в самом языке как часть
[PSR-15](https://www.php-fig.org/psr/psr-15/).

Всё это работает следующим образом: HTTP-запрос должен пройти через стек
объектов middleware, прежде чем приложение его обработает. Каждое
middleware может либо отклонить дальнейшую обработку запроса, либо
передать его следующему middleware. Как только запрос успешно пройдёт
все middleware, основной обработчик приложения сможет окончательно его
обработать.

Можно отметить, что такой подход --- своего рода инверсия
первоначального замысла паттерна. Действительно, в стандартной
реализации запрос передаётся по цепочке только в том случае, если
текущий обработчик НЕ МОЖЕТ его обработать, тогда как middleware
передаёт запрос дальше по цепочке, когда считает, что приложение МОЖЕТ
обработать запрос. Тем не менее, поскольку middleware соединены
цепочкой, вся концепция по-прежнему считается примером паттерна CoR.

#### []{#example-1--index-php .anchor} **index.php:** Пример из реальной жизни

<figure class="code">
<pre class="code" lang="php"><code>&lt;?php

namespace RefactoringGuru\ChainOfResponsibility\RealWorld;

/**
 * Классический паттерн CoR объявляет для объектов, составляющих цепочку,
 * единственную роль – Обработчик. В нашем примере давайте проводить различие
 * между middleware и конечным обработчиком приложения, который выполняется,
 * когда запрос проходит через все объекты middleware.
 *
 * Базовый класс Middleware объявляет интерфейс для связывания объектов
 * middleware в цепочку.
 */
abstract class Middleware
{
    /**
     * @var Middleware
     */
    private $next;

    /**
     * Этот метод можно использовать для построения цепочки объектов middleware.
     */
    public function linkWith(Middleware $next): Middleware
    {
        $this-&gt;next = $next;

        return $next;
    }

    /**
     * Подклассы должны переопределить этот метод, чтобы предоставить свои
     * собственные проверки. Подкласс может обратиться к родительской реализации
     * проверки, если сам не в состоянии обработать запрос.
     */
    public function check(string $email, string $password): bool
    {
        if (!$this-&gt;next) {
            return true;
        }

        return $this-&gt;next-&gt;check($email, $password);
    }
}

/**
 * Это Конкретное Middleware проверяет, существует ли пользователь с указанными
 * учётными данными.
 */
class UserExistsMiddleware extends Middleware
{
    private $server;

    public function __construct(Server $server)
    {
        $this-&gt;server = $server;
    }

    public function check(string $email, string $password): bool
    {
        if (!$this-&gt;server-&gt;hasEmail($email)) {
            echo &quot;UserExistsMiddleware: This email is not registered!\n&quot;;

            return false;
        }

        if (!$this-&gt;server-&gt;isValidPassword($email, $password)) {
            echo &quot;UserExistsMiddleware: Wrong password!\n&quot;;

            return false;
        }

        return parent::check($email, $password);
    }
}

/**
 * Это Конкретное Middleware проверяет, имеет ли пользователь, связанный с
 * запросом, достаточные права доступа.
 */
class RoleCheckMiddleware extends Middleware
{
    public function check(string $email, string $password): bool
    {
        if ($email === &quot;admin@example.com&quot;) {
            echo &quot;RoleCheckMiddleware: Hello, admin!\n&quot;;

            return true;
        }
        echo &quot;RoleCheckMiddleware: Hello, user!\n&quot;;

        return parent::check($email, $password);
    }
}

/**
 * Это Конкретное Middleware проверяет, не было ли превышено максимальное число
 * неудачных запросов авторизации.
 */
class ThrottlingMiddleware extends Middleware
{
    private $requestPerMinute;

    private $request;

    private $currentTime;

    public function __construct(int $requestPerMinute)
    {
        $this-&gt;requestPerMinute = $requestPerMinute;
        $this-&gt;currentTime = time();
    }

    /**
     * Обратите внимание, что вызов parent::check можно вставить как в начале
     * этого метода, так и в конце.
     *
     * Это даёт значительно большую свободу действий, чем простой цикл по всем
     * объектам middleware. Например, middleware может изменить порядок
     * проверок, запустив свою проверку после всех остальных.
     */
    public function check(string $email, string $password): bool
    {
        if (time() &gt; $this-&gt;currentTime + 60) {
            $this-&gt;request = 0;
            $this-&gt;currentTime = time();
        }

        $this-&gt;request++;

        if ($this-&gt;request &gt; $this-&gt;requestPerMinute) {
            echo &quot;ThrottlingMiddleware: Request limit exceeded!\n&quot;;
            die();
        }

        return parent::check($email, $password);
    }
}

/**
 * Это класс приложения, который осуществляет реальную обработку запроса. Класс
 * Сервер использует паттерн CoR для выполнения набора различных промежуточных
 * проверок перед запуском некоторой бизнес-логики, связанной с запросом.
 */
class Server
{
    private $users = [];

    /**
     * @var Middleware
     */
    private $middleware;

    /**
     * Клиент может настроить сервер с помощью цепочки объектов middleware.
     */
    public function setMiddleware(Middleware $middleware): void
    {
        $this-&gt;middleware = $middleware;
    }

    /**
     * Сервер получает email и пароль от клиента и отправляет запрос авторизации
     * в middleware.
     */
    public function logIn(string $email, string $password): bool
    {
        if ($this-&gt;middleware-&gt;check($email, $password)) {
            echo &quot;Server: Authorization has been successful!\n&quot;;

            // Выполняем что-нибудь полезное для авторизованных пользователей.

            return true;
        }

        return false;
    }

    public function register(string $email, string $password): void
    {
        $this-&gt;users[$email] = $password;
    }

    public function hasEmail(string $email): bool
    {
        return isset($this-&gt;users[$email]);
    }

    public function isValidPassword(string $email, string $password): bool
    {
        return $this-&gt;users[$email] === $password;
    }
}

/**
 * Клиентский код.
 */
$server = new Server();
$server-&gt;register(&quot;admin@example.com&quot;, &quot;admin_pass&quot;);
$server-&gt;register(&quot;user@example.com&quot;, &quot;user_pass&quot;);

// Все middleware соединены в цепочки. Клиент может построить различные
// конфигурации цепочек в зависимости от своих потребностей.
$middleware = new ThrottlingMiddleware(2);
$middleware
    -&gt;linkWith(new UserExistsMiddleware($server))
    -&gt;linkWith(new RoleCheckMiddleware());

// Сервер получает цепочку из клиентского кода.
$server-&gt;setMiddleware($middleware);

// ...

do {
    echo &quot;\nEnter your email:\n&quot;;
    $email = readline();
    echo &quot;Enter your password:\n&quot;;
    $password = readline();
    $success = $server-&gt;logIn($email, $password);
} while (!$success);</code></pre>
</figure>

#### []{#example-1--Output-txt .anchor} **Output.txt:** Результат выполнения

<figure class="code">
<pre class="code" lang="output"><code>Enter your email:
asd
Enter your password:
123
UserExistsMiddleware: This email is not registered!

Enter your email:
admin@example.com
Enter your password:
wrong
UserExistsMiddleware: Wrong password!

Enter your email:
admin@example.com
Enter your password:
letmein
ThrottlingMiddleware: Request limit exceeded!



Enter your email:
admin@example.com
Enter your password:
admin_pass
RoleCheckMiddleware: Hello, admin!
Server: Authorization has been successful!</code></pre>
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

[Команда на PHP []{.fa
.fa-arrow-right}](../../command/php/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Заместитель на
PHP](../../proxy/php/example.html){.btn .btn-default rel="prev"}
:::

## **Цепочка обязанностей** на других языках программирования

[![Цепочка обязанностей на
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Цепочка обязанностей на C#"){.prog-lang-link}
[![Цепочка обязанностей на
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Цепочка обязанностей на C++"){.prog-lang-link}
[![Цепочка обязанностей на
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Цепочка обязанностей на Go"){.prog-lang-link}
[![Цепочка обязанностей на
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Цепочка обязанностей на Java"){.prog-lang-link}
[![Цепочка обязанностей на
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Цепочка обязанностей на Python"){.prog-lang-link}
[![Цепочка обязанностей на
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Цепочка обязанностей на Ruby"){.prog-lang-link}
[![Цепочка обязанностей на
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Цепочка обязанностей на Rust"){.prog-lang-link}
[![Цепочка обязанностей на
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Цепочка обязанностей на Swift"){.prog-lang-link}
[![Цепочка обязанностей на
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Цепочка обязанностей на TypeScript"){.prog-lang-link}
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
