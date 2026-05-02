::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [Wzorce
projektowe](../design-patterns.html){.type} / [Wzorce
behawioralne](behavioral-patterns.html){.type}
:::

# Metoda szablonowa {#metoda-szablonowa .title}

::: aka
Znany też jako: [Template Method]{style="display:inline-block"}
:::

::: {.section .intent}
##  Cel {#intent}

**Metoda szablonowa** to behawioralny wzorzec projektowy definiujący
szkielet algorytmu w klasie bazowej, ale pozwalający podklasom nadpisać
pewne etapy tego algorytmu bez konieczności zmiany jego struktury.

<figure class="image">
<img
src="../../images/patterns/content/template-method/template-method4c1b.png?id=eee9461742f832814f19612ccf472819"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/template-method/template-method.png?id=eee9461742f832814f19612ccf472819 640w,/images/patterns/content/template-method/template-method-1.5x.png?id=2b4c179aaa02f5c45a59324b904cd130 960w,/images/patterns/content/template-method/template-method-2x.png?id=4e164dc41be4dcfa122628864c2be210 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Wzorzec projektowy Metoda szablonowa" />
</figure>
:::

::: {.section .problem}
##  Problem

Wyobraź sobie, że tworzysz aplikację zbierającą dane, która analizuje
dokumenty w korporacji. Użytkownicy przesyłają do niej dokumenty w
różnych formatach (PDF, DOC, CSV), a ta próbuje wydobyć z nich istotne
dane i przedstawić je w jednym formacie.

Pierwsza wersja aplikacji mogła współpracować tylko z plikami DOC,
kolejna wersja dodała obsługę plików CSV. Miesiąc później aplikacja
"umiała" już ekstrahować dane z PDF.

<figure class="image">
<img
src="../../images/patterns/diagrams/template-method/problem6793.png?id=60fa4f735c467ac1c9438231a1782807"
style="aspect-ratio: 1.35;"
srcset="/images/patterns/diagrams/template-method/problem.png?id=60fa4f735c467ac1c9438231a1782807 620w,/images/patterns/diagrams/template-method/problem-1.5x.png?id=83fa009f7727bdcc69335b946919f0d8 930w,/images/patterns/diagrams/template-method/problem-2x.png?id=fc8b434afec7b6135043d0d2f48189f0 1240w"
sizes="(max-width: 720px) 100vw, 620px" loading="lazy" width="620"
alt="Klasy eksploracji danych zawierały sporo powtarzającego się kodu" />
<figcaption><p>Klasy eksploracji danych zawierały sporo powtarzającego
się kodu.</p></figcaption>
</figure>

W jakimś momencie zauważasz, że wszystkie trzy klasy mają sporo
podobnego kodu. Fragmenty odpowiedzialne za pracę z różnymi formatami
danych są bardzo odmienne, ale kod odpowiedzialny za obróbkę i analizę
danych jest niemal identyczny. Świetnie byłoby się tych powtórzeń pozbyć
nie naruszając przy tym struktury algorytmów, prawda?

Istniał jeszcze jeden problem, dotyczący kodu klienckiego który
korzystał z tych klas. Miał pełno instrukcji warunkowych służących
wybieraniu odpowiedniego sposobu działania zależnie od klasy obiektu
przetwarzającego. Jeśli wszystkie trzy klasy przetwarzające miałyby
jeden wspólny interfejs lub wspólną klasę bazową, byłoby możliwe
usunięcie instrukcji warunkowych w kodzie klienckim i zastosowanie
polimorfizmu podczas wywoływania metod obiektu przetwarzającego.
:::

::: {.section .solution}
##  Rozwiązanie {#solution}

Rozwiązanie zawarte we wzorcu Metody szablonowej zakłada rozdzielenie
algorytmu na kolejne etapy, utworzenie z tych etapów metod i
umieszczenie ciągu wywołań poszczególnych metod w jednej *metodzie
szablonowej*. Etapy mogą być albo `abstrakcyjne`, albo posiadać jakąś
domyślną implementację. Aby skorzystać z algorytmu, klient powinien
dostarczyć swoją podklasę implementującą wszystkie etapy abstrakcyjne i
nadpisać opcjonalne etapy jeśli jest taka potrzeba (oprócz samej metody
szablonowej).

