::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [Patrons de
conception](../design-patterns.html){.type} / [Patrons de
création](creational-patterns.html){.type}
:::

# Fabrique abstraite {#fabrique-abstraite .title}

::: aka
Alias : [Abstract Factory]{style="display:inline-block"}
:::

::: {.section .intent}
##  Intention {#intent}

**Fabrique abstraite** est un patron de conception qui permet de créer
des familles d'objets apparentés sans préciser leur classe concrète.

<figure class="image">
<img
src="../../images/patterns/content/abstract-factory/abstract-factory-fr2f4d.png?id=24e9fad67f30bb567efb3cda598e6ade"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/abstract-factory/abstract-factory-fr.png?id=24e9fad67f30bb567efb3cda598e6ade 640w,/images/patterns/content/abstract-factory/abstract-factory-fr-1.5x.png?id=c06d8b7ca4a6cb15ffcbcc642d4db34f 960w,/images/patterns/content/abstract-factory/abstract-factory-fr-2x.png?id=ed3c46902f554b04034e58529c1dd73d 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Patron de conception fabrique abstraite" />
</figure>
:::

::: {.section .problem}
##  Problème {#problem}

Imaginons la création d'un simulateur pour un magasin de meubles. Votre
code contient les classes suivantes :

1.  Une famille de produits appartenant à un même thème : `Chaise` +
    `Sofa` + `TableBasse`.

2.  Plusieurs variantes de cette famille. Par exemple, les produits
    `Chaise` + `Sofa` + `TableBasse` sont disponibles dans les variantes
    suivantes : `Moderne`, `Victorien`, `ArtDéco`.

<figure class="image">
<img
src="../../images/patterns/diagrams/abstract-factory/problem-fr39bf.png?id=b7f5754ef70ccd475ea046840c3d48ac"
style="aspect-ratio: 1.4;"
srcset="/images/patterns/diagrams/abstract-factory/problem-fr.png?id=b7f5754ef70ccd475ea046840c3d48ac 420w,/images/patterns/diagrams/abstract-factory/problem-fr-1.5x.png?id=f97c67cec9c3e5cb9e3084d230b661d8 630w,/images/patterns/diagrams/abstract-factory/problem-fr-2x.png?id=ae96a1118b97bbfa513dafc5a0ca6328 840w"
sizes="(max-width: 720px) 100vw, 420px" loading="lazy" width="420"
alt="Familles de produits et leurs variantes" />
<figcaption><p>Familles de produits et leurs variantes.</p></figcaption>
</figure>

Vous devez trouver une solution pour créer des objets individuels (des
meubles) et les faire correspondre à d'autres objets de la même famille.
Les clients sont agacés lorsqu'ils reçoivent des meubles qui ne se
marient pas.

<figure class="image">
<img
src="../../images/patterns/content/abstract-factory/abstract-factory-comic-1-fr1a3d.png?id=3a0c01764620850d22f67316553a711f"
style="aspect-ratio: 2;"
srcset="/images/patterns/content/abstract-factory/abstract-factory-comic-1-fr.png?id=3a0c01764620850d22f67316553a711f 600w,/images/patterns/content/abstract-factory/abstract-factory-comic-1-fr-1.5x.png?id=b239f0557df797366a782a09f660fa01 900w,/images/patterns/content/abstract-factory/abstract-factory-comic-1-fr-2x.png?id=493ba35e925c284cb9c63ede1fd3e11d 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600" />
<figcaption><p>Un sofa moderne ne se marie pas avec des chaises de
style victorien.</p></figcaption>
</figure>

De plus, vous n'avez pas envie de réécrire votre code chaque fois que
vous ajoutez de nouvelles familles de produits à votre programme. Les
vendeurs de meubles alimentent régulièrement leurs catalogues et il
n'est pas envisageable de restructurer le code à chaque mise à jour.
:::

::: {.section .solution}
##  Solution

