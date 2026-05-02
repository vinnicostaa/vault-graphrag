::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [Wzorce
projektowe](../design-patterns.html){.type} / [Wzorce
behawioralne](behavioral-patterns.html){.type}
:::

# Obserwator {#obserwator .title}

::: aka
Znany też jako:
[Event-Subscriber, ]{style="display:inline-block"}[Listener, ]{style="display:inline-block"}[Observer]{style="display:inline-block"}
:::

::: {.section .intent}
##  Cel {#intent}

**Obserwator** to czynnościowy (behawioralny) wzorzec projektowy
pozwalający zdefiniować mechanizm subskrypcji w celu powiadamiania wielu
obiektów o zdarzeniach dziejących się w obserwowanym obiekcie.

<figure class="image">
<img
src="../../images/patterns/content/observer/observerd508.png?id=6088e31e1b0d4a417506a66614dcf065"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/observer/observer.png?id=6088e31e1b0d4a417506a66614dcf065 640w,/images/patterns/content/observer/observer-1.5x.png?id=aa64f9f336354462b57bbff5c09d8269 960w,/images/patterns/content/observer/observer-2x.png?id=d5a83e115528e9fd633f04ad2650f1db 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Wzorzec projektowy Obserwator" />
</figure>
:::

::: {.section .problem}
##  Problem

Wyobraź sobie, że masz dwa typy obiektów: `Klient` oraz `Sklep`. Klient
jest zainteresowany jakąś szczególną marką produktu, na przykład nowym
modelem iPhone, który ma się niedługo pojawić w sklepie.

Klient mógłby odwiedzać sklep codziennie i sprawdzać dostępność
produktu, ale dopóki produkt jest w drodze, większość tych spacerów
będzie bezcelowa.

<figure class="image">
<img
src="../../images/patterns/content/observer/observer-comic-1-plb902.png?id=6f9d34b6ac841038e7f4472ae59d9f95"
style="aspect-ratio: 2;"
srcset="/images/patterns/content/observer/observer-comic-1-pl.png?id=6f9d34b6ac841038e7f4472ae59d9f95 600w,/images/patterns/content/observer/observer-comic-1-pl-1.5x.png?id=fe26c72a95aeef28051caf7a1dc7cb05 900w,/images/patterns/content/observer/observer-comic-1-pl-2x.png?id=9c1f471a753db6e2109c9130246ef665 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Odwiedzanie sklepu a rozsyłanie spamu" />
<figcaption><p>Odwiedzanie sklepu a rozsyłanie spamu.</p></figcaption>
</figure>

Z drugiej strony, sklep mógłby rozesłać mnóstwo emaili do wszystkich
klientów jak tylko pojawi się nowy produkt, ale to może zostać odebrane
jako spamowanie. Zaoszczędziłoby wprawdzie niektórym klientom zbędnych
podróży do sklepu, ale jednocześnie niektórzy by się zdenerwowali
otrzymując nieistotną dla nich wiadomość.

Wygląda na to, że mamy tu konflikt. Albo klient traci czas sprawdzając
dostępność produktu, albo sklep ponosi koszty powiadamiając
niewłaściwych klientów.
:::

::: {.section .solution}
##  Rozwiązanie {#solution}

Obiekt który posiada jakiś interesujący stan nazywa się zwykle
*podmiotem*, ale skoro będzie powiadamiał inne obiekty o zmianach
swojego stanu, można nazwać go *publikującym*. Wszystkie pozostałe
obiekty, które chcą śledzić zmiany stanu nadawcy nazywa się
*subskrybentami*.

Wzorzec Obserwator proponuje dodanie mechanizmu subskrypcji do klasy
publikującego, aby pojedyncze obiekty mogły subskrybować lub przerwać
subskrypcję strumienia zdarzeń publikującego. Na szczęście nie jest to
tak skomplikowane jak brzmi. Tak naprawdę mechanizm ten składa się z 1)
pola tablicowego służącego przechowywaniu listy odniesień do
subskrybentów oraz 2) wielu metod publicznych pozwalających dodawać i
usuwać wpisy tej listy.

