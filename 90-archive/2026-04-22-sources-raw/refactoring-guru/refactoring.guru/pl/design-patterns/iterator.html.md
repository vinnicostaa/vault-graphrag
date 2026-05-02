::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [Wzorce
projektowe](../design-patterns.html){.type} / [Wzorce
behawioralne](behavioral-patterns.html){.type}
:::

# Iterator {#iterator .title}

::: aka
Znany też jako: [Kursor]{style="display:inline-block"}
:::

::: {.section .intent}
##  Cel {#intent}

**Iterator** to behawioralny wzorzec projektowy, pozwalający
sekwencyjnie przechodzić od elementu do elementu jakiegoś zbioru bez
konieczności eksponowania jego formy (lista, stos, drzewo, itp.).

<figure class="image">
<img
src="../../images/patterns/content/iterator/iterator-pl8c26.png?id=4c5344c41493e1ac4a5c81f5acfdbdea"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/iterator/iterator-pl.png?id=4c5344c41493e1ac4a5c81f5acfdbdea 640w,/images/patterns/content/iterator/iterator-pl-1.5x.png?id=46c788bced28e3b598f1033b3bd81a69 960w,/images/patterns/content/iterator/iterator-pl-2x.png?id=15dc4727957b76fe843e78da5ffc535f 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Wzorzec projektowy Iterator" />
</figure>
:::

::: {.section .problem}
##  Problem

Kolekcje są jednym z najpopularniejszych typów danych wśród
programistów, mimo że są to po prostu kontenery przechowujące grupy
obiektów.

<figure class="image">
<img
src="../../images/patterns/diagrams/iterator/problem14cac.png?id=52ef2fe2d4920e3fed696c221fe757f2"
style="aspect-ratio: 4.9;"
srcset="/images/patterns/diagrams/iterator/problem1.png?id=52ef2fe2d4920e3fed696c221fe757f2 490w,/images/patterns/diagrams/iterator/problem1-1.5x.png?id=b46e39ade7de292fdcacc9790066c72f 735w,/images/patterns/diagrams/iterator/problem1-2x.png?id=1f4384c3c631be238bfc23d6eb84a0ef 980w"
sizes="(max-width: 720px) 100vw, 490px" loading="lazy" width="490"
alt="Różne rodzaje kolekcji" />
<figcaption><p>Różne rodzaje kolekcji.</p></figcaption>
</figure>

Większość kolekcji przechowuje swoje elementy w prostych listach, ale
niektóre są zbudowane w oparciu o stosy, drzewa, grafy i inne złożone
struktury.

Jednak niezależnie od struktury kolekcji musi ona udostępniać dostęp do
swoich elementów tak, aby można było używać ich gdzie indziej w
programie. Powinien istnieć jakiś sposób przejścia po każdym elemencie
kolekcji bez konieczności powtórnego dostępu do jakiegoś elementu.

Wydaje się to łatwe, jeśli mamy do czynienia z kolekcją
ustrukturyzowaną w formie listy. Tworzymy pętlę przechodzącą po
elementach i gotowe. Ale jak przejść sekwencyjnie, element po elemencie,
przez złożoną strukturę, jak np. drzewo? Na początku być może wystarczy
poruszanie się w głąb, ale kolejnego dnia pojawi się wymóg przechodzenia
wszerz. Jeszcze później okazuje się, że przydałaby się możliwość
losowego dostępu do dowolnego elementu drzewa.

<figure class="image">
<img
src="../../images/patterns/diagrams/iterator/problem29f71.png?id=f9c1a746c787320291c85fdc2a314192"
style="aspect-ratio: 3.75;"
srcset="/images/patterns/diagrams/iterator/problem2.png?id=f9c1a746c787320291c85fdc2a314192 600w,/images/patterns/diagrams/iterator/problem2-1.5x.png?id=7a003d020e789773e0c833d7b1df00e6 900w,/images/patterns/diagrams/iterator/problem2-2x.png?id=656b13c109d4d4960ea1f9fa993bae4c 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Różne algorytmy sekwencyjnego poruszania się" />
<figcaption><p>Zawartość tej samej kolekcji można przejrzeć na wiele
różnych sposobów.</p></figcaption>
</figure>

