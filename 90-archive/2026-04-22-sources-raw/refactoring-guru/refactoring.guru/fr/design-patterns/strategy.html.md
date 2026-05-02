::::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [Patrons de
conception](../design-patterns.html){.type} / [Patrons
comportementaux](behavioral-patterns.html){.type}
:::

# Stratégie {#stratégie .title}

::: aka
Alias :
[Politique, ]{style="display:inline-block"}[Strategy]{style="display:inline-block"}
:::

::: {.section .intent}
##  Intention {#intent}

**Stratégie** est un patron de conception comportemental qui permet de
définir une famille d'algorithmes, de les mettre dans des classes
séparées et de rendre leurs objets interchangeables.

<figure class="image">
<img
src="../../images/patterns/content/strategy/strategy4ca6.png?id=379bfba335380500375881a3da6507e0"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/strategy/strategy.png?id=379bfba335380500375881a3da6507e0 640w,/images/patterns/content/strategy/strategy-1.5x.png?id=33ecebc66a9761454f2786a9b3e9bbe5 960w,/images/patterns/content/strategy/strategy-2x.png?id=1cee47d05a76fddf07dce9c67b700748 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Patron de conception stratégie" />
</figure>
:::

::: {.section .problem}
##  Problème {#problem}

Un beau jour, vous avez décidé de créer une application de navigation
pour les voyageurs occasionnels. Vous l'avez développé avec une superbe
carte comme fonctionnalité principale, qui aide les utilisateurs à
s'orienter rapidement dans n'importe quelle ville.

La fonctionnalité la plus demandée était la planification d'itinéraire.
Un utilisateur devrait pouvoir entrer une adresse et le chemin le plus
rapide pour arriver à destination s'afficherait sur la carte.

La première version de l'application ne pouvait tracer des itinéraires
que sur les routes. Les automobilistes étaient comblés. Mais
apparemment, certaines personnes préfèrent utiliser d'autres moyens de
locomotion pendant leurs vacances. Vous avez ajouté la possibilité de
créer des trajets à pied dans la version suivante. Juste après cela,
vous avez ajouté la possibilité d'utiliser les transports en commun dans
les itinéraires.

Mais tout ceci n'était que le début. Vous avez continué en adaptant
l'application pour les cyclistes, et plus tard, ajouté la possibilité de
construire les itinéraires en passant par les attractions touristiques
de la ville.

<figure class="image">
<img
src="../../images/patterns/diagrams/strategy/problem9b55.png?id=e5ca943e559a979bd31d20e232dc22b6"
style="aspect-ratio: 2.2;"
srcset="/images/patterns/diagrams/strategy/problem.png?id=e5ca943e559a979bd31d20e232dc22b6 330w,/images/patterns/diagrams/strategy/problem-1.5x.png?id=31d1042ffc28056845e45d5c616da2a9 495w,/images/patterns/diagrams/strategy/problem-2x.png?id=3974fb99c97aec525dd0ffcff2e48e78 660w"
sizes="(max-width: 720px) 100vw, 330px" loading="lazy" width="330"
alt="Le code du navigateur grossit à vue d’œil" />
<figcaption><p>Le code du navigateur grossit à
vue d’œil.</p></figcaption>
</figure>

L'application a beau avoir très bien marché d'un point de vue financier,
vous vous êtes arraché les cheveux sur le côté technique. Chaque fois
que vous ajoutiez un nouvel algorithme pour tracer les itinéraires, la
classe principale Navigateur doublait de taille. À un moment donné, la
bête n'était plus possible à maintenir.

Que ce soit pour corriger un petit problème ou pour ajuster les scores
des rues, la moindre touche apportée aux algorithmes impactait la
totalité de la classe, augmentant les chances de créer des bugs dans du
code qui fonctionnait très bien.

