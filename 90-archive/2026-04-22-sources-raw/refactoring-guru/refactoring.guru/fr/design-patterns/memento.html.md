::::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [Patrons de
conception](../design-patterns.html){.type} / [Patrons
comportementaux](behavioral-patterns.html){.type}
:::

# Mémento {#mémento .title}

::: aka
Alias :
[Jeton, ]{style="display:inline-block"}[Memento]{style="display:inline-block"}
:::

::: {.section .intent}
##  Intention {#intent}

**Mémento** est un patron de conception comportemental qui permet de
sauvegarder et de rétablir l'état précédent d'un objet sans révéler les
détails de son implémentation.

<figure class="image">
<img
src="../../images/patterns/content/memento/memento-fra9ff.png?id=9562d687ba053fc3b8d4aadc6baea1e3"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/memento/memento-fr.png?id=9562d687ba053fc3b8d4aadc6baea1e3 640w,/images/patterns/content/memento/memento-fr-1.5x.png?id=38b847f9f065aecb663338bbae55d1ed 960w,/images/patterns/content/memento/memento-fr-2x.png?id=3a2bafb2e99b4d86c6a1241b9836d5a4 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Patron de conception mémento" />
</figure>
:::

::: {.section .problem}
##  Problème {#problem}

Imaginez que vous êtes en train de créer un éditeur de texte. En plus de
pouvoir écrire du texte, votre éditeur permet de le formater, d'insérer
des images alignées, etc.

Au bout d'un moment, vous décidez d'ajouter une fonctionnalité qui
permet aux utilisateurs d'annuler une action sur le texte. Cette
fonctionnalité est devenue si classique au fil des années, que les
utilisateurs s'attendent systématiquement à en disposer. Vous choisissez
une approche directe pour la mettre en place. Avant d'effectuer une
action, l'application enregistre l'état de tous les objets et les
sauvegarde. Plus tard, lorsqu'un utilisateur décide d'annuler une
action, l'application récupère la dernière sauvegarde de l'historique et
s'en sert pour restaurer l'état de tous les objets.

<figure class="image">
<img
src="../../images/patterns/diagrams/memento/problem1-fr830b.png?id=997ea7877b15817b75b64dac1474d79d"
style="aspect-ratio: 2.61;"
srcset="/images/patterns/diagrams/memento/problem1-fr.png?id=997ea7877b15817b75b64dac1474d79d 470w,/images/patterns/diagrams/memento/problem1-fr-1.5x.png?id=6260ae68b51b005738bdceb5bb5f47e1 705w,/images/patterns/diagrams/memento/problem1-fr-2x.png?id=49a77bba6d927a84c224c2d14ceff53e 940w"
sizes="(max-width: 720px) 100vw, 470px" loading="lazy" width="470"
alt="Annuler des actions effectuées dans l’éditeur" />
<figcaption><p>Avant d’effectuer une action, l’application prend un
instantané (snapshot) du dernier état des objets. Il peut être utilisé
plus tard pour rétablir les objets dans leur
ancien état.</p></figcaption>
</figure>

Prenons un moment pour réfléchir à ces photos. Comment va-t-on s'y
prendre pour les créer ? Vous allez probablement devoir parcourir tous
les attributs d'un objet et copier leurs valeurs quelque part.
Cependant, cela ne fonctionnera que si le contenu de l'objet ne possède
pas trop de restrictions. Malheureusement, la plupart des objets ne se
laissent pas approcher si facilement et cachent les données importantes
dans des attributs privés.

Laissons de côté ce problème pour le moment et considérons que nos
objets se comportent en bons hippies : ils préfèrent avoir des relations
ouvertes et garder leur état public. Même si cette approche nous permet
de produire des instantanés de l'état des objets à volonté, de sérieux
problèmes demeurent. Dans le futur, vous pourriez décider de
refactoriser certaines classes de l'éditeur, ou d'ajouter ou de
supprimer certains attributs. Cela semble facile à première vue, mais
vous allez devoir également modifier les classes qui ont la
responsabilité de créer la copie de l'état des objets concernés.

