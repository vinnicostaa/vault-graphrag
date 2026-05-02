:::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::: {.main-content-container .center-content-container}
:::::: {.page .text}
::: breadcrumb
[](../../index.html){.home} / [Wzorce
projektowe](../design-patterns.html){.type} / [Wzorce
kreacyjne](creational-patterns.html){.type}
:::

# Porównanie Fabryk {#porównanie-fabryk .title}

Niniejszy artykuł opisuje różnice pomiędzy pojęciami:

1.  Fabryka
2.  Metoda kreacyjna
3.  Statyczna metoda kreacyjna (lub wytwórcza)
4.  Prosta fabryka
5.  [Metoda wytwórcza](factory-method.html)
6.  [Fabryka abstrakcyjna](abstract-factory.html)

Odniesienia do powyższych pojęć można z łatwością znaleźć w sieci. Wiele
osób nie zdaje sobie sprawy z tego, że chociaż wydają się podobne, to
mają różne znaczenia, a to prowadzi do dezorientacji i nieporozumień.

Spróbujmy więc zrozumieć te różnice i raz na zawsze rozwiązać ten
problem.

## 1. Fabryka

**Fabryka** to niejednoznaczny termin mogący oznaczać funkcję, metodę
lub klasę która ma za zadanie coś wytwarzać. Najczęściej fabryki tworzą
obiekty, ale mogą też produkować pliki, wpisy bazodanowe, itd.

Przykładowo, pojęcie "fabryka" może być luźno użyte w stosunku do:

-   funkcji lub metody która tworzy graficzny interfejs użytkownika
    (GUI) programu;
-   klasy która tworzy użytkowników;
-   statycznej metody wywołującej konstruktor klasy w określony sposób;
-   jednego z kreacyjnych wzorców projektowych.

Zazwyczaj, gdy ktoś używa słowa "fabryka", konkretnego znaczenia trzeba
się domyślić z kontekstu. Ale jeśli masz wątpliwości --- lepiej zapytaj.
Możliwe bowiem, że autor jest\... ignorantem.

## 2. Metoda kreacyjna

