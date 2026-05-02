::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [Patrons de
conception](../design-patterns.html){.type} / [Patrons
structurels](structural-patterns.html){.type}
:::

# Décorateur {#décorateur .title}

::: aka
Alias :
[Emballeur, ]{style="display:inline-block"}[Empaqueteur, ]{style="display:inline-block"}[Wrapper, ]{style="display:inline-block"}[Decorator]{style="display:inline-block"}
:::

::: {.section .intent}
##  Intention {#intent}

**Décorateur** est un patron de conception structurel qui permet
d'affecter dynamiquement de nouveaux comportements à des objets en les
plaçant dans des emballeurs qui implémentent ces comportements.

<figure class="image">
<img
src="../../images/patterns/content/decorator/decoratore8f5.png?id=710c66670c7123e0928d3b3758aea79e"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/decorator/decorator.png?id=710c66670c7123e0928d3b3758aea79e 640w,/images/patterns/content/decorator/decorator-1.5x.png?id=72aafc603a289fc64e028e83e8aa843b 960w,/images/patterns/content/decorator/decorator-2x.png?id=736ab07b1d8920ab2c7a70c9cb1305cc 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Patron de conception décorateur" />
</figure>
:::

::: {.section .problem}
##  Problème {#problem}

Imaginez que vous travaillez sur une librairie qui permet aux programmes
d'envoyer des notifications à leurs utilisateurs lorsque des événements
importants se produisent.

La version initiale de la librairie était basée sur la classe
`Notificateur` qui n'avait que quelques attributs, un constructeur et
une unique méthode `envoyer`. La méthode pouvait prendre un message en
paramètre et l'envoyait à une liste d'e-mails à l'aide du constructeur
du notificateur. Une application externe qui jouait le rôle du client
devait créer et configurer l'objet notificateur une première fois, puis
l'utiliser lorsqu'un événement important se produisait.

<figure class="image">
<img
src="../../images/patterns/diagrams/decorator/problem1-fr3171.png?id=d671be64982a2c9fcb92ae7c666e1406"
style="aspect-ratio: 2.45;"
srcset="/images/patterns/diagrams/decorator/problem1-fr.png?id=d671be64982a2c9fcb92ae7c666e1406 540w,/images/patterns/diagrams/decorator/problem1-fr-1.5x.png?id=4feea6f3692c5511abd6da6fd92f6664 810w,/images/patterns/diagrams/decorator/problem1-fr-2x.png?id=61034df029b25057742f4fd8875e618c 1080w"
sizes="(max-width: 720px) 100vw, 540px" loading="lazy" width="540"
alt="Structure de la librairie avant l’utilisation du patron de conception décorateur" />
<figcaption><p>Un programme peut utiliser la classe notificateur afin
d’envoyer des notifications au sujet d’événements importants à une liste
prédéfinie d’adresses e-mail.</p></figcaption>
</figure>

Au bout d'un certain temps, vous vous rendez compte que les utilisateurs
de la librairie veulent plus que les notifications qu'ils reçoivent sur
leur boîte mail. Ils sont nombreux à vouloir recevoir des SMS lorsque
leurs applications rencontrent des problèmes critiques. Certains
voudraient être prévenus sur Facebook et les employés de certaines
entreprises adoreraient recevoir des notifications Slack.

<figure class="image">
<img
src="../../images/patterns/diagrams/decorator/problem2a2f8.png?id=ba5d5e106ea8c4848d60e230feca9135"
style="aspect-ratio: 2.59;"
srcset="/images/patterns/diagrams/decorator/problem2.png?id=ba5d5e106ea8c4848d60e230feca9135 440w,/images/patterns/diagrams/decorator/problem2-1.5x.png?id=4f15b6208533188dd09e7da7dd6b509a 660w,/images/patterns/diagrams/decorator/problem2-2x.png?id=28b2c8509b4e78c031d728424b876ebc 880w"
sizes="(max-width: 720px) 100vw, 440px" loading="lazy" width="440"
alt="Structure de la librairie après avoir ajouté d’autres types de notifications" />
<figcaption><p>Chaque type de notification est implémenté dans une
sous-classe du notificateur.</p></figcaption>
</figure>

