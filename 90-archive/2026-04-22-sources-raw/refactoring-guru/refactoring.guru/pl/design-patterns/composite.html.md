::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [Wzorce
projektowe](../design-patterns.html){.type} / [Wzorce
strukturalne](structural-patterns.html){.type}
:::

# Kompozyt {#kompozyt .title}

::: aka
Znany też jako: [Drzewo obiektów, ]{style="display:inline-block"}[Object
Tree, ]{style="display:inline-block"}[Composite]{style="display:inline-block"}
:::

::: {.section .intent}
##  Cel {#intent}

**Kompozyt** to strukturalny wzorzec projektowy pozwalający komponować
obiekty w struktury drzewiaste, a następnie traktować te struktury jakby
były osobnymi obiektami.

<figure class="image">
<img
src="../../images/patterns/content/composite/compositec299.png?id=73bcf0d94db360b636cd745f710d19db"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/composite/composite.png?id=73bcf0d94db360b636cd745f710d19db 640w,/images/patterns/content/composite/composite-1.5x.png?id=098acddafae70b1ec1cb1b05e833c3ae 960w,/images/patterns/content/composite/composite-2x.png?id=8847e6f8e2cb892ed2229faba83bd1b7 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Wzorzec projektowy Kompozyt" />
</figure>
:::

::: {.section .problem}
##  Problem

Stosowanie wzorca Kompozyt ma sens tylko w przypadku, gdy główny model
twojej aplikacji można przedstawić w formie drzewa.

Na przykład, wyobraź sobie, że masz dwa typy obiektów: `Produkty` i
`Opakowania`. `Opakowanie` może zawierać wiele `Produktów`, a także
pewną liczbę mniejszych `Opakowań`. Te małe `Opakowania` także mogą
przechowywać zarówno `Produkty`, jak i jeszcze mniejsze `Opakowania` i
tak dalej.

Załóżmy, że postanowisz stworzyć system zamawiania, który wykorzystuje
jakieś klasy. Zamówienia mogą obejmować proste produkty nie posiadające
opakowań, a także pudła wypełnione produktami i... innymi pudełkami. Jak
wówczas określić całkowitą wartość pieniężną takiego zamówienia?

<figure class="image">
<img
src="../../images/patterns/diagrams/composite/problem-pl91cb.png?id=d3972bf541540b36bf70c4efba1226e5"
style="aspect-ratio: 1;"
srcset="/images/patterns/diagrams/composite/problem-pl.png?id=d3972bf541540b36bf70c4efba1226e5 370w,/images/patterns/diagrams/composite/problem-pl-1.5x.png?id=9b1ef16027cb7389b88dc22930f1f87f 555w,/images/patterns/diagrams/composite/problem-pl-2x.png?id=92970a2879ca2c436219ea566d0ebfa7 740w"
sizes="(max-width: 720px) 100vw, 370px" loading="lazy" width="370"
alt="Struktura złożonego zamówienia" />
<figcaption><p>Zamówienie może obejmować różnorakie produkty opakowane w
pudełka, które z kolei są zapakowane w większych pudełkach, i tak dalej.
Cała struktura przypomina odwrócone drzewo.</p></figcaption>
</figure>

Możesz spróbować bezpośredniego podejścia: rozpakuj wszystkie pudełka,
przejrzyj ich zawartość i oblicz sumę. W prawdziwym życiu jest to
wykonalne, ale w programie nie jest to niestety tak proste, jak
działanie w pętli. Musisz z góry znać klasy `Produktów` i `Opakowań`
jakie przeglądasz, poziom zagnieżdżenia opakowań i inne kłopotliwe
szczegóły. Wszystko to sprawia, że bezpośrednie podejście byłoby
problematyczne, albo wręcz niemożliwe.
:::

::: {.section .solution}
##  Rozwiązanie {#solution}

Wzorzec projektowy Kompozyt zakłada rozwiązanie w którym zarówno z
`Produktami`, jak i `Opakowaniami` pracujemy poprzez wspólny interfejs,
deklarujący metodę obliczania całej sumy.