Dodawanie kolejnych algorytmów przechodzenia przez kolekcję stopniowo
przyćmiewa jej główne zadanie, którym jest efektywne przechowywanie
danych. Ponadto niektóre algorytmy mogą być zoptymalizowane pod kątem
konkretnego zastosowania, więc włączanie ich do uogólnionej klasy
kolekcji byłoby dziwne.

Z drugiej strony, kod klienta który ma za zadanie działać na różnych
kolekcjach może nawet nie być zainteresowany sposobem w jaki przechowują
one elementy. Jednak skoro wszystkie kolekcje udostępniają różne sposoby
dostępu do swoich elementów, to nie ma innego wyjścia, niż związać swój
kod z klasą konkretnej kolekcji.
:::

::: {.section .solution}
##  Rozwiązanie {#solution}

Główną ideą wzorca Iterator jest ekstrakcja zadań związanych z
przechodzeniem przez elementy kolekcji do osobnego obiektu zwanego
*iteratorem*.

<figure class="image">
<img
src="../../images/patterns/diagrams/iterator/solution19739.png?id=2f5fbcce6099d8ea09b2fbb83e3e7059"
style="aspect-ratio: 0.85;"
srcset="/images/patterns/diagrams/iterator/solution1.png?id=2f5fbcce6099d8ea09b2fbb83e3e7059 400w,/images/patterns/diagrams/iterator/solution1-1.5x.png?id=68612372ede6e5029403d38b381fdc05 600w,/images/patterns/diagrams/iterator/solution1-2x.png?id=dcebcad4c197bfa5f25f680382d0e5ac 800w"
sizes="(max-width: 720px) 100vw, 400px" loading="lazy" width="400"
alt="Iteratory implementują różne algorytmy sekwencyjnego dostępu." />
<figcaption><p>Iteratory implementują różne algorytmy sekwencyjnego
dostępu do kolejnych elementów. Wiele obiektów iterator może
przeskakiwać po elementach jednej
kolekcji jednocześnie.</p></figcaption>
</figure>

Oprócz implementowania samego algorytmu, obiekt iteratora hermetyzuje
wszystkie szczegóły sposobu przechodzenia przez kolejne elementy, jak
bieżąca pozycja, czy ilość pozostałych elementów. Dzięki temu wiele
iteratorów może jednocześnie przeglądać tę samą kolekcję, niezależnie od
siebie.

Zazwyczaj, iteratory udostępniają jedną główną metodę pobierającą
elementy kolekcji. Klient może wywoływać ją raz za razem aż przestanie
ona zwracać kolejne obiekty, co oznacza osiągnięcie końca zbioru.

Wszystkie iteratory muszą implementować ten sam interfejs. Czyni to kod
klienta kompatybilnym z dowolną kolekcją czy algorytmem przechodzenia o
ile istnieje odpowiedni iterator. Jeśli potrzebny jest specjalny sposób
przeglądania kolekcji, można stworzyć nową klasę iteratora, bez
konieczności dokonywania zmian w kolekcji lub w kodzie klienta.
:::

::: {.section .analogy}
##  Analogia do prawdziwego życia {#analogy}

<figure class="image">
<img
src="../../images/patterns/content/iterator/iterator-comic-1-pl7cba.png?id=f27a521b92852badad884b753a6f2cb1"
style="aspect-ratio: 2.13;"
srcset="/images/patterns/content/iterator/iterator-comic-1-pl.png?id=f27a521b92852badad884b753a6f2cb1 640w,/images/patterns/content/iterator/iterator-comic-1-pl-1.5x.png?id=d450120f5958af5bb4c02146b973d3bb 960w,/images/patterns/content/iterator/iterator-comic-1-pl-2x.png?id=bb7b3f71dd6cbe03631150026e49c52d 1280w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="640"
alt="Różne sposoby zwiedzania Rzymu" />
<figcaption><p>Różne sposoby zwiedzania Rzymu.</p></figcaption>
</figure>

Zamierzasz odwiedzić Rzym na parę dni i zwiedzić wszystkie najważniejsze
miejsca i atrakcje turystyczne. Ale na miejscu łatwo zmarnować sporo
czasu chodząc w kółko, nie mogąc znaleźć nawet Koloseum.

