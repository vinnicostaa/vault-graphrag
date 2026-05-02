::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [Patrons de
conception](../design-patterns.html){.type} / [Patrons
structurels](structural-patterns.html){.type}
:::

# Composite {#composite .title}

::: aka
Alias : [Arbre d\'objets]{style="display:inline-block"}
:::

::: {.section .intent}
##  Intention {#intent}

**Composite** est un patron de conception structurel qui permet
d'agencer les objets dans des arborescences afin de pouvoir traiter
celles-ci comme des objets individuels.

<figure class="image">
<img
src="../../images/patterns/content/composite/compositec299.png?id=73bcf0d94db360b636cd745f710d19db"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/composite/composite.png?id=73bcf0d94db360b636cd745f710d19db 640w,/images/patterns/content/composite/composite-1.5x.png?id=098acddafae70b1ec1cb1b05e833c3ae 960w,/images/patterns/content/composite/composite-2x.png?id=8847e6f8e2cb892ed2229faba83bd1b7 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Patron de conception composite" />
</figure>
:::

::: {.section .problem}
##  Problème {#problem}

L'utilisation de ce patron doit être réservée aux applications dont la
structure principale peut être représentée sous la forme d'une
arborescence.

Prenons les deux objets suivants : `Produits` et `Boîtes`. Une boîte
peut contenir plusieurs produits ainsi qu'un certain nombre de boîtes
plus petites. Ces petites boîtes peuvent également contenir quelques
produits ou même d'autres boîtes encore plus petites, et ainsi de suite.

Vous décidez de mettre au point un système de commandes qui utilise ces
classes. Les commandes peuvent être composées de produits simples sans
emballage, d'autres boîtes remplies de produits\... et d'autres boîtes.
Comment allez-vous déterminer le coût total d'une telle commande ?

<figure class="image">
<img
src="../../images/patterns/diagrams/composite/problem-fr5d1a.png?id=16882f793d754179a18458b6426b36bb"
style="aspect-ratio: 1;"
srcset="/images/patterns/diagrams/composite/problem-fr.png?id=16882f793d754179a18458b6426b36bb 370w,/images/patterns/diagrams/composite/problem-fr-1.5x.png?id=17e547445ad1c4c22607c2aa9e4dca77 555w,/images/patterns/diagrams/composite/problem-fr-2x.png?id=21910b9a5da4903f54c68ccb81d2fbd6 740w"
sizes="(max-width: 720px) 100vw, 370px" loading="lazy" width="370"
alt="Structure d’une commande complexe" />
<figcaption><p>Une commande peut contenir divers produits empaquetés à
l’intérieur de boîtes, elles-mêmes rangées dans de plus grosses boîtes,
etc. La structure complète ressemble à un
arbre inversé.</p></figcaption>
</figure>

Vous pouvez tenter l'approche directe qui consiste à déballer toutes les
boîtes, prendre chaque produit et en faire la somme pour obtenir le
total. Ce mode de calcul peut facilement se mettre en place dans le
monde réel mais dans un programme, ce n'est pas aussi simple que de
créer une boucle. Il faut connaître à l'avance la classe des `Produits`
et des `Boîtes` que l'on parcourt, le niveau d'imbrication des boîtes
ainsi que d'autres détails. Tout ceci rend l'approche directe assez
compliquée et même parfois impossible.
:::

::: {.section .solution}
##  Solution

Le patron de conception composite vous propose de manipuler les
`Produits` et les `Boîtes` à l'aide d'une interface qui déclare une
méthode de calcul du prix total.

Comment cette méthode peut-elle fonctionner ? Pour un produit, on
retourne simplement son prix. Pour une boîte, on parcourt chacun de ses
objets, on leur demande leur prix, puis on retourne un total pour la
boîte. Si l'un de ces objets est une boîte plus petite, cette dernière
va aussi parcourir son propre contenu et ainsi de suite, jusqu'à ce que
tous les prix aient été calculés. Une boîte peut même ajouter des frais
supplémentaires, comme le prix de l'emballage.

<figure class="image">
<img
src="../../images/patterns/content/composite/composite-comic-1-fr7006.png?id=b318eb1564d5ce4f75a66faa97e1ca6f"
style="aspect-ratio: 2;"
srcset="/images/patterns/content/composite/composite-comic-1-fr.png?id=b318eb1564d5ce4f75a66faa97e1ca6f 600w,/images/patterns/content/composite/composite-comic-1-fr-1.5x.png?id=2da1aa04cdf57a1252814cc29fc12f14 900w,/images/patterns/content/composite/composite-comic-1-fr-2x.png?id=a8914cf5ab8a2bc11c9b80b35596bc61 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Solution proposée par le patron de conception composite" />
<figcaption><p>Le patron de conception composite emploie une méthode
récursive afin de parcourir tous les composants
d’une arborescence.</p></figcaption>
</figure>

La cerise sur le gâteau est que vous n'avez même pas besoin de connaître
la classe concrète des objets de l'arborescence. Vous n'avez pas besoin
de savoir si un objet est un produit tout simple ou une boîte
sophistiquée, vous les manipulez de la même manière grâce à une
interface commune. Lorsque vous faites appel à une méthode, les objets
s'occupent de faire transiter la requête en descendant vers les feuilles
de l'arbre.
:::

::: {.section .analogy}
##  Analogie {#analogy}

<figure class="image">
<img
src="../../images/patterns/diagrams/composite/live-examplee724.png?id=548a7cec45b493af66e8bfe524a137d3"
style="aspect-ratio: 1.22;"
srcset="/images/patterns/diagrams/composite/live-example.png?id=548a7cec45b493af66e8bfe524a137d3 280w,/images/patterns/diagrams/composite/live-example-1.5x.png?id=2129195f4103a5a4d3440a086ce3dc87 420w,/images/patterns/diagrams/composite/live-example-2x.png?id=b555458f20fc30425ae6dada5da492af 560w"
sizes="(max-width: 720px) 100vw, 280px" loading="lazy" width="280"
alt="Exemple d’une structure militaire" />
<figcaption><p>Un exemple de structure militaire.</p></figcaption>
</figure>

En général, les armées d'un pays sont structurées en hiérarchies. Une
armée comporte plusieurs divisions, une division est composée de
brigades, une brigade est composée de compagnies, qui peuvent
elles-mêmes être divisées en escouades. Pour finir, une escouade est un
petit groupe de soldats. Les ordres sont donnés au sommet de la
hiérarchie et passés au niveau directement inférieur à chaque soldat,
qui sait quoi en faire.
:::

::::: {.section .structure-container}
##  Structure

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/composite/structure-fr2d97.png?id=e41692527596ede08b8ab82668956e99"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 0.82;"
srcset="/images/patterns/diagrams/composite/structure-fr.png?id=e41692527596ede08b8ab82668956e99 360w,/images/patterns/diagrams/composite/structure-fr-1.5x.png?id=718462fdda8e570338ac4bfc45e17f92 540w,/images/patterns/diagrams/composite/structure-fr-2x.png?id=6aa9ad34817a1dbea45ca64a7366b1d6 720w"
sizes="(max-width: 720px) 100vw, 360px" loading="lazy" width="360"
alt="Structure du patron de conception composite" /><img
src="../../images/patterns/diagrams/composite/structure-fr-indexedb0f9.png?id=7b606c5349276db8aaff43279db06d2a"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 0.85;"
srcset="/images/patterns/diagrams/composite/structure-fr-indexed.png?id=7b606c5349276db8aaff43279db06d2a 400w,/images/patterns/diagrams/composite/structure-fr-indexed-1.5x.png?id=929322df6dabd9e4e7d412967fed49d7 600w,/images/patterns/diagrams/composite/structure-fr-indexed-2x.png?id=5e815e3a1a5c098cda12e92f4c27ab04 800w"
sizes="(max-width: 720px) 100vw, 400px" loading="lazy" width="400"
alt="Structure du patron de conception composite" />
</figure>
:::

1.  L'interface **Composant** décrit les opérations communes aux objets
    simples et complexes de l'arborescence.

2.  Une **Feuille** est un élément de base d'une branche qui n'a pas de
    sous-élément.

    En général les feuilles font le plus gros du travail, car elles
    n'ont personne à qui le déléguer.

3.  Le **Conteneur** (alias *composite*) est un élément composé de
    sous-éléments : des feuilles ou d'autres conteneurs. Un conteneur ne
    connait pas les classes de ses enfants. Il passe par l'interface
    composant pour interagir avec ses sous-éléments.

    Lorsqu'il reçoit une requête, un conteneur délègue la tâche à ses
    sous-éléments, traite les résultats intermédiaires, puis renvoie le
    résultat final au client.

4.  Le **Client** manipule les éléments depuis l'interface composant, ce
    qui lui permet de fonctionner de la même manière pour les éléments
    simples et complexes de l'arborescence.
::::
:::::

::: {.section .pseudocode}
##  Pseudo-code {#pseudocode}

Dans cet exemple, le patron de conception **Composite** nous permet de
gérer des imbrications de formes géométriques dans un éditeur graphique.

<figure class="image">
<img
src="../../images/patterns/diagrams/composite/example7ea7.png?id=98ba81d07c979038dd2ebf3c83a2e19f"
style="aspect-ratio: 0.84;"
srcset="/images/patterns/diagrams/composite/example.png?id=98ba81d07c979038dd2ebf3c83a2e19f 370w,/images/patterns/diagrams/composite/example-1.5x.png?id=ae008efd4146628cd7303925d4ce90be 555w,/images/patterns/diagrams/composite/example-2x.png?id=d21edef39d3792e8a4c6736727ac7305 740w"
sizes="(max-width: 720px) 100vw, 370px" loading="lazy" width="370"
alt="Structure de l’exemple utilisé pour le patron de conception composite" />
<figcaption><p>Exemple de l’éditeur des
formes géométriques.</p></figcaption>
</figure>

La classe `CompositionGraphique` est un conteneur doté d'un certain
nombre de formes, incluant même des formes composées. Une forme composée
possède les mêmes méthodes qu'une forme simple. Mais plutôt que de tout
gérer toute seule, une forme composée envoie une requête récursive à
tous ses enfants et « totalise » le résultat.

Le code client manipule ces formes en passant par l'interface commune.
De ce fait, le client ne sait jamais s'il est en train de manipuler une
forme ou une composition. Le client peut manipuler des structures
d'objets très complexes sans jamais être couplé avec les classes
concrètes de cette structure.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// L’interface du composant déclare des opérations communes pour
// les objets simples et complexes d’une composition.
interface Graphic is
    method move(x, y)
    method draw()

// La classe feuille représente les objets finaux d’une
// composition. Un objet feuille ne peut pas avoir de sous-
// objets. En général, ce sont les feuilles qui lancent les
// traitements. Les objets composite ne font que déléguer le
// travail à leurs sous-composants.
class Dot implements Graphic is
    field x, y

    constructor Dot(x, y) { ... }

    method move(x, y) is
        this.x += x, this.y += y

    method draw() is
        // Dessine un point aux coordonnées X et Y.

// Toutes les classes composant peuvent étendre d’autres
// composants.
class Circle extends Dot is
    field radius

    constructor Circle(x, y, radius) { ... }

    method draw() is
        // Trace un cercle de rayon R aux coordonnées X et Y.

// La classe composite représente les composants complexes qui
// peuvent avoir des enfants. Les objets composite délèguent
// généralement les tâches à leurs enfants et « additionnent »
// ensuite le résultat.
class CompoundGraphic implements Graphic is
    field children: array of Graphic

    // Un composite peut ajouter ou retirer d’autres composants
    // (simples ou complexes) de la liste de ses enfants.
    method add(child: Graphic) is
        // Ajoute un enfant au tableau d’enfants.

    method remove(child: Graphic) is
        // Retire un enfant du tableau d’enfants.

    method move(x, y) is
        foreach (child in children) do
            child.move(x, y)

    // Un composite exécute sa logique principale d’une certaine
    // manière : il parcourt récursivement tous ses enfants,
    // puis récupère et additionne leurs résultats. L’objet est
    // entièrement parcouru, car les enfants du composite
    // passent ces appels à leurs propres enfants et ainsi de
    // suite.
    method draw() is
        // 1. Pour chaque composant enfant :
        //     - Dessine le composant.
        //     - Met à jour le rectangle de délimitation.
        // 2. Dessine un rectangle en pointillé en utilisant les
        // coordonnées de la délimitation.


// Le code client manipule les composants grâce à leur interface
// de base. Ainsi, le code client peut aussi bien prendre en
// charge les composants simples que les complexes.
class ImageEditor is
    field all: CompoundGraphic

    method load() is
        all = new CompoundGraphic()
        all.add(new Dot(1, 2))
        all.add(new Circle(5, 3, 10))
        // ...

    // Combine les composants sélectionnés en un seul composant
    // complexe.
    method groupSelected(components: array of Graphic) is
        group = new CompoundGraphic()
        foreach (component in components) do
            group.add(component)
            all.remove(component)
        all.add(group)
        // Tous les composants vont être dessinés.
        all.draw()</code></pre>
</figure>
:::

:::::::: {.section .applicability-container}
##  Possibilités d'application {#applicability}

::::::: applicability
::: applicability-problem
Utilisez le composite si vous devez gérer une structure d'objets qui
ressemble à une arborescence.
:::

::: applicability-solution
Le patron de conception composite vous propose deux éléments de base qui
partagent la même interface : des feuilles simples et des conteneurs
complexes. Un conteneur peut être composé de feuilles et d'autres
conteneurs. Grâce à cela, vous pouvez construire une structure récursive
composée d'objets imbriqués qui ressemble à un arbre.
:::

::: applicability-problem
Utilisez ce patron si vous voulez que le client interagisse avec les
éléments simples aussi bien que complexes de façon uniforme.
:::

::: applicability-solution
Tous les éléments définis dans le patron composite partagent une
interface commune. En utilisant cette interface, le client n'a pas
besoin de connaître la classe concrète des objets qu'il manipule.
:::
:::::::
::::::::

::: {.section .checklist}
##  Mise en œuvre {#checklist}

1.  Assurez-vous que votre application possède bien la forme d'une
    arborescence. Décomposez-la en conteneurs et en éléments simples.
    Rappelez-vous que les conteneurs doivent pouvoir accueillir des
    éléments simples et d'autres conteneurs.

2.  Déclarez l'interface composant avec une liste de méthodes qui
    fonctionnent à la fois avec les composants simples et complexes.

3.  Créez une classe feuille pour représenter les éléments simples. Un
    même programme peut avoir plusieurs classes feuille différentes.

4.  Créez une classe conteneur pour représenter les éléments complexes.
    Fournissez un attribut de type tableau à cette classe, afin de
    stocker les références aux sous-éléments. Ce tableau doit pouvoir
    stocker les feuilles et les conteneurs, assurez-vous donc qu'il est
    bien déclaré avec le type d'interface du composant.

    Tout en implémentant les méthodes de l'interface composant, gardez
    en tête que les conteneurs sont censés déléguer la majeure partie du
    travail à leurs sous-éléments.

5.  Enfin, définissez des méthodes pour ajouter ou retirer des éléments
    enfants du conteneur.

    Elles peuvent être placées à l'intérieur de l'interface composant,
    mais ceci ne respecte pas le *principe de ségrégation des
    interfaces*, car la classe feuille contiendra des méthodes vides.
    L'avantage est que le client pourra traiter ces éléments
    uniformément, même lorsqu'il construit l'arborescence.
:::

:::::: {.section .pros-cons}
##  Avantages et inconvénients {#pros-cons}

::::: row
::: col-sm-6
-    Vous pouvez travailler dans des structures arborescentes complexes
    plus facilement en utilisant les avantages du polymorphisme et de la
    récursivité.
-    *Principe ouvert/fermé*. Vous pouvez introduire de nouveaux types
    d'éléments dans l'application qui pourront directement être intégrés
    dans l'arborescence, sans avoir à réécrire l'existant.
:::

::: col-sm-6
-    Vous rencontrerez parfois des difficultés pour définir une
    interface commune à certaines classes dont les fonctionnalités sont
    trop différentes. Dans certains scénarios, vous devez créer une
    interface composant bien trop générique, rendant le fonctionnement
    difficile à comprendre.
:::
:::::
::::::

::: {.section .relations}
##  Liens avec les autres patrons {#relations}

-   Vous pouvez utiliser le [Monteur](builder.html) lorsque vous créez
    des arbres [Composites](composite.html) complexes, car vous pouvez
    programmer les étapes de la construction récursivement.

-   La [Chaîne de Responsabilité](chain-of-responsibility.html) est
    souvent utilisée en conjonction avec le [Composite](composite.html).
    Dans ce cas, lorsqu'une feuille reçoit une demande, elle la passe le
    long de la chaîne de ses composants parent, jusqu'à la racine de
    l'arborescence.

-   Vous pouvez utiliser les [Itérateurs](iterator.html) pour parcourir
    des arbres [Composites](composite.html).

-   Vous pouvez utiliser le [Visiteur](visitor.html) pour lancer une
    opération sur un arbre [Composite](composite.html) entier.

-   Vous pouvez transformer des nœuds de feuilles de l'arbre
    [Composite](composite.html) en [Poids mouches](flyweight.html) et
    les partager pour économiser de la RAM.

-   Le [Composite](composite.html) et le [Décorateur](decorator.html)
    ont des diagrammes de structure similaires puisqu'ils reposent sur
    la composition récursive pour organiser un nombre variable d'objets.

    Un *décorateur* est comme un *composite*, mais avec un seul
    composant enfant. Il y a une autre différence importante : Le
    *décorateur* ajoute des responsabilités supplémentaires à l'objet
    emballé, alors que le *composite* se contente d'« additionner » les
    résultats de ses enfants.

    Mais ces patrons de conception peuvent également coopérer : vous
    pouvez utiliser le *décorateur* pour étendre le comportement d'un
    objet spécifique d'un arbre *Composite*.

-   Les conceptions qui reposent énormément sur le
    [Composite](composite.html) et le [Décorateur](decorator.html)
    tirent des avantages à utiliser le [Prototype](prototype.html). Il
    vous permet de cloner les structures complexes plutôt que de les
    reconstruire à partir de rien.
:::

::: {.section .implementations}
##  Exemples de code {#implementations}

[![Composite en
C#](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](composite/csharp/example.html "Composite en C#"){.prog-lang-link}
[![Composite en
C++](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](composite/cpp/example.html "Composite en C++"){.prog-lang-link}
[![Composite en
Go](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](composite/go/example.html "Composite en Go"){.prog-lang-link}
[![Composite en
Java](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](composite/java/example.html "Composite en Java"){.prog-lang-link}
[![Composite en
PHP](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](composite/php/example.html "Composite en PHP"){.prog-lang-link}
[![Composite en
Python](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](composite/python/example.html "Composite en Python"){.prog-lang-link}
[![Composite en
Ruby](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](composite/ruby/example.html "Composite en Ruby"){.prog-lang-link}
[![Composite en
Rust](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](composite/rust/example.html "Composite en Rust"){.prog-lang-link}
[![Composite en
Swift](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](composite/swift/example.html "Composite en Swift"){.prog-lang-link}
[![Composite en
TypeScript](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](composite/typescript/example.html "Composite en TypeScript"){.prog-lang-link}
:::

:::::: {#book-promo .banner-set2}
::::: {.prom .banner-content .banner-bg .banner-striped .banner-with-image data-id="DP: 1: Support our free website and own the eBook!" creative-id="standard-fr" position="content_bottom"}
::: banner-image
[![](../../images/patterns/banners/patterns-book-banner-3a29e.png?id=7d445df13c80287beaab234b4f3b698c){width="200"
height="200" loading="lazy"
srcset="/images/patterns/banners/patterns-book-banner-3-2x.png?id=0cc3f77ab421d1a5c02ee46488231c3a 2x"}](book.html)
:::

::: banner-text
### Soutenez notre site gratuit et obtenez l'ebook ! {#soutenez-notre-site-gratuit-et-obtenez-lebook .title}

-   22 patrons de conception et 8 principes expliqués en profondeur.
-   444 pages de vulgarisation, bien structurées et faciles à lire.
-   225 illustrations et diagrammes clairs pour aider à la
    compréhension.
-   Une archive avec des exemples de code dans 11 langages.
-   Tous les appareils sont pris en charge : formats PDF/EPUB/MOBI/KFX.

[ En savoir plus...](book.html){.btn .btn-secondary}
:::
:::::
::::::

::: next
#### Suivant

[Décorateur []{.fa .fa-arrow-right}](decorator.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Retour

[[]{.fa .fa-arrow-left} Pont](bridge.html){.btn .btn-default rel="prev"}
:::
::::::::::::::::::::::::::::::::

::::::: {.prom .banner-sidebar .banner-removable .banner-removable-patterns data-id="DP: Sidebar" creative-id="standard-sidebar-fr" position="sidebar"}
:::::: banner-inner
:::: image3d-book-right
::: {.image3d-cover style="background: #0b3752;"}
[![](../../images/patterns/book/web-cover-fre2bc.png?id=d908eb2db12c9d431826b573417bff08){width="250"
height="375" loading="lazy"
srcset="/images/patterns/book/web-cover-fr-2x.png?id=3d1335fce1b6dd01003f33aeddae05c4 2x"}](book.html)
:::
::::

::: {style="margin-top: 1rem"}
Cet article est tiré de notre ebook\
**Plongée au cœur des\
PATRONS DE CONCEPTION**

[ En savoir plus...](book.html){.btn .btn-secondary .btn-block}
:::
::::::
:::::::
::::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::::
