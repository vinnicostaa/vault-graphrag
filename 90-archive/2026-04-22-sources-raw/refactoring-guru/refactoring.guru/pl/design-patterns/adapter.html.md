:::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [Wzorce
projektowe](../design-patterns.html){.type} / [Wzorce
strukturalne](structural-patterns.html){.type}
:::

# Adapter {#adapter .title}

::: aka
Znany też jako:
[Opakowanie, ]{style="display:inline-block"}[Nakładka, ]{style="display:inline-block"}[Wrapper]{style="display:inline-block"}
:::

::: {.section .intent}
##  Cel {#intent}

**Adapter** jest strukturalnym wzorcem projektowym pozwalającym na
współdziałanie ze sobą obiektów o niekompatybilnych interfejsach.

<figure class="image">
<img
src="../../images/patterns/content/adapter/adapter-pl494a.png?id=48e0c0604a52f160675e1c503a13028f"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/adapter/adapter-pl.png?id=48e0c0604a52f160675e1c503a13028f 640w,/images/patterns/content/adapter/adapter-pl-1.5x.png?id=943414cac76aa9d9272daeebc65aef6b 960w,/images/patterns/content/adapter/adapter-pl-2x.png?id=5584abc0e8784409639a2ad391bad77f 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Wzorzec projektowy Adapter" />
</figure>
:::

::: {.section .problem}
##  Problem

Wyobraź sobie, że tworzysz aplikację monitorującą giełdę. Pobiera ona
dane rynkowe z wielu źródeł w formacie XML, a następnie wyświetla ładnie
wyglądające wykresy i diagramy.

Na jakimś etapie postanawiasz wzbogacić aplikację poprzez dodanie
inteligentnej biblioteki analitycznej innego producenta. Ale jest
haczyk: biblioteka ta działa wyłącznie z danymi w formacie JSON.

<figure class="image">
<img
src="../../images/patterns/diagrams/adapter/problem-ple58d.png?id=a82ab47e09e3c50b48822d0672f97810"
style="aspect-ratio: 2.41;"
srcset="/images/patterns/diagrams/adapter/problem-pl.png?id=a82ab47e09e3c50b48822d0672f97810 530w,/images/patterns/diagrams/adapter/problem-pl-1.5x.png?id=0c3131017f4ccd5ec7829a4e338dbf64 795w,/images/patterns/diagrams/adapter/problem-pl-2x.png?id=bec38829d2723d66492418cb58123f15 1060w"
sizes="(max-width: 720px) 100vw, 530px" loading="lazy" width="530"
alt="Struktura aplikacji przed integracją z biblioteką analityczną" />
<figcaption><p>Nie można zastosować biblioteki analitycznej od razu, bo
wymaga ona przedstawiania danych w formacie który jest niekompatybilny z
twoją aplikacją.</p></figcaption>
</figure>

Można przerobić bibliotekę tak, aby obsługiwała XML. To jednak może
naruszyć działanie istniejącego kodu korzystającego z tej biblioteki. Co
gorsza, możesz nie mieć dostępu do kodu źródłowego biblioteki, co czyni
ten plan niewykonalnym.
:::

::: {.section .solution}
##  Rozwiązanie {#solution}

Możesz stworzyć *adapter*. Jest to specjalny obiekt konwertujący
interfejs jednego z obiektów w taki sposób, że drugi obiekt go rozumie.

Adapter stanowi swego rodzaju opakowanie dla obiektu, ukrywając
szczegóły konwersji jakie odbywają się za kulisami. Obiekt opakowywany
może nawet nie wiedzieć o istnieniu adaptera. Można na przykład opakować
obiekt korzystający z jednostek kilometr i metr w adapter
konwertujący te dane na jednostki imperialne, takie jak stopy i mile.

Adaptery mogą nie tylko konwertować dane pomiędzy formatami, ale również
pozwolić na współpracę obiektów o różnych interfejsach. Działa to tak:

