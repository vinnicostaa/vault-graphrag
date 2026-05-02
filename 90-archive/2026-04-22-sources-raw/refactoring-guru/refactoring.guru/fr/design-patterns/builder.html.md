:::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [Patrons de
conception](../design-patterns.html){.type} / [Patrons de
création](creational-patterns.html){.type}
:::

# Monteur {#monteur .title}

::: aka
Alias : [Builder]{style="display:inline-block"}
:::

::: {.section .intent}
##  Intention {#intent}

**Monteur** est un patron de conception de création qui permet de
construire des objets complexes étape par étape. Il permet de produire
différentes variations ou représentations d'un objet en utilisant le
même code de construction.

<figure class="image">
<img
src="../../images/patterns/content/builder/builder-fr01a1.png?id=01d042c5ce039c220f76ec70b8e12129"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/builder/builder-fr.png?id=01d042c5ce039c220f76ec70b8e12129 640w,/images/patterns/content/builder/builder-fr-1.5x.png?id=4c78518192b68a4ae4ec8435b3ebf18b 960w,/images/patterns/content/builder/builder-fr-2x.png?id=5646a84dea0dbd258eb1d9d650a8ead7 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Patron de conception monteur" />
</figure>
:::

::: {.section .problem}
##  Problème {#problem}

Imaginez un objet complexe qui nécessite une initialisation fastidieuse,
composée de plusieurs parties avec de nombreux champs et objets
imbriqués. Le code d'initialisation va se retrouver dans un
constructeur, enterré sous une pile monstrueuse de paramètres, ou encore
pire : réparti un peu partout dans le code client.

<figure class="image">
<img
src="../../images/patterns/diagrams/builder/problem16274.png?id=11e715c5c97811f848c48e0f399bb05e"
style="aspect-ratio: 1.71;"
srcset="/images/patterns/diagrams/builder/problem1.png?id=11e715c5c97811f848c48e0f399bb05e 600w,/images/patterns/diagrams/builder/problem1-1.5x.png?id=a46778018c4f0f4bbf2357940c1f5f40 900w,/images/patterns/diagrams/builder/problem1-2x.png?id=02ffbd7ad294600e42aa78989d441b4d 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Créer de nombreuses sous-classes entraîne un autre problème" />
<figcaption><p>Créer une sous-classe pour chaque configuration possible
d’un objet risque de rendre le programme trop complexe.</p></figcaption>
</figure>

Réfléchissons à la manière de modéliser un objet `Maison`. Pour
fabriquer une maison de base, vous devez construire quatre murs et un
sol, installer une porte, poser quelques fenêtres et bâtir un toit. Mais
comment procéder si vous voulez une plus grande maison avec plus de
lumière, un peu de terrain et autres commodités (un système de
chauffage, de la plomberie et des câbles électriques) ?

La solution la plus simple est d'étendre la classe de base `Maison` et
de créer un ensemble de sous-classes pour couvrir toutes les
combinaisons de paramètres. Mais au bout d'un certain temps, vous allez
vous retrouver avec un nombre considérable de sous-classes. Le moindre
paramètre supplémentaire comme le style du porche par exemple, va encore
plus développer la hiérarchie.

Voici une autre approche qui n'implique pas de générer des
sous-classes : vous pouvez créer un constructeur géant dans la classe de
base `Maison` avec tous les paramètres contrôlant l'objet maison. Cette
solution élimine le besoin de sous-classes, mais entraîne un autre
problème.

<figure class="image">
<img
src="../../images/patterns/diagrams/builder/problem27ab5.png?id=2e91039b6c7d2d2df6ee519983a3b036"
style="aspect-ratio: 1.71;"
srcset="/images/patterns/diagrams/builder/problem2.png?id=2e91039b6c7d2d2df6ee519983a3b036 600w,/images/patterns/diagrams/builder/problem2-1.5x.png?id=3d302cf2762fd04d70cbb91cb54e923c 900w,/images/patterns/diagrams/builder/problem2-2x.png?id=5e7975a91c0e4f4ba960f908cc9c2ea2 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Le constructeur télescopique" />
<figcaption><p>Un constructeur qui possède de nombreux paramètres a ses
inconvénients : ces derniers ne sont pas toujours
tous utilisés.</p></figcaption>
</figure>

