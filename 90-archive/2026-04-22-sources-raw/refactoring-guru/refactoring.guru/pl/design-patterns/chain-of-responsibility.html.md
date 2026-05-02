::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [Wzorce
projektowe](../design-patterns.html){.type} / [Wzorce
behawioralne](behavioral-patterns.html){.type}
:::

# Łańcuch zobowiązań {#łańcuch-zobowiązań .title}

::: aka
Znany też jako: [Chain of command, ]{style="display:inline-block"}[Chain
of Responsibility]{style="display:inline-block"}
:::

::: {.section .intent}
##  Cel {#intent}

**Łańcuch zobowiązań** jest behawioralnym wzorcem projektowym, który
pozwala przekazywać żądania wzdłuż łańcucha obiektów obsługujących.
Otrzymawszy żądanie, każdy z obiektów obsługujących decyduje o
przetworzeniu żądania lub przekazaniu go do kolejnego obiektu
obsługującego w łańcuchu.

<figure class="image">
<img
src="../../images/patterns/content/chain-of-responsibility/chain-of-responsibilityf2d1.png?id=56c10d0dc712546cc283cfb3fb463458"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/chain-of-responsibility/chain-of-responsibility.png?id=56c10d0dc712546cc283cfb3fb463458 640w,/images/patterns/content/chain-of-responsibility/chain-of-responsibility-1.5x.png?id=770c03ad168fa797ec8ed4556ddf0b5c 960w,/images/patterns/content/chain-of-responsibility/chain-of-responsibility-2x.png?id=cc104b0a00a410f37fb39da80f392b88 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Wzorzec projektowy Łańcuch zobowiązań" />
</figure>
:::

::: {.section .problem}
##  Problem

Wyobraź sobie, że pracujesz nad systemem zamawiania online. Chcesz
ograniczyć dostęp do systemu, by wyłącznie użytkownicy uwierzytelnieni
mogli składać zamówienia. Ponadto użytkownicy z uprawnieniami
administracyjnymi powinni mieć pełen dostęp do wszystkich zamówień.

Po obmyśleniu planu, zdajesz sobie sprawę, że takie sprawdzenia powinno
się wykonywać sekwencyjnie. Aplikacja może spróbować uwierzytelnić
użytkownika otrzymawszy żądanie zawierające poświadczenia użytkownika.
Jednak jeśli poświadczenia nie są prawidłowe i uwierzytelnienie nie
powiedzie się, nie ma powodu dokonywać dalszych sprawdzeń.

<figure class="image">
<img
src="../../images/patterns/diagrams/chain-of-responsibility/problem1-pl250f.png?id=07cada95379f79004286b24ea50fb5af"
style="aspect-ratio: 2.5;"
srcset="/images/patterns/diagrams/chain-of-responsibility/problem1-pl.png?id=07cada95379f79004286b24ea50fb5af 600w,/images/patterns/diagrams/chain-of-responsibility/problem1-pl-1.5x.png?id=3bcc1bf713dcac79489bdc7fddba6898 900w,/images/patterns/diagrams/chain-of-responsibility/problem1-pl-2x.png?id=f3e7a04ddecfbcbb6952d3628c9da8af 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Rozwiązanie problemu z użyciem wzorca Łańcuch zobowiązań" />
<figcaption><p>Żądanie musi przejść serię sprawdzeń zanim system
zamawiania będzie mógł go obsłużyć.</p></figcaption>
</figure>

W kolejnych miesiącach, implementujesz wiele takich sekwencyjnych
sprawdzeń.

-   Jeden z twoich współpracowników zauważa, że przekazywanie surowych
    danych przez system zamawiania nie jest bezpieczne. Dodajesz więc
    etap walidacyjny, czyszczący dane zawarte w żądaniu.

-   Później ktoś zauważa, że system jest podatny na łamanie haseł metodą
    brute force. Aby się przed tym uchronić, dodajesz sprawdzenie
    odrzucające wielokrotne nieskuteczne próby uwierzytelnienia
    przychodzące z tego samego adresu IP.

