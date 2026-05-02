::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [Wzorce
projektowe](../design-patterns.html){.type} / [Wzorce
behawioralne](behavioral-patterns.html){.type}
:::

# Polecenie {#polecenie .title}

::: aka
Znany też jako:
[Action, ]{style="display:inline-block"}[Transaction, ]{style="display:inline-block"}[Command]{style="display:inline-block"}
:::

::: {.section .intent}
##  Cel {#intent}

**Polecenie** jest behawioralnym wzorcem projektowym który zmienia
żądanie w samodzielny obiekt zawierający wszystkie informacje o tym
żądaniu. Taka transformacja pozwala na parametryzowanie metod przy
użyciu różnych żądań. Oprócz tego umożliwia opóźnianie lub kolejkowanie
wykonywania żądań oraz pozwala na cofanie operacji.

<figure class="image">
<img
src="../../images/patterns/content/command/command-plc178.png?id=4b9a63143bde905546346c533053337f"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/command/command-pl.png?id=4b9a63143bde905546346c533053337f 640w,/images/patterns/content/command/command-pl-1.5x.png?id=43e9857b985a5def2be7e144450d4baa 960w,/images/patterns/content/command/command-pl-2x.png?id=3192c6ca4669aae28acac8956b576250 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Wzorzec projektowy Polecenie" />
</figure>
:::

::: {.section .problem}
##  Problem

Wyobraź sobie, że pracujesz nad nowym edytorem tekstu. Tworzysz pasek
narzędziowy z przyciskami wywołującymi różne działania edytora. Masz już
elegancką klasę `Przycisk` która może być używana zarówno do przycisków
paska, a także jako ogólne przyciski w różnych oknach dialogowych.

<figure class="image">
<img
src="../../images/patterns/diagrams/command/problem163e2.png?id=84189315a0e8d91da262792979005ab4"
style="aspect-ratio: 0.88;"
srcset="/images/patterns/diagrams/command/problem1.png?id=84189315a0e8d91da262792979005ab4 230w,/images/patterns/diagrams/command/problem1-1.5x.png?id=77bf0bc58e5c9c9b8369bc4f8dec457f 345w,/images/patterns/diagrams/command/problem1-2x.png?id=af4c4e9c1d1b4fa2c4104c5f6bb18719 460w"
sizes="(max-width: 720px) 100vw, 230px" loading="lazy" width="230"
alt="Rozwiązanie problemu za pomocą wzorca Polecenie" />
<figcaption><p>Wszystkie przyciski w aplikacji wywodzą się z tej
samej klasy.</p></figcaption>
</figure>

Mimo, że wszystkie te przyciski wyglądają podobnie, to mają wywoływać
różne działania. Gdzie więc umieścić kod obiektów obsługujących
kliknięcia przycisków? Najprostszym rozwiązaniem jest stworzenie wielu
podklas dla każdego przypadku użycia przycisku. Takie podklasy
zawierałyby kod wykonywany po wciśnięciu przycisku.

<figure class="image">
<img
src="../../images/patterns/diagrams/command/problem2f2f9.png?id=f0e33da1842b3a3ee3b4857de0b6ec93"
style="aspect-ratio: 2.11;"
srcset="/images/patterns/diagrams/command/problem2.png?id=f0e33da1842b3a3ee3b4857de0b6ec93 400w,/images/patterns/diagrams/command/problem2-1.5x.png?id=7ae15e2b07d5d076a878d99c3ed8ac32 600w,/images/patterns/diagrams/command/problem2-2x.png?id=5eea7d0f6247da011150bb63e423f405 800w"
sizes="(max-width: 720px) 100vw, 400px" loading="lazy" width="400"
alt="Mnóstwo podklas przycisku" />
<figcaption><p>Mnóstwo podklas przycisku. Co może pójść
nie tak?</p></figcaption>
</figure>

Szybko zauważasz, że to podejście jest wadliwe. Powstanie wielka liczba
podklas, co byłoby akceptowalne, gdybyśmy przy okazji nie ryzykowali
popsucia kodu tych podklas przy każdej zmianie klasy bazowej `Przycisk`.
Kod twojego interfejsu użytkownika byłby zależny od zmiennego kodu
logiki biznesowej.

