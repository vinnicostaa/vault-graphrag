::::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [Wzorce
projektowe](../design-patterns.html){.type} / [Wzorce
behawioralne](behavioral-patterns.html){.type}
:::

# Strategia {#strategia .title}

::: aka
Znany też jako: [Strategy]{style="display:inline-block"}
:::

::: {.section .intent}
##  Cel {#intent}

**Strategia** to behawioralny wzorzec projektowy pozwalający zdefiniować
rodzinę algorytmów, umieścić je w osobnych klasach i uczynić obiekty
tych klas wymienialnymi.

<figure class="image">
<img
src="../../images/patterns/content/strategy/strategy4ca6.png?id=379bfba335380500375881a3da6507e0"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/strategy/strategy.png?id=379bfba335380500375881a3da6507e0 640w,/images/patterns/content/strategy/strategy-1.5x.png?id=33ecebc66a9761454f2786a9b3e9bbe5 960w,/images/patterns/content/strategy/strategy-2x.png?id=1cee47d05a76fddf07dce9c67b700748 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Wzorzec projektowy Strategia" />
</figure>
:::

::: {.section .problem}
##  Problem

Pewnego dnia postanawiasz stworzyć aplikację służącą nawigacji dla
turystów. Aplikacja jest zbudowana w oparciu o piękną mapę ułatwiającą
użytkownikom szybko zorientować się w topografii miasta.

Jedną z funkcji, o którą użytkownicy prosili najczęściej, było
automatyczne planowanie trasy przez aplikację. Po wprowadzeniu
adresu, na mapie pokazałaby się najszybsza trasa do tego miejsca.

Pierwsza wersja aplikacji potrafiła planować trasy po drogach. Osoby
podróżujące autami były więc wniebowzięte. Okazało się jednak, że nie
wszyscy lubią jeździć na urlop autem. Dlatego kolejna aktualizacja
wprowadziła możliwość generowania szlaków pieszych. Niedługo po tym
aplikacja zyskała opcję wyznaczania tras w oparciu o komunikację
miejską.

To był jednak dopiero początek. W planach pojawiło się bowiem dodanie
obsługi tras dogodnych dla rowerzystów. A jeszcze później zaplanowano
tworzenie tras uwzględniających wszystkie atrakcje turystyczne miasta.

<figure class="image">
<img
src="../../images/patterns/diagrams/strategy/problem9b55.png?id=e5ca943e559a979bd31d20e232dc22b6"
style="aspect-ratio: 2.2;"
srcset="/images/patterns/diagrams/strategy/problem.png?id=e5ca943e559a979bd31d20e232dc22b6 330w,/images/patterns/diagrams/strategy/problem-1.5x.png?id=31d1042ffc28056845e45d5c616da2a9 495w,/images/patterns/diagrams/strategy/problem-2x.png?id=3974fb99c97aec525dd0ffcff2e48e78 660w"
sizes="(max-width: 720px) 100vw, 330px" loading="lazy" width="330"
alt="Kod nawigacji stał się bardzo zagmatwany." />
<figcaption><p>Kod nawigacji stał się zagmatwany.</p></figcaption>
</figure>

Chociaż z perspektywy biznesowej aplikacja odniosła sukces, aspekty
techniczne stały się dla ciebie zmorą. Po każdym dodaniu kolejnego
algorytmu wytyczania trasy, główna klasa nawigatora dwukrotnie się
powiększała. W pewnym momencie okiełznanie tej bestii stało się zbyt
trudne.

Każda zmiana któregoś z algorytmów --- usunięcie usterki, czy też
dostrajanie punktacji kolejnych odcinków trasy wpływało na całą klasę,
zwiększając tym samym ryzyko błędu w już działającym kodzie.

Na dodatek ucierpiała praca zespołowa. Twoi współpracownicy,
zatrudnieni po udanym wprowadzeniu programu na rynek, zaczęli
narzekać, że marnują zbyt wiele czasu na rozwiązywanie konfliktów
scalania. Implementacja każdej nowej funkcji wymaga zmiany tej samej
rozbudowanej klasy, nad którą jednocześnie pracują inni.
:::

