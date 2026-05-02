::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} /
[デザインパターン](../../../design-patterns.html){.type} /
[Adapter](../../adapter.html){.type} / [Java](../../java.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Adapter](../../../../images/patterns/cards/adapter-minid866.png?id=b2ee4f681fb589be5a0685b94692aebb){srcset="/images/patterns/cards/adapter-mini-2x.png?id=8274d99afbbe9c63bfbfd0d68ceeffc7 2x"}
:::

# **Adapter** を Java で {#adapter-を-java-で .pattern-example-title-block-title}
::::

:::::::::::::::::::::: pattern-example-body
::: pattern-example-brief
**Adapter** は[、]{.chpule2}[
]{.chpuri2}構造に関するデザインパターンの一つで[、]{.chpule2}[
]{.chpuri2}非互換なオブジェクト同士の協働を可能とします[。]{.chpule2}[
]{.chpuri2}

アダプターは二つのオブジェクト間のラッパーとして機能します[。]{.chpule2}[
]{.chpuri2}一方のオブジェクトへの呼び出しを捕まえ[、]{.chpule2}[
]{.chpuri2}二つ目のオブジェクトが認識可能なデータ形式とインターフェースに変換します[。]{.chpule2}[
]{.chpuri2}
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Adapter の詳細 ](../../adapter.html){.btn .btn-lg .btn-primary}
:::

::::::::::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
ナビゲーション
:::

