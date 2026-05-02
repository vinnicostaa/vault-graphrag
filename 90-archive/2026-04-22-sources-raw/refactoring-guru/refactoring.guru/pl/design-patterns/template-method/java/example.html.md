::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Wzorce
projektowe](../../../design-patterns.html){.type} / [Metoda
szablonowa](../../template-method.html){.type} /
[Java](../../java.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Metoda
szablonowa](../../../../images/patterns/cards/template-method-mini56e3.png?id=9f200248d88026d8e79d0f3dae411ab4){srcset="/images/patterns/cards/template-method-mini-2x.png?id=178bf56e39b3a1f548dd636076209c98 2x"}
:::

# **Metoda szablonowa** w języku Java {#metoda-szablonowa-w-języku-java .pattern-example-title-block-title}
::::

:::::::::::::::::: pattern-example-body
::: pattern-example-brief
**Metoda szablonowa** to behawioralny wzorzec projektowy według którego
definiuje się szkielet algorytmu w klasie bazowej i pozwala klasom
pochodnym nadpisać poszczególne jego etapy bez zmiany ogólnej struktury.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Dowiedz się więcej o Metoda szablonowa
](../../template-method.html){.btn .btn-lg .btn-primary}
:::

::::::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
Nawigacja
:::

::: en-intro
 [Intro](#)
:::

::: en-intro
 [Nadpisywanie standardowych etapów algorytmu](#example-0)
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

**Złożoność:** []{.fa-stars}

**Popularność:** []{.fa-stars}

**Przykłady użycia:** Metoda szablonowa jest dość powszechnie
stosowana we frameworkach Java. Twórcy oprogramowania często za jej
pomocą dają użytkownikom frameworku możliwość rozszerzenia jego
standardowej funkcjonalności poprzez dziedziczenie.

Oto przykłady metod szablonowych w głównych bibliotekach Java:

-   Wszystkie nie abstrakcyjne metody z
    [`java.io.InputStream`](http://docs.oracle.com/javase/8/docs/api/java/io/InputStream.html),
    [`java.io.OutputStream`](http://docs.oracle.com/javase/8/docs/api/java/io/OutputStream.html),
    [`java.io.Reader`](http://docs.oracle.com/javase/8/docs/api/java/io/Reader.html)
    oraz
    [`java.io.Writer`](http://docs.oracle.com/javase/8/docs/api/java/io/Writer.html).

-   Wszystkie nie abstrakcyjne metody z
    [`java.util.AbstractList`](http://docs.oracle.com/javase/8/docs/api/java/util/AbstractList.html),
    [`java.util.AbstractSet`](http://docs.oracle.com/javase/8/docs/api/java/util/AbstractSet.html)
    oraz
    [`java.util.AbstractMap`](http://docs.oracle.com/javase/8/docs/api/java/util/AbstractMap.html).

-   [`javax.servlet.http.HttpServlet`](http://docs.oracle.com/javaee/7/api/javax/servlet/http/HttpServlet.html),
    wszystkie metody `doXXX()` domyślnie przesyłają błąd HTTP 405
    "Niedozwolona metoda" w ramach odpowiedzi. Możesz nadpisać każdą z
    nich.

**Identyfikacja:** Zastosowanie tego wzorca można poznać po obecności
behawioralnych metod posiadających jakieś domyślne zachowanie
zdefiniowane przez klasę bazową.
::::::::::::::::::

[]{#example-0}

## Nadpisywanie standardowych etapów algorytmu {#example-0-title}

W poniższym przykładzie wzorzec Metoda szablonowa definiuje algorytm
współpracy z platformą społecznościową. Klasy pochodne odpowiadające
poszczególnym platformom implementują te etapy zgodnie z interfejsem
programowania aplikacji danej platformy.

### []{#example-0--networks .anchor} **networks** {#networks .example-title}

#### []{#example-0--networks-Network-java .anchor} **networks/Network.java:** Klasa bazowa platformy społecznościowej

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.template_method.example.networks;

/**
 * Base class of social network.
 */
public abstract class Network {
    String userName;
    String password;

    Network() {}

    /**
     * Publish the data to whatever network.
     */
    public boolean post(String message) {
        // Authenticate before posting. Every network uses a different
        // authentication method.
        if (logIn(this.userName, this.password)) {
            // Send the post data.
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

#### []{#example-0--networks-Facebook-java .anchor} **networks/Facebook.java:** Konkretna platforma społecznościowa

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.template_method.example.networks;

/**
 * Class of social network
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

#### []{#example-0--networks-Twitter-java .anchor} **networks/Twitter.java:** Kolejna platforma społecznościowa

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.template_method.example.networks;

/**
 * Class of social network
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

#### []{#example-0--Demo-java .anchor} **Demo.java:** Kod klienta

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.template_method.example;

import refactoring_guru.template_method.example.networks.Facebook;
import refactoring_guru.template_method.example.networks.Network;
import refactoring_guru.template_method.example.networks.Twitter;

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;

/**
 * Demo class. Everything comes together here.
 */
public class Demo {
    public static void main(String[] args) throws IOException {
        BufferedReader reader = new BufferedReader(new InputStreamReader(System.in));
        Network network = null;
        System.out.print(&quot;Input user name: &quot;);
        String userName = reader.readLine();
        System.out.print(&quot;Input password: &quot;);
        String password = reader.readLine();

        // Enter the message.
        System.out.print(&quot;Input message: &quot;);
        String message = reader.readLine();

        System.out.println(&quot;\nChoose social network for posting message.\n&quot; +
                &quot;1 - Facebook\n&quot; +
                &quot;2 - Twitter&quot;);
        int choice = Integer.parseInt(reader.readLine());

        // Create proper network object and send the message.
        if (choice == 1) {
            network = new Facebook(userName, password);
        } else if (choice == 2) {
            network = new Twitter(userName, password);
        }
        network.post(message);
    }
}</code></pre>
</figure>

#### []{#example-0--OutputDemo-txt .anchor} **OutputDemo.txt:** Wynik działania

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
#### Czytaj dalej

[Odwiedzający w języku Java []{.fa
.fa-arrow-right}](../../visitor/java/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Wróć

[[]{.fa .fa-arrow-left} Strategia w języku
Java](../../strategy/java/example.html){.btn .btn-default rel="prev"}
:::

## **Metoda szablonowa** w innych językach

[![Metoda szablonowa w języku
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Metoda szablonowa w języku C#"){.prog-lang-link}
[![Metoda szablonowa w języku
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Metoda szablonowa w języku C++"){.prog-lang-link}
[![Metoda szablonowa w języku
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Metoda szablonowa w języku Go"){.prog-lang-link}
[![Metoda szablonowa w języku
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Metoda szablonowa w języku PHP"){.prog-lang-link}
[![Metoda szablonowa w języku
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Metoda szablonowa w języku Python"){.prog-lang-link}
[![Metoda szablonowa w języku
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Metoda szablonowa w języku Ruby"){.prog-lang-link}
[![Metoda szablonowa w języku
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Metoda szablonowa w języku Rust"){.prog-lang-link}
[![Metoda szablonowa w języku
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Metoda szablonowa w języku Swift"){.prog-lang-link}
[![Metoda szablonowa w języku
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Metoda szablonowa w języku TypeScript"){.prog-lang-link}
::::::::::::::::::::::::

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
::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::