<figure class="image">
<img
src="../../images/patterns/diagrams/observer/solution1-pl91c2.png?id=2f0ca7142a07889c5b8fc42f6996e1e0"
style="aspect-ratio: 2.61;"
srcset="/images/patterns/diagrams/observer/solution1-pl.png?id=2f0ca7142a07889c5b8fc42f6996e1e0 470w,/images/patterns/diagrams/observer/solution1-pl-1.5x.png?id=ab9b113cb4a281cf6a0dc007940e33d9 705w,/images/patterns/diagrams/observer/solution1-pl-2x.png?id=1e69090141e14a2533550679c06a638c 940w"
sizes="(max-width: 720px) 100vw, 470px" loading="lazy" width="470"
alt="Mechanizm subskrypcji" />
<figcaption><p>Mechanizm subskrypcji pozwala pojedynczym obiektom
subskrybować powiadomienia o zdarzeniach.</p></figcaption>
</figure>

Za każdym razem, gdy wydarzy się coś ważnego publikującemu, może on
przejrzeć swoją listę subskrybentów i wywołać odpowiednią metodę
powiadamiania ich obiektów.

Prawdziwe aplikacje mogą mieć tuziny różnych klas subskrybentów które są
zainteresowane śledzeniem zdarzeń jednej klasy publikującego. Nie
chcielibyśmy sprzęgać nadawcy z tymi wszystkimi klasami. Poza tym możesz
nawet z góry nic o nich nie wiedzieć, jeśli klasę publikującą zamierzasz
udostępnić innym ludziom.

Dlatego właśnie ważnym jest, aby wszyscy subskrybenci implementowali ten
sam interfejs i żeby publikujący komunikował się z nimi wyłącznie
poprzez ten interfejs. Powinien on deklarować metodę powiadamiania
wraz z zestawem parametrów za pomocą których publikujący może przekazać
dodatkowe dane kontekstowe wraz z powiadomieniem.

<figure class="image">
<img
src="../../images/patterns/diagrams/observer/solution2-pl2408.png?id=8c5ef3452bb2fc4317e1b190e89490b3"
style="aspect-ratio: 1.24;"
srcset="/images/patterns/diagrams/observer/solution2-pl.png?id=8c5ef3452bb2fc4317e1b190e89490b3 460w,/images/patterns/diagrams/observer/solution2-pl-1.5x.png?id=8fda56ef7442d3ade0d3a5f64216d36f 690w,/images/patterns/diagrams/observer/solution2-pl-2x.png?id=d1d26a69e648d49daa58a4df11fefd5a 920w"
sizes="(max-width: 720px) 100vw, 460px" loading="lazy" width="460"
alt="Metody powiadamiania" />
<figcaption><p>Publikujący powiadamia subskrybentów wywołując ich
odpowiednie metody powiadamiania.</p></figcaption>
</figure>

Jeśli twoja aplikacja ma wiele różnych typów nadawców i chcesz uczynić
swoich subskrybentów kompatybilnymi z każdym z typów, możesz pójść o
krok dalej i zmusić publikujących do korzystania z tego samego
interfejsu. Taki interfejs musiałby opisywać tylko kilka metod
subskrybowania. Interfejs umożliwiłby subskrybentom obserwację stanów
obiektu publikującego bez konieczności sprzęgania z ich konkretnymi
klasami.
:::

::: {.section .analogy}
##  Analogia do prawdziwego życia {#analogy}

<figure class="image">
<img
src="../../images/patterns/content/observer/observer-comic-2-plc018.png?id=af348aa51efef0c739c8316bfb69ef0a"
style="aspect-ratio: 2;"
srcset="/images/patterns/content/observer/observer-comic-2-pl.png?id=af348aa51efef0c739c8316bfb69ef0a 600w,/images/patterns/content/observer/observer-comic-2-pl-1.5x.png?id=4bf03540e8d5a55074a140fb5404c34f 900w,/images/patterns/content/observer/observer-comic-2-pl-2x.png?id=164cf91934a502d1a9f3d36c6cd1fa57 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Subskrypcje czasopism i gazet" />
<figcaption><p>Subskrypcje czasopism i gazet.</p></figcaption>
</figure>

Jeśli subskrybujesz czasopismo lub gazetę, nie musisz więcej chodzić do
sklepu by sprawdzić czy jest już nowy numer. Zamiast tego, wydawca
przysyła ci nowe egzemplarze pocztą od razu po opublikowaniu, a czasem
nawet trochę wcześniej.

