::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [Patrons de
conception](../design-patterns.html){.type} / [Patrons
comportementaux](behavioral-patterns.html){.type}
:::

# Médiateur {#médiateur .title}

::: aka
Alias :
[Intermédiaire, ]{style="display:inline-block"}[Contrôleur, ]{style="display:inline-block"}[Mediator]{style="display:inline-block"}
:::

::: {.section .intent}
##  Intention {#intent}

**Médiateur** est un patron de conception comportemental qui diminue les
dépendances chaotiques entre les objets. Il restreint les communications
directes entre les objets et les force à collaborer uniquement via un
objet médiateur.

<figure class="image">
<img
src="../../images/patterns/content/mediator/mediator6761.png?id=0264bd857a231b6ea2d0c537c092e698"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/mediator/mediator.png?id=0264bd857a231b6ea2d0c537c092e698 640w,/images/patterns/content/mediator/mediator-1.5x.png?id=b3d5df41892274b5c84282bae8737441 960w,/images/patterns/content/mediator/mediator-2x.png?id=250c2bf72ca1fdee2e6d97ed5a4765f2 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Patron de conception médiateur" />
</figure>
:::

::: {.section .problem}
##  Problème {#problem}

Prenons une boîte de dialogue qui crée et modifie des profils client.
Elle comporte plusieurs contrôles de formulaires comme des champs de
texte, des cases à cocher, des boutons, etc.

<figure class="image">
<img
src="../../images/patterns/diagrams/mediator/problem1-fr45f9.png?id=20526fe9b3cc46033b0ddf09e365c0d9"
style="aspect-ratio: 2.22;"
srcset="/images/patterns/diagrams/mediator/problem1-fr.png?id=20526fe9b3cc46033b0ddf09e365c0d9 600w,/images/patterns/diagrams/mediator/problem1-fr-1.5x.png?id=affed08d7976ac8c6328eebb1db431b6 900w,/images/patterns/diagrams/mediator/problem1-fr-2x.png?id=2f4df0303e02cf1587c56122968cea4a 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Liens chaotiques entre éléments de l’interface utilisateur" />
<figcaption><p>Les dépendances entre les éléments de l’interface
utilisateur peuvent devenir chaotiques avec l’évolution
de l’application.</p></figcaption>
</figure>

Certains éléments du formulaire peuvent interagir. Par exemple, en
cochant la case « J'ai un chien », vous allez révéler un champ de texte
caché qui vous permet d'écrire le nom du chien. Vous pourriez également
avoir un bouton Envoyer qui doit valider les valeurs de tous les champs
avant de sauvegarder les données.

<figure class="image">
<img
src="../../images/patterns/diagrams/mediator/problem2a7f7.png?id=072c51eee4dd90c0972866440c87c1c3"
style="aspect-ratio: 3.27;"
srcset="/images/patterns/diagrams/mediator/problem2.png?id=072c51eee4dd90c0972866440c87c1c3 360w,/images/patterns/diagrams/mediator/problem2-1.5x.png?id=b805abc211e60fa567fb114192e24608 540w,/images/patterns/diagrams/mediator/problem2-2x.png?id=d298ec82a47fa2938ed6cf64b7da78c1 720w"
sizes="(max-width: 720px) 100vw, 360px" loading="lazy" width="360"
alt="Les éléments de l’UI sont interdépendants" />
<figcaption><p>Les éléments peuvent avoir beaucoup de liens. De plus, la
modification de certains éléments peut influer
sur d’autres.</p></figcaption>
</figure>

En écrivant directement la logique dans le code des éléments du
formulaire, vous rendez les classes de ces éléments bien plus difficiles
à réutiliser dans l'application. Par exemple, vous ne pourrez pas
utiliser la classe de la case à cocher dans un autre formulaire, car
elle est couplée avec le champ de texte du chien. Vous êtes par
conséquent obligé d'utiliser toutes les classes du formulaire du profil
si vous voulez vous servir de l'une d'entre elles.
:::