Z drugiej strony, można kupić aplikację wirtualnego przewodnika na
smartfon i nawigować z jej pomocą. Sprytne i niedrogie, a dodatkowo
można zatrzymać się przy dowolnym miejscu na ile się chce.

Trzecia alternatywa to poświęcenie części swojego wycieczkowego
budżetu na wynajęcie przewodnika który zna miasto jak własną kieszeń.
Przewodnik mógłby dostosować trasę wycieczki do twoich preferencji,
pokazać wszystko co najciekawsze i opowiedzieć wiele ciekawych historii.
Byłoby miło, ale również i drożej.

Wszystkie te opcje --- losowo obrane kierunki, smartfonowy przewodnik i
wynajęty przewodnik --- stanowią iteratory pozwalające na dostęp do
wielkiej kolekcji widoków i atrakcji Rzymu.
:::

::::: {.section .structure-container}
##  Struktura {#structure}

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/iterator/structure5247.png?id=35ea851f8f6bbe51d79eb91e6e6519d0"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.12;"
srcset="/images/patterns/diagrams/iterator/structure.png?id=35ea851f8f6bbe51d79eb91e6e6519d0 480w,/images/patterns/diagrams/iterator/structure-1.5x.png?id=19d4c2130ab2e3bd718f87e956fcd873 720w,/images/patterns/diagrams/iterator/structure-2x.png?id=b7b4708d3f279dd046eb1692f1cba710 960w"
sizes="(max-width: 720px) 100vw, 480px" loading="lazy" width="480"
alt="Struktura wzorca projektowego Iterator" /><img
src="../../images/patterns/diagrams/iterator/structure-indexed3472.png?id=7bc28907ff6b480db6635a93ebaa10ff"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.33;"
srcset="/images/patterns/diagrams/iterator/structure-indexed.png?id=7bc28907ff6b480db6635a93ebaa10ff 520w,/images/patterns/diagrams/iterator/structure-indexed-1.5x.png?id=32fde4c4c1cbfbe07aa6e716e5ac2346 780w,/images/patterns/diagrams/iterator/structure-indexed-2x.png?id=d809428b2262e2013972fe69d2434473 1040w"
sizes="(max-width: 720px) 100vw, 520px" loading="lazy" width="520"
alt="Struktura wzorca projektowego Iterator" />
</figure>
:::

1.  Interfejs **Iterator** deklaruje działania niezbędne do
    sekwencyjnego przechodzenia przez elementy kolekcji: pobieranie
    kolejnego elementu, ustalenie bieżącej pozycji, powrót do pierwszego
    elementu, itp.

2.  **Konkretne Iteratory** implementują specyficzne algorytmy
    przeglądania kolekcji. Obiekt iterator powinien samodzielnie śledzić
    postęp tego procesu. Pozwala to wielu iteratorom na jednoczesne
    przeglądanie tej samej kolekcji.

3.  Interfejs **Kolekcja** deklaruje jedną lub więcej metod służących
    kompatybilności kolekcji z iteratorami. Zwracany typ metody musi być
    zadeklarowany jako interfejs iterator, aby konkretne kolekcje mogły
    zwracać różne rodzaje iteratorów.

4.  **Konkretne Kolekcje** zwracają nowe instancje konkretnych klas
    iteratorów za każdym razem gdy klient ich zażąda. Pewnie ciekawi cię
    gdzie jest reszta kodu kolekcji? Nie martw się, powinien znajdować
    się w tej samej klasie. Po prostu te szczegóły nie są istotne dla
    konkretnego wzorca, więc je pomijamy.

5.  **Klient** współpracuje zarówno z kolekcjami jak i iteratorami za
    pośrednictwem ich interfejsów. Dzięki temu nie jest sprzężony z
    konkretnymi klasami, pozwalając na pracę z różnymi kolekcjami i
    operatorami w tym samym kodzie klienta.

    Zazwyczaj klienci nie tworzą iteratorów sami, lecz otrzymują je od
    kolekcji. Ale w pewnych przypadkach klient może stworzyć iterator
    bezpośrednio --- na przykład gdy sam definiuje swój specjalny
    iterator.
::::
:::::

::: {.section .pseudocode}
##  Pseudokod {#pseudocode}

