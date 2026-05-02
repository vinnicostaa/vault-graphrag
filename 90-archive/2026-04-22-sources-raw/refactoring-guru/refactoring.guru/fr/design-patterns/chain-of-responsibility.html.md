::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [Patrons de
conception](../design-patterns.html){.type} / [Patrons
comportementaux](behavioral-patterns.html){.type}
:::

# Chaîne de responsabilité {#chaîne-de-responsabilité .title}

::: aka
Alias : [CoR, ]{style="display:inline-block"}[Chaîne de
commande, ]{style="display:inline-block"}[Chain of
Responsibility]{style="display:inline-block"}
:::

::: {.section .intent}
##  Intention {#intent}

**Chaîne de responsabilité** est un patron de conception comportemental
qui permet de faire circuler des demandes dans une chaîne de handlers.
Lorsqu'un handler reçoit une demande, il décide de la traiter ou de
l'envoyer au handler suivant de la chaîne.

<figure class="image">
<img
src="../../images/patterns/content/chain-of-responsibility/chain-of-responsibilityf2d1.png?id=56c10d0dc712546cc283cfb3fb463458"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/chain-of-responsibility/chain-of-responsibility.png?id=56c10d0dc712546cc283cfb3fb463458 640w,/images/patterns/content/chain-of-responsibility/chain-of-responsibility-1.5x.png?id=770c03ad168fa797ec8ed4556ddf0b5c 960w,/images/patterns/content/chain-of-responsibility/chain-of-responsibility-2x.png?id=cc104b0a00a410f37fb39da80f392b88 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Patron de conception chaîne de responsabilité" />
</figure>
:::

::: {.section .problem}
##  Problème {#problem}

Imaginez que vous travaillez sur un système de commandes en ligne. Vous
voulez restreindre l'accès au système pour que seuls les utilisateurs
authentifiés puissent créer des commandes. Les utilisateurs qui ont les
autorisations administratives doivent avoir un accès total aux
commandes.

Après un travail de préparation, vous vous rendez compte que ces étapes
doivent être exécutées dans un ordre précis. L'application peut essayer
d'authentifier un utilisateur auprès du système lorsqu'il reçoit une
demande qui contient ses identifiants. Mais si ces derniers ne sont pas
corrects et que l'authentification échoue, ce n'est pas la peine de
lancer d'autres vérifications.

<figure class="image">
<img
src="../../images/patterns/diagrams/chain-of-responsibility/problem1-fr9516.png?id=b36e3f50e96fdb7a144ffd47da312552"
style="aspect-ratio: 2.5;"
srcset="/images/patterns/diagrams/chain-of-responsibility/problem1-fr.png?id=b36e3f50e96fdb7a144ffd47da312552 600w,/images/patterns/diagrams/chain-of-responsibility/problem1-fr-1.5x.png?id=7317c85bab7058230f4208e2018d8815 900w,/images/patterns/diagrams/chain-of-responsibility/problem1-fr-2x.png?id=b578afde6ad003c4cf9cbd2e767420c4 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Problème résolu par la chaîne de responsabilité" />
<figcaption><p>La demande doit d’abord passer par une série de
vérifications avant que le système de commandes ne prenne
le relais.</p></figcaption>
</figure>

Pendant les mois qui ont suivi, vous avez mis en place plusieurs
vérifications supplémentaires.

-   Un de vos collègues vous a fait remarquer qu'il n'était pas très
    prudent envoyer des données brutes directement dans le système de
    commandes. Vous avez donc ajouté une étape de validation
    supplémentaire pour purger les données de la demande.

-   Plus tard, quelqu'un a découvert que le système de mot de passe
    était vulnérable aux attaques par force brute. Vous avez ajouté une
    étape qui filtre les échecs répétés provenant de la même adresse IP
    pour y remédier.

-   Quelqu'un d'autre vous a suggéré d'améliorer la vitesse du système
    en envoyant les résultats directement depuis le cache, lorsque des
    demandes répétées retournent les mêmes résultats. Vous avez donc
    implémenté une autre étape qui laisse la demande passer si le
    système ne trouve pas la réponse correspondante dans le cache.

