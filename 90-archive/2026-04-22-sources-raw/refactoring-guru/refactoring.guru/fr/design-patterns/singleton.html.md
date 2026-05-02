:::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [Patrons de
conception](../design-patterns.html){.type} / [Patrons de
création](creational-patterns.html){.type}
:::

# Singleton {#singleton .title}

::: {.section .intent}
##  Intention {#intent}

**Singleton** est un patron de conception de création qui garantit que
l'instance d'une classe n'existe qu'en un seul exemplaire, tout en
fournissant un point d'accès global à cette instance.

<figure class="image">
<img
src="../../images/patterns/content/singleton/singleton4263.png?id=108a0b9b5ea5c4426e0afa4504491d6f"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/singleton/singleton.png?id=108a0b9b5ea5c4426e0afa4504491d6f 640w,/images/patterns/content/singleton/singleton-1.5x.png?id=29490ad6cd1426c63cf5f88243a1701c 960w,/images/patterns/content/singleton/singleton-2x.png?id=accb2cc7594f7a491ce01dddf0d2f876 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Patron de conception singleton" />
</figure>
:::

::: {.section .problem}
##  Problème {#problem}

Le singleton règle deux problèmes à la fois, mais ne respecte pas le
*principe de responsabilité unique*.

1.  **Il garantit l'unicité d'une instance pour une classe**. Pour
    quelle raison voudrait-on maîtriser le nombre d'instances d'une
    classe ? En général, cette situation se présente lorsque l'on veut
    contrôler l'accès à une ressource partagée --- une base de données
    ou un fichier par exemple.

    Son fonctionnement est le suivant : vous créez un objet, mais après
    un certain temps, vous décidez d'en créer un autre. Plutôt que de
    vous retrouver avec un objet flambant neuf, vous récupérez celui qui
    existe déjà.

    Vous noterez qu'il est impossible d'implémenter ce comportement avec
    un constructeur normal, puisqu'un constructeur **doit**
    théoriquement toujours retourner un nouvel objet.

<figure class="image">
<img
src="../../images/patterns/content/singleton/singleton-comic-1-frcb53.png?id=792833a40401e6e6112ba959f96f5438"
style="aspect-ratio: 2;"
srcset="/images/patterns/content/singleton/singleton-comic-1-fr.png?id=792833a40401e6e6112ba959f96f5438 600w,/images/patterns/content/singleton/singleton-comic-1-fr-1.5x.png?id=fb4a24934b5ca4ae16fa0ebb834f99a2 900w,/images/patterns/content/singleton/singleton-comic-1-fr-2x.png?id=ca3c22c6011b5cb00e6f0e6bd9b63205 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Accès global à un objet" />
<figcaption><p>Les clients ne se rendent pas forcément compte qu’ils
travaillent toujours avec le même objet.</p></figcaption>
</figure>

