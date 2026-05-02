::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [Wzorce
projektowe](../design-patterns.html){.type} / [Wzorce
strukturalne](structural-patterns.html){.type}
:::

# Dekorator {#dekorator .title}

::: aka
Znany też jako:
[Nakładka, ]{style="display:inline-block"}[Wrapper, ]{style="display:inline-block"}[Decorator]{style="display:inline-block"}
:::

::: {.section .intent}
##  Cel {#intent}

**Dekorator** to strukturalny wzorzec projektowy pozwalający dodawać
nowe obowiązki obiektom poprzez umieszczanie tych obiektów w specjalnych
obiektach opakowujących, które zawierają odpowiednie zachowania.

<figure class="image">
<img
src="../../images/patterns/content/decorator/decoratore8f5.png?id=710c66670c7123e0928d3b3758aea79e"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/decorator/decorator.png?id=710c66670c7123e0928d3b3758aea79e 640w,/images/patterns/content/decorator/decorator-1.5x.png?id=72aafc603a289fc64e028e83e8aa843b 960w,/images/patterns/content/decorator/decorator-2x.png?id=736ab07b1d8920ab2c7a70c9cb1305cc 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Wzorzec projektowy Dekorator" />
</figure>
:::

::: {.section .problem}
##  Problem

Wyobraź sobie, że pracujesz nad biblioteką powiadamiającą, która pozwala
innym programom informować użytkowników o istotnych zdarzeniach.

Wstępna wersja biblioteki bazowała na klasie `Powiadamiacz`, która miała
tylko parę pól, konstruktor oraz jedną metodę --- `Wyślij`. Metoda
przyjmowała wiadomość jako argument od klienta i przesyłała tę
wiadomość do listy emailów, które z kolei przekazywano powiadamiaczowi
przez jego konstruktor. Aplikacja innego producenta, pełniąca rolę
klienta, miała za zadanie stworzyć i skonfigurować obiekt powiadamiacza
jednorazowo, potem zaś korzystać z niego w razie istotnych zdarzeń.

<figure class="image">
<img
src="../../images/patterns/diagrams/decorator/problem1-pla7d3.png?id=53c931db8aee7ff6951a5a6a2fa35213"
style="aspect-ratio: 2.45;"
srcset="/images/patterns/diagrams/decorator/problem1-pl.png?id=53c931db8aee7ff6951a5a6a2fa35213 540w,/images/patterns/diagrams/decorator/problem1-pl-1.5x.png?id=86febd669cf2885cad1647b70b9c6015 810w,/images/patterns/diagrams/decorator/problem1-pl-2x.png?id=1fc60b3c12c8ee4ee03c99ca82003d4e 1080w"
sizes="(max-width: 720px) 100vw, 540px" loading="lazy" width="540"
alt="Struktura biblioteki przed zastosowaniem wzorca Dekorator" />
<figcaption><p>Program może korzystać z klasy powiadamiającej by wysyłać
powiadomienia o ważnych wydarzeniach na określony zestaw
adresów email.</p></figcaption>
</figure>

W jakimś momencie zauważasz, że użytkownicy biblioteki oczekują więcej,
niż tylko przysłania maila. Wielu chce otrzymywać SMS gdy zdarzy się coś
bardzo ważnego. Inni z kolei chcą być powiadamiani poprzez Facebooka, a
użytkownicy korporacyjni byliby zachwyceni powiadomieniami Slack.

<figure class="image">
<img
src="../../images/patterns/diagrams/decorator/problem2a2f8.png?id=ba5d5e106ea8c4848d60e230feca9135"
style="aspect-ratio: 2.59;"
srcset="/images/patterns/diagrams/decorator/problem2.png?id=ba5d5e106ea8c4848d60e230feca9135 440w,/images/patterns/diagrams/decorator/problem2-1.5x.png?id=4f15b6208533188dd09e7da7dd6b509a 660w,/images/patterns/diagrams/decorator/problem2-2x.png?id=28b2c8509b4e78c031d728424b876ebc 880w"
sizes="(max-width: 720px) 100vw, 440px" loading="lazy" width="440"
alt="Struktura biblioteki po zaimplementowaniu dodatkowych rodzajów powiadamiania." />
<figcaption><p>Każdy rodzaj powiadomienia zaimplementowano w osobnej
podklasie powiadamiacza.</p></figcaption>
</figure>

