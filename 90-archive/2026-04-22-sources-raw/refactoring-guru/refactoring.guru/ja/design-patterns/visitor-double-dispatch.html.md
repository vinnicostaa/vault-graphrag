:::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::: {.main-content-container .center-content-container}
:::::: {.page .text}
::: breadcrumb
[](../../index.html){.home} /
[デザインパターン](../design-patterns.html){.type} /
[振る舞いに関するパターン](behavioral-patterns.html){.type} /
[Visitor](visitor.html){.type}
:::

# ビジターと二重ディスパッチ {#ビジターと二重ディスパッチ .title}

以下の幾何学形状のクラス階層を眺めてみてください[
]{.chpule1}[（]{.chpuri1}注[：]{.chpule2}[
]{.chpuri2}疑似コードです）[：]{.chpule2}[ ]{.chpuri2}

<figure class="code">
<pre class="code" lang="pseudocode"><code>interface Graphic is
    method draw()

class Shape implements Graphic is
    field id
    method draw()
    // ……

class Dot extends Shape is
    field x, y
    method draw()
    // ……

class Circle extends Dot is
    field radius
    method draw()
    // ……

class Rectangle extends Shape is
    field width, height
    method draw()
    // ……

class CompoundGraphic implements Graphic is
    field children: array of Graphic
    method draw()
    // ……</code></pre>
</figure>

コードは正常に動作し[、]{.chpule2}[
]{.chpuri2}アプリは本番稼働しています[。]{.chpule2}[
]{.chpuri2}しかし[、]{.chpule2}[ ]{.chpuri2}ある日[、]{.chpule2}[
]{.chpuri2}エクスポート機能を追加することにしました[。]{.chpule2}[
]{.chpuri2}エクスポートのコードをこれらのクラス中に置くのは[、]{.chpule2}[
]{.chpuri2}ちょっと変です[。]{.chpule2}[
]{.chpuri2}エクスポート機能をこの階層のクラスに追加する代わりに[、]{.chpule2}[
]{.chpuri2}このクラス階層からは独立した新規クラスを一つ作り[、]{.chpule2}[
]{.chpuri2}そこにエクスポートに関する全てのロジックを入れることにしました[。]{.chpule2}[
]{.chpuri2}そのクラスには[、]{.chpule2}[
]{.chpuri2}各オブジェクトの公開状態を XML
の文字列としてエクスポートするための複数のメソッドが入ります[。]{.chpule2}[
]{.chpuri2}

<figure class="code">
<pre class="code" lang="pseudocode"><code>class Exporter is
    method export(s: Shape) is
        print(&quot;Exporting shape&quot;)
    method export(d: Dot)
        print(&quot;Exporting dot&quot;)
    method export(c: Circle)
        print(&quot;Exporting circle&quot;)
    method export(r: Rectangle)
        print(&quot;Exporting rectangle&quot;)
    method export(cs: CompoundGraphic)
        print(&quot;Exporting compound&quot;)</code></pre>
</figure>

コードは良さそうに見えます[。]{.chpule2}[
]{.chpuri2}試してみましょう[。]{.chpule2}[ ]{.chpuri2}

<figure class="code">
<pre class="code" lang="pseudocode"><code>class App() is
    method export(shape: Shape) is
        Exporter exporter = new Exporter()
        exporter.export(shape);

app.export(new Circle());
// 残念ながら、&quot;Exporting shape&quot; が出力されます。</code></pre>
</figure>

ちょと待って[！]{.chpule2}[ ]{.chpuri2}どうして[？]{.chpule2}[
]{.chpuri2}

## コンパイラーの立場で考える

注[：]{.chpule2}[ ]{.chpuri2}以下の情報は[、]{.chpule2}[
]{.chpuri2}近代的なオブジェクト指向プログラミング言語のほとんど[
]{.chpule1}[（]{.chpuri1}Java[、]{.chpule2}[
]{.chpuri2}C#[、]{.chpule2}[ ]{.chpuri2}PHP など[）]{.chpule2}[
]{.chpuri2}に当てはまります[。]{.chpule2}[ ]{.chpuri2}

### 遅延・動的バインディング

自分がコンパイラーになったと思ってください[。]{.chpule2}[
]{.chpuri2}以下のコードをどうコンパイルすればいいか決めるところです[：]{.chpule2}[
]{.chpuri2}

<figure class="code">
<pre class="code" lang="pseudocode"><code>method drawShape(shape: Shape) is
    shape.draw();</code></pre>
</figure>

