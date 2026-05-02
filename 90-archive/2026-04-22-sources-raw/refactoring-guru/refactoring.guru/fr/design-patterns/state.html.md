::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [Patrons de
conception](../design-patterns.html){.type} / [Patrons
comportementaux](behavioral-patterns.html){.type}
:::

# État {#état .title}

::: aka
Alias : [State]{style="display:inline-block"}
:::

::: {.section .intent}
##  Intention {#intent}

**État** est un patron de conception comportemental qui permet de
modifier le comportement d'un objet lorsque son état interne change.
L'objet donne l'impression qu'il change de classe.

<figure class="image">
<img
src="../../images/patterns/content/state/state-fr720c.png?id=cf788ce84e25a638aea877abb0851122"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/state/state-fr.png?id=cf788ce84e25a638aea877abb0851122 640w,/images/patterns/content/state/state-fr-1.5x.png?id=e76b4929f11a70c00b2100198dc2a1e8 960w,/images/patterns/content/state/state-fr-2x.png?id=08468fde3542872fad10aa495faf0f77 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Patron de conception état" />
</figure>
:::

::: {.section .problem}
##  Problème {#problem}

Le patron de conception est très proche du concept de l'*Automate
fini* []{.pop-i}[Automate fini:
[https://refactoring.guru/fr/fsm](https://fr.wikipedia.org/wiki/Automate_fini)]{.pop-content}.

<figure class="image">
<img
src="../../images/patterns/diagrams/state/problem10244.png?id=503968745461a0970d1fbb4362dc186f"
style="aspect-ratio: 1.45;"
srcset="/images/patterns/diagrams/state/problem1.png?id=503968745461a0970d1fbb4362dc186f 320w,/images/patterns/diagrams/state/problem1-1.5x.png?id=47c7ca068eceaa9896d320690e6f3368 480w,/images/patterns/diagrams/state/problem1-2x.png?id=ae03c2233939eace11d44925ddeb912d 640w"
sizes="(max-width: 720px) 100vw, 320px" loading="lazy" width="320"
alt="Automate fini" />
<figcaption><p>Automate fini.</p></figcaption>
</figure>

Le principe repose sur le fait qu'un programme possède un nombre *fini*
d\'*états*. Le programme se comporte différemment selon son état et peut
en changer instantanément. En revanche, selon l'état dans lequel il se
trouve, certains états ne lui sont pas accessibles. Ces règles de
changement d'état sont appelées *transitions*. Elles sont également
finies et prédéterminées.

Vous pouvez appliquer cette approche aux objets. Imaginons une classe
`Document`. Un document peut être dans l'un des trois états suivants :
`Brouillon` (draft), `Modération` et `Publié`. La méthode `publier` du
document fonctionne un peu différemment en fonction de son état :

-   Dans `Brouillon`, elle passe le document en modération.
-   Dans `Modération`, elle rend le document public si l'utilisateur
    actuel est un administrateur.
-   Dans `Publié`, elle ne fait rien du tout.

<figure class="image">
<img
src="../../images/patterns/diagrams/state/problem2-fr26c5.png?id=eba507ba051c672def37cc17623044cb"
style="aspect-ratio: 1.27;"
srcset="/images/patterns/diagrams/state/problem2-fr.png?id=eba507ba051c672def37cc17623044cb 560w,/images/patterns/diagrams/state/problem2-fr-1.5x.png?id=e925684fada630fb9912747c1c1a29ae 840w,/images/patterns/diagrams/state/problem2-fr-2x.png?id=43041b7b40e280cccbdfa8b38e793b42 1120w"
sizes="(max-width: 720px) 100vw, 560px" loading="lazy" width="560"
alt="Les états possibles d’un objet document" />
<figcaption><p>Les états et transitions possibles d’un
objet document.</p></figcaption>
</figure>

Les automates sont généralement implémentés avec beaucoup d'opérateurs
conditionnels (`if` ou `switch`) qui choisissent le comportement
approprié en fonction de l'état actuel de l'objet. Cet « état » se
limite souvent à un ensemble de valeurs dans les attributs de l'objet.
Même si vous n'avez jamais entendu parler des automates finis, vous avez
probablement déjà implémenté un état au moins une fois. La structure du
code suivant vous dit-elle quelque chose ?

<figure class="code">
<pre class="code" lang="pseudocode"><code>class Document is
    field state: string
    // ...
    method publish() is
        switch (state)
            &quot;draft&quot;:
                state = &quot;moderation&quot;
                break
            &quot;moderation&quot;:
                if (currentUser.role == &quot;admin&quot;)
                    state = &quot;published&quot;
                break
            &quot;published&quot;:
                // Do nothing.
                break
    // ...</code></pre>
</figure>

La plus grosse faiblesse de l'automate fini devient visible lorsque l'on
commence à ajouter de plus en plus d'états et de comportements qui en
sont dépendants à la classe `Document`. La majorité des méthodes va
contenir d'énormes blocs de conditions qui vont choisir le comportement
d'une méthode en fonction de l'état actuel. Ce genre de code est très
difficile à maintenir, car tout changement dans la logique de transition
demande de modifier les états conditionnels dans chaque méthode.

Plus le projet évolue et plus cette faiblesse s'aggrave. Il est très
difficile de prédire tous les états et transitions possibles lors de la
phase de conception. Un automate fini doté d'un nombre limité de
conditions peut se transformer en un bazar pas possible au bout d'un
certain temps.
:::

::: {.section .solution}
##  Solution

Le patron de conception état propose de créer de nouvelles classes pour
tous les états possibles d'un objet et d'extraire les comportements liés
aux états dans ces classes.

Plutôt que d'implémenter tous les comportements de lui-même, l'objet
original que l'on nomme *contexte*, stocke une référence vers un des
objets état qui représente son état actuel. Il délègue tout ce qui
concerne la manipulation des états à cet objet.

<figure class="image">
<img
src="../../images/patterns/diagrams/state/solution-fred7a.png?id=8820b90f26abec5dd0324c2e9f9245f3"
style="aspect-ratio: 1.53;"
srcset="/images/patterns/diagrams/state/solution-fr.png?id=8820b90f26abec5dd0324c2e9f9245f3 490w,/images/patterns/diagrams/state/solution-fr-1.5x.png?id=ee830837e1497e511e478b722e43793a 735w,/images/patterns/diagrams/state/solution-fr-2x.png?id=3e5e6688fdbba1a5ce225a59aa8ed3c0 980w"
sizes="(max-width: 720px) 100vw, 490px" loading="lazy" width="490"
alt="Document délègue la tâche à un objet état" />
<figcaption><p>Document délègue la tâche à un
objet état.</p></figcaption>
</figure>

Pour faire passer le contexte dans un autre état, remplacez l'objet état
par un autre qui représente son nouvel état. Vous ne pourrez le faire
que si toutes les classes suivent la même interface et si le contexte
utilise cette dernière pour manipuler ces objets.

Cette structure ressemble de près au patron de conception
[Stratégie](strategy.html), mais il y a une différence majeure. Dans le
patron de conception état, les états ont de la visibilité entre eux et
peuvent lancer les transitions d'un état à l'autre, alors que les
stratégies ne peuvent pas se voir.
:::

::: {.section .analogy}
##  Analogie {#analogy}

Les boutons de votre smartphone fonctionnent différemment selon l'état
de l'appareil :

-   Si le téléphone est déverrouillé, appuyer sur des boutons lance
    différentes fonctionnalités.
-   Si le téléphone est verrouillé, appuyer sur n'importe quel bouton
    envoie sur l'écran de déverrouillage.
-   Si la batterie du téléphone est faible, appuyer sur n'importe quel
    bouton montre l'écran de charge.
:::

::::: {.section .structure-container}
##  Structure

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/state/structure-fra11e.png?id=9e32a438e03129fdae6b4693a3ccaf17"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.32;"
srcset="/images/patterns/diagrams/state/structure-fr.png?id=9e32a438e03129fdae6b4693a3ccaf17 540w,/images/patterns/diagrams/state/structure-fr-1.5x.png?id=3358678223562d59254557c192d57422 810w,/images/patterns/diagrams/state/structure-fr-2x.png?id=0c7f21a32c3e0a75eac1c01a72e11e60 1080w"
sizes="(max-width: 720px) 100vw, 540px" loading="lazy" width="540"
alt="Structure du patron de conception état" /><img
src="../../images/patterns/diagrams/state/structure-fr-indexeda5c1.png?id=ba24712cc633eead27cce50453fd541e"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.26;"
srcset="/images/patterns/diagrams/state/structure-fr-indexed.png?id=ba24712cc633eead27cce50453fd541e 540w,/images/patterns/diagrams/state/structure-fr-indexed-1.5x.png?id=6e6ca39aad5d67fc7a800bb796e9ab62 810w,/images/patterns/diagrams/state/structure-fr-indexed-2x.png?id=38fa5e17e2f8c6a9f86da44676a7c2b8 1080w"
sizes="(max-width: 720px) 100vw, 540px" loading="lazy" width="540"
alt="Structure du patron de conception état" />
</figure>
:::

1.  Le **Contexte** stocke une référence vers un des objets concrets
    État et lui délègue toutes les tâches concernant les états. Il
    utilise l'interface état pour communiquer avec l'objet état. Il
    expose un setter pour lui passer un nouvel état.

2.  L'interface **État** déclare les méthodes spécifiques aux états. Ces
    méthodes doivent fonctionner avec tous les états concrets : des
    méthodes inutiles qui ne sont jamais appelées à l'intérieur de vos
    états sont à proscrire.

3.  Les **États Concrets** fournissent leurs propres implémentations aux
    méthodes qui agissent sur les états. Pour éviter d'écrire le même
    code dans les différents états, vous pouvez créer des classes
    abstraites intermédiaires qui encapsulent les comportements
    identiques.

    Les états peuvent garder une référence vers le contexte. Grâce à
    cette référence, l'état peut récupérer des informations depuis le
    contexte et lancer des transitions.

4.  Le contexte et les états concrets peuvent modifier le prochain état
    du contexte et lancer une transition en remplaçant l'état lié au
    contexte.
::::
:::::

::: {.section .pseudocode}
##  Pseudo-code {#pseudocode}

Dans cet exemple, le patron de conception **État** permet aux touches du
lecteur multimédia d'avoir un comportement relatif à l'état actuel de la
lecture.

<figure class="image">
<img
src="../../images/patterns/diagrams/state/example1dc9.png?id=1ffdb109b3ebb85d223b9d1651d2034c"
style="aspect-ratio: 1.64;"
srcset="/images/patterns/diagrams/state/example.png?id=1ffdb109b3ebb85d223b9d1651d2034c 590w,/images/patterns/diagrams/state/example-1.5x.png?id=a9ff56d0a139530fa103d496513dfa06 885w,/images/patterns/diagrams/state/example-2x.png?id=cd81e3ffb8aba5883983a59c111b805f 1180w"
sizes="(max-width: 720px) 100vw, 590px" loading="lazy" width="590"
alt="Structure de l’exemple utilisé pour le patron de conception état" />
<figcaption><p>Un exemple de modification du comportement de l’objet
effectué à l’aide d’objets état.</p></figcaption>
</figure>

L'objet principal du lecteur est toujours associé à un objet état qui
effectue la majeure partie du travail pour le lecteur. Certaines actions
remplacent l'état actuel du lecteur par un autre, modifiant sa manière
de réagir aux interactions de l'utilisateur.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// La classe lecteurAudio prend le rôle du contexte. Elle
// maintient également une référence vers l’instance de l’une
// des classes état qui représente l’état actuel du lecteur
// audio.
class AudioPlayer is
    field state: State
    field UI, volume, playlist, currentSong

    constructor AudioPlayer() is
        this.state = new ReadyState(this)

        // Le contexte délègue la gestion des interventions de
        // l’utilisateur à un objet état. Le résultat va
        // évidemment dépendre de l&#39;état actuel, puisque chaque
        // état réagit différemment aux manipulations des
        // utilisateurs.
        UI = new UserInterface()
        UI.lockButton.onClick(this.clickLock)
        UI.playButton.onClick(this.clickPlay)
        UI.nextButton.onClick(this.clickNext)
        UI.prevButton.onClick(this.clickPrevious)

    // Les autres objets doivent pouvoir changer l&#39;état du
    // lecteur audio.
    method changeState(state: State) is
        this.state = state

    // Les méthodes de l’UI délèguent l’exécution à l&#39;état
    // actuel.
    method clickLock() is
        state.clickLock()
    method clickPlay() is
        state.clickPlay()
    method clickNext() is
        state.clickNext()
    method clickPrevious() is
        state.clickPrevious()

    // Un état peut appeler les méthodes d’un service sur le
    // contexte.
    method startPlayback() is
        // ...
    method stopPlayback() is
        // ...
    method nextSong() is
        // ...
    method previousSong() is
        // ...
    method fastForward(time) is
        // ...
    method rewind(time) is
        // ...


// La classe de base état déclare des méthodes que tous les
// états concrets doivent obligatoirement implémenter et fournit
// aussi une référence arrière vers l’objet du contexte associé
// à l’état. Les états peuvent utiliser cette référence arrière
// pour permuter l’état du contexte.
abstract class State is
    protected field player: AudioPlayer

    // Le contexte s’envoie lui-même au constructeur de l’état,
    // permettant de donner un coup de pouce à l&#39;état pour
    // récupérer des données contextuelles si nécessaire.
    constructor State(player) is
        this.player = player

    abstract method clickLock()
    abstract method clickPlay()
    abstract method clickNext()
    abstract method clickPrevious()


// Les états concrets implémentent différents comportements
// associés à un état du contexte.
class LockedState extends State is

    // Lorsque vous déverrouillez un lecteur verrouillé, il peut
    // prendre l’un des deux états.
    method clickLock() is
        if (player.playing)
            player.changeState(new PlayingState(player))
        else
            player.changeState(new ReadyState(player))

    method clickPlay() is
        // Verrouillé, ne rien faire.

    method clickNext() is
        // Verrouillé, ne rien faire.

    method clickPrevious() is
        // Verrouillé, ne rien faire.


// Ils peuvent également déclencher les transitions de l’état
// dans le contexte.
class ReadyState extends State is
    method clickLock() is
        player.changeState(new LockedState(player))

    method clickPlay() is
        player.startPlayback()
        player.changeState(new PlayingState(player))

    method clickNext() is
        player.nextSong()

    method clickPrevious() is
        player.previousSong()


class PlayingState extends State is
    method clickLock() is
        player.changeState(new LockedState(player))

    method clickPlay() is
        player.stopPlayback()
        player.changeState(new ReadyState(player))

    method clickNext() is
        if (event.doubleclick)
            player.nextSong()
        else
            player.fastForward(5)

    method clickPrevious() is
        if (event.doubleclick)
            player.previous()
        else
            player.rewind(5)</code></pre>
</figure>
:::

:::::::::: {.section .applicability-container}
##  Possibilités d'application {#applicability}

::::::::: applicability
::: applicability-problem
Utilisez le patron de conception état lorsque le comportement de l'un de
vos objets varie en fonction de son état, qu'il y a beaucoup d'états
différents et que ce code change souvent.
:::

::: applicability-solution
Ce patron vous propose d'extraire tout le code lié aux états et de le
mettre dans des classes distinctes. Ceci vous permet d'ajouter de
nouveaux états ou de modifier ceux qui existent indépendamment des
autres, et de réduire les coûts de maintenance.
:::

::: applicability-problem
Utilisez ce patron si l'une de vos classes est polluée par d'énormes
blocs conditionnels qui modifient le comportement de la classe en
fonction de la valeur de ses attributs.
:::

::: applicability-solution
Le patron de conception état vous permet d'extraire des branches de ces
conditions et de les transformer en méthodes dans les classes état. Tout
en faisant vos modifications, vous pouvez retirer les attributs
temporaires et les méthodes qui gèrent les changements d'état du code de
votre classe principale.
:::

::: applicability-problem
Utilisez ce patron de conception si vous avez trop de code dupliqué dans
des états et transitions similaires de votre automate.
:::

::: applicability-solution
Le patron de conception état vous permet d'assembler des hiérarchies de
classes état et de réduire la duplication de code en regroupant le code
commun dans des classes de base abstraites.
:::
:::::::::
::::::::::

::: {.section .checklist}
##  Mise en œuvre {#checklist}

1.  Choisissez la classe qui va prendre le rôle du contexte. Cette
    classe peut déjà exister et posséder du code qui gère les états,
    mais vous pouvez en créer une nouvelle si ce code est réparti dans
    plusieurs classes.

2.  Déclarez l'interface état. Vous pourriez très bien vous contenter de
    recopier toutes les méthodes déclarées dans le contexte, mais ne
    reprenez que celles qui concernent les états.

3.  Pour chaque état, créez une classe qui dérive de l'interface état.
    Parcourez ensuite les méthodes du contexte pour identifier le code
    qui concerne cet état et recopiez-le dans votre nouvelle classe.

    En effectuant cette manipulation, vous pourriez tomber sur des
    membres privés dans le contexte. Il y a plusieurs moyens de
    contournement :

    -   Rendez ces attributs ou ces méthodes publics.
    -   Transformez le comportement que vous extrayez en méthode
        publique que vous mettez dans le contexte, puis appelez-la
        depuis la classe état. Ce n'est pas le plus esthétique, mais
        vous pourrez revenir dessus plus tard.
    -   Imbriquez les classes état dans la classe contexte si votre
        langage de programmation le permet.

4.  Dans votre classe contexte, ajoutez un attribut qui référence le
    type de l'interface état et un setter public qui permet de redéfinir
    la valeur de cet attribut.

5.  Parcourez à nouveau les méthodes du contexte et remplacez les
    conditions concernant les états par des appels aux méthodes
    correspondantes de l'objet état.

6.  Pour changer l'état du contexte, créez une instance de l'une des
    classes état et passez-la au contexte. Ceci peut être fait à
    l'intérieur du contexte, dans les différents états ou dans le
    client. Où qu'elle soit, cette classe devient dépendante de la
    classe concrète État qu'elle instancie.
:::

:::::: {.section .pros-cons}
##  Avantages et inconvénients {#pros-cons}

::::: row
::: col-sm-6
-    *Principe de responsabilité unique*. Organisez le code lié aux
    différents états dans des classes séparées.
-    *Principe ouvert/fermé*. Ajoutez de nouveaux états sans modifier
    les classes état ou le contexte existants.
-    Simplifiez le code du contexte en éliminant les gros blocs
    conditionnels de l'automate.
:::

::: col-sm-6
-    L'utilisation de ce patron est un peu exagérée si votre automate
    n'a que quelques états ou qu'il y a peu de transitions.
:::
:::::
::::::

::: {.section .relations}
##  Liens avec les autres patrons {#relations}

-   Le [Pont](bridge.html), l'[État](state.html), la
    [Stratégie](strategy.html) (et dans une certaine mesure
    l'[Adaptateur](adapter.html)) ont des structures très similaires. En
    effet, ces patrons sont basés sur la composition, qui délègue les
    tâches aux autres objets. Cependant, ils résolvent différents
    problèmes. Un patron n'est pas juste une recette qui vous aide à
    structurer votre code d'une certaine manière. C'est aussi une façon
    de communiquer aux autres développeurs le problème qu'il résout.

-   L'[État](state.html) peut être considéré comme une extension de la
    [Stratégie](strategy.html). Ces deux patrons de conception sont
    basés sur la composition : ils changent le comportement du contexte
    en déléguant certaines tâches aux objets assistant. La *stratégie*
    rend ces objets complètement indépendants sans aucune visibilité
    l'un sur l'autre. Cependant, l'*état* n'impose pas de restrictions
    sur les dépendances entre les états concrets, et leur laisse
    modifier l'état du contexte à volonté.
:::

::: {.section .implementations}
##  Exemples de code {#implementations}

[![État en
C#](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](state/csharp/example.html "État en C#"){.prog-lang-link}
[![État en
C++](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](state/cpp/example.html "État en C++"){.prog-lang-link}
[![État en
Go](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](state/go/example.html "État en Go"){.prog-lang-link}
[![État en
Java](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](state/java/example.html "État en Java"){.prog-lang-link}
[![État en
PHP](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](state/php/example.html "État en PHP"){.prog-lang-link}
[![État en
Python](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](state/python/example.html "État en Python"){.prog-lang-link}
[![État en
Ruby](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](state/ruby/example.html "État en Ruby"){.prog-lang-link}
[![État en
Rust](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](state/rust/example.html "État en Rust"){.prog-lang-link}
[![État en
Swift](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](state/swift/example.html "État en Swift"){.prog-lang-link}
[![État en
TypeScript](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](state/typescript/example.html "État en TypeScript"){.prog-lang-link}
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

[Stratégie []{.fa .fa-arrow-right}](strategy.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Retour

[[]{.fa .fa-arrow-left} Observateur](observer.html){.btn .btn-default
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