Czy to takie trudne? Wystarczy rozszerzyć klasę `Powiadamiacz` i
umieścić dodatkowe metody powiadamiania w nowych podklasach. Teraz
klient powinien stworzyć instancję potrzebnej mu klasy powiadomień i
może korzystać do woli.

Wszystko w porządku do momentu, aż ktoś zada całkiem sensowne pytanie:
"Czemu nie da się przesłać powiadomienia wieloma drogami naraz? Przecież
jeśli w twoim domu wybuchnie pożar, warto zastosować wszelkie możliwe
opcje powiadamiania".

Próbujesz więc spełnić to wymaganie tworząc specjalne podklasy łączące
różne metody powiadamiania w jednej klasie. Jednakże, szybko okazuje się
oczywiste, że wskutek tego podejścia kod strasznie spuchł, i to nie
tylko po stronie biblioteki, ale i klienta.

<figure class="image">
<img
src="../../images/patterns/diagrams/decorator/problem34033.png?id=f3b3e7a107d870871f2c3167adcb7ccb"
style="aspect-ratio: 1.85;"
srcset="/images/patterns/diagrams/decorator/problem3.png?id=f3b3e7a107d870871f2c3167adcb7ccb 630w,/images/patterns/diagrams/decorator/problem3-1.5x.png?id=f4ef9d367df838c77121e9f84260b09c 945w,/images/patterns/diagrams/decorator/problem3-2x.png?id=abb7a87b521ce97d7661dd9c0b988cc3 1260w"
sizes="(max-width: 720px) 100vw, 630px" loading="lazy" width="630"
alt="Struktura biblioteki po stworzeniu klas kombinacji powiadomień" />
<figcaption><p>Kombinatoryczna eksplozja podklas.</p></figcaption>
</figure>

Trzeba znaleźć jakiś inny pomysł na strukturę klas powiadomień, aby
uniknąć wpisania do księgi rekordów Guinessa przy dodawaniu kolejnych
możliwości powiadamiania.
:::

::: {.section .solution}
##  Rozwiązanie {#solution}

Rozszerzenie klasy jest pierwszym sposobem jaki przychodzi do głowy, gdy
stajemy wobec konieczności zmiany zachowania się obiektu. Jednakże
dziedziczenie wiąże się z wieloma obciążeniami, o których trzeba
pamiętać.

-   Dziedziczenie jest statyczne. Nie da się zmienić zachowania
    istniejącego obiektu po uruchomieniu programu. Można tylko zastąpić
    cały obiekt innym, stworzonym z innej podklasy.
-   Podklasy mogą mieć tylko jedną klasę-rodzica. W większości języków
    nie można odziedziczyć zachowania wielu klas jednocześnie.

Jednym ze sposobów uniknięcia tych ograniczeń jest zastosowanie
*Agregacji* lub *Kompozycji* []{.pop-i}[*Agregacja*: obiekt A zawiera
obiekty B; B może istnieć bez A.\
*Kompozycja*: obiekt A składa się z obiektów B; A zarządza cyklem życia
B; B nie może istnieć samodzielnie.]{.pop-content} zamiast
*Dziedziczenia*. Obie alternatywy działają prawie tak samo: jeden z
obiektów *posiada* odniesienie do innego i deleguje mu jakąś pracę,
zaś w przypadku dziedziczenia, obiekt *sam jest* w stanie wykonać tę
pracę, dziedzicząc zachowanie od swej nadklasy.

