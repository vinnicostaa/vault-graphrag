::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [Patrons de
conception](../design-patterns.html){.type} / [Patrons
comportementaux](behavioral-patterns.html){.type}
:::

# Commande {#commande .title}

::: aka
Alias :
[Action, ]{style="display:inline-block"}[Translations, ]{style="display:inline-block"}[Command]{style="display:inline-block"}
:::

::: {.section .intent}
##  Intention {#intent}

**Commande** est un patron de conception comportemental qui prend une
action à effectuer et la transforme en un objet autonome qui contient
tous les détails de cette action. Cette transformation permet de
paramétrer des méthodes avec différentes actions, planifier leur
exécution, les mettre dans une file d'attente ou d'annuler des
opérations effectuées.

<figure class="image">
<img
src="../../images/patterns/content/command/command-frbcab.png?id=25f8e28bed64e029dee83fbcff1dd970"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/command/command-fr.png?id=25f8e28bed64e029dee83fbcff1dd970 640w,/images/patterns/content/command/command-fr-1.5x.png?id=8514d55e06aa8118aeb34e910f97a9f2 960w,/images/patterns/content/command/command-fr-2x.png?id=b9df85ccd4dad98641fbb5c597145758 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Patron de conception commande" />
</figure>
:::

::: {.section .problem}
##  Problème {#problem}

Imaginez-vous en train de développer un nouvel éditeur de texte. Vous
travaillez actuellement sur la création d'une barre d'outils équipée
d'un ensemble de boutons qui permettent d'effectuer diverses opérations
dans l'éditeur. Vous avez créé une belle classe `Bouton` qui peut être
utilisée pour les boutons de la barre d'outils ou pour les boutons
génériques des différentes boîtes de dialogue.

<figure class="image">
<img
src="../../images/patterns/diagrams/command/problem163e2.png?id=84189315a0e8d91da262792979005ab4"
style="aspect-ratio: 0.88;"
srcset="/images/patterns/diagrams/command/problem1.png?id=84189315a0e8d91da262792979005ab4 230w,/images/patterns/diagrams/command/problem1-1.5x.png?id=77bf0bc58e5c9c9b8369bc4f8dec457f 345w,/images/patterns/diagrams/command/problem1-2x.png?id=af4c4e9c1d1b4fa2c4104c5f6bb18719 460w"
sizes="(max-width: 720px) 100vw, 230px" loading="lazy" width="230"
alt="Problème résolu grâce au patron de conception commande" />
<figcaption><p>Tous les boutons de l’application sont dérivés de la
même classe.</p></figcaption>
</figure>

Ces boutons ont beau se ressembler, ils sont censés effectuer des
actions différentes. Où va-t-on bien pouvoir mettre le code des actions
que ces différents boutons déclenchent ? Le plus simple est de créer des
tonnes de sous-classes où les boutons sont utilisés. Ces sous-classes
vont contenir le code exécuté lors d'un clic sur un bouton.

<figure class="image">
<img
src="../../images/patterns/diagrams/command/problem2f2f9.png?id=f0e33da1842b3a3ee3b4857de0b6ec93"
style="aspect-ratio: 2.11;"
srcset="/images/patterns/diagrams/command/problem2.png?id=f0e33da1842b3a3ee3b4857de0b6ec93 400w,/images/patterns/diagrams/command/problem2-1.5x.png?id=7ae15e2b07d5d076a878d99c3ed8ac32 600w,/images/patterns/diagrams/command/problem2-2x.png?id=5eea7d0f6247da011150bb63e423f405 800w"
sizes="(max-width: 720px) 100vw, 400px" loading="lazy" width="400"
alt="De nombreuses sous-classes de boutons" />
<figcaption><p>De nombreuses sous-classes de boutons. Tout semble aller
pour le mieux.</p></figcaption>
</figure>

Mais bientôt, vous allez remarquer que cette approche comporte de
sérieux défauts. Le premier que nous allons aborder ici, est le fait que
vous avez créé énormément de sous-classes. Lors de chaque modification
de la classe `Bouton`, vous risquez d'ajouter de nouveaux bugs dans le
code. Pour faire simple, votre GUI (interface graphique utilisateur) est
devenue un peu trop dépendante du code volatile de la logique métier.