::: {.section .solution}
##  Solution

Le patron de conception médiateur vous propose de mettre fin à toutes
les communications directes entre les composants et de rendre ces
derniers indépendants les uns des autres. À la place, ces composants
collaborent indirectement en utilisant un objet spécial médiateur qui
redirige les appels vers les composants appropriés. Ainsi, les
composants ne reposent que sur une seule classe médiateur plutôt que
d'être couplés à de nombreux collègues.

Dans notre exemple qui porte sur l'édition du formulaire d'un profil, la
classe dialogue peut prendre le rôle du médiateur. Elle connait déjà
probablement tous ses sous-éléments, vous n'avez donc pas besoin d'y
ajouter des dépendances.

<figure class="image">
<img
src="../../images/patterns/diagrams/mediator/solution1-fr9267.png?id=d9a0a6507df21bd10963ed95796958ef"
style="aspect-ratio: 2.22;"
srcset="/images/patterns/diagrams/mediator/solution1-fr.png?id=d9a0a6507df21bd10963ed95796958ef 600w,/images/patterns/diagrams/mediator/solution1-fr-1.5x.png?id=05bca4c7c2b062f450c9d3eca8f61a12 900w,/images/patterns/diagrams/mediator/solution1-fr-2x.png?id=7b3485f6578702e64dc5e23040fb563b 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Les éléments de l’UI doivent communiquer avec le médiateur" />
<figcaption><p>Les éléments de l’UI doivent communiquer directement avec
l’objet médiateur.</p></figcaption>
</figure>

Le changement le plus important intervient sur les éléments du
formulaire. Prenons le bouton Envoyer. Avant, chaque fois qu'un
utilisateur appuyait sur ce bouton, ce dernier devait valider les
valeurs des éléments individuels du formulaire. À présent, il se
contente d'informer la classe dialogue lorsqu'un clic a lieu. Après
avoir reçu cette notification, la classe dialogue effectue la validation
ou délègue la tâche aux différents éléments. Le bouton n'est ainsi
couplé qu'avec la classe dialogue, au lieu d'être relié à un paquet
d'éléments.

Vous pouvez pousser le vice plus loin et diminuer encore plus cette
dépendance en extrayant l'interface commune pour toutes ces boîtes de
dialogue. L'interface devrait déclarer la méthode de notification que
tous les éléments du formulaire utilisent pour prévenir la classe
dialogue des événements qui les affectent. Notre bouton Envoyer devrait
dorénavant pouvoir manipuler n'importe quelle boîte de dialogue qui
implémente cette interface.

Ainsi, le patron de conception médiateur vous permet d'encapsuler une
toile complexe de relations entre divers objets à l'intérieur d'un
simple objet médiateur. Moins une classe a de dépendances et plus il est
facile de la modifier, de l'étendre ou de la réutiliser.
:::

::: {.section .analogy}
##  Analogie {#analogy}

<figure class="image">
<img
src="../../images/patterns/diagrams/mediator/live-example3f2a.png?id=aa1de3cb7b63aa623e63578a1e43399a"
style="aspect-ratio: 1.85;"
srcset="/images/patterns/diagrams/mediator/live-example.png?id=aa1de3cb7b63aa623e63578a1e43399a 370w,/images/patterns/diagrams/mediator/live-example-1.5x.png?id=315109aa1099cc6e7c06057fa139881c 555w,/images/patterns/diagrams/mediator/live-example-2x.png?id=fd55a9f9d8cf49effa223555c7369504 740w"
sizes="(max-width: 720px) 100vw, 370px" loading="lazy" width="370"
alt="Tour de contrôle du trafic aérien" />
<figcaption><p>Les pilotes d’avion ne communiquent pas directement
ensemble pour déterminer qui sera le prochain à atterrir. Toute
communication passe par la tour de contrôle.</p></figcaption>
</figure>