Dzięki temu nowemu sposobowi można łatwo zamienić obiekt, z którym
istnieje połączenie, na inny, tym samym zmieniając zachowanie
kontenera w czasie działania programu. Obiekt może korzystać z zachowań
różnych klas, mając odniesienia do wielu obiektów i delegując im różne
rodzaje zadań. Agregacja/kompozycja jest kluczową koncepcją wielu
wzorców projektowych. Nie inaczej jest z Dekoratorem. Wróćmy więc do
omówienia wzorca.

<figure class="image">
<img
src="../../images/patterns/diagrams/decorator/solution1-pld865.png?id=a11e6812e09997b14dffb41860e5db06"
style="aspect-ratio: 3.44;"
srcset="/images/patterns/diagrams/decorator/solution1-pl.png?id=a11e6812e09997b14dffb41860e5db06 550w,/images/patterns/diagrams/decorator/solution1-pl-1.5x.png?id=d4ee365475bcb8c5afd0b364f975abb3 825w,/images/patterns/diagrams/decorator/solution1-pl-2x.png?id=3dd91a20fb47defab4e2dbc86ab5a594 1100w"
sizes="(max-width: 720px) 100vw, 550px" loading="lazy" width="550"
alt="Dziedziczenie a agregacja" />
<figcaption><p>Dziedziczenie a agregacja</p></figcaption>
</figure>

Dekorator znany jest też pod nazwą "Nakładka". To słowo dobrze wyraża
główną ideę tego wzorca. *Nakładka* jest obiektem który może być
połączony z jakimś *docelowym* obiektem. Nakładka zawiera ten sam zestaw
metod jak obiekt docelowy i deleguje mu wszelkie otrzymywane żądania.
Jednak nakładka może wpłynąć na rezultat, wykonując coś albo przed,
albo po przekazaniu żądania.

Kiedy więc prosta nakładka staje się prawdziwym dekoratorem? Jak
wspomniałem, nakładka implementuje ten sam interfejs co "opakowywany"
obiekt. Dlatego też z punktu widzenia klienta te obiekty są identyczne.
Niech pole referencyjne nakładki przyjmuje każdy obiekt zgodny z tym
interfejsem, pozwoli to wówczas "przykryć" obiekt wieloma warstwami,
sumując tym samym zachowania każdej z nich.

W naszym przykładzie z powiadomieniami, zostawmy proste powiadomienie
mailowe w klasie bazowej `Powiadamiacz`, ale zmieńmy inne metody
powiadamiania w dekoratory.

<figure class="image">
<img
src="../../images/patterns/diagrams/decorator/solution2e4ab.png?id=cbee4a27080ce3a0bf773482613e1347"
style="aspect-ratio: 1.39;"
srcset="/images/patterns/diagrams/decorator/solution2.png?id=cbee4a27080ce3a0bf773482613e1347 640w,/images/patterns/diagrams/decorator/solution2-1.5x.png?id=d2fa0c0b9bf5cba0e38be7aff7e7b7a8 960w,/images/patterns/diagrams/decorator/solution2-2x.png?id=7775f76b94dbd5cd25f711ce81f59262 1280w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="640"
alt="Rozwiązanie za pomocą wzorca Dekorator" />
<figcaption><p>Różne metody powiadamiania stały
się dekoratorami.</p></figcaption>
</figure>

Kod kliencki musiałby opakować podstawowy obiekt powiadamiacza w zestaw
dekoratorów stosowny do preferencji klienta. Wynikowy obiekt będzie miał
strukturę stosu.