1.  Adapter uzyskuje interfejs kompatybilny z interfejsem jednego z
    obiektów.
2.  Za pomocą tego interfejsu, istniejący obiekt może śmiało wywoływać
    metody adaptera.
3.  Otrzymawszy wywołanie, adapter przekazuje je dalej, ale już w
    formacie obsługiwanym przez opakowany obiekt.

Czasami jest nawet możliwe stworzenie adaptera dwukierunkowego,
potrafiącego konwertować wywołania w obu kierunkach.

<figure class="image">
<img
src="../../images/patterns/diagrams/adapter/solution-pl7679.png?id=7825b3eaf05215a2bc9d17b2679c2e9a"
style="aspect-ratio: 1.51;"
srcset="/images/patterns/diagrams/adapter/solution-pl.png?id=7825b3eaf05215a2bc9d17b2679c2e9a 530w,/images/patterns/diagrams/adapter/solution-pl-1.5x.png?id=b795d4d6da012640fd112b5a0f0d1714 795w,/images/patterns/diagrams/adapter/solution-pl-2x.png?id=4cd22868a709e6ef6d6fda910ca4e473 1060w"
sizes="(max-width: 720px) 100vw, 530px" loading="lazy" width="530"
alt="Rozwiązanie Adapterem" />
</figure>

Wróćmy do naszej aplikacji giełdowej. Aby rozwiązać dylemat niezgodności
formatów, można stworzyć adaptery XML-do-JSON dla każdej klasy
biblioteki analitycznej, która jest używana bezpośrednio przez nasz kod.
Potem zaś możemy dostosować kod tak, aby komunikował się z biblioteką
wyłącznie za pomocą adapterów. Gdy adapter otrzyma wywołanie, tłumaczy
przychodzące dane XML na strukturę JSON i przekazuje wywołanie dalej, do
odpowiedniej metody opakowywanego obiektu analitycznego.
:::

::: {.section .analogy}
##  Analogia do prawdziwego życia {#analogy}

<figure class="image">
<img
src="../../images/patterns/content/adapter/adapter-comic-1-plcd3a.png?id=44233a94359b715657445798658c6add"
style="aspect-ratio: 1.78;"
srcset="/images/patterns/content/adapter/adapter-comic-1-pl.png?id=44233a94359b715657445798658c6add 533w,/images/patterns/content/adapter/adapter-comic-1-pl-1.5x.png?id=c00677934729584f29a5a6f53fc98c1d 800w,/images/patterns/content/adapter/adapter-comic-1-pl-2x.png?id=f2db728215ead6cfd68a34e1b806491b 1067w"
sizes="(max-width: 720px) 100vw, 533px" loading="lazy" width="533"
alt="Przykład wzorca projektowego Adapter" />
<figcaption><p>Walizka przed i po zagranicznej podróży.</p></figcaption>
</figure>

Gdy pierwszy raz podróżujesz z Polski do Wielkiej Brytanii, możesz
zdziwić się próbując naładować laptop. Standardy wtyczek i gniazd są
różne. Twoja wtyczka nie będzie pasować do brytyjskiego gniazdka.
Problem można rozwiązać za pomocą przejściówki posiadającej gniazdo typu
europejskiego oraz wtyk typu brytyjskiego.
:::

:::::::: {.section .structure-container}
##  Struktura {#structure}

::::::: structure
#### Adapter obiektu

Ta implementacja stosuje zasadę kompozycji obiektu: adapter implementuje
interfejs jednego z obiektów i opakowuje drugi obiekt. Można to
zaimplementować we wszystkich popularnych językach programowania.

