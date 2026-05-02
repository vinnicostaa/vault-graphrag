:::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [Patrons de
conception](../design-patterns.html){.type} / [Patrons
structurels](structural-patterns.html){.type}
:::

# Pont {#pont .title}

::: aka
Alias : [Bridge]{style="display:inline-block"}
:::

::: {.section .intent}
##  Intention {#intent}

Le **Pont** est un patron de conception structurel qui permet de séparer
une grosse classe ou un ensemble de classes connexes en deux
hiérarchies --- abstraction et implémentation --- qui peuvent évoluer
indépendamment l'une de l'autre.

<figure class="image">
<img
src="../../images/patterns/content/bridge/bridge3e59.png?id=bd543d4fb32e11647767301581a5ad54"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/bridge/bridge.png?id=bd543d4fb32e11647767301581a5ad54 640w,/images/patterns/content/bridge/bridge-1.5x.png?id=8ef07b78d61cc3830b7e2b783c76c775 960w,/images/patterns/content/bridge/bridge-2x.png?id=1e905ae5742e5cd10a7eb0e3175ef00d 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Patron de conception pont" />
</figure>
:::

::: {.section .problem}
##  Problème {#problem}

*Abstraction ?* *Implémentation ?* Ces termes vous donnent des
frissons ? Rassurez-vous, nous allons prendre un exemple simple.

Prenons une classe de `Forme` géométrique avec les sous-classes
suivantes : `Cercle` et `Carré`. Vous voulez étendre cette hiérarchie de
classes pour incorporer des couleurs, vous créez donc des sous-classes
de formes : `Rouge` et `Bleu`. Mais vous avez déjà deux sous-classes,
vous devez donc créer quatre combinaisons comme par exemple `CercleBleu`
et `CarréRouge`.

<figure class="image">
<img
src="../../images/patterns/diagrams/bridge/problem-fr03da.png?id=d2e672a9422dedeb1a3f9de811ec60db"
style="aspect-ratio: 1.41;"
srcset="/images/patterns/diagrams/bridge/problem-fr.png?id=d2e672a9422dedeb1a3f9de811ec60db 480w,/images/patterns/diagrams/bridge/problem-fr-1.5x.png?id=069c74e7ecd09cbc6fda2f370fe15ed7 720w,/images/patterns/diagrams/bridge/problem-fr-2x.png?id=1b2e373f9204d236d5fafc3872088075 960w"
sizes="(max-width: 720px) 100vw, 480px" loading="lazy" width="480"
alt="Problème du patron de conception pont" />
<figcaption><p>Le nombre de combinaisons
augmente exponentiellement.</p></figcaption>
</figure>

Ajouter de nouvelles formes et couleurs va augmenter la taille de la
hiérarchie exponentiellement. Par exemple, pour ajouter une forme
triangle, vous devez créer deux nouvelles sous-classes : une par
couleur. Ensuite, ajouter une couleur demandera trois nouvelles
sous-classes : une pour chaque forme. Si l'on continue ainsi, la
situation ne fait qu'empirer.
:::

::: {.section .solution}
##  Solution

Nous rencontrons ce problème, car nous essayons d'étendre les classes
forme dans deux dimensions indépendantes : la forme et la couleur. C'est
un problème classique causé par l'héritage.

Le pont tente de résoudre ce problème en utilisant la composition à la
place de l'héritage. Pour ce faire, vous insérez une des dimensions dans
une hiérarchie de classes séparée afin que la classe originale puisse
référencer un objet de cette nouvelle hiérarchie, plutôt que de réunir
tous les états et comportements à l'intérieur d'une même classe.