Les pilotes qui vont décoller ou atterrir sur la piste ne communiquent
pas ensemble directement. Ils s'adressent à un contrôleur aérien qui est
assis dans une grande tour près de la piste d'atterrissage. Sans lui,
les pilotes seraient obligés de savoir si d'autres avions sont à
proximité et décider des priorités d'atterrissage avec un ensemble de
pilotes. Les accidents d'avion augmenteraient probablement beaucoup.

La tour n'a pas besoin de contrôler la totalité d'un vol. Elle n'est là
que pour faire respecter des contraintes au départ et à l'arrivée, pour
faire en sorte que les pilotes n'aient pas trop de paramètres à gérer.
:::

::::: {.section .structure-container}
##  Structure

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/mediator/structure71f0.png?id=1f2accc7820ecfe9665b6d30cbc0bc61"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.21;"
srcset="/images/patterns/diagrams/mediator/structure.png?id=1f2accc7820ecfe9665b6d30cbc0bc61 520w,/images/patterns/diagrams/mediator/structure-1.5x.png?id=958b373815bf6a56cd9b38763ed01dce 780w,/images/patterns/diagrams/mediator/structure-2x.png?id=5191daa1c9d4caa36e38af3c5b7d1522 1040w"
sizes="(max-width: 720px) 100vw, 520px" loading="lazy" width="520"
alt="Structure du patron de conception Médiateur" /><img
src="../../images/patterns/diagrams/mediator/structure-indexed5302.png?id=a82d4cf1b92a4f72af32f231ffd21131"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.21;"
srcset="/images/patterns/diagrams/mediator/structure-indexed.png?id=a82d4cf1b92a4f72af32f231ffd21131 520w,/images/patterns/diagrams/mediator/structure-indexed-1.5x.png?id=81d4f842ae5157a3cb09f8f3c05159dd 780w,/images/patterns/diagrams/mediator/structure-indexed-2x.png?id=88722da01a5c82b0452078c9339ca497 1040w"
sizes="(max-width: 720px) 100vw, 520px" loading="lazy" width="520"
alt="Structure du patron de conception Médiateur" />
</figure>
:::

1.  Les classes **Composant** contiennent de la logique métier. Chaque
    composant possède une référence vers un médiateur, déclaré avec le
    type de l'interface médiateur. Le composant ne connait pas la classe
    du médiateur, vous pouvez ainsi réutiliser les mêmes composants dans
    d'autres programmes en les liant à un autre médiateur.

2.  L'interface **Médiateur** déclare des méthodes pour communiquer avec
    les composants et n'est généralement dotée que d'une seule méthode
    de notification. Les composants peuvent passer n'importe quel
    contexte en paramètre de cette méthode (ce qui inclut leurs propres
    objets), mais en évitant de provoquer un couplage entre un composant
    récepteur et la classe du demandeur.

3.  Les **Médiateurs Concrets** encapsulent les relations entre les
    divers composants. Les médiateurs concrets gardent souvent des
    références vers les composants qu'ils gèrent et s'occupent même
    parfois de leur cycle de vie.

4.  Les composants ne doivent pas avoir de visibilité sur les autres
    composants. S'il arrive quelque chose d'important à un composant, il
    doit seulement en prévenir le médiateur. Quand ce dernier reçoit la
    notification, il doit pouvoir facilement identifier le demandeur, ce
    qui peut suffire pour déterminer le composant qui doit être
    déclenché en retour.

    De son point de vue, le composant ne voit qu'une boîte noire. Le
    demandeur ne sait pas qui va se charger de sa demande, et le
    récepteur ignore qui l'a envoyée.
::::
:::::

::: {.section .pseudocode}
##  Pseudo-code {#pseudocode}

Dans cet exemple, le **Médiateur** vous aide à éliminer les dépendances
mutuelles entre différentes classes de l'UI (boutons, cases à cocher et
libellés de texte).