<figure class="image">
<img
src="../../images/patterns/diagrams/command/problem3-fr4bc6.png?id=8b7df353030bb9544ebd231329cda839"
style="aspect-ratio: 4.36;"
srcset="/images/patterns/diagrams/command/problem3-fr.png?id=8b7df353030bb9544ebd231329cda839 480w,/images/patterns/diagrams/command/problem3-fr-1.5x.png?id=cf48dba6fe04c802af2e03b54b0b25ee 720w,/images/patterns/diagrams/command/problem3-fr-2x.png?id=ff7b2d15d9baaaf40d2dc310a1e98807 960w"
sizes="(max-width: 720px) 100vw, 480px" loading="lazy" width="480"
alt="Plusieurs classes implémentent la même fonctionnalité" />
<figcaption><p>Plusieurs classes implémentent la
même fonctionnalité.</p></figcaption>
</figure>

Abordons à présent le défaut majeur. Certaines opérations, comme
copier-coller du texte, doivent être appelées depuis différents
endroits. Par exemple, un utilisateur peut cliquer sur un petit bouton
« copier » de la barre d'outils, copier via le menu contextuel ou encore
faire `Ctrl+C` sur son clavier.

À l'époque où notre application n'avait qu'une barre d'outils, cela ne
posait pas de problème de mettre l'implémentation des différentes
actions dans la sous-classe du bouton. Vous pouviez écrire le code de la
copie du texte dans la sous-classe `BoutonCopier` sans problème. Mais
lorsque vous implémentez des menus contextuels, des raccourcis ou autres
fonctionnalités du même ordre, vous devez dupliquer le code de ces
actions dans plusieurs classes ou rendre les menus dépendants des
boutons, ce qui est encore pire.
:::

::: {.section .solution}
##  Solution

Un logiciel bien conçu repose souvent sur le *principe de séparation des
préoccupations*, qui est en général obtenu en décomposant l'application
en plusieurs couches. Le cas le plus classique consiste à avoir une
couche pour la GUI et une autre pour la logique métier. La couche GUI a
la responsabilité d'afficher une belle image à l'écran, de détecter les
actions de l'utilisateur et de l'application, et de les répercuter à
l'écran. Mais en ce qui concerne l'exécution des traitements importants
comme calculer la trajectoire vers la lune ou générer les rapports
annuels, la couche GUI délègue le travail à la couche métier.

Dans le code, on se retrouve avec un objet GUI qui appelle une méthode
de l'objet de la logique métier et lui passe des paramètres. Ce
processus est souvent décrit comme un objet envoyant une *demande* à un
autre.

<figure class="image">
<img
src="../../images/patterns/diagrams/command/solution1-fr8037.png?id=8da0a4d5dab6060349ff9e88989b883f"
style="aspect-ratio: 2.47;"
srcset="/images/patterns/diagrams/command/solution1-fr.png?id=8da0a4d5dab6060349ff9e88989b883f 470w,/images/patterns/diagrams/command/solution1-fr-1.5x.png?id=a68f8233cb0ed2c88930164749594339 705w,/images/patterns/diagrams/command/solution1-fr-2x.png?id=30d55a3c1b145aafd58dad2128748497 940w"
sizes="(max-width: 720px) 100vw, 470px" loading="lazy" width="470"
alt="La couche GUI possède un accès direct à la couche métier" />
<figcaption><p>Les objets de la GUI peuvent accéder directement aux
objets de la couche métier.</p></figcaption>
</figure>

Le patron de conception commande propose de ne pas envoyer ces demandes
directement depuis les objets de la GUI. À la place, vous devez extraire
le détail de ces demandes (un appel vers l'objet, le nom de la méthode
et la liste des paramètres) dans une classe *commande* séparée avec une
unique méthode qui déclenche cette demande.

Les objets commande servent de lien entre les diverses GUI et les objets
métier. À partir de maintenant, l'objet GUI ne saura plus quel objet
métier reçoit la demande et comment il la traite. L'objet GUI déclenche
juste la commande, et cette dernière se charge de gérer les détails.