Dans la majorité des cas, la plupart des paramètres resteront
inutilisés, rendant [l'appel au constructeur assez
hideux](../smells/long-parameter-list.html). Par exemple, le paramètre
recensant les piscines se révèle inutile neuf fois sur dix, car peu de
maisons en sont équipées.
:::

::: {.section .solution}
##  Solution

Le patron de conception monteur propose d'extraire le code du
constructeur d'objet de sa classe et de le déplacer dans des objets
distincts appelés *monteurs*.

<figure class="image">
<img
src="../../images/patterns/diagrams/builder/solution13e5f.png?id=8ce82137f8935998de802cae59e00e11"
style="aspect-ratio: 1.46;"
srcset="/images/patterns/diagrams/builder/solution1.png?id=8ce82137f8935998de802cae59e00e11 410w,/images/patterns/diagrams/builder/solution1-1.5x.png?id=8ab77eb73760a75c35940bae243d43f2 615w,/images/patterns/diagrams/builder/solution1-2x.png?id=a9c2ab02f0b2aca1a7512022194dd113 820w"
sizes="(max-width: 720px) 100vw, 410px" loading="lazy" width="410"
alt="Mise en place du patron de conception monteur" />
<figcaption><p>Le patron de conception monteur permet de construire des
objets complexes étape par étape. Le monteur empêche les autres objets
d’accéder au produit pendant sa construction.</p></figcaption>
</figure>

Il organise la construction de l'objet à l'aide d'une série d'étapes
(`construireMurs`, `construirePorte`, etc.). Pour créer un objet, vous
allez effectuer une séquence d'étapes dans un objet monteur. Le gros
avantage, c'est que vous n'avez pas besoin d'appeler toutes les étapes,
mais seulement celles nécessaires à la création de la configuration
particulière d'un objet.

Certaines étapes de la construction peuvent demander des implémentations
variables en fonction des différentes représentations du produit. Par
exemple, les murs d'une cabane peuvent être en bois, mais ceux d'un
château seront en pierre.

Dans ce cas, vous pouvez créer plusieurs monteurs qui implémentent le
même ensemble d'étapes de construction, mais d'une manière différente.
Vous pouvez ensuite utiliser ces monteurs dans le processus de
construction (c'est-à-dire une succession d'appels ordonnés des étapes)
pour créer différents types d'objets.

<figure class="image">
<img
src="../../images/patterns/content/builder/builder-comic-1-fr482a.png?id=4fe3e43a0b0d9090b837cc5ff836f8fc"
style="aspect-ratio: 2;"
srcset="/images/patterns/content/builder/builder-comic-1-fr.png?id=4fe3e43a0b0d9090b837cc5ff836f8fc 600w,/images/patterns/content/builder/builder-comic-1-fr-1.5x.png?id=d4a3a589c912b524961a03320f284ad8 900w,/images/patterns/content/builder/builder-comic-1-fr-2x.png?id=ece5733a95b6c1ff4cb86e32822df4a2 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600" />
<figcaption><p>Ces monteurs exécutent la même tâche, mais de
manière différente.</p></figcaption>
</figure>

Prenons un autre exemple : un premier monteur qui fabrique tout à partir
de bois et de verre, un deuxième qui utilise de la pierre et du fer et
un troisième qui se sert d'or et de diamants. En appelant les mêmes
étapes, vous pouvez construire une maison avec le premier, un petit
château avec le deuxième et un palais grâce au troisième. Mais tout ceci
ne peut fonctionner que si le code client qui appelle les étapes de la
construction peut interagir avec les monteurs via une interface commune.