<figure class="image">
<img
src="../../images/patterns/diagrams/decorator/solution3-pl8f26.png?id=ade45b11435a0e3b5d618546e424ee71"
style="aspect-ratio: 1.03;"
srcset="/images/patterns/diagrams/decorator/solution3-pl.png?id=ade45b11435a0e3b5d618546e424ee71 300w,/images/patterns/diagrams/decorator/solution3-pl-1.5x.png?id=ed169cfa6a11c739c2799bf6420c2928 450w,/images/patterns/diagrams/decorator/solution3-pl-2x.png?id=58f8d405982653264330d043a1449658 600w"
sizes="(max-width: 720px) 100vw, 300px" loading="lazy" width="300"
alt="Aplikacje mogą tworzyć złożone konfiguracje stosów dekoratorów powiadamiaczy" />
<figcaption><p>Aplikacje mogą tworzyć złożone konfiguracje stosów
dekoratorów powiadamiaczy.</p></figcaption>
</figure>

Ostatnim dekoratorem w stosie będzie obiekt, z którym klient faktycznie
pracuje. Skoro wszystkie dekoratory implementują ten sam interfejs co
powiadamiacz bazowy, reszta kodu klienta nie będzie musiała wiedzieć,
czy pracuje na "czystym" obiekcie powiadamiacza, czy "udekorowanym".

Moglibyśmy zastosować to samo podejście wobec innych obowiązków, jak
formatowanie wiadomości lub komponowanie listy odbiorców. Klient może
udekorować obiekt dowolnymi dekoratorami, o ile będą one zgodne ze
sobą co do interfejsu.
:::

::: {.section .analogy}
##  Analogia do prawdziwego życia {#analogy}

<figure class="image">
<img
src="../../images/patterns/content/decorator/decorator-comic-1cbff.png?id=80d95baacbfb91f5bcdbdc7814b0c64d"
style="aspect-ratio: 2;"
srcset="/images/patterns/content/decorator/decorator-comic-1.png?id=80d95baacbfb91f5bcdbdc7814b0c64d 600w,/images/patterns/content/decorator/decorator-comic-1-1.5x.png?id=490ef9754d7554c2046b69f6f213c6da 900w,/images/patterns/content/decorator/decorator-comic-1-2x.png?id=ba869f621b6e0ea173fdc2b535fc7eed 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Przykład wzorca Dekorator" />
<figcaption><p>Zyskujesz połączony efekt poszczególnych
elementów ubioru.</p></figcaption>
</figure>

Noszenie ubrań jest przykładem stosowania dekoratorów. Gdy ci zimno,
zakładasz sweter. Jeśli dalej ci zimno, zakładasz jeszcze kurtkę. A
jeśli do tego pada deszcz, możesz założyć płaszcz przeciwdeszczowy.
Wszystkie te elementy ubioru "rozszerzają" twoje domyślne zachowanie,
ale nie są częścią ciebie i możesz pozbyć się każdego z nich gdy nie
jest ci akurat potrzebny.
:::

::::: {.section .structure-container}
##  Struktura {#structure}

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/decorator/structure128f.png?id=8c95d894aecce5315cc1b12093a7ea0c"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 0.92;"
srcset="/images/patterns/diagrams/decorator/structure.png?id=8c95d894aecce5315cc1b12093a7ea0c 480w,/images/patterns/diagrams/decorator/structure-1.5x.png?id=4b2cd91d4483d9e3bba725f0e45d281d 720w,/images/patterns/diagrams/decorator/structure-2x.png?id=3cfa1f10417a4ef0c12580bc4a63b80d 960w"
sizes="(max-width: 720px) 100vw, 480px" loading="lazy" width="480"
alt="Struktura wzorca projektowego Dekorator" /><img
src="../../images/patterns/diagrams/decorator/structure-indexeda93c.png?id=09401b230a58f2249e4c9a1195d485a0"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 0.98;"
srcset="/images/patterns/diagrams/decorator/structure-indexed.png?id=09401b230a58f2249e4c9a1195d485a0 500w,/images/patterns/diagrams/decorator/structure-indexed-1.5x.png?id=dccc54182965078ccb4cfdeee41acbe5 750w,/images/patterns/diagrams/decorator/structure-indexed-2x.png?id=2733e7d0e322bfb2f150ccf8a878dbf6 1000w"
sizes="(max-width: 720px) 100vw, 500px" loading="lazy" width="500"
alt="Struktura wzorca projektowego Dekorator" />
</figure>
:::

