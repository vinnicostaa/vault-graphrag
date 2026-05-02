::::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [Wzorce
projektowe](../design-patterns.html){.type} / [Wzorce
behawioralne](behavioral-patterns.html){.type}
:::

# Pamiątka {#pamiątka .title}

::: aka
Znany też jako:
[Snapshot, ]{style="display:inline-block"}[Memento]{style="display:inline-block"}
:::

::: {.section .intent}
##  Cel {#intent}

**Pamiątka** to behawioralny wzorzec projektowy pozwalający zapisywać i
przywracać wcześniejszy stan obiektu bez ujawniania szczegółów
jego implementacji.

<figure class="image">
<img
src="../../images/patterns/content/memento/memento-pl5a05.png?id=cddcca7cbf1de242485296f48a74ac9d"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/memento/memento-pl.png?id=cddcca7cbf1de242485296f48a74ac9d 640w,/images/patterns/content/memento/memento-pl-1.5x.png?id=cc1de694b0dc6f7100649af310f7b8f3 960w,/images/patterns/content/memento/memento-pl-2x.png?id=a4b60d1d1276f702c843b7b06b004227 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Wzorzec projektowy Pamiątka" />
</figure>
:::

::: {.section .problem}
##  Problem

Wyobraź sobie, że tworzysz edytor tekstu. Poza zwykłym edytowaniem
treści, edytor może ją formatować, wstawiać obrazki, itd.

W jakimś momencie postanawiasz pozwolić użytkownikom cofać dowolną
operację wykonaną na tekście. Funkcja taka stała się przez lata
popularna, a użytkownicy oczekują jej w każdej aplikacji. W celu
implementacji obierasz podejście bezpośrednie. Przed wykonaniem
dowolnego działania, aplikacja zapamiętuje stan wszystkich obiektów i
zapisuje go w jakimś magazynie. Gdy użytkownik zechce wycofać jakąś
zmianę, aplikacja pobiera ostatnią migawkę z historii i za jej pomocą
przywraca wcześniejszy stan wszystkich obiektów.

<figure class="image">
<img
src="../../images/patterns/diagrams/memento/problem1-pl2379.png?id=7b98fe9aa8cd6b4d087e4bccf85a9806"
style="aspect-ratio: 2.61;"
srcset="/images/patterns/diagrams/memento/problem1-pl.png?id=7b98fe9aa8cd6b4d087e4bccf85a9806 470w,/images/patterns/diagrams/memento/problem1-pl-1.5x.png?id=1fddabacacca6da6500313d1f82ba2b4 705w,/images/patterns/diagrams/memento/problem1-pl-2x.png?id=46624cb4b612544ef67f6009c239dcdc 940w"
sizes="(max-width: 720px) 100vw, 470px" loading="lazy" width="470"
alt="Cofanie działań w edytorze" />
<figcaption><p>Przed wykonaniem działania, aplikacja zapisuje migawkę
stanu obiektów. Pozwoli ona później przywrócić obiekty do ich
poprzedniego stanu.</p></figcaption>
</figure>

Pomyślmy o tych migawkach. Jak dokładnie je tworzyć? Prawdopodobnie
trzeba byłoby przejrzeć wszystkie pola obiektu, skopiować ich wartości i
zapisać w jakimś magazynie. Jednak to zadziałałoby tylko jeśli obiekt ma
stosunkowo luźne ograniczenia dostępu do swojej zawartości. Niestety,
większość prawdziwych obiektów nie pozwoli obcym tak łatwo zajrzeć w
swoją zawartość i najistotniejsze dane będą ukryte w polach prywatnych.

Na razie zignorujmy jednak ten problem i załóżmy, że nasze obiekty są
niczym hipisi: wolą otwarte związki i nie ukrywają swojej natury.
Chociaż podejście takie pomija powyższe ograniczenie i pozwala wykonać
migawkę stanu obiektów, to tworzy przy okazji inne problemy. W
przyszłości bowiem możemy zdecydować o refaktoryzacji niektórych klas
edytora, lub dodać, bądź usunąć pola. Brzmi łatwo, ale wymagałoby przy
okazji zmiany klas odpowiedzialnych za kopiowanie stanów zmienianych
obiektów.

