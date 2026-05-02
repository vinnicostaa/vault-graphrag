::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} /
[デザインパターン](../design-patterns.html){.type} /
[構造に関するパターン](structural-patterns.html){.type}
:::

# Composite {#composite .title}

::: aka
別名：[Object
Tree、]{style="display:inline-block"}[コンポジット、]{style="display:inline-block"}[混成、]{style="display:inline-block"}[オブジェクト・ツリー]{style="display:inline-block"}
:::

::: {.section .intent}
##  一言でいうと {#intent}

**Composite**[ ]{.chpule1}[（]{.chpuri1}コンポジット[、]{.chpule2}[
]{.chpuri2}混成[）]{.chpule2}[ ]{.chpuri2}は[、]{.chpule2}[
]{.chpuri2}構造に関するデザインパターンの一つで[、]{.chpule2}[
]{.chpuri2}オブジェクトからツリー[
]{.chpule1}[（]{.chpuri1}木[）]{.chpule2}[
]{.chpuri2}構造を組み立て[、]{.chpule2}[
]{.chpuri2}そのツリー構造がまるで独立したオブジェクトであるかのように扱えるようにします[。]{.chpule2}[
]{.chpuri2}

<figure class="image">
<img
src="../../images/patterns/content/composite/compositec299.png?id=73bcf0d94db360b636cd745f710d19db"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/composite/composite.png?id=73bcf0d94db360b636cd745f710d19db 640w,/images/patterns/content/composite/composite-1.5x.png?id=098acddafae70b1ec1cb1b05e833c3ae 960w,/images/patterns/content/composite/composite-2x.png?id=8847e6f8e2cb892ed2229faba83bd1b7 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Composite デザインパターン" />
</figure>
:::

::: {.section .problem}
##  問題 {#problem}

Composite パターンは[、]{.chpule2}[
]{.chpuri2}アプリの中核となるモデルがツリー構造で表現できる場合にのみ適用する意味があります[。]{.chpule2}[
]{.chpuri2}

たとえば[、]{.chpule2}[ ]{.chpuri2}2
種類のオブジェクトがあることを想像してください[。]{.chpule2}[
]{.chpuri2}`Product`[ ]{.chpule1}[（]{.chpuri1}製品[）]{.chpule2}[
]{.chpuri2}と `Box`[ ]{.chpule1}[（]{.chpuri1}箱[）]{.chpule2}[
]{.chpuri2}です[。]{.chpule2}[ ]{.chpuri2}一つの `Box` はいくつかの
`Product` といくつかの小さな `Box` を含むことができます[。]{.chpule2}[
]{.chpuri2}これらの小さな `Box` も[、]{.chpule2}[ ]{.chpuri2}いくつかの
`Product` やさらに小さな `Box` を含むことができ[、]{.chpule2}[
]{.chpuri2}以下同様です[。]{.chpule2}[ ]{.chpuri2}

これらのクラスを使って[、]{.chpule2}[
]{.chpuri2}注文システムを作ることにしたとします[。]{.chpule2}[
]{.chpuri2}注文は[、]{.chpule2}[
]{.chpuri2}包装なしでシンプルな製品かもしれませんし[、]{.chpule2}[
]{.chpuri2}箱の中に製品と他の箱が詰め込まれたものかもしれません[。]{.chpule2}[
]{.chpuri2}このような注文の合計価格を計算するにはどうすればいいでしょう[？]{.chpule2}[
]{.chpuri2}

<figure class="image">
<img
src="../../images/patterns/diagrams/composite/problem-jaa831.png?id=6cf90871eeadb5c2ef6a645bf5bd1026"
style="aspect-ratio: 1;"
srcset="/images/patterns/diagrams/composite/problem-ja.png?id=6cf90871eeadb5c2ef6a645bf5bd1026 370w,/images/patterns/diagrams/composite/problem-ja-1.5x.png?id=f285be85fec9821f822f6b3e6a48e9c6 555w,/images/patterns/diagrams/composite/problem-ja-2x.png?id=86cc46ad70da11711b62576e7f7267be 740w"
sizes="(max-width: 720px) 100vw, 370px" loading="lazy" width="370"
alt="複雑な注文の例" />
<figcaption><p>注文は<span class="chpule2">、</span><span
class="chpuri2"> </span>箱に包装された様々な製品と<span
class="chpule2">、</span><span class="chpuri2">
</span>それを包む大きな箱かもしれない<span
class="chpule2">。</span><span class="chpuri2">
</span>全体の構造は上下逆にした木のようにみえる<span
class="chpule2">。</span><span class="chpuri2"> </span></p></figcaption>
</figure>

