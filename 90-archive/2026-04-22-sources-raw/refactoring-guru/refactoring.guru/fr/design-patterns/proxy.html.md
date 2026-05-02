::::::::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [Patrons de
conception](../design-patterns.html){.type} / [Patrons
structurels](structural-patterns.html){.type}
:::

# Procuration {#procuration .title}

::: aka
Alias : [Proxy]{style="display:inline-block"}
:::

::: {.section .intent}
##  Intention {#intent}

La **Procuration** est un patron de conception structurel qui vous
permet d'utiliser un substitut pour un objet. Elle donne le contrôle sur
l'objet original, vous permettant d'effectuer des manipulations avant ou
après que la demande ne lui parvienne.

<figure class="image">
<img
src="../../images/patterns/content/proxy/proxyf258.png?id=efece4647fb11e3f7539291796327666"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/proxy/proxy.png?id=efece4647fb11e3f7539291796327666 640w,/images/patterns/content/proxy/proxy-1.5x.png?id=80a11690fa49172c517470a834289b60 960w,/images/patterns/content/proxy/proxy-2x.png?id=fb3d14e21c210a758d4777f4d93dce09 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Patron de conception procuration" />
</figure>
:::

::: {.section .problem}
##  Problème {#problem}

Pourquoi vouloir maîtriser l'accès à un objet ? Voici un exemple : vous
avez un énorme objet qui consomme beaucoup de ressources, mais vous n'en
avez besoin que de temps en temps.

<figure class="image">
<img
src="../../images/patterns/diagrams/proxy/problem-frb932.png?id=477c3aa4f87946fb7f7c0136c3520779"
style="aspect-ratio: 3.19;"
srcset="/images/patterns/diagrams/proxy/problem-fr.png?id=477c3aa4f87946fb7f7c0136c3520779 510w,/images/patterns/diagrams/proxy/problem-fr-1.5x.png?id=c151ffc536ff48c5c2ebb405dc4c3acc 765w,/images/patterns/diagrams/proxy/problem-fr-2x.png?id=295838d92bd07249aad764c0bc6b8cc7 1020w"
sizes="(max-width: 720px) 100vw, 510px" loading="lazy" width="510"
alt="Problème résolu grâce au patron de conception procuration" />
<figcaption><p>Les requêtes sur la base de données sont parfois
très lentes.</p></figcaption>
</figure>

Vous pouvez recourir à l'instanciation paresseuse : créer l'objet
uniquement lorsque vous en avez besoin. Vous devrez implémenter une
initialisation différée dans tous les clients pour l'objet concerné.
Malheureusement, vous allez vous retrouver avec beaucoup de code
dupliqué.

Dans le meilleur des mondes, nous voudrions mettre ce code directement
dans la classe de l'objet, mais ce n'est pas toujours possible. La
classe pourrait par exemple faire partie d'une application externe non
modifiable.
:::

::: {.section .solution}
##  Solution

Ce patron de conception vous propose de créer une classe procuration qui
a la même interface que l'objet du service original. Vous passez ensuite
l'objet procuration à tous les clients de l'objet original. Lors de la
réception d'une demande d'un client, la procuration crée l'objet du
service original et lui délègue la tâche.

<figure class="image">
<img
src="../../images/patterns/diagrams/proxy/solution-fr155e.png?id=9a2ea427c1e526b658f3021c3ced566d"
style="aspect-ratio: 3.19;"
srcset="/images/patterns/diagrams/proxy/solution-fr.png?id=9a2ea427c1e526b658f3021c3ced566d 510w,/images/patterns/diagrams/proxy/solution-fr-1.5x.png?id=3fd0d848c5b4c92d4e96f7fe492a32db 765w,/images/patterns/diagrams/proxy/solution-fr-2x.png?id=a194832b4b32244ff1ce50df118fe1fb 1020w"
sizes="(max-width: 720px) 100vw, 510px" loading="lazy" width="510"
alt="Solution proposée par le patron de conception procuration" />
<figcaption><p>La procuration se déguise en objet base de données. Elle
peut gérer l’instanciation paresseuse et mettre en cache le résultat
sans que le client ou que l’objet de la base de données ne
le remarque.</p></figcaption>
</figure>