-   Ktoś inny zaś zasugerował, że można przyspieszyć działanie systemu,
    gdyby zwracał on przechowane w pamięci podręcznej wyniki żądań
    zawierające te same dane. Dodajesz więc kolejne sprawdzenie,
    pozwalające żądaniu przejść dalej tylko jeśli nie ma już stosownej
    odpowiedzi zapisanej w pamięci podręcznej.

<figure class="image">
<img
src="../../images/patterns/diagrams/chain-of-responsibility/problem2-plf16e.png?id=d9d7b1bd5941ae591016614d44833ac8"
style="aspect-ratio: 1.65;"
srcset="/images/patterns/diagrams/chain-of-responsibility/problem2-pl.png?id=d9d7b1bd5941ae591016614d44833ac8 610w,/images/patterns/diagrams/chain-of-responsibility/problem2-pl-1.5x.png?id=e5e54926812824b461ef7ba07eacf227 915w,/images/patterns/diagrams/chain-of-responsibility/problem2-pl-2x.png?id=fd6778c17a6713a769afe05c0f05ef75 1220w"
sizes="(max-width: 720px) 100vw, 610px" loading="lazy" width="610"
alt="Z dodaniem każdego kolejnego sprawdzenia, kod puchnie i pojawia się bałagan" />
<figcaption><p>Im bardziej kod urósł, tym bardziej stał
się zabałaganiony.</p></figcaption>
</figure>

Kod sprawdzeń, który już na początku wyglądał pogmatwanie, spuchł
jeszcze bardziej wraz z dodawaniem funkcjonalności. Zmiana jednego
sprawdzenia czasem wpływała na inne. A co najgorsze, próba ponownego
użycia sprawdzeń w zabezpieczeniu innych komponentów systemu spowodowała
duplikację części kodu ponieważ niektóre komponenty potrzebowały
sprawdzeń, ale inne nie.

System stał się trudny do zrozumienia i kosztowny w utrzymaniu. Po
okresie trudzenia się postanawiasz dokonać refaktoryzacji całości.
:::

::: {.section .solution}
##  Rozwiązanie {#solution}

Jak wiele innych wzorców behawioralnych, **Łańcuch Zobowiązań** zakłada
przekształcenie pewnych obowiązków w samodzielne obiekty zwane
*obiektami obsługującymi*. W naszym przypadku, każde sprawdzenie powinno
się wyekstrahować do osobnej klasy posiadającej jedną metodę dokonującą
sprawdzenia. Żądanie wraz z towarzyszącymi mu danymi przekazywane jest
jako argument tej metody.

Wzorzec sugeruje połączenie tych obiektów obsługujących w łańcuch. Każdy
obiekt obsługujący stanowiący ogniwo łańcucha posiada pole przechowujące
odniesienie do następnego obiektu w łańcuchu. Poza przetworzeniem
żądania, obiekty przekazują je dalej. Żądanie biegnie wzdłuż
łańcucha, by wszystkie ogniwa miały okazję je obsłużyć.

A co najlepsze, obiekt obsługujący może zdecydować o nieprzekazaniu
żądania dalej i tym samym kończy proces.

W naszym przykładzie systemu zamawiającego, obiekt obsługujący dokonuje
przetwarzania żądania i decyduje o przekazaniu go dalej, lub nie.
Zakładając, że żądanie zawiera właściwe dane, obiekty obsługujące mogą
wykonywać swoje obowiązki, takie jak uwierzytelnianie czy zapis w
pamięci podręcznej.

