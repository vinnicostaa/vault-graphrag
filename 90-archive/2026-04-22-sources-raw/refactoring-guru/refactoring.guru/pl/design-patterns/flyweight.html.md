:::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [Wzorce
projektowe](../design-patterns.html){.type} / [Wzorce
strukturalne](structural-patterns.html){.type}
:::

# Pyłek {#pyłek .title}

::: aka
Znany też jako:
[Cache, ]{style="display:inline-block"}[Flyweight]{style="display:inline-block"}
:::

::: {.section .intent}
##  Cel {#intent}

**Pyłek** jest strukturalnym wzorcem projektowym pozwalającym zmieścić
więcej obiektów w danej przestrzeni pamięci RAM poprzez współdzielenie
części opisu ich stanów.

<figure class="image">
<img
src="../../images/patterns/content/flyweight/flyweight7be2.png?id=e34fbacb847dd609b5e68aaf252c4db0"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/flyweight/flyweight.png?id=e34fbacb847dd609b5e68aaf252c4db0 640w,/images/patterns/content/flyweight/flyweight-1.5x.png?id=b32df2297473b8a7577e1900e57325ac 960w,/images/patterns/content/flyweight/flyweight-2x.png?id=6a8f17d9550c75c3d648a605c4d31b45 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Wzorzec projektowy Pyłek" />
</figure>
:::

::: {.section .problem}
##  Problem

Aby rozerwać się nieco po pracy, postanawiasz stworzyć prostą grę
komputerową: gracze poruszają się po mapie i strzelają do siebie. Chcesz
zaimplementować realistyczny system cząstek i uczynić z niego
wyróżniającą się zaletę gry. Niech wielkie ilości kul, rakiet i odłamków
fruwają po całej mapie, dostarczając ekscytującej rozrywki.

Ukończywszy pracę, wykonujesz ostatni commit, kompilujesz grę i wysyłasz
znajomemu na próbę. Chociaż gra chodzi płynnie na twoim komputerze,
kolega nie może długo pograć. Po paru minutach gra się wiesza. Po
wielogodzinnym poszukiwaniu przyczyn w dziennikach debugowych,
zauważasz, że grze zabrakło pamięci RAM. Okazało się bowiem, że komputer
kolegi jest słabszy niż twój i dlatego problem objawił się u niego tak
szybko.

Źródłem problemu był system cząstek. Każda cząstka, jak kula, rakieta
czy odłamek, reprezentowany był jako osobny obiekt zawierający mnóstwo
danych. W którymś momencie, w czasie renderowania strzelaniny, nowo
utworzone cząstki nie mieściły się w pamięci operacyjnej i gra kończyła
działanie.

<figure class="image">
<img
src="../../images/patterns/diagrams/flyweight/problem-pl41b6.png?id=6511554cd798a3ab41e4fc3bf13a900c"
style="aspect-ratio: 2.54;"
srcset="/images/patterns/diagrams/flyweight/problem-pl.png?id=6511554cd798a3ab41e4fc3bf13a900c 660w,/images/patterns/diagrams/flyweight/problem-pl-1.5x.png?id=2d124f4d9d25bd811ad17a4be9f83f73 990w,/images/patterns/diagrams/flyweight/problem-pl-2x.png?id=fb72ee7424d0201610b3f2ba6c9a722b 1320w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="660"
alt="Problem dla wzorca Pyłek" />
</figure>
:::

::: {.section .solution}
##  Rozwiązanie {#solution}

Przy dokładniejszej inspekcji klasy `Cząstka` zauważamy, że kolor i
sprite każdej cząstki zużywają znacznie więcej pamięci, niż inne pola
obiektu. Co gorsza, te dwa pola przechowują niemal identyczne dane we
wszystkich cząstkach. Na przykład --- wszystkie kule mają tę samą
barwę i sprite.