<figure class="image">
<img
src="../../images/patterns/diagrams/command/problem3-plb698.png?id=1468f43b3a213c0bd1f632ea9c4dbc90"
style="aspect-ratio: 4.36;"
srcset="/images/patterns/diagrams/command/problem3-pl.png?id=1468f43b3a213c0bd1f632ea9c4dbc90 480w,/images/patterns/diagrams/command/problem3-pl-1.5x.png?id=e2ff0e3b7693e4632d9cabba699077c7 720w,/images/patterns/diagrams/command/problem3-pl-2x.png?id=45f30e8d9d0ef55a94baf285b4982692 960w"
sizes="(max-width: 720px) 100vw, 480px" loading="lazy" width="480"
alt="Wiele klas implementuje tę samą funkcjonalność" />
<figcaption><p>Wiele klas implementuje tę
samą funkcjonalność.</p></figcaption>
</figure>

Ale to jeszcze nie wszystko. Niektóre operacje, jak kopiowanie/wklejanie
tekstu, powinny być dostępne z wielu miejsc: po kliknięciu na mały
przycisk "Kopiuj" na pasku narzędziowym, wybraniu z menu kontekstowego,
czy też po wciśnięciu skrótu klawiszowego `Ctrl+C`.

Na początku, gdy nasza aplikacja posiadała tylko pasek narzędziowy,
umieszczenie implementacji operacji w podklasach przycisku miało sens.
Innymi słowy, trzymanie kodu służącego do kopiowaniu tekstu w obrębie
podklasy `PrzyciskKopiuj` było w porządku. Jednak po zaimplementowaniu
menu kontekstowego, skrótów i innych --- trzeba duplikować kod
operacji w wielu klasach lub uczynić menu zależnymi od przycisków, co
jest jeszcze gorszą opcją.
:::

::: {.section .solution}
##  Rozwiązanie {#solution}

Dobre projektowanie oprogramowania często bazuje na *zasadzie separacji
odpowiedzialności*, co zwykle skutkuje podziałem aplikacji na warstwy.
Najczęstszy przykład: warstwa graficznego interfejsu użytkownika i
warstwa logiki biznesowej. Pierwsza jest odpowiedzialna za renderowanie
pięknego obrazu na ekranie, przechwytywanie sygnałów na wejściu i
wyświetlanie efektów pracy użytkownika i aplikacji. Jednak gdy chodzi o
wykonywanie ważnych zadań, jak obliczanie trajektorii księżyca, lub
generowanie rocznego bilansu, warstwa interfejsu użytkownika deleguje te
zadania warstwie logiki biznesowej.

W kodzie wyglądałoby to na przykład tak: obiekt graficznego interfejsu
użytkownika wywołuje metodę obiektu logiki biznesowej, przekazując jej
jakieś argumenty. Proces ten zwykle można opisać jako przesłanie
*żądania* przez jeden obiekt drugiemu obiektowi.

<figure class="image">
<img
src="../../images/patterns/diagrams/command/solution1-pl5700.png?id=24afecfef37ccf045a6e38b335411033"
style="aspect-ratio: 2.47;"
srcset="/images/patterns/diagrams/command/solution1-pl.png?id=24afecfef37ccf045a6e38b335411033 470w,/images/patterns/diagrams/command/solution1-pl-1.5x.png?id=df676906af0695039fdbe0cb7b63aca3 705w,/images/patterns/diagrams/command/solution1-pl-2x.png?id=985510d8cdb070922055048acb5a83e1 940w"
sizes="(max-width: 720px) 100vw, 470px" loading="lazy" width="470"
alt="Warstwa graficznego interfejsu użytkownika może mieć bezpośredni dostęp do warstwy logiki biznesowej" />
<figcaption><p>Obiekty graficznego interfejsu użytkownika mogą mieć
bezpośredni dostęp do obiektów warstwy
logiki biznesowej.</p></figcaption>
</figure>

