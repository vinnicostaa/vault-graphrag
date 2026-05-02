:::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [Wzorce
projektowe](../design-patterns.html){.type} / [Wzorce
strukturalne](structural-patterns.html){.type}
:::

# Most {#most .title}

::: aka
Znany też jako: [Bridge]{style="display:inline-block"}
:::

::: {.section .intent}
##  Cel {#intent}

**Most** jest strukturalnym wzorcem projektowym pozwalającym na
rozdzielenie dużej klasy lub zestawu spokrewnionych klas na dwie
hierarchie --- abstrakcję oraz implementację. Nad obiema można wówczas
pracować niezależnie.

<figure class="image">
<img
src="../../images/patterns/content/bridge/bridge3e59.png?id=bd543d4fb32e11647767301581a5ad54"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/bridge/bridge.png?id=bd543d4fb32e11647767301581a5ad54 640w,/images/patterns/content/bridge/bridge-1.5x.png?id=8ef07b78d61cc3830b7e2b783c76c775 960w,/images/patterns/content/bridge/bridge-2x.png?id=1e905ae5742e5cd10a7eb0e3175ef00d 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Wzorzec projektowy Most" />
</figure>
:::

::: {.section .problem}
##  Problem

*Abstrakcja?* *Implementacja?* --- brzmi strasznie? Spokojnie,
zacznijmy od prostego przykładu.

Załóżmy, że mamy klasę `Figura` wraz z dwiema podklasami: `Okrąg` i
`Kwadrat`. Chcesz rozszerzyć tę hierarchię klasową, aby zawierała
kolory, więc zamierzasz stworzyć podklasy dla figur `Czerwonych` i
`Niebieskich`. Jednak ponieważ już są dwie podklasy, konieczne byłoby
stworzenie czterech kombinacji, między innymi `NiebieskiOkrąg` i
`CzerwonyKwadrat`.

<figure class="image">
<img
src="../../images/patterns/diagrams/bridge/problem-pleb4a.png?id=63409753b3faa7391ae658394e243a68"
style="aspect-ratio: 1.41;"
srcset="/images/patterns/diagrams/bridge/problem-pl.png?id=63409753b3faa7391ae658394e243a68 480w,/images/patterns/diagrams/bridge/problem-pl-1.5x.png?id=09b0d2a3870aab783ba6e0a474780e43 720w,/images/patterns/diagrams/bridge/problem-pl-2x.png?id=df998c61f257e4cd469849eb07bd6da3 960w"
sizes="(max-width: 720px) 100vw, 480px" loading="lazy" width="480"
alt="Problem dla wzorca Most" />
<figcaption><p>Ilość kombinacji klas wzrasta w
postępie geometrycznym.</p></figcaption>
</figure>

Dodawanie do hierarchii nowych typów figur oraz kolorów spowoduje jej
wykładniczy wzrost. Przykładowo, aby dodać figurę trójkąt, musisz dodać
dwie podklasy --- po jednej na każdy kolor. Dodanie nowego koloru
wymagałoby stworzenia trzech podklas --- po jednej na każdą figurę. Im
dalej, tym gorzej.
:::

::: {.section .solution}
##  Rozwiązanie {#solution}

Problem ten powstaje, ponieważ próbujemy poszerzyć klasy figur w dwóch
niezależnych wymiarach: kształt oraz kolor. Jest to bardzo częsty
problem przy dziedziczeniu klas.

Wzorzec Most próbuje rozwiązać ten problem poprzez przestawienie się z
dziedziczenia na kompozycję obiektów. Oznacza to, że ekstrahujemy
jeden z tych wymiarów i tworzymy osobną hierarchię klas, przez co
pierwotne klasy będą posiadały odniesienie do obiektów z nowej
hierarchii, zamiast przechowywać wszystkie swoje stany i zachowanie
wewnątrz klasy.