::: {.section .solution}
##  Rozwiązanie {#solution}

Wzorzec Strategia proponuje ekstrakcję poszczególnych algorytmów
wykonujących dane zadanie na różne sposoby i umieszczenie ich w
odrębnych klasach, zwanych *strategiami*.

Pierwotna klasa, zwana *kontekstem*, musi zawierać pole służące
przechowywaniu odniesienia do którejś ze strategii. Kontekst deleguje
pracę powiązanemu obiektowi typu strategia, zamiast wykonywać ją
samodzielnie.

Kontekst nie jest odpowiedzialny za wybór stosownego algorytmu dla
danego zadania. To klient przekazuje żądaną strategię kontekstowi. Co
więcej, kontekst nie wie zbyt wiele o strategiach. Współpracuje ze
wszystkimi strategiami za pośrednictwem tego samego, ogólnego
interfejsu, który eksponuje pojedynczą metodę uruchamiającą algorytm
ukryty w danej strategii.

Tym sposobem kontekst staje się niezależny od konkretnych strategii,
więc można dodawać kolejne algorytmy, lub modyfikować istniejące, bez
zmieniania kodu kontekstu lub kodu innych strategii.

<figure class="image">
<img
src="../../images/patterns/diagrams/strategy/solution1435.png?id=0813a174b29a2ed5902d321aba28e5fc"
style="aspect-ratio: 2.04;"
srcset="/images/patterns/diagrams/strategy/solution.png?id=0813a174b29a2ed5902d321aba28e5fc 570w,/images/patterns/diagrams/strategy/solution-1.5x.png?id=ce3d4e57f4a2a06ebc96f2607b3d6691 855w,/images/patterns/diagrams/strategy/solution-2x.png?id=66b5ee048ea2ad25c4b20f180ebf94d7 1140w"
sizes="(max-width: 720px) 100vw, 570px" loading="lazy" width="570"
alt="Strategie planowania trasy" />
<figcaption><p>Strategie planowania trasy.</p></figcaption>
</figure>

W naszej aplikacji nawigacyjnej, każdy algorytm wyznaczania trasy może
zostać wyekstrahowany do swojej własnej, odrębnej klasy, posiadającej
jedną metodę `stwórzTrasę`. Metoda przyjmuje informacje o punkcie
początkowym i docelowym, a zwraca zestaw kluczowych etapów wyznaczonej
trasy.

Każda klasa wyznaczająca trasę może wytyczyć inną trasę w oparciu o te
same argumenty, a główna klasa nawigator nie musi wiedzieć o obranym
algorytmie, gdyż zajmuje się jedynie przedstawianiem trasy na mapie.
Klasa posiada metodę umożliwiającą zmianę aktywnej strategii planowania
trasy, dzięki czemu jej klienci, jak na przykład przyciski interfejsu
użytkownika, mogą zmienić bieżący sposób wyznaczania trasy na inny.
:::

::: {.section .analogy}
##  Analogia do prawdziwego życia {#analogy}

<figure class="image">
<img
src="../../images/patterns/content/strategy/strategy-comic-1-pl3a7d.png?id=44f98669806037b001158afb36149017"
style="aspect-ratio: 2.13;"
srcset="/images/patterns/content/strategy/strategy-comic-1-pl.png?id=44f98669806037b001158afb36149017 640w,/images/patterns/content/strategy/strategy-comic-1-pl-1.5x.png?id=31f36da79da29aa346d5624b88248ad4 960w,/images/patterns/content/strategy/strategy-comic-1-pl-2x.png?id=a8e9c4af42a950abec444240f65b9592 1280w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="640"
alt="Różne strategie przewozu" />
<figcaption><p>Różne strategie dotarcia na lotnisko.</p></figcaption>
</figure>

Wyobraź sobie, że musisz dotrzeć na lotnisko. Możesz złapać autobus,
zamówić taksówkę lub pojechać rowerem. To są twoje strategie przejazdu.
Możesz wybrać jedną z nich zależnie od czynników takich jak budżet lub
ograniczenia czasowe.
:::

