::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [Patrons de
conception](../design-patterns.html){.type} / [Patrons de
création](creational-patterns.html){.type}
:::

# Fabrique {#fabrique .title}

::: aka
Alias : [Constructeur virtuel, ]{style="display:inline-block"}[Factory
Method]{style="display:inline-block"}
:::

::: {.section .intent}
##  Intention {#intent}

**Fabrique** est un patron de conception de création qui définit une
interface pour créer des objets dans une classe mère, mais délègue le
choix des types d'objets à créer aux sous-classes.

<figure class="image">
<img
src="../../images/patterns/content/factory-method/factory-method-fr6091.png?id=19c57367eb8e520539a46f91a0751fb7"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/factory-method/factory-method-fr.png?id=19c57367eb8e520539a46f91a0751fb7 640w,/images/patterns/content/factory-method/factory-method-fr-1.5x.png?id=9461b8fcd7c79304e670e324570b243a 960w,/images/patterns/content/factory-method/factory-method-fr-2x.png?id=915435fc15ea2e64c0b95ce0da1957ef 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Patron de conception fabrique" />
</figure>
:::

::: {.section .problem}
##  Problème {#problem}

Imaginez que vous êtes en train de créer une application de gestion
logistique. La première version de votre application ne propose que le
transport par camion, la majeure partie de votre code est donc située
dans la classe `Camion`.

Au bout d'un certain temps, votre application devient populaire et de
nombreuses entreprises de transport maritime vous demandent tous les
jours d'ajouter la gestion de la logistique maritime dans l'application.

<figure class="image">
<img
src="../../images/patterns/diagrams/factory-method/problem1-fr478c.png?id=b075bd1d3cd23d86bb4aaf39d93957c8"
style="aspect-ratio: 2.4;"
srcset="/images/patterns/diagrams/factory-method/problem1-fr.png?id=b075bd1d3cd23d86bb4aaf39d93957c8 600w,/images/patterns/diagrams/factory-method/problem1-fr-1.5x.png?id=03a6bf31e0d286244335c47d4f0c5228 900w,/images/patterns/diagrams/factory-method/problem1-fr-2x.png?id=0d364f62859567bc3b2251e590ac7306 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Ajouter un nouveau moyen de transport au programme peut créer des problèmes" />
<figcaption><p>L’ajout d’une nouvelle classe au programme ne s’avère pas
si simple que cela si le reste du code est déjà couplé aux
classes existantes.</p></figcaption>
</figure>

C'est super, n'est-ce pas ? Mais qu'en est-il du code ? La majeure
partie est actuellement couplée à la classe `Camion`. Pour pouvoir
ajouter des `Bateaux` dans l'application, il faudrait revoir la base du
code. De plus, si vous décidez plus tard d'ajouter un autre type de
transport dans l'application, il faudra effectuer à nouveau ces
changements.

Par conséquent, vous allez vous retrouver avec du code pas très propre,
rempli de conditions qui modifient le comportement du programme en
fonction de la classe des objets de transport.
:::

::: {.section .solution}
##  Solution

Le patron de conception fabrique vous propose de remplacer les appels
directs au constructeur de l'objet (à l'aide de l'opérateur `new`) en
appelant une méthode *fabrique* spéciale. Pas d'inquiétude, les objets
sont toujours créés avec l'opérateur `new`, mais l'appel se fait à
l'intérieur de la méthode fabrique. Les objets qu'elle retourne sont
souvent appelés *produits*.

<figure class="image">
<img
src="../../images/patterns/diagrams/factory-method/solution16ec2.png?id=fc756d2af296b5b4d482e548214d08ef"
style="aspect-ratio: 2.3;"
srcset="/images/patterns/diagrams/factory-method/solution1.png?id=fc756d2af296b5b4d482e548214d08ef 620w,/images/patterns/diagrams/factory-method/solution1-1.5x.png?id=22d3b6bb74e966d1cb3a4d8f380cefa3 930w,/images/patterns/diagrams/factory-method/solution1-2x.png?id=c482b3cd7a4d8dd73b4c8c12dfcaa03c 1240w"
sizes="(max-width: 720px) 100vw, 620px" loading="lazy" width="620"
alt="La structure des classes de création" />
<figcaption><p>Les sous-classes peuvent modifier les classes des objets
retournés par la méthode fabrique.</p></figcaption>
</figure>

