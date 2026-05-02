:::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Padrões de
Projeto](../../../design-patterns.html){.type} /
[State](../../state.html){.type} / [Go](../../go.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![State](../../../../images/patterns/cards/state-mini63fb.png?id=f4018837e0641d1dade756b6678fd4ee){srcset="/images/patterns/cards/state-mini-2x.png?id=7e24398b27a43c7bd286fc0ea54d2a35 2x"}
:::

# **State** em Go {#state-em-go .pattern-example-title-block-title}
::::

::::::::::::::::: pattern-example-body
::: pattern-example-brief
O **State** é um padrão de projeto comportamental que permite que um
objeto altere o comportamento quando seu estado interno for alterado.

O padrão extrai comportamentos relacionados ao estado em classes
separadas de estado e força o objeto original a delegar o trabalho para
uma instância dessas classes, em vez de agir por conta própria.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Saiba mais sobre o State ](../../state.html){.btn .btn-lg
.btn-primary}
:::

:::::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
Navegação
:::

::: en-intro
 [Introdução](#)
:::

::: en-intro
 [Exemplo conceitual](#example-0)
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

## Exemplo conceitual {#example-0-title}

Vamos aplicar o padrão de design State no contexto das máquinas de venda
automática. Para simplificar, vamos supor que a máquina de venda
automática tenha apenas um tipo de item ou produto. Além disso, para
simplificar, vamos supor que uma máquina de venda automática pode estar
em 4 estados diferentes:

-   hasItem
-   noItem
-   itemRequested
-   hasMoney

Uma máquina de venda automática também terá ações diferentes. Mais uma
vez para simplificar, vamos supor que existam apenas quatro ações:

-   Selecionar o item
-   Adicionar o item
-   Inserir dinheiro
-   Dispensar item

O padrão de design State deve ser usado quando o objeto pode estar em
muitos estados diferentes e, dependendo da solicitação de entrada, o
objeto precisa alterar seu estado atual.

Em nosso exemplo, uma máquina de venda automática pode estar em muitos
estados diferentes, e esses estados mudarão continuamente de um para
outro. Digamos que a máquina de venda automática esteja em
`itemRequested`. Uma vez que a ação "Inserir dinheiro" ocorre, a máquina
se move para o estado `hasMoney`.

Dependendo de seu estado atual, a máquina pode se comportar de maneira
diferente para as mesmas solicitações. Por exemplo, se um usuário deseja
comprar um item, a máquina irá prosseguir se estiver em `hasItemState`
ou rejeitará em `noItemState`.

O código da máquina de venda automática não se polui com essa lógica;
todo o código dependente de estado vive nas respectivas implementações
de estado.

#### []{#example-0--vendingMachine-go .anchor} **vendingMachine.go:** Contexto

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

#### []{#example-0--state-go .anchor} **state.go:** Interface do state

<figure class="code">
<pre class="code" lang="go"><code>package main

type State interface {
    addItem(int) error
    requestItem() error
    insertMoney(money int) error
    dispenseItem() error
}</code></pre>
</figure>

#### []{#example-0--noItemState-go .anchor} **noItemState.go:** State concreto

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

#### []{#example-0--hasItemState-go .anchor} **hasItemState.go:** State concreto

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

#### []{#example-0--itemRequestedState-go .anchor} **itemRequestedState.go:** State concreto

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

#### []{#example-0--hasMoneyState-go .anchor} **hasMoneyState.go:** State concreto

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

#### []{#example-0--main-go .anchor} **main.go:** Código cliente

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

#### []{#example-0--output-txt .anchor} **output.txt:** Resultados da execução

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
#### Leia a seguir

[Strategy em Go []{.fa
.fa-arrow-right}](../../strategy/go/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Voltar

[[]{.fa .fa-arrow-left} Observer em
Go](../../observer/go/example.html){.btn .btn-default rel="prev"}
:::

## **State** em outras linguagens

[![State em
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "State em C#"){.prog-lang-link}
[![State em
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "State em C++"){.prog-lang-link}
[![State em
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "State em Java"){.prog-lang-link}
[![State em
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "State em PHP"){.prog-lang-link}
[![State em
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "State em Python"){.prog-lang-link}
[![State em
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "State em Ruby"){.prog-lang-link}
[![State em
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "State em Rust"){.prog-lang-link}
[![State em
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "State em Swift"){.prog-lang-link}
[![State em
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "State em TypeScript"){.prog-lang-link}
:::::::::::::::::::::::

::::::: {.prom .banner-sidebar .banner .banner-striped .banner-examples .banner-removable .banner-removable-patterns data-id="DP: Examples-IDE" creative-id="examples-sidebar-pt-br" position="sidebar"}
:::::: {.banner-inner style="font-size: 14px; line-height: 21px;"}
::: banner-examples-img
[![](../../../../images/patterns/banners/examples-ide91ca.png?id=3115b4b548fb96b75974e2de8f4f49bc){width="215"
height="158" loading="lazy"
srcset="/images/patterns/banners/examples-ide-2x.png?id=93c007a6d157b97c28eb213f59d716ae 2x"}](../../book.html)
:::

::: banner-examples-fade
[ Arquivo com exemplos de código](../../book.html){.btn .btn-secondary
.btn-block}
:::

::: {.banner-examples-text style="margin-top: 1rem;"}
Compre o eBook **Mergulho nos Padrões de Projeto** e tenha acesso ao
arquivo com dúzias de exemplos detalhados que podem ser abertos
diretamente em seu IDE.

[ Saiba mais...](../../book.html){.btn .btn-secondary .btn-block}
:::
::::::
:::::::
:::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::
