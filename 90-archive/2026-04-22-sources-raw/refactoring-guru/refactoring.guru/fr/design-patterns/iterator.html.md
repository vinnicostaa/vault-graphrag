::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [Patrons de
conception](../design-patterns.html){.type} / [Patrons
comportementaux](behavioral-patterns.html){.type}
:::

# Itérateur {#itérateur .title}

::: aka
Alias : [Iterator]{style="display:inline-block"}
:::

::: {.section .intent}
##  Intention {#intent}

**Itérateur** est un patron de conception comportemental qui permet de
parcourir les éléments d'une collection sans révéler sa représentation
interne (liste, pile, arbre, etc.).

<figure class="image">
<img
src="../../images/patterns/content/iterator/iterator-enfc4d.png?id=d19123d71d355d01b0ede4be173eb695"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/iterator/iterator-en.png?id=d19123d71d355d01b0ede4be173eb695 640w,/images/patterns/content/iterator/iterator-en-1.5x.png?id=54aa477cafffe8f9b1b6e63c2e88c21b 960w,/images/patterns/content/iterator/iterator-en-2x.png?id=2a85705e8e5fab257802b2ce36d6d236 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Patron de conception itérateur" />
</figure>
:::

::: {.section .problem}
##  Problème {#problem}

Les collections ne servent que de conteneur pour un groupe d'objets,
mais elles demeurent l'un des types de données les plus utilisés en
programmation.

<figure class="image">
<img
src="../../images/patterns/diagrams/iterator/problem14cac.png?id=52ef2fe2d4920e3fed696c221fe757f2"
style="aspect-ratio: 4.9;"
srcset="/images/patterns/diagrams/iterator/problem1.png?id=52ef2fe2d4920e3fed696c221fe757f2 490w,/images/patterns/diagrams/iterator/problem1-1.5x.png?id=b46e39ade7de292fdcacc9790066c72f 735w,/images/patterns/diagrams/iterator/problem1-2x.png?id=1f4384c3c631be238bfc23d6eb84a0ef 980w"
sizes="(max-width: 720px) 100vw, 490px" loading="lazy" width="490"
alt="Différents types de collections" />
<figcaption><p>Différents types de collections.</p></figcaption>
</figure>

La majorité des collections stockent leurs éléments dans de simples
listes, mais certaines d'entre elles sont basées sur les piles, les
arbres, les graphes ou d'autres structures complexes de données.

Quelle que soit sa structure, une collection doit fournir un moyen
d'accéder à ses éléments pour permettre au code de les utiliser. Elle
doit donner la possibilité de parcourir tous ses éléments sans passer
plusieurs fois par les mêmes.

À première vue, cela semble simple pour les collections qui ressemblent
à une liste. Vous lancez juste une boucle sur tous les éléments. Mais
comment faites-vous pour parcourir séquentiellement les éléments d'une
structure complexe de données comme un arbre ? Un jour donné, vous allez
peut-être vous en sortir avec un parcours en profondeur. Mais le
lendemain, vous allez avoir besoin d'un parcours en largeur. La semaine
d'après, vous allez avoir besoin d'autre chose pour établir un parcours
aléatoire des éléments de l'arbre.

<figure class="image">
<img
src="../../images/patterns/diagrams/iterator/problem29f71.png?id=f9c1a746c787320291c85fdc2a314192"
style="aspect-ratio: 3.75;"
srcset="/images/patterns/diagrams/iterator/problem2.png?id=f9c1a746c787320291c85fdc2a314192 600w,/images/patterns/diagrams/iterator/problem2-1.5x.png?id=7a003d020e789773e0c833d7b1df00e6 900w,/images/patterns/diagrams/iterator/problem2-2x.png?id=656b13c109d4d4960ea1f9fa993bae4c 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Différents algorithmes de parcours" />
<figcaption><p>Il y a plusieurs manières de parcourir une
même collection.</p></figcaption>
</figure>

Plus vous ajoutez d'algorithmes différents pour parcourir votre
collection et plus vous masquez sa responsabilité principale : stocker
efficacement les données. De plus, certains algorithmes sont dédiés à un
usage spécifique. Il serait donc bizarre de les ajouter au comportement
d'une collection générique.

Le code client qui va se servir de ces collections se fiche probablement
de la manière dont elles stockent leurs éléments. Cependant, ces
collections fournissent différentes manières d'accéder à leurs éléments,
vous couplez donc forcément votre code aux classes de ces collections.
:::

::: {.section .solution}
##  Solution

Le but du patron de conception itérateur est d'extraire le comportement
qui permet de parcourir une collection et de le mettre dans un objet que
l'on nomme *itérateur*.