直接的なやり方を試みることは可能です[：]{.chpule2}[
]{.chpuri2}箱を全部開けて[、]{.chpule2}[
]{.chpuri2}製品を取り出し[、]{.chpule2}[
]{.chpuri2}合計を計算[。]{.chpule2}[
]{.chpuri2}現実世界では可能なやり方です[。]{.chpule2}[
]{.chpuri2}しかしプログラム上では[、]{.chpule2}[
]{.chpuri2}単純にループを回すだけではすみません[。]{.chpule2}[
]{.chpuri2}`Product` と `Box` クラスを熟知し[、]{.chpule2}[
]{.chpuri2}箱の入れ子の度合いその他のどうでもいい細かいことに気を払う必要があります[。]{.chpule2}[
]{.chpuri2}こういう理由で[、]{.chpule2}[
]{.chpuri2}直接の解法は[、]{.chpule2}[
]{.chpuri2}とてもやりにくかったり[、]{.chpule2}[
]{.chpuri2}不可能かもしれません[。]{.chpule2}[ ]{.chpuri2}
:::

::: {.section .solution}
##  解決策 {#solution}

Composite パターンに従うと[、]{.chpule2}[
]{.chpuri2}合計価格を計算するためのメソッドが宣言された共通のインターフェースを通して
`Product` と `Box` をアクセスします[。]{.chpule2}[ ]{.chpuri2}

このメソッドの仕組みはどうなっているのでしょうか[？]{.chpule2}[
]{.chpuri2}製品の場合は[、]{.chpule2}[
]{.chpuri2}単に製品の価格を返すだけです[。]{.chpule2}[
]{.chpuri2}箱の場合は[、]{.chpule2}[
]{.chpuri2}箱に含まれている各項目ごとにその価格を尋ねてから[、]{.chpule2}[
]{.chpuri2}この箱の合計を返します[。]{.chpule2}[
]{.chpuri2}項目のいずれかが小さい箱の場合[、]{.chpule2}[
]{.chpuri2}その箱もその項目などを調べる[、]{.chpule2}[
]{.chpuri2}ということをすべての内容物の価格が計算されるまで繰り返します[。]{.chpule2}[
]{.chpuri2}箱は包装手数料のような追加料金を最終的な価格に加えることもできます[。]{.chpule2}[
]{.chpuri2}

<figure class="image">
<img
src="../../images/patterns/content/composite/composite-comic-1-ja475b.png?id=b4bb7878c59ccb8bfad583ad8fbfd637"
style="aspect-ratio: 2;"
srcset="/images/patterns/content/composite/composite-comic-1-ja.png?id=b4bb7878c59ccb8bfad583ad8fbfd637 600w,/images/patterns/content/composite/composite-comic-1-ja-1.5x.png?id=e1483a166d051d38b2dbfe048f6eeb76 900w,/images/patterns/content/composite/composite-comic-1-ja-2x.png?id=103e283a3587c52091cd8e6a5313cd14 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Composite パターンの提案する解決策" />
<figcaption><p>Composite パターンを使用して<span
class="chpule2">、</span><span class="chpuri2">
</span>ツリー構造のすべての内容物に対して再帰的にある振る舞いを実行できる<span
class="chpule2">。</span><span class="chpuri2"> </span></p></figcaption>
</figure>

このやり方の最大の利点は[、]{.chpule2}[
]{.chpuri2}ツリーを構成するオブジェクトの具象クラスを気にする必要がないことです[。]{.chpule2}[
]{.chpuri2}オブジェクトが単純な製品なのか[、]{.chpule2}[
]{.chpuri2}豪華な箱なのかを知る必要はありません[。]{.chpule2}[
]{.chpuri2}共通のインターフェースですべて同じものを扱うことができます[。]{.chpule2}[
]{.chpuri2}メソッドを呼び出すと[、]{.chpule2}[
]{.chpuri2}オブジェクト自身がリクエストをツリーの下方に渡します[。]{.chpule2}[
]{.chpuri2}
:::

::: {.section .analogy}
##  現実世界でのたとえ {#analogy}