À première vue, cette modification peut sembler inutile : nous avons
juste déplacé l'appel du constructeur dans une autre partie du
programme. Mais maintenant, vous pouvez redéfinir la méthode fabrique
dans la sous-classe et changer la classe des produits créés par la
méthode.

Il y a tout de même une petite limitation : les sous-classes peuvent
retourner des produits différents seulement si les produits ont une
classe de base ou une interface commune. De plus, cette interface doit
être le type retourné par la méthode fabrique de la classe de base.

<figure class="image">
<img
src="../../images/patterns/diagrams/factory-method/solution2-frc7ce.png?id=c9c8ac45a8ac60329bf10fb3085cda0c"
style="aspect-ratio: 1.96;"
srcset="/images/patterns/diagrams/factory-method/solution2-fr.png?id=c9c8ac45a8ac60329bf10fb3085cda0c 490w,/images/patterns/diagrams/factory-method/solution2-fr-1.5x.png?id=d048c4171f7416514b7a2a600baaba7e 735w,/images/patterns/diagrams/factory-method/solution2-fr-2x.png?id=c4d3d8d568d06abac32bed10878000cb 980w"
sizes="(max-width: 720px) 100vw, 490px" loading="lazy" width="490"
alt="Structure de la hiérarchie des produits" />
<figcaption><p>Les produits doivent tous implémenter la
même interface.</p></figcaption>
</figure>

Par exemple, les classes `Camion` et `Bateau` doivent toutes les deux
implémenter l'interface `Transport`, qui déclare une méthode `livrer`.
Chaque classe implémente cette méthode à sa façon : les camions livrent
par la route et les bateaux livrent par la mer. La méthode fabrique de
la classe `LogistiqueRoute` retourne des camions, alors que celle de la
classe `LogistiqueMer` retourne des bateaux.

<figure class="image">
<img
src="../../images/patterns/diagrams/factory-method/solution3-fr5d6a.png?id=e542399869b6eb7364e9a13467d408c2"
style="aspect-ratio: 1.83;"
srcset="/images/patterns/diagrams/factory-method/solution3-fr.png?id=e542399869b6eb7364e9a13467d408c2 640w,/images/patterns/diagrams/factory-method/solution3-fr-1.5x.png?id=38c64d29bfebaf61db76a8a3adfe0d7e 960w,/images/patterns/diagrams/factory-method/solution3-fr-2x.png?id=8790ef6372b00da0007242c0d7a2baec 1280w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="640"
alt="Structure du code après application du patron de conception fabrique" />
<figcaption><p>Tant que les classes produit implémentent une interface
commune, vous pouvez passer leurs objets au code client sans tout
faire planter.</p></figcaption>
</figure>

Le code qui appelle la méthode fabrique (souvent appelé le code
*client*) ne fait pas la distinction entre les différents produits
concrets retournés par les sous-classes, il les considère tous comme des
`Transports` abstraits. Le client sait que tous les objets transportés
sont censés avoir une méthode `livrer`, mais son fonctionnement lui
importe peu.
:::

::::: {.section .structure-container}
##  Structure

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/factory-method/structure78c3.png?id=4cba0803f42517cfe8548c9bc7dc4c9b"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.74;"
srcset="/images/patterns/diagrams/factory-method/structure.png?id=4cba0803f42517cfe8548c9bc7dc4c9b 660w,/images/patterns/diagrams/factory-method/structure-1.5x.png?id=ece8300890afc14494770a6b6d370428 990w,/images/patterns/diagrams/factory-method/structure-2x.png?id=9ea3aa8a47f8be22e12e523c15b448fd 1320w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="660"
alt="La structure du patron de conception fabrique" /><img
src="../../images/patterns/diagrams/factory-method/structure-indexed6136.png?id=4c603207859ca1f939b17b60a3a2e9e0"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.74;"
srcset="/images/patterns/diagrams/factory-method/structure-indexed.png?id=4c603207859ca1f939b17b60a3a2e9e0 660w,/images/patterns/diagrams/factory-method/structure-indexed-1.5x.png?id=626b6d7f577e1c265cce244678b2cf7f 990w,/images/patterns/diagrams/factory-method/structure-indexed-2x.png?id=c794e4f2d05013fb176464a1d1a5d7ab 1320w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="660"
alt="La structure du patron de conception fabrique" />
</figure>
:::

