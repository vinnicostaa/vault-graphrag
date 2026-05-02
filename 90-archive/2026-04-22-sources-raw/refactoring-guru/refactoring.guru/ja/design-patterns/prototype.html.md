:::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} /
[デザインパターン](../design-patterns.html){.type} /
[生成に関するパターン](creational-patterns.html){.type}
:::

# Prototype {#prototype .title}

::: aka
別名：[Clone、]{style="display:inline-block"}[プロトタイプ、]{style="display:inline-block"}[クローン]{style="display:inline-block"}
:::

::: {.section .intent}
##  一言でいうと {#intent}

**Prototype**[ ]{.chpule1}[（]{.chpuri1}プロトタイプ[、]{.chpule2}[
]{.chpuri2}原型[）]{.chpule2}[ ]{.chpuri2}は[、]{.chpule2}[
]{.chpuri2}既存オブジェクトのコピーをそのクラスに依存することなく可能とする[、]{.chpule2}[
]{.chpuri2}生成に関するデザインパターンの一つです[。]{.chpule2}[
]{.chpuri2}

<figure class="image">
<img
src="../../images/patterns/content/prototype/prototype3cb9.png?id=e912b1ada20bbf7b2ffc09e93b9fab20"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/prototype/prototype.png?id=e912b1ada20bbf7b2ffc09e93b9fab20 640w,/images/patterns/content/prototype/prototype-1.5x.png?id=a0f76894fb657783b65b9ed899857468 960w,/images/patterns/content/prototype/prototype-2x.png?id=670789c80c8a114e25838ede2da4a881 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Prototype デザインパターン" />
</figure>
:::

::: {.section .problem}
##  問題 {#problem}

ここにオブジェクトがあって[、]{.chpule2}[
]{.chpuri2}この正確なコピーを作成したいとします[。]{.chpule2}[
]{.chpuri2}どうすればいいでしょう[？]{.chpule2}[
]{.chpuri2}まず[、]{.chpule2}[
]{.chpuri2}同じクラスの新規オブジェクトを作成する必要があります[。]{.chpule2}[
]{.chpuri2}次に[、]{.chpule2}[
]{.chpuri2}元のオブジェクトのすべてのフィールドの値を一つずつ新しいオブジェクトにコピーしていきます[。]{.chpule2}[
]{.chpuri2}

よさそうですね[！]{.chpule2}[
]{.chpuri2}でも落とし穴があります[。]{.chpule2}[
]{.chpuri2}どんなオブジェクトでもこの方法でコピーできるわけではないのです[。]{.chpule2}[
]{.chpuri2}オブジェクトのフィールドの一部は `private`[
]{.chpule1}[（]{.chpuri1}非公開[）]{.chpule2}[
]{.chpuri2}で[、]{.chpule2}[
]{.chpuri2}オブジェクトの外からは見えないようになっているかもしれません[。]{.chpule2}[
]{.chpuri2}

<figure class="image">
<img
src="../../images/patterns/content/prototype/prototype-comic-1-ja48c4.png?id=794900eaaa259e800e1b4f143af2e2c0"
style="aspect-ratio: 2;"
srcset="/images/patterns/content/prototype/prototype-comic-1-ja.png?id=794900eaaa259e800e1b4f143af2e2c0 600w,/images/patterns/content/prototype/prototype-comic-1-ja-1.5x.png?id=ccd9fef984c927999bf0e1fc87b00cf4 900w,/images/patterns/content/prototype/prototype-comic-1-ja-2x.png?id=fb50ca8e74b41f7ffb3828f0e59f4d11 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="「外部で」コビーしてうまく行かない場合" />
<figcaption><p>外部からオブジェクトをコピーすることは常に<a
href="https://vimeo.com/120816425">可能とは限りません</a><span
class="chpule2">。</span><span class="chpuri2"> </span></p></figcaption>
</figure>

