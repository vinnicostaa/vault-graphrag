:::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Wzorce
projektowe](../../../design-patterns.html){.type} /
[Obserwator](../../observer.html){.type} / [Go](../../go.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Obserwator](../../../../images/patterns/cards/observer-minifa2e.png?id=fd2081ab1cff29c60b499bcf6a62786a){srcset="/images/patterns/cards/observer-mini-2x.png?id=f205b0655572ac8e4636691c0e0debfd 2x"}
:::

# **Obserwator** w języku Go {#obserwator-w-języku-go .pattern-example-title-block-title}
::::

::::::::::::::: pattern-example-body
::: pattern-example-brief
**Obserwator** to behawioralny wzorzec projektowy pozwalający obiektom
powiadamiać inne obiekty o zmianach swojego stanu.

Obserwator daje możliwość subskrypcji lub zrezygnowania z subskrypcji
zdarzeń dowolnego obiektu implementującego interfejs subskrybenta.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Dowiedz się więcej o Obserwator ](../../observer.html){.btn .btn-lg
.btn-primary}
:::

:::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
Nawigacja
:::

::: en-intro
 [Intro](#)
:::

::: en-intro
 [Przykład koncepcyjny](#example-0)
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

## Przykład koncepcyjny {#example-0-title}

Na stronie internetowej e-commerce co jakiś czas wyczerpuje się zapas
niektórych artykułów. Może się zdarzyć, że klient chce kupić coś, czego
akurat sklep nie ma na stanie. Są trzy rozwiązania takiego problemu:

1.  Klient co jakiś czas sprawdza dostępność artykułu na stronie.
2.  Aplikacja rozsyła wszystkim klientom informacje o wszystkich nowych
    pozycjach w katalogu które są dostępne.
3.  Klient subskrybuje informacje o jednym konkretnym produkcie który go
    interesuje. Wielu klientów może być w ten sposób powiadamianych
    odnośnie dostępności jakiegoś produktu.

Najsensowniejsza jest trzecia opcja i to właśnie jest podejście
proponowane przez wzorzec Obserwator. Głównymi komponentami
wchodzącymi w skład tego wzorca są:

-   Podmiot, czyli instancja publikująca informacje o zaistniałych
    zdarzeniach.
-   Obserwator, który subskrybuje zdarzenia dotyczące danego podmiotu i
    jest o nich powiadamiany.

#### []{#example-0--subject-go .anchor} **subject.go:** Podmiot

<figure class="code">
<pre class="code" lang="go"><code>package main

type Subject interface {
    register(observer Observer)
    deregister(observer Observer)
    notifyAll()
}</code></pre>
</figure>

#### []{#example-0--item-go .anchor} **item.go:** Konkretny podmiot

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

#### []{#example-0--observer-go .anchor} **observer.go:** Obserwator

<figure class="code">
<pre class="code" lang="go"><code>package main

type Observer interface {
    update(string)
    getID() string
}</code></pre>
</figure>

#### []{#example-0--customer-go .anchor} **customer.go:** Konkretny obserwator

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

#### []{#example-0--main-go .anchor} **main.go:** Kod klienta

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

#### []{#example-0--output-txt .anchor} **output.txt:** Wynik działania

<figure class="code">
<pre class="code" lang="output"><code>Item Nike Shirt is now in stock
Sending email to customer abc@gmail.com for item Nike Shirt
Sending email to customer xyz@gmail.com for item Nike Shirt</code></pre>
</figure>

::: next
#### Czytaj dalej

[Stan w języku Go []{.fa
.fa-arrow-right}](../../state/go/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Wróć

[[]{.fa .fa-arrow-left} Pamiątka w języku
Go](../../memento/go/example.html){.btn .btn-default rel="prev"}
:::

## **Obserwator** w innych językach

[![Obserwator w języku
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Obserwator w języku C#"){.prog-lang-link}
[![Obserwator w języku
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Obserwator w języku C++"){.prog-lang-link}
[![Obserwator w języku
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Obserwator w języku Java"){.prog-lang-link}
[![Obserwator w języku
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Obserwator w języku PHP"){.prog-lang-link}
[![Obserwator w języku
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Obserwator w języku Python"){.prog-lang-link}
[![Obserwator w języku
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Obserwator w języku Ruby"){.prog-lang-link}
[![Obserwator w języku
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Obserwator w języku Rust"){.prog-lang-link}
[![Obserwator w języku
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Obserwator w języku Swift"){.prog-lang-link}
[![Obserwator w języku
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Obserwator w języku TypeScript"){.prog-lang-link}
:::::::::::::::::::::

::::::: {.prom .banner-sidebar .banner .banner-striped .banner-examples .banner-removable .banner-removable-patterns data-id="DP: Examples-IDE" creative-id="examples-sidebar-pl" position="sidebar"}
:::::: {.banner-inner style="font-size: 14px; line-height: 21px;"}
::: banner-examples-img
[![](../../../../images/patterns/banners/examples-ide91ca.png?id=3115b4b548fb96b75974e2de8f4f49bc){width="215"
height="158" loading="lazy"
srcset="/images/patterns/banners/examples-ide-2x.png?id=93c007a6d157b97c28eb213f59d716ae 2x"}](../../book.html)
:::

::: banner-examples-fade
[ Archiwum\
przykładów kodu](../../book.html){.btn .btn-secondary .btn-block}
:::

::: {.banner-examples-text style="margin-top: 1rem;"}
Kupując eBook **Wzorce projektowe. Nowoczesny podręcznik** zyskujesz
dostęp do archiwum zawierającego mnóstwo szczegółowych przykładów, które
można od razu wkleić do swojego preferowanego zintegrowanego środowiska
programistycznego.

[ Dowiedz się więcej...](../../book.html){.btn .btn-secondary
.btn-block}
:::
::::::
:::::::
:::::::::::::::::::::::::::
::::::::::::::::::::::::::::
