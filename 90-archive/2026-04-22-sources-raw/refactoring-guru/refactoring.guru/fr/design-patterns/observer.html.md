::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [Patrons de
conception](../design-patterns.html){.type} / [Patrons
comportementaux](behavioral-patterns.html){.type}
:::

# Observateur {#observateur .title}

::: aka
Alias :
[Dépendants, ]{style="display:inline-block"}[Diffusion--Souscription, ]{style="display:inline-block"}[Observer]{style="display:inline-block"}
:::

::: {.section .intent}
##  Intention {#intent}

L'**Observateur** est un patron de conception comportemental qui permet
de mettre en place un mécanisme de souscription pour envoyer des
notifications à plusieurs objets, au sujet d'événements concernant les
objets qu'ils observent.

<figure class="image">
<img
src="../../images/patterns/content/observer/observerd508.png?id=6088e31e1b0d4a417506a66614dcf065"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/observer/observer.png?id=6088e31e1b0d4a417506a66614dcf065 640w,/images/patterns/content/observer/observer-1.5x.png?id=aa64f9f336354462b57bbff5c09d8269 960w,/images/patterns/content/observer/observer-2x.png?id=d5a83e115528e9fd633f04ad2650f1db 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Patron de conception observateur" />
</figure>
:::

::: {.section .problem}
##  Problème {#problem}

Imaginez que vous avez deux types d'objets : un `Client` et un
`Magasin`. Le client s'intéresse à une marque spécifique d'un produit
(disons que c'est un nouveau modèle d'iPhone) qui sera bientôt
disponible dans la boutique.

Le client pourrait se rendre sur place tous les jours et vérifier la
disponibilité du produit. Mais comme le produit n'est pas encore prêt,
ses allées et venues seraient inutiles.

<figure class="image">
<img
src="../../images/patterns/content/observer/observer-comic-1-fr755c.png?id=17b813c140262b8a62128fa5d9c516bb"
style="aspect-ratio: 2;"
srcset="/images/patterns/content/observer/observer-comic-1-fr.png?id=17b813c140262b8a62128fa5d9c516bb 600w,/images/patterns/content/observer/observer-comic-1-fr-1.5x.png?id=722ed6dbf43cd4fc667166b3c71ed7e7 900w,/images/patterns/content/observer/observer-comic-1-fr-2x.png?id=7c28ce7e69fce33d51b1638fcfd8ccd0 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Se rendre au magasin ou envoyer du spam" />
<figcaption><p>Se rendre au magasin ou envoyer du spam.</p></figcaption>
</figure>

À la place, le magasin pourrait envoyer des tonnes d'e-mails (ce qui
peut être vu comme du spam) à leurs clients chaque fois qu'un nouveau
produit est disponible. Cette solution économiserait bien des voyages à
leurs clients. En contrepartie, le magasin risque de se mettre à dos
ceux qui ne sont pas intéressés par les nouveaux produits.

Nous nous retrouvons dans une situation conflictuelle. Soit les clients
perdent leur temps à venir vérifier la disponibilité des produits, soit
le magasin gâche des ressources pour prévenir des clients qui ne sont
pas concernés.
:::

::: {.section .solution}
##  Solution

L'objet que l'on veut suivre est en général appelé *sujet*, mais comme
il va envoyer des notifications pour prévenir les autres objets dès
qu'il est modifié, nous l'appellerons *diffuseur* (publisher). Tous les
objets qui veulent suivre les modifications apportées au diffuseur sont
appelés des *souscripteurs* (subscribers).

Le patron de conception Observateur vous propose d'ajouter un mécanisme
de souscription à la classe diffuseur pour permettre aux objets
individuels de s'inscrire ou se désinscrire de ce diffuseur. Pas
d'inquiétude ! Ce n'est pas si compliqué que cela en a l'air. En
réalité, ce mécanisme est composé 1) d'un tableau d'attributs qui stocke
une liste de références vers les objets souscripteur et 2) de plusieurs
méthodes publiques qui permettent d'ajouter ou de supprimer des
souscripteurs de cette liste.