Mais quel est son intérêt ? Si vous voulez lancer un traitement avant ou
après avoir appliqué la logique principale de la classe, la procuration
peut intervenir sans modifier cette dernière. Elle implémente la même
interface que la classe originale, et peut donc être passée en paramètre
à n'importe quel client qui attend l'objet du service original.
:::

::: {.section .analogy}
##  Analogie {#analogy}

<figure class="image">
<img
src="../../images/patterns/diagrams/proxy/live-example66da.png?id=a268c57fdaf073ee81cf4dfc7239eae2"
style="aspect-ratio: 2.57;"
srcset="/images/patterns/diagrams/proxy/live-example.png?id=a268c57fdaf073ee81cf4dfc7239eae2 540w,/images/patterns/diagrams/proxy/live-example-1.5x.png?id=4b8ee7f90cf76180004e9f5d37661ee2 810w,/images/patterns/diagrams/proxy/live-example-2x.png?id=8b8083fa1954d2d92ca84a5a5636ead7 1080w"
sizes="(max-width: 720px) 100vw, 540px" loading="lazy" width="540"
alt="Une carte de crédit est une procuration pour une liasse de billets" />
<figcaption><p>Une carte de crédit peut être utilisée au même titre que
de l’argent liquide.</p></figcaption>
</figure>

Une carte de crédit est une procuration pour un compte bancaire, qui est
une procuration pour une liasse de billets. Ils implémentent la même
interface : tous deux peuvent être utilisés pour effectuer un paiement.
Le consommateur apprécie grandement, car il n'a pas besoin de garder une
grosse somme en liquide sur lui. Le commerçant est également très
heureux, car les transactions sont versées électroniquement vers son
compte en banque, sans courir le risque d'égarer son dépôt ou de se le
faire voler sur le chemin de la banque.
:::

::::: {.section .structure-container}
##  Structure

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/proxy/structure7fcd.png?id=f2478a82a84e1a1e512a8414bf1abd1c"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1;"
srcset="/images/patterns/diagrams/proxy/structure.png?id=f2478a82a84e1a1e512a8414bf1abd1c 370w,/images/patterns/diagrams/proxy/structure-1.5x.png?id=0db7bf3c1193b2a1961894349f1e07ad 555w,/images/patterns/diagrams/proxy/structure-2x.png?id=3d54eeca9af4aa373e989a73463539b5 740w"
sizes="(max-width: 720px) 100vw, 370px" loading="lazy" width="370"
alt="Structure du patron de conception procuration" /><img
src="../../images/patterns/diagrams/proxy/structure-indexed2b2c.png?id=e56d420f31925b3d41348c5036d3cd64"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.05;"
srcset="/images/patterns/diagrams/proxy/structure-indexed.png?id=e56d420f31925b3d41348c5036d3cd64 410w,/images/patterns/diagrams/proxy/structure-indexed-1.5x.png?id=5b24af632e76d81d24eab0b7c0be1a66 615w,/images/patterns/diagrams/proxy/structure-indexed-2x.png?id=ddf2b3b4807e52330c456c59fc52d504 820w"
sizes="(max-width: 720px) 100vw, 410px" loading="lazy" width="410"
alt="Structure du patron de conception procuration" />
</figure>
:::

1.  L'**Interface Service** déclare l'interface du service. La
    procuration doit implémenter cette interface afin de pouvoir se
    déguiser en objet du service.

2.  Le **Service** est une classe qui fournit la logique métier dont
    vous voulez vous servir.

3.  La **Procuration** est une classe dotée d'un attribut qui pointe
    vers un objet service. Une fois que la procuration a lancé tous ses
    traitements (instanciation paresseuse, historisation des logs,
    vérification des droits, mise en cache, etc.), elle envoie la
    demande à l'objet du service.

    En général, les procurations gèrent le cycle de vie de leurs objets
    service.

