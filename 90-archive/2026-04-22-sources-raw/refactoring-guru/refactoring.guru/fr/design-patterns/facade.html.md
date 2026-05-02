::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [Patrons de
conception](../design-patterns.html){.type} / [Patrons
structurels](structural-patterns.html){.type}
:::

# Façade {#façade .title}

::: aka
Alias : [Facade]{style="display:inline-block"}
:::

::: {.section .intent}
##  Intention {#intent}

**Façade** est un patron de conception structurel qui procure une
interface offrant un accès simplifié à une librairie, un framework ou à
n'importe quel ensemble complexe de classes.

<figure class="image">
<img
src="../../images/patterns/content/facade/facade721d.png?id=1f4be17305b6316fbd548edf1937ac3b"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/facade/facade.png?id=1f4be17305b6316fbd548edf1937ac3b 640w,/images/patterns/content/facade/facade-1.5x.png?id=a6af5003b243b59990842d24bdaae2df 960w,/images/patterns/content/facade/facade-2x.png?id=b69fce5943703f5f07c0ba38e3baaed0 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Patron de conception façade" />
</figure>
:::

::: {.section .problem}
##  Problème {#problem}

Imaginez que vous devez adapter votre code pour manipuler un ensemble
d'objets qui appartiennent à une librairie ou à un framework assez
sophistiqué. D'ordinaire, vous initialisez tous ces objets en premier,
gardez la trace des dépendances et appelez les méthodes dans le bon
ordre, etc.

Par conséquent, la logique métier de vos classes devient fortement
couplée avec les détails de l'implémentation des classes externes,
rendant cette logique difficile à comprendre et à maintenir.
:::

::: {.section .solution}
##  Solution

Une façade est une classe qui procure une interface simple vers un
sous-système complexe de parties mobiles. Les fonctionnalités proposées
par la façade seront plus limitées que si vous interagissiez directement
avec le sous-système, mais vous pouvez vous contenter de n'inclure que
les fonctionnalités qui intéressent votre client.

Une façade se révèle très pratique si votre application n'a besoin que
d'une partie des fonctionnalités d'une librairie sophistiquée parmi les
nombreuses qu'elle propose.

Par exemple, une application qui envoie des petites vidéos de chats
comiques sur des réseaux sociaux peut potentiellement utiliser une
librairie de conversion vidéo professionnelle. Mais la seule chose dont
vous ayez réellement besoin, c'est d'une classe dotée d'une méthode
`encoder(fichier, format)`. Après avoir créé cette classe et intégré la
librairie de conversion vidéo, votre façade est opérationnelle.
:::

::: {.section .analogy}
##  Analogie {#analogy}

<figure class="image">
<img
src="../../images/patterns/diagrams/facade/live-example-fr9060.png?id=b35a189c618a8a0d6dbb72d596164e0d"
style="aspect-ratio: 2.58;"
srcset="/images/patterns/diagrams/facade/live-example-fr.png?id=b35a189c618a8a0d6dbb72d596164e0d 490w,/images/patterns/diagrams/facade/live-example-fr-1.5x.png?id=3d83377f5df5bf0c0470752cb0eac8c1 735w,/images/patterns/diagrams/facade/live-example-fr-2x.png?id=7a851907e0736102174c07dbcfa77336 980w"
sizes="(max-width: 720px) 100vw, 490px" loading="lazy" width="490"
alt="Un exemple d’un passage de commande au téléphone" />
<figcaption><p>Passer une commande au téléphone.</p></figcaption>
</figure>

Lorsque vous téléphonez à un magasin pour commander, un opérateur joue
le rôle de la façade pour tous ses services. L'opérateur vous sert
d'interface vocale avec le système de commandes, les moyens de paiement
et les différents services de livraison.
:::

::::: {.section .structure-container}
##  Structure

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/facade/structure2883.png?id=258401362234ac77a2aaf1cde62339e7"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.47;"
srcset="/images/patterns/diagrams/facade/structure.png?id=258401362234ac77a2aaf1cde62339e7 560w,/images/patterns/diagrams/facade/structure-1.5x.png?id=98d134de0c368d382909ba162ec21767 840w,/images/patterns/diagrams/facade/structure-2x.png?id=528ca429555bce293b7c3bd90954e097 1120w"
sizes="(max-width: 720px) 100vw, 560px" loading="lazy" width="560"
alt="Structure du patron de conception façade" /><img
src="../../images/patterns/diagrams/facade/structure-indexeddb1c.png?id=2da06d6b850701ea15cf72f9d2642fb8"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.58;"
srcset="/images/patterns/diagrams/facade/structure-indexed.png?id=2da06d6b850701ea15cf72f9d2642fb8 600w,/images/patterns/diagrams/facade/structure-indexed-1.5x.png?id=fad7d667f4d8d9a7d522748bcd37dfde 900w,/images/patterns/diagrams/facade/structure-indexed-2x.png?id=4d181bcf1df5d58c533e6c92461e9571 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Structure du patron de conception façade" />
</figure>
:::

