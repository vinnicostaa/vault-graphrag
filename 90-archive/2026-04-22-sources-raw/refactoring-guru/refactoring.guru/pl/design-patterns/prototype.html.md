:::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [Wzorce
projektowe](../design-patterns.html){.type} / [Wzorce
kreacyjne](creational-patterns.html){.type}
:::

# Prototyp {#prototyp .title}

::: aka
Znany też jako:
[Klon, ]{style="display:inline-block"}[Clone, ]{style="display:inline-block"}[Prototype]{style="display:inline-block"}
:::

::: {.section .intent}
##  Cel {#intent}

**Prototyp** to kreacyjny wzorzec projektowy, który umożliwia kopiowanie
już istniejących obiektów bez tworzenia zależności pomiędzy twoim
kodem, a klasami obiektów.

<figure class="image">
<img
src="../../images/patterns/content/prototype/prototype3cb9.png?id=e912b1ada20bbf7b2ffc09e93b9fab20"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/prototype/prototype.png?id=e912b1ada20bbf7b2ffc09e93b9fab20 640w,/images/patterns/content/prototype/prototype-1.5x.png?id=a0f76894fb657783b65b9ed899857468 960w,/images/patterns/content/prototype/prototype-2x.png?id=670789c80c8a114e25838ede2da4a881 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Wzorzec Projektowy Prototyp" />
</figure>
:::

::: {.section .problem}
##  Problem

Przypuśćmy, że mamy jakiś obiekt i potrzebujemy jego dokładnej kopii.
Jak można tego dokonać? Na początek, trzeba stworzyć nowy obiekt tej
samej klasy. Potem zaś skopiować zawartość pól oryginału do pól kopii.

Fajnie! Ale jest i haczyk. Nie wszystkie obiekty można w ten sposób
kopiować, bo nie wszystkie pola obiektu są ogólnodostępne --- mogą być
prywatne i tym samym niedostępne spoza samego obiektu.

<figure class="image">
<img
src="../../images/patterns/content/prototype/prototype-comic-1-ple57f.png?id=f3a37546aa7b81d26885ee3f932ceb9d"
style="aspect-ratio: 2;"
srcset="/images/patterns/content/prototype/prototype-comic-1-pl.png?id=f3a37546aa7b81d26885ee3f932ceb9d 600w,/images/patterns/content/prototype/prototype-comic-1-pl-1.5x.png?id=b9d1af1ac0a700daec6597863196a93a 900w,/images/patterns/content/prototype/prototype-comic-1-pl-2x.png?id=95f686d9a806b9e6e105403ca3678202 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Co może pójść nie tak gdy kopiujemy coś oglądając to wyłącznie z zewnątrz?" />
<figcaption><p>Kopiowanie obiektu oglądając go tylko z zewnątrz <a
href="https://vimeo.com/120816425">nie zawsze</a>
jest możliwe.</p></figcaption>
</figure>

Jest jeszcze jeden kłopot z takim bezpośrednim podejściem. Skoro musisz
wiedzieć, jakiej klasy jest obiekt który chcesz skopiować, to twój kod
staje się zależny od tej klasy. Jeśli to nie wygląda na kłopot, to weź
pod uwagę, że czasem znamy tylko interfejs obiektu, a nie jego konkretną
klasę. Przykładem tego są metody przyjmujące w charakterze parametru
obiekty zgodne z jakimś wspólnym interfejsem.
:::

::: {.section .solution}
##  Rozwiązanie {#solution}

Wzorzec projektowy Prototyp deleguje proces klonowania samym obiektom,
które mają być sklonowane. We wzorcu tym deklarowany jest wspólny
interfejs dla wszystkich obiektów wspierających funkcjonalność
klonowania. Interfejs taki pozwala klonować obiekty bez konieczności
sprzęgania kodu z klasą obiektu. Zazwyczaj taki interfejs ogranicza się
tylko do pojedynczej metody `klonuj`.

