:::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [Wzorce
projektowe](../design-patterns.html){.type} / [Wzorce
kreacyjne](creational-patterns.html){.type}
:::

# Metoda wytwórcza {#metoda-wytwórcza .title}

::: aka
Znany też jako: [Konstruktor
wirtualny, ]{style="display:inline-block"}[Virtual
constructor, ]{style="display:inline-block"}[Factory
Method]{style="display:inline-block"}
:::

::: {.section .intent}
##  Cel {#intent}

**Metoda wytwórcza** jest kreacyjnym wzorcem projektowym, który
udostępnia interfejs do tworzenia obiektów w ramach klasy bazowej, ale
pozwala podklasom zmieniać typ tworzonych obiektów.

<figure class="image">
<img
src="../../images/patterns/content/factory-method/factory-method-pl423b.png?id=c20eaae65212ebe13228eb6cb1c5117d"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/factory-method/factory-method-pl.png?id=c20eaae65212ebe13228eb6cb1c5117d 640w,/images/patterns/content/factory-method/factory-method-pl-1.5x.png?id=ca65e889a1b6175a841e83bbca77a8ac 960w,/images/patterns/content/factory-method/factory-method-pl-2x.png?id=0bc30869f5bfb1fe1818fe90922d2bae 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Wzorzec Metody wytwórczej" />
</figure>
:::

::: {.section .problem}
##  Problem

Wyobraź sobie, że tworzysz aplikację do zarządzania logistyką. Pierwsza
wersja twojej aplikacji pozwala jedynie na obsługę transportu za
pośrednictwem ciężarówek, więc większość kodu znajduje się wewnątrz
klasy `Ciężarówka`.

Po jakimś czasie twoja aplikacja staje się całkiem popularna. Codziennie
otrzymujesz tuzin próśb od firm realizujących spedycję morską, abyś
dodał stosowną funkcjonalność do swej aplikacji.

<figure class="image">
<img
src="../../images/patterns/diagrams/factory-method/problem1-plf2a8.png?id=b90846eb202a1e9820c8f81b9b73ae4c"
style="aspect-ratio: 2.4;"
srcset="/images/patterns/diagrams/factory-method/problem1-pl.png?id=b90846eb202a1e9820c8f81b9b73ae4c 600w,/images/patterns/diagrams/factory-method/problem1-pl-1.5x.png?id=b7eb773db41c0de6f05b86979ebf0931 900w,/images/patterns/diagrams/factory-method/problem1-pl-2x.png?id=b86e8b4ee4b93489f7408a01b21dd895 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Dodanie nowej klasy transportu do programu powoduje problem" />
<figcaption><p>Dodanie nowej klasy do programu nie jest takie proste,
jeśli reszta kodu jest już związana z
istniejącymi klasami.</p></figcaption>
</figure>

Świetna wiadomość, prawda? Ale co z kodem? W tej chwili większość
twojego kodu jest powiązana z klasą `Ciężarówka`. Dodanie do aplikacji
klasy `Statki` wymagałoby dokonania zmian w całym kodzie. Co więcej,
jeśli później zdecydujesz się dodać kolejny rodzaj transportu, zapewne
będziesz musiał dokonać tych zmian jeszcze jeden raz.

Rezultatem powyższych działań będzie brzydki kod, pełen instrukcji
warunkowych, których zadaniem będzie dostosowanie zachowania aplikacji
zależnie od klasy transportu.
:::

::: {.section .solution}
##  Rozwiązanie {#solution}

Wzorzec projektowy Metody wytwórczej proponuje zamianę bezpośrednich
wywołań konstruktorów obiektów (wykorzystujących operator `new`) na
wywołania specjalnej metody *wytwórczej*. Jednak nie przejmuj się tym:
obiekty nadal powstają za pośrednictwem operatora `new`, ale teraz
dokonuje się to za kulisami --- z wnętrza metody wytwórczej. Obiekty
zwracane przez metodę wytwórczą często są nazywane *produktami*.