<figure class="image">
<img
src="../../images/patterns/diagrams/observer/solution1-fra6a4.png?id=b142518315f2a8da608520aed4a57f22"
style="aspect-ratio: 2.61;"
srcset="/images/patterns/diagrams/observer/solution1-fr.png?id=b142518315f2a8da608520aed4a57f22 470w,/images/patterns/diagrams/observer/solution1-fr-1.5x.png?id=4d40a8f8f1fe59b34580a4c341766b03 705w,/images/patterns/diagrams/observer/solution1-fr-2x.png?id=8961a3d66c8e95c6f572de9688c808d9 940w"
sizes="(max-width: 720px) 100vw, 470px" loading="lazy" width="470"
alt="Mécanisme de souscription" />
<figcaption><p>Un mécanisme de souscription qui permet aux objets
individuels de s’inscrire aux notifications
des événements.</p></figcaption>
</figure>

Quand un événement important arrive au diffuseur, il fait le tour de ses
souscripteurs et appelle la méthode de notification sur leurs objets.

Les applications peuvent comporter des dizaines de classes souscripteur
différentes qui veulent être tenues au courant des événements qui
affectent une même classe diffuseur. Vous n'avez sûrement pas envie de
coupler le diffuseur à toutes ces classes. De plus, certaines ne seront
peut-être pas connues à l'avance, dans les cas où votre classe diffuseur
est censée pouvoir être utilisée par d'autres personnes.

C'est pourquoi il est crucial que tous les souscripteurs implémentent la
même interface et qu'elle soit le seul moyen utilisé par le diffuseur
pour communiquer avec eux. Elle doit déclarer la méthode de notification
avec un ensemble de paramètres que le diffuseur peut utiliser pour
envoyer des données contextuelles avec la notification.

<figure class="image">
<img
src="../../images/patterns/diagrams/observer/solution2-fr5f6d.png?id=f53f5bd087084fdde0758d154ba7f610"
style="aspect-ratio: 1.24;"
srcset="/images/patterns/diagrams/observer/solution2-fr.png?id=f53f5bd087084fdde0758d154ba7f610 460w,/images/patterns/diagrams/observer/solution2-fr-1.5x.png?id=359921477a38339159022da491c31b9f 690w,/images/patterns/diagrams/observer/solution2-fr-2x.png?id=9698c9840c07db0f32803abebc24fff5 920w"
sizes="(max-width: 720px) 100vw, 460px" loading="lazy" width="460"
alt="Méthodes de notification" />
<figcaption><p>Le diffuseur envoie des notifications aux souscripteurs
en appelant la méthode de notification spécifique sur
leurs objets.</p></figcaption>
</figure>

De plus, les diffuseurs doivent tous suivre la même interface si votre
application en comporte plusieurs types et que vous voulez que vos
souscripteurs soient tous compatibles avec eux. Cette interface doit
contenir quelques méthodes de souscription et elle doit permettre aux
souscripteurs d'observer les états du diffuseur sans le coupler avec
leurs classes concrètes.
:::

::: {.section .analogy}
##  Analogie {#analogy}

<figure class="image">
<img
src="../../images/patterns/content/observer/observer-comic-2-fr99ca.png?id=bd2f92abc92fb001f1b6963578ddc864"
style="aspect-ratio: 2;"
srcset="/images/patterns/content/observer/observer-comic-2-fr.png?id=bd2f92abc92fb001f1b6963578ddc864 600w,/images/patterns/content/observer/observer-comic-2-fr-1.5x.png?id=9fdedbc68cae945f1588df9c03b1ad10 900w,/images/patterns/content/observer/observer-comic-2-fr-2x.png?id=95fe005acb7e7077dcdf5ec47ec57cff 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Abonnement aux magazines et aux journaux" />
<figcaption><p>Abonnement aux magazines et
aux journaux.</p></figcaption>
</figure>

Lorsque vous vous inscrivez à un journal ou à un magazine, vous n'avez
plus besoin de vous rendre en magasin pour vérifier si le dernier numéro
est sorti. À la place, le diffuseur vous envoie directement les nouveaux
numéros dans votre boîte aux lettres dès qu'ils le publient, ou même
parfois en avance.

Le diffuseur garde une liste de souscripteurs et connait les magazines
qui les intéressent. S'ils ne souhaitent plus recevoir les nouveaux
numéros, les souscripteurs peuvent quitter la liste à n'importe quel
moment.
:::