Implementacja metody `klonuj` jest bardzo podobna w poszczególnych
klasach. Metoda tworzy obiekt swojej klasy i kopiuje doń wartości jej
pól. Można wówczas skopiować także pola prywatne, gdyż większość języków
programowania pozwala obiektom na dostęp do prywatnych pól obiektów tej
samej klasy.

Obiekt, który posiada funkcjonalność klonowania zwany jest *prototypem*.
Gdy obiekty z którymi masz do czynienia mają mnóstwo pól i setki
możliwych konfiguracji, klonowanie ich może okazać się korzystną
alternatywą do tworzenia podklas.

<figure class="image">
<img
src="../../images/patterns/content/prototype/prototype-comic-2-pl156b.png?id=9f392a104e802cd5e7748df44529886c"
style="aspect-ratio: 1.14;"
srcset="/images/patterns/content/prototype/prototype-comic-2-pl.png?id=9f392a104e802cd5e7748df44529886c 343w,/images/patterns/content/prototype/prototype-comic-2-pl-1.5x.png?id=1b2135cc9f7205e09a090833d6dea2c4 515w,/images/patterns/content/prototype/prototype-comic-2-pl-2x.png?id=d2cd5dff0f3523d5cca3d2f53d694685 687w"
sizes="(max-width: 720px) 100vw, 343px" loading="lazy" width="343"
alt="Prefabrykowane prototypy" />
<figcaption><p>Prefabrykowane prototypy mogą być alternatywą do
tworzenia podklas.</p></figcaption>
</figure>

Działa to tak: tworzymy zestaw obiektów, każdy skonfigurowany na swój
sposób. Jeśli potrzebujesz obiektu, który ma taką samą konfigurację jak
już istniejący, klonujesz prototyp --- zamiast konstruować nowy
obiekt od podstaw.
:::

::: {.section .analogy}
##  Analogia do prawdziwego życia {#analogy}

W prawdziwym życiu prototypy stosuje się w testach, zanim rozpoczęta
zostanie seryjna produkcja wyrobu. Tu jednak prototypy nie biorą
udziału w faktycznej produkcji, lecz pełnią w niej rolę pasywną.

<figure class="image">
<img
src="../../images/patterns/content/prototype/prototype-comic-3-pl13e8.png?id=285e63ff53064586149ef6a595c04f6d"
style="aspect-ratio: 2;"
srcset="/images/patterns/content/prototype/prototype-comic-3-pl.png?id=285e63ff53064586149ef6a595c04f6d 600w,/images/patterns/content/prototype/prototype-comic-3-pl-1.5x.png?id=da042fddb9634787f95660505dacb5a3 900w,/images/patterns/content/prototype/prototype-comic-3-pl-2x.png?id=b451cdd1a7307ec66b2af1982f5fcb11 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Podział komórki" />
<figcaption><p>Podział żywej komórki.</p></figcaption>
</figure>

Ponieważ prototypy w przemyśle nie potrafią kopiować siebie, bliższą
analogią do prawdziwego życia będzie mitotyczny podział żywej komórki
(biologia, pamiętasz?). Po podziale mitotycznym powstaje para
identycznych komórek. Pierwotna komórka stanowi prototyp i jednocześnie
pełni aktywną rolę w duplikacji.
:::

:::::::: {.section .structure-container}
##  Struktura {#structure}

::::::: structure
#### Podstawowa implementacja

