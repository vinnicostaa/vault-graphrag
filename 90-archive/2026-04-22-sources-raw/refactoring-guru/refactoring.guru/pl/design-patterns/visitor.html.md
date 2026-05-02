:::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [Wzorce
projektowe](../design-patterns.html){.type} / [Wzorce
behawioralne](behavioral-patterns.html){.type}
:::

# Odwiedzający {#odwiedzający .title}

::: aka
Znany też jako: [Visitor]{style="display:inline-block"}
:::

::: {.section .intent}
##  Cel {#intent}

**Odwiedzający** to behawioralny wzorzec projektowy pozwalający
oddzielić algorytmy od obiektów na których pracują.

<figure class="image">
<img
src="../../images/patterns/content/visitor/visitor4a4c.png?id=f36d100188340db7a18854ef7916f972"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/visitor/visitor.png?id=f36d100188340db7a18854ef7916f972 640w,/images/patterns/content/visitor/visitor-1.5x.png?id=751e251363b632b20df856d72d02ee72 960w,/images/patterns/content/visitor/visitor-2x.png?id=2c5d9ab3046d782c19809d3b80650d65 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Wzorzec projektowy Odwiedzający" />
</figure>
:::

::: {.section .problem}
##  Problem

Wyobraź sobie, że twój zespół opracowuje aplikację korzystającą z danych
geograficznych ustrukturyzowanych w jeden wielki graf. Każdy węzeł grafu
odpowiada złożonemu podmiotowi jak miasto, ale również bardziej
szczegółowym elementom, takim jak obiekty przemysłowe, atrakcje
turystyczne, itp. Węzły są połączone ze sobą jeśli istnieje droga
pomiędzy faktycznymi obiektami jakie reprezentują. Za kulisami każdy typ
węzła reprezentowany jest przez osobną klasę, zaś poszczególne węzły to
ich obiekty.

<figure class="image">
<img
src="../../images/patterns/diagrams/visitor/problem14338.png?id=e7076532da1e936f3519c63270da8454"
style="aspect-ratio: 2.43;"
srcset="/images/patterns/diagrams/visitor/problem1.png?id=e7076532da1e936f3519c63270da8454 560w,/images/patterns/diagrams/visitor/problem1-1.5x.png?id=250216d3458c38e9855eda96bf71251b 840w,/images/patterns/diagrams/visitor/problem1-2x.png?id=2e5d5143ac55af218754f28761bec17e 1120w"
sizes="(max-width: 720px) 100vw, 560px" loading="lazy" width="560"
alt="Eksportowanie grafu do XML" />
<figcaption><p>Eksportowanie grafu do XML.</p></figcaption>
</figure>

W którymś momencie otrzymujesz zadanie implementacji eksportu grafu do
formatu XML. Początkowo zadanie wydało ci się proste. Planujesz dodanie
metody eksportującej do każdej klasy węzła, a następnie rekursywne
wywołanie jej w każdym z węzłów. Rozwiązanie wydaje się proste i
eleganckie: dzięki polimorfizmowi nie doszło do sprzęgnięcia kodu
wywołującego metodę eksportującą z konkretnymi klasami węzłów.

Niestety architekt systemu odmówił zgody na modyfikację istniejących
klas węzłów. Stwierdził, że kod jest już na etapie produkcji i nie może
sobie pozwolić na ryzyko związane z wprowadzeniem ewentualnego błędu
wraz z twoimi zmianami.

<figure class="image">
<img
src="../../images/patterns/diagrams/visitor/problem2-pl2c1a.png?id=51fd8cc1d37014c0b60ed4305ff61f72"
style="aspect-ratio: 1.92;"
srcset="/images/patterns/diagrams/visitor/problem2-pl.png?id=51fd8cc1d37014c0b60ed4305ff61f72 500w,/images/patterns/diagrams/visitor/problem2-pl-1.5x.png?id=a7334db042b52a36ee157b00ecd794fa 750w,/images/patterns/diagrams/visitor/problem2-pl-2x.png?id=270da12bd85a8b320140391c523b9d19 1000w"
sizes="(max-width: 720px) 100vw, 500px" loading="lazy" width="500"
alt="Do wszystkich klas węzłów trzeba było dodać metodę eksportu do XML" />
<figcaption><p>Do wszystkich klas węzłów trzeba było dodać metodę
eksportu do XML. Z wdrażaniem zmian wiązało się ryzyko wprowadzenia
błędu do aplikacji.</p></figcaption>
</figure>