<figure class="image">
<img
src="../../images/patterns/diagrams/flyweight/solution1-pl9707.png?id=9843aac8104e8b65202d18a62482284b"
style="aspect-ratio: 2.06;"
srcset="/images/patterns/diagrams/flyweight/solution1-pl.png?id=9843aac8104e8b65202d18a62482284b 640w,/images/patterns/diagrams/flyweight/solution1-pl-1.5x.png?id=01d0445950bf7504628e101486d3bd25 960w,/images/patterns/diagrams/flyweight/solution1-pl-2x.png?id=37841fac7084982398303eee7359d462 1280w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="640"
alt="Rozwiązanie z użyciem wzorca Pyłek" />
</figure>

Inne elementy opisujące stan cząstki, jak współrzędne, wektor ruchu i
prędkość są unikalne dla każdej z nich. Bo przecież te wartości ulegają
ciągłej zmianie. Dane te reprezentują wciąż zmieniający się kontekst, w
jakim cząstka się znajduje, zaś kolor i sprite pozostają jednakowe dla
każdej z nich.

Dane niezmienne, opisujące obiekt, nazywa się *stanem wewnętrznym*.
Opisany jest on w każdym z obiektów, zaś inne obiekty mają do niego
tylko prawo odczytu. Reszta stanu obiektu, często zmienianym "z
zewnątrz" przez inne obiekty, zwana jest *stanem zewnętrznym*.

Wzorzec Pyłek proponuje rezygnację z przechowywania stanu zewnętrznego w
obiekcie. Zamiast tego należy przekazywać ten stan konkretnym metodom
które go potrzebują. Tylko stan wewnętrzny powinien pozostać zapisany w
obrębie obiektu, pozwalając na użycie go ponownie w innych kontekstach.
Dzięki temu potrzebujemy mniej tych obiektów, ponieważ różnią się tylko
pod względem wewnętrznego stanu, którego możliwych kombinacji jest
znacznie mniej.

<figure class="image">
<img
src="../../images/patterns/diagrams/flyweight/solution3-plfba5.png?id=21d29e367dddafb95cac5827304c6079"
style="aspect-ratio: 0.76;"
srcset="/images/patterns/diagrams/flyweight/solution3-pl.png?id=21d29e367dddafb95cac5827304c6079 424w,/images/patterns/diagrams/flyweight/solution3-pl-1.5x.png?id=f7f5cd3f5917bef1ea103a19a38e8528 636w,/images/patterns/diagrams/flyweight/solution3-pl-2x.png?id=b050e830afeb76e6e6f0fd64f82424c4 848w"
sizes="(max-width: 720px) 100vw, 424px" loading="lazy" width="424"
alt="Rozwiązanie stosujące wzorzec Pyłek" />
</figure>

Wróćmy do naszej gry. Zakładając, że wyekstrahowaliśmy stan zewnętrzny z
naszej klasy-cząstki, wystarczą zaledwie 3 obiekty, aby reprezentować
wszystkie cząstki w grze: kulę, rakietę i odłamek. Jak zapewne już się
domyślasz, obiekt przechowujący tylko stan wewnętrzny nazywa się
Pyłkiem.

#### Przechowywanie danych zewnętrznych

Dokąd przenieść zewnętrzny stan? Jakaś klasa powinna go przechowywać,
prawda? W większości przypadków, przenosi się go do obiektu
kontenerowego, który agreguje obiekty zanim zastosujemy wzorzec.

W naszym przypadku to główny obiekt `Gra` przechowuje wszystkie
cząstki w polu `cząstki`. By przenieść zewnętrzne stany do tej klasy,
musisz stworzyć wiele pól tablicowych do przechowywania współrzędnych,
wektorów i prędkości każdej cząstki. Ale to nie wszystko ---
potrzebujesz jeszcze jednej tablicy w celu przechowania referencji do
konkretnego pyłku reprezentującego cząstkę. Te dwie tablice muszą być
zsynchronizowane, aby można było pobrać wszystkie dane cząstki stosując
ten sam indeks.

