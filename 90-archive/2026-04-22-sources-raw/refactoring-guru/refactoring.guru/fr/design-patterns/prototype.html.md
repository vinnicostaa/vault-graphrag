:::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [Patrons de
conception](../design-patterns.html){.type} / [Patrons de
création](creational-patterns.html){.type}
:::

# Prototype {#prototype .title}

::: aka
Alias : [Clone]{style="display:inline-block"}
:::

::: {.section .intent}
##  Intention {#intent}

**Prototype** est un patron de conception qui crée de nouveaux objets à
partir d'objets existants sans rendre le code dépendant de leur classe.

<figure class="image">
<img
src="../../images/patterns/content/prototype/prototype3cb9.png?id=e912b1ada20bbf7b2ffc09e93b9fab20"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/prototype/prototype.png?id=e912b1ada20bbf7b2ffc09e93b9fab20 640w,/images/patterns/content/prototype/prototype-1.5x.png?id=a0f76894fb657783b65b9ed899857468 960w,/images/patterns/content/prototype/prototype-2x.png?id=670789c80c8a114e25838ede2da4a881 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Patron de conception prototype" />
</figure>
:::

::: {.section .problem}
##  Problème {#problem}

Nous voulons une copie exacte d'un objet. Comment vous y
prendriez-vous ? Tout d'abord, vous devez créer un nouvel objet de cette
même classe. Ensuite, vous devez parcourir tous les attributs de l'objet
d'origine et copier leurs valeurs dans le nouvel objet.

Super ! Mais il y a un hic. Certains objets ne peuvent pas être copiés
ainsi, car leurs attributs peuvent être privés et ne pas être visibles
en dehors de l'objet.

<figure class="image">
<img
src="../../images/patterns/content/prototype/prototype-comic-1-fr9833.png?id=7aeba6300c286e63c3f48578a81a2236"
style="aspect-ratio: 2;"
srcset="/images/patterns/content/prototype/prototype-comic-1-fr.png?id=7aeba6300c286e63c3f48578a81a2236 600w,/images/patterns/content/prototype/prototype-comic-1-fr-1.5x.png?id=fb7fd2933979aa0dc55f0b737e626c8c 900w,/images/patterns/content/prototype/prototype-comic-1-fr-2x.png?id=2b6e69a5d1a4c910a3bbc752b3e46a45 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Ce qui peut mal tourner lorsque l’on construit un objet « depuis l’extérieur »" />
<figcaption><p>Copier un objet « depuis l’extérieur » <a
href="https://vimeo.com/120816425">n’est pas</a>
toujours possible.</p></figcaption>
</figure>

Un autre problème se profile avec l'approche directe. Étant donné que
vous devez connaître la classe de l'objet pour le dupliquer, votre code
devient dépendant de cette classe. Même si cette dépendance ne vous
effraie pas, il y a un autre hic. Parfois, vous ne connaissez que
l'interface implémentée par l'objet, mais pas sa classe concrète. Si le
paramètre d'une méthode accepte tous les objets qui implémentent une
interface donnée, vous pouvez rencontrer des problèmes.
:::

::: {.section .solution}
##  Solution

Le patron de conception prototype délègue le processus de clonage aux
objets qui vont être copiés. Il déclare une interface commune pour tous
les objets qui pourront être clonés. Cette interface vous permet de
cloner un objet sans coupler votre code à la classe de cet objet. En
général, une telle interface ne contient qu'une seule méthode `clone`.

L'implémentation de cette méthode se ressemble dans toutes les classes.
Elle crée un objet de la classe actuelle et reporte les valeurs des
attributs de l'objet d'origine dans le nouveau. Vous pouvez également
copier des attributs privés, car la plupart des langages de
programmation autorisent l'accès à des objets d'une même classe.

Un objet qui peut être cloné est appelé un *prototype*. Lorsque vos
objets possèdent des dizaines d'attributs et des centaines de
configurations possibles, le clonage est une alternative au
sous-classage.

