:::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [Wzorce
projektowe](../design-patterns.html){.type} / [Wzorce
kreacyjne](creational-patterns.html){.type}
:::

# Budowniczy {#budowniczy .title}

::: aka
Znany też jako: [Builder]{style="display:inline-block"}
:::

::: {.section .intent}
##  Cel {#intent}

**Budowniczy** jest kreacyjnym wzorcem projektowym, który daje możliwość
tworzenia złożonych obiektów etapami, krok po kroku. Wzorzec ten pozwala
produkować różne typy oraz reprezentacje obiektu używając tego samego
kodu konstrukcyjnego.

<figure class="image">
<img
src="../../images/patterns/content/builder/builder-pl4052.png?id=4700bac244168ecfba12ba17906c3041"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/builder/builder-pl.png?id=4700bac244168ecfba12ba17906c3041 640w,/images/patterns/content/builder/builder-pl-1.5x.png?id=cdfe885eda8d2dfaee4cdcc2dab1ef0f 960w,/images/patterns/content/builder/builder-pl-2x.png?id=8203e466c5f14d2080c8857b47be8611 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Wzorzec projektowy Budowniczy" />
</figure>
:::

::: {.section .problem}
##  Problem

Wyobraź sobie jakiś skomplikowany obiekt, którego inicjalizacja jest
pracochłonnym, wieloetapowym procesem obejmującym wiele pól i obiektów
zagnieżdżonych. Taki kod inicjalizacyjny jest często wrzucany do
wielgachnego konstruktora, przyjmującego mnóstwo parametrów. Albo
jeszcze gorzej: kod taki rozrzucono po całym kodzie klienckim.

<figure class="image">
<img
src="../../images/patterns/diagrams/builder/problem16274.png?id=11e715c5c97811f848c48e0f399bb05e"
style="aspect-ratio: 1.71;"
srcset="/images/patterns/diagrams/builder/problem1.png?id=11e715c5c97811f848c48e0f399bb05e 600w,/images/patterns/diagrams/builder/problem1-1.5x.png?id=a46778018c4f0f4bbf2357940c1f5f40 900w,/images/patterns/diagrams/builder/problem1-2x.png?id=02ffbd7ad294600e42aa78989d441b4d 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Zbyt wiele podklas to kolejny problem." />
<figcaption><p>Program może stać się nadmiernie skomplikowany, jeśli
każda możliwa konfiguracja oznacza dodanie
nowej podklasy.</p></figcaption>
</figure>

Na przykład pomyślmy, jak stworzyć obiekt `Dom`. Do zbudowania
najprostszego domu wystarczą cztery ściany i podłoga. Do tego drzwi,
parę okien i dach. Ale co, jeśli chcesz większy, jaśniejszy dom z
podwórkiem i innymi dodatkami (ogrzewanie, kanalizacja, elektryczność)?

Najprostsze rozwiązanie to rozszerzenie klasy bazowej `Dom` i stworzenie
zestawu podklas, które spełniałyby każdy możliwy zestaw wymogów. Ale
takie podejście doprowadzi do wielkiej liczby podklas. Dodanie kolejnego
parametru, jak styl werandy, jeszcze bardziej rozbuduje tę hierarchię.

Istnieje jednak inne rozwiązanie, które nie wiąże się z mnożeniem
podklas. Można stworzyć jeden wielki konstruktor w klasie bazowej `Dom`,
uwzględniający wszystkie możliwe parametry, które sterują obiektem typu
dom. W ten sposób nie mnożymy liczby klas, ale tworzymy nieco inny
problem.

<figure class="image">
<img
src="../../images/patterns/diagrams/builder/problem27ab5.png?id=2e91039b6c7d2d2df6ee519983a3b036"
style="aspect-ratio: 1.71;"
srcset="/images/patterns/diagrams/builder/problem2.png?id=2e91039b6c7d2d2df6ee519983a3b036 600w,/images/patterns/diagrams/builder/problem2-1.5x.png?id=3d302cf2762fd04d70cbb91cb54e923c 900w,/images/patterns/diagrams/builder/problem2-2x.png?id=5e7975a91c0e4f4ba960f908cc9c2ea2 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Konstruktor teleskopowy" />
<figcaption><p>Konstruktor przyjmujący mnóstwo parametrów ma swoją wadę:
nie wszystkie parametry będą potrzebne za każdym razem.</p></figcaption>
</figure>

