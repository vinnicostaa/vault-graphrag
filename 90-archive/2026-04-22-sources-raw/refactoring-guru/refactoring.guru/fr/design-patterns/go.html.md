:::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::: {.page .text}
:::: {.illustration-container data-width="960" data-height="720"}
![Les patrons de conception en
Go](../../images/patterns/languages/go49a2.png?id=70faef6e92828d6bc0c486bc59081e22){.dp-bg
width="960" loading="lazy"
srcset="/images/patterns/languages/go-2x.png?id=aa34ed3bdda3e9e29bf73f24250a3082 2x,/images/patterns/languages/go-3x.png 3x"}

::: mob-image
![Les patrons de conception en
Go](../../images/patterns/languages/mini/go561f.png?id=4ae96539199ae1c0b15d49fccc7d84cc){.dp-bg-mini
width="256" loading="lazy"
srcset="/images/patterns/languages/mini/go-2x.png?id=3246b120fdee2b5b942c937509404f16 2x,/images/patterns/languages/mini/go-3x.png 3x"}
:::

# [Les patrons]{.dp1-h-1} [de]{.dp-h-article} [conception]{.dp1-h-2} [en]{.dp-lang-in} [Go]{.dp-lang} {#les-patrons-de-conception-en-go .dp1-h .dp-abs .dp-c .dp-h1}
::::

::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::: dp-text
## Le catalogue des exemples en **Go** {#le-catalogue-des-exemples-en-go .dp-h2}

::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::: dp-double-cols
#### Patrons de création {#patrons-de-création .dp-h4 .dp-creational-title}

:::::::: {.dp-pattern-item .dp-creational}
::: dp-pattern-card
![Fabrique
abstraite](../../images/patterns/cards/abstract-factory-minid1c5.png?id=4c3927c446313a38ce77dfee38111e27){srcset="/images/patterns/cards/abstract-factory-mini-2x.png?id=22236aaa65ff52cbde1c713216d52c1f 2x,/images/patterns/cards/abstract-factory-mini-3x.png 3x"}
:::

#### Fabrique abstraite []{.dp-popularity} {#fabrique-abstraite .dp-pattern-title}

::: dp-pattern-intent
Permet de créer des familles d'objets apparentés sans préciser leur
classe concrète.
:::

::::: dp-pattern-links
<div>

[ Article principal](abstract-factory.html){.dp-pattern-link}

</div>

<div>