4.  Le **Client** passe par la même interface pour travailler avec les
    services et les procurations. Il est ainsi possible de passer une
    procuration à n'importe quel code qui attend un objet service.
::::
:::::

::: {.section .pseudocode}
##  Pseudo-code {#pseudocode}

L'exemple suivant montre comment le patron de conception **Procuration**
nous aide à utiliser l'instanciation paresseuse et la mise en cache pour
intégrer une librairie YouTube externe.

<figure class="image">
<img
src="../../images/patterns/diagrams/proxy/example4e80.png?id=ddb1402479562b9e2c776933cc861bed"
style="aspect-ratio: 1.23;"
srcset="/images/patterns/diagrams/proxy/example.png?id=ddb1402479562b9e2c776933cc861bed 490w,/images/patterns/diagrams/proxy/example-1.5x.png?id=9b7608e1ef46a52d4ca1d7d89986e5b0 735w,/images/patterns/diagrams/proxy/example-2x.png?id=40f22d12d183b9285a380e4a072efb16 980w"
sizes="(max-width: 720px) 100vw, 490px" loading="lazy" width="490"
alt="Structure de l’exemple utilisé pour la procuration" />
<figcaption><p>Résultats obtenus pour la mise en cache d’un service à
l’aide d’une procuration.</p></figcaption>
</figure>

La librairie nous fournit une classe pour télécharger des vidéos.
Malheureusement, elle n'est pas très efficace. Si l'application client
demande une même vidéo à plusieurs reprises, la librairie va la
télécharger encore et encore, plutôt que de la mettre en cache après la
première utilisation.

La classe procuration implémente la même interface que l'outil de
téléchargement original et délègue tout le travail à ce dernier. En
revanche, elle garde la trace de tous les fichiers téléchargés et
retourne les données du cache lorsque l'application fait plusieurs fois
appel à la même vidéo.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// L’interface d’un service distant
interface ThirdPartyYouTubeLib is
    method listVideos()
    method getVideoInfo(id)
    method downloadVideo(id)

// La méthode d’implémentation concrète d’un connecteur de
// service. Les méthodes de cette classe peuvent demander des
// informations à YouTube. La vitesse de la demande dépend de la
// connexion Internet de l’utilisateur, ainsi que de celle de
// YouTube. L’application sera plus lente si plusieurs demandes
// sont envoyées en même temps, même si elles recherchent les
// mêmes informations.
class ThirdPartyYouTubeClass implements ThirdPartyYouTubeLib is
    method listVideos() is
        // Envoie une demande via une API à YouTube.

    method getVideoInfo(id) is
        // Récupère les métadonnées d’une vidéo.

    method downloadVideo(id) is
        // Télécharge un fichier vidéo sur YouTube.

// Pour économiser de la bande passante, nous pouvons mettre en
// cache les résultats des demandes et les mettre de côté
// temporairement. Mais vous n’allez pas forcément pouvoir
// écrire ce code directement dans la classe du service. Elle
// pourrait provenir d’une librairie externe ou avoir été
// définie comme `final`. Pour cette raison, nous écrivons le
// code de la mise en cache dans une nouvelle classe procuration
// qui implémente la même interface que la classe du service.
// Elle délègue les tâches à l’objet du service lorsque les
// demandes doivent vraiment être envoyées.
class CachedYouTubeClass implements ThirdPartyYouTubeLib is
    private field service: ThirdPartyYouTubeLib
    private field listCache, videoCache
    field needReset

    constructor CachedYouTubeClass(service: ThirdPartyYouTubeLib) is
        this.service = service

    method listVideos() is
        if (listCache == null || needReset)
            listCache = service.listVideos()
        return listCache

    method getVideoInfo(id) is
        if (videoCache == null || needReset)
            videoCache = service.getVideoInfo(id)
        return videoCache

    method downloadVideo(id) is
        if (!downloadExists(id) || needReset)
            service.downloadVideo(id)