<figure class="image">
<img
src="../../images/patterns/diagrams/iterator/solution19739.png?id=2f5fbcce6099d8ea09b2fbb83e3e7059"
style="aspect-ratio: 0.85;"
srcset="/images/patterns/diagrams/iterator/solution1.png?id=2f5fbcce6099d8ea09b2fbb83e3e7059 400w,/images/patterns/diagrams/iterator/solution1-1.5x.png?id=68612372ede6e5029403d38b381fdc05 600w,/images/patterns/diagrams/iterator/solution1-2x.png?id=dcebcad4c197bfa5f25f680382d0e5ac 800w"
sizes="(max-width: 720px) 100vw, 400px" loading="lazy" width="400"
alt="Les itérateurs implémentent différents algorithmes de parcours" />
<figcaption><p>Les itérateurs implémentent différents algorithmes de
parcours. Plusieurs itérateurs peuvent parcourir une même
collection simultanément.</p></figcaption>
</figure>

En plus d'implémenter l'algorithme de parcours, un objet itérateur
encapsule tous les détails comme la position actuelle et le nombre
d'éléments restants avant d'atteindre la fin. Grâce à cela, plusieurs
itérateurs peuvent parcourir une même collection simultanément et
indépendamment les uns des autres.

En général, les itérateurs fournissent une méthode principale pour
récupérer les éléments d'une collection. Le client peut appeler la
méthode en continu jusqu'à ce qu'elle ne retourne plus rien, ce qui
signifie que l'itérateur a parcouru tous les éléments.

Les itérateurs doivent tous implémenter la même interface. Le code
client est ainsi compatible avec tous les types de collections et tous
les algorithmes de parcours, tant que l'itérateur adéquat existe. Si
vous voulez effectuer un parcours un peu spécial dans une collection, il
vous suffit de créer une nouvelle classe itérateur, sans toucher à la
collection ni au client.
:::

::: {.section .analogy}
##  Analogie {#analogy}

<figure class="image">
<img
src="../../images/patterns/content/iterator/iterator-comic-1-frd7cc.png?id=a6b6340a0af63b5528652846a62f9e7e"
style="aspect-ratio: 2.13;"
srcset="/images/patterns/content/iterator/iterator-comic-1-fr.png?id=a6b6340a0af63b5528652846a62f9e7e 640w,/images/patterns/content/iterator/iterator-comic-1-fr-1.5x.png?id=85b772ceb461794f495b82ba4d558d55 960w,/images/patterns/content/iterator/iterator-comic-1-fr-2x.png?id=39d1a099ba2067442c128364da2b719a 1280w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="640"
alt="Différentes manières de parcourir Rome" />
<figcaption><p>Différentes manières de parcourir Rome.</p></figcaption>
</figure>

Vous projetez de visiter Rome pendant quelques jours et voulez voir ses
principaux sites et attractions. Mais une fois sur place, vous risquez
de perdre beaucoup de temps à tourner en rond et même de ne pas réussir
à trouver le Colisée.

Vous pourriez acheter une application de guide virtuel sur votre
smartphone qui vous permettrait de trouver votre chemin. C'est pratique
et peu onéreux, et vous pourriez visiter des sites intéressants à
volonté.

Vous pourriez également dépenser une partie de votre budget pour engager
un guide local qui connait la ville comme sa poche. Il adapterait votre
séjour en fonction de vos goûts et de vos attentes, vous ferait visiter
toutes les attractions et vous raconterait des histoires passionnantes.
Ce serait vraiment génial, mais malheureusement, beaucoup plus cher.

Toutes ces possibilités --- des directions aléatoires que vous prenez,
l'application sur smartphone ou le guide humain --- sont des itérateurs
sur une vaste collection de sites et d'attractions dans Rome.
:::

::::: {.section .structure-container}
##  Structure

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/iterator/structure5247.png?id=35ea851f8f6bbe51d79eb91e6e6519d0"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.12;"
srcset="/images/patterns/diagrams/iterator/structure.png?id=35ea851f8f6bbe51d79eb91e6e6519d0 480w,/images/patterns/diagrams/iterator/structure-1.5x.png?id=19d4c2130ab2e3bd718f87e956fcd873 720w,/images/patterns/diagrams/iterator/structure-2x.png?id=b7b4708d3f279dd046eb1692f1cba710 960w"
sizes="(max-width: 720px) 100vw, 480px" loading="lazy" width="480"
alt="Structure du patron de conception itérateur" /><img
src="../../images/patterns/diagrams/iterator/structure-indexed3472.png?id=7bc28907ff6b480db6635a93ebaa10ff"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.33;"
srcset="/images/patterns/diagrams/iterator/structure-indexed.png?id=7bc28907ff6b480db6635a93ebaa10ff 520w,/images/patterns/diagrams/iterator/structure-indexed-1.5x.png?id=32fde4c4c1cbfbe07aa6e716e5ac2346 780w,/images/patterns/diagrams/iterator/structure-indexed-2x.png?id=d809428b2262e2013972fe69d2434473 1040w"
sizes="(max-width: 720px) 100vw, 520px" loading="lazy" width="520"
alt="Structure du patron de conception itérateur" />
</figure>
:::