2.  **Il fournit un point d'accès global à cette instance**. Vous
    rappelez-vous ces variables globales que vous (bon, d'accord : moi)
    avez utilisées pour stocker des objets essentiels ? Elles sont très
    pratiques mais peu fiables, puisque n'importe quelle partie du code
    peut potentiellement écraser leur contenu et faire planter
    l'application.

    Le singleton vous permet d'accéder à l'objet n'importe où dans le
    programme, telle une variable globale. Cependant, il protège son
    instance et l'empêche d'être modifiée.

    Un autre aspect majeur vient se glisser dans l'équation : le code
    qui résout le problème numéro 1 ne doit pas se retrouver éparpillé
    dans tout le programme. En effet, on préfèrera tout mettre dans une
    même classe, surtout si le reste du code repose dessus.

Aujourd'hui, le singleton est devenu très populaire et le terme
*singleton* est même parfois employé pour une entité ne résolvant qu'un
seul des problèmes listés.
:::

::: {.section .solution}
##  Solution

Toute mise en place d'un singleton est constituée des deux étapes
suivantes :

-   Rendre le constructeur par défaut privé afin d'empêcher les autres
    objets d'utiliser l'opérateur `new` avec la classe du singleton.
-   Mettre en place une méthode de création statique qui se comporte
    comme un constructeur. En coulisse, cette méthode appelle le
    constructeur privé pour créer un objet et le sauvegarde dans un
    attribut statique. Tous les appels ultérieurs à cette méthode
    retournent l'objet en cache.

Si votre code a accès à la classe du singleton, alors il peut appeler sa
méthode statique. À chaque appel de cette méthode, c'est toujours le
même objet qui est retourné.
:::

::: {.section .analogy}
##  Analogie {#analogy}

Prenons le gouvernement, qui est un excellent exemple de singleton. Un
pays ne peut avoir qu'un seul gouvernement officiel. Quels que soient
les individus qui composent un gouvernement, l'appellation « Le
gouvernement de X » est un point global d'accès qui identifie le groupe
de personnes au pouvoir.
:::

::::: {.section .structure-container}
##  Structure

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/singleton/structure-frcf10.png?id=c61f45af3dee82ffdbe7a737fa33efa3"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.48;"
srcset="/images/patterns/diagrams/singleton/structure-fr.png?id=c61f45af3dee82ffdbe7a737fa33efa3 430w,/images/patterns/diagrams/singleton/structure-fr-1.5x.png?id=cd01a3a4492abc9692359698825f9cbe 645w,/images/patterns/diagrams/singleton/structure-fr-2x.png?id=4b7b11dc87fcb8bfc8f0b8c3b5a87867 860w"
sizes="(max-width: 720px) 100vw, 430px" loading="lazy" width="430"
alt="La structure du patron de conception singleton" /><img
src="../../images/patterns/diagrams/singleton/structure-fr-indexed2925.png?id=a947e370fde9becc101d8a48718228bc"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.48;"
srcset="/images/patterns/diagrams/singleton/structure-fr-indexed.png?id=a947e370fde9becc101d8a48718228bc 430w,/images/patterns/diagrams/singleton/structure-fr-indexed-1.5x.png?id=105c89da2777a143db87f33860dc36ba 645w,/images/patterns/diagrams/singleton/structure-fr-indexed-2x.png?id=2184b8ad32248a8c0dab159c14bfc408 860w"
sizes="(max-width: 720px) 100vw, 430px" loading="lazy" width="430"
alt="La structure du patron de conception singleton" />
</figure>
:::

1.  La classe **Singleton** déclare la méthode statique `getInstance`
    qui retourne la même instance de sa propre classe.

    Le code client ne doit pas avoir de visibilité sur le constructeur
    du singleton. Seule la méthode `getInstance` doit permettre l'accès
    à l'objet du singleton.
::::
:::::

::: {.section .pseudocode}
##  Pseudo-code {#pseudocode}

Dans cet exemple, la classe de la connexion à la base de données est le
**Singleton**. Cette classe n'a pas de constructeur public, vous ne
pouvez y accéder que grâce à la méthode `getInstance`. Cette méthode met
en cache le premier objet créé puis retourne ce même objet lors des
appels ultérieurs.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// La classe baseDeDonnées définit la méthode `getInstance` qui
// permet aux clients d’accéder à la même instance de la
// connexion à la base de données dans tout le programme.
class Database is
    // L’attribut qui stocke l’instance du singleton doit être
    // ‘static’.
    private static field instance: Database

    // Le constructeur du singleton doit toujours être privé
    // afin d’empêcher les appels à l’opérateur `new`.
    private constructor Database() is
        // Code d’initialisation (la connexion au serveur de la
        // base de données par exemple).
        // ...

    // La méthode statique qui contrôle l’accès à l’instance du
    // singleton.
    public static method getInstance() is
        if (Database.instance == null) then
            acquireThreadLock() and then
                // Ce thread attend la levée du verrou (lock) le
                // temps de s’assurer que l’instance n’a pas
                // déjà été initialisée dans un autre thread.
                if (Database.instance == null) then
                    Database.instance = new Database()
        return Database.instance

    // Pour finir, tout singleton doit définir de la logique
    // métier qui peut être exécutée dans sa propre instance.
    public method query(sql) is
        // Par exemple, toutes les requêtes sur la base de
        // données d’une application passent par cette méthode.
        // Par conséquent, vous pouvez définir le code des
        // limitations ou de la mise en cache ici.
        // ...

class Application is
    method main() is
        Database foo = Database.getInstance()
        foo.query(&quot;SELECT ...&quot;)
        // ...
        Database bar = Database.getInstance()
        bar.query(&quot;SELECT ...&quot;)
        // La variable `bar` contiendra le même objet que la
        // variable `foo`.</code></pre>
</figure>
:::

:::::::: {.section .applicability-container}
##  Possibilités d'application {#applicability}

::::::: applicability
::: applicability-problem
Utilisez le singleton lorsque l'une de vos classes ne doit fournir
qu'une seule instance à tous ses clients. Par exemple, une base de
données partagée entre toutes les parties d'un programme.
:::

::: applicability-solution
La méthode spéciale de création devient le seul moyen de fabriquer des
objets pour la classe, car le singleton désactive les autres. Cette
méthode crée un objet ou retourne l'objet existant s'il a déjà été créé.
:::

::: applicability-problem
Utilisez le singleton lorsque vous voulez un contrôle absolu sur vos
variables globales.
:::

::: applicability-solution
Contrairement aux variables globales, le singleton garantit l'unicité de
l'instance de la classe. Seule la classe singleton peut remplacer
l'instance mise en cache.

Vous pouvez moduler le nombre d'instances du singleton comme vous le
voulez. Vous devez juste apporter une modification dans le corps de la
méthode `getInstance`.
:::
:::::::
::::::::

::: {.section .checklist}
##  Mise en œuvre {#checklist}

1.  Ajoutez un attribut statique à la classe pour stocker l'instance du
    singleton.

2.  Déclarez une méthode de création publique et statique pour récupérer
    l'instance du singleton.

3.  Implémentez une « instanciation paresseuse » (lazy initialization) à
    l'intérieur de la méthode statique. Elle devrait créer un nouvel
    objet lors du premier appel et le stocker dans l'attribut statique.
    La méthode doit retourner cette instance lors de tous les appels
    suivants.

4.  Rendez privé le constructeur de la classe. La méthode statique de la
    classe doit être la seule à pouvoir appeler le constructeur.

5.  Parcourez le code client et remplacez les appels directs au
    constructeur du singleton par des appels à la méthode statique.
:::

:::::: {.section .pros-cons}
##  Avantages et inconvénients {#pros-cons}

::::: row
::: col-sm-6
-    Vous garantissez l'unicité de l'instance de la classe.
-    Vous obtenez un point d'accès global à cette instance.
-    l'objet du singleton est uniquement initialisé la première fois
    qu'il est appelé.
:::

::: col-sm-6
-    Ne respecte pas le *principe de responsabilité unique*. Ce patron
    résout deux problèmes à la fois.
-    Le singleton peut masquer une mauvaise conception ; il se peut, par
    exemple, que les composants aient trop de visibilité les uns envers
    les autres.
-    Il doit bénéficier d'un traitement spécial pour fonctionner dans un
    environnement multithread afin que le singleton ne se retrouve pas
    en plusieurs exemplaires.
-    Les tests unitaires du code client peuvent se révéler difficiles,
    car de nombreux frameworks reposent sur l'héritage lorsqu'ils créent
    des objets fictifs. Étant donné que le constructeur de la classe du
    singleton est privé et que redéfinir la méthode statique est
    impossible dans la majorité des langages, vous allez devoir être
    créatif pour reproduire un singleton fictif. Ou ne pas faire de
    tests. Ou ne pas utiliser de singleton.
:::
:::::
::::::

::: {.section .relations}
##  Liens avec les autres patrons {#relations}

-   Une classe [Façade](facade.html) peut souvent être transformée en
    [Singleton](singleton.html), car un seul objet façade est en général
    suffisant.

-   Le [Poids mouche](flyweight.html) ressemble au
    [Singleton](singleton.html) si vous parvenez à compiler tous les
    états partagés des objets en un seul objet poids mouche. Mais ces
    patrons de conception ont deux différences fondamentales :

    1.  Il ne devrait y avoir qu'une seule instance de singleton, mais
        une classe *poids mouche* peut avoir plusieurs instances avec
        différents états intrinsèques.
    2.  L'objet *singleton* peut être modifiable alors que les objets
        poids mouche ne sont pas modifiables.

-   Les [Fabriques abstraites](abstract-factory.html),
    [Monteurs](builder.html) et [Prototypes](prototype.html) peuvent
    tous être implémentés comme des [Singletons](singleton.html).
:::

::: {.section .implementations}
##  Exemples de code {#implementations}

[![Singleton en
C#](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](singleton/csharp/example.html "Singleton en C#"){.prog-lang-link}
[![Singleton en
C++](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](singleton/cpp/example.html "Singleton en C++"){.prog-lang-link}
[![Singleton en
Go](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](singleton/go/example.html "Singleton en Go"){.prog-lang-link}
[![Singleton en
Java](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](singleton/java/example.html "Singleton en Java"){.prog-lang-link}
[![Singleton en
PHP](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](singleton/php/example.html "Singleton en PHP"){.prog-lang-link}
[![Singleton en
Python](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](singleton/python/example.html "Singleton en Python"){.prog-lang-link}
[![Singleton en
Ruby](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](singleton/ruby/example.html "Singleton en Ruby"){.prog-lang-link}
[![Singleton en
Rust](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](singleton/rust/example.html "Singleton en Rust"){.prog-lang-link}
[![Singleton en
Swift](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](singleton/swift/example.html "Singleton en Swift"){.prog-lang-link}
[![Singleton en
TypeScript](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](singleton/typescript/example.html "Singleton en TypeScript"){.prog-lang-link}
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

[Patrons structurels []{.fa
.fa-arrow-right}](structural-patterns.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Retour

[[]{.fa .fa-arrow-left} Prototype](prototype.html){.btn .btn-default
rel="prev"}
:::
:::::::::::::::::::::::::::::::

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
:::::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::::