Jak by to działało? W przypadku produktu, metoda zwracałaby jego cenę. W
przypadku pudła, przejrzałaby zawartość przedmiot po przedmiocie,
pytając o cenę każdego z nich, a na końcu zwróciła całkowitą wartość
opakowania. Jeśli któryś z przedmiotów w pudle okazałby się mniejszym
pudełkiem, również przejrzałaby jego zawartość i tak aż do obliczenia
wartości wszystkich obiektów wewnątrz. Samo opakowanie mogłoby nawet
dodawać swoją wartość do ostatecznej ceny.

<figure class="image">
<img
src="../../images/patterns/content/composite/composite-comic-1-plf21e.png?id=ac9f7bbaa6e9bea3078a1d50eb4cd13a"
style="aspect-ratio: 2;"
srcset="/images/patterns/content/composite/composite-comic-1-pl.png?id=ac9f7bbaa6e9bea3078a1d50eb4cd13a 600w,/images/patterns/content/composite/composite-comic-1-pl-1.5x.png?id=405efb8791778668544ea4cc7b3740c3 900w,/images/patterns/content/composite/composite-comic-1-pl-2x.png?id=f4336bfb9e9cfb242f3de31a280e972c 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Rozwiązanie sugerowane przez wzorzec Kompozyt" />
<figcaption><p>Wzorzec Kompozyt pozwala wykonywać działania
rekursywnie — po wszystkich komponentach drzewa.</p></figcaption>
</figure>

Największą zaletą tego podejścia jest to, że nie musimy się przejmować
konkretną klasą obiektów składających się na drzewo. Nie musimy
wiedzieć, czy obiekt jest prostym produktem, czy też złożonym
kontenerem. Traktujemy wszystko w taki sam sposób, za pomocą takiego
samego interfejsu. Gdy wywołasz metodę, same obiekty przekażą ją sobie
dalej.
:::

::: {.section .analogy}
##  Analogia do prawdziwego życia {#analogy}

<figure class="image">
<img
src="../../images/patterns/diagrams/composite/live-examplee724.png?id=548a7cec45b493af66e8bfe524a137d3"
style="aspect-ratio: 1.22;"
srcset="/images/patterns/diagrams/composite/live-example.png?id=548a7cec45b493af66e8bfe524a137d3 280w,/images/patterns/diagrams/composite/live-example-1.5x.png?id=2129195f4103a5a4d3440a086ce3dc87 420w,/images/patterns/diagrams/composite/live-example-2x.png?id=b555458f20fc30425ae6dada5da492af 560w"
sizes="(max-width: 720px) 100vw, 280px" loading="lazy" width="280"
alt="Przykład struktury w wojsku" />
<figcaption><p>Przykład struktury w wojsku.</p></figcaption>
</figure>

Armie większości państw mają strukturę hierarchiczną. Armia składa się z
wielu dywizji; dywizje dzielą się na brygady, a brygady składają się z
drużyn. Rozkazy wydaje się odgórnie, po czym są one przekazywane w
dół, po każdym z poziomów, aż każdy żołnierz będzie wiedział co jest do
zrobienia.
:::

::::: {.section .structure-container}
##  Struktura {#structure}

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/composite/structure-pl6bbb.png?id=fd6a90c6d8c2ae947f8ad16d06422c7f"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 0.82;"
srcset="/images/patterns/diagrams/composite/structure-pl.png?id=fd6a90c6d8c2ae947f8ad16d06422c7f 360w,/images/patterns/diagrams/composite/structure-pl-1.5x.png?id=7b83a511f5ae069f0ce85836c78939a8 540w,/images/patterns/diagrams/composite/structure-pl-2x.png?id=b43621494566d56f6e029cbcc5695b2b 720w"
sizes="(max-width: 720px) 100vw, 360px" loading="lazy" width="360"
alt="Struktura wzorca Kompozyt" /><img
src="../../images/patterns/diagrams/composite/structure-pl-indexed3e18.png?id=5f2c7901e1cb2d7ea25ff48c32a5dd4c"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 0.87;"
srcset="/images/patterns/diagrams/composite/structure-pl-indexed.png?id=5f2c7901e1cb2d7ea25ff48c32a5dd4c 400w,/images/patterns/diagrams/composite/structure-pl-indexed-1.5x.png?id=f871eecc5270a289231947802939faff 600w,/images/patterns/diagrams/composite/structure-pl-indexed-2x.png?id=9f9568c8ab50e238dd7ac1235ebfd34a 800w"
sizes="(max-width: 720px) 100vw, 400px" loading="lazy" width="400"
alt="Struktura wzorca Kompozyt" />
</figure>
:::