<figure class="image">
<img
src="../../images/patterns/diagrams/flyweight/solution2-pld998.png?id=2a26afb1edd1c47f5f0b3a3981807dc8"
style="aspect-ratio: 1.94;"
srcset="/images/patterns/diagrams/flyweight/solution2-pl.png?id=2a26afb1edd1c47f5f0b3a3981807dc8 640w,/images/patterns/diagrams/flyweight/solution2-pl-1.5x.png?id=95eec96612192e51c095f1bb7b69fe41 960w,/images/patterns/diagrams/flyweight/solution2-pl-2x.png?id=e6c995b7bfcbbf825c95bfdcf073ec0f 1280w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="640"
alt="Rozwiązanie stosujące wzorzec Pyłek" />
</figure>

Bardziej eleganckim rozwiązaniem jest utworzenie osobnej klasy
kontekstowej która przechowa zewnętrzny stan wraz z odniesieniem do
obiektu pyłek. W takiej sytuacji potrzebna jest tylko jedna tablica w
klasie kontenerowej.

Ale chwileczkę! Czy czasem nie będzie nam potrzebne tyle takich obiektów
kontekstowych, ile mieliśmy na samym początku? W zasadzie tak, ale te
obiekty są dużo mniejsze niż wcześniej. Pola zajmujące najwięcej pamięci
przeniesiono do kilku obiektów pyłków. Teraz tysiąc małych obiektów
kontekstowych może wykorzystać ponownie pojedynczy, duży obiekt pyłek,
zamiast przechowywać tysiąc kopii ich danych.

#### Pyłek a niezmienność

Skoro ten sam obiekt pyłek może być wykorzystany w różnych kontekstach,
musisz się upewnić, że jego stan nie może być zmieniony. Pyłek powinien
inicjalizować swój stan tylko jednorazowo, za pośrednictwem parametrów
konstruktora. Nie powinien eksponować innym obiektom żadnych setterów
ani pól publicznych.

#### Fabryka pyłków

Stworzenie metody wytwórczej zarządzającej pulą istniejących obiektów
pyłków daje nam wygodniejszy dostęp do różnych cząstek. Metoda przyjmuje
pożądany przez klienta opis stanu wewnętrznego, poszukuje istniejącego
obiektu o takim stanie i go zwraca. Jeśli go nie znajdzie --- tworzy
nowy i dodaje go do puli.

Istnieje wiele miejsc, gdzie można umieścić taką metodę. Najbardziej
oczywistym jest kontener pyłków. Innym sposobem jest stworzenie nowej
klasy fabrycznej. Można też uczynić metodę wytwórczą statyczną i
umieścić ją w faktycznej klasie pyłek.
:::

::::: {.section .structure-container}
##  Struktura {#structure}

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/flyweight/structure7b00.png?id=c1e7e1748f957a4792822f902bc1d420"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.64;"
srcset="/images/patterns/diagrams/flyweight/structure.png?id=c1e7e1748f957a4792822f902bc1d420 640w,/images/patterns/diagrams/flyweight/structure-1.5x.png?id=34cf08f6c2b09dcd1c521c7512cc52b6 960w,/images/patterns/diagrams/flyweight/structure-2x.png?id=a7c8347421bde16435fc90a706f5dd34 1280w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="640"
alt="Struktura wzorca projektowego Pyłek" /><img
src="../../images/patterns/diagrams/flyweight/structure-indexed086a.png?id=aa490792baa26b04464dacbc1f4a19cd"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.54;"
srcset="/images/patterns/diagrams/flyweight/structure-indexed.png?id=aa490792baa26b04464dacbc1f4a19cd 630w,/images/patterns/diagrams/flyweight/structure-indexed-1.5x.png?id=b22a5eab6aacbbd03c66c34802b240ca 945w,/images/patterns/diagrams/flyweight/structure-indexed-2x.png?id=205e2f7d08b4ac0695f445a9db8989c4 1260w"
sizes="(max-width: 720px) 100vw, 630px" loading="lazy" width="630"
alt="Struktura wzorca projektowego Pyłek" />
</figure>
:::