Cela n'a pas l'air bien difficile ! Vous avez étendu la classe
`Notificateur` et ajouté des méthodes de notification supplémentaires
dans de nouvelles sous-classes.

Le client devait instancier la classe de notification désirée et
utiliser cette instance pour toutes les autres notifications, mais
quelqu'un a remis en question ce fonctionnement en vous affirmant qu'il
aimerait bien utiliser plusieurs types de notifications simultanément.
Si votre maison prend feu, vous avez très certainement envie d'en être
informé par tous les moyens possibles.

Vous avez tenté de résoudre ce problème en créant des sous-classes
spéciales qui combinent plusieurs méthodes de notification dans une
seule classe. Mais il s'avère que cette approche va non seulement
gonfler le code de la librairie, mais aussi celui du client.

<figure class="image">
<img
src="../../images/patterns/diagrams/decorator/problem34033.png?id=f3b3e7a107d870871f2c3167adcb7ccb"
style="aspect-ratio: 1.85;"
srcset="/images/patterns/diagrams/decorator/problem3.png?id=f3b3e7a107d870871f2c3167adcb7ccb 630w,/images/patterns/diagrams/decorator/problem3-1.5x.png?id=f4ef9d367df838c77121e9f84260b09c 945w,/images/patterns/diagrams/decorator/problem3-2x.png?id=abb7a87b521ce97d7661dd9c0b988cc3 1260w"
sizes="(max-width: 720px) 100vw, 630px" loading="lazy" width="630"
alt="Structure de la librairie en créant toutes les combinaisons de classes" />
<figcaption><p>Explosion combinatoire des sous-classes.</p></figcaption>
</figure>

Vous devez structurer vos classes de notification différemment pour ne
pas trop les multiplier, sinon vous allez vous retrouver dans le livre
Guinness des records.
:::

::: {.section .solution}
##  Solution

La première chose qui nous vient à l'esprit lorsque l'on veut modifier
le comportement d'un objet, c'est d'étendre sa classe. Cependant, voici
quelques mises en garde concernant l'héritage :

-   L'héritage est statique. Vous ne pouvez pas modifier le comportement
    d'un objet au moment de l'exécution. Vous ne pouvez que remplacer la
    totalité de l'objet par un autre, généré grâce à une sous-classe
    différente.
-   Les sous-classes ne peuvent avoir qu'un seul parent. Dans la
    majorité des langages de programmation, vous ne pouvez hériter que
    d'une seule classe à la fois.

Une des solutions pour contourner ce problème consiste à utiliser
l'*aggrégation* ou la *composition* []{.pop-i}[*Aggrégation* : l'objet A
contient les objets B ; B peut exister sans A.\
*Composition* : l'objet A est composé d'objets B ; A gère le cycle de
vie de B ; B ne peut pas exister sans A.]{.pop-content} à la place de
l'*héritage*. Ces alternatives fonctionnent à peu près de la même
façon : un objet *possède* une référence à un autre objet et lui délègue
une partie de son travail. Avec l'héritage, l'objet *est* capable de
faire le travail tout seul en héritant du comportement de sa classe
mère.

Grâce à cette nouvelle approche, il est facile de remplacer l'objet
référencé par un autre, ce qui modifie le comportement du conteneur au
moment de l'exécution. Un objet peut utiliser le comportement de
diverses classes, posséder des références à de nombreux objets et leur
déléguer toutes sortes de tâches. L'agrégation et la composition sont
des principes clés dans le décorateur, tout comme dans de nombreux
autres patrons. Sur ces belles paroles, revenons à nos moutons.

<figure class="image">
<img
src="../../images/patterns/diagrams/decorator/solution1-frf5d3.png?id=5749683d05e81e4d6c6a402db20bfd4c"
style="aspect-ratio: 3.44;"
srcset="/images/patterns/diagrams/decorator/solution1-fr.png?id=5749683d05e81e4d6c6a402db20bfd4c 550w,/images/patterns/diagrams/decorator/solution1-fr-1.5x.png?id=4e9928e9171c85f9cffad79b695701e0 825w,/images/patterns/diagrams/decorator/solution1-fr-2x.png?id=999e2e44f2cc3e019c7e11ed3ff8adbe 1100w"
sizes="(max-width: 720px) 100vw, 550px" loading="lazy" width="550"
alt="Héritage contre Agrégation" />
<figcaption><p>Héritage contre Agrégation.</p></figcaption>
</figure>