W poniższym przykładzie, wzorzec **Iterator** służy przejściu przez
specjalny typ kolekcji, który hermetyzuje dostęp do diagramu relacji
pomiędzy profilami na Facebooku. Kolekcja udostępnia wiele iteratorów
które mogą przeglądać kolejne profile na różne sposoby.

<figure class="image">
<img
src="../../images/patterns/diagrams/iterator/examplee0a2.png?id=f2a24ef3787bf80ed450709240506ff2"
style="aspect-ratio: 0.95;"
srcset="/images/patterns/diagrams/iterator/example.png?id=f2a24ef3787bf80ed450709240506ff2 600w,/images/patterns/diagrams/iterator/example-1.5x.png?id=cab0e1459ffc3721579a3e7d6a4e9022 900w,/images/patterns/diagrams/iterator/example-2x.png?id=73c3e55f75ca0250bd84e8a1d8cc88d2 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Struktura przykładu użycia wzorca Iterator" />
<figcaption><p>Przykład iteracji po kolejnych
profilach użytkowników.</p></figcaption>
</figure>

Iterator `przyjaciele` może służyć przeglądaniu zaprzyjaźnionych profili
danej osoby. `Współpracownicy` robi to samo, ale pomija osoby które nie
pracują wraz ze wskazaną osobą. Oba iteratory implementują wspólny
interfejs który pozwala klientom pobierać profile bez zagłębiania się w
szczegóły implementacyjne takie jak uwierzytelnianie, czy wysyłanie
żądań REST.

Kod klienta nie jest sprzężony z konkretnymi klasami, ponieważ
współpracuje z kolekcjami i iteratorami wyłącznie za pośrednictwem
interfejsów. Jeśli postanowisz połączyć swoją aplikację z nowym portalem
społecznościowym, musisz jedynie dostarczyć odpowiednie klasy kolekcji i
iteratora, bez konieczności dokonywania zmian istniejącego kodu.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// Interfejs kolekcja musi deklarować metodę wytwórczą do
// produkowania iteratorów. Można zadeklarować wiele metod jeśli
// potrzebujesz kilku różnych sposobów iterowania.
interface SocialNetwork is
    method createFriendsIterator(profileId):ProfileIterator
    method createCoworkersIterator(profileId):ProfileIterator


// Każda konkretna kolekcja jest powiązana z zestawem
// konkretnych klas iteratorów jakie zwraca. Ale klient nie jest
// z nimi powiązany, bo sygnatury tych metod zwracają typ
// interfejs iteratora.
class Facebook implements SocialNetwork is
    // ... Większość kodu kolekcji powinna znaleźć się tutaj ...

    // Kod kreacyjny iteratora.
    method createFriendsIterator(profileId) is
        return new FacebookIterator(this, profileId, &quot;friends&quot;)
    method createCoworkersIterator(profileId) is
        return new FacebookIterator(this, profileId, &quot;coworkers&quot;)


// Wspólny interfejs wszystkich iteratorów.
interface ProfileIterator is
    method getNext():Profile
    method hasMore():bool


// Konkretna klasa iteratora.
class FacebookIterator implements ProfileIterator is
    // Iterator potrzebuje odniesienia do kolekcji po elementach
    // której ma przechodzić.
    private field facebook: Facebook
    private field profileId, type: string

    // Obiekt iteratora przechodzi po elementach kolekcji
    // niezależnie od innych iteratorów. Dlatego musi
    // przechowywać stan iteracji.
    private field currentPosition
    private field cache: array of Profile

    constructor FacebookIterator(facebook, profileId, type) is
        this.facebook = facebook
        this.profileId = profileId
        this.type = type

    private method lazyInit() is
        if (cache == null)
            cache = facebook.socialGraphRequest(profileId, type)

    // Każda konkretna klasa iteratora posiada własną
    // implementację wspólnego interfejsu iteratora.
    method getNext() is
        if (hasMore())
            result = cache[currentPosition]
            currentPosition++
            return result

    method hasMore() is
        lazyInit()
        return currentPosition &lt; cache.length