1.  **Komponent** deklaruje interfejs wspólny zarówno dla nakładek,
    jak i opakowywanych obiektów.

2.  **Konkretny Komponent** to klasa opakowywanych obiektów. Definiuje
    ona podstawowe zachowanie, które następnie można zmieniać za pomocą
    dekoratorów.

3.  Klasa **Bazowy Dekorator** posiada pole przeznaczone na
    referencję do opakowywanego obiektu. Typ pola powinien być
    zadeklarowany jako interfejs komponentu, aby mogło przechować
    zarówno konkretne komponenty, jak i inne dekoratory. Dekorator
    bazowy deleguje wszystkie działania opakowywanemu obiektowi.

4.  **Konkretni Dekoratorzy** definiują dodatkowe zachowania które można
    przypisać do komponentów dynamicznie. Konkretni dekoratorzy
    nadpisują metody dekoratora bazowego i wykonują swoje działania albo
    przed, albo po wywołaniu metody klasy-rodzica.

5.  **Klient** może opakowywać komponenty w wiele warstw dekoratorów, o
    ile działa na wszystkich obiektach poprzez interfejs komponentu.
::::
:::::

::: {.section .pseudocode}
##  Pseudokod {#pseudocode}

W tym przykładzie, wzorzec **Dekorator** pozwala skompresować i
zaszyfrować wrażliwe dane niezależnie od kodu który faktycznie
korzysta z tych danych.

<figure class="image">
<img
src="../../images/patterns/diagrams/decorator/example42cb.png?id=eec9dc488f00c85f50e764323baa723e"
style="aspect-ratio: 0.98;"
srcset="/images/patterns/diagrams/decorator/example.png?id=eec9dc488f00c85f50e764323baa723e 540w,/images/patterns/diagrams/decorator/example-1.5x.png?id=40ccaba4f5a8e6a2ebac697f04b9f10a 810w,/images/patterns/diagrams/decorator/example-2x.png?id=4891323a27d5601a174eec366187d833 1080w"
sizes="(max-width: 720px) 100vw, 540px" loading="lazy" width="540"
alt="Struktura przykładu wzorca Dekorator" />
<figcaption><p>Przykład dekoratorów kompresujących
i szyfrujących.</p></figcaption>
</figure>

Aplikacja opakowuje źródło danych w parę dekoratorów. Obie nakładki
zmieniają sposób, w jaki dane są zapisywane na i odczytywane z dysku:

-   Tuż przed **zapisaniem na dysk** danych, dekoratory szyfrują i
    kompresują je. Pierwotna klasa zapisuje do pliku dane już
    zaszyfrowane i skompresowane --- bez wiedzy o dokonanej obróbce.

-   Tuż po **odczytaniu z dysku** danych, przechodzą one przez te same
    dekoratory, które je dekompresują i deszyfrują.

Dekoratory i klasa źródła danych implementują ten sam interfejs, co
czyni je wymienialnymi w kodzie klienta.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// Interfejs komponentu definiuje działania które można
// modyfikować za pomocą dekoratorów.
interface DataSource is
    method writeData(data)
    method readData():data

// Konkretne komponenty dostarczają domyślnych implementacji
// działań. Może istnieć wiele odmian tych klas w całym
// programie.
class FileDataSource implements DataSource is
    constructor FileDataSource(filename) { ... }

    method writeData(data) is
        // Zapisz dane do pliku.

    method readData():data is
        // Wczytaj dane z pliku.

