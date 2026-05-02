:::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Паттерны
проектирования](../../../design-patterns.html){.type} /
[Стратегия](../../strategy.html){.type} / [Java](../../java.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Стратегия](../../../../images/patterns/cards/strategy-mini28f6.png?id=d38abee4fb6f2aed909d262bdadca936){srcset="/images/patterns/cards/strategy-mini-2x.png?id=f4e6608561f8e5d18be6927d4620ad29 2x"}
:::

# **Стратегия** на Java {#стратегия-на-java .pattern-example-title-block-title}
::::

::::::::::::::::::::::: pattern-example-body
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

:::::::::::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
Навигация
:::

::: en-intro
 [Интро](#)
:::

::: en-intro
 [Методы оплаты в интернет магазине](#example-0)
:::

::: en-file
 strategies
:::

:::: en-inside
::: en-file
  [Pay­Strategy](#example-0--strategies-PayStrategy-java)
:::
::::

:::: en-inside
::: en-file
  [Pay­By­Pay­Pal](#example-0--strategies-PayByPayPal-java)
:::
::::

:::: en-inside
::: en-file
  [Pay­By­Credit­Card](#example-0--strategies-PayByCreditCard-java)
:::
::::

:::: en-inside
::: en-file
  [Credit­Card](#example-0--strategies-CreditCard-java)
:::
::::

::: en-file
 order
:::

:::: en-inside
::: en-file
  [Order](#example-0--order-Order-java)
:::
::::

::: en-file
 [Demo](#example-0--Demo-java)
:::

::: en-file
 [Output­Demo](#example-0--OutputDemo-txt)
:::
::::::::::::::::::::

**Сложность:** []{.fa-stars}

**Популярность:** []{.fa-stars}

**Применимость:** Стратегия часто используется в Java-коде, особенно
там, где нужно подменять алгоритм во время выполнения программы. Начиная
с Java 8, многие примеры стратегии можно заменить простыми
lambda-выражениями.

Примеры Стратегии в стандартных библиотеках Java:

-   [`java.util.Comparator#compare()`](http://docs.oracle.com/javase/8/docs/api/java/util/Comparator.html#compare-T-T-),
    вызываемые из `Collections#sort()`.

-   [`javax.servlet.http.HttpServlet`](http://docs.oracle.com/javaee/7/api/javax/servlet/http/HttpServlet.html):
    метод `service()`, а также все методы `doXXX()` принимают объекты
    `HttpServletRequest` и `HttpServletResponse` в параметрах.

-   [`javax.servlet.Filter#doFilter()`](http://docs.oracle.com/javaee/7/api/javax/servlet/Filter.html#doFilter-javax.servlet.ServletRequest-javax.servlet.ServletResponse-javax.servlet.FilterChain-)

**Признаки применения паттерна:** Класс делегирует выполнение вложенному
объекту абстрактного типа или интерфейса.
:::::::::::::::::::::::

[]{#example-0}

## Методы оплаты в интернет магазине {#example-0-title}

В этом примере Стратегия реализует выбор платёжного метода в интернет
магазине. Когда пользователь сформировал заказ, он получает выбор из
нескольких платёжных стредств: электронного кошелька или кредитной
карты.

В данном случае конкретные стратегии платёжных методов не только
проводят саму оплату, но и собирают необходимые данные на форме заказа.

### []{#example-0--strategies .anchor} **strategies** {#strategies .example-title}

#### []{#example-0--strategies-PayStrategy-java .anchor} **strategies/PayStrategy.java:** Общий интерфейс стратегий оплаты

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.strategy.example.strategies;

/**
 * Общий интерфейс всех стратегий.
 */
public interface PayStrategy {
    boolean pay(int paymentAmount);
    void collectPaymentDetails();
}</code></pre>
</figure>

#### []{#example-0--strategies-PayByPayPal-java .anchor} **strategies/PayByPayPal.java:** Оплата через PayPal

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.strategy.example.strategies;

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.util.HashMap;
import java.util.Map;

/**
 * Конкретная стратегия. Реализует оплату корзины интернет магазина через
 * платежную систему PayPal.
 */
public class PayByPayPal implements PayStrategy {
    private static final Map&lt;String, String&gt; DATA_BASE = new HashMap&lt;&gt;();
    private final BufferedReader READER = new BufferedReader(new InputStreamReader(System.in));
    private String email;
    private String password;
    private boolean signedIn;

    static {
        DATA_BASE.put(&quot;amanda1985&quot;, &quot;amanda@ya.com&quot;);
        DATA_BASE.put(&quot;qwerty&quot;, &quot;john@amazon.eu&quot;);
    }

    /**
     * Собираем данные от клиента.
     */
    @Override
    public void collectPaymentDetails() {
        try {
            while (!signedIn) {
                System.out.print(&quot;Enter the user&#39;s email: &quot;);
                email = READER.readLine();
                System.out.print(&quot;Enter the password: &quot;);
                password = READER.readLine();
                if (verify()) {
                    System.out.println(&quot;Data verification has been successful.&quot;);
                } else {
                    System.out.println(&quot;Wrong email or password!&quot;);
                }
            }
        } catch (IOException ex) {
            ex.printStackTrace();
        }
    }

    private boolean verify() {
        setSignedIn(email.equals(DATA_BASE.get(password)));
        return signedIn;
    }

    /**
     * Если клиент уже вошел в систему, то для следующей оплаты данные вводить
     * не придется.
     */
    @Override
    public boolean pay(int paymentAmount) {
        if (signedIn) {
            System.out.println(&quot;Paying &quot; + paymentAmount + &quot; using PayPal.&quot;);
            return true;
        } else {
            return false;
        }
    }

    private void setSignedIn(boolean signedIn) {
        this.signedIn = signedIn;
    }
}</code></pre>
</figure>

#### []{#example-0--strategies-PayByCreditCard-java .anchor} **strategies/PayByCreditCard.java:** Оплата кредиткой

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.strategy.example.strategies;

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;

/**
 * Конкретная стратегия. Реализует оплату корзины интернет магазина кредитной
 * картой клиента.
 */
public class PayByCreditCard implements PayStrategy {
    private final BufferedReader READER = new BufferedReader(new InputStreamReader(System.in));
    private CreditCard card;

    /**
     * Собираем данные карты клиента.
     */
    @Override
    public void collectPaymentDetails() {
        try {
            System.out.print(&quot;Enter the card number: &quot;);
            String number = READER.readLine();
            System.out.print(&quot;Enter the card expiration date &#39;mm/yy&#39;: &quot;);
            String date = READER.readLine();
            System.out.print(&quot;Enter the CVV code: &quot;);
            String cvv = READER.readLine();
            card = new CreditCard(number, date, cvv);

            // Валидируем номер карты...

        } catch (IOException ex) {
            ex.printStackTrace();
        }
    }

    /**
     * После проверки карты мы можем совершить оплату. Если клиент продолжает
     * покупки, мы не запрашиваем карту заново.
     */
    @Override
    public boolean pay(int paymentAmount) {
        if (cardIsPresent()) {
            System.out.println(&quot;Paying &quot; + paymentAmount + &quot; using Credit Card.&quot;);
            card.setAmount(card.getAmount() - paymentAmount);
            return true;
        } else {
            return false;
        }
    }

    private boolean cardIsPresent() {
        return card != null;
    }
}</code></pre>
</figure>

#### []{#example-0--strategies-CreditCard-java .anchor} **strategies/CreditCard.java:** Кредитная карта

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.strategy.example.strategies;

/**
 * Очень наивная реализация кредитной карты.
 */
public class CreditCard {
    private int amount;
    private String number;
    private String date;
    private String cvv;

    CreditCard(String number, String date, String cvv) {
        this.amount = 100_000;
        this.number = number;
        this.date = date;
        this.cvv = cvv;
    }

    public void setAmount(int amount) {
        this.amount = amount;
    }

    public int getAmount() {
        return amount;
    }
}</code></pre>
</figure>

### []{#example-0--order .anchor} **order** {#order .example-title}

#### []{#example-0--order-Order-java .anchor} **order/Order.java:** Класс заказа

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.strategy.example.order;

import refactoring_guru.strategy.example.strategies.PayStrategy;

/**
 * Класс заказа. Ничего не знает о том каким способом (стратегией) будет
 * расчитыватся клиент, а просто вызывает метод оплаты. Все остальное стратегия
 * делает сама.
 */
public class Order {
    private int totalCost = 0;
    private boolean isClosed = false;

    public void processOrder(PayStrategy strategy) {
        strategy.collectPaymentDetails();
        // Здесь мы могли бы забрать и сохранить платежные данные из стратегии.
    }

    public void setTotalCost(int cost) {
        this.totalCost += cost;
    }

    public int getTotalCost() {
        return totalCost;
    }

    public boolean isClosed() {
        return isClosed;
    }

    public void setClosed() {
        isClosed = true;
    }
}</code></pre>
</figure>

#### []{#example-0--Demo-java .anchor} **Demo.java:** Клиентский код

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.strategy.example;

import refactoring_guru.strategy.example.order.Order;
import refactoring_guru.strategy.example.strategies.PayByCreditCard;
import refactoring_guru.strategy.example.strategies.PayByPayPal;
import refactoring_guru.strategy.example.strategies.PayStrategy;

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.util.HashMap;
import java.util.Map;

/**
 * Первый в мире консольный интерет магазин.
 */
public class Demo {
    private static Map&lt;Integer, Integer&gt; priceOnProducts = new HashMap&lt;&gt;();
    private static BufferedReader reader = new BufferedReader(new InputStreamReader(System.in));
    private static Order order = new Order();
    private static PayStrategy strategy;

    static {
        priceOnProducts.put(1, 2200);
        priceOnProducts.put(2, 1850);
        priceOnProducts.put(3, 1100);
        priceOnProducts.put(4, 890);
    }

    public static void main(String[] args) throws IOException {
        while (!order.isClosed()) {
            int cost;

            String continueChoice;
            do {
                System.out.print(&quot;Please, select a product:&quot; + &quot;\n&quot; +
                        &quot;1 - Mother board&quot; + &quot;\n&quot; +
                        &quot;2 - CPU&quot; + &quot;\n&quot; +
                        &quot;3 - HDD&quot; + &quot;\n&quot; +
                        &quot;4 - Memory&quot; + &quot;\n&quot;);
                int choice = Integer.parseInt(reader.readLine());
                cost = priceOnProducts.get(choice);
                System.out.print(&quot;Count: &quot;);
                int count = Integer.parseInt(reader.readLine());
                order.setTotalCost(cost * count);
                System.out.print(&quot;Do you wish to continue selecting products? Y/N: &quot;);
                continueChoice = reader.readLine();
            } while (continueChoice.equalsIgnoreCase(&quot;Y&quot;));

            if (strategy == null) {
                System.out.println(&quot;Please, select a payment method:&quot; + &quot;\n&quot; +
                        &quot;1 - PalPay&quot; + &quot;\n&quot; +
                        &quot;2 - Credit Card&quot;);
                String paymentMethod = reader.readLine();

                // Клиент создаёт различные стратегии на основании
                // пользовательских данных, конфигурации и прочих параметров.
                if (paymentMethod.equals(&quot;1&quot;)) {
                    strategy = new PayByPayPal();
                } else {
                    strategy = new PayByCreditCard();
                }
            }

            // Объект заказа делегирует сбор платёжных данны стратегии, т.к.
            // только стратегии знают какие данные им нужны для приёма оплаты.
            order.processOrder(strategy);

            System.out.print(&quot;Pay &quot; + order.getTotalCost() + &quot; units or Continue shopping? P/C: &quot;);
            String proceed = reader.readLine();
            if (proceed.equalsIgnoreCase(&quot;P&quot;)) {
                // И наконец, стратегия запускает приём платежа.
                if (strategy.pay(order.getTotalCost())) {
                    System.out.println(&quot;Payment has been successful.&quot;);
                } else {
                    System.out.println(&quot;FAIL! Please, check your data.&quot;);
                }
                order.setClosed();
            }
        }
    }
}</code></pre>
</figure>

#### []{#example-0--OutputDemo-txt .anchor} **OutputDemo.txt:** Результат выполнения

<figure class="code">
<pre class="code" lang="output"><code>Please, select a product:
1 - Mother board
2 - CPU
3 - HDD
4 - Memory
1
Count: 2
Do you wish to continue selecting products? Y/N: y
Please, select a product:
1 - Mother board
2 - CPU
3 - HDD
4 - Memory
2
Count: 1
Do you wish to continue selecting products? Y/N: n
Please, select a payment method:
1 - PalPay
2 - Credit Card
1
Enter the user&#39;s email: user@example.com
Enter the password: qwerty
Wrong email or password!
Enter user email: amanda@ya.com
Enter password: amanda1985
Data verification has been successful.
Pay 6250 units or Continue shopping?  P/C: p
Paying 6250 using PayPal.
Payment has been successful.</code></pre>
</figure>

::: next
#### Читаем дальше

[Шаблонный метод на Java []{.fa
.fa-arrow-right}](../../template-method/java/example.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Состояние на
Java](../../state/java/example.html){.btn .btn-default rel="prev"}
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
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Стратегия на PHP"){.prog-lang-link}
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
:::::::::::::::::::::::::::::

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
:::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::
