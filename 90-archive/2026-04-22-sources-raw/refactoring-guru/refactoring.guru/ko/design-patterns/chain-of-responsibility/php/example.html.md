:::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [디자인
패턴들](../../../design-patterns.html){.type} / [책임
연쇄](../../chain-of-responsibility.html){.type} /
[PHP](../../php.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![책임
연쇄](../../../../images/patterns/cards/chain-of-responsibility-mini6d42.png?id=36d85eba8d14986f053123de17aac7a7){srcset="/images/patterns/cards/chain-of-responsibility-mini-2x.png?id=8c81f7069e51259b2443801b91135f7f 2x"}
:::

# PHP로 작성된 **책임 연쇄** {#php로-작성된-책임-연쇄 .pattern-example-title-block-title}
::::

:::::::::::::: pattern-example-body
::: pattern-example-brief
**책임 연쇄** 패턴은 핸들러 중 하나가 요청을 처리할 때까지 핸들러들의
체인​(사슬)​을 따라 요청을 전달할 수 있게 해주는 행동 디자인 패턴입니다.

이 패턴은 발신자 클래스를 수신자들의 구상 클래스들에 연결하지 않고도
여러 객체가 요청을 처리할 수 있도록 합니다. 체인은 표준 핸들러
인터페이스를 따르는 모든 핸들러와 런타임 때 동적으로 구성될 수 있습니다.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ 책임 연쇄에 대하여 더 자세히 알아보세요
](../../chain-of-responsibility.html){.btn .btn-lg .btn-primary}
:::

::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
내비게이션
:::