// Bazowa klasa dekorator ma taki sam interfejs jak inne
// komponenty. Głównym celem tej klasy jest zdefiniowanie
// interfejsu, który będzie opakowywał wszystkie konkretne
// dekoratory. Domyślna implementacja kodu opakowującego może
// zawierać pole służące przechowywaniu opakowywanego komponentu
// oraz narzędzia służące jego inicjalizacji.
class DataSourceDecorator implements DataSource is
    protected field wrappee: DataSource

    constructor DataSourceDecorator(source: DataSource) is
        wrappee = source

    // Dekorator bazowy po prostu deleguje całą pracę
    // opakowywanemu komponentowi. Dodatkową funkcjonalność
    // można dodać w formie kolejnych konkretnych dekoratorów.
    method writeData(data) is
        wrappee.writeData(data)

    // Konkretni dekoratorzy mogą wywoływać implementację
    // działania z nadklasy zamiast bezpośrednio z opakowywanego
    // obiektu. To podejście upraszcza rozszerzanie klas
    // dekoratorów.
    method readData():data is
        return wrappee.readData()

// Konkretne dekoratory wywołują metody opakowywanego obiektu,
// ale mogą wzbogacać ich funkcjonalność. Dekoratory mogą
// wykonywać swoje działania albo przed, albo po wywołaniu
// metody opakowywanego obiektu.
class EncryptionDecorator extends DataSourceDecorator is
    method writeData(data) is
        // 1. Zaszyfruj przekazane dane.
        // 2. Przekaż zaszyfrowane dane metodzie writeData
        // obiektu opakowywanego.

    method readData():data is
        // 1. Pobierz dane od metody readData obiektu
        // opakowywanego.
        // 2. Spróbuj odszyfrować dane jeśli są zaszyfrowane.
        // 3. Zwróć wynik.

// Można opakowywać obiekty wieloma warstwami dekoratorów.
class CompressionDecorator extends DataSourceDecorator is
    method writeData(data) is
        // 1. Skompresuj przekazane dane.
        // 2. Przekaż skompresowane dane metodzie writeData
        // opakowywanego obiektu.

    method readData():data is
        // 1. Pobierz dane od metody readData opakowywanego
        // obiektu.
        // 2. Spróbuj je zdekompresować jeśli są skompresowane.
        // 3. Zwróć wynik.


// Opcja 1. Prosty przykład zestawu dekoratorów.
class Application is
    method dumbUsageExample() is
        source = new FileDataSource(&quot;somefile.dat&quot;)
        source.writeData(salaryRecords)
        // Docelowy plik wypełniono danymi w formie otwartego
        // tekstu.

        source = new CompressionDecorator(source)
        source.writeData(salaryRecords)
        // Plik docelowy wypełniono skompresowanymi danymi.

        source = new EncryptionDecorator(source)
        // Zmienna source zawiera teraz:
        // Szyfrowanie &gt; Kompresja &gt; FileDataSource
        source.writeData(salaryRecords)
        // Plik zapisano zaszyfrowanymi i skompresowanymi
        // danymi.


// Opcja 2. Kod kliencki korzystający z zewnętrznego źródła
// danych. Obiekty SalaryManager nie znają szczegółów
// magazynowania danych. Pracują z już skonfigurowanym źródłem
// danych które otrzymały od konfiguratora aplikacji.
class SalaryManager is
    field source: DataSource

    constructor SalaryManager(source: DataSource) { ... }

    method load() is
        return source.readData()

    method save() is
        source.writeData(salaryRecords)
    // ...Inne przydatne metody...


// W aplikacji można złożyć różne stosy dekoratorów w trakcie
// działania programu — zależnie od konfiguracji lub środowiska
// uruchomieniowego.
class ApplicationConfigurator is
    method configurationExample() is
        source = new FileDataSource(&quot;salary.dat&quot;)
        if (enabledEncryption)
            source = new EncryptionDecorator(source)
        if (enabledCompression)
            source = new CompressionDecorator(source)

        logger = new SalaryManager(source)
        salary = logger.load()
    // ...</code></pre>
</figure>
:::