<figure class="image">
<img
src="../../images/patterns/diagrams/memento/problem2-pl92a1.png?id=b8947e54b9419e1cf151f55ac0fac0d3"
style="aspect-ratio: 1.43;"
srcset="/images/patterns/diagrams/memento/problem2-pl.png?id=b8947e54b9419e1cf151f55ac0fac0d3 300w,/images/patterns/diagrams/memento/problem2-pl-1.5x.png?id=9652ada3a632c256b4a1e397df260018 450w,/images/patterns/diagrams/memento/problem2-pl-2x.png?id=ba4a4cab5add2e425c254f1cef6fd9cb 600w"
sizes="(max-width: 720px) 100vw, 300px" loading="lazy" width="300"
alt="Jak skopiować prywatną część stanu obiektu?" />
<figcaption><p>Jak skopiować prywatną część
stanu obiektu?</p></figcaption>
</figure>

Ale jest coś jeszcze. Rozważmy samą "migawkę" stanu edytora. Jakie dane
zawierałaby? W najprostszej formie: tekst, współrzędne kursora, bieżącą
pozycję przewijania, itd. Aby wykonać migawkę, trzeba zebrać te
wartości i umieścić je w jakimś kontenerze.

Najprawdopodobniej będzie trzeba przechowywać mnóstwo takich obiektów
kontenerowych w formie listy reprezentującej historię. Dlatego też
kontenery zapewne byłyby obiektami jednej klasy. Klasa ta nie miałaby
prawie żadnej metody, ale za to wiele pól, odzwierciedlających stan
edytora. Aby pozwolić innym obiektom zapisywać i odczytywać dane
migawki, trzeba by uczynić jej pola publicznymi. Ale to ujawniłoby
wszystkie stany edytora, także te prywatne. Inne klasy stałyby się
zależne od najdrobniejszej zmiany klasy migawki, które w przeciwnym
wypadku działyby się w obrębie jej prywatnych pól i metod bez wpływu na
zewnętrzne klasy.

Wygląda na to, że trafiliśmy w ślepą uliczkę: można albo eksponować
wszystkie wewnętrzne szczegóły klas, czyniąc je delikatnymi, albo
ograniczyć dostęp do ich stanu, uniemożliwiając tym samym wykonywanie
migawek. Czy jest jakiś inny sposób na implementację \"cofnij\"?
:::

::: {.section .solution}
##  Rozwiązanie {#solution}

Wszystkie problemy na jakie się natknęliśmy są spowodowane niewłaściwą
hermetyzacją. Niektóre obiekty próbują robić więcej niż powinny. Aby
zbierać dane w celu wykonania jakiegoś zadania, wkradają się w prywatną
przestrzeń innych obiektów, zamiast pozwolić im na samodzielne wykonanie
tego zadania.

Wzorzec Pamiątka deleguje tworzenie migawki stanu samemu właścicielowi
stanu --- obiektowi *źródło*. Dlatego też, zamiast pozwalać innym
obiektom próbować skopiować stan edytora "z zewnątrz", sama klasa
edytora może wykonać migawkę siebie, gdyż ma pełny dostęp do swojego
stanu.

Wzorzec proponuje przechowywanie kopii stanu w specjalnym obiekcie
zwanym *pamiątką*. Zawartość pamiątki nie jest dostępna innym obiektom,
oprócz jej twórcy. Inne obiekty muszą komunikować się z pamiątką za
pośrednictwem ograniczonego interfejsu, który pozwala na pobieranie
metadanych migawki (czas utworzenia, nazwa wykonanej operacji, itd.),
ale nie stanu pierwotnego obiektu zawartego w migawce.

<figure class="image">
<img
src="../../images/patterns/diagrams/memento/solution-plb69a.png?id=299e28d67a5e81eeb54653e8a86b93fd"
style="aspect-ratio: 1.36;"
srcset="/images/patterns/diagrams/memento/solution-pl.png?id=299e28d67a5e81eeb54653e8a86b93fd 610w,/images/patterns/diagrams/memento/solution-pl-1.5x.png?id=c274740e8520707801933a8cb8afff9f 915w,/images/patterns/diagrams/memento/solution-pl-2x.png?id=d9d1ef34dc8a7b80b4eb95ef049c0e72 1220w"
sizes="(max-width: 720px) 100vw, 610px" loading="lazy" width="610"
alt="Źródło ma pełen dostęp do pamiątki, zaś zarządca tylko do jej metadanych" />
<figcaption><p>Źródło ma pełen dostęp do pamiątki, zaś zarządca tylko do
jej metadanych.</p></figcaption>
</figure>

