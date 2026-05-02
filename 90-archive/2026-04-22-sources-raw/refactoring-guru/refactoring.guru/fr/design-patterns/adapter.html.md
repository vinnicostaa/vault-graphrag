:::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [Patrons de
conception](../design-patterns.html){.type} / [Patrons
structurels](structural-patterns.html){.type}
:::

# Adaptateur {#adaptateur .title}

::: aka
Alias :
[Wrapper, ]{style="display:inline-block"}[Adapter]{style="display:inline-block"}
:::

::: {.section .intent}
##  Intention {#intent}

L'**Adaptateur** est un patron de conception structurel qui permet de
faire collaborer des objets ayant des interfaces
normalement incompatibles.

<figure class="image">
<img
src="../../images/patterns/content/adapter/adapter-frff1a.png?id=9fcbfd455aadda9c3b92296ca1e79b59"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/adapter/adapter-fr.png?id=9fcbfd455aadda9c3b92296ca1e79b59 640w,/images/patterns/content/adapter/adapter-fr-1.5x.png?id=da9b9faf4e1cbea16dc7f44a07299994 960w,/images/patterns/content/adapter/adapter-fr-2x.png?id=d0c7fbc50ab04104cae93a347739174b 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Patron de conception adaptateur" />
</figure>
:::

::: {.section .problem}
##  Problème {#problem}

Imaginez que vous êtes en train de créer une application de surveillance
du marché boursier. L'application télécharge des données de la bourse
depuis diverses sources au format XML et affiche ensuite de jolis
graphiques et diagrammes destinés à l'utilisateur.

Après un certain temps, vous décidez d'améliorer l'application en
intégrant une librairie d'analyse externe. Mais il y a un hic ! Cette
librairie ne fonctionne qu'avec des données au format JSON.

<figure class="image">
<img
src="../../images/patterns/diagrams/adapter/problem-fr3dcc.png?id=b454cc4fe0491f1a461812d5ba5a959b"
style="aspect-ratio: 2.41;"
srcset="/images/patterns/diagrams/adapter/problem-fr.png?id=b454cc4fe0491f1a461812d5ba5a959b 530w,/images/patterns/diagrams/adapter/problem-fr-1.5x.png?id=5d4362b870ba130957c7083ba6ea32ec 795w,/images/patterns/diagrams/adapter/problem-fr-2x.png?id=3fed043a954e5854b615410c08a8db70 1060w"
sizes="(max-width: 720px) 100vw, 530px" loading="lazy" width="530"
alt="La structure de l’application avant intégration de la librairie d’analyse" />
<figcaption><p>Vous ne pouvez pas utiliser la librairie telle qu’elle
est actuellement, car elle attend des données incompatibles avec
votre application.</p></figcaption>
</figure>

Vous pourriez modifier la librairie afin qu'elle accepte du XML, mais
vous risquez de faire planter d'autres parties de code qui utilisent
déjà cette librairie. Ou alors, vous n'avez tout simplement pas accès au
code source de la librairie, rendant la tâche impossible.
:::

::: {.section .solution}
##  Solution

Vous créez un *adaptateur*. C'est un objet spécial qui convertit
l'interface d'un objet afin qu'un autre objet puisse le comprendre.

Un adaptateur encapsule un des objets afin de masquer la complexité de
la conversion, exécutée à l'ombre des regards. L'objet encapsulé n'a pas
conscience de ce que fait l'adaptateur. Par exemple, vous pouvez
encapsuler un objet qui calcule en mètres et en kilomètres avec un
adaptateur qui effectue la conversion de toutes les données en unités
impériales comme les pieds et les milles.

Les adaptateurs peuvent non seulement effectuer des conversions dans
différents formats, mais ils peuvent également aider différentes
interfaces à collaborer. Le fonctionnement de l'adaptateur est le
suivant :

1.  L'adaptateur prend une interface compatible avec un des objets
    existants.
2.  L'objet existant peut appeler les méthodes de l'adaptateur via cette
    interface en toute sécurité.
3.  Lorsque l'adaptateur reçoit un appel, il passe la requête au second
    objet dans un format et dans un ordre qu'il peut interpréter.

Il est même parfois possible de créer un adaptateur qui peut convertir
dans les deux sens !

