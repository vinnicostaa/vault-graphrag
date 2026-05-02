::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Patrons de
conception](../../../design-patterns.html){.type} / [Poids
mouche](../../flyweight.html){.type} / [Java](../../java.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Poids
mouche](../../../../images/patterns/cards/flyweight-minie3d8.png?id=422ca8d2f90614dce810a8812c626698){srcset="/images/patterns/cards/flyweight-mini-2x.png?id=857ca676fff84317bb0dab76abfce08e 2x"}
:::

# **Poids mouche** en Java {#poids-mouche-en-java .pattern-example-title-block-title}
::::

:::::::::::::::::::::: pattern-example-body
::: pattern-example-brief
Le **poids mouche** est un patron de conception structurel qui permet à
des programmes de limiter leur consommation de mémoire malgré un très
grand nombre d'objets.

Ce patron est obtenu en partageant des parties de l'état d'un objet à
plusieurs autres objets. En d'autres termes, le poids mouche économise
de la RAM en mettant en cache les données identiques chez différents
objets.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ En savoir plus sur la patron Poids mouche ](../../flyweight.html){.btn
.btn-lg .btn-primary}
:::

::::::::::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
Navigation
:::

::: en-intro
 [Intro](#)
:::

::: en-intro
 [Afficher une forêt](#example-0)
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

**Complexité :** []{.fa-stars}

**Popularité :** []{.fa-stars}

**Exemples d'utilisation :** Le poids mouche n'a qu'une seule utilité :
minimiser la consommation de mémoire. Si votre programme ne rencontre
aucun problème de RAM, ignorez ce patron pour le moment.

Voici quelques exemples tirés des bibliothèques principales de Java :

-   [`java.lang.Integer#valueOf(int)`](http://docs.oracle.com/javase/8/docs/api/java/lang/Integer.html#valueOf-int-)
    (mais aussi
    [`Boolean`](http://docs.oracle.com/javase/8/docs/api/java/lang/Boolean.html#valueOf-boolean-),
    [`Byte`](http://docs.oracle.com/javase/8/docs/api/java/lang/Byte.html#valueOf-byte-),
    [`Character`](http://docs.oracle.com/javase/8/docs/api/java/lang/Character.html#valueOf-char-),
    [`Short`](http://docs.oracle.com/javase/8/docs/api/java/lang/Short.html#valueOf-short-),
    [`Long`](http://docs.oracle.com/javase/8/docs/api/java/lang/Long.html#valueOf-long-)
    et
    [`BigDecimal`](https://docs.oracle.com/javase/8/docs/api/java/math/BigDecimal.html#valueOf-long-int-))

**Identification :** Le poids mouche peut être reconnu par une méthode
de création qui renvoie des objets du cache plutôt que d'en créer de
nouveaux.
::::::::::::::::::::::

[]{#example-0}

## Afficher une forêt {#example-0-title}

Dans cet exemple, nous allons procéder à l'affichage d'une forêt (1 000
000 d'arbres) ! Chaque arbre sera représenté par son propre objet avec
un certain état (coordonnées, texture, etc.). Même si le programme
remplit bien son rôle, il consomme naturellement énormément de RAM.

La raison est simple : de trop nombreux objets Arbre contiennent des
données dupliquées (nom, texture, couleur). Nous pouvons lui appliquer
le poids mouche afin de stocker ces valeurs dans des objets poids mouche
séparés (la classe `TypeArbre`). Plutôt que de stocker les mêmes données
des milliers de fois dans des objets `Arbre`, nous allons référencer un
objet poids mouche avec certaines valeurs.

Le code client ne s'en rendra même pas compte puisque la complexité de
la réutilisation des objets poids mouche est cachée à l'intérieur d'une
fabrique de poids mouches.

### []{#example-0--trees .anchor} **trees** {#trees .example-title}

#### []{#example-0--trees-Tree-java .anchor} **trees/Tree.java:** Contient un état unique pour chaque arbre

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

#### []{#example-0--trees-TreeType-java .anchor} **trees/TreeType.java:** Contient un état partagé entre plusieurs arbres

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

#### []{#example-0--trees-TreeFactory-java .anchor} **trees/TreeFactory.java:** Encapsule la complexité de la création de poids mouches

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

#### []{#example-0--forest-Forest-java .anchor} **forest/Forest.java:** La forêt que nous dessinons

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

#### []{#example-0--Demo-java .anchor} **Demo.java:** Code client

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

#### []{#example-0--OutputDemo-png .anchor} **OutputDemo.png:** Impression d'écran

![](../../../../images/patterns/examples/java/flyweight/OutputDemo5583.png?id=8bcaaf279411a0b13b42e09fdf38894e)

#### []{#example-0--OutputDemo-txt .anchor} **OutputDemo.txt:** Statistiques d'utilisation de la RAM

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
#### Suivant

[Procuration en Java []{.fa
.fa-arrow-right}](../../proxy/java/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Retour

[[]{.fa .fa-arrow-left} Façade en
Java](../../facade/java/example.html){.btn .btn-default rel="prev"}
:::

## **Poids mouche** dans les autres langues

[![Poids mouche en
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Poids mouche en C#"){.prog-lang-link}
[![Poids mouche en
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Poids mouche en C++"){.prog-lang-link}
[![Poids mouche en
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Poids mouche en Go"){.prog-lang-link}
[![Poids mouche en
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Poids mouche en PHP"){.prog-lang-link}
[![Poids mouche en
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Poids mouche en Python"){.prog-lang-link}
[![Poids mouche en
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Poids mouche en Ruby"){.prog-lang-link}
[![Poids mouche en
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Poids mouche en Rust"){.prog-lang-link}
[![Poids mouche en
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Poids mouche en Swift"){.prog-lang-link}
[![Poids mouche en
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Poids mouche en TypeScript"){.prog-lang-link}
::::::::::::::::::::::::::::

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
::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::
