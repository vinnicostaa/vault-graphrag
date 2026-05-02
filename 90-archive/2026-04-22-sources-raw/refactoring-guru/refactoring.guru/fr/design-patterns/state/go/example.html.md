:::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Patrons de
conception](../../../design-patterns.html){.type} /
[État](../../state.html){.type} / [Go](../../go.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![État](../../../../images/patterns/cards/state-mini63fb.png?id=f4018837e0641d1dade756b6678fd4ee){srcset="/images/patterns/cards/state-mini-2x.png?id=7e24398b27a43c7bd286fc0ea54d2a35 2x"}
:::

# **État** en Go {#état-en-go .pattern-example-title-block-title}
::::

::::::::::::::::: pattern-example-body
::: pattern-example-brief
L'**État** est un patron de conception comportemental qui permet à un
objet de modifier son comportement lorsque son état interne change.

Ce patron extrait les comportements liés aux états dans des classes
séparées et force l'objet original à déléguer les tâches à une instance
de ces classes, au lieu de le faire lui-même.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ En savoir plus sur la patron État ](../../state.html){.btn .btn-lg
.btn-primary}
:::

:::::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
Navigation
:::

::: en-intro
 [Intro](#)
:::

::: en-intro
 [Exemple conceptuel](#example-0)
:::

::: en-file
 [vending­Machine](#example-0--vendingMachine-go)
:::

::: en-file
 [state](#example-0--state-go)
:::

::: en-file
 [no­Item­State](#example-0--noItemState-go)
:::

::: en-file
 [has­Item­State](#example-0--hasItemState-go)
:::

::: en-file
 [item­Requested­State](#example-0--itemRequestedState-go)
:::

::: en-file
 [has­Money­State](#example-0--hasMoneyState-go)
:::

::: en-file
 [main](#example-0--main-go)
:::

::: en-file
 [output](#example-0--output-txt)
:::
::::::::::::::
:::::::::::::::::

[]{#example-0}

## Exemple conceptuel {#example-0-title}

Utilisons le patron de conception état pour des distributeurs
automatiques. Pour plus de simplicité, le distributeur ne vendra qu'un
seul type de produit et la machine va se limiter à quatre états
différents :

-   articlePrésent (hasItem)
-   articleAbsent (noItem)
-   articleDemandé (itemRequested)
-   possèdeArgent (hasMoney)

Le distributeur disposera de plusieurs actions et encore une fois pour
plus de simplicité, il se limitera à quatre :

-   Choisir l'article (Select the item)
-   Ajouter l'article (Add the item)
-   Insérer de l'argent (Insert money)
-   Donner l'article (Dispense item)

Le patron de conception état devrait être utilisé pour des objets qui
peuvent prendre différents états et dont l'état peut être modifié par
les demandes qui leur parviennent.

Dans notre exemple, le distributeur peut prendre beaucoup d'états
différents et il va constamment passer de l'un à l'autre. Imaginons que
notre distributeur est dans l'état `articleDemandé`. Si l'action
« Insérer de l'argent » se produit, la machine entre dans l'état
`possèdeArgent`.

La machine va se comporter différemment pour les mêmes demandes, selon
son état actuel. Par exemple, si un utilisateur veut acheter un article,
la machine va accepter si elle est dans l'état `articlePrésent`. Elle va
rejeter la demande si elle se trouve dans l'état `articleAbsent`.

Le code du distributeur n'est pas pollué par cette logique : tout ce qui
concerne l'état est écrit dans les implémentations respectives de
l'état.

#### []{#example-0--vendingMachine-go .anchor} **vendingMachine.go:** Contexte

<figure class="code">
<pre class="code" lang="go"><code>package main

import &quot;fmt&quot;

type VendingMachine struct {
    hasItem       State
    itemRequested State
    hasMoney      State
    noItem        State

    currentState State

    itemCount int
    itemPrice int
}

func newVendingMachine(itemCount, itemPrice int) *VendingMachine {
    v := &amp;VendingMachine{
        itemCount: itemCount,
        itemPrice: itemPrice,
    }
    hasItemState := &amp;HasItemState{
        vendingMachine: v,
    }
    itemRequestedState := &amp;ItemRequestedState{
        vendingMachine: v,
    }
    hasMoneyState := &amp;HasMoneyState{
        vendingMachine: v,
    }
    noItemState := &amp;NoItemState{
        vendingMachine: v,
    }

    v.setState(hasItemState)
    v.hasItem = hasItemState
    v.itemRequested = itemRequestedState
    v.hasMoney = hasMoneyState
    v.noItem = noItemState
    return v
}

func (v *VendingMachine) requestItem() error {
    return v.currentState.requestItem()
}

func (v *VendingMachine) addItem(count int) error {
    return v.currentState.addItem(count)
}

func (v *VendingMachine) insertMoney(money int) error {
    return v.currentState.insertMoney(money)
}

func (v *VendingMachine) dispenseItem() error {
    return v.currentState.dispenseItem()
}

func (v *VendingMachine) setState(s State) {
    v.currentState = s
}

func (v *VendingMachine) incrementItemCount(count int) {
    fmt.Printf(&quot;Adding %d items\n&quot;, count)
    v.itemCount = v.itemCount + count
}</code></pre>
</figure>

#### []{#example-0--state-go .anchor} **state.go:** Interface état

<figure class="code">
<pre class="code" lang="go"><code>package main

type State interface {
    addItem(int) error
    requestItem() error
    insertMoney(money int) error
    dispenseItem() error
}</code></pre>
</figure>

#### []{#example-0--noItemState-go .anchor} **noItemState.go:** État concret

<figure class="code">
<pre class="code" lang="go"><code>package main

import &quot;fmt&quot;

type NoItemState struct {
    vendingMachine *VendingMachine
}

func (i *NoItemState) requestItem() error {
    return fmt.Errorf(&quot;Item out of stock&quot;)
}

func (i *NoItemState) addItem(count int) error {
    i.vendingMachine.incrementItemCount(count)
    i.vendingMachine.setState(i.vendingMachine.hasItem)
    return nil
}

func (i *NoItemState) insertMoney(money int) error {
    return fmt.Errorf(&quot;Item out of stock&quot;)
}
func (i *NoItemState) dispenseItem() error {
    return fmt.Errorf(&quot;Item out of stock&quot;)
}</code></pre>
</figure>

#### []{#example-0--hasItemState-go .anchor} **hasItemState.go:** État concret

<figure class="code">
<pre class="code" lang="go"><code>package main

import &quot;fmt&quot;

type HasItemState struct {
    vendingMachine *VendingMachine
}

func (i *HasItemState) requestItem() error {
    if i.vendingMachine.itemCount == 0 {
        i.vendingMachine.setState(i.vendingMachine.noItem)
        return fmt.Errorf(&quot;No item present&quot;)
    }
    fmt.Printf(&quot;Item requestd\n&quot;)
    i.vendingMachine.setState(i.vendingMachine.itemRequested)
    return nil
}

func (i *HasItemState) addItem(count int) error {
    fmt.Printf(&quot;%d items added\n&quot;, count)
    i.vendingMachine.incrementItemCount(count)
    return nil
}

func (i *HasItemState) insertMoney(money int) error {
    return fmt.Errorf(&quot;Please select item first&quot;)
}
func (i *HasItemState) dispenseItem() error {
    return fmt.Errorf(&quot;Please select item first&quot;)
}</code></pre>
</figure>

#### []{#example-0--itemRequestedState-go .anchor} **itemRequestedState.go:** État concret

<figure class="code">
<pre class="code" lang="go"><code>package main

import &quot;fmt&quot;

type ItemRequestedState struct {
    vendingMachine *VendingMachine
}

func (i *ItemRequestedState) requestItem() error {
    return fmt.Errorf(&quot;Item already requested&quot;)
}

func (i *ItemRequestedState) addItem(count int) error {
    return fmt.Errorf(&quot;Item Dispense in progress&quot;)
}

func (i *ItemRequestedState) insertMoney(money int) error {
    if money &lt; i.vendingMachine.itemPrice {
        return fmt.Errorf(&quot;Inserted money is less. Please insert %d&quot;, i.vendingMachine.itemPrice)
    }
    fmt.Println(&quot;Money entered is ok&quot;)
    i.vendingMachine.setState(i.vendingMachine.hasMoney)
    return nil
}
func (i *ItemRequestedState) dispenseItem() error {
    return fmt.Errorf(&quot;Please insert money first&quot;)
}</code></pre>
</figure>

#### []{#example-0--hasMoneyState-go .anchor} **hasMoneyState.go:** État concret

<figure class="code">
<pre class="code" lang="go"><code>package main

import &quot;fmt&quot;

type HasMoneyState struct {
    vendingMachine *VendingMachine
}

func (i *HasMoneyState) requestItem() error {
    return fmt.Errorf(&quot;Item dispense in progress&quot;)
}

func (i *HasMoneyState) addItem(count int) error {
    return fmt.Errorf(&quot;Item dispense in progress&quot;)
}

func (i *HasMoneyState) insertMoney(money int) error {
    return fmt.Errorf(&quot;Item out of stock&quot;)
}
func (i *HasMoneyState) dispenseItem() error {
    fmt.Println(&quot;Dispensing Item&quot;)
    i.vendingMachine.itemCount = i.vendingMachine.itemCount - 1
    if i.vendingMachine.itemCount == 0 {
        i.vendingMachine.setState(i.vendingMachine.noItem)
    } else {
        i.vendingMachine.setState(i.vendingMachine.hasItem)
    }
    return nil
}</code></pre>
</figure>

#### []{#example-0--main-go .anchor} **main.go:** Code client

<figure class="code">
<pre class="code" lang="go"><code>package main

import (
    &quot;fmt&quot;
    &quot;log&quot;
)

func main() {
    vendingMachine := newVendingMachine(1, 10)

    err := vendingMachine.requestItem()
    if err != nil {
        log.Fatalf(err.Error())
    }

    err = vendingMachine.insertMoney(10)
    if err != nil {
        log.Fatalf(err.Error())
    }

    err = vendingMachine.dispenseItem()
    if err != nil {
        log.Fatalf(err.Error())
    }

    fmt.Println()

    err = vendingMachine.addItem(2)
    if err != nil {
        log.Fatalf(err.Error())
    }

    fmt.Println()

    err = vendingMachine.requestItem()
    if err != nil {
        log.Fatalf(err.Error())
    }

    err = vendingMachine.insertMoney(10)
    if err != nil {
        log.Fatalf(err.Error())
    }

    err = vendingMachine.dispenseItem()
    if err != nil {
        log.Fatalf(err.Error())
    }
}</code></pre>
</figure>

#### []{#example-0--output-txt .anchor} **output.txt:** Résultat de l'exécution

<figure class="code">
<pre class="code" lang="output"><code>Item requestd
Money entered is ok
Dispensing Item

Adding 2 items

Item requestd
Money entered is ok
Dispensing Item</code></pre>
</figure>

::: next
#### Suivant

[Stratégie en Go []{.fa
.fa-arrow-right}](../../strategy/go/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Retour

[[]{.fa .fa-arrow-left} Observateur en
Go](../../observer/go/example.html){.btn .btn-default rel="prev"}
:::

## **État** dans les autres langues

[![État en
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "État en C#"){.prog-lang-link}
[![État en
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "État en C++"){.prog-lang-link}
[![État en
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "État en Java"){.prog-lang-link}
[![État en
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "État en PHP"){.prog-lang-link}
[![État en
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "État en Python"){.prog-lang-link}
[![État en
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "État en Ruby"){.prog-lang-link}
[![État en
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "État en Rust"){.prog-lang-link}
[![État en
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "État en Swift"){.prog-lang-link}
[![État en
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "État en TypeScript"){.prog-lang-link}
:::::::::::::::::::::::

::::::: {.prom .banner-sidebar .banner .banner-striped .banner-examples .banner-removable .banner-removable-patterns data-id="DP: Examples-IDE" creative-id="examples-sidebar-fr" position="sidebar"}
:::::: {.banner-inner style="font-size: 14px; line-height: 21px;"}
::: banner-examples-img
[![](../../../../images/patterns/banners/examples-ide91ca.png?id=3115b4b548fb96b75974e2de8f4f49bc){width="215"
height="158" loading="lazy"
srcset="/images/patterns/banners/examples-ide-2x.png?id=93c007a6d157b97c28eb213f59d716ae 2x"}](../../book.html)
:::

::: banner-examples-fade
[ Archive avec exemples](../../book.html){.btn .btn-secondary
.btn-block}
:::

::: {.banner-examples-text style="margin-top: 1rem;"}
Achetez l'ebook **Plongée au cœur des patrons de conception** et obtenez
l'accès à une archive comprenant des dizaines d'exemples détaillés qui
peuvent être ouverts dans votre environnement de développement.

[ En savoir plus...](../../book.html){.btn .btn-secondary .btn-block}
:::
::::::
:::::::
:::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::