De plus, travailler en équipe n'était plus efficace. Les membres de
votre équipe embauchés juste après la sortie et le succès de votre
application se plaignaient de passer trop de temps à résoudre des
problèmes de fusion. Ajouter une nouvelle fonctionnalité vous demandait
de modifier une classe énorme, créant des conflits dans le code produit
par les autres développeurs.
:::

::: {.section .solution}
##  Solution

Le patron de conception stratégie vous propose de prendre une classe
dotée d'un comportement spécifique mais qui l'exécute de différentes
façons, et de décomposer ses algorithmes en classes séparées appelées
*stratégies*.

La classe originale (le *contexte*) doit avoir un attribut qui garde une
référence vers une des stratégies. Plutôt que de s'occuper de la tâche,
le contexte la délègue à l'objet stratégie associé.

Le contexte n'a pas la responsabilité de la sélection de l'algorithme
adapté, c'est le client qui lui envoie la stratégie. En fait, le
contexte n'y connait pas grand-chose en stratégies, c'est l'interface
générique qui lui permet de les utiliser. Elle n'expose qu'une seule
méthode pour déclencher l'algorithme encapsulé à l'intérieur de la
stratégie sélectionnée.

Le contexte devient indépendant des stratégies concrètes. Vous pouvez
ainsi modifier des algorithmes ou en ajouter de nouveaux sans toucher au
code du contexte ou aux autres stratégies.

<figure class="image">
<img
src="../../images/patterns/diagrams/strategy/solution1435.png?id=0813a174b29a2ed5902d321aba28e5fc"
style="aspect-ratio: 2.04;"
srcset="/images/patterns/diagrams/strategy/solution.png?id=0813a174b29a2ed5902d321aba28e5fc 570w,/images/patterns/diagrams/strategy/solution-1.5x.png?id=ce3d4e57f4a2a06ebc96f2607b3d6691 855w,/images/patterns/diagrams/strategy/solution-2x.png?id=66b5ee048ea2ad25c4b20f180ebf94d7 1140w"
sizes="(max-width: 720px) 100vw, 570px" loading="lazy" width="570"
alt="Stratégies de planification d’itinéraire" />
<figcaption><p>Stratégies de
planification d’itinéraire.</p></figcaption>
</figure>

Dans notre application de navigation, chaque algorithme d'itinéraire
peut être extrait de sa propre classe avec une seule méthode
`tracerItinéraire`. La méthode accepte une origine et une destination,
puis retourne une liste de points de passage.

Quand bien même les différentes classes itinéraire ne donneraient pas un
résultat identique avec les mêmes paramètres, la classe navigateur
principale ne se préoccupe pas de l'algorithme sélectionné, car sa
fonction première est d'afficher les points de passage sur la carte. La
classe navigateur possède une méthode pour changer la stratégie
d'itinéraire active afin que ses clients (les boutons de l'interface
utilisateur par exemple) puissent remplacer le comportement sélectionné
par un autre.
:::

::: {.section .analogy}
##  Analogie {#analogy}

<figure class="image">
<img
src="../../images/patterns/content/strategy/strategy-comic-1-fr38ca.png?id=1d3ad14476434dd642f689d176336a88"
style="aspect-ratio: 2.13;"
srcset="/images/patterns/content/strategy/strategy-comic-1-fr.png?id=1d3ad14476434dd642f689d176336a88 640w,/images/patterns/content/strategy/strategy-comic-1-fr-1.5x.png?id=2f37a0b2b875e715c483723ffb044ca9 960w,/images/patterns/content/strategy/strategy-comic-1-fr-2x.png?id=df7def0d47a546cd50e5c4bf6e9af495 1280w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="640"
alt="Différentes stratégies de transport" />
<figcaption><p>Différentes stratégies pour se rendre
à l’aéroport.</p></figcaption>
</figure>

Imaginez que vous devez vous rendre à l'aéroport. Vous pouvez prendre le
bus, appeler un taxi ou enfourcher votre vélo. Ce sont vos stratégies de
transport. Vous pouvez sélectionner une de ces stratégies en fonction de
certains facteurs, comme le budget ou les contraintes de temps.
:::

