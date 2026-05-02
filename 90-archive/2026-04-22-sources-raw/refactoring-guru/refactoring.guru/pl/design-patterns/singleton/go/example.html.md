:::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Wzorce
projektowe](../../../design-patterns.html){.type} /
[Singleton](../../singleton.html){.type} / [Go](../../go.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Singleton](../../../../images/patterns/cards/singleton-minif559.png?id=914e1565dfdf15f240e766163bd303ec){srcset="/images/patterns/cards/singleton-mini-2x.png?id=4a7348083d7719d02709a5c9ff878b9f 2x"}
:::

# **Singleton** w języku Go {#singleton-w-języku-go .pattern-example-title-block-title}
::::

:::::::::::::::: pattern-example-body
::: pattern-example-brief
**Singleton** to kreacyjny wzorzec projektowy gwarantujący istnienie
tylko jednego obiektu danego rodzaju. Udostępnia też pojedynczy punkt
dostępowy do takiego obiektu z dowolnego miejsca w programie.

Singleton charakteryzuje się prawie takimi samymi zaletami i wadami jak
zmienne globalne i chociaż jest bardzo poręczny, to jednak psuje
modularność kodu.

Nie można przenieść klasy zależnej od Singletona i użyć jej w innym
kontekście bez równoczesnego przeniesienia tego drugiego. To
ograniczenie zazwyczaj ujawnia się na etapie tworzenia testów
jednostkowych.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Dowiedz się więcej o Singleton ](../../singleton.html){.btn .btn-lg
.btn-primary}
:::