::::: {.section .structure-container}
##  Struktura {#structure}

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/strategy/structuree491.png?id=c6aa910c94960f35d100bfca02810ea1"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.19;"
srcset="/images/patterns/diagrams/strategy/structure.png?id=c6aa910c94960f35d100bfca02810ea1 440w,/images/patterns/diagrams/strategy/structure-1.5x.png?id=5b1c6376b06374719dcae95455b068d8 660w,/images/patterns/diagrams/strategy/structure-2x.png?id=5bd791857c3bab419bcf4fa86877439d 880w"
sizes="(max-width: 720px) 100vw, 440px" loading="lazy" width="440"
alt="Struktura wzorca Strategia" /><img
src="../../images/patterns/diagrams/strategy/structure-indexed64c7.png?id=ff55c5a6273cf82a5667f3cab5b605c7"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.15;"
srcset="/images/patterns/diagrams/strategy/structure-indexed.png?id=ff55c5a6273cf82a5667f3cab5b605c7 450w,/images/patterns/diagrams/strategy/structure-indexed-1.5x.png?id=3d6ba05b8a74a9fb8e3f88eb9ca1e30f 675w,/images/patterns/diagrams/strategy/structure-indexed-2x.png?id=9f8e2f838f16705775411e2b4616820e 900w"
sizes="(max-width: 720px) 100vw, 450px" loading="lazy" width="450"
alt="Struktura wzorca Strategia" />
</figure>
:::

1.  **Kontekst** przechowuje odniesienie do jednej z konkretnych
    strategii i komunikuje się z jej obiektem za pośrednictwem
    interfejsu strategia.

2.  Interfejs **Strategia** jest wspólny dla wszystkich konkretnych
    strategii. Deklaruje metodę za pomocą której kontekst uruchamia daną
    strategię.

3.  **Konkretne Strategie** implementują różne warianty algorytmu z
    którego korzysta kontekst.

4.  Kontekst wywołuje metodę uruchamiającą eksponowaną przez powiązany z
    nim obiekt strategii za każdym razem gdy chce uruchomić algorytm.
    Kontekst nie wie z jaką strategią ma do czynienia, lub jak działa
    algorytm.

5.  **Klient** tworzy określony obiekt strategii i przekazuje go
    kontekstowi. Kontekst eksponuje metodę setter pozwalającą klientom
    zamienić strategię skojarzoną z tym kontekstem w trakcie działania
    programu.
::::
:::::

::: {.section .pseudocode}
##  Pseudokod {#pseudocode}

W poniższym przykładzie, kontekst wykorzystuje kilka **strategii** by
wykonać różne działania arytmetyczne.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// Interfejs strategii deklaruje działania wspólne dla
// wszystkich wspieranych wersji jakiegoś algorytmu. Kontekst
// korzysta z tego interfejsu w celu wywoływania algorytmów
// zdefiniowanych przez konkretne strategie.
interface Strategy is
    method execute(a, b)

// Konkretne strategie implementują algorytm według poniższego
// interfejsu bazowej strategii. Interfejs sprawia, że są one
// wymienialne w kontekście.
class ConcreteStrategyAdd implements Strategy is
    method execute(a, b) is
        return a + b

class ConcreteStrategySubtract implements Strategy is
    method execute(a, b) is
        return a - b

class ConcreteStrategyMultiply implements Strategy is
    method execute(a, b) is
        return a * b

// Kontekst definiuje interfejs stanowiący przedmiot
// zainteresowania klientów.
class Context is
    // Kontekst posiada odniesienie do jednego z obiektów-
    // strategii. Kontekst nie zna konkretnej klasy strategii.
    // Powinien współpracować ze wszystkimi strategiami za
    // pośrednictwem wspólnego interfejsu strategii.
    private strategy: Strategy

    // Zazwyczaj kontekst przyjmuje strategię poprzez
    // konstruktor. Udostępnia także funkcję setter
    // umożliwiającą wymianę strategii na inną w trakcie
    // działania programu.
    method setStrategy(Strategy strategy) is
        this.strategy = strategy

    // Kontekst deleguje jakąś pracę obiektowi-strategii zamiast
    // samodzielnie implementować wiele wersji algorytmu.
    method executeStrategy(int a, int b) is
        return strategy.execute(a, b)