<figure class="image">
<img
src="../../images/patterns/diagrams/factory-method/solution16ec2.png?id=fc756d2af296b5b4d482e548214d08ef"
style="aspect-ratio: 2.3;"
srcset="/images/patterns/diagrams/factory-method/solution1.png?id=fc756d2af296b5b4d482e548214d08ef 620w,/images/patterns/diagrams/factory-method/solution1-1.5x.png?id=22d3b6bb74e966d1cb3a4d8f380cefa3 930w,/images/patterns/diagrams/factory-method/solution1-2x.png?id=c482b3cd7a4d8dd73b4c8c12dfcaa03c 1240w"
sizes="(max-width: 720px) 100vw, 620px" loading="lazy" width="620"
alt="Struktura klas kreacyjnych" />
<figcaption><p>Podklasy mogą zmieniać klasę obiektów zwracanych przez
metodę wytwórczą.</p></figcaption>
</figure>

Na pierwszy rzut oka zmiana ta może wydawać się bezcelowa. Przecież
przenieśliśmy jedynie wywołanie konstruktora z jednej części programu do
drugiej. Ale zwróć uwagę, że teraz możesz nadpisać metodę wytwórczą w
podklasie, a tym samym zmienić klasę produktów zwracanych przez metodę.

Istnieje jednak małe ograniczenie: podklasy mogą zwracać różne typy
produktów tylko wtedy, gdy produkty te mają wspólną klasę bazową lub
wspólny interfejs. Ponadto, zwracany typ metody wytwórczej w klasie
bazowej powinien być zgodny z tym interfejsem.

<figure class="image">
<img
src="../../images/patterns/diagrams/factory-method/solution2-pl50d3.png?id=2ea672b238353e97025c33b4beab6336"
style="aspect-ratio: 1.96;"
srcset="/images/patterns/diagrams/factory-method/solution2-pl.png?id=2ea672b238353e97025c33b4beab6336 490w,/images/patterns/diagrams/factory-method/solution2-pl-1.5x.png?id=ebd07c70ead363796d2524f3536009a3 735w,/images/patterns/diagrams/factory-method/solution2-pl-2x.png?id=eaea5f71b3e003306567be7831843c8f 980w"
sizes="(max-width: 720px) 100vw, 490px" loading="lazy" width="490"
alt="Struktura hierarchii produktów" />
<figcaption><p>Wszystkie produkty muszą być zgodne z tym
samym interfejsem.</p></figcaption>
</figure>

Na przykład zarówno klasy `Ciężarówka`, jak i `Statek` powinny
implementować interfejs `Transport`, który z kolei deklaruje metodę
`dostarczaj`. Każda klasa różnie implementuje tę metodę: ciężarówki
dostarczają towar drogą lądową, statki drogą morską. Metoda wytwórcza
znajdująca się w klasie `LogistykaDrogowa` zwraca obiekty `Ciężarówka`,
zaś metoda wytwórcza w klasie `LogistykaMorska` zwraca `Statki`.

<figure class="image">
<img
src="../../images/patterns/diagrams/factory-method/solution3-pl1c6d.png?id=d2585c4ef386e567a2d5f282423babfc"
style="aspect-ratio: 1.83;"
srcset="/images/patterns/diagrams/factory-method/solution3-pl.png?id=d2585c4ef386e567a2d5f282423babfc 640w,/images/patterns/diagrams/factory-method/solution3-pl-1.5x.png?id=932ce71bec7c26dcfc2dabd7f27581bd 960w,/images/patterns/diagrams/factory-method/solution3-pl-2x.png?id=1b6f658c62f440cc9a220ccc9286166f 1280w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="640"
alt="Struktura kodu po zastosowaniu wzorca projektowego Metody wytwórczej" />
<figcaption><p>O ile wszystkie klasy produktów implementują wspólny
interfejs, możesz przekazywać ich obiekty do kodu klienckiego bez
obawy o jego zepsucie.</p></figcaption>
</figure>

Kod, który wykorzystuje metodę wytwórczą (zwany często kodem
*klienckim*) nie widzi różnicy pomiędzy faktycznymi produktami
zwróconymi przez różne podklasy. Klient traktuje wszystkie produkty jako
abstrakcyjnie pojęty `Transport`. Klient wie także, że wszystkie obiekty
transportowe posiadają metodę `dostarczaj`, ale szczegóły jej działania
nie są dla niego istotne.
:::

