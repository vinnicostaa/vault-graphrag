::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} /
[デザインパターン](../../../design-patterns.html){.type} /
[Flyweight](../../flyweight.html){.type} /
[Java](../../java.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Flyweight](../../../../images/patterns/cards/flyweight-minie3d8.png?id=422ca8d2f90614dce810a8812c626698){srcset="/images/patterns/cards/flyweight-mini-2x.png?id=857ca676fff84317bb0dab76abfce08e 2x"}
:::

# **Flyweight** を Java で {#flyweight-を-java-で .pattern-example-title-block-title}
::::

:::::::::::::::::::::: pattern-example-body
::: pattern-example-brief
**Flyweight** は[、]{.chpule2}[
]{.chpuri2}構造に関するデザインパターンの一つで[、]{.chpule2}[
]{.chpuri2}メモリー消費量を低く抑えることで[、]{.chpule2}[
]{.chpuri2}プログラムが膨大な数のオブジェクトを支えることができるようにします[。]{.chpule2}[
]{.chpuri2}

複数のオブジェクト間でオブジェクトの状態の一部を共有することにより[、]{.chpule2}[
]{.chpuri2}これを実現します[。]{.chpule2}[
]{.chpuri2}つまり[、]{.chpule2}[ ]{.chpuri2}Flyweight は[、]{.chpule2}[
]{.chpuri2}異なるオブジェクトによって使われる同じデータをキャッシュすることにより[、]{.chpule2}[
]{.chpuri2}RAM を節約します[。]{.chpule2}[ ]{.chpuri2}
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Flyweight の詳細 ](../../flyweight.html){.btn .btn-lg .btn-primary}
:::

::::::::::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
ナビゲーション
:::