:::: prototype-normal
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/prototype/structure87f6.png?id=088102c5e9785ff45debbbce86f4df81"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.25;"
srcset="/images/patterns/diagrams/prototype/structure.png?id=088102c5e9785ff45debbbce86f4df81 500w,/images/patterns/diagrams/prototype/structure-1.5x.png?id=beec6a1a5242268e10e39f9fdc0b894b 750w,/images/patterns/diagrams/prototype/structure-2x.png?id=ba75079f42f08028ae4cdbda0cfecc26 1000w"
sizes="(max-width: 720px) 100vw, 500px" loading="lazy" width="500"
alt="Struktura wzorca projektowego Prototyp" /><img
src="../../images/patterns/diagrams/prototype/structure-indexed6499.png?id=0e1c809842f5c43aca0541a2eba1f844"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.27;"
srcset="/images/patterns/diagrams/prototype/structure-indexed.png?id=0e1c809842f5c43aca0541a2eba1f844 520w,/images/patterns/diagrams/prototype/structure-indexed-1.5x.png?id=05b072b5b83f5ff1024a7ba448ea9d59 780w,/images/patterns/diagrams/prototype/structure-indexed-2x.png?id=74584ac729c0f6b49d2a97a53cc266ff 1040w"
sizes="(max-width: 720px) 100vw, 520px" loading="lazy" width="520"
alt="Struktura wzorca projektowego Prototyp" />
</figure>
:::
::::

1.  Interfejs **Prototyp** deklaruje metody klonujące. W większości
    przypadków jest to pojedyncza metoda `klonuj`.

2.  Klasa **Konkretny Prototyp** implementuje metodę klonującą. Oprócz
    kopiowania danych pierwotnego obiektu do nowo powstałego, metoda ta
    może również rozwiązywać kwestie związane z klonowaniem obiektów
    powiązanych, czy też z zależnościami rekursywnymi.

3.  **Klient** może stworzyć kopię dowolnego obiektu który jest zgodny z
    interfejsem prototypu.

#### Implementacja rejestru prototypów

