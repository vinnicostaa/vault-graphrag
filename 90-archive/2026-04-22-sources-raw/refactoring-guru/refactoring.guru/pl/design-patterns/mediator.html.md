::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [Wzorce
projektowe](../design-patterns.html){.type} / [Wzorce
behawioralne](behavioral-patterns.html){.type}
:::

# Mediator {#mediator .title}

::: aka
Znany też jako:
[Intermediary, ]{style="display:inline-block"}[Controller]{style="display:inline-block"}
:::

::: {.section .intent}
##  Cel {#intent}

**Mediator** to behawioralny wzorzec projektowy pozwalający zredukować
chaos zależności pomiędzy obiektami. Wzorzec ten ogranicza bezpośrednią
komunikację pomiędzy obiektami i zmusza je do współpracy wyłącznie za
pośrednictwem obiektu mediatora

<figure class="image">
<img
src="../../images/patterns/content/mediator/mediator6761.png?id=0264bd857a231b6ea2d0c537c092e698"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/mediator/mediator.png?id=0264bd857a231b6ea2d0c537c092e698 640w,/images/patterns/content/mediator/mediator-1.5x.png?id=b3d5df41892274b5c84282bae8737441 960w,/images/patterns/content/mediator/mediator-2x.png?id=250c2bf72ca1fdee2e6d97ed5a4765f2 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Wzorzec projektowy Mediator" />
</figure>
:::

::: {.section .problem}
##  Problem

Załóżmy, że masz okno dialogowe służące tworzeniu i edycji profili
klientów. Składa się z różnych kontrolek, takich jak pola tekstowe, pola
wyboru, przyciski, itd.

<figure class="image">
<img
src="../../images/patterns/diagrams/mediator/problem1-plab3a.png?id=43c07db380e5aeca84f304ea9019d8ee"
style="aspect-ratio: 2.22;"
srcset="/images/patterns/diagrams/mediator/problem1-pl.png?id=43c07db380e5aeca84f304ea9019d8ee 600w,/images/patterns/diagrams/mediator/problem1-pl-1.5x.png?id=1de17ed64afe1760b04a21c94908978c 900w,/images/patterns/diagrams/mediator/problem1-pl-2x.png?id=a994a1c98a09808e61f619c6e34946b5 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Chaotyczne relacje pomiędzy elementami interfejsu użytkownika" />
<figcaption><p>Wraz z ewolucją aplikacji, powiązania między elementami
interfejsu użytkownika mogą stać się coraz
bardziej chaotyczne.</p></figcaption>
</figure>

Niektóre elementy formularza mogą współdziałać z innymi. Przykładowo,
zaznaczenie pola wyboru "Mam psa" spowoduje pojawienie się ukrytego
wcześniej pola tekstowego do wpisywania imienia. Inny przykład to
przycisk "Wyślij" który musi dokonać walidacji danych w polach zanim je
zapisze.

<figure class="image">
<img
src="../../images/patterns/diagrams/mediator/problem2a7f7.png?id=072c51eee4dd90c0972866440c87c1c3"
style="aspect-ratio: 3.27;"
srcset="/images/patterns/diagrams/mediator/problem2.png?id=072c51eee4dd90c0972866440c87c1c3 360w,/images/patterns/diagrams/mediator/problem2-1.5x.png?id=b805abc211e60fa567fb114192e24608 540w,/images/patterns/diagrams/mediator/problem2-2x.png?id=d298ec82a47fa2938ed6cf64b7da78c1 720w"
sizes="(max-width: 720px) 100vw, 360px" loading="lazy" width="360"
alt="Współzależności pomiędzy elementami UI" />
<figcaption><p>Elementy mogą mieć wiele relacji z innymi. W związku z
tym zmiany jednych wpłyną też na inne.</p></figcaption>
</figure>