<figure class="image">
<img
src="../../images/patterns/diagrams/chain-of-responsibility/solution1-ple818.png?id=20c3acd3a63e8e84db4bd58bc4c9869a"
style="aspect-ratio: 4;"
srcset="/images/patterns/diagrams/chain-of-responsibility/solution1-pl.png?id=20c3acd3a63e8e84db4bd58bc4c9869a 640w,/images/patterns/diagrams/chain-of-responsibility/solution1-pl-1.5x.png?id=021607c47777d7bdfe9fb497d66602a5 960w,/images/patterns/diagrams/chain-of-responsibility/solution1-pl-2x.png?id=60d6457762f74a970c3e2787e2018c8a 1280w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="640"
alt="Obiekty obsługujące ułożone są jeden przy drugim, tworząc łańcuch" />
<figcaption><p>Obiekty obsługujące są ułożone jeden obok drugiego,
tworząc łańcuch.</p></figcaption>
</figure>

Istnieje jednak nieco inne podejście (które weszło do kanonu), według
którego obiekt obsługujący otrzymawszy żądanie decyduje czy może je
obsłużyć i jeśli tak, to nie przekazuje go dalej. Więc albo tylko jeden
obiekt obsługuje jedno żądanie, albo żaden. Podejście to jest bardzo
powszechne w przypadku stosu zdarzeń w obrębie graficznego interfejsu
użytkownika.

Na przykład, gdy użytkownik kliknie przycisk, zdarzenie rozpropaguje się
wzdłuż łańcucha elementów UI, zaczynając od przycisku, poprzez jego
kontenery (formatki lub panele) i dociera do głównego okna aplikacji.
Zdarzenie jest przetwarzane przez pierwszy element w łańcuchu który
jest w stanie je obsłużyć. Ten przykład jest też godny uwagi, bo
pokazuje jak z każdego drzewa obiektów można wyekstrahować łańcuch.

<figure class="image">
<img
src="../../images/patterns/diagrams/chain-of-responsibility/solution2-ple625.png?id=87b00868915d08b3f3d36a735c6e35b8"
style="aspect-ratio: 1.73;"
srcset="/images/patterns/diagrams/chain-of-responsibility/solution2-pl.png?id=87b00868915d08b3f3d36a735c6e35b8 520w,/images/patterns/diagrams/chain-of-responsibility/solution2-pl-1.5x.png?id=f927356e09712d324d19b88790c7dc88 780w,/images/patterns/diagrams/chain-of-responsibility/solution2-pl-2x.png?id=1479975664efa644ad9e3356e9153d3a 1040w"
sizes="(max-width: 720px) 100vw, 520px" loading="lazy" width="520"
alt="Z gałęzi drzewa obiektów można uformować łańcuch" />
<figcaption><p>Z gałęzi drzewa obiektów można
uformować łańcuch.</p></figcaption>
</figure>

Istotnym jest, że wszystkie klasy obiektów obsługujących implementują
ten sam interfejs. Każdy konkretny obiekt obsługujący powinien wiedzieć
tylko o następnym, posiadającym metodę `wykonaj`. W ten sposób można
komponować łańcuchy w trakcie działania programu, stosując różne obiekty
obsługujące bez sprzęgania kodu z ich konkretnymi klasami.
:::

::: {.section .analogy}
##  Analogia do prawdziwego życia {#analogy}

<figure class="image">
<img
src="../../images/patterns/content/chain-of-responsibility/chain-of-responsibility-comic-1-plda47.png?id=8bc6439250b8e29d974d05819b6dd16d"
style="aspect-ratio: 2;"
srcset="/images/patterns/content/chain-of-responsibility/chain-of-responsibility-comic-1-pl.png?id=8bc6439250b8e29d974d05819b6dd16d 600w,/images/patterns/content/chain-of-responsibility/chain-of-responsibility-comic-1-pl-1.5x.png?id=7e7cc44acefa373fe9151d9ca6ea54b1 900w,/images/patterns/content/chain-of-responsibility/chain-of-responsibility-comic-1-pl-2x.png?id=e50b1cd2672bcff132648566230e5b1c 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Rozmowa z pomocą techniczną może być trudna" />
<figcaption><p>Połączenie z pomocą techniczną może być przekazane
kolejnym jej pracownikom.</p></figcaption>
</figure>

