::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Patrons de
conception](../../../design-patterns.html){.type} / [Fabrique
abstraite](../../abstract-factory.html){.type} /
[Go](../../go.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Fabrique
abstraite](../../../../images/patterns/cards/abstract-factory-minid1c5.png?id=4c3927c446313a38ce77dfee38111e27){srcset="/images/patterns/cards/abstract-factory-mini-2x.png?id=22236aaa65ff52cbde1c713216d52c1f 2x"}
:::

# **Fabrique abstraite** en Go {#fabrique-abstraite-en-go .pattern-example-title-block-title}
::::

:::::::::::::::::::: pattern-example-body
::: pattern-example-brief
La **Fabrique abstraite** est un patron de conception de création qui
permet de créer des familles de produits complètes sans avoir à préciser
leurs classes concrètes.

La fabrique abstraite définit une interface pour la création de chaque
produit, mais délègue la véritable création des produits aux classes
concrètes de la fabrique. Chaque type de fabrique correspond à une
certaine variété de produits.

Le code client appelle les méthodes de création d'un objet Fabrique
plutôt que de créer directement les produits à l'aide d'un constructeur
(opérateur `new`). Comme chaque fabrique possède sa propre variante de
produit, tous ses produits seront compatibles.

Le code client manipule les fabriques et les produits uniquement via
leurs interfaces abstraites, ce qui lui permet de travailler avec
n'importe quelle variante de produit créée par un objet Fabrique. Créez
juste une nouvelle classe concrète Fabrique et passez-la au code client.

> Lisez notre [Comparaison des fabriques](../../factory-comparison.html)
> si vous avez du mal à comprendre la différence entre les divers
> concepts et patrons.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ En savoir plus sur la patron Fabrique abstraite
](../../abstract-factory.html){.btn .btn-lg .btn-primary}
:::

