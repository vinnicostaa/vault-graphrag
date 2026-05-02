::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [Wzorce
projektowe](../design-patterns.html){.type} / [Wzorce
kreacyjne](creational-patterns.html){.type}
:::

# Fabryka abstrakcyjna {#fabryka-abstrakcyjna .title}

::: aka
Znany też jako: [Abstract Factory]{style="display:inline-block"}
:::

::: {.section .intent}
##  Cel {#intent}

**Fabryka abstrakcyjna** jest kreacyjnym wzorcem projektowym, który
pozwala tworzyć rodziny spokrewnionych ze sobą obiektów bez określania
ich konkretnych klas.

<figure class="image">
<img
src="../../images/patterns/content/abstract-factory/abstract-factory-pl7fbc.png?id=0c6af53b4192e819e2ff8cbe2d076b43"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/abstract-factory/abstract-factory-pl.png?id=0c6af53b4192e819e2ff8cbe2d076b43 640w,/images/patterns/content/abstract-factory/abstract-factory-pl-1.5x.png?id=6f1be881ec36ac72af4a55fbff4eee8e 960w,/images/patterns/content/abstract-factory/abstract-factory-pl-2x.png?id=654e32acd799564691e101eed7b57724 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Wzorzec fabryki abstrakcyjnej" />
</figure>
:::

::: {.section .problem}
##  Problem

Wyobraź sobie, że tworzysz symulator sklepu meblowego. Twój kod składa
się z klas, które reprezentują:

1.  Rodzinę spokrewnionych produktów, powiedzmy: `Fotel` + `Sofa` +
    `StolikKawowy`.

2.  Różne warianty w ramach powyższej rodziny. Na przykład, produkty
    `Fotel` + `Sofa` są dostępne w wariantach: `Nowoczesny`,
    `Wiktoriański`, `ArtDeco`.

<figure class="image">
<img
src="../../images/patterns/diagrams/abstract-factory/problem-pla7ee.png?id=35c33773c4ef135e42608e34996a3af3"
style="aspect-ratio: 1.4;"
srcset="/images/patterns/diagrams/abstract-factory/problem-pl.png?id=35c33773c4ef135e42608e34996a3af3 420w,/images/patterns/diagrams/abstract-factory/problem-pl-1.5x.png?id=741a378703b2ef29390ea020de0adeae 630w,/images/patterns/diagrams/abstract-factory/problem-pl-2x.png?id=cfedba595790964ed59d78c0f9e64f81 840w"
sizes="(max-width: 720px) 100vw, 420px" loading="lazy" width="420"
alt="Rodziny produktów i ich warianty." />
<figcaption><p>Rodziny produktów i ich warianty.</p></figcaption>
</figure>

Trzeba produkować poszczególne meble w taki sposób, aby do siebie
pasowały. Klienci nie cierpią bowiem otrzymywać mebli w zupełnie różnych
stylizacjach.

<figure class="image">
<img
src="../../images/patterns/content/abstract-factory/abstract-factory-comic-1-pl7feb.png?id=0d30c71afaaa82b3ff4325942085b949"
style="aspect-ratio: 2;"
srcset="/images/patterns/content/abstract-factory/abstract-factory-comic-1-pl.png?id=0d30c71afaaa82b3ff4325942085b949 600w,/images/patterns/content/abstract-factory/abstract-factory-comic-1-pl-1.5x.png?id=90977201c581835a3d1800cb0e54b88f 900w,/images/patterns/content/abstract-factory/abstract-factory-comic-1-pl-2x.png?id=b07dca3a7e9babb33d6bcf0317558496 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600" />
<figcaption><p>Sofa zaprojektowana w stylu nowoczesnym nie pasuje do
foteli wiktoriańskich.</p></figcaption>
</figure>

Ponadto, nie chciałbyś zmieniać istniejącego kodu tylko po to, aby dodać
nowy produkt lub rodzinę produktów do programu. Producenci mebli dość
często wypuszczają nowe katalogi i nie chciałbyś zmieniać głównej części
kodu za każdym razem gdy tak się stanie.
:::

::: {.section .solution}
##  Rozwiązanie {#solution}

Pierwszą rzeczą, jaką proponuje wzorzec projektowy Fabryka abstrakcyjna,
jest wyraźne określenie interfejsów dla każdego konkretnego produktu z
jakiejś rodziny (np. fotel, sofa, stolik kawowy). Następnie trzeba
sprawić, aby wszystkie warianty produktów były zgodne z tymi
interfejsami. Na przykład wszystkie fotele implementują interfejs
`Fotel`, wszystkie stoliki kawowe implementują interfejs
`StolikKawowy` i tak dalej.