Właśnie kupiłeś sobie i zainstalowałeś jakąś część do komputera.
Ponieważ jesteś geekiem, na komputerze jest kilka systemów operacyjnych.
Uruchamiasz więc jeden po drugim, sprawdzając czy urządzenie jest
obsługiwane. Windows wykrywa i włącza urządzenie automatycznie. Jednak
twoja ukochana dystrybucja Linuksa odmawia współpracy. W nikłym
przebłysku nadziei, dzwonisz na numer pomocy technicznej podany na
opakowaniu.

Pierwsze, co słyszysz, to sztucznie brzmiący głos automatu
zgłoszeniowego. Sugeruje on dziewięć typowych rozwiązań różnych
problemów, ale żaden z nich nie dotyczy twego przypadku. Po jakimś
czasie, automat poddaje się i łączy cię z żywym człowiekiem.

Ale żywy pracownik również nie jest w stanie zasugerować nic
pożytecznego. Cytuje długie ustępy instrukcji obsługi i nie słucha
twoich uwag. Po usłyszeniu dziesiąty raz sugestii "proszę spróbować
wyłączyć i włączyć ponownie komputer", żądasz połączenia z prawdziwym
inżynierem.

Ostatecznie łączy cię z jednym z inżynierów, który zapewne od wielu
godzin tęskni za rozmową z żywym człowiekiem, siedząc w swojej
odosobnionej serwerowni gdzieś w ciemnej piwnicy. Inżynier podaje ci
link do odpowiednich sterowników do urządzenia i tłumaczy jak je
zainstalować pod Linuksem. Wreszcie --- rozwiązanie! Rozłączasz się
pełen radości.
:::

::::: {.section .structure-container}
##  Struktura {#structure}

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/chain-of-responsibility/structure03c9.png?id=848f0fc8dca57a44974d63f8181f5406"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 0.93;"
srcset="/images/patterns/diagrams/chain-of-responsibility/structure.png?id=848f0fc8dca57a44974d63f8181f5406 380w,/images/patterns/diagrams/chain-of-responsibility/structure-1.5x.png?id=49dfbae38f146af7319f80dce9ea7e2a 570w,/images/patterns/diagrams/chain-of-responsibility/structure-2x.png?id=bb837faaac88e7f2a16f751d0beaa201 760w"
sizes="(max-width: 720px) 100vw, 380px" loading="lazy" width="380"
alt="Struktura wzorca Łańcuch zobowiązań" /><img
src="../../images/patterns/diagrams/chain-of-responsibility/structure-indexedfcb6.png?id=e13a5bf44f9ca47299223116af77cbef"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 0.88;"
srcset="/images/patterns/diagrams/chain-of-responsibility/structure-indexed.png?id=e13a5bf44f9ca47299223116af77cbef 380w,/images/patterns/diagrams/chain-of-responsibility/structure-indexed-1.5x.png?id=ca660e2cb697b512aadc07fdd3e109cd 570w,/images/patterns/diagrams/chain-of-responsibility/structure-indexed-2x.png?id=4f27e2c48e635f45a78472d707a8df3c 760w"
sizes="(max-width: 720px) 100vw, 380px" loading="lazy" width="380"
alt="Struktura wzorca Łańcuch zobowiązań" />
</figure>
:::

1.  **Obiekt Obsługujący** deklaruje wspólny dla wszystkich obiektów
    obsługujących interfejs. Zazwyczaj posiada on tylko jedną metodę do
    obsługi żądań, ale czasem może zawierać też drugą, służącą do
    wybierania kolejnego obiektu w łańcuchu.

2.  **Bazowy Obiekt Obsługujący** to opcjonalna klasa, gdzie można
    umieścić kod przygotowawczy, wspólny dla wszystkich klas
    obsługujących.

    Na ogół klasa ta definiuje pole służące przechowywaniu
    odniesienia do kolejnego obiektu obsługującego. Klienci mogą
    sformować łańcuch przekazując obiekt obsługujący konstruktorowi lub
    metodzie setter poprzedniego obiektu. Klasa może też implementować
    domyślną obsługę: przekazać wykonanie kolejnemu obiektowi
    obsługującemu, sprawdziwszy, czy taki istnieje.