Według wzorca Polecenie, obiekty GUI nie powinny wysyłać żądań
bezpośrednio. Zamiast tego należy wyekstrahować szczegóły żądania, takie
jak obiekt docelowy, nazwę metody i listę argumentów do osobnej klasy
*polecenie* posiadającej tylko jedną metodę --- wywołującą to żądanie.

Obiekty polecenie stanowią łącza pomiędzy obiektami interfejsu
użytkownika i logiki biznesowej. Od teraz, obiekt GUI nie musi wiedzieć
który obiekt logiki biznesowej otrzyma żądanie i jak je obsłuży. Obiekt
interfejsu użytkownika jedynie wywołuje polecenie, a ono samo zajmuje
się szczegółami.

<figure class="image">
<img
src="../../images/patterns/diagrams/command/solution2-pl7852.png?id=944b27ff007c626dacef5dcd9efe31d2"
style="aspect-ratio: 2.89;"
srcset="/images/patterns/diagrams/command/solution2-pl.png?id=944b27ff007c626dacef5dcd9efe31d2 550w,/images/patterns/diagrams/command/solution2-pl-1.5x.png?id=72fcaff12c3f9c4123bdb540ad6d1e73 825w,/images/patterns/diagrams/command/solution2-pl-2x.png?id=b3e49f95b52ea71a8751ad95e0325cbc 1100w"
sizes="(max-width: 720px) 100vw, 550px" loading="lazy" width="550"
alt="Dostęp do warstwy logiki biznesowej za pośrednictwem polecenia." />
<figcaption><p>Dostęp do warstwy logiki biznesowej za
pośrednictwem polecenia.</p></figcaption>
</figure>

Kolejnym etapem jest zaimplementowanie wszystkim poleceniom jednakowego
interfejsu. Zazwyczaj posiada on jedną tylko metodę wywołującą
działanie, która nie przyjmuje parametrów. Taki interfejs pozwala
jednemu nadawcy wywoływać wiele różnych poleceń bez konieczności
sprzęgania go z konkretnymi klasami poleceń. Dodatkowo można teraz
wymieniać obiekty-polecenia powiązane z nadawcą, a tym samym zmieniać
jego zachowanie w trakcie działania programu.

Brakuje jeszcze jednego elementu układanki --- parametrów żądania.
Obiekt graficznego interfejsu użytkownika mógł dostarczyć obiektowi
warstwy logiki biznesowej jakieś parametry. Skoro wykonanie polecenia
nie przyjmuje żadnych parametrów, to jak przekazać odbiorcy szczegóły
żądania? Otóż albo te dane powinny być wcześniej skonfigurowane w
poleceniu, albo polecenie powinno móc pozyskać je samodzielnie.

<figure class="image">
<img
src="../../images/patterns/diagrams/command/solution3-plbcd7.png?id=84110d77fcb480b45cfa6b2138b59fce"
style="aspect-ratio: 1.83;"
srcset="/images/patterns/diagrams/command/solution3-pl.png?id=84110d77fcb480b45cfa6b2138b59fce 440w,/images/patterns/diagrams/command/solution3-pl-1.5x.png?id=a275dc4733a9f53694d1582be4bb1c1d 660w,/images/patterns/diagrams/command/solution3-pl-2x.png?id=449f395f351cee014f9b552de9ffde3b 880w"
sizes="(max-width: 720px) 100vw, 440px" loading="lazy" width="440"
alt="Obiekty graficznego interfejsu użytkownika delegują pracę poleceniom" />
<figcaption><p>Obiekty graficznego interfejsu użytkownika delegują
pracę poleceniom.</p></figcaption>
</figure>

Wróćmy do naszego edytora tekstu. Po zastosowaniu wzorca Polecenie, nie
potrzebujemy tych wszystkich podklas przycisku, by zaimplementować różne
reakcje na kliknięcie. Wystarczy umieścić w klasie bazowej `Przycisk`
jedno pole przechowujące odniesienie do obiektu typu polecenie i
sprawić, by kliknięcie powodowało uruchomienie tego polecenia.

Należy zaimplementować kilka klas polecenie dla każdej możliwej
operacji i połączyć je z konkretnymi przyciskami, zależnie od planowanej
reakcji na wciskanie ich.