// Oto kolejna użyteczna sztuczka: możesz przekazać iterator
// klasie klienckiej zamiast dawać jej dostęp do całej kolekcji.
// Dzięki temu nie trzeba eksponować całej kolekcji klientowi.
//
// Kolejna zaleta takiego podejścia to możliwość zmiany sposobu
// w jaki klient współpracuje z kolekcją w trakcie działania
// programu poprzez przekazanie klientowi innego iteratora. Jest
// to możliwe ponieważ kod klienta nie jest sprzęgnięty z
// konkretnymi klasami iteratora.
class SocialSpammer is
    method send(iterator: ProfileIterator, message: string) is
        while (iterator.hasMore())
            profile = iterator.getNext()
            System.sendEmail(profile.getEmail(), message)


// Klasa aplikacji konfiguruje kolekcje i iteratory, a następnie
// przekazuje je kodowi klienta.
class Application is
    field network: SocialNetwork
    field spammer: SocialSpammer

    method config() is
        if working with Facebook
            this.network = new Facebook()
        if working with LinkedIn
            this.network = new LinkedIn()
        this.spammer = new SocialSpammer()

    method sendSpamToFriends(profile) is
        iterator = network.createFriendsIterator(profile.getId())
        spammer.send(iterator, &quot;Very important message&quot;)

    method sendSpamToCoworkers(profile) is
        iterator = network.createCoworkersIterator(profile.getId())
        spammer.send(iterator, &quot;Very important message&quot;)</code></pre>
</figure>
:::

:::::::::: {.section .applicability-container}
##  Zastosowanie {#applicability}

::::::::: applicability
::: applicability-problem
Stosuj wzorzec Iterator gdy kolekcja z którą masz do czynienia posiada
skomplikowaną strukturę, ale zależy ci na ukryciu jej przed klientem
(dla wygody, lub dla bezpieczeństwa).
:::

::: applicability-solution
Iterator hermetyzuje szczegóły współpracy ze złożonymi strukturami
danych, dając klientowi pewną liczbę prostych metod służących
dostępowi do elementów kolekcji. To podejście jest dla klienta wygodne,
ale również chroni kolekcję przed nieuważnym lub złośliwym działaniem,
którego ryzyko istnieje przy bezpośredniej pracy ze strukturą.
:::

::: applicability-problem
Stosuj wzorzec w celu redukcji duplikowania kodu przeglądania elementów
zbiorów na przestrzeni całego programu.
:::

::: applicability-solution
Kod nietrywialnych algorytmów iteracji bywa obszerny. Gdy umieści się go
w ramach logiki biznesowej aplikacji, zwykle zaciera główną
odpowiedzialność pierwotnego kodu i czyni go trudniejszym do utrzymania.
Przeniesienie kodu przeglądania elementów do stosownych iteratorów
pomaga uczynić kod aplikacji czystszym i prostszym.
:::

::: applicability-problem
Stosuj Iterator gdy chcesz, aby twój kod był w stanie przeglądać
elementy różnych struktur danych, lub gdy nie znasz z góry szczegółów
ich struktury.
:::

::: applicability-solution
Wzorzec Iterator udostępnia parę ogólnych interfejsów zarówno kolekcji,
jak i iteratorów. Skoro twój kod korzysta z tych interfejsów, to będzie
nadal działał, nawet gdy przekażesz mu różne rodzaje kolekcji i
iteratorów, o ile implementują te interfejsy.
:::
:::::::::
::::::::::

::: {.section .checklist}
##  Jak zaimplementować {#checklist}

1.  Zadeklaruj interfejs iteratora. W najprostszym przypadku musi
    posiadać metodę pobierającą kolejny element kolekcji. Ale dla wygody
    możesz dodać parę innych metod, np. do pobierania poprzedniego
    elementu, śledzenia bieżącej pozycji i ustalenia końca procesu
    iteracji.

2.  Zadeklaruj interfejs kolekcji i opisz metodę pobierającą iteratory.
    Typ zwracany przez metodę powinien być zgodny z interfejsem
    iteratora. Możesz zadeklarować podobne metody, jeśli zamierzasz mieć
    wiele różnych grup iteratorów.

3.  Zaimplementuj konkretne klasy iterator dla kolekcji które powinny
    być możliwe do przeglądania za pomocą iteratorów. Obiekt iterator
    musi być powiązany z jedną instancją kolekcji. Zazwyczaj tworzy się
    takie powiązanie w konstruktorze iteratora.