::: en-intro
 [はじめに](#)
:::

::: en-intro
 [森林の描画](#example-0)
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

**複雑度[：]{.chpule2}[ ]{.chpuri2}**[]{.fa-stars}

**人気度[：]{.chpule2}[ ]{.chpuri2}**[]{.fa-stars}

**使用例[：]{.chpule2}[ ]{.chpuri2}** Flyweight
の目的はただ一つです[：]{.chpule2}[
]{.chpuri2}メモリー摂取を最小にする[。]{.chpule2}[
]{.chpuri2}もしご自分のプログラムが RAM
不足で困っていない場合は[、]{.chpule2}[
]{.chpuri2}このパターンのことはしばらく忘れて大丈夫です[。]{.chpule2}[
]{.chpuri2}

Java のコア・ライブラリーでの Flyweight の例[：]{.chpule2}[ ]{.chpuri2}

-   [`java.lang.Integer#valueOf(int)`](http://docs.oracle.com/javase/8/docs/api/java/lang/Integer.html#valueOf-int-)[
    ]{.chpule1}[（]{.chpuri1}[`Boolean`](http://docs.oracle.com/javase/8/docs/api/java/lang/Boolean.html#valueOf-boolean-)[、]{.chpule2}[
    ]{.chpuri2}[`Byte`](http://docs.oracle.com/javase/8/docs/api/java/lang/Byte.html#valueOf-byte-)[、]{.chpule2}[
    ]{.chpuri2}[`Character`](http://docs.oracle.com/javase/8/docs/api/java/lang/Character.html#valueOf-char-)[、]{.chpule2}[
    ]{.chpuri2}[`Short`](http://docs.oracle.com/javase/8/docs/api/java/lang/Short.html#valueOf-short-)[、]{.chpule2}[
    ]{.chpuri2}[`Long`](http://docs.oracle.com/javase/8/docs/api/java/lang/Long.html#valueOf-long-)[、]{.chpule2}[
    ]{.chpuri2}[`Big­Decimal`](https://docs.oracle.com/javase/8/docs/api/java/math/BigDecimal.html#valueOf-long-int-)
    も同様[）]{.chpule2}[ ]{.chpuri2}

**見つけ方[：]{.chpule2}[ ]{.chpuri2}** Flyweight は[、]{.chpule2}[
]{.chpuri2}新規オブジェクトの代わりにキャッシュされたオブジェクトを返す生成メソッドの存在により識別できます[。]{.chpule2}[
]{.chpuri2}
::::::::::::::::::::::

[]{#example-0}

## 森林の描画 {#example-0-title}

この例では[、]{.chpule2}[ ]{.chpuri2}100
万本もの木からなる森林を描画します[。]{.chpule2}[
]{.chpuri2}木の一本一本は[、]{.chpule2}[
]{.chpuri2}個別のオブジェクトで表現され[、]{.chpule2}[
]{.chpuri2}何らかの状態[ ]{.chpule1}[（]{.chpuri1}座標[、]{.chpule2}[
]{.chpuri2}質感など[）]{.chpule2}[ ]{.chpuri2}を持ちます[。]{.chpule2}[
]{.chpuri2}プログラムはその本来の目的である仕事を行いますが[、]{.chpule2}[
]{.chpuri2}当然大量の RAM を消費します[。]{.chpule2}[ ]{.chpuri2}

理由は単純です[：]{.chpule2}[
]{.chpuri2}あまりに多くの木オブジェクトが重複データ[
]{.chpule1}[（]{.chpuri1}名前[、]{.chpule2}[
]{.chpuri2}質感[、]{.chpule2}[ ]{.chpuri2}色[）]{.chpule2}[
]{.chpuri2}をかかえている[。]{.chpule2}[
]{.chpuri2}これが[、]{.chpule2}[ ]{.chpuri2}Flyweight
を適用可能な理由で[、]{.chpule2}[
]{.chpuri2}これらの値は別個のフライウェイト・オブジェクト[
]{.chpule1}[（]{.chpuri1}`Tree­Type` クラス[）]{.chpule2}[
]{.chpuri2}の内部に保存できます[。]{.chpule2}[
]{.chpuri2}こうすると[、]{.chpule2}[ ]{.chpuri2}同じデータを何千もの
`Tree` オブジェクト内に保存する代わりに[、]{.chpule2}[
]{.chpuri2}特定の値の組み合わせに対応するフライウェイト・オブジェクトへの参照を行えます[。]{.chpule2}[
]{.chpuri2}

フライウェイト・オブジェクトを再利用するための煩雑さはフライウェイト・ファクトリーの中に隠蔽されているため[、]{.chpule2}[
]{.chpuri2}クライアント・コードは何も違いに気づきません[。]{.chpule2}[
]{.chpuri2}

### []{#example-0--trees .anchor} **trees** {#trees .example-title}

#### []{#example-0--trees-Tree-java .anchor} **trees/Tree.java:** それぞれの木の固有状態を含む

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

#### []{#example-0--trees-TreeType-java .anchor} **trees/TreeType.java:** いくつかの木が共有する状態を含む

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

#### []{#example-0--trees-TreeFactory-java .anchor} **trees/TreeFactory.java:** フライウェイト作成の複雑さをカプセル化する

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

#### []{#example-0--forest-Forest-java .anchor} **forest/Forest.java:** 描画対象の森林

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

#### []{#example-0--Demo-java .anchor} **Demo.java:** クライアント・コード

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

#### []{#example-0--OutputDemo-png .anchor} **OutputDemo.png:** スクリーン・ショット

![](../../../../images/patterns/examples/java/flyweight/OutputDemo5583.png?id=8bcaaf279411a0b13b42e09fdf38894e)

#### []{#example-0--OutputDemo-txt .anchor} **OutputDemo.txt:** RAM 使用状況

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
#### 次を読む

[Proxy を Java で []{.fa
.fa-arrow-right}](../../proxy/java/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### 戻る

[[]{.fa .fa-arrow-left} Facade を Java
で](../../facade/java/example.html){.btn .btn-default rel="prev"}
:::

## 他言語での **Flyweight**

[![Flyweight を C#
で](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Flyweight を C# で"){.prog-lang-link}
[![Flyweight を C++
で](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Flyweight を C++ で"){.prog-lang-link}
[![Flyweight を Go
で](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Flyweight を Go で"){.prog-lang-link}
[![Flyweight を PHP
で](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Flyweight を PHP で"){.prog-lang-link}
[![Flyweight を Python
で](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Flyweight を Python で"){.prog-lang-link}
[![Flyweight を Ruby
で](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Flyweight を Ruby で"){.prog-lang-link}
[![Flyweight を Rust
で](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Flyweight を Rust で"){.prog-lang-link}
[![Flyweight を Swift
で](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Flyweight を Swift で"){.prog-lang-link}
[![Flyweight を TypeScript
で](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Flyweight を TypeScript で"){.prog-lang-link}
::::::::::::::::::::::::::::

::::::: {.prom .banner-sidebar .banner .banner-striped .banner-examples .banner-removable .banner-removable-patterns data-id="DP: Examples-IDE" creative-id="examples-sidebar-ja" position="sidebar"}
:::::: {.banner-inner style="font-size: 14px; line-height: 21px;"}
::: banner-examples-img
[![](../../../../images/patterns/banners/examples-ide91ca.png?id=3115b4b548fb96b75974e2de8f4f49bc){width="215"
height="158" loading="lazy"
srcset="/images/patterns/banners/examples-ide-2x.png?id=93c007a6d157b97c28eb213f59d716ae 2x"}](../../book.html)
:::

::: banner-examples-fade
[ コード例の書庫](../../book.html){.btn .btn-secondary .btn-block}
:::

::: {.banner-examples-text style="margin-top: 1rem;"}
**直撃！デザインパターン**を電子書籍で購入し、IDE
で開くことができる数多くの詳細な例を含む書庫にアクセス。

[ 詳細](../../book.html){.btn .btn-secondary .btn-block}
:::
::::::
:::::::
::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::