<figure class="image">
<img
src="../../images/patterns/diagrams/bridge/solution-plbd21.png?id=356ecf5339d7bff96b3ce5472b4e4283"
style="aspect-ratio: 2.3;"
srcset="/images/patterns/diagrams/bridge/solution-pl.png?id=356ecf5339d7bff96b3ce5472b4e4283 460w,/images/patterns/diagrams/bridge/solution-pl-1.5x.png?id=dc363d503a5148cff48a7451023d811f 690w,/images/patterns/diagrams/bridge/solution-pl-2x.png?id=c2508ee9fcc2b4a93db0cb6a55de5eab 920w"
sizes="(max-width: 720px) 100vw, 460px" loading="lazy" width="460"
alt="Rozwiązanie proponowane przez wzorzec Most" />
<figcaption><p>Możesz zapobiec eksplozji hierarchii klas do wielkich
rozmiarów poprzez przekształcenie jej na kilka
powiązanych hierarchii.</p></figcaption>
</figure>

Stosując to podejście, możemy zebrać kod dotyczący kolorów i umieścić go
w swojej własnej klasie z dwiema podklasami: `Czerwony` i `Niebieski`.
Klasa `Figura` następnie zyskuje pole przechowujące odniesienie do
jednego z obiektów-kolorów. W efekcie figura geometryczna może
oddelegować wszelkie działania związane z kolorami do odpowiedniego
obiektu-koloru. Odniesienie to pełni rolę mostu pomiędzy klasami
`Figura` a `Kolor`. Od teraz, dodawanie nowych kolorów nie będzie
wymagało zmiany hierarchii figur --- i odwrotnie.

#### Abstrakcja i implementacja