W większości przypadków parametry pozostaną nieużyte, a [wywołania
konstruktora będą wyglądać
niechlujnie](../smells/long-parameter-list.html). Na przykład tylko
niektóre domy mają basen, więc parametry dotyczące basenu w
dziewięciu na dziesięć przypadków będą niepotrzebne.
:::

::: {.section .solution}
##  Rozwiązanie {#solution}

Wzorzec projektowy Budowniczy proponuje ekstrakcję kodu konstrukcyjnego
obiektu z jego klasy i umieszczenie go w osobnych obiektach zwanych
*budowniczymi*.

<figure class="image">
<img
src="../../images/patterns/diagrams/builder/solution13e5f.png?id=8ce82137f8935998de802cae59e00e11"
style="aspect-ratio: 1.46;"
srcset="/images/patterns/diagrams/builder/solution1.png?id=8ce82137f8935998de802cae59e00e11 410w,/images/patterns/diagrams/builder/solution1-1.5x.png?id=8ab77eb73760a75c35940bae243d43f2 615w,/images/patterns/diagrams/builder/solution1-2x.png?id=a9c2ab02f0b2aca1a7512022194dd113 820w"
sizes="(max-width: 720px) 100vw, 410px" loading="lazy" width="410"
alt="Zastosowanie wzorca budowniczy" />
<figcaption><p>Wzorzec Budowniczy pozwala konstruować złożone obiekty
krok po kroku. Budowniczy ponadto nie pozwala na dostęp do nich innym
obiektom, dopóki nie zostaną ukończone.</p></figcaption>
</figure>

Ten wzorzec projektowy dzieli konstrukcję obiektu na pewne etapy
(`budujŚciany`, `wstawDrzwi`, itd.). Aby powołać do życia obiekt,
wykonuje się ciąg takich etapów za pośrednictwem obiektu-budowniczego.
Istotne jest to, że nie musisz wywoływać wszystkich etapów. Możesz
bowiem ograniczyć się tylko do tych kroków, które są niezbędne do
określenia potrzebnej nam konfiguracji obiektu.

Niektóre etapy konstrukcji mogą wymagać odmiennych implementacji,
zależnie od potrzebnej w danej chwili reprezentacji produktu. Na
przykład, ściany leśnej chatki mogą być drewniane, ale mury zamku
warownego --- kamienne.

W takim przypadku, można utworzyć wiele różnych klas budowniczych które
implementują te same etapy konstrukcji, ale w różny sposób. Można
następnie korzystać z tych budowniczych podczas procesu konstrukcji (np.
odpowiednia kolejność wywołań etapów budowy) aby wytworzyć różne rodzaje
obiektów.

<figure class="image">
<img
src="../../images/patterns/content/builder/builder-comic-1-pl37af.png?id=317323a7cd8961e1cb5b8da717cb20ea"
style="aspect-ratio: 2;"
srcset="/images/patterns/content/builder/builder-comic-1-pl.png?id=317323a7cd8961e1cb5b8da717cb20ea 600w,/images/patterns/content/builder/builder-comic-1-pl-1.5x.png?id=96fbd68f06696e7f877d86d9f7f4818a 900w,/images/patterns/content/builder/builder-comic-1-pl-2x.png?id=7bc938c9c8b7e62587c11ddd22591ef2 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600" />
<figcaption><p>Różni budowniczowie wykonują to samo zadanie w
różny sposób.</p></figcaption>
</figure>

Przykładowo, wyobraź sobie budowniczego, który konstruuje wyłącznie z
drewna i szkła, drugi zaś stosuje tylko kamień i żelazo, a trzeci ---
złoto i diamenty. Wywołując te same etapy, uzyskasz zwykły dom autorstwa
pierwszego budowniczego, drugi z nich zbuduje mały zamek, a trzeci ---
pałac. Jednakże, to zadziała tylko pod warunkiem, że kod kliencki, który
wywołuje etapy budowy, jest w stanie komunikować się z budowniczymi za
pośrednictwem wspólnego interfejsu.

#### Kierownik

Można pójść jeszcze dalej i przenieść kolejkę bezpośrednich wywołań
budowniczego do osobnej klasy, zwanej *kierownikiem*. Kierownik określa
kolejność etapów jaką musi zachować budowniczy, który z kolei
implementuje te etapy konstrukcji obiektu.