::::::::::::::::: {.sidebar-navigation .with-scroll}
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
 [i­Sports­Factory](#example-0--iSportsFactory-go)
:::

::: en-file
 [adidas](#example-0--adidas-go)
:::

::: en-file
 [nike](#example-0--nike-go)
:::

::: en-file
 [i­Shoe](#example-0--iShoe-go)
:::

::: en-file
 [adidas­Shoe](#example-0--adidasShoe-go)
:::

::: en-file
 [nike­Shoe](#example-0--nikeShoe-go)
:::

::: en-file
 [i­Shirt](#example-0--iShirt-go)
:::

::: en-file
 [adidas­Shirt](#example-0--adidasShirt-go)
:::

::: en-file
 [nike­Shirt](#example-0--nikeShirt-go)
:::

::: en-file
 [main](#example-0--main-go)
:::

::: en-file
 [output](#example-0--output-txt)
:::
:::::::::::::::::
::::::::::::::::::::

[]{#example-0}

## Exemple conceptuel {#example-0-title}

Prenons le cas où vous voulez acheter des affaires de sport et plus
précisément un ensemble de deux produits : une paire de chaussures et un
maillot. Vous aimeriez acquérir une tenue assortie de la même marque.

Pour transformer tout ceci en code, la fabrique abstraite va nous aider
à créer des ensembles de produits qui seraient systématiquement
assortis.

#### []{#example-0--iSportsFactory-go .anchor} **iSportsFactory.go:** Interface de la fabrique abstraite

<figure class="code">
<pre class="code" lang="go"><code>package main

import &quot;fmt&quot;

type ISportsFactory interface {
    makeShoe() IShoe
    makeShirt() IShirt
}

func GetSportsFactory(brand string) (ISportsFactory, error) {
    if brand == &quot;adidas&quot; {
        return &amp;Adidas{}, nil
    }

    if brand == &quot;nike&quot; {
        return &amp;Nike{}, nil
    }

    return nil, fmt.Errorf(&quot;Wrong brand type passed&quot;)
}</code></pre>
</figure>

#### []{#example-0--adidas-go .anchor} **adidas.go:** Fabrique concrète

<figure class="code">
<pre class="code" lang="go"><code>package main

type Adidas struct {
}

func (a *Adidas) makeShoe() IShoe {
    return &amp;AdidasShoe{
        Shoe: Shoe{
            logo: &quot;adidas&quot;,
            size: 14,
        },
    }
}

func (a *Adidas) makeShirt() IShirt {
    return &amp;AdidasShirt{
        Shirt: Shirt{
            logo: &quot;adidas&quot;,
            size: 14,
        },
    }
}</code></pre>
</figure>

#### []{#example-0--nike-go .anchor} **nike.go:** Fabrique concrète

<figure class="code">
<pre class="code" lang="go"><code>package main

type Nike struct {
}

func (n *Nike) makeShoe() IShoe {
    return &amp;NikeShoe{
        Shoe: Shoe{
            logo: &quot;nike&quot;,
            size: 14,
        },
    }
}

func (n *Nike) makeShirt() IShirt {
    return &amp;NikeShirt{
        Shirt: Shirt{
            logo: &quot;nike&quot;,
            size: 14,
        },
    }
}</code></pre>
</figure>

#### []{#example-0--iShoe-go .anchor} **iShoe.go:** Produit abstrait

<figure class="code">
<pre class="code" lang="go"><code>package main

type IShoe interface {
    setLogo(logo string)
    setSize(size int)
    getLogo() string
    getSize() int
}

type Shoe struct {
    logo string
    size int
}

func (s *Shoe) setLogo(logo string) {
    s.logo = logo
}

func (s *Shoe) getLogo() string {
    return s.logo
}

func (s *Shoe) setSize(size int) {
    s.size = size
}

func (s *Shoe) getSize() int {
    return s.size
}</code></pre>
</figure>

#### []{#example-0--adidasShoe-go .anchor} **adidasShoe.go:** Produit concret

<figure class="code">
<pre class="code" lang="go"><code>package main

type AdidasShoe struct {
    Shoe
}</code></pre>
</figure>

#### []{#example-0--nikeShoe-go .anchor} **nikeShoe.go:** Produit concret

<figure class="code">
<pre class="code" lang="go"><code>package main

type NikeShoe struct {
    Shoe
}</code></pre>
</figure>

#### []{#example-0--iShirt-go .anchor} **iShirt.go:** Produit abstrait

<figure class="code">
<pre class="code" lang="go"><code>package main

type IShirt interface {
    setLogo(logo string)
    setSize(size int)
    getLogo() string
    getSize() int
}

type Shirt struct {
    logo string
    size int
}

func (s *Shirt) setLogo(logo string) {
    s.logo = logo
}

func (s *Shirt) getLogo() string {
    return s.logo
}

func (s *Shirt) setSize(size int) {
    s.size = size
}

func (s *Shirt) getSize() int {
    return s.size
}</code></pre>
</figure>

#### []{#example-0--adidasShirt-go .anchor} **adidasShirt.go:** Produit concret

<figure class="code">
<pre class="code" lang="go"><code>package main

type AdidasShirt struct {
    Shirt
}</code></pre>
</figure>

#### []{#example-0--nikeShirt-go .anchor} **nikeShirt.go:** Produit concret

<figure class="code">
<pre class="code" lang="go"><code>package main

type NikeShirt struct {
    Shirt
}</code></pre>
</figure>

#### []{#example-0--main-go .anchor} **main.go:** Code client

<figure class="code">
<pre class="code" lang="go"><code>package main

import &quot;fmt&quot;

func main() {
    adidasFactory, _ := GetSportsFactory(&quot;adidas&quot;)
    nikeFactory, _ := GetSportsFactory(&quot;nike&quot;)

    nikeShoe := nikeFactory.makeShoe()
    nikeShirt := nikeFactory.makeShirt()

    adidasShoe := adidasFactory.makeShoe()
    adidasShirt := adidasFactory.makeShirt()

    printShoeDetails(nikeShoe)
    printShirtDetails(nikeShirt)

    printShoeDetails(adidasShoe)
    printShirtDetails(adidasShirt)
}

func printShoeDetails(s IShoe) {
    fmt.Printf(&quot;Logo: %s&quot;, s.getLogo())
    fmt.Println()
    fmt.Printf(&quot;Size: %d&quot;, s.getSize())
    fmt.Println()
}

func printShirtDetails(s IShirt) {
    fmt.Printf(&quot;Logo: %s&quot;, s.getLogo())
    fmt.Println()
    fmt.Printf(&quot;Size: %d&quot;, s.getSize())
    fmt.Println()
}</code></pre>
</figure>

#### []{#example-0--output-txt .anchor} **output.txt:** Résultat de l'exécution

<figure class="code">
<pre class="code" lang="output"><code>Logo: nike
Size: 14
Logo: nike
Size: 14
Logo: adidas
Size: 14
Logo: adidas
Size: 14</code></pre>
</figure>

::: next
#### Suivant

[Monteur en Go []{.fa
.fa-arrow-right}](../../builder/go/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Retour

[[]{.fa .fa-arrow-left} Les patrons de conception en
Go](../../go.html){.btn .btn-default rel="prev"}
:::

## **Fabrique abstraite** dans les autres langues

[![Fabrique abstraite en
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Fabrique abstraite en C#"){.prog-lang-link}
[![Fabrique abstraite en
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Fabrique abstraite en C++"){.prog-lang-link}
[![Fabrique abstraite en
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Fabrique abstraite en Java"){.prog-lang-link}
[![Fabrique abstraite en
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Fabrique abstraite en PHP"){.prog-lang-link}
[![Fabrique abstraite en
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Fabrique abstraite en Python"){.prog-lang-link}
[![Fabrique abstraite en
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Fabrique abstraite en Ruby"){.prog-lang-link}
[![Fabrique abstraite en
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Fabrique abstraite en Rust"){.prog-lang-link}
[![Fabrique abstraite en
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Fabrique abstraite en Swift"){.prog-lang-link}
[![Fabrique abstraite en
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Fabrique abstraite en TypeScript"){.prog-lang-link}
::::::::::::::::::::::::::

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
::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::
