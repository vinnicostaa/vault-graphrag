::::::::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [Wzorce
projektowe](../design-patterns.html){.type} / [Wzorce
strukturalne](structural-patterns.html){.type}
:::

# Pełnomocnik {#pełnomocnik .title}

::: aka
Znany też jako: [Proxy]{style="display:inline-block"}
:::

::: {.section .intent}
##  Cel {#intent}

**Pełnomocnik** to strukturalny wzorzec projektowy pozwalający stworzyć
obiekt zastępczy w miejsce innego obiektu. Pełnomocnik nadzoruje
dostęp do pierwotnego obiektu, pozwalając na wykonanie jakiejś czynności
przed lub po przekazaniu do niego żądania.

<figure class="image">
<img
src="../../images/patterns/content/proxy/proxyf258.png?id=efece4647fb11e3f7539291796327666"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/proxy/proxy.png?id=efece4647fb11e3f7539291796327666 640w,/images/patterns/content/proxy/proxy-1.5x.png?id=80a11690fa49172c517470a834289b60 960w,/images/patterns/content/proxy/proxy-2x.png?id=fb3d14e21c210a758d4777f4d93dce09 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Wzorzec projektowy Pełnomocnik" />
</figure>
:::

::: {.section .problem}
##  Problem

Po co właściwie kontrolować dostęp do obiektu? Załóżmy, że mamy duży
obiekt, który zużywa dużo zasobów. Potrzebujesz go co jakiś czas, ale
nie ciągle.

<figure class="image">
<img
src="../../images/patterns/diagrams/proxy/problem-pl856f.png?id=94053f69f628299844fd5d5a14044c19"
style="aspect-ratio: 3.19;"
srcset="/images/patterns/diagrams/proxy/problem-pl.png?id=94053f69f628299844fd5d5a14044c19 510w,/images/patterns/diagrams/proxy/problem-pl-1.5x.png?id=82f1a70a63f0a07334c695f02970daed 765w,/images/patterns/diagrams/proxy/problem-pl-2x.png?id=4797ff005fc3b3807348b699ea84c1c9 1020w"
sizes="(max-width: 720px) 100vw, 510px" loading="lazy" width="510"
alt="Rozwiązanie problemu za pomocą wzorca Pełnomocnik" />
<figcaption><p>Zapytania bazodanowe mogą być
bardzo powolne.</p></figcaption>
</figure>

Moglibyśmy zastosować leniwą inicjalizację: tworzyć ten obiekt tylko gdy
staje się faktycznie potrzebny. Wszystkie jego klienty musiałyby wykonać
jakiś opóźniony kod inicjalizujący. Niestety doprowadziłoby to do
powielania kodu.

W idealnym świecie, chcielibyśmy umieścić ten kod w klasie obiektu, ale
nie zawsze jest to możliwe. Klasa może być bowiem częścią zamkniętej
biblioteki innego dostawcy.
:::

::: {.section .solution}
##  Rozwiązanie {#solution}

Wzorzec Pełnomocnik zakłada stworzenie nowej klasy pośredniczącej, o
takim samym interfejsie co pierwotny obiekt udostępniający usługę.
Następnie aktualizujemy nasz program tak, aby przekazywał obiekt
pełnomocnika wszystkim klientom pierwotnego obiektu. Otrzymawszy
żądanie od klienta, pełnomocnik tworzy prawdziwy obiekt usługi i
deleguje mu całą pracę.

<figure class="image">
<img
src="../../images/patterns/diagrams/proxy/solution-pl5675.png?id=96ba3c8bcc5661c42075e48dcb74106b"
style="aspect-ratio: 3.19;"
srcset="/images/patterns/diagrams/proxy/solution-pl.png?id=96ba3c8bcc5661c42075e48dcb74106b 510w,/images/patterns/diagrams/proxy/solution-pl-1.5x.png?id=aafffc25775305e09e0218516884d947 765w,/images/patterns/diagrams/proxy/solution-pl-2x.png?id=43f220fcb5ed91054e421a27107f98c8 1020w"
sizes="(max-width: 720px) 100vw, 510px" loading="lazy" width="510"
alt="Rozwiązanie z użyciem wzorca Pełnomocnik" />
<figcaption><p>Pełnomocnik udaje obiekt bazodanowy. Może zająć się
leniwą inicjalizacją i przechowywaniem wyników bez wiedzy klienta i
samej bazy danych.</p></figcaption>
</figure>