1.  La **Façade** procure un accès pratique aux différentes parties des
    fonctionnalités du sous-système. Elle sait où diriger les requêtes
    du client et comment utiliser les différentes parties mobiles.

2.  Une classe **Façade Supplémentaire** peut être créée pour éviter de
    polluer une autre façade avec des fonctionnalités qui pourraient la
    rendre trop complexe. Les façades supplémentaires peuvent être
    utilisées à la fois par le client et par les autres façades.

3.  Le **Sous-système Complexe** est composé de dizaines d'objets
    variés. Pour leur donner une réelle utilité, vous devez plonger au
    cœur des détails de l'implémentation du sous-système, comme
    initialiser les objets dans le bon ordre et leur fournir les données
    dans le bon format.

    Les classes du sous-système ne sont pas conscientes de l'existence
    de la façade. Elles opèrent et interagissent directement à
    l'intérieur de leur propre système.

4.  Le **Client** passe par la façade plutôt que d'appeler les objets du
    sous-système directement.
::::
:::::

::: {.section .pseudocode}
##  Pseudo-code {#pseudocode}

Dans cet exemple, la **Façade** simplifie l'interaction avec un
framework complexe de conversion vidéo.

<figure class="image">
<img
src="../../images/patterns/diagrams/facade/example16bc.png?id=2249d134e3ff83819dfc19032f02eced"
style="aspect-ratio: 1.5;"
srcset="/images/patterns/diagrams/facade/example.png?id=2249d134e3ff83819dfc19032f02eced 570w,/images/patterns/diagrams/facade/example-1.5x.png?id=034ecbe0f3cc108f4ae49d05d1c77dbe 855w,/images/patterns/diagrams/facade/example-2x.png?id=f2c846d74041626c923ff3e8919b68a9 1140w"
sizes="(max-width: 720px) 100vw, 570px" loading="lazy" width="570"
alt="La structure de la façade dans l’exemple utilisé" />
<figcaption><p>Utilisation d’une classe façade dans un exemple
d’isolation de dépendances multiples.</p></figcaption>
</figure>

Plutôt que d'adapter directement la base de votre code à des dizaines de
classes, vous créez une façade qui permet d'encapsuler cette
fonctionnalité et de la cacher du reste du code. Cette structure permet
de minimiser le travail nécessaire aux futures évolutions du framework,
ou au remplacement de ce dernier par un autre. Vous pouvez vous
contenter de modifier l'implémentation des méthodes de la façade.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// Voici quelques classes d’un framework complexe de conversion
// vidéo externe. Nous n’avons pas la main sur le code, nous ne
// pouvons donc pas le simplifier.
class VideoFile
// ...

class OggCompressionCodec
// ...

class MPEG4CompressionCodec
// ...

class CodecFactory
// ...

class BitrateReader
// ...

class AudioMixer
// ...


// Nous créons une classe façade pour cacher la complexité du
// framework derrière une interface simple. C’est un compromis
// entre fonctionnalité et simplicité.
class VideoConverter is
    method convert(filename, format):File is
        file = new VideoFile(filename)
        sourceCodec = (new CodecFactory).extract(file)
        if (format == &quot;mp4&quot;)
            destinationCodec = new MPEG4CompressionCodec()
        else
            destinationCodec = new OggCompressionCodec()
        buffer = BitrateReader.read(filename, sourceCodec)
        result = BitrateReader.convert(buffer, destinationCodec)
        result = (new AudioMixer()).fix(result)
        return new File(result)

// Les classes application ne dépendent pas d’un milliard de
// classes fournies par un framework complexe. Si vous décidez
// de changer de framework, vous avez seulement besoin de
// réécrire la classe façade.
class Application is
    method main() is
        convertor = new VideoConverter()
        mp4 = convertor.convert(&quot;funny-cats-video.ogg&quot;, &quot;mp4&quot;)
        mp4.save()</code></pre>
</figure>
:::

:::::::: {.section .applicability-container}
##  Possibilités d'application {#applicability}

::::::: applicability
::: applicability-problem
Utilisez la façade si vous avez besoin d'une interface limitée mais
directe à un sous-système complexe.
:::

::: applicability-solution
Souvent, les sous-systèmes gagnent en complexité avec le temps et mettre
en place un patron de conception implique de créer de nouvelles classes.
Dans de nombreux contextes, un sous-système peut se révéler plus souple
et facile à réutiliser, mais la quantité de travail pour configurer et
écrire le code de base ne fait que croitre. La façade tente de remédier
à ce problème en fournissant un raccourci aux fonctionnalités du
sous-système qui sont adaptées au besoin du client.
:::