Tak restrykcyjna polityka pozwala przechowywać pamiątki w obrębie innych
obiektów, zwykle zwanych *zarządcami*. Skoro zarządca współpracuje z
pamiątką tylko za pośrednictwem ograniczonego interfejsu, to nie jest w
stanie naruszyć stanu w niej przechowywanego. Ponadto, źródło ma pełen
dostęp do wszystkich pól pamiątki, co pozwala na przywracanie
poprzednich stanów wedle potrzeb.

W naszym przykładzie edytora tekstowego możemy stworzyć osobną klasę
historii która przyjmie rolę zarządcy. Stos pamiątek przechowany w
zarządcy powiększy się przed każdą operacją edytora. Można nawet
przedstawić reprezentację tego stosu w graficznym interfejsie
użytkownika aplikacji, wyświetlając historię wcześniej wykonanych
operacji.

Gdy użytkownik wywoła wycofanie operacji, historia pobierze najnowszą
pamiątkę ze stosu i przekaże ją edytorowi z żądaniem cofnięcia.
Edytor ma pełny dostęp do pamiątki, więc zmieni swój stan w oparciu o
wartości w niej zawarte.
:::

:::::::::: {.section .structure-container}
##  Struktura {#structure}

::::::::: structure
#### Implementacja w oparciu o zagnieżdżone klasy