:::::::: {.section .applicability-container}
##  Zastosowanie {#applicability}

::::::: applicability
::: applicability-problem
Stosuj wzorzec Dekorator gdy chcesz przypisywać dodatkowe obowiązki
obiektom w trakcie działania programu, bez psucia kodu, który z tych
obiektów korzysta.
:::

::: applicability-solution
Dekorator pozwala ustrukturyzować logikę biznesową w formie warstw,
tworząc dekorator dla każdej warstwy i składać obiekty z różnymi
kombinacjami tej logiki w czasie działania programu. Kod klienta może
traktować wszystkie obiekty w taki sam sposób, ponieważ wszystkie są
zgodne pod względem wspólnego interfejsu.
:::

::: applicability-problem
Stosuj ten wzorzec gdy rozszerzenie zakresu obowiązków obiektu za pomocą
dziedziczenia byłoby niepraktyczne, lub niemożliwe.
:::

::: applicability-solution
Wiele języków programowania posiada słowo kluczowe `final`, za pomocą
którego uniemożliwia się dalsze rozszerzanie klasy. W przypadku klasy
finalnej, jedynym sposobem na ponowne wykorzystanie istniejącego
zachowania jest opakowanie jej nakładkami swojego autorstwa ---
zgodnie ze wzorcem Dekorator.
:::
:::::::
::::::::

::: {.section .checklist}
##  Jak zaimplementować {#checklist}

1.  Upewnij się, że twoja domena biznesowa może zostać przedstawiona w
    formie podstawowego komponentu z nałożonymi nań wieloma opcjonalnymi
    warstwami.

2.  Ustal jakie metody są wspólne zarówno dla podstawowego komponentu,
    jak i warstw opcjonalnych. Stwórz interfejs komponentu i zadeklaruj
    tam owe wspólne metody.

3.  Stwórz klasę konkretnego komponentu i zdefiniuj w niej podstawowe
    zachowanie.

4.  Stwórz bazową klasę dekoratora. Powinna ona zawierać pole do
    przechowywania odniesienia do opakowywanego obiektu. Pole takie
    powinno być zadeklarowane jako typ interfejsu komponenta, aby
    umożliwić wiązanie z konkretnymi komponentami oraz dekoratorami.
    Dekorator bazowy musi delegować pracę obiektowi opakowywanemu.

5.  Upewnij się, że wszystkie klasy implementują interfejs komponentu.

6.  Stwórz konkretne dekoratory poprzez rozszerzanie dekoratora
    bazowego. Konkretny dekorator musi wykonywać swoje zadania przed
    lub po wywołaniu metody rodzica (który zawsze deleguje opakowywanemu
    obiektowi).

7.  Kod kliencki musi być odpowiedzialny za tworzenie dekoratorów oraz
    składanie ich wedle swoich potrzeb.
:::

:::::: {.section .pros-cons}
##  Zalety i wady {#pros-cons}

::::: row
::: col-sm-6
-    Można rozszerzać zachowanie obiektu bez tworzenia podklasy.
-    Można dodawać lub usuwać obowiązki obiektu w trakcie działania
    programu.
-    Możliwe jest łączenie wielu zachowań poprzez nałożenie wielu
    dekoratorów na obiekt.
-    *Zasada pojedynczej odpowiedzialności*. Można podzielić klasę
    monolityczną, która implementuje wiele wariantów zachowań, na
    mniejsze klasy.
:::

::: col-sm-6
-    Zabranie jednej konkretnej nakładki ze środka stosu nakładek jest
    trudne.
-    Trudno jest zaimplementować dekorator w taki sposób, aby jego
    zachowanie nie zależało od kolejności ułożenia nakładek na stosie.
-    Kod wstępnie konfigurujący warstwy może wyglądać brzydko.
:::
:::::
::::::

::: {.section .relations}
##  Powiązania z innymi wzorcami {#relations}

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