<figure class="image">
<img
src="../../images/patterns/diagrams/mediator/example576a.png?id=3151c153533e816e226be0ef977211e8"
style="aspect-ratio: 1.24;"
srcset="/images/patterns/diagrams/mediator/example.png?id=3151c153533e816e226be0ef977211e8 560w,/images/patterns/diagrams/mediator/example-1.5x.png?id=cf441780d96f3306ac54d6809f85b87d 840w,/images/patterns/diagrams/mediator/example-2x.png?id=02064e5a7c4f065f806747e1b04ac1b0 1120w"
sizes="(max-width: 720px) 100vw, 560px" loading="lazy" width="560"
alt="Structure de l’exemple utilisé pour le patron de conception médiateur" />
<figcaption><p>Structure des classes des boîtes de dialogue
de l’UI.</p></figcaption>
</figure>

Un élément déclenché par un utilisateur ne communique pas directement
avec les autres éléments, même si c'est l'impression qui s'en dégage. À
la place, l'élément n'a besoin que de prévenir son médiateur qu'un
événement a eu lieu, en lui donnant les informations contextuelles avec
la notification.

Dans cet exemple, la boîte de dialogue d'identification prend le rôle du
médiateur. Elle sait comment les éléments concrets sont censés
collaborer et facilite leur communication indirecte. Lorsque la boîte de
dialogue reçoit une notification d'événement, elle sait quel élément est
concerné et redirige l'appel en conséquence.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// L’interface médiateur déclare une méthode qui permet aux
// composants d’avertir le médiateur au sujet de divers
// événements. Le médiateur peut réagir à ces événements et
// passer les appels aux autres composants.
interface Mediator is
    method notify(sender: Component, event: string)


// La classe concrète Médiateur. Les interconnexions entre les
// composants individuels ont été démêlées et transférées dans
// le médiateur.
class AuthenticationDialog implements Mediator is
    private field title: string
    private field loginOrRegisterChkBx: Checkbox
    private field loginUsername, loginPassword: Textbox
    private field registrationUsername, registrationPassword,
                  registrationEmail: Textbox
    private field okBtn, cancelBtn: Button

    constructor AuthenticationDialog() is
        // Crée les composants et passe le médiateur actuel dans
        // leurs constructeurs pour établir les liens.

    // Le médiateur est averti si un composant est affecté par
    // quoi que ce soit. Lorsqu’un médiateur reçoit une
    // notification, il peut lancer un traitement ou envoyer la
    // demande à un autre composant.
    method notify(sender, event) is
        if (sender == loginOrRegisterChkBx and event == &quot;check&quot;)
            if (loginOrRegisterChkBx.checked)
                title = &quot;Log in&quot;
                // 1. Affiche les composants du formulaire
                // d’identification.
                // 2. Cache les composants du formulaire
                // d’inscription.
            else
                title = &quot;Register&quot;
                // 1. Affiche les composants du formulaire
                // d’inscription.
                // 2. Cache les composants du formulaire
                // d’identification.

        if (sender == okBtn &amp;&amp; event == &quot;click&quot;)
            if (loginOrRegister.checked)
                // Essaye de trouver un utilisateur à l’aide de
                // ses identifiants.
                if (!found)
                    // Montre un message d’erreur au-dessus du
                    // champ identifiant.
            else
                // 1. Crée un compte utilisateur en utilisant
                // les données saisies dans les champs
                // d’inscription.
                // 2. Connecte cet utilisateur.
                // ...


// Les composants communiquent avec un médiateur en utilisant
// son interface. Grâce à cela, vous pouvez utiliser les mêmes
// composants dans d’autres contextes en les associant avec
// d’autres objets médiateur.
class Component is
    field dialog: Mediator

    constructor Component(dialog) is
        this.dialog = dialog

    method click() is
        dialog.notify(this, &quot;click&quot;)

    method keypress() is
        dialog.notify(this, &quot;keypress&quot;)