La première chose que propose la fabrique abstraite est de déclarer
explicitement des interfaces pour chaque produit de la famille de
produits (dans notre cas : chaise, sofa, table basse). Toutes les autres
variantes de produits peuvent ensuite se servir de ces interfaces. Par
exemple, toutes les variantes de chaises peuvent implémenter l'interface
`Chaise` ; toutes les variantes de tables basses peuvent implémenter
`TableBasse`, etc.

<figure class="image">
<img
src="../../images/patterns/diagrams/abstract-factory/solution1a1ac.png?id=71f2018d8bb443b9cce90d57110675e3"
style="aspect-ratio: 1.5;"
srcset="/images/patterns/diagrams/abstract-factory/solution1.png?id=71f2018d8bb443b9cce90d57110675e3 420w,/images/patterns/diagrams/abstract-factory/solution1-1.5x.png?id=4366e971c850514cde5d33cb7956de8b 630w,/images/patterns/diagrams/abstract-factory/solution1-2x.png?id=eacec286153e058db9255d26541c0529 840w"
sizes="(max-width: 720px) 100vw, 420px" loading="lazy" width="420"
alt="La hiérarchie des classes chaise" />
<figcaption><p>Toutes les variantes du même objet peuvent être rangées
dans la même hiérarchie de classe.</p></figcaption>
</figure>

La prochaine étape est de déclarer la *fabrique abstraite* --- une
interface armée d'une liste de méthodes de création pour toutes les
familles de produits (par exemple : `créerChaise`, `créerSofa` et
`créerTableBasse`). Ces méthodes doivent renvoyer tous les types de
produits **abstraits** des interfaces que nous avons créées
précédemment : `Chaise`, `Sofa`, `TableBasse`, etc.

<figure class="image">
<img
src="../../images/patterns/diagrams/abstract-factory/solution24d8e.png?id=53975d6e4714c6f942633a879f7ac571"
style="aspect-ratio: 2;"
srcset="/images/patterns/diagrams/abstract-factory/solution2.png?id=53975d6e4714c6f942633a879f7ac571 640w,/images/patterns/diagrams/abstract-factory/solution2-1.5x.png?id=581a6cdc0dcd5511f1c30e509b1d4a7f 960w,/images/patterns/diagrams/abstract-factory/solution2-2x.png?id=b21557081f75ac7b4110427e89a10378 1280w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="640"
alt="La hiérarchie des classes des _fabriques_" />
<figcaption><p>Chaque fabrique concrète correspond à une variante d’un
produit spécifique.</p></figcaption>
</figure>

Mais que deviennent les variantes des produits ? Pour chaque variante
d'une famille de produits, nous créons une classe fabrique qui
implémente l'interface `FabriqueAbstraite`. Une fabrique est une classe
qui retourne un certain type de produits. Par exemple, la
`FabriqueMeubleModerne` peut uniquement créer des `ChaiseModerne`, des
`SofaModerne` et des `TableBasseModerne`.

Le code client travaille simultanément avec les interfaces abstraites
respectives des fabriques et des produits. Nous pouvons ainsi changer le
type de fabrique passé au code client et la variante de produit qu'il
reçoit, sans avoir à le modifier.

<figure class="image">
<img
src="../../images/patterns/content/abstract-factory/abstract-factory-comic-2-fr4a19.png?id=d262da7fbd41398a735aa6937505c9a9"
style="aspect-ratio: 2;"
srcset="/images/patterns/content/abstract-factory/abstract-factory-comic-2-fr.png?id=d262da7fbd41398a735aa6937505c9a9 600w,/images/patterns/content/abstract-factory/abstract-factory-comic-2-fr-1.5x.png?id=3f6f5f6b8a2389752dc8a0d5708e4c87 900w,/images/patterns/content/abstract-factory/abstract-factory-comic-2-fr-2x.png?id=507704f964936df2e468cedd9b29b752 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600" />
<figcaption><p>Le client n’a pas besoin de connaître la classe concrète
des fabriques qu’il utilise.</p></figcaption>
</figure>

