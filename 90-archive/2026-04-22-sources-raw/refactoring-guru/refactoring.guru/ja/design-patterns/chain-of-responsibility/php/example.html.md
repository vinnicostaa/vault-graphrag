:::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} /
[デザインパターン](../../../design-patterns.html){.type} / [Chain of
Responsibility](../../chain-of-responsibility.html){.type} /
[PHP](../../php.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Chain of
Responsibility](../../../../images/patterns/cards/chain-of-responsibility-mini6d42.png?id=36d85eba8d14986f053123de17aac7a7){srcset="/images/patterns/cards/chain-of-responsibility-mini-2x.png?id=8c81f7069e51259b2443801b91135f7f 2x"}
:::

# **Chain of Responsibility** を PHP で {#chain-of-responsibility-を-php-で .pattern-example-title-block-title}
::::

:::::::::::::: pattern-example-body
::: pattern-example-brief
**Chain of Responsibility** は[、]{.chpule2}[
]{.chpuri2}振る舞いに関するデザインパターンの一つで[、]{.chpule2}[
]{.chpuri2}潜在的なハンドラーの連鎖の上を[、]{.chpule2}[
]{.chpuri2}ハンドラーのどれかが処理するまで[、]{.chpule2}[
]{.chpuri2}リクエストを回していきます[。]{.chpule2}[ ]{.chpuri2}

このパターンを利用すると[、]{.chpule2}[
]{.chpuri2}送り手のクラスと受け手の具象クラスとを結合することなく[、]{.chpule2}[
]{.chpuri2}複数のオブジェクトにリクエストを処理する機会を与えることができます[。]{.chpule2}[
]{.chpuri2}連鎖は実行時に[、]{.chpule2}[
]{.chpuri2}標準のハンドラー・インターフェースに従うハンドラーから動的に構成されます[。]{.chpule2}[
]{.chpuri2}
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Chain of Responsibility の詳細
](../../chain-of-responsibility.html){.btn .btn-lg .btn-primary}
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

**使用例[：]{.chpule2}[ ]{.chpuri2}** Chain of Responsibility
パターンは[、]{.chpule2}[ ]{.chpuri2}PHP
の種々のフレームワークで結構よく使われます[。]{.chpule2}[
]{.chpuri2}議論の余地はありますが[、]{.chpule2}[ ]{.chpuri2}PHP
でこのパターンを使う最も有名な例としては[、]{.chpule2}[
]{.chpuri2}PSR-15 に記述されている [PHP
リクエスト・ミドルウェア](https://www.php-fig.org/psr/psr-15/)があげられます[。]{.chpule2}[
]{.chpuri2}

**見つけ方[：]{.chpule2}[
]{.chpuri2}**共通のインターフェースに従うオブジェクトのグループで[、]{.chpule2}[
]{.chpuri2}実作業を行うメソッドが[、]{.chpule2}[
]{.chpuri2}別のオブジェクトの同一メソッドを呼ぶことから[、]{.chpule2}[
]{.chpuri2}このパターンを識別できます[。]{.chpule2}[ ]{.chpuri2}
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

この例は[、]{.chpule2}[ ]{.chpuri2}**Chain of Responsibility**
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

#### []{#example-0--Output-txt .anchor} **Output.txt:** 実行結果

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
## 現実的な例 {#example-1-title}

PHP の世界で最も広く知られている **Chain of Responsibility**[
]{.chpule1}[（]{.chpuri1}Co­R[）]{.chpule2}[
]{.chpuri2}パターンの使用は[、]{.chpule2}[ ]{.chpuri2}HTTP
リクエストのミドルウェアにあります[。]{.chpule2}[
]{.chpuri2}これらは最も人気のある PHP
フレームワークによって実装されており[、]{.chpule2}[ ]{.chpuri2}PSR-15
の一部として標準化されています[。]{.chpule2}[ ]{.chpuri2}

以下のように動作します[。]{.chpule2}[ ]{.chpuri2}HTTP
リクエストがアプリによって処理されるためには[、]{.chpule2}[
]{.chpuri2}ミドルウェア・オブジェクトの積み重ねを通過する必要があります[。]{.chpule2}[
]{.chpuri2}各ミドルウェアは[、]{.chpule2}[
]{.chpuri2}リクエストの処理を拒否するか[、]{.chpule2}[
]{.chpuri2}次のミドルウェアに渡すことができます[。]{.chpule2}[
]{.chpuri2}リクエストがすべてのミドルウェアを正常に通過して初めて[、]{.chpule2}[
]{.chpuri2}アプリの主ハンドラーが最終的にそれを処理できます[。]{.chpule2}[
]{.chpuri2}

このやり方は[、]{.chpule2}[
]{.chpuri2}パターンの元々の意図の逆のようであることにお気づきかもしれません[。]{.chpule2}[
]{.chpuri2}その通りで[、]{.chpule2}[
]{.chpuri2}典型的な実装では[、]{.chpule2}[
]{.chpuri2}リクエストは現ハンドラーが**取り扱えない**場合にだけ次に渡されます[。]{.chpule2}[
]{.chpuri2}それに対して[、]{.chpule2}[
]{.chpuri2}ミドルウェアは[、]{.chpule2}[
]{.chpuri2}アプリがリクエストを**取り扱える**と思われる場合に[、]{.chpule2}[
]{.chpuri2}連鎖を通してリクエストを渡します[。]{.chpule2}[
]{.chpuri2}にもかかわらず[、]{.chpule2}[
]{.chpuri2}ミドルウェア・オブジェクトは連鎖をなしているため[、]{.chpule2}[
]{.chpuri2}概念全体は CoR パターンとみなされます[。]{.chpule2}[
]{.chpuri2}

#### []{#example-1--index-php .anchor} **index.php:** 現実的な例

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

#### []{#example-1--Output-txt .anchor} **Output.txt:** 実行結果

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
[概念的な例](#example-0){#example-0-tab-bottom .nav-item .nav-link
.active toggle="tab" role="tab" aria-controls="example-0"
aria-selected="true"} [現実的な例](#example-1){#example-1-tab-bottom
.nav-item .nav-link toggle="tab" role="tab" aria-controls="example-1"
aria-selected="true"}
:::

::: next
#### 次を読む

[Command を PHP で []{.fa
.fa-arrow-right}](../../command/php/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### 戻る

[[]{.fa .fa-arrow-left} Proxy を PHP
で](../../proxy/php/example.html){.btn .btn-default rel="prev"}
:::

## 他言語での **Chain of Responsibility**

[![Chain of Responsibility を C#
で](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Chain of Responsibility を C# で"){.prog-lang-link}
[![Chain of Responsibility を C++
で](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Chain of Responsibility を C++ で"){.prog-lang-link}
[![Chain of Responsibility を Go
で](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Chain of Responsibility を Go で"){.prog-lang-link}
[![Chain of Responsibility を Java
で](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Chain of Responsibility を Java で"){.prog-lang-link}
[![Chain of Responsibility を Python
で](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Chain of Responsibility を Python で"){.prog-lang-link}
[![Chain of Responsibility を Ruby
で](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Chain of Responsibility を Ruby で"){.prog-lang-link}
[![Chain of Responsibility を Rust
で](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Chain of Responsibility を Rust で"){.prog-lang-link}
[![Chain of Responsibility を Swift
で](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Chain of Responsibility を Swift で"){.prog-lang-link}
[![Chain of Responsibility を TypeScript
で](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Chain of Responsibility を TypeScript で"){.prog-lang-link}
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
