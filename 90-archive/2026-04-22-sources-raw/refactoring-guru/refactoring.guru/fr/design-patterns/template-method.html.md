::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [Patrons de
conception](../design-patterns.html){.type} / [Patrons
comportementaux](behavioral-patterns.html){.type}
:::

# Patron de méthode {#patron-de-méthode .title}

::: aka
Alias : [Méthode socle, ]{style="display:inline-block"}[Template
Method]{style="display:inline-block"}
:::

::: {.section .intent}
##  Intention {#intent}

**Patron de Méthode** est un patron de conception comportemental qui
permet de mettre le squelette d'un algorithme dans la classe mère, mais
laisse les sous-classes redéfinir certaines étapes de l'algorithme sans
changer sa structure.

<figure class="image">
<img
src="../../images/patterns/content/template-method/template-method4c1b.png?id=eee9461742f832814f19612ccf472819"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/template-method/template-method.png?id=eee9461742f832814f19612ccf472819 640w,/images/patterns/content/template-method/template-method-1.5x.png?id=2b4c179aaa02f5c45a59324b904cd130 960w,/images/patterns/content/template-method/template-method-2x.png?id=4e164dc41be4dcfa122628864c2be210 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Patron de conception patron de méthode" />
</figure>
:::

::: {.section .problem}
##  Problème {#problem}

Imaginez que vous êtes en train de créer une application de data mining
(exploration de données) qui analyse les documents d'une entreprise. Les
utilisateurs alimentent l'application avec différents formats (PDF, DOC,
CSV) et celle-ci tente de récupérer les données utiles dans un format
uniforme.

La première version de l'application ne fonctionnait qu'avec les
fichiers DOC. Dans la version suivante, les fichiers CSV étaient
acceptés. Un mois plus tard, vous lui avez « appris » à récupérer les
données des fichiers PDF.

<figure class="image">
<img
src="../../images/patterns/diagrams/template-method/problem6793.png?id=60fa4f735c467ac1c9438231a1782807"
style="aspect-ratio: 1.35;"
srcset="/images/patterns/diagrams/template-method/problem.png?id=60fa4f735c467ac1c9438231a1782807 620w,/images/patterns/diagrams/template-method/problem-1.5x.png?id=83fa009f7727bdcc69335b946919f0d8 930w,/images/patterns/diagrams/template-method/problem-2x.png?id=fc8b434afec7b6135043d0d2f48189f0 1240w"
sizes="(max-width: 720px) 100vw, 620px" loading="lazy" width="620"
alt="Les classes de data mining contiennent beaucoup de code dupliqué" />
<figcaption><p>Les classes de data mining contiennent beaucoup de
code dupliqué.</p></figcaption>
</figure>

Au bout d'un moment, vous remarquez que ces trois classes comportent
beaucoup de code similaire. Bien que le code qui gère les différents
formats de données soit complètement différent d'une classe à l'autre,
celui qui traite et analyse les données est presque identique. Ne
serait-ce pas super de se débarrasser de tout ce code dupliqué tout en
laissant la structure de l'algorithme intact ?

Un autre problème se pose avec le code client qui utilise ces classes.
Il y a de gros blocs conditionnels qui choisissent un comportement en
fonction de la classe de l'objet traité. Si ces trois classes avaient
une interface commune (ou une classe de base), vous pourriez enlever
toutes les conditions du code client et utiliser le polymorphisme
lorsque vous appelez les méthodes sur un objet à traiter.
:::

::: {.section .solution}
##  Solution

Le patron de méthode vous propose de découper un algorithme en une série
d'étapes, de transformer ces étapes en méthodes et de mettre l'ensemble
des appels à ces méthodes dans une seule méthode socle, le *patron de
méthode*. Les étapes peuvent être `abstraites` ou avoir une
implémentation par défaut. Pour utiliser l'algorithme, le client doit
fournir sa propre sous-classe, implémenter toutes les étapes abstraites
et redéfinir certaines d'entre elles si besoin (mais pas la méthode
socle elle-même).