Imaginons un cas où le client désire une fabrique qui peut produire une
chaise. Le client n'a pas à se préoccuper de la classe de la fabrique ni
du type de chaise qu'il va obtenir. Il doit traiter les chaises de la
même manière, que ce soit un modèle de style moderne ou victorien, en
utilisant l'interface abstraite `Chaise`. Grâce à cette approche, le
client ne sait qu'une seule chose à propos de la chaise, c'est qu'elle
implémente la méthode `sasseoir`. De plus, quelle que soit la variante
de la chaise renvoyée, elle correspondra systématiquement au type de
sofa ou de table basse produit par le même objet fabrique.

Il ne reste plus qu'un point à éclaircir : si le client n'est lié qu'aux
interfaces abstraites, comment les objets de la fabrique sont-ils
créés ? En général, l'application crée un objet fabrique concret lors de
l'initialisation. Mais avant cela, l'application doit choisir le type de
la fabrique en fonction de la configuration ou des paramètres
d'environnement.
:::

::::: {.section .structure-container}
##  Structure

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/abstract-factory/structureb809.png?id=a3112cdd98765406af94595a3c5e7762"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/diagrams/abstract-factory/structure.png?id=a3112cdd98765406af94595a3c5e7762 720w,/images/patterns/diagrams/abstract-factory/structure-1.5x.png?id=d2964e541620d9c087e693bd0e0a0836 1080w,/images/patterns/diagrams/abstract-factory/structure-2x.png?id=c4d3634ec2e74e02a0fe1a83ce9b50f6 1440w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="720"
alt="Patron de conception fabrique abstraite" /><img
src="../../images/patterns/diagrams/abstract-factory/structure-indexed028d.png?id=6ae1c99cbd90cf58753c633624fb1a04"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.56;"
srcset="/images/patterns/diagrams/abstract-factory/structure-indexed.png?id=6ae1c99cbd90cf58753c633624fb1a04 700w,/images/patterns/diagrams/abstract-factory/structure-indexed-1.5x.png?id=473be2975c5716162e6969e6532210ac 1050w,/images/patterns/diagrams/abstract-factory/structure-indexed-2x.png?id=cb6d4e1e89826c42966dc7097374f889 1400w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="700"
alt="Patron de conception fabrique abstraite" />
</figure>
:::

1.  Les **Produits Abstraits** déclarent une interface pour un ensemble
    d'objets distincts mais apparentés, qui forment une famille de
    produits.

2.  Les **Produits Concrets** sont des implémentations des produits
    abstraits groupés par variantes. Chaque produit abstrait
    (chaise/sofa) doit être implémenté dans toutes les variantes
    (victorien/moderne).

3.  L'interface **Fabrique Abstraite** déclare un ensemble de méthodes
    pour créer chacun des produits abstraits.

4.  Les **Fabriques Concrètes** implémentent les opérations de création
    d'objets de la fabrique abstraite. Chaque fabrique concrète
    correspond à une variante spécifique de produits et ne crée que ces
    variantes.

5.  Bien que les fabriques concrètes instancient des produits concrets,
    leurs méthodes de création ont une valeur de retour correspondant
    aux produits *abstraits*. Le code client qui sollicite une fabrique
    est ainsi isolé de la variante du produit obtenu. Le **client** peut
    travailler avec n'importe quelle variante de fabrique ou produit,
    tant qu'il interagit avec les interfaces abstraites.
::::
:::::

::: {.section .pseudocode}
##  Pseudo-code {#pseudocode}

Cet exemple montre comment la **Fabrique abstraite** peut être utilisée
pour créer des éléments d'une UI multiplateforme sans coupler le code
client avec les classes concrètes de l'UI, tout en gardant une cohérence
entre les éléments créés et le système d'exploitation sélectionné.