1.  Wzorzec Pyłek jest jedynie optymalizacją. Przed zastosowaniem go,
    upewnij się, że twój program ma potencjalny problem z wyczerpywaniem
    pamięci RAM wskutek istnienia jednocześnie wielkiej liczby podobnych
    obiektów. Odpowiedz sobie na pytanie, czy nie da się takiego
    problemu rozwiązać w inny sposób.

2.  Klasa **Pyłek** zawiera tę porcję stanu pierwotnego obiektu, która
    może być współdzielona pomiędzy wieloma instancjami. Ten sam
    obiekt-pyłek może być wykorzystany w wielu kontekstach. Stan
    przechowywany w pyłku nazywa się *wewnętrznym*. Stan przekazywany
    metodom pyłka to dane *zewnętrzne*.

3.  Klasa **Kontekst** zawiera opis zewnętrznego stanu, unikalny dla
    każdego z pierwotnych obiektów. Gdy kontekst skojarzy się z jednym z
    obiektów-pyłków, otrzymuje się reprezentację pełnego stanu
    pierwotnego obiektu.

4.  Zazwyczaj obowiązki pierwotnego obiektu pozostają w klasie pyłek. W
    takim przypadku, w momencie wywołania metody pyłka, trzeba przekazać
    jej również odpowiednie elementy stanu zewnętrznego. Z drugiej
    strony, obowiązki można przenieść do klasy kontekstowej, która
    korzysta ze skojarzonego pyłku tylko jako obiektu danych.

5.  **Klient** oblicza lub przechowuje zewnętrzny stan pyłków. Z punktu
    widzenia klienta, pyłek to obiekt szablonowy który może być
    skonfigurowany w trakcie działania programu poprzez przekazanie
    jakichś danych kontekstowych w charakterze parametrów jego metod.

6.  **Fabryka Pyłków** zarządza pulą istniejących pyłków. Dzięki
    fabryce, klienci nie tworzą pyłków w sposób bezpośredni. Zamiast
    tego wywołują fabrykę, przekazują jej fragmenty danych o wewnętrznym
    stanie pożądanego pyłka. Fabryka przegląda poprzednio stworzone
    pyłki i albo zwraca odpowiedni, albo go tworzy.
::::
:::::

::: {.section .pseudocode}
##  Pseudokod {#pseudocode}

W poniższym przykładzie, wzorzec **Pyłek** pomaga zredukować zużycie
pamięci podczas renderowania milionów obiektów-drzew na ekranie.

<figure class="image">
<img
src="../../images/patterns/diagrams/flyweight/example5053.png?id=0818d078c1a79f373e96397f37b7ee06"
style="aspect-ratio: 1.54;"
srcset="/images/patterns/diagrams/flyweight/example.png?id=0818d078c1a79f373e96397f37b7ee06 540w,/images/patterns/diagrams/flyweight/example-1.5x.png?id=449fa5906e277c134870c07e33dd4b62 810w,/images/patterns/diagrams/flyweight/example-2x.png?id=9423640fe3688a64201389b6e7aa1f48 1080w"
sizes="(max-width: 720px) 100vw, 540px" loading="lazy" width="540"
alt="Przykład użycia wzorca Pyłek" />
</figure>

Działając według wzorca, ekstrahuje się powtarzający, wewnętrzny stan z
głównej klasy `Drzewo` i przenosi do klasy pyłek o nazwie `TypDrzewa`.