<figure class="image">
<img
src="../../images/patterns/diagrams/abstract-factory/solution1a1ac.png?id=71f2018d8bb443b9cce90d57110675e3"
style="aspect-ratio: 1.5;"
srcset="/images/patterns/diagrams/abstract-factory/solution1.png?id=71f2018d8bb443b9cce90d57110675e3 420w,/images/patterns/diagrams/abstract-factory/solution1-1.5x.png?id=4366e971c850514cde5d33cb7956de8b 630w,/images/patterns/diagrams/abstract-factory/solution1-2x.png?id=eacec286153e058db9255d26541c0529 840w"
sizes="(max-width: 720px) 100vw, 420px" loading="lazy" width="420"
alt="Hierarchia klas Foteli" />
<figcaption><p>Wszystkie warianty tego samego obiektu muszą się
znaleźć w jednej hierarchii klasowej.</p></figcaption>
</figure>

Kolejnym krokiem jest deklaracja interfejsu *Fabryka abstrakcyjna*,
który zawrze listę metod kreacyjnych wszystkich produktów w ramach
jednej rodziny (na przykład, `stwórzFotel`, `stwórzSofę`,
`stwórzStolikKawowy`). Metody te muszą zwracać wyłącznie
**abstrakcyjne** typy produktów, reprezentowane uprzednio określonymi
interfejsami: `Fotel`, `Sofa`, `StolikKawowy` i tak dalej.

<figure class="image">
<img
src="../../images/patterns/diagrams/abstract-factory/solution24d8e.png?id=53975d6e4714c6f942633a879f7ac571"
style="aspect-ratio: 2;"
srcset="/images/patterns/diagrams/abstract-factory/solution2.png?id=53975d6e4714c6f942633a879f7ac571 640w,/images/patterns/diagrams/abstract-factory/solution2-1.5x.png?id=581a6cdc0dcd5511f1c30e509b1d4a7f 960w,/images/patterns/diagrams/abstract-factory/solution2-2x.png?id=b21557081f75ac7b4110427e89a10378 1280w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="640"
alt="Hierarchia klas _Fabrycznych_" />
<figcaption><p>Każda konkretna fabryka odpowiada konkretnemu
wariantowi produktu.</p></figcaption>
</figure>

A co z poszczególnymi wariantami produktów? Otóż, dla każdego wariantu
rodziny produktów tworzymy osobną klasę fabryczną na podstawie
interfejsu `FabrykaAbstrakcyjna`. Klasa fabryczna to taka klasa, która
zwraca produkty danego rodzaju. A więc, `FabrykaNowoczesnychMebli` może
zwracać wyłącznie obiekty: `NowoczesneFotele`, `NowoczesneSofy` oraz
`NowoczesneStolikiKawowe`.

Kod kliencki będzie korzystał z fabryk oraz produktów za pośrednictwem
ich interfejsów abstrakcyjnych. Dzięki temu będzie można zmienić typ
fabryki przekazywanej kodowi klienckiemu oraz zmienić wariant produktu
jaki otrzyma kod kliencki i to wszystko bez ryzyka popsucia samego kodu
klienckiego.

<figure class="image">
<img
src="../../images/patterns/content/abstract-factory/abstract-factory-comic-2-pl567c.png?id=4c724a9d020e05e739a495282d31cc01"
style="aspect-ratio: 2;"
srcset="/images/patterns/content/abstract-factory/abstract-factory-comic-2-pl.png?id=4c724a9d020e05e739a495282d31cc01 600w,/images/patterns/content/abstract-factory/abstract-factory-comic-2-pl-1.5x.png?id=c9565f7adff78fdf0e20e5cc175c1889 900w,/images/patterns/content/abstract-factory/abstract-factory-comic-2-pl-2x.png?id=814ae016ce77d105bc7604526669e180 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600" />
<figcaption><p>Klienta nie powinno obchodzić to, z jaką konkretnie klasą
fabryczną ma do czynienia.</p></figcaption>
</figure>

