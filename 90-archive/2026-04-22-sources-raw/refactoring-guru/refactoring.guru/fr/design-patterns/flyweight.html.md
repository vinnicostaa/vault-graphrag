:::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [Patrons de
conception](../design-patterns.html){.type} / [Patrons
structurels](structural-patterns.html){.type}
:::

# Poids mouche {#poids-mouche .title}

::: aka
Alias : [Poids
plume, ]{style="display:inline-block"}[Flyweight]{style="display:inline-block"}
:::

::: {.section .intent}
##  Intention {#intent}

**Poids mouche** est un patron de conception structurel qui permet de
stocker plus d'objets dans la RAM en partageant les états similaires
entre de multiples objets, plutôt que de stocker les données dans
chaque objet.

<figure class="image">
<img
src="../../images/patterns/content/flyweight/flyweight7be2.png?id=e34fbacb847dd609b5e68aaf252c4db0"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/flyweight/flyweight.png?id=e34fbacb847dd609b5e68aaf252c4db0 640w,/images/patterns/content/flyweight/flyweight-1.5x.png?id=b32df2297473b8a7577e1900e57325ac 960w,/images/patterns/content/flyweight/flyweight-2x.png?id=6a8f17d9550c75c3d648a605c4d31b45 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Patron de conception poids mouche" />
</figure>
:::

::: {.section .problem}
##  Problème {#problem}

Pour vous détendre après de longues heures de travail, vous décidez de
créer un jeu vidéo tout simple dans lequel les joueurs peuvent se
déplacer sur une carte et se tirer dessus. Vous choisissez de mettre en
place un système réaliste de particules et d'en faire l'une des
caractéristiques les plus importantes du jeu. De grandes quantités de
balles, missiles et shrapnels projetés par des explosions vont être
envoyés dans toutes les directions et offrir au joueur une expérience
palpitante.

Une fois la programmation terminée, vous lancez le dernier commit,
assemblez le jeu, puis l'envoyez à votre ami afin qu'il le teste. Le jeu
fonctionnait parfaitement sur votre machine, mais votre ami n'a pas pu
jouer bien longtemps : son ordinateur plante systématiquement au bout de
quelques minutes. Après plusieurs heures à écumer les logs de debug,
vous découvrez que le plantage survient à cause d'une quantité de RAM
insuffisante. Il semblerait que la machine de votre ami soit bien moins
puissante que la vôtre, le problème se produit donc rapidement sur sa
machine.

Il semblerait que le problème soit causé par votre système de
particules. Chaque particule (une balle, un missile ou un morceau de
shrapnel) est représentée par un objet séparé qui contient de nombreuses
données. Lorsque l'action bat son plein sur l'écran du joueur, les
nouvelles particules ne peuvent plus tenir dans la RAM et le programme
plante.

<figure class="image">
<img
src="../../images/patterns/diagrams/flyweight/problem-fr9850.png?id=df845ca7d601fd10440ae9767cc2b372"
style="aspect-ratio: 2.54;"
srcset="/images/patterns/diagrams/flyweight/problem-fr.png?id=df845ca7d601fd10440ae9767cc2b372 660w,/images/patterns/diagrams/flyweight/problem-fr-1.5x.png?id=eac96e7a0a3b064819a55267a6b88e85 990w,/images/patterns/diagrams/flyweight/problem-fr-2x.png?id=063d46ad8a989c660943edf3e346216c 1320w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="660"
alt="Problème du patron de conception poids mouche" />
</figure>
:::

::: {.section .solution}
##  Solution

En inspectant la classe `Particule` de plus près, vous pourrez remarquer
que les attributs de couleur et de sprite consomment beaucoup plus de
mémoire que les autres attributs. Le pire est que ces deux attributs
stockent des données quasi identiques dans toutes les particules. Par
exemple, toutes les balles ont la même couleur et le même sprite.

<figure class="image">
<img
src="../../images/patterns/diagrams/flyweight/solution1-fr408e.png?id=3846611963ef83c88ab7916bf1e9ea6f"
style="aspect-ratio: 2.06;"
srcset="/images/patterns/diagrams/flyweight/solution1-fr.png?id=3846611963ef83c88ab7916bf1e9ea6f 640w,/images/patterns/diagrams/flyweight/solution1-fr-1.5x.png?id=c3f7962dcd7887068eb3971fa1f60dcd 960w,/images/patterns/diagrams/flyweight/solution1-fr-2x.png?id=078e1f79814dd5af06943aa2ffcbce21 1280w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="640"
alt="La solution du patron de conception poids mouche" />
</figure>