::::: {.section .structure-container}
##  Structure

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/observer/structurea121.png?id=365b7e2b8fbecc8948f34b9f8f16f33c"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.97;"
srcset="/images/patterns/diagrams/observer/structure.png?id=365b7e2b8fbecc8948f34b9f8f16f33c 610w,/images/patterns/diagrams/observer/structure-1.5x.png?id=7f23a48c9a2d789844f912d3a0f289e6 915w,/images/patterns/diagrams/observer/structure-2x.png?id=228af9bded4d6ee6daf43a0e23cca9ff 1220w"
sizes="(max-width: 720px) 100vw, 610px" loading="lazy" width="610"
alt="Structure du patron de conception observateur" /><img
src="../../images/patterns/diagrams/observer/structure-indexed8afd.png?id=2ca2c123503ede860740af2a22bc4b4d"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.88;"
srcset="/images/patterns/diagrams/observer/structure-indexed.png?id=2ca2c123503ede860740af2a22bc4b4d 620w,/images/patterns/diagrams/observer/structure-indexed-1.5x.png?id=20b9e7765e83ca75514c202968d9e9fa 930w,/images/patterns/diagrams/observer/structure-indexed-2x.png?id=910eec855bc41f05199e510494078926 1240w"
sizes="(max-width: 720px) 100vw, 620px" loading="lazy" width="620"
alt="Structure du patron de conception observateur" />
</figure>
:::

1.  Le **Diffuseur** envoie des événements intéressants à d'autres
    objets. Ces événements se produisent quand le diffuseur change
    d'état ou exécute certains comportements. Le diffuseur possède une
    infrastructure d'inscription qui permet aux nouveaux souscripteurs
    de rejoindre la liste et aux souscripteurs actuels de la quitter.

2.  Quand un nouvel événement survient, le diffuseur parcourt la liste
    d'inscriptions et appelle la méthode de notification déclarée dans
    l'interface des souscripteurs sur chaque objet souscripteur.

3.  L'interface **Souscripteur** déclare les méthodes de notification.
    Dans la majorité des cas, il n'y a qu'une seule méthode `update`.
    Elle peut prendre plusieurs paramètres pour que le diffuseur leur
    envoie plus de détails concernant la modification.

4.  Les **Souscripteurs Concrets** exécutent certaines actions en
    réponse aux notifications envoyées par le diffuseur. Toutes ces
    classes doivent implémenter la même interface pour ne pas coupler le
    diffuseur avec leurs classes concrètes.

5.  En général, les souscripteurs ont besoin de détails à propos du
    contexte afin d'exécuter correctement la mise à jour. C'est pour
    cela que les diffuseurs passent souvent des données du contexte en
    paramètre de la méthode de notification. Le diffuseur peut même
    s'envoyer lui-même en paramètre et laisser les souscripteurs
    récupérer directement les données nécessaires.

6.  Le **Client** crée des objets diffuseur et Souscripteur séparément
    et inscrit les souscripteurs aux mises à jour du diffuseur.
::::
:::::

::: {.section .pseudocode}
##  Pseudo-code {#pseudocode}

Dans cet exemple, le patron de conception **Observateur** permet à
l'objet éditeur de texte d'avertir d'autres objets de service des
changements de son état.

<figure class="image">
<img
src="../../images/patterns/diagrams/observer/example04a2.png?id=6d0603ab5a00e4463b81d9639cd746a2"
style="aspect-ratio: 1.19;"
srcset="/images/patterns/diagrams/observer/example.png?id=6d0603ab5a00e4463b81d9639cd746a2 510w,/images/patterns/diagrams/observer/example-1.5x.png?id=67adccd1030a38dd263bfa09867f8dbe 765w,/images/patterns/diagrams/observer/example-2x.png?id=e2838e1562325e485fc7c2828a8ca445 1020w"
sizes="(max-width: 720px) 100vw, 510px" loading="lazy" width="510"
alt="Structure de l’exemple utilisé pour le patron de conception Observateur" />
<figcaption><p>Envoyer des notifications aux objets au sujet
d’événements qui affectent d’autres objets.</p></figcaption>
</figure>

La liste des souscripteurs est compilée dynamiquement : les objets
peuvent commencer ou arrêter de suivre les notifications lors du
lancement de l'application, en fonction du comportement voulu.