Metoda kreacyjna zdefiniowana jest w książce [Refaktoryzacja do wzorców
projektowych](https://www.amazon.com/-/dp/0321213351) jako "metoda która
kreuje obiekty". To oznacza, że każdy rezultat zastosowania wzorca
metody wytwórczej jest "metodą kreacyjną", ale niekoniecznie odwrotnie.
Oznacza to też, że pojęcia "metoda kreacyjna", "metoda wytwórcza" (w
interpretacji Martina Fowlera ---
[Refactoring](https://www.amazon.com/-/dp/0201485672) oraz "statyczna
metoda wytwórcza" (jak opisuje ją Joshua Bloch w [Effective
Java](https://www.amazon.com/-/dp/0134685997) można stosować zamiennie.

W rzeczywistości, metoda kreacyjna stanowi po prostu opakowanie
wywołania konstruktora. Nazwa jedynie lepiej oddaje intencje. Z drugiej
strony, może warto odizolować swój kod od zmian w konstruktorze. Mógłby
wtedy nawet zawierać logikę zwracającą istniejące obiekty, zamiast
tworzyć nowe.

Wiele osób nazwałoby takie metody "metodami wytwórczymi" tylko
dlatego, że produkują nowe obiekty. Logika jest prosta: metoda tworzy
obiekty i skoro wszystkie *fabryki* tworzą obiekty, to metoda jest
*metodą wytwórczą*. To oczywiście powoduje zamieszanie gdy w grę wchodzi
prawdziwy wzorzec [Metody wytwórczej](factory-method.html).

W poniższym przykładzie, `next` jest metodą kreacyjną:

<figure class="code">
<pre class="code" lang="php"><code>class Number {
    private $value;

    public function __construct($value) {
        $this-&gt;value = $value;
    }

    public function next() {
        return new Number ($this-&gt;value + 1);
    }
}</code></pre>
</figure>

## 3. Statyczna metoda kreacyjna

**Statyczna metoda kreacyjna** to\... metoda kreacyjna zadeklarowana
jako `statyczna`. Innymi słowy, można ją wywołać z klasy bez
konieczności tworzenia instancji tej klasy.

Nie zdziw się, gdy ktoś nazwie tego typu metodę "statyczną metodą
wytwórczą". To po prostu zły nawyk. [Metoda
wytwórcza](factory-method.html) jest wzorcem projektowym opartym na
dziedziczeniu. Jeśli uczynisz metodę `statyczną`, nie będzie się dało
rozszerzyć jej w podklasach, a to sprzeczne z sensem tego wzorca.

Gdy statyczna metoda kreacyjna zwraca nowe obiekty, staje się
alternatywnym konstruktorem.

Może to być korzystne gdy:

-   Chcesz mieć wiele różnych konstruktorów, które różnią się
    zastosowaniem, ale mają jednakowe sygnatury. Na przykład nie da się
    mieć jednocześnie `Losowy(int max)` i `Losowy(int min)` w Java, C++,
    C# i wielu innych językach programowania. Najpopularniejszym
    sposobem na obejście tego ograniczenia jest stworzenie wielu
    statycznych metod wywołujących domyślny konstruktor i następnie
    nadających stosowne wartości.

-   Chcesz ponownie wykorzystać istniejące obiekty, zamiast tworzyć nowe
    instancje (patrz wzorzec [Singleton](singleton.html)).
    Konstruktory w większości języków programowania muszą zwracać nowe
    instancje klas. Statyczna metoda kreacyjna pozwala zrealizować
    ponowne użycie istniejącej instancji. Wewnątrz metody statycznej
    twój kod może zdecydować o stworzeniu świeżej instancji wywołując
    konstruktor lub zwrócić istniejący obiekt z jakiejś pamięci
    podręcznej.

W poniższym przykładzie, metoda `load` jest statyczną metodą kreacyjną.
Daje wygodny sposób na pobranie danych użytkownika z bazy danych.

<figure class="code">
<pre class="code" lang="php"><code>class User {
    private $id, $name, $email, $phone;

    public function __construct($id, $name, $email, $phone) {
        $this-&gt;id = $id;
        $this-&gt;name = $name;
        $this-&gt;email = $email;
        $this-&gt;phone = $phone;
    }

    public static function load($id) {
        list($id, $name, $email, $phone) = DB::load_data(&#39;users&#39;, &#39;id&#39;, &#39;name&#39;, &#39;email&#39;, &#39;phone&#39;);
        $user = new User($id, $name, $email, $phone);
        return $user;
    }
}</code></pre>
</figure>

## 4. Wzorzec *Fabryka prosta*

Wzorzec **Fabryka prosta** []{.pop-i}[Zdefiniowany w książce [Head First
Design
Patterns](http://shop.oreilly.com/product/9780596007126.do).]{.pop-content}
przedstawia klasę posiadającą jedną metodę kreacyjną z obszerną
instrukcją warunkową, która na podstawie parametrów metody decyduje
jakiej klasy produktu instancję stworzyć i zwrócić.

Ludzie często mylą *proste fabryki* z ogólnymi *fabrykami* lub którymś z
kreacyjnych wzorców projektowych. W większości przypadków, prosta
fabryka jest etapem pośrednim przy wprowadzaniu wzorców [Metody
wytwórczej](factory-method.html) lub [Fabryki
abstrakcyjnej](abstract-factory.html).

Proste fabryki zazwyczaj nie mają podklas, ale po wyekstrahowaniu
podklas z prostej fabryki zaczynają przypominać klasyczne przykłady
wzorca *metody wytwórczej*.

Przy okazji, jeśli zadeklarujesz prostą fabrykę jako `abstract`, nie
uczyni to z niej automatycznie *fabryki abstrakcyjnej*.

Oto przykład *prostej fabryki*:

<figure class="code">
<pre class="code" lang="php"><code>class UserFactory {
    public static function create($type) {
        switch ($type) {
            case &#39;user&#39;: return new User();
            case &#39;customer&#39;: return new Customer();
            case &#39;admin&#39;: return new Admin();
            default:
                throw new Exception(&#39;Wrong user type passed.&#39;);
        }
    }
}</code></pre>
</figure>

## 5. Wzorzec *Metoda wytwórcza*

**Metoda wytwórcza** []{.pop-i}[Zdefiniowana w książce "Gangu Czworga"
[Wzorce projektowe. Elementy oprogramowania obiektowego wielokrotnego
użytku](https://helion.pl/ksiazki/wzorce-projektowe-elementy-oprogramowania-obiektowego-wielokrotnego-uzytku-erich-gamma-richard-helm-ralph-johnson-john-vli,wzoelv.htm).]{.pop-content}
jest kreacyjnym wzorcem projektowym zakładającym dodanie interfejsu do
tworzenia obiektów, ale pozwala podklasom zmieniać typ tworzonych
obiektów.

Jeśli masz w klasie bazowej metodę kreacyjną i podklasy, które ją
następnie rozszerzają, to być może masz już --- właśnie --- przykład
Metody wytwórczej.

<figure class="code">
<pre class="code" lang="php"><code>abstract class Department {
    public abstract function createEmployee($id);

    public function fire($id) {
        $employee = $this-&gt;createEmployee($id);
        $employee-&gt;paySalary();
        $employee-&gt;dismiss();
    }
}

class ITDepartment extends Department {
    public function createEmployee($id) {
        return new Programmer($id);
    }
}

class AccountingDepartment extends Department {
    public function createEmployee($id) {
        return new Accountant($id);
    }
}</code></pre>
</figure>

## 6. Wzorzec *Fabryka abstrakcyjna*

**Fabryka abstrakcyjna** []{.pop-i}[Także zdefiniowana w [powyższej
książce](https://helion.pl/ksiazki/wzorce-projektowe-elementy-oprogramowania-obiektowego-wielokrotnego-uzytku-erich-gamma-richard-helm-ralph-johnson-john-vli,wzoelv.htm).]{.pop-content} to
kreacyjny wzorzec projektowy umożliwiający produkcję rodzin
spokrewnionych lub współzależnych obiektów bez określania ich
konkretnych klas.

O jakie "pokrewieństwo" chodzi? Spójrzmy na przykład na zestaw klas:
`Transport` + `Silnik` + `Sterowanie`. Może istnieć wiele wariantów:

1.  `Samochód` + `SilnikSpalinowy` + `Kierownica`
2.  `Samolot` + `SilnikOdrzutowy` + `DrążekSterowy`

Jeśli jednak twój projekt nie przewiduje rodzin produktów, to nie
potrzebujesz fabryki abstrakcyjnej.

Jeszcze raz - wiele osób myli wzorzec *fabryka abstrakcyjna* z prostą
klasą fabryczną zadeklarowaną jako `abstract`. Strzeż się!

### Posłowie

Teraz gdy znasz już różnice, spójrz raz jeszcze na wzorce projektowe:

-   [Metoda wytwórcza](factory-method.html)
-   [Fabryka abstrakcyjna](abstract-factory.html)

::: next
#### Czytaj dalej

[Budowniczy []{.fa .fa-arrow-right}](builder.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Wróć

[[]{.fa .fa-arrow-left} Fabryka
abstrakcyjna](abstract-factory.html){.btn .btn-default rel="prev"}
:::
::::::
:::::::
::::::::