Wydawca zarządza listą subskrybentów i wie które czasopisma kogo
interesują. Subskrybenci mogą wypisać się z listy kiedy nie chcą już
otrzymywać kolejnych edycji.
:::

::::: {.section .structure-container}
##  Struktura {#structure}

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/observer/structurea121.png?id=365b7e2b8fbecc8948f34b9f8f16f33c"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.97;"
srcset="/images/patterns/diagrams/observer/structure.png?id=365b7e2b8fbecc8948f34b9f8f16f33c 610w,/images/patterns/diagrams/observer/structure-1.5x.png?id=7f23a48c9a2d789844f912d3a0f289e6 915w,/images/patterns/diagrams/observer/structure-2x.png?id=228af9bded4d6ee6daf43a0e23cca9ff 1220w"
sizes="(max-width: 720px) 100vw, 610px" loading="lazy" width="610"
alt="Struktura wzorca projektowego Obserwator" /><img
src="../../images/patterns/diagrams/observer/structure-indexed8afd.png?id=2ca2c123503ede860740af2a22bc4b4d"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.88;"
srcset="/images/patterns/diagrams/observer/structure-indexed.png?id=2ca2c123503ede860740af2a22bc4b4d 620w,/images/patterns/diagrams/observer/structure-indexed-1.5x.png?id=20b9e7765e83ca75514c202968d9e9fa 930w,/images/patterns/diagrams/observer/structure-indexed-2x.png?id=910eec855bc41f05199e510494078926 1240w"
sizes="(max-width: 720px) 100vw, 620px" loading="lazy" width="620"
alt="Struktura wzorca projektowego Obserwator" />
</figure>
:::

1.  **Publikujący** rozsyła zdarzenia interesujące inne obiekty.
    Zdarzenia te zachodzą gdy publikujący zmienia swój stan lub wykona
    jakieś obowiązki. Publikujący posiadają infrastrukturę dającą
    możliwość subskrybowania ich zdarzeń lub przerwania subskrypcji.

2.  Gdy nastąpi nowe zdarzenie, nadawca przegląda listę subskrybentów i
    wywołuje metody powiadamiania zadeklarowane w ich interfejsach.

3.  Interfejs **Subskrybenta** deklaruje interfejs powiadamiania. W
    większości przypadków składa się on z jednej metody `aktualizuj`.
    Metoda ta może przyjmować wiele parametrów pozwalających
    publikującemu przekazać za ich pomocą szczegóły dotyczące
    aktualizacji.

4.  **Konkretni Subskrybenci** wykonują jakieś czynności w odpowiedzi na
    powiadomienia wysłane przez publikującego. Wszystkie te klasy muszą
    implementować ten sam interfejs, aby nadawca nie musiał być
    sprzęgnięty z ich konkretnymi klasami.

5.  Zazwyczaj, subskrybenci potrzebują jakichś kontekstowych informacji
    aby poprawnie obsłużyć aktualizacje. Dlatego publikujący na ogół
    przekazują dane kontekstowe jako argumenty metody powiadamiania.
    Publikujący może przekazać samego siebie jako argument, umożliwiając
    subskrybentom pobranie sobie potrzebnych danych bezpośrednio.

6.  **Klient** tworzy obiekty publikujące i subskrybujące osobno, po
    czym rejestruje subskrybentów by mogli otrzymywać aktualizacje od
    publikującego.
::::
:::::

::: {.section .pseudocode}
##  Pseudokod {#pseudocode}

W poniższym przykładzie, wzorzec **Obserwator** pozwala obiektowi
będącemu edytorem tekstu powiadamiać inne obiekty usługowe o zmianach
swojego stanu.

<figure class="image">
<img
src="../../images/patterns/diagrams/observer/example04a2.png?id=6d0603ab5a00e4463b81d9639cd746a2"
style="aspect-ratio: 1.19;"
srcset="/images/patterns/diagrams/observer/example.png?id=6d0603ab5a00e4463b81d9639cd746a2 510w,/images/patterns/diagrams/observer/example-1.5x.png?id=67adccd1030a38dd263bfa09867f8dbe 765w,/images/patterns/diagrams/observer/example-2x.png?id=e2838e1562325e485fc7c2828a8ca445 1020w"
sizes="(max-width: 720px) 100vw, 510px" loading="lazy" width="510"
alt="Struktura przykładu użycia wzorca Obserwator" />
<figcaption><p>Powiadamianie obiektów o zdarzeniach dotyczących
innych obiektów.</p></figcaption>
</figure>