Inne elementy GUI, jak menu, skróty czy całe okna dialogowe można
zaimplementować w taki sam sposób: powiązać je z poleceniem które będzie
uruchamiane w odpowiedzi na interakcję użytkownika z danym elementem.
Jak być może się już domyślasz, elementy związane z tymi samymi
działaniami będą połączone z tymi samymi poleceniami, zapobiegając tym
samym duplikacji kodu.

W rezultacie polecenia stają się poręczną warstwą pośrednią redukującą
sprzężenie pomiędzy graficznym elementem użytkownika i warstwami logiki
biznesowej. A to tylko część zysków płynących z użycia wzorca Polecenie!
:::

::: {.section .analogy}
##  Analogia do prawdziwego życia {#analogy}

<figure class="image">
<img
src="../../images/patterns/content/command/command-comic-16e6f.png?id=551df832f445080976f3116e0dc120c9"
style="aspect-ratio: 2;"
srcset="/images/patterns/content/command/command-comic-1.png?id=551df832f445080976f3116e0dc120c9 600w,/images/patterns/content/command/command-comic-1-1.5x.png?id=82dc5c38ce372ed4dfd4a37c04faeb72 900w,/images/patterns/content/command/command-comic-1-2x.png?id=47b3c00b2cfbda7157a1ed9da6ce2948 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Składanie zamówienia w restauracji" />
<figcaption><p>Składanie zamówienia w restauracji.</p></figcaption>
</figure>

Podczas długiego spaceru po mieście, docierasz do miłej restauracji i
siadasz przy oknie. Przyjazny kelner szybko przyjmuje zamówienie,
spisując je na małym kawałku papieru. Następnie kelner idzie do kuchni i
przykleja kartkę na ścianie. Po jakimś czasie zamówienie dociera do
szefa kuchni, który przygotowuje danie, a następnie umieszcza posiłek na
tacce wraz z zamówieniem. Kelner znajduje tackę, sprawdza zgodność z
zamówieniem i zanosi ją do stolika.

Zamówienie na papierze stanowi polecenie. Trafia do kolejki, do
momentu aż szef kuchni je przygotuje. Zamówienie zawiera wszystkie
niezbędne informacje wymagane do przygotowania posiłku. Umożliwia to
kucharzowi rozpoczęcie gotowania od razu, zamiast ustalać szczegóły z
klientem na własną rękę.
:::

::::: {.section .structure-container}
##  Struktura {#structure}

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/command/structure44ec.png?id=1cd7833638f4c43630f4a84017d31195"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.7;"
srcset="/images/patterns/diagrams/command/structure.png?id=1cd7833638f4c43630f4a84017d31195 630w,/images/patterns/diagrams/command/structure-1.5x.png?id=6e5b68277c7747b22266e408891d5841 945w,/images/patterns/diagrams/command/structure-2x.png?id=1dfaa84354ffe49ef7ad46ce069482d2 1260w"
sizes="(max-width: 720px) 100vw, 630px" loading="lazy" width="630"
alt="Struktura wzorca projektowego Polecenie" /><img
src="../../images/patterns/diagrams/command/structure-indexed49b8.png?id=95529d7282dc7bc1c5bc443423b1cf4f"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.64;"
srcset="/images/patterns/diagrams/command/structure-indexed.png?id=95529d7282dc7bc1c5bc443423b1cf4f 640w,/images/patterns/diagrams/command/structure-indexed-1.5x.png?id=29d6c5c7a06da2747ed756c0ddad6169 960w,/images/patterns/diagrams/command/structure-indexed-2x.png?id=e4cc286a44768c7d060af33497da7df6 1280w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="640"
alt="Struktura wzorca projektowego Polecenie" />
</figure>
:::

1.  Klasa **Nadawca** (lub *wywołująca*) jest odpowiedzialna za
    inicjowanie żądań. Musi ona zawierać pole przechowujące
    odniesienia do obiektu polecenia. Nadawca uruchamia polecenie
    zamiast przesyłać żądanie bezpośrednio do odbiorcy. Zauważ, że
    nadawca nie jest odpowiedzialny za tworzenie obiektu polecenie.
    Zazwyczaj otrzymuje wcześniej przygotowane polecenie od klienta za
    pośrednictwem konstruktora.