<figure class="image">
<img
src="../../images/patterns/diagrams/bridge/solution-fr2c63.png?id=87f747bd967a5d5c03de44a57539e262"
style="aspect-ratio: 2.3;"
srcset="/images/patterns/diagrams/bridge/solution-fr.png?id=87f747bd967a5d5c03de44a57539e262 460w,/images/patterns/diagrams/bridge/solution-fr-1.5x.png?id=43d4c1f551f2c3f2c0a96739e66733ce 690w,/images/patterns/diagrams/bridge/solution-fr-2x.png?id=a6fd3c8291fe08e7000be7b44e571d02 920w"
sizes="(max-width: 720px) 100vw, 460px" loading="lazy" width="460"
alt="Solution proposée par le patron de conception pont" />
<figcaption><p>Vous évitez l’explosion de la hiérarchie de classes en la
transformant en plusieurs hiérarchies connexes.</p></figcaption>
</figure>

Nous allons appliquer ce procédé et récupérer le code concernant les
couleurs dans sa propre classe avec deux sous-classes : `Rouge` et
`Bleu`. La classe `Forme` est ensuite dotée d'un attribut qui référence
l'un des objets couleur. La forme peut maintenant déléguer tous les
traitements concernant la couleur de l'objet. Cette référence agira
comme un pont entre les classes `Forme` et `Couleur`. Dorénavant,
l'ajout de nouvelles couleurs ne bouleversera plus la hiérarchie des
formes et inversement.

#### Abstraction et implémentation

