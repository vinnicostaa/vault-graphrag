:::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Patrons de
conception](../../../design-patterns.html){.type} /
[Stratégie](../../strategy.html){.type} / [PHP](../../php.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Stratégie](../../../../images/patterns/cards/strategy-mini28f6.png?id=d38abee4fb6f2aed909d262bdadca936){srcset="/images/patterns/cards/strategy-mini-2x.png?id=f4e6608561f8e5d18be6927d4620ad29 2x"}
:::

# **Stratégie** en PHP {#stratégie-en-php .pattern-example-title-block-title}
::::

:::::::::::::: pattern-example-body
::: pattern-example-brief
La **Stratégie** est un patron de conception comportemental qui
transforme un ensemble de comportements en objets, et les rend
interchangeables à l'intérieur de l'objet du contexte original.

L'objet original, que l'on appelle contexte, garde une référence vers un
objet stratégie et lui délègue l'exécution du comportement. Les autres
objets doivent remplacer l'objet stratégie associé afin de modifier la
manière dont le contexte fonctionne.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ En savoir plus sur la patron Stratégie ](../../strategy.html){.btn
.btn-lg .btn-primary}
:::

::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
Navigation
:::

::: en-intro
 [Intro](#)
:::

::: en-intro
 [Exemple conceptuel](#example-0)
:::

::: en-file
 [index](#example-0--index-php)
:::

::: en-file
 [Output](#example-0--Output-txt)
:::

::: en-intro
 [Analogie du monde réel](#example-1)
:::

::: en-file
 [index](#example-1--index-php)
:::

::: en-file
 [Output](#example-1--Output-txt)
:::
:::::::::::

**Complexité :** []{.fa-stars}

**Popularité :** []{.fa-stars}

**Exemples d'utilisation :** La stratégie est souvent utilisée en PHP,
principalement lorsqu'il est nécessaire de changer l'algorithme à
l'exécution. Cependant, les fonctions anonymes se sont présentées comme
un sérieux concurrent depuis qu'elles ont été introduites en PHP 5.3 en
2009.

**Identification :** La stratégie peut être reconnue par des méthodes
qui laissent un objet imbriqué exécuter les tâches, ainsi que par le
setter qui permet de remplacer cet objet par un autre.
::::::::::::::

::: {#nav-tab .nav .nav-tabs .nav-justified role="tablist"}
[Exemple conceptuel](#example-0){#example-0-tab .nav-item .nav-link
.active toggle="tab" role="tab" aria-controls="example-0"
aria-selected="true"} [Analogie du monde
réel](#example-1){#example-1-tab .nav-item .nav-link toggle="tab"
role="tab" aria-controls="example-1" aria-selected="true"}
:::

::::: {#nav-tabContent .pattern-example-tab-content .tab-content}
::: {#example-0 .tab-pane .show .active role="tabpanel" aria-labelledby="example-0-tab" style="padding-top: 24px"}
## Exemple conceptuel {#example-0-title}

Dans cet exemple, nous allons voir la structure de la **Stratégie** et
répondre aux questions suivantes :

-   Que contiennent les classes ?
-   Quels rôles jouent-elles ?
-   Comment les éléments du patron sont-ils reliés ?

Après avoir étudié la structure du patron, vous pourrez plus facilement
comprendre l'exemple suivant qui est basé sur un cas d'utilisation réel
en PHP.

#### []{#example-0--index-php .anchor} **index.php:** Exemple conceptuel

<figure class="code">
<pre class="code" lang="php"><code>&lt;?php

namespace RefactoringGuru\Strategy\Conceptual;

/**
 * The Context defines the interface of interest to clients.
 */
class Context
{
    /**
     * @var Strategy The Context maintains a reference to one of the Strategy
     * objects. The Context does not know the concrete class of a strategy. It
     * should work with all strategies via the Strategy interface.
     */
    private $strategy;

    /**
     * Usually, the Context accepts a strategy through the constructor, but also
     * provides a setter to change it at runtime.
     */
    public function __construct(Strategy $strategy)
    {
        $this-&gt;strategy = $strategy;
    }

    /**
     * Usually, the Context allows replacing a Strategy object at runtime.
     */
    public function setStrategy(Strategy $strategy)
    {
        $this-&gt;strategy = $strategy;
    }

    /**
     * The Context delegates some work to the Strategy object instead of
     * implementing multiple versions of the algorithm on its own.
     */
    public function doSomeBusinessLogic(): void
    {
        // ...

        echo &quot;Context: Sorting data using the strategy (not sure how it&#39;ll do it)\n&quot;;
        $result = $this-&gt;strategy-&gt;doAlgorithm([&quot;a&quot;, &quot;b&quot;, &quot;c&quot;, &quot;d&quot;, &quot;e&quot;]);
        echo implode(&quot;,&quot;, $result) . &quot;\n&quot;;

        // ...
    }
}

/**
 * The Strategy interface declares operations common to all supported versions
 * of some algorithm.
 *
 * The Context uses this interface to call the algorithm defined by Concrete
 * Strategies.
 */
interface Strategy
{
    public function doAlgorithm(array $data): array;
}

/**
 * Concrete Strategies implement the algorithm while following the base Strategy
 * interface. The interface makes them interchangeable in the Context.
 */
class ConcreteStrategyA implements Strategy
{
    public function doAlgorithm(array $data): array
    {
        sort($data);

        return $data;
    }
}

class ConcreteStrategyB implements Strategy
{
    public function doAlgorithm(array $data): array
    {
        rsort($data);

        return $data;
    }
}

/**
 * The client code picks a concrete strategy and passes it to the context. The
 * client should be aware of the differences between strategies in order to make
 * the right choice.
 */
$context = new Context(new ConcreteStrategyA());
echo &quot;Client: Strategy is set to normal sorting.\n&quot;;
$context-&gt;doSomeBusinessLogic();

echo &quot;\n&quot;;

echo &quot;Client: Strategy is set to reverse sorting.\n&quot;;
$context-&gt;setStrategy(new ConcreteStrategyB());
$context-&gt;doSomeBusinessLogic();</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** Résultat de l'exécution

<figure class="code">
<pre class="code" lang="output"><code>Client: Strategy is set to normal sorting.
Context: Sorting data using the strategy (not sure how it&#39;ll do it)
a,b,c,d,e

Client: Strategy is set to reverse sorting.
Context: Sorting data using the strategy (not sure how it&#39;ll do it)
e,d,c,b,a</code></pre>
</figure>
:::

::: {#example-1 .tab-pane role="tabpanel" aria-labelledby="example-1-tab" style="padding-top: 24px"}
## Analogie du monde réel {#example-1-title}

Dans cet exemple, la **Stratégie** est utilisée pour représenter les
diverses méthodes de paiement pour une boutique en ligne.

Chaque méthode de paiement peut afficher un formulaire pour récupérer
les informations de paiement de l'utilisateur, puis les envoie à la
société qui s'en occupe. Une fois le paiement effectué, la société
redirige l'utilisateur vers notre site Web. La méthode de paiement
contrôle les paramètres de retour et en déduit si la commande a été
validée ou non.

#### []{#example-1--index-php .anchor} **index.php:** Exemple du monde réel

<figure class="code">
<pre class="code" lang="php"><code>&lt;?php

namespace RefactoringGuru\Strategy\RealWorld;

/**
 * This is the router and controller of our application. Upon receiving a
 * request, this class decides what behavior should be executed. When the app
 * receives a payment request, the OrderController class also decides which
 * payment method it should use to process the request. Thus, the class acts as
 * the Context and the Client at the same time.
 */
class OrderController
{
    /**
     * Handle POST requests.
     *
     * @param $url
     * @param $data
     * @throws \Exception
     */
    public function post(string $url, array $data)
    {
        echo &quot;Controller: POST request to $url with &quot; . json_encode($data) . &quot;\n&quot;;

        $path = parse_url($url, PHP_URL_PATH);

        if (preg_match(&#39;#^/orders?$#&#39;, $path, $matches)) {
            $this-&gt;postNewOrder($data);
        } else {
            echo &quot;Controller: 404 page\n&quot;;
        }
    }

    /**
     * Handle GET requests.
     *
     * @param $url
     * @throws \Exception
     */
    public function get(string $url): void
    {
        echo &quot;Controller: GET request to $url\n&quot;;

        $path = parse_url($url, PHP_URL_PATH);
        $query = parse_url($url, PHP_URL_QUERY);
        parse_str($query, $data);

        if (preg_match(&#39;#^/orders?$#&#39;, $path, $matches)) {
            $this-&gt;getAllOrders();
        } elseif (preg_match(&#39;#^/order/([0-9]+?)/payment/([a-z]+?)(/return)?$#&#39;, $path, $matches)) {
            $order = Order::get($matches[1]);

            // The payment method (strategy) is selected according to the value
            // passed along with the request.
            $paymentMethod = PaymentFactory::getPaymentMethod($matches[2]);

            if (!isset($matches[3])) {
                $this-&gt;getPayment($paymentMethod, $order, $data);
            } else {
                $this-&gt;getPaymentReturn($paymentMethod, $order, $data);
            }
        } else {
            echo &quot;Controller: 404 page\n&quot;;
        }
    }

    /**
     * POST /order {data}
     */
    public function postNewOrder(array $data): void
    {
        $order = new Order($data);
        echo &quot;Controller: Created the order #{$order-&gt;id}.\n&quot;;
    }

    /**
     * GET /orders
     */
    public function getAllOrders(): void
    {
        echo &quot;Controller: Here&#39;s all orders:\n&quot;;
        foreach (Order::get() as $order) {
            echo json_encode($order, JSON_PRETTY_PRINT) . &quot;\n&quot;;
        }
    }

    /**
     * GET /order/123/payment/XX
     */
    public function getPayment(PaymentMethod $method, Order $order, array $data): void
    {
        // The actual work is delegated to the payment method object.
        $form = $method-&gt;getPaymentForm($order);
        echo &quot;Controller: here&#39;s the payment form:\n&quot;;
        echo $form . &quot;\n&quot;;
    }

    /**
     * GET /order/123/payment/XXX/return?key=AJHKSJHJ3423&amp;success=true
     */
    public function getPaymentReturn(PaymentMethod $method, Order $order, array $data): void
    {
        try {
            // Another type of work delegated to the payment method.
            if ($method-&gt;validateReturn($order, $data)) {
                echo &quot;Controller: Thanks for your order!\n&quot;;
                $order-&gt;complete();
            }
        } catch (\Exception $e) {
            echo &quot;Controller: got an exception (&quot; . $e-&gt;getMessage() . &quot;)\n&quot;;
        }
    }
}

/**
 * A simplified representation of the Order class.
 */
class Order
{
    /**
     * For the sake of simplicity, we&#39;ll store all created orders here...
     *
     * @var array
     */
    private static $orders = [];

    /**
     * ...and access them from here.
     *
     * @param int $orderId
     * @return mixed
     */
    public static function get(int $orderId = null)
    {
        if ($orderId === null) {
            return static::$orders;
        } else {
            return static::$orders[$orderId];
        }
    }

    /**
     * The Order constructor assigns the values of the order&#39;s fields. To keep
     * things simple, there is no validation whatsoever.
     *
     * @param array $attributes
     */
    public function __construct(array $attributes)
    {
        $this-&gt;id = count(static::$orders);
        $this-&gt;status = &quot;new&quot;;
        foreach ($attributes as $key =&gt; $value) {
            $this-&gt;{$key} = $value;
        }
        static::$orders[$this-&gt;id] = $this;
    }

    /**
     * The method to call when an order gets paid.
     */
    public function complete(): void
    {
        $this-&gt;status = &quot;completed&quot;;
        echo &quot;Order: #{$this-&gt;id} is now {$this-&gt;status}.&quot;;
    }
}

/**
 * This class helps to produce a proper strategy object for handling a payment.
 */
class PaymentFactory
{
    /**
     * Get a payment method by its ID.
     *
     * @param $id
     * @return PaymentMethod
     * @throws \Exception
     */
    public static function getPaymentMethod(string $id): PaymentMethod
    {
        switch ($id) {
            case &quot;cc&quot;:
                return new CreditCardPayment();
            case &quot;paypal&quot;:
                return new PayPalPayment();
            default:
                throw new \Exception(&quot;Unknown Payment Method&quot;);
        }
    }
}

/**
 * The Strategy interface describes how a client can use various Concrete
 * Strategies.
 *
 * Note that in most examples you can find on the Web, strategies tend to do
 * some tiny thing within one method. However, in reality, your strategies can
 * be much more robust (by having several methods, for example).
 */
interface PaymentMethod
{
    public function getPaymentForm(Order $order): string;

    public function validateReturn(Order $order, array $data): bool;
}

/**
 * This Concrete Strategy provides a payment form and validates returns for
 * credit card payments.
 */
class CreditCardPayment implements PaymentMethod
{
    private static $store_secret_key = &quot;swordfish&quot;;

    public function getPaymentForm(Order $order): string
    {
        $returnURL = &quot;https://our-website.com/&quot; .
            &quot;order/{$order-&gt;id}/payment/cc/return&quot;;

        return &lt;&lt;&lt;FORM
&lt;form action=&quot;https://my-credit-card-processor.com/charge&quot; method=&quot;POST&quot;&gt;
    &lt;input type=&quot;hidden&quot; id=&quot;email&quot; value=&quot;{$order-&gt;email}&quot;&gt;
    &lt;input type=&quot;hidden&quot; id=&quot;total&quot; value=&quot;{$order-&gt;total}&quot;&gt;
    &lt;input type=&quot;hidden&quot; id=&quot;returnURL&quot; value=&quot;$returnURL&quot;&gt;
    &lt;input type=&quot;text&quot; id=&quot;cardholder-name&quot;&gt;
    &lt;input type=&quot;text&quot; id=&quot;credit-card&quot;&gt;
    &lt;input type=&quot;text&quot; id=&quot;expiration-date&quot;&gt;
    &lt;input type=&quot;text&quot; id=&quot;ccv-number&quot;&gt;
    &lt;input type=&quot;submit&quot; value=&quot;Pay&quot;&gt;
&lt;/form&gt;
FORM;
    }

    public function validateReturn(Order $order, array $data): bool
    {
        echo &quot;CreditCardPayment: ...validating... &quot;;

        if ($data[&#39;key&#39;] != md5($order-&gt;id . static::$store_secret_key)) {
            throw new \Exception(&quot;Payment key is wrong.&quot;);
        }

        if (!isset($data[&#39;success&#39;]) || !$data[&#39;success&#39;] || $data[&#39;success&#39;] == &#39;false&#39;) {
            throw new \Exception(&quot;Payment failed.&quot;);
        }

        // ...

        if (floatval($data[&#39;total&#39;]) &lt; $order-&gt;total) {
            throw new \Exception(&quot;Payment amount is wrong.&quot;);
        }

        echo &quot;Done!\n&quot;;

        return true;
    }
}

/**
 * This Concrete Strategy provides a payment form and validates returns for
 * PayPal payments.
 */
class PayPalPayment implements PaymentMethod
{
    public function getPaymentForm(Order $order): string
    {
        $returnURL = &quot;https://our-website.com/&quot; .
            &quot;order/{$order-&gt;id}/payment/paypal/return&quot;;

        return &lt;&lt;&lt;FORM
&lt;form action=&quot;https://paypal.com/payment&quot; method=&quot;POST&quot;&gt;
    &lt;input type=&quot;hidden&quot; id=&quot;email&quot; value=&quot;{$order-&gt;email}&quot;&gt;
    &lt;input type=&quot;hidden&quot; id=&quot;total&quot; value=&quot;{$order-&gt;total}&quot;&gt;
    &lt;input type=&quot;hidden&quot; id=&quot;returnURL&quot; value=&quot;$returnURL&quot;&gt;
    &lt;input type=&quot;submit&quot; value=&quot;Pay on PayPal&quot;&gt;
&lt;/form&gt;
FORM;
    }

    public function validateReturn(Order $order, array $data): bool
    {
        echo &quot;PayPalPayment: ...validating... &quot;;

        // ...

        echo &quot;Done!\n&quot;;

        return true;
    }
}

/**
 * The client code.
 */

$controller = new OrderController();

echo &quot;Client: Let&#39;s create some orders\n&quot;;

$controller-&gt;post(&quot;/orders&quot;, [
    &quot;email&quot; =&gt; &quot;me@example.com&quot;,
    &quot;product&quot; =&gt; &quot;ABC Cat food (XL)&quot;,
    &quot;total&quot; =&gt; 9.95,
]);

$controller-&gt;post(&quot;/orders&quot;, [
    &quot;email&quot; =&gt; &quot;me@example.com&quot;,
    &quot;product&quot; =&gt; &quot;XYZ Cat litter (XXL)&quot;,
    &quot;total&quot; =&gt; 19.95,
]);

echo &quot;\nClient: List my orders, please\n&quot;;

$controller-&gt;get(&quot;/orders&quot;);

echo &quot;\nClient: I&#39;d like to pay for the second, show me the payment form\n&quot;;

$controller-&gt;get(&quot;/order/1/payment/paypal&quot;);

echo &quot;\nClient: ...pushes the Pay button...\n&quot;;
echo &quot;\nClient: Oh, I&#39;m redirected to the PayPal.\n&quot;;
echo &quot;\nClient: ...pays on the PayPal...\n&quot;;
echo &quot;\nClient: Alright, I&#39;m back with you, guys.\n&quot;;

$controller-&gt;get(&quot;/order/1/payment/paypal/return&quot; .
    &quot;?key=c55a3964833a4b0fa4469ea94a057152&amp;success=true&amp;total=19.95&quot;);</code></pre>
</figure>

#### []{#example-1--Output-txt .anchor} **Output.txt:** Résultat de l'exécution

<figure class="code">
<pre class="code" lang="output"><code>Client: Let&#39;s create some orders
Controller: POST request to /orders with {&quot;email&quot;:&quot;me@example.com&quot;,&quot;product&quot;:&quot;ABC Cat food (XL)&quot;,&quot;total&quot;:9.95}
Controller: Created the order #0.
Controller: POST request to /orders with {&quot;email&quot;:&quot;me@example.com&quot;,&quot;product&quot;:&quot;XYZ Cat litter (XXL)&quot;,&quot;total&quot;:19.95}
Controller: Created the order #1.

Client: List my orders, please
Controller: GET request to /orders
Controller: Here&#39;s all orders:
{
    &quot;id&quot;: 0,
    &quot;status&quot;: &quot;new&quot;,
    &quot;email&quot;: &quot;me@example.com&quot;,
    &quot;product&quot;: &quot;ABC Cat food (XL)&quot;,
    &quot;total&quot;: 9.95
}
{
    &quot;id&quot;: 1,
    &quot;status&quot;: &quot;new&quot;,
    &quot;email&quot;: &quot;me@example.com&quot;,
    &quot;product&quot;: &quot;XYZ Cat litter (XXL)&quot;,
    &quot;total&quot;: 19.95
}

Client: I&#39;d like to pay for the second, show me the payment form
Controller: GET request to /order/1/payment/paypal
Controller: here&#39;s the payment form:
&lt;form action=&quot;https://paypal.com/payment&quot; method=&quot;POST&quot;&gt;
    &lt;input type=&quot;hidden&quot; id=&quot;email&quot; value=&quot;me@example.com&quot;&gt;
    &lt;input type=&quot;hidden&quot; id=&quot;total&quot; value=&quot;19.95&quot;&gt;
    &lt;input type=&quot;hidden&quot; id=&quot;returnURL&quot; value=&quot;https://our-website.com/order/1/payment/paypal/return&quot;&gt;
    &lt;input type=&quot;submit&quot; value=&quot;Pay on PayPal&quot;&gt;
&lt;/form&gt;

Client: ...pushes the Pay button...

Client: Oh, I&#39;m redirected to the PayPal.

Client: ...pays on the PayPal...

Client: Alright, I&#39;m back with you, guys.
Controller: GET request to /order/1/payment/paypal/return?key=c55a3964833a4b0fa4469ea94a057152&amp;success=true&amp;total=19.95
PayPalPayment: ...validating... Done!
Controller: Thanks for your order!
Order: #1 is now completed.</code></pre>
</figure>
:::
:::::

::: {#nav-tab .nav .nav-tabs .nav-tabs-bottom .nav-justified role="tablist"}
[Exemple conceptuel](#example-0){#example-0-tab-bottom .nav-item
.nav-link .active toggle="tab" role="tab" aria-controls="example-0"
aria-selected="true"} [Analogie du monde
réel](#example-1){#example-1-tab-bottom .nav-item .nav-link toggle="tab"
role="tab" aria-controls="example-1" aria-selected="true"}
:::

::: next
#### Suivant

[Patron de méthode en PHP []{.fa
.fa-arrow-right}](../../template-method/php/example.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Retour

[[]{.fa .fa-arrow-left} État en PHP](../../state/php/example.html){.btn
.btn-default rel="prev"}
:::

## **Stratégie** dans les autres langues

[![Stratégie en
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Stratégie en C#"){.prog-lang-link}
[![Stratégie en
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Stratégie en C++"){.prog-lang-link}
[![Stratégie en
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Stratégie en Go"){.prog-lang-link}
[![Stratégie en
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Stratégie en Java"){.prog-lang-link}
[![Stratégie en
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Stratégie en Python"){.prog-lang-link}
[![Stratégie en
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Stratégie en Ruby"){.prog-lang-link}
[![Stratégie en
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Stratégie en Rust"){.prog-lang-link}
[![Stratégie en
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Stratégie en Swift"){.prog-lang-link}
[![Stratégie en
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Stratégie en TypeScript"){.prog-lang-link}
:::::::::::::::::::::::::

::::::: {.prom .banner-sidebar .banner .banner-striped .banner-examples .banner-removable .banner-removable-patterns data-id="DP: Examples-IDE" creative-id="examples-sidebar-fr" position="sidebar"}
:::::: {.banner-inner style="font-size: 14px; line-height: 21px;"}
::: banner-examples-img
[![](../../../../images/patterns/banners/examples-ide91ca.png?id=3115b4b548fb96b75974e2de8f4f49bc){width="215"
height="158" loading="lazy"
srcset="/images/patterns/banners/examples-ide-2x.png?id=93c007a6d157b97c28eb213f59d716ae 2x"}](../../book.html)
:::

::: banner-examples-fade
[ Archive avec exemples](../../book.html){.btn .btn-secondary
.btn-block}
:::

::: {.banner-examples-text style="margin-top: 1rem;"}
Achetez l'ebook **Plongée au cœur des patrons de conception** et obtenez
l'accès à une archive comprenant des dizaines d'exemples détaillés qui
peuvent être ouverts dans votre environnement de développement.

[ En savoir plus...](../../book.html){.btn .btn-secondary .btn-block}
:::
::::::
:::::::
:::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::
