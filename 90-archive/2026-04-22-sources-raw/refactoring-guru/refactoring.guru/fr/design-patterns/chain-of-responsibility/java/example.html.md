:::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Patrons de
conception](../../../design-patterns.html){.type} / [Chaîne de
responsabilité](../../chain-of-responsibility.html){.type} /
[Java](../../java.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Chaîne de
responsabilité](../../../../images/patterns/cards/chain-of-responsibility-mini6d42.png?id=36d85eba8d14986f053123de17aac7a7){srcset="/images/patterns/cards/chain-of-responsibility-mini-2x.png?id=8c81f7069e51259b2443801b91135f7f 2x"}
:::

# **Chaîne de responsabilité** en Java {#chaîne-de-responsabilité-en-java .pattern-example-title-block-title}
::::

::::::::::::::::::::::: pattern-example-body
::: pattern-example-brief
La **Chaîne de responsabilité** est un patron de conception
comportemental qui permet de faire circuler une demande tout au long
d'une chaîne de handlers, jusqu'à ce que l'un d'entre eux la traite.

Ce patron permet à plusieurs objets de traiter une demande sans coupler
la classe du demandeur aux classes concrètes des récepteurs. La chaîne
peut être assemblée dynamiquement à l'exécution à l'aide de tout handler
implémentant l'interface standard des handlers.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ En savoir plus sur la patron Chaîne de responsabilité
](../../chain-of-responsibility.html){.btn .btn-lg .btn-primary}
:::

:::::::::::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
Navigation
:::