4.  Zaimplementuj interfejs kolekcji w swoich klasach kolekcji. Główną
    ideą jest udostępnienie klientowi skrótu do tworzenia iteratorów
    optymalnych dla danej klasy kolekcji. Obiekt kolekcji musi
    przekazywać siebie samego do konstruktora iteratora aby stworzyć
    między nimi powiązanie.

5.  Przejrzyj swój kod klienta i zamień cały kod przeglądania
    kolekcji na wykorzystujący iteratory. Klient pobiera nowy obiekt
    iteratora za każdym razem gdy chce przejrzeć zawartość kolekcji.
:::

:::::: {.section .pros-cons}
##  Zalety i wady {#pros-cons}

::::: row
::: col-sm-6
-    *Zasada pojedynczej odpowiedzialności*. Można uprzątnąć kod
    klienta i kolekcje, ekstrahując obszerny kod przeglądania do
    osobnych klas.
-    *Zasada otwarte/zamknięte*. Można zaimplementować nowe typy
    kolekcji i iteratorów oraz przekazywać je do istniejącego kodu bez
    psucia czegokolwiek.
-    Można przeglądać tę samą kolekcję równolegle wieloma iteratorami,
    gdyż każdy z nich przechowuje informacje o swoim stanie.
-    Z powyższego powodu można opóźniać iterację i kontynuować ją gdy
    zachodzi taka potrzeba.
:::

::: col-sm-6
-    Zastosowanie tego wzorca będzie przesadą jeśli twoja aplikacja
    korzysta wyłącznie z prostych kolekcji.
-    Używanie iteratora może być mniej efektywne niż bezpośrednie
    przejście po elementach jakiejś wyspecjalizowanej kolekcji.
:::
:::::
::::::

::: {.section .relations}
##  Powiązania z innymi wzorcami {#relations}

-   [Iteratory](iterator.html) służą do sekwencyjnego przemieszczania
    się po drzewie [Kompozytowym](composite.html) element po elemencie.

-   Możesz zastosować [Metodę wytwórczą](factory-method.html) wraz z
    [Iteratorem](iterator.html) aby pozwolić podklasom kolekcji zwracać
    różne typy iteratorów kompatybilnych z kolekcją.

-   Można zastosować [Pamiątkę](memento.html) wraz z
    [Iteratorem](iterator.html) by zapisać bieżący stan iteracji, co
    pozwoli w razie potrzeby do niego powrócić.

-   Połączenie [Odwiedzającego](visitor.html) z
    [Iteratorem](iterator.html) może służyć sekwencyjnemu przeglądowi
    elementów złożonej struktury danych i wykonaniu na nich jakiegoś
    działania, nawet jeśli te elementy są obiektami różnych klas.
:::

::: {.section .implementations}
##  Przykłady kodu {#implementations}

[![Iterator w języku
C#](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](iterator/csharp/example.html "Iterator w języku C#"){.prog-lang-link}
[![Iterator w języku
C++](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](iterator/cpp/example.html "Iterator w języku C++"){.prog-lang-link}
[![Iterator w języku
Go](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](iterator/go/example.html "Iterator w języku Go"){.prog-lang-link}
[![Iterator w języku
Java](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](iterator/java/example.html "Iterator w języku Java"){.prog-lang-link}
[![Iterator w języku
PHP](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](iterator/php/example.html "Iterator w języku PHP"){.prog-lang-link}
[![Iterator w języku
Python](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](iterator/python/example.html "Iterator w języku Python"){.prog-lang-link}
[![Iterator w języku
Ruby](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](iterator/ruby/example.html "Iterator w języku Ruby"){.prog-lang-link}
[![Iterator w języku
Rust](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](iterator/rust/example.html "Iterator w języku Rust"){.prog-lang-link}
[![Iterator w języku
Swift](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](iterator/swift/example.html "Iterator w języku Swift"){.prog-lang-link}
[![Iterator w języku
TypeScript](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](iterator/typescript/example.html "Iterator w języku TypeScript"){.prog-lang-link}
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

[Mediator []{.fa .fa-arrow-right}](mediator.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Wróć

[[]{.fa .fa-arrow-left} Polecenie](command.html){.btn .btn-default
rel="prev"}
:::
::::::::::::::::::::::::::::::::::

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
::::::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::::::