Teraz, zamiast przechowywać te same dane w wielu obiektach, znajdują się
one tylko w kilku obiektach-pyłkach, skojarzonych ze stosownymi
obiektami `Drzewo` które służą za kontekst. Kod klienta tworzy nowe
drzewa za pośrednictwem fabryki pyłków, która hermetyzuje złożoność
poszukiwania odpowiedniego obiektu i jego ewentualnego ponownego użycia.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// Klasa pyłek zawiera część stanu drzewa. Pola te przechowują
// wartości które są unikalne dla każdego drzewa. Przykładowo
// nie znajdziemy tu współrzędnych drzewa, ale teksturę oraz
// wspólne barwy — owszem. Ponieważ te dane są zazwyczaj
// WIELKIE, zmarnowalibyśmy bardzo dużo pamięci operacyjnej,
// przechowując ich kopie w obrębie każdego z obiektów-drzew.
// Ekstrahujemy więc tekstury, barwy i inne powtarzające się
// dane do odrębnego obiektu. Wszystkie drzewa będą posiadać
// odniesienie do nowego obiektu.
class TreeType is
    field name
    field color
    field texture
    constructor TreeType(name, color, texture) { ... }
    method draw(canvas, x, y) is
        // 1. Utwórz mapę bitową o danym typie, kolorze i
        // teksturze.
        // 2. Narysuj mapę bitową na ekranie w punkcie o
        // współrzędnych X i Y.

// Fabryka pyłków podejmuje decyzję o ponownym użyciu
// istniejącego obiektu-pyłka lub utworzeniu nowego.
class TreeFactory is
    static field treeTypes: collection of tree types
    static method getTreeType(name, color, texture) is
        type = treeTypes.find(name, color, texture)
        if (type == null)
            type = new TreeType(name, color, texture)
            treeTypes.add(type)
        return type

// Obiekt-kontekst zawiera zewnętrzne elementy stanu drzewa.
// Aplikacja może stworzyć miliardy drzew, bo są one bardzo
// małe: opisują je dwie liczby całkowite oznaczające
// współrzędne i jedno pole przechowujące odniesienie do obiektu
// zawierającego opis stanu zewnętrznego.
class Tree is
    field x,y
    field type: TreeType
    constructor Tree(x, y, type) { ... }
    method draw(canvas) is
        type.draw(canvas, this.x, this.y)

// Klasy Tree i Forest są klientami pyłku. Możesz je połączyć,
// jeśli nie zamierzasz dalej rozwijać klasy Tree.
class Forest is
    field trees: collection of Trees

    method plantTree(x, y, name, color, texture) is
        type = TreeFactory.getTreeType(name, color, texture)
        tree = new Tree(x, y, type)
        trees.add(tree)

    method draw(canvas) is
        foreach (tree in trees) do
            tree.draw(canvas)</code></pre>
</figure>
:::

:::::: {.section .applicability-container}
##  Zastosowanie {#applicability}

::::: applicability
::: applicability-problem
Stosuj wzorzec Pyłek gdy twój program musi pracować z wielką ilością
obiektów, które ledwo mieszczą się w dostępnej pamięci RAM.
:::

::: applicability-solution
Zyski z wprowadzenia tego wzorca zależą od tego jak i gdzie się go
zastosuje. Największy pożytek uzyskuje się gdy:

-   aplikacja musi tworzyć wielką ilość podobnych obiektów,
-   powyższa sytuacja poważnie obciąża dostępną pamięć RAM urządzenia,
-   obiekty zawierają wielokrotnie powtarzające się opisy stanów, dające
    się wyekstrahować i pozwoli się na współdzielenie ich pomiędzy
    wieloma obiektami.
:::
:::::
::::::

::: {.section .checklist}
##  Jak zaimplementować {#checklist}

1.  Podziel na dwie części pola klasy z których powstanie pyłek:

    -   stan wewnętrzny: pola, które przechowają niezmienne dane,
        powtarzające się w wielu obiektach
    -   stan zewnętrzny: pola, które przechowają dane kontekstowe,
        unikalne dla każdego obiektu

2.  Pozostaw pola reprezentujące wewnętrzny stan w klasie, ale upewnij
    się, że nie mogą być zmieniane. Powinny one przyjmować swój stan
    początkowy wyłącznie w konstruktorze.

3.  Przejrzyj metody korzystające z pól zewnętrznego stanu. Dla każdego
    pola użytego w metodzie, dodaj nowy parametr i używaj go zamiast
    pola.