Ale co z tego mamy? Jeśli musisz uruchomić coś albo przed, albo po
głównej logice klasy, pełnomocnik pozwala uczynić to bez modyfikowania
tej klasy. Ponieważ pełnomocnik implementuje ten sam interfejs, co
pierwotna klasa, może być przekazany do dowolnego klienta wymagającego
prawdziwego obiektu udostępniającego usługę.
:::

::: {.section .analogy}
##  Analogia do prawdziwego życia {#analogy}

<figure class="image">
<img
src="../../images/patterns/diagrams/proxy/live-example66da.png?id=a268c57fdaf073ee81cf4dfc7239eae2"
style="aspect-ratio: 2.57;"
srcset="/images/patterns/diagrams/proxy/live-example.png?id=a268c57fdaf073ee81cf4dfc7239eae2 540w,/images/patterns/diagrams/proxy/live-example-1.5x.png?id=4b8ee7f90cf76180004e9f5d37661ee2 810w,/images/patterns/diagrams/proxy/live-example-2x.png?id=8b8083fa1954d2d92ca84a5a5636ead7 1080w"
sizes="(max-width: 720px) 100vw, 540px" loading="lazy" width="540"
alt="Karta kredytowa służy za pełnomocnika dla zwitka banknotów" />
<figcaption><p>Płacić można zarówno kartą kredytową, jak
i gotówką.</p></figcaption>
</figure>

Karta kredytowa jest pełnomocnikiem konta bankowego, które z kolei jest
pełnomocnikiem gotówki. Oba implementują taki sam interfejs: można za
ich pomocą płacić. Klient jest zadowolony, gdyż nie musi nosić przy
sobie gotówki. Sklepikarz również jest zadowolony, bo przychody z
transakcji są elektronicznie dodawane do konta bankowego sklepu bez
ryzyka zagubienia czy kradzieży utargu.
:::

::::: {.section .structure-container}
##  Struktura {#structure}

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/proxy/structure7fcd.png?id=f2478a82a84e1a1e512a8414bf1abd1c"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1;"
srcset="/images/patterns/diagrams/proxy/structure.png?id=f2478a82a84e1a1e512a8414bf1abd1c 370w,/images/patterns/diagrams/proxy/structure-1.5x.png?id=0db7bf3c1193b2a1961894349f1e07ad 555w,/images/patterns/diagrams/proxy/structure-2x.png?id=3d54eeca9af4aa373e989a73463539b5 740w"
sizes="(max-width: 720px) 100vw, 370px" loading="lazy" width="370"
alt="Struktura wzorca projektowego Pełnomocnik" /><img
src="../../images/patterns/diagrams/proxy/structure-indexed2b2c.png?id=e56d420f31925b3d41348c5036d3cd64"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.05;"
srcset="/images/patterns/diagrams/proxy/structure-indexed.png?id=e56d420f31925b3d41348c5036d3cd64 410w,/images/patterns/diagrams/proxy/structure-indexed-1.5x.png?id=5b24af632e76d81d24eab0b7c0be1a66 615w,/images/patterns/diagrams/proxy/structure-indexed-2x.png?id=ddf2b3b4807e52330c456c59fc52d504 820w"
sizes="(max-width: 720px) 100vw, 410px" loading="lazy" width="410"
alt="Struktura wzorca projektowego Pełnomocnik" />
</figure>
:::

1.  **Interfejs Usługi** deklaruje interfejs z którym Pełnomocnik musi
    być zgodny, aby móc udawać obiekt usługi.

2.  **Usługa** to klasa udostępniająca jakąś użyteczną logikę biznesową.

3.  Klasa **Pełnomocnik** zawiera pole z odniesieniem do konkretnego
    obiektu udostępniającego usługę. Po wykonaniu swoich zadań (leniwa
    inicjalizacja, zapis w dzienniku, kontrola dostępu, przechowanie w
    pamięci podręcznej, itp.) Pełnomocnik przekazuje żądanie do tego
    obiektu.

    Pośrednicy zazwyczaj zarządzają całym cyklem życia swych obiektów
    usługowych.

4.  **Klient** powinien móc pracować zarówno z usługami, jak i
    pełnomocnikami za pośrednictwem takiego samego interfejsu. Dzięki
    temu można umieścić pełnomocnika w dowolnym kodzie, który ma
    współdziałać z obiektem usługowym.
::::
:::::

::: {.section .pseudocode}
##  Pseudokod {#pseudocode}

Poniższy przykład ilustruje jak wzorzec **Pełnomocnik** może pomóc dodać
obsługę leniwej inicjalizacji i pamięci podręcznej do biblioteki innego
producenta, odpowiedzialnej za integrację z YouTube.