Załóżmy, że klient potrzebuje fabrykę do stworzenia fotela. Nie powinien
musieć być świadom klasy tej fabryki, ani martwić się o rodzaj fotelu z
jakim przyjdzie mu działać. Czy będzie to fotel nowoczesny, czy też
wiktoriański, klient powinien traktować wszystkie w taki sam sposób, za
pośrednictwem interfejsu abstrakcyjnego `Fotel`. Dzięki temu podejściu,
klient wie tylko tyle, że fotele implementują jakąś formę metody
`usiądźNa`. Ponadto, niezależnie od wariantu zwracanego fotelu, zawsze
będą one pasowały do sof lub stolików kawowych jakie produkuje dany
obiekt fabryczny.

Pozostaje do wyjaśnienia jeszcze jedna sprawa: jeśli klient ma do
czynienia wyłącznie z interfejsami abstrakcyjnymi, to co właściwie
tworzy rzeczywiste obiekty fabryczne? Na ogół aplikacja tworzy konkretny
obiekt fabryczny na etapie inicjalizacji. Tuż przed tym wybiera stosowny
typ fabryki zależnie od konfiguracji lub środowiska.
:::

::::: {.section .structure-container}
##  Struktura {#structure}

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/abstract-factory/structureb809.png?id=a3112cdd98765406af94595a3c5e7762"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/diagrams/abstract-factory/structure.png?id=a3112cdd98765406af94595a3c5e7762 720w,/images/patterns/diagrams/abstract-factory/structure-1.5x.png?id=d2964e541620d9c087e693bd0e0a0836 1080w,/images/patterns/diagrams/abstract-factory/structure-2x.png?id=c4d3634ec2e74e02a0fe1a83ce9b50f6 1440w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="720"
alt="Wzorzec projektowy Fabryki abstrakcyjnej" /><img
src="../../images/patterns/diagrams/abstract-factory/structure-indexed028d.png?id=6ae1c99cbd90cf58753c633624fb1a04"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.56;"
srcset="/images/patterns/diagrams/abstract-factory/structure-indexed.png?id=6ae1c99cbd90cf58753c633624fb1a04 700w,/images/patterns/diagrams/abstract-factory/structure-indexed-1.5x.png?id=473be2975c5716162e6969e6532210ac 1050w,/images/patterns/diagrams/abstract-factory/structure-indexed-2x.png?id=cb6d4e1e89826c42966dc7097374f889 1400w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="700"
alt="Wzorzec projektowy Fabryki abstrakcyjnej" />
</figure>
:::

1.  **Produkty Abstrakcyjne** deklarują interfejsy odmiennych produktów,
    które składają się na wspólną rodzinę.

2.  **Konkretne Produkty** to różnorakie implementacje abstrakcyjnych
    produktów, pogrupowane według wariantów. Każdy abstrakcyjny produkt
    (fotel/sofa) musi być zaimplementowany we wszystkich zadanych
    wariantach (Wiktoriański/Nowoczesny).

3.  Interfejs **Fabryki Abstrakcyjnej** deklaruje zestaw metod służących
    tworzeniu każdego z abstrakcyjnych produktów.

4.  **Konkretne Fabryki** implementują metody kreacyjne fabryki
    abstrakcyjnej. Każda konkretna fabryka jest związana z jakimś
    określonym wariantem produktu i produkuje wyłącznie meble w tym
    stylu.

5.  Mimo, że konkretne fabryki tworzą konkretne egzemplarze produktu,
    sygnatury ich metod kreacyjnych muszą zwracać stosowne
    *abstrakcyjne* produkty. Dzięki temu kod kliencki, który korzysta z
    fabryki, nie zostanie sprzęgnięty z jakimś konkretnym wariantem
    produktu, jaki otrzymuje z fabryki. **Klient** może działać na
    dowolnym konkretnym wariancie fabryki/produktu, o ile będzie
    korzystał z interfejsów abstrakcyjnych ich obiektów.
::::
:::::

::: {.section .pseudocode}
##  Pseudokod {#pseudocode}

Poniższy przykład ilustruje zastosowanie **Fabryki abstrakcyjnej** do
tworzenia międzyplatformowych elementów interfejsu użytkownika (UI),
unikając tym samym sprzężenia kodu klienckiego z konkretnymi klasami
interfejsu użytkownika oraz zachowując zgodność tworzonych elementów UI
z danym systemem operacyjnym.