:::: adapter-object
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/adapter/structure-object-adaptera83b.png?id=33dffbe3aece294162440c7ddd3d5d4f"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.81;"
srcset="/images/patterns/diagrams/adapter/structure-object-adapter.png?id=33dffbe3aece294162440c7ddd3d5d4f 580w,/images/patterns/diagrams/adapter/structure-object-adapter-1.5x.png?id=c1b8a87b74fc4ce5639212fe19ee06fe 870w,/images/patterns/diagrams/adapter/structure-object-adapter-2x.png?id=03e8052e168c962d6bc369bbb13b0945 1160w"
sizes="(max-width: 720px) 100vw, 580px" loading="lazy" width="580"
alt="Struktura wzorca projektowego Adapter (adapter obiektu)" /><img
src="../../images/patterns/diagrams/adapter/structure-object-adapter-indexedd6ac.png?id=a20b311948b361a058097e5bcdbf067a"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.76;"
srcset="/images/patterns/diagrams/adapter/structure-object-adapter-indexed.png?id=a20b311948b361a058097e5bcdbf067a 600w,/images/patterns/diagrams/adapter/structure-object-adapter-indexed-1.5x.png?id=72a1c8fde064c4915379fb34a1a66881 900w,/images/patterns/diagrams/adapter/structure-object-adapter-indexed-2x.png?id=759771515f08d74d53cf4fe500f814a3 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Struktura wzorca projektowego Adapter (adapter obiektu)" />
</figure>
:::
::::

1.  **Klient** jest klasą zawierającą istniejącą logikę biznesową
    programu.

2.  **Interfejs Klienta** opisuje protokół którego muszą się trzymać
    pozostałe klasy by móc współdziałać z kodem klienckim.

3.  **Usługa** to jakaś użyteczna klasa (na ogół od innego producenta
    lub starsza). Klient nie jest w stanie użyć jej bezpośrednio z
    powodu niekompatybilnego interfejsu.

4.  **Adapter** to klasa która jest w stanie współdziałać zarówno z
    klientem jak i z usługą: implementuje interfejs klienta, opakowując
    obiekt-usługę. Adapter otrzymuje wywołania od klienta za
    pośrednictwem interfejsu klienta i tłumaczy je na wywołania których
    format zrozumie obiekt udostępniający usługę.

5.  Kod kliencki nie ulegnie sprzęgnięciu z konkretną klasą adaptera, o
    ile może współpracować z adapterem za pośrednictwem interfejsu
    klienta. Dzięki temu można wprowadzać nowe typy adapterów do
    programu bez psucia istniejącego kodu klienckiego. To przydatne, gdy
    interfejs klasy udostępniającej usługę ulegnie zmianie, bo wówczas
    wystarczy dodać nową klasę adapter bez konieczności zmiany kodu
    klienckiego.

#### Adapter klasy

Ta implementacja stosuje dziedziczenie: adapter dziedziczy interfejsy od
obu obiektów jednocześnie. Ten sposób można zaimplementować tylko w
językach programowania obsługujących wielokrotne dziedziczenie --- np.
C++.