#### Directeur (Director)

Vous pouvez aller encore plus loin en prenant tous les appels aux étapes
utilisées pour construire un produit, et en les mettant dans une classe
séparée que l'on nomme *directeur*. La classe directeur va définir
l'ordre d'exécution des différentes étapes et le monteur fournit les
implémentations de ces étapes.

<figure class="image">
<img
src="../../images/patterns/content/builder/builder-comic-2-frc81a.png?id=4f97c229d8d635e0b46e1f65e4760509"
style="aspect-ratio: 1.14;"
srcset="/images/patterns/content/builder/builder-comic-2-fr.png?id=4f97c229d8d635e0b46e1f65e4760509 343w,/images/patterns/content/builder/builder-comic-2-fr-1.5x.png?id=5c8b081701be35f10ed89818e4968ac1 515w,/images/patterns/content/builder/builder-comic-2-fr-2x.png?id=b537bd0f2e231b13c0d6ea0661700db9 687w"
sizes="(max-width: 720px) 100vw, 343px" loading="lazy" width="343" />
<figcaption><p>Le directeur connait les étapes à suivre pour construire
un produit fonctionnel.</p></figcaption>
</figure>

La classe directeur n'est pas obligatoire. Vous pouvez toujours appeler
les étapes de construction dans un ordre spécifique depuis le code
client. Cependant, la classe directeur se révèle idéale pour y placer
les routines de construction et pouvoir les réutiliser ensuite dans
votre programme.

De plus, le directeur cache au client les détails de la construction du
produit. Le client doit juste associer un monteur avec un directeur,
lancer la construction via le directeur, puis récupérer le résultat
auprès du monteur.
:::

::::: {.section .structure-container}
##  Structure

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/builder/structure05e4.png?id=fe9e23559923ea0657aa5fe75efef333"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 0.79;"
srcset="/images/patterns/diagrams/builder/structure.png?id=fe9e23559923ea0657aa5fe75efef333 460w,/images/patterns/diagrams/builder/structure-1.5x.png?id=91ea8cd3137b403542c23abf63938f00 690w,/images/patterns/diagrams/builder/structure-2x.png?id=dca1b1508e23c266cbedc80ffb84311a 920w"
sizes="(max-width: 720px) 100vw, 460px" loading="lazy" width="460"
alt="Structure du patron de conception monteur" /><img
src="../../images/patterns/diagrams/builder/structure-indexedadac.png?id=44b3d763ce91dbada5d8394ef777437f"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 0.81;"
srcset="/images/patterns/diagrams/builder/structure-indexed.png?id=44b3d763ce91dbada5d8394ef777437f 470w,/images/patterns/diagrams/builder/structure-indexed-1.5x.png?id=d57a6ff9342ea31736ea98e5066e0748 705w,/images/patterns/diagrams/builder/structure-indexed-2x.png?id=153eed9a51784cbe00d0ca8b3d6b268d 940w"
sizes="(max-width: 720px) 100vw, 470px" loading="lazy" width="470"
alt="Structure du patron de conception monteur" />
</figure>
:::

1.  L'interface du **Monteur** déclare les étapes communes de la
    construction du produit entre tous les monteurs.

2.  Les **Monteurs Concrets** fournissent différentes implémentations
    des étapes de la construction. Ils peuvent créer des produits qui ne
    reprennent pas l'interface commune.

3.  Les **Produits** sont les résultats retournés. Les produits
    construits par les différents monteurs ne sont pas obligés
    d'appartenir à la même hiérarchie de classes ni d'avoir la même
    interface.

4.  Le **Directeur** indique l'ordonnancement des étapes de construction
    et offre la possibilité de créer et de réutiliser des configurations
    spécifiques pour les produits.