<figure class="image">
<img
src="../../images/patterns/diagrams/chain-of-responsibility/problem2-fr9b2a.png?id=9323722f2f5dc0ebe344ec2712ef690d"
style="aspect-ratio: 1.65;"
srcset="/images/patterns/diagrams/chain-of-responsibility/problem2-fr.png?id=9323722f2f5dc0ebe344ec2712ef690d 610w,/images/patterns/diagrams/chain-of-responsibility/problem2-fr-1.5x.png?id=bb7ba7e7764fa3b9c2b5e0397fbd7309 915w,/images/patterns/diagrams/chain-of-responsibility/problem2-fr-2x.png?id=baf38f59c17e09c014bbd7bca41ed1cf 1220w"
sizes="(max-width: 720px) 100vw, 610px" loading="lazy" width="610"
alt="Avec l’ajout de chaque nouvelle étape, le code devient plus gros, plus désordonné et plus moche" />
<figcaption><p>Plus le code grossit et plus il devient moche
et désordonné.</p></figcaption>
</figure>

Le code des vérifications --- qui n'était déjà pas très ordonné au
départ --- a grossi au fur et à mesure de l'ajout de nouvelles
fonctionnalités. La modification d'une vérification affecte parfois les
autres. Le pire dans tout cela, c'est qu'en voulant réutiliser les
vérifications existantes pour protéger les autres composants du système,
vous avez été obligé de dupliquer du code, car ces composants n'avaient
pas besoin de toutes les étapes.

Le système est devenu très difficile à comprendre et cher à maintenir.
Vous vous êtes débattu avec le code pendant un moment, jusqu'au jour où
vous avez décidé de tout refaire.
:::

::: {.section .solution}
##  Solution

Tout comme plusieurs patrons de conception comportementaux, la **Chaîne
de Responsabilité** repose sur la transformation de comportements
particuliers en objets autonomes que l'on appelle *handlers*. Dans notre
cas, chaque étape doit être extraite de sa propre classe avec une seule
méthode qui effectue la vérification. La demande est passée en paramètre
de la méthode avec toutes ses données.

Le patron vous propose de relier ces handlers par une chaîne. Chaque
handler stocke une référence vers le prochain handler de la chaîne dans
l'un de ses attributs. En plus de traiter la demande, les handlers la
font passer plus loin dans la chaîne. La demande fait le tour de la
chaîne jusqu'à ce que tous les handlers aient eu l'occasion de la
traiter.

Le mieux dans tout cela, c'est qu'un handler peut décider de ne pas
envoyer la demande plus loin dans la chaîne et de mettre fin à son
traitement.

Dans notre exemple du système de commandes, un handler effectue le
traitement et décide s'il doit envoyer la demande plus loin dans la
chaîne. Si la commande contient les bonnes données, les handlers peuvent
exécuter leur traitement, qu'il s'agisse de l'authentification ou de la
mise en cache.

<figure class="image">
<img
src="../../images/patterns/diagrams/chain-of-responsibility/solution1-frf753.png?id=44195a9f54a53cd9f96f697f0a7facff"
style="aspect-ratio: 4;"
srcset="/images/patterns/diagrams/chain-of-responsibility/solution1-fr.png?id=44195a9f54a53cd9f96f697f0a7facff 640w,/images/patterns/diagrams/chain-of-responsibility/solution1-fr-1.5x.png?id=1ff64aeb8c68ed94d54f8930eb05109d 960w,/images/patterns/diagrams/chain-of-responsibility/solution1-fr-2x.png?id=b2f0055f2bf33a29146d617e868af628 1280w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="640"
alt="Les handlers forment une chaîne" />
<figcaption><p>Les handlers forment une chaîne.</p></figcaption>
</figure>

Il existe une approche légèrement différente (un peu plus canonique)
dans laquelle un handler décide s'il traite la demande dès sa réception.
S'il peut la traiter, la demande n'ira pas plus loin. Dans ce cas de
figure, un seul handler s'occupera de traiter la demande (ou aucun).
C'est une approche classique utilisée dans les piles d'éléments d'une
interface graphique (GUI).