Le décorateur est également appelé « emballeur » ou « empaqueteur ». Ces
surnoms révèlent l'idée générale derrière le concept. Un *emballeur* est
un objet qui peut être lié par un objet *cible*. L'emballeur possède le
même ensemble de méthodes que la cible et lui délègue toutes les
demandes qu'il reçoit. Il peut exécuter un traitement et modifier le
résultat avant ou après avoir envoyé sa demande à la cible.

À quel moment un emballeur devient-il réellement un décorateur ? Comme
je l'ai déjà dit, un emballeur implémente la même interface que l'objet
emballé. Du point de vue du client, ces objets sont identiques.
L'attribut de la référence de l'emballeur doit pouvoir accueillir
n'importe quel objet qui implémente cette interface. Vous pouvez ainsi
utiliser plusieurs emballeurs sur un seul objet et lui attribuer les
comportements de plusieurs emballeurs en même temps.

Reprenons notre exemple et mettons-y une notification par e-mail dans la
classe de base `Notificateur`, mais transformons toutes les autres
méthodes de notification en décorateurs.

<figure class="image">
<img
src="../../images/patterns/diagrams/decorator/solution2e4ab.png?id=cbee4a27080ce3a0bf773482613e1347"
style="aspect-ratio: 1.39;"
srcset="/images/patterns/diagrams/decorator/solution2.png?id=cbee4a27080ce3a0bf773482613e1347 640w,/images/patterns/diagrams/decorator/solution2-1.5x.png?id=d2fa0c0b9bf5cba0e38be7aff7e7b7a8 960w,/images/patterns/diagrams/decorator/solution2-2x.png?id=7775f76b94dbd5cd25f711ce81f59262 1280w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="640"
alt="La solution avec utilisation du patron de conception décorateur" />
<figcaption><p>Des méthodes de notification deviennent
des décorateurs.</p></figcaption>
</figure>

Le code client doit emballer un objet basique Notificateur pour le
transformer en un ensemble de décorateurs adapté aux préférences du
client. Les objets qui en résultent sont empilés.

<figure class="image">
<img
src="../../images/patterns/diagrams/decorator/solution3-fr97df.png?id=f79483d9b36637049cf510441087e9c6"
style="aspect-ratio: 1.03;"
srcset="/images/patterns/diagrams/decorator/solution3-fr.png?id=f79483d9b36637049cf510441087e9c6 300w,/images/patterns/diagrams/decorator/solution3-fr-1.5x.png?id=9991a0aa3c75ae1c07bd16a3609e629f 450w,/images/patterns/diagrams/decorator/solution3-fr-2x.png?id=d749cd6187ac0e2d36f338553eb947aa 600w"
sizes="(max-width: 720px) 100vw, 300px" loading="lazy" width="300"
alt="Les applications peuvent configurer des piles complexes de décorateurs pour les notifications" />
<figcaption><p>Les applications peuvent configurer des piles complexes
de décorateurs pour les notifications.</p></figcaption>
</figure>

Le client traite le dernier objet de la pile. Les décorateurs
implémentent tous la même interface (le notificateur de base). Le reste
du code client manipule indifféremment l'objet notificateur original et
l'objet décoré.

Nous pouvons utiliser cette technique pour tous les autres
comportements, comme la mise en page des messages ou la création de la
liste des destinataires. Le client peut décorer l'objet avec des
décorateurs personnalisés tant qu'ils suivent la même interface que les
autres.
:::

::: {.section .analogy}
##  Analogie {#analogy}