::: en-intro
 [Intro](#)
:::

::: en-intro
 [Filtrer les accès](#example-0)
:::

::: en-file
 middleware
:::

:::: en-inside
::: en-file
  [Middleware](#example-0--middleware-Middleware-java)
:::
::::

:::: en-inside
::: en-file
  [Throttling­Middleware](#example-0--middleware-ThrottlingMiddleware-java)
:::
::::

:::: en-inside
::: en-file
  [User­Exists­Middleware](#example-0--middleware-UserExistsMiddleware-java)
:::
::::

:::: en-inside
::: en-file
  [Role­Check­Middleware](#example-0--middleware-RoleCheckMiddleware-java)
:::
::::

::: en-file
 server
:::

:::: en-inside
::: en-file
  [Server](#example-0--server-Server-java)
:::
::::

::: en-file
 [Demo](#example-0--Demo-java)
:::

::: en-file
 [Output­Demo](#example-0--OutputDemo-txt)
:::
::::::::::::::::::::

**Complexité :** []{.fa-stars}

**Popularité :** []{.fa-stars}

**Exemples d'utilisation :** La chaîne de responsabilité n'est pas
souvent invitée dans les programmes Java, car son intérêt réside dans la
gestion du chaînage.

Elle est plus souvent utilisée lorsque l'on veut faire remonter des
événements vers les composants parents dans les classes d'une interface
utilisateur graphique. Elle est également utilisée dans les filtres
d'accès séquentiels.

Voici quelques exemples tirés des bibliothèques principales de Java :

-   [`javax.servlet.Filter#doFilter()`](http://docs.oracle.com/javaee/7/api/javax/servlet/Filter.html#doFilter-javax.servlet.ServletRequest-javax.servlet.ServletResponse-javax.servlet.FilterChain-)
-   [`java.util.logging.Logger#log()`](http://docs.oracle.com/javase/8/docs/api/java/util/logging/Logger.html#log-java.util.logging.Level-java.lang.String-)

**Identification :** Ce patron peut être reconnu grâce aux méthodes
comportementales d'un groupe d'objets qui appellent indirectement les
mêmes méthodes dans d'autres objets, et tous suivent la même interface.
:::::::::::::::::::::::

[]{#example-0}

## Filtrer les accès {#example-0-title}

Dans cet exemple, vous pouvez voir comment une demande composée de
données utilisateur va parcourir une chaîne de handlers et lancer divers
traitements tels que l'authentification, l'autorisation et la
validation.

Cet exemple diffère légèrement de ceux que vous retrouverez chez les
autres auteurs, beaucoup plus canoniques. La majorité des exemples pour
ce patron sont construits dans l'idée de rechercher le bon handler,
l'exécuter, puis de sortir de la chaîne. Mais ici, nous exécutons chaque
handler jusqu'à ce que l'un d'entre eux ne puisse **pas traiter** la
demande. Nous sommes toujours en présence du patron chaîne de
responsabilité, même si le déroulement est légèrement différent.

### []{#example-0--middleware .anchor} **middleware** {#middleware .example-title}

#### []{#example-0--middleware-Middleware-java .anchor} **middleware/Middleware.java:** Interface de validation basique

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.chain_of_responsibility.example.middleware;

/**
 * Base middleware class.
 */
public abstract class Middleware {
    private Middleware next;

    /**
     * Builds chains of middleware objects.
     */
    public static Middleware link(Middleware first, Middleware... chain) {
        Middleware head = first;
        for (Middleware nextInChain: chain) {
            head.next = nextInChain;
            head = nextInChain;
        }
        return first;
    }

    /**
     * Subclasses will implement this method with concrete checks.
     */
    public abstract boolean check(String email, String password);

    /**
     * Runs check on the next object in chain or ends traversing if we&#39;re in
     * last object in chain.
     */
    protected boolean checkNext(String email, String password) {
        if (next == null) {
            return true;
        }
        return next.check(email, password);
    }
}</code></pre>
</figure>

#### []{#example-0--middleware-ThrottlingMiddleware-java .anchor} **middleware/ThrottlingMiddleware.java:** Vérifier la limite du nombre de demandes

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.chain_of_responsibility.example.middleware;

/**
 * ConcreteHandler. Checks whether there are too many failed login requests.
 */
public class ThrottlingMiddleware extends Middleware {
    private int requestPerMinute;
    private int request;
    private long currentTime;

    public ThrottlingMiddleware(int requestPerMinute) {
        this.requestPerMinute = requestPerMinute;
        this.currentTime = System.currentTimeMillis();
    }

    /**
     * Please, not that checkNext() call can be inserted both in the beginning
     * of this method and in the end.
     *
     * This gives much more flexibility than a simple loop over all middleware
     * objects. For instance, an element of a chain can change the order of
     * checks by running its check after all other checks.
     */
    public boolean check(String email, String password) {
        if (System.currentTimeMillis() &gt; currentTime + 60_000) {
            request = 0;
            currentTime = System.currentTimeMillis();
        }

        request++;
        
        if (request &gt; requestPerMinute) {
            System.out.println(&quot;Request limit exceeded!&quot;);
            Thread.currentThread().stop();
        }
        return checkNext(email, password);
    }
}</code></pre>
</figure>

#### []{#example-0--middleware-UserExistsMiddleware-java .anchor} **middleware/UserExistsMiddleware.java:** Vérifier les identifiants de l'utilisateur

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.chain_of_responsibility.example.middleware;

import refactoring_guru.chain_of_responsibility.example.server.Server;

/**
 * ConcreteHandler. Checks whether a user with the given credentials exists.
 */
public class UserExistsMiddleware extends Middleware {
    private Server server;

    public UserExistsMiddleware(Server server) {
        this.server = server;
    }

    public boolean check(String email, String password) {
        if (!server.hasEmail(email)) {
            System.out.println(&quot;This email is not registered!&quot;);
            return false;
        }
        if (!server.isValidPassword(email, password)) {
            System.out.println(&quot;Wrong password!&quot;);
            return false;
        }
        return checkNext(email, password);
    }
}</code></pre>
</figure>

#### []{#example-0--middleware-RoleCheckMiddleware-java .anchor} **middleware/RoleCheckMiddleware.java:** Vérifier le rôle de l'utilisateur

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.chain_of_responsibility.example.middleware;

/**
 * ConcreteHandler. Checks a user&#39;s role.
 */
public class RoleCheckMiddleware extends Middleware {
    public boolean check(String email, String password) {
        if (email.equals(&quot;admin@example.com&quot;)) {
            System.out.println(&quot;Hello, admin!&quot;);
            return true;
        }
        System.out.println(&quot;Hello, user!&quot;);
        return checkNext(email, password);
    }
}</code></pre>
</figure>

### []{#example-0--server .anchor} **server** {#server .example-title}

#### []{#example-0--server-Server-java .anchor} **server/Server.java:** Autorisation de la cible

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.chain_of_responsibility.example.server;

import refactoring_guru.chain_of_responsibility.example.middleware.Middleware;

import java.util.HashMap;
import java.util.Map;

/**
 * Server class.
 */
public class Server {
    private Map&lt;String, String&gt; users = new HashMap&lt;&gt;();
    private Middleware middleware;

    /**
     * Client passes a chain of object to server. This improves flexibility and
     * makes testing the server class easier.
     */
    public void setMiddleware(Middleware middleware) {
        this.middleware = middleware;
    }

    /**
     * Server gets email and password from client and sends the authorization
     * request to the chain.
     */
    public boolean logIn(String email, String password) {
        if (middleware.check(email, password)) {
            System.out.println(&quot;Authorization have been successful!&quot;);

            // Do something useful here for authorized users.

            return true;
        }
        return false;
    }

    public void register(String email, String password) {
        users.put(email, password);
    }

    public boolean hasEmail(String email) {
        return users.containsKey(email);
    }

    public boolean isValidPassword(String email, String password) {
        return users.get(email).equals(password);
    }
}</code></pre>
</figure>

#### []{#example-0--Demo-java .anchor} **Demo.java:** Code client

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.chain_of_responsibility.example;

import refactoring_guru.chain_of_responsibility.example.middleware.Middleware;
import refactoring_guru.chain_of_responsibility.example.middleware.RoleCheckMiddleware;
import refactoring_guru.chain_of_responsibility.example.middleware.ThrottlingMiddleware;
import refactoring_guru.chain_of_responsibility.example.middleware.UserExistsMiddleware;
import refactoring_guru.chain_of_responsibility.example.server.Server;

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;

/**
 * Demo class. Everything comes together here.
 */
public class Demo {
    private static BufferedReader reader = new BufferedReader(new InputStreamReader(System.in));
    private static Server server;

    private static void init() {
        server = new Server();
        server.register(&quot;admin@example.com&quot;, &quot;admin_pass&quot;);
        server.register(&quot;user@example.com&quot;, &quot;user_pass&quot;);

        // All checks are linked. Client can build various chains using the same
        // components.
        Middleware middleware = Middleware.link(
            new ThrottlingMiddleware(2),
            new UserExistsMiddleware(server),
            new RoleCheckMiddleware()
        );

        // Server gets a chain from client code.
        server.setMiddleware(middleware);
    }

    public static void main(String[] args) throws IOException {
        init();

        boolean success;
        do {
            System.out.print(&quot;Enter email: &quot;);
            String email = reader.readLine();
            System.out.print(&quot;Input password: &quot;);
            String password = reader.readLine();
            success = server.logIn(email, password);
        } while (!success);
    }
}</code></pre>
</figure>

#### []{#example-0--OutputDemo-txt .anchor} **OutputDemo.txt:** Résultat de l'exécution

<figure class="code">
<pre class="code" lang="output"><code>Enter email: admin@example.com
Input password: admin_pass
Hello, admin!
Authorization have been successful!


Enter email: wrong@example.com
Input password: wrong_pass
This email is not registered!
Enter email: wrong@example.com
Input password: wrong_pass
This email is not registered!
Enter email: wrong@example.com
Input password: wrong_pass
Request limit exceeded!</code></pre>
</figure>

::: next
#### Suivant

[Commande en Java []{.fa
.fa-arrow-right}](../../command/java/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Retour

[[]{.fa .fa-arrow-left} Procuration en
Java](../../proxy/java/example.html){.btn .btn-default rel="prev"}
:::

## **Chaîne de responsabilité** dans les autres langues

[![Chaîne de responsabilité en
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Chaîne de responsabilité en C#"){.prog-lang-link}
[![Chaîne de responsabilité en
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Chaîne de responsabilité en C++"){.prog-lang-link}
[![Chaîne de responsabilité en
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Chaîne de responsabilité en Go"){.prog-lang-link}
[![Chaîne de responsabilité en
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Chaîne de responsabilité en PHP"){.prog-lang-link}
[![Chaîne de responsabilité en
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Chaîne de responsabilité en Python"){.prog-lang-link}
[![Chaîne de responsabilité en
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Chaîne de responsabilité en Ruby"){.prog-lang-link}
[![Chaîne de responsabilité en
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Chaîne de responsabilité en Rust"){.prog-lang-link}
[![Chaîne de responsabilité en
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Chaîne de responsabilité en Swift"){.prog-lang-link}
[![Chaîne de responsabilité en
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Chaîne de responsabilité en TypeScript"){.prog-lang-link}
:::::::::::::::::::::::::::::

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
:::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::