1.  L'interface est déclarée par le **Produit** et est commune à tous
    les objets qui peuvent être conçus par le créateur et ses
    sous-classes.

2.  Les **Produits Concrets** sont différentes implémentations de
    l'interface produit.

3.  La méthode fabrique est déclarée par la classe **Créateur** et
    retourne les nouveaux produits. Il est important que son type de
    retour concorde avec l'interface produit.

    Vous pouvez rendre la méthode fabrique abstraite afin d'obliger ses
    sous-classes à implémenter leur propre version de la méthode ou vous
    pouvez modifier la méthode fabrique de la classe de base afin
    qu'elle retourne un type de produit par défaut.

    Il faut bien comprendre que malgré son nom, la création de produits
    n'est **pas** la responsabilité principale du créateur. La classe
    créateur a en général déjà un fonctionnement propre lié à la nature
    de ses produits. La fabrique aide à découpler cette logique des
    produits concrets. C'est un peu comme une grande entreprise de
    développement de logiciels : elle peut posséder un département
    spécialisé dans la formation des développeurs, mais son activité
    principale reste d'écrire du code, pas de produire des développeurs.

4.  Les **Créateurs Concrets** redéfinissent la méthode fabrique de la
    classe de base afin de pouvoir retourner les différents types de
    produits.

    Notez toutefois que la méthode fabrique n'est pas obligée de
    **créer** tout le temps de nouvelles instances. Elle peut retourner
    des objets depuis un cache, un réservoir d'objets ou une autre
    source.
::::
:::::

::: {.section .pseudocode}
##  Pseudo-code {#pseudocode}

Cet exemple montre comment la **fabrique** peut être utilisée pour créer
des éléments d'une UI (interface utilisateur) multiplateforme sans
coupler le code client aux classes concrètes de l'UI.

<figure class="image">
<img
src="../../images/patterns/diagrams/factory-method/examplec59f.png?id=67db9a5cb817913444efcb1c067c9835"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/diagrams/factory-method/example.png?id=67db9a5cb817913444efcb1c067c9835 640w,/images/patterns/diagrams/factory-method/example-1.5x.png?id=921d97276e5e2fd8e64609c9cfa021e7 960w,/images/patterns/diagrams/factory-method/example-2x.png?id=a2470830778e318263155000dbdc5870 1280w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="640"
alt="Structure de l’exemple du patron de conception fabrique" />
<figcaption><p>Exemple multiplateforme pour une boîte
de dialogue.</p></figcaption>
</figure>

La classe de base dialogue utilise différents éléments d'UI pour le
rendu de ses fenêtres. Ces éléments peuvent un peu varier en fonction du
système d'exploitation, mais ils doivent garder le même comportement. Un
bouton sous Windows est aussi un bouton sous Linux.

Lorsque la fabrique entre dans l'équation, il est inutile de réécrire la
logique de la boîte de dialogue pour chaque système d'exploitation. Si
nous déclarons une méthode fabrique qui crée des boutons à l'intérieur
de la classe de base dialogue, nous pourrons par la suite sous-classer
cette dernière afin que la méthode fabrique retourne des boutons dotés
du style Windows. La sous-classe hérite alors du code de la classe de
base dialogue, mais grâce à la fabrique, elle est capable d'afficher à
l'écran des boutons à l'allure Windows.

Pour le bon fonctionnement de ce patron, la classe de base dialogue doit
utiliser des boutons abstraits représentés par une classe de base ou une
interface dont tous les boutons concrets hériteront. Ainsi, quel que
soit le type de boutons, le code de la classe dialogue reste
fonctionnel.