Certaines parties de l'état d'une particule, comme les coordonnées, le
mouvement vectoriel et la vitesse sont uniques pour chaque particule.
Après tout, les valeurs de ces attributs changent constamment. Ces
données représentent le contexte évolutif de la particule, alors que la
couleur et le sprite restent constants.

Les données constantes d'un objet forment ce qu'on appelle en général
l'*état intrinsèque*. Elles vivent à l'intérieur de l'objet ; les autres
objets peuvent seulement les lire, mais pas les modifier. Le reste des
données de l'objet sont souvent modifiées « depuis l'extérieur » par
d'autres objets et constituent l'*état extrinsèque*.

Le patron de conception poids mouche propose de ne plus stocker tous les
états extrinsèques à l'intérieur de l'objet. À la place, vous devriez
passer cet état à des méthodes spécialisées qui en dépendent. Seul
l'état intrinsèque va rester à l'intérieur de l'objet, vous permettant
de le réutiliser dans différents contextes. Vous n'aurez besoin que de
quelques-uns de ces objets, car ils diffèrent uniquement dans l'état
intrinsèque. Ce dernier possède bien moins de variantes que l'état
extrinsèque.

<figure class="image">
<img
src="../../images/patterns/diagrams/flyweight/solution3-fr7267.png?id=6bf325740b61c585bbb6e74f721daddb"
style="aspect-ratio: 0.76;"
srcset="/images/patterns/diagrams/flyweight/solution3-fr.png?id=6bf325740b61c585bbb6e74f721daddb 424w,/images/patterns/diagrams/flyweight/solution3-fr-1.5x.png?id=0761557f2250fadc965b81089a15958e 636w,/images/patterns/diagrams/flyweight/solution3-fr-2x.png?id=bea4ab85c24c6b4443755dee2a26fc82 848w"
sizes="(max-width: 720px) 100vw, 424px" loading="lazy" width="424"
alt="La solution du patron de conception poids mouche" />
</figure>

Revenons à notre jeu. Si nous extrayons l'état extrinsèque de notre
classe particule, trois objets seraient suffisants pour représenter
toutes les particules du jeu : une balle, un missile et un morceau de
shrapnel. Vous l'avez probablement déjà deviné, un objet qui ne stocke
que l'état intrinsèque est appelé poids mouche.

#### Stockage d'état extrinsèque

Où allons-nous pouvoir mettre l'état extrinsèque ? Il faut bien qu'une
classe l'accueille, non ? En général, il se retrouve dans l'objet du
contenant, celui qui agrège les objets avant la mise en place du patron.

Dans notre cas, c'est l'objet principal `Jeu` qui stocke toutes les
particules dans l'attribut `particules`. Pour déplacer l'état
extrinsèque dans cette classe, vous devez créer plusieurs attributs de
type tableau pour stocker les coordonnées, les vecteurs et la vitesse de
chaque particule. Mais ce n'est pas tout. Vous allez devoir utiliser un
autre tableau pour stocker les références à un poids mouche spécifique
qui représente la particule. Ces tableaux doivent être synchronisés afin
de pouvoir accéder à toutes les données d'une particule en utilisant le
même index.

<figure class="image">
<img
src="../../images/patterns/diagrams/flyweight/solution2-fr8b2f.png?id=6af3ca829df9b46e25c9827bf91968ee"
style="aspect-ratio: 1.94;"
srcset="/images/patterns/diagrams/flyweight/solution2-fr.png?id=6af3ca829df9b46e25c9827bf91968ee 640w,/images/patterns/diagrams/flyweight/solution2-fr-1.5x.png?id=be62db8e895cee7bf66e6eeafe1d1a32 960w,/images/patterns/diagrams/flyweight/solution2-fr-2x.png?id=2d20b3e9748c85b5b7f0dfda241d60b1 1280w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="640"
alt="La solution proposée par le patron de conception poids mouche" />
</figure>

Il existe une solution plus élégante qui consiste à créer une classe
contexte séparée, qui va stocker l'état extrinsèque avec une référence à
l'objet poids mouche. Cette approche ne nécessite qu'un seul tableau
dans la classe du contenant.

Attendez une minute ! Au tout début, il y avait de nombreux objets
contextuels, en avons-nous encore besoin ? Techniquement, oui. Mais ces
objets sont bien plus petits qu'avant. Les attributs qui demandent le
plus de mémoire ont été déplacés dans quelques objets poids mouche. À
présent, des milliers de petits objets contextuels vont pouvoir
réutiliser un seul gros objet poids mouche, plutôt que de stocker des
milliers de copies de ses données.