Implementacja tej logiki bezpośrednio w kodzie elementów formularza
sprawi, że klasy elementów będzie trudno ponownie użyć w innych
formularzach. Przykładowo, nie będzie można użyć klasy pola wyboru w
innym formularzu, ponieważ jest sprzężony z polem tekstowym imienia psa.
Można albo użyć wszystkie klasy biorące udział w renderowaniu formularza
profilu, albo nie używać żadnych.
:::

::: {.section .solution}
##  Rozwiązanie {#solution}

Wzorzec Mediator sugeruje przerwanie bezpośredniej komunikacji między
komponentami które mają być niezależne. W zamian, komponenty te muszą
współpracować pośrednio, wywołując specjalny obiekt mediatora, który
przekierowuje wywołania do odpowiednich komponentów. W wyniku tego
komponenty zależą tylko od pojedynczej klasy mediatora, zamiast
sprzężenia ze sobą nawzajem.

W naszym przykładzie z formularzem edycji profilu, klasa okna
dialogowego może pełnić rolę mediatora. Prawdopodobnie klasa ta wie
już o swoich podelementach, więc nie trzeba nawet wprowadzać nowych
zależności do tej klasy.

<figure class="image">
<img
src="../../images/patterns/diagrams/mediator/solution1-pl893e.png?id=9815ed5a58d0ce232a2a0dc8fa51678e"
style="aspect-ratio: 2.22;"
srcset="/images/patterns/diagrams/mediator/solution1-pl.png?id=9815ed5a58d0ce232a2a0dc8fa51678e 600w,/images/patterns/diagrams/mediator/solution1-pl-1.5x.png?id=2a31e6e6e02b4b3baaa6b36c7d9dc630 900w,/images/patterns/diagrams/mediator/solution1-pl-2x.png?id=a105f78951de80252b4fc0274d920310 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Elementy interfejsu użytkownika powinny komunikować się za pośrednictwem mediatora." />
<figcaption><p>Elementy interfejsu użytkownika powinny komunikować się
pośrednio, poprzez obiekt mediator.</p></figcaption>
</figure>

Najistotniejsza zmiana dotyczy samych elementów formularza. Rozważmy
przycisk wysyłania. Wcześniej, za każdym razem gdy kliknięto przycisk,
musiał on dokonać walidacji wartości we wszystkich elementach
formularza. Teraz jego jedynym zadaniem jest powiadomienie okna
dialogowego o kliknięciu. Otrzymawszy powiadomienie, okno dialogowe
dokonuje walidacji lub przekazuje to zadanie poszczególnym elementom
formularza. Tym samym, zamiast sprzężenia z wieloma elementami, przycisk
zależy jedynie od klasy okna dialogowego.

Można pójść o krok dalej i rozluźnić powiązanie jeszcze bardziej,
ekstrahując wspólny interfejs dla wszystkich typów okien dialogowych.
Taki interfejs deklarowałby metodę powiadamiania, za pomocą której
wszystkie elementy formularza mogłyby powiadamiać okno dialogowe o
zdarzeniach z nimi związanych. Dzięki temu przycisk wysyłania będzie
mógł współdziałać z dowolnym oknem dialogowym implementującym ten
interfejs.

Jest to sposób w jaki wzorzec Mediator pozwala hermetyzować złożone
plątaniny relacji pomiędzy obiektami w jednym obiekcie. Im mniej
zależności ma klasa, tym łatwiej ją modyfikować, rozszerzać lub użyć
ponownie.
:::

::: {.section .analogy}
##  Analogia do prawdziwego życia {#analogy}