えっと[、]{.chpule2}[ ]{.chpuri2}`draw` メソッドは[、]{.chpule2}[
]{.chpuri2}`Shape` クラスで定義されています[。]{.chpule2}[
]{.chpuri2}あ[、]{.chpule2}[ ]{.chpuri2}ちょっと待って[、]{.chpule2}[
]{.chpuri2}四つのサブクラスでこのメソッドを上書きしています[。]{.chpule2}[
]{.chpuri2}どの実装がここで呼ばれるのか[、]{.chpule2}[
]{.chpuri2}安全に決定できるかな[？]{.chpule2}[
]{.chpuri2}そうは見えない[。]{.chpule2}[
]{.chpuri2}確実な唯一の方法は[、]{.chpule2}[
]{.chpuri2}プログラムを走らせて[、]{.chpule2}[
]{.chpuri2}メソッドに渡されたオブジェクトのクラスを調べること[。]{.chpule2}[
]{.chpuri2}確実にわかっていることは[、]{.chpule2}[
]{.chpuri2}オブジェクトが `draw`
メソッドの実装を**行う予定**ということだけ[。]{.chpule2}[ ]{.chpuri2}

なので[、]{.chpule2}[ ]{.chpuri2}結果の機械語は[、]{.chpule2}[
]{.chpuri2}`shape`
パラメーターに渡されたオブジェクトのクラスを調べて[、]{.chpule2}[
]{.chpuri2}適切なクラスの `draw`
メソッドの実装を選ぶことをします[。]{.chpule2}[ ]{.chpuri2}

このような動的な型のチェックは[、]{.chpule2}[ ]{.chpuri2}遅延[
]{.chpule1}[（]{.chpuri1}または動的[）]{.chpule2}[
]{.chpuri2}バインディングと呼ばれます[。]{.chpule2}[ ]{.chpuri2}

-   **遅延**は[、]{.chpule2}[ ]{.chpuri2}コンパイル後[、]{.chpule2}[
    ]{.chpuri2}実行時にオブジェクトと実装をリンクするため[。]{.chpule2}[
    ]{.chpuri2}
-   **動的**は[、]{.chpule2}[
    ]{.chpuri2}どの新規オブジェクトも[、]{.chpule2}[
    ]{.chpuri2}異なる実装にリンクする必要があるかもしれないから[。]{.chpule2}[
    ]{.chpuri2}

### 早期・静的バインディング

では[、]{.chpule2}[ ]{.chpuri2}以下のコードを[
]{.chpule1}[「]{.chpuri1}コンパイル[」]{.chpule2}[
]{.chpuri2}してみましょう[：]{.chpule2}[ ]{.chpuri2}

<figure class="code">
<pre class="code" lang="pseudocode"><code>method exportShape(shape: Shape) is
    Exporter exporter = new Exporter()
    exporter.export(shape);</code></pre>
</figure>

すべてのことは 2 行目で明らかです[。]{.chpule2}[ ]{.chpuri2}`Exporter`
クラスは[、]{.chpule2}[
]{.chpuri2}独自定義のコンストラクターがないので[、]{.chpule2}[
]{.chpuri2}オブジェクトのインスタンスを一つ作るだけです[。]{.chpule2}[
]{.chpuri2}`export` の呼び出しはどうでしょうか[？]{.chpule2}[
]{.chpuri2}`Exporter` には[、]{.chpule2}[
]{.chpuri2}名前が同じでパラメーター型が異なる五つのメソッドがあります[。]{.chpule2}[
]{.chpuri2}どれを呼べばいいでしょうか[？]{.chpule2}[
]{.chpuri2}どうやら[、]{.chpule2}[
]{.chpuri2}ここでも動的バインディングが必要そうです[。]{.chpule2}[
]{.chpuri2}

もう一つ問題点があります[。]{.chpule2}[ ]{.chpuri2}適切な `export`
メソッドが `Exporter`
クラス中にない新たな形状クラスにはどう対応しますか[？]{.chpule2}[
]{.chpuri2}たとえば[、]{.chpule2}[ ]{.chpuri2}`Ellipse`[
]{.chpule1}[（]{.chpuri1}楕円[）]{.chpule2}[
]{.chpuri2}オブジェクトとかです[。]{.chpule2}[
]{.chpuri2}コンパイラーは[、]{.chpule2}[ ]{.chpuri2}適切な多重定義[
]{.chpule1}[（]{.chpuri1}＝ overload[。]{.chpule2}[ ]{.chpuri2}上書き＝
overwrite とは異なる[）]{.chpule2}[
]{.chpuri2}されたメソッドが存在することは保証できません[。]{.chpule2}[
]{.chpuri2}このようなあいまいな状況は[、]{.chpule2}[
]{.chpuri2}コンパイラーの許容範囲外です[。]{.chpule2}[ ]{.chpuri2}