<figure class="image">
<img
src="../../images/patterns/diagrams/proxy/example4e80.png?id=ddb1402479562b9e2c776933cc861bed"
style="aspect-ratio: 1.23;"
srcset="/images/patterns/diagrams/proxy/example.png?id=ddb1402479562b9e2c776933cc861bed 490w,/images/patterns/diagrams/proxy/example-1.5x.png?id=9b7608e1ef46a52d4ca1d7d89986e5b0 735w,/images/patterns/diagrams/proxy/example-2x.png?id=40f22d12d183b9285a380e4a072efb16 980w"
sizes="(max-width: 720px) 100vw, 490px" loading="lazy" width="490"
alt="Struktura przykładu wzorca Pełnomocnik" />
<figcaption><p>Przechowywanie w pamięci podręcznej danych uzyskanych od
usługi za pomocą pełnomocnika.</p></figcaption>
</figure>

Biblioteka udostępnia klasę do pobierania filmów wideo. Jest jednak
bardzo nieefektywna. W przypadku otrzymania żądania pobrania drugi raz
tego samego filmu, biblioteka pobierze go od nowa, zamiast buforować
dane pozyskane podczas poprzedniego pobrania.

Klasa pośrednicząca implementuje ten sam interfejs, co pierwotne
narzędzie do pobierania i deleguje mu całą pracę. Zapamiętuje jednak
pobrane pliki i zwraca dane z pamięci podręcznej, jeśli otrzyma żądanie
pobrania tego samego filmu kolejny raz.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// Interfejs zdalnej usługi.
interface ThirdPartyYouTubeLib is
    method listVideos()
    method getVideoInfo(id)
    method downloadVideo(id)

// Konkretna implementacja łącza do usługi. Metody tej klasy
// mogą żądać informacji z YouTube. Szybkość realizacji żądania
// zależy od połączenia internetowego użytkownika oraz od samego
// YouTube. Aplikacja będzie działać wolniej, jeśli wiele żądań
// zostanie wysłanych jednocześnie, nawet jeśli wszystkie
// żądania dotyczą tych samych danych.
class ThirdPartyYouTubeClass implements ThirdPartyYouTubeLib is
    method listVideos() is
        // Wyślij żądanie do interfejsu programowania aplikacji
        // (API) YouTube.

    method getVideoInfo(id) is
        // Pobierz metadane pliku wideo.

    method downloadVideo(id) is
        // Pobierz plik wideo z YouTube.

// Aby zaoszczędzić nieco przepustowości, możemy stworzyć pamięć
// podręczną do przechowywania pobranych danych i przechowywać
// je jakiś czas. Jednak umieszczenie takiego kodu bezpośrednio
// w klasie usługi może być niemożliwe, jeśli na przykład
// stanowi część biblioteki od zewnętrznego dostawcy i/lub jest
// zdefiniowana jako `final`. Dlatego też umieszczamy kod
// pamięci podręcznej w odrębnej klasie pośredniczącej która
// implementuje taki sam interfejs co klasa usługi. Nowo
// powstała klasa deleguje żądania faktycznemu obiektowi usługi
// tylko wtedy gdy faktycznie trzeba przesłać żądanie przez
// sieć.
class CachedYouTubeClass implements ThirdPartyYouTubeLib is
    private field service: ThirdPartyYouTubeLib
    private field listCache, videoCache
    field needReset

    constructor CachedYouTubeClass(service: ThirdPartyYouTubeLib) is
        this.service = service

    method listVideos() is
        if (listCache == null || needReset)
            listCache = service.listVideos()
        return listCache

    method getVideoInfo(id) is
        if (videoCache == null || needReset)
            videoCache = service.getVideoInfo(id)
        return videoCache

    method downloadVideo(id) is
        if (!downloadExists(id) || needReset)
            service.downloadVideo(id)

// Klasa GUI która dotychczas współpracowała bezpośrednio z
// obiektem oferującym usługę pozostaje niezmieniona, o ile
// będzie współpracować z obiektem usługi poprzez interfejs.
// Możemy śmiało przekazać obiekt pośrednika zamiast obiektu
// usługi ponieważ implementują one ten sam interfejs.
class YouTubeManager is
    protected field service: ThirdPartyYouTubeLib

    constructor YouTubeManager(service: ThirdPartyYouTubeLib) is
        this.service = service

    method renderVideoPage(id) is
        info = service.getVideoInfo(id)
        // Renderuj stronę z filmem.

    method renderListPanel() is
        list = service.listVideos()
        // Renderuj listę miniaturek plików wideo.

    method reactOnUserInput() is
        renderVideoPage()
        renderListPanel()

