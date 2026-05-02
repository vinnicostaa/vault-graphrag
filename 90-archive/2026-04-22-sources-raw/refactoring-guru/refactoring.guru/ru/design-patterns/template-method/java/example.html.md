::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Паттерны
проектирования](../../../design-patterns.html){.type} / [Шаблонный
метод](../../template-method.html){.type} /
[Java](../../java.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Шаблонный
метод](../../../../images/patterns/cards/template-method-mini56e3.png?id=9f200248d88026d8e79d0f3dae411ab4){srcset="/images/patterns/cards/template-method-mini-2x.png?id=178bf56e39b3a1f548dd636076209c98 2x"}
:::

# **Шаблонный метод** на Java {#шаблонный-метод-на-java .pattern-example-title-block-title}
::::

:::::::::::::::::: pattern-example-body
::: pattern-example-brief
**Шаблонный метод** --- это поведенческий паттерн, задающий скелет
алгоритма в суперклассе и заставляющий подклассы реализовать конкретные
шаги этого алгоритма.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Подробней о паттерне Шаблонный метод
](../../template-method.html){.btn .btn-lg .btn-primary}
:::

::::::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
Навигация
:::

::: en-intro
 [Интро](#)
:::

::: en-intro
 [Переопределение шагов алгоритма](#example-0)
:::

::: en-file
 networks
:::

:::: en-inside
::: en-file
  [Network](#example-0--networks-Network-java)
:::
::::

:::: en-inside
::: en-file
  [Facebook](#example-0--networks-Facebook-java)
:::
::::

:::: en-inside
::: en-file
  [Twitter](#example-0--networks-Twitter-java)
:::
::::

::: en-file
 [Demo](#example-0--Demo-java)
:::

::: en-file
 [Output­Demo](#example-0--OutputDemo-txt)
:::
:::::::::::::::

**Сложность:** []{.fa-stars}

**Популярность:** []{.fa-stars}

**Применимость:** Шаблонные методы можно встретить во многих
библиотечных классах Java. Разработчики создают их, чтобы позволить
клиентам легко и быстро расширять стандартный код при помощи
наследования.

Примеры Шаблонных методов в стандартных библиотеках Java:

-   Все не-абстрактные методы классов
    [`java.io.InputStream`](http://docs.oracle.com/javase/8/docs/api/java/io/InputStream.html),
    [`java.io.OutputStream`](http://docs.oracle.com/javase/8/docs/api/java/io/OutputStream.html),
    [`java.io.Reader`](http://docs.oracle.com/javase/8/docs/api/java/io/Reader.html)
    и
    [`java.io.Writer`](http://docs.oracle.com/javase/8/docs/api/java/io/Writer.html).

-   Все не-абстрактные методы классов
    [`java.util.AbstractList`](http://docs.oracle.com/javase/8/docs/api/java/util/AbstractList.html),
    [`java.util.AbstractSet`](http://docs.oracle.com/javase/8/docs/api/java/util/AbstractSet.html)
    и
    [`java.util.AbstractMap`](http://docs.oracle.com/javase/8/docs/api/java/util/AbstractMap.html).

-   [`javax.servlet.http.HttpServlet`](http://docs.oracle.com/javaee/7/api/javax/servlet/http/HttpServlet.html),
    все методы `doXXX()` по умолчанию возвращают HTTP-код 405 \"Method
    Not Allowed\". Однако вы можете переопределить их при желании.

**Признаки применения паттерна:** Класс заставляет своих потомков
реализовать методы-шаги, но самостоятельно реализует структуру
алгоритма.
::::::::::::::::::

[]{#example-0}

## Переопределение шагов алгоритма {#example-0-title}

Социальные сети предоставляют собственные методы API для авторизации,
постинга и выхода, но общий процесс для всех сетей совпадает.

### []{#example-0--networks .anchor} **networks** {#networks .example-title}

#### []{#example-0--networks-Network-java .anchor} **networks/Network.java:** Базовый класс социальной сети

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.template_method.example.networks;

/**
 * Базовый класс социальной сети.
 */
public abstract class Network {
    String userName;
    String password;

    Network() {}

    /**
     * Публикация данных в любой сети.
     */
    public boolean post(String message) {
        // Проверка данных пользователя перед постом в соцсеть. Каждая сеть для
        // проверки использует разные методы.
        if (logIn(this.userName, this.password)) {
            // Отправка данных.
            boolean result =  sendData(message.getBytes());
            logOut();
            return result;
        }
        return false;
    }

    abstract boolean logIn(String userName, String password);
    abstract boolean sendData(byte[] data);
    abstract void logOut();
}</code></pre>
</figure>

#### []{#example-0--networks-Facebook-java .anchor} **networks/Facebook.java:** Конкретная социальная сеть

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.template_method.example.networks;

/**
 * Класс социальной сети.
 */
public class Facebook extends Network {
    public Facebook(String userName, String password) {
        this.userName = userName;
        this.password = password;
    }

    public boolean logIn(String userName, String password) {
        System.out.println(&quot;\nChecking user&#39;s parameters&quot;);
        System.out.println(&quot;Name: &quot; + this.userName);
        System.out.print(&quot;Password: &quot;);
        for (int i = 0; i &lt; this.password.length(); i++) {
            System.out.print(&quot;*&quot;);
        }
        simulateNetworkLatency();
        System.out.println(&quot;\n\nLogIn success on Facebook&quot;);
        return true;
    }

    public boolean sendData(byte[] data) {
        boolean messagePosted = true;
        if (messagePosted) {
            System.out.println(&quot;Message: &#39;&quot; + new String(data) + &quot;&#39; was posted on Facebook&quot;);
            return true;
        } else {
            return false;
        }
    }

    public void logOut() {
        System.out.println(&quot;User: &#39;&quot; + userName + &quot;&#39; was logged out from Facebook&quot;);
    }

    private void simulateNetworkLatency() {
        try {
            int i = 0;
            System.out.println();
            while (i &lt; 10) {
                System.out.print(&quot;.&quot;);
                Thread.sleep(500);
                i++;
            }
        } catch (InterruptedException ex) {
            ex.printStackTrace();
        }
    }
}</code></pre>
</figure>

#### []{#example-0--networks-Twitter-java .anchor} **networks/Twitter.java:** Ещё одна конкретная социальная сеть

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.template_method.example.networks;

/**
 * Класс социальной сети.
 */
public class Twitter extends Network {

    public Twitter(String userName, String password) {
        this.userName = userName;
        this.password = password;
    }

    public boolean logIn(String userName, String password) {
        System.out.println(&quot;\nChecking user&#39;s parameters&quot;);
        System.out.println(&quot;Name: &quot; + this.userName);
        System.out.print(&quot;Password: &quot;);
        for (int i = 0; i &lt; this.password.length(); i++) {
            System.out.print(&quot;*&quot;);
        }
        simulateNetworkLatency();
        System.out.println(&quot;\n\nLogIn success on Twitter&quot;);
        return true;
    }

    public boolean sendData(byte[] data) {
        boolean messagePosted = true;
        if (messagePosted) {
            System.out.println(&quot;Message: &#39;&quot; + new String(data) + &quot;&#39; was posted on Twitter&quot;);
            return true;
        } else {
            return false;
        }
    }

    public void logOut() {
        System.out.println(&quot;User: &#39;&quot; + userName + &quot;&#39; was logged out from Twitter&quot;);
    }

    private void simulateNetworkLatency() {
        try {
            int i = 0;
            System.out.println();
            while (i &lt; 10) {
                System.out.print(&quot;.&quot;);
                Thread.sleep(500);
                i++;
            }
        } catch (InterruptedException ex) {
            ex.printStackTrace();
        }
    }
}</code></pre>
</figure>

#### []{#example-0--Demo-java .anchor} **Demo.java:** Клиентский код

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.template_method.example;

import refactoring_guru.template_method.example.networks.Facebook;
import refactoring_guru.template_method.example.networks.Network;
import refactoring_guru.template_method.example.networks.Twitter;

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;

/**
 * Демо-класс. Здесь всё сводится воедино.
 */
public class Demo {
    public static void main(String[] args) throws IOException {
        BufferedReader reader = new BufferedReader(new InputStreamReader(System.in));
        Network network = null;
        System.out.print(&quot;Input user name: &quot;);
        String userName = reader.readLine();
        System.out.print(&quot;Input password: &quot;);
        String password = reader.readLine();

        // Вводим сообщение.
        System.out.print(&quot;Input message: &quot;);
        String message = reader.readLine();

        System.out.println(&quot;\nChoose social network for posting message.\n&quot; +
                &quot;1 - Facebook\n&quot; +
                &quot;2 - Twitter&quot;);
        int choice = Integer.parseInt(reader.readLine());

        // Создаем сетевые объекты и публикуем пост.
        if (choice == 1) {
            network = new Facebook(userName, password);
        } else if (choice == 2) {
            network = new Twitter(userName, password);
        }
        network.post(message);
    }
}</code></pre>
</figure>

#### []{#example-0--OutputDemo-txt .anchor} **OutputDemo.txt:** Результат выполнения

<figure class="code">
<pre class="code" lang="output"><code>Input user name: Jhonatan
Input password: qswe
Input message: Hello, World!

Choose social network for posting message.
1 - Facebook
2 - Twitter
2

Checking user&#39;s parameters
Name: Jhonatan
Password: ****
..........

LogIn success on Twitter
Message: &#39;Hello, World!&#39; was posted on Twitter
User: &#39;Jhonatan&#39; was logged out from Twitter</code></pre>
</figure>

::: next
#### Читаем дальше

[Посетитель на Java []{.fa
.fa-arrow-right}](../../visitor/java/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Стратегия на
Java](../../strategy/java/example.html){.btn .btn-default rel="prev"}
:::

## **Шаблонный метод** на других языках программирования

[![Шаблонный метод на
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Шаблонный метод на C#"){.prog-lang-link}
[![Шаблонный метод на
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Шаблонный метод на C++"){.prog-lang-link}
[![Шаблонный метод на
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Шаблонный метод на Go"){.prog-lang-link}
[![Шаблонный метод на
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Шаблонный метод на PHP"){.prog-lang-link}
[![Шаблонный метод на
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Шаблонный метод на Python"){.prog-lang-link}
[![Шаблонный метод на
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Шаблонный метод на Ruby"){.prog-lang-link}
[![Шаблонный метод на
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Шаблонный метод на Rust"){.prog-lang-link}
[![Шаблонный метод на
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Шаблонный метод на Swift"){.prog-lang-link}
[![Шаблонный метод на
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Шаблонный метод на TypeScript"){.prog-lang-link}
::::::::::::::::::::::::

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
::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::