Lista subskrybentów jest kompilowana dynamicznie: obiekty mogą zacząć
lub przestać oczekiwać na powiadomienia w trakcie działania programu,
zależnie od potrzebnego zachowania twojej aplikacji.

W tej implementacji, klasa edytora sama nie zarządza listą
subskrybentów. Deleguje to zadanie specjalnemu obiektowi obsługującemu
właśnie tę czynność. Można ulepszyć ten obiekt, aby służył jako
scentralizowany dyspozytor, pozwalając każdemu obiektowi pełnić rolę
publikującego.

Dodawanie nowych subskrybentów do programu nie wymaga zmian w
istniejących klasach publikujących, o ile będą one współpracować ze
wszystkimi subskrybentami za pośrednictwem tego samego interfejsu.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// Bazowa klasa publikującego zawiera kod zarządzania
// subskrypcją i metody powiadamiania.
class EventManager is
    // Tablica asocjacyjna typów zdarzeń i słuchaczy.
    private field listeners: hash map

    method subscribe(eventType, listener) is
        listeners.add(eventType, listener)

    method unsubscribe(eventType, listener) is
        listeners.remove(eventType, listener)

    method notify(eventType, data) is
        foreach (listener in listeners.of(eventType)) do
            listener.update(data)

// Konkretny publikujący zawiera prawdziwą logikę biznesową
// która jest istotna z punktu widzenia niektórych
// subskrybentów. Moglibyśmy pozyskać tę klasę z klasy bazowej
// publikującego, ale nie jest to zawsze możliwe w prawdziwym
// świecie, ponieważ konkretny publikujący może już być
// podklasą. W takim przypadku można wkomponować logikę
// subskrypcji w sposób pokazany poniżej.
class Editor is
    public field events: EventManager
    private field file: File

    constructor Editor() is
        events = new EventManager()

    // Metody logiki biznesowej mogą powiadamiać subskrybentów o
    // zmianach.
    method openFile(path) is
        this.file = new File(path)
        events.notify(&quot;open&quot;, file.name)

    method saveFile() is
        file.write()
        events.notify(&quot;save&quot;, file.name)

    // ...


// Oto interfejs subskrybenta. Jeśli język programowania którego
// używasz obsługuje typy funkcyjne, możesz wymienić całą
// hierarchię subskrypcji zestawem funkcji.
interface EventListener is
    method update(filename)

