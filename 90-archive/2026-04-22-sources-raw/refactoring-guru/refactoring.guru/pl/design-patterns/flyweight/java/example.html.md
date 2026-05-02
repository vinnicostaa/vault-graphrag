::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Wzorce
projektowe](../../../design-patterns.html){.type} /
[Pyłek](../../flyweight.html){.type} / [Java](../../java.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Pyłek](../../../../images/patterns/cards/flyweight-minie3d8.png?id=422ca8d2f90614dce810a8812c626698){srcset="/images/patterns/cards/flyweight-mini-2x.png?id=857ca676fff84317bb0dab76abfce08e 2x"}
:::

# **Pyłek** w języku Java {#pyłek-w-języku-java .pattern-example-title-block-title}
::::

:::::::::::::::::::::: pattern-example-body
::: pattern-example-brief
**Pyłek** to strukturalny wzorzec projektowy umożliwiający obsługę
wielkich ilości obiektów przy jednoczesnej oszczędności pamięci.

Wzorzec Pyłek umożliwia zmniejszenie wymogów w zakresie pamięci RAM
poprzez współdzielenie części opisu stanu przez wiele obiektów. Innymi
słowy Pyłek przechowuje w pamięci podręcznej te dane, które są wspólne
dla wielu różnych obiektów.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Dowiedz się więcej o Pyłek ](../../flyweight.html){.btn .btn-lg
.btn-primary}
:::

::::::::::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
Nawigacja
:::