// Aplikacja może konfigurować pośredników w trakcie działania.
class Application is
    method init() is
        aYouTubeService = new ThirdPartyYouTubeClass()
        aYouTubeProxy = new CachedYouTubeClass(aYouTubeService)
        manager = new YouTubeManager(aYouTubeProxy)
        manager.reactOnUserInput()</code></pre>
</figure>
:::

:::::::::::::::: {.section .applicability-container}
##  Zastosowanie {#applicability}

::::::::::::::: applicability
Wzorzec Pełnomocnik może przydać się w wielu przypadkach. Przejrzyjmy
najpopularniejsze przypadki użycia tej koncepcji.

::: applicability-problem
Leniwa inicjalizacja (wirtualny pełnomocnik). Gdy masz do czynienia z
zasobożernym obiektem usługi, którego potrzebujesz jedynie co jakiś
czas.
:::

::: applicability-solution
Zamiast tworzyć obiekt podczas uruchamiania aplikacji, możesz opóźnić
inicjalizację obiektu do momentu gdy faktycznie staje się potrzebny.
:::

::: applicability-problem
Kontrola dostępu (pełnomocnik ochronny). Przydatne, gdy chcesz pozwolić
tylko niektórym klientom na korzystanie z obiektu usługi. Na przykład,
gdy usługi stanowią kluczową część systemu operacyjnego, a klienci to
różne uruchamiane aplikacje (również te szkodliwe).
:::

::: applicability-solution
Pełnomocnik może przekazać żądanie obiektowi usługi tylko wtedy, gdy
klient przedstawi odpowiednie poświadczenia.
:::

::: applicability-problem
Lokalne uruchamianie zdalnej usługi (pełnomocnik zdalny). Użyteczne, gdy
obiekt udostępniający usługę znajduje się na zdalnym serwerze.
:::

::: applicability-solution
 W takim przypadku, pełnomocnik przekazuje żądania klienta przez sieć,
biorąc na siebie kłopotliwe szczegóły przesyłu.
:::

::: applicability-problem
Prowadzenie dziennika żądań (pełnomocnik prowadzący dziennik). Pozwala
prowadzić rejestr żądań przesyłanych do obiektu usługi.
:::

::: applicability-solution
Pełnomocnik może zapisywać do dziennika każde żądanie przed
przekazaniem go usłudze.
:::

::: applicability-problem
Przechowywanie w pamięci podręcznej wyników działań (pełnomocnik z
pamięcią podręczną). Pozwala przechować wyniki przekazywanych żądań i
zarządzać cyklem życia pamięci podręcznej. Szczególnie ważne przy dużych
wielkościach danych wynikowych.
:::

::: applicability-solution
Pełnomocnik może implementować pamięć podręczną często powtarzających
się żądań dających ten sam wynik. Można wykorzystać parametry żądania w
charakterze kluczy identyfikujących odpowiedni obszar pamięci
podręcznej.
:::

::: applicability-problem
Sprytne referencje. Można likwidować zasobożerny obiekt, gdy nie ma
klientów którzy go potrzebują.
:::

::: applicability-solution
Pełnomocnik może pozwolić na śledzenie klientów którzy otrzymali
referencję do obiektu usługi lub wyników jego pracy. Co jakiś czas
pełnomocnik może przejrzeć listę klientów, sprawdzając czy wciąż są
aktywni. Jeśli lista klientów okazuje się pusta, pełnomocnik może
zlikwidować obiekt usługi i tym samym zwolnić zasoby systemowe.

Pełnomocnik może też pamiętać, że klient zmodyfikował obiekt-usługę.
Dzięki temu niezmienione obiekty mogą być ponownie wykorzystane przez
innych klientów.
:::
:::::::::::::::
::::::::::::::::

::: {.section .checklist}
##  Jak zaimplementować {#checklist}

1.  Jeśli nie ma istniejącego interfejsu usługi, stwórz go, aby uczynić
    pełnomocnika i obiekt usługi wymiennymi. Ekstrakcja interfejsu z
    klasy usługi nie zawsze jest możliwa, ponieważ trzeba by było
    zmienić wszystkich klientów usługi tak, by komunikowali się przez
    ten interfejs. Plan B to stworzenie pełnomocnika w formie podklasy
    klasy usługi. Dzięki temu pełnomocnik odziedziczy interfejs usługi.

2.  Stwórz klasę pełnomocnika. Powinna posiadać pole służące do
    przechowywania odniesienia do usługi. Zazwyczaj pośrednicy
    zarządzają całym cyklem życia usługodawców, włącznie z tworzeniem
    ich obiektów. W pewnych przypadkach obiekt usługi jest przekazywany
    przez klienta pełnomocnikowi za pośrednictwem konstruktora.

