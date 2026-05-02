:::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::: {.main-content-container .center-content-container}
:::::: {.page .text}
::: breadcrumb
[](../../index.html){.home} / [Wzorce
projektowe](../design-patterns.html){.type} / [Wzorce
behawioralne](behavioral-patterns.html){.type} /
[Odwiedzający](visitor.html){.type}
:::

# Odwiedzający i podwójna dyspozycja {#odwiedzający-i-podwójna-dyspozycja .title}

Spójrzmy na następującą hierarchię klas figur geometrycznych
(uwaga --- to tylko pseudokod):

<figure class="code">
<pre class="code" lang="pseudocode"><code>interface Graphic is
    method draw()

class Shape implements Graphic is
    field id
    method draw()
    // ...

class Dot extends Shape is
    field x, y
    method draw()
    // ...

class Circle extends Dot is
    field radius
    method draw()
    // ...

class Rectangle extends Shape is
    field width, height
    method draw()
    // ...

class CompoundGraphic implements Graphic is
    field children: array of Graphic
    method draw()
    // ...</code></pre>
</figure>

Kod działa poprawnie i aplikacja przechodzi do etapu produkcji. Pewnego
dnia jednak postanawiasz dodać funkcjonalność eksportowania. Kod
eksportu wyglądałby obco, gdyby znajdował się w klasach figur. Więc
zamiast dodawać go do wszystkich klas tej hierarchii, postanawiasz
stworzyć nową klasę poza tą hierarchią i tam umieścić logikę eksportu.
Klasa taka otrzymałaby metody eksportowania publicznego stanu każdego
obiektu do łańcuchów XML:

<figure class="code">
<pre class="code" lang="pseudocode"><code>class Exporter is
    method export(s: Shape) is
        print(&quot;Eksportuje shape&quot;)
    method export(d: Dot)
        print(&quot;Eksportuje dot&quot;)
    method export(c: Circle)
        print(&quot;Eksportuje circle&quot;)
    method export(r: Rectangle)
        print(&quot;Eksportuje rectangle&quot;)
    method export(cs: CompoundGraphic)
        print(&quot;Eksportuje złożoną&quot;)</code></pre>
</figure>

Kod wygląda w porządku, ale wypróbujmy go:

<figure class="code">
<pre class="code" lang="pseudocode"><code>class App() is
    method export(shape: Shape) is
        Exporter exporter = new Exporter()
        exporter.export(shape);

app.export(new Circle());
// Niestety, wyświetli się komunikat &quot;Exportuje shape&quot;.</code></pre>
</figure>

Zaraz, ale dlaczego?

## Myśląc jak kompilator