::: en-intro
 [はじめに](#)
:::

::: en-intro
 [四角いくいを丸い穴に合わせる](#example-0)
:::

::: en-file
 round
:::

:::: en-inside
::: en-file
  [Round­Hole](#example-0--round-RoundHole-java)
:::
::::

:::: en-inside
::: en-file
  [Round­Peg](#example-0--round-RoundPeg-java)
:::
::::

::: en-file
 square
:::

:::: en-inside
::: en-file
  [Square­Peg](#example-0--square-SquarePeg-java)
:::
::::

::: en-file
 adapters
:::

:::: en-inside
::: en-file
  [Square­Peg­Adapter](#example-0--adapters-SquarePegAdapter-java)
:::
::::

::: en-file
 [Demo](#example-0--Demo-java)
:::

::: en-file
 [Output­Demo](#example-0--OutputDemo-txt)
:::
:::::::::::::::::::

**複雑度[：]{.chpule2}[ ]{.chpuri2}**[]{.fa-stars}

**人気度[：]{.chpule2}[ ]{.chpuri2}**[]{.fa-stars}

**使用例[：]{.chpule2}[ ]{.chpuri2}** Adapter パターンは Java
コードではよく見かけます[。]{.chpule2}[
]{.chpuri2}旧来のコードに基づいたシステムでよく使用されます[。]{.chpule2}[
]{.chpuri2}そのような場合[、]{.chpule2}[
]{.chpuri2}アダプターは旧来のコードを現代のクラスから使用可能にします[。]{.chpule2}[
]{.chpuri2}

Java コア・ライブラリーには[、]{.chpule2}[
]{.chpuri2}いくつかの標準アダプターがあります[：]{.chpule2}[ ]{.chpuri2}

-   [`java.util.Arrays#asList()`](https://docs.oracle.com/javase/8/docs/api/java/util/Arrays.html#asList-T...-)
-   [`java.util.Collections#list()`](https://docs.oracle.com/javase/8/docs/api/java/util/Collections.html#list-java.util.Enumeration-)
-   [`java.util.Collections#enumeration()`](https://docs.oracle.com/javase/8/docs/api/java/util/Collections.html#enumeration-java.util.Collection-)
-   [`java.io.InputStreamReader(InputStream)`](https://docs.oracle.com/javase/8/docs/api/java/io/InputStreamReader.html#InputStreamReader-java.io.InputStream-)
    [ ]{.chpule1}[（]{.chpuri1}`Reader`
    オブジェクトを返す[）]{.chpule2}[ ]{.chpuri2}
-   [`java.io.OutputStreamWriter(OutputStream)`](https://docs.oracle.com/javase/8/docs/api/java/io/OutputStreamWriter.html#OutputStreamWriter-java.io.OutputStream-)
    [ ]{.chpule1}[（]{.chpuri1}`Writer`
    オブジェクトを返す[）]{.chpule2}[ ]{.chpuri2}
-   [`javax.xml.bind.annotation.adapters.XmlAdapter#marshal()`](https://docs.oracle.com/javase/8/docs/api/javax/xml/bind/annotation/adapters/XmlAdapter.html#marshal-BoundType-)
    および `#unmarshal()`

**見つけ方[：]{.chpule2}[ ]{.chpuri2}**アダプターは[、]{.chpule2}[
]{.chpuri2}異なる抽象クラスまたはインターフェース型のインスタンスを取るコンストラクターの存在により認識できます[。]{.chpule2}[
]{.chpuri2}アダプターは[、]{.chpule2}[
]{.chpuri2}いずれかのメソッドへの呼び出しを受け取ると[、]{.chpule2}[
]{.chpuri2}パラメーターを適切な形式に変換し[、]{.chpule2}[
]{.chpuri2}ラップされたオブジェクトの一つまたは複数のメソッドを呼び出します[。]{.chpule2}[
]{.chpuri2}
::::::::::::::::::::::

[]{#example-0}

## 四角いくいを丸い穴に合わせる {#example-0-title}

この簡単な例では[、]{.chpule2}[
]{.chpuri2}アダプターが互換性のないオブジェクト同士を動作させる方法を示します[。]{.chpule2}[
]{.chpuri2}

### []{#example-0--round .anchor} **round** {#round .example-title}

#### []{#example-0--round-RoundHole-java .anchor} **round/RoundHole.java:** 丸い穴

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.adapter.example.round;

/**
 * RoundHoles are compatible with RoundPegs.
 */
public class RoundHole {
    private double radius;

    public RoundHole(double radius) {
        this.radius = radius;
    }

    public double getRadius() {
        return radius;
    }

    public boolean fits(RoundPeg peg) {
        boolean result;
        result = (this.getRadius() &gt;= peg.getRadius());
        return result;
    }
}</code></pre>
</figure>

#### []{#example-0--round-RoundPeg-java .anchor} **round/RoundPeg.java:** 丸いくい

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.adapter.example.round;

/**
 * RoundPegs are compatible with RoundHoles.
 */
public class RoundPeg {
    private double radius;

    public RoundPeg() {}

    public RoundPeg(double radius) {
        this.radius = radius;
    }

    public double getRadius() {
        return radius;
    }
}</code></pre>
</figure>

### []{#example-0--square .anchor} **square** {#square .example-title}

#### []{#example-0--square-SquarePeg-java .anchor} **square/SquarePeg.java:** 正方形のくい

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.adapter.example.square;

/**
 * SquarePegs are not compatible with RoundHoles (they were implemented by
 * previous development team). But we have to integrate them into our program.
 */
public class SquarePeg {
    private double width;

    public SquarePeg(double width) {
        this.width = width;
    }

    public double getWidth() {
        return width;
    }

    public double getSquare() {
        double result;
        result = Math.pow(this.width, 2);
        return result;
    }
}</code></pre>
</figure>

### []{#example-0--adapters .anchor} **adapters** {#adapters .example-title}

#### []{#example-0--adapters-SquarePegAdapter-java .anchor} **adapters/SquarePegAdapter.java:** 正方形のくいの丸い穴へのアダプター

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.adapter.example.adapters;

import refactoring_guru.adapter.example.round.RoundPeg;
import refactoring_guru.adapter.example.square.SquarePeg;

/**
 * Adapter allows fitting square pegs into round holes.
 */
public class SquarePegAdapter extends RoundPeg {
    private SquarePeg peg;

    public SquarePegAdapter(SquarePeg peg) {
        this.peg = peg;
    }

    @Override
    public double getRadius() {
        double result;
        // Calculate a minimum circle radius, which can fit this peg.
        result = (Math.sqrt(Math.pow((peg.getWidth() / 2), 2) * 2));
        return result;
    }
}</code></pre>
</figure>

#### []{#example-0--Demo-java .anchor} **Demo.java:** クライアント・コード

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.adapter.example;

import refactoring_guru.adapter.example.adapters.SquarePegAdapter;
import refactoring_guru.adapter.example.round.RoundHole;
import refactoring_guru.adapter.example.round.RoundPeg;
import refactoring_guru.adapter.example.square.SquarePeg;

/**
 * Somewhere in client code...
 */
public class Demo {
    public static void main(String[] args) {
        // Round fits round, no surprise.
        RoundHole hole = new RoundHole(5);
        RoundPeg rpeg = new RoundPeg(5);
        if (hole.fits(rpeg)) {
            System.out.println(&quot;Round peg r5 fits round hole r5.&quot;);
        }

        SquarePeg smallSqPeg = new SquarePeg(2);
        SquarePeg largeSqPeg = new SquarePeg(20);
        // hole.fits(smallSqPeg); // Won&#39;t compile.

        // Adapter solves the problem.
        SquarePegAdapter smallSqPegAdapter = new SquarePegAdapter(smallSqPeg);
        SquarePegAdapter largeSqPegAdapter = new SquarePegAdapter(largeSqPeg);
        if (hole.fits(smallSqPegAdapter)) {
            System.out.println(&quot;Square peg w2 fits round hole r5.&quot;);
        }
        if (!hole.fits(largeSqPegAdapter)) {
            System.out.println(&quot;Square peg w20 does not fit into round hole r5.&quot;);
        }
    }
}</code></pre>
</figure>

#### []{#example-0--OutputDemo-txt .anchor} **OutputDemo.txt:** 実行結果

<figure class="code">
<pre class="code" lang="output"><code>Round peg r5 fits round hole r5.
Square peg w2 fits round hole r5.
Square peg w20 does not fit into round hole r5.</code></pre>
</figure>

::: next
#### 次を読む

[Bridge を Java で []{.fa
.fa-arrow-right}](../../bridge/java/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### 戻る

[[]{.fa .fa-arrow-left} Singleton を Java
で](../../singleton/java/example.html){.btn .btn-default rel="prev"}
:::

## 他言語での **Adapter**

[![Adapter を C#
で](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Adapter を C# で"){.prog-lang-link}
[![Adapter を C++
で](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Adapter を C++ で"){.prog-lang-link}
[![Adapter を Go
で](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Adapter を Go で"){.prog-lang-link}
[![Adapter を PHP
で](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Adapter を PHP で"){.prog-lang-link}
[![Adapter を Python
で](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Adapter を Python で"){.prog-lang-link}
[![Adapter を Ruby
で](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Adapter を Ruby で"){.prog-lang-link}
[![Adapter を Rust
で](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Adapter を Rust で"){.prog-lang-link}
[![Adapter を Swift
で](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Adapter を Swift で"){.prog-lang-link}
[![Adapter を TypeScript
で](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Adapter を TypeScript で"){.prog-lang-link}
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