<figure class="image">
<img
src="../../images/patterns/diagrams/composite/live-examplee724.png?id=548a7cec45b493af66e8bfe524a137d3"
style="aspect-ratio: 1.22;"
srcset="/images/patterns/diagrams/composite/live-example.png?id=548a7cec45b493af66e8bfe524a137d3 280w,/images/patterns/diagrams/composite/live-example-1.5x.png?id=2129195f4103a5a4d3440a086ce3dc87 420w,/images/patterns/diagrams/composite/live-example-2x.png?id=b555458f20fc30425ae6dada5da492af 560w"
sizes="(max-width: 720px) 100vw, 280px" loading="lazy" width="280"
alt="軍事組織例" />
<figcaption><p>軍事組織の例</p></figcaption>
</figure>

ほとんどの国の軍隊は[、]{.chpule2}[
]{.chpuri2}階層構造をしています[。]{.chpule2}[
]{.chpuri2}軍はいくつかの師団で構成されています[。]{.chpule2}[
]{.chpuri2}師団は旅団の集まりで[、]{.chpule2}[
]{.chpuri2}旅団は小隊の集合で[、]{.chpule2}[
]{.chpuri2}小隊は分隊で構成されています[。]{.chpule2}[
]{.chpuri2}最終的に[、]{.chpule2}[
]{.chpuri2}分隊は兵士たちで構成された小さい班に分けられます[。]{.chpule2}[
]{.chpuri2}命令は[、]{.chpule2}[
]{.chpuri2}すべての兵士が何をすべきかを知るまで[、]{.chpule2}[
]{.chpuri2}階層の上に与えられ[、]{.chpule2}[
]{.chpuri2}各レベルに引き渡されます[。]{.chpule2}[ ]{.chpuri2}
:::

::::: {.section .structure-container}
##  構造 {#structure}

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/composite/structure-jaa91c.png?id=88ff6efa0f4aeb685976bddd178fb927"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 0.82;"
srcset="/images/patterns/diagrams/composite/structure-ja.png?id=88ff6efa0f4aeb685976bddd178fb927 360w,/images/patterns/diagrams/composite/structure-ja-1.5x.png?id=0afcfe7ca22b21eeb995b5e5abf82cfe 540w,/images/patterns/diagrams/composite/structure-ja-2x.png?id=d3f57d0d8cf6008b6cf9b93032ea2855 720w"
sizes="(max-width: 720px) 100vw, 360px" loading="lazy" width="360"
alt="Composite デザインパターンの構造" /><img
src="../../images/patterns/diagrams/composite/structure-ja-indexed6ca6.png?id=16ad523ec80fff8cc4c215664877405b"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 0.87;"
srcset="/images/patterns/diagrams/composite/structure-ja-indexed.png?id=16ad523ec80fff8cc4c215664877405b 400w,/images/patterns/diagrams/composite/structure-ja-indexed-1.5x.png?id=aff0a2696b4a38ce01cb9be5b14a1ee0 600w,/images/patterns/diagrams/composite/structure-ja-indexed-2x.png?id=e2f3daa8f0ce32599760ce97801428f0 800w"
sizes="(max-width: 720px) 100vw, 400px" loading="lazy" width="400"
alt="Composite デザインパターンの構造" />
</figure>
:::

1.  **コンポーネント**[
    ]{.chpule1}[（]{.chpuri1}Component[）]{.chpule2}[
    ]{.chpuri2}インターフェースは[、]{.chpule2}[
    ]{.chpuri2}ツリーの単純な要素と複雑な要素の両方に共通する操作を記述します[。]{.chpule2}[
    ]{.chpuri2}

2.  **リーフ**[ ]{.chpule1}[（]{.chpuri1}Leaf[）]{.chpule2}[
    ]{.chpuri2}は[、]{.chpule2}[
    ]{.chpuri2}ツリーの基本要素で[、]{.chpule2}[
    ]{.chpuri2}子要素を持ちません[。]{.chpule2}[ ]{.chpuri2}

    リーフのコンポーネントは[、]{.chpule2}[
    ]{.chpuri2}仕事の移譲先がないため[、]{.chpule2}[
    ]{.chpuri2}通常[、]{.chpule2}[
    ]{.chpuri2}実際の作業のほとんどは[、]{.chpule2}[
    ]{.chpuri2}ここで行われることになります[。]{.chpule2}[ ]{.chpuri2}