#### Poids mouche et objet immuable

Étant donné que le même objet poids mouche peut être utilisé dans
différents contextes, vous devez vous assurer que son état ne peut pas
être modifié. Un poids mouche doit initialiser son état une seule fois
grâce aux paramètres de son constructeur. Il ne doit pas laisser les
autres objets accéder à quelque setter ou attribut public que ce soit.

#### Fabrique poids mouche

Pour accéder plus facilement aux différents poids mouches, vous pouvez
créer une méthode fabrique qui gère une réserve d'objets poids mouche
existants. La méthode prend en paramètre l'état intrinsèque du poids
mouche demandé par le client, cherche s'il en existe un qui correspond à
cet état, et le retourne s'il le trouve. Sinon, il crée un nouveau poids
mouche et l'ajoute à la réserve.

Cette méthode peut être placée à différents endroits. Il parait logique
de l'écrire dans le contenant du poids mouche, mais vous pouvez aussi
bien créer une nouvelle classe fabrique. Vous pouvez également rendre la
méthode fabrique statique et la mettre dans une classe poids mouche.
:::

::::: {.section .structure-container}
##  Structure

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/flyweight/structure7b00.png?id=c1e7e1748f957a4792822f902bc1d420"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.64;"
srcset="/images/patterns/diagrams/flyweight/structure.png?id=c1e7e1748f957a4792822f902bc1d420 640w,/images/patterns/diagrams/flyweight/structure-1.5x.png?id=34cf08f6c2b09dcd1c521c7512cc52b6 960w,/images/patterns/diagrams/flyweight/structure-2x.png?id=a7c8347421bde16435fc90a706f5dd34 1280w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="640"
alt="Structure du patron de conception poids mouche" /><img
src="../../images/patterns/diagrams/flyweight/structure-indexed086a.png?id=aa490792baa26b04464dacbc1f4a19cd"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.54;"
srcset="/images/patterns/diagrams/flyweight/structure-indexed.png?id=aa490792baa26b04464dacbc1f4a19cd 630w,/images/patterns/diagrams/flyweight/structure-indexed-1.5x.png?id=b22a5eab6aacbbd03c66c34802b240ca 945w,/images/patterns/diagrams/flyweight/structure-indexed-2x.png?id=205e2f7d08b4ac0695f445a9db8989c4 1260w"
sizes="(max-width: 720px) 100vw, 630px" loading="lazy" width="630"
alt="Structure du patron de conception poids mouche" />
</figure>
:::

1.  Le patron de conception poids mouche ne permet que de faire de
    l'optimisation. Avant de le mettre en place, assurez-vous que votre
    programme souffre bien d'un problème de RAM insuffisante lié à un
    nombre trop important d'objets similaires en mémoire, et que
    d'autres solutions ne sont pas envisageables.

2.  Le **Poids mouche** contient la part de l'état de l'objet original
    qui peut être partagée entre plusieurs objets. Le même objet poids
    mouche peut être utilisé dans de nombreux contextes. L'état stocké à
    l'intérieur d'un poids mouche est *intrinsèque*. L'état passé aux
    méthodes du poids mouche est *extrinsèque*.

3.  La classe **Contexte** détient l'état extrinsèque, qui est unique
    dans chaque objet original. Lorsqu'un contexte est relié avec un
    objet poids mouche, il est identique à l'objet original.

4.  En général, le comportement de l'objet original reste dans la classe
    poids mouche. Dans ce cas, tout appel à une méthode du poids mouche
    doit également passer en paramètre les parties de l'état extrinsèque
    appropriées. Vous pouvez déplacer le comportement dans la classe
    contexte, ce qui permet ensuite de manipuler le poids mouche associé
    comme n'importe quelles données d'un objet.

5.  Le **Client** calcule et stocke l'état extrinsèque des poids
    mouches. Du point de vue du client, un poids mouche est un objet
    modèle qui peut être configuré à l'exécution en passant des données
    contextuelles en paramètre de ses méthodes.

6.  La **Fabrique du Poids mouche** gère une réserve de poids mouche
    existants. Si une fabrique est présente, les clients ne créent pas
    de poids mouche directement. À la place, ils appellent la fabrique
    et lui passent les morceaux de l'état intrinsèque du poids mouche
    désiré. La fabrique parcourt les poids mouches existants et en
    retourne un s'il correspond aux critères de recherche. Elle en crée
    un nouveau si elle n'en a pas trouvé.
::::
:::::