<figure class="image">
<img
src="../../images/patterns/content/decorator/decorator-comic-1cbff.png?id=80d95baacbfb91f5bcdbdc7814b0c64d"
style="aspect-ratio: 2;"
srcset="/images/patterns/content/decorator/decorator-comic-1.png?id=80d95baacbfb91f5bcdbdc7814b0c64d 600w,/images/patterns/content/decorator/decorator-comic-1-1.5x.png?id=490ef9754d7554c2046b69f6f213c6da 900w,/images/patterns/content/decorator/decorator-comic-1-2x.png?id=ba869f621b6e0ea173fdc2b535fc7eed 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Exemple utilisé pour le patron de conception décorateur" />
<figcaption><p>Les effets se cumulent si vous portez plusieurs couches
de vêtements.</p></figcaption>
</figure>

Porter des vêtements est un bon exemple d'utilisation. Si vous avez
froid, vous vous enroulez dans un pull. Si vous avez encore froid, vous
pouvez porter un blouson par-dessus. S'il pleut, vous enfilez un
imperméable. Tous ces vêtements « étendent » votre comportement de base
mais ne font pas partie de vous, et vous pouvez facilement enlever un
vêtement lorsque vous n'en avez plus besoin.
:::

::::: {.section .structure-container}
##  Structure

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/decorator/structure128f.png?id=8c95d894aecce5315cc1b12093a7ea0c"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 0.92;"
srcset="/images/patterns/diagrams/decorator/structure.png?id=8c95d894aecce5315cc1b12093a7ea0c 480w,/images/patterns/diagrams/decorator/structure-1.5x.png?id=4b2cd91d4483d9e3bba725f0e45d281d 720w,/images/patterns/diagrams/decorator/structure-2x.png?id=3cfa1f10417a4ef0c12580bc4a63b80d 960w"
sizes="(max-width: 720px) 100vw, 480px" loading="lazy" width="480"
alt="Structure du patron de conception décorateur" /><img
src="../../images/patterns/diagrams/decorator/structure-indexeda93c.png?id=09401b230a58f2249e4c9a1195d485a0"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 0.98;"
srcset="/images/patterns/diagrams/decorator/structure-indexed.png?id=09401b230a58f2249e4c9a1195d485a0 500w,/images/patterns/diagrams/decorator/structure-indexed-1.5x.png?id=dccc54182965078ccb4cfdeee41acbe5 750w,/images/patterns/diagrams/decorator/structure-indexed-2x.png?id=2733e7d0e322bfb2f150ccf8a878dbf6 1000w"
sizes="(max-width: 720px) 100vw, 500px" loading="lazy" width="500"
alt="Structure du patron de conception décorateur" />
</figure>
:::

1.  Le **Composant** déclare l'interface commune pour les décorateurs et
    les objets décorés.

2.  Le **Composant Concret** est une classe contenant des objets qui
    vont être emballés. Il définit le comportement par défaut qui peut
    être modifié par les décorateurs.

3.  Le **Décorateur de Base** possède un attribut pour référencer un
    objet emballé. L'attribut doit être déclaré avec le type de
    l'interface du composant afin de contenir à la fois les composants
    concrets et les décorateurs. Le décorateur de base délègue toutes
    les opérations à l'objet emballé.

4.  Les **Décorateurs Concrets** définissent des comportements
    supplémentaires qui peuvent être ajoutés dynamiquement aux
    composants. Les décorateurs concrets redéfinissent les méthodes du
    décorateur de base et exécutent leur traitement avant ou après
    l'appel à la méthode du parent.

5.  Le **Client** peut emballer les composants dans plusieurs couches de
    décorateurs, tant qu'il manipule les objets à l'aide de l'interface
    du composant.
::::
:::::

::: {.section .pseudocode}
##  Pseudo-code {#pseudocode}

Dans cet exemple, le **Décorateur** permet la compression et le
chiffrage des données indépendamment du code qui les utilise.