// Les composants concrets ne communiquent pas ensemble. Leur
// unique canal de communication leur sert à envoyer des
// notifications au médiateur.
class Button extends Component is
    // ...

class Textbox extends Component is
    // ...

class Checkbox extends Component is
    method check() is
        dialog.notify(this, &quot;check&quot;)
    // ...</code></pre>
</figure>
:::

:::::::::: {.section .applicability-container}
##  Possibilités d'application {#applicability}

::::::::: applicability
::: applicability-problem
Utilisez ce patron si vous rencontrez des difficultés pour modifier
certaines classes trop fortement couplées avec d'autres.
:::

::: applicability-solution
Le médiateur vous permet d'extraire toutes les relations entre les
classes dans une classe séparée, en isolant les modifications appliquées
à un composant spécifique du reste des composants.
:::

::: applicability-problem
Utilisez ce patron quand vous ne pouvez pas réutiliser un composant
ailleurs, car il est trop dépendant des autres composants.
:::

::: applicability-solution
Après avoir mis en place le médiateur, les composants individuels n'ont
plus de visibilité sur les autres composants. Ils peuvent toujours
communiquer ensemble mais indirectement, via un objet médiateur. Pour
réutiliser un composant dans une application différente, vous n'avez
besoin que de lui fournir une classe médiateur.
:::

::: applicability-problem
Utilisez le médiateur lorsque vous créez des tonnes de sous-classes pour
les composants, juste pour pouvoir bénéficier de leur comportement de
base dans différents contextes.
:::

::: applicability-solution
Toutes les relations entre composants se trouvent à l'intérieur d'un
médiateur. De nouvelles classes médiateur peuvent donc facilement
définir de nouveaux moyens de collaboration pour ces composants sans
avoir à les modifier.
:::
:::::::::
::::::::::

::: {.section .checklist}
##  Mise en œuvre {#checklist}

1.  Identifiez des classes qui sont fortement couplées et qui pourraient
    bénéficier de plus d'indépendance (pour une maintenance plus aisée
    ou pour faciliter la réutilisation de ces classes).

2.  Déclarez l'interface médiateur et écrivez le protocole de
    communication voulu entre les médiateurs et les différents
    composants. Dans la plupart des cas, une seule méthode est
    suffisante pour recevoir les notifications des composants.

    Cette interface est cruciale si vous voulez pouvoir réutiliser les
    classes des composants dans différents contextes. Tant que le
    composant communique avec le médiateur en passant par l'interface
    générique, vous pouvez relier le composant avec une implémentation
    différente du médiateur.

3.  Implémentez la classe concrète du médiateur. Faites en sorte que
    cette classe garde des références vers tous les composants qu'elle
    gère. Elle en tirera de gros bénéfices.

4.  Vous pouvez encore aller plus loin en rendant le médiateur
    responsable de la création et de la destruction des objets
    composant. Le médiateur ressemblera ainsi à une
    [fabrique](abstract-factory.html) ou à une [façade](facade.html).

5.  Les composants doivent avoir une référence vers l'objet médiateur.
    La connexion est souvent établie dans le constructeur du composant,
    dans lequel un objet médiateur est passé en paramètre.

6.  Modifiez le code des composants afin qu'ils appellent la méthode de
    notification du médiateur plutôt que des méthodes écrites dans
    d'autres composants. Mettez tout le code qui appelle les autres
    composants dans la classe médiateur, puis appelez-le lorsque le
    médiateur reçoit des notifications de ce composant.
:::

:::::: {.section .pros-cons}
##  Avantages et inconvénients {#pros-cons}

::::: row
::: col-sm-6
-    *Principe de responsabilité unique*. Vous pouvez mettre les
    communications entre les différents composants au même endroit,
    rendant le code plus facile à comprendre et à maintenir.
-    *Principe ouvert/fermé*. Vous pouvez ajouter de nouveaux médiateurs
    sans avoir à modifier les composants déjà en place.
-    Vous diminuez le couplage entre les différents composants d'un
    programme.
