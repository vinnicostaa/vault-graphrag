::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Patrones de
diseño](../../../design-patterns.html){.type} /
[Flyweight](../../flyweight.html){.type} /
[Java](../../java.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Flyweight](../../../../images/patterns/cards/flyweight-minie3d8.png?id=422ca8d2f90614dce810a8812c626698){srcset="/images/patterns/cards/flyweight-mini-2x.png?id=857ca676fff84317bb0dab76abfce08e 2x"}
:::

# **Flyweight** en Java {#flyweight-en-java .pattern-example-title-block-title}
::::

:::::::::::::::::::::: pattern-example-body
::: pattern-example-brief
**Flyweight** es un patrón de diseño estructural que permite a los
programas soportar grandes cantidades de objetos manteniendo un bajo uso
de memoria.

El patrón lo logra compartiendo partes del estado del objeto entre
varios objetos. En otras palabras, el Flyweight ahorra memoria RAM
guardando en caché la misma información utilizada por distintos objetos.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Aprende más sobre el patrón Flyweight ](../../flyweight.html){.btn
.btn-lg .btn-primary}
:::

::::::::::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
Navegación
:::

::: en-intro
 [Intro](#)
:::

::: en-intro
 [Representación de un bosque](#example-0)
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

**Complejidad:** []{.fa-stars}

**Popularidad:** []{.fa-stars}

**Ejemplos de uso:** El patrón Flyweight tiene un único propósito:
minimizar el consumo de memoria. Si tu programa no tiene problemas de
escasez de RAM, puedes ignorar este patrón por una temporada.

Ejemplos del patrón Flyweight en las principales bibliotecas de Java:

-   [`java.lang.Integer#valueOf(int)`](http://docs.oracle.com/javase/8/docs/api/java/lang/Integer.html#valueOf-int-)
    (también
    [`Boolean`](http://docs.oracle.com/javase/8/docs/api/java/lang/Boolean.html#valueOf-boolean-),
    [`Byte`](http://docs.oracle.com/javase/8/docs/api/java/lang/Byte.html#valueOf-byte-),
    [`Character`](http://docs.oracle.com/javase/8/docs/api/java/lang/Character.html#valueOf-char-),
    [`Short`](http://docs.oracle.com/javase/8/docs/api/java/lang/Short.html#valueOf-short-),
    [`Long`](http://docs.oracle.com/javase/8/docs/api/java/lang/Long.html#valueOf-long-)
    y
    [`BigDecimal`](https://docs.oracle.com/javase/8/docs/api/java/math/BigDecimal.html#valueOf-long-int-))

**Identificación:** El patrón Flyweight puede reconocerse por un método
de creación que devuelve objetos guardados en caché en lugar de crear
objetos nuevos.
::::::::::::::::::::::

[]{#example-0}

## Representación de un bosque {#example-0-title}

En este ejemplo, vamos a representar un bosque (¡1 000 000 de árboles!).
Cada árbol será representado por su propio objeto, que tiene cierto
estado (coordenadas, textura, etcétera). Aunque el programa hace su
trabajo principal, naturalmente consume mucha memoria RAM.

La razón es sencilla: demasiados objetos de árbol duplican la
información (nombre, textura, color). Por eso podemos aplicar el patrón
Flyweight y almacenar estos valores dentro de objetos flyweight
separados (la clase `TreeType` (TipodeÁrbol)). Ahora, en lugar de
almacenar la misma información en miles de objetos `Tree` (Árbol), vamos
a referenciar uno de los objetos flyweight con un grupo particular de
valores.

El código cliente no se dará cuenta de nada, ya que la complejidad de
reutilizar objetos flyweight queda enterrada dentro de una fábrica
flyweight.

### []{#example-0--trees .anchor} **trees** {#trees .example-title}

#### []{#example-0--trees-Tree-java .anchor} **trees/Tree.java:** Contiene estado único para cada árbol

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

#### []{#example-0--trees-TreeType-java .anchor} **trees/TreeType.java:** Contiene estado compartido por varios árboles

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

#### []{#example-0--trees-TreeFactory-java .anchor} **trees/TreeFactory.java:** Encapsula la complejidad de la creación de flyweight

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

#### []{#example-0--forest-Forest-java .anchor} **forest/Forest.java:** Bosque que dibujamos

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

#### []{#example-0--Demo-java .anchor} **Demo.java:** Código cliente

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

#### []{#example-0--OutputDemo-png .anchor} **OutputDemo.png:** Captura de pantalla

![](../../../../images/patterns/examples/java/flyweight/OutputDemo5583.png?id=8bcaaf279411a0b13b42e09fdf38894e)

#### []{#example-0--OutputDemo-txt .anchor} **OutputDemo.txt:** Datos de uso de RAM

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
#### Leer siguiente

[Proxy en Java []{.fa
.fa-arrow-right}](../../proxy/java/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Volver

[[]{.fa .fa-arrow-left} Facade en
Java](../../facade/java/example.html){.btn .btn-default rel="prev"}
:::

## **Flyweight** en otros lenguajes

[![Flyweight en
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Flyweight en C#"){.prog-lang-link}
[![Flyweight en
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Flyweight en C++"){.prog-lang-link}
[![Flyweight en
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Flyweight en Go"){.prog-lang-link}
[![Flyweight en
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Flyweight en PHP"){.prog-lang-link}
[![Flyweight en
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Flyweight en Python"){.prog-lang-link}
[![Flyweight en
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Flyweight en Ruby"){.prog-lang-link}
[![Flyweight en
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Flyweight en Rust"){.prog-lang-link}
[![Flyweight en
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Flyweight en Swift"){.prog-lang-link}
[![Flyweight en
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Flyweight en TypeScript"){.prog-lang-link}
::::::::::::::::::::::::::::

::::::: {.prom .banner-sidebar .banner .banner-striped .banner-examples .banner-removable .banner-removable-patterns data-id="DP: Examples-IDE" creative-id="examples-sidebar-es" position="sidebar"}
:::::: {.banner-inner style="font-size: 14px; line-height: 21px;"}
::: banner-examples-img
[![](../../../../images/patterns/banners/examples-ide91ca.png?id=3115b4b548fb96b75974e2de8f4f49bc){width="215"
height="158" loading="lazy"
srcset="/images/patterns/banners/examples-ide-2x.png?id=93c007a6d157b97c28eb213f59d716ae 2x"}](../../book.html)
:::

::: banner-examples-fade
[ Archivo con ejemplos de código](../../book.html){.btn .btn-secondary
.btn-block}
:::

::: {.banner-examples-text style="margin-top: 1rem;"}
Compra el libro electrónico **Sumérgete en los patrones de diseño** para
tener acceso al archivo con decenas de ejemplos detallados que puedes
abrir en tu IDE.

[ Saber más...](../../book.html){.btn .btn-secondary .btn-block}
:::
::::::
:::::::
::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::