<figure class="image">
<img
src="../../images/patterns/diagrams/decorator/example42cb.png?id=eec9dc488f00c85f50e764323baa723e"
style="aspect-ratio: 0.98;"
srcset="/images/patterns/diagrams/decorator/example.png?id=eec9dc488f00c85f50e764323baa723e 540w,/images/patterns/diagrams/decorator/example-1.5x.png?id=40ccaba4f5a8e6a2ebac697f04b9f10a 810w,/images/patterns/diagrams/decorator/example-2x.png?id=4891323a27d5601a174eec366187d833 1080w"
sizes="(max-width: 720px) 100vw, 540px" loading="lazy" width="540"
alt="La structure du patron de conception décorateur dans l’exemple utilisé" />
<figcaption><p>L’exemple de chiffrage et de
compression utilisé.</p></figcaption>
</figure>

L'application emballe l'objet de la source de données à l'aide de deux
décorateurs. Ils modifient la manière dont les données sont lues et
écrites sur le disque :

-   Les décorateurs chiffrent et compressent les données juste avant
    qu'elles soient **écrites sur le disque**. La classe d'origine écrit
    les données chiffrées et protégées dans le fichier sans savoir qu'il
    y a eu une modification.

-   Une fois que les données sont **lues depuis le disque**, elles
    repassent dans ces mêmes décorateurs qui les décompressent et les
    déchiffrent.

Les décorateurs et la classe de la source de données implémentent la
même interface, ce qui les rend interchangeables aux yeux du code
client.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// L’interface du composant définit les traitements qui peuvent
// être modifiés par les décorateurs.
interface DataSource is
    method writeData(data)
    method readData():data

// Les composants concrets fournissent des implémentations par
// défaut pour les traitements. Plusieurs variantes de ces
// classes peuvent se trouver dans un programme.
class FileDataSource implements DataSource is
    constructor FileDataSource(filename) { ... }

    method writeData(data) is
        // Écrit les données dans le fichier.

    method readData():data is
        // Lit les données du fichier.

// La classe de base du décorateur implémente la même interface
// que les autres composants. Le but principal de cette classe
// est de définir l’interface d’emballage pour tous les
// décorateurs concrets. L’implémentation par défaut du code
// d’emballage peut comprendre un attribut pour stocker un
// composant emballé et tout le nécessaire pour l’initialiser.
class DataSourceDecorator implements DataSource is
    protected field wrappee: DataSource

    constructor DataSourceDecorator(source: DataSource) is
        wrappee = source

    // Le décorateur de base délègue tout le travail à l’objet
    // emballé. Les comportements supplémentaires peuvent être
    // ajoutés aux décorateurs concrets.
    method writeData(data) is
        wrappee.writeData(data)

    // Les décorateurs concrets peuvent appeler l’implémentation
    // parent du traitement plutôt que d’appeler l’objet emballé
    // directement. Cette approche simplifie l’extension des
    // classes décorateur.
    method readData():data is
        return wrappee.readData()

// Les décorateurs concrets doivent appeler les méthodes de
// l’objet emballé, mais ils peuvent ajouter quelque chose au
// résultat. Les décorateurs peuvent exécuter ce comportement
// supplémentaire avant ou après l’appel à l’objet emballé.
class EncryptionDecorator extends DataSourceDecorator is
    method writeData(data) is
        // 1. Chiffre les données passées.
        // 2. Envoie les données chiffrées à la méthode emballée
        // écrireDonnées

    method readData():data is
        // 1. Récupère les données de la méthode emballée
        // lireDonnées.
        // 2. La déchiffre si besoin.
        // 3. Retourne le résultat.

// Vous pouvez emballer les objets dans plusieurs couches de
// décorateurs.
class CompressionDecorator extends DataSourceDecorator is
    method writeData(data) is
        // 1. Compresse les données passées.
        // 2. Envoie les données compressées à la méthode
        // emballée écrireDonnées

    method readData():data is
        // 1. Récupère les données de la méthode emballée
        // lireDonnées.
        // 2. La décompresse si besoin.
        // 3. Retourne le résultat.


// Option 1 : un exemple simple de l’assemblage d’un décorateur.
class Application is
    method dumbUsageExample() is
        source = new FileDataSource(&quot;somefile.dat&quot;)
        source.writeData(salaryRecords)
        // Le fichier cible a été écrit avec des données brutes.

        source = new CompressionDecorator(source)
        source.writeData(salaryRecords)
        // Le fichier cible a été écrit avec des données
        // compressées.

        source = new EncryptionDecorator(source)
        // La variable source contient à présent :
        // Chiffrage &gt; Compression &gt; FileDataSource.
        source.writeData(salaryRecords)
        // Le fichier cible a été écrit avec des données
        // compressées et chiffrées.


