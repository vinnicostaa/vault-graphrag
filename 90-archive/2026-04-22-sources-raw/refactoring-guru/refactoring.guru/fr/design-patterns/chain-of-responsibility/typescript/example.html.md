:::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Patrons de
conception](../../../design-patterns.html){.type} / [Chaîne de
responsabilité](../../chain-of-responsibility.html){.type} /
[TypeScript](../../typescript.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Chaîne de
responsabilité](../../../../images/patterns/cards/chain-of-responsibility-mini6d42.png?id=36d85eba8d14986f053123de17aac7a7){srcset="/images/patterns/cards/chain-of-responsibility-mini-2x.png?id=8c81f7069e51259b2443801b91135f7f 2x"}
:::

# **Chaîne de responsabilité** en TypeScript {#chaîne-de-responsabilité-en-typescript .pattern-example-title-block-title}
::::

::::::::::: pattern-example-body
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

:::::::: {.sidebar-navigation .with-scroll}
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
 [index](#example-0--index-ts)
:::

::: en-file
 [Output](#example-0--Output-txt)
:::
::::::::

**Complexité :** []{.fa-stars}

**Popularité :** []{.fa-stars}

**Exemples d'utilisation :** La chaîne de responsabilité n'est pas
souvent invitée dans les programmes TypeScript, car son intérêt réside
dans la gestion du chaînage.

**Identification :** Ce patron peut être reconnu grâce aux méthodes
comportementales d'un groupe d'objets qui appellent indirectement les
mêmes méthodes dans d'autres objets, et tous suivent la même interface.
:::::::::::

[]{#example-0}

## Exemple conceptuel {#example-0-title}

Dans cet exemple, nous allons voir la structure de la **Chaîne de
responsabilité** et répondre aux questions suivantes :

-   Que contiennent les classes ?
-   Quels rôles jouent-elles ?
-   Comment les éléments du patron sont-ils reliés ?

#### []{#example-0--index-ts .anchor} **index.ts:** Exemple conceptuel

<figure class="code">
<pre class="code" lang="typescript"><code>/**
 * The Handler interface declares a method for building the chain of handlers.
 * It also declares a method for executing a request.
 */
interface Handler&lt;Request = string, Result = string&gt; {
    setNext(handler: Handler&lt;Request, Result&gt;): Handler&lt;Request, Result&gt;;

    handle(request: Request): Result;
}

/**
 * The default chaining behavior can be implemented inside a base handler class.
 */
abstract class AbstractHandler implements Handler
{
    private nextHandler: Handler;

    public setNext(handler: Handler): Handler {
        this.nextHandler = handler;
        // Returning a handler from here will let us link handlers in a
        // convenient way like this:
        // monkey.setNext(squirrel).setNext(dog);
        return handler;
    }

    public handle(request: string): string {
        if (this.nextHandler) {
            return this.nextHandler.handle(request);
        }

        return null;
    }
}

/**
 * All Concrete Handlers either handle a request or pass it to the next handler
 * in the chain.
 */
class MonkeyHandler extends AbstractHandler {
    public handle(request: string): string {
        if (request === &#39;Banana&#39;) {
            return `Monkey: I&#39;ll eat the ${request}.`;
        }
        return super.handle(request);

    }
}

class SquirrelHandler extends AbstractHandler {
    public handle(request: string): string {
        if (request === &#39;Nut&#39;) {
            return `Squirrel: I&#39;ll eat the ${request}.`;
        }
        return super.handle(request);
    }
}

class DogHandler extends AbstractHandler {
    public handle(request: string): string {
        if (request === &#39;MeatBall&#39;) {
            return `Dog: I&#39;ll eat the ${request}.`;
        }
        return super.handle(request);
    }
}

/**
 * The client code is usually suited to work with a single handler. In most
 * cases, it is not even aware that the handler is part of a chain.
 */
function clientCode(handler: Handler) {
    const foods = [&#39;Nut&#39;, &#39;Banana&#39;, &#39;Cup of coffee&#39;];

    for (const food of foods) {
        console.log(`Client: Who wants a ${food}?`);

        const result = handler.handle(food);
        if (result) {
            console.log(`  ${result}`);
        } else {
            console.log(`  ${food} was left untouched.`);
        }
    }
}

/**
 * The other part of the client code constructs the actual chain.
 */
const monkey = new MonkeyHandler();
const squirrel = new SquirrelHandler();
const dog = new DogHandler();

monkey.setNext(squirrel).setNext(dog);

/**
 * The client should be able to send a request to any handler, not just the
 * first one in the chain.
 */
console.log(&#39;Chain: Monkey &gt; Squirrel &gt; Dog\n&#39;);
clientCode(monkey);
console.log(&#39;&#39;);

console.log(&#39;Subchain: Squirrel &gt; Dog\n&#39;);
clientCode(squirrel);</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** Résultat de l'exécution

<figure class="code">
<pre class="code" lang="output"><code>Chain: Monkey &gt; Squirrel &gt; Dog

Client: Who wants a Nut?
  Squirrel: I&#39;ll eat the Nut.
Client: Who wants a Banana?
  Monkey: I&#39;ll eat the Banana.
Client: Who wants a Cup of coffee?
  Cup of coffee was left untouched.

Subchain: Squirrel &gt; Dog

Client: Who wants a Nut?
  Squirrel: I&#39;ll eat the Nut.
Client: Who wants a Banana?
  Banana was left untouched.
Client: Who wants a Cup of coffee?
  Cup of coffee was left untouched.</code></pre>
</figure>

::: next
#### Suivant

[Commande en TypeScript []{.fa
.fa-arrow-right}](../../command/typescript/example.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Retour

[[]{.fa .fa-arrow-left} Procuration en
TypeScript](../../proxy/typescript/example.html){.btn .btn-default
rel="prev"}
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
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Chaîne de responsabilité en Java"){.prog-lang-link}
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
:::::::::::::::::

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
:::::::::::::::::::::::
::::::::::::::::::::::::