1.  Interfejs **Komponentu** opisuje operacje wspólne zarówno dla
    prostych, jak i złożonych elementów drzewa.

2.  **Liść** jest podstawowym elementem drzewa i nie posiada elementów
    podrzędnych.

    Zazwyczaj, to właśnie komponenty-liście wykonują większość
    faktycznej pracy, ponieważ nie mają komu jej zlecić.

3.  **Kontener** (zwany też *kompozytem*) jest elementem posiadającym
    elementy podrzędne: liście, lub inne kontenery. Kontener nie zna
    konkretnych klas swojej zawartości. Komunikuje się ze wszystkimi
    elementami podrzędnymi tylko poprzez interfejs komponentu.

    Otrzymawszy żądanie, kontener deleguje pracę do swoich podelementów,
    przetwarza wyniki pośrednie i zwraca ostateczny wynik klientowi.

4.  **Klient** współpracuje ze wszystkimi elementami za pośrednictwem
    interfejsu komponentu. W wyniku tego klient może działać w taki sam
    sposób zarówno na prostych, jak i złożonych elementach drzewa.
::::
:::::

::: {.section .pseudocode}
##  Pseudokod {#pseudocode}

W tym przykładzie, wzorzec **Kompozyt** pozwala zaimplementować
układanie figur geometrycznych w stos w programie graficznym.

<figure class="image">
<img
src="../../images/patterns/diagrams/composite/example7ea7.png?id=98ba81d07c979038dd2ebf3c83a2e19f"
style="aspect-ratio: 0.84;"
srcset="/images/patterns/diagrams/composite/example.png?id=98ba81d07c979038dd2ebf3c83a2e19f 370w,/images/patterns/diagrams/composite/example-1.5x.png?id=ae008efd4146628cd7303925d4ce90be 555w,/images/patterns/diagrams/composite/example-2x.png?id=d21edef39d3792e8a4c6736727ac7305 740w"
sizes="(max-width: 720px) 100vw, 370px" loading="lazy" width="370"
alt="Struktura przykładu Kompozytu" />
<figcaption><p>Przykład edytora figur geometrycznych.</p></figcaption>
</figure>

Klasa `ZłożonaGrafika` jest kontenerem, na który może składać się każda
liczba podrzędnych figur, w tym inne figury złożone. Figura złożona ma
takie same metody jak pojedyncza figura. Jednakże, zamiast robić coś
samodzielnie, figura złożona przekazuje żądanie rekursywnie wszystkim
swoim elementom, a sama "zsumowuje" wynik.

Kod kliencki współpracuje ze wszystkimi figurami za pomocą interfejsu
który jest jednakowy dla wszystkich klas figur. Tym samym klient nie
jest świadomy, czy ma do czynienia z prostym kształtem, czy złożonym.
Klient może pracować z bardzo złożonymi strukturami bez wiązania się z
konkretnymi klasami tworzącymi strukturę.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// Interfejs komponentu deklaruje działania wspólne zarówno dla
// prostych jak i skomplikowanych obiektów struktury.
interface Graphic is
    method move(x, y)
    method draw()

// Klasa liść reprezentuje końcowe elementy struktury. Liść nie
// posiada pod-obiektów. Na ogół to właśnie te obiekty wykonują
// faktyczne działania, zaś obiekty złożone jedynie je delegują
// swoim pod-komponentom.
class Dot implements Graphic is
    field x, y

    constructor Dot(x, y) { ... }

    method move(x, y) is
        this.x += x, this.y += y

    method draw() is
        // Narysuj kropkę w miejscu o współrzędnych X i Y.

// Wszystkie klasy komponentów mogą rozszerzać inne komponenty.
class Circle extends Dot is
    field radius

    constructor Circle(x, y, radius) { ... }

    method draw() is
        // Narysuj okrąg w punkcie X i Y o promieniu R.