<figure class="image">
<img
src="../../images/patterns/diagrams/abstract-factory/exampled62f.png?id=5928a61d18bf00b047463471c599100a"
style="aspect-ratio: 1.42;"
srcset="/images/patterns/diagrams/abstract-factory/example.png?id=5928a61d18bf00b047463471c599100a 640w,/images/patterns/diagrams/abstract-factory/example-1.5x.png?id=baee3bac793ec97e0ec91c49d9382e7d 960w,/images/patterns/diagrams/abstract-factory/example-2x.png?id=eb5127b1d6486f6fad73be2d5e444449 1280w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="640"
alt="Diagram klas przykładu użycia wzorca Fabryki abstrakcyjnej" />
<figcaption><p>Przykład międzyplatformowych klas UI.</p></figcaption>
</figure>

Elementy interfejsu użytkownika tego samego typu powinny zachowywać się
podobnie niezależnie od platformy, ale mogą wyglądać nieco inaczej na
różnych systemach operacyjnych. Co więcej, to twoim zadaniem jest
zapewnić zgodność wizualnego stylu elementów UI ze stylem konkretnego
systemu operacyjnego. Nie chcemy przecież wyświetlać kontrolek w stylu
macOS w programie uruchamianym pod Windows.

Interfejs fabryki abstrakcyjnej deklaruje pewien zestaw metod
kreacyjnych, dzięki którym kod kliencki może tworzyć różne typy
elementów interfejsu użytkownika. Konkretne fabryki odpowiadają
poszczególnym systemom operacyjnym i tworzą zgodne z nimi wizualnie
elementy UI.

Działa to tak: kiedy aplikacja jest uruchamiana, sprawdza pod jakim
pracuje systemem operacyjnym. Mając tę wiedzę, aplikacja tworzy obiekt
fabryczny z klasy odpowiadającej danemu systemowi. Reszta kodu z kolei
korzysta z tej fabryki przy tworzeniu elementów interfejsu użytkownika.
Dzięki temu zapobiega się tworzeniu niewłaściwych kontrolek.

Dzięki takiemu podejściu, kod kliencki nie jest zależny od konkretnych
klas fabryk oraz elementów UI, pod warunkiem, że będzie korzystał z ich
interfejsów abstrakcyjnych. Pozwoli to także kodowi klienckiemu zachować
zgodność z fabrykami lub elementami UI mogącymi pojawić się w
przyszłości.

Wynikiem takiego projektowania, uniknąć można zmian w kodzie klienckim w
razie dodania obsługi nowych stylów wizualnych kontrolek. Wystarczy
stworzyć nową klasę fabryczną, która wytwarzać będzie te elementy oraz
nieco zmodyfikować kod inicjalizujący aplikacji, aby mógł obrać tę
klasę.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// Interfejs fabryki abstrakcyjnej deklaruje pewien zestaw metod
// które zwracają różne produkty abstrakcyjne. Produkty te
// łącznie nazywa się rodziną i łączy je jakiś wysokopoziomowy
// wspólny motyw lub koncepcja. Produkty z jednej rodziny są
// zwykle zdolne do współpracy z sobą nawzajem. Rodzina
// produktów może mieć wiele wariantów, ale produkty jednego
// wariantu są niekompatybilne z produktami z rodziny innego
// wariantu.
interface GUIFactory is
    method createButton():Button
    method createCheckbox():Checkbox

// Konkretne fabryki tworzą produkty należące do jednego
// wariantu jednej rodziny. Fabryka gwarantuje że powstałe
// produkty będą kompatybilne. Zwracane typy w sygnaturach metod
// wytwórczych konkretnej fabryki określone są jako produkty
// abstrakcyjne, zaś konkretna instancja produktu powstaje
// wewnątrz metody.
class WinFactory implements GUIFactory is
    method createButton():Button is
        return new WinButton()
    method createCheckbox():Checkbox is
        return new WinCheckbox()

// Każda konkretna fabryka ma swój wariant produktu.
class MacFactory implements GUIFactory is
    method createButton():Button is
        return new MacButton()
    method createCheckbox():Checkbox is
        return new MacCheckbox()

// Każdy odrębny produkt z rodziny powinien mieć interfejs
// bazowy. Wszystkie warianty produktu muszą zaimplementować ten
// interfejs.
interface Button is
    method paint()

// Konkretne fabryki mają swoje konkretne produkty.
class WinButton implements Button is
    method paint() is
        // Renderuj przycisk w stylu Windows.

class MacButton implements Button is
    method paint() is
        // Renderuj przycisk w stylu macOS.

// Oto interfejs bazowy innego produktu. Wszystkie produkty mogą
// ze sobą współdziałać, ale właściwa interakcja możliwa jest
// wyłącznie pomiędzy produktami jednego konkretnego wariantu.
interface Checkbox is
    method paint()