<figure class="image">
<img
src="../../images/patterns/diagrams/adapter/solution-fr0e60.png?id=cb2b5f0f40e8eef1fd9ce1d3027fc7d5"
style="aspect-ratio: 1.51;"
srcset="/images/patterns/diagrams/adapter/solution-fr.png?id=cb2b5f0f40e8eef1fd9ce1d3027fc7d5 530w,/images/patterns/diagrams/adapter/solution-fr-1.5x.png?id=990464a9d2d8e79b1321cc7c0cb33da1 795w,/images/patterns/diagrams/adapter/solution-fr-2x.png?id=abc6c8200f3540599c42b1090d8ab568 1060w"
sizes="(max-width: 720px) 100vw, 530px" loading="lazy" width="530"
alt="Solution utilisant l’adaptateur" />
</figure>

Retournons à notre application de surveillance du marché boursier. Pour
résoudre le problème des formats incompatibles, vous pouvez créer des
adaptateurs XML vers JSON pour chaque classe de la librairie que notre
code veut utiliser. Vous n'avez plus qu'à ajuster votre code pour
communiquer avec la librairie à l'aide de ces adaptateurs. Lorsqu'un
adaptateur reçoit un appel, il convertit les données XML en une
structure JSON. Il renvoie ensuite l'appel à la méthode appropriée dans
un objet d'analyse encapsulé.
:::

::: {.section .analogy}
##  Analogie {#analogy}

<figure class="image">
<img
src="../../images/patterns/content/adapter/adapter-comic-1-fre17f.png?id=b600a8af344dde0544aec5691c028ee9"
style="aspect-ratio: 1.78;"
srcset="/images/patterns/content/adapter/adapter-comic-1-fr.png?id=b600a8af344dde0544aec5691c028ee9 533w,/images/patterns/content/adapter/adapter-comic-1-fr-1.5x.png?id=8e5b4d0b23c141f1c756448af072bc63 800w,/images/patterns/content/adapter/adapter-comic-1-fr-2x.png?id=b0c76dc68d3bfc9a71ccd3ef295a6a7c 1067w"
sizes="(max-width: 720px) 100vw, 533px" loading="lazy" width="533"
alt="Exemple du patron de conception adaptateur" />
<figcaption><p>Une valise avant et après un voyage
à l’étranger.</p></figcaption>
</figure>

Si vous voyagez aux États-Unis pour la première fois, vous allez avoir
une petite surprise lorsque vous allez essayer de brancher votre
ordinateur portable. Les câbles et prises de courant sont différents
dans les autres pays : les câbles français ne rentrent pas dans les
prises américaines. Ce problème peut être résolu en utilisant un
adaptateur qui accepte un câble européen d'un côté et une prise
américaine de l'autre.
:::

:::::::: {.section .structure-container}
##  Structure

::::::: structure
#### Adaptateur d'objets

Cette implémentation a recours au principe de composition : l'adaptateur
implémente l'interface d'un objet et en encapsule un autre. Elle peut
être utilisée dans tous les langages de programmation classiques.

:::: adapter-object
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/adapter/structure-object-adaptera83b.png?id=33dffbe3aece294162440c7ddd3d5d4f"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.81;"
srcset="/images/patterns/diagrams/adapter/structure-object-adapter.png?id=33dffbe3aece294162440c7ddd3d5d4f 580w,/images/patterns/diagrams/adapter/structure-object-adapter-1.5x.png?id=c1b8a87b74fc4ce5639212fe19ee06fe 870w,/images/patterns/diagrams/adapter/structure-object-adapter-2x.png?id=03e8052e168c962d6bc369bbb13b0945 1160w"
sizes="(max-width: 720px) 100vw, 580px" loading="lazy" width="580"
alt="Structure du patron de conception adaptateur (adaptateur d’objets/object adapter)" /><img
src="../../images/patterns/diagrams/adapter/structure-object-adapter-indexedd6ac.png?id=a20b311948b361a058097e5bcdbf067a"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.76;"
srcset="/images/patterns/diagrams/adapter/structure-object-adapter-indexed.png?id=a20b311948b361a058097e5bcdbf067a 600w,/images/patterns/diagrams/adapter/structure-object-adapter-indexed-1.5x.png?id=72a1c8fde064c4915379fb34a1a66881 900w,/images/patterns/diagrams/adapter/structure-object-adapter-indexed-2x.png?id=759771515f08d74d53cf4fe500f814a3 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Structure du patron de conception adaptateur (adaptateur d’objets/object adapter)" />
</figure>
:::
::::

1.  Le **Client** est une classe qui contient la logique métier du
    programme.

