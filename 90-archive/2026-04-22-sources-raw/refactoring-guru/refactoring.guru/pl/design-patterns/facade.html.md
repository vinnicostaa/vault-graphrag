::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [Wzorce
projektowe](../design-patterns.html){.type} / [Wzorce
strukturalne](structural-patterns.html){.type}
:::

# Fasada {#fasada .title}

::: aka
Znany też jako: [Facade]{style="display:inline-block"}
:::

::: {.section .intent}
##  Cel {#intent}

**Fasada** jest strukturalnym wzorcem projektowym, który wyposaża
bibliotekę, framework lub inny złożony zestaw klas w
uproszczony interfejs.

<figure class="image">
<img
src="../../images/patterns/content/facade/facade721d.png?id=1f4be17305b6316fbd548edf1937ac3b"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/facade/facade.png?id=1f4be17305b6316fbd548edf1937ac3b 640w,/images/patterns/content/facade/facade-1.5x.png?id=a6af5003b243b59990842d24bdaae2df 960w,/images/patterns/content/facade/facade-2x.png?id=b69fce5943703f5f07c0ba38e3baaed0 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Wzorzec projektowy Fasada" />
</figure>
:::

::: {.section .problem}
##  Problem

Wyobraź sobie, że twój kod musi współdziałać z szerokim zestawem
obiektów należących do jakiejś skomplikowanej biblioteki lub frameworku.
Zazwyczaj należy zainicjalizować wszystkie te obiekty, śledzić
zależności, wywoływać metody w odpowiedniej kolejności i tak dalej.

W rezultacie doszłoby do ścisłego sprzęgnięcia logiki biznesowej twoich
klas ze szczegółami implementacji klas innych dostawców, co utrudniłoby
utrzymanie i zrozumienie kodu.
:::

::: {.section .solution}
##  Rozwiązanie {#solution}

Fasada to klasa stanowiąca prosty interfejs dla złożonego podsystemu,
zawierającego mnóstwo ruchomych części. Fasada może dawać ograniczoną
funkcjonalność, w porównaniu z korzystaniem z elementów podsystemu
bezpośrednio, ale za to eksponuje tylko te możliwości, których klient
naprawdę potrzebuje.

Stworzenie fasady jest wygodnym sposobem integracji twej aplikacji ze
skomplikowaną biblioteką posiadającą wiele funkcji, gdy potrzebujesz
tylko wąskiego zakresu jej funkcji.

Na przykład, aplikacja która publikuje krótkie zabawne filmiki z
kotami w mediach społecznościowych, może korzystać z profesjonalnej
biblioteki konwersji wideo. Z całej biblioteki potrzebuje jednak tylko
klasy z jedną metodą --- `zakoduj(nazwa_pliku, format)`. Po stworzeniu
takiej klasy i podłączeniu jej do biblioteki konwersji wideo otrzymamy
naszą pierwszą fasadę.
:::

::: {.section .analogy}
##  Analogia do prawdziwego życia {#analogy}

<figure class="image">
<img
src="../../images/patterns/diagrams/facade/live-example-pl4ed0.png?id=ea54d3d4513ec376e293904c6bfe8562"
style="aspect-ratio: 2.58;"
srcset="/images/patterns/diagrams/facade/live-example-pl.png?id=ea54d3d4513ec376e293904c6bfe8562 490w,/images/patterns/diagrams/facade/live-example-pl-1.5x.png?id=3d2fe22a7769c501f6f079887a6366c6 735w,/images/patterns/diagrams/facade/live-example-pl-2x.png?id=8d708d10e9be62adceecc7dddef6e74e 980w"
sizes="(max-width: 720px) 100vw, 490px" loading="lazy" width="490"
alt="Przykład składania zamówienia przez telefon" />
<figcaption><p>Składanie zamówienia przez telefon.</p></figcaption>
</figure>

Gdy dzwonisz do sklepu aby złożyć zamówienie, biuro jest twoją fasadą
dla wszystkich usług i oddziałów tego sklepu. Pracownik sklepu, czy
automat zgłoszeniowy, stanowią prosty interfejs głosowy do systemu
zamawiania, płacenia i różnych usług dostawczych.
:::