<figure class="image">
<img
src="../../images/patterns/content/prototype/prototype-comic-2-fr0557.png?id=dc8f9e7450a49406a25a1ce5ae66b2a2"
style="aspect-ratio: 1.14;"
srcset="/images/patterns/content/prototype/prototype-comic-2-fr.png?id=dc8f9e7450a49406a25a1ce5ae66b2a2 343w,/images/patterns/content/prototype/prototype-comic-2-fr-1.5x.png?id=4e4a70fb274b9d9327c4d397bfd0eca9 515w,/images/patterns/content/prototype/prototype-comic-2-fr-2x.png?id=aafd2e03c4bfd7773f5d85c77b9d987f 687w"
sizes="(max-width: 720px) 100vw, 343px" loading="lazy" width="343"
alt="Prototypes préconstruits" />
<figcaption><p>Les prototypes préconstruits sont une alternative
au sous-classage.</p></figcaption>
</figure>

Leur fonctionnement est le suivant : vous créez un ensemble d'objets
configurés différemment. Dès que vous avez besoin de l'un de ces objets,
vous clonez un prototype plutôt que d'en construire un nouveau.
:::

::: {.section .analogy}
##  Analogie {#analogy}

Dans la vie réelle, des prototypes sont utilisés pour exécuter des tests
avant de lancer les chaînes de production d'un produit. Dans ce cas
toutefois, les prototypes ne font pas partie intégrante de la
fabrication, ils ne jouent qu'un rôle passif.

<figure class="image">
<img
src="../../images/patterns/content/prototype/prototype-comic-3-fr752b.png?id=ecb26cadc5647a7e53e4d0cdaed9254c"
style="aspect-ratio: 2;"
srcset="/images/patterns/content/prototype/prototype-comic-3-fr.png?id=ecb26cadc5647a7e53e4d0cdaed9254c 600w,/images/patterns/content/prototype/prototype-comic-3-fr-1.5x.png?id=54250e3d08529703805ac89aa7807352 900w,/images/patterns/content/prototype/prototype-comic-3-fr-2x.png?id=93f2d3a01655528e489901f6fc087a59 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="La division cellulaire" />
<figcaption><p>La division cellulaire.</p></figcaption>
</figure>

Puisque les prototypes industriels ne se clonent pas eux-mêmes, étudions
le processus de la mitose d'une cellule (petit rappel de vos cours de
biologie). Une fois la division mitotique terminée, une paire de
cellules identiques est créée. La cellule originale sert de prototype et
joue un rôle actif dans la création de la copie.
:::

:::::::: {.section .structure-container}
##  Structure

::::::: structure
#### Implémentation de base

:::: prototype-normal
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/prototype/structure87f6.png?id=088102c5e9785ff45debbbce86f4df81"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.25;"
srcset="/images/patterns/diagrams/prototype/structure.png?id=088102c5e9785ff45debbbce86f4df81 500w,/images/patterns/diagrams/prototype/structure-1.5x.png?id=beec6a1a5242268e10e39f9fdc0b894b 750w,/images/patterns/diagrams/prototype/structure-2x.png?id=ba75079f42f08028ae4cdbda0cfecc26 1000w"
sizes="(max-width: 720px) 100vw, 500px" loading="lazy" width="500"
alt="La structure du patron de conception prototype" /><img
src="../../images/patterns/diagrams/prototype/structure-indexed6499.png?id=0e1c809842f5c43aca0541a2eba1f844"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.27;"
srcset="/images/patterns/diagrams/prototype/structure-indexed.png?id=0e1c809842f5c43aca0541a2eba1f844 520w,/images/patterns/diagrams/prototype/structure-indexed-1.5x.png?id=05b072b5b83f5ff1024a7ba448ea9d59 780w,/images/patterns/diagrams/prototype/structure-indexed-2x.png?id=74584ac729c0f6b49d2a97a53cc266ff 1040w"
sizes="(max-width: 720px) 100vw, 520px" loading="lazy" width="520"
alt="La structure du patron de conception prototype" />
</figure>
:::
::::

1.  L'interface **Prototype** déclare les méthodes de clonage. Dans la
    majorité des cas, il n'y a qu'une seule méthode `clone`.

2.  La classe **Prototype Concret** implémente la méthode de clonage. En
    plus de copier les données de l'objet original dans le clone, cette
    méthode peut gérer des cas particuliers du processus de clonage,
    comme des objets imbriqués, démêler les dépendances récursives, etc.

3.  Le **Client** peut produire une copie de n'importe quel objet
    implémentant l'interface prototype.

#### Implémentation du registre de prototypes