<figure class="image">
<img
src="../../images/patterns/diagrams/command/solution2-fr583a.png?id=b9afb21422e7e0f2222e11b843dc56e3"
style="aspect-ratio: 2.89;"
srcset="/images/patterns/diagrams/command/solution2-fr.png?id=b9afb21422e7e0f2222e11b843dc56e3 550w,/images/patterns/diagrams/command/solution2-fr-1.5x.png?id=fd705a7aa1ae96c2fa2f6bbb3dbfc9e4 825w,/images/patterns/diagrams/command/solution2-fr-2x.png?id=4a065df99636778f74ae0967b40ece83 1100w"
sizes="(max-width: 720px) 100vw, 550px" loading="lazy" width="550"
alt="Accéder à la couche métier via une commande" />
<figcaption><p>Accéder à la couche métier via
une commande.</p></figcaption>
</figure>

La prochaine étape consiste à implémenter la même interface pour toutes
vos commandes. En général, elle ne possède qu'une seule méthode
d'exécution et ne prend aucun paramètre. Cette interface vous permet
d'utiliser diverses commandes avec le même demandeur (invoker), sans
avoir à la coupler avec les classes concrètes des commandes. En bonus,
vous pouvez dorénavant échanger les objets de la commande liés au
demandeur, ce qui permet de modifier le comportement du demandeur lors
de l'exécution.

Vous vous êtes peut-être rendu compte qu'il manquait une pièce du
puzzle : les paramètres de la demande. Un objet GUI devrait les fournir
à la couche métier. Mais comme la méthode d'exécution de la commande ne
possède aucun paramètre, comment allons-nous nous en sortir ? En fait,
la commande doit être pré configurée avec ces données ou être capable de
les récupérer par elle-même.

<figure class="image">
<img
src="../../images/patterns/diagrams/command/solution3-fr9a48.png?id=fb6a64890eaa54dbe4e111b76581948f"
style="aspect-ratio: 1.83;"
srcset="/images/patterns/diagrams/command/solution3-fr.png?id=fb6a64890eaa54dbe4e111b76581948f 440w,/images/patterns/diagrams/command/solution3-fr-1.5x.png?id=ad45dd0cc54b6153060108601928e162 660w,/images/patterns/diagrams/command/solution3-fr-2x.png?id=d4f1490d3a3c93b14fbe06e83d0614c2 880w"
sizes="(max-width: 720px) 100vw, 440px" loading="lazy" width="440"
alt="Les objets de la GUI délèguent le travail aux commandes" />
<figcaption><p>Les objets de la GUI délèguent le travail
aux commandes.</p></figcaption>
</figure>

Revenons à notre éditeur de texte. Après avoir mis en place le patron de
conception commande, nous n'avons plus besoin de toutes ces sous-classes
pour implémenter le comportement des différents clics. Vous pouvez vous
contenter de mettre un seul attribut dans la classe de base `Bouton` qui
conserve une référence vers un objet commande et lui permet de lancer
cette commande lors d'un clic.

Vous allez implémenter plusieurs classes commande pour chaque opération
possible, et allez les relier avec les boutons en fonction du
comportement attendu.

Vous pouvez implémenter tous les autres éléments de la GUI comme des
menus, raccourcis ou des boîtes de dialogue complètes de la même
manière. Ils seront reliés à une commande qui est exécutée lorsqu'un
utilisateur interagit avec l'élément de la GUI. Vous l'avez probablement
deviné, les éléments qui déclenchent les mêmes actions seront reliés aux
mêmes commandes afin d'éviter la duplication de code.

Les commandes forment ainsi une couche intermédiaire très pratique et
réduisent le couplage entre la GUI et les couches métier. Et nous
n'avons abordé qu'une infime partie des bénéfices que le patron de
conception commande peut apporter.
:::

::: {.section .analogy}
##  Analogie {#analogy}

<figure class="image">
<img
src="../../images/patterns/content/command/command-comic-16e6f.png?id=551df832f445080976f3116e0dc120c9"
style="aspect-ratio: 2;"
srcset="/images/patterns/content/command/command-comic-1.png?id=551df832f445080976f3116e0dc120c9 600w,/images/patterns/content/command/command-comic-1-1.5x.png?id=82dc5c38ce372ed4dfd4a37c04faeb72 900w,/images/patterns/content/command/command-comic-1-2x.png?id=47b3c00b2cfbda7157a1ed9da6ce2948 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Passer une commande au restaurant" />
<figcaption><p>Passer une commande au restaurant.</p></figcaption>
</figure>

