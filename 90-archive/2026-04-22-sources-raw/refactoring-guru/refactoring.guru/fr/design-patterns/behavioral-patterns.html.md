::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::: {.main-content-container .center-content-container}
::::::: {.page .text}
::: breadcrumb
[](../../index.html){.home} / [Patrons de
conception](../design-patterns.html){.type} /
[Catalogue](catalog.html){.type}
:::

# Patrons comportementaux {#patrons-comportementaux .title}

Les patrons comportementaux s'occupent des algorithmes et de la
répartition des responsabilités entre les objets.

::: {.d-flex .flex-wrap .justify-content-between}
[[ [![Chaîne de
responsabilité](../../images/patterns/cards/chain-of-responsibility-mini6d42.png?id=36d85eba8d14986f053123de17aac7a7){srcset="/images/patterns/cards/chain-of-responsibility-mini-2x.png?id=8c81f7069e51259b2443801b91135f7f 2x,/images/patterns/cards/chain-of-responsibility-mini-3x.png 3x"}]{.pattern-image}
[Chaîne de responsabilité]{.pattern-name} [Chain of
Responsibility]{.pattern-aka} ]{.pattern-card-top} [
]{.pattern-card-bottom}](chain-of-responsibility.html){.pattern-card-extended
.chain-of-responsibility}

Permet de faire circuler une demande dans une chaîne de handlers.
Lorsqu\'un handler reçoit une demande, il décide de la traiter ou de
l'envoyer au handler suivant de la chaîne.

[[
[![Commande](../../images/patterns/cards/command-mini8482.png?id=b149eda017c0583c1e92343b83cfb1eb){srcset="/images/patterns/cards/command-mini-2x.png?id=e5f6332e057f6d352a209da963a8fc54 2x,/images/patterns/cards/command-mini-3x.png 3x"}]{.pattern-image}
[Commande]{.pattern-name} [Command]{.pattern-aka} ]{.pattern-card-top} [
]{.pattern-card-bottom}](command.html){.pattern-card-extended .command}

Prend une action à effectuer et la transforme en un objet autonome qui
contient tous les détails de cette action. Cette transformation permet
de paramétrer des méthodes avec différentes actions, planifier leur
exécution, les mettre dans une file d'attente ou d'annuler des
opérations effectuées.

[[
[![Itérateur](../../images/patterns/cards/iterator-minie2c2.png?id=76c28bb48f997b36965983dd2b41f02e){srcset="/images/patterns/cards/iterator-mini-2x.png?id=7d28f66a487066774a743cfceaa40ea4 2x,/images/patterns/cards/iterator-mini-3x.png 3x"}]{.pattern-image}
[Itérateur]{.pattern-name} [Iterator]{.pattern-aka} ]{.pattern-card-top}
[ ]{.pattern-card-bottom}](iterator.html){.pattern-card-extended
.iterator}

Permet de parcourir les éléments d'une collection sans révéler sa
représentation interne (liste, pile, arbre, etc.).

[[
[![Médiateur](../../images/patterns/cards/mediator-mini1561.png?id=a7e43ee8e17e4474737b1fcb3201d7ba){srcset="/images/patterns/cards/mediator-mini-2x.png?id=d288d7c73f5581ae09701d61200ae09e 2x,/images/patterns/cards/mediator-mini-3x.png 3x"}]{.pattern-image}
[Médiateur]{.pattern-name} [Mediator]{.pattern-aka} ]{.pattern-card-top}
[ ]{.pattern-card-bottom}](mediator.html){.pattern-card-extended
.mediator}

Permet de diminuer les dépendances chaotiques entre les objets. Ce
patron restreint les communications directes entre les objets et les
force à collaborer uniquement via un objet médiateur.

[[
[![Mémento](../../images/patterns/cards/memento-mini723d.png?id=8b2ea4dc2c5d15775a654808cc9de099){srcset="/images/patterns/cards/memento-mini-2x.png?id=1d7cba189261dd84b11369a6838b1055 2x,/images/patterns/cards/memento-mini-3x.png 3x"}]{.pattern-image}
[Mémento]{.pattern-name} [Memento]{.pattern-aka} ]{.pattern-card-top} [
]{.pattern-card-bottom}](memento.html){.pattern-card-extended .memento}