3.  **Konkretne Obiekty Obsługujące** zawierają faktyczny kod służący
    obsłudze żądań. Otrzymawszy żądanie, każdy obiekt musi zdecydować,
    czy je obsłużyć i czy przekazać je dalej.

    Obiekty obsługujące są zazwyczaj samodzielne i niezmienne, akceptują
    wszystkie konieczne dane jednorazowo za pośrednictwem konstruktora.

4.  **Klient** może skomponować łańcuch raz, albo robić to dynamicznie,
    zależnie od logiki aplikacji. Warto pamiętać, że żądanie może być
    przekazane dowolnemu ogniwu łańcucha --- niekoniecznie pierwszemu.
::::
:::::

::: {.section .pseudocode}
##  Pseudokod {#pseudocode}

W poniższym przykładzie, wzorzec **Łańcuch Zobowiązań** jest
odpowiedzialny za wyświetlanie pomocy kontekstowej dotyczącej aktywnych
elementów interfejsu użytkownika.

<figure class="image">
<img
src="../../images/patterns/diagrams/chain-of-responsibility/example-pl09df.png?id=90c80cf0098431ed8ba748f05dd0ee29"
style="aspect-ratio: 1.09;"
srcset="/images/patterns/diagrams/chain-of-responsibility/example-pl.png?id=90c80cf0098431ed8ba748f05dd0ee29 610w,/images/patterns/diagrams/chain-of-responsibility/example-pl-1.5x.png?id=d88dc6653a9df980049e1a6a8d110cfa 915w,/images/patterns/diagrams/chain-of-responsibility/example-pl-2x.png?id=3cc959e7422b121c77f21512f481e5b9 1220w"
sizes="(max-width: 720px) 100vw, 610px" loading="lazy" width="610"
alt="Struktura przykładu użycia Łańcucha zobowiązań" />
<figcaption><p>Klasy interfejsu użytkownika (GUI) zbudowano według
wzorca Kompozyt. Każdy element jest powiązany ze swoim kontenerem. W
dowolnym momencie można stworzyć łańcuch elementów, zaczynający się od
samego elementu i prowadzący przez wszystkie
elementy kontenera.</p></figcaption>
</figure>

Interfejs użytkownika aplikacji zazwyczaj jest ustrukturyzowany w formie
drzewa obiektów. Na przykład klasa `Dialog`, która renderuje główne okno
aplikacji byłaby korzeniem drzewa obiektów. Dialog zawiera `Panele`,
które mogą z kolei zawierać inne panele lub proste niskopoziomowe
elementy jak `Przyciski` i `PolaTekstowe`.

Prosty komponent może pokazywać krótkie kontekstowe podpowiedzi, o
ile ma przypisaną mu jakąś treść pomocy. Ale bardziej złożone komponenty
definiują swoje sposoby na wyświetlanie pomocy kontekstowej, jak
prezentacja stosownego fragmentu instrukcji obsługi lub otwarcie strony
internetowej w przeglądarce.

<figure class="image">
<img
src="../../images/patterns/diagrams/chain-of-responsibility/example2-pl0a1e.png?id=d8739510347a975edf7f33e7f70d3dbd"
style="aspect-ratio: 0.81;"
srcset="/images/patterns/diagrams/chain-of-responsibility/example2-pl.png?id=d8739510347a975edf7f33e7f70d3dbd 250w,/images/patterns/diagrams/chain-of-responsibility/example2-pl-1.5x.png?id=f14e561e2c4355e838056be64f01db8f 375w,/images/patterns/diagrams/chain-of-responsibility/example2-pl-2x.png?id=1c6b8af5f660b61886892ab9385a9004 500w"
sizes="(max-width: 720px) 100vw, 250px" loading="lazy" width="250"
alt="Struktura użycia wzorca Łańcuch zobowiązań" />
<figcaption><p>Sposób w jaki żądanie pomocy przechodzi przez kolejne
obiekty GUI.</p></figcaption>
</figure>