::::: {.section .structure-container}
##  Struktura {#structure}

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/factory-method/structure78c3.png?id=4cba0803f42517cfe8548c9bc7dc4c9b"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.74;"
srcset="/images/patterns/diagrams/factory-method/structure.png?id=4cba0803f42517cfe8548c9bc7dc4c9b 660w,/images/patterns/diagrams/factory-method/structure-1.5x.png?id=ece8300890afc14494770a6b6d370428 990w,/images/patterns/diagrams/factory-method/structure-2x.png?id=9ea3aa8a47f8be22e12e523c15b448fd 1320w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="660"
alt="Struktura wzorca Metoda Wytwórcza" /><img
src="../../images/patterns/diagrams/factory-method/structure-indexed6136.png?id=4c603207859ca1f939b17b60a3a2e9e0"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.74;"
srcset="/images/patterns/diagrams/factory-method/structure-indexed.png?id=4c603207859ca1f939b17b60a3a2e9e0 660w,/images/patterns/diagrams/factory-method/structure-indexed-1.5x.png?id=626b6d7f577e1c265cce244678b2cf7f 990w,/images/patterns/diagrams/factory-method/structure-indexed-2x.png?id=c794e4f2d05013fb176464a1d1a5d7ab 1320w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="660"
alt="Struktura wzorca Metoda Wytwórcza" />
</figure>
:::

1.  **Produkt** deklaruje interfejs, który jest wspólny dla wszystkich
    obiektów zwracanych przez twórcę oraz jego podklasy.

2.  **Konkretne Produkty** są różnymi implementacjami interfejsu
    produktów.

3.  Klasa **Twórca** deklaruje metodę wytwórczą, która zwraca nowe
    obiekty-produkty. Istotne jest, że typ zwracany przez tę metodę jest
    zgodny z interfejsem produktu.

    Możesz zadeklarować metodę wytwórczą jako abstrakcyjną. Wówczas
    każda podklasa będzie musiała zaimplementować swoją jej wersję.
    Innym sposobem jest sprawienie, aby bazowa metoda wytwórcza zwracała
    jakiś domyślny typ produktu.

    Weź jednak pod uwagę, że wbrew swojej nazwie, tworzenie produktów
    **nie** jest główną odpowiedzialnością Klasy Twórcy. Zazwyczaj klasa
    kreacyjna zawiera już jakąś ważną logikę biznesową związaną z
    produktami. Metoda wytwórcza pomaga rozprzęgnąć tę logikę i
    konkretne klasy produktów. Oto analogia: duża firma tworząca
    oprogramowanie może posiadać swój dział szkoleniowy dla
    programistów. Ale głównym zadaniem firmy jako całości jest nadal
    tworzenie kodu, a nie tworzenie programistów.

4.  **Konkretni Twórcy** nadpisują bazową metodę wytwórczą, co
    sprawia, że zwraca ona inny typ obiektu.

    Tu jednak uwaga: Metoda wytwórcza nie musi wciąż **tworzyć** nowych
    instancji. Może też zwrócić istniejący już obiekt z pamięci
    podręcznej, puli obiektów lub z innego źródła.
::::
:::::

::: {.section .pseudocode}
##  Pseudokod {#pseudocode}

Przykład ten ilustruje, jak **Metoda Wytwórcza** może służyć tworzeniu
elementów interfejsu użytkownika (UI) które będą przenośne między
platformami. Nie występuje tu sprzęgnięcie kodu klienckiego z
konkretnymi klasami interfejsu użytkownika.