1.  L'interface **Itérateur** déclare les opérations nécessaires au
    parcours d'une collection : récupérer le prochain élément, donner la
    position actuelle, recommencer la boucle depuis le début, etc.

2.  Les **Itérateurs Concrets** implémentent les algorithmes qui servent
    au parcours d'une collection. L'objet itérateur doit garder la trace
    du parcours actuel. Grâce à cela, plusieurs itérateurs peuvent
    parcourir la même collection de manière indépendante.

3.  L'interface **Collection** déclare une ou plusieurs méthodes pour
    récupérer des itérateurs compatibles avec la collection. Le type de
    retour de la méthode doit être l'interface de l'itérateur afin de
    permettre aux collections concrètes de renvoyer tous types
    d'itérateurs.

4.  Les **Collections Concrètes** renvoient les nouvelles instances de
    la classe concrète de l'itérateur lorsque le client en demande une.
    Vous vous demandez peut-être où l'on doit mettre le reste du code de
    la collection. Ne vous inquiétez pas, il devrait être dans la même
    classe. Ces détails ne sont pas très importants pour ce patron de
    conception, je me permets donc de les omettre.

5.  Le **Client** manipule les collections et les itérateurs grâce à
    leurs interfaces. Ceci permet de ne pas coupler le client avec les
    classes concrètes et d'utiliser différents itérateurs et collections
    avec le même code client.

    En général, les clients ne créent pas les itérateurs, ils les
    récupèrent auprès des collections. Mais dans certains cas, le client
    peut en créer un directement. Le client peut par exemple définir son
    propre itérateur spécial.
::::
:::::

::: {.section .pseudocode}
##  Pseudo-code {#pseudocode}

Dans cet exemple, le patron de conception **Itérateur** va servir à
parcourir un type spécial de collection qui encapsule l'accès au graphe
social de Facebook. Cette collection procure plusieurs itérateurs qui
parcourent les profils de différentes manières.

<figure class="image">
<img
src="../../images/patterns/diagrams/iterator/examplee0a2.png?id=f2a24ef3787bf80ed450709240506ff2"
style="aspect-ratio: 0.95;"
srcset="/images/patterns/diagrams/iterator/example.png?id=f2a24ef3787bf80ed450709240506ff2 600w,/images/patterns/diagrams/iterator/example-1.5x.png?id=cab0e1459ffc3721579a3e7d6a4e9022 900w,/images/patterns/diagrams/iterator/example-2x.png?id=73c3e55f75ca0250bd84e8a1d8cc88d2 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Structure de l’exemple utilisé pour le patron de conception itérateur" />
<figcaption><p>Un exemple d’itération sur les
profils sociaux.</p></figcaption>
</figure>

L'itérateur des 'amis' peut être utilisé pour parcourir les amis d'un
profil donné. L'itérateur des 'collègues' fait la même chose, sauf qu'il
ignore les amis qui ne travaillent pas dans la même entreprise que la
personne sélectionnée. Ces deux itérateurs implémentent une interface
commune qui permet aux clients d'aller chercher les profils sans tenir
compte des détails de l'implémentation, comme l'identification ou
l'envoi de requêtes REST.

Le code client n'est pas couplé aux classes concrètes, car il manipule
les collections et les itérateurs en passant par leurs interfaces. Si
vous décidez de connecter votre application à un réseau social, vous
devez simplement fournir de nouvelles classes de collections et
d'itérateurs sans toucher au code existant.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// L’interface collection doit déclarer une fabrique pour créer
// des itérateurs. Si vous avez besoin de plusieurs types
// d’itérations dans votre programme, vous pouvez déclarer
// plusieurs méthodes.
interface SocialNetwork is
    method createFriendsIterator(profileId):ProfileIterator
    method createCoworkersIterator(profileId):ProfileIterator