Gdy użytkownik wskaże kursorem jakiś element i wciśnie klawisz `F1`,
aplikacja sprawdza jaki komponent znajduje się pod kursorem i wysyła mu
żądanie wyświetlenia pomocy. Żądanie przechodzi przez wszystkie
kontenery elementu, aż dotrze do tego, który jest w stanie obsłużyć
żądanie.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// Interfejs obiektu obsługującego deklaruje metodę wykonującą
// żądanie.
interface ComponentWithContextualHelp is
    method showHelp()


// Klasa bazowa prostych komponentów.
abstract class Component implements ComponentWithContextualHelp is
    field tooltipText: string

    // Kontener komponentu pełni rolę kolejnego ogniwa łańcucha
    // obiektów obsługujących żądanie.
    protected field container: Container

    // Komponent pokazuje podpowiedź jeśli przypisano mu jakiś
    // tekst pomocy. W przeciwnym razie przekazuje wywołanie
    // kontenerowi, o ile takowy istnieje.
    method showHelp() is
        if (tooltipText != null)
            // Pokaż podpowiedź.
        else
            container.showHelp()


// Kontenery mogą zawierać zarówno proste komponenty, jak i inne
// kontenery podrzędne. Tu ustala się relacje łańcucha. Klasa
// dziedziczy zachowanie showHelp od klasy-rodzica.
abstract class Container extends Component is
    protected field children: array of Component

    method add(child) is
        children.add(child)
        child.container = this


// Prymitywne komponenty mogą posiadać tylko domyślną
// implementację funkcji pomoc...
class Button extends Component is
    // ...

// Ale złożone komponenty mogą nadpisywać domyślną
// implementację. Jeśli nie ma innej treści pomocy, komponent
// może zawsze wywołać implementację z klasy bazowej (patrz
// klasa Component).
class Panel extends Container is
    field modalHelpText: string

    method showHelp() is
        if (modalHelpText != null)
            // Wyświetl modalne okno dialogowe z treścią pomocy.
        else
            super.showHelp()

// ...jak wyżej...
class Dialog extends Container is
    field wikiPageURL: string

    method showHelp() is
        if (wikiPageURL != null)
            // Otwórz stronę z pomocą na wiki.
        else
            super.showHelp()


// Kod klienta.
class Application is
    // Każda aplikacja inaczej konfiguruje łańcuch.
    method createUI() is
        dialog = new Dialog(&quot;Budget Reports&quot;)
        dialog.wikiPageURL = &quot;http://...&quot;
        panel = new Panel(0, 0, 400, 800)
        panel.modalHelpText = &quot;This panel does...&quot;
        ok = new Button(250, 760, 50, 20, &quot;OK&quot;)
        ok.tooltipText = &quot;This is an OK button that...&quot;
        cancel = new Button(320, 760, 50, 20, &quot;Cancel&quot;)
        // ...
        panel.add(ok)
        panel.add(cancel)
        dialog.add(panel)

    // Wyobraź sobie co tu się może dziać.
    method onF1KeyPress() is
        component = this.getComponentAtMouseCoords()
        component.showHelp()</code></pre>
</figure>
:::

:::::::::: {.section .applicability-container}
##  Zastosowanie {#applicability}

::::::::: applicability
::: applicability-problem
Stosuj wzorzec Łańcuch zobowiązań gdy twój program ma obsługiwać różne
rodzaje żądań na różne sposoby, ale dokładne typy żądań i ich sekwencji
nie są wcześniej znane.
:::