Bien sûr, cette approche fonctionne également avec les autres éléments
de l'UI. Cependant, chaque nouvelle méthode fabrique ajoutée à Dialogue
nous rapproche du patron de conception [Fabrique
abstraite](abstract-factory.html). Pas de panique, nous aborderons ce
patron plus tard !

<figure class="code">
<pre class="code" lang="pseudocode"><code>// La classe créateur déclare la méthode fabrique qui doit
// renvoyer un objet de la classe produit. Les sous-classes du
// créateur fournissent en général une implémentation de cette
// méthode.
class Dialog is
    // Le créateur peut également fournir des implémentations
    // par défaut de la méthode fabrique.
    abstract method createButton():Button

    // Ne vous laissez pas berner par son nom, la responsabilité
    // principale du créateur n’est pas de fabriquer des
    // produits. Il héberge en général de la logique métier qui
    // concerne les produits retournés par la méthode fabrique.
    // Les sous-classes peuvent modifier indirectement cette
    // logique métier en redéfinissant la méthode fabrique et en
    // lui faisant retourner un type de produit différent.
    method render() is
        // Appelle la méthode fabrique pour créer un objet
        // Produit.
        Button okButton = createButton()
        // Utilise le produit.
        okButton.onClick(closeDialog)
        okButton.render()


// Les créateurs concrets redéfinissent la méthode fabrique pour
// changer le type du produit qui en résulte.
class WindowsDialog extends Dialog is
    method createButton():Button is
        return new WindowsButton()

class WebDialog extends Dialog is
    method createButton():Button is
        return new HTMLButton()


// L’interface du produit déclare les traitements que tous les
// produits concrets doivent implémenter.
interface Button is
    method render()
    method onClick(f)

// Les produits concrets fournissent diverses implémentations de
// l’interface du produit.
class WindowsButton implements Button is
    method render(a, b) is
        // Affiche un bouton avec le style Windows.
    method onClick(f) is
        // Attribue un événement sur un clic natif dans un
        // système d’exploitation.

class HTMLButton implements Button is
    method render(a, b) is
        // Retourne une représentation HTML d’un bouton.
    method onClick(f) is
        // Attribue un événement sur un clic dans un navigateur
        // Internet.


class Application is
    field dialog: Dialog

    // L’application choisit un type de créateur en fonction de
    // la configuration actuelle ou des paramètres
    // d’environnement.
    method initialize() is
        config = readApplicationConfigFile()

        if (config.OS == &quot;Windows&quot;) then
            dialog = new WindowsDialog()
        else if (config.OS == &quot;Web&quot;) then
            dialog = new WebDialog()
        else
            throw new Exception(&quot;Error! Unknown operating system.&quot;)

    // Le code client manipule une instance d’un créateur
    // concret, mais uniquement au moyen de son interface de
    // base. Tant que le client continue de passer par cette
    // interface pour manipuler le créateur, vous pouvez passer
    // n’importe quelle sous-classe du créateur.
    method main() is
        this.initialize()
        dialog.render()</code></pre>
</figure>
:::

:::::::::: {.section .applicability-container}
##  Possibilités d'application {#applicability}

::::::::: applicability
::: applicability-problem
Utilisez la fabrique si vous ne connaissez pas à l'avance les types et
dépendances précis des objets que vous allez utiliser dans votre code.
:::

::: applicability-solution
La fabrique effectue une séparation entre le code du constructeur et le
code qui utilise réellement le produit. Le code du constructeur devient
ainsi plus évolutif et indépendant du reste du code.

Par exemple, si vous voulez ajouter un nouveau produit dans
l'application, il vous suffit d'ajouter une sous-classe de création et
d'y redéfinir la méthode fabrique.
:::

::: applicability-problem
Utilisez la fabrique si vous voulez mettre à disposition une librairie
ou un framework pour vos utilisateurs avec un moyen d'étendre ses
composants internes.
:::

::: applicability-solution
L'héritage est probablement le moyen le plus simple pour étendre le
comportement par défaut d'une librairie ou d'un framework. Mais comment
le framework peut-il savoir qu'il doit utiliser votre sous-classe plutôt
qu'un composant standard ?