<figure class="image">
<img
src="../../images/patterns/content/builder/builder-comic-2-pl8903.png?id=0b56315ebd2dc5f4e388f2c0373ba6fb"
style="aspect-ratio: 1.14;"
srcset="/images/patterns/content/builder/builder-comic-2-pl.png?id=0b56315ebd2dc5f4e388f2c0373ba6fb 343w,/images/patterns/content/builder/builder-comic-2-pl-1.5x.png?id=25c7da670f232acf62729f017d0dcd65 515w,/images/patterns/content/builder/builder-comic-2-pl-2x.png?id=66c4f4e4af9a9c3dda6cfd812ab0e985 687w"
sizes="(max-width: 720px) 100vw, 343px" loading="lazy" width="343" />
<figcaption><p>Kierownik wie jakie kroki należy wykonać, aby otrzymać
działający produkt.</p></figcaption>
</figure>

Posiadanie w programie klasy kierownika nie jest niezbędne. Można bowiem
zawsze wywoływać etapy konstrukcji w odpowiedniej kolejności z poziomu
kodu klienckiego. Jednakże, kierownik może okazać się dobrym miejscem na
umieszczenie czynności konstrukcyjnych, potrzebnych w innych miejscach
programu.

Dodatkowo, klasa kierownik ukrywa szczegóły konstrukcji produktu przed
kodem klienckim. Klient musi tylko skojarzyć budowniczego z
kierownikiem, wywołać proces budowy za pośrednictwem tego pierwszego, a
następnie odebrać wynik pracy od drugiego.
:::

::::: {.section .structure-container}
##  Struktura {#structure}

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/builder/structure05e4.png?id=fe9e23559923ea0657aa5fe75efef333"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 0.79;"
srcset="/images/patterns/diagrams/builder/structure.png?id=fe9e23559923ea0657aa5fe75efef333 460w,/images/patterns/diagrams/builder/structure-1.5x.png?id=91ea8cd3137b403542c23abf63938f00 690w,/images/patterns/diagrams/builder/structure-2x.png?id=dca1b1508e23c266cbedc80ffb84311a 920w"
sizes="(max-width: 720px) 100vw, 460px" loading="lazy" width="460"
alt="Struktura wzorca projektowego Budowniczy" /><img
src="../../images/patterns/diagrams/builder/structure-indexedadac.png?id=44b3d763ce91dbada5d8394ef777437f"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 0.81;"
srcset="/images/patterns/diagrams/builder/structure-indexed.png?id=44b3d763ce91dbada5d8394ef777437f 470w,/images/patterns/diagrams/builder/structure-indexed-1.5x.png?id=d57a6ff9342ea31736ea98e5066e0748 705w,/images/patterns/diagrams/builder/structure-indexed-2x.png?id=153eed9a51784cbe00d0ca8b3d6b268d 940w"
sizes="(max-width: 720px) 100vw, 470px" loading="lazy" width="470"
alt="Struktura wzorca projektowego Budowniczy" />
</figure>
:::

1.  Interfejs **Budowniczego** deklaruje etapy konstrukcji produktu
    wspólne dla wszystkich typów budowniczych.

2.  **Konkretni Budowniczowie** zapewniają różne implementacje etapów
    konstrukcji. Konkretni budowniczowie mogę tworzyć produkty które nie
    mają wspólnego interfejsu.

3.  **Produkty** to powstałe obiekty. Produkty konstruowane przez
    różnych budowniczych nie muszą należeć do tej samej hierarchii klas,
    czy interfejsu.

4.  Klasa **Kierownik** definiuje kolejność w jakiej należy wywołać
    etapy konstrukcyjne, aby móc stworzyć i następnie użyć ponownie
    określone konfiguracje produktów.

5.  **Klient** musi dopasować jeden z obiektów budowniczych do
    kierownika. Zazwyczaj robi się to tylko raz, za pośrednictwem
    parametru przekazywanego do konstruktora kierownika. Następnie
    kierownik za pomocą obiektu budowniczego wykonuje dalszą
    konstrukcję. Jednakże istnieje alternatywne podejście w przypadku
    przekazania obiektu budowniczego metodzie produkcyjnej kierownika. W
    takim przypadku kierownik może skorzystać z różnych budowniczych.
::::
:::::

::: {.section .pseudocode}
##  Pseudokod {#pseudocode}