3.  **コンテナ**[ ]{.chpule1}[（]{.chpuri1}Container[、]{.chpule2}[
    ]{.chpuri2}別名[：]{.chpule2}[ ]{.chpuri2}*Composite*[）]{.chpule2}[
    ]{.chpuri2}は[、]{.chpule2}[
    ]{.chpuri2}子要素を持った要素です[。]{.chpule2}[
    ]{.chpuri2}コンテナはその子要素の具象クラスが何なのかを知らず[、]{.chpule2}[
    ]{.chpuri2}コンポーネント・インターフェースを介してのみ[、]{.chpule2}[
    ]{.chpuri2}子要素とやりとりをします[。]{.chpule2}[ ]{.chpuri2}

    リクエストを受け取ると[、]{.chpule2}[
    ]{.chpuri2}コンテナはその作業を子要素に委任し[、]{.chpule2}[
    ]{.chpuri2}中間結果を処理し[、]{.chpule2}[
    ]{.chpuri2}最終結果をクライアントに返します[。]{.chpule2}[
    ]{.chpuri2}

4.  **クライアント**[ ]{.chpule1}[（]{.chpuri1}Client[）]{.chpule2}[
    ]{.chpuri2}は[、]{.chpule2}[
    ]{.chpuri2}コンポーネントインターフェースを介してすべての要素とやりとりします[。]{.chpule2}[
    ]{.chpuri2}その結果[、]{.chpule2}[
    ]{.chpuri2}クライアントは[、]{.chpule2}[
    ]{.chpuri2}ツリーの単純要素と複雑な要素の両方に対して同じように機能できます[。]{.chpule2}[
    ]{.chpuri2}
::::
:::::

::: {.section .pseudocode}
##  擬似コード {#pseudocode}

この例では[、]{.chpule2}[ ]{.chpuri2}**Composite**
パターンで[、]{.chpule2}[
]{.chpuri2}グラフィック・エディターでの幾何学形状の積み重ねの実装をします[。]{.chpule2}[
]{.chpuri2}

<figure class="image">
<img
src="../../images/patterns/diagrams/composite/example7ea7.png?id=98ba81d07c979038dd2ebf3c83a2e19f"
style="aspect-ratio: 0.84;"
srcset="/images/patterns/diagrams/composite/example.png?id=98ba81d07c979038dd2ebf3c83a2e19f 370w,/images/patterns/diagrams/composite/example-1.5x.png?id=ae008efd4146628cd7303925d4ce90be 555w,/images/patterns/diagrams/composite/example-2x.png?id=d21edef39d3792e8a4c6736727ac7305 740w"
sizes="(max-width: 720px) 100vw, 370px" loading="lazy" width="370"
alt="Composite 例の構造" />
<figcaption><p>幾何学形状エディターの例</p></figcaption>
</figure>

`Compound­Graphic`[
]{.chpule1}[（]{.chpuri1}複合グラフィック[）]{.chpule2}[
]{.chpuri2}クラスは[、]{.chpule2}[
]{.chpuri2}任意の数の子形状を含むコンテナです[。]{.chpule2}[
]{.chpuri2}子形状は[、]{.chpule2}[
]{.chpuri2}複合形状かもしれません[。]{.chpule2}[
]{.chpuri2}複合形状は[、]{.chpule2}[
]{.chpuri2}単純形状と同じメソッドを持っています[。]{.chpule2}[
]{.chpuri2}しかし[、]{.chpule2}[ ]{.chpuri2}複合形状は[、]{.chpule2}[
]{.chpuri2}自分自身で何かを行うのではなく[、]{.chpule2}[
]{.chpuri2}リクエストをすべての子要素に再帰的に渡し[、]{.chpule2}[
]{.chpuri2}結果を[ ]{.chpule1}[「]{.chpuri1}集計[」]{.chpule2}[
]{.chpuri2}します[。]{.chpule2}[ ]{.chpuri2}