Par exemple, lorsqu'un utilisateur clique sur un bouton, l'événement est
propagé à travers la chaîne des éléments de la GUI. Cette chaîne débute
par le bouton, continue avec ses conteneurs (les formulaires ou les
panneaux) et se termine avec la fenêtre principale de l'application.
L'événement est traité par le premier élément de la chaîne qui est en
mesure de s'en occuper. Cet exemple est particulièrement intéressant,
car il démontre qu'une chaîne peut toujours être extraite depuis une
arborescence.

<figure class="image">
<img
src="../../images/patterns/diagrams/chain-of-responsibility/solution2-fr89a4.png?id=122669a759512193bf3159529b082406"
style="aspect-ratio: 1.73;"
srcset="/images/patterns/diagrams/chain-of-responsibility/solution2-fr.png?id=122669a759512193bf3159529b082406 520w,/images/patterns/diagrams/chain-of-responsibility/solution2-fr-1.5x.png?id=056ee649fa2a2dd578d50036ffedef15 780w,/images/patterns/diagrams/chain-of-responsibility/solution2-fr-2x.png?id=3381b2294d1ab06d84d15fa4709bfef8 1040w"
sizes="(max-width: 720px) 100vw, 520px" loading="lazy" width="520"
alt="Une chaîne peut être construite à partir de la branche d’une arborescence" />
<figcaption><p>Une chaîne peut être construite à partir de la branche
d’une arborescence.</p></figcaption>
</figure>

Les classes handler doivent toutes implémenter la même interface. Chaque
handler concret ne se préoccupe que de l'existence d'une méthode
`traiter` chez le handler suivant. Ainsi, vous pouvez créer vos chaînes
à l'exécution et utiliser divers handlers sans coupler votre code à
leurs classes concrètes.
:::

::: {.section .analogy}
##  Analogie {#analogy}

<figure class="image">
<img
src="../../images/patterns/content/chain-of-responsibility/chain-of-responsibility-comic-1-frf3be.png?id=d316dc995db49db3385fa67bbb77d5a6"
style="aspect-ratio: 2;"
srcset="/images/patterns/content/chain-of-responsibility/chain-of-responsibility-comic-1-fr.png?id=d316dc995db49db3385fa67bbb77d5a6 600w,/images/patterns/content/chain-of-responsibility/chain-of-responsibility-comic-1-fr-1.5x.png?id=5ad56673b313da4678bf456922c49488 900w,/images/patterns/content/chain-of-responsibility/chain-of-responsibility-comic-1-fr-2x.png?id=4038b2f2f47d6d71725371b87db52607 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Discuter avec le support technique se révèle parfois difficile" />
<figcaption><p>Un appel au support technique passe par
plusieurs opérateurs.</p></figcaption>
</figure>

Vous venez juste d'installer un nouveau composant sur votre ordinateur.
Comme vous êtes un geek, vous avez installé plusieurs systèmes
d'exploitation. Vous essayez de tous les démarrer pour voir si votre
matériel est bien pris en compte. Windows détecte votre nouveau
composant et l'active automatiquement. En revanche, votre petit chouchou
Linux refuse de le faire fonctionner. Avec un soupçon d'espoir, vous
appelez le support technique dont le numéro est indiqué sur la boîte.

Votre premier interlocuteur n'est autre que la voix robotique du
répondeur. Il vous propose neuf solutions à des problèmes classiques,
mais aucun ne vous concerne. Au bout d'un moment, le robot vous redirige
vers un opérateur humain.

Malheureusement, ce dernier ne vous répond rien de bien intéressant. Il
ne cesse de répéter des extraits de son manuel et ignore vos
commentaires. Après avoir entendu « avez-vous essayé de redémarrer votre
ordinateur ? » pour la dixième fois, vous demandez à parler avec un vrai
technicien.

