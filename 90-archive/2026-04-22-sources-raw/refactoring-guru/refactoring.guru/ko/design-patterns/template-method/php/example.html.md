:::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [디자인
패턴들](../../../design-patterns.html){.type} / [템플릿
메서드](../../template-method.html){.type} /
[PHP](../../php.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![템플릿
메서드](../../../../images/patterns/cards/template-method-mini56e3.png?id=9f200248d88026d8e79d0f3dae411ab4){srcset="/images/patterns/cards/template-method-mini-2x.png?id=178bf56e39b3a1f548dd636076209c98 2x"}
:::

# PHP로 작성된 **템플릿 메서드** {#php로-작성된-템플릿-메서드 .pattern-example-title-block-title}
::::

:::::::::::::: pattern-example-body
::: pattern-example-brief
**템플릿 메서드**는 기초 클래스에서 알고리즘의 골격을 정의할 수 있도록
하는 행동 디자인 패턴입니다. 또 이 패턴은 자식 클래스들이 전체
알고리즘의 구조를 변경하지 않고도 기본 알고리즘의 단계들을 오버라이드할
수 있도록 합니다.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ 템플릿 메서드에 대하여 더 자세히 알아보세요
](../../template-method.html){.btn .btn-lg .btn-primary}
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

**사용 사례들:** 템플릿 메서드 패턴은 PHP 프레임워크들에서 매우
일반적입니다. 이 패턴은 클래스 상속을 사용하여 기초 프레임워크의 행동을
확장하는 작업을 단순화합니다.

**식별:** 템플릿 메서드는 기초 클래스에 추상적이거나 비어 있는 다른 여러
메서드들을 호출하는 메서드가 있습니다.
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

이 예시는 **템플릿 메서드**의 구조를 보여주고 다음 질문에 중점을 둡니다:

-   패턴은 어떤 클래스들로 구성되어 있나요?
-   이 클래스들은 어떤 역할을 하나요?
-   패턴의 요소들은 어떻게 서로 연관되어 있나요?

이 패턴의 구조를 배우면 실제 PHP 사용 사례를 기반으로 하는 다음 예시를
더욱 쉽게 이해할 수 있을 것입니다.

#### []{#example-0--index-php .anchor} **index.php:** 개념적인 예시

<figure class="code">
<pre class="code" lang="php"><code>&lt;?php

namespace RefactoringGuru\TemplateMethod\Conceptual;

/**
 * The Abstract Class defines a template method that contains a skeleton of some
 * algorithm, composed of calls to (usually) abstract primitive operations.
 *
 * Concrete subclasses should implement these operations, but leave the template
 * method itself intact.
 */
abstract class AbstractClass
{
    /**
     * The template method defines the skeleton of an algorithm.
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
     * These operations already have implementations.
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
     * These operations have to be implemented in subclasses.
     */
    abstract protected function requiredOperations1(): void;

    abstract protected function requiredOperation2(): void;

    /**
     * These are &quot;hooks.&quot; Subclasses may override them, but it&#39;s not mandatory
     * since the hooks already have default (but empty) implementation. Hooks
     * provide additional extension points in some crucial places of the
     * algorithm.
     */
    protected function hook1(): void
    {
    }

    protected function hook2(): void
    {
    }
}

/**
 * Concrete classes have to implement all abstract operations of the base class.
 * They can also override some operations with a default implementation.
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
 * Usually, concrete classes override only a fraction of base class&#39; operations.
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
 * The client code calls the template method to execute the algorithm. Client
 * code does not have to know the concrete class of an object it works with, as
 * long as it works with objects through the interface of their base class.
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

#### []{#example-0--Output-txt .anchor} **Output.txt:** 실행 결과

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
## 실제 사례 예시 {#example-1-title}

이 예시에서 **템플릿 메서드** 패턴은 소셜 네트워크에 메시지를 게시하는
알고리즘의 골격을 정의합니다. 각 자식 클래스는 별도의 소셜 네트워크를
나타내며 모든 단계를 다르게 구현하나 기초 알고리즘을 재사용합니다.

#### []{#example-1--index-php .anchor} **index.php:** 실제 사례 예시

<figure class="code">
<pre class="code" lang="php"><code>&lt;?php

namespace RefactoringGuru\TemplateMethod\RealWorld;

/**
 * The Abstract Class defines the template method and declares all its steps.
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
     * The actual template method calls abstract steps in a specific order. A
     * subclass may implement all of the steps, allowing this method to actually
     * post something to a social network.
     */
    public function post(string $message): bool
    {
        // Authenticate before posting. Every network uses a different
        // authentication method.
        if ($this-&gt;logIn($this-&gt;username, $this-&gt;password)) {
            // Send the post data. All networks have different APIs.
            $result = $this-&gt;sendData($message);
            // ...
            $this-&gt;logOut();

            return $result;
        }

        return false;
    }

    /**
     * The steps are declared abstract to force the subclasses to implement them
     * all.
     */
    abstract public function logIn(string $userName, string $password): bool;

    abstract public function sendData(string $message): bool;

    abstract public function logOut(): void;
}

/**
 * This Concrete Class implements the Facebook API (all right, it pretends to).
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
 * This Concrete Class implements the Twitter API.
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
 * A little helper function that makes waiting times feel real.
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
 * The client code.
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

// Now, let&#39;s create a proper social network object and send the message.
if ($choice == 1) {
    $network = new Facebook($username, $password);
} elseif ($choice == 2) {
    $network = new Twitter($username, $password);
} else {
    die(&quot;Sorry, I&#39;m not sure what you mean by that.\n&quot;);
}
$network-&gt;post($message);</code></pre>
</figure>

#### []{#example-1--Output-txt .anchor} **Output.txt:** 실행 결과

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
[개념적인 예시](#example-0){#example-0-tab-bottom .nav-item .nav-link
.active toggle="tab" role="tab" aria-controls="example-0"
aria-selected="true"} [실제 사례 예시](#example-1){#example-1-tab-bottom
.nav-item .nav-link toggle="tab" role="tab" aria-controls="example-1"
aria-selected="true"}
:::

::: next
#### 다음을 보세요

[PHP로 작성된 비지터 []{.fa
.fa-arrow-right}](../../visitor/php/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### 돌아가기

[[]{.fa .fa-arrow-left} PHP로 작성된
전략](../../strategy/php/example.html){.btn .btn-default rel="prev"}
:::

## 다른 언어로 작성된 **템플릿 메서드**

[![C#으로 작성된 템플릿
메서드](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "C#으로 작성된 템플릿 메서드"){.prog-lang-link}
[![C++로 작성된 템플릿
메서드](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "C++로 작성된 템플릿 메서드"){.prog-lang-link}
[![Go로 작성된 템플릿
메서드](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Go로 작성된 템플릿 메서드"){.prog-lang-link}
[![자바로 작성된 템플릿
메서드](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "자바로 작성된 템플릿 메서드"){.prog-lang-link}
[![파이썬으로 작성된 템플릿
메서드](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "파이썬으로 작성된 템플릿 메서드"){.prog-lang-link}
[![루비로 작성된 템플릿
메서드](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "루비로 작성된 템플릿 메서드"){.prog-lang-link}
[![러스트로 작성된 템플릿
메서드](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "러스트로 작성된 템플릿 메서드"){.prog-lang-link}
[![스위프트로 작성된 템플릿
메서드](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "스위프트로 작성된 템플릿 메서드"){.prog-lang-link}
[![타입스크립트로 작성된 템플릿
메서드](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "타입스크립트로 작성된 템플릿 메서드"){.prog-lang-link}
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