// La classe GUI qui fonctionnait auparavant avec un objet
// Service n’a pas besoin d’être modifiée tant qu’elle interagit
// avec lui par le biais d’une interface. Nous pouvons passer en
// toute sécurité un objet procuration à la place d’un objet du
// service, car ils implémentent la même interface.
class YouTubeManager is
    protected field service: ThirdPartyYouTubeLib

    constructor YouTubeManager(service: ThirdPartyYouTubeLib) is
        this.service = service

    method renderVideoPage(id) is
        info = service.getVideoInfo(id)
        // Affiche la page de la vidéo.

    method renderListPanel() is
        list = service.listVideos()
        // Affiche la liste des miniatures des vidéos.

    method reactOnUserInput() is
        renderVideoPage()
        renderListPanel()

// L’application peut configurer les procurations à la volée.
class Application is
    method init() is
        aYouTubeService = new ThirdPartyYouTubeClass()
        aYouTubeProxy = new CachedYouTubeClass(aYouTubeService)
        manager = new YouTubeManager(aYouTubeProxy)
        manager.reactOnUserInput()</code></pre>
</figure>
:::

:::::::::::::::: {.section .applicability-container}
##  Possibilités d'application {#applicability}

::::::::::::::: applicability
La procuration possède de nombreux usages. Passons en revue les plus
populaires.

::: applicability-problem
Instanciation paresseuse (procuration virtuelle). À utiliser lorsque
l'objet du service est très consommateur en ressources système car il
est actif en permanence, mais vous n'en avez pas tout le temps besoin.
:::

::: applicability-solution
Vous pouvez différer l'initialisation de l'objet plutôt que de le créer
au lancement de l'application.
:::

::: applicability-problem
Vérification des droits (procuration de protection). Si vous avez besoin
de limiter l'accès de vos clients à l'objet du service, par exemple si
vos objets sont des parties cruciales d'un système d'exploitation et que
les clients sont différentes applications lancées (dont certaines sont
malveillantes).
:::

::: applicability-solution
La procuration peut ne transmettre une demande à l'objet du service que
si les identifiants du client remplissent certains critères.
:::

::: applicability-problem
Lancement local d'un service distant (procuration à distance). L'objet
du service se situe sur un serveur distant.
:::

::: applicability-solution
Dans ce cas, la procuration envoie la demande par le réseau en
s'occupant de gérer tous les détails compliqués de la gestion du réseau.
:::

::: applicability-problem
Demande de logs (procuration de logs). Pour garder un historique des
demandes passées auprès de l'objet du service.
:::

::: applicability-solution
La procuration peut garder la trace de toutes les demandes passées au
service.
:::

::: applicability-problem
Mettre en cache les résultats des demandes (procuration de cache). Si
vous voulez mettre en cache les résultats des demandes faites au client
et gérer le cycle de vie de ce cache, principalement lorsque les
résultats sont imposants.
:::

::: applicability-solution
La procuration peut gérer la mise en cache des demandes récurrentes qui
retournent toujours le même résultat. Les paramètres des demandes
peuvent servir de clé.
:::

::: applicability-problem
Référencement intelligent. Si vous voulez vous débarrasser d'un objet
très consommateur en ressources lorsqu'aucun client ne l'utilise.
:::

::: applicability-solution
La procuration peut garder la trace des clients qui ont récupéré une
référence vers l'objet du service ou vers ses résultats. De temps en
temps, la procuration peut faire le tour des clients et vérifier s'ils
sont toujours actifs. Si la liste des clients est vide, la procuration
peut supprimer l'objet du service afin de libérer des ressources.

Elle peut également savoir si le client avait modifié l'objet du
service. Les objets non modifiés peuvent ensuite être réutilisés par les
autres clients.
:::
:::::::::::::::
::::::::::::::::

::: {.section .checklist}
##  Mise en œuvre {#checklist}