::: applicability-solution
Wzorzec pozwala połączyć wiele obiektów obsługujących w jeden łańcuch i
otrzymawszy żądanie "odpytać" każde ogniwo czy jest w stanie je
obsłużyć. W ten sposób wszystkie obiekty obsługujące mają okazję
przetworzyć żądanie.
:::

::: applicability-problem
Stosuj ten wzorzec gdy istotne jest uruchomienie wielu obiektów
obsługujących w pewnej kolejności.
:::

::: applicability-solution
Skoro można połączyć obiekty obsługujące w dowolnej kolejności,
wszystkie żądania przejdą przez łańcuch w takim porządku, jaki
zaplanowano.
:::

::: applicability-problem
Łańcuch zobowiązań pozwala ustawić obiekty obsługujące i ich kolejność w
czasie działania programu.
:::

::: applicability-solution
Jeśli eksponujesz w klasie obsługującej metodę setter, ustawiające pole
przechowujące odniesienie, będzie można wstawiać, usuwać lub zmieniać
kolejność ogniw łańcucha dynamicznie.
:::
:::::::::
::::::::::

::: {.section .checklist}
##  Jak zaimplementować {#checklist}

1.  Zadeklaruj interfejs obiektu obsługującego i opisz sygnaturę metody
    obsługującej żądania.

    Zdecyduj jak klient będzie przekazywał dane żądań do metody.
    Najbardziej elastycznym sposobem jest konwersja żądania na obiekt i
    przekazywanie go metodzie obsługującej w charakterze argumentu.

2.  Aby wyeliminować powtarzający się kod przygotowawczy w konkretnych
    obiektach obsługujących, być może warto utworzyć abstrakcyjną bazową
    klasę obiektu obsługującego, wywodzącą się z interfejsu obiektu
    obsługującego.

    Klasa taka powinna zawierać pole przechowujące odniesienie do
    kolejnego ogniwa łańcucha. Rozważ uczynienie tej klasy
    niezmienialną. Jednak jeśli planujesz modyfikować łańcuch w czasie
    działania programu, musisz też zdefiniować setter zmieniający
    wartość pola z odniesieniem.

    Można także zaimplementować wygodne domyślne zachowanie metody
    obsługującej, która przekieruje żądanie do kolejnego obiektu o ile
    takowy istnieje. Konkretne obiekty obsługujące będą w stanie
    skorzystać z tego zachowania wywołując metodę nadklasy.

3.  Jeden po drugim twórz podklasy obiektów obsługujących i
    zaimplementuj im metody obsługujące. Każdy obiekt obsługujący
    powinien podjąć dwie decyzję otrzymawszy żądanie:

    -   Czy przetworzyć żądanie.
    -   Czy przekazać żądanie dalej wzdłuż łańcucha.

4.  Klient może albo złożyć łańcuch samodzielnie, albo otrzymać
    wcześniej przygotowany od innego obiektu. W drugim przypadku, trzeba
    zaimplementować jakieś klasy fabryczne do budowy łańcuchów zgodnie z
    konfiguracją lub ustawieniami środowiska.

5.  Klient może uruchomić kolejny obiekt w łańcuchu, niekoniecznie
    pierwszy. Żądanie zostanie przekazane dalej wzdłuż łańcucha, aż
    jakiś obiekt obsługujący odmówi przekazania go dalej, albo nie
    będzie już komu je przekazać.

6.  W związku z dynamiczną naturą łańcucha, klient powinien być gotów
    obsłużyć następujące scenariusze:

    -   Łańcuch zawiera tylko jedno ogniwo.
    -   Niektóre żądania mogą nie dotrzeć do końca łańcucha.
    -   Inne żądania mogą dotrzeć do końca łańcucha i nie zostać
        obsłużone.
:::

:::::: {.section .pros-cons}
##  Zalety i wady {#pros-cons}

::::: row
::: col-sm-6
-    Można ustalać porządek obsługi żądania.
-    *Zasada pojedynczej odpowiedzialności*. Można rozprzęgnąć klasy
    wywołujące działania klas od klas wykonujących działania.
