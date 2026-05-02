:::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Паттерны
проектирования](../../../design-patterns.html){.type} /
[Стратегия](../../strategy.html){.type} / [PHP](../../php.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Стратегия](../../../../images/patterns/cards/strategy-mini28f6.png?id=d38abee4fb6f2aed909d262bdadca936){srcset="/images/patterns/cards/strategy-mini-2x.png?id=f4e6608561f8e5d18be6927d4620ad29 2x"}
:::

# **Стратегия** на PHP {#стратегия-на-php .pattern-example-title-block-title}
::::

:::::::::::::: pattern-example-body
::: pattern-example-brief
**Стратегия** --- это поведенческий паттерн, выносит набор алгоритмов в
собственные классы и делает их взаимозаменимыми.

Другие объекты содержат ссылку на объект-стратегию и делегируют ей
работу. Программа может подменить этот объект другим, если требуется
иной способ решения задачи.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Подробней о паттерне Стратегия ](../../strategy.html){.btn .btn-lg
.btn-primary}
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

**Применимость:** Стратегию часто используют в PHP-коде, особенно там,
где нужно подменять алгоритм во время выполнения программы. Но у
паттерна есть довольно сильный конкурент в лице анонимных функций,
которые PHP уже довольно давно поддерживает.

**Признаки применения паттерна:** Класс делегирует выполнение вложенному
объекту абстрактного типа или интерфейса.
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

Этот пример показывает структуру паттерна **Стратегия**, а именно --- из
каких классов он состоит, какие роли эти классы выполняют и как они
взаимодействуют друг с другом.

После ознакомления со структурой, вам будет легче воспринимать второй
пример, который рассматривает реальный случай использования паттерна в
мире PHP.

#### []{#example-0--index-php .anchor} **index.php:** Пример структуры паттерна

<figure class="code">
<pre class="code" lang="php"><code>&lt;?php

namespace RefactoringGuru\Strategy\Conceptual;

/**
 * Контекст определяет интерфейс, представляющий интерес для клиентов.
 */
class Context
{
    /**
     * @var Strategy Контекст хранит ссылку на один из объектов Стратегии.
     * Контекст не знает конкретного класса стратегии. Он должен работать со
     * всеми стратегиями через интерфейс Стратегии.
     */
    private $strategy;

    /**
     * Обычно Контекст принимает стратегию через конструктор, а также
     * предоставляет сеттер для её изменения во время выполнения.
     */
    public function __construct(Strategy $strategy)
    {
        $this-&gt;strategy = $strategy;
    }

    /**
     * Обычно Контекст позволяет заменить объект Стратегии во время выполнения.
     */
    public function setStrategy(Strategy $strategy)
    {
        $this-&gt;strategy = $strategy;
    }

    /**
     * Вместо того, чтобы самостоятельно реализовывать множественные версии
     * алгоритма, Контекст делегирует некоторую работу объекту Стратегии.
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
 * Интерфейс Стратегии объявляет операции, общие для всех поддерживаемых версий
 * некоторого алгоритма.
 *
 * Контекст использует этот интерфейс для вызова алгоритма, определённого
 * Конкретными Стратегиями.
 */
interface Strategy
{
    public function doAlgorithm(array $data): array;
}

/**
 * Конкретные Стратегии реализуют алгоритм, следуя базовому интерфейсу
 * Стратегии. Этот интерфейс делает их взаимозаменяемыми в Контексте.
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
 * Клиентский код выбирает конкретную стратегию и передаёт её в контекст. Клиент
 * должен знать о различиях между стратегиями, чтобы сделать правильный выбор.
 */
$context = new Context(new ConcreteStrategyA());
echo &quot;Client: Strategy is set to normal sorting.\n&quot;;
$context-&gt;doSomeBusinessLogic();

echo &quot;\n&quot;;

echo &quot;Client: Strategy is set to reverse sorting.\n&quot;;
$context-&gt;setStrategy(new ConcreteStrategyB());
$context-&gt;doSomeBusinessLogic();</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** Результат выполнения

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
## Пример из реальной жизни {#example-1-title}

В этом примере паттерн **Стратегия** используется для представления
способов оплаты в приложении электронной коммерции.

Каждый способ оплаты может отображать форму оплаты для сбора надлежащих
платёжных реквизитов пользователя и отправки его в компанию по обработке
платежей. После того, как компания по обработке платежей перенаправляет
пользователя обратно на сайт, метод оплаты проверяет возвращаемые
параметры и помогает решить, был ли заказ завершён.

#### []{#example-1--index-php .anchor} **index.php:** Пример из реальной жизни

<figure class="code">
<pre class="code" lang="php"><code>&lt;?php

namespace RefactoringGuru\Strategy\RealWorld;

/**
 * Это роутер и контроллер нашего приложения. Получив запрос, этот класс решает,
 * какое поведение должно выполняться. Когда приложение получает требование об
 * оплате, класс OrderController также решает, какой способ оплаты следует
 * использовать для его обработки. Таким образом, этот класс действует как
 * Контекст и в то же время как Клиент.
 */
class OrderController
{
    /**
     * Обрабатываем запросы POST.
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
     * Обрабатываем запросы GET.
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

            // Способ оплаты (стратегия) выбирается в соответствии со значением,
            // переданным в запросе.
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
        // Фактическая работа делегируется объекту метода оплаты.
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
            // Другой тип работы, делегированный методу оплаты.
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
 * Упрощенное представление класса Заказа.
 */
class Order
{
    /**
     * Для простоты, мы будем хранить все созданные заказы здесь...
     */
    private static $orders = [];

