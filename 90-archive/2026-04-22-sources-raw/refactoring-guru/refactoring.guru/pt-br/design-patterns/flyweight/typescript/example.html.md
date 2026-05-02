:::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Padrões de
Projeto](../../../design-patterns.html){.type} /
[Flyweight](../../flyweight.html){.type} /
[TypeScript](../../typescript.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Flyweight](../../../../images/patterns/cards/flyweight-minie3d8.png?id=422ca8d2f90614dce810a8812c626698){srcset="/images/patterns/cards/flyweight-mini-2x.png?id=857ca676fff84317bb0dab76abfce08e 2x"}
:::

# **Flyweight** em TypeScript {#flyweight-em-typescript .pattern-example-title-block-title}
::::

::::::::::: pattern-example-body
::: pattern-example-brief
O **O Flyweight** é um padrão de projeto estrutural que permite que os
programas suportem grandes quantidades de objetos, mantendo baixo o
consumo de memória.

O padrão consegue isso compartilhando partes do estado do objeto entre
vários objetos. Em outras palavras, o Flyweight economiza RAM
armazenando em cache os mesmos dados usados por objetos diferentes.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Saiba mais sobre o Flyweight ](../../flyweight.html){.btn .btn-lg
.btn-primary}
:::

:::::::: {.sidebar-navigation .with-scroll}
::: en-title
Navegação
:::

::: en-intro
 [Introdução](#)
:::

::: en-intro
 [Exemplo conceitual](#example-0)
:::

::: en-file
 [index](#example-0--index-ts)
:::

::: en-file
 [Output](#example-0--Output-txt)
:::
::::::::

**Complexidade:** []{.fa-stars}

**Popularidade:** []{.fa-stars}

**Exemplos de uso:** O padrão Flyweight tem uma única finalidade:
minimizar a entrada de memória. Se o seu programa não apresentar
problemas de falta de RAM, você poderá ignorar esse padrão por um tempo.

**Identificação:** O Flyweight pode ser reconhecido por um método de
criação que retorna objetos em cache em vez de criar novos.
:::::::::::

[]{#example-0}

## Exemplo conceitual {#example-0-title}

Este exemplo ilustra a estrutura do padrão de projeto **Flyweight**. Ele
se concentra em responder a estas perguntas:

-   De quais classes ele consiste?
-   Quais papéis essas classes desempenham?
-   De que maneira os elementos do padrão estão relacionados?

#### []{#example-0--index-ts .anchor} **index.ts:** Exemplo conceitual

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

#### []{#example-0--Output-txt .anchor} **Output.txt:** Resultados da execução

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
#### Leia a seguir

[Proxy em TypeScript []{.fa
.fa-arrow-right}](../../proxy/typescript/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Voltar

[[]{.fa .fa-arrow-left} Facade em
TypeScript](../../facade/typescript/example.html){.btn .btn-default
rel="prev"}
:::

## **Flyweight** em outras linguagens

[![Flyweight em
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Flyweight em C#"){.prog-lang-link}
[![Flyweight em
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Flyweight em C++"){.prog-lang-link}
[![Flyweight em
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Flyweight em Go"){.prog-lang-link}
[![Flyweight em
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Flyweight em Java"){.prog-lang-link}
[![Flyweight em
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Flyweight em PHP"){.prog-lang-link}
[![Flyweight em
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Flyweight em Python"){.prog-lang-link}
[![Flyweight em
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Flyweight em Ruby"){.prog-lang-link}
[![Flyweight em
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Flyweight em Rust"){.prog-lang-link}
[![Flyweight em
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Flyweight em Swift"){.prog-lang-link}
:::::::::::::::::

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
:::::::::::::::::::::::
::::::::::::::::::::::::