Architekt systemu miał również wątpliwości co do sensu umieszczania kodu
eksportu do XML w klasach węzłów. Przecież głównym zadaniem tych klas
jest działanie na danych geograficznych, a obecność kodu eksportu do XML
wyglądałaby nie na miejscu.

Istniał też jeszcze jeden powód odmowy. Było bowiem bardzo możliwe, że
po implementacji tej funkcjonalności, ktoś z działu marketingu
poprosiłby o dodanie możliwości eksportu do jakiegoś innego formatu,
lub o inne dziwactwa. Zmusiłoby to ciebie do ponownych zmian w tych
drogocennych, delikatnych klasach.
:::

::: {.section .solution}
##  Rozwiązanie {#solution}

Wzorzec projektowy Odwiedzający proponuje umieszczenie nowych
obowiązków w osobnej klasie zwanej *odwiedzającym*, zamiast próbować
zintegrować je z istniejącymi klasami. Pierwotny obiekt, który miał
wykonywać te obowiązki, teraz jest przekazywany do jednej z metod
odwiedzającego w charakterze argumentu. Daje to metodzie dostęp do
wszystkich potrzebnych danych znajdujących się w obiekcie.

Ale co jeśli te czynności można wykonać także na obiektach-węzłach
innych klas? W naszym przykładzie z eksportem do XML, faktyczna
implementacja zapewne będzie się nieco różnić pomiędzy poszczególnymi
klasami węzłów. Dlatego też klasa odwiedzający może definiować nie
jedną, ale cały zestaw metod, z których każda przyjmuje argumenty
różnych typów, jak na poniższym przykładzie:

<figure class="code">
<pre class="code" lang="pseudocode"><code>class ExportVisitor implements Visitor is
    method doForCity(City c) { ... }
    method doForIndustry(Industry f) { ... }
    method doForSightSeeing(SightSeeing ss) { ... }
    // ...</code></pre>
</figure>

Ale jak właściwie wywoływalibyśmy te metody, zwłaszcza mając do
czynienia z całym grafem? Metody z zestawu mają różne sygnatury, więc
nie możemy zastosować polimorfizmu. Aby wybrać taką metodę
odwiedzającego, która będzie w stanie obsłużyć dany obiekt, trzeba znać
jego klasę. Brzmi koszmarnie, prawda?

<figure class="code">
<pre class="code" lang="pseudocode"><code>foreach (Node node in graph)
    if (node instanceof City)
        exportVisitor.doForCity((City) node)
    if (node instanceof Industry)
        exportVisitor.doForIndustry((Industry) node)
    // ...
}</code></pre>
</figure>