<figure class="image">
<img
src="../../images/patterns/diagrams/memento/problem2-fr1b6b.png?id=9e8c34309c4d00804220436446aea263"
style="aspect-ratio: 1.43;"
srcset="/images/patterns/diagrams/memento/problem2-fr.png?id=9e8c34309c4d00804220436446aea263 300w,/images/patterns/diagrams/memento/problem2-fr-1.5x.png?id=53fd3245d75a906af28b519ff8f08837 450w,/images/patterns/diagrams/memento/problem2-fr-2x.png?id=bedab26f4d4c02524f6d1d1e52c08c87 600w"
sizes="(max-width: 720px) 100vw, 300px" loading="lazy" width="300"
alt="Comment faire une copie de l’état privé d’un objet ?" />
<figcaption><p>Comment faire une copie de l’état privé d’un
objet ?</p></figcaption>
</figure>

Mais ce n'est pas tout ! Regardons un peu du côté des « photos » de
l'état de l'éditeur. Quel genre de données contiennent-elles ? On doit
au moins avoir accès aux coordonnées du curseur de la souris, à la
position de la barre de défilement, etc. Pour prendre une photo, vous
devez récupérer ces valeurs et les mettre dans un conteneur.

Vous allez très certainement stocker un paquet de ces conteneurs dans
une liste qui représente l'historique. Ces conteneurs seront
probablement les objets d'une classe. Cette classe possèdera très peu de
méthodes, mais aura beaucoup d'attributs qui répliquent l'état de
l'éditeur. Pour permettre à d'autres objets de lire et d'écrire les
données d'une photo, vous allez devoir rendre ses attributs publics, ce
qui exposerait les états de l'éditeur, qu'ils soient privés ou non. Les
autres classes deviendraient dépendantes au moindre changement apporté à
la classe photo. Si nous rendons ses attributs et méthodes privés, ces
changements ne seraient de toute façon pas répercutés sur les autres
classes.

Il semblerait que nous sommes dans une impasse : soit on expose tous les
détails d'une classe, ce qui les rend trop fragiles, soit on restreint
l'accès à leur état, ce qui empêche de prendre des instantanés.
Dispose-t-on d'une autre technique pour implémenter un « annuler » ?
:::

::: {.section .solution}
##  Solution

Tous les problèmes que nous rencontrons sont provoqués par une mauvaise
encapsulation. Certains objets essayent d'en faire plus que ce qu'ils
sont supposés faire. Pour récupérer les données dont ils ont besoin pour
effectuer une action, ils envahissent l'espace personnel des autres
objets, plutôt que de leur demander d'exécuter l'action eux-mêmes.

Le mémento délègue la création des états des photos à leur propriétaire,
l'objet *créateur* (originator). Plutôt que d'essayer de copier l'état
de l'éditeur depuis « l'extérieur », la classe de l'éditeur de texte
peut prendre la photo elle-même, car elle a accès à son propre état.

Ce patron propose de stocker la copie de l'état de l'objet dans un objet
spécial appelé *mémento*. Son contenu n'est accessible que pour l'objet
qui l'a créé. Les autres objets peuvent communiquer avec les mémentos
via une interface limitée qui leur permet de récupérer certaines
métadonnées de la photo (date de création, nom de l'action effectuée,
etc.), mais pas l'état de l'objet original contenu dans la photo.