Książka GoF []{.pop-i}["Gang of Four --- Banda Czterech" to przezwisko
nadane czterem autorom znanej książki przedstawiającej wzorce
projektowe: *Wzorce projektowe. Elementy oprogramowania obiektowego
wielokrotnego użytku*
[https://refactoring.guru/gof-book](https://helion.pl/ksiazki/wzorce-projektowe-elementy-oprogramowania-obiektowego-wielokrotnego-uzytku-erich-gamma-richard-helm-ralph-johnson-john-vli,wzoelv.htm).]{.pop-content}
wprowadza terminy *Abstrakcji* i *Implementacji* jako elementy definicji
mostu. Moim zdaniem, pojęcia te brzmią zbyt fachowo i tworzą
wrażenie, że wzorzec ten jest przesadnie skomplikowany. Przeczytawszy
nasz prosty przykład o figurach i kolorach, spróbujmy rozszyfrować
znaczenie niepokojącej terminologii.

*Abstrakcja* (zwana też *interfejsem*) stanowi wysokopoziomową warstwę
umożliwiającą kontrolę nad czymś. Nie ma ona wykonywać konkretnych prac,
ale delegować zadania do warstwy *implementacyjnej* (zwanej też
*platformą*).

Zwróć uwagę, że nie mówimy tu o znanych ci z języków programowania
*interfejsach* i *klasach abstrakcyjnych* --- chodzi o coś innego.

Mówiąc o prawdziwych aplikacjach, za abstrakcję można uznać graficzny
interfejs użytkownika (GUI), zaś implementację stanowi znajdujący się
poniżej interfejs programowania aplikacji (API). Użytkownik za pomocą
interfejsu użytkownika wydaje polecenia niższej warstwie.

Ogólnie mówiąc, taką aplikację można rozwijać w dwóch niezależnych
kierunkach:

-   Posiadać wiele różnych interfejsów użytkownika (wersje dla zwykłych
    użytkowników oraz dla administratora)
-   Współpracować z wieloma różnymi API (możliwość uruchomienia na
    systemie Windows, Linux oraz macOS).

W najgorszym przypadku aplikacja będzie przypominała spaghetti, gdzie
setki instrukcji warunkowych łączą różne interfejsy użytkownika z
różnymi interfejsami programowania aplikacji.

<figure class="image">
<img
src="../../images/patterns/content/bridge/bridge-3-ple0df.png?id=9046cfcde03f907263061faf1638f2f0"
style="aspect-ratio: 2;"
srcset="/images/patterns/content/bridge/bridge-3-pl.png?id=9046cfcde03f907263061faf1638f2f0 600w,/images/patterns/content/bridge/bridge-3-pl-1.5x.png?id=1ff95cc6793383e9583c0e591ab9f8c5 900w,/images/patterns/content/bridge/bridge-3-pl-2x.png?id=a0cf42d4c3580bf12f56dd1969ca6319 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Zarządzanie zmianami jest znacznie prostsze w kodzie modularnym" />
<figcaption><p>Dokonywanie nawet najmniejszych zmian w kodzie
monolitycznym jest trudne, ponieważ konieczne jest dobre rozumienie
<em>całości</em>. Wprowadzanie zmian w małych, dobrze określonych
modułach jest znacznie prostsze.</p></figcaption>
</figure>

Można zaprowadzić trochę ładu wśród tego chaosu ekstrahując kod
związany z kombinacjami interfejs-platforma i umieszczając go w osobnych
klasach. Wkrótce jednak odkryjesz, że takich klas powstanie *mnóstwo*.
Hierarchia klasowa rozrośnie się wykładniczo, ponieważ dodanie obsługi
nowego GUI lub wsparcia dla nowego API będzie wymagać tworzenia wciąż to
nowych klas.

Spróbujmy rozwiązać problem stosując wzorzec Most. Według jego założeń,
dzielimy klasy na dwie hierarchie:

-   Abstrakcja: warstwa graficznego interfejsu użytkownika aplikacji.
-   Implementacja: interfejs programowania aplikacji systemu
    operacyjnego.

<figure class="image">
<img
src="../../images/patterns/content/bridge/bridge-2-pl5652.png?id=21553b4d323ea5f6f30d7264a6574ea6"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/bridge/bridge-2-pl.png?id=21553b4d323ea5f6f30d7264a6574ea6 640w,/images/patterns/content/bridge/bridge-2-pl-1.5x.png?id=b59d9b13850e050fd528b440fff7186b 960w,/images/patterns/content/bridge/bridge-2-pl-2x.png?id=88ffc6976a04fad4d2da9995caecad7d 1280w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="640"
alt="Architektura wieloplatformowa" />
<figcaption><p>Jeden ze sposobów ustrukturyzowania
aplikacji wieloplatformowej.</p></figcaption>
</figure>

Obiekt abstrakcyjny steruje wyglądem aplikacji, delegując faktyczne
zadania do powiązanego z nim obiektu implementacyjnego. Różne
implementacje są wymienne, o ile zachowują zgodność ze wspólnym
interfejsem, umożliwiając w ten sposób stworzenie jednolitego
graficznego interfejsu użytkownika i pod Windows i pod Linux.

W rezultacie, możemy zmieniać klasy GUI bez konieczności modyfikacji
klas odnoszących się do API. Co więcej, dodanie obsługi kolejnego
systemu operacyjnego wymaga jedynie utworzenia podklasy w hierarchii
implementacyjnej.
:::

::::: {.section .structure-container}
##  Struktura {#structure}

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/bridge/structure-pl495d.png?id=23ac8be13c2729ad7289e83112a2a2e7"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.47;"
srcset="/images/patterns/diagrams/bridge/structure-pl.png?id=23ac8be13c2729ad7289e83112a2a2e7 560w,/images/patterns/diagrams/bridge/structure-pl-1.5x.png?id=331e229bc545b08d2e681b620352399d 840w,/images/patterns/diagrams/bridge/structure-pl-2x.png?id=9898d8199fdd565e5412d6af9006c37b 1120w"
sizes="(max-width: 720px) 100vw, 560px" loading="lazy" width="560"
alt="Wzorzec projektowy Most" /><img
src="../../images/patterns/diagrams/bridge/structure-pl-indexedf0bf.png?id=e4edc863b98e0c38f4cd8efb15ad32c7"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.44;"
srcset="/images/patterns/diagrams/bridge/structure-pl-indexed.png?id=e4edc863b98e0c38f4cd8efb15ad32c7 560w,/images/patterns/diagrams/bridge/structure-pl-indexed-1.5x.png?id=eecefde5874b8b4dd0b508a3e30e2af0 840w,/images/patterns/diagrams/bridge/structure-pl-indexed-2x.png?id=8fa369a7f92f1fb2b5cf82bace9fff9b 1120w"
sizes="(max-width: 720px) 100vw, 560px" loading="lazy" width="560"
alt="Wzorzec projektowy Most" />
</figure>
:::

1.  **Abstrakcja** obejmuje logikę kontrolną wysokiego poziomu.
    Potrzebuje obiektu implementacyjnego by wykonywać faktyczne
    działania.

2.  **Implementacja** deklaruje interfejs, który jest wspólny dla
    wszystkich konkretnych implementacji. Abstrakcja może komunikować
    się z obiektem implementacyjnym wyłącznie za pomocą
    zadeklarowanych tu metod.

    Abstrakcja może zawierać listę tych samych metod, co implementacja,
    ale zazwyczaj deklaruje bardziej złożone działania, na które składa
    się wiele prostszych działań deklarowanych przez implementację.

3.  **Konkretne implementacje** zawierają kod specyficzny dla danej
    platformy.

4.  **Wzbogacona Abstrakcja** udostępnia warianty logiki kontrolnej. Tak
    jak ich klasa-rodzic, współdziałają z różnymi implementacjami
    poprzez ogólny interfejs implementacyjny.

5.  Zazwyczaj, **Klienta** interesuje tylko współpraca z abstrakcją.
    Ale to zadaniem klienta jest połączenie obiektu abstrakcji z
    jednym z obiektów implementacji.
::::
:::::

::: {.section .pseudocode}
##  Pseudokod {#pseudocode}

Poniższy przykład ilustruje jak wzorzec **Most** może pomóc podzielić
monolityczny kod aplikacji zarządzającej urządzeniami i pilotami do
nich. Klasy `Urządzenie` stanowią implementację, natomiast `Piloty` ---
abstrakcję.

<figure class="image">
<img
src="../../images/patterns/diagrams/bridge/example-plda1c.png?id=89fa14f5bd0d560180c4f85fafbd7a12"
style="aspect-ratio: 1.33;"
srcset="/images/patterns/diagrams/bridge/example-pl.png?id=89fa14f5bd0d560180c4f85fafbd7a12 640w,/images/patterns/diagrams/bridge/example-pl-1.5x.png?id=d32089df7a25b89684b73da704b774a9 960w,/images/patterns/diagrams/bridge/example-pl-2x.png?id=382f59a1d9f2f4a39803cb6ece52aab0 1280w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="640"
alt="Struktura przykładu wzorca Most" />
<figcaption><p>Pierwotna hierarchia klas podzielona na dwie części:
urządzenia i piloty zdalnego sterowania.</p></figcaption>
</figure>

Bazowa klasa pilota deklaruje pole z odniesieniem do obiektu urządzenia.
Wszystkie piloty sterują urządzeniami za pośrednictwem uogólnionego
interfejsu, który pozwala jednemu pilotowi obsługiwać wiele typów
urządzeń.

Można pracować nad klasą pilota zdalnego sterowania niezależnie od klas
urządzeń. Potrzeba jedynie nowej podklasy pilota. Na przykład,
podstawowy pilot miałby tylko dwa przyciski, ale można by go było
wzbogacić o dodatkowe funkcje, jak dodatkowa bateria, lub ekran
dotykowy.

Klient dobiera pożądany typ pilota z konkretnym obiektem urządzenia za
pośrednictwem konstruktora pilota.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// &quot;Abstrakcja&quot; definiuje interfejs dla części &quot;kontrolującej&quot;
// obu hierarchii klas. Posiada odniesienie do obiektu z
// hierarchii &quot;implementacyjnej&quot; i deleguje temu obiektowi
// faktyczną pracę.
class RemoteControl is
    protected field device: Device
    constructor RemoteControl(device: Device) is
        this.device = device
    method togglePower() is
        if (device.isEnabled()) then
            device.disable()
        else
            device.enable()
    method volumeDown() is
        device.setVolume(device.getVolume() - 10)
    method volumeUp() is
        device.setVolume(device.getVolume() + 10)
    method channelDown() is
        device.setChannel(device.getChannel() - 1)
    method channelUp() is
        device.setChannel(device.getChannel() + 1)


// Można rozszerzać klasy należące do hierarchii abstrakcyjnej
// niezależnie od klas urządzeń.
class AdvancedRemoteControl extends RemoteControl is
    method mute() is
        device.setVolume(0)


// Interfejs &quot;implementacji&quot; deklaruje metody wspólne dla
// wszystkich konkretnych klas implementacji. Nie musi zgadzać
// się z interfejsem abstrakcji. Co więcej, oba interfejsy mogą
// być zupełnie różne. Zazwyczaj interfejs implementacji posiada
// tylko proste działania, podczas gdy abstrakcja definiuje
// działania wysokopoziomowe oparte na tych podstawowych.
interface Device is
    method isEnabled()
    method enable()
    method disable()
    method getVolume()
    method setVolume(percent)
    method getChannel()
    method setChannel(channel)


// Wszystkie urządzenia są zgodne pod względem interfejsu.
class Tv implements Device is
    // ...

class Radio implements Device is
    // ...


// Gdzieś w kodzie klienckim.
tv = new Tv()
remote = new RemoteControl(tv)
remote.togglePower()

radio = new Radio()
remote = new AdvancedRemoteControl(radio)</code></pre>
</figure>
:::

:::::::::: {.section .applicability-container}
##  Zastosowanie {#applicability}

::::::::: applicability
::: applicability-problem
Stosuj wzorzec Most gdy chcesz rozdzielić i przeorganizować monolityczną
klasę posiadającą wiele wariantów takiej samej funkcjonalności (na
przykład, jeśli klasa ma współpracować z wieloma serwerami
bazodanowymi).
:::

::: applicability-solution
 Im większa staje się klasa, tym ciężej zrozumieć jej działanie, a
dodawanie kolejnych zmian staje się coraz bardziej czasochłonne. Zmiana
jednego wariantu funkcjonalności może wymagać dokonania zmian na
przestrzeni całej klasy, a to z kolei wiąże się z wprowadzaniem błędów
lub przegapieniem jakichś krytycznych efektów ubocznych.

Wzorzec Most pozwala rozdzielić monolityczną klasę w wiele hierarchii
klas. Możemy wówczas zmieniać klasy w jednej z hierarchii niezależnie od
klas w drugiej. Podejście to upraszcza utrzymanie kodu i pozwala
zminimalizować ryzyko zepsucia istniejącego kodu.
:::

::: applicability-problem
Użyj tego wzorca gdy chcesz rozszerzyć klasę na kilku niezależnych
płaszczyznach.
:::

::: applicability-solution
Most proponuje ekstrakcję osobnej hierarchii klas dla każdej z takich
płaszczyzn rozbudowy. Pierwotna klasa deleguje pracę obiektom
należącym do tych hierarchii zamiast wykonywać ją sama.
:::

::: applicability-problem
Most pozwala spełnić wymóg możliwości wyboru implementacji w trakcie
działania programu.
:::

::: applicability-solution
Chociaż jest to opcjonalne, Most umożliwia zamianę obiektu implementacji
znajdującego się w abstrakcji. Jest to tak proste, jak przypisanie polu
klasy nowej wartości.

*Tak przy okazji, ostatnia cecha Mostu jest głównym powodem, dla którego
wiele ludzi myli Most ze wzorcem [Strategia](strategy.html).
Pamiętaj, że wzorzec to więcej niż pewien sposób strukturyzowania klas.
Może bowiem też sugerować pewien zamiar i wskazać rozwiązanie jakiegoś
problemu.*
:::
:::::::::
::::::::::

::: {.section .checklist}
##  Jak zaimplementować {#checklist}

1.  Określ płaszczyzny swoich klas. Takie niezależne koncepcje to na
    przykład: abstrakcja/platforma, domena/infrastruktura,
    front-end/back-end, interfejs/implementacja.

2.  Określ jakie operacje są klientowi potrzebne i zdefiniuj je w
    bazowej klasie abstrakcji.

3.  Określ zakres operacji dostępnych na wszystkich platformach.
    Zadeklaruj te, których abstrakcja potrzebuje w ogólnym interfejsie
    implementacji.

4.  Stwórz konkretne klasy implementacji dla wszystkich obsługiwanych
    platform. Upewnij się jednak, aby wszystkie były zgodne z
    interfejsem implementacji.

5.  Wewnątrz klasy abstrakcji, dodaj pole odnoszące się do typu
    implementacji. Abstrakcja będzie delegować większość zadań temu
    obiektowi implementacji.

6.  Jeśli masz wiele wariantów wysokopoziomowej logiki, stwórz
    wzbogacone abstrakcje dla każdego z wariantów poprzez rozszerzenie
    bazowej klasy abstrakcji.

7.  Kod kliencki powinien przekazywać obiekt implementacji
    konstruktorowi abstrakcji, aby skojarzyć obie hierarchie ze sobą. Po
    tym, klient może zapomnieć o implementacji i korzystać tylko z
    obiektu abstrakcji.
:::

:::::: {.section .pros-cons}
##  Zalety i wady {#pros-cons}

::::: row
::: col-sm-6
-    Możesz tworzyć niezależne od platformy klasy i aplikacje.
-    Kod klienta działa na wyższym poziomie abstrakcji. Nie musi mieć do
    czynienia ze szczegółami platformy.
-    *Zasada otwarte/zamknięte*. Możesz wprowadzać nowe abstrakcje i
    implementacje niezależnie od siebie.
-    *Zasada pojedynczej odpowiedzialności*. W abstrakcji możesz skupić
    się na wysokopoziomowej logice, zaś w implementacji na szczegółach
    platformy.
:::

::: col-sm-6
-    Kod może stać się bardziej skomplikowany gdy zastosuje się ten
    wzorzec w przypadku wysoce zwartej klasy.
:::
:::::
::::::

::: {.section .relations}
##  Powiązania z innymi wzorcami {#relations}

-   [Most](bridge.html) zazwyczaj wykorzystuje się od początku
    projektu, by pozwolić na niezależną pracę nad poszczególnymi
    częściami aplikacji. Z drugiej strony, [Adapter](adapter.html) jest
    rozwiązaniem stosowanym w istniejącej aplikacji w celu umożliwienia
    współpracy pomiędzy niekompatybilnymi klasami.

-   [Most](bridge.html), [Stan](state.html), [Strategia](strategy.html)
    (i w pewnym stopniu [Adapter](adapter.html)) mają podobną strukturę.
    Wszystkie oparte są na kompozycji, co oznacza delegowanie zadań
    innym obiektom. Jednak każdy z tych wzorców rozwiązuje inne
    problemy. Wzorzec nie jest bowiem tylko receptą na ustrukturyzowanie
    kodu w pewien sposób, lecz także informacją dla innych deweloperów o
    charakterze rozwiązywanego problemu.

-   [Fabryka abstrakcyjna](abstract-factory.html) może być stosowana
    wraz z [Mostem](bridge.html). Takie sparowanie jest użyteczne gdy
    niektóre abstrakcje zdefiniowane przez *Most* mogą współdziałać
    wyłącznie z określonymi implementacjami. W tym przypadku, *Fabryka
    abstrakcyjna* może hermetyzować te relacje i ukryć zawiłości przed
    kodem klienckim.

-   Możliwe jest połączenie wzorców [Budowniczy](builder.html) i
    [Most](bridge.html): klasa *kierownik* pełni rolę abstrakcji, zaś
    poszczególni *budowniczy* stanowią *implementacje*.
:::

::: {.section .implementations}
##  Przykłady kodu {#implementations}

[![Most w języku
C#](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](bridge/csharp/example.html "Most w języku C#"){.prog-lang-link}
[![Most w języku
C++](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](bridge/cpp/example.html "Most w języku C++"){.prog-lang-link}
[![Most w języku
Go](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](bridge/go/example.html "Most w języku Go"){.prog-lang-link}
[![Most w języku
Java](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](bridge/java/example.html "Most w języku Java"){.prog-lang-link}
[![Most w języku
PHP](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](bridge/php/example.html "Most w języku PHP"){.prog-lang-link}
[![Most w języku
Python](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](bridge/python/example.html "Most w języku Python"){.prog-lang-link}
[![Most w języku
Ruby](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](bridge/ruby/example.html "Most w języku Ruby"){.prog-lang-link}
[![Most w języku
Rust](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](bridge/rust/example.html "Most w języku Rust"){.prog-lang-link}
[![Most w języku
Swift](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](bridge/swift/example.html "Most w języku Swift"){.prog-lang-link}
[![Most w języku
TypeScript](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](bridge/typescript/example.html "Most w języku TypeScript"){.prog-lang-link}
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

[Kompozyt []{.fa .fa-arrow-right}](composite.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Wróć

[[]{.fa .fa-arrow-left} Adapter](adapter.html){.btn .btn-default
rel="prev"}
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