Być może zastanawiasz się, dlaczego nie zastosujemy w tym miejscu
przeciążania metod? Polegałoby to na nadaniu wszystkim metodom tej samej
nazwy, mimo że przyjmują inne parametry. Niestety, nawet zakładając że
stosowany język programowania posiada tę funkcjonalność (jak Java i
C#), w niczym nam to nie pomoże. Klasa konkretnego obiektu węzła jest
zawczasu nieznana, więc mechanizm przeciążania nie będzie w stanie
określić właściwej metody którą trzeba wywołać. Domyślnym działaniem w
takim przypadku będzie wywołanie metody która przyjmuje obiekt klasy
bazowej `Węzeł`.

Wzorzec projektowy Odwiedzający odwołuje się właśnie do takiego
problemu. Stosuje technikę zwaną [Podwójną
dyspozycją](visitor-double-dispatch.html), która pozwala wywołać
odpowiednią metodę obiektu bez uciekania się do instrukcji warunkowych.
Zamiast pozwalać klientowi wybrać odpowiednią wersję metody, można
oddelegować ten wybór samym obiektom przekazywanym odwiedzającemu w
charakterze argumentu. Skoro obiekty wiedzą jakiej są klasy, będą w
stanie wybrać właściwą metodę odwiedzającego w naturalniejszy sposób.
"Przyjmują" one odwiedzającego i informują którą jego metodę należy
wywołać.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// Kod klienta
foreach (Node node in graph)
    node.accept(exportVisitor)

// Miasto
class City is
    method accept(Visitor v) is
        v.doForCity(this)
    // ...

// Przemysł
class Industry is
    method accept(Visitor v) is
        v.doForIndustry(this)
    // ...</code></pre>
</figure>

Przyznaję --- jednak musieliśmy zmienić klasy węzłów. Ale za to zmiana
jest trywialna i umożliwia dalsze ewentualne dodawanie obowiązków już
bez konieczności ponownej zmiany klas.

Jeśli uda się wyekstrahować wspólny interfejs wszystkich odwiedzających,
wszystkie istniejące węzły będą mogły współdziałać z dowolnym
odwiedzającym jakiego dodamy do aplikacji. Jeśli będzie trzeba
wprowadzić jakieś nowe czynności związane z węzłami, wystarczy
zaimplementować nową klasę odwiedzającego.
:::

::: {.section .analogy}
##  Analogia do prawdziwego życia {#analogy}

<figure class="image">
<img
src="../../images/patterns/content/visitor/visitor-comic-17f02.png?id=7ee4fa8800f7c4df4e1aa3b1aca2b7f1"
style="aspect-ratio: 2;"
srcset="/images/patterns/content/visitor/visitor-comic-1.png?id=7ee4fa8800f7c4df4e1aa3b1aca2b7f1 600w,/images/patterns/content/visitor/visitor-comic-1-1.5x.png?id=c9cadd73d25cc63fce94312f3c82bfee 900w,/images/patterns/content/visitor/visitor-comic-1-2x.png?id=439032451eb49ebbcb5257f25ecee790 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Agent ubezpieczeniowy" />
<figcaption><p>Dobry agent ubezpieczeniowy jest gotów zawsze zaoferować
polisę odpowiednią do danej organizacji.</p></figcaption>
</figure>

Wyobraźmy sobie doświadczonego agenta ubezpieczeniowego który chce
pozyskać nowych klientów. Może odwiedzić wszystkie budynki danej
dzielnicy, próbując sprzedać polisy każdemu kogo napotka. Zależnie od
rodzaju organizacji znajdującej się w budynku, może zaoferować
specjalistyczne polisy:

-   Jeśli jest to obiekt mieszkalny, sprzedaje ubezpieczenie zdrowotne.
-   Jeśli jest to bank, sprzedaje ubezpieczenie od kradzieży.
-   Jeśli jest to kawiarnia, sprzedaje ubezpieczenie od ognia i wody.
:::

::::: {.section .structure-container}
##  Struktura {#structure}

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/visitor/structure-pla85d.png?id=c3faa0941fab7c6c18cb4dc5e43a5eb5"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 0.96;"
srcset="/images/patterns/diagrams/visitor/structure-pl.png?id=c3faa0941fab7c6c18cb4dc5e43a5eb5 520w,/images/patterns/diagrams/visitor/structure-pl-1.5x.png?id=4cb77402a64a510a3b59472078ebaef2 780w,/images/patterns/diagrams/visitor/structure-pl-2x.png?id=a3ab6ee43fc959e4687b54ad4dfe44ab 1040w"
sizes="(max-width: 720px) 100vw, 520px" loading="lazy" width="520"
alt="Struktura wzorca projektowego Odwiedzający" /><img
src="../../images/patterns/diagrams/visitor/structure-pl-indexede18b.png?id=587ca74fbb7c04e8655d53392f60e0a9"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 0.93;"
srcset="/images/patterns/diagrams/visitor/structure-pl-indexed.png?id=587ca74fbb7c04e8655d53392f60e0a9 520w,/images/patterns/diagrams/visitor/structure-pl-indexed-1.5x.png?id=e32c9aacc3c5c907f41258613d98b81f 780w,/images/patterns/diagrams/visitor/structure-pl-indexed-2x.png?id=14dedbfc2582545c8d98b3fb1d290879 1040w"
sizes="(max-width: 720px) 100vw, 520px" loading="lazy" width="520"
alt="Struktura wzorca projektowego Odwiedzający" />
</figure>
:::

1.  Interfejs **Odwiedzający** deklaruje zestaw metod odwiedzania które
    przyjmują w charakterze argumentów konkretne elementy struktury
    obiektu. Metody te mogą mieć takie same nazwy jeśli stosowany jest
    język programowania obsługujący przeciążanie, ale typ parametrów
    musi być różny.

2.  Każdy **Konkretny Odwiedzający** implementuje wiele wersji tego
    samego zachowania, dostosowane do różnych konkretnych klas
    elementów.

3.  Interfejs **Element** deklaruje metodę służącą "przyjmowaniu"
    odwiedzających. Typem przyjmowanego parametru takiej metody powinien
    być interfejs odwiedzającego.

4.  Każdy **Konkretny Element** musi implementować metodę przyjmowania.
    Zadaniem tej metody jest przekierowanie wywołania do właściwej
    metody odwiedzającego, odpowiadającej bieżącej klasie elementu.
    Trzeba pamiętać, że nawet jeśli bazowa klasa elementu
    implementuje tę metodę, wszystkie podklasy muszą ją nadpisywać w
    swoich klasach i wywoływać stosowną metodę obiektu odwiedzającego.

5.  **Klient** to na ogół kolekcja lub inny złożony obiekt (na przykład
    drzewo [Kompozytowe](composite.html)). Na ogół klienci nie są
    świadomi wszystkich konkretnych klas swoich elementów, gdyż
    współpracują z nimi za pośrednictwem jakiegoś abstrakcyjnego
    interfejsu.
::::
:::::

::: {.section .pseudocode}
##  Pseudokod {#pseudocode}

W poniższym przykładzie, wzorzec **Odwiedzający** służy wyposażaniu
hierarchii klas figur geometrycznych we wsparcie eksportu do XML.

<figure class="image">
<img
src="../../images/patterns/diagrams/visitor/example9405.png?id=d66acd1b9096c47db17ab3bb82b54a59"
style="aspect-ratio: 1.14;"
srcset="/images/patterns/diagrams/visitor/example.png?id=d66acd1b9096c47db17ab3bb82b54a59 560w,/images/patterns/diagrams/visitor/example-1.5x.png?id=534425a20f1128cb81f418cbee30cba8 840w,/images/patterns/diagrams/visitor/example-2x.png?id=f44438b98f13fcb50898baefad67ffff 1120w"
sizes="(max-width: 720px) 100vw, 560px" loading="lazy" width="560"
alt="Struktura przykładu użycia wzorca Odwiedzający" />
<figcaption><p>Eksport różnych typów obiektów do formatu XML za pomocą
obiektu odwiedzającego.</p></figcaption>
</figure>

<figure class="code">
<pre class="code" lang="pseudocode"><code>// Interfejs elementu deklaruje metodę `accept` która przyjmuje
// argument typu interfejs bazowy odwiedzającego.
interface Shape is
    method move(x, y)
    method draw()
    method accept(v: Visitor)

// Każda konkretna klasa elementu musi implementować metodę
// `accept` w taki sposób, aby wywoływała tę metodę
// odwiedzającego, która odpowiada klasie elementu.
class Dot implements Shape is
    // ...

    // Zwróć uwagę, że wywołujemy `visitDot`, a więc metodę
    // zgodną z nazwą bieżącej klasy. Dzięki temu informujemy
    // odwiedzającego o klasie elementu z jakim współpracuje.
    method accept(v: Visitor) is
        v.visitDot(this)

class Circle implements Shape is
    // ...
    method accept(v: Visitor) is
        v.visitCircle(this)

class Rectangle implements Shape is
    // ...
    method accept(v: Visitor) is
        v.visitRectangle(this)

class CompoundShape implements Shape is
    // ...
    method accept(v: Visitor) is
        v.visitCompoundShape(this)


// Interfejs odwiedzającego deklaruje zestaw metod służących
// odwiedzaniu, które odpowiadają poszczególnym klasom
// elementów. Sygnatura metody odwiedzającej pozwala
// odwiedzającemu określić dokładną klasę elementu z jakim ma do
// czynienia.
interface Visitor is
    method visitDot(d: Dot)
    method visitCircle(c: Circle)
    method visitRectangle(r: Rectangle)
    method visitCompoundShape(cs: CompoundShape)

// Konkretni odwiedzający implementują wiele wersji tego samego
// algorytmu, które mogą działać ze wszystkimi konkretnymi
// klasami elementów.
//
// Zaobserwuje się najwięcej korzyści ze stosowania wzorca
// Odwiedzający, gdy ma się do czynienia ze złożoną strukturą
// obiektów, taką jak drzewo kompozytowe. W takim przypadku
// pomocne może być przechowanie jakiegoś pośredniego stanu
// algorytmu w czasie uruchamiania metod odwiedzającego na
// kolejnych obiektach struktury.
class XMLExportVisitor implements Visitor is
    method visitDot(d: Dot) is
        // Eksportuj ID i współrzędne środka kropki.

    method visitCircle(c: Circle) is
        // Eksportuj ID okręgu, współrzędne środka i promień.

    method visitRectangle(r: Rectangle) is
        // Eksportuj ID prostokąta, współrzędne lewego górnego
        // wierzchołka, szerokość i wysokość.

    method visitCompoundShape(cs: CompoundShape) is
        // Eksportuj ID figury oraz listę ID składających się na
        // nią.


// Kod klienta może uruchamiać działania odwiedzającego na
// dowolnym zestawie elementów bez konieczności ustalania ich
// konkretnych klas. Operacja przyjmująca kieruje wywołanie do
// odpowiedniego działania obiektu odwiedzającego.
class Application is
    field allShapes: array of Shapes

    method export() is
        exportVisitor = new XMLExportVisitor()

        foreach (shape in allShapes) do
            shape.accept(exportVisitor)</code></pre>
</figure>

Jeśli zastanawiasz się nad celowością metody `przyjmującej` w tym
przykładzie, mój artykuł [Odwiedzający i podwójna
dyspozycja](visitor-double-dispatch.html) szczegółowo tłumaczy tę
kwestię.
:::

:::::::::: {.section .applicability-container}
##  Zastosowanie {#applicability}

::::::::: applicability
::: applicability-problem
Stosuj wzorzec Odwiedzający gdy istnieje potrzeba wykonywania jakiegoś
działania na wszystkich elementach złożonej struktury obiektów (jak
drzewo obiektów).
:::

::: applicability-solution
Wzorzec Odwiedzający pozwala wykonać jakieś działanie na zestawie
obiektów różnych klas dzięki istnieniu obiektu odwiedzającego. On z
kolei implementuje wiele wariantów tego działania, odpowiadających
poszczególnym klasom docelowym.
:::

::: applicability-problem
Stosowanie Odwiedzającego pozwala uprzątnąć logikę biznesową czynności
pomocniczych.
:::

::: applicability-solution
Wzorzec Odwiedzający daje możliwość ograniczenia zakresu obowiązków
głównych klas aplikacji tylko do tych najważniejszych poprzez ekstrakcję
wszystkich innych obowiązków do zestawu klas odwiedzających.
:::

::: applicability-problem
Warto stosować ten wzorzec gdy jakieś zachowanie ma sens tylko w
kontekście niektórych klas wchodzących w skład hierarchii klas, ale nie
wszystkich.
:::

::: applicability-solution
Możesz wyekstrahować główne obowiązki do osobnej klasy odwiedzający i
zaimplementować tylko te metody odwiedzania, które przyjmują obiekty
istotnych klas, zaś resztę metod pozostawić pustą.
:::
:::::::::
::::::::::

::: {.section .checklist}
##  Jak zaimplementować {#checklist}

1.  Zadeklaruj interfejs odwiedzający z zestawem metod "odwiedzania", po
    jednej na każdą konkretną klasę elementu istniejącą w programie.

2.  Zadeklaruj interfejs elementu. Jeśli pracujesz z istniejącym
    elementem hierarchii klas, dodaj abstrakcyjną metodę
    "przyjmującą" do klasy bazowej hierarchii. Metodzie tej będzie się
    przekazywać w charakterze argumentu obiekt odwiedzającego.

3.  Zaimplementuj metody przyjmujące we wszystkich konkretnych klasach
    elementów. Metody te muszą przekierowywać wywołanie do tej metody
    odwiedzania otrzymanego obiektu odwiedzającego, która odpowiada
    klasie bieżącego elementu.

4.  Klasy elementów powinny współpracować z odwiedzającymi wyłącznie za
    pośrednictwem interfejsu odwiedzającego. Odwiedzający jednak muszą
    być świadomi wszystkich klas konkretnych elementów, do których
    odnoszą się typy parametrów metod odwiedzających.

5.  Dla każdego zachowania którego nie da się zaimplementować w obrębie
    hierarchii elementów, należy utworzyć nową konkretną klasę
    odwiedzającego i zaimplementować wszystkie metody odwiedzania.

    Możesz natknąć się na sytuację w której odwiedzający będzie
    potrzebował dostępu do jakichś prywatnych pól klasy elementu. W
    takim przypadku można albo uczynić te pola i metody publicznymi,
    psując jednak tym samym hermetyzację elementu, albo zagnieździć
    klasę odwiedzającego w klasie elementu. To drugie możliwe jest tylko
    wtedy gdy (szczęśliwie) masz do czynienia z językiem programowania
    obsługującym zagnieżdżanie klas.

6.  Klient musi tworzyć obiekty odwiedzającego i przekazywać je
    elementom za pośrednictwem metod "przyjmujących".
:::

:::::: {.section .pros-cons}
##  Zalety i wady {#pros-cons}

::::: row
::: col-sm-6
-    *Zasada otwarte/zamknięte*. Pozwala wprowadzać nowe zachowanie
    odnoszące się do obiektów różnych klas bez konieczności zmiany tych
    klas.
-    *Zasada pojedynczej odpowiedzialności*. Można przenieść kilka
    wersji danego zachowania do jednej klasy.
-    Obiekt odwiedzający może zebrać użyteczne informacje
    współpracując z różnymi obiektami. Może się to przydać, gdy
    zaistnieje potrzeba przejrzenia złożonej struktury danych element po
    elemencie (takiej jak drzewo obiektów) i zastosowania
    odwiedzającego do każdego obiektu struktury.
:::

::: col-sm-6
-    Trzeba zaktualizować wszystkich odwiedzających za każdym razem gdy
    hierarchia elementów zyskuje nową klasę lub którąś traci.
-    Odwiedzający mogą nie mieć dostępu do prywatnych pól i metod
    elementów z którymi mają współpracować.
:::
:::::
::::::

::: {.section .relations}
##  Powiązania z innymi wzorcami {#relations}

-   Wzorzec [Odwiedzający](visitor.html) można traktować jak
    potężniejszą wersję [Polecenia](command.html). Jego obiekty mogą
    wykonywać różne polecenia na obiektach różnych klas.

-   [Odwiedzający](visitor.html) może wykonać działanie na całym drzewie
    [Kompozytowym](composite.html).

-   Połączenie [Odwiedzającego](visitor.html) z
    [Iteratorem](iterator.html) może służyć sekwencyjnemu przeglądowi
    elementów złożonej struktury danych i wykonaniu na nich jakiegoś
    działania, nawet jeśli te elementy są obiektami różnych klas.
:::

::: {.section .implementations}
##  Przykłady kodu {#implementations}

[![Odwiedzający w języku
C#](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](visitor/csharp/example.html "Odwiedzający w języku C#"){.prog-lang-link}
[![Odwiedzający w języku
C++](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](visitor/cpp/example.html "Odwiedzający w języku C++"){.prog-lang-link}
[![Odwiedzający w języku
Go](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](visitor/go/example.html "Odwiedzający w języku Go"){.prog-lang-link}
[![Odwiedzający w języku
Java](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](visitor/java/example.html "Odwiedzający w języku Java"){.prog-lang-link}
[![Odwiedzający w języku
PHP](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](visitor/php/example.html "Odwiedzający w języku PHP"){.prog-lang-link}
[![Odwiedzający w języku
Python](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](visitor/python/example.html "Odwiedzający w języku Python"){.prog-lang-link}
[![Odwiedzający w języku
Ruby](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](visitor/ruby/example.html "Odwiedzający w języku Ruby"){.prog-lang-link}
[![Odwiedzający w języku
Rust](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](visitor/rust/example.html "Odwiedzający w języku Rust"){.prog-lang-link}
[![Odwiedzający w języku
Swift](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](visitor/swift/example.html "Odwiedzający w języku Swift"){.prog-lang-link}
[![Odwiedzający w języku
TypeScript](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](visitor/typescript/example.html "Odwiedzający w języku TypeScript"){.prog-lang-link}
:::

::: {.section .extras}
##  Dodatek {#extras}

-   Jeśli nie rozumiesz dlaczego nie możemy po prostu stosować
    przeciążania metod zamiast wprowadzać wzorzec Odwiedzający,
    przeczytaj mój artykuł [Odwiedzający i podwójna
    dyspozycja](visitor-double-dispatch.html) aby zapoznać się ze
    szczegółami.
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

[Odwiedzający i podwójna dyspozycja []{.fa
.fa-arrow-right}](visitor-double-dispatch.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Wróć

[[]{.fa .fa-arrow-left} Metoda szablonowa](template-method.html){.btn
.btn-default rel="prev"}
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
