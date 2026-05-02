:::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Patrons de
conception](../../../design-patterns.html){.type} / [Poids
mouche](../../flyweight.html){.type} /
[TypeScript](../../typescript.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Poids
mouche](../../../../images/patterns/cards/flyweight-minie3d8.png?id=422ca8d2f90614dce810a8812c626698){srcset="/images/patterns/cards/flyweight-mini-2x.png?id=857ca676fff84317bb0dab76abfce08e 2x"}
:::

# **Poids mouche** en TypeScript {#poids-mouche-en-typescript .pattern-example-title-block-title}
::::

::::::::::: pattern-example-body
::: pattern-example-brief
Le **poids mouche** est un patron de conception structurel qui permet à
des programmes de limiter leur consommation de mémoire malgré un très
grand nombre d'objets.

Ce patron est obtenu en partageant des parties de l'état d'un objet à
plusieurs autres objets. En d'autres termes, le poids mouche économise
de la RAM en mettant en cache les données identiques chez différents
objets.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ En savoir plus sur la patron Poids mouche ](../../flyweight.html){.btn
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

**Exemples d'utilisation :** Le poids mouche n'a qu'une seule utilité :
minimiser l'utilisation de la mémoire. Si votre programme ne rencontre
aucun problème de RAM, ignorez ce patron pour le moment.

**Identification :** Le poids mouche peut être reconnu par une méthode
de création qui renvoie des objets du cache plutôt que d'en créer de
nouveaux.
:::::::::::

[]{#example-0}

## Exemple conceptuel {#example-0-title}

Dans cet exemple, nous allons voir la structure du **Poids mouche** et
répondre aux questions suivantes :

-   Que contiennent les classes ?
-   Quels rôles jouent-elles ?
-   Comment les éléments du patron sont-ils reliés ?

#### []{#example-0--index-ts .anchor} **index.ts:** Exemple conceptuel

<figure class="code">
<pre class="code" lang="typescript"><code>/**
 * The Flyweight stores a common portion of the state (also called intrinsic
 * state) that belongs to multiple real business entities. The Flyweight accepts
 * the rest of the state (extrinsic state, unique for each entity) via its
 * method parameters.
 */
class Flyweight {
    private sharedState: any;

    constructor(sharedState: any) {
        this.sharedState = sharedState;
    }

    public operation(uniqueState): void {
        const s = JSON.stringify(this.sharedState);
        const u = JSON.stringify(uniqueState);
        console.log(`Flyweight: Displaying shared (${s}) and unique (${u}) state.`);
    }
}

/**
 * The Flyweight Factory creates and manages the Flyweight objects. It ensures
 * that flyweights are shared correctly. When the client requests a flyweight,
 * the factory either returns an existing instance or creates a new one, if it
 * doesn&#39;t exist yet.
 */
class FlyweightFactory {
    private flyweights: {[key: string]: Flyweight} = &lt;any&gt;{};

    constructor(initialFlyweights: string[][]) {
        for (const state of initialFlyweights) {
            this.flyweights[this.getKey(state)] = new Flyweight(state);
        }
    }

    /**
     * Returns a Flyweight&#39;s string hash for a given state.
     */
    private getKey(state: string[]): string {
        return state.join(&#39;_&#39;);
    }

    /**
     * Returns an existing Flyweight with a given state or creates a new one.
     */
    public getFlyweight(sharedState: string[]): Flyweight {
        const key = this.getKey(sharedState);

        if (!(key in this.flyweights)) {
            console.log(&#39;FlyweightFactory: Can\&#39;t find a flyweight, creating new one.&#39;);
            this.flyweights[key] = new Flyweight(sharedState);
        } else {
            console.log(&#39;FlyweightFactory: Reusing existing flyweight.&#39;);
        }

        return this.flyweights[key];
    }

    public listFlyweights(): void {
        const count = Object.keys(this.flyweights).length;
        console.log(`\nFlyweightFactory: I have ${count} flyweights:`);
        for (const key in this.flyweights) {
            console.log(key);
        }
    }
}

/**
 * The client code usually creates a bunch of pre-populated flyweights in the
 * initialization stage of the application.
 */
const factory = new FlyweightFactory([
    [&#39;Chevrolet&#39;, &#39;Camaro2018&#39;, &#39;pink&#39;],
    [&#39;Mercedes Benz&#39;, &#39;C300&#39;, &#39;black&#39;],
    [&#39;Mercedes Benz&#39;, &#39;C500&#39;, &#39;red&#39;],
    [&#39;BMW&#39;, &#39;M5&#39;, &#39;red&#39;],
    [&#39;BMW&#39;, &#39;X6&#39;, &#39;white&#39;],
    // ...
]);
factory.listFlyweights();

// ...

function addCarToPoliceDatabase(
    ff: FlyweightFactory, plates: string, owner: string,
    brand: string, model: string, color: string,
) {
    console.log(&#39;\nClient: Adding a car to database.&#39;);
    const flyweight = ff.getFlyweight([brand, model, color]);

    // The client code either stores or calculates extrinsic state and passes it
    // to the flyweight&#39;s methods.
    flyweight.operation([plates, owner]);
}

addCarToPoliceDatabase(factory, &#39;CL234IR&#39;, &#39;James Doe&#39;, &#39;BMW&#39;, &#39;M5&#39;, &#39;red&#39;);

addCarToPoliceDatabase(factory, &#39;CL234IR&#39;, &#39;James Doe&#39;, &#39;BMW&#39;, &#39;X1&#39;, &#39;red&#39;);

factory.listFlyweights();</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** Résultat de l'exécution

<figure class="code">
<pre class="code" lang="output"><code>FlyweightFactory: I have 5 flyweights:
Chevrolet_Camaro2018_pink
Mercedes Benz_C300_black
Mercedes Benz_C500_red
BMW_M5_red
BMW_X6_white

Client: Adding a car to database.
FlyweightFactory: Reusing existing flyweight.
Flyweight: Displaying shared ([&quot;BMW&quot;,&quot;M5&quot;,&quot;red&quot;]) and unique ([&quot;CL234IR&quot;,&quot;James Doe&quot;]) state.

Client: Adding a car to database.
FlyweightFactory: Can&#39;t find a flyweight, creating new one.
Flyweight: Displaying shared ([&quot;BMW&quot;,&quot;X1&quot;,&quot;red&quot;]) and unique ([&quot;CL234IR&quot;,&quot;James Doe&quot;]) state.

FlyweightFactory: I have 6 flyweights:
Chevrolet_Camaro2018_pink
Mercedes Benz_C300_black
Mercedes Benz_C500_red
BMW_M5_red
BMW_X6_white
BMW_X1_red</code></pre>
</figure>

::: next
#### Suivant

[Procuration en TypeScript []{.fa
.fa-arrow-right}](../../proxy/typescript/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Retour

[[]{.fa .fa-arrow-left} Façade en
TypeScript](../../facade/typescript/example.html){.btn .btn-default
rel="prev"}
:::

## **Poids mouche** dans les autres langues

[![Poids mouche en
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Poids mouche en C#"){.prog-lang-link}
[![Poids mouche en
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Poids mouche en C++"){.prog-lang-link}
[![Poids mouche en
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Poids mouche en Go"){.prog-lang-link}
[![Poids mouche en
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Poids mouche en Java"){.prog-lang-link}
[![Poids mouche en
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Poids mouche en PHP"){.prog-lang-link}
[![Poids mouche en
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Poids mouche en Python"){.prog-lang-link}
[![Poids mouche en
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Poids mouche en Ruby"){.prog-lang-link}
[![Poids mouche en
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Poids mouche en Rust"){.prog-lang-link}
[![Poids mouche en
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Poids mouche en Swift"){.prog-lang-link}
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