Finalement, l'opérateur vous envoie vers un de leurs techniciens qui
était probablement assis tout seul depuis des heures dans la salle des
serveurs, située quelque part dans la cave sombre des bureaux de leur
entreprise, et attendait avec impatience de pouvoir parler à quelqu'un.
Le technicien vous indique un lien de téléchargement pour récupérer les
bons pilotes et la marche à suivre pour les installer sur Linux. Enfin,
la solution ! Vous mettez fin à l'appel, fou de joie !
:::

::::: {.section .structure-container}
##  Structure

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/chain-of-responsibility/structure03c9.png?id=848f0fc8dca57a44974d63f8181f5406"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 0.93;"
srcset="/images/patterns/diagrams/chain-of-responsibility/structure.png?id=848f0fc8dca57a44974d63f8181f5406 380w,/images/patterns/diagrams/chain-of-responsibility/structure-1.5x.png?id=49dfbae38f146af7319f80dce9ea7e2a 570w,/images/patterns/diagrams/chain-of-responsibility/structure-2x.png?id=bb837faaac88e7f2a16f751d0beaa201 760w"
sizes="(max-width: 720px) 100vw, 380px" loading="lazy" width="380"
alt="Structure du patron de conception chaîne de responsabilité" /><img
src="../../images/patterns/diagrams/chain-of-responsibility/structure-indexedfcb6.png?id=e13a5bf44f9ca47299223116af77cbef"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 0.88;"
srcset="/images/patterns/diagrams/chain-of-responsibility/structure-indexed.png?id=e13a5bf44f9ca47299223116af77cbef 380w,/images/patterns/diagrams/chain-of-responsibility/structure-indexed-1.5x.png?id=ca660e2cb697b512aadc07fdd3e109cd 570w,/images/patterns/diagrams/chain-of-responsibility/structure-indexed-2x.png?id=4f27e2c48e635f45a78472d707a8df3c 760w"
sizes="(max-width: 720px) 100vw, 380px" loading="lazy" width="380"
alt="Structure du patron de conception chaîne de responsabilité" />
</figure>
:::

1.  Le **Handler** déclare une interface commune pour tous les handlers
    concrets. En général, il ne comporte qu'une seule méthode pour gérer
    les demandes, mais il peut parfois en contenir une autre pour
    désigner le prochain handler de la chaîne.

2.  Le **Handler de Base** est une classe facultative dans laquelle le
    code commun à tous les handlers peut être écrit.

    En général, cette classe définit un attribut qui pointe vers le
    prochain handler. Les clients peuvent assembler une chaîne en
    passant un handler au constructeur ou au setter du handler
    précédent. La classe peut également implémenter le comportement par
    défaut d'un handler : il s'assure de l'existence du prochain
    handler, puis lui délègue le travail.

3.  Les **Handlers Concrets** contiennent le code qui traite les
    demandes. Lors de la réception d'une demande, chaque handler décide
    s'il doit la traiter et s'il doit l'envoyer plus loin dans la
    chaîne.

    Les handlers sont généralement autonomes et non modifiables, et
    n'accepteront qu'une seule fois les données nécessaires par le biais
    du constructeur.

4.  Le **Client** peut créer les chaînes juste une fois ou les assembler
    dynamiquement en fonction de la logique métier. Notez bien que la
    demande initiale n'est pas obligatoirement envoyée au premier
    handler de la chaîne.
::::
:::::

::: {.section .pseudocode}
##  Pseudo-code {#pseudocode}

Dans cet exemple, la **Chaîne de responsabilité** est chargée d'afficher
l'aide contextuelle pour les éléments actifs de la GUI.

<figure class="image">
<img
src="../../images/patterns/diagrams/chain-of-responsibility/example-fr00d8.png?id=574b82acede40a2d6cbfcc952f523584"
style="aspect-ratio: 1.09;"
srcset="/images/patterns/diagrams/chain-of-responsibility/example-fr.png?id=574b82acede40a2d6cbfcc952f523584 610w,/images/patterns/diagrams/chain-of-responsibility/example-fr-1.5x.png?id=23e9ded11f16598b2a25bacbc30d0a4d 915w,/images/patterns/diagrams/chain-of-responsibility/example-fr-2x.png?id=aefe60b991041ac9c614112b4b7f5893 1220w"
sizes="(max-width: 720px) 100vw, 610px" loading="lazy" width="610"
alt="Structure de l’exemple utilisé pour la chaîne de responsabilité" />
<figcaption><p>Les classes de la GUI sont construites à l’aide du patron
composite. Chaque élément est relié à son conteneur. À n’importe quel
moment, vous pouvez bâtir une chaîne d’éléments qui commence par
l’élément lui-même et parcourt tous ses conteneurs.</p></figcaption>
</figure>