class WinCheckbox implements Checkbox is
    method paint() is
        // Renderuj pole wyboru w stylu Windows.

class MacCheckbox implements Checkbox is
    method paint() is
        // Renderuj pole wyboru w stylu macOS.

// Kod kliencki współpracuje z fabrykami i produktami wyłącznie
// stosując typy abstrakcyjne: GUIFactory, Button i Checkbox.
// Pozwala to przekazywać klientowi dowolną podklasę produktu
// lub fabryki.
class Application is
    private field factory: GUIFactory
    private field button: Button
    constructor Application(factory: GUIFactory) is
        this.factory = factory
    method createUI() is
        this.button = factory.createButton()
    method paint() is
        button.paint()

// Aplikacja wybiera typ fabryki na podstawie bieżącej
// konfiguracji lub zmiennych środowiskowych i tworzy jej
// instancję w trakcie działania programu (zazwyczaj na etapie
// inicjalizacji).
class ApplicationConfigurator is
    method main() is
        config = readApplicationConfigFile()

        if (config.OS == &quot;Windows&quot;) then
            factory = new WinFactory()
        else if (config.OS == &quot;Mac&quot;) then
            factory = new MacFactory()
        else
            throw new Exception(&quot;Error! Unknown operating system.&quot;)

        Application app = new Application(factory)</code></pre>
</figure>
:::

:::::::: {.section .applicability-container}
##  Zastosowanie {#applicability}

::::::: applicability
::: applicability-problem
Stosuj Fabrykę abstrakcyjną, gdy twój kod ma działać na produktach z
różnych rodzin, ale jednocześnie nie chcesz, aby ściśle zależał od
konkretnych klas produktów. Mogą one bowiem być nieznane na
wcześniejszym etapie tworzenia programu, albo chcesz umożliwić przyszłą
rozszerzalność aplikacji.
:::

::: applicability-solution
Fabryka abstrakcyjna dostarcza ci interfejs służący tworzeniu obiektów z
różnych klas danej rodziny produktów. O ile twój kod będzie kreował
obiekty za pośrednictwem tego interfejsu --- nie musisz się martwić
stworzeniem produktu w niezgodnym z innymi wariancie.
:::

::: applicability-problem
Przemyśl ewentualną implementację wzorca Fabryki abstrakcyjnej, gdy
masz do czynienia z klasą posiadającą zestaw [Metod
wytwórczych](factory-method.html) które zbytnio przyćmiewają główną
odpowiedzialność tej klasy.
:::

::: applicability-solution
 W prawidłowo zaprojektowanym programie *każda klasa jest
odpowiedzialna za jedną rzecz*. Gdy zaś klasa ma do czynienia z wieloma
typami produktów, warto być może zebrać jej metody wytwórcze i
umieścić je w osobnej klasie fabrycznej, albo nawet w pełni
zaimplementować Fabrykę abstrakcyjną z ich pomocą.
:::
:::::::
::::::::

::: {.section .checklist}
##  Jak zaimplementować {#checklist}

1.  Stwórz mapę poszczególnych typów produktów z uwzględnieniem
    wariantów w jakich mogą one być dostępne.

2.  Dla każdego typu produktu zaimplementuj abstrakcyjny interfejs.
    Niech wszystkie konkretne klasy produktów implementują powyższe
    interfejsy.

3.  Zadeklaruj interfejs fabryki abstrakcyjnej zawierający zestaw metod
    kreacyjnych wszystkich produktów abstrakcyjnych.

4.  Zaimplementuj zestaw konkretnych klas fabrycznych --- po jednym dla
    każdego wariantu produktu.

5.  Gdzieś w programie umieść kod inicjalizujący fabrykę. Kod ten
    powinien powołać do życia obiekt jednej z konkretnych klas
    fabrycznych --- zależnie od konfiguracji programu, czy też
    środowiska, w jakim został uruchomiony. Przekaż następnie ten obiekt
    fabryczny każdej klasie, której zadaniem jest konstrukcja produktów.

6.  Przejrzyj kod aplikacji, wyszukując wszelkie bezpośrednie wywołania
    konstruktorów produktów. Zamień te wywołania na takie, które odnoszą
    się do stosownych metod kreacyjnych obiektu fabrycznego.
:::

:::::: {.section .pros-cons}
##  Zalety i wady {#pros-cons}