2.  Interfejs **Polecenie** zwykle deklaruje pojedynczą metodę służącą
    wykonaniu polecenia.

3.  **Konkretne polecenia** implementują różne rodzaje żądań. Konkretne
    polecenie nie powinno wykonywać pracy samodzielnie, lecz
    przekazać je do jednego z obiektów logiki biznesowej. Jednak dla
    uproszczenia kodu, klasy te można złączyć.

    Parametry potrzebne do uruchomienia metody na obiekcie odbiorcy
    można zadeklarować w formie pól konkretnego polecenia. Obiekty
    poleceń można uczynić niezmienialnymi, zezwalając na inicjalizację
    tych pól wyłącznie za pośrednictwem konstruktora.

4.  Klasa **Odbiorca** zawiera jakąś logikę biznesową. Prawie każdy
    obiekt może pełnić rolę odbiorcy. Większość poleceń obsługuje tylko
    szczegóły przekazania żądania do odbiorcy, zaś faktyczną pracę
    wykonuje ten ostatni.

5.  **Klient** tworzy i konfiguruje konkretne obiekty żądań. Klient musi
    przekazać wszystkie parametry żądania, włącznie z instancją
    odbiorcy, do konstruktora polecenia. Następnie otrzymane polecenie
    można skojarzyć z jednym lub wieloma nadawcami.
::::
:::::

::: {.section .pseudocode}
##  Pseudokod {#pseudocode}

W poniższym przykładzie, wzorzec **Polecenie** pozwala śledzić historię
wykonanych działań i umożliwia cofnięcie danej operacji jeśli zaistnieje
potrzeba.

<figure class="image">
<img
src="../../images/patterns/diagrams/command/example31bf.png?id=1f42c8395fe54d0e409026b91881e2a0"
style="aspect-ratio: 1.07;"
srcset="/images/patterns/diagrams/command/example.png?id=1f42c8395fe54d0e409026b91881e2a0 640w,/images/patterns/diagrams/command/example-1.5x.png?id=435055a78a82c005eebb1bef869998ae 960w,/images/patterns/diagrams/command/example-2x.png?id=ed44dfd9b02eccf1ae7e2714d018ed36 1280w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="640"
alt="Struktura przykładu użycia wzorca Polecenie" />
<figcaption><p>Odwracalne działania w edytorze tekstu.</p></figcaption>
</figure>

Polecenia skutkujące zmianą stanu edytora (na przykład wycinanie i
wklejanie tekstu) wykonują kopię stanu edytora zanim wywołają działanie
skojarzone z tym poleceniem. Po wykonaniu polecenia jest ono
umieszczane w historii poleceń (stos obiektów polecenie) wraz z kopią
zapasową stanu edytora na tamten moment. Jeśli użytkownik zechce cofnąć
jakieś działanie, aplikacja może pobrać ostatnie polecenie z historii,
odczytać skojarzoną z nim kopię zapasową stanu edytora i przywrócić ją.

Kod klienta (elementy GUI, historia poleceń, itd.) nie jest sprzężony z
konkretnymi klasami poleceń ponieważ współpracuje z poleceniami za
pośrednictwem interfejsu polecenia. Takie podejście pozwala wdrożyć nowe
polecenia do aplikacji bez psucia istniejącego kodu.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// Bazowa klasa polecenie definiuje wspólny interfejs wszystkich
// konkretnych poleceń.
abstract class Command is
    protected field app: Application
    protected field editor: Editor
    protected field backup: text

    constructor Command(app: Application, editor: Editor) is
        this.app = app
        this.editor = editor

    // Zrób kopię zapasową stanu edytora.
    method saveBackup() is
        backup = editor.text

    // Przywróć stan edytora.
    method undo() is
        editor.text = backup

    // Metoda wykonująca zadeklarowana jest jako abstrakcyjna,
    // aby zmusić wszystkie konkretne polecenia do
    // zaimplementowania jej we własnym zakresie. Metoda musi
    // zwracać prawdę lub fałsz zależnie od tego, czy polecenie
    // zmienia stan edytora lub pozostawia stan bez zmian.
    abstract method execute()