// Option 2 : un code client qui utilise une source de données
// externe. Les objets responsablePaye (SalaryManager) ne
// s’encombrent pas des détails concernant le stockage des
// données. Ils utilisent une source de données pré configurée
// reçue par le configurateur de l’application.
class SalaryManager is
    field source: DataSource

    constructor SalaryManager(source: DataSource) { ... }

    method load() is
        return source.readData()

    method save() is
        source.writeData(salaryRecords)
    // ...d’autres méthodes utiles...


// L’application peut assembler différentes piles de décorateurs
// à l’exécution, en fonction de la configuration ou de
// l’environnement.
class ApplicationConfigurator is
    method configurationExample() is
        source = new FileDataSource(&quot;salary.dat&quot;)
        if (enabledEncryption)
            source = new EncryptionDecorator(source)
        if (enabledCompression)
            source = new CompressionDecorator(source)

        logger = new SalaryManager(source)
        salary = logger.load()
    // ...</code></pre>
</figure>
:::

:::::::: {.section .applicability-container}
##  Possibilités d'application {#applicability}

::::::: applicability
::: applicability-problem
Utilisez le décorateur si vous avez besoin d'ajouter des comportements
supplémentaires au moment de l'exécution sans avoir à altérer le code
source de ces objets.
:::

::: applicability-solution
Le décorateur vous permet de structurer votre logique métier en couches,
de créer un décorateur pour chacune de ces couches et de décorer les
objets avec différentes combinaisons au moment de l'exécution. Le code
client peut traiter les objets uniformément puisqu'ils implémentent la
même interface.
:::

::: applicability-problem
Utilisez ce patron si l'héritage est impossible ou peu logique pour
étendre le comportement d'un objet.
:::

::: applicability-solution
De nombreux langages de programmation permettent l'utilisation du mot
clé `final` pour interdire l'héritage d'une classe. Le seul moyen
d'étendre le comportement d'une telle classe est de l'emballer en
utilisant un décorateur.
:::
:::::::
::::::::

::: {.section .checklist}
##  Mise en œuvre {#checklist}

1.  Assurez-vous que votre domaine peut être représenté sous la forme
    d'un composant principal recouvert par plusieurs couches
    facultatives.

2.  Déterminez les méthodes communes entre le composant principal et les
    couches facultatives. Créez l'interface du composant et déclarez-y
    ces méthodes.

3.  Créez une classe concrète pour le composant et définissez son
    comportement de base.

4.  Créez une classe de base décorateur. Elle doit inclure un attribut
    qui va permettre de stocker la référence à un objet emballé. Cet
    attribut doit être déclaré avec le type de l'interface du composant,
    afin de le relier aux composants concrets et aux décorateurs. Le
    décorateur de base doit déléguer tout le travail à l'objet emballé.

5.  Assurez-vous que les classes implémentent l'interface du composant.

6.  Créez des décorateurs concrets en les implémentant à partir du
    décorateur de base. Un décorateur concret doit exécuter son
    comportement avant ou après l'appel à la méthode de son parent (qui
    délègue toujours la tâche à l'objet emballé).

7.  Le code client doit être responsable de la création des décorateurs
    et de leur agencement en fonction des besoins du client.
:::

:::::: {.section .pros-cons}
##  Avantages et inconvénients {#pros-cons}

::::: row
::: col-sm-6
-    Vous pouvez étendre le comportement d'un objet sans avoir recours à
    la création d'une nouvelle sous-classe.
-    Vous pouvez ajouter ou retirer dynamiquement des responsabilités à
    un objet au moment de l'exécution.
-    Vous pouvez combiner plusieurs comportements en emballant un objet
    dans plusieurs décorateurs.
-    *Principe de responsabilité unique*. Vous pouvez découper une
    classe monolithique qui implémente plusieurs comportements
    différents en plusieurs petits morceaux.
