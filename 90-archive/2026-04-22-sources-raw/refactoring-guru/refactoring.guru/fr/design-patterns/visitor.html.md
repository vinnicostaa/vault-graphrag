:::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [Patrons de
conception](../design-patterns.html){.type} / [Patrons
comportementaux](behavioral-patterns.html){.type}
:::

# Visiteur {#visiteur .title}

::: aka
Alias : [Visitor]{style="display:inline-block"}
:::

::: {.section .intent}
##  Intention {#intent}

**Visiteur** est un patron de conception comportemental qui vous permet
de séparer les algorithmes et les objets sur lesquels ils opèrent.

<figure class="image">
<img
src="../../images/patterns/content/visitor/visitor4a4c.png?id=f36d100188340db7a18854ef7916f972"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/visitor/visitor.png?id=f36d100188340db7a18854ef7916f972 640w,/images/patterns/content/visitor/visitor-1.5x.png?id=751e251363b632b20df856d72d02ee72 960w,/images/patterns/content/visitor/visitor-2x.png?id=2c5d9ab3046d782c19809d3b80650d65 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Patron de conception visiteur" />
</figure>
:::

::: {.section .problem}
##  Problème {#problem}

Imaginez que votre équipe développe une application avec des
informations géographiques qui prennent la forme d'un graphe géant.
Chaque nœud du graphe peut représenter une entité complexe comme une
ville, mais aussi des choses plus spécifiques comme des usines, des
sites touristiques, etc. Les nœuds sont interconnectés s'il est possible
de les relier. Dans le code, chaque type de nœud est représenté par sa
propre classe et chaque nœud spécifique est un objet.

<figure class="image">
<img
src="../../images/patterns/diagrams/visitor/problem14338.png?id=e7076532da1e936f3519c63270da8454"
style="aspect-ratio: 2.43;"
srcset="/images/patterns/diagrams/visitor/problem1.png?id=e7076532da1e936f3519c63270da8454 560w,/images/patterns/diagrams/visitor/problem1-1.5x.png?id=250216d3458c38e9855eda96bf71251b 840w,/images/patterns/diagrams/visitor/problem1-2x.png?id=2e5d5143ac55af218754f28761bec17e 1120w"
sizes="(max-width: 720px) 100vw, 560px" loading="lazy" width="560"
alt="Export du graphe en XML" />
<figcaption><p>Export du graphe en XML.</p></figcaption>
</figure>

Le jour vient où vous devez vous attaquer à la mise en place de l'export
du graphe au format XML. À première vue, cela semble assez simple. Vous
avez prévu de mettre en place une méthode d'export pour chaque classe
nœud, puis vous exécutez la méthode sur chaque nœud du graphe en le
parcourant récursivement. La solution était simple et élégante : grâce
au polymorphisme, vous n'avez pas couplé le code qui appelait la méthode
d'exportation aux classes concrètes des nœuds.

Malheureusement, l'architecte du système vous a interdit de modifier les
classes nœud existantes. Sa décision était basée sur le fait que le code
était déjà en production et que vos modifications pourraient causer des
bugs.

<figure class="image">
<img
src="../../images/patterns/diagrams/visitor/problem2-fr66de.png?id=e2e9cb6a43d5b22ae48e8fa5c878577e"
style="aspect-ratio: 1.92;"
srcset="/images/patterns/diagrams/visitor/problem2-fr.png?id=e2e9cb6a43d5b22ae48e8fa5c878577e 500w,/images/patterns/diagrams/visitor/problem2-fr-1.5x.png?id=cd97199c58ae82729d276c2a8607d752 750w,/images/patterns/diagrams/visitor/problem2-fr-2x.png?id=f9a39cbb3c10215c11a7463a54f81278 1000w"
sizes="(max-width: 720px) 100vw, 500px" loading="lazy" width="500"
alt="La méthode d’export en XML devait être ajoutée dans toutes les classes nœud" />
<figcaption><p>La méthode d’export XML devait être ajoutée dans toutes
les classes nœud, ce qui risquait de compromettre l’intégrité du code
de l’application.</p></figcaption>
</figure>