::::: row
::: col-sm-6
-    Zyskujesz pewność, że produkty, jakie otrzymujesz stosując
    fabrykę, są ze sobą kompatybilne.
-    Zapobiegasz ścisłemu sprzęgnięciu konkretnych produktów z kodem
    klienckim.
-    *Zasada pojedynczej odpowiedzialności*. Możesz zebrać kod kreacyjny
    produktów w jednym miejscu w programie, ułatwiając tym samym
    późniejsze utrzymanie kodu.
-    *Zasada otwarte/zamknięte*. Możesz wprowadzać wsparcie dla nowych
    wariantów produktów bez psucia istniejącego kodu klienckiego.
:::

::: col-sm-6
-    Kod może stać się bardziej skomplikowany, niż powinien. Wynika to z
    konieczności wprowadzenia wielu nowych interfejsów i klas w toku
    wdrażania tego wzorca projektowego.
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

-   Klasy [Fabryka abstrakcyjna](abstract-factory.html) często wywodzą
    się z zestawu [Metod wytwórczych](factory-method.html), ale można
    także użyć [Prototypu](prototype.html) do skomponowania metod w tych
    klasach.

-   [Fabryka abstrakcyjna](abstract-factory.html) może służyć jako
    alternatywa do [Fasady](facade.html) gdy jedyne co chcesz zrobić, to
    ukrycie przed kodem klienckim procesu tworzenia obiektów podsystemu.

-   [Fabryka abstrakcyjna](abstract-factory.html) może być stosowana
    wraz z [Mostem](bridge.html). Takie sparowanie jest użyteczne gdy
    niektóre abstrakcje zdefiniowane przez *Most* mogą współdziałać
    wyłącznie z określonymi implementacjami. W tym przypadku, *Fabryka
    abstrakcyjna* może hermetyzować te relacje i ukryć zawiłości przed
    kodem klienckim.

-   [Fabryki abstrakcyjne](abstract-factory.html),
    [Budowniczych](builder.html) oraz [Prototypy](prototype.html) można
    zaimplementować jako [Singletony](singleton.html).
:::

::: {.section .implementations}
##  Przykłady kodu {#implementations}

[![Fabryka abstrakcyjna w języku
C#](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](abstract-factory/csharp/example.html "Fabryka abstrakcyjna w języku C#"){.prog-lang-link}
[![Fabryka abstrakcyjna w języku
C++](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](abstract-factory/cpp/example.html "Fabryka abstrakcyjna w języku C++"){.prog-lang-link}
[![Fabryka abstrakcyjna w języku
Go](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](abstract-factory/go/example.html "Fabryka abstrakcyjna w języku Go"){.prog-lang-link}
[![Fabryka abstrakcyjna w języku
Java](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](abstract-factory/java/example.html "Fabryka abstrakcyjna w języku Java"){.prog-lang-link}
[![Fabryka abstrakcyjna w języku
PHP](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](abstract-factory/php/example.html "Fabryka abstrakcyjna w języku PHP"){.prog-lang-link}
[![Fabryka abstrakcyjna w języku
Python](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](abstract-factory/python/example.html "Fabryka abstrakcyjna w języku Python"){.prog-lang-link}
[![Fabryka abstrakcyjna w języku
Ruby](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](abstract-factory/ruby/example.html "Fabryka abstrakcyjna w języku Ruby"){.prog-lang-link}
[![Fabryka abstrakcyjna w języku
Rust](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](abstract-factory/rust/example.html "Fabryka abstrakcyjna w języku Rust"){.prog-lang-link}
[![Fabryka abstrakcyjna w języku
Swift](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](abstract-factory/swift/example.html "Fabryka abstrakcyjna w języku Swift"){.prog-lang-link}
[![Fabryka abstrakcyjna w języku
TypeScript](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](abstract-factory/typescript/example.html "Fabryka abstrakcyjna w języku TypeScript"){.prog-lang-link}
:::

::: {.section .extras}
##  Dodatek {#extras}

-   Przeczytaj nasze [Porównanie fabryk](factory-comparison.html), jeśli
    chcesz dowiedzieć się więcej o różnicach pomiędzy poszczególnymi
    wzorcami fabrycznymi i koncepcjami.
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

[Porównanie Fabryk []{.fa
.fa-arrow-right}](factory-comparison.html){.btn .btn-primary rel="next"}
:::

::: prev
#### Wróć

[[]{.fa .fa-arrow-left} Metoda wytwórcza](factory-method.html){.btn
.btn-default rel="prev"}
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
