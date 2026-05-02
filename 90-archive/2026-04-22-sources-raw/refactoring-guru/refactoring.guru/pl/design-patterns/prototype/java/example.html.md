:::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Wzorce
projektowe](../../../design-patterns.html){.type} /
[Prototyp](../../prototype.html){.type} / [Java](../../java.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Prototyp](../../../../images/patterns/cards/prototype-mini6540.png?id=bc3046bb39ff36574c08d49839fd1c8e){srcset="/images/patterns/cards/prototype-mini-2x.png?id=b871f789a736e7efbb1cd082d2de6398 2x"}
:::

# **Prototyp** w języku Java {#prototyp-w-języku-java .pattern-example-title-block-title}
::::

::::::::::::::::::::::: pattern-example-body
::: pattern-example-brief
**Prototyp** To kreacyjny wzorzec projektowy pozwalający klonować
obiekty --- również te złożone --- bez konieczności sprzęgania z ich
klasami.

Wszystkie klasy prototyp powinny mieć wspólny interfejs który pozwoli
kopiować ich obiekty nawet gdy nie zna się ich konkretnych klas.
Obiekty-prototypy mogą tworzyć kompletne kopie, ponieważ pola prywatne
danej klasy są dostępne dla innych obiektów tej samej klasy.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Dowiedz się więcej o Prototyp ](../../prototype.html){.btn .btn-lg
.btn-primary}
:::

:::::::::::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
Nawigacja
:::