De plus, il a remis en question la pertinence de placer le code d'export
XML à l'intérieur des nœuds. Le rôle principal de ces classes est de
manipuler les données géographiques, ce code serait perçu comme un
intrus.

Il a avancé un autre argument contre votre modification. Il semblait
très probable qu'une fois cette fonctionnalité en place, une personne du
service marketing demande un export dans un format différent ou d'autres
trucs bizarres. Cela vous obligerait à modifier une fois de plus ces
petites classes fragiles.
:::

::: {.section .solution}
##  Solution

Le patron de conception visiteur vous propose de placer ce nouveau
comportement dans une classe séparée que l'on appelle *visiteur*, plutôt
que de l'intégrer dans des classes existantes. L'objet qui devait lancer
ce traitement à l'origine est maintenant passé en paramètre des méthodes
du visiteur, ce qui permet à la méthode d'avoir accès à toutes les
données nécessaires qui se trouvent à l'intérieur de l'objet.

Comment fait-on pour que ce comportement puisse être exécuté sur des
objets de différentes classes ? Par exemple, dans le cas de notre export
XML, l'implémentation sera probablement légèrement différente pour
chaque nœud. La classe visiteur va donc avoir besoin d'un ensemble de
méthodes et chacune d'entre elles pourra prendre des paramètres de
différents types, comme ce qui suit :

<figure class="code">
<pre class="code" lang="pseudocode"><code>class ExportVisitor implements Visitor is
    method doForCity(City c) { ... }
    method doForIndustry(Industry f) { ... }
    method doForSightSeeing(SightSeeing ss) { ... }
    // ...</code></pre>
</figure>

Mais comment allons-nous appeler ces méthodes, surtout celles qui gèrent
le graphe complet ? Nous ne pouvons pas utiliser le polymorphisme, car
ces méthodes ont différentes signatures. Pour sélectionner une méthode
qui peut traiter un objet donné, nous devons vérifier sa classe. On se
croirait dans un cauchemar !

<figure class="code">
<pre class="code" lang="pseudocode"><code>foreach (Node node in graph)
    if (node instanceof City)
        exportVisitor.doForCity((City) node)
    if (node instanceof Industry)
        exportVisitor.doForIndustry((Industry) node)
    // ...
}</code></pre>
</figure>