4.  Opcjonalnie, utwórz klasę fabryczną służącą zarządzaniu pulą pyłków.
    Powinna ona poszukać istniejącego pyłka przed utworzeniem nowego.
    Gdy fabryka jest już gotowa, klienci powinni wnioskować o pyłki
    wyłącznie przez nią, przekazując opis stanu wewnętrznego żądanego
    obiektu.

5.  Klient musi przechowywać lub wyliczać wartości opisujące stan
    zewnętrzny (kontekst), by mógł wywoływać metody obiektów-pyłków. Dla
    wygody, zewnętrzny stan wraz z polem odnoszącym się do pyłka można
    przenieść do osobnej klasy kontekstowej.
:::

:::::: {.section .pros-cons}
##  Zalety i wady {#pros-cons}

::::: row
::: col-sm-6
-    Możesz zaoszczędzić mnóstwo pamięci RAM, o ile twój program tworzy
    mnóstwo podobnych obiektów.
:::

::: col-sm-6
-    Może się zdarzyć, że oszczędność pamięci odbędzie się kosztem czasu
    procesora, gdyż część danych kontekstowych musi być wyliczana przy
    każdym wywołaniu metody pyłka.
-    Kod staje się dużo bardziej skomplikowany. Nowi członkowie
    zespołu z pewnością będą się zastanawiać dlaczego stan czegoś został
    odseparowany.
:::
:::::
::::::

::: {.section .relations}
##  Powiązania z innymi wzorcami {#relations}

-   Węzły będące liśćmi drzewa [Kompozytowego](composite.html) można
    zaimplementować jako [Pyłki](flyweight.html) by zaoszczędzić nieco
    pamięci RAM.

-   [Pyłek](flyweight.html) przedstawia sposób na stworzenie wielkiej
    liczby małych obiektów, zaś [Fasada](facade.html) na stworzenie
    pojedynczego obiektu reprezentującego cały podsystem.

-   [Pyłek](flyweight.html) mógłby przypominać
    [Singleton](singleton.html), gdybyśmy zdołali zredukować wszystkie
    współdzielone stany obiektów do tylko jednego obiektu-pyłka. Ale są
    jeszcze dwie fundamentalne różnice między tymi wzorcami:

    1.  Powinna istnieć tylko jedna instancja interfejsu Singleton, zaś
        instancji *Pyłka* będzie wiele, o różnym stanie wewnętrznym.
    2.  Obiekt *Singleton* może być zmienny. Pyłki są zaś niezmienne.
:::

::: {.section .implementations}
##  Przykłady kodu {#implementations}

[![Pyłek w języku
C#](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](flyweight/csharp/example.html "Pyłek w języku C#"){.prog-lang-link}
[![Pyłek w języku
C++](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](flyweight/cpp/example.html "Pyłek w języku C++"){.prog-lang-link}
[![Pyłek w języku
Go](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](flyweight/go/example.html "Pyłek w języku Go"){.prog-lang-link}
[![Pyłek w języku
Java](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](flyweight/java/example.html "Pyłek w języku Java"){.prog-lang-link}
[![Pyłek w języku
PHP](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](flyweight/php/example.html "Pyłek w języku PHP"){.prog-lang-link}
[![Pyłek w języku
Python](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](flyweight/python/example.html "Pyłek w języku Python"){.prog-lang-link}
[![Pyłek w języku
Ruby](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](flyweight/ruby/example.html "Pyłek w języku Ruby"){.prog-lang-link}
[![Pyłek w języku
Rust](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](flyweight/rust/example.html "Pyłek w języku Rust"){.prog-lang-link}
[![Pyłek w języku
Swift](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](flyweight/swift/example.html "Pyłek w języku Swift"){.prog-lang-link}
[![Pyłek w języku
TypeScript](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](flyweight/typescript/example.html "Pyłek w języku TypeScript"){.prog-lang-link}
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

[Pełnomocnik []{.fa .fa-arrow-right}](proxy.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Wróć

[[]{.fa .fa-arrow-left} Fasada](facade.html){.btn .btn-default
rel="prev"}
:::
:::::::::::::::::::::::::::::

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
:::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::