<figure class="image">
<img
src="../../images/patterns/diagrams/memento/solution-fr549d.png?id=61e190b167107ad4f8c74b29f9fb7b22"
style="aspect-ratio: 1.36;"
srcset="/images/patterns/diagrams/memento/solution-fr.png?id=61e190b167107ad4f8c74b29f9fb7b22 610w,/images/patterns/diagrams/memento/solution-fr-1.5x.png?id=636e8f3dc12910cd0aa291e2beea23fc 915w,/images/patterns/diagrams/memento/solution-fr-2x.png?id=a95f7fe35ed11e0eff0bbf6341f31c27 1220w"
sizes="(max-width: 720px) 100vw, 610px" loading="lazy" width="610"
alt="Le créateur a un accès total au mémento, alors que le gardien ne peut que consulter les métadonnées" />
<figcaption><p>Le créateur a un accès total au mémento, alors que le
gardien ne peut que consulter les métadonnées.</p></figcaption>
</figure>

Une telle stratégie vous permet de stocker des mémentos à l'intérieur
d'autres objets que l'on appelle *gardiens* (caretakers). Le gardien ne
manipule le mémento qu'en passant par l'interface limitée. Il ne peut
donc pas modifier les états qui y sont stockés. De son côté, le créateur
rétablit les états qu'il désire, car il a accès à tous les attributs du
mémento.

Dans l'exemple de notre éditeur de texte, nous pouvons créer une classe
historique séparée qui prend le rôle du gardien. La pile de mémentos
stockée dans le gardien va grandir chaque fois que l'éditeur va
effectuer une action. Vous pouvez même afficher cette pile dans
l'interface utilisateur, afin de montrer les dernières opérations
effectuées par un utilisateur.

Lorsqu'un utilisateur veut annuler une action, l'historique prend le
mémento le plus récent de la pile et l'envoie à l'éditeur en demandant
un rollback. Comme l'éditeur possède un accès total au mémento, il
change lui-même son état en copiant les valeurs du mémento.
:::

:::::::::: {.section .structure-container}
##  Structure

::::::::: structure
#### Implémentation basée sur des classes imbriquées

