:::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [디자인
패턴들](../../../design-patterns.html){.type} /
[전략](../../strategy.html){.type} / [자바](../../java.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![전략](../../../../images/patterns/cards/strategy-mini28f6.png?id=d38abee4fb6f2aed909d262bdadca936){srcset="/images/patterns/cards/strategy-mini-2x.png?id=f4e6608561f8e5d18be6927d4620ad29 2x"}
:::

# 자바로 작성된 **전략** {#자바로-작성된-전략 .pattern-example-title-block-title}
::::

::::::::::::::::::::::: pattern-example-body
::: pattern-example-brief
**전략**은 행동들의 객체들을 객체들로 변환하며 이들이 원래 콘텍스트 객체
내에서 상호 교환이 가능하게 만드는 행동 디자인 패턴입니다.

원래 객체는 콘텍스트라고 불리며 전략 객체에 대한 참조를 포함합니다.
콘텍스트는 행동의 실행을 연결된 전략 객체에 위임합니다. 콘텍스트가
작업을 수행하는 방식을 변경하기 위하여 다른 객체들은 현재 연결된 전략
객체를 다른 전략 객체와 대체할 수 있습니다.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ 전략에 대하여 더 자세히 알아보세요 ](../../strategy.html){.btn .btn-lg
.btn-primary}
:::

:::::::::::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
내비게이션
:::