5.  Le **Client** doit associer l'un des monteurs au directeur. En
    général cette association n'est réalisée qu'une seule fois, grâce
    aux paramètres du constructeur du directeur. Pour toute construction
    ultérieure, le directeur utilise l'objet monteur. En guise
    d'alternative, le client peut passer l'objet monteur à la méthode de
    production du directeur. Dans ce cas, vous pouvez utiliser un
    monteur différent chaque fois que vous lancez une production avec le
    directeur.
::::
:::::

::: {.section .pseudocode}
##  Pseudo-code {#pseudocode}

Voici un exemple qui montre comment un **Monteur** peut réutiliser le
même code de construction d'objet pour assembler différents types de
produits comme des voitures, et créer leurs manuels respectifs.

<figure class="image">
<img
src="../../images/patterns/diagrams/builder/example-fr8111.png?id=637ab49290b69bd0847dcc615f97fd02"
style="aspect-ratio: 0.85;"
srcset="/images/patterns/diagrams/builder/example-fr.png?id=637ab49290b69bd0847dcc615f97fd02 500w,/images/patterns/diagrams/builder/example-fr-1.5x.png?id=a862d51820f4f1ed22e10879cc22a2c9 750w,/images/patterns/diagrams/builder/example-fr-2x.png?id=f10d6c4875e178591a4479a3ca611905 1000w"
sizes="(max-width: 720px) 100vw, 500px" loading="lazy" width="500"
alt="Le diagramme de classe de l’exemple utilisé pour le monteur" />
<figcaption><p>Un exemple de construction de voitures étape par étape et
le manuel d’utilisation qui correspond à leur modèle.</p></figcaption>
</figure>

Une voiture est un objet complexe. Elle peut être fabriquée de cent
manières différentes. Plutôt que d'encombrer la classe `Voiture` avec un
énorme constructeur, nous avons extrait le code dans une classe monteur
séparée pour la voiture. Cette classe est composée d'un ensemble de
méthodes pour configurer les différentes parties d'une voiture.

Si le code client veut assembler un modèle spécial de voiture
bénéficiant d'un réglage de précision, il peut s'adresser directement au
monteur. Il peut également déléguer l'assemblage à la classe directeur
qui connait le processus de fabrication à indiquer au monteur pour les
modèles de voitures les plus populaires.

Ce qui suit va peut-être vous choquer, mais chaque voiture doit posséder
son propre manuel (franchement, qui les lit ?). Le manuel décrit toutes
les fonctionnalités de la voiture et les détails vont varier selon les
modèles. Il semble donc approprié de réutiliser un processus de
construction existant pour les voitures et leurs manuels. Bien entendu,
la création d'un manuel et d'une voiture sont deux procédés complètement
différents et nous devons concevoir des monteurs spécialisés pour les
manuels. Cette classe va implémenter les mêmes méthodes de construction
que sa cousine assembleuse de voitures, mais au lieu de fabriquer des
pièces de voitures, elle se contente de les décrire. Nous pouvons
construire une voiture ou un manuel en passant ces monteurs au même
objet directeur.

L'étape finale consiste à récupérer l'objet qui en résulte. Même si une
voiture en métal et un manuel en papier sont directement liés, ce sont
deux choses complètement différentes. Nous ne pouvons pas insérer une
méthode pour récupérer les résultats dans le directeur sans le coupler
aux classes concrètes du produit. Par conséquent, c'est le monteur qui
effectue le travail qui nous donne ensuite le résultat.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// L’utilisation du patron de conception monteur n’est
// conseillée que si vos produits sont complexes et nécessitent
// une configuration étendue. Bien qu’ils n’aient pas la même
// interface, les deux produits suivants sont liés.
class Car is
    // Une voiture est équipée d’un GPS, d’un ordinateur de bord
    // et d&#39;un certain nombre de sièges. Les différents modèles
    // de voitures (sport, SUV, cabriolet) ont différentes
    // fonctionnalités installées ou activées.

class Manual is
    // Chaque voiture doit avoir un manuel d’utilisation qui
    // correspond à sa configuration et décrit toutes ses
    // fonctionnalités.