Le livre du GoF []{.pop-i}[« La bande des quatre » (Gang of Four/GoF)
est le surnom donné aux quatre auteurs du livre sur les patrons de
conception : *Design patterns. Catalogue de modèles de conception
réutilisables*
[https://refactoring.guru/fr/gof-book](https://www.amazon.com/-/dp/2711786447).]{.pop-content}
ajoute les termes *abstraction* et *implémentation* à la définition du
pont. Trop académiques, ces termes rendent le patron plus compliqué
qu'il ne l'est en réalité. En gardant en tête l'exemple avec les formes
et couleurs, déchiffrons la signification de ces termes barbares.

L'*abstraction* (aussi appelée *interface*) est une couche de contrôle
de haut niveau pour une entité. Cette couche n'est pas censée effectuer
de traitements toute seule. Elle doit déléguer le travail à la couche
*implémentation* (appelée également *plateforme*).

Notez bien qu'il ne s'agit pas des *interfaces* ou des *classes
abstraites* d'un langage de programmation.

Si l'on prend un logiciel comme exemple concret, l'interface utilisateur
graphique (GUI) prend le rôle de l'abstraction et le code du système
d'exploitation (API) prend le rôle de l'implémentation que la couche GUI
appelle en réponse aux interactions de l'utilisateur.

D'une manière générale, vous pouvez développer un tel programme en deux
parties indépendantes :

-   Disposer de plusieurs GUI différentes (on lance celle qui est
    adaptée à l'utilisateur, client ou admin).
-   Gérer plusieurs API différentes (pouvoir exécuter l'application sous
    Windows, Linux et macOS).

Dans le pire des cas, cette application pourrait ressembler à un énorme
plat de spaghettis avec des centaines de conditions connectant
différents types de GUI et d'API, réparties un peu partout à travers le
code.

<figure class="image">
<img
src="../../images/patterns/content/bridge/bridge-3-fr4b31.png?id=6ae81f251cfa7a4e8c869d90decb7c08"
style="aspect-ratio: 2;"
srcset="/images/patterns/content/bridge/bridge-3-fr.png?id=6ae81f251cfa7a4e8c869d90decb7c08 600w,/images/patterns/content/bridge/bridge-3-fr-1.5x.png?id=607485a5f2605adb17bff3a46d184f2d 900w,/images/patterns/content/bridge/bridge-3-fr-2x.png?id=9acae58232ee890b3707d8ede773989d 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="En programmation modulaire, la gestion des modifications est bien plus simple" />
<figcaption><p>La moindre modification dans une architecture
monolithique peut se révéler compliquée, car vous devez comprendre
<em>la totalité du code</em>. Vous pouvez facilement effectuer des
modifications dans de plus petits modules bien définis.</p></figcaption>
</figure>

Vous pouvez mettre de l'ordre dans ce chaos en rangeant le code
concernant les combinaisons spécifiques de l'interface/plateforme dans
des classes séparées, mais vous découvrirez rapidement que ces classes
sont *nombreuses*. La hiérarchie des classes croît rapidement, car
l'ajout d'une nouvelle GUI ou l'intégration d'une nouvelle API
requièrent la création de classes supplémentaires.

Tentons de résoudre ce problème avec le pont. Il nous propose de mettre
les classes dans deux hiérarchies :

-   Abstraction : la couche GUI de l'application.
-   Implémentation : les API des systèmes d'exploitation.

<figure class="image">
<img
src="../../images/patterns/content/bridge/bridge-2-fr6f10.png?id=45fce938a080c613c742c2c2ffa413f7"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/bridge/bridge-2-fr.png?id=45fce938a080c613c742c2c2ffa413f7 640w,/images/patterns/content/bridge/bridge-2-fr-1.5x.png?id=abf6e9e6da1ed4c78726e5a9b4eee8d2 960w,/images/patterns/content/bridge/bridge-2-fr-2x.png?id=1520e181effe59ef60bca88d6a59e6ed 1280w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="640"
alt="Architecture multiplateforme" />
<figcaption><p>Une des techniques pour organiser une
application multiplateforme.</p></figcaption>
</figure>

L'objet abstraction contrôle l'apparence de l'application et délègue la
partie métier à l'objet d'implémentation correspondant. Les
implémentations sont interchangeables tant qu'elles implémentent la même
interface, ce qui permet à la même GUI de fonctionner aussi bien sous
Windows que sous Linux.

Grâce à cela, vous pouvez modifier les classes de la GUI sans toucher
aux classes des API. De plus, adapter le code pour gérer un autre
système d'exploitation ne requiert que l'ajout d'une sous-classe dans la
hiérarchie de l'implémentation.
:::

::::: {.section .structure-container}
##  Structure

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/bridge/structure-fr2c93.png?id=5504468a65337fa9900415595b93a34a"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.47;"
srcset="/images/patterns/diagrams/bridge/structure-fr.png?id=5504468a65337fa9900415595b93a34a 560w,/images/patterns/diagrams/bridge/structure-fr-1.5x.png?id=7d8283eeffac227215f7b8bcb54d98d4 840w,/images/patterns/diagrams/bridge/structure-fr-2x.png?id=a9fb73e7adb5a49e150b0796dc089206 1120w"
sizes="(max-width: 720px) 100vw, 560px" loading="lazy" width="560"
alt="Patron de conception pont" /><img
src="../../images/patterns/diagrams/bridge/structure-fr-indexed42b6.png?id=8dc35879c6cbefc162ffb5e8e6736038"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.44;"
srcset="/images/patterns/diagrams/bridge/structure-fr-indexed.png?id=8dc35879c6cbefc162ffb5e8e6736038 560w,/images/patterns/diagrams/bridge/structure-fr-indexed-1.5x.png?id=23935b248e863f5908428b090ea8e271 840w,/images/patterns/diagrams/bridge/structure-fr-indexed-2x.png?id=370d0dd30a67ce28b1522d9b833bf479 1120w"
sizes="(max-width: 720px) 100vw, 560px" loading="lazy" width="560"
alt="Patron de conception pont" />
</figure>
:::

1.  L'**Abstraction** offre une logique de contrôle de haut niveau. Elle
    compte sur l'objet de l'implémentation pour s'occuper des tâches de
    bas niveau.

2.  L'**Implémentation** déclare une interface commune pour toutes les
    implémentations concrètes. L'abstraction ne peut communiquer avec
    les objets de l'implémentation que grâce aux méthodes qui y sont
    déclarées.

    L'abstraction peut contenir les mêmes méthodes que l'implémentation,
    mais en général l'abstraction déclare des comportements complexes
    qui reposent sur une grande variété d'opérations primitives
    déclarées par l'implémentation.

3.  Les **Implémentations Concrètes** contiennent du code spécialisé
    pour les plateformes.

4.  L'**Abstraction Fine** procure des variantes pour la logique de
    contrôle. Tout comme leur parent, elles travaillent avec différentes
    implémentations en passant par l'interface d'implémentation
    principale.

5.  En général, le **Client** ne veut interagir qu'avec l'abstraction,
    mais c'est son rôle de faire correspondre l'objet d'abstraction avec
    un des objets d'implémentation.
::::
:::::

::: {.section .pseudocode}
##  Pseudo-code {#pseudocode}

Cet exemple montre comment le **Pont** aide à diviser le code
monolithique d'une application qui gère les appareils et leurs
télécommandes. Les `Appareils` prennent le rôle de l'implémentation et
les `Télécommandes` font office d'abstraction.

<figure class="image">
<img
src="../../images/patterns/diagrams/bridge/example-fr6e18.png?id=0b419af02ed13b641ada1841a381fbb2"
style="aspect-ratio: 1.33;"
srcset="/images/patterns/diagrams/bridge/example-fr.png?id=0b419af02ed13b641ada1841a381fbb2 640w,/images/patterns/diagrams/bridge/example-fr-1.5x.png?id=840f6771ea7390cce9299241ea4d2543 960w,/images/patterns/diagrams/bridge/example-fr-2x.png?id=1b03d177fcdf37c08f27bd0b01e96a43 1280w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="640"
alt="Structure de l’exemple utilisé pour le pont" />
<figcaption><p>La hiérarchie de la classe originale est divisée en deux
parties : appareils et télécommandes.</p></figcaption>
</figure>

La classe de base télécommande déclare un attribut de référence qui la
lie avec un objet appareil. Toutes les télécommandes utilisent
l'interface principale des appareils, ce qui leur permet de fonctionner
avec tous les types d'appareils.

Vous pouvez faire évoluer les télécommandes indépendamment des
appareils, vous devez juste créer une nouvelle sous-classe de
télécommande. Par exemple, une télécommande basique pourrait juste avoir
deux boutons, mais vous pouvez lui rajouter des fonctionnalités comme
une batterie supplémentaire ou un écran tactile.

Le code client établit le lien entre le type de télécommande désiré et
un appareil spécifique en passant par le constructeur de la
télécommande.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// L’« abstraction » définit l’interface pour la partie
// « télécommande » des deux hiérarchies de classes. Elle garde
// une référence sur un objet de la hiérarchie de
// l’« implémentation » et lui délègue les tâches.
class RemoteControl is
    protected field device: Device
    constructor RemoteControl(device: Device) is
        this.device = device
    method togglePower() is
        if (device.isEnabled()) then
            device.disable()
        else
            device.enable()
    method volumeDown() is
        device.setVolume(device.getVolume() - 10)
    method volumeUp() is
        device.setVolume(device.getVolume() + 10)
    method channelDown() is
        device.setChannel(device.getChannel() - 1)
    method channelUp() is
        device.setChannel(device.getChannel() + 1)


// Vous pouvez étendre les classes de la hiérarchie de
// l’abstraction indépendamment des classes appareil.
class AdvancedRemoteControl extends RemoteControl is
    method mute() is
        device.setVolume(0)


// L’interface de l’implémentation déclare les méthodes communes
// à toutes les classes concrètes de l’implémentation. Elle n’a
// pas besoin de correspondre à l’interface de l’abstraction. En
// fait, les deux interfaces peuvent être complètement
// différentes. En général, l’interface de l’implémentation ne
// fournit que des opérations primitives, alors que
// l’abstraction définit des opérations de plus haut niveau
// basées sur ces primitives.
interface Device is
    method isEnabled()
    method enable()
    method disable()
    method getVolume()
    method setVolume(percent)
    method getChannel()
    method setChannel(channel)


// Tous les appareils suivent la même interface.
class Tv implements Device is
    // ...

class Radio implements Device is
    // ...


// Quelque part dans le code client.
tv = new Tv()
remote = new RemoteControl(tv)
remote.togglePower()

radio = new Radio()
remote = new AdvancedRemoteControl(radio)</code></pre>
</figure>
:::

:::::::::: {.section .applicability-container}
##  Possibilités d'application {#applicability}

::::::::: applicability
::: applicability-problem
Utilisez le pont dans les situations où vous souhaitez diviser et
organiser une classe monolithique composée de plusieurs variantes d'une
fonctionnalité (par exemple, si la classe fonctionne avec différents
serveurs de base de données).
:::

::: applicability-solution
Plus une classe grandit, plus il est difficile de comprendre son
fonctionnement et plus les modifications prennent du temps. Les
modifications apportées à l'une des variantes de la fonctionnalité vont
demander des changements dans toute la classe, ce qui risque de créer
des erreurs ou de provoquer des effets de bord critiques.

Le pont vous permet de diviser la classe monolithique en plusieurs
hiérarchies de classes. Ensuite, les classes d'une hiérarchie peuvent
être modifiées indépendamment des classes des autres hiérarchies. La
maintenance du code devient ainsi plus simple et minimise les risques de
bugs.
:::

::: applicability-problem
Utilisez le pont si vous voulez étendre une classe dans plusieurs
dimensions orthogonales (indépendantes).
:::

::: applicability-solution
Le pont vous propose de construire une hiérarchie de classes séparée
pour chaque dimension. La classe d'origine délègue la tâche aux objets
de ces hiérarchies plutôt que de tout faire par elle-même.
:::

::: applicability-problem
Utilisez ce patron si vous voulez être en mesure de changer
d'implémentation dès le lancement de l'application.
:::

::: applicability-solution
Grâce à ce patron, l'objet de l'implémentation peut être déplacé à
l'intérieur de l'abstraction. Cette manipulation n'est pas obligatoire,
mais elle est aussi simple à mettre en place que de donner une valeur à
un attribut.

*J'en profite pour vous informer que ce dernier point pousse souvent les
développeurs à confondre le pont et la [Stratégie](strategy.html).
Rappelez-vous bien qu'un patron n'est pas seulement une manière de
structurer vos classes, c'est aussi un moyen de communiquer votre
intention ou de répondre à un problème.*
:::
:::::::::
::::::::::

::: {.section .checklist}
##  Mise en œuvre {#checklist}

1.  Identifiez les dimensions orthogonales de vos classes. Ces concepts
    indépendants peuvent représenter les couples suivants :
    abstraction/plateforme, domaine/infrastructure, front-end/back-end,
    interface/implémentation.

2.  Déterminez les opérations que le client veut utiliser et
    définissez-les dans la classe d'abstraction de base.

3.  Établissez la liste des opérations disponibles sur toutes les
    plateformes. L'abstraction a besoin de certaines de ces opérations :
    déclarez-les dans l'interface de l'implémentation principale.

4.  Créez des classes d'implémentations concrètes pour chaque plateforme
    de votre domaine, et assurez-vous qu'elles implémentent l'interface
    de l'implémentation.

5.  Ajoutez un attribut de référence pour le type de l'implémentation à
    l'intérieur de la classe abstraction. Cette dernière délègue la
    majorité des tâches à l'objet implémentation référencé dans cet
    attribut.

6.  Si vous avez plusieurs variantes de logique de haut niveau, créez
    des abstractions fines pour chacune d'entre elles en étendant la
    classe de base abstraction.

7.  Le code client doit en principe passer un objet d'implémentation au
    constructeur de l'abstraction afin d'associer les deux. Ensuite, le
    client n'a plus besoin de s'occuper de l'implémentation et peut se
    contenter de travailler uniquement avec l'abstraction.
:::

:::::: {.section .pros-cons}
##  Avantages et inconvénients {#pros-cons}

::::: row
::: col-sm-6
-    Vous pouvez créer des classes et des applications multiplateformes.
-    Le code client manipule des abstractions de haut niveau. Il n'est
    pas dépendant des détails de la plateforme.
-    *Principe ouvert/fermé*. Vous pouvez introduire de nouvelles
    abstractions et implémentations indépendamment les unes des autres.
-    *Principe de responsabilité unique*. Vous pouvez vous concentrer
    sur la logique de haut niveau dans l'abstraction, et sur les détails
    de la plateforme dans l'implémentation.
:::

::: col-sm-6
-    Le code va devenir plus compliqué si vous introduisez ce patron
    dans une classe très cohésive.
:::
:::::
::::::

::: {.section .relations}
##  Liens avec les autres patrons {#relations}

-   Le [Pont](bridge.html) est habituellement mis en place durant la
    conception, ce qui vous permet de développer les différentes parties
    de l'application indépendamment. L'[adaptateur](adapter.html) quant
    à lui est plus souvent utilisé dans une application existante pour
    permettre à des classes normalement incompatibles de fonctionner
    ensemble.

-   Le [Pont](bridge.html), l'[État](state.html), la
    [Stratégie](strategy.html) (et dans une certaine mesure
    l'[Adaptateur](adapter.html)) ont des structures très similaires. En
    effet, ces patrons sont basés sur la composition, qui délègue les
    tâches aux autres objets. Cependant, ils résolvent différents
    problèmes. Un patron n'est pas juste une recette qui vous aide à
    structurer votre code d'une certaine manière. C'est aussi une façon
    de communiquer aux autres développeurs le problème qu'il résout.

-   Vous pouvez utiliser la [Fabrique abstraite](abstract-factory.html)
    avec le [Pont](bridge.html). Ce couple est très utile quand les
    abstractions définies par le *pont* ne fonctionnent qu'avec
    certaines implémentations spécifiques. Dans ce cas, la *fabrique
    abstraite* peut encapsuler ces relations et cacher la complexité au
    code client.

-   Vous pouvez combiner le [Monteur](builder.html) avec le
    [Pont](bridge.html) : la classe directeur joue le rôle de
    l'abstraction, et les différents *monteurs* prennent le rôle des
    implémentations.
:::

::: {.section .implementations}
##  Exemples de code {#implementations}

[![Pont en
C#](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](bridge/csharp/example.html "Pont en C#"){.prog-lang-link}
[![Pont en
C++](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](bridge/cpp/example.html "Pont en C++"){.prog-lang-link}
[![Pont en
Go](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](bridge/go/example.html "Pont en Go"){.prog-lang-link}
[![Pont en
Java](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](bridge/java/example.html "Pont en Java"){.prog-lang-link}
[![Pont en
PHP](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](bridge/php/example.html "Pont en PHP"){.prog-lang-link}
[![Pont en
Python](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](bridge/python/example.html "Pont en Python"){.prog-lang-link}
[![Pont en
Ruby](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](bridge/ruby/example.html "Pont en Ruby"){.prog-lang-link}
[![Pont en
Rust](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](bridge/rust/example.html "Pont en Rust"){.prog-lang-link}
[![Pont en
Swift](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](bridge/swift/example.html "Pont en Swift"){.prog-lang-link}
[![Pont en
TypeScript](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](bridge/typescript/example.html "Pont en TypeScript"){.prog-lang-link}
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

[Composite []{.fa .fa-arrow-right}](composite.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Retour

[[]{.fa .fa-arrow-left} Adaptateur](adapter.html){.btn .btn-default
rel="prev"}
:::
:::::::::::::::::::::::::::::::::

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
:::::::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::::::