::: {.section .pseudocode}
##  Pseudo-code {#pseudocode}

Dans l'exemple suivant, le **Poids mouche** diminue l'utilisation de la
mémoire lorsqu'il affiche les millions d'arbres d'un canevas.

<figure class="image">
<img
src="../../images/patterns/diagrams/flyweight/example5053.png?id=0818d078c1a79f373e96397f37b7ee06"
style="aspect-ratio: 1.54;"
srcset="/images/patterns/diagrams/flyweight/example.png?id=0818d078c1a79f373e96397f37b7ee06 540w,/images/patterns/diagrams/flyweight/example-1.5x.png?id=449fa5906e277c134870c07e33dd4b62 810w,/images/patterns/diagrams/flyweight/example-2x.png?id=9423640fe3688a64201389b6e7aa1f48 1080w"
sizes="(max-width: 720px) 100vw, 540px" loading="lazy" width="540"
alt="Exemple utilisé pour le patron de conception poids mouche" />
</figure>

Le patron récupère l'état intrinsèque qui se répète dans une classe
principale `Arbre` et la stocke dans une classe `TypeArbre`.

À présent, on ne stocke plus les mêmes données dans différents objets.
On les met dans quelques poids mouche et on les associe avec les
`Arbres` correspondants qui jouent le rôle du contexte. Le code client
crée de nouveaux objets arbre en utilisant la fabrique du poids mouche.
Cette dernière encapsule la complexité de la recherche du bon objet et
le réutilise si possible.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// La classe poids mouche contient une partie de l&#39;état d’un
// arbre. Ces attributs stockent des valeurs uniques pour chaque
// arbre. Par exemple, vous n’y trouverez pas les coordonnées de
// l’arbre, mais plutôt les textures et les couleurs partagées
// entre plusieurs arbres. Comme ces données sont souvent très
// GROSSES, vous gâcheriez beaucoup de mémoire en stockant
// chacune d’entre elles dans chaque arbre. À la place, nous
// pouvons extraire la texture, la couleur et d’autres données
// redondantes dans un objet séparé que de nombreux arbres vont
// référencer.
class TreeType is
    field name
    field color
    field texture
    constructor TreeType(name, color, texture) { ... }
    method draw(canvas, x, y) is
        // 1. Crée un bitmap d’une couleur, structure et d’un
        // type donnés.
        // 2. Dessine le bitmap sur le canevas aux coordonnées X
        // et Y.

// La fabrique de poids mouche décide si elle veut réutiliser un
// poids mouche existant ou si elle veut en créer un nouveau.
class TreeFactory is
    static field treeTypes: collection of tree types
    static method getTreeType(name, color, texture) is
        type = treeTypes.find(name, color, texture)
        if (type == null)
            type = new TreeType(name, color, texture)
            treeTypes.add(type)
        return type

// L’objet contextuel contient la partie extrinsèque de l’état
// de l’arbre. Une application peut en créer des milliards, car
// ils sont petits : ils sont seulement constitués de deux
// coordonnées entières et d’un attribut pour une référence.
class Tree is
    field x,y
    field type: TreeType
    constructor Tree(x, y, type) { ... }
    method draw(canvas) is
        type.draw(canvas, this.x, this.y)

// Les classes arbre et Forêt sont les clients du poids mouche.
// Vous pouvez les fusionner si vous n’avez pas l’intention
// d’ajouter d’autres évolutions à la classe arbre.
class Forest is
    field trees: collection of Trees

    method plantTree(x, y, name, color, texture) is
        type = TreeFactory.getTreeType(name, color, texture)
        tree = new Tree(x, y, type)
        trees.add(tree)

    method draw(canvas) is
        foreach (tree in trees) do
            tree.draw(canvas)</code></pre>
</figure>
:::

:::::: {.section .applicability-container}
##  Possibilités d'application {#applicability}

::::: applicability
::: applicability-problem
Utilisez le poids mouche uniquement si votre programme génère de
nombreux objets et consomme trop de RAM.
:::

::: applicability-solution
Les bénéfices apportés par ce patron varient énormément en fonction de
son utilisation et de son contexte. Il est surtout utile dans les cas
suivants :

-   Votre application demande un grand nombre d'objets similaires.
-   La RAM de l'appareil ciblé est saturée.
-   Les objets contiennent des états identiques qui peuvent être
    extraits et partagés entre plusieurs objets.
:::
:::::
::::::

::: {.section .checklist}
##  Mise en œuvre {#checklist}