Zobaczmy jak się to sprawdzi w naszej aplikacji do eksploracji danych.
Możemy stworzyć klasę bazową dla wszystkich trzech algorytmów
parsowania. Ta klasa definiuje metodę szablonową składającą się z ciągu
wywołań różnych etapów przetwarzania dokumentu.

<figure class="image">
<img
src="../../images/patterns/diagrams/template-method/solution-pl1409.png?id=8a3a6cd2425f62036cd3b7ec036fc826"
style="aspect-ratio: 1.43;"
srcset="/images/patterns/diagrams/template-method/solution-pl.png?id=8a3a6cd2425f62036cd3b7ec036fc826 600w,/images/patterns/diagrams/template-method/solution-pl-1.5x.png?id=306e34a0d11a5b8a55a62acafd081f1e 900w,/images/patterns/diagrams/template-method/solution-pl-2x.png?id=5bae18393f996cb3b7bd6df220e24798 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Metoda szablonowa definiuje szkielet algorytmu" />
<figcaption><p>Metoda szablonowa dzieli algorytm na etapy, umożliwiając
podklasom ich nadpisywanie, ale nie samą
metodę szablonową.</p></figcaption>
</figure>

Na początek możemy zadeklarować wszystkie etapy jako `abstrakcyjne`,
zmuszając podklasy do ich zaimplementowania. W naszym przypadku podklasy
już posiadają konieczne implementacje, więc jedyną rzeczą jaka
pozostaje do zrobienia jest dostosowanie sygnatur metod tak, aby
zgadzały się z metodami klasy bazowej.

A teraz zobaczmy, co da się zrobić by zlikwidować duplikacje kodu. Kod
otwierania/zamykania pliku oraz ekstrakcji/parsowania wydaje się różnić
pomiędzy formatami danych, więc nie ma sensu się tymi metodami zajmować.
Jednakże implementacja pozostałych etapów, jak analiza surowych danych i
układania raportów, jest bardzo podobna. Można więc przenieść te
fragmenty do klasy bazowej, dzięki czemu podklasy będą mogły je
współdzielić.

Jak widać, mamy dwa rodzaje etapów:

-   *etapy abstrakcyjne*, które trzeba zaimplementować w każdej
    podklasie
-   *etapy opcjonalne*, które mają już jakąś domyślną implementację, ale
    nadal mogą być nadpisane, jeśli zaistnieje taka potrzeba

Istnieje jednak jeszcze jeden rodzaj etapów, zwane *hookami*. Hook to
opcjonalny, pusty etap. Metoda szablonowa będzie działać nawet jeśli nie
nadpisze się hooków. Zazwyczaj umieszcza się je przed i po istotnych
etapach algorytmu, dając tym samym podklasom dogodne punkty zaczepienia,
mogące służyć rozbudowie algorytmu.
:::

::: {.section .analogy}
##  Analogia do prawdziwego życia {#analogy}

<figure class="image">
<img
src="../../images/patterns/diagrams/template-method/live-exampleea7a.png?id=2485d52852f87da06c9cc0e2fd257d6a"
style="aspect-ratio: 1.74;"
srcset="/images/patterns/diagrams/template-method/live-example.png?id=2485d52852f87da06c9cc0e2fd257d6a 590w,/images/patterns/diagrams/template-method/live-example-1.5x.png?id=92c5fd9345329da09e3233302158678c 885w,/images/patterns/diagrams/template-method/live-example-2x.png?id=89083a3dcd1fe2b627b9b6e6ff4986dc 1180w"
sizes="(max-width: 720px) 100vw, 590px" loading="lazy" width="590"
alt="Seryjna produkcja domów" />
<figcaption><p>Typowy projekt architektoniczny można nieco zmienić by
zaspokoić potrzeby klienta.</p></figcaption>
</figure>