-    Vous pouvez réutiliser les composants individuels plus facilement.
:::

::: col-sm-6
-    Avec le temps, un médiateur peut évoluer en [Objet
    Omniscient](https://fr.wikipedia.org/wiki/God_object).
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

-   La [Façade](facade.html) et le [Médiateur](mediator.html) ont des
    rôles similaires : ils essayent de faire collaborer des classes
    étroitement liées.

    -   La *façade* définit une interface simplifiée pour un
        sous-système d'objets, mais elle n'ajoute pas de nouvelles
        fonctionnalités. Le sous-système lui-même n'a pas connaissance
        de la façade. Les objets situés à l'intérieur du sous-système
        peuvent communiquer directement.
    -   Le *médiateur* centralise la communication entre les composants
        du système. Les composants ne voient que l'objet médiateur et ne
        communiquent pas directement.

-   La différence entre le [Médiateur](mediator.html) et
    l'[Observateur](observer.html) est souvent très fine. Dans la
    majorité des cas, vous pouvez implémenter l'un ou l'autre, mais
    parfois vous pouvez les utiliser simultanément. Regardons comment
    faire.

    Le but principal du *médiateur* est d'éliminer les dépendances
    mutuelles entre un ensemble de composants du système. À la place,
    ces composants peuvent devenir dépendants d'un unique objet
    médiateur. Le but de l'*observateur* est d'établir des connexions
    dynamiques à sens unique entre les objets, où certains objets
    peuvent être les subordonnés d'autres objets.

    Il existe une implémentation populaire du *médiateur* qui repose sur
    l'*observateur*. L'objet médiateur joue le rôle du diffuseur et les
    composants agissent comme des souscripteurs qui s'inscrivent et se
    désinscrivent des événements du médiateur. Lorsque ce type de
    conception est mis en place, le *médiateur* ressemble de près à
    l'*observateur*.

    Si vous êtes un peu perdu, rappelez-vous qu'il y a plusieurs
    manières d'implémenter le médiateur. Par exemple, vous pouvez
    associer de manière permanente tous les composants au même objet
    médiateur. Cette implémentation ne ressemblera pas à
    l'*observateur*, mais sera tout de même une instance du patron de
    conception médiateur.

    Maintenant, imaginez un programme dont tous les composants sont
    devenus des diffuseurs, permettant des connexions dynamiques les uns
    avec les autres. Nous n'aurons pas d'objet médiateur centralisé,
    seulement un ensemble d'observateurs distribués.
:::

::: {.section .implementations}
##  Exemples de code {#implementations}

[![Médiateur en
C#](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](mediator/csharp/example.html "Médiateur en C#"){.prog-lang-link}
[![Médiateur en
C++](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](mediator/cpp/example.html "Médiateur en C++"){.prog-lang-link}
[![Médiateur en
Go](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](mediator/go/example.html "Médiateur en Go"){.prog-lang-link}
[![Médiateur en
Java](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](mediator/java/example.html "Médiateur en Java"){.prog-lang-link}
[![Médiateur en
PHP](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](mediator/php/example.html "Médiateur en PHP"){.prog-lang-link}
[![Médiateur en
Python](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](mediator/python/example.html "Médiateur en Python"){.prog-lang-link}
[![Médiateur en
Ruby](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](mediator/ruby/example.html "Médiateur en Ruby"){.prog-lang-link}
[![Médiateur en
Rust](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](mediator/rust/example.html "Médiateur en Rust"){.prog-lang-link}
[![Médiateur en
Swift](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](mediator/swift/example.html "Médiateur en Swift"){.prog-lang-link}
[![Médiateur en
TypeScript](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](mediator/typescript/example.html "Médiateur en TypeScript"){.prog-lang-link}
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

[Mémento []{.fa .fa-arrow-right}](memento.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Retour

[[]{.fa .fa-arrow-left} Itérateur](iterator.html){.btn .btn-default
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