::::: {.section .structure-container}
##  Struktura {#structure}

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/facade/structure2883.png?id=258401362234ac77a2aaf1cde62339e7"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.47;"
srcset="/images/patterns/diagrams/facade/structure.png?id=258401362234ac77a2aaf1cde62339e7 560w,/images/patterns/diagrams/facade/structure-1.5x.png?id=98d134de0c368d382909ba162ec21767 840w,/images/patterns/diagrams/facade/structure-2x.png?id=528ca429555bce293b7c3bd90954e097 1120w"
sizes="(max-width: 720px) 100vw, 560px" loading="lazy" width="560"
alt="Struktura wzorca projektowego Fasada" /><img
src="../../images/patterns/diagrams/facade/structure-indexeddb1c.png?id=2da06d6b850701ea15cf72f9d2642fb8"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.58;"
srcset="/images/patterns/diagrams/facade/structure-indexed.png?id=2da06d6b850701ea15cf72f9d2642fb8 600w,/images/patterns/diagrams/facade/structure-indexed-1.5x.png?id=fad7d667f4d8d9a7d522748bcd37dfde 900w,/images/patterns/diagrams/facade/structure-indexed-2x.png?id=4d181bcf1df5d58c533e6c92461e9571 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Struktura wzorca projektowego Fasada" />
</figure>
:::

1.  **Fasada** daje wygodny dostęp do pewnego zakresu funkcjonalności
    podsystemu. Wie dokąd przekierować żądanie klienta i jak pokierować
    wszystkimi szczegółami.

2.  Można stworzyć klasę **Dodatkowa Fasada**, by zapobiec zaśmieceniu
    pojedynczej fasady niepotrzebnymi funkcjami, które ponownie
    ograniczyłyby prostotę używania. Dodatkowe fasady mogą być
    wykorzystane zarówno przez klienta, jak i inne fasady.

3.  **Złożony podsystem** składa się z wielu różnych obiektów. Aby mogły
    one wszystkie wykonać coś pożytecznego, trzeba zagłębić się w
    szczegóły implementacyjne podsystemu --- inicjalizację obiektów we
    właściwej kolejności i przekazanie im danych w odpowiednim formacie.

    Klasy podsystemu nie są świadome istnienia fasady. Działają wewnątrz
    systemu i współpracują ze sobą bezpośrednio.

4.  **Klient** korzysta z fasady zamiast wywoływać obiekty podsystemu
    bezpośrednio.
::::
:::::

::: {.section .pseudocode}
##  Pseudokod {#pseudocode}

W poniższym przykładzie, wzorzec **Fasada** upraszcza interakcję ze
złożonym frameworkiem do konwersji wideo.

<figure class="image">
<img
src="../../images/patterns/diagrams/facade/example16bc.png?id=2249d134e3ff83819dfc19032f02eced"
style="aspect-ratio: 1.5;"
srcset="/images/patterns/diagrams/facade/example.png?id=2249d134e3ff83819dfc19032f02eced 570w,/images/patterns/diagrams/facade/example-1.5x.png?id=034ecbe0f3cc108f4ae49d05d1c77dbe 855w,/images/patterns/diagrams/facade/example-2x.png?id=f2c846d74041626c923ff3e8919b68a9 1140w"
sizes="(max-width: 720px) 100vw, 570px" loading="lazy" width="570"
alt="Struktura przykładu wzorca projektowego Fasada" />
<figcaption><p>Przykład izolowania wielu zależności w pojedynczej
klasie fasada.</p></figcaption>
</figure>

Zamiast wiązać bezpośrednio kod z wieloma klasami składowymi frameworku,
tworzymy klasę fasady która hermetyzuje funkcjonalność i ukrywa ją przed
resztą kodu. Ta struktura pozwala też zminimalizować wysiłek związany z
aktualizacją frameworku do nowszej wersji lub wręcz wymiany na inny.
Jedyna rzecz, jaką trzeba będzie wówczas zmienić, to implementacja metod
fasady.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// Oto klasy złożonego frameworku od zewnętrznego dostawcy
// służącego konwersji wideo. Nie mamy wpływu na ten kod, więc
// nie możemy go uprościć.

class VideoFile
// ...

class OggCompressionCodec
// ...

class MPEG4CompressionCodec
// ...

class CodecFactory
// ...