<figure class="image">
<img
src="../../images/patterns/diagrams/abstract-factory/exampled62f.png?id=5928a61d18bf00b047463471c599100a"
style="aspect-ratio: 1.42;"
srcset="/images/patterns/diagrams/abstract-factory/example.png?id=5928a61d18bf00b047463471c599100a 640w,/images/patterns/diagrams/abstract-factory/example-1.5x.png?id=baee3bac793ec97e0ec91c49d9382e7d 960w,/images/patterns/diagrams/abstract-factory/example-2x.png?id=eb5127b1d6486f6fad73be2d5e444449 1280w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="640"
alt="Le diagramme de classe de l’exemple utilisé pour la fabrique abstraite" />
<figcaption><p>Exemple des classes d’une
UI multiplateforme</p></figcaption>
</figure>

Tous les éléments de l'UI d'une application multiplateforme sont censés
avoir un comportement identique quel que soit le système d'exploitation,
mais leur apparence peut légèrement varier. Vous devez faire en sorte
que les éléments de l'interface correspondent bien au style du système
d'exploitation. Vous ne voudriez pas vous retrouver avec des boutons
macOS dans Windows.

L'interface de la fabrique abstraite déclare un ensemble de méthodes de
création que le code client peut utiliser pour produire différents types
d'éléments de l'UI. Chaque fabrique concrète correspond à un système
d'exploitation particulier et crée ses propres éléments de l'UI en
fonction de ce système d'exploitation.

Le fonctionnement est le suivant : lorsqu'une application est exécutée,
elle vérifie le système d'exploitation utilisé et exploite cette
information pour créer un objet de la fabrique qui y correspond. Cette
fabrique est ensuite utilisée pour générer tout le reste des éléments de
l'UI, ce qui évite les erreurs.

Grâce à cette approche, tant qu'il utilise leurs interfaces abstraites,
le code client ne dépend plus des classes concrètes de la fabrique et
des éléments de l'UI. Cela lui permet également d'exploiter les nouveaux
éléments de l'UI ou les fabriques que vous pourriez ajouter dans le
futur.

Nous n'avons ainsi plus besoin de modifier le code client à chaque
nouvel élément de l'interface utilisateur que vous voulez ajouter dans
votre application. Vous devrez simplement créer une nouvelle classe
fabrique produisant ces éléments et apporter quelques modifications au
code d'initialisation afin qu'il choisisse la classe appropriée.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// L’interface de la fabrique abstraite déclare un ensemble de
// méthodes qui retournent différents produits abstraits. Ces
// produits font partie de ce que l’on appelle une famille et
// sont reliés par un concept ou un thème de haut niveau. Les
// produits d’une famille sont généralement capables de
// collaborer les uns avec les autres. Une famille de produits
// peut avoir plusieurs variantes, mais les produits d’une
// variante ne sont pas compatibles avec les produits d’une
// autre.
interface GUIFactory is
    method createButton():Button
    method createCheckbox():Checkbox


// Les fabriques concrètes construisent une famille de produits
// qui appartiennent à une seule variante. La fabrique garantit
// la compatibilité des produits qui en sortent. Les signatures
// des méthodes des fabriques concrètes renvoient un produit
// abstrait, et à l’intérieur de la méthode, un produit concret
// est instancié.
class WinFactory implements GUIFactory is
    method createButton():Button is
        return new WinButton()
    method createCheckbox():Checkbox is
        return new WinCheckbox()

// Chaque fabrique concrète possède une variante de produit
// correspondante.
class MacFactory implements GUIFactory is
    method createButton():Button is
        return new MacButton()
    method createCheckbox():Checkbox is
        return new MacCheckbox()


// Chaque produit d’une famille de produits doit avoir une
// interface de base. Toutes les variantes du produit doivent
// implémenter cette interface.
interface Button is
    method paint()