Poniższy przykład użycia wzorca projektowego **Budowniczy** pokazuje,
jak można ponownie wykorzystać ten sam kod konstrukcyjny obiektu budując
różne typy produktów, takie jak samochody oraz odpowiednie do nich
instrukcje obsługi.

<figure class="image">
<img
src="../../images/patterns/diagrams/builder/example-pl1cd3.png?id=3d7d22ad950d7852ca6f8cf868e5a7fa"
style="aspect-ratio: 0.85;"
srcset="/images/patterns/diagrams/builder/example-pl.png?id=3d7d22ad950d7852ca6f8cf868e5a7fa 500w,/images/patterns/diagrams/builder/example-pl-1.5x.png?id=26c736f06d9575348bf86e806b7e4c9f 750w,/images/patterns/diagrams/builder/example-pl-2x.png?id=e5dc9a4c159bd22eca79f44e8af342d7 1000w"
sizes="(max-width: 720px) 100vw, 500px" loading="lazy" width="500"
alt="Struktura przykładu użycia wzorca Budowniczy" />
<figcaption><p>Przykład kilkuetapowej konstrukcji samochodów oraz
instrukcji obsługi ich modeli.</p></figcaption>
</figure>

Samochód jest skomplikowanym obiektem, który można skonstruować na setki
sposobów. Zamiast obciążać klasę `Samochód` olbrzymim konstruktorem,
wyekstrahowaliśmy kod montażu auta do osobnej klasy budowniczego
samochodu. Klasa ta ma zestaw metod pozwalających skonfigurować dowolną
część auta.

Jeśli kod kliencki musi utworzyć specjalny model samochodu na
zamówienie, może skorzystać bezpośrednio z budowniczego. Z drugiej
strony, klient może oddelegować montaż klasie kierownika, która wie
jak za pomocą budowniczego skonstruować wiele najpopularniejszych modeli
aut.

Może cię to zaskoczyć, ale do każdego auta powinna istnieć instrukcja
obsługi (poważnie? ktoś je czyta?). Instrukcje opisują cechy i
wyposażenie samochodów, więc ich zawartość będzie się różnić pomiędzy
modelami. Dlatego warto ponownie wykorzystywać istniejący proces
konstrukcyjny zarówno dla aut, jak i dla ich instrukcji. Oczywiście
tworzenie instrukcji to nie to samo, co montaż auta i dlatego musimy
dodać kolejną klasę budowniczego specjalizującą się w tworzeniu
instrukcji. Klasa ta implementuje takie same metody budowy, co jej
montująca auta krewna, ale zamiast montować --- opisuje. Poprzez
przekazanie tych budowniczych do tego samego obiektu kierownika,
konstruujemy albo pojazd, albo instrukcję obsługi.

Ostatni etap to pobranie nowo utworzonego obiektu. Metalowy samochód i
papierowa instrukcja obsługi, to jednak bardzo różne rzeczy, choć ze
sobą związane. Nie możemy umieścić metody pobierającej wynik pracy w
kierowniku, zanim nie powiążemy go z konkretną klasą produktów. Dlatego
też odbieramy wynik u budowniczego, który jest jego autorem.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// Stosowanie wzorca Budowniczy ma sens tylko gdy produkty w
// programie są złożone i wymagają kompleksowej konfiguracji.
// Poniższe dwa produkty są powiązane, ale nie mają wspólnego
// interfejsu.
class Car is
    // Samochód może mieć GPS, komputer pokładowy i pewną liczbę
    // siedzeń. Różne modele samochodów (sportowe, SUV-y,
    // kabriolety) mogą mieć różne opcjonalne funkcjonalności.

class Manual is
    // Do każdego samochodu powinna istnieć instrukcja obsługi,
    // która opisuje jego konfigurację i funkcje.


// Interfejs budowniczy określa metody służące tworzeniu
// poszczególnych części obiektów-produktów.
interface Builder is
    method reset()
    method setSeats(...)
    method setEngine(...)
    method setTripComputer(...)
    method setGPS(...)