Après une longue balade en ville, vous trouvez un restaurant sympa et
vous vous asseyez à une table près de la fenêtre. Un serveur aimable se
rapproche et prend votre commande en l'écrivant sur un bout de papier.
Il se rend dans la cuisine et colle la commande sur le mur. Au bout d'un
moment, la commande arrive au chef, qui la lit et prépare le plat. Le
cuisinier place le repas sur un plateau avec la commande. Le serveur
découvre le plateau, vérifie la commande pour s'assurer que tout est
conforme, et l'apporte à votre table.

Le bout de papier joue le rôle de la commande. Il reste dans une file
d'attente jusqu'à ce que le chef soit prêt à la servir. Il peut
commencer à cuisiner directement, car la commande contient toutes les
informations permettant au chef de préparer le plat. C'est mieux que de
perdre du temps à venir vous demander les détails de la commande.
:::

::::: {.section .structure-container}
##  Structure

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/command/structure44ec.png?id=1cd7833638f4c43630f4a84017d31195"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.7;"
srcset="/images/patterns/diagrams/command/structure.png?id=1cd7833638f4c43630f4a84017d31195 630w,/images/patterns/diagrams/command/structure-1.5x.png?id=6e5b68277c7747b22266e408891d5841 945w,/images/patterns/diagrams/command/structure-2x.png?id=1dfaa84354ffe49ef7ad46ce069482d2 1260w"
sizes="(max-width: 720px) 100vw, 630px" loading="lazy" width="630"
alt="Structure du patron de conception commande" /><img
src="../../images/patterns/diagrams/command/structure-indexed49b8.png?id=95529d7282dc7bc1c5bc443423b1cf4f"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.64;"
srcset="/images/patterns/diagrams/command/structure-indexed.png?id=95529d7282dc7bc1c5bc443423b1cf4f 640w,/images/patterns/diagrams/command/structure-indexed-1.5x.png?id=29d6c5c7a06da2747ed756c0ddad6169 960w,/images/patterns/diagrams/command/structure-indexed-2x.png?id=e4cc286a44768c7d060af33497da7df6 1280w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="640"
alt="Structure du patron de conception commande" />
</figure>
:::

1.  La classe **Demandeur** (*invoker*) a la responsabilité de l'envoi
    des demandes. Elle doit inclure un attribut qui stocke la référence
    à un objet commande. Le demandeur déclenche la commande plutôt que
    de l'envoyer directement au récepteur. Gardez bien en tête que le
    demandeur n'est pas responsable de la création de l'objet commande.
    En général, il s'agit d'une commande créée auparavant dans le
    constructeur du client.

2.  L'interface **Commande** déclare habituellement une seule méthode
    pour exécuter la commande.

3.  Les **Commandes Concrètes** implémentent plusieurs types de
    demandes. Une commande concrète n'est pas censée accomplir la tâche
    d'elle-même. Elle transmet l'appel aux objets de la logique métier.
    Mais pour simplifier le code, ces classes peuvent être fusionnées.

    Les paramètres nécessaires à l'exécution d'une méthode sur un objet
    récepteur peuvent être déclarés comme des attributs de la commande
    concrète. Vous pouvez rendre les objets de la commande non
    modifiables en autorisant uniquement l'initialisation de ces
    attributs dans le constructeur.

4.  La classe **Récepteur** porte la logique métier. À peu près
    n'importe quel objet peut prendre le rôle de récepteur. La majorité
    des commandes se contentent de passer la demande au récepteur et ce
    dernier se charge du traitement.

5.  Le **Client** crée et configure les objets de la commande concrète.
    Il doit passer tous les paramètres de la demande (dont l'instance du
    récepteur) dans le constructeur de la commande. Ensuite, la commande
    qui en résulte peut être associée avec un ou plusieurs demandeurs.
::::
:::::

::: {.section .pseudocode}
##  Pseudo-code {#pseudocode}

Dans cet exemple, le patron de conception **Commande** aide à tracer
l'historique des traitements et est en mesure d'annuler les
modifications effectuées par un traitement.

