:::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Padrões de
Projeto](../../../design-patterns.html){.type} / [Chain of
Responsibility](../../chain-of-responsibility.html){.type} /
[Java](../../java.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Chain of
Responsibility](../../../../images/patterns/cards/chain-of-responsibility-mini6d42.png?id=36d85eba8d14986f053123de17aac7a7){srcset="/images/patterns/cards/chain-of-responsibility-mini-2x.png?id=8c81f7069e51259b2443801b91135f7f 2x"}
:::

# **Chain of Responsibility** em Java {#chain-of-responsibility-em-java .pattern-example-title-block-title}
::::

::::::::::::::::::::::: pattern-example-body
::: pattern-example-brief
O **Chain of Responsibility** é um padrão de projeto comportamental que
permite passar a solicitação ao longo da cadeia de handlers em potencial
até que um deles lide com a solicitação.

O padrão permite que vários objetos tratem a solicitação sem acoplar a
classe remetente às classes concretas dos destinatários. A cadeia pode
ser composta dinamicamente em tempo de execução com qualquer handler que
siga uma interface de handler padrão.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Saiba mais sobre o Chain of Responsibility
](../../chain-of-responsibility.html){.btn .btn-lg .btn-primary}
:::

:::::::::::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
Navegação
:::

::: en-intro
 [Introdução](#)
:::

::: en-intro
 [Filtrando o acesso](#example-0)
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

**Complexidade:** []{.fa-stars}

**Popularidade:** []{.fa-stars}

**Exemplos de uso:** O padrão Chain of Responsibility não é um padrão
frequente em um programa Java, pois é relevante apenas quando o código
opera com cadeias de objetos.

Um dos casos de uso mais populares para o padrão é a interferência de
eventos nos componentes pai nas classes da GUI. Outro caso de uso
notável são os filtros de acesso seqüencial.

Aqui estão alguns exemplos do padrão nas principais bibliotecas Java:

-   [`javax.servlet.Filter#doFilter()`](http://docs.oracle.com/javaee/7/api/javax/servlet/Filter.html#doFilter-javax.servlet.ServletRequest-javax.servlet.ServletResponse-javax.servlet.FilterChain-)
-   [`java.util.logging.Logger#log()`](http://docs.oracle.com/javase/8/docs/api/java/util/logging/Logger.html#log-java.util.logging.Level-java.lang.String-)

**Identificação:** O padrão é reconhecível pelos métodos comportamentais
de um grupo de objetos que indiretamente chamam os mesmos métodos em
outros objetos, enquanto todos os objetos seguem a interface comum.
:::::::::::::::::::::::

[]{#example-0}

## Filtrando o acesso {#example-0-title}

Este exemplo mostra como uma solicitação que contém dados do usuário
passa por uma cadeia sequencial de manipuladores que executam várias
tarefas, como autenticação, autorização, e validação.

Este exemplo é um pouco diferente da versão canônica do padrão fornecida
por vários autores. A maioria dos exemplos do padrão é construída com a
noção de procurar o handler correto, iniciá-lo, e sair da cadeia depois
disso. Mas aqui executamos todos os handlers até que um que **não possa
lidar** com uma solicitação seja encontrado. Esteja ciente de que esse
ainda é o padrão Chain of Responsibility, mesmo que o fluxo seja um
pouco diferente.

### []{#example-0--middleware .anchor} **middleware** {#middleware .example-title}

#### []{#example-0--middleware-Middleware-java .anchor} **middleware/Middleware.java:** Interface de validação básica

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

#### []{#example-0--middleware-ThrottlingMiddleware-java .anchor} **middleware/ThrottlingMiddleware.java:** Checar limite de pedidos

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

#### []{#example-0--middleware-UserExistsMiddleware-java .anchor} **middleware/UserExistsMiddleware.java:** Checar credenciais do usuário

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

#### []{#example-0--middleware-RoleCheckMiddleware-java .anchor} **middleware/RoleCheckMiddleware.java:** Checar papel do usuário

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

#### []{#example-0--server-Server-java .anchor} **server/Server.java:** Alvo de autorização

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

#### []{#example-0--Demo-java .anchor} **Demo.java:** Código cliente

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

#### []{#example-0--OutputDemo-txt .anchor} **OutputDemo.txt:** Resultados da execução

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
#### Leia a seguir

[Command em Java []{.fa
.fa-arrow-right}](../../command/java/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Voltar

[[]{.fa .fa-arrow-left} Proxy em
Java](../../proxy/java/example.html){.btn .btn-default rel="prev"}
:::

## **Chain of Responsibility** em outras linguagens

[![Chain of Responsibility em
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Chain of Responsibility em C#"){.prog-lang-link}
[![Chain of Responsibility em
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Chain of Responsibility em C++"){.prog-lang-link}
[![Chain of Responsibility em
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Chain of Responsibility em Go"){.prog-lang-link}
[![Chain of Responsibility em
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Chain of Responsibility em PHP"){.prog-lang-link}
[![Chain of Responsibility em
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Chain of Responsibility em Python"){.prog-lang-link}
[![Chain of Responsibility em
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Chain of Responsibility em Ruby"){.prog-lang-link}
[![Chain of Responsibility em
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Chain of Responsibility em Rust"){.prog-lang-link}
[![Chain of Responsibility em
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Chain of Responsibility em Swift"){.prog-lang-link}
[![Chain of Responsibility em
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Chain of Responsibility em TypeScript"){.prog-lang-link}
:::::::::::::::::::::::::::::

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
:::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::