クライアントは[、]{.chpule2}[
]{.chpuri2}すべての形状クラスに共通の単一のインターフェースを通じて[、]{.chpule2}[
]{.chpuri2}すべての形状とやりとりします[。]{.chpule2}[
]{.chpuri2}したがって[、]{.chpule2}[
]{.chpuri2}クライアントは[、]{.chpule2}[
]{.chpuri2}相手が単純形状なのか複合形状なのかを知りません[。]{.chpule2}[
]{.chpuri2}クライアントは[、]{.chpule2}[
]{.chpuri2}具体的なクラスと密に結合することなく[、]{.chpule2}[
]{.chpuri2}非常に複雑なオブジェクト構造を扱うことができます[。]{.chpule2}[
]{.chpuri2}

<figure class="code">
<pre class="code" lang="pseudocode"><code>// コンポーネント・インターフェースは、コンポジット中の単純なオブジェクト
// と複雑なオブジェクトの両方に共通する操作を宣言。
interface Graphic is
    method move(x, y)
    method draw()

// リーフ・クラスは、合成の末端のオブジェクトを表現する。リーフのオブジェ
// クトは、副オブジェクトを持てない。通常、本当の仕事をするのは、リーフ・
// オブジェクトで、コンポジット・オブジェクトは仕事を副オブジェクトに委任。
class Dot implements Graphic is
    field x, y

    constructor Dot(x, y) { …… }

    method move(x, y) is
        this.x += x, this.y += y

    method draw() is
        // X と Y に点を描画。

// すべてのコンポーネントクラスは他のコンポーネントを拡張可能。
class Circle extends Dot is
    field radius

    constructor Circle(x, y, radius) { …… }

    method draw() is
        // X と Y に半径 R の円を描画。

// コンポジット・クラスは子を持つかもしれない複雑なコンポーネントを表現す
// る。コンポジット・オブジェクトは通常実際の仕事を子に委任し、結果を「ま
// とめあげる」。
class CompoundGraphic implements Graphic is
    field children: array of Graphic

    // コンポジット・オブジェクトは、他のコンポーネント（単純、複雑どちら
    // の種類でも）を子リストに追加も削除もできる。
    method add(child: Graphic) is
        // 子配列に子を一つ追加。

    method remove(child: Graphic) is
        // 子配列から子を一つ削除。

    method move(x, y) is
        foreach (child in children) do
            child.move(x, y)

    // コンポジットは、特定の方法でその主要なロジックを実行。すべての子た
    // ちを再帰的に探索し、結果を収集し、まとめあげる。コンポジットの子要
    // 素が自身の子要素にこれらの呼び出しを行い、それがまたその子要素に、
    // と続くため、結果としてオブジェクト・ツリー全体が探索される。
    method draw() is
        // 1. 各子コンポーネントに対して：
        //     - コンポーネントを描画。
        //     - 境界長方形を更新。
        // 2. 境界座標を使用して破線の長方形を描画。


// クライアント・コードは、基底インターフェースを介してすべてのコンポーネ
// ントと動作。このようにしてクライアント・コードは単純なリーフ・コンポー
// ネントと複雑なコンポジットの両方をサポート可能。
class ImageEditor is
    field all: CompoundGraphic

    method load() is
        all = new CompoundGraphic()
        all.add(new Dot(1, 2))
        all.add(new Circle(5, 3, 10))
        // ……

    // 選択したコンポーネントを組み合わせて、一つの複雑なコンポジット・コ
    // ンポーネントにする。
    method groupSelected(components: array of Graphic) is
        group = new CompoundGraphic()
        foreach (component in components) do
            group.add(component)
            all.remove(component)
        all.add(group)
        // 全コンポーネントが描画される。
        all.draw()</code></pre>
</figure>
:::

:::::::: {.section .applicability-container}
##  適応性 {#applicability}

::::::: applicability
::: applicability-problem
ツリーのようなオブジェクト構造を実装する場合は[、]{.chpule2}[
]{.chpuri2}Composite パターンを使用します[。]{.chpule2}[ ]{.chpuri2}
:::

::: applicability-solution
Composite パターンは[、]{.chpule2}[
]{.chpuri2}単純なリーフと複雑なコンテナという[、]{.chpule2}[
]{.chpuri2}共通のインターフェースを持った二つの基本的な要素型からなっています[。]{.chpule2}[
]{.chpuri2}コンテナは[、]{.chpule2}[
]{.chpuri2}リーフと他のコンテナの両方から成り立っています[。]{.chpule2}[
]{.chpuri2}これにより[、]{.chpule2}[
]{.chpuri2}木に似た入れ子になった再帰オブジェクト構造を構築できます[。]{.chpule2}[
]{.chpuri2}
:::

