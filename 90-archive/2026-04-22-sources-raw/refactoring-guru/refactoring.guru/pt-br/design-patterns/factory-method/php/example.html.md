:::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Padrões de
Projeto](../../../design-patterns.html){.type} / [Factory
Method](../../factory-method.html){.type} / [PHP](../../php.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Factory
Method](../../../../images/patterns/cards/factory-method-minid7f6.png?id=72619e9527893374b98a5913779ac167){srcset="/images/patterns/cards/factory-method-mini-2x.png?id=fa9d4a8d61a67cc3822e52b9daf69dad 2x"}
:::

# **Factory Method** em PHP {#factory-method-em-php .pattern-example-title-block-title}
::::

:::::::::::::: pattern-example-body
::: pattern-example-brief
O **Factory method** é um padrão de projeto criacional, que resolve o
problema de criar objetos de produtos sem especificar suas classes
concretas.

O Factory Method define um método, que deve ser usado para criar objetos
em vez da chamada direta ao construtor (operador `new`). As subclasses
podem substituir esse método para alterar a classe de objetos que serão
criados.

> Se você não conseguir descobrir a diferença entre os padrões
> **Factory**, **Factory Method** e **Abstract Factory**, leia nossa
> [Comparação Factory](../../factory-comparison.html).
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Saiba mais sobre o Factory Method ](../../factory-method.html){.btn
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

**Exemplos de uso:** O padrão Factory Method é amplamente utilizado no
código PHP. É muito útil quando você precisa fornecer um alto nível de
flexibilidade para seu código.

**Identificação:** Os métodos fábrica podem ser reconhecidos por métodos
de criação, que criam objetos de classes concretas, mas os retornam como
objetos de tipo ou interface abstrata.
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

Este exemplo ilustra a estrutura do padrão de projeto **Factory
Method**. Ele se concentra em responder a estas perguntas:

-   De quais classes ele consiste?
-   Quais papéis essas classes desempenham?
-   De que maneira os elementos do padrão estão relacionados?

Depois de aprender sobre a estrutura do padrão, será mais fácil entender
o exemplo a seguir, com base em um caso de uso PHP do mundo real.

#### []{#example-0--index-php .anchor} **index.php:** Exemplo conceitual

<figure class="code">
<pre class="code" lang="php"><code>&lt;?php

namespace RefactoringGuru\FactoryMethod\Conceptual;

/**
 * The Creator class declares the factory method that is supposed to return an
 * object of a Product class. The Creator&#39;s subclasses usually provide the
 * implementation of this method.
 */
abstract class Creator
{
    /**
     * Note that the Creator may also provide some default implementation of the
     * factory method.
     */
    abstract public function factoryMethod(): Product;

    /**
     * Also note that, despite its name, the Creator&#39;s primary responsibility is
     * not creating products. Usually, it contains some core business logic that
     * relies on Product objects, returned by the factory method. Subclasses can
     * indirectly change that business logic by overriding the factory method
     * and returning a different type of product from it.
     */
    public function someOperation(): string
    {
        // Call the factory method to create a Product object.
        $product = $this-&gt;factoryMethod();
        // Now, use the product.
        $result = &quot;Creator: The same creator&#39;s code has just worked with &quot; .
            $product-&gt;operation();

        return $result;
    }
}

/**
 * Concrete Creators override the factory method in order to change the
 * resulting product&#39;s type.
 */
class ConcreteCreator1 extends Creator
{
    /**
     * Note that the signature of the method still uses the abstract product
     * type, even though the concrete product is actually returned from the
     * method. This way the Creator can stay independent of concrete product
     * classes.
     */
    public function factoryMethod(): Product
    {
        return new ConcreteProduct1();
    }
}

class ConcreteCreator2 extends Creator
{
    public function factoryMethod(): Product
    {
        return new ConcreteProduct2();
    }
}

/**
 * The Product interface declares the operations that all concrete products must
 * implement.
 */
interface Product
{
    public function operation(): string;
}

/**
 * Concrete Products provide various implementations of the Product interface.
 */
class ConcreteProduct1 implements Product
{
    public function operation(): string
    {
        return &quot;{Result of the ConcreteProduct1}&quot;;
    }
}

class ConcreteProduct2 implements Product
{
    public function operation(): string
    {
        return &quot;{Result of the ConcreteProduct2}&quot;;
    }
}

/**
 * The client code works with an instance of a concrete creator, albeit through
 * its base interface. As long as the client keeps working with the creator via
 * the base interface, you can pass it any creator&#39;s subclass.
 */
function clientCode(Creator $creator)
{
    // ...
    echo &quot;Client: I&#39;m not aware of the creator&#39;s class, but it still works.\n&quot;
        . $creator-&gt;someOperation();
    // ...
}

/**
 * The Application picks a creator&#39;s type depending on the configuration or
 * environment.
 */
echo &quot;App: Launched with the ConcreteCreator1.\n&quot;;
clientCode(new ConcreteCreator1());
echo &quot;\n\n&quot;;

echo &quot;App: Launched with the ConcreteCreator2.\n&quot;;
clientCode(new ConcreteCreator2());</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** Resultados da execução

<figure class="code">
<pre class="code" lang="output"><code>App: Launched with the ConcreteCreator1.
Client: I&#39;m not aware of the creator&#39;s class, but it still works.
Creator: The same creator&#39;s code has just worked with {Result of the ConcreteProduct1}

App: Launched with the ConcreteCreator2.
Client: I&#39;m not aware of the creator&#39;s class, but it still works.
Creator: The same creator&#39;s code has just worked with {Result of the ConcreteProduct2}</code></pre>
</figure>
:::

::: {#example-1 .tab-pane role="tabpanel" aria-labelledby="example-1-tab" style="padding-top: 24px"}
## Exemplo do mundo real {#example-1-title}

Neste exemplo, o padrão **Factory Method** fornece uma interface para a
criação de conectores de redes sociais, que podem ser usados para
efetuar login na rede, criar postagens e potencialmente realizar outras
atividades---e tudo isso sem acoplar o cliente código para classes
específicas da rede social em particular.

#### []{#example-1--index-php .anchor} **index.php:** Exemplo do mundo real

<figure class="code">
<pre class="code" lang="php"><code>&lt;?php

namespace RefactoringGuru\FactoryMethod\RealWorld;

/**
 * The Creator declares a factory method that can be used as a substitution for
 * the direct constructor calls of products, for instance:
 *
 * - Before: $p = new FacebookConnector();
 * - After: $p = $this-&gt;getSocialNetwork;
 *
 * This allows changing the type of the product being created by
 * SocialNetworkPoster&#39;s subclasses.
 */
abstract class SocialNetworkPoster
{
    /**
     * The actual factory method. Note that it returns the abstract connector.
     * This lets subclasses return any concrete connectors without breaking the
     * superclass&#39; contract.
     */
    abstract public function getSocialNetwork(): SocialNetworkConnector;

    /**
     * When the factory method is used inside the Creator&#39;s business logic, the
     * subclasses may alter the logic indirectly by returning different types of
     * the connector from the factory method.
     */
    public function post($content): void
    {
        // Call the factory method to create a Product object...
        $network = $this-&gt;getSocialNetwork();
        // ...then use it as you will.
        $network-&gt;logIn();
        $network-&gt;createPost($content);
        $network-&gt;logout();
    }
}

/**
 * This Concrete Creator supports Facebook. Remember that this class also
 * inherits the &#39;post&#39; method from the parent class. Concrete Creators are the
 * classes that the Client actually uses.
 */
class FacebookPoster extends SocialNetworkPoster
{
    private $login, $password;

    public function __construct(string $login, string $password)
    {
        $this-&gt;login = $login;
        $this-&gt;password = $password;
    }

    public function getSocialNetwork(): SocialNetworkConnector
    {
        return new FacebookConnector($this-&gt;login, $this-&gt;password);
    }
}

/**
 * This Concrete Creator supports LinkedIn.
 */
class LinkedInPoster extends SocialNetworkPoster
{
    private $email, $password;

    public function __construct(string $email, string $password)
    {
        $this-&gt;email = $email;
        $this-&gt;password = $password;
    }

    public function getSocialNetwork(): SocialNetworkConnector
    {
        return new LinkedInConnector($this-&gt;email, $this-&gt;password);
    }
}

/**
 * The Product interface declares behaviors of various types of products.
 */
interface SocialNetworkConnector
{
    public function logIn(): void;

    public function logOut(): void;

    public function createPost($content): void;
}

/**
 * This Concrete Product implements the Facebook API.
 */
class FacebookConnector implements SocialNetworkConnector
{
    private $login, $password;

    public function __construct(string $login, string $password)
    {
        $this-&gt;login = $login;
        $this-&gt;password = $password;
    }

    public function logIn(): void
    {
        echo &quot;Send HTTP API request to log in user $this-&gt;login with &quot; .
            &quot;password $this-&gt;password\n&quot;;
    }

    public function logOut(): void
    {
        echo &quot;Send HTTP API request to log out user $this-&gt;login\n&quot;;
    }

    public function createPost($content): void
    {
        echo &quot;Send HTTP API requests to create a post in Facebook timeline.\n&quot;;
    }
}

/**
 * This Concrete Product implements the LinkedIn API.
 */
class LinkedInConnector implements SocialNetworkConnector
{
    private $email, $password;

    public function __construct(string $email, string $password)
    {
        $this-&gt;email = $email;
        $this-&gt;password = $password;
    }

    public function logIn(): void
    {
        echo &quot;Send HTTP API request to log in user $this-&gt;email with &quot; .
            &quot;password $this-&gt;password\n&quot;;
    }

    public function logOut(): void
    {
        echo &quot;Send HTTP API request to log out user $this-&gt;email\n&quot;;
    }

    public function createPost($content): void
    {
        echo &quot;Send HTTP API requests to create a post in LinkedIn timeline.\n&quot;;
    }
}

/**
 * The client code can work with any subclass of SocialNetworkPoster since it
 * doesn&#39;t depend on concrete classes.
 */
function clientCode(SocialNetworkPoster $creator)
{
    // ...
    $creator-&gt;post(&quot;Hello world!&quot;);
    $creator-&gt;post(&quot;I had a large hamburger this morning!&quot;);
    // ...
}

/**
 * During the initialization phase, the app can decide which social network it
 * wants to work with, create an object of the proper subclass, and pass it to
 * the client code.
 */
echo &quot;Testing ConcreteCreator1:\n&quot;;
clientCode(new FacebookPoster(&quot;john_smith&quot;, &quot;******&quot;));
echo &quot;\n\n&quot;;

echo &quot;Testing ConcreteCreator2:\n&quot;;
clientCode(new LinkedInPoster(&quot;john_smith@example.com&quot;, &quot;******&quot;));</code></pre>
</figure>

#### []{#example-1--Output-txt .anchor} **Output.txt:** Resultados da execução

<figure class="code">
<pre class="code" lang="output"><code>Testing ConcreteCreator1:
Send HTTP API request to log in user john_smith with password ******
Send HTTP API requests to create a post in Facebook timeline.
Send HTTP API request to log out user john_smith
Send HTTP API request to log in user john_smith with password ******
Send HTTP API requests to create a post in Facebook timeline.
Send HTTP API request to log out user john_smith


Testing ConcreteCreator2:
Send HTTP API request to log in user john_smith@example.com with password ******
Send HTTP API requests to create a post in LinkedIn timeline.
Send HTTP API request to log out user john_smith@example.com
Send HTTP API request to log in user john_smith@example.com with password ******
Send HTTP API requests to create a post in LinkedIn timeline.
Send HTTP API request to log out user john_smith@example.com</code></pre>
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

[Prototype em PHP []{.fa
.fa-arrow-right}](../../prototype/php/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Voltar

[[]{.fa .fa-arrow-left} Builder em
PHP](../../builder/php/example.html){.btn .btn-default rel="prev"}
:::

## **Factory Method** em outras linguagens

[![Factory Method em
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Factory Method em C#"){.prog-lang-link}
[![Factory Method em
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Factory Method em C++"){.prog-lang-link}
[![Factory Method em
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Factory Method em Go"){.prog-lang-link}
[![Factory Method em
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Factory Method em Java"){.prog-lang-link}
[![Factory Method em
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Factory Method em Python"){.prog-lang-link}
[![Factory Method em
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Factory Method em Ruby"){.prog-lang-link}
[![Factory Method em
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Factory Method em Rust"){.prog-lang-link}
[![Factory Method em
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Factory Method em Swift"){.prog-lang-link}
[![Factory Method em
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Factory Method em TypeScript"){.prog-lang-link}
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