::::: {.section .structure-container}
##  Structure

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/strategy/structuree491.png?id=c6aa910c94960f35d100bfca02810ea1"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.19;"
srcset="/images/patterns/diagrams/strategy/structure.png?id=c6aa910c94960f35d100bfca02810ea1 440w,/images/patterns/diagrams/strategy/structure-1.5x.png?id=5b1c6376b06374719dcae95455b068d8 660w,/images/patterns/diagrams/strategy/structure-2x.png?id=5bd791857c3bab419bcf4fa86877439d 880w"
sizes="(max-width: 720px) 100vw, 440px" loading="lazy" width="440"
alt="Structure du patron de conception stratégie" /><img
src="../../images/patterns/diagrams/strategy/structure-indexed64c7.png?id=ff55c5a6273cf82a5667f3cab5b605c7"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.15;"
srcset="/images/patterns/diagrams/strategy/structure-indexed.png?id=ff55c5a6273cf82a5667f3cab5b605c7 450w,/images/patterns/diagrams/strategy/structure-indexed-1.5x.png?id=3d6ba05b8a74a9fb8e3f88eb9ca1e30f 675w,/images/patterns/diagrams/strategy/structure-indexed-2x.png?id=9f8e2f838f16705775411e2b4616820e 900w"
sizes="(max-width: 720px) 100vw, 450px" loading="lazy" width="450"
alt="Structure du patron de conception stratégie" />
</figure>
:::

1.  Le **Contexte** garde une référence vers une des stratégies
    concrètes et communique avec cet objet uniquement au travers de
    l'interface stratégie.

2.  L'interface **Stratégie** est commune à toutes les stratégies
    concrètes. Elle déclare une méthode que le contexte utilise pour
    exécuter une stratégie.

3.  Les **Stratégies Concrètes** implémentent différentes variantes
    d'algorithmes utilisées par le contexte.

4.  Chaque fois qu'il veut lancer un algorithme, le contexte appelle la
    méthode d'exécution de l'objet stratégie associé. Le contexte ne
    sait pas comment la stratégie fonctionne ni comment l'algorithme est
    lancé.

5.  Le **Client** crée un objet spécifique Stratégie et le passe au
    contexte. Le contexte expose un setter qui permet aux clients de
    remplacer la stratégie associée au contexte lors de l'exécution.
::::
:::::

::: {.section .pseudocode}
##  Pseudo-code {#pseudocode}

Dans cet exemple, le contexte utilise plusieurs **Stratégies** pour
lancer diverses opérations arithmétiques.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// L’interface stratégie déclare les traitements communs à
// toutes les versions supportées de l’algorithme. Le contexte
// utilise cette interface pour appeler l’algorithme défini par
// les stratégies concrètes.
interface Strategy is
    method execute(a, b)

// Les stratégies concrètes implémentent l’interface de base
// Stratégie et hébergent l’algorithme. L’interface les rend
// interchangeables dans le contexte.
class ConcreteStrategyAdd implements Strategy is
    method execute(a, b) is
        return a + b

class ConcreteStrategySubtract implements Strategy is
    method execute(a, b) is
        return a - b

class ConcreteStrategyMultiply implements Strategy is
    method execute(a, b) is
        return a * b

// Le contexte définit l’interface dont les clients ont besoin.
class Context is
    // Le contexte maintient une référence à l’un des objets
    // Stratégie. Le contexte ne connait pas la classe concrète
    // de la stratégie. Il doit manipuler toutes les stratégies
    // via l’interface stratégie.
    private strategy: Strategy

    // En général, le contexte accepte une stratégie en
    // paramètre du constructeur et fournit un setter pour
    // permettre de changer de stratégie lors de l’exécution.
    method setStrategy(Strategy strategy) is
        this.strategy = strategy

    // Le contexte délègue certaines tâches à l’objet stratégie,
    // plutôt que d’implémenter plusieurs versions de
    // l’algorithme.
    method executeStrategy(int a, int b) is
        return strategy.execute(a, b)