::: applicability-problem
クライアント・コードが[、]{.chpule2}[
]{.chpuri2}単純な要素と複雑な要素を同等に扱えるようにするために[、]{.chpule2}[
]{.chpuri2}このパターンを適用してください[。]{.chpule2}[ ]{.chpuri2}
:::

::: applicability-solution
Composite パターンで定義されたすべての要素は[、]{.chpule2}[
]{.chpuri2}共通のインターフェースを共有します[。]{.chpule2}[
]{.chpuri2}このインターフェースを使用することにより[、]{.chpule2}[
]{.chpuri2}クライアントは扱うオブジェクトの具体的なクラスが何なのかを心配する必要がなくなります[。]{.chpule2}[
]{.chpuri2}
:::
:::::::
::::::::

::: {.section .checklist}
##  実装方法 {#checklist}

1.  アプリの中核のモデルが[、]{.chpule2}[
    ]{.chpuri2}ツリー構造で表現可能なことをまず確認してください[。]{.chpule2}[
    ]{.chpuri2}単純項目とコンテナに分けてみてください[。]{.chpule2}[
    ]{.chpuri2}コンテナは[、]{.chpule2}[
    ]{.chpuri2}単純要素と他のコンテナを含むことができる[、]{.chpule2}[
    ]{.chpuri2}ということをお忘れなく[。]{.chpule2}[ ]{.chpuri2}

2.  単純要素と複合要素の両方に意味のあるメソッドを並べたコンポーネント・インターフェースを宣言してください[。]{.chpule2}[
    ]{.chpuri2}

3.  単純要素を表現する[、]{.chpule2}[
    ]{.chpuri2}リーフ・クラスを作成します[。]{.chpule2}[
    ]{.chpuri2}一つのプログラムには[、]{.chpule2}[
    ]{.chpuri2}複数のリーフ・クラスがあるかもしれません[。]{.chpule2}[
    ]{.chpuri2}

4.  複雑な要素を表現する[、]{.chpule2}[
    ]{.chpuri2}コンテナ・クラスを作成します[。]{.chpule2}[
    ]{.chpuri2}このクラスには[、]{.chpule2}[
    ]{.chpuri2}子要素への参照をしまっておくための配列フィールドを設けます[。]{.chpule2}[
    ]{.chpuri2}配列は[、]{.chpule2}[
    ]{.chpuri2}リーフとコンテナの両方をしまうことが可能である必要があります[。]{.chpule2}[
    ]{.chpuri2}そのため[、]{.chpule2}[
    ]{.chpuri2}コンポーネント・インターフェースを使って宣言してください[。]{.chpule2}[
    ]{.chpuri2}

    コンポーネント・インターフェースのメソッド実装の際には[、]{.chpule2}[
    ]{.chpuri2}コンテナがほとんどの作業を子要素に委任するべきだということをお忘れなく[。]{.chpule2}[
    ]{.chpuri2}

5.  最後に[、]{.chpule2}[
    ]{.chpuri2}コンテナに子要素を追加したり削除するためのメソッドを定義してください[。]{.chpule2}[
    ]{.chpuri2}

    これらの操作は[、]{.chpule2}[
    ]{.chpuri2}コンポーネント・インターフェースで宣言することもできる[、]{.chpule2}[
    ]{.chpuri2}ということを心にとめておいてください[。]{.chpule2}[
    ]{.chpuri2}これは[、]{.chpule2}[
    ]{.chpuri2}リーフ・クラスではこれらのメソッドは空になるため[、]{.chpule2}[
    ]{.chpuri2}*[イ]{.cjk}[ン]{.cjk}[タ]{.cjk}[ー]{.cjk}[フ]{.cjk}[ェ]{.cjk}[ー]{.cjk}[ス]{.cjk}[分]{.cjk}[離]{.cjk}[の]{.cjk}[原]{.cjk}[則]{.cjk}*に違反します[。]{.chpule2}[
    ]{.chpuri2}しかし[、]{.chpule2}[
    ]{.chpuri2}そうすることで[、]{.chpule2}[
    ]{.chpuri2}ツリーの作成時に[、]{.chpule2}[
    ]{.chpuri2}クライアントは全要素を同等に扱えます[。]{.chpule2}[
    ]{.chpuri2}
:::

