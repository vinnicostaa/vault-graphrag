:::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Wzorce
projektowe](../../../design-patterns.html){.type} /
[Pyłek](../../flyweight.html){.type} / [Go](../../go.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Pyłek](../../../../images/patterns/cards/flyweight-minie3d8.png?id=422ca8d2f90614dce810a8812c626698){srcset="/images/patterns/cards/flyweight-mini-2x.png?id=857ca676fff84317bb0dab76abfce08e 2x"}
:::

# **Pyłek** w języku Go {#pyłek-w-języku-go .pattern-example-title-block-title}
::::

::::::::::::::::: pattern-example-body
::: pattern-example-brief
**Pyłek** to strukturalny wzorzec projektowy umożliwiający obsługę
wielkich ilości obiektów przy jednoczesnej oszczędności pamięci.

Wzorzec Pyłek umożliwia zmniejszenie wymogów w zakresie pamięci RAM
poprzez współdzielenie części opisu stanu przez wiele obiektów. Innymi
słowy Pyłek przechowuje w pamięci podręcznej te dane, które są wspólne
dla wielu różnych obiektów.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Dowiedz się więcej o Pyłek ](../../flyweight.html){.btn .btn-lg
.btn-primary}
:::

:::::::::::::: {.sidebar-navigation .with-scroll}
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
 [dress­Factory](#example-0--dressFactory-go)
:::

::: en-file
 [dress](#example-0--dress-go)
:::

::: en-file
 [terrorist­Dress](#example-0--terroristDress-go)
:::

::: en-file
 [counter­Terrorist­Dress](#example-0--counterTerroristDress-go)
:::

::: en-file
 [player](#example-0--player-go)
:::

::: en-file
 [game](#example-0--game-go)
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

## Przykład koncepcyjny {#example-0-title}

W grze Counter-Strike Terroryści i Kontrterroryści różnią się ubiorem.
Upraszczając, załóżmy że każda z grup ma po jednym stroju. Informacja o
ubiorze jest zawarta w obiekcie gracza, jak pokazano poniżej.

Oto struktura opisująca gracza. Widzimy, że obiekt odpowiadający
strojowi jest zagnieżdżony wewnątrz tej struktury:

<figure class="code">
<pre class="code" lang="go"><code>type player struct {
    dress      dress
    playerType string // Może przyjąć wartość T lub CT
    lat        int
    long       int
}</code></pre>
</figure>

A teraz załóżmy, że mamy 5 terrorystów i 5 kontrterrorystów, czyli w
sumie 10 graczy. W kwestii ubioru są dwie opcje:

1.  Każdy z 10 obiektów graczy tworzy osobny obiekt ubioru i przechowuje
    go. Stworzone zostanie więc 10 obiektów reprezentujących stroje.

2.  Tworzymy tylko dwa obiekty-stroje:

    -   Jeden obiekt-strój terrorysty: Współdzielony przez 5
        terrorystów.
    -   Jeden obiekt-strój kontrterrorysty: Współdzielony przez 5
        kontrterrorystów.

Widzimy więc, że w pierwszym podejściu stworzonych zostanie 10 obiektów
reprezentujących ubiór, zaś w drugim jedynie 2. To właśnie drugie
podejście jest ideą wzorca projektowego Pyłek. Oba obiekty-stroje, które
zostały stworzone, nazwiemy obiektami-pyłkami.

Wzorzec Pyłek zakłada tworzenie obiektów przechowujących wspólne
elementy wielu innych obiektów. Tak stworzone pyłki (tu: ubiór) mogą być
współdzielone przez wiele obiektów (graczy). Pozwala to drastycznie
zmniejszyć liczbę obiektów opisujących stroje, a pojawienie się większej
liczby graczy nie skutkuje powieleniem obiektu-stroju.

Zgodnie ze wzorcem Pyłek przechowujemy obiekty-pyłki w polu-mapie. Gdy
utworzony zostanie nowy obiekt korzystający z któregoś ze strojów, mapa
zwróci pyłek.

Sprawdźmy które elementy tego układu będą stanami wewnętrznymi, a które
zewnętrznymi:

-   *Stan wewnętrzny*: Strój jest częścią stanu wewnętrznego, gdyż może
    być współdzielony przez wiele obiektów odpowiadających Terrorystom i
    Kontrterrorystom.

-   *Stan zewnętrzny*: Lokalizacja gracza i jego broń są częścią stanu
    zewnętrznego, gdyż są różne dla każdego gracza.

#### []{#example-0--dressFactory-go .anchor} **dressFactory.go:** Fabryka pyłków

<figure class="code">
<pre class="code" lang="go"><code>package main

import &quot;fmt&quot;

const (
    //TerroristDressType terrorist dress type
    TerroristDressType = &quot;tDress&quot;
    //CounterTerrroristDressType terrorist dress type
    CounterTerrroristDressType = &quot;ctDress&quot;
)

var (
    dressFactorySingleInstance = &amp;DressFactory{
        dressMap: make(map[string]Dress),
    }
)

type DressFactory struct {
    dressMap map[string]Dress
}

func (d *DressFactory) getDressByType(dressType string) (Dress, error) {
    if d.dressMap[dressType] != nil {
        return d.dressMap[dressType], nil
    }

    if dressType == TerroristDressType {
        d.dressMap[dressType] = newTerroristDress()
        return d.dressMap[dressType], nil
    }
    if dressType == CounterTerrroristDressType {
        d.dressMap[dressType] = newCounterTerroristDress()
        return d.dressMap[dressType], nil
    }

    return nil, fmt.Errorf(&quot;Wrong dress type passed&quot;)
}

func getDressFactorySingleInstance() *DressFactory {
    return dressFactorySingleInstance
}</code></pre>
</figure>

#### []{#example-0--dress-go .anchor} **dress.go:** Interfejs pyłku

<figure class="code">
<pre class="code" lang="go"><code>package main

type Dress interface {
    getColor() string
}</code></pre>
</figure>

#### []{#example-0--terroristDress-go .anchor} **terroristDress.go:** Konkretny obiekt-pyłek

<figure class="code">
<pre class="code" lang="go"><code>package main

type TerroristDress struct {
    color string
}

func (t *TerroristDress) getColor() string {
    return t.color
}

func newTerroristDress() *TerroristDress {
    return &amp;TerroristDress{color: &quot;red&quot;}
}</code></pre>
</figure>

#### []{#example-0--counterTerroristDress-go .anchor} **counterTerroristDress.go:** Konkretny obiekt-pyłek

<figure class="code">
<pre class="code" lang="go"><code>package main

type CounterTerroristDress struct {
    color string
}

func (c *CounterTerroristDress) getColor() string {
    return c.color
}

func newCounterTerroristDress() *CounterTerroristDress {
    return &amp;CounterTerroristDress{color: &quot;green&quot;}
}</code></pre>
</figure>

#### []{#example-0--player-go .anchor} **player.go:** Kontekst

<figure class="code">
<pre class="code" lang="go"><code>package main

type Player struct {
    dress      Dress
    playerType string
    lat        int
    long       int
}

func newPlayer(playerType, dressType string) *Player {
    dress, _ := getDressFactorySingleInstance().getDressByType(dressType)
    return &amp;Player{
        playerType: playerType,
        dress:      dress,
    }
}

func (p *Player) newLocation(lat, long int) {
    p.lat = lat
    p.long = long
}</code></pre>
</figure>

#### []{#example-0--game-go .anchor} **game.go:** Kod klienta

<figure class="code">
<pre class="code" lang="go"><code>package main

type game struct {
    terrorists        []*Player
    counterTerrorists []*Player
}

func newGame() *game {
    return &amp;game{
        terrorists:        make([]*Player, 1),
        counterTerrorists: make([]*Player, 1),
    }
}

func (c *game) addTerrorist(dressType string) {
    player := newPlayer(&quot;T&quot;, dressType)
    c.terrorists = append(c.terrorists, player)
    return
}

func (c *game) addCounterTerrorist(dressType string) {
    player := newPlayer(&quot;CT&quot;, dressType)
    c.counterTerrorists = append(c.counterTerrorists, player)
    return
}</code></pre>
</figure>

#### []{#example-0--main-go .anchor} **main.go:** Kod klienta

<figure class="code">
<pre class="code" lang="go"><code>package main

import &quot;fmt&quot;

func main() {
    game := newGame()

    //Add Terrorist
    game.addTerrorist(TerroristDressType)
    game.addTerrorist(TerroristDressType)
    game.addTerrorist(TerroristDressType)
    game.addTerrorist(TerroristDressType)

    //Add CounterTerrorist
    game.addCounterTerrorist(CounterTerrroristDressType)
    game.addCounterTerrorist(CounterTerrroristDressType)
    game.addCounterTerrorist(CounterTerrroristDressType)

    dressFactoryInstance := getDressFactorySingleInstance()

    for dressType, dress := range dressFactoryInstance.dressMap {
        fmt.Printf(&quot;DressColorType: %s\nDressColor: %s\n&quot;, dressType, dress.getColor())
    }
}</code></pre>
</figure>

#### []{#example-0--output-txt .anchor} **output.txt:** Wynik działania

<figure class="code">
<pre class="code" lang="output"><code>DressColorType: ctDress
DressColor: green
DressColorType: tDress
DressColor: red</code></pre>
</figure>

::: next
#### Czytaj dalej

[Pełnomocnik w języku Go []{.fa
.fa-arrow-right}](../../proxy/go/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Wróć

[[]{.fa .fa-arrow-left} Fasada w języku
Go](../../facade/go/example.html){.btn .btn-default rel="prev"}
:::

## **Pyłek** w innych językach

[![Pyłek w języku
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Pyłek w języku C#"){.prog-lang-link}
[![Pyłek w języku
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Pyłek w języku C++"){.prog-lang-link}
[![Pyłek w języku
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Pyłek w języku Java"){.prog-lang-link}
[![Pyłek w języku
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Pyłek w języku PHP"){.prog-lang-link}
[![Pyłek w języku
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Pyłek w języku Python"){.prog-lang-link}
[![Pyłek w języku
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Pyłek w języku Ruby"){.prog-lang-link}
[![Pyłek w języku
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Pyłek w języku Rust"){.prog-lang-link}
[![Pyłek w języku
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Pyłek w języku Swift"){.prog-lang-link}
[![Pyłek w języku
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Pyłek w języku TypeScript"){.prog-lang-link}
:::::::::::::::::::::::

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
:::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::