::: applicability-problem
Utilisez la façade si vous voulez structurer un sous-système en
plusieurs couches.
:::

::: applicability-solution
Créez des façades pour définir des points d'entrée à chaque niveau d'un
sous-système. Vous pouvez réduire le couplage entre plusieurs
sous-systèmes en les obligeant à communiquer au travers de façades.

Reprenons notre exemple de framework de conversion vidéo. Il peut être
désassemblé en deux couches : tout ce qui concerne la vidéo d'un côté et
l'audio de l'autre. Créez une façade pour chaque couche. Vous pouvez
utiliser ce genre de façades pour faire communiquer les classes de ces
couches ensemble. Cette approche peut fortement ressembler au patron de
conception [Médiateur](mediator.html).
:::
:::::::
::::::::

::: {.section .checklist}
##  Mise en œuvre {#checklist}

1.  Vérifiez s'il est possible de fournir une interface plus simple que
    ce qu'un sous-système vous propose. Si cette interface peut
    découpler le code client des nombreuses classes du sous-système,
    vous êtes sur la bonne voie !

2.  Déclarez et implémentez cette interface en tant que nouvelle façade.
    Elle doit rediriger les appels du code client aux objets appropriés
    du sous-système. Elle doit également être responsable de
    l'initialisation du sous-système et gérer son cycle de vie, sauf si
    le code client s'en occupe déjà.

3.  Pour exploiter au maximum les avantages de la façade, toute
    communication du code client avec le sous-système doit passer par
    elle. Cette manipulation protège le code client de tout effet de
    bord que des modifications apportées au sous-système pourraient
    provoquer. Si un sous-système est mis à jour, vous n'aurez besoin
    que de modifier le code de la façade.

4.  Si la façade devient [trop grande](../smells/large-class.html), vous
    devriez envisager de transférer une partie de ses comportements vers
    une nouvelle façade plus spécialisée.
:::

:::::: {.section .pros-cons}
##  Avantages et inconvénients {#pros-cons}

::::: row
::: col-sm-6
-    Vous pouvez isoler votre code de la complexité d'un sous-système.
:::

::: col-sm-6
-    Une façade peut devenir [un objet
    omniscient](https://fr.wikipedia.org/wiki/God_object) couplé à
    toutes les classes d'une application.
:::
:::::
::::::

::: {.section .relations}
##  Liens avec les autres patrons {#relations}

-   La [Façade](facade.html) définit une nouvelle interface pour les
    objets existants, alors que l'[Adaptateur](adapter.html) essaye de
    rendre l'interface existante utilisable. L'*adaptateur* emballe
    généralement un seul objet alors que la *façade* s'utilise pour un
    sous-système complet d'objets.

-   La [Fabrique abstraite](abstract-factory.html) peut remplacer la
    [Façade](facade.html) si vous voulez simplement cacher au code
    client la création des objets du sous-système.

-   Le [Poids mouche](flyweight.html) vous montre comment créer plein de
    petits objets alors que la [Façade](facade.html) permet de créer un
    seul objet qui représente un sous-système complet.

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

-   Une classe [Façade](facade.html) peut souvent être transformée en
    [Singleton](singleton.html), car un seul objet façade est en général
    suffisant.

-   La [Façade](facade.html) et la [Procuration](proxy.html) ont une
    similarité : ils mettent en tampon une entité complexe et
    l'initialisent individuellement. Contrairement à la *façade*, la
    *procuration* implémente la même interface que son objet service, ce
    qui les rend interchangeables.
:::

::: {.section .implementations}
##  Exemples de code {#implementations}

[![Façade en
C#](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](facade/csharp/example.html "Façade en C#"){.prog-lang-link}
[![Façade en
C++](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](facade/cpp/example.html "Façade en C++"){.prog-lang-link}
[![Façade en
Go](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](facade/go/example.html "Façade en Go"){.prog-lang-link}
[![Façade en
Java](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](facade/java/example.html "Façade en Java"){.prog-lang-link}
[![Façade en
PHP](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](facade/php/example.html "Façade en PHP"){.prog-lang-link}
[![Façade en
Python](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](facade/python/example.html "Façade en Python"){.prog-lang-link}
[![Façade en
Ruby](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](facade/ruby/example.html "Façade en Ruby"){.prog-lang-link}
[![Façade en
Rust](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](facade/rust/example.html "Façade en Rust"){.prog-lang-link}
[![Façade en
Swift](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](facade/swift/example.html "Façade en Swift"){.prog-lang-link}
[![Façade en
TypeScript](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](facade/typescript/example.html "Façade en TypeScript"){.prog-lang-link}
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

[Poids mouche []{.fa .fa-arrow-right}](flyweight.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Retour

[[]{.fa .fa-arrow-left} Décorateur](decorator.html){.btn .btn-default
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