<figure class="image">
<img
src="../../images/patterns/diagrams/factory-method/examplec59f.png?id=67db9a5cb817913444efcb1c067c9835"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/diagrams/factory-method/example.png?id=67db9a5cb817913444efcb1c067c9835 640w,/images/patterns/diagrams/factory-method/example-1.5x.png?id=921d97276e5e2fd8e64609c9cfa021e7 960w,/images/patterns/diagrams/factory-method/example-2x.png?id=a2470830778e318263155000dbdc5870 1280w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="640"
alt="Struktura przykładu wzorca projektowego Metody Wytwórczej" />
<figcaption><p>Przykład międzyplatformowego
okna dialogowego.</p></figcaption>
</figure>

Bazowa klasa okna dialogowego wykorzystuje różne elementy interfejsu
użytkownika rysując na ekranie okno. W różnych systemach operacyjnych,
elementy te mogą różnić się nieco wyglądem, ale powinny zachowywać się w
sposób spójny. Przycisk w Windows powinien być wciąż przyciskiem
również w Linux.

Wraz z pojawieniem się metody wytwórczej, znika konieczność
przepisywania logiki okna dialogowego dla poszczególnych systemów
operacyjnych. Jeśli zadeklarujemy metodę wytwórczą, która tworzy
przyciski w ramach klasy bazowej okna dialogowego, możemy później
stworzyć dodatkową podklasę okna dialogowego, która będzie zwracała
przyciski w stylu Windows z poziomu metody wytwórczej. Podklasa
odziedziczy wówczas większość kodu okna z klasy bazowej, ale dzięki
metodzie wytwórczej, będzie renderowała przyciski w stylu Windows.

Aby ten wzorzec zadziałał, klasa bazowa okna dialogowego musi działać
korzystając z przycisków abstrakcyjnych klasy bazowej lub interfejsu,
zgodnie z którym powstaną wszystkie konkretne przyciski. Dzięki temu,
kod okna dialogowego pozostanie funkcjonalny niezależnie od tego, jaki
będzie rodzaj przycisku.

Oczywiście możesz zastosować to podejście również w stosunku do innych
elementów interfejsu użytkownika. Jednak wraz z dodaniem do okna
dialogowego każdej kolejnej metody wytwórczej, zbliżasz się do wzorca
projektowego zwanego [Fabryka abstrakcyjna](abstract-factory.html). Ale
nie obawiaj się, później zajmiemy się również tym problemem.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// Klasa kreacyjna deklaruje metodę wytwórczą która musi zwracać
// obiekt klasy produktu. Poszczególne podklasy kreatora na ogół
// implementują tę metodę.
class Dialog is
    // Kreator może również posiadać jakąś domyślną
    // implementację metody wytwórczej.
    abstract method createButton():Button

    // Zwróć uwagę, że pomimo swojej nazwy, głównym zadaniem
    // kreatora nie jest tworzenie produktów. Zamiast tego
    // zawiera jakąś kluczową logikę biznesową która jest
    // zależna od obiektów-produktów zwróconych przez metodę
    // wytwórczą. Podklasy mogą pośrednio zmieniać tę logikę
    // biznesową poprzez nadpisywanie metody wytwórczej i
    // zwracanie innych typów produktów.
    method render() is
        // Wywołanie metody wytwórczej w celu stworzenia
        // obiektu-produktu.
        Button okButton = createButton()
        // A następnie użycie produktu.
        okButton.onClick(closeDialog)
        okButton.render()

// Konkretni kreatorzy nadpisują metodę wytwórczą w celu zmiany
// zwracanego typu produktu.
class WindowsDialog extends Dialog is
    method createButton():Button is
        return new WindowsButton()

class WebDialog extends Dialog is
    method createButton():Button is
        return new HTMLButton()

// Interfejs produktu deklaruje wszystkie działania które
// konkretne produkty muszą zaimplementować.
interface Button is
    method render()
    method onClick(f)

// Konkretne produkty posiadają różne implementacje interfejsu
// produktu.
class WindowsButton implements Button is
    method render(a, b) is
        // Renderuj przycisk w stylu Windows.
    method onClick(f) is
        // Powiąż z wbudowanym w system operacyjny zdarzeniem
        // kliknięcia

class HTMLButton implements Button is
    method render(a, b) is
        // Zwróć wersję HTML przycisku.
    method onClick(f) is
        // Powiąż z wbudowanym w przeglądarkę zdarzeniem
        // kliknięcia


