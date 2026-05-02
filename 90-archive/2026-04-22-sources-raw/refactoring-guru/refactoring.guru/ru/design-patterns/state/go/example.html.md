:::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Паттерны
проектирования](../../../design-patterns.html){.type} /
[Состояние](../../state.html){.type} / [Go](../../go.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Состояние](../../../../images/patterns/cards/state-mini63fb.png?id=f4018837e0641d1dade756b6678fd4ee){srcset="/images/patterns/cards/state-mini-2x.png?id=7e24398b27a43c7bd286fc0ea54d2a35 2x"}
:::

# **Состояние** на Go {#состояние-на-go .pattern-example-title-block-title}
::::

::::::::::::::::: pattern-example-body
::: pattern-example-brief
**Состояние** --- это поведенческий паттерн, позволяющий динамически
изменять поведение объекта при смене его состояния.

Поведения, зависящие от состояния, переезжают в отдельные классы.
Первоначальный класс хранит ссылку на один из таких объектов-состояний и
делегирует ему работу.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Подробней о паттерне Состояние ](../../state.html){.btn .btn-lg
.btn-primary}
:::

:::::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
Навигация
:::

::: en-intro
 [Интро](#)
:::

::: en-intro
 [Концептуальный пример](#example-0)
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

## Концептуальный пример {#example-0-title}

Давайте применим паттерн проектирования Состояние в контексте торговых
автоматов. Для упрощения задачи представим, что торговый автомат может
выдавать только один товар. Также представим, что автомат может
пребывать только в одном из четырех состояний:

-   hasItem (имеетПредмет)
-   noItem (неИмеетПредмет)
-   itemRequested (выдаётПредмет)
-   hasMoney (получилДеньги)

Торговый автомат может иметь различные действия. Опять-таки, для
простоты оставим только четыре из них:

-   Выбрать предмет
-   Добавить предмет
-   Ввести деньги
-   Выдать предмет

Паттерн Состояние нужно использовать в случаях, когда объект может иметь
много различных состояний, которые он должен менять в зависимости от
конкретного поступившего запроса.

В нашем примере, автомат может быть в одном из множества состояний,
которые непрерывно меняются. Припустим, что торговый автомат находится в
режиме `itemRequested`. Как только произойдет действие «Ввести деньги»,
он сразу же перейдет в состояние `hasMoney`.

В зависимости от состояния торгового автомата, в котором он находится на
данный момент, он может по-разному отвечать на одни и те же запросы.
Например, если пользователь хочет купить предмет, машина выполнит
действие, если она находится в режиме `hasItemState`, и отклонит запрос
в режиме `noItemState`.

Программа торгового автомата не захламлена этой логикой; весь
режимозависимый код обитает в реализациях соответствующих состояний.

#### []{#example-0--vendingMachine-go .anchor} **vendingMachine.go:** Контекст

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

#### []{#example-0--state-go .anchor} **state.go:** Интерфейс состояния

<figure class="code">
<pre class="code" lang="go"><code>package main

type State interface {
    addItem(int) error
    requestItem() error
    insertMoney(money int) error
    dispenseItem() error
}</code></pre>
</figure>

#### []{#example-0--noItemState-go .anchor} **noItemState.go:** Конкретный интерфейс

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

#### []{#example-0--hasItemState-go .anchor} **hasItemState.go:** Конкретный интерфейс

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

#### []{#example-0--itemRequestedState-go .anchor} **itemRequestedState.go:** Конкретный интерфейс

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

#### []{#example-0--hasMoneyState-go .anchor} **hasMoneyState.go:** Конкретный интерфейс

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

#### []{#example-0--main-go .anchor} **main.go:** Клиентский код

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

#### []{#example-0--output-txt .anchor} **output.txt:** Результат выполнения

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
#### Читаем дальше

[Стратегия на Go []{.fa
.fa-arrow-right}](../../strategy/go/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Наблюдатель на
Go](../../observer/go/example.html){.btn .btn-default rel="prev"}
:::

## **Состояние** на других языках программирования

[![Состояние на
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Состояние на C#"){.prog-lang-link}
[![Состояние на
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Состояние на C++"){.prog-lang-link}
[![Состояние на
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Состояние на Java"){.prog-lang-link}
[![Состояние на
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Состояние на PHP"){.prog-lang-link}
[![Состояние на
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Состояние на Python"){.prog-lang-link}
[![Состояние на
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Состояние на Ruby"){.prog-lang-link}
[![Состояние на
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Состояние на Rust"){.prog-lang-link}
[![Состояние на
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Состояние на Swift"){.prog-lang-link}
[![Состояние на
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Состояние на TypeScript"){.prog-lang-link}
:::::::::::::::::::::::

::::::: {.prom .banner-sidebar .banner .banner-striped .banner-examples .banner-removable .banner-removable-patterns data-id="DP: Examples-IDE" creative-id="examples-sidebar-ru" position="sidebar"}
:::::: {.banner-inner style="font-size: 14px; line-height: 21px;"}
::: banner-examples-img
[![](../../../../images/patterns/banners/examples-ide91ca.png?id=3115b4b548fb96b75974e2de8f4f49bc){width="215"
height="158" loading="lazy"
srcset="/images/patterns/banners/examples-ide-2x.png?id=93c007a6d157b97c28eb213f59d716ae 2x"}](../../book.html)
:::

::: banner-examples-fade
[ Архив с примерами](../../book.html){.btn .btn-secondary .btn-block}
:::

::: {.banner-examples-text style="margin-top: 1rem;"}
Купи книгу **Погружение в Паттерны** и получи архив с десятками
детальных примеров, которые можно открывать прямо в IDE.

[ Узнать больше...](../../book.html){.btn .btn-secondary .btn-block}
:::
::::::
:::::::
:::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::