La solution est de réunir dans une seule méthode fabrique le code qui
construit les composants dans le framework, et non seulement d'étendre
ceux-ci, mais de laisser la possibilité de redéfinir la méthode
fabrique.

Voyons un exemple d'utilisation. Imaginez la conception d'une
application qui utilise un framework d'UI open source. Vous désirez
utiliser des boutons ronds, mais le framework ne fournit que des boutons
carrés. Vous étendez le `Bouton` standard avec une magnifique
sous-classe `BoutonRond`. Mais vous devez à présent expliquer à la
classe principale `UIFramework` qu'elle doit utiliser la sous-classe du
nouveau bouton plutôt que celle par défaut.

Pour ce faire, vous créez une sous-classe `UIAvecBoutonsRonds` depuis
une classe de base du framework et redéfinissez sa méthode
`créerBouton`. Même si la méthode de la classe de base retourne des
`Boutons`, votre sous-classe renvoie des `BoutonsRonds`. Vous pouvez
dorénavant utiliser `UIAvecBoutonsRonds` à la place de `UIFramework`. Et
c'est à peu près tout !
:::

::: applicability-problem
Utilisez la fabrique lorsque vous voulez économiser des ressources
système en réutilisant des objets au lieu d'en construire de nouveaux.
:::

::: applicability-solution
Le besoin se présente souvent lorsque l'on utilise des objets qui
prennent beaucoup de ressources tels que des bases de données, des
systèmes de fichiers ou des ressources réseau.

Que faut-il pour réutiliser un objet existant ?

1.  Tout d'abord, vous devez créer un moyen de stockage afin de garder
    la trace de tous les objets créés.
2.  Lorsqu'un nouvel objet est demandé, le programme doit chercher un
    objet libre dans cette réserve.
3.  ... et le renvoyer au code client.
4.  Si aucun objet n'est disponible, le programme en crée un nouveau (et
    l'ajoute à la réserve).

Cela représente un paquet de code ! De plus, il faut tout mettre au même
endroit afin de ne pas polluer le code avec des doublons.

Il serait probablement plus pratique de l'écrire dans le constructeur de
la classe de l'objet que l'on veut réutiliser, mais par définition, un
constructeur doit toujours renvoyer de **nouveaux objets**. Il ne peut
pas retourner des instances existantes.

C'est pourquoi vous devez disposer d'une méthode non seulement capable
de créer de nouveaux objets, mais aussi de réutiliser ceux qui existent
déjà. Cela ressemble énormément à un patron de conception fabrique.
:::
:::::::::
::::::::::

::: {.section .checklist}
##  Mise en œuvre {#checklist}

1.  Implémentez la même interface pour tous les produits. Cette
    interface doit déclarer des méthodes que tous les produits peuvent
    avoir en commun.

2.  Ajoutez une méthode fabrique vide à l'intérieur de la classe
    créateur. Le type de retour de la méthode doit correspondre à
    l'interface commune des produits.

3.  Localisez toutes les références aux constructeurs des produits dans
    le code du créateur. Remplacez-les une par une par des appels à la
    méthode fabrique et déplacez le code de la création de produits dans
    la méthode fabrique.

    Vous allez peut-être devoir ajouter un paramètre temporaire à la
    méthode fabrique pour vérifier le type du produit retourné.

    À ce stade, le code de la méthode fabrique peut paraître désordonné.
    Il pourrait même contenir un gros `switch` qui choisit la classe à
    instancier. Ne vous inquiétez pas, tout va bientôt rentrer dans
    l'ordre.

4.  Pour chaque type de produit listé dans la méthode fabrique, créez
    une sous-classe de Créateur. Redéfinissez la méthode fabrique dans
    les sous-classes et récupérez les morceaux de code appropriés de la
    méthode de base.