:::::: {.section .pros-cons}
##  長所と短所 {#pros-cons}

::::: row
::: col-sm-6
-    複雑なツリー構造をより便利に扱うことができます[。]{.chpule2}[
    ]{.chpuri2}多相性と再帰を活用[。]{.chpule2}[ ]{.chpuri2}
-   
    *[開]{.cjk}[放]{.cjk}[閉]{.cjk}[鎖]{.cjk}[の]{.cjk}[原]{.cjk}[則]{.cjk}*[。]{.chpule2}[
    ]{.chpuri2}既存のコードを壊すことなく[、]{.chpule2}[
    ]{.chpuri2}オブジェクト・ツリーと動作可能な新規の要素型をアプリに導入可能[。]{.chpule2}[
    ]{.chpuri2}
:::

::: col-sm-6
-   
    機能が大きく異なるクラスの共通インターフェース作成は困難である可能性[。]{.chpule2}[
    ]{.chpuri2}特定の状況下では[、]{.chpule2}[
    ]{.chpuri2}コンポーネント・インターフェースの過度な一般化が必要で[、]{.chpule2}[
    ]{.chpuri2}理解困難[。]{.chpule2}[ ]{.chpuri2}
:::
:::::
::::::

::: {.section .relations}
##  他のパターンとの関係 {#relations}

-   [Builder](builder.html) は[、]{.chpule2}[ ]{.chpuri2}複雑な
    [Composite](composite.html) ツリー作成に使用できます[。]{.chpule2}[
    ]{.chpuri2}構築ステップを再帰的に行なうように
    プログラムします[。]{.chpule2}[ ]{.chpuri2}

-   [Chain of Responsibility](chain-of-responsibility.html)
    は[、]{.chpule2}[ ]{.chpuri2}よく [Composite](composite.html)
    と一緒に使われます[。]{.chpule2}[ ]{.chpuri2}この場合[、]{.chpule2}[
    ]{.chpuri2}リーフ[ ]{.chpule1}[（]{.chpuri1}末端[）]{.chpule2}[
    ]{.chpuri2}のコンポーネントがリクエストを受ける時[、]{.chpule2}[
    ]{.chpuri2}リクエストは[、]{.chpule2}[
    ]{.chpuri2}全部の親コンポーネントからオブジェクト・ツリーのルート[
    ]{.chpule1}[（]{.chpuri1}根[）]{.chpule2}[
    ]{.chpuri2}までを通るかもしれません[。]{.chpule2}[ ]{.chpuri2}

-   [Iterators](iterator.html) を使用して [Composite](composite.html)
    ツリーを探索することができます[。]{.chpule2}[ ]{.chpuri2}

-   [Visitor](visitor.html) を使用して[、]{.chpule2}[
    ]{.chpuri2}[Composite](composite.html)
    ツリー全体に対して一つの操作を実行できます[。]{.chpule2}[
    ]{.chpuri2}

-   RAM を節約するために[、]{.chpule2}[
    ]{.chpuri2}[Composite](composite.html) ツリーの共有リーフ・ノードを
    [Flyweights](flyweight.html) として実装できます[。]{.chpule2}[
    ]{.chpuri2}

-   [Composite](composite.html) と [Decorator](decorator.html)
    は両方とも[、]{.chpule2}[
    ]{.chpuri2}任意の数のオブジェクトを組織するために再起的合成を使用するので[、]{.chpule2}[
    ]{.chpuri2}似たような構造図をしています[。]{.chpule2}[ ]{.chpuri2}

    *Decorator* は[、]{.chpule2}[ ]{.chpuri2}*Composite*
    に似ていますが[、]{.chpule2}[
    ]{.chpuri2}子コンポーネントは一つしかありません[。]{.chpule2}[
    ]{.chpuri2}もう一つの大きな違いは[、]{.chpule2}[
    ]{.chpuri2}*Decorator*
    は内包するオブジェクトに責任を追加するのに対し[、]{.chpule2}[
    ]{.chpuri2}*Composite* は[、]{.chpule2}[
    ]{.chpuri2}単にその子たちの結果を[
    ]{.chpule1}[「]{.chpuri1}まとめあげる[」]{.chpule2}[
    ]{.chpuri2}だけです[。]{.chpule2}[ ]{.chpuri2}

    しかしながら[、]{.chpule2}[
    ]{.chpuri2}これらのパターンは互いに協力できます[。]{.chpule2}[
    ]{.chpuri2}*Decorator* を使用して[、]{.chpule2}[
    ]{.chpuri2}*Composite*
    ツリー内の特定のオブジェクトの振る舞いを拡張できます[。]{.chpule2}[
    ]{.chpuri2}

