:::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Patrons de
conception](../../../design-patterns.html){.type} /
[Médiateur](../../mediator.html){.type} /
[TypeScript](../../typescript.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Médiateur](../../../../images/patterns/cards/mediator-mini1561.png?id=a7e43ee8e17e4474737b1fcb3201d7ba){srcset="/images/patterns/cards/mediator-mini-2x.png?id=d288d7c73f5581ae09701d61200ae09e 2x"}
:::

# **Médiateur** en TypeScript {#médiateur-en-typescript .pattern-example-title-block-title}
::::

::::::::::: pattern-example-body
::: pattern-example-brief
Le **Médiateur** est un patron de conception comportemental qui diminue
le couplage entre les composants d'un programme, en les faisant
communiquer indirectement, via un objet médiateur spécial.

Le médiateur rend la modification, l'extension et la réutilisation de
composants individuels aisées, car ils ne dépendent plus d'une dizaine
de classes.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ En savoir plus sur la patron Médiateur ](../../mediator.html){.btn
.btn-lg .btn-primary}
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

**Exemples d'utilisation :** Le médiateur est le plus souvent utilisé en
TypeScript pour faciliter les communications entre les composants de
l'interface graphique d'une application. Le contrôleur du modèle MVC est
le synonyme du médiateur.
:::::::::::

[]{#example-0}

## Exemple conceptuel {#example-0-title}

Dans cet exemple, nous allons voir la structure du **Médiateur** et
répondre aux questions suivantes :

-   Que contiennent les classes ?
-   Quels rôles jouent-elles ?
-   Comment les éléments du patron sont-ils reliés ?

#### []{#example-0--index-ts .anchor} **index.ts:** Exemple conceptuel

<figure class="code">
<pre class="code" lang="typescript"><code>/**
 * The Mediator interface declares a method used by components to notify the
 * mediator about various events. The Mediator may react to these events and
 * pass the execution to other components.
 */
interface Mediator {
    notify(sender: object, event: string): void;
}

/**
 * Concrete Mediators implement cooperative behavior by coordinating several
 * components.
 */
class ConcreteMediator implements Mediator {
    private component1: Component1;

    private component2: Component2;

    constructor(c1: Component1, c2: Component2) {
        this.component1 = c1;
        this.component1.setMediator(this);
        this.component2 = c2;
        this.component2.setMediator(this);
    }

    public notify(sender: object, event: string): void {
        if (event === &#39;A&#39;) {
            console.log(&#39;Mediator reacts on A and triggers following operations:&#39;);
            this.component2.doC();
        }

        if (event === &#39;D&#39;) {
            console.log(&#39;Mediator reacts on D and triggers following operations:&#39;);
            this.component1.doB();
            this.component2.doC();
        }
    }
}

/**
 * The Base Component provides the basic functionality of storing a mediator&#39;s
 * instance inside component objects.
 */
class BaseComponent {
    protected mediator: Mediator;

    constructor(mediator?: Mediator) {
        this.mediator = mediator!;
    }

    public setMediator(mediator: Mediator): void {
        this.mediator = mediator;
    }
}

/**
 * Concrete Components implement various functionality. They don&#39;t depend on
 * other components. They also don&#39;t depend on any concrete mediator classes.
 */
class Component1 extends BaseComponent {
    public doA(): void {
        console.log(&#39;Component 1 does A.&#39;);
        this.mediator.notify(this, &#39;A&#39;);
    }

    public doB(): void {
        console.log(&#39;Component 1 does B.&#39;);
        this.mediator.notify(this, &#39;B&#39;);
    }
}

class Component2 extends BaseComponent {
    public doC(): void {
        console.log(&#39;Component 2 does C.&#39;);
        this.mediator.notify(this, &#39;C&#39;);
    }

    public doD(): void {
        console.log(&#39;Component 2 does D.&#39;);
        this.mediator.notify(this, &#39;D&#39;);
    }
}

/**
 * The client code.
 */
const c1 = new Component1();
const c2 = new Component2();
const mediator = new ConcreteMediator(c1, c2);

console.log(&#39;Client triggers operation A.&#39;);
c1.doA();

console.log(&#39;&#39;);
console.log(&#39;Client triggers operation D.&#39;);
c2.doD();</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** Résultat de l'exécution

<figure class="code">
<pre class="code" lang="output"><code>Client triggers operation A.
Component 1 does A.
Mediator reacts on A and triggers following operations:
Component 2 does C.

Client triggers operation D.
Component 2 does D.
Mediator reacts on D and triggers following operations:
Component 1 does B.
Component 2 does C.</code></pre>
</figure>

::: next
#### Suivant

[Mémento en TypeScript []{.fa
.fa-arrow-right}](../../memento/typescript/example.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Retour

[[]{.fa .fa-arrow-left} Itérateur en
TypeScript](../../iterator/typescript/example.html){.btn .btn-default
rel="prev"}
:::

## **Médiateur** dans les autres langues

[![Médiateur en
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Médiateur en C#"){.prog-lang-link}
[![Médiateur en
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Médiateur en C++"){.prog-lang-link}
[![Médiateur en
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Médiateur en Go"){.prog-lang-link}
[![Médiateur en
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Médiateur en Java"){.prog-lang-link}
[![Médiateur en
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Médiateur en PHP"){.prog-lang-link}
[![Médiateur en
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Médiateur en Python"){.prog-lang-link}
[![Médiateur en
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Médiateur en Ruby"){.prog-lang-link}
[![Médiateur en
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Médiateur en Rust"){.prog-lang-link}
[![Médiateur en
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Médiateur en Swift"){.prog-lang-link}
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