// Konkretne klasy budowniczy są zgodne pod względem interfejsu
// i zawierają szczegółowe implementacje etapów budowania. Twój
// program może mieć wiele różnie zaimplementowanych wariacji
// budowniczych.
class CarBuilder implements Builder is
    private field car:Car

    // Nowo utworzona instancja budowniczego powinna zawierać
    // pusty obiekt-produkt, który będzie podstawą do dalszej
    // budowy.
    constructor CarBuilder() is
        this.reset()

    // Metoda resetująca zeruje budowany obiekt.
    method reset() is
        this.car = new Car()

    // Wszystkie etapy produkcji działają na tej samej instancji
    // produktu.
    method setSeats(...) is
        // Ustaw ilość siedzeń w aucie.

    method setEngine(...) is
        // Zamontuj dany silnik.

    method setTripComputer(...) is
        // Zamontuj komputer pokładowy.

    method setGPS(...) is
        // Zamontuj nawigację GPS.

    // Konkretni budowniczy powinni posiadać własne metody
    // pozyskiwania wyników działań, gdyż różni budowniczowie
    // tworzą różne produkty i nie zawsze zgodne pod względem
    // interfejsu. Dlatego też nie można zadeklarować takich
    // metod w interfejsie budowniczego (a przynajmniej nie
    // można tego dokonać w statycznie typowanym języku
    // programowania).
    //
    // Zazwyczaj po zwróceniu wyniku działania klientowi,
    // instancja budowniczego powinna być gotowa na rozpoczęcie
    // produkcji od nowa. Dlatego typową praktyką jest wywołanie
    // metody resetującej na końcu kodu metody `getProduct`. Nie
    // jest to jednak obowiązkowe i można odroczyć zerowanie do
    // momentu gdy klient prześle stosowne polecenie.
    method getProduct():Car is
        product = this.car
        this.reset()
        return product

// W przeciwieństwie do innych wzorców kreacyjnych, budowniczy
// pozwala konstruować produkty które nie mają wspólnego
// interfejsu.
class CarManualBuilder implements Builder is
    private field manual:Manual

    constructor CarManualBuilder() is
        this.reset()

    method reset() is
        this.manual = new Manual()

    method setSeats(...) is
        // Udokumentuj specyfikację siedzeń.

    method setEngine(...) is
        // Dodaj instrukcję silnika.

    method setTripComputer(...) is
        // Dodaj instrukcję komputera pokładowego.

    method setGPS(...) is
        // Dodaj instrukcję nawigacji GPS.

    method getProduct():Manual is
        // Zwróć instrukcję obsługi i zresetuj budowniczego.


// Kierownik jest odpowiedzialny tylko za wywoływanie etapów
// budowy w odpowiedniej kolejności. Przydaje się to gdy
// tworzymy produkty według określonego porządku lub
// konfiguracji. Doprecyzowując — klasa kierownik jest
// opcjonalna, ponieważ klient może kontrolować budowniczych
// bezpośrednio.
class Director is
    // Kierownik może współdziałać z dowolną instancją
    // budowniczego jaką klient mu przekaże. Dzięki temu kod
    // klienta może zmienić ostateczny typ nowo utworzonego
    // produktu. Kierownik może skonstruować wiele wariacji
    // danego produktu postępując według tych samych etapów
    // budowy.
    method constructSportsCar(builder: Builder) is
        builder.reset()
        builder.setSeats(2)
        builder.setEngine(new SportEngine())
        builder.setTripComputer(true)
        builder.setGPS(true)

    method constructSUV(builder: Builder) is
        // ...


// Kod kliencki tworzy obiekt budowniczego, przekazuje go
// kierownikowi a następnie inicjuje proces konstrukcji.
// Ostateczny wynik działania pobiera się od obiektu
// budowniczego.
class Application is

    method makeCar() is
        director = new Director()

        CarBuilder builder = new CarBuilder()
        director.constructSportsCar(builder)
        Car car = builder.getProduct()

        CarManualBuilder builder = new CarManualBuilder()
        director.constructSportsCar(builder)

        // Finalny produkt zwykle pobiera się od obiektu
        // budowniczego, ponieważ kierownik nic o nim nie wie i
        // nie jest zależny od konkretnych budowniczych czy
        // konkretnych produktów.
        Manual manual = builder.getProduct()</code></pre>
</figure>
:::

:::::::::: {.section .applicability-container}
##  Zastosowanie {#applicability}

::::::::: applicability
::: applicability-problem
Stosuj wzorzec Budowniczy, aby pozbyć się "teleskopowych konstruktorów".
:::

