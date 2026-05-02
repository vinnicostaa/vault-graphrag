:::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [Wzorce
projektowe](../design-patterns.html){.type} / [Wzorce
kreacyjne](creational-patterns.html){.type}
:::

# Singleton {#singleton .title}

::: {.section .intent}
##  Cel {#intent}

**Singleton** jest kreacyjnym wzorcem projektowym, który pozwala
zapewnić istnienie wyłącznie jednej instancji danej klasy. Ponadto daje
globalny punkt dostępowy do tejże instancji.

<figure class="image">
<img
src="../../images/patterns/content/singleton/singleton4263.png?id=108a0b9b5ea5c4426e0afa4504491d6f"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/singleton/singleton.png?id=108a0b9b5ea5c4426e0afa4504491d6f 640w,/images/patterns/content/singleton/singleton-1.5x.png?id=29490ad6cd1426c63cf5f88243a1701c 960w,/images/patterns/content/singleton/singleton-2x.png?id=accb2cc7594f7a491ce01dddf0d2f876 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Wzorzec Singleton" />
</figure>
:::

::: {.section .problem}
##  Problem

Wzorzec Singleton rozwiązuje jednocześnie dwa problemy, ale jednocześnie
łamie *Zasadę pojedynczej odpowiedzialności*:

1.  **Zapewnia istnienie wyłącznie jednej instancji danej klasy**.
    Dlaczego w ogóle ktokolwiek musiałby liczyć obiekty danej klasy?
    Otóż, najczęstszym powodem jest potrzeba kontroli dostępu do
    jakiegoś współdzielonego zasobu --- na przykład bazy danych, lub
    pliku.

    Działa to tak: wyobraź sobie, że masz już stworzony obiekt, ale po
    jakimś czasie potrzebujesz kolejnego. Zamiast otrzymać nowy,
    dostaniesz ten uprzednio stworzony.

    Zwróć uwagę, że takiego zachowania nie da się zaimplementować
    stosując zwykły konstruktor, ponieważ metody te z definicji
    **muszą** zawsze zwracać nowe obiekty.

<figure class="image">
<img
src="../../images/patterns/content/singleton/singleton-comic-1-plf14a.png?id=b8d840f35eea355c62de52d4fdc579d6"
style="aspect-ratio: 2;"
srcset="/images/patterns/content/singleton/singleton-comic-1-pl.png?id=b8d840f35eea355c62de52d4fdc579d6 600w,/images/patterns/content/singleton/singleton-comic-1-pl-1.5x.png?id=110924d28ec6d776347497284d3f708f 900w,/images/patterns/content/singleton/singleton-comic-1-pl-2x.png?id=fc8a61f3254c5f4b6547bfca851a0ce9 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Globalny dostęp do obiektu" />
<figcaption><p>Klienci mogą nawet nie zauważyć, że cały czas
korzystają z tego samego obiektu.</p></figcaption>
</figure>

2.  **Pozwala na dostęp do tej instancji w przestrzeni globalnej**.
    Pamiętasz te globalne zmienne, z których korzystaliśmy (no dobra, ja
    korzystałem) do przechowywania istotnych obiektów? Chociaż są one
    bardzo poręczne, to wiążą się też z poważnym ryzykiem nadpisania ich
    zawartości i tym samym awarii programu.

    Zupełnie, jak w przypadku zmiennej globalnej, wzorzec Singleton
    pozwala skorzystać z jakiegoś obiektu w dowolnym miejscu programu.
    Jednakże, zapewnia też ochronę tego obiektu przed działaniami innego
    kodu.

    Jest też inna strona tego problemu: nie chcemy, aby kod, który
    pozwala nam rozwiązać pierwszy z powyższych problemów był
    porozrzucany po całym programie. Dużo lepiej jest trzymać go w
    jednej klasie, szczególnie, jeśli reszta kodu już od niego zależy.

Wzorzec Singleton stał się obecnie tak popularny, że czasem nazywa się
Singletonem rozwiązania odnoszące się tylko do jednego z powyższych
problemów.
:::