Mettons ceci en application dans notre logiciel de data mining. Nous
pouvons créer une classe de base pour les trois algorithmes d'analyse
syntaxique (parsing). Cette classe définit une méthode socle, composée
d'une série d'appels à plusieurs étapes qui traitent les documents.

<figure class="image">
<img
src="../../images/patterns/diagrams/template-method/solution-fr5f8d.png?id=b7c4a8e9de095643a7b47d063ecd585c"
style="aspect-ratio: 1.43;"
srcset="/images/patterns/diagrams/template-method/solution-fr.png?id=b7c4a8e9de095643a7b47d063ecd585c 600w,/images/patterns/diagrams/template-method/solution-fr-1.5x.png?id=75614c73f87c157f8ecaf53666fd7c59 900w,/images/patterns/diagrams/template-method/solution-fr-2x.png?id=1f36c215057afaf6caa759c399033cce 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="La méthode socle définit le squelette de l’algorithme" />
<figcaption><p>La méthode socle scinde l’algorithme en plusieurs étapes,
permettant aux sous-classes de les redéfinir, mais on ne redéfinit pas
la méthode socle elle-même.</p></figcaption>
</figure>

En premier lieu, nous pouvons déclarer toutes les étapes `abstraites`
afin de forcer les sous-classes à définir leur propre implémentation
pour ces méthodes. Dans notre cas, les sous-classes ont déjà les
implémentations nécessaires. Nous devons donc uniquement ajuster les
signatures des méthodes en les faisant correspondre à celles de la
classe mère.

À présent, voyons ce que l'on peut faire pour se débarrasser du code
dupliqué. Il semblerait que le code pour ouvrir/fermer les fichiers et
récupérer/parser les données est différent pour chaque format de
données, il n'y a donc aucune raison de toucher à ces méthodes. En
revanche, les autres étapes (analyser les données brutes et établir les
rapports) se ressemblent de près et leur implémentation peut donc être
déplacée dans la classe de base, où les sous-classes se partagent le
code.

Comme vous pouvez le voir, nous avons deux types d'étapes :

-   les *opérations abstraites* qui doivent être implémentées dans
    chaque sous-classe.
-   les *opérations facultatives* qui possèdent déjà une implémentation
    par défaut, mais peuvent être redéfinies si besoin.

Toutefois, un autre type d'étape existe : les *crochets* (hooks). Un
crochet est une étape facultative dont le corps de méthode est laissé
vide. Un patron de méthode peut fonctionner même si un crochet n'est pas
redéfini. En général, les crochets sont placés avant ou après les étapes
cruciales des algorithmes et procurent aux sous-classes des points
d'extension supplémentaires.
:::

::: {.section .analogy}
##  Analogie {#analogy}

<figure class="image">
<img
src="../../images/patterns/diagrams/template-method/live-exampleea7a.png?id=2485d52852f87da06c9cc0e2fd257d6a"
style="aspect-ratio: 1.74;"
srcset="/images/patterns/diagrams/template-method/live-example.png?id=2485d52852f87da06c9cc0e2fd257d6a 590w,/images/patterns/diagrams/template-method/live-example-1.5x.png?id=92c5fd9345329da09e3233302158678c 885w,/images/patterns/diagrams/template-method/live-example-2x.png?id=89083a3dcd1fe2b627b9b6e6ff4986dc 1180w"
sizes="(max-width: 720px) 100vw, 590px" loading="lazy" width="590"
alt="Construction en série de maisons" />
<figcaption><p>Un plan architectural typique peut être légèrement
remanié pour mieux répondre aux besoins d’un client.</p></figcaption>
</figure>

Le patron de méthode peut être utilisé pour construire des maisons en
série. Le plan architectural pour construire une maison standard peut
être doté de plusieurs points d'extensions qui permettent à un
propriétaire potentiel d'ajuster certains détails dans la maison.