// Kod klienta wybiera jakąś konkretną strategię i przekazuje ją
// kontekstowi. Klient powinien być świadom różnic pomiędzy
// poszczególnymi strategiami aby móc podjąć właściwy wybór.
class ExampleApplication is
    method main() is
        Create context object.

        Read first number.
        Read last number.
        Read the desired action from user input.

        if (action == addition) then
            context.setStrategy(new ConcreteStrategyAdd())

        if (action == subtraction) then
            context.setStrategy(new ConcreteStrategySubtract())

        if (action == multiplication) then
            context.setStrategy(new ConcreteStrategyMultiply())

        result = context.executeStrategy(First number, Second number)

        Print result.</code></pre>
</figure>
:::

:::::::::::: {.section .applicability-container}
##  Zastosowanie {#applicability}

::::::::::: applicability
::: applicability-problem
Stosuj wzorzec Strategia gdy chcesz używać różnych wariantów jednego
algorytmu w obrębie obiektu i zyskać możliwość zmiany wyboru wariantu w
trakcie działania programu.
:::

::: applicability-solution
Wzorzec Strategia pozwala pośrednio zmienić zachowanie obiektu w trakcie
działania programu poprzez przypisywanie temu obiektowi różnych
podobiektów wykonujących określone poddziałania na różne sposoby.
:::

::: applicability-problem
Warto stosować ten wzorzec gdy masz w programie wiele podobnych klas,
różniących się jedynie sposobem wykonywania jakichś zadań.
:::

::: applicability-solution
Wzorzec Strategia umożliwia ekstrakcję różniących się zachowań do
odrębnej hierarchii klas i połączenie pierwotnych klas w jedną,
redukując tym samym powtórzenia kodu.
:::

::: applicability-problem
Strategia pozwala odizolować logikę biznesową klasy od szczegółów
implementacyjnych algorytmów, które nie są istotne w kontekście tej
logiki.
:::

::: applicability-solution
Strategia umożliwia odizolowanie kodu różnych algorytmów, ich danych
wewnętrznych oraz zależności od reszty kodu. Klienci otrzymują prosty
interfejs umożliwiający uruchamianie algorytmów i wymiany ich na inne w
trakcie działania programu.
:::

::: applicability-problem
Stosuj ten wzorzec gdy twoja klasa zawiera duży operator warunkowy,
którego zadaniem jest wybór odpowiedniego wariantu tego samego
algorytmu.
:::

::: applicability-solution
Wzorzec Strategia pozwala pozbyć się wyżej wymienionych kawałków kodu
poprzez ekstrakcję algorytmów do odrębnych klas implementujących taki
sam interfejs. Pierwotny obiekt deleguje uruchamianie jednemu z tych
obiektów, zamiast samodzielnie implementować wszystkie warianty
algorytmu.
:::
:::::::::::
::::::::::::

::: {.section .checklist}
##  Jak zaimplementować {#checklist}

1.  W klasie kontekstu zidentyfikuj algorytm który często może ulegać
    zmianom. Może to być też obszerna instrukcja warunkowa wybierająca i
    uruchamiająca wariant tego samego algorytmu w trakcie działania
    programu.

2.  Zadeklaruj interfejs strategii który będzie wspólny dla wszystkich
    wariantów algorytmu.

3.  Jeden po drugim ekstrahuj wszystkie algorytmy do odrębnych klas
    implementujących interfejs strategii.

4.  W klasie kontekstu, dodaj pole służące przechowywaniu odniesienia do
    obiektu strategii. Przygotuj metodę setter służącą zmianie wartości
    tego pola. Kontekst powinien współpracować z obiektem strategii
    wyłącznie przez interfejs strategii. Kontekst może definiować
    interfejs dający strategii dostęp do swoich danych.

5.  Klienci kontekstu muszą skojarzyć go ze stosowną strategią, która
    odpowiada sposobowi w jaki kontekst ma wykonać swoje zadanie.
:::

:::::: {.section .pros-cons}
##  Zalety i wady {#pros-cons}

::::: row
::: col-sm-6
-    Możesz zamieniać algorytmy stosowane w obrębie obiektu w trakcie
    działania programu.