Opisywane tu podejście można zastosować w seryjnej budowie domów.
Projekt architektoniczny standardowego domu może mieć wiele punktów
zaczepienia, które pozwolą potencjalnemu właścicielowi dostosować
finalną budowlę do swoich potrzeb.

Każdy etap budowy, jak kładzenie fundamentów, szkielet, wznoszenie
ścian, instalację wodno-kanalizacyjna i elektryczną, itd., można nieco
zmodyfikować, czyniąc dom odmiennym od reszty.
:::

::::: {.section .structure-container}
##  Struktura {#structure}

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/template-method/structure1c3b.png?id=924692f994bff6578d8408d90f6fc459"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 0.89;"
srcset="/images/patterns/diagrams/template-method/structure.png?id=924692f994bff6578d8408d90f6fc459 340w,/images/patterns/diagrams/template-method/structure-1.5x.png?id=613d78ad8ec08536ec70f4e0976b5a1a 510w,/images/patterns/diagrams/template-method/structure-2x.png?id=25082d6d6a76f51c6b64d8aeeaffdbb5 680w"
sizes="(max-width: 720px) 100vw, 340px" loading="lazy" width="340"
alt="Struktura wzorca projektowego Metoda szablonowa" /><img
src="../../images/patterns/diagrams/template-method/structure-indexedb94b.png?id=4ced6107519bc66710d2f05c0f4097a1"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 0.92;"
srcset="/images/patterns/diagrams/template-method/structure-indexed.png?id=4ced6107519bc66710d2f05c0f4097a1 350w,/images/patterns/diagrams/template-method/structure-indexed-1.5x.png?id=58e3a9092786c824eb738a7771702093 525w,/images/patterns/diagrams/template-method/structure-indexed-2x.png?id=86f28789cdcc5a4c415d6a1100de56fc 700w"
sizes="(max-width: 720px) 100vw, 350px" loading="lazy" width="350"
alt="Struktura wzorca projektowego Metoda szablonowa" />
</figure>
:::

1.  **Klasa Abstrakcyjna** deklaruje metody stanowiące etapy algorytmu
    oraz faktyczną metodę szablonową która wywołuje poszczególne etapy w
    określonej kolejności. Etapy mogą być zadeklarowane jako
    `abstrakcyjne`, albo posiadać domyślną implementację.

2.  **Konkretne Klasy** mogą nadpisać każdy z etapów, oprócz samej
    metody szablonowej.
::::
:::::

::: {.section .pseudocode}
##  Pseudokod {#pseudocode}

W poniższym przykładzie, wzorzec **Metoda Szablonowa** daje "szkielet"
dla różnorakich gałęzi sztucznej inteligencji w prostej grze
strategicznej.

<figure class="image">
<img
src="../../images/patterns/diagrams/template-method/example32c6.png?id=c0ce5cc8070925a1cd345fac6afa16b6"
style="aspect-ratio: 1;"
srcset="/images/patterns/diagrams/template-method/example.png?id=c0ce5cc8070925a1cd345fac6afa16b6 430w,/images/patterns/diagrams/template-method/example-1.5x.png?id=d85c6b81c900f46d55688406170600bc 645w,/images/patterns/diagrams/template-method/example-2x.png?id=d8b309659c4b722237ec97733da935bd 860w"
sizes="(max-width: 720px) 100vw, 430px" loading="lazy" width="430"
alt="Struktura przykładu użycia wzorca Metoda szablonowa" />
<figcaption><p>Klasy SI prostej gry komputerowej.</p></figcaption>
</figure>