class BitrateReader
// ...

class AudioMixer
// ...


// Tworzymy klasę fasada która ukryje złożoność frameworku za
// prostym interfejsem. Takie podejście jest kompromisem
// pomiędzy funkcjonalnością, a łatwością użycia.
class VideoConverter is
    method convert(filename, format):File is
        file = new VideoFile(filename)
        sourceCodec = (new CodecFactory).extract(file)
        if (format == &quot;mp4&quot;)
            destinationCodec = new MPEG4CompressionCodec()
        else
            destinationCodec = new OggCompressionCodec()
        buffer = BitrateReader.read(filename, sourceCodec)
        result = BitrateReader.convert(buffer, destinationCodec)
        result = (new AudioMixer()).fix(result)
        return new File(result)

// Klasy aplikacji nie są zależne od masy klas wchodzących w
// skład frameworku. Ponadto, w przypadku konieczności wymiany
// frameworku na inny, trzeba będzie zmodyfikować jedynie klasę
// fasada.
class Application is
    method main() is
        convertor = new VideoConverter()
        mp4 = convertor.convert(&quot;funny-cats-video.ogg&quot;, &quot;mp4&quot;)
        mp4.save()</code></pre>
</figure>
:::

:::::::: {.section .applicability-container}
##  Zastosowanie {#applicability}

::::::: applicability
::: applicability-problem
Użyj wzorca Fasada gdy potrzebujesz ograniczonego, ale łatwego w użyciu
interfejsu do złożonego podsystemu.
:::

::: applicability-solution
Zazwyczaj podsystemy stają się coraz bardziej złożone z biegiem czasu.
Nawet stosowanie wzorców projektowych prowadzi do przyrostu liczby klas.
Podsystem może wprawdzie stać się elastyczniejszym i łatwiejszym do
ponownego użycia w różnych kontekstach, ale ilość kodu konfigurującego i
przygotowawczego wymagana od klienta wzrośnie. Fasada jest sposobem
rozwiązania tego problemu poprzez udostępnienie skrótów do najczęściej
używanych funkcji podsystemu, zgodnie z wymaganiami klienta.
:::

::: applicability-problem
Stosuj Fasadę gdy chcesz ustrukturyzować podsystem w warstwy.
:::

::: applicability-solution
Twórz fasady by zdefiniować punkty wejścia do każdego poziomu
podsystemu. Możesz ograniczyć sprzęgnięcie pomiędzy wieloma
podsystemami, zmuszając je do komunikacji ze sobą wyłącznie poprzez
fasady.

Dla przykładu, wróćmy do naszego frameworku konwersji wideo. Można go
podzielić na dwie warstwy: związaną z obrazem i związaną z dźwiękiem.
Dla każdej z warstw możesz utworzyć fasadę i sprawić, by klasy każdej
warstwy komunikowały się ze sobą za pośrednictwem tych fasad. Takie
podejście przypomina wzorzec [Mediator](mediator.html).
:::
:::::::
::::::::

::: {.section .checklist}
##  Jak zaimplementować {#checklist}

1.  Sprawdź czy możliwe jest utworzenie prostszego interfejsu w
    porównaniu z tym, jaki dostarcza istniejący podsystem. Jeśli
    planowany interfejs sprawiłby, że kod kliencki będzie niezależny od
    wielu klas podsystemu --- jesteś na właściwej drodze.

2.  Zadeklaruj i zaimplementuj ten interfejs jako nową klasę fasada.
    Fasada powinna przekierowywać wywołania pochodzące od kodu
    klienckiego do stosownych obiektów podsystemu. Ponadto, fasada
    powinna być odpowiedzialna za inicjalizację podsystemu i zarządzanie
    jego dalszym cyklem życia, chyba, że kod kliencki już pełni tę rolę.

3.  Aby w pełni skorzystać na tym wzorcu, spraw, aby kod kliencki
    komunikował się z podsystemem wyłącznie poprzez fasadę. W ten sposób
    kod kliencki pozostanie chroniony przed zmianami dokonywanymi w
    kodzie podsystemu. Na przykład, aktualizacja podsystemu będzie
    wymagała jedynie zmian w kodzie fasady.