Uwaga: poniższa treść dotyczy większości współczesnych języków
programowania zorientowanych obiektowo (Java, C#, PHP i inne).

### Późne/dynamiczne wiązanie

Wyobraź sobie, że jesteś kompilatorem. Musisz zdecydować jak skompilować
poniższy kod:

<figure class="code">
<pre class="code" lang="pseudocode"><code>method drawShape(shape: Shape) is
    shape.draw();</code></pre>
</figure>

Popatrzmy\... metoda `draw` zdefiniowana w klasie `Shape`. Ale zaraz, są
też cztery podklasy nadpisujące tę metodę. Czy możemy bezpiecznie
zdecydować którą implementację wywołać? Nie wydaje się. Jedyny sposób
aby się upewnić to uruchomić program i sprawdzić jaka jest klasa obiektu
przekazanego metodzie. Wiemy tylko tyle, że obiekt **będzie posiadał**
implementację metody `draw`.

Powstały kod maszynowy będzie więc sprawdzał klasę parametru `s` i
wybierał implementację metody `draw` z właściwej klasy.

Takie dynamiczne ustalanie typu nazywa się późnym (lub dynamicznym)
wiązaniem:

-   **Późne**, bo łączy się obiekt z jego implementacją po kompilacji, w
    trakcie pracy programu.
-   **Dynamiczne**, bo każdy nowo powstały obiekt może wymagać
    połączenia z inną implementacją.

### Wczesne/statyczne wiązanie

A teraz "skompilujmy" następujący kod:

<figure class="code">
<pre class="code" lang="pseudocode"><code>method exportShape(shape: Shape) is
    Exporter exporter = new Exporter()
    exporter.export(shape);</code></pre>
</figure>

Wszystko jasne przy drugiej linijce: klasa `Exporter` nie ma
konstruktora, więc jedynie tworzymy instancję obiektu. A co z wywołaniem
`export`? `Exporter` ma pięć metod o takiej samej nazwie, które różnią
się pod względem typu parametru. Który więc wywołać? Wygląda na to, że i
tutaj będziemy potrzebować dynamicznego wiązania.

Ale jest kolejny problem. Co jeśli istnieje klasa figury która nie
posiada odpowiedniej metody `export` w klasie `Exporter`? Na przykład
obiekt `Ellipse`? Kompilator nie jest w stanie zagwarantować, że
istnieje stosowna przeciążona metoda, tak jak jest to możliwe w
przypadku metod nadpisanych. Powstałaby niejasna sytuacja na którą
kompilator nie pozwoli.

Dlatego też twórcy kompilatorów stosują bezpieczniejszą drogę i
korzystają z wczesnego (lub statycznego) wiązania w przypadku
przeciążonych metod:

-   **Wczesne**, bo odbywa się w momencie kompilacji, zanim uruchomi się
    program.
-   **Statyczne**, bo nie da się zmienić w trakcie działania programu.

Wróćmy do naszego przykładu. Wiemy na pewno, że przyjmowany argument
będzie pochodził z hierarchii klas `Shape`: albo sama `Shape`, albo
któraś z jej podklas. Wiemy też, że klasa `Exporter` posiada podstawową
implementację funkcjonalności eksportowania zdolną obsługiwać klasę
`Shape`: `export(s: Shape)`.

Jest to jedyna implementacja, którą można bezpiecznie powiązać z danym
kodem bez wprowadzania niejasności. Dlatego też nawet jeśli przekazujemy
obiekt klasy `Rectangle` metodzie `exportShape`, to eksporter nadal
wywoła metodę `export(s: Shape)`.

## Podwójna dyspozycja

**Podwójna dyspozycja** jest sztuczką pozwalającą zastosować dynamiczne
wiązanie razem z przeciążaniem metod. Oto jak:

<figure class="code">
<pre class="code" lang="pseudocode"><code>class Visitor is
    method visit(s: Shape) is
        print(&quot;Odwiedzono shape&quot;)
    method visit(d: Dot)
        print(&quot;Odwiedzono dot&quot;)

interface Graphic is
    method accept(v: Visitor)

class Shape implements Graphic is
    method accept(v: Visitor)
        // Kompilator wie na pewno, że `this` jest klasy `Shape`.
        // Oznacza to, że można bezpiecznie wywołać `visit(s: Shape)`.
        v.visit(this)

class Dot extends Shape is
    method accept(v: Visitor)
        // Kompilator wie, że `this` jest klasy `Dot`.
        // Oznacza to, że można bezpiecznie wywołać `visit(s: Dot)`.
        v.visit(this)


Visitor v = new Visitor();
Graphic g = new Dot();

// Metoda `accept` została nadpisana, nie przeciążona. Kompilator wiąże ją
// dynamicznie. Dlatego `accept` będzie uruchomiona na klasie która jest
// zgodna z obiektem wywołującym metodę (w naszym przypadku klasa `Dot`).
g.accept(v);

// Wyświetli się: &quot;Odwiedzono dot&quot;</code></pre>
</figure>

### Posłowie

Mimo, że wzorzec [Odwiedzający](visitor.html) stworzono na podstawie
zasady podwójnej dyspozycji, to nie jest to jego główne przeznaczenie.
Odwiedzający pozwala bowiem dodawać "zewnętrzne" operacje całej
hierarchii klas bez zmiany istniejącego kodu tych klas.

::: next
#### Czytaj dalej

[Wzorce projektowe w C# []{.fa .fa-arrow-right}](csharp.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Wróć

[[]{.fa .fa-arrow-left} Odwiedzający](visitor.html){.btn .btn-default
rel="prev"}
:::
::::::
:::::::
::::::::
