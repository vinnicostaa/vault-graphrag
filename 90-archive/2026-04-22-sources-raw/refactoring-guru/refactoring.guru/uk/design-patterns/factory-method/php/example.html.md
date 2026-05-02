:::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Патерни
проектування](../../../design-patterns.html){.type} / [Фабричний
метод](../../factory-method.html){.type} / [PHP](../../php.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Фабричний
метод](../../../../images/patterns/cards/factory-method-minid7f6.png?id=72619e9527893374b98a5913779ac167){srcset="/images/patterns/cards/factory-method-mini-2x.png?id=fa9d4a8d61a67cc3822e52b9daf69dad 2x"}
:::

# **Фабричний метод** на PHP {#фабричний-метод-на-php .pattern-example-title-block-title}
::::

:::::::::::::: pattern-example-body
::: pattern-example-brief
**Фабричний метод** --- це породжуючий патерн проектування, який вирішує
проблему створення різних продуктів, без прив'язки коду до конкретних
класів продуктів.

Фабричний метод задає метод, який необхідно використовувати замість
виклику оператора `new` для створення об'єктів-продуктів. Підкласи
можуть перевизначити цей метод, щоб змінювати тип створюваних продуктів.

> Якщо ви вже чули про *Фабрику*, *Фабричний метод* чи *Абстрактну
> фабрику*, але вам все одно важко їх розрізняти, то прочитайте нашу
> статтю [Порівняння фабрик](../../factory-comparison.html).
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Детальніше про Фабричний метод ](../../factory-method.html){.btn
.btn-lg .btn-primary}
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

**Застосування:** Патерн можна часто зустріти в будь-якому PHP-коді,
коли потрібна гнучкість при створенні продуктів.

**Ознаки застосування патерна:** Фабричний метод можна визначити за
створюючими методами, які повертають об'єкти продуктів через абстрактні
типи або інтерфейси. Це дозволяє змінювати типи створюваних продуктів у
підкласах.
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

Цей приклад показує структуру патерна **Фабричний метод**, а саме --- з
яких класів він складається, які ролі ці класи виконують і як вони
взаємодіють один з одним.

Після ознайомлення зі структурою, вам буде легше сприймати наступний
приклад, що розглядає реальний випадок використання патерна в світі PHP.

#### []{#example-0--index-php .anchor} **index.php:** Приклад структури патерна

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

#### []{#example-0--Output-txt .anchor} **Output.txt:** Результат виконання

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
## Життєвий приклад {#example-1-title}

#### []{#example-1--index-php .anchor} **index.php:** Приклад з реального світу

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

#### []{#example-1--Output-txt .anchor} **Output.txt:** Результат виконання

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
[Концептуальний приклад](#example-0){#example-0-tab-bottom .nav-item
.nav-link .active toggle="tab" role="tab" aria-controls="example-0"
aria-selected="true"} [Життєвий
приклад](#example-1){#example-1-tab-bottom .nav-item .nav-link
toggle="tab" role="tab" aria-controls="example-1" aria-selected="true"}
:::

::: next
#### Читаємо далі

[Прототип на PHP []{.fa
.fa-arrow-right}](../../prototype/php/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Повернутись назад

[[]{.fa .fa-arrow-left} Будівельник на
PHP](../../builder/php/example.html){.btn .btn-default rel="prev"}
:::

## **Фабричний метод** іншими мовами програмування

[![Фабричний метод на
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Фабричний метод на C#"){.prog-lang-link}
[![Фабричний метод на
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Фабричний метод на C++"){.prog-lang-link}
[![Фабричний метод на
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Фабричний метод на Go"){.prog-lang-link}
[![Фабричний метод на
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Фабричний метод на Java"){.prog-lang-link}
[![Фабричний метод на
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Фабричний метод на Python"){.prog-lang-link}
[![Фабричний метод на
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Фабричний метод на Ruby"){.prog-lang-link}
[![Фабричний метод на
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Фабричний метод на Rust"){.prog-lang-link}
[![Фабричний метод на
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Фабричний метод на Swift"){.prog-lang-link}
[![Фабричний метод на
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Фабричний метод на TypeScript"){.prog-lang-link}
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