::: en-intro
 [소개](#)
:::

::: en-intro
 [전자상거래 앱의 결제 메서드](#example-0)
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

**복잡도:** []{.fa-stars}

**인기도:** []{.fa-stars}

**사용 사례들:** 전략 패턴은 자바 코드에서 매우 일반적입니다. 이 패턴은
다양한 프레임워크에서 사용자들이 클래스를 확장하지 않고 클래스의 행동을
변경할 수 있도록 자주 사용됩니다.

자바 8은 람다 함수를 지원하기 시작했으며, 해당 함수는 전략 패턴의 더
간단한 대안으로 사용될 수 있습니다.

다음은 코어 자바 라이브러리로부터 가져온 전략 패턴의 몇 가지
예시들입니다:

-   `Collections#sort()`로부터 호출된
    [`java.util.Comparator#compare()`](http://docs.oracle.com/javase/8/docs/api/java/util/Comparator.html#compare-T-T-).

-   [`javax.servlet.http.HttpServlet`](http://docs.oracle.com/javaee/7/api/javax/servlet/http/HttpServlet.html):
    `service­()` 메서드, 또 `Http­Servlet­Request` 와 `Http­Servlet­Response`
    객체들을 매개변수로 받는 `do­XXX()`의 모든 메서드.

-   [`javax.servlet.Filter#doFilter()`](http://docs.oracle.com/javaee/7/api/javax/servlet/Filter.html#doFilter-javax.servlet.ServletRequest-javax.servlet.ServletResponse-javax.servlet.FilterChain-)

**식별:** 전략 패턴은 중첩된 객체가 실제 작업을 수행할 수 있도록 하는
메서드가 있으며 또 해당 객체를 다른 객체로 대체할 수 있는 세터가
있습니다.
:::::::::::::::::::::::

[]{#example-0}

## 전자상거래 앱의 결제 메서드 {#example-0-title}

이 예시에서의 전략 패턴은 전자 상거래 앱의 다양한 지불 메서드들을
구현하는 데 사용합니다. 구매할 제품을 선택한 후 고객은 페이팔 또는 신용
카드 지불 메서드를 선택합니다.

구상 전략들은 실제 지불을 수행할 뿐만 아니라 체크아웃 양식의 행동을
변경하여 지불 세부 정보들을 기록하는 적절한 필드들을 제공합니다.

### []{#example-0--strategies .anchor} **strategies** {#strategies .example-title}

#### []{#example-0--strategies-PayStrategy-java .anchor} **strategies/PayStrategy.java:** 결제 메서드들의 공통 인터페이스

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.strategy.example.strategies;

/**
 * Common interface for all strategies.
 */
public interface PayStrategy {
    boolean pay(int paymentAmount);
    void collectPaymentDetails();
}</code></pre>
</figure>

#### []{#example-0--strategies-PayByPayPal-java .anchor} **strategies/PayByPayPal.java:** 페이팔을 통한 결제

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.strategy.example.strategies;

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.util.HashMap;
import java.util.Map;

/**
 * Concrete strategy. Implements PayPal payment method.
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
     * Collect customer&#39;s data.
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
     * Save customer data for future shopping attempts.
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

#### []{#example-0--strategies-PayByCreditCard-java .anchor} **strategies/PayByCreditCard.java:** 크레딧 카드를 통한 결제

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.strategy.example.strategies;

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;

/**
 * Concrete strategy. Implements credit card payment method.
 */
public class PayByCreditCard implements PayStrategy {
    private final BufferedReader READER = new BufferedReader(new InputStreamReader(System.in));
    private CreditCard card;

    /**
     * Collect credit card data.
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

            // Validate credit card number...

        } catch (IOException ex) {
            ex.printStackTrace();
        }
    }

    /**
     * After card validation we can charge customer&#39;s credit card.
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

#### []{#example-0--strategies-CreditCard-java .anchor} **strategies/CreditCard.java:** 크레딧 카드 클래스

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.strategy.example.strategies;

/**
 * Dummy credit card class.
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

#### []{#example-0--order-Order-java .anchor} **order/Order.java:** 주문 클래스

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.strategy.example.order;

import refactoring_guru.strategy.example.strategies.PayStrategy;

/**
 * Order class. Doesn&#39;t know the concrete payment method (strategy) user has
 * picked. It uses common strategy interface to delegate collecting payment data
 * to strategy object. It can be used to save order to database.
 */
public class Order {
    private int totalCost = 0;
    private boolean isClosed = false;

    public void processOrder(PayStrategy strategy) {
        strategy.collectPaymentDetails();
        // Here we could collect and store payment data from the strategy.
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

#### []{#example-0--Demo-java .anchor} **Demo.java:** 클라이언트 코드

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
 * World first console e-commerce application.
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

                // Client creates different strategies based on input from user,
                // application configuration, etc.
                if (paymentMethod.equals(&quot;1&quot;)) {
                    strategy = new PayByPayPal();
                } else {
                    strategy = new PayByCreditCard();
                }
            }

            // Order object delegates gathering payment data to strategy object,
            // since only strategies know what data they need to process a
            // payment.
            order.processOrder(strategy);

            System.out.print(&quot;Pay &quot; + order.getTotalCost() + &quot; units or Continue shopping? P/C: &quot;);
            String proceed = reader.readLine();
            if (proceed.equalsIgnoreCase(&quot;P&quot;)) {
                // Finally, strategy handles the payment.
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

#### []{#example-0--OutputDemo-txt .anchor} **OutputDemo.txt:** 실행 결과

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
#### 다음을 보세요

[자바로 작성된 템플릿 메서드 []{.fa
.fa-arrow-right}](../../template-method/java/example.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### 돌아가기

[[]{.fa .fa-arrow-left} 자바로 작성된
상태](../../state/java/example.html){.btn .btn-default rel="prev"}
:::

## 다른 언어로 작성된 **전략**

[![C#으로 작성된
전략](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "C#으로 작성된 전략"){.prog-lang-link}
[![C++로 작성된
전략](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "C++로 작성된 전략"){.prog-lang-link}
[![Go로 작성된
전략](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Go로 작성된 전략"){.prog-lang-link}
[![PHP로 작성된
전략](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "PHP로 작성된 전략"){.prog-lang-link}
[![파이썬으로 작성된
전략](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "파이썬으로 작성된 전략"){.prog-lang-link}
[![루비로 작성된
전략](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "루비로 작성된 전략"){.prog-lang-link}
[![러스트로 작성된
전략](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "러스트로 작성된 전략"){.prog-lang-link}
[![스위프트로 작성된
전략](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "스위프트로 작성된 전략"){.prog-lang-link}
[![타입스크립트로 작성된
전략](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "타입스크립트로 작성된 전략"){.prog-lang-link}
:::::::::::::::::::::::::::::

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
:::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::