この直接的なやり方には[、]{.chpule2}[
]{.chpuri2}もう一つ問題があります[。]{.chpule2}[
]{.chpuri2}複製を作成するためには[、]{.chpule2}[
]{.chpuri2}オブジェクトのクラスについてよく知っている必要があるので[、]{.chpule2}[
]{.chpuri2}コードはそのクラスに依存するようになります[。]{.chpule2}[
]{.chpuri2}余分な依存関係なんてへっちゃらという方には[、]{.chpule2}[
]{.chpuri2}もう一つ別の落とし穴があります[。]{.chpule2}[
]{.chpuri2}オブジェクトが従うインターフェースはわかっているが[、]{.chpule2}[
]{.chpuri2}具象クラスは知らない[、]{.chpule2}[
]{.chpuri2}という場合があります[。]{.chpule2}[
]{.chpuri2}たとえば[、]{.chpule2}[
]{.chpuri2}あるメソッドは[、]{.chpule2}[
]{.chpuri2}パラメーターとして[、]{.chpule2}[
]{.chpuri2}あるインターフェースに従うどんなオブジェクトでも受け取るといった場合です[。]{.chpule2}[
]{.chpuri2}
:::

::: {.section .solution}
##  解決策 {#solution}

このパターンでは[、]{.chpule2}[ ]{.chpuri2}クローン[
]{.chpule1}[（]{.chpuri1}複製[）]{.chpule2}[
]{.chpuri2}される実際のオブジェクトにクローン作成の作業が任されます[。]{.chpule2}[
]{.chpuri2}クローンをサポートするすべてのオブジェクトに対する共通インターフェースを宣言します[。]{.chpule2}[
]{.chpuri2}このインターフェースを使用すると[、]{.chpule2}[
]{.chpuri2}コードをそのオブジェクトのクラスに密に結合せずにオブジェクトのクローン作成ができます[。]{.chpule2}[
]{.chpuri2}通常[、]{.chpule2}[
]{.chpuri2}この手のインターフェースは[、]{.chpule2}[ ]{.chpuri2}`clone`
メソッド一つだけからなっています[。]{.chpule2}[ ]{.chpuri2}

`clone` メソッドの実装はすべてのクラスで非常に似ています[。]{.chpule2}[
]{.chpuri2}このメソッドは現在のクラスのオブジェクトを作成し[、]{.chpule2}[
]{.chpuri2}古いオブジェクトのすべてのフィールド値を新しいクラスに引き継ぎます[。]{.chpule2}[
]{.chpuri2}ほとんどのプログラミング言語では[、]{.chpule2}[
]{.chpuri2}同じクラスに属するオブジェクトは[、]{.chpule2}[
]{.chpuri2}他のオブジェクトの非公開フィールドにアクセスできるため[、]{.chpule2}[
]{.chpuri2}非公開フィールドをコピーすることもできます[。]{.chpule2}[
]{.chpuri2}

クローン作成をサポートするオブジェクトは[、]{.chpule2}[
]{.chpuri2}*[プ]{.cjk}[ロ]{.cjk}[ト]{.cjk}[タ]{.cjk}[イ]{.cjk}[プ]{.cjk}*と呼ばれます[。]{.chpule2}[
]{.chpuri2}オブジェクトに数十のフィールドと数百の可能な構成がある場合[、]{.chpule2}[
]{.chpuri2}サブクラスを作る代わりにクローン作成が有効な代替手段となるかもしれません[。]{.chpule2}[
]{.chpuri2}

<figure class="image">
<img
src="../../images/patterns/content/prototype/prototype-comic-2-ja3ead.png?id=6687d7d5c44d3ff7811c379d7628acf0"
style="aspect-ratio: 1.14;"
srcset="/images/patterns/content/prototype/prototype-comic-2-ja.png?id=6687d7d5c44d3ff7811c379d7628acf0 343w,/images/patterns/content/prototype/prototype-comic-2-ja-1.5x.png?id=aabc80ac75ed3d6f24ca1e21b90988a1 515w,/images/patterns/content/prototype/prototype-comic-2-ja-2x.png?id=117077b1169daa7dd1d34de3ed49ad55 687w"
sizes="(max-width: 720px) 100vw, 343px" loading="lazy" width="343"
alt="構築済みプロトタイプ" />
<figcaption><p>事前に構築されたプロトタイプはサブクラス化の代替手段<span
class="chpule2">。</span><span class="chpuri2"> </span></p></figcaption>
</figure>

仕組み[：]{.chpule2}[
]{.chpuri2}様々な構成のオブジェクトをあらかじめ作成しておきます[。]{.chpule2}[
]{.chpuri2}これら構成ずみオブジェクトの一つに似たオブジェクトが必要な場合[、]{.chpule2}[
]{.chpuri2}新しいオブジェクトを一から構築する代わりにプロトタイプをクローンします[。]{.chpule2}[
]{.chpuri2}
:::