2.  L'**Interface Client** décrit un protocole que les autres classes
    doivent implémenter afin de pouvoir collaborer avec le code client.

3.  Le **Service** représente une classe que l'on veut utiliser (souvent
    une application externe ou héritée). Le client ne peut pas
    l'utiliser directement, car son interface n'est pas compatible.

4.  L'**Adaptateur** est une classe qui peut interagir à la fois avec le
    client et le service : il implémente l'interface client et encapsule
    l'objet service. L'adaptateur reçoit des appels du client via
    l'interface client et les convertit en appels à l'objet du service
    encapsulé, dans un format qu'il peut gérer.

5.  Le code client n'est pas couplé avec la classe de l'adaptateur
    concret tant qu'il se contente d'utiliser l'interface du client.
    Grâce à cela, vous pouvez ajouter de nouveaux types d'adaptateurs
    dans le programme sans modifier le code client existant. Ce
    fonctionnement se révèle très pratique si l'interface d'une classe
    d'un service est modifiée ou remplacée : créez juste une nouvelle
    classe adaptateur sans toucher au code client.

#### Adaptateur de classe

Cette implémentation utilise l'héritage : l'adaptateur hérite de
l'interface des deux objets en même temps. Vous remarquerez que cette
approche ne peut être mise en place que si le langage de programmation
gère l'héritage multiple, comme le C++.