La GUI de l'application prend généralement la forme d'une arborescence.
Par exemple, la classe `Dialogue` qui s'occupe du rendu de la fenêtre
principale de l'application, est la racine de l'arbre. La boîte de
dialogue contient des `Panneaux`, qui peuvent eux-mêmes être composés
d'autres panneaux ou d'éléments simples de plus bas niveau comme des
`Boutons` et des `ChampsTexte`.

Un composant simple peut afficher brièvement des infobulles
contextuelles si son texte d'aide a été configuré. Les composants plus
complexes ont leur propre manière d'afficher l'aide contextuelle. Ils
peuvent par exemple consulter l'aperçu du manuel ou ouvrir une page dans
un navigateur.

<figure class="image">
<img
src="../../images/patterns/diagrams/chain-of-responsibility/example2-fr6fbc.png?id=22da8e55fc16e4b1ef272eae1951bf03"
style="aspect-ratio: 0.81;"
srcset="/images/patterns/diagrams/chain-of-responsibility/example2-fr.png?id=22da8e55fc16e4b1ef272eae1951bf03 250w,/images/patterns/diagrams/chain-of-responsibility/example2-fr-1.5x.png?id=bb6f2e25191238de00e7ec935d511339 375w,/images/patterns/diagrams/chain-of-responsibility/example2-fr-2x.png?id=6d852636e01903a3e34d585d13a3ea86 500w"
sizes="(max-width: 720px) 100vw, 250px" loading="lazy" width="250"
alt="Structure de l’exemple utilisé pour la chaîne de responsabilité" />
<figcaption><p>Voici comment une demande d’aide parcourt les objets de
la GUI.</p></figcaption>
</figure>

Si un utilisateur positionne le pointeur de sa souris sur un élément et
appuie sur la touche `F1`, l'application détecte le composant situé sous
le pointeur et lui envoie une demande d'aide. La demande remonte vers la
surface en parcourant tous les conteneurs jusqu'à ce qu'elle atteigne un
élément qui peut afficher les informations de l'aide.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// L’interface du handler déclare une méthode pour exécuter la
// demande.
interface ComponentWithContextualHelp is
    method showHelp()


// La classe de base des composants simples.
abstract class Component implements ComponentWithContextualHelp is
    field tooltipText: string

    // Le conteneur du composant agit comme le prochain maillon
    // de la chaîne des handlers.
    protected field container: Container

    // Le composant affiche une infobulle si un texte d’aide y
    // est associé.  Sinon, il envoie l’appel vers le conteneur
    // (s’il existe).
    method showHelp() is
        if (tooltipText != null)
            // Affiche l’infobulle.
        else
            container.showHelp()


// Les conteneurs peuvent avoir des composants simples et
// d’autres conteneurs enfants. Les liens de la chaîne sont
// établis ici. La classe hérite du comportement de la méthode
// montrerAide (showHelp) de son parent.
abstract class Container extends Component is
    protected field children: array of Component

    method add(child) is
        children.add(child)
        child.container = this


// Les composants primitifs peuvent se contenter de l’aide par
// défaut...
class Button extends Component is
    // ...

// Mais les composants complexes peuvent redéfinir
// l’implémentation par défaut. Si le texte d’aide ne peut être
// fourni d’une autre manière, le composant peut toujours
// appeler l’implémentation de base (se référer à la classe
// Composant).
class Panel extends Container is
    field modalHelpText: string

    method showHelp() is
        if (modalHelpText != null)
            // Affiche une fenêtre modale avec le texte d’aide.
        else
            super.showHelp()