::: applicability-solution
Załóżmy, że masz konstruktor, przyjmujący 10 opcjonalnych parametrów.
Wywołanie takiego potwora jest co najmniej niewygodne, dlatego
przeciążamy konstruktor i tworzymy wiele jego krótszych wersji,
wymagających mniej parametrów. Będą one wciąż odwoływać się do głównego
konstruktora, przekazując jakieś domyślne wartości w miejsce pominiętych
argumentów.

<figure class="code">
<pre class="code" lang="java"><code>class Pizza {
    Pizza(int size) { ... }
    Pizza(int size, boolean cheese) { ... }
    Pizza(int size, boolean cheese, boolean pepperoni) { ... }
    // ...</code></pre>
<figcaption><p>Tworzenie takich potworów jest możliwe jedynie w językach
obsługujących przeciążanie metod, a więc na przykład w C#
oraz Javie.</p></figcaption>
</figure>

Wzorzec Budowniczy pozwala konstruować obiekty krok po kroku, w miarę
jak staje się to w programie potrzebne. Po zaimplementowaniu tego
wzorca, nie musisz przekazywać konstruktorowi tuzina parametrów.
:::

::: applicability-problem
Stosuj wzorzec Budowniczy, gdy potrzebujesz możliwości tworzenia różnych
reprezentacji jakiegoś produktu (na przykład, domy z kamienia i domy z
drewna).
:::

::: applicability-solution
Wzorzec Budowniczy można użyć gdy konstruowanie różnorakich
reprezentacji produktu obejmuje podobne etapy, które różnią się jedynie
szczegółami.

Bazowy interfejs budowniczego definiuje wszelkie możliwe etapy
konstrukcji, a konkretni budowniczy implementują te kroki by móc tworzyć
poszczególne reprezentacje obiektów. Natomiast klasa kierownik pilnuje
właściwego porządku konstruowania.
:::

::: applicability-problem
Stosuj ten wzorzec do konstruowania drzew
[Kompozytowych](composite.html) lub innych złożonych obiektów.
:::

::: applicability-solution
Wzorzec budowniczego umożliwia konstrukcję w etapach. Niektóre z nich
możemy odroczyć bez szkody dla finalnego produktu. Możemy nawet
wywoływać etapy rekursywnie, co przydaje się przy budowie drzewa
obiektów.

Budowniczy uniemożliwia dostęp do nieskończonego produktu przez okres
jego konstrukcji. Zapobiega to pozyskiwaniu niekompletnych wyników przez
kod kliencki.
:::
:::::::::
::::::::::

::: {.section .checklist}
##  Jak zaimplementować {#checklist}

1.  Upewnij się, że jesteś w stanie zdefiniować konkretne, wspólne
    etapy, wykonywane przy tworzeniu wszystkich dostępnych reprezentacji
    produktu. Bez tego nie uda się wdrożyć tego wzorca.

2.  Zadeklaruj te etapy w interfejsie bazowego budowniczego.

3.  Stwórz konkretną klasę budowniczego dla każdej reprezentacji
    produktu i zaimplementuj specyficzne dla nich etapy konstrukcyjne.

    Nie zapomnij zaimplementować metodę pobierającą wynik konstrukcji.
    Powodem, dla którego taka metoda nie może być zadeklarowana w ramach
    interfejsu budowniczego jest to, że różni budowniczowie mogą tworzyć
    obiekty które nie posiadają wspólnego interfejsu. Dlatego też nie
    byłoby wiadomo jaki typ obiektu zwracałaby taka metoda. Jednakże,
    jeśli masz do czynienia wyłącznie z produktami wchodzącymi w skład
    jednej hierarchii, metodę taką można bezpiecznie dodać do interfejsu
    bazowego.

4.  Rozważ stworzenie klasy kierownika. Może ona zawrzeć różne sposoby
    konstruowania produktu przy pomocy tego samego obiektu budowniczego.

5.  Kod kliencki tworzy zarówno obiekty budowniczego, jak i kierownika.
    Przed rozpoczęciem konstrukcji, klient musi przekazać kierownikowi
    obiekt budowniczego. Zazwyczaj klient musi to zrobić jednorazowo, za
    pośrednictwem parametrów konstruktora kierownika. Kierownik potem
    wykonuje wszelkie prace konstrukcyjne za pomocą tego budowniczego.
    Jest też inny sposób, w którym budowniczy jest przekazywany
    bezpośrednio metodzie konstrukcyjnej kierownika.

6.  Wynik konstrukcji może być odebrany bezpośrednio od kierownika tylko
    wtedy, gdy wszystkie produkty współdzielą taki sam interfejs. W
    przeciwnym razie, klient powinien pobrać wynik od budowniczego.
:::

:::::: {.section .pros-cons}
##  Zalety i wady {#pros-cons}

::::: row
::: col-sm-6
-    Możesz konstruować obiekty etapami, odkładać niektóre etapy, lub
    wykonywać je rekursywnie.
-    Możesz wykorzystać ponownie ten sam kod konstrukcyjny budując
    kolejne reprezentacje produktów.
-    *Zasada pojedynczej odpowiedzialności*. Można odizolować
    skomplikowany kod konstrukcyjny od logiki biznesowej produktu.
:::

::: col-sm-6
-    Kod staje się bardziej skomplikowany, gdyż wdrożenie tego wzorca
    wiąże się z dodaniem wielu nowych klas.
:::
:::::
::::::

::: {.section .relations}
##  Powiązania z innymi wzorcami {#relations}

-   Wiele projektów zaczyna się od zastosowania [Metody
    wytwórczej](factory-method.html) (mniej skomplikowanej i dającej się
    dostosować poprzez tworzenie podklas). Projekty następnie ewoluują
    stopniowo w [Fabrykę abstrakcyjną](abstract-factory.html),
    [Prototyp](prototype.html) lub [Budowniczego](builder.html)
    (bardziej elastyczne, ale i bardziej skomplikowane wzorce).

-   [Budowniczy](builder.html) koncentruje się na konstruowaniu
    złożonych obiektów krok po kroku. [Fabryka
    abstrakcyjna](abstract-factory.html) specjalizuje się w tworzeniu
    rodzin spokrewnionych ze sobą obiektów. *Fabryka abstrakcyjna*
    zwraca produkt natychmiast, zaś *Budowniczy* pozwala dołączyć
    dodatkowe etapy konstrukcji zanim będzie można pobrać finalny
    produkt.

-   Możesz zastosować wzorzec [Budowniczy](builder.html) by tworzyć
    złożone drzewa [Kompozytowe](composite.html) dzięki możliwości
    zaprogramowania ich etapów konstrukcji tak, aby odbywały się
    rekurencyjnie.

-   Możliwe jest połączenie wzorców [Budowniczy](builder.html) i
    [Most](bridge.html): klasa *kierownik* pełni rolę abstrakcji, zaś
    poszczególni *budowniczy* stanowią *implementacje*.

-   [Fabryki abstrakcyjne](abstract-factory.html),
    [Budowniczych](builder.html) oraz [Prototypy](prototype.html) można
    zaimplementować jako [Singletony](singleton.html).
:::

::: {.section .implementations}
##  Przykłady kodu {#implementations}

[![Budowniczy w języku
C#](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](builder/csharp/example.html "Budowniczy w języku C#"){.prog-lang-link}
[![Budowniczy w języku
C++](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](builder/cpp/example.html "Budowniczy w języku C++"){.prog-lang-link}
[![Budowniczy w języku
Go](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](builder/go/example.html "Budowniczy w języku Go"){.prog-lang-link}
[![Budowniczy w języku
Java](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](builder/java/example.html "Budowniczy w języku Java"){.prog-lang-link}
[![Budowniczy w języku
PHP](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](builder/php/example.html "Budowniczy w języku PHP"){.prog-lang-link}
[![Budowniczy w języku
Python](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](builder/python/example.html "Budowniczy w języku Python"){.prog-lang-link}
[![Budowniczy w języku
Ruby](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](builder/ruby/example.html "Budowniczy w języku Ruby"){.prog-lang-link}
[![Budowniczy w języku
Rust](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](builder/rust/example.html "Budowniczy w języku Rust"){.prog-lang-link}
[![Budowniczy w języku
Swift](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](builder/swift/example.html "Budowniczy w języku Swift"){.prog-lang-link}
[![Budowniczy w języku
TypeScript](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](builder/typescript/example.html "Budowniczy w języku TypeScript"){.prog-lang-link}
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

[Prototyp []{.fa .fa-arrow-right}](prototype.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Wróć

[[]{.fa .fa-arrow-left} Porównanie Fabryk](factory-comparison.html){.btn
.btn-default rel="prev"}
:::
:::::::::::::::::::::::::::::::::

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
:::::::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::::::
