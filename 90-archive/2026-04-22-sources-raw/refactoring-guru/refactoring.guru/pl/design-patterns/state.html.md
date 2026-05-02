::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [Wzorce
projektowe](../design-patterns.html){.type} / [Wzorce
behawioralne](behavioral-patterns.html){.type}
:::

# Stan {#stan .title}

::: aka
Znany też jako: [State]{style="display:inline-block"}
:::

::: {.section .intent}
##  Cel {#intent}

**Stan** to behawioralny wzorzec projektowy pozwalający obiektowi
zmienić swoje zachowanie gdy zmieni się jego stan wewnętrzny. Wygląda to
tak, jakby obiekt zmienił swoją klasę.

<figure class="image">
<img
src="../../images/patterns/content/state/state-pl62a7.png?id=6836381177829794b3cca14ec0729030"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/state/state-pl.png?id=6836381177829794b3cca14ec0729030 640w,/images/patterns/content/state/state-pl-1.5x.png?id=f9646a7719f974923bbaa32ce932c30a 960w,/images/patterns/content/state/state-pl-2x.png?id=2c81df0d430b34cca7568fc729c388a2 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Wzorzec projektowy Stan" />
</figure>
:::

::: {.section .problem}
##  Problem

Wzorzec Stan jest powiązany z koncepcją *Automatu
skończonego* []{.pop-i}[Automat skończony:
[https://refactoring.guru/pl/fsm](https://pl.wikipedia.org/wiki/Automat_sko%C5%84czony)]{.pop-content}.

<figure class="image">
<img
src="../../images/patterns/diagrams/state/problem10244.png?id=503968745461a0970d1fbb4362dc186f"
style="aspect-ratio: 1.45;"
srcset="/images/patterns/diagrams/state/problem1.png?id=503968745461a0970d1fbb4362dc186f 320w,/images/patterns/diagrams/state/problem1-1.5x.png?id=47c7ca068eceaa9896d320690e6f3368 480w,/images/patterns/diagrams/state/problem1-2x.png?id=ae03c2233939eace11d44925ddeb912d 640w"
sizes="(max-width: 720px) 100vw, 320px" loading="lazy" width="320"
alt="Automat skończony" />
<figcaption><p>Automat skończony.</p></figcaption>
</figure>

Sednem tej koncepcji jest to, że w dowolnym momencie istnieje
*skończona* liczba *stanów* w których program może się znajdować. W
każdym z nich program zachowuje się różnie i może zostać przełączony z
jednego stanu w drugi natychmiastowo. Możliwe stany, w jakich obiekt
może się znaleźć, zależą od bieżącego stanu. Liczba reguł przełączeń,
zwanych *przejściami* również jest skończona i są one z góry określone.

Można też zastosować to podejście wobec obiektów. Wyobraźmy sobie klasę
`Dokument`. Dokument może znajdować się w jednym z trzech stanów:
`Szkic`, `Korekta` i `Opublikowany`. Metoda `publikuj` dokumentu działa
nieco inaczej zależnie od jego stanu:

-   W przypadku `Szkicu` przenosi dokument do moderacji.
-   W stanie `Korekta` czyni dokument dostępnym publicznie, ale tylko
    jeśli bieżący użytkownik jest administratorem.
-   W stanie `Opublikowany` nie robi nic.

<figure class="image">
<img
src="../../images/patterns/diagrams/state/problem2-pl25e3.png?id=136ed3c45d30b606372b21cf964a85d9"
style="aspect-ratio: 1.27;"
srcset="/images/patterns/diagrams/state/problem2-pl.png?id=136ed3c45d30b606372b21cf964a85d9 560w,/images/patterns/diagrams/state/problem2-pl-1.5x.png?id=e6f120b7d0fa8ae4d3ee19fb4328d1ac 840w,/images/patterns/diagrams/state/problem2-pl-2x.png?id=0b6d6b9326296f254bcd8c52967c06db 1120w"
sizes="(max-width: 720px) 100vw, 560px" loading="lazy" width="560"
alt="Możliwe stany obiektu dokument" />
<figcaption><p>Możliwe stany i przejścia między stanami
obiektu dokument.</p></figcaption>
</figure>

Maszyny stanów są zwykle implementowane za pomocą wielu struktur
warunkowych (`if` lub `switch`) które wybierają odpowiednie zachowanie
zależnie od bieżącego stanu dokumentu. Zazwyczaj "stan" jest tylko
zestawem wartości pól obiektu. Nawet jeśli nie wiesz nic o automatach
skończonych, prawdopodobnie przynajmniej raz implementowałeś stan. Czy
poniższy kawałek kodu coś ci przypomina?

<figure class="code">
<pre class="code" lang="pseudocode"><code>class Document is
    field state: string
    // ...
    method publish() is
        switch (state)
            &quot;draft&quot;:
                state = &quot;moderation&quot;
                break
            &quot;moderation&quot;:
                if (currentUser.role == &quot;admin&quot;)
                    state = &quot;published&quot;
                break
            &quot;published&quot;:
                // Nie rób nic.
                break
    // ...</code></pre>
</figure>

Największa słabość maszyny stanów opartej na instrukcjach warunkowych
ujawnia się w miarę dodawania kolejnych stanów oraz zachowań
zależnych od tych stanów do klasy `Dokument`. Większość metod będzie
zawierała olbrzymie instrukcje warunkowe wybierające stosowne zachowanie
się metody zależnie od bieżącego stanu. Taki kod jest trudny w
utrzymaniu, ponieważ każda zmiana logiki przechodzenia między stanami
może wymagać zmian instrukcji warunkowych w każdej z metod.

Problem narasta wraz z rozbudową projektu. Trudno jest przewidzieć
wszystkie możliwe stany i przejścia między nimi na etapie projektowania.
Dlatego też prosta maszyna stanów, zbudowana z ograniczonej liczby
instrukcji warunkowych, może z czasem spuchnąć do niebotycznych
rozmiarów.
:::

::: {.section .solution}
##  Rozwiązanie {#solution}

Wzorzec Stan proponuje stworzenie nowych klas dla każdego z możliwych
stanów obiektu oraz ekstrakcję wszystkich zachowań zależnych od stanu do
tychże klas.

Zamiast implementować wszystkie zachowania samodzielnie, pierwotny
obiekt, zwany *kontekstem*, przechowuje odniesienie do jednego z
obiektów-stanów który w danej chwili reprezentuje jego bieżący stan i
deleguje mu zadania związane z tym stanem.

<figure class="image">
<img
src="../../images/patterns/diagrams/state/solution-pl2268.png?id=3752e1200d48b50d227fa92bcb8da5d5"
style="aspect-ratio: 1.53;"
srcset="/images/patterns/diagrams/state/solution-pl.png?id=3752e1200d48b50d227fa92bcb8da5d5 490w,/images/patterns/diagrams/state/solution-pl-1.5x.png?id=60662888e773ef440838f0ba0375d0b6 735w,/images/patterns/diagrams/state/solution-pl-2x.png?id=e2f4297a94e7a9ec5daf450e84183593 980w"
sizes="(max-width: 720px) 100vw, 490px" loading="lazy" width="490"
alt="Dokument deleguje pracę obiektowi stanu" />
<figcaption><p>Dokument deleguje pracę obiektowi stanu.</p></figcaption>
</figure>

Aby przełączyć kontekst do innego stanu, zamieniamy aktywny obiekt
stanu na inny obiekt reprezentujący nowy stan. Jest to możliwe wyłącznie
gdy wszystkie klasy stanu są zgodne pod względem interfejsu, zaś
kontekst współpracuje z tymi obiektami tylko poprzez ów interfejs.

Taka struktura może przypominać wzorzec [Strategia](strategy.html),
ale z jedną kluczową różnicą. W przypadku wzorca Stan, poszczególne
stany mogą być świadome siebie nawzajem i inicjować przejścia z jednego
stanu w drugi, zaś strategie prawie nigdy nie wiedzą nic o sobie.
:::

::: {.section .analogy}
##  Analogia do prawdziwego życia {#analogy}

Przyciski i przełączniki w twoim smartfonie zachowują się w różny
sposób, w zależności od bieżącego stanu urządzenia:

-   Gdy telefon jest odblokowany, wciskanie przycisków wywołuje różne
    funkcje.
-   Gdy telefon jest zablokowany, wciskanie przycisków wyświetli ekran
    służący odblokowaniu.
-   Gdy bateria telefonu jest na wyczerpaniu, wciśnięcie dowolnego
    przycisku wyświetli ekran ładowania.
:::

::::: {.section .structure-container}
##  Struktura {#structure}

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/state/structure-plcbd6.png?id=1ed102350ba5e18a2fc3fce254dc5c6d"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.32;"
srcset="/images/patterns/diagrams/state/structure-pl.png?id=1ed102350ba5e18a2fc3fce254dc5c6d 540w,/images/patterns/diagrams/state/structure-pl-1.5x.png?id=18b2a95f56e31b68879e753571fdac6e 810w,/images/patterns/diagrams/state/structure-pl-2x.png?id=d3b853b88478a883c6cc2b4a99a57c22 1080w"
sizes="(max-width: 720px) 100vw, 540px" loading="lazy" width="540"
alt="Struktura wzorca Stan" /><img
src="../../images/patterns/diagrams/state/structure-pl-indexedfbee.png?id=4f5123440d84e8280184b204fa05735e"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.26;"
srcset="/images/patterns/diagrams/state/structure-pl-indexed.png?id=4f5123440d84e8280184b204fa05735e 540w,/images/patterns/diagrams/state/structure-pl-indexed-1.5x.png?id=eb4e3a92a778693f8a9bcd9922accabb 810w,/images/patterns/diagrams/state/structure-pl-indexed-2x.png?id=9791ad5adafdb14b7a2d13a1db704a46 1080w"
sizes="(max-width: 720px) 100vw, 540px" loading="lazy" width="540"
alt="Struktura wzorca Stan" />
</figure>
:::

1.  **Kontekst** przechowuje odniesienie do jednego z konkretnych
    obiektów-stanów i deleguje mu zadania specyficzne dla danego stanu.
    Kontekst porozumiewa się z obiektem stanu za pośrednictwem
    interfejsu stanu. Kontekst eksponuje metodę setter, przez którą
    przekazuje się obiekt nowego stanu dla kontekstu.

2.  Interfejs **Stanu** deklaruje metody specyficzne dla stanu.
    Metody te powinny być sensowne dla konkretnych stanów, ponieważ nie
    chcemy, aby któreś ze stanów posiadały bezużyteczne metody które nie
    zostaną nigdy wywołane.

3.  **Konkretne Stany** dostarczają swoje implementacje metod
    specyficznych dla poszczególnych stanów. Aby uniknąć powtórzeń
    podobnego kodu w wielu stanach, można utworzyć pośrednie klasy
    abstrakcyjne które hermetyzują jakieś wspólne zachowania.

    Obiekty stanu mogą przechowywać referencje wsteczne do obiektu
    kontekst. Za pośrednictwem tego odniesienia stan może pobrać dowolne
    informacje z obiektu kontekst, a także zainicjować zmianę stanu.

4.  Zarówno kontekst, jak i konkretne stany mogą ustawić kolejny stan
    kontekstu i wykonać samą zmianę stanu poprzez zamianę obiektu stanu
    powiązanego z kontekstem na inny.
::::
:::::

::: {.section .pseudocode}
##  Pseudokod {#pseudocode}

W tym przykładzie, wzorzec **Stan** pozwala tym samym kontrolkom
odtwarzacza multimedialnego zachowywać się nieco inaczej, zależnie od
stanu odtwarzania.

<figure class="image">
<img
src="../../images/patterns/diagrams/state/example1dc9.png?id=1ffdb109b3ebb85d223b9d1651d2034c"
style="aspect-ratio: 1.64;"
srcset="/images/patterns/diagrams/state/example.png?id=1ffdb109b3ebb85d223b9d1651d2034c 590w,/images/patterns/diagrams/state/example-1.5x.png?id=a9ff56d0a139530fa103d496513dfa06 885w,/images/patterns/diagrams/state/example-2x.png?id=cd81e3ffb8aba5883983a59c111b805f 1180w"
sizes="(max-width: 720px) 100vw, 590px" loading="lazy" width="590"
alt="Struktura przykładu użycia wzorca Stan" />
<figcaption><p>Przykład zmiany zachowania się obiektu zależnie od
obiektu stanu.</p></figcaption>
</figure>

Główny obiekt odtwarzacza jest zawsze powiązany z obiektem stanu, który
wykonuje większość zadań odtwarzacza. Niektóre czynności zamieniają
bieżący obiekt stanu na inny, co przy okazji zmienia reakcje
odtwarzacza na polecenia od użytkownika.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// Klasa AudioPlayer pełni rolę kontekstu. Posiada także
// odniesienie do instancji jednej z klas-stanów reprezentującej
// bieżący stan odtwarzacza audio.
class AudioPlayer is
    field state: State
    field UI, volume, playlist, currentSong

    constructor AudioPlayer() is
        this.state = new ReadyState(this)

        // Kontekst deleguje obsługę danych wejściowych
        // użytkownika obiektowi stanu. Oczywiście wynik zależy
        // od tego jaki stan jest aktualnie aktywny, ponieważ
        // każdy ze stanów może obsługiwać dane wejściowe nieco
        // inaczej.
        UI = new UserInterface()
        UI.lockButton.onClick(this.clickLock)
        UI.playButton.onClick(this.clickPlay)
        UI.nextButton.onClick(this.clickNext)
        UI.prevButton.onClick(this.clickPrevious)

    // Inne obiekty muszą być w stanie przełączyć aktywny stan
    // odtwarzacza audio.
    method changeState(state: State) is
        this.state = state

    // Metody interfejsu użytkownika delegują wykonanie
    // aktywnemu stanowi.
    method clickLock() is
        state.clickLock()
    method clickPlay() is
        state.clickPlay()
    method clickNext() is
        state.clickNext()
    method clickPrevious() is
        state.clickPrevious()

    // Stan może wywoływać jakieś metody-usługi kontekstu.
    method startPlayback() is
        // ...
    method stopPlayback() is
        // ...
    method nextSong() is
        // ...
    method previousSong() is
        // ...
    method fastForward(time) is
        // ...
    method rewind(time) is
        // ...


// Klasa bazowa stanu deklaruje metody które muszą być
// zaimplementowane przez wszystkie konkretne stany. Posiada też
// referencję zwrotną do obiektu-kontekstu skojarzonego ze
// stanem. Stany mogą za pomocą tej referencji zwrotnej wywołać
// przejście kontekstu z jednego stanu w inny.
abstract class State is
    protected field player: AudioPlayer

    // Kontekst przekazuje siebie do konstruktora stanu. Pomoże
    // to stanowi pozyskiwać różne użyteczne dane kontekstu
    // jeśli zaistnieje potrzeba.
    constructor State(player) is
        this.player = player

    abstract method clickLock()
    abstract method clickPlay()
    abstract method clickNext()
    abstract method clickPrevious()


// Konkretne stany implementują różnorakie zachowania związane z
// danym stanem kontekstu.
class LockedState extends State is

    // Gdy odblokuje się zablokowany odtwarzacz, może on znaleźć
    // się w którymś z dwóch stanów.
    method clickLock() is
        if (player.playing)
            player.changeState(new PlayingState(player))
        else
            player.changeState(new ReadyState(player))

    method clickPlay() is
        // Zablokowany, więc nie rób nic.

    method clickNext() is
        // Zablokowany, więc nie rób nic.

    method clickPrevious() is
        // Zablokowany, więc nie rób nic.


// Konkretne stany mogą też wyzwolić przejście kontekstu z
// jednego stanu w inny.
class ReadyState extends State is
    method clickLock() is
        player.changeState(new LockedState(player))

    method clickPlay() is
        player.startPlayback()
        player.changeState(new PlayingState(player))

    method clickNext() is
        player.nextSong()

    method clickPrevious() is
        player.previousSong()


class PlayingState extends State is
    method clickLock() is
        player.changeState(new LockedState(player))

    method clickPlay() is
        player.stopPlayback()
        player.changeState(new ReadyState(player))

    method clickNext() is
        if (event.doubleclick)
            player.nextSong()
        else
            player.fastForward(5)

    method clickPrevious() is
        if (event.doubleclick)
            player.previous()
        else
            player.rewind(5)</code></pre>
</figure>
:::

:::::::::: {.section .applicability-container}
##  Zastosowanie {#applicability}

::::::::: applicability
::: applicability-problem
Stosuj wzorzec Stan gdy masz do czynienia z obiektem którego zachowanie
jest zależne od jego stanu, liczba możliwych stanów jest wielka, a kod
specyficzny dla danego stanu często ulega zmianom.
:::

::: applicability-solution
Wzorzec proponuje ekstrakcję całego kodu właściwego poszczególnym
stanom do zestawu osobnych klas. W wyniku tego można będzie dodawać nowe
stany lub zmieniać istniejące niezależnie od siebie, zmniejszając koszty
utrzymania.
:::

::: applicability-problem
Stosuj ten wzorzec gdy masz klasę zaśmieconą rozbudowanymi instrukcjami
warunkowymi zmieniającymi zachowanie klasy zależnie od wartości jej pól.
:::

::: applicability-solution
Wzorzec Stan pozwala wyekstrahować rozgałęzienia tych instrukcji
warunkowych do metod które znajdą się w klasach reprezentujących
poszczególne stany. W ten sposób uprzątnąć można przy okazji tymczasowe
pola i metody pomocnicze związane z kodem odnoszącym się do stanów.
:::

::: applicability-problem
Wzorzec Stan pomaga poradzić sobie z dużą ilością kodu który się
powtarza w wielu stanach i przejściach między stanami automatu
skończonego, bazującego na instrukcjach warunkowych.
:::

::: applicability-solution
Wzorzec Stan pozwala komponować hierarchie klas stanów i zmniejszyć
ilość powtórzeń kodu poprzez ekstrakcję wspólnego kodu do abstrakcyjnych
klas bazowych.
:::
:::::::::
::::::::::

::: {.section .checklist}
##  Jak zaimplementować {#checklist}

1.  Zdecyduj która klasa będzie pełniła rolę kontekstu. Może to być
    istniejąca klasa zawierająca już jakiś kod zależny od stanu obiektu,
    ale może to być także nowa klasa, jeśli kod specyficzny dla stanów
    jest rozrzucony po wielu klasach.

2.  Zadeklaruj interfejs stanu. Mimo że może on odzwierciedlać wszystkie
    metody zadeklarowane w kontekście, skup się tylko na tych, które
    dotyczą zachowania specyficznego dla danego stanu.

3.  Dla każdego faktycznego stanu stwórz klasę wywodzącą się z
    interfejsu stanu. Następnie przejrzyj metody kontekstu i wyekstrahuj
    cały kod dotyczący tego stanu do nowo utworzonej klasy.

    Przenosząc kod do klasy stanu możesz zauważyć, że zależy on od
    prywatnych składowych klasy kontekstu. Można sobie z tym poradzić w
    następujący sposób:

    -   Uczyń te pola lub metody publicznymi.
    -   Zmień ekstrahowane zachowanie na publiczną metodę kontekstu i
        wywołuj ją z klasy stanu. To brzydkie, ale szybkie rozwiązanie,
        które można później naprawić.
    -   Zagnieźdź klasy stanów w klasie kontekstu, ale tylko jeśli
        używany język programowania obsługuje zagnieżdżanie klas.

4.  W klasie kontekstu dodaj pole przechowujące odniesienie do
    interfejsu stanu i publicznie dostępną metodę setter, która
    umożliwia nadpisanie wartości tego pola.

5.  Przejrzyj metodę kontekstu raz jeszcze i zamień puste instrukcje
    warunkowe dotyczące stanu na wywołania stosownych metod obiektu
    stanu.

6.  Aby przełączyć stan kontekstu, utwórz instancję jednej z klas
    stanu i przekaż ją kontekstowi. Można tego dokonać w ramach samego
    kontekstu, w którymś ze stanów, bądź po stronie klienta. Za każdym
    razem, gdy to się dzieje, klasa staje się zależna od konkretnej
    klasy stanu której instancja powstaje.
:::

:::::: {.section .pros-cons}
##  Zalety i wady {#pros-cons}

::::: row
::: col-sm-6
-    *Zasada pojedynczej odpowiedzialności*. Zorganizuj kod związany z
    konkretnymi stanami w osobne klasy.
-    *Zasada otwarte/zamknięte*. Można wprowadzać nowe stany bez zmiany
    istniejących klas stanu lub kontekstu.
-    Upraszcza kod kontekstu eliminując obszerne instrukcje warunkowe
    automatu skończonego.
:::

::: col-sm-6
-    Zastosowanie tego wzorca może być przesadą jeśli mamy do czynienia
    zaledwie z kilkoma stanami i rzadkimi zmianami.
:::
:::::
::::::

::: {.section .relations}
##  Powiązania z innymi wzorcami {#relations}

-   [Most](bridge.html), [Stan](state.html), [Strategia](strategy.html)
    (i w pewnym stopniu [Adapter](adapter.html)) mają podobną strukturę.
    Wszystkie oparte są na kompozycji, co oznacza delegowanie zadań
    innym obiektom. Jednak każdy z tych wzorców rozwiązuje inne
    problemy. Wzorzec nie jest bowiem tylko receptą na ustrukturyzowanie
    kodu w pewien sposób, lecz także informacją dla innych deweloperów o
    charakterze rozwiązywanego problemu.

-   [Stan](state.html) można uważać za rozszerzenie
    [Strategii](strategy.html). Oba wzorce oparte są o kompozycję:
    zmieniają zachowanie kontekstu przez delegowanie części zadań
    obiektom pomocniczym. *Strategia* czyni te obiekty całkowicie
    niezależnymi i nieświadomymi siebie nawzajem. Jednakże *Stan* nie
    ogranicza zależności pomiędzy konkretnymi stanami i pozwala im
    zmieniać stan kontekstu według uznania.
:::

::: {.section .implementations}
##  Przykłady kodu {#implementations}

[![Stan w języku
C#](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](state/csharp/example.html "Stan w języku C#"){.prog-lang-link}
[![Stan w języku
C++](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](state/cpp/example.html "Stan w języku C++"){.prog-lang-link}
[![Stan w języku
Go](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](state/go/example.html "Stan w języku Go"){.prog-lang-link}
[![Stan w języku
Java](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](state/java/example.html "Stan w języku Java"){.prog-lang-link}
[![Stan w języku
PHP](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](state/php/example.html "Stan w języku PHP"){.prog-lang-link}
[![Stan w języku
Python](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](state/python/example.html "Stan w języku Python"){.prog-lang-link}
[![Stan w języku
Ruby](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](state/ruby/example.html "Stan w języku Ruby"){.prog-lang-link}
[![Stan w języku
Rust](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](state/rust/example.html "Stan w języku Rust"){.prog-lang-link}
[![Stan w języku
Swift](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](state/swift/example.html "Stan w języku Swift"){.prog-lang-link}
[![Stan w języku
TypeScript](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](state/typescript/example.html "Stan w języku TypeScript"){.prog-lang-link}
:::

:::::: {#book-promo .banner-set}
::::: {.prom .banner-content .banner-bg .banner-striped .banner-with-image data-id="DP: 1: Ne vtykay" creative-id="standard-pl" position="content_bottom"}
::: banner-image
[![](../../images/patterns/banners/patterns-book-banner-1-b158a.png?id=0877cab2f0102d98cd07b50af0e5beea){width="200"
height="200" loading="lazy"
srcset="/images/patterns/banners/patterns-book-banner-1-b-2x.png?id=5572aa55e5b09e59780aca9e0ea8e44b 2x"}](book.html)
:::

::: banner-text
### Nie utknij w podróży! {#nie-utknij-w-podróży .title}

Niech czas spędzony w pociągu/autobusie/samolocie działa na twoją
korzyść.

Nie potrzebujesz dostępu do Internetu, zakładek, lampki ani dodatkowego
miejsca w walizce.

[ Dowiedz się więcej...](book.html){.btn .btn-secondary}
:::
:::::
::::::

::: next
#### Czytaj dalej

[Strategia []{.fa .fa-arrow-right}](strategy.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Wróć

[[]{.fa .fa-arrow-left} Obserwator](observer.html){.btn .btn-default
rel="prev"}
:::
::::::::::::::::::::::::::::::::::

::::::: {.prom .banner-sidebar .banner-removable .banner-removable-patterns data-id="DP: Sidebar" creative-id="standard-sidebar-pl" position="sidebar"}
:::::: banner-inner
:::: image3d-book-right
::: {.image3d-cover style="background: #0b3752;"}
[![](../../images/patterns/book/web-cover-ple6f3.png?id=f953bc70414a52220fc93e0c654543da){width="250"
height="375" loading="lazy"
srcset="/images/patterns/book/web-cover-pl-2x.png?id=ce168392842e737843bd90996318b24c 2x"}](book.html)
:::
::::

::: {style="margin-top: 1rem"}
Ten artykuł jest fragmentem naszego eBooka **Wzorce projektowe.
Nowoczesny podręcznik**.

[ Dowiedz się więcej...](book.html){.btn .btn-secondary .btn-block}
:::
::::::
:::::::
::::::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::::::