-   [Łańcuch zobowiązań](chain-of-responsibility.html) i
    [Dekorator](decorator.html) mają bardzo podobne struktury klas. Oba
    wzorce bazują na rekursywnej kompozycji w celu przekazania obowiązku
    wykonania przez ciąg obiektów. Istnieją jednak kluczowe różnice.

    Obsługujący *Łańcucha zobowiązań* mogą wykonywać działania
    niezależnie od siebie. Mogą również zatrzymać dalsze przekazywanie
    żądania na dowolnym etapie. Z drugiej strony, różne *Dekoratory*
    mogą rozszerzać obowiązki obiektu zachowując zgodność z interfejsem
    bazowym. Dodatkowo, dekoratory nie mają możliwości przerwania
    przepływu żądania.

-   [Kompozyt](composite.html) i [Dekorator](decorator.html) mają
    podobne diagramy struktur ponieważ oba bazują na rekursywnej
    kompozycji w celu zorganizowania nieokreślonej liczby obiektów.

    *Dekorator* przypomina *Kompozyt*, ale posiada tylko jeden element
    podrzędny. Ponadto, kolejną różnicą jest to, że *Dekorator*
    przypisuje dodatkowe obowiązki opakowywanemu obiektowi, zaś
    *Kompozyt* jedynie "sumuje" wyniki otrzymane od elementów
    podrzędnych.

    Wzorce mogą też współpracować: *Dekorator* może służyć rozszerzeniu
    zachowania określonego obiektu w drzewie *Kompozytowym*

-   Projekty intensywnie korzystające ze wzorców
    [Kompozyt](composite.html) i [Dekorator](decorator.html) mogą
    skorzystać również na zastosowaniu [Prototypu](prototype.html).
    Zastosowanie tego wzorca pozwala klonować złożone struktury zamiast
    konstruować je ponownie od zera.

-   [Dekorator](decorator.html) pozwala zmienić otoczkę obiektu, zaś
    [Strategia](strategy.html) jej wnętrze.

-   [Dekorator](decorator.html) i [Pełnomocnik](proxy.html) mają podobne
    struktury, ale inne cele. Oba wzorce bazują na zasadzie
    kompozycji --- jeden obiekt deleguje część zadań innemu.
    *Pełnomocnik* dodatkowo zarządza cyklem życia obiektu
    udostępniającego jakąś usługę, zaś komponowanie *Dekoratorów* leży w
    gestii klienta.
:::

::: {.section .implementations}
##  Przykłady kodu {#implementations}

[![Dekorator w języku
C#](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](decorator/csharp/example.html "Dekorator w języku C#"){.prog-lang-link}
[![Dekorator w języku
C++](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](decorator/cpp/example.html "Dekorator w języku C++"){.prog-lang-link}
[![Dekorator w języku
Go](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](decorator/go/example.html "Dekorator w języku Go"){.prog-lang-link}
[![Dekorator w języku
Java](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](decorator/java/example.html "Dekorator w języku Java"){.prog-lang-link}
[![Dekorator w języku
PHP](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](decorator/php/example.html "Dekorator w języku PHP"){.prog-lang-link}
[![Dekorator w języku
Python](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](decorator/python/example.html "Dekorator w języku Python"){.prog-lang-link}
[![Dekorator w języku
Ruby](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](decorator/ruby/example.html "Dekorator w języku Ruby"){.prog-lang-link}
[![Dekorator w języku
Rust](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](decorator/rust/example.html "Dekorator w języku Rust"){.prog-lang-link}
[![Dekorator w języku
Swift](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](decorator/swift/example.html "Dekorator w języku Swift"){.prog-lang-link}
[![Dekorator w języku
TypeScript](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](decorator/typescript/example.html "Dekorator w języku TypeScript"){.prog-lang-link}
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

[Fasada []{.fa .fa-arrow-right}](facade.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Wróć

[[]{.fa .fa-arrow-left} Kompozyt](composite.html){.btn .btn-default
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