// Tu umieszczane są konkretne polecenia.
class CopyCommand extends Command is
    // Polecenie kopiuj nie jest zapisywane w historii ponieważ
    // nie zmienia stanu edytora.
    method execute() is
        app.clipboard = editor.getSelection()
        return false

class CutCommand extends Command is
    // Polecenie wytnij zmienia stan edytora, dlatego trzeba
    // zapisać stan w historii. Stan będzie zapisywany zawsze
    // gdy metoda zwróci prawdę.
    method execute() is
        saveBackup()
        app.clipboard = editor.getSelection()
        editor.deleteSelection()
        return true

class PasteCommand extends Command is
    method execute() is
        saveBackup()
        editor.replaceSelection(app.clipboard)
        return true

// Funkcja cofania operacji również jest poleceniem.
class UndoCommand extends Command is
    method execute() is
        app.undo()
        return false


// Globalna historia poleceń jest po prostu stosem.
class CommandHistory is
    private field history: array of Command

    // Ostatni na wejściu...
    method push(c: Command) is
        // Wepchnij polecenie na koniec tablicy historii.

    // ...pierwszy na wyjściu.
    method pop():Command is
        // Pobierz najświeższe polecenie z historii.


// Klasa edytora posiada narzędzia do edycji tekstu. Pełni rolę
// odbiorcy: wszystkie polecenia delegują faktyczne wykonanie
// metodom edytora.
class Editor is
    field text: string

    method getSelection() is
        // Zwróć zaznaczony tekst.

    method deleteSelection() is
        // Skasuj zaznaczony tekst.

    method replaceSelection(text) is
        // Wstaw zawartość schowka w bieżącym miejscu.


// Klasa aplikacji ustanawia relacje między obiektami. Pełni
// rolę nadawcy: gdy trzeba coś zrobić, tworzy obiekt polecenia
// i uruchamia go.
class Application is
    field clipboard: string
    field editors: array of Editors
    field activeEditor: Editor
    field history: CommandHistory

    // Kod przypisujący polecenia do obiektów interfejsu
    // użytkownika może wyglądać w sposób następujący.
    method createUI() is
        // ...
        copy = function() { executeCommand(
            new CopyCommand(this, activeEditor)) }
        copyButton.setCommand(copy)
        shortcuts.onKeyPress(&quot;Ctrl+C&quot;, copy)

        cut = function() { executeCommand(
            new CutCommand(this, activeEditor)) }
        cutButton.setCommand(cut)
        shortcuts.onKeyPress(&quot;Ctrl+X&quot;, cut)

        paste = function() { executeCommand(
            new PasteCommand(this, activeEditor)) }
        pasteButton.setCommand(paste)
        shortcuts.onKeyPress(&quot;Ctrl+V&quot;, paste)

        undo = function() { executeCommand(
            new UndoCommand(this, activeEditor)) }
        undoButton.setCommand(undo)
        shortcuts.onKeyPress(&quot;Ctrl+Z&quot;, undo)

    // Uruchom polecenie i sprawdź czy trzeba je zapisać w
    // historii.
    method executeCommand(command) is
        if (command.execute())
            history.push(command)

    // Weź najświeższe polecenie z historii i uruchom jego
    // metodę wycofującą. Zwróć uwagę, że nie znamy klasy tego
    // polecenia i nie musimy znać, bo polecenie samo wie jak
    // cofnąć rezultat swojego działania.
    method undo() is
        command = history.pop()
        if (command != null)
            command.undo()</code></pre>
</figure>
:::

:::::::::: {.section .applicability-container}
##  Zastosowanie {#applicability}

::::::::: applicability
::: applicability-problem
Zastosuj wzorzec Polecenie gdy chcesz parametryzować obiekty za pomocą
działań.
:::