Il vous est peut-être venu à l'esprit d'utiliser la surcharge. La
surcharge, c'est une technique qui permet de donner le même nom à toutes
les méthodes, même si elles n'ont pas des paramètres identiques.
Malheureusement, même si l'on suppose que notre langage de programmation
nous le permet (le Java et le C# par exemple), cela ne nous sera
d'aucune aide. Comme nous ne connaissons pas la classe d'un nœud à
l'avance, le mécanisme de la surcharge ne sera pas capable de déterminer
la bonne méthode à exécuter. Il ira systématiquement chercher la méthode
qui prend un objet de la classe de base `Nœud`.

Heureusement, le patron de conception visiteur résout ce problème. Il
utilise une technique appelée [double
répartition](visitor-double-dispatch.html) (double dispatch), qui aide à
lancer la bonne méthode sans s'encombrer avec des blocs conditionnels.
Plutôt que de laisser le client choisir la version de la méthode à
appeler, pourquoi ne déléguons-nous pas la décision aux objets que nous
passons en paramètre au visiteur ? Comme les objets connaissent leur
propre classe, ils seront plus à même de choisir la méthode adaptée au
visiteur. Ils « acceptent » un visiteur et lui indiquent la méthode à
exécuter.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// Code client
foreach (Node node in graph)
    node.accept(exportVisitor)

// City
class City is
    method accept(Visitor v) is
        v.doForCity(this)
    // ...

// Industry
class Industry is
    method accept(Visitor v) is
        v.doForIndustry(this)
    // ...</code></pre>
</figure>

J'avoue. Nous avons finalement été obligés de toucher aux classes nœud.
Mais cette modification est mineure et elle nous permet d'ajouter de
nouveaux comportements sans avoir à retoucher le code.

À présent, si nous extrayons une interface pour tous les visiteurs, les
nœuds existants vont accepter tous les visiteurs ajoutés dans
l'application. Pour ajouter un nouveau comportement aux nœuds, il vous
suffit d'implémenter une nouvelle classe visiteur.
:::

::: {.section .analogy}
##  Analogie {#analogy}

<figure class="image">
<img
src="../../images/patterns/content/visitor/visitor-comic-17f02.png?id=7ee4fa8800f7c4df4e1aa3b1aca2b7f1"
style="aspect-ratio: 2;"
srcset="/images/patterns/content/visitor/visitor-comic-1.png?id=7ee4fa8800f7c4df4e1aa3b1aca2b7f1 600w,/images/patterns/content/visitor/visitor-comic-1-1.5x.png?id=c9cadd73d25cc63fce94312f3c82bfee 900w,/images/patterns/content/visitor/visitor-comic-1-2x.png?id=439032451eb49ebbcb5257f25ecee790 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Agent d’assurance" />
<figcaption><p>Un bon agent d’assurance est toujours prêt à proposer
différents contrats pour différentes organisations.</p></figcaption>
</figure>

Imaginez un agent d'assurance expérimenté qui veut absolument acquérir
de nouveaux clients. Il peut faire du porte-à-porte dans tous les
bâtiments d'un quartier et essayer de vendre des assurances à tous ceux
qu'il rencontre. En fonction du type d'entreprise qui occupe le
bâtiment, il peut proposer des contrats d'assurance adaptés :

-   Si c'est un bâtiment résidentiel, il vend des assurances maladie.
-   Si c'est une banque, il vend des assurances contre le vol.
-   Si c'est un café, il vend des assurances contre les incendies et les
    inondations.
:::

::::: {.section .structure-container}
##  Structure

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/visitor/structure-fre556.png?id=41414651c6e0a43124f0485eb4169bf2"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 0.96;"
srcset="/images/patterns/diagrams/visitor/structure-fr.png?id=41414651c6e0a43124f0485eb4169bf2 520w,/images/patterns/diagrams/visitor/structure-fr-1.5x.png?id=4552b3d522dbd3017234e22873a18ad9 780w,/images/patterns/diagrams/visitor/structure-fr-2x.png?id=4498a33a35e91adb424afb3937f82638 1040w"
sizes="(max-width: 720px) 100vw, 520px" loading="lazy" width="520"
alt="Structure du patron de conception visiteur" /><img
src="../../images/patterns/diagrams/visitor/structure-fr-indexedc840.png?id=a2c413487a1227be494eb0918a2fe97d"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 0.93;"
srcset="/images/patterns/diagrams/visitor/structure-fr-indexed.png?id=a2c413487a1227be494eb0918a2fe97d 520w,/images/patterns/diagrams/visitor/structure-fr-indexed-1.5x.png?id=050c2b741413c58b5c16ad4f785fa299 780w,/images/patterns/diagrams/visitor/structure-fr-indexed-2x.png?id=723edc2b43537119293742ec64000343 1040w"
sizes="(max-width: 720px) 100vw, 520px" loading="lazy" width="520"
alt="Structure du patron de conception visiteur" />
</figure>
:::

1.  L'interface **Visiteur** déclare un ensemble de méthodes de parcours
    qui peuvent prendre les éléments concrets d'une structure d'objets
    en paramètre. Ces méthodes peuvent avoir le même nom si le programme
    est écrit dans un langage qui gère la surcharge, mais le type de ses
    paramètres sera différent.

2.  Chaque **Visiteur Concret** implémente plusieurs versions des mêmes
    comportements, en fonction des classes des éléments concrets.

3.  L'interface **Élément** déclare une méthode qui « accepte » les
    visiteurs. Cette méthode déclare un paramètre du type de l'interface
    visiteur.

4.  Chaque **Élément Concret** doit implémenter une méthode
    d'acceptation. Le but de cette méthode est de rediriger l'appel vers
    la méthode appropriée du visiteur en fonction de la classe de
    l'élément actuel. Soyez conscient que même si la classe d'un élément
    de base implémente cette méthode, toutes les sous-classes doivent
    tout de même la redéfinir et appeler la méthode appropriée sur
    l'objet visiteur.

5.  Le **Client** représente en général une collection ou tout autre
    objet complexe (par exemple un arbre [Composite](composite.html)).
    En général, les clients n'ont pas de visibilité sur les classes des
    éléments concrets, car ils manipulent les objets de cette collection
    via une interface abstraite.
::::
:::::

::: {.section .pseudocode}
##  Pseudo-code {#pseudocode}

Dans cet exemple, le **Visiteur** implémente l'export XML dans la
hiérarchie de classes des formes géométriques.

<figure class="image">
<img
src="../../images/patterns/diagrams/visitor/example9405.png?id=d66acd1b9096c47db17ab3bb82b54a59"
style="aspect-ratio: 1.14;"
srcset="/images/patterns/diagrams/visitor/example.png?id=d66acd1b9096c47db17ab3bb82b54a59 560w,/images/patterns/diagrams/visitor/example-1.5x.png?id=534425a20f1128cb81f418cbee30cba8 840w,/images/patterns/diagrams/visitor/example-2x.png?id=f44438b98f13fcb50898baefad67ffff 1120w"
sizes="(max-width: 720px) 100vw, 560px" loading="lazy" width="560"
alt="Structure de l’exemple utilisé pour le patron de conception visiteur" />
<figcaption><p>Exporter différents types d’objets vers le format XML à
l’aide d’un objet visiteur.</p></figcaption>
</figure>

<figure class="code">
<pre class="code" lang="pseudocode"><code>// L’interface élément déclare une méthode `accepter` qui prend
// l’interface de base visiteur en argument.
interface Shape is
    method move(x, y)
    method draw()
    method accept(v: Visitor)

// Chaque classe concrète Élément doit implémenter la méthode
// `accepter` et la faire appeler la méthode du visiteur qui
// correspond à la classe de l’élément.
class Dot implements Shape is
    // ...

    // Vous remarquerez que nous appelons `visiterPoint`, ce qui
    // correspond au nom de la classe actuelle. Ainsi, nous
    // fournissons le nom de la classe de l’élément au visiteur
    // qui le manipule.
    method accept(v: Visitor) is
        v.visitDot(this)

class Circle implements Shape is
    // ...
    method accept(v: Visitor) is
        v.visitCircle(this)

class Rectangle implements Shape is
    // ...
    method accept(v: Visitor) is
        v.visitRectangle(this)

class CompoundShape implements Shape is
    // ...
    method accept(v: Visitor) is
        v.visitCompoundShape(this)


// L’interface visiteur déclare un ensemble de méthodes visiter
// qui correspondent aux classes élément. La signature de la
// méthode visiter permet au visiteur d’identifier la classe
// exacte de l’élément qu’il manipule.
interface Visitor is
    method visitDot(d: Dot)
    method visitCircle(c: Circle)
    method visitRectangle(r: Rectangle)
    method visitCompoundShape(cs: CompoundShape)

// Les visiteurs concrets implémentent plusieurs versions du
// même algorithme. Ce dernier fonctionne avec toutes les
// classes concrètes Élément.
//
// Vous tirez tous les bénéfices du patron de conception
// visiteur lorsque vous l’utilisez avec une structure complexe
// d’objets, comme une arborescence. Dans ce cas, il peut être
// pratique de stocker certains états intermédiaires de
// l’algorithme tout en exécutant les méthodes du visiteur sur
// les différents objets de la structure.
class XMLExportVisitor implements Visitor is
    method visitDot(d: Dot) is
        // Exporte l’ID du point et ses coordonnées.

    method visitCircle(c: Circle) is
        // Exporte l’ID du cercle, les coordonnées de son centre
        // et son rayon.

    method visitRectangle(r: Rectangle) is
        // Exporte l’ID du rectangle, les coordonnées du point
        // supérieur gauche, ainsi que sa largeur et sa
        // longueur.

    method visitCompoundShape(cs: CompoundShape) is
        // Exporte l’ID de la forme, ainsi qu’une liste des ID
        // de ses enfants.


// Le code client peut lancer des traitements du visiteur sur
// n’importe quel ensemble d’éléments sans connaître leurs
// classes concrètes. La méthode accepter envoie un appel au
// traitement approprié de l’objet visiteur.
class Application is
    field allShapes: array of Shapes

    method export() is
        exportVisitor = new XMLExportVisitor()

        foreach (shape in allShapes) do
            shape.accept(exportVisitor)</code></pre>
</figure>

Si vous voulez savoir pourquoi nous avons besoin d'une méthode
`accepter` dans cet exemple, vous pouvez consulter mon article [Visiteur
et Double répartition](visitor-double-dispatch.html) qui décrit le sujet
en détail.
:::

:::::::::: {.section .applicability-container}
##  Possibilités d'application {#applicability}

::::::::: applicability
::: applicability-problem
Utilisez le visiteur lorsque vous voulez lancer des traitements sur les
éléments d'un objet ayant une structure complexe (une arborescence par
exemple).
:::