::: {.section .solution}
##  Rozwiązanie {#solution}

Wszystkie implementacje wzorca Singleton współdzielą poniższe dwa etapy:

-   Ograniczenie dostępu do domyślnego konstruktora przez uczynienie go
    prywatnym, aby zapobiec stosowaniu operatora `new` w stosunku do
    klasy Singleton.
-   Utworzenie statycznej metody kreacyjnej, która będzie pełniła rolę
    konstruktora. Za kulisami, metoda ta wywoła prywatny konstruktor,
    aby utworzyć instancję obiektu i umieści go w polu statycznym klasy.
    Wszystkie kolejne wywołania tej metody zwrócą już istniejący obiekt.

Jeżeli twój kod ma dostęp do klasy Singleton, to będzie mógł wywołać jej
statyczną metodę i tym samym za każdym razem otrzyma ten sam obiekt.
:::

::: {.section .analogy}
##  Analogia do prawdziwego życia {#analogy}

Rząd jest doskonałym przykładem wzorca Singleton. Kraj może mieć
wyłącznie jeden oficjalny rząd. Niezależnie od składu personalnego
członków rządu, pojęcie "Rząd kraju X" jest uniwersalnym odwołaniem do
organu władzy kraju.
:::

::::: {.section .structure-container}
##  Struktura {#structure}

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/singleton/structure-plf88b.png?id=442bf5dc13d3303acf426fda5d4c4861"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.48;"
srcset="/images/patterns/diagrams/singleton/structure-pl.png?id=442bf5dc13d3303acf426fda5d4c4861 430w,/images/patterns/diagrams/singleton/structure-pl-1.5x.png?id=2e50b143e28c274e9d15f20a609ac9f0 645w,/images/patterns/diagrams/singleton/structure-pl-2x.png?id=2cbd074f11295569999a87f7af984f63 860w"
sizes="(max-width: 720px) 100vw, 430px" loading="lazy" width="430"
alt="Struktura wzorca projektowego Singleton" /><img
src="../../images/patterns/diagrams/singleton/structure-pl-indexeda353.png?id=817adf3c847d82f2a628ac892d67cdd1"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.48;"
srcset="/images/patterns/diagrams/singleton/structure-pl-indexed.png?id=817adf3c847d82f2a628ac892d67cdd1 430w,/images/patterns/diagrams/singleton/structure-pl-indexed-1.5x.png?id=b083978d7ed5da18b5d70506da9efcf2 645w,/images/patterns/diagrams/singleton/structure-pl-indexed-2x.png?id=1a8d7be980af40790184cf3e50f90636 860w"
sizes="(max-width: 720px) 100vw, 430px" loading="lazy" width="430"
alt="Struktura wzorca projektowego Singleton" />
</figure>
:::

1.  Klasa **Singleton** deklaruje statyczną metodę `getInstance`, która
    zawsze zwraca tę samą instancję swej klasy.

    Konstruktor klasy Singleton musi być ukryty przed kodem klienckim.
    Tylko i wyłącznie wywołanie metody `getInstance` powinno dawać
    dostęp do takiego obiektu.
::::
:::::

::: {.section .pseudocode}
##  Pseudokod {#pseudocode}