5.  S'il y a trop de types de produits et peu d'intérêt de créer des
    sous-classes pour tous, vous pouvez réutiliser le paramètre de
    contrôle de la classe de base dans les sous-classes.

    Imaginons la hiérarchie de classes suivante : la classe de base
    `Courrier` avec les sous-classes `CourrierAérien` et
    `CourrierTerrestre` ; les classes de `Transport` sont `Avion`,
    `Camion` et `Train`. La classe `CourrierAérien` n'utilise que des
    `Avions` et la classe `CourrierTerrestre` peut utiliser à la fois
    des `Camions` et des `Trains`. Vous pouvez créer une nouvelle
    sous-classe (`CourrierFerroviaire` par exemple) pour gérer les deux
    cas, mais il y a une autre possibilité. Le code client peut passer
    un argument à la méthode fabrique du `CourrierTerrestre` pour
    désigner le type de produit qu'elle veut recevoir.

6.  Si après tous ces changements la méthode fabrique de base est
    devenue complètement vide, vous pouvez la rendre abstraite. S'il
    reste encore quelques lignes, vous pouvez y laisser un comportement
    par défaut.
:::

:::::: {.section .pros-cons}
##  Avantages et inconvénients {#pros-cons}

::::: row
::: col-sm-6
-    Vous désolidarisez le Créateur des produits concrets.
-    *Principe de responsabilité unique*. Vous pouvez déplacer tout le
    code de création des produits au même endroit, permettant ainsi une
    meilleure maintenabilité.
-    *Principe ouvert/fermé*. Vous pouvez ajouter de nouveaux types de
    produits dans le programme sans endommager l'existant.
:::

::: col-sm-6
-    Le code peut devenir plus complexe puisque vous devez introduire de
    nombreuses sous-classes pour la mise en place du patron. La
    condition optimale d'intégration du patron dans du code existant se
    présente lorsque vous avez déjà une hiérarchie existante de classes
    de création.
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

-   Vous pouvez utiliser la [Fabrique](factory-method.html) avec
    l'[Itérateur](iterator.html) pour permettre aux sous-classes des
    collections de renvoyer différents types d'itérateurs compatibles
    avec les collections.

-   Le [Prototype](prototype.html) n'est pas basé sur l'héritage, il n'a
    donc pas ses désavantages. Mais le *prototype* requiert une
    initialisation compliquée pour l'objet cloné. La
    [Fabrique](factory-method.html) est basée sur l'héritage, mais n'a
    pas besoin d'une étape d'initialisation.

-   La [Fabrique](factory-method.html) est une spécialisation du [Patron
    de méthode](template-method.html). Une *fabrique* peut aussi faire
    office d'étape dans un grand *patron de méthode*.
:::

::: {.section .implementations}
##  Exemples de code {#implementations}

[![Fabrique en
C#](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](factory-method/csharp/example.html "Fabrique en C#"){.prog-lang-link}
[![Fabrique en
C++](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](factory-method/cpp/example.html "Fabrique en C++"){.prog-lang-link}
[![Fabrique en
Go](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](factory-method/go/example.html "Fabrique en Go"){.prog-lang-link}
[![Fabrique en
Java](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](factory-method/java/example.html "Fabrique en Java"){.prog-lang-link}
[![Fabrique en
PHP](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](factory-method/php/example.html "Fabrique en PHP"){.prog-lang-link}
[![Fabrique en
Python](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](factory-method/python/example.html "Fabrique en Python"){.prog-lang-link}
[![Fabrique en
Ruby](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](factory-method/ruby/example.html "Fabrique en Ruby"){.prog-lang-link}
[![Fabrique en
Rust](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](factory-method/rust/example.html "Fabrique en Rust"){.prog-lang-link}
[![Fabrique en
Swift](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](factory-method/swift/example.html "Fabrique en Swift"){.prog-lang-link}
[![Fabrique en
TypeScript](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](factory-method/typescript/example.html "Fabrique en TypeScript"){.prog-lang-link}
:::

::: {.section .extras}
##  Informations supplémentaires {#extras}

-   Lisez notre [Comparaison des fabriques](factory-comparison.html) si
    vous peinez à saisir la nuance entre les différents concepts et
    patrons.
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

[Fabrique abstraite []{.fa .fa-arrow-right}](abstract-factory.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Retour

[[]{.fa .fa-arrow-left} Patrons de
création](creational-patterns.html){.btn .btn-default rel="prev"}
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