// Les produits concrets sont créés par les fabriques concrètes
// correspondantes.
class WinButton implements Button is
    method paint() is
        // Affiche un bouton avec le style Windows.

class MacButton implements Button is
    method paint() is
        // Affiche un bouton avec le style macOS.

// Voici l’interface de base d’un autre produit. Tous les
// produits peuvent interagir les uns avec les autres, mais les
// interactions adéquates devraient être implémentées de
// préférence entre les produits d’une même variante concrète.
interface Checkbox is
    method paint()

class WinCheckbox implements Checkbox is
    method paint() is
        // Affiche une case à cocher avec le style Windows.

class MacCheckbox implements Checkbox is
    method paint() is
        // Affiche une case à cocher avec le style macOS.


// Le code client travaille uniquement avec les types abstraits
// des fabriques et des produits : FabriqueGUI, Bouton et
// CaseÀCocher. Ceci vous permet d’envoyer n’importe quelle
// sous-classe de fabrique ou de produit au code client sans
// toucher à son code.
class Application is
    private field factory: GUIFactory
    private field button: Button
    constructor Application(factory: GUIFactory) is
        this.factory = factory
    method createUI() is
        this.button = factory.createButton()
    method paint() is
        button.paint()


// L’application choisit le type de la fabrique en fonction de
// la configuration actuelle ou des paramètres d’environnement,
// et la crée à l’exécution (en général lors de la phase
// d’initialisation).
class ApplicationConfigurator is
    method main() is
        config = readApplicationConfigFile()

        if (config.OS == &quot;Windows&quot;) then
            factory = new WinFactory()
        else if (config.OS == &quot;Mac&quot;) then
            factory = new MacFactory()
        else
            throw new Exception(&quot;Error! Unknown operating system.&quot;)

        Application app = new Application(factory)</code></pre>
</figure>
:::

:::::::: {.section .applicability-container}
##  Possibilités d'application {#applicability}

::::::: applicability
::: applicability-problem
Utilisez la fabrique abstraite si votre code a besoin de manipuler des
produits d'un même thème, mais que vous ne voulez pas qu'il dépende des
classes concrètes de ces produits --- soit vous ne les connaissez pas
encore, soit vous voulez juste rendre votre code évolutif.
:::

::: applicability-solution
La fabrique abstraite fournit une interface qui permet de créer des
objets pour chaque classe de la famille de produits. Tant que votre code
utilise cette interface pour créer ses objets, il prendra
systématiquement les bonnes variantes des produits disponibles dans
votre application.
:::

::: applicability-problem
Étudiez la possibilité d'utiliser la fabrique abstraite lorsqu'une
classe se retrouve avec un ensemble de patrons
[Fabrique](factory-method.html) qui occultent sa fonction principale.
:::

::: applicability-solution
Dans un programme bien conçu *chaque classe n'a qu'une seule
responsabilité*. Lorsqu'une classe gère plusieurs types de produits,
déplacer les méthodes fabrique dans des fabriques individuelles ou dans
une implémentation à part entière d'une fabrique abstraite est souvent
plus pratique.
:::
:::::::
::::::::

::: {.section .checklist}
##  Mise en œuvre {#checklist}

1.  Établissez une matrice de vos différents types de produits et leurs
    variantes.

2.  Déclarez des interfaces abstraites pour tous vos types de produits.
    Mettez en place vos produits concrets dans des classes qui
    implémentent ces interfaces.

3.  Déclarez une interface abstraite de la fabrique avec un ensemble de
    méthodes de création pour tous les produits abstraits.

4.  Implémentez une classe fabrique concrète pour chaque variante des
    produits.

5.  Insérez le code de l'initialisation de la fabrique quelque part dans
    votre application. Il doit instancier une des fabriques concrètes en
    fonction de la configuration de l'application ou de l'environnement
    d'exécution. Passez cette fabrique à toutes les classes qui
    construisent des produits.