-   [Composite](composite.html) と [Decorator](decorator.html)
    を多用する設計に対しては[、]{.chpule2}[
    ]{.chpuri2}[Prototype](prototype.html)
    の使用が有益かもしれません[。]{.chpule2}[
    ]{.chpuri2}このパターンを適用すると[、]{.chpule2}[
    ]{.chpuri2}複雑な構造を初めから再構築するのではなく[、]{.chpule2}[
    ]{.chpuri2}それをクローンします[。]{.chpule2}[ ]{.chpuri2}
:::

::: {.section .implementations}
##  コード例 {#implementations}

[![Composite を C#
で](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](composite/csharp/example.html "Composite を C# で"){.prog-lang-link}
[![Composite を C++
で](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](composite/cpp/example.html "Composite を C++ で"){.prog-lang-link}
[![Composite を Go
で](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](composite/go/example.html "Composite を Go で"){.prog-lang-link}
[![Composite を Java
で](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](composite/java/example.html "Composite を Java で"){.prog-lang-link}
[![Composite を PHP
で](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](composite/php/example.html "Composite を PHP で"){.prog-lang-link}
[![Composite を Python
で](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](composite/python/example.html "Composite を Python で"){.prog-lang-link}
[![Composite を Ruby
で](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](composite/ruby/example.html "Composite を Ruby で"){.prog-lang-link}
[![Composite を Rust
で](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](composite/rust/example.html "Composite を Rust で"){.prog-lang-link}
[![Composite を Swift
で](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](composite/swift/example.html "Composite を Swift で"){.prog-lang-link}
[![Composite を TypeScript
で](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](composite/typescript/example.html "Composite を TypeScript で"){.prog-lang-link}
:::

:::::: {#book-promo .banner-set2}
::::: {.prom .banner-content .banner-bg .banner-striped .banner-with-image data-id="DP: 1: Support our free website and own the eBook!" creative-id="standard-ja" position="content_bottom"}
::: banner-image
[![](../../images/patterns/banners/patterns-book-banner-3a29e.png?id=7d445df13c80287beaab234b4f3b698c){width="200"
height="200" loading="lazy"
srcset="/images/patterns/banners/patterns-book-banner-3-2x.png?id=0cc3f77ab421d1a5c02ee46488231c3a 2x"}](book.html)
:::

::: banner-text
### 無料サイトをサポートして電子書籍をゲット！ {#無料サイトをサポートして電子書籍をゲット .title}

-   22 種類のデザインパターンと 8 つの原則を詳しく説明。
-   よく構成されていて読みやすい、専門用語なしの 409 ページ。
-   225 点の明確で役に立つイラストと図表。
-   11 言語によるコード例の書庫。
-   全デバイスをサポート：PDF、EPUB、MOBI、KFX フォーマットに対応

[ 詳細](book.html){.btn .btn-secondary}
:::
:::::
::::::

::: next
#### 次を読む

[Decorator []{.fa .fa-arrow-right}](decorator.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### 戻る

[[]{.fa .fa-arrow-left} Bridge](bridge.html){.btn .btn-default
rel="prev"}
:::
::::::::::::::::::::::::::::::::

::::::: {.prom .banner-sidebar .banner-removable .banner-removable-patterns data-id="DP: Sidebar" creative-id="standard-sidebar-ja" position="sidebar"}
:::::: banner-inner
:::: image3d-book-right
::: {.image3d-cover style="background: #0b3752;"}
[![](../../images/patterns/book/web-cover-ja1087.png?id=0465cbe2ad4c821aa572197897cadd4f){width="250"
height="375" loading="lazy"
srcset="/images/patterns/book/web-cover-ja-2x.png?id=6a037d9111395efa95d119723d7ae98b 2x"}](book.html)
:::
::::

::: {style="margin-top: 1rem"}
この記事は、電子書籍\
『**直撃！デザインパターン**』の一部です。

[ 詳細](book.html){.btn .btn-secondary .btn-block}
:::
::::::
:::::::
::::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::::
