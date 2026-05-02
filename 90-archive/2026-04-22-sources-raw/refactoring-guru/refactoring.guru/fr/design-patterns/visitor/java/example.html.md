:::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Patrons de
conception](../../../design-patterns.html){.type} /
[Visiteur](../../visitor.html){.type} / [Java](../../java.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Visiteur](../../../../images/patterns/cards/visitor-mini0a1c.png?id=854a35a62963bec1d75eab996918989b){srcset="/images/patterns/cards/visitor-mini-2x.png?id=9b87f3f3b772f434b28a25876829b504 2x"}
:::

# **Visiteur** en Java {#visiteur-en-java .pattern-example-title-block-title}
::::

::::::::::::::::::::::::::: pattern-example-body
::: pattern-example-brief
Le **Visiteur** est un patron de conception comportemental qui permet
d'ajouter de nouveaux comportements à une hiérarchie de classes sans
modifier l'existant.

> Découvrez pourquoi les visiteurs ne peuvent pas être remplacés par la
> surcharge de méthodes dans notre article [Visiteur et double
> répartition](../../visitor-double-dispatch.html).
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ En savoir plus sur la patron Visiteur ](../../visitor.html){.btn
.btn-lg .btn-primary}
:::

:::::::::::::::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
Navigation
:::

::: en-intro
 [Intro](#)
:::

::: en-intro
 [Exporter des formes en XML](#example-0)
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
  [Dot](#example-0--shapes-Dot-java)
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

:::: en-inside
::: en-file
  [Compound­Shape](#example-0--shapes-CompoundShape-java)
:::
::::

::: en-file
 visitor
:::

:::: en-inside
::: en-file
  [Visitor](#example-0--visitor-Visitor-java)
:::
::::

:::: en-inside
::: en-file
  [XMLExport­Visitor](#example-0--visitor-XMLExportVisitor-java)
:::
::::

::: en-file
 [Demo](#example-0--Demo-java)
:::

::: en-file
 [Output­Demo](#example-0--OutputDemo-txt)
:::
::::::::::::::::::::::::

**Complexité :** []{.fa-stars}

**Popularité :** []{.fa-stars}

**Exemples d'utilisation :** Le visiteur n'est pas un patron très
répandu à cause de sa complexité et de la rareté de ses cas
d'utilisation.

Voici quelques exemples tirés des bibliothèques principales de Java :

-   [`javax.lang.model.element.AnnotationValue`](http://docs.oracle.com/javase/8/docs/api/javax/lang/model/element/AnnotationValue.html)
    et
    [`AnnotationValueVisitor`](http://docs.oracle.com/javase/8/docs/api/javax/lang/model/element/AnnotationValueVisitor.html)
-   [`javax.lang.model.element.Element`](http://docs.oracle.com/javase/8/docs/api/javax/lang/model/element/Element.html)
    et
    [`ElementVisitor`](http://docs.oracle.com/javase/8/docs/api/javax/lang/model/element/ElementVisitor.html)
-   [`javax.lang.model.type.TypeMirror`](http://docs.oracle.com/javase/8/docs/api/javax/lang/model/type/TypeMirror.html)
    et
    [`TypeVisitor`](http://docs.oracle.com/javase/8/docs/api/javax/lang/model/type/TypeVisitor.html)
-   [`java.nio.file.FileVisitor`](http://docs.oracle.com/javase/8/docs/api/java/nio/file/FileVisitor.html)
    et
    [`SimpleFileVisitor`](http://docs.oracle.com/javase/8/docs/api/java/nio/file/SimpleFileVisitor.html)
-   [`javax.faces.component.visit.VisitContext`](http://docs.oracle.com/javaee/7/api/javax/faces/component/visit/VisitContext.html)
    et
    [`VisitCallback`](http://docs.oracle.com/javaee/7/api/javax/faces/component/visit/VisitCallback.html)
:::::::::::::::::::::::::::

[]{#example-0}

## Exporter des formes en XML {#example-0-title}

Dans cet exemple, nous voulons exporter un ensemble de formes
géométriques en XML. Le truc, c'est que nous ne voulons pas modifier le
code des formes, ou tout du moins éviter au maximum d'y toucher.

Au final, le visiteur va établir une infrastructure qui nous permet
d'ajouter n'importe quels comportements à la hiérarchie des formes sans
toucher au code existant de ces classes.

### []{#example-0--shapes .anchor} **shapes** {#shapes .example-title}

#### []{#example-0--shapes-Shape-java .anchor} **shapes/Shape.java:** Interface Forme commune

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.visitor.example.shapes;

import refactoring_guru.visitor.example.visitor.Visitor;

public interface Shape {
    void move(int x, int y);
    void draw();
    String accept(Visitor visitor);
}</code></pre>
</figure>

#### []{#example-0--shapes-Dot-java .anchor} **shapes/Dot.java:** Un point

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.visitor.example.shapes;

import refactoring_guru.visitor.example.visitor.Visitor;

public class Dot implements Shape {
    private int id;
    private int x;
    private int y;

    public Dot() {
    }

    public Dot(int id, int x, int y) {
        this.id = id;
        this.x = x;
        this.y = y;
    }

    @Override
    public void move(int x, int y) {
        // move shape
    }

    @Override
    public void draw() {
        // draw shape
    }

    @Override
    public String accept(Visitor visitor) {
        return visitor.visitDot(this);
    }

    public int getX() {
        return x;
    }

    public int getY() {
        return y;
    }

    public int getId() {
        return id;
    }
}</code></pre>
</figure>

#### []{#example-0--shapes-Circle-java .anchor} **shapes/Circle.java:** Un cercle

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.visitor.example.shapes;

import refactoring_guru.visitor.example.visitor.Visitor;

public class Circle extends Dot {
    private int radius;

    public Circle(int id, int x, int y, int radius) {
        super(id, x, y);
        this.radius = radius;
    }

    @Override
    public String accept(Visitor visitor) {
        return visitor.visitCircle(this);
    }

    public int getRadius() {
        return radius;
    }
}</code></pre>
</figure>

#### []{#example-0--shapes-Rectangle-java .anchor} **shapes/Rectangle.java:** Un rectangle

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.visitor.example.shapes;

import refactoring_guru.visitor.example.visitor.Visitor;

public class Rectangle implements Shape {
    private int id;
    private int x;
    private int y;
    private int width;
    private int height;

    public Rectangle(int id, int x, int y, int width, int height) {
        this.id = id;
        this.x = x;
        this.y = y;
        this.width = width;
        this.height = height;
    }

    @Override
    public String accept(Visitor visitor) {
        return visitor.visitRectangle(this);
    }

    @Override
    public void move(int x, int y) {
        // move shape
    }

    @Override
    public void draw() {
        // draw shape
    }

    public int getId() {
        return id;
    }

    public int getX() {
        return x;
    }

    public int getY() {
        return y;
    }

    public int getWidth() {
        return width;
    }

    public int getHeight() {
        return height;
    }
}</code></pre>
</figure>

#### []{#example-0--shapes-CompoundShape-java .anchor} **shapes/CompoundShape.java:** Une forme composée

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.visitor.example.shapes;

import refactoring_guru.visitor.example.visitor.Visitor;

import java.util.ArrayList;
import java.util.List;

public class CompoundShape implements Shape {
    public int id;
    public List&lt;Shape&gt; children = new ArrayList&lt;&gt;();

    public CompoundShape(int id) {
        this.id = id;
    }

    @Override
    public void move(int x, int y) {
        // move shape
    }

    @Override
    public void draw() {
        // draw shape
    }

    public int getId() {
        return id;
    }

    @Override
    public String accept(Visitor visitor) {
        return visitor.visitCompoundGraphic(this);
    }

    public void add(Shape shape) {
        children.add(shape);
    }
}</code></pre>
</figure>

### []{#example-0--visitor .anchor} **visitor** {#visitor .example-title}

#### []{#example-0--visitor-Visitor-java .anchor} **visitor/Visitor.java:** Interface Visiteur commune

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.visitor.example.visitor;

import refactoring_guru.visitor.example.shapes.Circle;
import refactoring_guru.visitor.example.shapes.CompoundShape;
import refactoring_guru.visitor.example.shapes.Dot;
import refactoring_guru.visitor.example.shapes.Rectangle;

public interface Visitor {
    String visitDot(Dot dot);

    String visitCircle(Circle circle);

    String visitRectangle(Rectangle rectangle);

    String visitCompoundGraphic(CompoundShape cg);
}</code></pre>
</figure>

#### []{#example-0--visitor-XMLExportVisitor-java .anchor} **visitor/XMLExportVisitor.java:** Visiteur concret qui exporte toutes les formes en XML

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.visitor.example.visitor;

import refactoring_guru.visitor.example.shapes.*;

public class XMLExportVisitor implements Visitor {

    public String export(Shape... args) {
        StringBuilder sb = new StringBuilder();
        sb.append(&quot;&lt;?xml version=\&quot;1.0\&quot; encoding=\&quot;utf-8\&quot;?&gt;&quot; + &quot;\n&quot;);
        for (Shape shape : args) {
            sb.append(shape.accept(this)).append(&quot;\n&quot;);
        }
        return sb.toString();
    }

    public String visitDot(Dot d) {
        return &quot;&lt;dot&gt;&quot; + &quot;\n&quot; +
                &quot;    &lt;id&gt;&quot; + d.getId() + &quot;&lt;/id&gt;&quot; + &quot;\n&quot; +
                &quot;    &lt;x&gt;&quot; + d.getX() + &quot;&lt;/x&gt;&quot; + &quot;\n&quot; +
                &quot;    &lt;y&gt;&quot; + d.getY() + &quot;&lt;/y&gt;&quot; + &quot;\n&quot; +
                &quot;&lt;/dot&gt;&quot;;
    }

    public String visitCircle(Circle c) {
        return &quot;&lt;circle&gt;&quot; + &quot;\n&quot; +
                &quot;    &lt;id&gt;&quot; + c.getId() + &quot;&lt;/id&gt;&quot; + &quot;\n&quot; +
                &quot;    &lt;x&gt;&quot; + c.getX() + &quot;&lt;/x&gt;&quot; + &quot;\n&quot; +
                &quot;    &lt;y&gt;&quot; + c.getY() + &quot;&lt;/y&gt;&quot; + &quot;\n&quot; +
                &quot;    &lt;radius&gt;&quot; + c.getRadius() + &quot;&lt;/radius&gt;&quot; + &quot;\n&quot; +
                &quot;&lt;/circle&gt;&quot;;
    }

    public String visitRectangle(Rectangle r) {
        return &quot;&lt;rectangle&gt;&quot; + &quot;\n&quot; +
                &quot;    &lt;id&gt;&quot; + r.getId() + &quot;&lt;/id&gt;&quot; + &quot;\n&quot; +
                &quot;    &lt;x&gt;&quot; + r.getX() + &quot;&lt;/x&gt;&quot; + &quot;\n&quot; +
                &quot;    &lt;y&gt;&quot; + r.getY() + &quot;&lt;/y&gt;&quot; + &quot;\n&quot; +
                &quot;    &lt;width&gt;&quot; + r.getWidth() + &quot;&lt;/width&gt;&quot; + &quot;\n&quot; +
                &quot;    &lt;height&gt;&quot; + r.getHeight() + &quot;&lt;/height&gt;&quot; + &quot;\n&quot; +
                &quot;&lt;/rectangle&gt;&quot;;
    }

    public String visitCompoundGraphic(CompoundShape cg) {
        return &quot;&lt;compound_graphic&gt;&quot; + &quot;\n&quot; +
                &quot;   &lt;id&gt;&quot; + cg.getId() + &quot;&lt;/id&gt;&quot; + &quot;\n&quot; +
                _visitCompoundGraphic(cg) +
                &quot;&lt;/compound_graphic&gt;&quot;;
    }

    private String _visitCompoundGraphic(CompoundShape cg) {
        StringBuilder sb = new StringBuilder();
        for (Shape shape : cg.children) {
            String obj = shape.accept(this);
            // Proper indentation for sub-objects.
            obj = &quot;    &quot; + obj.replace(&quot;\n&quot;, &quot;\n    &quot;) + &quot;\n&quot;;
            sb.append(obj);
        }
        return sb.toString();
    }

}</code></pre>
</figure>

#### []{#example-0--Demo-java .anchor} **Demo.java:** Code client

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.visitor.example;

import refactoring_guru.visitor.example.shapes.*;
import refactoring_guru.visitor.example.visitor.XMLExportVisitor;

public class Demo {
    public static void main(String[] args) {
        Dot dot = new Dot(1, 10, 55);
        Circle circle = new Circle(2, 23, 15, 10);
        Rectangle rectangle = new Rectangle(3, 10, 17, 20, 30);

        CompoundShape compoundShape = new CompoundShape(4);
        compoundShape.add(dot);
        compoundShape.add(circle);
        compoundShape.add(rectangle);

        CompoundShape c = new CompoundShape(5);
        c.add(dot);
        compoundShape.add(c);

        export(circle, compoundShape);
    }

    private static void export(Shape... shapes) {
        XMLExportVisitor exportVisitor = new XMLExportVisitor();
        System.out.println(exportVisitor.export(shapes));
    }
}</code></pre>
</figure>

#### []{#example-0--OutputDemo-txt .anchor} **OutputDemo.txt:** Résultat de l'exécution

<figure class="code">
<pre class="code" lang="output"><code>&lt;?xml version=&quot;1.0&quot; encoding=&quot;utf-8&quot;?&gt;
&lt;circle&gt;
    &lt;id&gt;2&lt;/id&gt;
    &lt;x&gt;23&lt;/x&gt;
    &lt;y&gt;15&lt;/y&gt;
    &lt;radius&gt;10&lt;/radius&gt;
&lt;/circle&gt;

&lt;?xml version=&quot;1.0&quot; encoding=&quot;utf-8&quot;?&gt;
&lt;compound_graphic&gt;
   &lt;id&gt;4&lt;/id&gt;
    &lt;dot&gt;
        &lt;id&gt;1&lt;/id&gt;
        &lt;x&gt;10&lt;/x&gt;
        &lt;y&gt;55&lt;/y&gt;
    &lt;/dot&gt;
    &lt;circle&gt;
        &lt;id&gt;2&lt;/id&gt;
        &lt;x&gt;23&lt;/x&gt;
        &lt;y&gt;15&lt;/y&gt;
        &lt;radius&gt;10&lt;/radius&gt;
    &lt;/circle&gt;
    &lt;rectangle&gt;
        &lt;id&gt;3&lt;/id&gt;
        &lt;x&gt;10&lt;/x&gt;
        &lt;y&gt;17&lt;/y&gt;
        &lt;width&gt;20&lt;/width&gt;
        &lt;height&gt;30&lt;/height&gt;
    &lt;/rectangle&gt;
    &lt;compound_graphic&gt;
       &lt;id&gt;5&lt;/id&gt;
        &lt;dot&gt;
            &lt;id&gt;1&lt;/id&gt;
            &lt;x&gt;10&lt;/x&gt;
            &lt;y&gt;55&lt;/y&gt;
        &lt;/dot&gt;
    &lt;/compound_graphic&gt;
&lt;/compound_graphic&gt;</code></pre>
</figure>

::: next
#### Suivant

[Les patrons de conception en PHP []{.fa
.fa-arrow-right}](../../php.html){.btn .btn-primary rel="next"}
:::

::: prev
#### Retour

[[]{.fa .fa-arrow-left} Patron de méthode en
Java](../../template-method/java/example.html){.btn .btn-default
rel="prev"}
:::

## **Visiteur** dans les autres langues

[![Visiteur en
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Visiteur en C#"){.prog-lang-link}
[![Visiteur en
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Visiteur en C++"){.prog-lang-link}
[![Visiteur en
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Visiteur en Go"){.prog-lang-link}
[![Visiteur en
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Visiteur en PHP"){.prog-lang-link}
[![Visiteur en
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Visiteur en Python"){.prog-lang-link}
[![Visiteur en
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Visiteur en Ruby"){.prog-lang-link}
[![Visiteur en
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Visiteur en Rust"){.prog-lang-link}
[![Visiteur en
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Visiteur en Swift"){.prog-lang-link}
[![Visiteur en
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Visiteur en TypeScript"){.prog-lang-link}
:::::::::::::::::::::::::::::::::

::::::: {.prom .banner-sidebar .banner .banner-striped .banner-examples .banner-removable .banner-removable-patterns data-id="DP: Examples-IDE" creative-id="examples-sidebar-fr" position="sidebar"}
:::::: {.banner-inner style="font-size: 14px; line-height: 21px;"}
::: banner-examples-img
[![](../../../../images/patterns/banners/examples-ide91ca.png?id=3115b4b548fb96b75974e2de8f4f49bc){width="215"
height="158" loading="lazy"
srcset="/images/patterns/banners/examples-ide-2x.png?id=93c007a6d157b97c28eb213f59d716ae 2x"}](../../book.html)
:::

::: banner-examples-fade
[ Archive avec exemples](../../book.html){.btn .btn-secondary
.btn-block}
:::

::: {.banner-examples-text style="margin-top: 1rem;"}
Achetez l'ebook **Plongée au cœur des patrons de conception** et obtenez
l'accès à une archive comprenant des dizaines d'exemples détaillés qui
peuvent être ouverts dans votre environnement de développement.

[ En savoir plus...](../../book.html){.btn .btn-secondary .btn-block}
:::
::::::
:::::::
:::::::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::::::