// Chaque collection concrète est couplée à un ensemble de
// classes concrètes Itérateur qu’elle retourne. Mais le client
// en est indépendant, car la signature de ces méthodes renvoie
// des interfaces itérateur.
class Facebook implements SocialNetwork is
    // ...Le code principal de la collection devrait être écrit
    // ici...

    // Code de création de l’itérateur.
    method createFriendsIterator(profileId) is
        return new FacebookIterator(this, profileId, &quot;friends&quot;)
    method createCoworkersIterator(profileId) is
        return new FacebookIterator(this, profileId, &quot;coworkers&quot;)


// L’interface commune à tous les itérateurs.
interface ProfileIterator is
    method getNext():Profile
    method hasMore():bool


// La classe concrète Itérateur.
class FacebookIterator implements ProfileIterator is
    // L’itérateur a besoin d’une référence à la collection
    // qu’il parcourt.
    private field facebook: Facebook
    private field profileId, type: string

    // Un objet itérateur parcourt la collection indépendamment
    // des autres itérateurs. Il doit donc stocker l’état de
    // l’itération.
    private field currentPosition
    private field cache: array of Profile

    constructor FacebookIterator(facebook, profileId, type) is
        this.facebook = facebook
        this.profileId = profileId
        this.type = type

    private method lazyInit() is
        if (cache == null)
            cache = facebook.socialGraphRequest(profileId, type)

    // Chaque classe concrète Itérateur a sa propre
    // implémentation de l’interface commune Itérateur.
    method getNext() is
        if (hasMore())
            result = cache[currentPosition]
            currentPosition++
            return result

    method hasMore() is
        lazyInit()
        return currentPosition &lt; cache.length


// Voici une astuce très pratique : vous pouvez passer un
// itérateur à une classe du client, plutôt que de lui donner
// accès à une collection complète. Cette manipulation vous
// permet de ne pas exposer la collection au client.
//
// Elle vous permet également de modifier la façon dont le
// client fonctionne avec les collections lors de son exécution,
// en lui passant un itérateur différent. Cette manipulation
// serait impossible si le code client était couplé aux classes
// concrètes de l’itérateur.
class SocialSpammer is
    method send(iterator: ProfileIterator, message: string) is
        while (iterator.hasMore())
            profile = iterator.getNext()
            System.sendEmail(profile.getEmail(), message)


// La classe application configure les collections et les
// itérateurs, puis les envoie au code client.
class Application is
    field network: SocialNetwork
    field spammer: SocialSpammer

    method config() is
        if working with Facebook
            this.network = new Facebook()
        if working with LinkedIn
            this.network = new LinkedIn()
        this.spammer = new SocialSpammer()

    method sendSpamToFriends(profile) is
        iterator = network.createFriendsIterator(profile.getId())
        spammer.send(iterator, &quot;Very important message&quot;)

    method sendSpamToCoworkers(profile) is
        iterator = network.createCoworkersIterator(profile.getId())
        spammer.send(iterator, &quot;Very important message&quot;)</code></pre>
</figure>
:::

:::::::::: {.section .applicability-container}
##  Possibilités d'application {#applicability}

::::::::: applicability
::: applicability-problem
Utilisez le patron de conception itérateur quand la structure de données
de votre collection est trop complexe et que vous voulez cacher ces
détails aux clients (parce que c'est pratique ou pour des raisons de
sécurité).
:::

::: applicability-solution
L'itérateur encapsule le détail du fonctionnement de la structure
complexe de données et passe plusieurs méthodes simples au client qui
lui permettent d'accéder aux éléments de la collection. En plus d'être
très pratique pour le client, ce fonctionnement permet de protéger la
collection d'actions maladroites ou malicieuses que le client pourrait
entreprendre s'il travaillait directement avec la collection.
:::

::: applicability-problem
Utilisez l'itérateur pour que le code du parcours soit le moins dupliqué
possible dans votre application.
:::

::: applicability-solution
Un algorithme qui permet une itération complexe sur la collection a
tendance à être très volumineux. Si on le place à l'intérieur de la
logique de l'application, cela va masquer la responsabilité du code
original et le rendre moins maintenable. Pour rendre le code encore plus
simple et propre, vous pouvez écrire l'algorithme de parcours dans leur
itérateur.
:::

::: applicability-problem
Utilisez ce patron pour permettre à votre code de naviguer dans des
structures de données complexes ou inconnues.
:::

::: applicability-solution
Ce patron procure des interfaces génériques pour les collections et les
itérateurs. Si votre code utilise bien ces interfaces, il va pouvoir
manipuler toutes les collections (et leurs itérateurs) qui les
implémentent.
:::
:::::::::
::::::::::

::: {.section .checklist}
##  Mise en œuvre {#checklist}