Dans cette implémentation, la classe éditeur ne gère pas la liste des
souscripteurs toute seule. Elle délègue la tâche à l'objet spécial
assistant dont c'est la seule tâche. Vous pouvez moduler cet objet afin
qu'il serve de centrale de distribution, transformant n'importe quel
objet en diffuseur.

Tant que les classes diffuseur passent toutes par la même interface pour
communiquer avec les souscripteurs, l'ajout de nouveaux souscripteurs ne
nécessite aucun changement dans les classes diffuseur.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// La classe de base diffuseur contient le code pour
// l’inscription et les méthodes de notification.
class EventManager is
    private field listeners: hash map of event types and listeners

    method subscribe(eventType, listener) is
        listeners.add(eventType, listener)

    method unsubscribe(eventType, listener) is
        listeners.remove(eventType, listener)

    method notify(eventType, data) is
        foreach (listener in listeners.of(eventType)) do
            listener.update(data)

// Le diffuseur concret abrite de la logique métier dédiée à
// certains souscripteurs. Nous pourrions dériver cette classe
// depuis le diffuseur de base, mais ce n’est pas toujours
// possible, car le diffuseur concret pourrait déjà être une
// sous-classe. Dans ce cas, vous pouvez utiliser la composition
// pour effectuer le lien avec la logique de souscription, comme
// effectué ci-dessous :
class Editor is
    public field events: EventManager
    private field file: File

    constructor Editor() is
        events = new EventManager()

    // Les méthodes de la logique métier peuvent prévenir les
    // souscripteurs de toute modification.
    method openFile(path) is
        this.file = new File(path)
        events.notify(&quot;open&quot;, file.name)

    method saveFile() is
        file.write()
        events.notify(&quot;save&quot;, file.name)

    // ...


// Voici l’interface des souscripteurs. Si votre langage de
// programmation prend en charge les types fonctionnels, vous
// pouvez remplacer toute la hiérarchie des souscripteurs par un
// ensemble de fonctions.
interface EventListener is
    method update(filename)