::: applicability-solution
Le patron de conception visiteur vous permet de lancer des traitements
sur un ensemble d'objets de différentes classes à l'aide d'un objet
visiteur qui implémente une variante d'un même traitement pour chaque
classe visée.
:::

::: applicability-problem
Utilisez le visiteur pour nettoyer la logique métier de tous les
comportements secondaires.
:::

::: applicability-solution
Ce patron vous permet de spécialiser encore plus les classes principales
de votre application, en transférant les autres comportements dans des
classes visiteur.
:::

::: applicability-problem
Utilisez le visiteur si un comportement n'est adapté que pour certaines
classes d'une hiérarchie de classes, mais pas pour les autres.
:::

::: applicability-solution
Vous pouvez envoyer ce comportement dans une classe visiteur séparée,
implémenter seulement les méthodes de visite qui acceptent les objets
des classes concernées et laisser le reste vide.
:::
:::::::::
::::::::::

::: {.section .checklist}
##  Mise en œuvre {#checklist}

1.  Déclarez l'interface visiteur avec un ensemble de méthodes
    « visiter » ; une pour chaque classe d'élément concret qui existe
    dans le programme.

2.  Déclarez l'interface élément. Si vous avez déjà une hiérarchie de
    classes élément, ajoutez la méthode abstraite « accepter » à la
    classe de base de la hiérarchie. Cette méthode doit prendre un
    Visiteur en paramètre.