Chaque étape de construction (poser les fondations, poser la charpente,
monter les murs, installer la plomberie pour l'eau et les câbles pour
l'électricité, etc.) peut être légèrement modifiée pour différencier un
peu la maison des autres.
:::

::::: {.section .structure-container}
##  Structure

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/template-method/structure1c3b.png?id=924692f994bff6578d8408d90f6fc459"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 0.89;"
srcset="/images/patterns/diagrams/template-method/structure.png?id=924692f994bff6578d8408d90f6fc459 340w,/images/patterns/diagrams/template-method/structure-1.5x.png?id=613d78ad8ec08536ec70f4e0976b5a1a 510w,/images/patterns/diagrams/template-method/structure-2x.png?id=25082d6d6a76f51c6b64d8aeeaffdbb5 680w"
sizes="(max-width: 720px) 100vw, 340px" loading="lazy" width="340"
alt="Structure du patron de conception patron de méthode" /><img
src="../../images/patterns/diagrams/template-method/structure-indexedb94b.png?id=4ced6107519bc66710d2f05c0f4097a1"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 0.92;"
srcset="/images/patterns/diagrams/template-method/structure-indexed.png?id=4ced6107519bc66710d2f05c0f4097a1 350w,/images/patterns/diagrams/template-method/structure-indexed-1.5x.png?id=58e3a9092786c824eb738a7771702093 525w,/images/patterns/diagrams/template-method/structure-indexed-2x.png?id=86f28789cdcc5a4c415d6a1100de56fc 700w"
sizes="(max-width: 720px) 100vw, 350px" loading="lazy" width="350"
alt="Structure du patron de conception patron de méthode" />
</figure>
:::

1.  La **Classe Abstraite** déclare des méthodes (qui représentent les
    étapes d'un algorithme) et la méthode patronDeMéthode qui appelle
    toutes ces méthodes dans un ordre spécifique. Les étapes peuvent
    être déclarées abstraites ou posséder une implémentation par défaut.

2.  Les **Classes Concrètes** peuvent redéfinir toutes les étapes, mais
    pas patronDeMéthode.
::::
:::::

::: {.section .pseudocode}
##  Pseudo-code {#pseudocode}

Dans cet exemple, le patron de conception **Patron de Méthode** fournit
un squelette pour les branches de l'IA (intelligence artificielle) d'un
jeu vidéo de stratégie simple.

<figure class="image">
<img
src="../../images/patterns/diagrams/template-method/example32c6.png?id=c0ce5cc8070925a1cd345fac6afa16b6"
style="aspect-ratio: 1;"
srcset="/images/patterns/diagrams/template-method/example.png?id=c0ce5cc8070925a1cd345fac6afa16b6 430w,/images/patterns/diagrams/template-method/example-1.5x.png?id=d85c6b81c900f46d55688406170600bc 645w,/images/patterns/diagrams/template-method/example-2x.png?id=d8b309659c4b722237ec97733da935bd 860w"
sizes="(max-width: 720px) 100vw, 430px" loading="lazy" width="430"
alt="Structure du patron de méthode dans l’exemple utilisé" />
<figcaption><p>Les classes d’IA dans un jeu
vidéo simple.</p></figcaption>
</figure>

Toutes les races du jeu ont à peu près les mêmes unités et bâtiments.
Vous pouvez donc réutiliser la même structure d'IA pour les différentes
races, tout en personnalisant certains détails. Grâce à cette approche,
vous pouvez redéfinir l'IA des orcs pour la rendre plus agressive,
obtenir des humains plus défensifs, et faire en sorte que les monstres
ne puissent pas construire de bâtiments. Pour ajouter une autre race au
jeu, vous devez simplement créer une nouvelle sous-classe d'IA et
redéfinir les méthodes par défaut déclarées dans la classe de base de
l'IA.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// La classe abstraite définit un patron de méthode qui contient
// le squelette d’un algorithme. Ce dernier est généralement
// composé d’appels à des opérations primitives abstraites. Les
// sous-classes concrètes implémentent ces opérations, mais ne
// touchent pas au patron de méthode.
class GameAI is
    // Le patron de méthode définit le squelette d’un
    // algorithme.
    method turn() is
        collectResources()
        buildStructures()
        buildUnits()
        attack()

    // Certaines étapes peuvent être implémentées directement
    // dans une classe de base.
    method collectResources() is
        foreach (s in this.builtStructures) do
            s.collect()

    // Et certaines peuvent être abstraites.
    abstract method buildStructures()
    abstract method buildUnits()

    // Une classe peut avoir plusieurs patrons de méthode.
    method attack() is
        enemy = closestEnemy()
        if (enemy == null)
            sendScouts(map.center)
        else
            sendWarriors(enemy.position)

    abstract method sendScouts(position)
    abstract method sendWarriors(position)

// Les classes concrètes doivent implémenter toutes les
// opérations abstraites de la classe de base, mais elles ne
// doivent pas redéfinir le patron de méthode.
class OrcsAI extends GameAI is
    method buildStructures() is
        if (there are some resources) then
            // Construit des fermes, des casernes, puis une
            // forteresse.

    method buildUnits() is
        if (there are plenty of resources) then
            if (there are no scouts)
                // Construit un ouvrier et l’ajoute au groupe
                // d’éclaireurs.
            else
                // Construit un combattant et l’ajoute au groupe
                // des guerriers.

    // ...

    method sendScouts(position) is
        if (scouts.length &gt; 0) then
            // Envoie les éclaireurs à la position.

    method sendWarriors(position) is
        if (warriors.length &gt; 5) then
            // Envoie les guerriers à la position.

// Les sous-classes peuvent redéfinir certaines opérations avec
// une implémentation par défaut.
class MonstersAI extends GameAI is
    method collectResources() is
        // Les monstres ne récoltent pas de ressources.

    method buildStructures() is
        // Les monstres ne fabriquent pas de bâtiments.

    method buildUnits() is
        // Les monstres ne construisent pas d&#39;unités.</code></pre>
</figure>
:::

:::::::: {.section .applicability-container}
##  Possibilités d'application {#applicability}

::::::: applicability
::: applicability-problem
Utilisez le patron de méthode si vous voulez que vos clients puissent
étendre des étapes spécifiques d'un algorithme, mais pas l'algorithme
entier ou sa structure.
:::

::: applicability-solution
Le patron de méthode vous permet de transformer un algorithme
monolithique en une série d'étapes individuelles qui peuvent être
facilement étendues par des sous-classes, tout en gardant intacte la
structure établie dans une classe mère.
:::

::: applicability-problem
Utilisez ce patron si vous avez plusieurs classes qui contiennent des
algorithmes presque identiques, avec seulement quelques différences
mineures. Par conséquent, vous risqueriez de devoir retoucher toutes les
classes lorsque vous modifiez l'algorithme.
:::

::: applicability-solution
Lorsque vous transformez un tel algorithme en un patron de méthode, vous
pouvez également remonter les étapes dotées d'implémentations similaires
dans la classe mère, afin d'éviter la duplication de code. Vous pouvez
laisser le reste du code dans les sous-classes.
:::
:::::::
::::::::

::: {.section .checklist}
##  Mise en œuvre {#checklist}

1.  Analysez l'algorithme ciblé pour voir si vous pouvez le décomposer
    en étapes. Déterminez les étapes communes à toutes les sous-classes
    et celles qui sont uniques.

2.  Créez une classe de base abstraite et déclarez le patron de méthode
    et un ensemble de méthodes abstraites pour représenter les
    opérations de l'algorithme. Faites une ébauche de la structure de
    l'algorithme dans ce patron de méthode en appelant les opérations
    correspondantes. Rendez ce patron `final` pour empêcher les
    sous-classes de la redéfinir.

3.  Cela ne pose aucun problème si toutes les opérations sont
    abstraites, mais une implémentation par défaut bénéficierait à
    certaines opérations. Les sous-classes n'ont pas besoin
    d'implémenter ces méthodes.

4.  Pensez à ajouter des crochets entre les étapes cruciales de votre
    algorithme.

5.  Pour chaque variante de l'algorithme, créez une nouvelle
    sous-classe. Elle *doit* implémenter toutes les opérations
    abstraites, mais *peut* également redéfinir les opérations
    facultatives.
:::

:::::: {.section .pros-cons}
##  Avantages et inconvénients {#pros-cons}

::::: row
::: col-sm-6
-    Vous permettez aux clients de redéfinir certaines parties d'un
    grand algorithme. Elles sont ainsi moins affectées par les
    modifications apportées aux autres parties de l'algorithme.
-    Vous pouvez remonter le code dupliqué dans la classe mère.
:::

::: col-sm-6
-    Certains clients peuvent être limités à cause du squelette de
    l'algorithme.
-    Vous ne respectez pas le *Principe de substitution de Liskov*, si
    vous supprimez l'implémentation d'une étape par défaut dans une
    sous-classe.
-    Plus vous avez d'étapes, plus le patron de méthode devient
    difficile à maintenir.
:::
:::::
::::::

::: {.section .relations}
##  Liens avec les autres patrons {#relations}

-   La [Fabrique](factory-method.html) est une spécialisation du [Patron
    de méthode](template-method.html). Une *fabrique* peut aussi faire
    office d'étape dans un grand *patron de méthode*.

-   Le [Patron de méthode](template-method.html) est basé sur
    l'héritage : il vous laisse modifier certaines parties d'un
    algorithme en les étendant dans les sous-classes. La
    [Stratégie](strategy.html) est basée sur la composition : vous
    pouvez modifier certaines parties du comportement de l'objet en lui
    fournissant différentes stratégies qui correspondent à ce
    comportement. Le *patron de méthode* agit au niveau de la classe, il
    est donc statique. La *stratégie* agit au niveau de l'objet et vous
    laisse permuter les comportements à l'exécution.
:::

::: {.section .implementations}
##  Exemples de code {#implementations}

[![Patron de méthode en
C#](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](template-method/csharp/example.html "Patron de méthode en C#"){.prog-lang-link}
[![Patron de méthode en
C++](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](template-method/cpp/example.html "Patron de méthode en C++"){.prog-lang-link}
[![Patron de méthode en
Go](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](template-method/go/example.html "Patron de méthode en Go"){.prog-lang-link}
[![Patron de méthode en
Java](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](template-method/java/example.html "Patron de méthode en Java"){.prog-lang-link}
[![Patron de méthode en
PHP](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](template-method/php/example.html "Patron de méthode en PHP"){.prog-lang-link}
[![Patron de méthode en
Python](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](template-method/python/example.html "Patron de méthode en Python"){.prog-lang-link}
[![Patron de méthode en
Ruby](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](template-method/ruby/example.html "Patron de méthode en Ruby"){.prog-lang-link}
[![Patron de méthode en
Rust](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](template-method/rust/example.html "Patron de méthode en Rust"){.prog-lang-link}
[![Patron de méthode en
Swift](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](template-method/swift/example.html "Patron de méthode en Swift"){.prog-lang-link}
[![Patron de méthode en
TypeScript](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](template-method/typescript/example.html "Patron de méthode en TypeScript"){.prog-lang-link}
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

[Visiteur []{.fa .fa-arrow-right}](visitor.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Retour

[[]{.fa .fa-arrow-left} Stratégie](strategy.html){.btn .btn-default
rel="prev"}
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