// L’interface du monteur contient des méthodes spécialisées
// pour créer les différentes parties des objets du produit.
interface Builder is
    method reset()
    method setSeats(...)
    method setEngine(...)
    method setTripComputer(...)
    method setGPS(...)

// Les classes concrètes du monteur suivent l’interface monteur
// et procurent des implémentations spécifiques pour les étapes
// de la fabrication. Votre programme peut contenir plusieurs
// variantes de monteurs, chacune avec sa propre implémentation.
class CarBuilder implements Builder is
    private field car:Car

    // Une instance monteur fraichement créée doit contenir un
    // objet produit vide qu’elle va ensuite assembler.
    constructor CarBuilder() is
        this.reset()

    // La méthode reset nettoie l’objet qui est construit.
    method reset() is
        this.car = new Car()

    // Toutes les étapes de la fabrication manipulent la même
    // instance du produit.
    method setSeats(...) is
        // Configure le nombre de sièges dans la voiture.

    method setEngine(...) is
        // Installe un moteur donné.

    method setTripComputer(...) is
        // Installe un ordinateur de bord.

    method setGPS(...) is
        // Installe un système de géolocalisation.

    // Les monteurs concrets sont censés fournir leurs propres
    // méthodes pour récupérer les résultats. Ce fonctionnement
    // permet aux différents types de monteurs de créer des
    // produits entièrement différents qui ne suivent pas la
    // même interface. Par conséquent, les méthodes ne peuvent
    // pas être déclarées dans l’interface du monteur (en tout
    // cas pas dans un langage de programmation doté d’un
    // système de typage statique).
    //
    // En général, après avoir retourné le résultat final au
    // client, une instance de monteur doit être prête à lancer
    // la fabrication d’un autre produit. C’est pour cette
    // raison que l’on appelle généralement la méthode reset à
    // la fin du corps de la méthode `getProduct`. Mais ce
    // comportement n’est pas obligatoire et votre monteur peut
    // attendre que le code client lance un appel explicite à un
    // reset pour vous débarrasser du résultat précédent.
    method getProduct():Car is
        product = this.car
        this.reset()
        return product

// Contrairement aux autres patrons de création, le monteur vous
// permet de fabriquer des produits qui ne suivent pas la même
// interface.
class CarManualBuilder implements Builder is
    private field manual:Manual

    constructor CarManualBuilder() is
        this.reset()

    method reset() is
        this.manual = new Manual()

    method setSeats(...) is
        // Fonctionnalités des sièges dans le manuel.

    method setEngine(...) is
        // Ajoute les informations concernant le moteur.

    method setTripComputer(...) is
        // Ajoute les instructions concernant l’ordinateur de
        // bord.

    method setGPS(...) is
        // Ajoute les instructions concernant le GPS.

    method getProduct():Manual is
        // Retourne le manuel et remet à zéro le monteur
        // (reset).


// Le directeur a pour seule responsabilité l’ordonnancement des
// étapes de la fabrication. Il se révèle utile lorsque vous
// fabriquez des produits avec un ordre précis ou dans une
// configuration particulière. À proprement parler, la classe
// Directeur n’est pas obligatoire, car le client peut manipuler
// les monteurs directement.
class Director is
    // Le directeur manipule n’importe quelle instance de
    // monteur que le code client lui envoie. Ainsi, le code
    // client peut modifier le type final du produit qui vient
    // d’être assemblé. Le directeur peut fabriquer plusieurs
    // variantes de produits en utilisant les mêmes étapes de
    // fabrication.
    method constructSportsCar(builder: Builder) is
        builder.reset()
        builder.setSeats(2)
        builder.setEngine(new SportEngine())
        builder.setTripComputer(true)
        builder.setGPS(true)

    method constructSUV(builder: Builder) is
        // ...