::::::::::::: {.sidebar-navigation .with-scroll}
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
 [single](#example-0--single-go)
:::

::: en-file
 [main](#example-0--main-go)
:::

::: en-file
 [output](#example-0--output-txt)
:::

::: en-intro
 [Inny przykład](#example-1)
:::

::: en-file
 [sync­Once](#example-1--syncOnce-go)
:::

::: en-file
 [main](#example-1--main-go)
:::

::: en-file
 [output](#example-1--output-txt)
:::
:::::::::::::
::::::::::::::::

::: {#nav-tab .nav .nav-tabs .nav-justified role="tablist"}
[Przykład koncepcyjny](#example-0){#example-0-tab .nav-item .nav-link
.active toggle="tab" role="tab" aria-controls="example-0"
aria-selected="true"} [Inny przykład](#example-1){#example-1-tab
.nav-item .nav-link toggle="tab" role="tab" aria-controls="example-1"
aria-selected="true"}
:::

::::: {#nav-tabContent .pattern-example-tab-content .tab-content}
::: {#example-0 .tab-pane .show .active role="tabpanel" aria-labelledby="example-0-tab" style="padding-top: 24px"}
## Przykład koncepcyjny {#example-0-title}

Na ogół instancja singletona tworzona jest przy okazji pierwszej
inicjalizacji struktury. Aby tak się stało, definiujemy metodę
`getInstance` struktury. Metoda ta będzie odpowiedzialna za tworzenie i
zwracanie instancji singletona. Za każdym razem gdy wywoła się metodę
`getInstance`, zwraca będzie ta sama instancja singletona.

A co z goroutines? Struktura singleton musi zwracać tę samą instancję za
każdym razem gdy wiele goroutines chce uzyskać dostęp do tej
instancji. Na skutek tego łatwo można zaimplementować wzorzec singleton
niewłaściwie. Poniższy przykład przedstawia prawidłowy sposób tworzenia
singletona.

Na co warto zwrócić uwagę:

-   Na starcie upewniamy się, że `singleInstance` jest wstępnie pusty.
    Dzięki temu zapobiegamy kosztownym operacjom zakładania blokady przy
    każdym wywołaniu metody `getInstance`. Jeśli wynik sprawdzenia jest
    różny niż `null`, oznacza to, że pole `singleInstance` jest już
    zapełnione.

-   W obrębie blokady tworzona jest struktura `singleInstance`.

-   Kolejne sprawdzenie `null` wykonuje się po założeniu blokady. Dzięki
    temu mamy pewność, że jeżeli więcej niż jedna goroutine obejdzie
    pierwsze sprawdzenie to tylko jedna z nich będzie mogła utworzyć
    instancję singletona. W przeciwnym wypadku wszystkie goroutines
    utworzą swoje własne instancje struktury singleton.

#### []{#example-0--single-go .anchor} **single.go:** Singleton

<figure class="code">
<pre class="code" lang="go"><code>package main

import (
    &quot;fmt&quot;
    &quot;sync&quot;
)

var lock = &amp;sync.Mutex{}

type single struct {
}

var singleInstance *single

func getInstance() *single {
    if singleInstance == nil {
        lock.Lock()
        defer lock.Unlock()
        if singleInstance == nil {
            fmt.Println(&quot;Creating single instance now.&quot;)
            singleInstance = &amp;single{}
        } else {
            fmt.Println(&quot;Single instance already created.&quot;)
        }
    } else {
        fmt.Println(&quot;Single instance already created.&quot;)
    }

    return singleInstance
}</code></pre>
</figure>

#### []{#example-0--main-go .anchor} **main.go:** Kod klienta

<figure class="code">
<pre class="code" lang="go"><code>package main

import (
    &quot;fmt&quot;
)

func main() {

    for i := 0; i &lt; 30; i++ {
        go getInstance()
    }

    // Scanln is similar to Scan, but stops scanning at a newline and
    // after the final item there must be a newline or EOF.
    fmt.Scanln()
}</code></pre>
</figure>

#### []{#example-0--output-txt .anchor} **output.txt:** Wynik działania

<figure class="code">
<pre class="code" lang="output"><code>Creating single instance now.
Single instance already created.
Single instance already created.
Single instance already created.
Single instance already created.
Single instance already created.
Single instance already created.
Single instance already created.
Single instance already created.
Single instance already created.
Single instance already created.
Single instance already created.
Single instance already created.
Single instance already created.
Single instance already created.
Single instance already created.
Single instance already created.
Single instance already created.
Single instance already created.
Single instance already created.
Single instance already created.
Single instance already created.
Single instance already created.
Single instance already created.
Single instance already created.
Single instance already created.
Single instance already created.
Single instance already created.
Single instance already created.
Single instance already created.</code></pre>
</figure>
:::

::: {#example-1 .tab-pane role="tabpanel" aria-labelledby="example-1-tab" style="padding-top: 24px"}
## Inny przykład {#example-1-title}

Są też inne sposoby na tworzenie instancji singletona w Go:

1.  Funkcja `init`

Możemy stworzyć pojedynczą instancję w obrębie funkcji `init`. Warunkiem
jest pomyślna wstępna inicjalizacja instancji. Funkcja `init` jest
wywoływana raz dla każdego pliku w paczce, możemy mieć więc pewność, że
powstanie tylko jedna instancja.

2.  `sync.Once`

`Sync.Once` wykona operację tylko jednorazowo. Spójrzmy na poniższy kod:

#### []{#example-1--syncOnce-go .anchor} **syncOnce.go:** Singleton

<figure class="code">
<pre class="code" lang="go"><code>package main

import (
    &quot;fmt&quot;
    &quot;sync&quot;
)

var once sync.Once

type single struct {
}

var singleInstance *single

func getInstance() *single {
    if singleInstance == nil {
        once.Do(
            func() {
                fmt.Println(&quot;Creating single instance now.&quot;)
                singleInstance = &amp;single{}
            })
    } else {
        fmt.Println(&quot;Single instance already created.&quot;)
    }

    return singleInstance
}</code></pre>
</figure>

#### []{#example-1--main-go .anchor} **main.go:** Kod klienta

<figure class="code">
<pre class="code" lang="go"><code>package main

import (
    &quot;fmt&quot;
)

func main() {

    for i := 0; i &lt; 30; i++ {
        go getInstance()
    }

    // Scanln is similar to Scan, but stops scanning at a newline and
    // after the final item there must be a newline or EOF.
    fmt.Scanln()
}</code></pre>
</figure>

#### []{#example-1--output-txt .anchor} **output.txt:** Wynik działania

<figure class="code">
<pre class="code" lang="output"><code>Creating single instance now.
Single instance already created.
Single instance already created.</code></pre>
</figure>
:::
:::::

::: {#nav-tab .nav .nav-tabs .nav-tabs-bottom .nav-justified role="tablist"}
[Przykład koncepcyjny](#example-0){#example-0-tab-bottom .nav-item
.nav-link .active toggle="tab" role="tab" aria-controls="example-0"
aria-selected="true"} [Inny przykład](#example-1){#example-1-tab-bottom
.nav-item .nav-link toggle="tab" role="tab" aria-controls="example-1"
aria-selected="true"}
:::

::: next
#### Czytaj dalej

[Adapter w języku Go []{.fa
.fa-arrow-right}](../../adapter/go/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Wróć

[[]{.fa .fa-arrow-left} Prototyp w języku
Go](../../prototype/go/example.html){.btn .btn-default rel="prev"}
:::

## **Singleton** w innych językach

[![Singleton w języku
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Singleton w języku C#"){.prog-lang-link}
[![Singleton w języku
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Singleton w języku C++"){.prog-lang-link}
[![Singleton w języku
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Singleton w języku Java"){.prog-lang-link}
[![Singleton w języku
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Singleton w języku PHP"){.prog-lang-link}
[![Singleton w języku
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Singleton w języku Python"){.prog-lang-link}
[![Singleton w języku
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Singleton w języku Ruby"){.prog-lang-link}
[![Singleton w języku
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Singleton w języku Rust"){.prog-lang-link}
[![Singleton w języku
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Singleton w języku Swift"){.prog-lang-link}
[![Singleton w języku
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Singleton w języku TypeScript"){.prog-lang-link}
:::::::::::::::::::::::::::

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
:::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::