// ...idem qu’au-dessus...
class Dialog extends Container is
    field wikiPageURL: string

    method showHelp() is
        if (wikiPageURL != null)
            // Ouvre la page wiki d’aide.
        else
            super.showHelp()


// Code client.
class Application is
    // Chaque application configure la chaîne différemment.
    method createUI() is
        dialog = new Dialog(&quot;Budget Reports&quot;)
        dialog.wikiPageURL = &quot;http://...&quot;
        panel = new Panel(0, 0, 400, 800)
        panel.modalHelpText = &quot;This panel does...&quot;
        ok = new Button(250, 760, 50, 20, &quot;OK&quot;)
        ok.tooltipText = &quot;This is an OK button that...&quot;
        cancel = new Button(320, 760, 50, 20, &quot;Cancel&quot;)
        // ...
        panel.add(ok)
        panel.add(cancel)
        dialog.add(panel)

    // Imaginez ce qui se passe ici.
    method onF1KeyPress() is
        component = this.getComponentAtMouseCoords()
        component.showHelp()</code></pre>
</figure>
:::

:::::::::: {.section .applicability-container}
##  Possibilités d'application {#applicability}

::::::::: applicability
::: applicability-problem
Utilisez la chaîne de responsabilité quand votre programme doit traiter
des types de demandes variées de différentes manières, mais que leur
type exact et leur ordre dans la chaîne ne sont pas connus à l'avance.
:::

::: applicability-solution
Ce patron vous permet de former une chaîne avec les handlers et
d'interroger chacun d'entre eux lors de la réception de la demande afin
de savoir s'ils peuvent la traiter. Chaque handler a ainsi l'opportunité
de traiter la demande.
:::

::: applicability-problem
Utilisez ce patron si vos handlers doivent absolument respecter un ordre
donné.
:::

::: applicability-solution
Comme vous pouvez définir les liens entre les handlers de la chaîne, les
demandes la parcourront selon l'ordonnancement que vous avez configuré.
:::

::: applicability-problem
Utilisez la chaîne de responsabilité si l'ensemble des handlers et leur
ordre dans la chaîne peuvent changer lors de l'exécution.
:::

::: applicability-solution
Si vous fournissez des setters à un attribut à l'intérieur d'une classe
handler, vous serez en mesure d'ajouter, de retirer ou d'ordonner
dynamiquement les handlers.
:::
:::::::::
::::::::::

::: {.section .checklist}
##  Mise en œuvre {#checklist}

1.  Déclarez l'interface du handler et la méthode qui gère les demandes.

    Déterminez la manière dont le client passera les données de la
    demande dans la méthode. La manière la plus flexible consiste à
    convertir la demande en objet et à le passer en paramètre de la
    méthode.