::: en-intro
 [소개](#)
:::

::: en-intro
 [개념적인 예시](#example-0)
:::

::: en-file
 [index](#example-0--index-php)
:::

::: en-file
 [Output](#example-0--Output-txt)
:::

::: en-intro
 [실제 사례 예시](#example-1)
:::

::: en-file
 [index](#example-1--index-php)
:::

::: en-file
 [Output](#example-1--Output-txt)
:::
:::::::::::

**복잡도:** []{.fa-stars}

**인기도:** []{.fa-stars}

**사용 예시들:** 책임 연쇄 패턴은 다양한 PHP 프레임워크에서 매우
일반적입니다. PHP에서 이 패턴을 사용하는 가장 유명한 예 중 하나는 PSR
-15에 설명된 [HTTP 요청
미들웨어](https://www.php-fig.org/psr/psr-15/)입니다.

**식별:** 패턴의 모든 객체는 공통 인터페이스를 따르며, 다른 객체들의
같은 메서드들을 간접적으로 호출하는 한 객체 그룹의 행동 메서드들이
있습니다.
::::::::::::::

::: {#nav-tab .nav .nav-tabs .nav-justified role="tablist"}
[개념적인 예시](#example-0){#example-0-tab .nav-item .nav-link .active
toggle="tab" role="tab" aria-controls="example-0" aria-selected="true"}
[실제 사례 예시](#example-1){#example-1-tab .nav-item .nav-link
toggle="tab" role="tab" aria-controls="example-1" aria-selected="true"}
:::

::::: {#nav-tabContent .pattern-example-tab-content .tab-content}
::: {#example-0 .tab-pane .show .active role="tabpanel" aria-labelledby="example-0-tab" style="padding-top: 24px"}
## 개념적인 예시 {#example-0-title}

이 예시는 **책임 연쇄** 패턴의 구조를 보여주고 다음 질문에 중점을
둡니다:

-   패턴은 어떤 클래스들로 구성되어 있나요?
-   이 클래스들은 어떤 역할을 하나요?
-   패턴의 요소들은 어떻게 서로 연관되어 있나요?

이 패턴의 구조를 배우면 실제 PHP 사용 사례를 기반으로 하는 다음 예시를
더욱 쉽게 이해할 수 있을 것입니다.

#### []{#example-0--index-php .anchor} **index.php:** 개념적인 예시

<figure class="code">
<pre class="code" lang="php"><code>&lt;?php

namespace RefactoringGuru\ChainOfResponsibility\Conceptual;

/**
 * The Handler interface declares a method for building the chain of handlers.
 * It also declares a method for executing a request.
 */
interface Handler
{
    public function setNext(Handler $handler): Handler;

    public function handle(string $request): ?string;
}

/**
 * The default chaining behavior can be implemented inside a base handler class.
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
        // Returning a handler from here will let us link handlers in a
        // convenient way like this:
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
 * All Concrete Handlers either handle a request or pass it to the next handler
 * in the chain.
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
 * The client code is usually suited to work with a single handler. In most
 * cases, it is not even aware that the handler is part of a chain.
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
 * The other part of the client code constructs the actual chain.
 */
$monkey = new MonkeyHandler();
$squirrel = new SquirrelHandler();
$dog = new DogHandler();

$monkey-&gt;setNext($squirrel)-&gt;setNext($dog);

/**
 * The client should be able to send a request to any handler, not just the
 * first one in the chain.
 */
echo &quot;Chain: Monkey &gt; Squirrel &gt; Dog\n\n&quot;;
clientCode($monkey);
echo &quot;\n&quot;;

echo &quot;Subchain: Squirrel &gt; Dog\n\n&quot;;
clientCode($squirrel);</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** 실행 결과

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
## 실제 사례 예시 {#example-1-title}

PHP 세계에서 가장 널리 알려진 **책임 연쇄**(CoR) 패턴은 HTTP 요청
미들웨어에서 볼 수 있습니다. 이러한 미들웨어들은 가장 인기 있는 PHP
프레임워크에 의해 구현되며 PSR -15의 일부로 표준화되었습니다.

위는 다음과 같이 작동합니다: HTTP 요청은 앱에서 처리되기 위해 미들웨어
객체들의 스택을 통과해야 합니다. 앱에서 처리하려면 HTTP 요청이 미들웨어
개체 스택을 통과해야 합니다. 각 미들웨어는 요청의 추가 처리를 거부하거나
다음 미들웨어로 전달할 수 있습니다. 요청이 모든 미들웨어를 성공적으로
통과하면 앱의 기본 핸들러가 마침내 이를 처리할 수 있습니다.

이 접근 방식이 패턴의 원래 의도와 약간 반대라는 것을 눈치채셨을 수도
있습니다. 실제로 일반적인 구현에서 요청은 현재 핸들러가 해당 요청을
처리할 수 없는 경우에만 체인을 따라 전달됩니다. 미들웨어는 앱이 요청을
처리할 수 있다고 생각할 때 요청을 체인 아래로 더 전달합니다. 그럼에도
불구하고 미들웨어 객체들이 사슬식으로 연결되어 있으므로 이 개념은 여전히
책임 연쇄 패턴의 예시로 간주됩니다.

#### []{#example-1--index-php .anchor} **index.php:** 실제 사례 예시

<figure class="code">
<pre class="code" lang="php"><code>&lt;?php

namespace RefactoringGuru\ChainOfResponsibility\RealWorld;

/**
 * The classic CoR pattern declares a single role for objects that make up a
 * chain, which is a Handler. In our example, let&#39;s differentiate between
 * middleware and a final application&#39;s handler, which is executed when a
 * request gets through all the middleware objects.
 *
 * The base Middleware class declares an interface for linking middleware
 * objects into a chain.
 */
abstract class Middleware
{
    /**
     * @var Middleware
     */
    private $next;

    /**
     * This method can be used to build a chain of middleware objects.
     */
    public function linkWith(Middleware $next): Middleware
    {
        $this-&gt;next = $next;

        return $next;
    }

    /**
     * Subclasses must override this method to provide their own checks. A
     * subclass can fall back to the parent implementation if it can&#39;t process a
     * request.
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
 * This Concrete Middleware checks whether a user with given credentials exists.
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
 * This Concrete Middleware checks whether a user associated with the request
 * has sufficient permissions.
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
 * This Concrete Middleware checks whether there are too many failed login
 * requests.
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
     * Please, note that the parent::check call can be inserted both at the
     * beginning of this method and at the end.
     *
     * This gives much more flexibility than a simple loop over all middleware
     * objects. For instance, a middleware can change the order of checks by
     * running its check after all the others.
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
 * This is an application&#39;s class that acts as a real handler. The Server class
 * uses the CoR pattern to execute a set of various authentication middleware
 * before launching some business logic associated with a request.
 */
class Server
{
    private $users = [];

    /**
     * @var Middleware
     */
    private $middleware;

    /**
     * The client can configure the server with a chain of middleware objects.
     */
    public function setMiddleware(Middleware $middleware): void
    {
        $this-&gt;middleware = $middleware;
    }

    /**
     * The server gets the email and password from the client and sends the
     * authorization request to the middleware.
     */
    public function logIn(string $email, string $password): bool
    {
        if ($this-&gt;middleware-&gt;check($email, $password)) {
            echo &quot;Server: Authorization has been successful!\n&quot;;

            // Do something useful for authorized users.

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
 * The client code.
 */
$server = new Server();
$server-&gt;register(&quot;admin@example.com&quot;, &quot;admin_pass&quot;);
$server-&gt;register(&quot;user@example.com&quot;, &quot;user_pass&quot;);

// All middleware are chained. The client can build various configurations of
// chains depending on its needs.
$middleware = new ThrottlingMiddleware(2);
$middleware
    -&gt;linkWith(new UserExistsMiddleware($server))
    -&gt;linkWith(new RoleCheckMiddleware());

// The server gets a chain from the client code.
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

#### []{#example-1--Output-txt .anchor} **Output.txt:** 실행 결과

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
[개념적인 예시](#example-0){#example-0-tab-bottom .nav-item .nav-link
.active toggle="tab" role="tab" aria-controls="example-0"
aria-selected="true"} [실제 사례 예시](#example-1){#example-1-tab-bottom
.nav-item .nav-link toggle="tab" role="tab" aria-controls="example-1"
aria-selected="true"}
:::

::: next
#### 다음을 보세요

[PHP로 작성된 커맨드 []{.fa
.fa-arrow-right}](../../command/php/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### 돌아가기

[[]{.fa .fa-arrow-left} PHP로 작성된
프록시](../../proxy/php/example.html){.btn .btn-default rel="prev"}
:::

## 다른 언어로 작성된 **책임 연쇄**

[![C#으로 작성된 책임
연쇄](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "C#으로 작성된 책임 연쇄"){.prog-lang-link}
[![C++로 작성된 책임
연쇄](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "C++로 작성된 책임 연쇄"){.prog-lang-link}
[![Go로 작성된 책임
연쇄](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Go로 작성된 책임 연쇄"){.prog-lang-link}
[![자바로 작성된 책임
연쇄](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "자바로 작성된 책임 연쇄"){.prog-lang-link}
[![파이썬으로 작성된 책임
연쇄](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "파이썬으로 작성된 책임 연쇄"){.prog-lang-link}
[![루비로 작성된 책임
연쇄](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "루비로 작성된 책임 연쇄"){.prog-lang-link}
[![러스트로 작성된 책임
연쇄](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "러스트로 작성된 책임 연쇄"){.prog-lang-link}
[![스위프트로 작성된 책임
연쇄](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "스위프트로 작성된 책임 연쇄"){.prog-lang-link}
[![타입스크립트로 작성된 책임
연쇄](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "타입스크립트로 작성된 책임 연쇄"){.prog-lang-link}
:::::::::::::::::::::::::

::::::: {.prom .banner-sidebar .banner .banner-striped .banner-examples .banner-removable .banner-removable-patterns data-id="DP: Examples-IDE" creative-id="examples-sidebar-ko" position="sidebar"}
:::::: {.banner-inner style="font-size: 14px; line-height: 21px;"}
::: banner-examples-img
[![](../../../../images/patterns/banners/examples-ide91ca.png?id=3115b4b548fb96b75974e2de8f4f49bc){width="215"
height="158" loading="lazy"
srcset="/images/patterns/banners/examples-ide-2x.png?id=93c007a6d157b97c28eb213f59d716ae 2x"}](../../book.html)
:::

::: banner-examples-fade
[ 예시들을 포함한 저장소](../../book.html){.btn .btn-secondary
.btn-block}
:::

::: {.banner-examples-text style="margin-top: 1rem;"}
전자책 **디자인 패턴에 뛰어들기**를 구매하면 IDE에서 바로 열 수 있는
수십 가지의 상세한 코드 예시들이 포함된 저장소를 사용할 수 있습니다.

[ 더 알아보기...](../../book.html){.btn .btn-secondary .btn-block}
:::
::::::
:::::::
:::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::