class Application is
    field dialog: Dialog

    // Aplikacja wybiera typ kreatora na podstawie bieżącej
    // konfiguracji lub zmiennych środowiskowych.
    method initialize() is
        config = readApplicationConfigFile()

        if (config.OS == &quot;Windows&quot;) then
            dialog = new WindowsDialog()
        else if (config.OS == &quot;Web&quot;) then
            dialog = new WebDialog()
        else
            throw new Exception(&quot;Error! Unknown operating system.&quot;)

    // Kod kliencki współpracuje z instancją konkretnego twórcy
    // za pośrednictwem interfejsu bazowego. Tak długo jak
    // klient będzie współpracował z kreatorem za pośrednictwem
    // interfejsu bazowego, można będzie mu przekazywać dowolną
    // podklasę twórcy.
    method main() is
        this.initialize()
        dialog.render()</code></pre>
</figure>
:::

:::::::::: {.section .applicability-container}
##  Zastosowanie {#applicability}

::::::::: applicability
::: applicability-problem
Stosuj Metodę Wytwórczą gdy nie wiesz z góry jakie typy obiektów pojawią
się w twoim programie i jakie będą między nimi zależności.
:::

::: applicability-solution
Metoda Wytwórcza oddziela kod konstruujący produkty od kodu który
faktycznie z tych produktów korzysta. Dlatego też łatwiej jest
rozszerzać kod konstruujący produkty bez konieczności ingerencji w
resztę kodu.

Przykładowo, aby dodać nowy typ produktu do aplikacji, będziesz musiał
utworzyć jedynie podklasę kreacyjną i nadpisać jej metodę wytwórczą.
:::

::: applicability-problem
Korzystaj z Metody Wytwórczej gdy zamierzasz pozwolić użytkującym twą
bibliotekę lub framework rozbudowywać jej wewnętrzne komponenty.
:::

::: applicability-solution
Dziedziczenie jest prawdopodobnie najłatwiejszym sposobem rozszerzania
domyślnego zachowania się biblioteki lub frameworku. Ale skąd framework
wiedziałby o konieczności zastosowania twojej podklasy, zamiast
standardowego komponentu?

Rozwiązaniem jest zredukowanie kodu konstruującego komponenty na
przestrzeni frameworku do pojedynczej metody wytwórczej. Trzeba też
umożliwić nadpisywanie tej metody, a nie tylko rozbudowywanie samego
komponentu.

Sprawdźmy, jak by to wyglądało. Wyobraź sobie, że piszesz aplikację
korzystając z open source'owego frameworku interfejsu użytkownika (UI).
Twoja aplikacja ma posiadać okrągłe przyciski, ale framework oferuje
jedynie prostokątne. Rozszerzasz więc standardową klasę `Przycisk` o
podklasę `OkrągłyPrzycisk`. Ale teraz trzeba również sprawić, by główna
klasa `UIFramework` korzystała z nowo utworzonej podklasy, zamiast z
domyślnej.

Aby to osiągnąć, tworzysz podklasę `UIZOkrągłymiPrzyciskami` z bazowej
klasy frameworku i nadpisujesz jej metodę `stwórzPrzycisk`. Metoda ta
będzie zwracać obiekty klasy `Przycisk` w klasie bazowej, zaś twoja jej
podklasa zwróci obiekty `OkrągłyPrzycisk`. Od teraz skorzystasz z klasy
`UIZOkrągłymiPrzyciskami` zamiast klasy `UIFramework`. I to tyle!
:::

::: applicability-problem
Korzystaj z Metody wytwórczej gdy chcesz oszczędniej wykorzystać zasoby
systemowe poprzez ponowne wykorzystanie już istniejących obiektów,
zamiast odbudowywać je raz za razem.
:::

::: applicability-solution
Powyższa sytuacja może wyłonić się na pierwszy plan, gdy mamy do
czynienia z dużymi obiektami, wymagającymi sporej ilości zasobów.
Mogą do nich należeć połączenia do bazy danych, systemy plików oraz
zasoby sieciowe.