// Le code client choisit une stratégie concrète et la passe au
// contexte. Le client doit connaitre les différences entre les
// stratégies afin de faire le bon choix.
class ExampleApplication is
    method main() is
        Create context object.

        Read first number.
        Read last number.
        Read the desired action from user input.

        if (action == addition) then
            context.setStrategy(new ConcreteStrategyAdd())

        if (action == subtraction) then
            context.setStrategy(new ConcreteStrategySubtract())

        if (action == multiplication) then
            context.setStrategy(new ConcreteStrategyMultiply())

        result = context.executeStrategy(First number, Second number)

        Print result.</code></pre>
</figure>
:::

:::::::::::: {.section .applicability-container}
##  Possibilités d'application {#applicability}

::::::::::: applicability
::: applicability-problem
Utilisez le patron de conception stratégie si vous voulez avoir
différentes variantes d'un algorithme à l'intérieur d'un objet à
disposition, et pouvoir passer d'un algorithme à l'autre lors de
l'exécution.
:::

::: applicability-solution
Ce patron vous permet de modifier indirectement le comportement de
l'objet lors de l'exécution, en l'associant avec différents sous-objets
qui peuvent accomplir des sous-tâches spécifiques de différentes
manières.
:::

::: applicability-problem
Utilisez la stratégie si vous avez beaucoup de classes dont la seule
différence est leur façon d'exécuter un comportement.
:::

::: applicability-solution
Ce patron vous permet d'extraire des variantes d'un comportement dans
une hiérarchie de classes séparées et de combiner les classes originales
dans une seule, évitant de dupliquer du code.
:::

::: applicability-problem
Utilisez la stratégie pour isoler la logique métier d'une classe, de
l'implémentation des algorithmes dont les détails ne sont pas forcément
importants pour le contexte.
:::

::: applicability-solution
Ce patron vous permet de séparer le code, les données internes et les
dépendances des divers algorithmes du reste du code. Une interface toute
simple permet aux clients d'exécuter les algorithmes et d'en changer
lors de l'exécution.
:::

::: applicability-problem
Utilisez ce patron si votre classe possède un gros bloc conditionnel qui
choisit entre différentes variantes du même algorithme.
:::

::: applicability-solution
La stratégie vous débarrasse de toutes ces conditions en extrayant tous
les algorithmes dans des classes séparées, et ces dernières implémentent
toutes la même interface. L'objet original délègue l'exécution à l'un de
ces objets, au lieu d'implémenter toutes les variantes de l'algorithme.
:::
:::::::::::
::::::::::::

::: {.section .checklist}
##  Mise en œuvre {#checklist}

1.  Dans la classe contexte, identifiez un algorithme qui varie souvent.
    Il peut s'agir d'un gros bloc conditionnel qui sélectionne une
    variante du même algorithme lors de l'exécution.

2.  Déclarez l'interface stratégie commune à toutes les variantes de
    l'algorithme.

3.  Extrayez tous les algorithmes un par un et mettez-les dans leurs
    propres classes. Elles doivent toutes implémenter l'interface
    stratégie.

4.  Ajoutez un attribut pour garder une référence vers un objet
    stratégie dans la classe contexte. Créez un setter pour modifier le
    contenu de cet attribut. Le contexte ne doit manipuler l'objet
    stratégie qu'au travers de l'interface stratégie. Le contexte peut
    définir une interface qui laisse la stratégie accéder à ses données.

5.  Les clients d'un contexte doivent l'associer avec une stratégie
    adaptée au comportement attendu.
:::

:::::: {.section .pros-cons}
##  Avantages et inconvénients {#pros-cons}

::::: row
::: col-sm-6
-    Vous pouvez permuter l'algorithme utilisé à l'intérieur d'un objet
    à l'exécution.