2.  Pour éviter la duplication du code de base dans les handlers
    concrets, il peut être utile de créer une classe abstraite de base
    pour le handler, dérivée de l'interface handler.

    Cette classe doit posséder un attribut qui est une référence vers le
    prochain handler de la chaîne et vous devriez envisager de la rendre
    non modifiable. Mais si vous prévoyez de modifier les chaînes lors
    de l'exécution, vous devez définir des setters pour changer la
    valeur de l'attribut qui stocke les références.

    Vous pouvez également mettre en place le comportement par défaut des
    méthodes des handlers qui consiste à envoyer la demande au prochain
    objet (s'il en reste). Les handlers concrets peuvent utiliser ce
    comportement en faisant appel à la méthode de leur parent.

3.  Créez les sous-classes des handlers concrets une par une et mettez
    en place leurs traitements. Chaque handler doit prendre deux
    décisions lors de la réception d'une demande :

    -   Traiter ou non la demande.
    -   Passer ou non la demande au handler suivant de la chaîne.

4.  Le client doit assembler lui-même les chaînes ou recevoir des
    chaînes pré construites via d'autres objets. Dans le dernier cas,
    vous devez mettre en place des fabriques pour assembler des chaînes
    selon la configuration ou selon les paramètres d'environnement.

5.  Le client peut déclencher n'importe quel élément de la chaîne, pas
    forcément le premier. La demande continuera de parcourir la chaîne
    jusqu'à ce qu'un handler refuse de la laisser continuer, ou jusqu'à
    ce que l'on atteigne la fin de la chaîne.

6.  La chaîne étant dynamique, le client doit être capable de gérer les
    scénarios suivants :

    -   La chaîne peut être composée d'un unique lien.
    -   Certaines demandes n'iront pas jusqu'au bout de la chaîne.
    -   Certaines demandes atteindront la fin de la chaîne sans être
        traitées.
:::

:::::: {.section .pros-cons}
##  Avantages et inconvénients {#pros-cons}

::::: row
::: col-sm-6
-    Vous pouvez contrôler l'ordre des traitements de la demande.
-    *Principe de responsabilité unique*. Vous pouvez découpler les
    classes qui appellent des traitements, de celles qui les exécutent.
-    *Principe ouvert/fermé*. Vous pouvez ajouter de nouveaux handlers
    dans le programme sans toucher au code client existant.
:::

::: col-sm-6
-    Il se peut que certaines demandes ne soient pas traitées.
:::
:::::
::::::

::: {.section .relations}
##  Liens avec les autres patrons {#relations}

-   La [Chaîne de responsabilité](chain-of-responsibility.html), la
    [Commande](command.html), le [Médiateur](mediator.html) et
    l'[Observateur](observer.html) proposent différentes solutions pour
    associer les demandeurs et les récepteurs.
    -   La *chaîne de responsabilité* envoie une demande ordonnée qui
        est passée tout au long d'une chaîne dynamique de récepteurs
        potentiels, jusqu'à ce que l'un d'entre eux décide de la
        traiter.
    -   La *commande* établit des connexions unidirectionnelles entre
        les demandeurs et les récepteurs.
    -   Le *médiateur* élimine les liens directs entre les demandeurs et
        les récepteurs, et les force à communiquer indirectement via un
        objet médiateur.
    -   L'*observateur* permet aux récepteurs de s'inscrire et de se
        désinscrire dynamiquement à la réception des demandes.
:::

::: {.section .implementations}
##  Exemples de code {#implementations}

[![Chaîne de responsabilité en
C#](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](chain-of-responsibility/csharp/example.html "Chaîne de responsabilité en C#"){.prog-lang-link}
[![Chaîne de responsabilité en
C++](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](chain-of-responsibility/cpp/example.html "Chaîne de responsabilité en C++"){.prog-lang-link}
[![Chaîne de responsabilité en
Go](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](chain-of-responsibility/go/example.html "Chaîne de responsabilité en Go"){.prog-lang-link}
[![Chaîne de responsabilité en
Java](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](chain-of-responsibility/java/example.html "Chaîne de responsabilité en Java"){.prog-lang-link}
[![Chaîne de responsabilité en
PHP](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](chain-of-responsibility/php/example.html "Chaîne de responsabilité en PHP"){.prog-lang-link}
[![Chaîne de responsabilité en
Python](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](chain-of-responsibility/python/example.html "Chaîne de responsabilité en Python"){.prog-lang-link}
[![Chaîne de responsabilité en
Ruby](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](chain-of-responsibility/ruby/example.html "Chaîne de responsabilité en Ruby"){.prog-lang-link}
[![Chaîne de responsabilité en
Rust](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](chain-of-responsibility/rust/example.html "Chaîne de responsabilité en Rust"){.prog-lang-link}
[![Chaîne de responsabilité en
Swift](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](chain-of-responsibility/swift/example.html "Chaîne de responsabilité en Swift"){.prog-lang-link}
[![Chaîne de responsabilité en
TypeScript](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](chain-of-responsibility/typescript/example.html "Chaîne de responsabilité en TypeScript"){.prog-lang-link}
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

[Commande []{.fa .fa-arrow-right}](command.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Retour

[[]{.fa .fa-arrow-left} Patrons
comportementaux](behavioral-patterns.html){.btn .btn-default rel="prev"}
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