:::: adapter-class
::: {.struct-image2 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/adapter/structure-class-adapterf043.png?id=e1c60240508146ed3b98ac562cc8e510"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.72;"
srcset="/images/patterns/diagrams/adapter/structure-class-adapter.png?id=e1c60240508146ed3b98ac562cc8e510 550w,/images/patterns/diagrams/adapter/structure-class-adapter-1.5x.png?id=299a79bdfa10ac53213ba02408255430 825w,/images/patterns/diagrams/adapter/structure-class-adapter-2x.png?id=ddca3e3e4d972b7c58207daba8d24866 1100w"
sizes="(max-width: 720px) 100vw, 550px" loading="lazy" width="550"
alt="Wzorzec projektowy Adapter (adapter klasy)" /><img
src="../../images/patterns/diagrams/adapter/structure-class-adapter-indexedd30e.png?id=250b5c485a7dfba7c16b89a9201538fb"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.72;"
srcset="/images/patterns/diagrams/adapter/structure-class-adapter-indexed.png?id=250b5c485a7dfba7c16b89a9201538fb 550w,/images/patterns/diagrams/adapter/structure-class-adapter-indexed-1.5x.png?id=b56d736f8076d34b1896de0a2b22a219 825w,/images/patterns/diagrams/adapter/structure-class-adapter-indexed-2x.png?id=9ae1182ef2a34d2ea65f4b4f94a4019e 1100w"
sizes="(max-width: 720px) 100vw, 550px" loading="lazy" width="550"
alt="Wzorzec projektowy Adapter (adapter klasy)" />
</figure>
:::
::::

1.  **Adapter Klasy** nie musi opakowywać żadnych obiektów, ponieważ
    dziedziczy zachowanie zarówno po kliencie, jak i po usłudze.
    Adaptacja odbywa się wewnątrz przeciążonych metod. Otrzymany w ten
    sposób adapter może być użyty w miejsce istniejącej klasy
    klienckiej.
:::::::
::::::::

::: {.section .pseudocode}
##  Pseudokod {#pseudocode}

Ten przykład użycia **Adaptera** bazuje na klasycznym konflikcie
pomiędzy kwadratowym klockiem i okrągłym otworem.

<figure class="image">
<img
src="../../images/patterns/diagrams/adapter/examplef4c9.png?id=9d2b6857ce256f2c669383ce4df3d0aa"
style="aspect-ratio: 1.68;"
srcset="/images/patterns/diagrams/adapter/example.png?id=9d2b6857ce256f2c669383ce4df3d0aa 640w,/images/patterns/diagrams/adapter/example-1.5x.png?id=76e68b9cea3b371e465e81587e57e5d9 960w,/images/patterns/diagrams/adapter/example-2x.png?id=0ac62d1bc151e8ce6abad8e8502756cf 1280w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="640"
alt="Struktura przykładu wzorca Adapter" />
<figcaption><p>Adaptacja kwadratowych klocków do
okrągłych otworów.</p></figcaption>
</figure>

Adapter udaje, że jest okrągłym klockiem o promieniu równym połowie
przekątnej kwadratu (innymi słowy, promienia najmniejszego okręgu w
jakim zmieści się kwadratowy klocek).

<figure class="code">
<pre class="code" lang="pseudocode"><code>// Powiedzmy że masz dwie klasy o kompatybilnych interfejsach:
// RoundHole i RoundPeg.
class RoundHole is
    constructor RoundHole(radius) { ... }

    method getRadius() is
        // Zwraca promień otworu.

    method fits(peg: RoundPeg) is
        return this.getRadius() &gt;= peg.getRadius()

class RoundPeg is
    constructor RoundPeg(radius) { ... }

    method getRadius() is
        // Zwraca promień klocka (ang. peg).


// Mamy jednak niekompatybilną klasę: SquarePeg.
class SquarePeg is
    constructor SquarePeg(width) { ... }

    method getWidth() is
        // Zwróć długość boku kwadratowego klocka.

// Klasa adapter pozwala zmieścić kwadratowy klocek w okrągłym
// otworze. Rozszerza klasę RoundPeg pozwalając obiektom-
// adapterom zachowywać się jak okrągłe klocki.
class SquarePegAdapter extends RoundPeg is
    // W rzeczywistości, adapter zawiera instancję klasy
    // SquarePeg.
    private field peg: SquarePeg

    constructor SquarePegAdapter(peg: SquarePeg) is
        this.peg = peg

    method getRadius() is
        // Ten adapter udaje okrągły klocek o promieniu
        // pozwalającym zmieścić w sobie opakowywany kwadratowy
        // klocek.
        return peg.getWidth() * Math.sqrt(2) / 2


// Gdzieś w kodzie klienta.
hole = new RoundHole(5)
rpeg = new RoundPeg(5)
hole.fits(rpeg) // prawda

small_sqpeg = new SquarePeg(5)
large_sqpeg = new SquarePeg(10)
hole.fits(small_sqpeg) // to się nie skompiluje (niezgodne typy)

small_sqpeg_adapter = new SquarePegAdapter(small_sqpeg)
large_sqpeg_adapter = new SquarePegAdapter(large_sqpeg)
hole.fits(small_sqpeg_adapter) // prawda
hole.fits(large_sqpeg_adapter) // fałsz</code></pre>
</figure>
:::

:::::::: {.section .applicability-container}
##  Zastosowanie {#applicability}

::::::: applicability
::: applicability-problem
Stosuj klasę Adapter gdy chcesz wykorzystać jakąś istniejącą klasę, ale
jej interfejs nie jest kompatybilny z resztą twojego programu.
:::

::: applicability-solution
Wzorzec Adapter pozwala utworzyć klasę która stanowi warstwę
pośredniczącą pomiędzy twoim kodem, a klasą pochodzącą z zewnątrz, lub
inną, posiadającą jakiś nietypowy interfejs.
:::

::: applicability-problem
Stosuj ten wzorzec gdy chcesz wykorzystać ponownie wiele istniejących
podklas którym brakuje jakiejś wspólnej funkcjonalności, niedającej się
dodać do ich nadklasy.
:::

::: applicability-solution
Możesz rozszerzyć każdą podklasę i umieścić potrzebną funkcjonalność w
nowych klasach pochodnych. Jednak wtedy trzeba by było duplikować kod i
umieścić go we wszystkich nowych klasach, a to [psuje zapach
kodu](../smells/duplicate-code.html).

Znacznie bardziej eleganckim rozwiązaniem jest umieszczenie brakującej
funkcjonalności w klasie adapter i opakowanie nią obiektów pozbawionych
potrzebnych funkcji. Aby to zadziałało, klasy docelowe muszą mieć
wspólny interfejs, a pole adaptera musi być z nim zgodne. Podejście to
bardzo przypomina wzorzec [Dekorator](decorator.html).
:::
:::::::
::::::::

::: {.section .checklist}
##  Jak zaimplementować {#checklist}

1.  Upewnij się, że masz przynajmniej dwie klasy o niekompatybilnych
    interfejsach:

    -   Jakąś użyteczną klasę *usługową* której nie możesz zmienić
        (innego producenta, przestarzałą, albo ze zbyt wieloma
        istniejącymi zależnościami)
    -   Jedną lub wiele klas *klienckich* które zyskałyby na możliwości
        skorzystania z powyższej usługi.

2.  Zadeklaruj interfejs klienta i opisz jak ma wyglądać komunikacja
    klientów z usługą.

3.  Stwórz klasę adapter zgodną z interfejsem klienckim. Póki co,
    pozostaw metody pustymi.

4.  Dodaj pole do klasy adapter, które przechowa odniesienie do obiektu
    usługi. Typową praktyką jest inicjalizacja tego pola za
    pośrednictwem konstruktora, ale czasem wygodniej jest przekazać
    usługę adapterowi wywołując jego metody.

5.  Jeden po drugim, zaimplementuj wszystkie metody interfejsu klienta w
    klasie adapter. Adapter powinien delegować większość faktycznej
    pracy obiektowi oferującemu usługę i zajmować się wyłącznie
    pośrednictwem lub konwersją danych.

6.  Klienci powinni stosować adapter za pośrednictwem interfejsu
    klienta. Pozwoli to zmieniać lub rozszerzać adaptery bez
    wpływania na kodu kliencki.
:::

:::::: {.section .pros-cons}
##  Zalety i wady {#pros-cons}

::::: row
::: col-sm-6
-    *Zasada pojedynczej odpowiedzialności*. Można oddzielić interfejs
    lub kod konwertujący dane od głównej logiki biznesowej programu.
-    *Zasada otwarte/zamknięte*. Można wprowadzać do programu nowe typy
    adapterów bez psucia istniejącego kodu klienckiego, o ile będzie on
    korzystał z adapterów poprzez interfejs kliencki.
:::

::: col-sm-6
-    Ogólna złożoność kodu zwiększa się, ponieważ trzeba wprowadzić
    zestaw nowych interfejsów i klas. Czasem łatwiej zmienić klasę
    udostępniającą jakąś potrzebną usługę, aby pasowała do reszty kodu.
:::
:::::
::::::

::: {.section .relations}
##  Powiązania z innymi wzorcami {#relations}

-   [Most](bridge.html) zazwyczaj wykorzystuje się od początku
    projektu, by pozwolić na niezależną pracę nad poszczególnymi
    częściami aplikacji. Z drugiej strony, [Adapter](adapter.html) jest
    rozwiązaniem stosowanym w istniejącej aplikacji w celu umożliwienia
    współpracy pomiędzy niekompatybilnymi klasami.

-   [Adapter](adapter.html) zapewnia zupełnie inny interfejs dostępu do
    istniejącego obiektu. Z drugiej strony, w przypadku wzorca
    [Dekorator](decorator.html) interfejs albo pozostaje taki sam, albo
    zostaje rozszerzony. Ponadto, *Decorator* obsługuje rekurencyjną
    kompozycję, co nie jest możliwe w przypadku użycia *Adapter*.

-   Za pomocą [Adapter](adapter.html) można uzyskać dostęp do
    istniejącego obiektu za pośrednictwem innego interfejsu. W przypadku
    [Pełnomocnik](proxy.html) interfejs pozostaje taki sam. Za pomocą
    [Dekorator](decorator.html) uzyskuje się dostęp do obiektu za
    pośrednictwem ulepszonego interfejsu.

-   [Fasada](facade.html) definiuje nowy interfejs istniejącym obiektom,
    zaś [Adapter](adapter.html) zakłada zwiększenie użyteczności
    zastanego interfejsu. *Adapter* na ogół opakowuje pojedynczy obiekt,
    zaś *Fasada* obejmuje cały podsystem obiektów.

-   [Most](bridge.html), [Stan](state.html), [Strategia](strategy.html)
    (i w pewnym stopniu [Adapter](adapter.html)) mają podobną strukturę.
    Wszystkie oparte są na kompozycji, co oznacza delegowanie zadań
    innym obiektom. Jednak każdy z tych wzorców rozwiązuje inne
    problemy. Wzorzec nie jest bowiem tylko receptą na ustrukturyzowanie
    kodu w pewien sposób, lecz także informacją dla innych deweloperów o
    charakterze rozwiązywanego problemu.
:::

::: {.section .implementations}
##  Przykłady kodu {#implementations}

[![Adapter w języku
C#](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](adapter/csharp/example.html "Adapter w języku C#"){.prog-lang-link}
[![Adapter w języku
C++](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](adapter/cpp/example.html "Adapter w języku C++"){.prog-lang-link}
[![Adapter w języku
Go](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](adapter/go/example.html "Adapter w języku Go"){.prog-lang-link}
[![Adapter w języku
Java](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](adapter/java/example.html "Adapter w języku Java"){.prog-lang-link}
[![Adapter w języku
PHP](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](adapter/php/example.html "Adapter w języku PHP"){.prog-lang-link}
[![Adapter w języku
Python](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](adapter/python/example.html "Adapter w języku Python"){.prog-lang-link}
[![Adapter w języku
Ruby](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](adapter/ruby/example.html "Adapter w języku Ruby"){.prog-lang-link}
[![Adapter w języku
Rust](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](adapter/rust/example.html "Adapter w języku Rust"){.prog-lang-link}
[![Adapter w języku
Swift](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](adapter/swift/example.html "Adapter w języku Swift"){.prog-lang-link}
[![Adapter w języku
TypeScript](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](adapter/typescript/example.html "Adapter w języku TypeScript"){.prog-lang-link}
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

[Most []{.fa .fa-arrow-right}](bridge.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Wróć

[[]{.fa .fa-arrow-left} Wzorce
strukturalne](structural-patterns.html){.btn .btn-default rel="prev"}
:::
:::::::::::::::::::::::::::::::::::

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
:::::::::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::::::::