// Le code client crée un objet monteur, le passe au directeur
// et lance le processus de fabrication. Le résultat final est
// récupéré dans l’objet monteur.
class Application is

    method makeCar() is
        director = new Director()

        CarBuilder builder = new CarBuilder()
        director.constructSportsCar(builder)
        Car car = builder.getProduct()

        CarManualBuilder builder = new CarManualBuilder()
        director.constructSportsCar(builder)

        // Le produit final est souvent récupéré depuis un objet
        // monteur, car le directeur est indépendant des
        // monteurs et des produits concrets et il ne les voit
        // donc pas.
        Manual manual = builder.getProduct()</code></pre>
</figure>
:::

:::::::::: {.section .applicability-container}
##  Possibilités d'application {#applicability}

::::::::: applicability
::: applicability-problem
Utilisez le patron de conception monteur afin de vous débarrasser d'un
« constructeur télescopique ».
:::

::: applicability-solution
Prenons un constructeur avec dix paramètres facultatifs. Faire un appel
à cette monstruosité n'est pas très pratique : vous surchargez le
constructeur avec plusieurs petites versions, mais avec moins de
paramètres. Ces constructeurs font toujours référence au constructeur
principal en donnant des valeurs par défaut aux paramètres optionnels.

<figure class="code">
<pre class="code" lang="java"><code>class Pizza {
    Pizza(int size) { ... }
    Pizza(int size, boolean cheese) { ... }
    Pizza(int size, boolean cheese, boolean pepperoni) { ... }
    // ...</code></pre>
<figcaption><p>Cette monstruosité ne peut être créée que dans certains
langages qui permettent la surcharge tels que le C# ou
le Java.</p></figcaption>
</figure>

Le monteur vous permet de créer des objets étape par étape, en utilisant
uniquement celles qui sont nécessaires. Après avoir implémenté le
patron, vous n'avez plus besoin d'entasser les paramètres dans vos
constructeurs.
:::

::: applicability-problem
Utilisez le monteur pour rendre votre code capable de créer différentes
représentations de produits (par exemple des maisons en pierre et en
bois).
:::

::: applicability-solution
Le monteur est utile lorsque les étapes de la construction des
différentes représentations du produit se ressemblent (seuls quelques
détails diffèrent).

L'interface de base du monteur définit toutes les étapes possibles de la
construction. Les monteurs concrets implémentent les étapes pour
construire les représentations particulières du produit. Le directeur
quant à lui s'occupe de gérer l'ordonnancement des différentes étapes de
construction.
:::

::: applicability-problem
Utilisez le monteur afin de construire une arborescence
[composite](composite.html) ou d'autres objets complexes.
:::

::: applicability-solution
Le monteur vous permet de construire des produits étape par étape. Vous
pouvez déléguer l'exécution de certaines étapes sans endommager le
produit final. Vous pouvez même appeler les étapes de manière récursive,
ce qui est très pratique dans le cas de la construction d'un objet
composite.

Le monteur ne met pas à disposition le produit tant que les étapes de la
construction ne sont pas terminées. Par conséquent, le code client ne
récupèrera jamais un résultat incomplet.
:::
:::::::::
::::::::::

::: {.section .checklist}
##  Mise en œuvre {#checklist}

1.  Assurez-vous de définir clairement les étapes communes de la
    construction pour toutes les représentations de produits
    disponibles, sinon vous ne pourrez pas mettre en place le patron.

2.  Déclarez ces étapes dans l'interface du monteur.

3.  Pour chaque représentation du produit, créez une classe concrète
    monteur puis implémentez ses étapes de construction.

    N'oubliez pas de mettre en place une méthode pour récupérer le
    résultat de la construction. Cette méthode ne peut pas être déclarée
    dans l'interface du monteur, car les différents monteurs peuvent
    fabriquer des produits qui n'ont pas forcément une interface
    commune. Nous ne pouvons pas connaitre à l'avance le type de retour
    d'une telle méthode. En revanche, si vos produits appartiennent à
    une hiérarchie unique, la méthode qui récupère le résultat peut être
    ajoutée à l'interface de base sans problème.

4.  Réfléchissez à la création d'une classe directeur. Elle peut
    encapsuler différents processus de construction d'un produit en
    utilisant le même objet monteur.

5.  Le code client crée les objets monteur et directeur. Le client doit
    passer un objet monteur au directeur avant le début de la
    construction. En général, il ne le fait qu'une seule fois, via les
    paramètres du constructeur du directeur. Le directeur utilise
    l'objet monteur pour toute construction ultérieure. Nous pourrions
    également passer le monteur directement à la méthode de construction
    du directeur.

6.  Le résultat de la construction peut être obtenu directement à partir
    du directeur, si tous les produits sont branchés sur la même
    interface. Sinon, le client doit aller directement chercher le
    résultat auprès du monteur.
:::

:::::: {.section .pros-cons}
##  Avantages et inconvénients {#pros-cons}

::::: row
::: col-sm-6
-    Vous pouvez construire les objets étape par étape et les déléguer
    ou les exécuter récursivement.
-    Vous pouvez réutiliser le même code de construction lorsque vous
    construisez différentes représentations des produits.
-    *Principe de responsabilité unique*. Vous pouvez découpler le code
    complexe de la construction et la logique métier du produit.
:::

::: col-sm-6
-    Le monteur nécessite de créer beaucoup nouvelles classes, ce qui
    accroit la complexité générale du code.
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

-   Le [Monteur](builder.html) se concentre sur la construction d'objets
    complexes étape par étape. La [Fabrique
    abstraite](abstract-factory.html) se spécialise dans la création de
    familles d'objets associés. La *fabrique abstraite* retourne le
    produit immédiatement, alors que le *monteur* vous permet de lancer
    des étapes supplémentaires avant de récupérer le produit.

-   Vous pouvez utiliser le [Monteur](builder.html) lorsque vous créez
    des arbres [Composites](composite.html) complexes, car vous pouvez
    programmer les étapes de la construction récursivement.

-   Vous pouvez combiner le [Monteur](builder.html) avec le
    [Pont](bridge.html) : la classe directeur joue le rôle de
    l'abstraction, et les différents *monteurs* prennent le rôle des
    implémentations.

-   Les [Fabriques abstraites](abstract-factory.html),
    [Monteurs](builder.html) et [Prototypes](prototype.html) peuvent
    tous être implémentés comme des [Singletons](singleton.html).
:::

::: {.section .implementations}
##  Exemples de code {#implementations}

[![Monteur en
C#](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](builder/csharp/example.html "Monteur en C#"){.prog-lang-link}
[![Monteur en
C++](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](builder/cpp/example.html "Monteur en C++"){.prog-lang-link}
[![Monteur en
Go](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](builder/go/example.html "Monteur en Go"){.prog-lang-link}
[![Monteur en
Java](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](builder/java/example.html "Monteur en Java"){.prog-lang-link}
[![Monteur en
PHP](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](builder/php/example.html "Monteur en PHP"){.prog-lang-link}
[![Monteur en
Python](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](builder/python/example.html "Monteur en Python"){.prog-lang-link}
[![Monteur en
Ruby](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](builder/ruby/example.html "Monteur en Ruby"){.prog-lang-link}
[![Monteur en
Rust](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](builder/rust/example.html "Monteur en Rust"){.prog-lang-link}
[![Monteur en
Swift](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](builder/swift/example.html "Monteur en Swift"){.prog-lang-link}
[![Monteur en
TypeScript](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](builder/typescript/example.html "Monteur en TypeScript"){.prog-lang-link}
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

[Prototype []{.fa .fa-arrow-right}](prototype.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Retour

[[]{.fa .fa-arrow-left} Comparaison des
fabriques](factory-comparison.html){.btn .btn-default rel="prev"}
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