:::: adapter-class
::: {.struct-image2 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/adapter/structure-class-adapterf043.png?id=e1c60240508146ed3b98ac562cc8e510"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.72;"
srcset="/images/patterns/diagrams/adapter/structure-class-adapter.png?id=e1c60240508146ed3b98ac562cc8e510 550w,/images/patterns/diagrams/adapter/structure-class-adapter-1.5x.png?id=299a79bdfa10ac53213ba02408255430 825w,/images/patterns/diagrams/adapter/structure-class-adapter-2x.png?id=ddca3e3e4d972b7c58207daba8d24866 1100w"
sizes="(max-width: 720px) 100vw, 550px" loading="lazy" width="550"
alt="Patron de conception adaptateur (adaptateur de classe/class adapter)" /><img
src="../../images/patterns/diagrams/adapter/structure-class-adapter-indexedd30e.png?id=250b5c485a7dfba7c16b89a9201538fb"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.72;"
srcset="/images/patterns/diagrams/adapter/structure-class-adapter-indexed.png?id=250b5c485a7dfba7c16b89a9201538fb 550w,/images/patterns/diagrams/adapter/structure-class-adapter-indexed-1.5x.png?id=b56d736f8076d34b1896de0a2b22a219 825w,/images/patterns/diagrams/adapter/structure-class-adapter-indexed-2x.png?id=9ae1182ef2a34d2ea65f4b4f94a4019e 1100w"
sizes="(max-width: 720px) 100vw, 550px" loading="lazy" width="550"
alt="Patron de conception adaptateur (adaptateur de classe/class adapter)" />
</figure>
:::
::::

1.  L'**Adaptateur de Classe** n'a pas besoin d'encapsuler des objets,
    car il hérite des comportements du client et du service. La totalité
    de l'adaptation se déroule à l'intérieur des méthodes redéfinies.
    Cet adaptateur peut remplacer une classe existante du client.
:::::::
::::::::

::: {.section .pseudocode}
##  Pseudo-code {#pseudocode}

Voici un exemple d'utilisation du patron de conception **Adaptateur**
qui résout le problème classique de la pièce carrée à insérer dans le
trou rond.

<figure class="image">
<img
src="../../images/patterns/diagrams/adapter/examplef4c9.png?id=9d2b6857ce256f2c669383ce4df3d0aa"
style="aspect-ratio: 1.68;"
srcset="/images/patterns/diagrams/adapter/example.png?id=9d2b6857ce256f2c669383ce4df3d0aa 640w,/images/patterns/diagrams/adapter/example-1.5x.png?id=76e68b9cea3b371e465e81587e57e5d9 960w,/images/patterns/diagrams/adapter/example-2x.png?id=0ac62d1bc151e8ce6abad8e8502756cf 1280w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="640"
alt="La structure de l’adaptateur dans l’exemple utilisé" />
<figcaption><p>Adapter des pièces carrées avec des
trous ronds.</p></figcaption>
</figure>

L'adaptateur se fait passer pour un cylindre, avec un rayon égal à la
moitié de la diagonale du carré (en d'autres termes, le rayon minimal du
trou pour accueillir la pièce carrée).

<figure class="code">
<pre class="code" lang="pseudocode"><code>// Prenons deux classes avec des interfaces compatibles :
// TrouRond (RoundHole) et PièceRonde (RoundPeg).
class RoundHole is
    constructor RoundHole(radius) { ... }

    method getRadius() is
        // Retourne le rayon du trou.

    method fits(peg: RoundPeg) is
        return this.getRadius() &gt;= peg.radius()

class RoundPeg is
    constructor RoundPeg(radius) { ... }

    method getRadius() is
        // Retourne le rayon de la pièce.


// Mais une classe est incompatible : PièceCarrée (SquarePeg).
class SquarePeg is
    constructor SquarePeg(width) { ... }

    method getWidth() is
        // Retourne la largeur de la pièce carrée.


// Une classe adaptateur permet de faire rentrer des pièces
// carrées dans des trous ronds. Elle étend la classe pièceRonde
// pour permettre aux objets adaptateur de se comporter comme
// des pièces rondes.
class SquarePegAdapter extends RoundPeg is
    // En réalité, l’adaptateur contient une instance de la
    // classe pièceCarrée.
    private field peg: SquarePeg

    constructor SquarePegAdapter(peg: SquarePeg) is
        this.peg = peg

    method getRadius() is
        // L’adaptateur se fait passer pour une pièce ronde avec
        // un rayon qui pourrait faire rentrer la pièce carrée
        // emballée par l’adaptateur.
        return peg.getWidth() * Math.sqrt(2) / 2


// Quelque part dans le code client.
hole = new RoundHole(5)
rpeg = new RoundPeg(5)
hole.fits(rpeg) // true

small_sqpeg = new SquarePeg(5)
large_sqpeg = new SquarePeg(10)
hole.fits(small_sqpeg) // Ça ne compilera pas (types incompatibles).

small_sqpeg_adapter = new SquarePegAdapter(small_sqpeg)
large_sqpeg_adapter = new SquarePegAdapter(large_sqpeg)
hole.fits(small_sqpeg_adapter) // true
hole.fits(large_sqpeg_adapter) // false</code></pre>
</figure>
:::

:::::::: {.section .applicability-container}
##  Possibilités d'application {#applicability}

::::::: applicability
::: applicability-problem
Utilisez l'adaptateur de classe si vous avez besoin d'une classe
existante, mais que son interface est incompatible avec votre code.
:::

::: applicability-solution
L'adaptateur permet de créer une classe faisant office de couche
intermédiaire. Cette couche sert de convertisseur entre votre code et
une classe héritée ou externe, ou n'importe quelle classe avec une
interface incongrue.
:::

::: applicability-problem
Mettez en place l'adaptateur si vous désirez réutiliser plusieurs
sous-classes existantes à qui il manque des fonctionnalités communes qui
ne peuvent pas être remontées dans la classe mère.
:::

::: applicability-solution
Vous pourriez étendre chaque sous-classe et mettre la fonctionnalité
manquante dans les nouvelles sous-classes. En revanche, vous allez
devoir dupliquer le code dans ces nouvelles classes, ce qui [n'est pas
terrible](../smells/duplicate-code.html).

Pour une solution un peu plus élégante, vous pouvez mettre la
fonctionnalité manquante dans une classe adaptateur. Ensuite, encapsulez
les objets avec les fonctionnalités manquantes à l'intérieur de
l'adaptateur, les rendant disponibles dynamiquement. Pour que cela
fonctionne, les classes ciblées doivent implémenter une interface
commune, et l'attribut de l'adaptateur doit implémenter cette interface.
Cette solution se rapproche du patron de conception
[Décorateur](decorator.html).
:::
:::::::
::::::::

::: {.section .checklist}
##  Mise en œuvre {#checklist}

1.  Assurez-vous d'avoir au moins deux classes avec des interfaces
    incompatibles :

    -   Une classe *service* dont vous voulez vous servir, mais que vous
        ne pouvez pas modifier (application externe, héritée ou dotée
        d'un grand nombre de dépendances).
    -   Une ou plusieurs classes *client* qui pourraient bénéficier de
        l'utilisation de la classe service.

2.  Déclarez l'interface client et décrivez la manière dont les clients
    vont communiquer avec le service.

3.  Créez la classe adaptateur et faites-la implémenter l'interface
    client. Laissez les méthodes vides pour le moment.

4.  Ajoutez un attribut à la classe adaptateur pour y mettre une
    référence vers l'objet service. En général on initialise cet
    attribut à l'aide du constructeur, mais il est parfois plus pratique
    de l'envoyer à l'adaptateur au moment de l'appel de ses méthodes.

5.  Implémentez toutes les méthodes de l'interface client une par une
    dans la classe adaptateur. L'adaptateur doit déléguer le gros du
    travail à l'objet service et ne s'occuper que de l'interface ou de
    la conversion du format des données.

6.  Les clients doivent utiliser l'adaptateur en passant par l'interface
    client. Vous pouvez ainsi modifier ou étendre les adaptateurs sans
    toucher au code client.
:::

:::::: {.section .pros-cons}
##  Avantages et inconvénients {#pros-cons}

::::: row
::: col-sm-6
-    *Principe de responsabilité unique*. Vous découplez l'interface ou
    le code de conversion des données, de la logique métier du
    programme.
-    *Principe ouvert/fermé*. Vous pouvez ajouter de nouveaux types
    d'adaptateurs dans le programme sans modifier le code client
    existant. Ces adaptateurs doivent forcément passer par l'interface
    du client.
:::

::: col-sm-6
-    La complexité générale du code augmente, car vous devez créer un
    ensemble de nouvelles classes et interfaces. Parfois, il est plus
    simple de modifier la classe du service afin de la faire
    correspondre avec votre code.
:::
:::::
::::::

::: {.section .relations}
##  Liens avec les autres patrons {#relations}

-   L\'[Adaptateur](adapter.html) fournit une interface complètement
    différente pour accéder à un objet existant. En revanche, avec le
    modèle du [Décorateur](decorator.html), l'interface reste la même ou
    est étendue. De plus, *Décorateur* supporte la composition
    récursive, ce qui n'est pas possible lorsque vous utilisez
    *Adaptateur*.

-   Avec [Adaptateur](adapter.html), vous accédez à un objet existant
    via une interface différente. Avec [Procuration](proxy.html),
    l'interface reste la même. Avec [Décorateur](decorator.html), vous
    accédez à l'objet via une interface améliorée.

-   La [Façade](facade.html) définit une nouvelle interface pour les
    objets existants, alors que l'[Adaptateur](adapter.html) essaye de
    rendre l'interface existante utilisable. L'*adaptateur* emballe
    généralement un seul objet alors que la *façade* s'utilise pour un
    sous-système complet d'objets.

-   Le [Pont](bridge.html), l'[État](state.html), la
    [Stratégie](strategy.html) (et dans une certaine mesure
    l'[Adaptateur](adapter.html)) ont des structures très similaires. En
    effet, ces patrons sont basés sur la composition, qui délègue les
    tâches aux autres objets. Cependant, ils résolvent différents
    problèmes. Un patron n'est pas juste une recette qui vous aide à
    structurer votre code d'une certaine manière. C'est aussi une façon
    de communiquer aux autres développeurs le problème qu'il résout.
:::

::: {.section .implementations}
##  Exemples de code {#implementations}

[![Adaptateur en
C#](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](adapter/csharp/example.html "Adaptateur en C#"){.prog-lang-link}
[![Adaptateur en
C++](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](adapter/cpp/example.html "Adaptateur en C++"){.prog-lang-link}
[![Adaptateur en
Go](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](adapter/go/example.html "Adaptateur en Go"){.prog-lang-link}
[![Adaptateur en
Java](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](adapter/java/example.html "Adaptateur en Java"){.prog-lang-link}
[![Adaptateur en
PHP](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](adapter/php/example.html "Adaptateur en PHP"){.prog-lang-link}
[![Adaptateur en
Python](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](adapter/python/example.html "Adaptateur en Python"){.prog-lang-link}
[![Adaptateur en
Ruby](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](adapter/ruby/example.html "Adaptateur en Ruby"){.prog-lang-link}
[![Adaptateur en
Rust](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](adapter/rust/example.html "Adaptateur en Rust"){.prog-lang-link}
[![Adaptateur en
Swift](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](adapter/swift/example.html "Adaptateur en Swift"){.prog-lang-link}
[![Adaptateur en
TypeScript](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](adapter/typescript/example.html "Adaptateur en TypeScript"){.prog-lang-link}
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

[Pont []{.fa .fa-arrow-right}](bridge.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Retour

[[]{.fa .fa-arrow-left} Patrons
structurels](structural-patterns.html){.btn .btn-default rel="prev"}
:::
:::::::::::::::::::::::::::::::::::

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
:::::::::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::::::::