:::: prototype-pool
::: {.struct-image2 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/prototype/structure-prototype-cache9d19.png?id=609c2af5d14ed55dcbb218a00f98e7d5"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.15;"
srcset="/images/patterns/diagrams/prototype/structure-prototype-cache.png?id=609c2af5d14ed55dcbb218a00f98e7d5 550w,/images/patterns/diagrams/prototype/structure-prototype-cache-1.5x.png?id=8ca0b829185d49c5acbe19966a0659d4 825w,/images/patterns/diagrams/prototype/structure-prototype-cache-2x.png?id=a1e4514bbcc9b10968b856f19b407105 1100w"
sizes="(max-width: 720px) 100vw, 550px" loading="lazy" width="550"
alt="Le registre de prototypes" /><img
src="../../images/patterns/diagrams/prototype/structure-prototype-cache-indexed1801.png?id=10a4a84a1a318f59dbc2b806fc936d04"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.12;"
srcset="/images/patterns/diagrams/prototype/structure-prototype-cache-indexed.png?id=10a4a84a1a318f59dbc2b806fc936d04 550w,/images/patterns/diagrams/prototype/structure-prototype-cache-indexed-1.5x.png?id=cb56c95533a4020368c48db9f9e2a37d 825w,/images/patterns/diagrams/prototype/structure-prototype-cache-indexed-2x.png?id=47b99eb7ae51158bdbb31deea4f5e98f 1100w"
sizes="(max-width: 720px) 100vw, 550px" loading="lazy" width="550"
alt="Le registre de prototypes" />
</figure>
:::
::::

1.  Le **Registre de Prototypes** facilite l'accès aux prototypes
    fréquemment utilisés. Il stocke un ensemble d'objets préconstruits
    qui ont déjà été copiés. Le registre de prototypes le plus simple
    est une table de hachage (hash map) `nom → prototype`. Si vous
    voulez de meilleurs critères de recherche qu'un simple nom,
    construisez une version plus robuste du registre.
:::::::
::::::::

::: {.section .pseudocode}
##  Pseudo-code {#pseudocode}

Dans cet exemple, le **Prototype** produit des copies exactes d'objets
géométriques, sans coupler le code à leurs classes.

<figure class="image">
<img
src="../../images/patterns/diagrams/prototype/examplee491.png?id=47bc6c1058cb100b81e675b5ca6bda6c"
style="aspect-ratio: 1.42;"
srcset="/images/patterns/diagrams/prototype/example.png?id=47bc6c1058cb100b81e675b5ca6bda6c 470w,/images/patterns/diagrams/prototype/example-1.5x.png?id=067f3a2415370fadf5f92aadaa12b5e2 705w,/images/patterns/diagrams/prototype/example-2x.png?id=80393e0df17ae8130e5ada832d494949 940w"
sizes="(max-width: 720px) 100vw, 470px" loading="lazy" width="470"
alt="La structure du patron de conception prototype dans l’exemple utilisé" />
<figcaption><p>Cloner un ensemble d’objets qui appartiennent à une
hiérarchie de classes.</p></figcaption>
</figure>

Toutes les formes implémentent la même interface, et cette dernière
fournit une méthode de clonage. Une sous-classe peut appeler la méthode
de clonage de son parent avant de copier les valeurs de ses propres
attributs dans l'objet.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// Prototype de base.
abstract class Shape is
    field X: int
    field Y: int
    field color: string

    // Un constructeur normal.
    constructor Shape() is
        // ...

    // Le constructeur du prototype. Un nouvel objet est
    // initialisé avec les valeurs de l’objet existant.
    constructor Shape(source: Shape) is
        this()
        this.X = source.X
        this.Y = source.Y
        this.color = source.color

    // Le traitement de clonage retourne l’une des sous-classes
    // Forme (Shape)
    abstract method clone():Shape


// Prototype concret. La méthode de clonage crée un nouvel objet
// et le passe au constructeur. Il garde une référence à un
// nouveau clone tant que le constructeur n’a pas terminé. Il
// est donc impossible de se retrouver avec un clone incomplet
// et les résultats du clonage sont toujours fiables.
class Rectangle extends Shape is
    field width: int
    field height: int

    constructor Rectangle(source: Rectangle) is
        // Un appel au constructeur parent est requis pour
        // copier les attributs privés définis dans la classe
        // mère.
        super(source)
        this.width = source.width
        this.height = source.height

    method clone():Shape is
        return new Rectangle(this)


class Circle extends Shape is
    field radius: int

    constructor Circle(source: Circle) is
        super(source)
        this.radius = source.radius

    method clone():Shape is
        return new Circle(this)


// Quelque part dans le code client.
class Application is
    field shapes: array of Shape

    constructor Application() is
        Circle circle = new Circle()
        circle.X = 10
        circle.Y = 10
        circle.radius = 20
        shapes.add(circle)

        Circle anotherCircle = circle.clone()
        shapes.add(anotherCircle)
        // La variable `autreCercle` contient la copie exacte de
        // l’objet `Cercle`.

        Rectangle rectangle = new Rectangle()
        rectangle.width = 10
        rectangle.height = 20
        shapes.add(rectangle)

    method businessLogic() is
        // Le prototype est très pratique, car il vous permet de
        // créer une copie d’un objet en ignorant tout de son
        // type.
        Array shapesCopy = new Array of Shapes.

        // Par exemple, nous ne savons pas exactement quels
        // éléments figurent dans le tableau des formes. Nous
        // savons juste que ce sont tous des formes. Grâce au
        // polymorphisme, lorsque nous appelons la méthode
        // `clone` sur une forme, le programme vérifie sa classe
        // et lance la méthode clone appropriée de cette classe.
        // C’est pour cela que nous fabriquons des clones,
        // plutôt qu’un ensemble d’objets forme.
        foreach (s in shapes) do
            shapesCopy.add(s.clone())

        // Le tableau `CopiesFormes` (shapesCopy) contient les
        // copies exactes du tableau des `formes`.</code></pre>
</figure>
:::

:::::::: {.section .applicability-container}
##  Possibilités d'application {#applicability}

::::::: applicability
::: applicability-problem
Utilisez le prototype si vous ne voulez pas que votre code dépende des
classes concrètes des objets que vous clonez.
:::

::: applicability-solution
Vous rencontrerez ce cas si votre code manipule des objets obtenus via
l'interface d'une application externe. Il arrive que votre code ne
puisse pas se baser sur les classes concrètes de ces objets car elles
sont inconnues.

Le prototype procure une interface générale au code client et lui permet
de travailler avec tous les objets clonables. Cette interface rend le
code client indépendant des classes concrètes des objets qu'il clone.
:::

::: applicability-problem
Utilisez ce patron si vous voulez réduire le nombre de sous-classes
quand elles ne diffèrent que dans leur manière d'initialiser leurs
objets respectifs. Ces sous-classes pourraient avoir été prévues pour
créer des objets similaires dans des configurations spécifiques.
:::

::: applicability-solution
Le patron de conception prototype vous permet d'utiliser un ensemble
d'objets préconstruits (des prototypes) de différentes manières.

Plutôt que d'instancier une sous-classe qui correspond à une
configuration précise, le client peut se contenter de chercher le
prototype approprié et de le cloner.
:::
:::::::
::::::::

::: {.section .checklist}
##  Mise en œuvre {#checklist}

1.  Créez l'interface du prototype et déclarez-y la méthode `clone`,
    mais si vous avez une hiérarchie de classes existante, ajoutez juste
    la méthode à toutes ses classes.

2.  Une classe prototype doit définir un autre constructeur capable de
    prendre un objet de cette classe comme paramètre. Le constructeur
    doit copier les valeurs des attributs définis dans la classe de
    l'objet dans l'instance qui vient d'être créée. Si une sous-classe
    est concernée, vous devez appeler le constructeur du parent pour
    gérer le clonage de ses attributs privés.

    Si le langage de programmation que vous utilisez ne gère pas la
    surcharge, vous pouvez définir une méthode spéciale pour copier les
    données de l'objet. Faire tout ceci à l'intérieur du constructeur
    est plus pratique, car il retourne le résultat directement après
    avoir appelé l'opérateur `new`.

3.  La méthode de clonage ne contient qu'une seule ligne de code : un
    opérateur `new` avec la version prototype du constructeur. Notez
    bien que chaque classe doit redéfinir la méthode de clonage et
    utiliser le nom de sa propre classe avec l'opérateur `new`. Si vous
    ne le faites pas, la méthode de clonage sera capable de produire des
    objets de la classe mère.

4.  Vous pouvez également créer un registre centralisé des prototypes
    pour garder un catalogue de prototypes fréquemment utilisés.

    Ce registre peut être implémenté avec une classe fabrique, ou mis
    dans une classe prototype de base avec une méthode statique qui
    récupère le prototype. Cette méthode ira chercher un prototype en
    fonction du critère de recherche que le code client passe à la
    méthode. Ce critère peut être une chaîne de caractères ou un
    ensemble de paramètres de recherche. Dès que le prototype recherché
    est trouvé, le registre crée un clone et le retourne au client.

    Pour finir, remplacez tous les appels directs aux constructeurs des
    sous-classes par des appels à la méthode fabrique du registre de
    prototypes.
:::

:::::: {.section .pros-cons}
##  Avantages et inconvénients {#pros-cons}

::::: row
::: col-sm-6
-    Vous pouvez cloner les objets sans les coupler avec leurs classes
    concrètes.
-    Vous clonez des prototypes préconstruits et vous pouvez vous
    débarrasser du code d'initialisation redondant.
-    Vous pouvez créer des objets complexes plus facilement.
-    Vous obtenez une alternative à l'héritage pour gérer les modèles de
    configuration d'objets complexes.
:::

::: col-sm-6
-    Cloner des objets complexes dotés de références circulaires peut se
    révéler très difficile.
:::
:::::
::::::

::: {.section .relations}
##  Liens avec les autres patrons {#relations}

-   La [Fabrique](factory-method.html) est souvent utilisée dès le début
    de la conception (moins compliquée et plus personnalisée grâce aux
    sous-classes) et évolue vers la [Fabrique
    abstraite](abstract-factory.html), le [Prototype](prototype.html),
    ou le [Monteur](builder.html) (ce dernier étant plus flexible, mais
    plus compliqué).

-   Les classes [Fabrique abstraite](abstract-factory.html) sont souvent
    basées sur un ensemble de [Fabriques](factory-method.html), mais
    vous pouvez également utiliser le [Prototype](prototype.html) pour
    écrire leurs méthodes.

-   Le [Prototype](prototype.html) se révèle utile lorsque vous voulez
    sauvegarder des copies de [Commandes](command.html) dans
    l'historique.

-   Les conceptions qui reposent énormément sur le
    [Composite](composite.html) et le [Décorateur](decorator.html)
    tirent des avantages à utiliser le [Prototype](prototype.html). Il
    vous permet de cloner les structures complexes plutôt que de les
    reconstruire à partir de rien.

-   Le [Prototype](prototype.html) n'est pas basé sur l'héritage, il n'a
    donc pas ses désavantages. Mais le *prototype* requiert une
    initialisation compliquée pour l'objet cloné. La
    [Fabrique](factory-method.html) est basée sur l'héritage, mais n'a
    pas besoin d'une étape d'initialisation.

-   Parfois le [Prototype](prototype.html) peut venir remplacer le
    [Mémento](memento.html) et proposer une solution plus simple. Cela
    n'est possible que si l'objet (l'état que vous voulez stocker dans
    l'historique) est assez direct et ne possède pas de liens vers des
    ressources externes, ou que les liens sont faciles à recréer.

-   Les [Fabriques abstraites](abstract-factory.html),
    [Monteurs](builder.html) et [Prototypes](prototype.html) peuvent
    tous être implémentés comme des [Singletons](singleton.html).
:::

::: {.section .implementations}
##  Exemples de code {#implementations}

[![Prototype en
C#](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](prototype/csharp/example.html "Prototype en C#"){.prog-lang-link}
[![Prototype en
C++](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](prototype/cpp/example.html "Prototype en C++"){.prog-lang-link}
[![Prototype en
Go](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](prototype/go/example.html "Prototype en Go"){.prog-lang-link}
[![Prototype en
Java](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](prototype/java/example.html "Prototype en Java"){.prog-lang-link}
[![Prototype en
PHP](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](prototype/php/example.html "Prototype en PHP"){.prog-lang-link}
[![Prototype en
Python](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](prototype/python/example.html "Prototype en Python"){.prog-lang-link}
[![Prototype en
Ruby](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](prototype/ruby/example.html "Prototype en Ruby"){.prog-lang-link}
[![Prototype en
Rust](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](prototype/rust/example.html "Prototype en Rust"){.prog-lang-link}
[![Prototype en
Swift](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](prototype/swift/example.html "Prototype en Swift"){.prog-lang-link}
[![Prototype en
TypeScript](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](prototype/typescript/example.html "Prototype en TypeScript"){.prog-lang-link}
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

[Singleton []{.fa .fa-arrow-right}](singleton.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Retour

[[]{.fa .fa-arrow-left} Monteur](builder.html){.btn .btn-default
rel="prev"}
:::
:::::::::::::::::::::::::::::::::::

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
:::::::::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::::::::