::: en-intro
 [Intro](#)
:::

::: en-intro
 [Kopiowanie figur geometrycznych](#example-0)
:::

::: en-file
 shapes
:::

:::: en-inside
::: en-file
  [Shape](#example-0--shapes-Shape-java)
:::
::::

:::: en-inside
::: en-file
  [Circle](#example-0--shapes-Circle-java)
:::
::::

:::: en-inside
::: en-file
  [Rectangle](#example-0--shapes-Rectangle-java)
:::
::::

::: en-file
 [Demo](#example-0--Demo-java)
:::

::: en-file
 [Output­Demo](#example-0--OutputDemo-txt)
:::

::: en-file
 cache
:::

:::: en-inside
::: en-file
  [Bundled­Shape­Cache](#example-0--cache-BundledShapeCache-java)
:::
::::

::: en-file
 [Demo](#example-0--Demo-java)
:::

::: en-file
 [Output­Demo](#example-0--OutputDemo-txt)
:::
::::::::::::::::::::

**Złożoność:** []{.fa-stars}

**Popularność:** []{.fa-stars}

**Przykłady użycia:** Wzorzec Prototyp jest w Javie dostępny od razu ---
dzięki interfejsowi `Cloneable`.

Każda klasa może zaimplementować ten interfejs i stać się klonowalną.

-   [`java.lang.Object#clone()`](http://docs.oracle.com/javase/8/docs/api/java/lang/Object.html#clone--)
    (klasa powinna implementować interfejs
    [`java.lang.Cloneable`](http://docs.oracle.com/javase/8/docs/api/java/lang/Cloneable.html))

**Identyfikacja:** Prototyp można łatwo rozpoznać po obecności metod
`clone` lub `copy`, itp.
:::::::::::::::::::::::

[]{#example-0}

## Kopiowanie figur geometrycznych {#example-0-title}

Spójrzmy jak można zaimplementować wzorzec Prototyp bez użycia
standardowego interfejsu `Cloneable`.

### []{#example-0--shapes .anchor} **shapes:** Lista figur {#shapes-lista-figur .example-title}

#### []{#example-0--shapes-Shape-java .anchor} **shapes/Shape.java:** Wspólny interfejs figur

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.prototype.example.shapes;

import java.util.Objects;

public abstract class Shape {
    public int x;
    public int y;
    public String color;

    public Shape() {
    }

    public Shape(Shape target) {
        if (target != null) {
            this.x = target.x;
            this.y = target.y;
            this.color = target.color;
        }
    }

    public abstract Shape clone();

    @Override
    public boolean equals(Object object2) {
        if (!(object2 instanceof Shape)) return false;
        Shape shape2 = (Shape) object2;
        return shape2.x == x &amp;&amp; shape2.y == y &amp;&amp; Objects.equals(shape2.color, color);
    }
}</code></pre>
</figure>

#### []{#example-0--shapes-Circle-java .anchor} **shapes/Circle.java:** Prosta figura

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.prototype.example.shapes;

public class Circle extends Shape {
    public int radius;

    public Circle() {
    }

    public Circle(Circle target) {
        super(target);
        if (target != null) {
            this.radius = target.radius;
        }
    }

    @Override
    public Shape clone() {
        return new Circle(this);
    }

    @Override
    public boolean equals(Object object2) {
        if (!(object2 instanceof Circle) || !super.equals(object2)) return false;
        Circle shape2 = (Circle) object2;
        return shape2.radius == radius;
    }
}</code></pre>
</figure>

#### []{#example-0--shapes-Rectangle-java .anchor} **shapes/Rectangle.java:** Kolejna figura

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.prototype.example.shapes;

public class Rectangle extends Shape {
    public int width;
    public int height;

    public Rectangle() {
    }

    public Rectangle(Rectangle target) {
        super(target);
        if (target != null) {
            this.width = target.width;
            this.height = target.height;
        }
    }

    @Override
    public Shape clone() {
        return new Rectangle(this);
    }

    @Override
    public boolean equals(Object object2) {
        if (!(object2 instanceof Rectangle) || !super.equals(object2)) return false;
        Rectangle shape2 = (Rectangle) object2;
        return shape2.width == width &amp;&amp; shape2.height == height;
    }
}</code></pre>
</figure>

#### []{#example-0--Demo-java .anchor} **Demo.java:** Przykład klonowania

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.prototype.example;

import refactoring_guru.prototype.example.shapes.Circle;
import refactoring_guru.prototype.example.shapes.Rectangle;
import refactoring_guru.prototype.example.shapes.Shape;

import java.util.ArrayList;
import java.util.List;

public class Demo {
    public static void main(String[] args) {
        List&lt;Shape&gt; shapes = new ArrayList&lt;&gt;();
        List&lt;Shape&gt; shapesCopy = new ArrayList&lt;&gt;();

        Circle circle = new Circle();
        circle.x = 10;
        circle.y = 20;
        circle.radius = 15;
        circle.color = &quot;red&quot;;
        shapes.add(circle);

        Circle anotherCircle = (Circle) circle.clone();
        shapes.add(anotherCircle);

        Rectangle rectangle = new Rectangle();
        rectangle.width = 10;
        rectangle.height = 20;
        rectangle.color = &quot;blue&quot;;
        shapes.add(rectangle);

        cloneAndCompare(shapes, shapesCopy);
    }

    private static void cloneAndCompare(List&lt;Shape&gt; shapes, List&lt;Shape&gt; shapesCopy) {
        for (Shape shape : shapes) {
            shapesCopy.add(shape.clone());
        }

        for (int i = 0; i &lt; shapes.size(); i++) {
            if (shapes.get(i) != shapesCopy.get(i)) {
                System.out.println(i + &quot;: Shapes are different objects (yay!)&quot;);
                if (shapes.get(i).equals(shapesCopy.get(i))) {
                    System.out.println(i + &quot;: And they are identical (yay!)&quot;);
                } else {
                    System.out.println(i + &quot;: But they are not identical (booo!)&quot;);
                }
            } else {
                System.out.println(i + &quot;: Shape objects are the same (booo!)&quot;);
            }
        }
    }
}</code></pre>
</figure>

#### []{#example-0--OutputDemo-txt .anchor} **OutputDemo.txt:** Wynik działania

<figure class="code">
<pre class="code" lang="output"><code>0: Shapes are different objects (yay!)
0: And they are identical (yay!)
1: Shapes are different objects (yay!)
1: And they are identical (yay!)
2: Shapes are different objects (yay!)
2: And they are identical (yay!)</code></pre>
</figure>

### Rejestr prototypów

Możemy zaimplementować scentralizowany rejestr prototypów (lub ich
fabrykę) który zawierałby zestaw predefiniowanych obiektów-prototypów.
Będzie można w ten sposób pozyskiwać nowe obiekty z fabryki przekazując
jej nazwę lub inne parametry potrzebnego prototypu. Fabryka po
znalezieniu odpowiedniego prototypu sklonowałaby go i zwróciła kopię.

### []{#example-0--cache .anchor} **cache** {#cache .example-title}

#### []{#example-0--cache-BundledShapeCache-java .anchor} **cache/BundledShapeCache.java:** Fabryka prototypów

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.prototype.caching.cache;

import refactoring_guru.prototype.example.shapes.Circle;
import refactoring_guru.prototype.example.shapes.Rectangle;
import refactoring_guru.prototype.example.shapes.Shape;

import java.util.HashMap;
import java.util.Map;

public class BundledShapeCache {
    private Map&lt;String, Shape&gt; cache = new HashMap&lt;&gt;();

    public BundledShapeCache() {
        Circle circle = new Circle();
        circle.x = 5;
        circle.y = 7;
        circle.radius = 45;
        circle.color = &quot;Green&quot;;

        Rectangle rectangle = new Rectangle();
        rectangle.x = 6;
        rectangle.y = 9;
        rectangle.width = 8;
        rectangle.height = 10;
        rectangle.color = &quot;Blue&quot;;

        cache.put(&quot;Big green circle&quot;, circle);
        cache.put(&quot;Medium blue rectangle&quot;, rectangle);
    }

    public Shape put(String key, Shape shape) {
        cache.put(key, shape);
        return shape;
    }

    public Shape get(String key) {
        return cache.get(key).clone();
    }
}</code></pre>
</figure>

#### []{#example-0--Demo-java .anchor} **Demo.java:** Przykład klonowania

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.prototype.caching;

import refactoring_guru.prototype.caching.cache.BundledShapeCache;
import refactoring_guru.prototype.example.shapes.Shape;

public class Demo {
    public static void main(String[] args) {
        BundledShapeCache cache = new BundledShapeCache();

        Shape shape1 = cache.get(&quot;Big green circle&quot;);
        Shape shape2 = cache.get(&quot;Medium blue rectangle&quot;);
        Shape shape3 = cache.get(&quot;Medium blue rectangle&quot;);

        if (shape1 != shape2 &amp;&amp; !shape1.equals(shape2)) {
            System.out.println(&quot;Big green circle != Medium blue rectangle (yay!)&quot;);
        } else {
            System.out.println(&quot;Big green circle == Medium blue rectangle (booo!)&quot;);
        }

        if (shape2 != shape3) {
            System.out.println(&quot;Medium blue rectangles are two different objects (yay!)&quot;);
            if (shape2.equals(shape3)) {
                System.out.println(&quot;And they are identical (yay!)&quot;);
            } else {
                System.out.println(&quot;But they are not identical (booo!)&quot;);
            }
        } else {
            System.out.println(&quot;Rectangle objects are the same (booo!)&quot;);
        }
    }
}</code></pre>
</figure>

#### []{#example-0--OutputDemo-txt .anchor} **OutputDemo.txt:** Wynik działania

<figure class="code">
<pre class="code" lang="output"><code>Big green circle != Medium blue rectangle (yay!)
Medium blue rectangles are two different objects (yay!)
And they are identical (yay!)</code></pre>
</figure>

::: next
#### Czytaj dalej

[Singleton w języku Java []{.fa
.fa-arrow-right}](../../singleton/java/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Wróć

[[]{.fa .fa-arrow-left} Metoda wytwórcza w języku
Java](../../factory-method/java/example.html){.btn .btn-default
rel="prev"}
:::

## **Prototyp** w innych językach

[![Prototyp w języku
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Prototyp w języku C#"){.prog-lang-link}
[![Prototyp w języku
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Prototyp w języku C++"){.prog-lang-link}
[![Prototyp w języku
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Prototyp w języku Go"){.prog-lang-link}
[![Prototyp w języku
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Prototyp w języku PHP"){.prog-lang-link}
[![Prototyp w języku
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Prototyp w języku Python"){.prog-lang-link}
[![Prototyp w języku
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Prototyp w języku Ruby"){.prog-lang-link}
[![Prototyp w języku
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Prototyp w języku Rust"){.prog-lang-link}
[![Prototyp w języku
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Prototyp w języku Swift"){.prog-lang-link}
[![Prototyp w języku
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Prototyp w języku TypeScript"){.prog-lang-link}
:::::::::::::::::::::::::::::

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
:::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::