4.  Jeśli fasada stanie się [zbyt duża](../smells/large-class.html),
    rozważ ekstrakcję części jej obowiązków do nowej, doskonalszej klasy
    fasady.
:::

:::::: {.section .pros-cons}
##  Zalety i wady {#pros-cons}

::::: row
::: col-sm-6
-    Można odizolować kod od złożoności podsystemu.
:::

::: col-sm-6
-    Fasada może stać się [boskim
    obiektem](https://en.wikipedia.org/wiki/God_object) sprzężonym ze
    wszystkimi klasami aplikacji.
:::
:::::
::::::

::: {.section .relations}
##  Powiązania z innymi wzorcami {#relations}

-   [Fasada](facade.html) definiuje nowy interfejs istniejącym obiektom,
    zaś [Adapter](adapter.html) zakłada zwiększenie użyteczności
    zastanego interfejsu. *Adapter* na ogół opakowuje pojedynczy obiekt,
    zaś *Fasada* obejmuje cały podsystem obiektów.

-   [Fabryka abstrakcyjna](abstract-factory.html) może służyć jako
    alternatywa do [Fasady](facade.html) gdy jedyne co chcesz zrobić, to
    ukrycie przed kodem klienckim procesu tworzenia obiektów podsystemu.

-   [Pyłek](flyweight.html) przedstawia sposób na stworzenie wielkiej
    liczby małych obiektów, zaś [Fasada](facade.html) na stworzenie
    pojedynczego obiektu reprezentującego cały podsystem.

-   [Fasada](facade.html) i [Mediator](mediator.html) mają podobne
    zadania: służą zorganizowaniu współpracy pomiędzy wieloma ściśle
    sprzęgniętymi klasami.

    -   *Fasada* definiuje uproszczony interfejs podsystemu obiektów,
        ale nie wprowadza nowej funkcjonalności. Podsystem jest
        nieświadomy istnienia fasady. Obiekty w obrębie podsystemu mogą
        komunikować się bezpośrednio.
    -   *Mediator* centralizuje komunikację pomiędzy komponentami
        podsystemu. Komponenty wiedzą tylko o obiekcie mediator i nie
        komunikują się ze sobą bezpośrednio.

-   Klasa [Fasada](facade.html) może często być przekształcona w
    [Singleton](singleton.html), ponieważ pojedynczy obiekt fasady
    jest w większości przypadków wystarczający.

-   [Fasada](facade.html) przypomina wzorzec [Pełnomocnik](proxy.html) w
    tym sensie, że oba buforują złożony podmiot i inicjalizują go
    samodzielnie. W przeciwieństwie do *Fasady*, *Pełnomocnik* ma taki
    sam interfejs jak obiekt udostępniający usługę który ją
    reprezentuje, co czyni je wymienialnymi.
:::

::: {.section .implementations}
##  Przykłady kodu {#implementations}

[![Fasada w języku
C#](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](facade/csharp/example.html "Fasada w języku C#"){.prog-lang-link}
[![Fasada w języku
C++](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](facade/cpp/example.html "Fasada w języku C++"){.prog-lang-link}
[![Fasada w języku
Go](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](facade/go/example.html "Fasada w języku Go"){.prog-lang-link}
[![Fasada w języku
Java](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](facade/java/example.html "Fasada w języku Java"){.prog-lang-link}
[![Fasada w języku
PHP](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](facade/php/example.html "Fasada w języku PHP"){.prog-lang-link}
[![Fasada w języku
Python](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](facade/python/example.html "Fasada w języku Python"){.prog-lang-link}
[![Fasada w języku
Ruby](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](facade/ruby/example.html "Fasada w języku Ruby"){.prog-lang-link}
[![Fasada w języku
Rust](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](facade/rust/example.html "Fasada w języku Rust"){.prog-lang-link}
[![Fasada w języku
Swift](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](facade/swift/example.html "Fasada w języku Swift"){.prog-lang-link}
[![Fasada w języku
TypeScript](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](facade/typescript/example.html "Fasada w języku TypeScript"){.prog-lang-link}
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

[Pyłek []{.fa .fa-arrow-right}](flyweight.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Wróć

[[]{.fa .fa-arrow-left} Dekorator](decorator.html){.btn .btn-default
rel="prev"}
:::
::::::::::::::::::::::::::::::::

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
::::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::::