// Les souscripteurs concrets réagissent aux mises à jour de
// leur diffuseur.
class LoggingListener implements EventListener is
    private field log: File
    private field message: string

    constructor LoggingListener(log_filename, message) is
        this.log = new File(log_filename)
        this.message = message

    method update(filename) is
        log.write(replace(&#39;%s&#39;,filename,message))

class EmailAlertsListener implements EventListener is
    private field email: string
    private field message: string

    constructor EmailAlertsListener(email, message) is
        this.email = email
        this.message = message

    method update(filename) is
        system.email(email, replace(&#39;%s&#39;,filename,message))


// Une application peut configurer des diffuseurs et des
// souscripteurs à l’exécution.
class Application is
    method config() is
        editor = new Editor()

        logger = new LoggingListener(
            &quot;/path/to/log.txt&quot;,
            &quot;Someone has opened the file: %s&quot;)
        editor.events.subscribe(&quot;open&quot;, logger)

        emailAlerts = new EmailAlertsListener(
            &quot;admin@example.com&quot;,
            &quot;Someone has changed the file: %s&quot;)
        editor.events.subscribe(&quot;save&quot;, emailAlerts)</code></pre>
</figure>
:::

:::::::: {.section .applicability-container}
##  Possibilités d'application {#applicability}

::::::: applicability
::: applicability-problem
Utilisez le patron de conception Observateur quand des modifications de
l'état d'un objet peuvent en impacter d'autres, et que l'ensemble des
objets n'est pas connu à l'avance ou qu'il change dynamiquement.
:::

::: applicability-solution
Ce problème est souvent rencontré lorsque l'on travaille sur des classes
d'une interface utilisateur graphique. Par exemple, si vous créez des
classes bouton personnalisées et que vous voulez que les clients
puissent y ajouter du code déclenché par le clic d'un utilisateur.

L'observateur permet à tous les objets qui suivent l'interface
souscripteur de s'inscrire aux notifications des événements des objets
diffuseur. Vous pouvez ajouter le mécanisme de souscription à tous vos
boutons et laisser les clients mettre leur code personnalisé dans des
classes souscripteur personnalisées.
:::

::: applicability-problem
Utilisez ce patron quand certains objets de votre application doivent en
suivre d'autres, mais seulement pendant un certain temps ou dans des cas
spécifiques.
:::

::: applicability-solution
La liste d'inscription est dynamique, les souscripteurs peuvent donc
rejoindre ou quitter la liste quand ils le désirent.
:::
:::::::
::::::::

::: {.section .checklist}
##  Mise en œuvre {#checklist}

1.  Passez en revue votre logique métier et découpez-la en deux
    parties : la fonctionnalité principale (indépendante du reste du
    code) prendra le rôle du diffuseur ; le reste va être transformé en
    classes souscripteur.

2.  Déclarez l'interface souscripteur. Elle doit au moins contenir une
    méthode `update`.

3.  Déclarez l'interface diffuseur et écrivez des méthodes qui
    permettent d'ajouter et de retirer des objets souscripteur de la
    liste. Rappelez-vous que les diffuseurs doivent manipuler les
    souscripteurs uniquement en passant par l'interface des
    souscripteurs.

4.  Décidez où vous allez mettre la liste de souscription ainsi que
    l'implémentation des méthodes de souscription. En général, ce code
    est presque identique pour tous les types de diffuseurs. Le mieux
    est donc de le mettre dans une classe abstraite directement dérivée
    de l'interface diffuseur. Les diffuseurs concrets étendent cette
    classe et héritent du comportement de la souscription.

    Si vous implémentez ce patron dans une hiérarchie de classes
    existante, pensez à la composition : mettez la logique de
    souscription dans un objet séparé et faites en sorte que les
    diffuseurs l'utilisent.

5.  Créez les classes concrètes Diffuseur. Chaque fois que quelque chose
    d'important se produit chez un diffuseur, il doit prévenir ses
    souscripteurs.

6.  Implémentez les méthodes de notifications (`update`) dans les
    classes concrètes Souscripteur. La majorité des souscripteurs va
    vouloir des données du contexte qui concernent l'événement en
    question. Vous pouvez les envoyer en paramètre de la méthode de
    notification.

    Mais il y a une autre possibilité. Lors de la réception d'une
    notification, le souscripteur peut aller chercher les données
    directement dans la notification. Dans ce cas, le diffuseur doit
    s'envoyer lui-même en paramètre de la méthode `update`. On peut
    également lier un diffuseur à un souscripteur de manière permanente
    dans le constructeur, mais cette possibilité est moins flexible.

7.  Le client doit créer tous les souscripteurs nécessaires et les
    enregistrer auprès des diffuseurs.
:::

:::::: {.section .pros-cons}
##  Avantages et inconvénients {#pros-cons}

::::: row
::: col-sm-6
-    *Principe ouvert/fermé*. Vous pouvez ajouter de nouvelles classes
    souscripteur sans avoir à modifier le code du diffuseur (et
    inversement si vous avez une interface diffuseur).
-    Vous pouvez établir des relations entre les objets lors du
    lancement de l'application.
:::

::: col-sm-6
-    Les souscripteurs sont avertis dans un ordre aléatoire.
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

[![Observateur en
C#](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](observer/csharp/example.html "Observateur en C#"){.prog-lang-link}
[![Observateur en
C++](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](observer/cpp/example.html "Observateur en C++"){.prog-lang-link}
[![Observateur en
Go](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](observer/go/example.html "Observateur en Go"){.prog-lang-link}
[![Observateur en
Java](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](observer/java/example.html "Observateur en Java"){.prog-lang-link}
[![Observateur en
PHP](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](observer/php/example.html "Observateur en PHP"){.prog-lang-link}
[![Observateur en
Python](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](observer/python/example.html "Observateur en Python"){.prog-lang-link}
[![Observateur en
Ruby](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](observer/ruby/example.html "Observateur en Ruby"){.prog-lang-link}
[![Observateur en
Rust](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](observer/rust/example.html "Observateur en Rust"){.prog-lang-link}
[![Observateur en
Swift](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](observer/swift/example.html "Observateur en Swift"){.prog-lang-link}
[![Observateur en
TypeScript](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](observer/typescript/example.html "Observateur en TypeScript"){.prog-lang-link}
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

[État []{.fa .fa-arrow-right}](state.html){.btn .btn-primary rel="next"}
:::

::: prev
#### Retour

[[]{.fa .fa-arrow-left} Mémento](memento.html){.btn .btn-default
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