<figure class="image">
<img
src="../../images/patterns/diagrams/command/example31bf.png?id=1f42c8395fe54d0e409026b91881e2a0"
style="aspect-ratio: 1.07;"
srcset="/images/patterns/diagrams/command/example.png?id=1f42c8395fe54d0e409026b91881e2a0 640w,/images/patterns/diagrams/command/example-1.5x.png?id=435055a78a82c005eebb1bef869998ae 960w,/images/patterns/diagrams/command/example-2x.png?id=ed44dfd9b02eccf1ae7e2714d018ed36 1280w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="640"
alt="Structure du patron de conception commande pour l’exemple utilisé" />
<figcaption><p>Actions pouvant être annulées dans un éditeur
de texte.</p></figcaption>
</figure>

Les commandes qui modifient l'état de l'éditeur (par exemple couper et
coller) font une sauvegarde de son état avant de lancer le traitement
associé à la commande. Une fois que la commande a été exécutée, elle est
placée dans un historique de commandes (une pile d'objets commande) avec
une sauvegarde de l'état actuel de l'éditeur. Si l'utilisateur a besoin
d'annuler un traitement, l'application peut prendre la commande la plus
récente de l'historique, lire la sauvegarde associée à l'état de
l'éditeur et la restaurer.

Le code client (éléments de la GUI, historique des commandes, etc.)
n'est pas couplé avec les classes de la commande concrète, car il
interagit avec les commandes via l'interface commande. Cette approche
vous permet d'ajouter de nouvelles commandes dans l'application sans
endommager l'existant.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// La classe de base commande définit une interface commune pour
// toutes les commandes concrètes.
abstract class Command is
    protected field app: Application
    protected field editor: Editor
    protected field backup: text

    constructor Command(app: Application, editor: Editor) is
        this.app = app
        this.editor = editor

    // Fait une sauvegarde de l’état de l’éditeur.
    method saveBackup() is
        backup = editor.text

    // Rétablit l’état de l’éditeur.
    method undo() is
        editor.text = backup

    // La méthode d’exécution est abstraite pour forcer les
    // commandes concrètes à fournir leurs propres
    // implémentations. La méthode doit retourner un booléen qui
    // indique si la commande a modifié l&#39;état de l’éditeur.
    abstract method execute()


// Les commandes concrètes sont écrites ici.
class CopyCommand extends Command is
    // La commande pour copier n’est pas sauvegardée dans
    // l’historique, car elle ne modifie pas l&#39;état de
    // l’éditeur.
    method execute() is
        app.clipboard = editor.getSelection()
        return false

class CutCommand extends Command is
    // La commande ‘couper’ change l’état de l’éditeur, elle
    // doit donc être sauvegardée dans l’historique. Elle sera
    // sauvegardée tant que la méthode renvoie vrai.
    method execute() is
        saveBackup()
        app.clipboard = editor.getSelection()
        editor.deleteSelection()
        return true

class PasteCommand extends Command is
    method execute() is
        saveBackup()
        editor.replaceSelection(app.clipboard)
        return true

// Le traitement ‘annuler’ est aussi une commande.
class UndoCommand extends Command is
    method execute() is
        app.undo()
        return false


// L’historique principal des commandes est juste une pile.
class CommandHistory is
    private field history: array of Command

    // Dernier entré...
    method push(c: Command) is
        // Pousse la commande à la fin du tableau de
        // l’historique.

    // ...premier sorti.
    method pop():Command is
        // Récupère la commande la plus récente de l’historique.


// La classe éditeur possède des fonctionnalités de traitement
// de texte. Elle joue le rôle du récepteur : toutes les
// commandes finissent par déléguer l’exécution aux méthodes de
// l’éditeur.
class Editor is
    field text: string

    method getSelection() is
        // Retourne le texte sélectionné.

    method deleteSelection() is
        // Supprime le texte sélectionné.

    method replaceSelection(text) is
        // Insère le contenu du presse-papiers à la position
        // actuelle.


// La classe application configure les relations de l’objet.
// Elle agit comme un demandeur : lorsque quelque chose doit
// être fait, elle crée un objet commande et lance son
// traitement.
class Application is
    field clipboard: string
    field editors: array of Editors
    field activeEditor: Editor
    field history: CommandHistory

    // Le code qui assigne les commandes à l’objet UI pourrait
    // ressembler à ceci.
    method createUI() is
        // ...
        copy = function() { executeCommand(
            new CopyCommand(this, activeEditor)) }
        copyButton.setCommand(copy)
        shortcuts.onKeyPress(&quot;Ctrl+C&quot;, copy)

        cut = function() { executeCommand(
            new CutCommand(this, activeEditor)) }
        cutButton.setCommand(cut)
        shortcuts.onKeyPress(&quot;Ctrl+X&quot;, cut)

        paste = function() { executeCommand(
            new PasteCommand(this, activeEditor)) }
        pasteButton.setCommand(paste)
        shortcuts.onKeyPress(&quot;Ctrl+V&quot;, paste)

        undo = function() { executeCommand(
            new UndoCommand(this, activeEditor)) }
        undoButton.setCommand(undo)
        shortcuts.onKeyPress(&quot;Ctrl+Z&quot;, undo)

    // Lance une commande et vérifie si elle doit être ajoutée à
    // l’historique.
    method executeCommand(command) is
        if (command.execute())
            history.push(command)

    // Prend la commande la plus récente dans l’historique et
    // lance sa méthode ‘annuler’. Vous remarquerez que nous ne
    // connaissons pas la classe de la commande. Nous n’avons
    // pas besoin de la connaître puisque la commande sait
    // comment annuler sa propre action.
    method undo() is
        command = history.pop()
        if (command != null)
            command.undo()</code></pre>
</figure>
:::

:::::::::: {.section .applicability-container}
##  Possibilités d'application {#applicability}

::::::::: applicability
::: applicability-problem
Utilisez le patron de conception commande lorsque vous voulez paramétrer
des objets avec des traitements.
:::

::: applicability-solution
Le patron de conception commande peut transformer l'appel d'une méthode
en un objet autonome. Cette approche permet des utilisations très
intéressantes : vous pouvez passer des commandes dans les paramètres
d'une méthode, les stocker à l'intérieur d'objets, modifier les liens
des commandes à l'exécution, etc.

Voici un exemple : vous développez un composant de GUI (un menu
contextuel par exemple) et vous voulez que vos utilisateurs puissent
configurer des éléments de menu qui déclenchent des traitements
lorsqu'un utilisateur final clique dessus.
:::

::: applicability-problem
Utilisez ce patron si vous voulez mettre des traitements dans une file
d'attente, programmer leur déclenchement, ou les exécuter à distance.
:::

::: applicability-solution
Comme pour tous les objets, une commande peut être sérialisée, ce qui
veut dire qu'on la convertit en une chaîne de caractères qui peut
facilement être écrite dans un fichier ou dans une base de données. La
chaîne de caractère pourra ensuite être restaurée et utilisée par
l'objet de la commande originale. Vous pouvez ainsi différer et
planifier l'exécution de la commande. Mais ce n'est pas tout ! De la
même manière, vous pouvez mettre une commande dans une file d'attente,
l'historiser ou l'envoyer par le réseau.
:::

::: applicability-problem
Utilisez le patron de conception commande quand vous voulez implémenter
des opérations réversibles.
:::

::: applicability-solution
Parmi les nombreuses techniques qui permettent d'implémenter la
fonctionnalité annuler/rétablir, le patron de conception commande est
probablement le plus populaire.

Pour pouvoir annuler des traitements, vous devez mettre en place un
historique des actions effectuées. L'historique des commandes est une
pile qui contient tous les objets des commandes exécutées avec l'état de
l'application correspondant.

Cette méthode possède deux défauts. Premièrement, l'état d'une
application n'est pas facile à sauvegarder, car une partie peut être
privée. Ce problème peut être résolu grâce à l'intervention du patron de
conception [Mémento](memento.html).

Deuxièmement, la sauvegarde de l'état risque de consommer beaucoup de
RAM. Par conséquent, vous pouvez utiliser une autre alternative : plutôt
que de rétablir l'état précédent, la commande effectue l'inverse de la
modification, ce qui est malheureusement parfois impossible ou difficile
à mettre en place.
:::
:::::::::
::::::::::

::: {.section .checklist}
##  Mise en œuvre {#checklist}

1.  Déclarez l'interface commande avec une seule méthode d'exécution.

2.  Commencez à récupérer les demandes et à les mettre dans les classes
    concrètes Commande qui implémentent l'interface commande. Chaque
    classe doit comporter un ensemble d'attributs qui permettent de
    stocker les paramètres des demandes, avec une référence à l'objet
    initial du récepteur. Ces valeurs doivent toutes être initialisées
    dans le constructeur de la commande.

3.  Identifiez les classes qui vont jouer le rôle de *Demandeurs*. Créez
    les attributs de ces classes qui vont stocker les commandes. Les
    demandeurs doivent uniquement communiquer avec leurs commandes via
    l'interface commande. En général, ils ne créent pas les objets
    commande, c'est le code client qui s'en charge.

4.  Plutôt que d'envoyer la demande directement au récepteur, modifiez
    les demandeurs afin qu'ils exécutent la commande.

5.  Le client doit initialiser les objets dans l'ordre suivant :

    -   Créer les récepteurs.
    -   Créer les commandes et les associer avec les récepteurs si
        besoin.
    -   Créer les demandeurs et les associer avec les commandes.
:::

:::::: {.section .pros-cons}
##  Avantages et inconvénients {#pros-cons}

::::: row
::: col-sm-6
-    *Principe de responsabilité unique*. Vous pouvez découpler les
    classes qui appellent des traitements, de celles qui les exécutent.
-    *Principe ouvert/fermé*. Vous pouvez ajouter de nouvelles commandes
    dans le programme sans endommager le code client existant.
-    Vous pouvez mettre en place une fonctionnalité annuler-rétablir.
-    Vous pouvez différer l'exécution de vos traitements.
-    Vous pouvez assembler plusieurs commandes simples en une seule plus
    complexe.
:::

::: col-sm-6
-    Le code peut devenir plus compliqué, car vous créez une nouvelle
    couche entre les demandeurs et les récepteurs.
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

-   Les handlers dans la [Chaîne de
    Responsabilité](chain-of-responsibility.html) peuvent être
    implémentés comme des [Commandes](command.html). Dans ce cas, vous
    pouvez lancer beaucoup de traitements différents sur le même objet
    contexte, représenté par une demande.

    Vous pouvez cependant utiliser une autre approche dans laquelle la
    demande en elle-même est un objet *commande*. Vous pouvez ainsi
    exécuter le même traitement dans une série de différents contextes
    connectés par une chaîne.

-   Vous pouvez utiliser la [Commande](command.html) et le
    [Mémento](memento.html) ensemble pour implémenter la fonctionnalité
    « annuler ». Dans ce cas, les commandes ont la responsabilité
    d'exécuter les divers traitements sur un objet cible. Les mémentos
    sauvegardent l'état de cet objet juste avant le lancement de la
    commande.

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

-   Le [Prototype](prototype.html) se révèle utile lorsque vous voulez
    sauvegarder des copies de [Commandes](command.html) dans
    l'historique.

-   Vous pouvez traiter le [Visiteur](visitor.html) comme une version
    plus puissante du patron de conception [Commande](command.html). Ses
    objets peuvent lancer des traitements sur divers objets dans
    différentes classes.
:::

::: {.section .implementations}
##  Exemples de code {#implementations}

[![Commande en
C#](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](command/csharp/example.html "Commande en C#"){.prog-lang-link}
[![Commande en
C++](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](command/cpp/example.html "Commande en C++"){.prog-lang-link}
[![Commande en
Go](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](command/go/example.html "Commande en Go"){.prog-lang-link}
[![Commande en
Java](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](command/java/example.html "Commande en Java"){.prog-lang-link}
[![Commande en
PHP](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](command/php/example.html "Commande en PHP"){.prog-lang-link}
[![Commande en
Python](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](command/python/example.html "Commande en Python"){.prog-lang-link}
[![Commande en
Ruby](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](command/ruby/example.html "Commande en Ruby"){.prog-lang-link}
[![Commande en
Rust](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](command/rust/example.html "Commande en Rust"){.prog-lang-link}
[![Commande en
Swift](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](command/swift/example.html "Commande en Swift"){.prog-lang-link}
[![Commande en
TypeScript](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](command/typescript/example.html "Commande en TypeScript"){.prog-lang-link}
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

[Itérateur []{.fa .fa-arrow-right}](iterator.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Retour

[[]{.fa .fa-arrow-left} Chaîne de
responsabilité](chain-of-responsibility.html){.btn .btn-default
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