::: {.section .analogy}
##  現実世界でのたとえ {#analogy}

現実世界では[、]{.chpule2}[
]{.chpuri2}製品の大量生産を開始する前に[、]{.chpule2}[
]{.chpuri2}様々なテストを行うためにプロトタイプが使用されます[。]{.chpule2}[
]{.chpuri2}しかし[、]{.chpule2}[ ]{.chpuri2}この場合[、]{.chpule2}[
]{.chpuri2}プロトタイプは実際の生産には使われません[。]{.chpule2}[
]{.chpuri2}受動的な役割を果たすだけです[。]{.chpule2}[ ]{.chpuri2}

<figure class="image">
<img
src="../../images/patterns/content/prototype/prototype-comic-3-ja0a50.png?id=a0a7129addeeb0294d43826ffa493102"
style="aspect-ratio: 2;"
srcset="/images/patterns/content/prototype/prototype-comic-3-ja.png?id=a0a7129addeeb0294d43826ffa493102 600w,/images/patterns/content/prototype/prototype-comic-3-ja-1.5x.png?id=5e90e65436e1b8068dfa842260593373 900w,/images/patterns/content/prototype/prototype-comic-3-ja-2x.png?id=a00b44f85da402bc6d7e49a175676120 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="細胞分裂" />
<figcaption aria-hidden="true"><p>細胞分裂</p></figcaption>
</figure>

工業用プロトタイプは本当に自分自身をコピーしないので[、]{.chpule2}[
]{.chpuri2}パターンに非常に近い類似は[、]{.chpule2}[
]{.chpuri2}細胞分裂[ ]{.chpule1}[（]{.chpuri1}生物学[、]{.chpule2}[
]{.chpuri2}覚えていますか?[）]{.chpule2}[
]{.chpuri2}のプロセスです[。]{.chpule2}[
]{.chpuri2}分裂後[、]{.chpule2}[
]{.chpuri2}同一細胞のペアが形成されます[。]{.chpule2}[
]{.chpuri2}元のセルはプロトタイプとして機能し[、]{.chpule2}[
]{.chpuri2}コピーの作成に積極的な役割を果たします[。]{.chpule2}[
]{.chpuri2}
:::

:::::::: {.section .structure-container}
##  構造 {#structure}

::::::: structure
#### 基本的な実装

:::: prototype-normal
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/prototype/structure87f6.png?id=088102c5e9785ff45debbbce86f4df81"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.25;"
srcset="/images/patterns/diagrams/prototype/structure.png?id=088102c5e9785ff45debbbce86f4df81 500w,/images/patterns/diagrams/prototype/structure-1.5x.png?id=beec6a1a5242268e10e39f9fdc0b894b 750w,/images/patterns/diagrams/prototype/structure-2x.png?id=ba75079f42f08028ae4cdbda0cfecc26 1000w"
sizes="(max-width: 720px) 100vw, 500px" loading="lazy" width="500"
alt="Prototype デザインパターンの構造" /><img
src="../../images/patterns/diagrams/prototype/structure-indexed6499.png?id=0e1c809842f5c43aca0541a2eba1f844"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.27;"
srcset="/images/patterns/diagrams/prototype/structure-indexed.png?id=0e1c809842f5c43aca0541a2eba1f844 520w,/images/patterns/diagrams/prototype/structure-indexed-1.5x.png?id=05b072b5b83f5ff1024a7ba448ea9d59 780w,/images/patterns/diagrams/prototype/structure-indexed-2x.png?id=74584ac729c0f6b49d2a97a53cc266ff 1040w"
sizes="(max-width: 720px) 100vw, 520px" loading="lazy" width="520"
alt="Prototype デザインパターンの構造" />
</figure>
:::
::::

1.  **プロトタイプ**[ ]{.chpule1}[（]{.chpuri1}Prototype[）]{.chpule2}[
    ]{.chpuri2}インターフェースはクローン作成メソッドを宣言します[。]{.chpule2}[
    ]{.chpuri2}ほとんどの場合[、]{.chpule2}[ ]{.chpuri2}ただ一つの
    `clone` メソッドからできています[。]{.chpule2}[ ]{.chpuri2}