:::

::: col-sm-6
-    Retirer un emballeur spécifique de la pile n'est pas chose aisée.
-    Il n'est pas non plus aisé de mettre en place un décorateur dont le
    comportement ne varie pas en fonction de sa position dans la pile.
-    Le code de configuration initial des couches peut avoir l'air assez
    moche.
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

-   La [Chaîne de Responsabilité](chain-of-responsibility.html) et le
    [Décorateur](decorator.html) ont des structures de classes très
    similaires. Ils reposent tous deux sur la composition récursive pour
    passer les appels à une suite d'objets. Ils possèdent cependant
    plusieurs différences majeures.

    Les handlers de la *chaîne de responsabilité* peuvent lancer des
    opérations arbitraires indépendantes l'une de l'autre. Ils peuvent
    également décider de ne pas propager la demande plus loin dans la
    chaîne. Les *décorateurs* quant à eux peuvent étendre le
    comportement de l'objet sans perturber leur fonctionnement avec
    l'interface de base. De plus, les décorateurs n'ont pas le droit
    d'interrompre la demande.

-   Le [Composite](composite.html) et le [Décorateur](decorator.html)
    ont des diagrammes de structure similaires puisqu'ils reposent sur
    la composition récursive pour organiser un nombre variable d'objets.

    Un *décorateur* est comme un *composite*, mais avec un seul
    composant enfant. Il y a une autre différence importante : Le
    *décorateur* ajoute des responsabilités supplémentaires à l'objet
    emballé, alors que le *composite* se contente d'« additionner » les
    résultats de ses enfants.

    Mais ces patrons de conception peuvent également coopérer : vous
    pouvez utiliser le *décorateur* pour étendre le comportement d'un
    objet spécifique d'un arbre *Composite*.

-   Les conceptions qui reposent énormément sur le
    [Composite](composite.html) et le [Décorateur](decorator.html)
    tirent des avantages à utiliser le [Prototype](prototype.html). Il
    vous permet de cloner les structures complexes plutôt que de les
    reconstruire à partir de rien.

-   Le [Décorateur](decorator.html) vous permet de changer la peau d'un
    objet, alors que la [Stratégie](strategy.html) vous permet de
    changer ses tripes.

-   Le [Décorateur](decorator.html) et la [Procuration](proxy.html) ont
    des structures similaires, mais des intentions différentes. Ces deux
    patrons sont bâtis sur le principe de la composition, où un objet
    est censé déléguer certains traitements à un autre. La différence
    est qu'en principe, la *procuration* gère elle-même le cycle de vie
    de son objet service, alors que la composition des *décorateurs* est
    toujours contrôlée par le client.
:::

::: {.section .implementations}
##  Exemples de code {#implementations}

[![Décorateur en
C#](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](decorator/csharp/example.html "Décorateur en C#"){.prog-lang-link}
[![Décorateur en
C++](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](decorator/cpp/example.html "Décorateur en C++"){.prog-lang-link}
[![Décorateur en
Go](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](decorator/go/example.html "Décorateur en Go"){.prog-lang-link}
[![Décorateur en
Java](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](decorator/java/example.html "Décorateur en Java"){.prog-lang-link}
[![Décorateur en
PHP](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](decorator/php/example.html "Décorateur en PHP"){.prog-lang-link}
[![Décorateur en
Python](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](decorator/python/example.html "Décorateur en Python"){.prog-lang-link}
[![Décorateur en
Ruby](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](decorator/ruby/example.html "Décorateur en Ruby"){.prog-lang-link}
[![Décorateur en
Rust](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](decorator/rust/example.html "Décorateur en Rust"){.prog-lang-link}
[![Décorateur en
Swift](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](decorator/swift/example.html "Décorateur en Swift"){.prog-lang-link}
[![Décorateur en
TypeScript](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](decorator/typescript/example.html "Décorateur en TypeScript"){.prog-lang-link}
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

[Façade []{.fa .fa-arrow-right}](facade.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Retour

[[]{.fa .fa-arrow-left} Composite](composite.html){.btn .btn-default
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