-    *Zasada otwarte/zamknięte*. Można wprowadzać do programu nowe
    obiekty obsługujące bez psucia istniejącego kodu klienta.
:::

::: col-sm-6
-    Niektóre żądania mogą wcale nie zostać obsłużone.
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

-   [Łańcuch zobowiązań](chain-of-responsibility.html) często stosuje
    się w połączeniu z [Kompozytem](composite.html). W takim przypadku,
    gdy komponent-liść otrzymuje żądanie, może je przekazać poprzez
    łańcuch nadrzędnych komponentów aż do korzenia drzewa obiektów.

-   Obsługujący w [Łańcuchu zobowiązań](chain-of-responsibility.html)
    mogą być zaimplementowani jako [Polecenia](command.html). Można
    wówczas wykonać wiele różnych działań reprezentowanych jako
    żądania na tym samym obiekcie-kontekście.

    Istnieje jednak jeszcze jedno podejście, według którego samo żądanie
    jest obiektem [Polecenie](command.html). W takim przypadku możesz
    wykonać to samo działanie na łańcuchu różnych kontekstów.

-   [Łańcuch zobowiązań](chain-of-responsibility.html) i
    [Dekorator](decorator.html) mają bardzo podobne struktury klas. Oba
    wzorce bazują na rekursywnej kompozycji w celu przekazania obowiązku
    wykonania przez ciąg obiektów. Istnieją jednak kluczowe różnice.

    Obsługujący *Łańcucha zobowiązań* mogą wykonywać działania
    niezależnie od siebie. Mogą również zatrzymać dalsze przekazywanie
    żądania na dowolnym etapie. Z drugiej strony, różne *Dekoratory*
    mogą rozszerzać obowiązki obiektu zachowując zgodność z interfejsem
    bazowym. Dodatkowo, dekoratory nie mają możliwości przerwania
    przepływu żądania.
:::

::: {.section .implementations}
##  Przykłady kodu {#implementations}

[![Łańcuch zobowiązań w języku
C#](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](chain-of-responsibility/csharp/example.html "Łańcuch zobowiązań w języku C#"){.prog-lang-link}
[![Łańcuch zobowiązań w języku
C++](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](chain-of-responsibility/cpp/example.html "Łańcuch zobowiązań w języku C++"){.prog-lang-link}
[![Łańcuch zobowiązań w języku
Go](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](chain-of-responsibility/go/example.html "Łańcuch zobowiązań w języku Go"){.prog-lang-link}
[![Łańcuch zobowiązań w języku
Java](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](chain-of-responsibility/java/example.html "Łańcuch zobowiązań w języku Java"){.prog-lang-link}
[![Łańcuch zobowiązań w języku
PHP](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](chain-of-responsibility/php/example.html "Łańcuch zobowiązań w języku PHP"){.prog-lang-link}
[![Łańcuch zobowiązań w języku
Python](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](chain-of-responsibility/python/example.html "Łańcuch zobowiązań w języku Python"){.prog-lang-link}
[![Łańcuch zobowiązań w języku
Ruby](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](chain-of-responsibility/ruby/example.html "Łańcuch zobowiązań w języku Ruby"){.prog-lang-link}
[![Łańcuch zobowiązań w języku
Rust](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](chain-of-responsibility/rust/example.html "Łańcuch zobowiązań w języku Rust"){.prog-lang-link}
[![Łańcuch zobowiązań w języku
Swift](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](chain-of-responsibility/swift/example.html "Łańcuch zobowiązań w języku Swift"){.prog-lang-link}
[![Łańcuch zobowiązań w języku
TypeScript](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](chain-of-responsibility/typescript/example.html "Łańcuch zobowiązań w języku TypeScript"){.prog-lang-link}
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

[Polecenie []{.fa .fa-arrow-right}](command.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Wróć

[[]{.fa .fa-arrow-left} Wzorce
behawioralne](behavioral-patterns.html){.btn .btn-default rel="prev"}
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