::: applicability-solution
Wzorzec Polecenie pozwala przekształcić wywołanie metody w samodzielny
obiekt. Zmiana taka otwiera wiele ciekawych zastosowań: można
przekazywać polecenia jako argumenty metody, przechowywać je w innych
obiektach, zamieniać powiązane polecenia w trakcie działania programu,
itp.

Oto przykład: pracujesz nad komponentem graficznego interfejsu
użytkownika takim jak menu kontekstowe i chcesz aby użytkownicy mogli
konfigurować elementy menu odpowiadające działaniom.
:::

::: applicability-problem
Wzorzec Polecenie pozwala układać kolejki zadań, ustalać harmonogram ich
wykonania bądź uruchamiać je zdalnie.
:::

::: applicability-solution
Jak każdy inny obiekt, polecenie można serializować, co oznacza
przekształcenie go w łańcuch znaków dający się łatwo zapisać w pliku lub
bazie danych. Można później taki łańcuch znaków przywrócić do formy
pierwotnego obiektu polecenia. Dzięki temu można opóźniać i ustalać
harmonogram wykonywania poleceń. Co więcej, w taki sam sposób można
kolejkować, notować w dzienniku lub wysyłać polecenia przez sieć.
:::

::: applicability-problem
Stosuj wzorzec Polecenie gdy chcesz zaimplementować operacje odwracalne.
:::

::: applicability-solution
Chociaż istnieje wiele sposobów na implementację funkcjonalności
cofnij/ponów, wzorzec Polecenie jest prawdopodobnie najpopularniejszym.

Aby móc wycofywać działania, trzeba zaimplementować historię wykonanych
działań. Historia poleceń jest stosem zawierającym wszystkie obiekty
wykonanych poleceń wraz ze skojarzonymi z nimi kopiami zapasowymi stanu
aplikacji.

Ta metoda ma dwie wady. Po pierwsze, zapisanie stanu aplikacji może nie
być tak proste, gdyż część jej danych może być prywatna. Problem ten
można obejść stosując wzorzec [Pamiątka](memento.html).

Po drugie, kopie zapasowe stanów mogą zużywać sporo pamięci RAM. Dlatego
czasem można uciec się do alternatywnej implementacji: zamiast
przywracać przeszły stan, można wykonać polecenie odwrotne. Takie
polecenie również jednak miałoby swoją cenę: może okazać się trudne lub
wręcz niemożliwe do zaimplementowania.
:::
:::::::::
::::::::::

::: {.section .checklist}
##  Jak zaimplementować {#checklist}

1.  Zadeklaruj interfejs polecenia z pojedynczą metodą uruchamiającą.

2.  Dokonaj ekstrakcji żądań do konkretnych, odrębnych klas poleceń
    które implementują interfejs polecenia. Każda klasa powinna mieć
    zestaw pól służących przechowywaniu argumentów żądania wraz z
    odniesieniem do faktycznego obiektu odbiorcy. Wszystkie te wartości
    muszą być inicjalizowane za pośrednictwem konstruktora polecenia.

3.  Zidentyfikuj klasy które będą pełnić rolę *nadawców*. Dodaj tym
    klasom pola służące przechowywaniu poleceń. Nadawcy powinni
    komunikować się z poleceniami wyłącznie za pośrednictwem interfejsu
    polecenia. Nadawcy na ogół nie tworzą obiektów polecenie sami, lecz
    otrzymują je od strony kodu klienta.

4.  Zmień nadawców w taki sposób, aby uruchamiali polecenie zamiast
    wysyłania żądania bezpośrednio do odbiorcy.

5.  Klient powinien inicjalizować obiekty w następującej kolejności:

    -   Tworzyć odbiorców.
    -   Tworzyć polecenia i kojarzyć je z odpowiednimi odbiorcami, jeśli
        istnieje potrzeba,
    -   Tworzyć nadawców i kojarzyć ich z konkretnymi poleceniami.
:::

:::::: {.section .pros-cons}
##  Zalety i wady {#pros-cons}

::::: row
::: col-sm-6
-    *Zasada pojedynczej odpowiedzialności*. Można rozprzęgnąć klasy
    wywołujące polecenia od klas faktycznie je wykonujących.