そのため[、]{.chpule2}[
]{.chpuri2}コンパイラー開発者は安全性を優先し[、]{.chpule2}[
]{.chpuri2}多重定義されたメソッドには早期[
]{.chpule1}[（]{.chpuri1}または静的[）]{.chpule2}[
]{.chpuri2}バインディングを使用します[：]{.chpule2}[ ]{.chpuri2}

-   **早期**は[、]{.chpule2}[
    ]{.chpuri2}プログラム開始時より前のコンパイル時に行われるから[。]{.chpule2}[
    ]{.chpuri2}
-   **静的**は[、]{.chpule2}[
    ]{.chpuri2}実行時の変更ができないから[。]{.chpule2}[ ]{.chpuri2}

最初の例に戻りましょう[。]{.chpule2}[
]{.chpuri2}渡される引数が[、]{.chpule2}[ ]{.chpuri2}`Shape`
階層のものであることはわかっています[。]{.chpule2}[ ]{.chpuri2}`Shape`
クラスか[、]{.chpule2}[
]{.chpuri2}そのサブクラスのうちの一つです[。]{.chpule2}[
]{.chpuri2}それと[、]{.chpule2}[ ]{.chpuri2}`Exporter`
クラスには[、]{.chpule2}[ ]{.chpuri2}`Shape`
クラスのエクスポートをサポートする基本的な実装[、]{.chpule2}[
]{.chpuri2}`export(s: Shape)` があることがわかっています[。]{.chpule2}[
]{.chpuri2}

あいまいな状況になることなく与えられたコードに安全にリンクできる唯一の実装は[、]{.chpule2}[
]{.chpuri2}そうなります[。]{.chpule2}[ ]{.chpuri2}`Rectangle`
オブジェクトを `export­Shape` に渡しても `Exporter` が `export(s: Shape)`
メソッドを呼び続けるのは[、]{.chpule2}[
]{.chpuri2}そういう理由によります[。]{.chpule2}[ ]{.chpuri2}

## 二重ディスパッチ

**二重ディスパッチ**は[、]{.chpule2}[
]{.chpuri2}多重定義メソッドとともに動的バインディングを使用することを可能にする技法です[。]{.chpule2}[
]{.chpuri2}このように行われます[：]{.chpule2}[ ]{.chpuri2}

<figure class="code">
<pre class="code" lang="pseudocode"><code>class Visitor is
    method visit(s: Shape) is
        print(&quot;Visited shape&quot;)
    method visit(d: Dot)
        print(&quot;Visited dot&quot;)

interface Graphic is
    method accept(v: Visitor)

class Shape implements Graphic is
    method accept(v: Visitor)
        // コンパイラーは、`this` が確実に `Shape` であることを知っている。
        // そのため、`visit(s: Shape)` を安全に呼び出し可能。
        v.visit(this)

class Dot extends Shape is
    method accept(v: Visitor)
        // コンパイラーは、`this` が `Dot` であることを知っている。
        // そのため、`visit(s: Dot)` を安全に呼び出し可能。
        v.visit(this)


Visitor v = new Visitor();
Graphic g = new Dot();

// `accept` は、（多重定義ではなく）上書きされている。コンパイラー
// は、それを動的にバインディング。従って、`accept` は、メソッド
// を呼び出すオブジェクトに対応したクラス（この場合 `Dot`）上で実行
// される。
g.accept(v);

// 「Visited dot」を表示。</code></pre>
</figure>

### 後書き

[Visitor](visitor.html)
パターンは二重ディスパッチの原則に基づいていますが[、]{.chpule2}[
]{.chpuri2}それが主目的ではありません[。]{.chpule2}[ ]{.chpuri2}Visitor
を使うと[、]{.chpule2}[
]{.chpuri2}クラス階層中のクラスの既存コードを変更せずに[[、]{.ls-1}]{.chpule2}[
]{.chpuri2}​[ ]{.chpule1}[「]{.chpuri1}外的[」]{.chpule2}[
]{.chpuri2}操作をクラス階層全体に追加できます[。]{.chpule2}[ ]{.chpuri2}

::: next
#### 次を読む

[デザインパターンを C# で []{.fa .fa-arrow-right}](csharp.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### 戻る

[[]{.fa .fa-arrow-left} Visitor](visitor.html){.btn .btn-default
rel="prev"}
:::
::::::
:::::::
::::::::