6.  Parcourez votre code et repérez tous les appels aux constructeurs
    des produits. Remplacez-les par des appels à la méthode de création
    appropriée de la fabrique.
:::

:::::: {.section .pros-cons}
##  Avantages et inconvénients {#pros-cons}

::::: row
::: col-sm-6
-    Vous êtes assurés que les produits d'une fabrique sont compatibles
    entre eux.
-    Vous découplez le code client des produits concrets.
-    *Principe de responsabilité unique*. Vous pouvez déplacer tout le
    code de création des produits au même endroit, pour une meilleure
    maintenabilité.
-    *Principe ouvert/fermé*. Vous pouvez ajouter de nouvelles variantes
    de produits sans endommager l'existant.
:::

::: col-sm-6
-    Le code peut devenir plus complexe que nécessaire, car ce patron de
    conception impose l'ajout de nouvelles classes et interfaces.
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

-   Les classes [Fabrique abstraite](abstract-factory.html) sont souvent
    basées sur un ensemble de [Fabriques](factory-method.html), mais
    vous pouvez également utiliser le [Prototype](prototype.html) pour
    écrire leurs méthodes.

-   La [Fabrique abstraite](abstract-factory.html) peut remplacer la
    [Façade](facade.html) si vous voulez simplement cacher au code
    client la création des objets du sous-système.

-   Vous pouvez utiliser la [Fabrique abstraite](abstract-factory.html)
    avec le [Pont](bridge.html). Ce couple est très utile quand les
    abstractions définies par le *pont* ne fonctionnent qu'avec
    certaines implémentations spécifiques. Dans ce cas, la *fabrique
    abstraite* peut encapsuler ces relations et cacher la complexité au
    code client.

-   Les [Fabriques abstraites](abstract-factory.html),
    [Monteurs](builder.html) et [Prototypes](prototype.html) peuvent
    tous être implémentés comme des [Singletons](singleton.html).
:::

::: {.section .implementations}
##  Exemples de code {#implementations}

[![Fabrique abstraite en
C#](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](abstract-factory/csharp/example.html "Fabrique abstraite en C#"){.prog-lang-link}
[![Fabrique abstraite en
C++](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](abstract-factory/cpp/example.html "Fabrique abstraite en C++"){.prog-lang-link}
[![Fabrique abstraite en
Go](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](abstract-factory/go/example.html "Fabrique abstraite en Go"){.prog-lang-link}
[![Fabrique abstraite en
Java](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](abstract-factory/java/example.html "Fabrique abstraite en Java"){.prog-lang-link}
[![Fabrique abstraite en
PHP](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](abstract-factory/php/example.html "Fabrique abstraite en PHP"){.prog-lang-link}
[![Fabrique abstraite en
Python](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](abstract-factory/python/example.html "Fabrique abstraite en Python"){.prog-lang-link}
[![Fabrique abstraite en
Ruby](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](abstract-factory/ruby/example.html "Fabrique abstraite en Ruby"){.prog-lang-link}
[![Fabrique abstraite en
Rust](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](abstract-factory/rust/example.html "Fabrique abstraite en Rust"){.prog-lang-link}
[![Fabrique abstraite en
Swift](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](abstract-factory/swift/example.html "Fabrique abstraite en Swift"){.prog-lang-link}
[![Fabrique abstraite en
TypeScript](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](abstract-factory/typescript/example.html "Fabrique abstraite en TypeScript"){.prog-lang-link}
:::

::: {.section .extras}
##  Informations supplémentaires {#extras}

-   Lisez notre [Comparaison des fabriques](factory-comparison.html)
    pour en savoir plus sur la différence entre les divers patrons et
    concepts.
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

[Comparaison des fabriques []{.fa
.fa-arrow-right}](factory-comparison.html){.btn .btn-primary rel="next"}
:::

::: prev
#### Retour

[[]{.fa .fa-arrow-left} Fabrique](factory-method.html){.btn .btn-default
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