[ Exemple de
code](abstract-factory/go/example.html#example-0){.dp-pattern-link}

</div>
:::::
::::::::

:::::::: {.dp-pattern-item .dp-creational}
::: dp-pattern-card
![Monteur](../../images/patterns/cards/builder-mini6d24.png?id=19b95fd05e6469679752c0554b116815){srcset="/images/patterns/cards/builder-mini-2x.png?id=de6d0938678b86903a1426dddfdd13bf 2x,/images/patterns/cards/builder-mini-3x.png 3x"}
:::

#### Monteur []{.dp-popularity} {#monteur .dp-pattern-title}

::: dp-pattern-intent
Permet de construire des objets complexes étape par étape. Ce patron
permet de construire différentes variations ou représentations d'un
objet en utilisant le même code de construction.
:::

::::: dp-pattern-links
<div>

[ Article principal](builder.html){.dp-pattern-link}

</div>

<div>

[ Exemple de code](builder/go/example.html#example-0){.dp-pattern-link}

</div>
:::::
::::::::

:::::::: {.dp-pattern-item .dp-creational}
::: dp-pattern-card
![Fabrique](../../images/patterns/cards/factory-method-minid7f6.png?id=72619e9527893374b98a5913779ac167){srcset="/images/patterns/cards/factory-method-mini-2x.png?id=fa9d4a8d61a67cc3822e52b9daf69dad 2x,/images/patterns/cards/factory-method-mini-3x.png 3x"}
:::

#### Fabrique []{.dp-popularity} {#fabrique .dp-pattern-title}

::: dp-pattern-intent
Définit une interface pour la création d'objets dans une classe mère,
mais délègue aux sous-classes le choix des types d'objets à créer.
:::

::::: dp-pattern-links
<div>

[ Article principal](factory-method.html){.dp-pattern-link}

</div>

<div>

[ Exemple de
code](factory-method/go/example.html#example-0){.dp-pattern-link}

</div>
:::::
::::::::

:::::::: {.dp-pattern-item .dp-creational}
::: dp-pattern-card
![Prototype](../../images/patterns/cards/prototype-mini6540.png?id=bc3046bb39ff36574c08d49839fd1c8e){srcset="/images/patterns/cards/prototype-mini-2x.png?id=b871f789a736e7efbb1cd082d2de6398 2x,/images/patterns/cards/prototype-mini-3x.png 3x"}
:::

#### Prototype []{.dp-popularity} {#prototype .dp-pattern-title}

::: dp-pattern-intent
Permet de créer de nouveaux objets à partir d'objets existants sans
rendre le code dépendant de leur classe.
:::

::::: dp-pattern-links
<div>

[ Article principal](prototype.html){.dp-pattern-link}

</div>

<div>

[ Exemple de
code](prototype/go/example.html#example-0){.dp-pattern-link}

</div>
:::::
::::::::

:::::::: {.dp-pattern-item .dp-creational}
::: dp-pattern-card
![Singleton](../../images/patterns/cards/singleton-minif559.png?id=914e1565dfdf15f240e766163bd303ec){srcset="/images/patterns/cards/singleton-mini-2x.png?id=4a7348083d7719d02709a5c9ff878b9f 2x,/images/patterns/cards/singleton-mini-3x.png 3x"}
:::

#### Singleton []{.dp-popularity} {#singleton .dp-pattern-title}

::: dp-pattern-intent
Permet de garantir que l'instance d'une classe n'existe qu'en un seul
exemplaire, tout en fournissant un point d'accès global à cette
instance.
:::

::::: dp-pattern-links
<div>

[ Article principal](singleton.html){.dp-pattern-link}

</div>

<div>

[ Exemple de
code](singleton/go/example.html#example-0){.dp-pattern-link}

</div>
:::::
::::::::

#### Patrons structurels {#patrons-structurels .dp-h4 .dp-structural-title}

:::::::: {.dp-pattern-item .dp-structural}
::: dp-pattern-card
![Adaptateur](../../images/patterns/cards/adapter-minid866.png?id=b2ee4f681fb589be5a0685b94692aebb){srcset="/images/patterns/cards/adapter-mini-2x.png?id=8274d99afbbe9c63bfbfd0d68ceeffc7 2x,/images/patterns/cards/adapter-mini-3x.png 3x"}
:::

#### Adaptateur []{.dp-popularity} {#adaptateur .dp-pattern-title}

::: dp-pattern-intent
Permet de faire collaborer des objets ayant des interfaces normalement
incompatibles.
:::

::::: dp-pattern-links
<div>

[ Article principal](adapter.html){.dp-pattern-link}

</div>

<div>

[ Exemple de code](adapter/go/example.html#example-0){.dp-pattern-link}

</div>
:::::
::::::::

:::::::: {.dp-pattern-item .dp-structural}
::: dp-pattern-card
![Pont](../../images/patterns/cards/bridge-mini9e60.png?id=b389101d8ee8e23ffa1b534c704d0774){srcset="/images/patterns/cards/bridge-mini-2x.png?id=2622384cf623ed150ee9c21a0812dd87 2x,/images/patterns/cards/bridge-mini-3x.png 3x"}
:::

#### Pont []{.dp-popularity} {#pont .dp-pattern-title}

::: dp-pattern-intent
Permet de séparer une grosse classe ou un ensemble de classes connexes
en deux hiérarchies --- abstraction et implémentation --- qui peuvent
évoluer indépendamment l'une de l'autre.
:::

::::: dp-pattern-links
<div>

[ Article principal](bridge.html){.dp-pattern-link}

</div>

<div>

[ Exemple de code](bridge/go/example.html#example-0){.dp-pattern-link}

</div>
:::::
::::::::

:::::::: {.dp-pattern-item .dp-structural}
::: dp-pattern-card
![Composite](../../images/patterns/cards/composite-minib543.png?id=a369d98d18b417f255d04568fd0131b8){srcset="/images/patterns/cards/composite-mini-2x.png?id=3f7f811fefeb0b64f6774746eb42af09 2x,/images/patterns/cards/composite-mini-3x.png 3x"}
:::

#### Composite []{.dp-popularity} {#composite .dp-pattern-title}

::: dp-pattern-intent
Permet d\'agencer les objets dans des arborescences afin de pouvoir
traiter celles-ci comme des objets individuels.
:::

::::: dp-pattern-links
<div>

[ Article principal](composite.html){.dp-pattern-link}

</div>

<div>

[ Exemple de
code](composite/go/example.html#example-0){.dp-pattern-link}

</div>
:::::
::::::::

:::::::: {.dp-pattern-item .dp-structural}
::: dp-pattern-card
![Décorateur](../../images/patterns/cards/decorator-mini6adb.png?id=d30458908e315af195cb183bc52dbef9){srcset="/images/patterns/cards/decorator-mini-2x.png?id=3b58e540d7d28523080cad341ed9b2e9 2x,/images/patterns/cards/decorator-mini-3x.png 3x"}
:::

#### Décorateur []{.dp-popularity} {#décorateur .dp-pattern-title}

::: dp-pattern-intent
Permet d'affecter dynamiquement de nouveaux comportements à des objets
en les plaçant dans des emballeurs qui implémentent ces comportements.
:::

::::: dp-pattern-links
<div>

[ Article principal](decorator.html){.dp-pattern-link}

</div>

<div>

[ Exemple de
code](decorator/go/example.html#example-0){.dp-pattern-link}

</div>
:::::
::::::::

:::::::: {.dp-pattern-item .dp-structural}
::: dp-pattern-card
![Façade](../../images/patterns/cards/facade-mini14fd.png?id=71ad6fa98b168c11cb3a1a9517dedf78){srcset="/images/patterns/cards/facade-mini-2x.png?id=d4cc6a5d81a31143cc665f7ac1481ac8 2x,/images/patterns/cards/facade-mini-3x.png 3x"}
:::

#### Façade []{.dp-popularity} {#façade .dp-pattern-title}

::: dp-pattern-intent
Procure une interface qui offre un accès simplifié à une librairie, un
framework ou à n'importe quel ensemble complexe de classes.
:::

::::: dp-pattern-links
<div>

[ Article principal](facade.html){.dp-pattern-link}

</div>

<div>

[ Exemple de code](facade/go/example.html#example-0){.dp-pattern-link}

</div>
:::::
::::::::

:::::::: {.dp-pattern-item .dp-structural}
::: dp-pattern-card
![Poids
mouche](../../images/patterns/cards/flyweight-minie3d8.png?id=422ca8d2f90614dce810a8812c626698){srcset="/images/patterns/cards/flyweight-mini-2x.png?id=857ca676fff84317bb0dab76abfce08e 2x,/images/patterns/cards/flyweight-mini-3x.png 3x"}
:::

#### Poids mouche []{.dp-popularity} {#poids-mouche .dp-pattern-title}

::: dp-pattern-intent
Permet de stocker plus d'objets dans la RAM en partageant les états
similaires entre de multiples objets, plutôt que de stocker les données
dans chaque objet.
:::

::::: dp-pattern-links
<div>

[ Article principal](flyweight.html){.dp-pattern-link}

</div>

<div>

[ Exemple de
code](flyweight/go/example.html#example-0){.dp-pattern-link}

</div>
:::::
::::::::

:::::::: {.dp-pattern-item .dp-structural}
::: dp-pattern-card
![Procuration](../../images/patterns/cards/proxy-minid8d7.png?id=25890b11e7dc5af29625ccd0678b63a8){srcset="/images/patterns/cards/proxy-mini-2x.png?id=8638fac9dc08c992852492f9cb29d9c6 2x,/images/patterns/cards/proxy-mini-3x.png 3x"}
:::

#### Procuration []{.dp-popularity} {#procuration .dp-pattern-title}

::: dp-pattern-intent
Permet de fournir un substitut d'un objet. Une procuration donne le
contrôle sur l'objet original, vous permettant d'effectuer des
manipulations avant ou après que la demande ne lui parvienne.
:::

::::: dp-pattern-links
<div>

[ Article principal](proxy.html){.dp-pattern-link}

</div>

<div>

[ Exemple de code](proxy/go/example.html#example-0){.dp-pattern-link}

</div>
:::::
::::::::
:::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::

#### Patrons comportementaux {#patrons-comportementaux .dp-h4 .dp-behavioral-title}

::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::: dp-double-cols
:::::::: {.dp-pattern-item .dp-behavioral}
::: dp-pattern-card
![Chaîne de
responsabilité](../../images/patterns/cards/chain-of-responsibility-mini6d42.png?id=36d85eba8d14986f053123de17aac7a7){srcset="/images/patterns/cards/chain-of-responsibility-mini-2x.png?id=8c81f7069e51259b2443801b91135f7f 2x,/images/patterns/cards/chain-of-responsibility-mini-3x.png 3x"}
:::

#### Chaîne de responsabilité []{.dp-popularity} {#chaîne-de-responsabilité .dp-pattern-title}

::: dp-pattern-intent
Permet de faire circuler une demande dans une chaîne de handlers.
Lorsqu\'un handler reçoit une demande, il décide de la traiter ou de
l'envoyer au handler suivant de la chaîne.
:::

::::: dp-pattern-links
<div>

[ Article principal](chain-of-responsibility.html){.dp-pattern-link}

</div>

<div>

[ Exemple de
code](chain-of-responsibility/go/example.html#example-0){.dp-pattern-link}

</div>
:::::
::::::::

:::::::: {.dp-pattern-item .dp-behavioral}
::: dp-pattern-card
![Commande](../../images/patterns/cards/command-mini8482.png?id=b149eda017c0583c1e92343b83cfb1eb){srcset="/images/patterns/cards/command-mini-2x.png?id=e5f6332e057f6d352a209da963a8fc54 2x,/images/patterns/cards/command-mini-3x.png 3x"}
:::

#### Commande []{.dp-popularity} {#commande .dp-pattern-title}

::: dp-pattern-intent
Prend une action à effectuer et la transforme en un objet autonome qui
contient tous les détails de cette action. Cette transformation permet
de paramétrer des méthodes avec différentes actions, planifier leur
exécution, les mettre dans une file d'attente ou d'annuler des
opérations effectuées.
:::

::::: dp-pattern-links
<div>

[ Article principal](command.html){.dp-pattern-link}

</div>

<div>

[ Exemple de code](command/go/example.html#example-0){.dp-pattern-link}

</div>
:::::
::::::::

:::::::: {.dp-pattern-item .dp-behavioral}
::: dp-pattern-card
![Itérateur](../../images/patterns/cards/iterator-minie2c2.png?id=76c28bb48f997b36965983dd2b41f02e){srcset="/images/patterns/cards/iterator-mini-2x.png?id=7d28f66a487066774a743cfceaa40ea4 2x,/images/patterns/cards/iterator-mini-3x.png 3x"}
:::

#### Itérateur []{.dp-popularity} {#itérateur .dp-pattern-title}

::: dp-pattern-intent
Permet de parcourir les éléments d'une collection sans révéler sa
représentation interne (liste, pile, arbre, etc.).
:::

::::: dp-pattern-links
<div>

[ Article principal](iterator.html){.dp-pattern-link}

</div>

<div>

[ Exemple de code](iterator/go/example.html#example-0){.dp-pattern-link}

</div>
:::::
::::::::

:::::::: {.dp-pattern-item .dp-behavioral}
::: dp-pattern-card
![Médiateur](../../images/patterns/cards/mediator-mini1561.png?id=a7e43ee8e17e4474737b1fcb3201d7ba){srcset="/images/patterns/cards/mediator-mini-2x.png?id=d288d7c73f5581ae09701d61200ae09e 2x,/images/patterns/cards/mediator-mini-3x.png 3x"}
:::

#### Médiateur []{.dp-popularity} {#médiateur .dp-pattern-title}

::: dp-pattern-intent
Permet de diminuer les dépendances chaotiques entre les objets. Ce
patron restreint les communications directes entre les objets et les
force à collaborer uniquement via un objet médiateur.
:::

::::: dp-pattern-links
<div>

[ Article principal](mediator.html){.dp-pattern-link}

</div>

<div>

[ Exemple de code](mediator/go/example.html#example-0){.dp-pattern-link}

</div>
:::::
::::::::

:::::::: {.dp-pattern-item .dp-behavioral}
::: dp-pattern-card
![Mémento](../../images/patterns/cards/memento-mini723d.png?id=8b2ea4dc2c5d15775a654808cc9de099){srcset="/images/patterns/cards/memento-mini-2x.png?id=1d7cba189261dd84b11369a6838b1055 2x,/images/patterns/cards/memento-mini-3x.png 3x"}
:::

#### Mémento []{.dp-popularity} {#mémento .dp-pattern-title}

::: dp-pattern-intent
Permet de sauvegarder et de rétablir l\'état précédent d'un objet sans
révéler les détails de son implémentation.
:::

::::: dp-pattern-links
<div>

[ Article principal](memento.html){.dp-pattern-link}

</div>

<div>

[ Exemple de code](memento/go/example.html#example-0){.dp-pattern-link}

</div>
:::::
::::::::

:::::::: {.dp-pattern-item .dp-behavioral}
::: dp-pattern-card
![Observateur](../../images/patterns/cards/observer-minifa2e.png?id=fd2081ab1cff29c60b499bcf6a62786a){srcset="/images/patterns/cards/observer-mini-2x.png?id=f205b0655572ac8e4636691c0e0debfd 2x,/images/patterns/cards/observer-mini-3x.png 3x"}
:::

#### Observateur []{.dp-popularity} {#observateur .dp-pattern-title}

::: dp-pattern-intent
Permet de mettre en place un mécanisme de souscription pour envoyer des
notifications à plusieurs objets, au sujet d'événements concernant les
objets qu'ils observent.
:::

::::: dp-pattern-links
<div>

[ Article principal](observer.html){.dp-pattern-link}

</div>

<div>

[ Exemple de code](observer/go/example.html#example-0){.dp-pattern-link}

</div>
:::::
::::::::

:::::::: {.dp-pattern-item .dp-behavioral}
::: dp-pattern-card
![État](../../images/patterns/cards/state-mini63fb.png?id=f4018837e0641d1dade756b6678fd4ee){srcset="/images/patterns/cards/state-mini-2x.png?id=7e24398b27a43c7bd286fc0ea54d2a35 2x,/images/patterns/cards/state-mini-3x.png 3x"}
:::

#### État []{.dp-popularity} {#état .dp-pattern-title}

::: dp-pattern-intent
Modifie le comportement d'un objet lorsque son état interne change.
L'objet donne l'impression qu'il change de classe.
:::

::::: dp-pattern-links
<div>

[ Article principal](state.html){.dp-pattern-link}

</div>

<div>

[ Exemple de code](state/go/example.html#example-0){.dp-pattern-link}

</div>
:::::
::::::::

:::::::: {.dp-pattern-item .dp-behavioral}
::: dp-pattern-card
![Stratégie](../../images/patterns/cards/strategy-mini28f6.png?id=d38abee4fb6f2aed909d262bdadca936){srcset="/images/patterns/cards/strategy-mini-2x.png?id=f4e6608561f8e5d18be6927d4620ad29 2x,/images/patterns/cards/strategy-mini-3x.png 3x"}
:::

#### Stratégie []{.dp-popularity} {#stratégie .dp-pattern-title}

::: dp-pattern-intent
Permet de définir une famille d'algorithmes, de les mettre dans des
classes séparées et de rendre leurs objets interchangeables.
:::

::::: dp-pattern-links
<div>

[ Article principal](strategy.html){.dp-pattern-link}

</div>

<div>

[ Exemple de code](strategy/go/example.html#example-0){.dp-pattern-link}

</div>
:::::
::::::::

:::::::: {.dp-pattern-item .dp-behavioral}
::: dp-pattern-card
![Patron de
méthode](../../images/patterns/cards/template-method-mini56e3.png?id=9f200248d88026d8e79d0f3dae411ab4){srcset="/images/patterns/cards/template-method-mini-2x.png?id=178bf56e39b3a1f548dd636076209c98 2x,/images/patterns/cards/template-method-mini-3x.png 3x"}
:::

#### Patron de méthode []{.dp-popularity} {#patron-de-méthode .dp-pattern-title}

::: dp-pattern-intent
Permet de mettre le squelette d'un algorithme dans la classe mère, mais
laisse les sous-classes redéfinir certaines étapes de l'algorithme sans
changer sa structure.
:::

::::: dp-pattern-links
<div>

[ Article principal](template-method.html){.dp-pattern-link}

</div>

<div>

[ Exemple de
code](template-method/go/example.html#example-0){.dp-pattern-link}

</div>
:::::
::::::::

:::::::: {.dp-pattern-item .dp-behavioral}
::: dp-pattern-card
![Visiteur](../../images/patterns/cards/visitor-mini0a1c.png?id=854a35a62963bec1d75eab996918989b){srcset="/images/patterns/cards/visitor-mini-2x.png?id=9b87f3f3b772f434b28a25876829b504 2x,/images/patterns/cards/visitor-mini-3x.png 3x"}
:::

#### Visiteur []{.dp-popularity} {#visiteur .dp-pattern-title}

::: dp-pattern-intent
Permet de séparer les algorithmes et les objets sur lesquels ils
opèrent.
:::

::::: dp-pattern-links
<div>

[ Article principal](visitor.html){.dp-pattern-link}

</div>

<div>

[ Exemple de code](visitor/go/example.html#example-0){.dp-pattern-link}

</div>
:::::
::::::::
:::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::