3.  Implémentez les méthodes d'acceptation dans toutes les classes des
    éléments concrets. Ces méthodes doivent simplement rediriger l'appel
    vers la méthode de visite de l'objet visiteur qui correspond à la
    classe de l'élément actuel.

4.  Les classes élément doivent uniquement interagir avec les visiteurs
    via l'interface visiteur. En revanche, les visiteurs doivent avoir
    la visibilité sur toutes les classes des éléments concrets, qui sont
    référencés comme les types des paramètres des méthodes de visite.

5.  Pour chaque comportement qui ne peut être écrit à l'intérieur de la
    hiérarchie des éléments, créez une nouvelle classe concrète Visiteur
    et implémentez toutes les méthodes de visite.

    Vous pourriez vous retrouver dans une situation où le visiteur aura
    besoin d'un accès aux membres privés d'une classe élément. Dans ce
    cas, vous pouvez rendre ces attributs ou méthodes publics (ce qui ne
    respecte pas l'encapsulation de l'élément) ou imbriquer la classe
    visiteur dans la classe de l'élément. Cette dernière possibilité
    n'est envisageable que si votre langage de programmation gère les
    classes imbriquées.

6.  Le client doit créer des objets visiteur et les passer dans des
    éléments via des méthodes « accepter ».
:::