<figure class="image">
<img
src="../../images/patterns/diagrams/mediator/live-example3f2a.png?id=aa1de3cb7b63aa623e63578a1e43399a"
style="aspect-ratio: 1.85;"
srcset="/images/patterns/diagrams/mediator/live-example.png?id=aa1de3cb7b63aa623e63578a1e43399a 370w,/images/patterns/diagrams/mediator/live-example-1.5x.png?id=315109aa1099cc6e7c06057fa139881c 555w,/images/patterns/diagrams/mediator/live-example-2x.png?id=fd55a9f9d8cf49effa223555c7369504 740w"
sizes="(max-width: 720px) 100vw, 370px" loading="lazy" width="370"
alt="Wieża kontroli lotów" />
<figcaption><p>Piloci statków powietrznych nie rozmawiają ze sobą
nawzajem bezpośrednio, gdy ustalają kto następny będzie lądował. Cała
komunikacja odbywa się za pośrednictwem wieży
kontroli lotów.</p></figcaption>
</figure>

Piloci statków powietrznych zbliżający się do lotniska lub je
opuszczający nie rozmawiają ze sobą nawzajem, lecz za pośrednictwem
kontrolera lotów, siedzącego w wieży z widokiem na lądowisko. Bez
kontrolera, piloci musieliby wiedzieć o każdym samolocie lub śmigłowcu w
okolicy lotniska, dyskutować na temat pierwszeństwa lądowania.
Mogłoby to niekorzystnie wpłynąć na statystyki bezpieczeństwa\...

Wieża nie musi kontrolować całego lotu. Jej funkcja służy nakładaniu
ograniczeń w obszarze terminala aby żaden z pilotów nie był przytłoczony
dużą ilością aktorów biorących udział w funkcjonowaniu lotniska.
:::

::::: {.section .structure-container}
##  Struktura {#structure}

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/mediator/structure71f0.png?id=1f2accc7820ecfe9665b6d30cbc0bc61"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.21;"
srcset="/images/patterns/diagrams/mediator/structure.png?id=1f2accc7820ecfe9665b6d30cbc0bc61 520w,/images/patterns/diagrams/mediator/structure-1.5x.png?id=958b373815bf6a56cd9b38763ed01dce 780w,/images/patterns/diagrams/mediator/structure-2x.png?id=5191daa1c9d4caa36e38af3c5b7d1522 1040w"
sizes="(max-width: 720px) 100vw, 520px" loading="lazy" width="520"
alt="Struktura wzorca projektowego Mediator" /><img
src="../../images/patterns/diagrams/mediator/structure-indexed5302.png?id=a82d4cf1b92a4f72af32f231ffd21131"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.21;"
srcset="/images/patterns/diagrams/mediator/structure-indexed.png?id=a82d4cf1b92a4f72af32f231ffd21131 520w,/images/patterns/diagrams/mediator/structure-indexed-1.5x.png?id=81d4f842ae5157a3cb09f8f3c05159dd 780w,/images/patterns/diagrams/mediator/structure-indexed-2x.png?id=88722da01a5c82b0452078c9339ca497 1040w"
sizes="(max-width: 720px) 100vw, 520px" loading="lazy" width="520"
alt="Struktura wzorca projektowego Mediator" />
</figure>
:::

1.  **Komponenty** to różne klasy zawierające jakąś logikę biznesową.
    Każdy komponent posiada odniesienie do mediatora, zadeklarowany jako
    typ interfejsu mediatora. Komponent ten nie jest świadom faktycznej
    klasy obiektu mediatora, więc można go używać ponownie w innych
    programach, łącząc z innym mediatorem.

2.  Interfejs **Mediator** deklaruje metody komunikacji z komponentami,
    które na ogół ograniczają się do jednej metody powiadamiania.
    Komponenty mogą przekazywać dowolny kontekst jako argument tej
    metody, włącznie z samymi sobą, ale wyłącznie w sposób który nie
    spowoduje sprzęgnięcia komponentu otrzymującego z klasą nadawcy.

3.  **Konkretni Mediatorzy** hermetyzują relacje pomiędzy różnorakimi
    komponentami. Konkretni mediatorzy często przechowują odniesienia do
    wszystkich komponentów jakimi zarządzają i czasem nawet zarządzają
    ich cyklem życia.