L'implémentation classique du patron repose sur le principe des classes
imbriquées, disponibles dans de nombreux langages de programmation
populaires (comme le C++, C# et Java).

:::: memento-classic
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/memento/structure180c1.png?id=4b4a42363a005b617d4df06689787385"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.87;"
srcset="/images/patterns/diagrams/memento/structure1.png?id=4b4a42363a005b617d4df06689787385 580w,/images/patterns/diagrams/memento/structure1-1.5x.png?id=82cf757f153840c85d27bd63f3f3e133 870w,/images/patterns/diagrams/memento/structure1-2x.png?id=d4e77295e51c2417f22b7abb396d5977 1160w"
sizes="(max-width: 720px) 100vw, 580px" loading="lazy" width="580"
alt="Mémento basé sur les classes imbriquées" /><img
src="../../images/patterns/diagrams/memento/structure1-indexeda7c3.png?id=f79a8356b087ae6b004aec42b787ae2e"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.87;"
srcset="/images/patterns/diagrams/memento/structure1-indexed.png?id=f79a8356b087ae6b004aec42b787ae2e 580w,/images/patterns/diagrams/memento/structure1-indexed-1.5x.png?id=0687dc84dd4c98b4591a70ebd05c4d8e 870w,/images/patterns/diagrams/memento/structure1-indexed-2x.png?id=62fea7bdbc96420568869ea3bd25f6ad 1160w"
sizes="(max-width: 720px) 100vw, 580px" loading="lazy" width="580"
alt="Mémento basé sur les classes imbriquées" />
</figure>
:::
::::

1.  La classe **Créateur** (Originator) peut prendre des instantanés de
    son propre état, et le restaurer à volonté.

2.  Le **Mémento** est un objet de valeur qui joue le rôle d'un
    instantané d'un état du créateur. En général, le mémento n'est pas
    modifiable et écrit ses données une seule fois dans son
    constructeur.

3.  Le **Gardien** sait « quand » et « pourquoi » il doit photographier
    l'état du créateur, mais il sait également quand l'état doit être
    restauré.

    Le gardien peut conserver la trace de l'historique du créateur en
    enregistrant une pile de mémentos. Lorsque le créateur veut revenir
    dans le passé, le gardien récupère l'élément du haut de la pile et
    l'envoie à la méthode de restauration du créateur.

4.  Dans cette implémentation, la classe mémento est imbriquée à
    l'intérieur du créateur. Ceci permet au créateur d'accéder aux
    attributs et méthodes du mémento, même s'ils sont privés. Le gardien
    quant à lui n'a qu'un accès limité aux attributs et méthodes du
    mémento : on le laisse entasser les mémentos dans la pile, mais il
    ne peut pas les modifier.

#### Implémentation basée sur une interface intermédiaire

Voici une autre possibilité très pratique pour les langages qui ne
gèrent pas les classes imbriquées (oui PHP, je te regarde).

:::: memento-empty-interface
::: {.struct-image2 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/memento/structure21ec9.png?id=fcff71cb648389be2e302fbe55e2f847"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/diagrams/memento/structure2.png?id=fcff71cb648389be2e302fbe55e2f847 560w,/images/patterns/diagrams/memento/structure2-1.5x.png?id=5c05495a7ca6545fcc57f70ea7ee904a 840w,/images/patterns/diagrams/memento/structure2-2x.png?id=aa7fb5d0f622d4344a2cb590f437f8c8 1120w"
sizes="(max-width: 720px) 100vw, 560px" loading="lazy" width="560"
alt="Le mémento sans classes imbriquées" /><img
src="../../images/patterns/diagrams/memento/structure2-indexede045.png?id=2c98b4f64b03f2a30e159de31ca9f718"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.6;"
srcset="/images/patterns/diagrams/memento/structure2-indexed.png?id=2c98b4f64b03f2a30e159de31ca9f718 560w,/images/patterns/diagrams/memento/structure2-indexed-1.5x.png?id=1ba6e0f22bb613f3f1dcf86850c3b604 840w,/images/patterns/diagrams/memento/structure2-indexed-2x.png?id=2fb637daef1110dfa89f15b2d4627894 1120w"
sizes="(max-width: 720px) 100vw, 560px" loading="lazy" width="560"
alt="Le mémento sans classes imbriquées" />
</figure>
:::
::::

1.  En l'absence de classes imbriquées, vous pouvez restreindre l'accès
    aux attributs du mémento en établissant une convention : les
    gardiens ne vont pouvoir manipuler un mémento qu'à travers une
    interface intermédiaire déclarée explicitement. Cette interface ne
    déclare que des méthodes liées aux métadonnées du mémento.

2.  De leur côté, les créateurs peuvent manipuler directement les objets
    mémento, accéder à leurs attributs et méthodes déclarés dans la
    classe mémento. L'inconvénient de cette technique est que vous devez
    déclarer tous les membres du mémento en public.

#### Implémentation avec une encapsulation encore plus stricte

Si vous ne voulez vraiment pas que les autres classes accèdent au
créateur en passant par le mémento, vous pouvez vous y prendre
différemment.

:::: memento-safe
::: {.struct-image3 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/memento/structure3f138.png?id=6c3ef6a916be8c8ec6d6659d19a6a79f"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.74;"
srcset="/images/patterns/diagrams/memento/structure3.png?id=6c3ef6a916be8c8ec6d6659d19a6a79f 590w,/images/patterns/diagrams/memento/structure3-1.5x.png?id=9bb6d9dd5567bc71d9e93efc9183ef3a 885w,/images/patterns/diagrams/memento/structure3-2x.png?id=988c37f92059457153d26ba3458d371e 1180w"
sizes="(max-width: 720px) 100vw, 590px" loading="lazy" width="590"
alt="Le mémento avec une encapsulation stricte" /><img
src="../../images/patterns/diagrams/memento/structure3-indexedadf8.png?id=17e84b0ef89a41bb3fb844c8d7a445ad"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.74;"
srcset="/images/patterns/diagrams/memento/structure3-indexed.png?id=17e84b0ef89a41bb3fb844c8d7a445ad 590w,/images/patterns/diagrams/memento/structure3-indexed-1.5x.png?id=c121c75333433b70b9a67b75e536214c 885w,/images/patterns/diagrams/memento/structure3-indexed-2x.png?id=fef9ae2a0151c105976075aafb8939dd 1180w"
sizes="(max-width: 720px) 100vw, 590px" loading="lazy" width="590"
alt="Le mémento avec une encapsulation stricte" />
</figure>
:::
::::

1.  Cette implémentation permet de gérer plusieurs créateurs et
    plusieurs mémentos. Chaque créateur manipule sa propre classe
    mémento. Les créateurs et les mémentos n'exposent pas leur état aux
    autres.

2.  Il est désormais explicité que les gardiens ne peuvent pas modifier
    l'état stocké dans le mémento. De plus, la classe gardien n'est plus
    couplée avec le créateur, car la méthode de restauration est
    maintenant définie dans la classe mémento.

3.  Chaque mémento devient lié au créateur qui l'a produite. Le créateur
    s'envoie lui-même et toutes les valeurs de son état au constructeur
    du mémento. Grâce à l'étroite proximité de ces deux classes, un
    mémento peut restaurer l'état de son créateur, tant que ce dernier a
    bien défini les setters appropriés.
:::::::::
::::::::::

::: {.section .pseudocode}
##  Pseudo-code {#pseudocode}

Cet exemple utilise le patron [Commande](command.html) en plus du
mémento pour stocker les photos de l'état de l'éditeur de texte
complexe, et rétablit un état précédent lorsqu'on le lui demande.

<figure class="image">
<img
src="../../images/patterns/diagrams/memento/example4526.png?id=fb2196b065f374a1c2a64a0943463760"
style="aspect-ratio: 4.29;"
srcset="/images/patterns/diagrams/memento/example.png?id=fb2196b065f374a1c2a64a0943463760 600w,/images/patterns/diagrams/memento/example-1.5x.png?id=174b686f918a2c49e2545d5573c52d42 900w,/images/patterns/diagrams/memento/example-2x.png?id=41a73f3cc22bc3dd180f53e6968974d4 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Structure de l’exemple utilisé pour le mémento" />
<figcaption><p>Sauvegarder des photos de l’état de l’éditeur
de texte.</p></figcaption>
</figure>

Les objets commande prennent le rôle de gardiens. Ils vont chercher le
mémento de l'éditeur avant de lancer les actions déclenchées par les
commandes. Lorsqu'un utilisateur veut annuler l'action la plus récente,
l'éditeur peut se servir du mémento stocké dans cette commande pour
revenir à son état précédent.

La classe mémento ne déclare aucun attribut public, ni de getters et de
setters. Aucun objet ne peut modifier son contenu. Les mémentos sont
reliés à l'objet éditeur qui les a créés. Le mémento peut aussi
restaurer l'état de l'éditeur qui lui est associé, en passant les
données dans les setters de l'objet éditeur. Comme les mémentos sont
liés à des objets éditeur spécifiques, votre application peut gérer
plusieurs fenêtres avec des éditeurs indépendants et une pile
centralisée d'états à rétablir.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// Le créateur conserve des données importantes qui varient
// parfois avec le temps. Il définit également une méthode pour
// sauvegarder son état dans un mémento, et une autre méthode
// pour rétablir cet état.
class Editor is
    private field text, curX, curY, selectionWidth

    method setText(text) is
        this.text = text

    method setCursor(x, y) is
        this.curX = x
        this.curY = y

    method setSelectionWidth(width) is
        this.selectionWidth = width

    // Sauvegarde l’état actuel dans un mémento.
    method createSnapshot():Snapshot is
        // Le mémento est un objet non modifiable. C’est pour
        // cela que le créateur passe son état dans les
        // paramètres du constructeur du mémento.
        return new Snapshot(this, text, curX, curY, selectionWidth)

// La classe mémento conserve un ancien état de l’éditeur.
class Snapshot is
    private field editor: Editor
    private field text, curX, curY, selectionWidth

    constructor Snapshot(editor, text, curX, curY, selectionWidth) is
        this.editor = editor
        this.text = text
        this.curX = x
        this.curY = y
        this.selectionWidth = selectionWidth

    // Un objet mémento peut être utilisé pour rétablir un
    // ancien état de l’éditeur.
    method restore() is
        editor.setText(text)
        editor.setCursor(curX, curY)
        editor.setSelectionWidth(selectionWidth)

// Un objet commande peut prendre le rôle du gardien. Dans ce
// cas, un mémento est affecté à la commande juste avant que
// l&#39;état du créateur ne change. Lorsqu’une demande de
// restauration arrive, il rétablit l’état du créateur à partir
// du mémento.
class Command is
    private field backup: Snapshot

    method makeBackup() is
        backup = editor.createSnapshot()

    method undo() is
        if (backup != null)
            backup.restore()
    // ...</code></pre>
</figure>
:::

:::::::: {.section .applicability-container}
##  Possibilités d'application {#applicability}

::::::: applicability
::: applicability-problem
Utilisez le mémento si vous voulez prendre des photos de l'état d'un
objet afin de pouvoir rétablir un de ses états précédents.
:::

::: applicability-solution
Le mémento vous permet de faire des copies complètes de l'état d'un
objet (attributs privés inclus), et les stocke en dehors de l'objet. Ce
patron est surtout connu pour l'utilisation de la fonctionnalité
annuler, mais est également indispensable pour les transactions (par
exemple, pour permettre le rollback d'une opération en erreur).
:::

::: applicability-problem
Utilisez ce patron lorsque vous ne pouvez pas accéder directement aux
attributs/getters/setters d'un objet sans enfreindre les principes de
l'encapsulation.
:::

::: applicability-solution
Le mémento rend l'objet responsable de lui-même, ce qui sécurise les
données de l'état de l'objet. Aucun autre objet ne peut lire les données
de la photo, l'objet original est donc complètement sécurisé.
:::
:::::::
::::::::

::: {.section .checklist}
##  Mise en œuvre {#checklist}

1.  Déterminez la classe qui jouera le rôle du créateur. Il est
    important de savoir si le programme va utiliser un seul objet
    central ou plusieurs petits objets.

2.  Créez la classe mémento. Déclarez un à un l'ensemble des attributs
    qui recopient les attributs de la classe créateur.

3.  Rendez la classe mémento non modifiable. Un mémento ne doit écrire
    ses données qu'une seule fois via son constructeur. Sa classe ne
    doit pas avoir de setters.

4.  Si votre langage de programmation accepte les classes imbriquées,
    mettez le mémento à l'intérieur du créateur. Sinon, extrayez une
    interface vide depuis la classe du mémento et obligez les autres
    objets à utiliser cette interface pour y accéder. Vous pouvez
    ajouter des opérations sur les métadonnées dans l'interface, mais
    rien qui ne puisse exposer l'état du créateur.

5.  Ajoutez une méthode qui crée des mémentos à la classe créateur. Le
    créateur doit passer son état dans un ou plusieurs paramètres du
    constructeur du mémento.

    Le type de la valeur de retour de la méthode doit être l'interface
    extraite dans l'étape précédente (si vous avez suivi cette étape).
    La méthode qui crée le mémento doit directement manipuler la classe
    mémento.

6.  Écrivez la méthode qui permet de rétablir l'état du créateur dans sa
    propre classe. Il doit prendre un objet mémento en paramètre. Si
    vous avez extrait une interface lors de l'une des étapes
    précédentes, c'est le type de la valeur de retour que cette méthode
    doit accepter. Dans ce cas, vous devez caster cet objet dans la
    classe mémento, car le créateur a besoin d'un accès total à cet
    objet.

7.  Le gardien, que ce soit une commande, un historique ou n'importe
    quoi d'autre, doit savoir quand demander de nouveaux mémentos au
    créateur, comment les stocker et quand restaurer le créateur à
    l'aide d'un mémento donné.

8.  Le lien entre les gardiens et les créateurs peut être déplacé dans
    la classe mémento. Dans ce cas, chaque mémento doit être associé à
    son propre créateur. La méthode de restauration doit également être
    déplacée dans la classe mémento. Mais faites-le uniquement si la
    classe mémento est imbriquée dans son créateur, ou si la classe du
    créateur fournit assez de setters pour redéfinir son état.
:::

:::::: {.section .pros-cons}
##  Avantages et inconvénients {#pros-cons}

::::: row
::: col-sm-6
-    Vous pouvez prendre des instantanés de l'état d'un objet tout en
    respectant son encapsulation.
-    Vous pouvez simplifier le code du créateur en laissant le gardien
    gérer l'historique de l'état du créateur.
:::

::: col-sm-6
-    L'application pourrait consommer beaucoup de RAM si les clients
    créent des mémentos trop souvent.
-    Les gardiens doivent garder la trace du cycle de vie des créateurs
    pour pouvoir détruire les mémentos obsolètes.
-    La majorité des langages de programmation dynamiques comme le PHP,
    Python ou le JavaScript ne peuvent pas garantir que l'état
    sauvegardé dans le mémento ne sera pas modifié.
:::
:::::
::::::

::: {.section .relations}
##  Liens avec les autres patrons {#relations}

-   Vous pouvez utiliser la [Commande](command.html) et le
    [Mémento](memento.html) ensemble pour implémenter la fonctionnalité
    « annuler ». Dans ce cas, les commandes ont la responsabilité
    d'exécuter les divers traitements sur un objet cible. Les mémentos
    sauvegardent l'état de cet objet juste avant le lancement de la
    commande.

-   Vous pouvez utiliser le [Mémento](memento.html) avec
    l'[Itérateur](iterator.html) pour récupérer l'état actuel de
    l'itération et rétablir sa valeur plus tard si besoin.

-   Parfois le [Prototype](prototype.html) peut venir remplacer le
    [Mémento](memento.html) et proposer une solution plus simple. Cela
    n'est possible que si l'objet (l'état que vous voulez stocker dans
    l'historique) est assez direct et ne possède pas de liens vers des
    ressources externes, ou que les liens sont faciles à recréer.
:::

::: {.section .implementations}
##  Exemples de code {#implementations}

[![Mémento en
C#](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](memento/csharp/example.html "Mémento en C#"){.prog-lang-link}
[![Mémento en
C++](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](memento/cpp/example.html "Mémento en C++"){.prog-lang-link}
[![Mémento en
Go](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](memento/go/example.html "Mémento en Go"){.prog-lang-link}
[![Mémento en
Java](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](memento/java/example.html "Mémento en Java"){.prog-lang-link}
[![Mémento en
PHP](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](memento/php/example.html "Mémento en PHP"){.prog-lang-link}
[![Mémento en
Python](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](memento/python/example.html "Mémento en Python"){.prog-lang-link}
[![Mémento en
Ruby](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](memento/ruby/example.html "Mémento en Ruby"){.prog-lang-link}
[![Mémento en
Rust](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](memento/rust/example.html "Mémento en Rust"){.prog-lang-link}
[![Mémento en
Swift](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](memento/swift/example.html "Mémento en Swift"){.prog-lang-link}
[![Mémento en
TypeScript](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](memento/typescript/example.html "Mémento en TypeScript"){.prog-lang-link}
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

[Observateur []{.fa .fa-arrow-right}](observer.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Retour

[[]{.fa .fa-arrow-left} Médiateur](mediator.html){.btn .btn-default
rel="prev"}
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