2.  **具象プロトタイプ**[ ]{.chpule1}[（]{.chpuri1}Concrete
    Prototype[）]{.chpule2}[ ]{.chpuri2}のクラスは[、]{.chpule2}[
    ]{.chpuri2}クローン作成メソッドを実装します[。]{.chpule2}[
    ]{.chpuri2}元のオブジェクトのデータをコピーするだけではなく[、]{.chpule2}[
    ]{.chpuri2}このメソッドはクローン作成時にクローン同士をリンクするとか[、]{.chpule2}[
    ]{.chpuri2}再起的依存性を取り払うなどの作業も行うかもしれません[。]{.chpule2}[
    ]{.chpuri2}

3.  **クライアント**[ ]{.chpule1}[（]{.chpuri1}Client[）]{.chpule2}[
    ]{.chpuri2}は[、]{.chpule2}[
    ]{.chpuri2}プロトタイプのインターフェースに従うどんなオブジェクトでも[、]{.chpule2}[
    ]{.chpuri2}そのコピーを生成できます[。]{.chpule2}[ ]{.chpuri2}

#### プロトタイプ・レジストリーの実装

:::: prototype-pool
::: {.struct-image2 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/prototype/structure-prototype-cache9d19.png?id=609c2af5d14ed55dcbb218a00f98e7d5"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.15;"
srcset="/images/patterns/diagrams/prototype/structure-prototype-cache.png?id=609c2af5d14ed55dcbb218a00f98e7d5 550w,/images/patterns/diagrams/prototype/structure-prototype-cache-1.5x.png?id=8ca0b829185d49c5acbe19966a0659d4 825w,/images/patterns/diagrams/prototype/structure-prototype-cache-2x.png?id=a1e4514bbcc9b10968b856f19b407105 1100w"
sizes="(max-width: 720px) 100vw, 550px" loading="lazy" width="550"
alt="プロトタイプ・レジストリー" /><img
src="../../images/patterns/diagrams/prototype/structure-prototype-cache-indexed1801.png?id=10a4a84a1a318f59dbc2b806fc936d04"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.12;"
srcset="/images/patterns/diagrams/prototype/structure-prototype-cache-indexed.png?id=10a4a84a1a318f59dbc2b806fc936d04 550w,/images/patterns/diagrams/prototype/structure-prototype-cache-indexed-1.5x.png?id=cb56c95533a4020368c48db9f9e2a37d 825w,/images/patterns/diagrams/prototype/structure-prototype-cache-indexed-2x.png?id=47b99eb7ae51158bdbb31deea4f5e98f 1100w"
sizes="(max-width: 720px) 100vw, 550px" loading="lazy" width="550"
alt="プロトタイプ・レジストリー" />
</figure>
:::
::::

1.  **プロトタイプ・レジストリー**[ ]{.chpule1}[（]{.chpuri1}Prototype
    Registry[）]{.chpule2}[ ]{.chpuri2}により[、]{.chpule2}[
    ]{.chpuri2}よく使われるプロトタイプに簡単にアクセスできます[。]{.chpule2}[
    ]{.chpuri2}そこには[、]{.chpule2}[
    ]{.chpuri2}いつでも複製可能な状態の構築ずみオブジェクトが蓄えられています[。]{.chpule2}[
    ]{.chpuri2}一番簡単な実装方法は[、]{.chpule2}[
    ]{.chpuri2}`名前 → プロトタイプ`
    のハッシュマップを使うことです[。]{.chpule2}[
    ]{.chpuri2}しかし[、]{.chpule2}[
    ]{.chpuri2}単に名前に頼る以上の検索方式が必要な場合は[、]{.chpule2}[
    ]{.chpuri2}もっと堅牢なレジストリーを作ることも可能です[。]{.chpule2}[
    ]{.chpuri2}
:::::::
::::::::

::: {.section .pseudocode}
##  擬似コード {#pseudocode}

この例では[、]{.chpule2}[ ]{.chpuri2}**Prototype**
パターンを使用して[、]{.chpule2}[
]{.chpuri2}コードをクラスに密に結合することなく[、]{.chpule2}[
]{.chpuri2}幾何学的なオブジェクトの正確なコピーを生成します[。]{.chpule2}[
]{.chpuri2}

<figure class="image">
<img
src="../../images/patterns/diagrams/prototype/examplee491.png?id=47bc6c1058cb100b81e675b5ca6bda6c"
style="aspect-ratio: 1.42;"
srcset="/images/patterns/diagrams/prototype/example.png?id=47bc6c1058cb100b81e675b5ca6bda6c 470w,/images/patterns/diagrams/prototype/example-1.5x.png?id=067f3a2415370fadf5f92aadaa12b5e2 705w,/images/patterns/diagrams/prototype/example-2x.png?id=80393e0df17ae8130e5ada832d494949 940w"
sizes="(max-width: 720px) 100vw, 470px" loading="lazy" width="470"
alt="Prototype パターン例の構造" />
<figcaption><p>クラス階層に属するオブジェクトの集合をクローンする<span
class="chpule2">。</span><span class="chpuri2"> </span></p></figcaption>
</figure>

すべての形状クラスは[、]{.chpule2}[
]{.chpuri2}クローン作成メソッドを持つ同一インターフェースに従っています[。]{.chpule2}[
]{.chpuri2}サブクラスは[、]{.chpule2}[
]{.chpuri2}サブクラス特有のフィールド値を結果のオブジェクトにコピーする前に[、]{.chpule2}[
]{.chpuri2}親のクローン作成メソッドを呼び出すこともできます[。]{.chpule2}[
]{.chpuri2}

<figure class="code">
<pre class="code" lang="pseudocode"><code>// プロトタイプの基底クラス。
abstract class Shape is
    field X: int
    field Y: int
    field color: string

    // 通常のコンストラクター。
    constructor Shape() is
        // ……

    // プロトタイプのコンストラクター。新鮮なオブジェクトは、既存のオブ
    // ジェクトの値を使って初期化される。
    constructor Shape(source: Shape) is
        this()
        this.X = source.X
        this.Y = source.Y
        this.color = source.color

    // クローン操作は、Shape のサブクラスのいずれかを返す。
    abstract method clone():Shape


// 具象プロトタイプ。クローン作成メソッドは、現クラスのコンストラクターを
// 呼び、引数として現オブジェクトを渡すことにより新規オブジェクトを一度に
// 作成。実際のコピー作業をコンストラクターの中で行い、結果の一貫性を保証。
// コンストラクターは完全なオブジェクトが完成するまで帰らないので、他の
// オブジェクトが部分的に作成されたクローンへの参照を持つことは不可能。
class Rectangle extends Shape is
    field width: int
    field height: int

    constructor Rectangle(source: Rectangle) is
        // 親クラスで定義された非公開フィールドのコピーを行うため、親のコ
        // ンストラクターを呼ぶことは必須。
        super(source)
        this.width = source.width
        this.height = source.height

    method clone():Shape is
        return new Rectangle(this)


class Circle extends Shape is
    field radius: int

    constructor Circle(source: Circle) is
        super(source)
        this.radius = source.radius

    method clone():Shape is
        return new Circle(this)


// クライアント・コードのどこか。
class Application is
    field shapes: array of Shape

    constructor Application() is
        Circle circle = new Circle()
        circle.X = 10
        circle.Y = 10
        circle.radius = 20
        shapes.add(circle)

        Circle anotherCircle = circle.clone()
        shapes.add(anotherCircle)
        // anotherCircle 変数は、circle オブジェクトの完全なコピーを含
        // む。

        Rectangle rectangle = new Rectangle()
        rectangle.width = 10
        rectangle.height = 20
        shapes.add(rectangle)

    method businessLogic() is
        // Prototype はすごい！ここでは、オブジェクトのコピーの作成をそ
        // の型について一切知らずに行う。
        Array shapesCopy = new Array of Shapes.

        // 我々は、shapes 配列内の要素が厳密に何であるかを知らない。わ
        // かっているのは、どれも形状だということだけ。しかし、多相性のお
        // かげで、形状の一つに対して clone メソッドを呼ぶと、プログラム
        // がその本当のクラスが何かをチェックし、そのクラスで定義された適
        // 切な clone メソッドを実行する。一連の単純な Shape オブジェク
        // トではなく、適切なクローンを取得可能なのはこのため。
        foreach (s in shapes) do
            shapesCopy.add(s.clone())

        // shapesCopy 配列には、shape 配列の子要素が正確にコピーされる。</code></pre>
</figure>
:::

:::::::: {.section .applicability-container}
##  適応性 {#applicability}

::::::: applicability
::: applicability-problem
自分のコードがコピー対象オブジェクトの具象クラスに依存したくない場合に[、]{.chpule2}[
]{.chpuri2}Prototype パターンを使用します[。]{.chpule2}[ ]{.chpuri2}
:::

::: applicability-solution
この状況は[、]{.chpule2}[
]{.chpuri2}自分のコードが何らかのインターフェースを介して渡された外部開発コードからのオブジェクトと動作する場合によく発生します[。]{.chpule2}[
]{.chpuri2}これらのオブジェクトの具象クラスは不明であり[、]{.chpule2}[
]{.chpuri2}仮に望んでもそれに依存することはできません[。]{.chpule2}[
]{.chpuri2}

Prototype パターンを適応すると[、]{.chpule2}[
]{.chpuri2}クライアントのコードは[、]{.chpule2}[
]{.chpuri2}クローンをサポートするすべてのオブジェクトを扱う一般的なインターフェースを使います[。]{.chpule2}[
]{.chpuri2}このインターフェースにより[、]{.chpule2}[
]{.chpuri2}クライアントのコードは[、]{.chpule2}[
]{.chpuri2}クローン対象オブジェクトの具象クラスから独立したものとなります[。]{.chpule2}[
]{.chpuri2}
:::

::: applicability-problem
オブジェクトの初期化方法のみが異なるサブクラスの数を減らしたい場合に[、]{.chpule2}[
]{.chpuri2}このパターンを使用します[。]{.chpule2}[ ]{.chpuri2}
:::

::: applicability-solution
使用する前に手の込んだ設定を必要とする複雑なクラスがあるとします[。]{.chpule2}[
]{.chpuri2}このクラスを設定するには[、]{.chpule2}[
]{.chpuri2}いくつかの共通する方法がありますが[、]{.chpule2}[
]{.chpuri2}それはアプリのあちこちに散らばっています[。]{.chpule2}[
]{.chpuri2}重複を減らすために[、]{.chpule2}[
]{.chpuri2}複数のサブクラスを作成し[、]{.chpule2}[
]{.chpuri2}共通設定コードをサブクラスのコンストラクターに入れます[。]{.chpule2}[
]{.chpuri2}重複問題を解決しましたが[、]{.chpule2}[
]{.chpuri2}今度はダミーのサブクラスがたくさんできました[。]{.chpule2}[
]{.chpuri2}

Prototype パターンでは[、]{.chpule2}[
]{.chpuri2}いろいろな方法で事前構築したオブジェクトを使うことができます[。]{.chpule2}[
]{.chpuri2}何かの構成に一致するサブクラスをインスタンス化する代わりに[、]{.chpule2}[
]{.chpuri2}クライアントは適切なプロトタイプを探してクローンするということができます[。]{.chpule2}[
]{.chpuri2}
:::
:::::::
::::::::

::: {.section .checklist}
##  実装方法 {#checklist}

1.  プロトタイプのインターフェースを作成し[、]{.chpule2}[
    ]{.chpuri2}そこで `clone` メソッドを宣言します[。]{.chpule2}[
    ]{.chpuri2}または[、]{.chpule2}[
    ]{.chpuri2}既存のクラス階層がある場合は[、]{.chpule2}[
    ]{.chpuri2}階層下のすべてのクラスにメソッドを追加するだけでもかまいません[。]{.chpule2}[
    ]{.chpuri2}

2.  プロトタイプ・クラスは[、]{.chpule2}[
    ]{.chpuri2}そのクラスのオブジェクトを引数として受け取る代替コンストラクターを定義しなければなりません[。]{.chpule2}[
    ]{.chpuri2}コンストラクターは[、]{.chpule2}[
    ]{.chpuri2}渡されたオブジェクトから新しく作成されたインスタンスにクラスで定義されたすべてのフィールドの値をコピーする必要があります[。]{.chpule2}[
    ]{.chpuri2}サブクラスを変更する場合は[、]{.chpule2}[
    ]{.chpuri2}スーパークラスが非公開フィールドのクローン作成処理できるように親コンストラクターを呼び出す必要があります[。]{.chpule2}[
    ]{.chpuri2}

    お使いのプログラミング言語がメソッドの多重定義をサポートしていない場合は[、]{.chpule2}[
    ]{.chpuri2}オブジェクトのデータを複製するための特別なメソッドを定義することになります[。]{.chpule2}[
    ]{.chpuri2}コンストラクターは[、]{.chpule2}[ ]{.chpuri2}`new`
    演算子を呼び出した直後に生成されたオブジェクトを使えるため[、]{.chpule2}[
    ]{.chpuri2}これを行うためのより便利な場所です[。]{.chpule2}[
    ]{.chpuri2}

3.  クローン作成メソッドは通常[、]{.chpule2}[
    ]{.chpuri2}プロトタイプのクラスの `new`
    演算子を呼び出すだけの一行だけです[。]{.chpule2}[
    ]{.chpuri2}どのクラスもクローン作成メソッドを[、]{.chpule2}[
    ]{.chpuri2}`new`
    に続けてそのクラス名が来るように上書きしなければなりません[。]{.chpule2}[
    ]{.chpuri2}そうしないと[、]{.chpule2}[
    ]{.chpuri2}親クラスのオブジェクトがクローンされてしまいます[。]{.chpule2}[
    ]{.chpuri2}

4.  必要ならば[、]{.chpule2}[
    ]{.chpuri2}頻繁に使用されるプロトタイプのカタログを備蓄しておく[、]{.chpule2}[
    ]{.chpuri2}集中型プロトタイプ・レジストリーを作成します[。]{.chpule2}[
    ]{.chpuri2}

    レジストリーは[、]{.chpule2}[
    ]{.chpuri2}新しいファクトリー・クラスとして実装することもできますし[、]{.chpule2}[
    ]{.chpuri2}またはプロトタイプを取得する静的メソッドをプロトタイプの基底クラスに追加することもできます[。]{.chpule2}[
    ]{.chpuri2}このメソッドは[、]{.chpule2}[
    ]{.chpuri2}クライアント・コードからメソッドに渡される検索条件に基づいてプロトタイプを検索します[。]{.chpule2}[
    ]{.chpuri2}検索条件は単純な文字列によるタグかもしれませんし[、]{.chpule2}[
    ]{.chpuri2}または複雑な検索パラメータの組み合わせかもしれません[。]{.chpule2}[
    ]{.chpuri2}適切なプロトタイプが見つかった後[、]{.chpule2}[
    ]{.chpuri2}レジストリーはそれをクローンし[、]{.chpule2}[
    ]{.chpuri2}クライアントに返します[。]{.chpule2}[ ]{.chpuri2}

    最後に[、]{.chpule2}[
    ]{.chpuri2}サブクラスのコンストラクターへの直接呼び出しを[、]{.chpule2}[
    ]{.chpuri2}プロトタイプ・レジストリーのファクトリー・メソッドへの呼び出しに置き換えます[。]{.chpule2}[
    ]{.chpuri2}
:::

:::::: {.section .pros-cons}
##  長所と短所 {#pros-cons}

::::: row
::: col-sm-6
-   
    具象クラスと密に結合せずにオブジェクトのクローンが可能[。]{.chpule2}[
    ]{.chpuri2}
-    構築済みのプロトタイプのクローン作成を使うことにより[、]{.chpule2}[
    ]{.chpuri2}初期化コードの重複を削減[。]{.chpule2}[ ]{.chpuri2}
-    複雑なオブジェクトの生成がより便利[。]{.chpule2}[ ]{.chpuri2}
-   
    複雑なオブジェクトに対する構成の事前設定を扱う上で継承に代わる方法を提供[。]{.chpule2}[
    ]{.chpuri2}
:::

::: col-sm-6
-   
    循環参照のある複雑なオブジェクトのクローン作成は一筋縄ではいかない場合あり[。]{.chpule2}[
    ]{.chpuri2}
:::
:::::
::::::

::: {.section .relations}
##  他のパターンとの関係 {#relations}

-   多くの設計は[、]{.chpule2}[
    ]{.chpuri2}まず比較的単純でサブクラスによりカスタマイズ可能な[、]{.chpule2}[
    ]{.chpuri2}[Factory Method](factory-method.html)
    から始まり[、]{.chpule2}[ ]{.chpuri2}次第に[、]{.chpule2}[
    ]{.chpuri2}もっと柔軟だが複雑な [Abstract
    Factory](abstract-factory.html) や [Prototype](prototype.html) や
    [Builder](builder.html) へと発展していきます[。]{.chpule2}[
    ]{.chpuri2}

-   [Abstract Factory](abstract-factory.html) クラスは[、]{.chpule2}[
    ]{.chpuri2}多くの場合 [Factory Methods](factory-method.html)
    の集まりですが[、]{.chpule2}[ ]{.chpuri2}[Prototype](prototype.html)
    を使ってメソッドを書くこともできます[。]{.chpule2}[ ]{.chpuri2}

-   [Prototype](prototype.html) は[、]{.chpule2}[
    ]{.chpuri2}[Commands](command.html)
    のコピーを履歴に保存する必要がある場合に役立ちます[。]{.chpule2}[
    ]{.chpuri2}

-   [Composite](composite.html) と [Decorator](decorator.html)
    を多用する設計に対しては[、]{.chpule2}[
    ]{.chpuri2}[Prototype](prototype.html)
    の使用が有益かもしれません[。]{.chpule2}[
    ]{.chpuri2}このパターンを適用すると[、]{.chpule2}[
    ]{.chpuri2}複雑な構造を初めから再構築するのではなく[、]{.chpule2}[
    ]{.chpuri2}それをクローンします[。]{.chpule2}[ ]{.chpuri2}

-   [Prototype](prototype.html)
    は継承に基づいていないので[、]{.chpule2}[
    ]{.chpuri2}継承の欠点はありません[。]{.chpule2}[
    ]{.chpuri2}一方[、]{.chpule2}[ ]{.chpuri2}*Prototype*
    は[、]{.chpule2}[
    ]{.chpuri2}クローンされたオブジェクトの複雑な初期化が必要となります[。]{.chpule2}[
    ]{.chpuri2}[Factory Method](factory-method.html)
    は継承に基づいていますが[、]{.chpule2}[
    ]{.chpuri2}初期化のステップは必要ありません[。]{.chpule2}[
    ]{.chpuri2}

-   場合によっては[、]{.chpule2}[ ]{.chpuri2}[Prototype](prototype.html)
    を [Memento](memento.html)
    の代わりに使用した方が簡単な場合があります[。]{.chpule2}[
    ]{.chpuri2}状態の履歴を保存したいオブジェクトが比較的単純で[、]{.chpule2}[
    ]{.chpuri2}他の外部リソースへのリンクを持たないか簡単に再現できる場合に[、]{.chpule2}[
    ]{.chpuri2}この方法が使えます[。]{.chpule2}[ ]{.chpuri2}

-   [Abstract Factories](abstract-factory.html)[、]{.chpule2}[
    ]{.chpuri2}[Builders](builder.html)[、]{.chpule2}[
    ]{.chpuri2}[Prototypes](prototype.html) はどれも
    [Singletons](singleton.html) で実装可能です[。]{.chpule2}[
    ]{.chpuri2}
:::

::: {.section .implementations}
##  コード例 {#implementations}

[![Prototype を C#
で](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](prototype/csharp/example.html "Prototype を C# で"){.prog-lang-link}
[![Prototype を C++
で](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](prototype/cpp/example.html "Prototype を C++ で"){.prog-lang-link}
[![Prototype を Go
で](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](prototype/go/example.html "Prototype を Go で"){.prog-lang-link}
[![Prototype を Java
で](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](prototype/java/example.html "Prototype を Java で"){.prog-lang-link}
[![Prototype を PHP
で](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](prototype/php/example.html "Prototype を PHP で"){.prog-lang-link}
[![Prototype を Python
で](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](prototype/python/example.html "Prototype を Python で"){.prog-lang-link}
[![Prototype を Ruby
で](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](prototype/ruby/example.html "Prototype を Ruby で"){.prog-lang-link}
[![Prototype を Rust
で](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](prototype/rust/example.html "Prototype を Rust で"){.prog-lang-link}
[![Prototype を Swift
で](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](prototype/swift/example.html "Prototype を Swift で"){.prog-lang-link}
[![Prototype を TypeScript
で](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](prototype/typescript/example.html "Prototype を TypeScript で"){.prog-lang-link}
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

[Singleton []{.fa .fa-arrow-right}](singleton.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### 戻る

[[]{.fa .fa-arrow-left} Builder](builder.html){.btn .btn-default
rel="prev"}
:::
:::::::::::::::::::::::::::::::::::

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
:::::::::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::::::::
