:::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Wzorce
projektowe](../../../design-patterns.html){.type} /
[Strategia](../../strategy.html){.type} / [PHP](../../php.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Strategia](../../../../images/patterns/cards/strategy-mini28f6.png?id=d38abee4fb6f2aed909d262bdadca936){srcset="/images/patterns/cards/strategy-mini-2x.png?id=f4e6608561f8e5d18be6927d4620ad29 2x"}
:::

# **Strategia** w języku PHP {#strategia-w-języku-php .pattern-example-title-block-title}
::::

:::::::::::::: pattern-example-body
::: pattern-example-brief
**Strategia** to behawioralny wzorzec projektowy zakładający
przekształcenie zestawu zachowań w obiekty, które można stosować
zamiennie w pierwotnym obiekcie.

Pierwotny obiekt, zwany kontekstem, przechowuje odniesienie do
obiektu-strategii i deleguje mu działania związane z danym zachowaniem.
Aby zmienić sposób, w jaki kontekst wykonuje swą pracę, należy zamienić
bieżąco przypisany obiekt strategii na inny.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Dowiedz się więcej o Strategia ](../../strategy.html){.btn .btn-lg
.btn-primary}
:::

::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
Nawigacja
:::

::: en-intro
 [Intro](#)
:::

::: en-intro
 [Przykład koncepcyjny](#example-0)
:::

::: en-file
 [index](#example-0--index-php)
:::

::: en-file
 [Output](#example-0--Output-txt)
:::

::: en-intro
 [Przykład z prawdziwego życia](#example-1)
:::

::: en-file
 [index](#example-1--index-php)
:::

::: en-file
 [Output](#example-1--Output-txt)
:::
:::::::::::

**Złożoność:** []{.fa-stars}

**Popularność:** []{.fa-stars}

**Przykłady użycia:** Wzorzec Strategia jest często stosowany w kodzie
PHP, szczególnie gdy istnieje potrzeba zamiany algorytmu w trakcie
działania programu. Wzorzec ten jednak konkuruje z funkcjami
anonimowymi, obsługiwanymi przez PHP od 2009 roku.

**Identyfikacja:** Wzorzec Strategia można rozpoznać po obecności metody
pozwalającej wykonywać faktyczną pracę zagnieżdżonemu obiektowi oraz po
obecności settera umożliwiającego wymianę tego obiektu na inny.
::::::::::::::

::: {#nav-tab .nav .nav-tabs .nav-justified role="tablist"}
[Przykład koncepcyjny](#example-0){#example-0-tab .nav-item .nav-link
.active toggle="tab" role="tab" aria-controls="example-0"
aria-selected="true"} [Przykład z prawdziwego
życia](#example-1){#example-1-tab .nav-item .nav-link toggle="tab"
role="tab" aria-controls="example-1" aria-selected="true"}
:::

::::: {#nav-tabContent .pattern-example-tab-content .tab-content}
::: {#example-0 .tab-pane .show .active role="tabpanel" aria-labelledby="example-0-tab" style="padding-top: 24px"}
## Przykład koncepcyjny {#example-0-title}

Poniższy przykład ilustruje strukturę wzorca **Strategia** ze
szczególnym naciskiem na następujące kwestie:

-   Z jakich składa się klas?
-   Jakie role pełnią te klasy?
-   W jaki sposób elementy wzorca są ze sobą powiązane?

Poznawszy strukturę wzorca będzie ci łatwiej zrozumieć następujący
przykład, oparty na prawdziwym przypadku użycia PHP.

#### []{#example-0--index-php .anchor} **index.php:** Przykład koncepcyjny

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

#### []{#example-0--Output-txt .anchor} **Output.txt:** Wynik działania

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
## Przykład z prawdziwego życia {#example-1-title}

W tym przykładzie, wzorzec **Strategia** reprezentuje metody płatności w
aplikacji e-commerce.

Każda z metod płatności może wyświetlać formularz służący zebraniu od
użytkownika szczegółów płatności i przesłaniu ich bramce płatniczej.
Następnie, po przekierowaniu nas z powrotem na naszą stronę, metoda
płatności waliduje zwrócone przez bramkę parametry i pomaga określić czy
udało się zrealizować zamówienie.

#### []{#example-1--index-php .anchor} **index.php:** Przykład z prawdziwego życia

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

#### []{#example-1--Output-txt .anchor} **Output.txt:** Wynik działania

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
[Przykład koncepcyjny](#example-0){#example-0-tab-bottom .nav-item
.nav-link .active toggle="tab" role="tab" aria-controls="example-0"
aria-selected="true"} [Przykład z prawdziwego
życia](#example-1){#example-1-tab-bottom .nav-item .nav-link
toggle="tab" role="tab" aria-controls="example-1" aria-selected="true"}
:::

::: next
#### Czytaj dalej

[Metoda szablonowa w języku PHP []{.fa
.fa-arrow-right}](../../template-method/php/example.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Wróć

[[]{.fa .fa-arrow-left} Stan w języku
PHP](../../state/php/example.html){.btn .btn-default rel="prev"}
:::

## **Strategia** w innych językach

[![Strategia w języku
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Strategia w języku C#"){.prog-lang-link}
[![Strategia w języku
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Strategia w języku C++"){.prog-lang-link}
[![Strategia w języku
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Strategia w języku Go"){.prog-lang-link}
[![Strategia w języku
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Strategia w języku Java"){.prog-lang-link}
[![Strategia w języku
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Strategia w języku Python"){.prog-lang-link}
[![Strategia w języku
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Strategia w języku Ruby"){.prog-lang-link}
[![Strategia w języku
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Strategia w języku Rust"){.prog-lang-link}
[![Strategia w języku
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Strategia w języku Swift"){.prog-lang-link}
[![Strategia w języku
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Strategia w języku TypeScript"){.prog-lang-link}
:::::::::::::::::::::::::

::::::: {.prom .banner-sidebar .banner .banner-striped .banner-examples .banner-removable .banner-removable-patterns data-id="DP: Examples-IDE" creative-id="examples-sidebar-pl" position="sidebar"}
:::::: {.banner-inner style="font-size: 14px; line-height: 21px;"}
::: banner-examples-img
[![](../../../../images/patterns/banners/examples-ide91ca.png?id=3115b4b548fb96b75974e2de8f4f49bc){width="215"
height="158" loading="lazy"
srcset="/images/patterns/banners/examples-ide-2x.png?id=93c007a6d157b97c28eb213f59d716ae 2x"}](../../book.html)
:::

::: banner-examples-fade
[ Archiwum\
przykładów kodu](../../book.html){.btn .btn-secondary .btn-block}
:::

::: {.banner-examples-text style="margin-top: 1rem;"}
Kupując eBook **Wzorce projektowe. Nowoczesny podręcznik** zyskujesz
dostęp do archiwum zawierającego mnóstwo szczegółowych przykładów, które
można od razu wkleić do swojego preferowanego zintegrowanego środowiska
programistycznego.

[ Dowiedz się więcej...](../../book.html){.btn .btn-secondary
.btn-block}
:::
::::::
:::::::
:::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::