// Konkretni subskrybenci reagują na aktualizacje emitowane
// przez obiekt publikujący do którego są przywiązane.
class LoggingListener implements EventListener is
    private field log: File
    private field message: string

    constructor LoggingListener(log_filename, message) is
        this.log = new File(log_filename)
        this.message = message

    method update(filename) is
        log.write(replace(&#39;%s&#39;,filename,message))

class EmailAlertsListener implements EventListener is
    private field email: string
    private field message: string

    constructor EmailAlertsListener(email, message) is
        this.email = email
        this.message = message

    method update(filename) is
        system.email(email, replace(&#39;%s&#39;,filename,message))


// Aplikacja może konfigurować publikujących i subskrybentów w
// trakcie działania programu.
class Application is
    method config() is
        editor = new Editor()

        logger = new LoggingListener(
            &quot;/path/to/log.txt&quot;,
            &quot;Someone has opened the file: %s&quot;)
        editor.events.subscribe(&quot;open&quot;, logger)

        emailAlerts = new EmailAlertsListener(
            &quot;admin@example.com&quot;,
            &quot;Someone has changed the file: %s&quot;)
        editor.events.subscribe(&quot;save&quot;, emailAlerts)</code></pre>
</figure>
:::

:::::::: {.section .applicability-container}
##  Zastosowanie {#applicability}

::::::: applicability
::: applicability-problem
Stosuj wzorzec Obserwator gdy zmiany stanu jednego obiektu mogą wymagać
zmiany w innych obiektach, a konkretny zestaw obiektów nie jest zawczasu
znany lub ulega zmianom dynamicznie.
:::

::: applicability-solution
Można często natknąć się na ten problem pracując z klasami graficznego
interfejsu użytkownika. Przykładowo, stworzyliśmy własne klasy
przycisków i chcemy aby klienci mogli podpiąć jakiś własny kod do twoich
przycisków, aby uruchamiał się po ich naciśnięciu.

Wzorzec Obserwator pozwala każdemu obiektowi implementującemu interfejs
subskrypcji otrzymywać powiadomienia o zdarzeniach w obiektach
publikujących. Można dodać mechanizm subskrypcji do swoich przycisków,
pozwalając klientom na podłączenie do przycisków ich kodu za
pośrednictwem własnych klas subskrybentów.
:::

::: applicability-problem
Stosuj ten wzorzec gdy jakieś obiekty w twojej aplikacji muszą
obserwować inne, ale tylko przez jakiś czas lub w niektórych
przypadkach.
:::

::: applicability-solution
Lista subskrybentów jest dynamiczna, więc subskrybenci mogą zapisać się
lub wypisać z listy kiedy chcą.
:::
:::::::
::::::::

::: {.section .checklist}
##  Jak zaimplementować {#checklist}

1.  Przejrzyj logikę biznesową swojego programu i spróbuj podzielić ją
    na dwie części. Główną funkcjonalność, która jest niezależna od
    innego kodu, uczynimy obiektem publikującym. Reszta natomiast
    zostanie przekształcona na zestaw klas subskrybujących.

2.  Zadeklaruj interfejs subskrybenta. W najprostszej postaci powinien
    deklarować pojedynczą metodę `aktualizuj`.

3.  Zadeklaruj interfejs publikujący i opisz metody służące do
    dodawania i usuwania subskrybentów z listy. Pamiętaj, że obiekty
    publikujące muszą współdziałać z subskrybentami wyłącznie poprzez
    interfejs subskrybenta.

4.  Zdecyduj gdzie umieścić faktyczną listę subskrybentów oraz
    implementację metod zarządzających nią. Zazwyczaj taki kod wygląda
    jednakowo dla wszystkich typów obiektów publikujących, więc
    najbardziej oczywistym miejscem wydaje się klasa abstrakcyjna
    wywodząca się bezpośrednio z interfejsu publikującego. Konkretni
    publikujący rozszerzają tę klasę, dziedzicząc funkcjonalność
    subskrypcji.

    Jednak jeśli stosujesz ten wzorzec w kontekście istniejącej
    hierarchii klas, rozważ podejście bazujące na kompozycji: umieść
    logikę subskrypcji w osobnym obiekcie i pozwól by wszyscy
    publikujący z niej korzystali.

5.  Stwórz konkretne klasy publikujące. Za każdym razem gdy zdarzy się
    coś ważnego w obiekcie publikującym, musi on powiadomić o tym swoich
    subskrybentów.

6.  Zaimplementuj metody powiadamiania o aktualizacji w konkretnych
    klasach subskrybentów. Większość subskrybentów będzie potrzebować
    jakichś danych kontekstowych o zdarzeniu. Można je przekazać w
    formie argumentu metody powiadamiania.

    Istnieje jednak jeszcze jedna opcja. Otrzymawszy powiadomienie,
    subskrybent może pobrać dane bezpośrednio od powiadomienia. W takim
    przypadku, obiekt publikujący musi przekazać samego siebie za
    pośrednictwem metody aktualizacji. Mniej elastyczną opcją jest
    powiązanie obiektu publikującego z subskrybentem na stałe za
    pośrednictwem konstruktora.

7.  Klient musi stworzyć wszystkich niezbędnych subskrybentów i
    zarejestrować ich u odpowiednich publikujących.
:::

:::::: {.section .pros-cons}
##  Zalety i wady {#pros-cons}

::::: row
::: col-sm-6
-    *Zasada otwarte/zamknięte*. Można wprowadzać do programu nowe klasy
    subskrybujące bez konieczności zmieniania kodu publikującego (i
    odwrotnie, jeśli istnieje interfejs publikujący).
-    Można utworzyć związek pomiędzy obiektami w trakcie działania
    programu.
:::

::: col-sm-6
-    Subskrybenci powiadamiani są w przypadkowej kolejności.
:::
:::::
::::::

::: {.section .relations}
##  Powiązania z innymi wzorcami {#relations}

-   Wzorce [Łańcuch zobowiązań](chain-of-responsibility.html),
    [Polecenie](command.html), [Mediator](mediator.html) i
    [Obserwator](observer.html) dotyczą różnych sposobów na łączenie
    nadawców z odbiorcami żądań:

    -   *Łańcuch zobowiązań* przekazuje żądanie sekwencyjnie wzdłuż
        dynamicznego łańcucha potencjalnych odbiorców, aż któryś z
        nich je obsłuży.
    -   *Polecenie* pozwala nawiązywać jednokierunkowe połączenia
        pomiędzy nadawcami i odbiorcami.
    -   *Mediator* eliminuje bezpośrednie połączenia pomiędzy
        nadawcami a odbiorcami, zmuszając ich do komunikacji za
        pośrednictwem obiektu mediator.
    -   *Obserwator* pozwala odbiorcom dynamicznie zasubskrybować się i
        zrezygnować z subskrypcji żądań.

-   Różnica pomiędzy [Mediatorem](mediator.html) a
    [Obserwatorem](observer.html) jest często trudna do uchwycenia. W
    większości przypadków można implementować je zamiennie, a czasem
    jednocześnie. Zobaczmy, jak by to wyglądało.

    Głównym celem *Mediatora* jest eliminacja wzajemnych zależności
    pomiędzy zestawem komponentów systemu. Zamiast tego uzależnia się te
    komponenty od jednego obiektu-mediatora. Celem *Obserwatora* jest
    ustanowienie dynamicznych, jednokierunkowych połączeń między
    obiektami, których część jest podległa innym.

    Istnieje popularna implementacja wzorca Mediator która bazuje na
    *Obserwatorze*. Obiekt mediatora pełni w niej rolę publikującego,
    zaś komponenty są subskrybentami mogącymi "prenumerować" zdarzenia
    które nadaje mediator. Gdy *Mediator* jest zaimplementowany w ten
    sposób, może przypominać *Obserwatora*.

    Jeśli nie jest to zrozumiałe, warto przypomnieć sobie, że można
    zaimplementować wzorzec Mediator na inne sposoby. Na przykład
    można na stałe powiązać wszystkie komponenty z tym samym obiektem
    mediator. Taka implementacja nie będzie przypominać *Obserwatora*,
    ale nadal będzie instancją wzorca Mediator.

    A teraz wyobraźmy sobie program w którym wszystkie komponenty stały
    się publikującymi, pozwalając na dynamiczne połączenia pomiędzy
    sobą. Nie będzie wówczas scentralizowanego obiektu mediatora, a
    tylko rozproszony zestaw obserwatorów.
:::

::: {.section .implementations}
##  Przykłady kodu {#implementations}

[![Obserwator w języku
C#](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](observer/csharp/example.html "Obserwator w języku C#"){.prog-lang-link}
[![Obserwator w języku
C++](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](observer/cpp/example.html "Obserwator w języku C++"){.prog-lang-link}
[![Obserwator w języku
Go](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](observer/go/example.html "Obserwator w języku Go"){.prog-lang-link}
[![Obserwator w języku
Java](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](observer/java/example.html "Obserwator w języku Java"){.prog-lang-link}
[![Obserwator w języku
PHP](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](observer/php/example.html "Obserwator w języku PHP"){.prog-lang-link}
[![Obserwator w języku
Python](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](observer/python/example.html "Obserwator w języku Python"){.prog-lang-link}
[![Obserwator w języku
Ruby](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](observer/ruby/example.html "Obserwator w języku Ruby"){.prog-lang-link}
[![Obserwator w języku
Rust](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](observer/rust/example.html "Obserwator w języku Rust"){.prog-lang-link}
[![Obserwator w języku
Swift](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](observer/swift/example.html "Obserwator w języku Swift"){.prog-lang-link}
[![Obserwator w języku
TypeScript](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](observer/typescript/example.html "Obserwator w języku TypeScript"){.prog-lang-link}
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

[Stan []{.fa .fa-arrow-right}](state.html){.btn .btn-primary rel="next"}
:::

::: prev
#### Wróć

[[]{.fa .fa-arrow-left} Pamiątka](memento.html){.btn .btn-default
rel="prev"}
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
