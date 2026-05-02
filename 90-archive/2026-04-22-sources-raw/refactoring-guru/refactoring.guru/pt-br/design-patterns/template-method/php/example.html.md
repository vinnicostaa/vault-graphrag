:::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Padrões de
Projeto](../../../design-patterns.html){.type} / [Template
Method](../../template-method.html){.type} /
[PHP](../../php.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Template
Method](../../../../images/patterns/cards/template-method-mini56e3.png?id=9f200248d88026d8e79d0f3dae411ab4){srcset="/images/patterns/cards/template-method-mini-2x.png?id=178bf56e39b3a1f548dd636076209c98 2x"}
:::

# **Template Method** em PHP {#template-method-em-php .pattern-example-title-block-title}
::::

:::::::::::::: pattern-example-body
::: pattern-example-brief
O **Template Method** é um padrão de projeto comportamental que permite
definir o esqueleto de um algoritmo em uma classe base e permitir que as
subclasses substituam as etapas sem alterar a estrutura geral do
algoritmo.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Saiba mais sobre o Template Method ](../../template-method.html){.btn
.btn-lg .btn-primary}
:::

::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
Navegação
:::

::: en-intro
 [Introdução](#)
:::

::: en-intro
 [Exemplo conceitual](#example-0)
:::

::: en-file
 [index](#example-0--index-php)
:::

::: en-file
 [Output](#example-0--Output-txt)
:::

::: en-intro
 [Exemplo do mundo real](#example-1)
:::

::: en-file
 [index](#example-1--index-php)
:::

::: en-file
 [Output](#example-1--Output-txt)
:::
:::::::::::

**Complexidade:** []{.fa-stars}

**Popularidade:** []{.fa-stars}

**Exemplos de uso:** O padrão Template Method é bastante comum nos
frameworks PHP. O padrão simplifica a extensão do comportamento de uma
estrutura padrão usando a herança de classe.

**Identificação:** O Template Method pode ser reconhecido por métodos
comportamentais que já possuem um comportamento "padrão" definido pela
classe base.
::::::::::::::

::: {#nav-tab .nav .nav-tabs .nav-justified role="tablist"}
[Exemplo conceitual](#example-0){#example-0-tab .nav-item .nav-link
.active toggle="tab" role="tab" aria-controls="example-0"
aria-selected="true"} [Exemplo do mundo real](#example-1){#example-1-tab
.nav-item .nav-link toggle="tab" role="tab" aria-controls="example-1"
aria-selected="true"}
:::

::::: {#nav-tabContent .pattern-example-tab-content .tab-content}
::: {#example-0 .tab-pane .show .active role="tabpanel" aria-labelledby="example-0-tab" style="padding-top: 24px"}
## Exemplo conceitual {#example-0-title}

Este exemplo ilustra a estrutura do padrão de projeto **Template
Method**. Ele se concentra em responder a estas perguntas:

-   De quais classes ele consiste?
-   Quais papéis essas classes desempenham?
-   De que maneira os elementos do padrão estão relacionados?

Depois de aprender sobre a estrutura do padrão, será mais fácil entender
o exemplo a seguir, com base em um caso de uso PHP do mundo real.

#### []{#example-0--index-php .anchor} **index.php:** Exemplo conceitual

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

#### []{#example-0--Output-txt .anchor} **Output.txt:** Resultados da execução

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
## Exemplo do mundo real {#example-1-title}

Neste exemplo, o padrão **Template Method** define um esqueleto do
algoritmo de postagem de mensagens nas redes sociais. Cada subclasse
representa uma rede social separada e implementa todas as etapas de
maneira diferente, mas reutiliza o algoritmo base.

#### []{#example-1--index-php .anchor} **index.php:** Exemplo do mundo real

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

#### []{#example-1--Output-txt .anchor} **Output.txt:** Resultados da execução

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
[Exemplo conceitual](#example-0){#example-0-tab-bottom .nav-item
.nav-link .active toggle="tab" role="tab" aria-controls="example-0"
aria-selected="true"} [Exemplo do mundo
real](#example-1){#example-1-tab-bottom .nav-item .nav-link toggle="tab"
role="tab" aria-controls="example-1" aria-selected="true"}
:::

::: next
#### Leia a seguir

[Visitor em PHP []{.fa
.fa-arrow-right}](../../visitor/php/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Voltar

[[]{.fa .fa-arrow-left} Strategy em
PHP](../../strategy/php/example.html){.btn .btn-default rel="prev"}
:::

## **Template Method** em outras linguagens

[![Template Method em
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Template Method em C#"){.prog-lang-link}
[![Template Method em
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Template Method em C++"){.prog-lang-link}
[![Template Method em
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Template Method em Go"){.prog-lang-link}
[![Template Method em
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Template Method em Java"){.prog-lang-link}
[![Template Method em
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Template Method em Python"){.prog-lang-link}
[![Template Method em
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Template Method em Ruby"){.prog-lang-link}
[![Template Method em
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Template Method em Rust"){.prog-lang-link}
[![Template Method em
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Template Method em Swift"){.prog-lang-link}
[![Template Method em
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Template Method em TypeScript"){.prog-lang-link}
:::::::::::::::::::::::::

::::::: {.prom .banner-sidebar .banner .banner-striped .banner-examples .banner-removable .banner-removable-patterns data-id="DP: Examples-IDE" creative-id="examples-sidebar-pt-br" position="sidebar"}
:::::: {.banner-inner style="font-size: 14px; line-height: 21px;"}
::: banner-examples-img
[![](../../../../images/patterns/banners/examples-ide91ca.png?id=3115b4b548fb96b75974e2de8f4f49bc){width="215"
height="158" loading="lazy"
srcset="/images/patterns/banners/examples-ide-2x.png?id=93c007a6d157b97c28eb213f59d716ae 2x"}](../../book.html)
:::

::: banner-examples-fade
[ Arquivo com exemplos de código](../../book.html){.btn .btn-secondary
.btn-block}
:::

::: {.banner-examples-text style="margin-top: 1rem;"}
Compre o eBook **Mergulho nos Padrões de Projeto** e tenha acesso ao
arquivo com dúzias de exemplos detalhados que podem ser abertos
diretamente em seu IDE.

[ Saiba mais...](../../book.html){.btn .btn-secondary .btn-block}
:::
::::::
:::::::
:::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::