-    Vous pouvez séparer les détails de l'implémentation d'un algorithme
    et le code qui l'utilise.
-    Vous pouvez remplacer l'héritage par la composition.
-    *Principe ouvert/fermé*. Vous pouvez ajouter de nouvelles
    stratégies sans avoir à modifier le contexte.
:::

::: col-sm-6
-    Si vous n'avez que quelques algorithmes qui ne varient pas
    beaucoup, nul besoin de rendre votre programme plus compliqué avec
    les nouvelles classes et interfaces qui accompagnent la mise en
    place du patron.
-    Les clients doivent pouvoir comparer les différentes stratégies et
    choisir la bonne.
-    De nombreux langages de programmation modernes gèrent les types
    fonctionnels et vous permettent d'implémenter différentes versions
    d'un algorithme à l'intérieur d'un ensemble de fonctions anonymes.
    Vous pouvez ensuite utiliser ces fonctions exactement comme vous le
    feriez pour des objets stratégie, sans encombrer votre code avec des
    classes et interfaces supplémentaires.
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

-   La [Commande](command.html) et la [Stratégie](strategy.html) peuvent
    se ressembler, car vous les utilisez toutes les deux pour paramétrer
    un objet avec une action. Cependant, ces patrons ont des intentions
    très différentes.

    -   Vous pouvez utiliser la *commande* pour convertir un traitement
        en un objet. Les paramètres du traitement deviennent des
        attributs de cet objet. La conversion vous permet de différer le
        lancement du traitement, le mettre dans une file d'attente,
        stocker l'historique des commandes, envoyer les commandes à des
        services distants, etc.

    -   La *stratégie* quant à elle, décrit généralement différentes
        manières de faire la même chose et vous laisse permuter entre
        ces algorithmes à l'intérieur d'une unique classe contexte.

-   Le [Décorateur](decorator.html) vous permet de changer la peau d'un
    objet, alors que la [Stratégie](strategy.html) vous permet de
    changer ses tripes.

-   Le [Patron de méthode](template-method.html) est basé sur
    l'héritage : il vous laisse modifier certaines parties d'un
    algorithme en les étendant dans les sous-classes. La
    [Stratégie](strategy.html) est basée sur la composition : vous
    pouvez modifier certaines parties du comportement de l'objet en lui
    fournissant différentes stratégies qui correspondent à ce
    comportement. Le *patron de méthode* agit au niveau de la classe, il
    est donc statique. La *stratégie* agit au niveau de l'objet et vous
    laisse permuter les comportements à l'exécution.

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

[![Stratégie en
C#](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](strategy/csharp/example.html "Stratégie en C#"){.prog-lang-link}
[![Stratégie en
C++](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](strategy/cpp/example.html "Stratégie en C++"){.prog-lang-link}
[![Stratégie en
Go](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](strategy/go/example.html "Stratégie en Go"){.prog-lang-link}
[![Stratégie en
Java](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](strategy/java/example.html "Stratégie en Java"){.prog-lang-link}
[![Stratégie en
PHP](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](strategy/php/example.html "Stratégie en PHP"){.prog-lang-link}
[![Stratégie en
Python](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](strategy/python/example.html "Stratégie en Python"){.prog-lang-link}
[![Stratégie en
Ruby](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](strategy/ruby/example.html "Stratégie en Ruby"){.prog-lang-link}
[![Stratégie en
Rust](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](strategy/rust/example.html "Stratégie en Rust"){.prog-lang-link}
[![Stratégie en
Swift](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](strategy/swift/example.html "Stratégie en Swift"){.prog-lang-link}
[![Stratégie en
TypeScript](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](strategy/typescript/example.html "Stratégie en TypeScript"){.prog-lang-link}
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

[Patron de méthode []{.fa .fa-arrow-right}](template-method.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Retour

[[]{.fa .fa-arrow-left} État](state.html){.btn .btn-default rel="prev"}
:::
::::::::::::::::::::::::::::::::::::

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
::::::::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::::::::