4.  Komponenty nie mogą być świadome innych komponentów. Jeśli coś
    istotnego zdarzy się w innym komponencie, musi on powiadamiać
    wyłącznie mediatora. Gdy zaś ten otrzyma powiadomienie, może w
    prosty sposób zidentyfikować nadawcę i to czasem wystarcza, by
    zdecydować jaki komponent uruchomić w odpowiedzi.

    Z perspektywy komponentu, wszystko przypomina czarną skrzynkę.
    Nadawca nie wie kto ostatecznie obsłuży jego żądanie, a odbiorca nie
    wie kto je nadał.
::::
:::::

::: {.section .pseudocode}
##  Pseudokod {#pseudocode}

W poniższym przykładzie, wzorzec **Mediator** pomaga wyeliminować
wzajemne zależności pomiędzy różnymi klasami UI: przyciskami, polami
wyboru i etykietami tekstowymi.

<figure class="image">
<img
src="../../images/patterns/diagrams/mediator/example576a.png?id=3151c153533e816e226be0ef977211e8"
style="aspect-ratio: 1.24;"
srcset="/images/patterns/diagrams/mediator/example.png?id=3151c153533e816e226be0ef977211e8 560w,/images/patterns/diagrams/mediator/example-1.5x.png?id=cf441780d96f3306ac54d6809f85b87d 840w,/images/patterns/diagrams/mediator/example-2x.png?id=02064e5a7c4f065f806747e1b04ac1b0 1120w"
sizes="(max-width: 720px) 100vw, 560px" loading="lazy" width="560"
alt="Struktura przykładu użycia wzorca Mediator" />
<figcaption><p>Struktura klas okna dialogowego.</p></figcaption>
</figure>

Element, uruchomiony przez użytkownika, nie komunikuje się
bezpośrednio z innymi elementami, nawet jeśli wydaje się, że mógłby. W
zamian element musi dać znać o zdarzeniu tylko swojemu mediatorowi,
przekazując ewentualne dane kontekstowe wraz z powiadomieniem.

W tym przykładzie, całe okno dialogowe uwierzytelniania pełni rolę
mediatora. Wie jak konkretne elementy powinny współpracować i
zapewnia im pośrednią komunikację ze sobą. Otrzymawszy powiadomienie o
zdarzeniu, okno dialogowe decyduje który element powinien zająć się jego
obsługą i odpowiednio przekierowuje wywołanie.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// Interfejs mediatora deklaruje metodę za pomocą której
// komponenty powiadamiają go o różnych zdarzeniach. Mediator
// może zareagować na te zdarzenia i przekazać wykonanie innym
// komponentom.
interface Mediator is
    method notify(sender: Component, event: string)


// Konkretna klasa mediator. Splątana sieć połączeń pomiędzy
// poszczególnymi komponentami została uporządkowana i
// przeniesiona do mediatora.
class AuthenticationDialog implements Mediator is
    private field title: string
    private field loginOrRegisterChkBx: Checkbox
    private field loginUsername, loginPassword: Textbox
    private field registrationUsername, registrationPassword,
                  registrationEmail: Textbox
    private field okBtn, cancelBtn: Button

    constructor AuthenticationDialog() is
        // Utwórz wszystkie obiekty-komponenty i przekaż bieżący
        // mediator ich konstruktorom by ustanowić połączenia.

    // Gdy coś się zdarzy komponentowi, poinformuje on o tym
    // mediatora. Mediator otrzymawszy powiadomienie może coś
    // zrobić, lub przekazać żądanie innemu komponentowi.
    method notify(sender, event) is
        if (sender == loginOrRegisterChkBx and event == &quot;check&quot;)
            if (loginOrRegisterChkBx.checked)
                title = &quot;Log in&quot;
                // 1. Pokaż komponenty formularza logowania.
                // 2. Ukryj komponenty formularza rejestracji.
            else
                title = &quot;Register&quot;
                // 1. Pokaż komponenty formularza rejestracji.
                // 2. Ukryj komponenty formularza logowania.

        if (sender == okBtn &amp;&amp; event == &quot;click&quot;)
            if (loginOrRegister.checked)
                // Spróbuj znaleźć użytkownika po
                // poświadczeniach.
                if (!found)
                    // Pokaż komunikat błędu nad polem nazwy
                    // użytkownika.
            else
                // 1. Utwórz konto użytkownika korzystając z
                // danych w polach formularza rejestracji.
                // 2. Zaloguj użytkownika.
                // ...