Zastanówmy się, co musi się stać, aby istniejący obiekt mógł zostać
wykorzystany ponownie:

1.  Najpierw musisz stworzyć jakiś magazyn, który będzie pamiętał
    wszystkie utworzone obiekty.
2.  Gdy ktoś zgłosi zapotrzebowanie na obiekt, program powinien odszukać
    wolny obiekt spośród tych już istniejących w puli.
3.  \... i wreszcie zwrócić go kodowi klienckiemu.
4.  Jeśli nie ma żadnych wolnych obiektów, program powinien utworzyć
    nowy (i dodać go do puli magazynowej).

To strasznie dużo kodu! I na dodatek cały musi się znaleźć w jednym
miejscu, aby uniknąć rozrzucania jego kopii po całym programie.

Prawdopodobnie, najbardziej oczywistym i odpowiednim miejscem na
umieszczenie tego nowego kodu jest konstruktor tej klasy, której obiekty
chcemy ponownie wykorzystywać. Ale konstruktor musi z definicji zawsze
zwracać **nowe obiekty**. Nie może zwracać istniejących jego instancji.

Dlatego też będzie potrzebna zwykła metoda, która zdolna jest zarówno
tworzyć nowe obiekty, jak i pozwolić na wykorzystanie już
istniejących. A to już brzmi jak metoda wytwórcza.
:::
:::::::::
::::::::::

## Jak implementować

1.  Wszystkie produkty powinny być zgodne z tym samym interfejsem.
    Interfejs ten powinien deklarować metody, które są sensowne dla
    każdego produktu.

2.  Dodaj pustą metodę wytwórczą do klasy kreacyjnej. Zwracany typ tej
    metody powinien być zgodny z interfejsem wspólnym dla produktów.

3.  W kodzie kreacyjnym należy odnaleźć wszystkie odniesienia do
    konstruktorów produktów. Jeden po drugim, trzeba zamienić je na
    wywołania metody wytwórczej, ekstrahując przy tym kod kreacyjny
    produktów, aby umieścić go w metodzie wytwórczej.

    Możliwe, że konieczne okaże się ustanowienie tymczasowego
    parametru w metodzie wytwórczej, aby kontrolować jaki typ produktu
    będzie zwrócony.

    Na tym etapie, kod metody wytwórczej może brzydko wyglądać. Może
    się na przykład okazać, że wewnątrz znajduje się wielki operator
    `switch` wybierający stosowną klasę produktu, jaką należy powołać do
    istnienia. Ale nie martw się, niedługo się tym zajmiemy.

4.  Teraz stwórz zestaw podklas kreacyjnych dla każdego typu produktu
    wymienionego w metodzie wytwórczej. Nadpisz metodę wytwórczą w
    każdej z podklas i wyekstrahuj stosowne fragmenty kodu
    konstruującego z metody bazowej.

5.  Jeśli mamy zbyt wiele typów produktów i nie ma sensu tworzyć
    podklasy dla każdego z nich, możesz wykorzystać ponownie parametr
    kontrolny klasy bazowej w podklasach.

    Na przykład wyobraź sobie, że masz następującą hierarchię klas.
    Istnieje klasa bazowa `Poczta` z kilkoma podklasami:
    `PocztaLotnicza` oraz `PocztaLądowa`. Klasy `Transport` to:
    `Samolot`, `Ciężarówka` oraz `Pociąg`. Klasa `PocztaLotnicza` używa
    jedynie obiektów klasy `Samolot`, ale `PocztaLądowa` zarówno
    obiektów klasy `Ciężarówka`, jak i `Pociąg`. Można więc stworzyć
    kolejną podklasę (powiedzmy, że `PocztaKolejowa`) aby obsłużyć oba
    przypadki, ale istnieje lepszy sposób. Kod kliencki może bowiem
    przekazać parametr metodzie wytwórczej znajdującej się w klasie
    `PocztaLądowa` który zdecyduje o typie produktów, jakie są
    potrzebne.