3.  Zaimplementuj metody pełnomocnika zgodnie z ich przeznaczeniem. W
    większości przypadków, po wykonaniu jakiejś części pracy,
    pełnomocnik powinien oddelegować jej resztę obiektowi usługi.

4.  Rozważ utworzenie metody kreacyjnej która decyduje o tym, czy klient
    otrzyma obiekt pełnomocnika, czy faktyczny obiekt usługi. Może to
    być prosta, statyczna metoda w klasie pełnomocnika, albo w pełni
    rozwinięta metoda wytwórcza.

5.  Przemyśl implementację leniwej inicjalizacji obiektu usługi.
:::

:::::: {.section .pros-cons}
##  Zalety i wady {#pros-cons}

::::: row
::: col-sm-6
-    Można sterować obiektem usługi bez wiedzy klientów.
-    Można zarządzać cyklem życia obiektu usługi, gdy klientów to nie
    interesuje.
-    Pełnomocnik działa nawet wtedy, gdy obiekt udostępniający usługę
    nie jest jeszcze gotowy lub dostępny.
-    *Zasada otwarte/zamknięte*. Można wprowadzać nowych
    pełnomocników do aplikacji bez modyfikowania usług lub klientów.
:::

::: col-sm-6
-    Kod może ulec skomplikowaniu, ponieważ trzeba wprowadzić wiele
    nowych klas.
-    Odpowiedzi ze strony usługi mogą ulec opóźnieniu.
:::
:::::
::::::

::: {.section .relations}
##  Powiązania z innymi wzorcami {#relations}

-   Za pomocą [Adapter](adapter.html) można uzyskać dostęp do
    istniejącego obiektu za pośrednictwem innego interfejsu. W przypadku
    [Pełnomocnik](proxy.html) interfejs pozostaje taki sam. Za pomocą
    [Dekorator](decorator.html) uzyskuje się dostęp do obiektu za
    pośrednictwem ulepszonego interfejsu.

-   [Fasada](facade.html) przypomina wzorzec [Pełnomocnik](proxy.html) w
    tym sensie, że oba buforują złożony podmiot i inicjalizują go
    samodzielnie. W przeciwieństwie do *Fasady*, *Pełnomocnik* ma taki
    sam interfejs jak obiekt udostępniający usługę który ją
    reprezentuje, co czyni je wymienialnymi.

-   [Dekorator](decorator.html) i [Pełnomocnik](proxy.html) mają podobne
    struktury, ale inne cele. Oba wzorce bazują na zasadzie
    kompozycji --- jeden obiekt deleguje część zadań innemu.
    *Pełnomocnik* dodatkowo zarządza cyklem życia obiektu
    udostępniającego jakąś usługę, zaś komponowanie *Dekoratorów* leży w
    gestii klienta.
:::

::: {.section .implementations}
##  Przykłady kodu {#implementations}

[![Pełnomocnik w języku
C#](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](proxy/csharp/example.html "Pełnomocnik w języku C#"){.prog-lang-link}
[![Pełnomocnik w języku
C++](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](proxy/cpp/example.html "Pełnomocnik w języku C++"){.prog-lang-link}
[![Pełnomocnik w języku
Go](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](proxy/go/example.html "Pełnomocnik w języku Go"){.prog-lang-link}
[![Pełnomocnik w języku
Java](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](proxy/java/example.html "Pełnomocnik w języku Java"){.prog-lang-link}
[![Pełnomocnik w języku
PHP](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](proxy/php/example.html "Pełnomocnik w języku PHP"){.prog-lang-link}
[![Pełnomocnik w języku
Python](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](proxy/python/example.html "Pełnomocnik w języku Python"){.prog-lang-link}
[![Pełnomocnik w języku
Ruby](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](proxy/ruby/example.html "Pełnomocnik w języku Ruby"){.prog-lang-link}
[![Pełnomocnik w języku
Rust](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](proxy/rust/example.html "Pełnomocnik w języku Rust"){.prog-lang-link}
[![Pełnomocnik w języku
Swift](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](proxy/swift/example.html "Pełnomocnik w języku Swift"){.prog-lang-link}
[![Pełnomocnik w języku
TypeScript](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](proxy/typescript/example.html "Pełnomocnik w języku TypeScript"){.prog-lang-link}
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

[Wzorce behawioralne []{.fa
.fa-arrow-right}](behavioral-patterns.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Wróć

[[]{.fa .fa-arrow-left} Pyłek](flyweight.html){.btn .btn-default
rel="prev"}
:::
::::::::::::::::::::::::::::::::::::::::

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
::::::::::::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::::::::::::