// Klasa kompozyt reprezentuje komponenty złożone które mogą
// posiadać potomstwo. Obiekty kompozytowe zazwyczaj delegują
// faktyczną pracę swoim dzieciom a następnie &quot;podsumowują&quot;
// otrzymane wyniki.
class CompoundGraphic implements Graphic is
    field children: array of Graphic

    // Obiekt będący kompozytem może dodawać lub usuwać inne
    // komponenty (zarówno proste jak i złożone) do/ze swej
    // listy obiektów-dzieci.
    method add(child: Graphic) is
        // Dodaj obiekt potomny do tablicy potomstwa.

    method remove(child: Graphic) is
        // Usuń obiekt-dziecko z tablicy potomstwa.

    method move(x, y) is
        foreach (child in children) do
            child.move(x, y)

    // Kompozyt wykonuje swoje podstawowe zadania w konkretny
    // sposób. Przechodzi rekursywnie przez wszystkie obiekty
    // podrzędne, zbierając od nich wyniki i je sumując. Skoro
    // obiekty-dzieci kompozytu przekazują te wywołania swoim
    // obiektom-dzieciom i tak dalej, całe drzewo obiektów
    // zostaje przejrzane.
    method draw() is
        // 1. Dla każdego komponentu-dziecka:
        //     - Narysuj komponent.
        //     - Zaktualizuj otaczający prostokąt.
        // 2. Narysuj prostokąt przerywaną linią korzystając ze
        // współrzędnych granicy.


// Kod klienta współpracuje ze wszystkimi komponentami za
// pośrednictwem ich interfejsu bazowego. Dzięki temu kod
// kliencki posiada wsparcie zarówno prostych obiektów-liści,
// jak i złożonych kompozytów.
class ImageEditor is
    field all: CompoundGraphic

    method load() is
        all = new CompoundGraphic()
        all.add(new Dot(1, 2))
        all.add(new Circle(5, 3, 10))
        // ...

    // Połącz wybrane komponenty w jeden złożony komponent
    // kompozytowy.
    method groupSelected(components: array of Graphic) is
        group = new CompoundGraphic()
        foreach (component in components) do
            group.add(component)
            all.remove(component)
        all.add(group)
        // Wszystkie komponenty zostaną narysowane.
        all.draw()</code></pre>
</figure>
:::

:::::::: {.section .applicability-container}
##  Zastosowanie {#applicability}

::::::: applicability
::: applicability-problem
Stosuj wzorzec Kompozyt gdy musisz zaimplementować drzewiastą strukturę
obiektów.
:::

::: applicability-solution
Wzorzec Kompozyt określa dwa podstawowe typy elementów współdzielących
jednakowy interfejs: proste liście oraz złożone kontenery. Kontener może
być złożony zarówno z liści, jak i z innych kontenerów. Pozwala to
skonstruować zagnieżdżoną, rekurencyjną strukturę obiektów
przypominającą drzewo.
:::

::: applicability-problem
Stosuj ten wzorzec gdy chcesz, aby kod kliencki traktował zarówno
proste, jak i złożone elementy jednakowo.
:::

::: applicability-solution
Wszystkie elementy zdefiniowane przez wzorzec Kompozyt współdzielą jeden
interfejs. Dzięki temu, klient nie musi martwić się konkretną klasą
obiektów z jakimi ma do czynienia.
:::
:::::::
::::::::

::: {.section .checklist}
##  Jak zaimplementować {#checklist}

1.  Upewnij się, że główny model twojej aplikacji można przedstawić w
    formie struktury drzewiastej. Spróbuj rozdzielić go na proste
    elementy i kontenery. Pamiętaj, że kontenery muszą móc zawierać w
    sobie zarówno proste elementy, jak i inne kontenery.

2.  Zadeklaruj interfejs komponentu z listą metod które mają sens
    zarówno w przypadku prostych, jak i złożonych komponentów.

3.  Stwórz klasę-liść reprezentującą proste elementy. Program może
    posiadać wiele różnych klas-liści.

4.  Stwórz klasę-kontener, reprezentującą złożone elementy. W tej klasie
    umieść pole tablicowe, które przechowywać będzie odniesienia do
    elementów podrzędnych. Tablica musi być w stanie przechowywać
    zarówno liście, jak i kontenery, więc zadeklaruj jej typ jako
    interfejs komponentu.

    Implementując metody interfejsu komponentu, pamiętaj, że kontener ma
    delegować większość swych obowiązków elementom podrzędnym.