::: en-intro
 [Intro](#)
:::

::: en-intro
 [Renderowanie lasu](#example-0)
:::

::: en-file
 trees
:::

:::: en-inside
::: en-file
  [Tree](#example-0--trees-Tree-java)
:::
::::

:::: en-inside
::: en-file
  [Tree­Type](#example-0--trees-TreeType-java)
:::
::::

:::: en-inside
::: en-file
  [Tree­Factory](#example-0--trees-TreeFactory-java)
:::
::::

::: en-file
 forest
:::

:::: en-inside
::: en-file
  [Forest](#example-0--forest-Forest-java)
:::
::::

::: en-file
 [Demo](#example-0--Demo-java)
:::

::: en-file
 [Output­Demo](#example-0--OutputDemo-png)
:::

::: en-file
 [Output­Demo](#example-0--OutputDemo-txt)
:::
:::::::::::::::::::

**Złożoność:** []{.fa-stars}

**Popularność:** []{.fa-stars}

**Przykłady użycia:** Wzorzec Pyłek ma jedno zastosowanie: minimalizacja
zużycia pamięci. Możesz póki co zignorować ten wzorzec jeśli twój
program nie cierpi na niedostatek pamięci.

Przykłady użycia wzorca Pyłek w głównych bibliotekach Java:

-   [`java.lang.Integer#valueOf(int)`](http://docs.oracle.com/javase/8/docs/api/java/lang/Integer.html#valueOf-int-)
    (oraz
    [`Boolean`](http://docs.oracle.com/javase/8/docs/api/java/lang/Boolean.html#valueOf-boolean-),
    [`Byte`](http://docs.oracle.com/javase/8/docs/api/java/lang/Byte.html#valueOf-byte-),
    [`Character`](http://docs.oracle.com/javase/8/docs/api/java/lang/Character.html#valueOf-char-),
    [`Short`](http://docs.oracle.com/javase/8/docs/api/java/lang/Short.html#valueOf-short-),
    [`Long`](http://docs.oracle.com/javase/8/docs/api/java/lang/Long.html#valueOf-long-)
    oraz
    [`BigDecimal`](https://docs.oracle.com/javase/8/docs/api/java/math/BigDecimal.html#valueOf-long-int-))

**Identyfikacja:** Pyłek można poznać po obecności metody kreacyjnej
zwracającej obiekty z pamięci podręcznej, zamiast nowo utworzonych.
::::::::::::::::::::::

[]{#example-0}

## Renderowanie lasu {#example-0-title}

W poniższym przykładzie renderujemy las (1 000 000 drzew)! Każde z drzew
będzie reprezentowane przez jeden obiekt posiadający pewien stan
(współrzędne, tekstura, itd.). Mimo że program wykonuje swoje
zadanie, to zużywa przy tym mnóstwo pamięci RAM.

Przyczyna jest prosta: zbyt wiele obiektów-drzew zawiera identyczne dane
(nazwa, tekstura, barwa). Możemy więc zastosować wzorzec Pyłek by
przenieść te elementy stanu do osobnych obiektów-pyłków (klasa
`TreeType`). Dzięki temu nie musimy przechowywać tych samych danych w
tysiącach obiektów `Tree`, a jedynie odniesienie do odpowiedniego
obiektu-pyłka zawierającego dany zestaw wartości.

Z punktu widzenia kodu klienta nie będzie widać zmiany, bo mechanizm
odpowiadający za ponowne wykorzystanie pyłków jest umieszczony w fabryce
pyłków.

### []{#example-0--trees .anchor} **trees** {#trees .example-title}

#### []{#example-0--trees-Tree-java .anchor} **trees/Tree.java:** Zawiera unikalny stan każdego drzewa

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.flyweight.example.trees;

import java.awt.*;

public class Tree {
    private int x;
    private int y;
    private TreeType type;

    public Tree(int x, int y, TreeType type) {
        this.x = x;
        this.y = y;
        this.type = type;
    }

    public void draw(Graphics g) {
        type.draw(g, x, y);
    }
}</code></pre>
</figure>

#### []{#example-0--trees-TreeType-java .anchor} **trees/TreeType.java:** Zawiera stan współdzielony przez wiele drzew

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.flyweight.example.trees;

import java.awt.*;

public class TreeType {
    private String name;
    private Color color;
    private String otherTreeData;

    public TreeType(String name, Color color, String otherTreeData) {
        this.name = name;
        this.color = color;
        this.otherTreeData = otherTreeData;
    }

    public void draw(Graphics g, int x, int y) {
        g.setColor(Color.BLACK);
        g.fillRect(x - 1, y, 3, 5);
        g.setColor(color);
        g.fillOval(x - 5, y - 10, 10, 10);
    }
}</code></pre>
</figure>

#### []{#example-0--trees-TreeFactory-java .anchor} **trees/TreeFactory.java:** Hermetyzuje złożoność tworzenia pyłków

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.flyweight.example.trees;

import java.awt.*;
import java.util.HashMap;
import java.util.Map;

public class TreeFactory {
    static Map&lt;String, TreeType&gt; treeTypes = new HashMap&lt;&gt;();

    public static TreeType getTreeType(String name, Color color, String otherTreeData) {
        TreeType result = treeTypes.get(name);
        if (result == null) {
            result = new TreeType(name, color, otherTreeData);
            treeTypes.put(name, result);
        }
        return result;
    }
}</code></pre>
</figure>

### []{#example-0--forest .anchor} **forest** {#forest .example-title}

#### []{#example-0--forest-Forest-java .anchor} **forest/Forest.java:** Las który rysujemy

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.flyweight.example.forest;

import refactoring_guru.flyweight.example.trees.Tree;
import refactoring_guru.flyweight.example.trees.TreeFactory;
import refactoring_guru.flyweight.example.trees.TreeType;

import javax.swing.*;
import java.awt.*;
import java.util.ArrayList;
import java.util.List;

public class Forest extends JFrame {
    private List&lt;Tree&gt; trees = new ArrayList&lt;&gt;();

    public void plantTree(int x, int y, String name, Color color, String otherTreeData) {
        TreeType type = TreeFactory.getTreeType(name, color, otherTreeData);
        Tree tree = new Tree(x, y, type);
        trees.add(tree);
    }

    @Override
    public void paint(Graphics graphics) {
        for (Tree tree : trees) {
            tree.draw(graphics);
        }
    }
}</code></pre>
</figure>

#### []{#example-0--Demo-java .anchor} **Demo.java:** Kod klienta

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.flyweight.example;

import refactoring_guru.flyweight.example.forest.Forest;

import java.awt.*;

public class Demo {
    static int CANVAS_SIZE = 500;
    static int TREES_TO_DRAW = 1000000;
    static int TREE_TYPES = 2;

    public static void main(String[] args) {
        Forest forest = new Forest();
        for (int i = 0; i &lt; Math.floor(TREES_TO_DRAW / TREE_TYPES); i++) {
            forest.plantTree(random(0, CANVAS_SIZE), random(0, CANVAS_SIZE),
                    &quot;Summer Oak&quot;, Color.GREEN, &quot;Oak texture stub&quot;);
            forest.plantTree(random(0, CANVAS_SIZE), random(0, CANVAS_SIZE),
                    &quot;Autumn Oak&quot;, Color.ORANGE, &quot;Autumn Oak texture stub&quot;);
        }
        forest.setSize(CANVAS_SIZE, CANVAS_SIZE);
        forest.setVisible(true);

        System.out.println(TREES_TO_DRAW + &quot; trees drawn&quot;);
        System.out.println(&quot;---------------------&quot;);
        System.out.println(&quot;Memory usage:&quot;);
        System.out.println(&quot;Tree size (8 bytes) * &quot; + TREES_TO_DRAW);
        System.out.println(&quot;+ TreeTypes size (~30 bytes) * &quot; + TREE_TYPES + &quot;&quot;);
        System.out.println(&quot;---------------------&quot;);
        System.out.println(&quot;Total: &quot; + ((TREES_TO_DRAW * 8 + TREE_TYPES * 30) / 1024 / 1024) +
                &quot;MB (instead of &quot; + ((TREES_TO_DRAW * 38) / 1024 / 1024) + &quot;MB)&quot;);
    }

    private static int random(int min, int max) {
        return min + (int) (Math.random() * ((max - min) + 1));
    }
}</code></pre>
</figure>

#### []{#example-0--OutputDemo-png .anchor} **OutputDemo.png:** Zrzut ekranu

![](../../../../images/patterns/examples/java/flyweight/OutputDemo5583.png?id=8bcaaf279411a0b13b42e09fdf38894e)

#### []{#example-0--OutputDemo-txt .anchor} **OutputDemo.txt:** Statystyki zużycia RAM

<figure class="code">
<pre class="code" lang="output"><code>1000000 trees drawn
---------------------
Memory usage:
Tree size (8 bytes) * 1000000
+ TreeTypes size (~30 bytes) * 2
---------------------
Total: 7MB (instead of 36MB)</code></pre>
</figure>

::: next
#### Czytaj dalej

[Pełnomocnik w języku Java []{.fa
.fa-arrow-right}](../../proxy/java/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Wróć

[[]{.fa .fa-arrow-left} Fasada w języku
Java](../../facade/java/example.html){.btn .btn-default rel="prev"}
:::

## **Pyłek** w innych językach

[![Pyłek w języku
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Pyłek w języku C#"){.prog-lang-link}
[![Pyłek w języku
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Pyłek w języku C++"){.prog-lang-link}
[![Pyłek w języku
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Pyłek w języku Go"){.prog-lang-link}
[![Pyłek w języku
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Pyłek w języku PHP"){.prog-lang-link}
[![Pyłek w języku
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Pyłek w języku Python"){.prog-lang-link}
[![Pyłek w języku
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Pyłek w języku Ruby"){.prog-lang-link}
[![Pyłek w języku
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Pyłek w języku Rust"){.prog-lang-link}
[![Pyłek w języku
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Pyłek w języku Swift"){.prog-lang-link}
[![Pyłek w języku
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Pyłek w języku TypeScript"){.prog-lang-link}
::::::::::::::::::::::::::::

::::::: {.prom .banner-sidebar .banner .banner-striped .banner-examples .banner-removable .banner-removable-patterns data-id="DP: Examples-IDE" creative-id="examples-sidebar-pl" position="sidebar"}
:::::: {.banner-inner style="font-size: 14px; line-height: 21px;"}
::: banner-examples-img
[![](../../../../images/patterns/banners/examples-ide91ca.png?id=3115b4b548fb96b75974e2de8f4f49bc){width="215"
height="158" loading="lazy"
srcset="/images/patterns/banners/examples-ide-2x.png?id=93c007a6d157b97c28eb213f59d716ae 2x"}](../../book.html)
:::

::: banner-examples-fade
[ Archiwum\
przykładów kodu](../../book.html){.btn .btn-secondary .btn-block}
:::

::: {.banner-examples-text style="margin-top: 1rem;"}
Kupując eBook **Wzorce projektowe. Nowoczesny podręcznik** zyskujesz
dostęp do archiwum zawierającego mnóstwo szczegółowych przykładów, które
można od razu wkleić do swojego preferowanego zintegrowanego środowiska
programistycznego.

[ Dowiedz się więcej...](../../book.html){.btn .btn-secondary
.btn-block}
:::
::::::
:::::::
::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::
