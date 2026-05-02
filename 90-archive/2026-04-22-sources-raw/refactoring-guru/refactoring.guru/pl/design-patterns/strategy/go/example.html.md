::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Wzorce
projektowe](../../../design-patterns.html){.type} /
[Strategia](../../strategy.html){.type} / [Go](../../go.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Strategia](../../../../images/patterns/cards/strategy-mini28f6.png?id=d38abee4fb6f2aed909d262bdadca936){srcset="/images/patterns/cards/strategy-mini-2x.png?id=f4e6608561f8e5d18be6927d4620ad29 2x"}
:::

# **Strategia** w języku Go {#strategia-w-języku-go .pattern-example-title-block-title}
::::

:::::::::::::::: pattern-example-body
::: pattern-example-brief
**Strategia** to behawioralny wzorzec projektowy zakładający
przekształcenie zestawu zachowań w obiekty, które można stosować
zamiennie w pierwotnym obiekcie.

Pierwotny obiekt, zwany kontekstem, przechowuje odniesienie do
obiektu-strategii i deleguje mu działania związane z danym zachowaniem.
Aby zmienić sposób, w jaki kontekst wykonuje swą pracę, należy zamienić
bieżąco przypisany obiekt strategii na inny.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Dowiedz się więcej o Strategia ](../../strategy.html){.btn .btn-lg
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
 [eviction­Algo](#example-0--evictionAlgo-go)
:::

::: en-file
 [fifo](#example-0--fifo-go)
:::

::: en-file
 [lru](#example-0--lru-go)
:::

::: en-file
 [lfu](#example-0--lfu-go)
:::

::: en-file
 [cache](#example-0--cache-go)
:::

::: en-file
 [main](#example-0--main-go)
:::

::: en-file
 [output](#example-0--output-txt)
:::
:::::::::::::
::::::::::::::::

[]{#example-0}

## Przykład koncepcyjny {#example-0-title}

Przypuśćmy, że tworzymy schowek w obrębie pamięci. Skoro znajduje się w
pamięci, to ma pewien ograniczony rozmiar. Gdy ilość danych w schowku
wyczerpie dostępną pamięć, niektóre wpisy muszą zostać z niego usunięte
aby zwolnić miejsce. Może się to odbywać według kilku algorytmów, z
których najpopularniejsze to:

-   Najrzadziej ostatnio używane (LRU): usuwa wpis który był ostatnio
    najrzadziej potrzebny.
-   Pierwszy na wejściu, pierwszy na wyjściu (FIFO): usuwa
    najwcześniejszy wpis.
-   Najrzadziej używane (LFU): usuwa najrzadziej używany wpis.

Problemem jest teraz rozsprzęgnięcie naszej klasy pamięci podręcznej i
powyższych algorytmów tak, aby można było wybrać odpowiedni w trakcie
działania programu. Ponadto, klasa pamięci podręcznej nie powinna ulegać
zmianom gdy dodawany jest nowy algorytm.

Tu właśnie z pomocą przychodzi nam wzorzec Strategia. Proponuje on
utworzenie rodziny algorytmów z których każdy z nich będzie osobną
klasą. Każda z tych klas będzie zgodna pod względem interfejsu, co
pozwoli na wymianę jednego algorytmu na inny. Powiedzmy, że nazwą
wspólnego interfejsu jest `evictionAlgo`.

Od tego momentu nasza główna klasa pamięci podręcznej będzie zawierała
interfejs `evictionAlgo`. Zamiast implementować wszystkie algorytmy
zwalniania przestrzeni w klasie, będzie ona delegować te zadania
interfejsowi `evictionAlgo`. Skoro `evictionAlgo` jest interfejsem,
możemy wybrać stosowany algorytm (LRU, FIFO, LFU) w trakcie działania
programu bez konieczności zmiany w klasie pamięci podręcznej.

#### []{#example-0--evictionAlgo-go .anchor} **evictionAlgo.go:** Interfejs strategii

<figure class="code">
<pre class="code" lang="go"><code>package main

type EvictionAlgo interface {
    evict(c *Cache)
}</code></pre>
</figure>

#### []{#example-0--fifo-go .anchor} **fifo.go:** Konkretna strategia

<figure class="code">
<pre class="code" lang="go"><code>package main

import &quot;fmt&quot;

type Fifo struct {
}

func (l *Fifo) evict(c *Cache) {
    fmt.Println(&quot;Evicting by fifo strtegy&quot;)
}</code></pre>
</figure>

#### []{#example-0--lru-go .anchor} **lru.go:** Konkretna strategia

<figure class="code">
<pre class="code" lang="go"><code>package main

import &quot;fmt&quot;

type Lru struct {
}

func (l *Lru) evict(c *Cache) {
    fmt.Println(&quot;Evicting by lru strtegy&quot;)
}</code></pre>
</figure>

#### []{#example-0--lfu-go .anchor} **lfu.go:** Konkretna strategia

<figure class="code">
<pre class="code" lang="go"><code>package main

import &quot;fmt&quot;

type Lfu struct {
}

func (l *Lfu) evict(c *Cache) {
    fmt.Println(&quot;Evicting by lfu strtegy&quot;)
}</code></pre>
</figure>

#### []{#example-0--cache-go .anchor} **cache.go:** Kontekst

<figure class="code">
<pre class="code" lang="go"><code>package main

type Cache struct {
    storage      map[string]string
    evictionAlgo EvictionAlgo
    capacity     int
    maxCapacity  int
}

func initCache(e EvictionAlgo) *Cache {
    storage := make(map[string]string)
    return &amp;Cache{
        storage:      storage,
        evictionAlgo: e,
        capacity:     0,
        maxCapacity:  2,
    }
}

func (c *Cache) setEvictionAlgo(e EvictionAlgo) {
    c.evictionAlgo = e
}

func (c *Cache) add(key, value string) {
    if c.capacity == c.maxCapacity {
        c.evict()
    }
    c.capacity++
    c.storage[key] = value
}

func (c *Cache) get(key string) {
    delete(c.storage, key)
}

func (c *Cache) evict() {
    c.evictionAlgo.evict(c)
    c.capacity--
}</code></pre>
</figure>

#### []{#example-0--main-go .anchor} **main.go:** Kod klienta

<figure class="code">
<pre class="code" lang="go"><code>package main

func main() {
    lfu := &amp;Lfu{}
    cache := initCache(lfu)

    cache.add(&quot;a&quot;, &quot;1&quot;)
    cache.add(&quot;b&quot;, &quot;2&quot;)

    cache.add(&quot;c&quot;, &quot;3&quot;)

    lru := &amp;Lru{}
    cache.setEvictionAlgo(lru)

    cache.add(&quot;d&quot;, &quot;4&quot;)

    fifo := &amp;Fifo{}
    cache.setEvictionAlgo(fifo)

    cache.add(&quot;e&quot;, &quot;5&quot;)

}</code></pre>
</figure>

#### []{#example-0--output-txt .anchor} **output.txt:** Wynik działania

<figure class="code">
<pre class="code" lang="output"><code>Evicting by lfu strtegy
Evicting by lru strtegy
Evicting by fifo strtegy</code></pre>
</figure>

::: next
#### Czytaj dalej

[Metoda szablonowa w języku Go []{.fa
.fa-arrow-right}](../../template-method/go/example.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Wróć

[[]{.fa .fa-arrow-left} Stan w języku
Go](../../state/go/example.html){.btn .btn-default rel="prev"}
:::

## **Strategia** w innych językach

[![Strategia w języku
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Strategia w języku C#"){.prog-lang-link}
[![Strategia w języku
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Strategia w języku C++"){.prog-lang-link}
[![Strategia w języku
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Strategia w języku Java"){.prog-lang-link}
[![Strategia w języku
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Strategia w języku PHP"){.prog-lang-link}
[![Strategia w języku
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Strategia w języku Python"){.prog-lang-link}
[![Strategia w języku
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Strategia w języku Ruby"){.prog-lang-link}
[![Strategia w języku
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Strategia w języku Rust"){.prog-lang-link}
[![Strategia w języku
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Strategia w języku Swift"){.prog-lang-link}
[![Strategia w języku
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Strategia w języku TypeScript"){.prog-lang-link}
::::::::::::::::::::::

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
::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::