1.  Déclarez l'interface itérateur. Elle doit comporter au moins une
    méthode qui récupère le prochain élément de la collection. Mais pour
    qu'elle soit vraiment utile, vous devriez en ajouter d'autres, par
    exemple récupérer l'élément précédent, connaitre la position
    actuelle ou vérifier la fin de l'itération.

2.  Déclarez l'interface de la collection et écrivez une méthode pour
    récupérer les itérateurs. Le type de la valeur de retour doit être
    l'interface de l'itérateur. Vous pouvez déclarer des itérateurs
    supplémentaires si vous pensez en utiliser d'autres.

3.  Mettez en place les classes concrètes des itérateurs pour les
    collections que vous allez parcourir. Un objet itérateur ne peut
    être lié qu'à une seule instance d'une collection. En général, ce
    lien est établi dans le constructeur de l'itérateur.

4.  Implémentez l'interface de la collection dans les classes de
    collection. Son but est de fournir un raccourci pour le client qui
    crée les itérateurs spécifiques à chaque classe de la collection.
    L'objet collection doit s'envoyer lui-même au constructeur de
    l'itérateur pour établir un lien entre eux.

5.  Écumez le code et remplacez toutes les occurrences de parcours de
    collection par vos itérateurs. Le client va chercher un nouvel
    itérateur chaque fois qu'il parcourt les éléments d'une collection.
:::

:::::: {.section .pros-cons}
##  Avantages et inconvénients {#pros-cons}

::::: row
::: col-sm-6
-    *Principe de responsabilité unique*. Vous allez nettoyer le code
    client et les collections en déplaçant les algorithmes de
    parcours --- souvent très lourds --- dans des classes séparées.
-    *Principe ouvert/fermé*. Vous pouvez implémenter de nouveaux types
    de collections et d'itérateurs et les utiliser avec le code existant
    sans rien endommager.
-    Vous pouvez parcourir une même collection avec plusieurs itérateurs
    simultanément, car chacun possède son propre état d'itération.
-    Pour cette même raison, vous pouvez arrêter une itération et la
    reprendre quand vous le souhaitez.
:::

::: col-sm-6
-    L'utilisation de ce patron est exagérée si votre application ne se
    sert que de collections simples.
-    Les itérateurs sont parfois moins efficaces que certaines
    collections spécialisées.
:::
:::::
::::::

::: {.section .relations}
##  Liens avec les autres patrons {#relations}

-   Vous pouvez utiliser les [Itérateurs](iterator.html) pour parcourir
    des arbres [Composites](composite.html).

-   Vous pouvez utiliser la [Fabrique](factory-method.html) avec
    l'[Itérateur](iterator.html) pour permettre aux sous-classes des
    collections de renvoyer différents types d'itérateurs compatibles
    avec les collections.

-   Vous pouvez utiliser le [Mémento](memento.html) avec
    l'[Itérateur](iterator.html) pour récupérer l'état actuel de
    l'itération et rétablir sa valeur plus tard si besoin.

-   Vous pouvez utiliser le [Visiteur](visitor.html) avec
    l'[Itérateur](iterator.html) pour parcourir une structure de données
    complexe et lancer un traitement sur ses éléments, même s'ils ont
    des classes différentes.
:::

::: {.section .implementations}
##  Exemples de code {#implementations}

[![Itérateur en
C#](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](iterator/csharp/example.html "Itérateur en C#"){.prog-lang-link}
[![Itérateur en
C++](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](iterator/cpp/example.html "Itérateur en C++"){.prog-lang-link}
[![Itérateur en
Go](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](iterator/go/example.html "Itérateur en Go"){.prog-lang-link}
[![Itérateur en
Java](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](iterator/java/example.html "Itérateur en Java"){.prog-lang-link}
[![Itérateur en
PHP](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](iterator/php/example.html "Itérateur en PHP"){.prog-lang-link}
[![Itérateur en
Python](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](iterator/python/example.html "Itérateur en Python"){.prog-lang-link}
[![Itérateur en
Ruby](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](iterator/ruby/example.html "Itérateur en Ruby"){.prog-lang-link}
[![Itérateur en
Rust](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](iterator/rust/example.html "Itérateur en Rust"){.prog-lang-link}
[![Itérateur en
Swift](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](iterator/swift/example.html "Itérateur en Swift"){.prog-lang-link}
[![Itérateur en
TypeScript](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](iterator/typescript/example.html "Itérateur en TypeScript"){.prog-lang-link}
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

[Médiateur []{.fa .fa-arrow-right}](mediator.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Retour

[[]{.fa .fa-arrow-left} Commande](command.html){.btn .btn-default
rel="prev"}
:::
::::::::::::::::::::::::::::::::::

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
::::::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::::::