// Komponenty współpracują z mediatorem za pośrednictwem
// interfejsu mediatora. Dzięki temu można używać tych samych
// komponentów w różnych kontekstach poprzez łączenie ich z
// różnymi obiektami mediator.
class Component is
    field dialog: Mediator

    constructor Component(dialog) is
        this.dialog = dialog

    method click() is
        dialog.notify(this, &quot;click&quot;)

    method keypress() is
        dialog.notify(this, &quot;keypress&quot;)

// Konkretne komponenty nie komunikują się ze sobą. Mają tylko
// jeden kanał komunikacyjny, którym przesyłają powiadomienia
// mediatorowi.
class Button extends Component is
    // ...

class Textbox extends Component is
    // ...

class Checkbox extends Component is
    method check() is
        dialog.notify(this, &quot;check&quot;)
    // ...</code></pre>
</figure>
:::

:::::::::: {.section .applicability-container}
##  Zastosowanie {#applicability}

::::::::: applicability
::: applicability-problem
Stosuj wzorzec Mediator gdy zmiana jakichś klas jest trudna z powodu
ścisłego sprzęgnięcia z innymi klasami.
:::

::: applicability-solution
Wzorzec pozwala wyekstrahować wszystkie relacje pomiędzy klasami do
osobnej klasy, izolując ewentualne zmiany określonego komponentu od
reszty komponentów.
:::

::: applicability-problem
Stosuj ten wzorzec gdy nie możesz ponownie użyć jakiegoś komponentu w
innym programie, z powodu zbytniej jego zależności od innych
komponentów.
:::

::: applicability-solution
 Po zastosowaniu wzorca Mediator, pojedyncze komponenty stają się
nieświadome innych komponentów. Nadal mogą komunikować się ze sobą, ale
pośrednio --- poprzez obiekt mediator. Aby ponownie użyć komponent w
innej aplikacji, musisz zapewnić mu nową klasę mediator.
:::

::: applicability-problem
Stosuj wzorzec Mediator gdy zauważysz, że tworzysz mnóstwo podklas
komponentu tylko aby móc ponownie użyć jakieś zachowanie w innych
kontekstach.
:::

::: applicability-solution
Skoro wszystkie relacje pomiędzy komponentami zawierają się w obrębie
mediatora, łatwo zdefiniować całkowicie nowe metody współpracy tych
komponentów wprowadzając nowe klasy mediatorów, bez konieczności zmian w
samych komponentach.
:::
:::::::::
::::::::::

::: {.section .checklist}
##  Jak zaimplementować {#checklist}

1.  Zidentyfikuj grupę ściśle sprzężonych klas które zyskałyby na
    niezależności (np. dla łatwiejszego utrzymania lub ponownego
    użycia).

2.  Zadeklaruj interfejs mediatora i określ potrzebny protokół
    komunikacji pomiędzy mediatorami i innymi komponentami. W większości
    przypadków, wystarczy pojedyncza metoda otrzymywania powiadomień od
    komponentów.

    Taki interfejs jest kluczowy gdy chcemy ponownie wykorzystać klasy
    komponentów w innych kontekstach. O ile komponent współpracuje ze
    swoim mediatorem za pośrednictwem ogólnego interfejsu, można łączyć
    komponent z innymi implementacjami mediatora.