Wszystkie rasy mają niemal takie same typy jednostek i budynków. Możesz
więc ponownie wykorzystać tę samą strukturę SI w poszczególnych rasach,
nadpisując pewne szczegóły. Dzięki takiemu podejściu SI orków czyni je
agresywniejszymi, zaś ludzie charakteryzują się bardziej defensywną
postawą. Ponadto, potwory nie potrafią wznosić budynków. Dodanie
kolejnej rasy do gry wymagałoby stworzenia nowej podklasy SI i
nadpisania domyślnych metod zadeklarowanych w klasie bazowej SI.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// Klasa abstrakcyjna definiuje metodę szablonową która zawiera
// szkielet jakiegoś algorytmu skomponowany w formie zestawu
// wywołań prostych, abstrakcyjnych działań. Konkretne podklasy
// implementują te działania, ale pozostawiają samą metodę
// szablonową nietkniętą.
class GameAI is
    // Metoda szablonowa definiuje szkielet algorytmu.
    method turn() is
        collectResources()
        buildStructures()
        buildUnits()
        attack()

    // Niektóre etapy mogą zostać zaimplementowane już w klasie
    // bazowej.
    method collectResources() is
        foreach (s in this.builtStructures) do
            s.collect()

    // A inne mogą być zdefiniowane jako abstrakcyjne.
    abstract method buildStructures()
    abstract method buildUnits()

    // Klasa może mieć wiele metod szablonowych.
    method attack() is
        enemy = closestEnemy()
        if (enemy == null)
            sendScouts(map.center)
        else
            sendWarriors(enemy.position)

    abstract method sendScouts(position)
    abstract method sendWarriors(position)

// Konkretne klasy muszą zaimplementować wszystkie abstrakcyjne
// działania klasy bazowej, ale nie mogą nadpisać samej metody
// szablonowej.
class OrcsAI extends GameAI is
    method buildStructures() is
        if (there are some resources) then
            // Buduj farmy, potem koszary, potem twierdzę.

    method buildUnits() is
        if (there are plenty of resources) then
            if (there are no scouts)
                // Buduj piechura, dodaj go do grupy zwiadowców.
            else
                // Buduj żołnierza, dodaj go do grupy
                // wojowników.

    // ...

    method sendScouts(position) is
        if (scouts.length &gt; 0) then
            // Wyślij zwiadowców na pozycję.

    method sendWarriors(position) is
        if (warriors.length &gt; 5) then
            // Wyślij wojowników na pozycję.

// Podklasy mogą też nadpisać niektóre działania domyślną
// implementacją.
class MonstersAI extends GameAI is
    method collectResources() is
        // Potwory nie zbierają zasobów.

    method buildStructures() is
        // Potwory nie wznoszą budowli.

    method buildUnits() is
        // Potwory nie budują jednostek.</code></pre>
</figure>
:::

:::::::: {.section .applicability-container}
##  Zastosowanie {#applicability}

::::::: applicability
::: applicability-problem
Stosuj wzorzec Metoda szablonowa gdy chcesz pozwolić klientom na
rozszerzanie niektórych tylko etapów algorytmu, ale nie całego, ani też
jego struktury.
:::

::: applicability-solution
Metoda szablonowa pozwala zmienić monolityczny algorytm w ciąg
pojedynczych etapów, które można następnie łatwo rozszerzać w podklasach
bez naruszania struktury opisanej w klasie bazowej.
:::

::: applicability-problem
Wzorzec ten jest przydatny gdy masz wiele klas zawierających niemal
identyczne algorytmy różniące się jedynie szczegółami.  W takiej
sytuacji bowiem konieczność modyfikacji algorytmu skutkuje koniecznością
modyfikacji wszystkich klas.
:::

::: applicability-solution
Zmieniając taki algorytm w metodę szablonową możesz także przenieść jego
etapy o podobnej implementacji do klasy bazowej, eliminując tym samym
duplikację kodu. Kod który różni się pomiędzy podklasami, może w nich
pozostać.
:::
:::::::
::::::::

::: {.section .checklist}
##  Jak zaimplementować {#checklist}

1.  Przeanalizuj algorytm docelowy pod kątem możliwego podziału na
    etapy. Rozważ które z nich są wspólne dla wszystkich podklas, a
    które zawsze będą unikalne.

2.  Stwórz abstrakcyjną klasę bazową i zadeklaruj metodę szablonową oraz
    zestaw abstrakcyjnych metod reprezentujących etapy algorytmu.
    Nakreśl strukturę algorytmu w metodzie szablonowej poprzez
    uruchamianie odpowiednich etapów. Rozważ zastosowanie słowa
    kluczowego `final` w stosunku do metody szablonowej aby zapobiec
    nadpisaniu jej przez podklasy.