W poniższym przykładzie, połączenie z bazą danych zrealizowane jest jako
**Singleton**. Klasa ta nie ma publicznie dostępnego konstruktora, więc
aby uzyskać dostęp do jej instancji, trzeba wywołać metodę
`getInstance`. Metoda ta zachowuje pierwszy stworzony egzemplarz klasy i
zwraca go przy każdym kolejnym wywołaniu.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// Klasa Database definiuje metodę `getInstance` która pozwala
// klientom na dostęp do tej samej instancji połączenia
// bazodanowego z dowolnego miejsca w programie.
class Database is
    // Pole służące przechowywaniu instancji singleton powinno
    // być zadeklarowane jako statyczne.
    private static field instance: Database

    // Konstruktor singletona powinien zawsze być zadeklarowany
    // jako prywatny, aby uniemożliwić wywoływanie na klasie
    // operatora `new`.
    private constructor Database() is
        // Jakiś kod inicjalizacyjny, na przykład faktyczne
        // nawiązanie połączenia z serwerem bazy danych.
        // ...

    // Statyczna metoda kontrolująca dostęp do instancji
    // singletona.
    public static method getInstance() is
        if (Database.instance == null) then
            acquireThreadLock() and then
                // Trzeba się upewnić, że instancja nie została
                // już zainicjalizowana przez inny wątek w
                // czasie gdy ten wątek oczekiwał na zwolnienie
                // blokady.
                if (Database.instance == null) then
                    Database.instance = new Database()
        return Database.instance

    // Wreszcie — każdy singleton powinien definiować jakąś
    // logikę biznesową którą można wywołać z instancji
    // singletona.
    public method query(sql) is
        // Przykładowo, wszystkie zapytania do bazy danych w
        // całej aplikacji muszą przejść przez tę metodę. Możesz
        // tu więc umieścić kod pamięci podręcznej lub kontroli
        // przepustowości.
        // ...

class Application is
    method main() is
        Database foo = Database.getInstance()
        foo.query(&quot;SELECT ...&quot;)
        // ...
        Database bar = Database.getInstance()
        bar.query(&quot;SELECT ...&quot;)
        // Zmienna `bar` będzie zawierała ten sam obiekt, co
        // zmienna `foo`.</code></pre>
</figure>
:::

:::::::: {.section .applicability-container}
##  Zastosowanie {#applicability}

::::::: applicability
::: applicability-problem
Korzystaj z wzorca Singleton, gdy w twoim programie ma prawo istnieć
wyłącznie jeden ogólnodostępny obiekt danej klasy. Przykładem może być
połączenie z bazą danych, którego używa wiele fragmentów programu.
:::

::: applicability-solution
Wzorzec projektowy Singleton uniemożliwia tworzenie obiektów danej klasy
inaczej, niż przez stosowną metodę kreacyjną. Ta z kolei zwróci albo
nowy obiekt, albo wcześniej stworzony.
:::

::: applicability-problem
Stosuj wzorzec Singleton gdy potrzebujesz ściślejszej kontroli nad
zmiennymi globalnymi.
:::

::: applicability-solution
 W przeciwieństwie do zmiennych globalnych, wzorzec Singleton gwarantuje
istnienie tylko jednego obiektu danej klasy. Nic, oprócz samej klasy,
nie jest w stanie zamienić tego obiektu.

Zwróć uwagę, że zawsze można zmienić ograniczenie dotyczące ilości i
pozwolić na jakąś inną maksymalną liczbę jej instancji. Wystarczy
zmienić jeden fragment kodu --- metodę `getInstance`.
:::
:::::::
::::::::

::: {.section .checklist}
##  Jak zaimplementować {#checklist}

1.  Dodaj prywatne, statyczne pole klasy celem przechowywania w nim
    instancji singleton.

2.  Zadeklaruj publicznie dostępną, statyczną metodę kreacyjną która
    daje dostęp do instancji klasy singleton.

3.  Zaimplementuj w tej metodzie statycznej "leniwą inicjalizację".
    Oznacza to, że nowy obiekt powinien być tworzony tylko przy
    pierwszym wywołaniu metody i zdeponowany w statycznym polu klasy.
    Przy kolejnych wywołaniach, metoda powinna zwracać już istniejący
    egzemplarz.

4.  Uczyń konstruktor klasy prywatnym. Statyczna metoda klasy będzie
    miała do niego dostęp, ale obiekty innych klas --- już nie.

5.  Przejrzyj kod kliencki i zamień wszystkie bezpośrednie wywołania
    konstruktora klasy singleton na wywołania jej statycznej metody
    kreacyjnej.
:::

:::::: {.section .pros-cons}
##  Zalety i wady {#pros-cons}