3.  Zaimplementuj konkretną klasę mediator. Najlepiej, gdyby klasa ta
    przechowywała odniesienia do wszystkich komponentów jakimi zarządza.

4.  Można nawet pójść o krok dalej i uczynić mediatora
    odpowiedzialnym za tworzenie i niszczenie obiektów komponentów.
    Wówczas mediator zacznie przypominać
    [fabrykę](abstract-factory.html) lub [fasadę](facade.html).

5.  Komponenty powinny przechowywać odniesienie do obiektu mediatora.
    Połączenie zwykle nawiązuje się w konstruktorze komponentu, do
    którego obiekt mediatora przekazywany jest w roli argumentu.

6.  Zmień kod komponentów tak, aby wywoływały metodę powiadamiania
    mediatora zamiast metod powiadamiania innych komponentów.
    Wyekstrahuj kod zawierający wywołania do innych komponentów do klasy
    mediatora. Wykonuj ten kod gdy mediator otrzyma powiadomienie od
    komponentu.
:::

:::::: {.section .pros-cons}
##  Zalety i wady {#pros-cons}

::::: row
::: col-sm-6
-    *Zasada pojedynczej odpowiedzialności*. Możesz wyekstrahować
    komunikację pomiędzy różnymi komponentami w jedno miejsce,
    czyniąc ją łatwiejszą do zrozumienia i utrzymania.
-    *Zasada otwarte/zamknięte*. Można wprowadzać kolejnych mediatorów
    bez konieczności zmiany samych komponentów.
-    Można zredukować sprzężenie pomiędzy różnymi komponentami programu.
-    Można ułatwić ponowne wykorzystanie komponentów.
:::

::: col-sm-6
-    Z czasem mediator może ewoluować do postaci [Boskiego
    Obiektu](https://en.wikipedia.org/wiki/God_object).
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

-   [Fasada](facade.html) i [Mediator](mediator.html) mają podobne
    zadania: służą zorganizowaniu współpracy pomiędzy wieloma ściśle
    sprzęgniętymi klasami.

    -   *Fasada* definiuje uproszczony interfejs podsystemu obiektów,
        ale nie wprowadza nowej funkcjonalności. Podsystem jest
        nieświadomy istnienia fasady. Obiekty w obrębie podsystemu mogą
        komunikować się bezpośrednio.
    -   *Mediator* centralizuje komunikację pomiędzy komponentami
        podsystemu. Komponenty wiedzą tylko o obiekcie mediator i nie
        komunikują się ze sobą bezpośrednio.

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

[![Mediator w języku
C#](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](mediator/csharp/example.html "Mediator w języku C#"){.prog-lang-link}
[![Mediator w języku
C++](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](mediator/cpp/example.html "Mediator w języku C++"){.prog-lang-link}
[![Mediator w języku
Go](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](mediator/go/example.html "Mediator w języku Go"){.prog-lang-link}
[![Mediator w języku
Java](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](mediator/java/example.html "Mediator w języku Java"){.prog-lang-link}
[![Mediator w języku
PHP](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](mediator/php/example.html "Mediator w języku PHP"){.prog-lang-link}
[![Mediator w języku
Python](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](mediator/python/example.html "Mediator w języku Python"){.prog-lang-link}
[![Mediator w języku
Ruby](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](mediator/ruby/example.html "Mediator w języku Ruby"){.prog-lang-link}
[![Mediator w języku
Rust](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](mediator/rust/example.html "Mediator w języku Rust"){.prog-lang-link}
[![Mediator w języku
Swift](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](mediator/swift/example.html "Mediator w języku Swift"){.prog-lang-link}
[![Mediator w języku
TypeScript](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](mediator/typescript/example.html "Mediator w języku TypeScript"){.prog-lang-link}
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

[Pamiątka []{.fa .fa-arrow-right}](memento.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Wróć

[[]{.fa .fa-arrow-left} Iterator](iterator.html){.btn .btn-default
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