-    Możesz odizolować szczegóły implementacyjne algorytmu od kodu
    który z niego korzysta.
-    Umożliwia zamianę dziedziczenia na kompozycję.
-    *Zasada otwarte/zamknięte*. Możliwe jest wprowadzanie nowych
    strategii bez konieczności dokonywania zmian w kontekście.
:::

::: col-sm-6
-    Jeśli masz zaledwie kilka algorytmów i rzadko ulegają one zmianie,
    nie ma wyraźnej potrzeby nadmiernego komplikowania programu przez
    dodawanie nowych klas i interfejsów związanych z tym wzorcem.
-    Klienci muszą być świadomi różnic pomiędzy poszczególnymi
    strategiami, aby mogli wybrać właściwą.
-    Wiele nowoczesnych języków programowania posiada wsparcie dla typów
    funkcyjnych pozwalających zaimplementować różne wersje algorytmu
    wewnątrz zestawu anonimowych funkcji. Można następnie korzystać z
    tych funkcji dokładnie tak jak z obiektów strategia, ale bez
    konieczności rozbudowy kodu o kolejne klasy i interfejsy.
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

-   [Polecenie](command.html) i [Strategia](strategy.html) mogą wydawać
    się podobne, ponieważ oba mogą służyć parametryzacji obiektu jakimś
    działaniem. Mają jednak inne cele.

    -   Za pomocą *Polecenia* można konwertować dowolne działanie na
        obiekt. Parametry działania stają się polami tego obiektu.
        Konwersja zaś pozwala odroczyć wykonanie działania,
        kolejkować je i przechowywać historię wykonanych działań, a
        także wysyłać polecenia zdalnym usługom, itd.

    -   Z drugiej strony, *Strategia* zazwyczaj opisuje różne sposoby
        wykonywania danej czynności, pozwalając zamieniać algorytmy w
        ramach jednej klasy kontekstu.

-   [Dekorator](decorator.html) pozwala zmienić otoczkę obiektu, zaś
    [Strategia](strategy.html) jej wnętrze.

-   [Metoda szablonowa](template-method.html) polega na mechanizmie
    dziedziczenia: pozwala zmieniać części algorytmu rozszerzając je w
    podklasach. [Strategia](strategy.html) bazuje na kompozycji: można
    zmienić część zachowania obiektu poprzez nadanie mu różnych
    strategii odpowiadających temu zachowaniu. *Metoda szablonowa*
    działa na poziomie klasy, więc jest statyczna. *Strategia* działa na
    poziomie obiektu, więc pozwala przełączać zachowania w trakcie
    działania programu.

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

[![Strategia w języku
C#](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](strategy/csharp/example.html "Strategia w języku C#"){.prog-lang-link}
[![Strategia w języku
C++](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](strategy/cpp/example.html "Strategia w języku C++"){.prog-lang-link}
[![Strategia w języku
Go](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](strategy/go/example.html "Strategia w języku Go"){.prog-lang-link}
[![Strategia w języku
Java](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](strategy/java/example.html "Strategia w języku Java"){.prog-lang-link}
[![Strategia w języku
PHP](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](strategy/php/example.html "Strategia w języku PHP"){.prog-lang-link}
[![Strategia w języku
Python](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](strategy/python/example.html "Strategia w języku Python"){.prog-lang-link}
[![Strategia w języku
Ruby](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](strategy/ruby/example.html "Strategia w języku Ruby"){.prog-lang-link}
[![Strategia w języku
Rust](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](strategy/rust/example.html "Strategia w języku Rust"){.prog-lang-link}
[![Strategia w języku
Swift](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](strategy/swift/example.html "Strategia w języku Swift"){.prog-lang-link}
[![Strategia w języku
TypeScript](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](strategy/typescript/example.html "Strategia w języku TypeScript"){.prog-lang-link}
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

[Metoda szablonowa []{.fa .fa-arrow-right}](template-method.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Wróć

[[]{.fa .fa-arrow-left} Stan](state.html){.btn .btn-default rel="prev"}
:::
::::::::::::::::::::::::::::::::::::

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
::::::::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::::::::