1.  Divisez les attributs d'une classe en deux parties pour les
    transformer en poids mouche :

    -   L'état intrinsèque : les attributs qui contiennent des données
        invariables, dupliquées dans de nombreux objets.
    -   L'état extrinsèque : les attributs qui contiennent des données
        contextuelles uniques à chaque objet.

2.  Laissez les attributs de l'état intrinsèque dans la classe, mais
    empêchez leur modification. Leur valeur initiale ne doit être
    modifiable qu'à l'intérieur de leur constructeur.

3.  Parcourez toutes les méthodes qui utilisent les attributs de l'état
    extrinsèque. Pour chacun de ces attributs, introduisez un nouveau
    paramètre que vous utiliserez à la place.

4.  Il est envisageable de créer une fabrique pour gérer une réserve de
    poids mouches. Elle devra vérifier l'existence du poids mouche avant
    d'en créer un nouveau. Une fois que la fabrique est en place, les
    clients doivent obligatoirement passer par elle pour récupérer des
    poids mouches. Ils doivent donner une description du poids mouche
    désiré en passant leur état intrinsèque à la fabrique.

5.  Le client doit stocker ou calculer les valeurs de l'état extrinsèque
    (contexte) pour appeler des méthodes du poids mouche. Il est parfois
    plus pratique de déplacer l'état extrinsèque et l'attribut qui
    référence le poids mouche dans une classe contexte séparée.
:::

:::::: {.section .pros-cons}
##  Avantages et inconvénients {#pros-cons}

::::: row
::: col-sm-6
-    Vous économisez beaucoup de RAM si votre programme est noyé par des
    tonnes d'objets similaires.
:::

::: col-sm-6
-    Vous allez gagner en RAM mais perdre en cycles microprocesseur
    (CPU) si les données du contexte sont recalculées lors de chaque
    appel d'une méthode poids mouche.
-    Le code devient beaucoup plus compliqué. Les développeurs qui
    rejoignent l'équipe vont se demander pourquoi les états d'une entité
    ont été séparés de cette manière.
:::
:::::
::::::

::: {.section .relations}
##  Liens avec les autres patrons {#relations}

-   Vous pouvez transformer des nœuds de feuilles de l'arbre
    [Composite](composite.html) en [Poids mouches](flyweight.html) et
    les partager pour économiser de la RAM.

-   Le [Poids mouche](flyweight.html) vous montre comment créer plein de
    petits objets alors que la [Façade](facade.html) permet de créer un
    seul objet qui représente un sous-système complet.

-   Le [Poids mouche](flyweight.html) ressemble au
    [Singleton](singleton.html) si vous parvenez à compiler tous les
    états partagés des objets en un seul objet poids mouche. Mais ces
    patrons de conception ont deux différences fondamentales :

    1.  Il ne devrait y avoir qu'une seule instance de singleton, mais
        une classe *poids mouche* peut avoir plusieurs instances avec
        différents états intrinsèques.
    2.  L'objet *singleton* peut être modifiable alors que les objets
        poids mouche ne sont pas modifiables.
:::

::: {.section .implementations}
##  Exemples de code {#implementations}

[![Poids mouche en
C#](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](flyweight/csharp/example.html "Poids mouche en C#"){.prog-lang-link}
[![Poids mouche en
C++](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](flyweight/cpp/example.html "Poids mouche en C++"){.prog-lang-link}
[![Poids mouche en
Go](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](flyweight/go/example.html "Poids mouche en Go"){.prog-lang-link}
[![Poids mouche en
Java](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](flyweight/java/example.html "Poids mouche en Java"){.prog-lang-link}
[![Poids mouche en
PHP](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](flyweight/php/example.html "Poids mouche en PHP"){.prog-lang-link}
[![Poids mouche en
Python](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](flyweight/python/example.html "Poids mouche en Python"){.prog-lang-link}
[![Poids mouche en
Ruby](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](flyweight/ruby/example.html "Poids mouche en Ruby"){.prog-lang-link}
[![Poids mouche en
Rust](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](flyweight/rust/example.html "Poids mouche en Rust"){.prog-lang-link}
[![Poids mouche en
Swift](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](flyweight/swift/example.html "Poids mouche en Swift"){.prog-lang-link}
[![Poids mouche en
TypeScript](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](flyweight/typescript/example.html "Poids mouche en TypeScript"){.prog-lang-link}
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

[Procuration []{.fa .fa-arrow-right}](proxy.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Retour

[[]{.fa .fa-arrow-left} Façade](facade.html){.btn .btn-default
rel="prev"}
:::
:::::::::::::::::::::::::::::

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
:::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::