5.  Na koniec zdefiniuj metody pozwalające dodawać i usuwać elementy
    podrzędne w kontenerze.

    Pamiętaj, że powyższe operacje można zadeklarować w interfejsie
    komponentu. Łamie to *Zasadę segregacji interfejsów*, ponieważ
    metody będą puste w klasie-liść. W zamian, klient będzie w stanie
    traktować wszystkie elementy jednakowo, nawet komponując drzewo.
:::

:::::: {.section .pros-cons}
##  Zalety i wady {#pros-cons}

::::: row
::: col-sm-6
-    Można pracować ze skomplikowanymi strukturami drzewiastymi w
    wygodny sposób: wykorzystaj na swoją korzyść polimorfizm i rekursję.
-    *Zasada otwarte/zamknięte*. Możesz wprowadzać do programu obsługę
    nowych typów elementów bez psucia istniejącego kodu, gdyż pracuje on
    teraz z drzewem różnych obiektów.
:::

::: col-sm-6
-    Ustalenie wspólnego interfejsu dla klas o diametralnie różnych
    funkcjonalnościach może okazać się trudne. W pewnych przypadkach
    trzeba przesadnie uogólnić interfejs komponentu, co uczyni go
    trudniejszym do zrozumienia.
:::
:::::
::::::

::: {.section .relations}
##  Powiązania z innymi wzorcami {#relations}

-   Możesz zastosować wzorzec [Budowniczy](builder.html) by tworzyć
    złożone drzewa [Kompozytowe](composite.html) dzięki możliwości
    zaprogramowania ich etapów konstrukcji tak, aby odbywały się
    rekurencyjnie.

-   [Łańcuch zobowiązań](chain-of-responsibility.html) często stosuje
    się w połączeniu z [Kompozytem](composite.html). W takim przypadku,
    gdy komponent-liść otrzymuje żądanie, może je przekazać poprzez
    łańcuch nadrzędnych komponentów aż do korzenia drzewa obiektów.

-   [Iteratory](iterator.html) służą do sekwencyjnego przemieszczania
    się po drzewie [Kompozytowym](composite.html) element po elemencie.

-   [Odwiedzający](visitor.html) może wykonać działanie na całym drzewie
    [Kompozytowym](composite.html).

-   Węzły będące liśćmi drzewa [Kompozytowego](composite.html) można
    zaimplementować jako [Pyłki](flyweight.html) by zaoszczędzić nieco
    pamięci RAM.

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
:::

::: {.section .implementations}
##  Przykłady kodu {#implementations}

[![Kompozyt w języku
C#](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](composite/csharp/example.html "Kompozyt w języku C#"){.prog-lang-link}
[![Kompozyt w języku
C++](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](composite/cpp/example.html "Kompozyt w języku C++"){.prog-lang-link}
[![Kompozyt w języku
Go](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](composite/go/example.html "Kompozyt w języku Go"){.prog-lang-link}
[![Kompozyt w języku
Java](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](composite/java/example.html "Kompozyt w języku Java"){.prog-lang-link}
[![Kompozyt w języku
PHP](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](composite/php/example.html "Kompozyt w języku PHP"){.prog-lang-link}
[![Kompozyt w języku
Python](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](composite/python/example.html "Kompozyt w języku Python"){.prog-lang-link}
[![Kompozyt w języku
Ruby](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](composite/ruby/example.html "Kompozyt w języku Ruby"){.prog-lang-link}
[![Kompozyt w języku
Rust](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](composite/rust/example.html "Kompozyt w języku Rust"){.prog-lang-link}
[![Kompozyt w języku
Swift](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](composite/swift/example.html "Kompozyt w języku Swift"){.prog-lang-link}
[![Kompozyt w języku
TypeScript](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](composite/typescript/example.html "Kompozyt w języku TypeScript"){.prog-lang-link}
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

[Dekorator []{.fa .fa-arrow-right}](decorator.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Wróć

[[]{.fa .fa-arrow-left} Most](bridge.html){.btn .btn-default rel="prev"}
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