:::::: {.section .pros-cons}
##  Avantages et inconvénients {#pros-cons}

::::: row
::: col-sm-6
-    *Principe ouvert/fermé*. Vous pouvez ajouter un nouveau
    comportement qui acceptera les objets de différentes classes sans
    les modifier.
-    *Principe de responsabilité unique*. Vous pouvez déplacer plusieurs
    versions du même comportement dans une seule classe.
-    Un objet visiteur peut accumuler des informations utiles en
    manipulant différents objets. Cela peut se révéler pratique si vous
    voulez parcourir une structure complexe d'objets comme un arbre, et
    lancer le traitement du visiteur sur chaque objet de cette
    structure.
:::

::: col-sm-6
-    Vous devez mettre à jour les visiteurs chaque fois qu'une classe
    est ajoutée ou retirée de la hiérarchie des éléments.
-    Les visiteurs n'ont parfois pas les accès nécessaires aux attributs
    ou méthodes privés des éléments qu'ils sont censés manipuler.
:::
:::::
::::::

::: {.section .relations}
##  Liens avec les autres patrons {#relations}

-   Vous pouvez traiter le [Visiteur](visitor.html) comme une version
    plus puissante du patron de conception [Commande](command.html). Ses
    objets peuvent lancer des traitements sur divers objets dans
    différentes classes.

-   Vous pouvez utiliser le [Visiteur](visitor.html) pour lancer une
    opération sur un arbre [Composite](composite.html) entier.

-   Vous pouvez utiliser le [Visiteur](visitor.html) avec
    l'[Itérateur](iterator.html) pour parcourir une structure de données
    complexe et lancer un traitement sur ses éléments, même s'ils ont
    des classes différentes.
:::

::: {.section .implementations}
##  Exemples de code {#implementations}

[![Visiteur en
C#](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](visitor/csharp/example.html "Visiteur en C#"){.prog-lang-link}
[![Visiteur en
C++](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](visitor/cpp/example.html "Visiteur en C++"){.prog-lang-link}
[![Visiteur en
Go](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](visitor/go/example.html "Visiteur en Go"){.prog-lang-link}
[![Visiteur en
Java](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](visitor/java/example.html "Visiteur en Java"){.prog-lang-link}
[![Visiteur en
PHP](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](visitor/php/example.html "Visiteur en PHP"){.prog-lang-link}
[![Visiteur en
Python](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](visitor/python/example.html "Visiteur en Python"){.prog-lang-link}
[![Visiteur en
Ruby](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](visitor/ruby/example.html "Visiteur en Ruby"){.prog-lang-link}
[![Visiteur en
Rust](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](visitor/rust/example.html "Visiteur en Rust"){.prog-lang-link}
[![Visiteur en
Swift](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](visitor/swift/example.html "Visiteur en Swift"){.prog-lang-link}
[![Visiteur en
TypeScript](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](visitor/typescript/example.html "Visiteur en TypeScript"){.prog-lang-link}
:::

::: {.section .extras}
##  Informations supplémentaires {#extras}

-   Vous demandez-vous toujours pourquoi vous ne pouvez pas remplacer le
    patron de conception visiteur par la surcharge ? Lisez mon article
    [Visiteur et double répartition](visitor-double-dispatch.html) pour
    mieux comprendre ces vilains détails.
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

[Visiteur et double répartition []{.fa
.fa-arrow-right}](visitor-double-dispatch.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Retour

[[]{.fa .fa-arrow-left} Patron de méthode](template-method.html){.btn
.btn-default rel="prev"}
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