3.  Może się zdarzyć, że wszystkie etapy pozostaną abstrakcyjnymi.
    Jednak niektóre z nich skorzystałyby na posiadaniu domyślnej
    implementacji. Podklasy nie muszą implementować tych metod.

4.  Zastanów się nad dodaniem hooków pomiędzy kluczowymi etapami
    algorytmu.

5.  Dla każdego wariantu algorytmu stwórz nową konkretną podklasę.
    *Musi* ona implementować wszystkie abstrakcyjne etapy, ale *może*
    także nadpisać część opcjonalnych.
:::

:::::: {.section .pros-cons}
##  Zalety i wady {#pros-cons}

::::: row
::: col-sm-6
-    Można pozwolić klientom nadpisać tylko niektóre partie dużego
    algorytmu czyniąc go odporniejszym na szkody wskutek zmian
    poszczególnych jego części.
-    Można przenieść powtarzający się kod do klasy bazowej.
:::

::: col-sm-6
-    Dla niektórych klientów przygotowany szkielet algorytmu może
    stanowić ograniczenie.
-    Może prowadzić do naruszenia *Zasady podstawienia Liskov* wskutek
    stłumienia domyślnych implementacji etapów w podklasach.
-    Metody szablonowe zwykle trudniej utrzymywać w miarę jak przybywa
    etapów.
:::
:::::
::::::

::: {.section .relations}
##  Powiązania z innymi wzorcami {#relations}

-   [Metoda wytwórcza](factory-method.html) to wyspecjalizowana [Metoda
    szablonowa](template-method.html). Może stanowić także jeden z
    etapów większej *Metody szablonowej*.

-   [Metoda szablonowa](template-method.html) polega na mechanizmie
    dziedziczenia: pozwala zmieniać części algorytmu rozszerzając je w
    podklasach. [Strategia](strategy.html) bazuje na kompozycji: można
    zmienić część zachowania obiektu poprzez nadanie mu różnych
    strategii odpowiadających temu zachowaniu. *Metoda szablonowa*
    działa na poziomie klasy, więc jest statyczna. *Strategia* działa na
    poziomie obiektu, więc pozwala przełączać zachowania w trakcie
    działania programu.
:::

::: {.section .implementations}
##  Przykłady kodu {#implementations}

[![Metoda szablonowa w języku
C#](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](template-method/csharp/example.html "Metoda szablonowa w języku C#"){.prog-lang-link}
[![Metoda szablonowa w języku
C++](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](template-method/cpp/example.html "Metoda szablonowa w języku C++"){.prog-lang-link}
[![Metoda szablonowa w języku
Go](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](template-method/go/example.html "Metoda szablonowa w języku Go"){.prog-lang-link}
[![Metoda szablonowa w języku
Java](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](template-method/java/example.html "Metoda szablonowa w języku Java"){.prog-lang-link}
[![Metoda szablonowa w języku
PHP](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](template-method/php/example.html "Metoda szablonowa w języku PHP"){.prog-lang-link}
[![Metoda szablonowa w języku
Python](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](template-method/python/example.html "Metoda szablonowa w języku Python"){.prog-lang-link}
[![Metoda szablonowa w języku
Ruby](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](template-method/ruby/example.html "Metoda szablonowa w języku Ruby"){.prog-lang-link}
[![Metoda szablonowa w języku
Rust](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](template-method/rust/example.html "Metoda szablonowa w języku Rust"){.prog-lang-link}
[![Metoda szablonowa w języku
Swift](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](template-method/swift/example.html "Metoda szablonowa w języku Swift"){.prog-lang-link}
[![Metoda szablonowa w języku
TypeScript](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](template-method/typescript/example.html "Metoda szablonowa w języku TypeScript"){.prog-lang-link}
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

[Odwiedzający []{.fa .fa-arrow-right}](visitor.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Wróć

[[]{.fa .fa-arrow-left} Strategia](strategy.html){.btn .btn-default
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