::::: row
::: col-sm-6
-    Masz pewność, że istnieje tylko jedna instancja klasy.
-    Zyskujesz globalny dostęp do tej instancji.
-    Obiekt singleton inicjalizowany jest dopiero wtedy, gdy jest po raz
    pierwszy potrzebny.
:::

::: col-sm-6
-    Łamie *Zasadę pojedynczej odpowiedzialności*. Wzorzec rozwiązuje
    bowiem dwa różne problemy jednocześnie.
-    Zastosowanie wzorca Singleton może zamaskować niewłaściwe
    projektowanie. Można na przykład doprowadzić do sytuacji, w której
    komponenty programu wiedzą zbyt wiele o sobie nawzajem.
-    Wzorzec ten wymaga specjalnej uwagi w środowisku wielowątkowym, w
    którym trzeba unikać tworzenia wielu instancji singletona przez
    wiele wątków.
-    Utrudnieniu mogą ulec testy jednostkowe kodu klienckiego klasy
    singleton, ponieważ wiele frameworków testujących polega na
    dziedziczeniu przy produkcji atrap obiektów. Ponieważ konstruktor
    klasy Singleton jest prywatny, a nadpisywanie statycznych metod jest
    niemożliwe w większości języków programowania, będzie trzeba wykazać
    się kreatywnością i znaleźć jakiś inny sposób tworzenia atrapy
    singletona. Albo nie pisać testów, albo zrezygnować z tego wzorca.
:::
:::::
::::::

::: {.section .relations}
##  Powiązania z innymi wzorcami {#relations}

-   Klasa [Fasada](facade.html) może często być przekształcona w
    [Singleton](singleton.html), ponieważ pojedynczy obiekt fasady
    jest w większości przypadków wystarczający.

-   [Pyłek](flyweight.html) mógłby przypominać
    [Singleton](singleton.html), gdybyśmy zdołali zredukować wszystkie
    współdzielone stany obiektów do tylko jednego obiektu-pyłka. Ale są
    jeszcze dwie fundamentalne różnice między tymi wzorcami:

    1.  Powinna istnieć tylko jedna instancja interfejsu Singleton, zaś
        instancji *Pyłka* będzie wiele, o różnym stanie wewnętrznym.
    2.  Obiekt *Singleton* może być zmienny. Pyłki są zaś niezmienne.

-   [Fabryki abstrakcyjne](abstract-factory.html),
    [Budowniczych](builder.html) oraz [Prototypy](prototype.html) można
    zaimplementować jako [Singletony](singleton.html).
:::

::: {.section .implementations}
##  Przykłady kodu {#implementations}

[![Singleton w języku
C#](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](singleton/csharp/example.html "Singleton w języku C#"){.prog-lang-link}
[![Singleton w języku
C++](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](singleton/cpp/example.html "Singleton w języku C++"){.prog-lang-link}
[![Singleton w języku
Go](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](singleton/go/example.html "Singleton w języku Go"){.prog-lang-link}
[![Singleton w języku
Java](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](singleton/java/example.html "Singleton w języku Java"){.prog-lang-link}
[![Singleton w języku
PHP](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](singleton/php/example.html "Singleton w języku PHP"){.prog-lang-link}
[![Singleton w języku
Python](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](singleton/python/example.html "Singleton w języku Python"){.prog-lang-link}
[![Singleton w języku
Ruby](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](singleton/ruby/example.html "Singleton w języku Ruby"){.prog-lang-link}
[![Singleton w języku
Rust](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](singleton/rust/example.html "Singleton w języku Rust"){.prog-lang-link}
[![Singleton w języku
Swift](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](singleton/swift/example.html "Singleton w języku Swift"){.prog-lang-link}
[![Singleton w języku
TypeScript](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](singleton/typescript/example.html "Singleton w języku TypeScript"){.prog-lang-link}
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

[Wzorce strukturalne []{.fa
.fa-arrow-right}](structural-patterns.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Wróć

[[]{.fa .fa-arrow-left} Prototyp](prototype.html){.btn .btn-default
rel="prev"}
:::
:::::::::::::::::::::::::::::::

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
:::::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::::
