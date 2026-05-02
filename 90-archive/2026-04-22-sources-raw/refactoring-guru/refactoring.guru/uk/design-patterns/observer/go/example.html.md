:::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Патерни
проектування](../../../design-patterns.html){.type} /
[Спостерігач](../../observer.html){.type} / [Go](../../go.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Спостерігач](../../../../images/patterns/cards/observer-minifa2e.png?id=fd2081ab1cff29c60b499bcf6a62786a){srcset="/images/patterns/cards/observer-mini-2x.png?id=f205b0655572ac8e4636691c0e0debfd 2x"}
:::

# **Спостерігач** на Go {#спостерігач-на-go .pattern-example-title-block-title}
::::

::::::::::::::: pattern-example-body
::: pattern-example-brief
**Спостерігач** --- це поведінковий патерн, який дозволяє об'єктам
повідомляти інші об'єкти про зміни свого стану.

При цьому спостерігачі можуть вільно підписуватися і відписуватись від
цих повідомлень.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Детальніше про Спостерігач ](../../observer.html){.btn .btn-lg
.btn-primary}
:::

:::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
Навігація
:::

::: en-intro
 [Інтро](#)
:::

::: en-intro
 [Концептуальний приклад](#example-0)
:::

::: en-file
 [subject](#example-0--subject-go)
:::

::: en-file
 [item](#example-0--item-go)
:::

::: en-file
 [observer](#example-0--observer-go)
:::

::: en-file
 [customer](#example-0--customer-go)
:::

::: en-file
 [main](#example-0--main-go)
:::

::: en-file
 [output](#example-0--output-txt)
:::
::::::::::::
:::::::::::::::

[]{#example-0}

## Концептуальний приклад {#example-0-title}

На сайті інтернет-магазину періодично може закінчуватися певний товар.
Водночас деякі користувачі можуть бути зацікавлені у цьому предметі,
якого поки що немає у наявності. У цієї проблеми може бути 3 варіанти
вирішення:

1.  Покупець самостійно періодично перевіряє наявність товару.
2.  Інтернет-магазин завалює користувачів сповіщеннями про надходження
    всіх нових товарів.
3.  Користувач підписується лише на той конкретний предмет, який його
    цікавить, і одержує сповіщення про його повернення на полиці
    магазину. Також, на один і той же продукт можуть підписатися
    декілька покупців.

Варіант 3 звучить найбільш ефективно, і фактично це і є суть патерна
Спостерігач. Головні елементи цього патерна проектування наступні:

-   Видавець --- публікує подію, коли щось відбувається.
-   Спостерігач --- підписується на події суб'єкта і одержує сповіщення
    в разі їх виникнення.

#### []{#example-0--subject-go .anchor} **subject.go:** Видавець

<figure class="code">
<pre class="code" lang="go"><code>package main

type Subject interface {
    register(observer Observer)
    deregister(observer Observer)
    notifyAll()
}</code></pre>
</figure>

#### []{#example-0--item-go .anchor} **item.go:** Конкретний видавець

<figure class="code">
<pre class="code" lang="go"><code>package main

import &quot;fmt&quot;

type Item struct {
    observerList []Observer
    name         string
    inStock      bool
}

func newItem(name string) *Item {
    return &amp;Item{
        name: name,
    }
}
func (i *Item) updateAvailability() {
    fmt.Printf(&quot;Item %s is now in stock\n&quot;, i.name)
    i.inStock = true
    i.notifyAll()
}
func (i *Item) register(o Observer) {
    i.observerList = append(i.observerList, o)
}

func (i *Item) deregister(o Observer) {
    i.observerList = removeFromslice(i.observerList, o)
}

func (i *Item) notifyAll() {
    for _, observer := range i.observerList {
        observer.update(i.name)
    }
}

func removeFromslice(observerList []Observer, observerToRemove Observer) []Observer {
    observerListLength := len(observerList)
    for i, observer := range observerList {
        if observerToRemove.getID() == observer.getID() {
            observerList[observerListLength-1], observerList[i] = observerList[i], observerList[observerListLength-1]
            return observerList[:observerListLength-1]
        }
    }
    return observerList
}</code></pre>
</figure>

#### []{#example-0--observer-go .anchor} **observer.go:** Спостерігач

<figure class="code">
<pre class="code" lang="go"><code>package main

type Observer interface {
    update(string)
    getID() string
}</code></pre>
</figure>

#### []{#example-0--customer-go .anchor} **customer.go:** Конкретний спостерігач

<figure class="code">
<pre class="code" lang="go"><code>package main

import &quot;fmt&quot;

type Customer struct {
    id string
}

func (c *Customer) update(itemName string) {
    fmt.Printf(&quot;Sending email to customer %s for item %s\n&quot;, c.id, itemName)
}

func (c *Customer) getID() string {
    return c.id
}</code></pre>
</figure>

#### []{#example-0--main-go .anchor} **main.go:** Клієнтський код

<figure class="code">
<pre class="code" lang="go"><code>package main

func main() {

    shirtItem := newItem(&quot;Nike Shirt&quot;)

    observerFirst := &amp;Customer{id: &quot;abc@gmail.com&quot;}
    observerSecond := &amp;Customer{id: &quot;xyz@gmail.com&quot;}

    shirtItem.register(observerFirst)
    shirtItem.register(observerSecond)

    shirtItem.updateAvailability()
}</code></pre>
</figure>

#### []{#example-0--output-txt .anchor} **output.txt:** Результат виконання

<figure class="code">
<pre class="code" lang="output"><code>Item Nike Shirt is now in stock
Sending email to customer abc@gmail.com for item Nike Shirt
Sending email to customer xyz@gmail.com for item Nike Shirt</code></pre>
</figure>

::: next
#### Читаємо далі

[Стан на Go []{.fa .fa-arrow-right}](../../state/go/example.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Повернутись назад

[[]{.fa .fa-arrow-left} Знімок на
Go](../../memento/go/example.html){.btn .btn-default rel="prev"}
:::

## **Спостерігач** іншими мовами програмування

[![Спостерігач на
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Спостерігач на C#"){.prog-lang-link}
[![Спостерігач на
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Спостерігач на C++"){.prog-lang-link}
[![Спостерігач на
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Спостерігач на Java"){.prog-lang-link}
[![Спостерігач на
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Спостерігач на PHP"){.prog-lang-link}
[![Спостерігач на
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Спостерігач на Python"){.prog-lang-link}
[![Спостерігач на
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Спостерігач на Ruby"){.prog-lang-link}
[![Спостерігач на
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Спостерігач на Rust"){.prog-lang-link}
[![Спостерігач на
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Спостерігач на Swift"){.prog-lang-link}
[![Спостерігач на
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Спостерігач на TypeScript"){.prog-lang-link}
:::::::::::::::::::::

::::::: {.prom .banner-sidebar .banner .banner-striped .banner-examples .banner-removable .banner-removable-patterns data-id="DP: Examples-IDE" creative-id="examples-sidebar-uk" position="sidebar"}
:::::: {.banner-inner style="font-size: 14px; line-height: 21px;"}
::: banner-examples-img
[![](../../../../images/patterns/banners/examples-ide91ca.png?id=3115b4b548fb96b75974e2de8f4f49bc){width="215"
height="158" loading="lazy"
srcset="/images/patterns/banners/examples-ide-2x.png?id=93c007a6d157b97c28eb213f59d716ae 2x"}](../../book.html)
:::

::: banner-examples-fade
[ Архів з прикладами](../../book.html){.btn .btn-secondary .btn-block}
:::

::: {.banner-examples-text style="margin-top: 1rem;"}
Придбай книгу **Занурення в Патерни** та отримай архів з десятками
детальних прикладів, які можна відкривати прямо в IDE.

[ Дізнатися більше...](../../book.html){.btn .btn-secondary .btn-block}
:::
::::::
:::::::
:::::::::::::::::::::::::::
::::::::::::::::::::::::::::