-    *Zasada otwarte/zamknięte*. Można wprowadzić nowe polecenia do
    aplikacji bez psucia istniejącego kodu klienta.
-    Pozwala zaimplementować cofnij/ponów.
-    Pozwala zaimplementować opóźnione wykonywanie działań.
-    Można złożyć zestaw prostszych poleceń w jedno skomplikowane.
:::

::: col-sm-6
-    Kod może stać się bardziej skomplikowany gdyż wprowadzamy całą nową
    warstwę pomiędzy nadawcami a odbiorcami.
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

-   Obsługujący w [Łańcuchu zobowiązań](chain-of-responsibility.html)
    mogą być zaimplementowani jako [Polecenia](command.html). Można
    wówczas wykonać wiele różnych działań reprezentowanych jako
    żądania na tym samym obiekcie-kontekście.

    Istnieje jednak jeszcze jedno podejście, według którego samo żądanie
    jest obiektem [Polecenie](command.html). W takim przypadku możesz
    wykonać to samo działanie na łańcuchu różnych kontekstów.

-   Można stosować [Polecenie](command.html) i [Pamiątkę](memento.html)
    jednocześnie --- implementując funkcjonalność "cofnij". W takim
    przypadku, polecenia są odpowiedzialne za wykonywanie różnych
    działań na obiekcie docelowym, zaś pamiątki służą zapamiętaniu stanu
    obiektu tuż przed wykonaniem polecenia.

-   [Polecenie](command.html) i [Strategia](strategy.html) mogą wydawać
    się podobne, ponieważ oba mogą służyć parametryzacji obiektu jakimś
    działaniem. Mają jednak inne cele.

    -   Za pomocą *Polecenia* można konwertować dowolne działanie na
        obiekt. Parametry działania stają się polami tego obiektu.
        Konwersja zaś pozwala odroczyć wykonanie działania,
        kolejkować je i przechowywać historię wykonanych działań, a
        także wysyłać polecenia zdalnym usługom, itd.

    -   Z drugiej strony, *Strategia* zazwyczaj opisuje różne sposoby
        wykonywania danej czynności, pozwalając zamieniać algorytmy w
        ramach jednej klasy kontekstu.

-   [Prototyp](prototype.html) może pomóc stworzyć historię, zapisując
    kopie [Poleceń](command.html).

-   Wzorzec [Odwiedzający](visitor.html) można traktować jak
    potężniejszą wersję [Polecenia](command.html). Jego obiekty mogą
    wykonywać różne polecenia na obiektach różnych klas.
:::

::: {.section .implementations}
##  Przykłady kodu {#implementations}

[![Polecenie w języku
C#](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](command/csharp/example.html "Polecenie w języku C#"){.prog-lang-link}
[![Polecenie w języku
C++](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](command/cpp/example.html "Polecenie w języku C++"){.prog-lang-link}
[![Polecenie w języku
Go](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](command/go/example.html "Polecenie w języku Go"){.prog-lang-link}
[![Polecenie w języku
Java](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](command/java/example.html "Polecenie w języku Java"){.prog-lang-link}
[![Polecenie w języku
PHP](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](command/php/example.html "Polecenie w języku PHP"){.prog-lang-link}
[![Polecenie w języku
Python](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](command/python/example.html "Polecenie w języku Python"){.prog-lang-link}
[![Polecenie w języku
Ruby](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](command/ruby/example.html "Polecenie w języku Ruby"){.prog-lang-link}
[![Polecenie w języku
Rust](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](command/rust/example.html "Polecenie w języku Rust"){.prog-lang-link}
[![Polecenie w języku
Swift](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](command/swift/example.html "Polecenie w języku Swift"){.prog-lang-link}
[![Polecenie w języku
TypeScript](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](command/typescript/example.html "Polecenie w języku TypeScript"){.prog-lang-link}
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

[Iterator []{.fa .fa-arrow-right}](iterator.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Wróć

[[]{.fa .fa-arrow-left} Łańcuch
zobowiązań](chain-of-responsibility.html){.btn .btn-default rel="prev"}
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