    /**
     * ...и получать к ним доступ отсюда.
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
     * Конструктор Заказа присваивает значения полям заказа. Чтобы всё было
     * просто, нет никакой проверки.
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
     * Метод позвонить при оплате заказа.
     */
    public function complete(): void
    {
        $this-&gt;status = &quot;completed&quot;;
        echo &quot;Order: #{$this-&gt;id} is now {$this-&gt;status}.&quot;;
    }
}

/**
 * Этот класс помогает создать правильный объект стратегии для обработки
 * платежа.
 */
class PaymentFactory
{
    /**
     * Получаем способ оплаты по его ID.
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
 * Интерфейс Стратегии описывает, как клиент может использовать различные
 * Конкретные Стратегии.
 *
 * Обратите внимание, что в большинстве примеров, которые можно найти в
 * интернете, стратегии чаще всего делают какую-нибудь мелочь в рамках одного
 * метода.
 */
interface PaymentMethod
{
    public function getPaymentForm(Order $order): string;

    public function validateReturn(Order $order, array $data): bool;
}

/**
 * Эта Конкретная Стратегия предоставляет форму оплаты и проверяет результаты
 * платежей кредитными картам.
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
 * Эта Конкретная Стратегия предоставляет форму оплаты и проверяет результаты
 * платежей PayPal.
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
 * Клиентский код.
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

#### []{#example-1--Output-txt .anchor} **Output.txt:** Результат выполнения

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
[Концептуальный пример](#example-0){#example-0-tab-bottom .nav-item
.nav-link .active toggle="tab" role="tab" aria-controls="example-0"
aria-selected="true"} [Пример из реальной
жизни](#example-1){#example-1-tab-bottom .nav-item .nav-link
toggle="tab" role="tab" aria-controls="example-1" aria-selected="true"}
:::

::: next
#### Читаем дальше

[Шаблонный метод на PHP []{.fa
.fa-arrow-right}](../../template-method/php/example.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Состояние на
PHP](../../state/php/example.html){.btn .btn-default rel="prev"}
:::

## **Стратегия** на других языках программирования

[![Стратегия на
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Стратегия на C#"){.prog-lang-link}
[![Стратегия на
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Стратегия на C++"){.prog-lang-link}
[![Стратегия на
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Стратегия на Go"){.prog-lang-link}
[![Стратегия на
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Стратегия на Java"){.prog-lang-link}
[![Стратегия на
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Стратегия на Python"){.prog-lang-link}
[![Стратегия на
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Стратегия на Ruby"){.prog-lang-link}
[![Стратегия на
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Стратегия на Rust"){.prog-lang-link}
[![Стратегия на
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Стратегия на Swift"){.prog-lang-link}
[![Стратегия на
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Стратегия на TypeScript"){.prog-lang-link}
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