Permet de sauvegarder et de rétablir l\'état précédent d'un objet sans
révéler les détails de son implémentation.

[[
[![Observateur](../../images/patterns/cards/observer-minifa2e.png?id=fd2081ab1cff29c60b499bcf6a62786a){srcset="/images/patterns/cards/observer-mini-2x.png?id=f205b0655572ac8e4636691c0e0debfd 2x,/images/patterns/cards/observer-mini-3x.png 3x"}]{.pattern-image}
[Observateur]{.pattern-name} [Observer]{.pattern-aka}
]{.pattern-card-top} [
]{.pattern-card-bottom}](observer.html){.pattern-card-extended
.observer}

Permet de mettre en place un mécanisme de souscription pour envoyer des
notifications à plusieurs objets, au sujet d'événements concernant les
objets qu'ils observent.

[[
[![État](../../images/patterns/cards/state-mini63fb.png?id=f4018837e0641d1dade756b6678fd4ee){srcset="/images/patterns/cards/state-mini-2x.png?id=7e24398b27a43c7bd286fc0ea54d2a35 2x,/images/patterns/cards/state-mini-3x.png 3x"}]{.pattern-image}
[État]{.pattern-name} [State]{.pattern-aka} ]{.pattern-card-top} [
]{.pattern-card-bottom}](state.html){.pattern-card-extended .state}

Modifie le comportement d'un objet lorsque son état interne change.
L'objet donne l'impression qu'il change de classe.

[[
[![Stratégie](../../images/patterns/cards/strategy-mini28f6.png?id=d38abee4fb6f2aed909d262bdadca936){srcset="/images/patterns/cards/strategy-mini-2x.png?id=f4e6608561f8e5d18be6927d4620ad29 2x,/images/patterns/cards/strategy-mini-3x.png 3x"}]{.pattern-image}
[Stratégie]{.pattern-name} [Strategy]{.pattern-aka} ]{.pattern-card-top}
[ ]{.pattern-card-bottom}](strategy.html){.pattern-card-extended
.strategy}

Permet de définir une famille d'algorithmes, de les mettre dans des
classes séparées et de rendre leurs objets interchangeables.

[[ [![Patron de
méthode](../../images/patterns/cards/template-method-mini56e3.png?id=9f200248d88026d8e79d0f3dae411ab4){srcset="/images/patterns/cards/template-method-mini-2x.png?id=178bf56e39b3a1f548dd636076209c98 2x,/images/patterns/cards/template-method-mini-3x.png 3x"}]{.pattern-image}
[Patron de méthode]{.pattern-name} [Template Method]{.pattern-aka}
]{.pattern-card-top} [
]{.pattern-card-bottom}](template-method.html){.pattern-card-extended
.template-method}

Permet de mettre le squelette d'un algorithme dans la classe mère, mais
laisse les sous-classes redéfinir certaines étapes de l'algorithme sans
changer sa structure.

[[
[![Visiteur](../../images/patterns/cards/visitor-mini0a1c.png?id=854a35a62963bec1d75eab996918989b){srcset="/images/patterns/cards/visitor-mini-2x.png?id=9b87f3f3b772f434b28a25876829b504 2x,/images/patterns/cards/visitor-mini-3x.png 3x"}]{.pattern-image}
[Visiteur]{.pattern-name} [Visitor]{.pattern-aka} ]{.pattern-card-top} [
]{.pattern-card-bottom}](visitor.html){.pattern-card-extended .visitor}

Permet de séparer les algorithmes et les objets sur lesquels ils
opèrent.
:::

::: next
#### Suivant

[Chaîne de responsabilité []{.fa
.fa-arrow-right}](chain-of-responsibility.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Retour

[[]{.fa .fa-arrow-left} Procuration](proxy.html){.btn .btn-default
rel="prev"}
:::
:::::::
::::::::
:::::::::