6.  Jeśli po dokonaniu wszystkich ekstrakcji, bazowa metoda wytwórcza
    została pusta, możesz uczynić ją abstrakcyjną. Jeśli jednak coś w
    niej pozostało, możesz pozostawić to jako domyślne zachowanie się
    metody.

:::::: {.section .pros-cons}
##  Zalety i wady {#pros-cons}

::::: row
::: col-sm-6
-    Unikasz ścisłego sprzęgnięcia pomiędzy twórcą a konkretnymi
    produktami.
-    *Zasada pojedynczej odpowiedzialności*. Możesz przenieść kod
    kreacyjny produktów w jedno miejsce programu, ułatwiając tym samym
    utrzymanie kodu.
-    *Zasada otwarte/zamknięte*. Możesz wprowadzić do programu nowe typy
    produktów bez psucia istniejącego kodu klienckiego.
:::

::: col-sm-6
-    Kod może się skomplikować, ponieważ aby zaimplementować wzorzec,
    musisz utworzyć liczne podklasy. W najlepszej sytuacji
    wprowadzisz ów wzorzec projektowy do już istniejącej hierarchii klas
    kreacyjnych.
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

-   Możesz zastosować [Metodę wytwórczą](factory-method.html) wraz z
    [Iteratorem](iterator.html) aby pozwolić podklasom kolekcji zwracać
    różne typy iteratorów kompatybilnych z kolekcją.

-   [Prototyp](prototype.html) nie bazuje na dziedziczeniu, więc nie
    posiada właściwych temu podejściu wad. Z drugiej strony jednak
    *Prototyp* wymaga skomplikowanej inicjalizacji klonowanego obiektu.
    [Metoda wytwórcza](factory-method.html) bazuje na dziedziczeniu, ale
    nie wymaga etapu inicjalizacji.

-   [Metoda wytwórcza](factory-method.html) to wyspecjalizowana [Metoda
    szablonowa](template-method.html). Może stanowić także jeden z
    etapów większej *Metody szablonowej*.
:::

::: {.section .implementations}
##  Przykłady kodu {#implementations}

[![Metoda wytwórcza w języku
C#](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](factory-method/csharp/example.html "Metoda wytwórcza w języku C#"){.prog-lang-link}
[![Metoda wytwórcza w języku
C++](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](factory-method/cpp/example.html "Metoda wytwórcza w języku C++"){.prog-lang-link}
[![Metoda wytwórcza w języku
Go](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](factory-method/go/example.html "Metoda wytwórcza w języku Go"){.prog-lang-link}
[![Metoda wytwórcza w języku
Java](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](factory-method/java/example.html "Metoda wytwórcza w języku Java"){.prog-lang-link}
[![Metoda wytwórcza w języku
PHP](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](factory-method/php/example.html "Metoda wytwórcza w języku PHP"){.prog-lang-link}
[![Metoda wytwórcza w języku
Python](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](factory-method/python/example.html "Metoda wytwórcza w języku Python"){.prog-lang-link}
[![Metoda wytwórcza w języku
Ruby](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](factory-method/ruby/example.html "Metoda wytwórcza w języku Ruby"){.prog-lang-link}
[![Metoda wytwórcza w języku
Rust](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](factory-method/rust/example.html "Metoda wytwórcza w języku Rust"){.prog-lang-link}
[![Metoda wytwórcza w języku
Swift](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](factory-method/swift/example.html "Metoda wytwórcza w języku Swift"){.prog-lang-link}
[![Metoda wytwórcza w języku
TypeScript](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](factory-method/typescript/example.html "Metoda wytwórcza w języku TypeScript"){.prog-lang-link}
:::

::: {.section .extras}
##  Dodatek {#extras}

-   Przeczytaj nasze [Porównanie fabryk](factory-comparison.html), jeśli
    masz trudności z rozróżnieniem poszczególnych koncepcji i wzorców
    wytwórczych.
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

[Fabryka abstrakcyjna []{.fa
.fa-arrow-right}](abstract-factory.html){.btn .btn-primary rel="next"}
:::

::: prev
#### Wróć

[[]{.fa .fa-arrow-left} Wzorce kreacyjne](creational-patterns.html){.btn
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