1.  Si le service n'a pas encore d'interface, créez-en une pour rendre
    la procuration et les objets du service interchangeables. Il n'est
    pas toujours possible d'extraire l'interface de la classe du
    service, car vous devez effectuer des modifications pour que tous
    les clients du service utilisent cette interface. Heureusement, nous
    avons un plan B. Transformez la procuration en sous-classe de la
    classe service, afin qu'elle hérite de l'interface du service.

2.  Créez la classe procuration. Elle doit inclure un attribut qui
    stocke la référence au service. En général, les procurations créent
    et gèrent le cycle de vie de leurs services. En de rares occasions,
    un service est passé à la procuration par le biais d'un constructeur
    du client.

3.  Mettez en place les méthodes de la procuration et leur
    fonctionnement. Dans la plupart des cas, la procuration doit
    déléguer le travail à l'objet du service après avoir lancé ses
    traitements.

4.  Réfléchissez à l'implémentation d'une méthode de création qui décide
    si votre client doit utiliser directement le service ou passer par
    la procuration. Il peut s'agir d'une méthode statique toute simple
    ou d'une classe procuration avec une méthode fabrique complète.

5.  Envisagez également l'implémentation d'une instanciation paresseuse
    pour l'objet du service.
:::

:::::: {.section .pros-cons}
##  Avantages et inconvénients {#pros-cons}

::::: row
::: col-sm-6
-    Vous pouvez contrôler l'objet du service sans que le client ne s'en
    aperçoive.
-    Vous pouvez gérer le cycle de vie de l'objet du service si les
    clients ne s'en occupent pas.
-    La procuration fonctionne même si l'objet du service n'est pas prêt
    ou pas disponible.
-    *Principe ouvert/fermé*. Vous pouvez ajouter de nouvelles
    procurations sans toucher au service ou aux clients.
:::

::: col-sm-6
-    Le code peut devenir plus complexe puisque vous devez y introduire
    de nombreuses classes.
-    La réponse du service peut mettre plus de temps à arriver.
:::
:::::
::::::

::: {.section .relations}
##  Liens avec les autres patrons {#relations}

-   Avec [Adaptateur](adapter.html), vous accédez à un objet existant
    via une interface différente. Avec [Procuration](proxy.html),
    l'interface reste la même. Avec [Décorateur](decorator.html), vous
    accédez à l'objet via une interface améliorée.

-   La [Façade](facade.html) et la [Procuration](proxy.html) ont une
    similarité : ils mettent en tampon une entité complexe et
    l'initialisent individuellement. Contrairement à la *façade*, la
    *procuration* implémente la même interface que son objet service, ce
    qui les rend interchangeables.

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

[![Procuration en
C#](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](proxy/csharp/example.html "Procuration en C#"){.prog-lang-link}
[![Procuration en
C++](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](proxy/cpp/example.html "Procuration en C++"){.prog-lang-link}
[![Procuration en
Go](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](proxy/go/example.html "Procuration en Go"){.prog-lang-link}
[![Procuration en
Java](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](proxy/java/example.html "Procuration en Java"){.prog-lang-link}
[![Procuration en
PHP](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](proxy/php/example.html "Procuration en PHP"){.prog-lang-link}
[![Procuration en
Python](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](proxy/python/example.html "Procuration en Python"){.prog-lang-link}
[![Procuration en
Ruby](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](proxy/ruby/example.html "Procuration en Ruby"){.prog-lang-link}
[![Procuration en
Rust](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](proxy/rust/example.html "Procuration en Rust"){.prog-lang-link}
[![Procuration en
Swift](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](proxy/swift/example.html "Procuration en Swift"){.prog-lang-link}
[![Procuration en
TypeScript](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](proxy/typescript/example.html "Procuration en TypeScript"){.prog-lang-link}
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

[Patrons comportementaux []{.fa
.fa-arrow-right}](behavioral-patterns.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Retour

[[]{.fa .fa-arrow-left} Poids mouche](flyweight.html){.btn .btn-default
rel="prev"}
:::
::::::::::::::::::::::::::::::::::::::::

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
::::::::::::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::::::::::::