Klasyczna implementacja wzorca polega na zagnieżdżaniu klas, które jest
możliwe w wielu popularnych językach programowania (jak C++, C# i Java).

:::: memento-classic
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/memento/structure180c1.png?id=4b4a42363a005b617d4df06689787385"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.87;"
srcset="/images/patterns/diagrams/memento/structure1.png?id=4b4a42363a005b617d4df06689787385 580w,/images/patterns/diagrams/memento/structure1-1.5x.png?id=82cf757f153840c85d27bd63f3f3e133 870w,/images/patterns/diagrams/memento/structure1-2x.png?id=d4e77295e51c2417f22b7abb396d5977 1160w"
sizes="(max-width: 720px) 100vw, 580px" loading="lazy" width="580"
alt="Pamiątka w oparciu o zagnieżdżanie klas" /><img
src="../../images/patterns/diagrams/memento/structure1-indexeda7c3.png?id=f79a8356b087ae6b004aec42b787ae2e"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.87;"
srcset="/images/patterns/diagrams/memento/structure1-indexed.png?id=f79a8356b087ae6b004aec42b787ae2e 580w,/images/patterns/diagrams/memento/structure1-indexed-1.5x.png?id=0687dc84dd4c98b4591a70ebd05c4d8e 870w,/images/patterns/diagrams/memento/structure1-indexed-2x.png?id=62fea7bdbc96420568869ea3bd25f6ad 1160w"
sizes="(max-width: 720px) 100vw, 580px" loading="lazy" width="580"
alt="Pamiątka w oparciu o zagnieżdżanie klas" />
</figure>
:::
::::

1.  Klasa **Źródło** może tworzyć migawki swego stanu, a także
    przywracać wcześniejszy stan z migawek, gdy zachodzi taka potrzeba.

2.  **Pamiątka** to obiekt wartości pełniący rolę pamiątki stanu źródła.
    Popularną praktyką jest czynienie pamiątki niezmienialną i
    ustawianie jej danych jednorazowo --- poprzez konstruktor.

3.  **Zarządca** wie nie tylko "kiedy" i "po co" rejestrować stan
    zarządcy, ale także kiedy należy przywrócić wcześniejszy stan.

    Zarządca może śledzić historię źródła przechowując stos pamiątek.
    Gdy źródło musi wrócić się w przeszłość, zarządca pobiera najnowszą
    pamiątkę ze stosu i przekazuje ją metodzie przywracającej źródła.

4.  W tej implementacji klasa pamiątka jest zagnieżdżona wewnątrz klasy
    źródła. Pozwala to źródłu mieć dostęp do pól i metod pamiątki,
    mimo że są zadeklarowane jako prywatne. Z drugiej strony,
    zarządca ma bardzo ograniczony dostęp do pól i metod pamiątek, co
    pozwala mu przechowywać je w formie stosu ale bez możliwości zmiany
    ich stanu.

#### Implementacja na podstawie interfejsu pośredniego

Istnieje alternatywna implementacja, odpowiednia w przypadku języków
programowania które nie wspierają zagnieżdżania klas (mówię o tobie,
PHP!).

:::: memento-empty-interface
::: {.struct-image2 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/memento/structure21ec9.png?id=fcff71cb648389be2e302fbe55e2f847"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/diagrams/memento/structure2.png?id=fcff71cb648389be2e302fbe55e2f847 560w,/images/patterns/diagrams/memento/structure2-1.5x.png?id=5c05495a7ca6545fcc57f70ea7ee904a 840w,/images/patterns/diagrams/memento/structure2-2x.png?id=aa7fb5d0f622d4344a2cb590f437f8c8 1120w"
sizes="(max-width: 720px) 100vw, 560px" loading="lazy" width="560"
alt="Pamiątka bez użycia klas zagnieżdżonych" /><img
src="../../images/patterns/diagrams/memento/structure2-indexede045.png?id=2c98b4f64b03f2a30e159de31ca9f718"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.6;"
srcset="/images/patterns/diagrams/memento/structure2-indexed.png?id=2c98b4f64b03f2a30e159de31ca9f718 560w,/images/patterns/diagrams/memento/structure2-indexed-1.5x.png?id=1ba6e0f22bb613f3f1dcf86850c3b604 840w,/images/patterns/diagrams/memento/structure2-indexed-2x.png?id=2fb637daef1110dfa89f15b2d4627894 1120w"
sizes="(max-width: 720px) 100vw, 560px" loading="lazy" width="560"
alt="Pamiątka bez użycia klas zagnieżdżonych" />
</figure>
:::
::::

1.  Wobec niemożności zagnieżdżania klas można ograniczyć dostęp do pól
    pamiątki stosując konwencję według której zarządcy współdziałają z
    pamiątkami wyłącznie za pośrednictwem jasno zadeklarowanego
    interfejsu pośredniego. Taki interfejs deklarowałby tylko metody
    związane z metadanymi pamiątki.

2.  Z drugiej strony, źródła mogą współpracować z pamiątkami
    bezpośrednio, poprzez dostęp do pól i metod w nich zadeklarowanych.
    Wadą tego podejścia jest konieczność deklaracji wszystkich
    składowych pamiątki jako publiczne.

#### Implementacja z jeszcze ściślejszą hermetyzacją

Kolejna implementacja jest użyteczna, gdy nie chcemy pozostawić nawet
najmniejszego ryzyka, że obce klasy uzyskają dostęp do stanu źródła za
pośrednictwem pamiątki.

:::: memento-safe
::: {.struct-image3 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/memento/structure3f138.png?id=6c3ef6a916be8c8ec6d6659d19a6a79f"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.74;"
srcset="/images/patterns/diagrams/memento/structure3.png?id=6c3ef6a916be8c8ec6d6659d19a6a79f 590w,/images/patterns/diagrams/memento/structure3-1.5x.png?id=9bb6d9dd5567bc71d9e93efc9183ef3a 885w,/images/patterns/diagrams/memento/structure3-2x.png?id=988c37f92059457153d26ba3458d371e 1180w"
sizes="(max-width: 720px) 100vw, 590px" loading="lazy" width="590"
alt="Pamiątka ze ścisłą hermetyzacją" /><img
src="../../images/patterns/diagrams/memento/structure3-indexedadf8.png?id=17e84b0ef89a41bb3fb844c8d7a445ad"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.74;"
srcset="/images/patterns/diagrams/memento/structure3-indexed.png?id=17e84b0ef89a41bb3fb844c8d7a445ad 590w,/images/patterns/diagrams/memento/structure3-indexed-1.5x.png?id=c121c75333433b70b9a67b75e536214c 885w,/images/patterns/diagrams/memento/structure3-indexed-2x.png?id=fef9ae2a0151c105976075aafb8939dd 1180w"
sizes="(max-width: 720px) 100vw, 590px" loading="lazy" width="590"
alt="Pamiątka ze ścisłą hermetyzacją" />
</figure>
:::
::::

1.  Ta implementacja pozwala mieć wiele typów źródeł i pamiątek. Każde
    źródło współdziała z odpowiednią dla niego klasą pamiątki. Ani
    źródła, ani pamiątki nie eksponują swojego stanu komukolwiek.

2.  Zarządcom uniemożliwiono teraz zmianę stanu przechowywanego w
    pamiątkach. Co więcej, klasa zarządcy staje się niezależna od
    źródła, ponieważ metoda przywracająca stan jest teraz zdefiniowana w
    klasie pamiątki.

3.  Każda pamiątka staje się powiązana ze źródłem które ją utworzyło.
    Źródło przekazuje samo siebie do konstruktora pamiątki wraz z
    wartościami opisującymi jego stan. Dzięki bliskiemu związkowi
    pomiędzy tymi klasami, pamiątka może przywrócić stan źródła, gdyż
    ten drugi ma zdefiniowane stosowne metody setter.
:::::::::
::::::::::

::: {.section .pseudocode}
##  Pseudokod {#pseudocode}

W poniższym przykładzie zastosowano wzorce Pamiątka oraz
[Polecenie](command.html) w celu przechowywania migawek stanu
rozbudowanego edytora tekstu i przywracania wcześniejszego stanu z owych
migawek gdy zajdzie potrzeba.

<figure class="image">
<img
src="../../images/patterns/diagrams/memento/example4526.png?id=fb2196b065f374a1c2a64a0943463760"
style="aspect-ratio: 4.29;"
srcset="/images/patterns/diagrams/memento/example.png?id=fb2196b065f374a1c2a64a0943463760 600w,/images/patterns/diagrams/memento/example-1.5x.png?id=174b686f918a2c49e2545d5573c52d42 900w,/images/patterns/diagrams/memento/example-2x.png?id=41a73f3cc22bc3dd180f53e6968974d4 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Struktura przykładu użycia wzorca Pamiątka" />
<figcaption><p>Zapisywanie migawek stanu
edytora tekstu.</p></figcaption>
</figure>

Obiekty typu polecenie pełnią rolę zarządców. Pobierają pamiątkę edytora
przed wykonaniem działań związanych z poleceniem. Gdy użytkownik
spróbuje cofnąć ostatnio wykonane polecenie, edytor może skorzystać z
pamiątki przechowanej w tym poleceniu aby przywrócić wcześniejszy stan
siebie.

Klasa pamiątka nie deklaruje żadnych pól publicznych, getterów, ani
setterów. Dzięki temu żaden obiekt nie może zmienić zawartości pamiątki.
Pamiątki są powiązane z tym obiektem edytora, który je utworzył.
Pozwala to pamiątce przywrócić stan związanego z nią edytora przekazując
przechowywane dane obiektowi edytora za pośrednictwem funkcji setter.
Ponieważ pamiątki są połączone z konkretnymi obiektami edytorów,
aplikacja mogłaby wspierać wiele niezależnych okien edycji ze
scentralizowanym stosem historii.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// Źródło posiada jakieś istotne dane które mogą się z czasem
// zmienić. Definiuje także metodę służącą zapisywaniu swojego
// stanu wewnątrz pamiątki i inną metodę do przywracania swojego
// stanu na podstawie pamiątki.
class Editor is
    private field text, curX, curY, selectionWidth

    method setText(text) is
        this.text = text

    method setCursor(x, y) is
        this.curX = x
        this.curY = y

    method setSelectionWidth(width) is
        this.selectionWidth = width

    // Zapisuje bieżący stan w pamiątce.
    method createSnapshot():Snapshot is
        // Pamiątka to niezmienialny obiekt i dlatego źródło
        // przekazuje swój stan pamiątce w formie parametru
        // konstruktora.
        return new Snapshot(this, text, curX, curY, selectionWidth)

// Pamiątka przechowuje przeszły stan edytora.
class Snapshot is
    private field editor: Editor
    private field text, curX, curY, selectionWidth

    constructor Snapshot(editor, text, curX, curY, selectionWidth) is
        this.editor = editor
        this.text = text
        this.curX = x
        this.curY = y
        this.selectionWidth = selectionWidth

    // W którymś momencie będzie można przywrócić poprzedni stan
    // edytora korzystając z obiektu pamiątki.
    method restore() is
        editor.setText(text)
        editor.setCursor(curX, curY)
        editor.setSelectionWidth(selectionWidth)

// Obiekt polecenie może pełnić rolę nadzorcy. W takim przypadku
// polecenie zapisuje w sobie pamiątkę zanim zmieni stan źródła.
// Gdy otrzyma rozkaz wycofania działania, przywraca stan źródła
// na podstawie zachowanej pamiątki.
class Command is
    private field backup: Snapshot

    method makeBackup() is
        backup = editor.createSnapshot()

    method undo() is
        if (backup != null)
            backup.restore()
    // ...</code></pre>
</figure>
:::

:::::::: {.section .applicability-container}
##  Zastosowanie {#applicability}

::::::: applicability
::: applicability-problem
Stosuj wzorzec Pamiątka gdy chcesz tworzyć migawki stanu obiektu i móc
przywracać poprzedni jego stan.
:::

::: applicability-solution
Wzorzec Pamiątka pozwala tworzyć pełne kopie stanu obiektu, wraz z jego
danymi prywatnymi i przechowywać je poza obiektem. Chociaż większość
kojarzy ten wzorzec z przypadkiem użycia "cofnij", to jest on również
nieodzowny w przypadku transakcji (np. gdy chcesz cofnąć wykonywane
działanie w razie pojawienia się błędu).
:::

::: applicability-problem
Stosuj ten wzorzec gdy bezpośredni dostęp do pól/getterów/setterów
obiektu psuje hermetyzację.
:::

::: applicability-solution
Pamiątka czyni obiekt odpowiedzialnym za tworzenie migawek swojego
stanu. Żaden inny obiekt nie ma prawa odczytać migawki, co zabezpiecza
stan pierwotnego obiektu.
:::
:::::::
::::::::

::: {.section .checklist}
##  Jak zaimplementować {#checklist}

1.  Określ która klasa będzie pełniła rolę źródła. Ważne jest ustalenie
    czy program będzie miał jeden centralny obiekt tego typu, czy parę
    mniejszych.

2.  Stwórz klasę pamiątki. Jeden po drugim zadeklaruj zestaw pól
    odzwierciedlających pola zadeklarowane w klasie źródła.

3.  Uczyń klasę pamiątki niezmienialną. Pamiątka powinna przyjmować dane
    tylko raz --- za pośrednictwem konstruktora. Klasa nie powinna mieć
    metod setter.

4.  Jeśli stosowany język programowania wspiera zagnieżdżane klasy,
    zagnieźdź pamiątkę w obrębie klasy źródła. Jeśli nie wspiera,
    wyekstrahuj pusty interfejs z klasy pamiątka i spraw, by inne
    obiekty korzystały z niego w kontakcie z pamiątką. Można dodać do
    takiego interfejsu funkcje związane z metadanymi, ale żadnych
    funkcji eksponujących stan źródła.

5.  Dodaj metodę tworzącą pamiątki do klasy źródła. Źródło powinno
    przekazywać swój stan do pamiątki za pośrednictwem jednego lub wielu
    argumentów konstruktora pamiątki.

    Typem zwracanym przez metodę powinien być interfejs wyekstrahowany w
    poprzednim etapie (zakładając, że w ogóle się go wyekstrahowało). Za
    kulisami, metoda tworząca pamiątkę powinna współdziałać
    bezpośrednio z klasą pamiątka.

6.  Dodaj do klasy źródła metodę służącą przywracaniu stanu. Powinna
    przyjmować w charakterze argumentu obiekt typu pamiątka. Jeśli
    wyekstrahowano interfejs w poprzednim etapie, uczyń go typem
    parametru. W tym przypadku, trzeba rzutować obiekt przyjmowany na
    klasę pamiątka, ponieważ źródło musi mieć pełen dostęp do tego
    obiektu.

7.  Zarządca, niezależnie od tego czy reprezentuje obiekt polecenie,
    historię lub coś jeszcze innego, powinien wiedzieć kiedy żądać
    nowych pamiątek od źródła, jak je przechowywać i kiedy przywracać
    stan źródła za pomocą jakiejś pamiątki.

8.  Połączenie między zarządcami a źródłami można przenieść do klasy
    pamiątka. W tym przypadku, każda pamiątka musi być połączona ze
    źródłem które je utworzyło. Metoda przywracająca również
    trafiłaby do klasy pamiątka. To wszystko jednak ma sens tylko jeśli
    klasa pamiątka jest zagnieżdżona wewnątrz klasy źródła lub jeśli
    klasa źródła udostępnia funkcje setter służące nadpisywaniu jej
    stanu.
:::

:::::: {.section .pros-cons}
##  Zalety i wady {#pros-cons}

::::: row
::: col-sm-6
-    Można tworzyć migawki stanu obiektów bez naruszania ich
    hermetyzacji.
-    Można uprościć kod źródła, pozwalając zarządcy śledzić historię
    stanu źródła.
:::

::: col-sm-6
-    Aplikacja może wymagać dużej ilości pamięci RAM jeśli klienci zbyt
    często będą tworzyć pamiątki.
-    Zarządcy powinni śledzić cykl życia źródła, aby być w stanie
    kasować zbędne pamiątki.
-    Większość dynamicznych języków programowania, jak PHP, Python i
    JavaScript nie daje gwarancji niezmienialności stanu pamiątki.
:::
:::::
::::::

::: {.section .relations}
##  Powiązania z innymi wzorcami {#relations}

-   Można stosować [Polecenie](command.html) i [Pamiątkę](memento.html)
    jednocześnie --- implementując funkcjonalność "cofnij". W takim
    przypadku, polecenia są odpowiedzialne za wykonywanie różnych
    działań na obiekcie docelowym, zaś pamiątki służą zapamiętaniu stanu
    obiektu tuż przed wykonaniem polecenia.

-   Można zastosować [Pamiątkę](memento.html) wraz z
    [Iteratorem](iterator.html) by zapisać bieżący stan iteracji, co
    pozwoli w razie potrzeby do niego powrócić.

-   Czasem [Prototyp](prototype.html) może być prostszą alternatywą dla
    [Pamiątki](memento.html). Jest to możliwe jeśli obiekt, którego stan
    chcesz przechować w historii, jest w miarę nieskomplikowany. Obiekt
    taki nie powinien też mieć powiązań z zewnętrznymi zasobami, albo
    muszą one być łatwe do ponownego nawiązania.
:::

::: {.section .implementations}
##  Przykłady kodu {#implementations}

[![Pamiątka w języku
C#](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](memento/csharp/example.html "Pamiątka w języku C#"){.prog-lang-link}
[![Pamiątka w języku
C++](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](memento/cpp/example.html "Pamiątka w języku C++"){.prog-lang-link}
[![Pamiątka w języku
Go](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](memento/go/example.html "Pamiątka w języku Go"){.prog-lang-link}
[![Pamiątka w języku
Java](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](memento/java/example.html "Pamiątka w języku Java"){.prog-lang-link}
[![Pamiątka w języku
PHP](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](memento/php/example.html "Pamiątka w języku PHP"){.prog-lang-link}
[![Pamiątka w języku
Python](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](memento/python/example.html "Pamiątka w języku Python"){.prog-lang-link}
[![Pamiątka w języku
Ruby](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](memento/ruby/example.html "Pamiątka w języku Ruby"){.prog-lang-link}
[![Pamiątka w języku
Rust](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](memento/rust/example.html "Pamiątka w języku Rust"){.prog-lang-link}
[![Pamiątka w języku
Swift](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](memento/swift/example.html "Pamiątka w języku Swift"){.prog-lang-link}
[![Pamiątka w języku
TypeScript](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](memento/typescript/example.html "Pamiątka w języku TypeScript"){.prog-lang-link}
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

[Obserwator []{.fa .fa-arrow-right}](observer.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Wróć

[[]{.fa .fa-arrow-left} Mediator](mediator.html){.btn .btn-default
rel="prev"}
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