:::: prototype-pool
::: {.struct-image2 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/prototype/structure-prototype-cache9d19.png?id=609c2af5d14ed55dcbb218a00f98e7d5"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.15;"
srcset="/images/patterns/diagrams/prototype/structure-prototype-cache.png?id=609c2af5d14ed55dcbb218a00f98e7d5 550w,/images/patterns/diagrams/prototype/structure-prototype-cache-1.5x.png?id=8ca0b829185d49c5acbe19966a0659d4 825w,/images/patterns/diagrams/prototype/structure-prototype-cache-2x.png?id=a1e4514bbcc9b10968b856f19b407105 1100w"
sizes="(max-width: 720px) 100vw, 550px" loading="lazy" width="550"
alt="Rejestr prototypów" /><img
src="../../images/patterns/diagrams/prototype/structure-prototype-cache-indexed1801.png?id=10a4a84a1a318f59dbc2b806fc936d04"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.12;"
srcset="/images/patterns/diagrams/prototype/structure-prototype-cache-indexed.png?id=10a4a84a1a318f59dbc2b806fc936d04 550w,/images/patterns/diagrams/prototype/structure-prototype-cache-indexed-1.5x.png?id=cb56c95533a4020368c48db9f9e2a37d 825w,/images/patterns/diagrams/prototype/structure-prototype-cache-indexed-2x.png?id=47b99eb7ae51158bdbb31deea4f5e98f 1100w"
sizes="(max-width: 720px) 100vw, 550px" loading="lazy" width="550"
alt="Rejestr prototypów" />
</figure>
:::
::::

1.  **Rejestr prototypów** udostępnia łatwy dostęp do często używanych
    prototypów. Przechowuje zestaw prefabrykowanych obiektów, które są
    gotowe do skopiowania. Najprostszym rejestrem prototypów będzie
    tablica asocjacyjna `nazwa → prototyp`. Jednakże, jeśli potrzebne są
    bardziej szczegółowe kryteria wyszukiwania, aniżeli tylko nazwa,
    można zbudować bardziej złożoną wersję rejestru.
:::::::
::::::::

::: {.section .pseudocode}
##  Pseudokod {#pseudocode}

W tym przykładzie, wzorzec **Prototyp** pozwala tworzyć dokładne kopie
figur geometrycznych bez sprzęgania kodu z klasami figur.

<figure class="image">
<img
src="../../images/patterns/diagrams/prototype/examplee491.png?id=47bc6c1058cb100b81e675b5ca6bda6c"
style="aspect-ratio: 1.42;"
srcset="/images/patterns/diagrams/prototype/example.png?id=47bc6c1058cb100b81e675b5ca6bda6c 470w,/images/patterns/diagrams/prototype/example-1.5x.png?id=067f3a2415370fadf5f92aadaa12b5e2 705w,/images/patterns/diagrams/prototype/example-2x.png?id=80393e0df17ae8130e5ada832d494949 940w"
sizes="(max-width: 720px) 100vw, 470px" loading="lazy" width="470"
alt="Struktura przykładu użycia wzorca projektowego Prototyp" />
<figcaption><p>Klonowanie obiektów należących do jednej
hierarchii klasowej.</p></figcaption>
</figure>

Wszystkie klasy figur są zgodne pod względem interfejsu, który z kolei
posiada metodę klonującą. Podklasa może wywołać metodę klonującą
klasy-rodzica przed skopiowaniem wartości swoich pól do obiektu
wynikowego.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// Bazowy prototyp.
abstract class Shape is
    field X: int
    field Y: int
    field color: string

    // Zwyczajny konstruktor.
    constructor Shape() is
        // ...

    // Konstruktor prototypu. Nowy obiekt jest inicjalizowany
    // wartościami istniejącego obiektu.
    constructor Shape(source: Shape) is
        this()
        this.X = source.X
        this.Y = source.Y
        this.color = source.color

    // Klonowanie zwraca jedną z podklas Shape.
    abstract method clone():Shape


// Konkretny prototyp. Metoda klonująca tworzy świeży obiekt i
// przekazuje go do konstruktora. Dopóki konstruktor nie skończy
// pracy, posiada jako jedyny odniesienie do świeżego klona.
// Gwarantuje to spójność klonowania.
class Rectangle extends Shape is
    field width: int
    field height: int

    constructor Rectangle(source: Rectangle) is
        // Trzeba wywołać konstruktor nadklasy w celu
        // skopiowania pól zdefiniowanych w nadklasie.
        super(source)
        this.width = source.width
        this.height = source.height

    method clone():Shape is
        return new Rectangle(this)


class Circle extends Shape is
    field radius: int

    constructor Circle(source: Circle) is
        super(source)
        this.radius = source.radius

    method clone():Shape is
        return new Circle(this)


// Gdzieś w kodzie klienckim.
class Application is
    field shapes: array of Shape

    constructor Application() is
        Circle circle = new Circle()
        circle.X = 10
        circle.Y = 10
        circle.radius = 20
        shapes.add(circle)

        Circle anotherCircle = circle.clone()
        shapes.add(anotherCircle)
        // Zmienna `anotherCircle` zawiera dokładną kopię
        // obiektu `circle`.

        Rectangle rectangle = new Rectangle()
        rectangle.width = 10
        rectangle.height = 20
        shapes.add(rectangle)

    method businessLogic() is
        // Wzorzec Prototyp rządzi, bo pozwala stworzyć kopię
        // obiektu bez wiedzy o jego typie.
        Array shapesCopy = new Array of Shapes.

        // Na przykład — nie musimy znać dokładnych elementów w
        // tablicy figur (shapes). Wiemy tylko, że wszystkie są
        // figurami. Ale dzięki polimorfizmowi, gdy wywołujemy
        // metodę `klonuj` na figurze, program sprawdza jej
        // klasę i uruchamia odpowiednią metodę klonującą
        // zdefiniowaną w tejże klasie. Dlatego też otrzymujemy
        // odpowiednie obiekty-klony, a nie ogólne obiekty
        // Shape.
        foreach (s in shapes) do
            shapesCopy.add(s.clone())

        // Tablica `shapesCopy` zawiera wierne kopie podklas z
        // tablicy `shape`.</code></pre>
</figure>
:::

:::::::: {.section .applicability-container}
##  Zastosowanie {#applicability}

::::::: applicability
::: applicability-problem
Stosuj wzorzec Prototyp gdy chcesz aby twój kod nie był zależny od
konkretnej klasy kopiowanego obiektu.
:::

::: applicability-solution
Powyższa sytuacja zdarza się często, gdy twój kod pracuje na obiektach
przekazanych z zewnętrznego źródła poprzez jakiś interfejs. Nieznana
jest wówczas konkretna klasa takich obiektów.

Wzorzec Prototyp pozwala kodowi klienckiemu na pracę ze wszystkimi
obiektami wspierającymi klonowanie za pomocą uogólnionego interfejsu.
Interfejs czyni kod kliencki niezależnym od konkretnych klas klonowanych
obiektów.
:::

::: applicability-problem
Stosuj ten wzorzec gdy chcesz zredukować ilość podklas różniących się
jedynie sposobem inicjalizacji swych obiektów. Ktoś inny bowiem mógł
stworzyć takie podklasy tylko w celu tworzenia obiektów o określonej
konfiguracji.
:::

::: applicability-solution
Wzorzec Prototyp pozwala korzystać z zestawu prefabrykowanych obiektów w
różnorakich konfiguracjach, stanowiących prototypy.

Zamiast tworzyć instancję podklasy zgodnej z jakąś konfiguracją, klient
może po prostu wyszukać i sklonować odpowiedni prototyp.
:::
:::::::
::::::::

::: {.section .checklist}
##  Jak zaimplementować {#checklist}

1.  Stwórz interfejs prototypu i zadeklaruj w nim metodę `klonuj`.
    Albo po prostu dodaj tę metodę do wszystkich klas należących do
    istniejącej hierarchii, o ile taką masz.

2.  Klasa prototypowa musi definiować alternatywny konstruktor, taki,
    który akceptuje jako argument obiekt swej klasy. Taki konstruktor
    powinien skopiować cechy przekazanego obiektu do nowo utworzonej
    instancji. Jeśli modyfikujemy podklasę, musimy wywołać konstruktor
    klasy-rodzica, aby to nadklasa dokonała klonowania pól
    zdefiniowanych jako prywatne.

    Jeśli używany przez ciebie język programowania nie wspiera
    przeciążania metod, możesz zdefiniować specjalną metodę służącą
    kopiowaniu danych obiektu. Konstruktor jest na nią dogodniejszym
    miejscem, bo zwraca nowo utworzony obiekt od razu po wywołaniu
    operatora `new`.

3.  Metoda klonująca zazwyczaj składa się tylko z jednej linii kodu:
    wywołanie operatora `new` z prototypową wersją konstruktora.
    Pamiętać trzeba, że każda klasa musi wyraźnie nadpisywać metodę
    klonującą, by stosowała nazwę własnej klasy wraz z operatorem
    `new`. W przeciwnym razie metoda klonująca może stworzyć obiekt
    klasy nadrzędnej.

4.  Opcjonalnie, można stworzyć scentralizowany rejestr prototypów
    przechowujący katalog najczęściej używanych.

    Możesz zaimplementować rejestr w formie nowej klasy fabrycznej, albo
    umieścić go w bazowej klasie prototypowej ze statyczną metodą
    służącą pobieraniu prototypu. Metoda ta powinna szukać prototypu na
    podstawie określonych przez klienta, przekazanych jej kryteriów.
    Kryteriami takimi może być albo prosty łańcuch tekstowy, albo też
    bardziej skomplikowany zestaw parametrów wyszukiwania. Po
    znalezieniu stosownego prototypu, rejestr powinien sklonować go i
    zwrócić kopię klientowi.

    Na końcu należy zamienić bezpośrednie wywołania konstruktorów
    podklas na wywołania metody fabrycznej rejestru prototypów.
:::

:::::: {.section .pros-cons}
##  Zalety i wady {#pros-cons}

::::: row
::: col-sm-6
-    Możesz klonować obiekty bez konieczności sprzęgania ze szczegółami
    ich konkretnych klas.
-    Możesz pozbyć się wielokrotnie powtarzanego kodu
    inicjalizacyjnego na rzecz klonowania prefabrykowanych prototypów.
-    Dużo wygodniejsze produkowanie złożonych obiektów.
-    Podejście to stanowi alternatywę do dziedziczenia w przypadku gdy
    mamy do czynienia z wcześniej zdefiniowanymi konfiguracjami
    złożonych obiektów.
:::

::: col-sm-6
-    Klonowanie złożonych obiektów, które mają odniesienia cykliczne,
    może być trudne.
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

-   Klasy [Fabryka abstrakcyjna](abstract-factory.html) często wywodzą
    się z zestawu [Metod wytwórczych](factory-method.html), ale można
    także użyć [Prototypu](prototype.html) do skomponowania metod w tych
    klasach.

-   [Prototyp](prototype.html) może pomóc stworzyć historię, zapisując
    kopie [Poleceń](command.html).

-   Projekty intensywnie korzystające ze wzorców
    [Kompozyt](composite.html) i [Dekorator](decorator.html) mogą
    skorzystać również na zastosowaniu [Prototypu](prototype.html).
    Zastosowanie tego wzorca pozwala klonować złożone struktury zamiast
    konstruować je ponownie od zera.

-   [Prototyp](prototype.html) nie bazuje na dziedziczeniu, więc nie
    posiada właściwych temu podejściu wad. Z drugiej strony jednak
    *Prototyp* wymaga skomplikowanej inicjalizacji klonowanego obiektu.
    [Metoda wytwórcza](factory-method.html) bazuje na dziedziczeniu, ale
    nie wymaga etapu inicjalizacji.

-   Czasem [Prototyp](prototype.html) może być prostszą alternatywą dla
    [Pamiątki](memento.html). Jest to możliwe jeśli obiekt, którego stan
    chcesz przechować w historii, jest w miarę nieskomplikowany. Obiekt
    taki nie powinien też mieć powiązań z zewnętrznymi zasobami, albo
    muszą one być łatwe do ponownego nawiązania.

-   [Fabryki abstrakcyjne](abstract-factory.html),
    [Budowniczych](builder.html) oraz [Prototypy](prototype.html) można
    zaimplementować jako [Singletony](singleton.html).
:::

::: {.section .implementations}
##  Przykłady kodu {#implementations}

[![Prototyp w języku
C#](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](prototype/csharp/example.html "Prototyp w języku C#"){.prog-lang-link}
[![Prototyp w języku
C++](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](prototype/cpp/example.html "Prototyp w języku C++"){.prog-lang-link}
[![Prototyp w języku
Go](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](prototype/go/example.html "Prototyp w języku Go"){.prog-lang-link}
[![Prototyp w języku
Java](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](prototype/java/example.html "Prototyp w języku Java"){.prog-lang-link}
[![Prototyp w języku
PHP](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](prototype/php/example.html "Prototyp w języku PHP"){.prog-lang-link}
[![Prototyp w języku
Python](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](prototype/python/example.html "Prototyp w języku Python"){.prog-lang-link}
[![Prototyp w języku
Ruby](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](prototype/ruby/example.html "Prototyp w języku Ruby"){.prog-lang-link}
[![Prototyp w języku
Rust](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](prototype/rust/example.html "Prototyp w języku Rust"){.prog-lang-link}
[![Prototyp w języku
Swift](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](prototype/swift/example.html "Prototyp w języku Swift"){.prog-lang-link}
[![Prototyp w języku
TypeScript](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](prototype/typescript/example.html "Prototyp w języku TypeScript"){.prog-lang-link}
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

[Singleton []{.fa .fa-arrow-right}](singleton.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Wróć

[[]{.fa .fa-arrow-left} Budowniczy](builder.html){.btn .btn-default
rel="prev"}
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
