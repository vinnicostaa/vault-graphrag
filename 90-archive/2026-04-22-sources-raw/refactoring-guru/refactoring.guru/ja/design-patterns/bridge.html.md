:::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} /
[デザインパターン](../design-patterns.html){.type} /
[構造に関するパターン](structural-patterns.html){.type}
:::

# Bridge {#bridge .title}

::: aka
別名：[ブリッジ]{style="display:inline-block"}
:::

::: {.section .intent}
##  一言でいうと {#intent}

**Bridge**[ ]{.chpule1}[（]{.chpuri1}ブリッジ[、]{.chpule2}[
]{.chpuri2}架け橋[）]{.chpule2}[ ]{.chpuri2}は[、]{.chpule2}[
]{.chpuri2}構造的パターンの一つです[。]{.chpule2}[
]{.chpuri2}巨大なクラスや密接に関連したクラスの集まりを[、]{.chpule2}[
]{.chpuri2}抽象部分と実装部分という[、]{.chpule2}[
]{.chpuri2}二つの階層に分離し[、]{.chpule2}[
]{.chpuri2}それぞれが独立して開発できるようにします[。]{.chpule2}[
]{.chpuri2}

<figure class="image">
<img
src="../../images/patterns/content/bridge/bridge3e59.png?id=bd543d4fb32e11647767301581a5ad54"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/bridge/bridge.png?id=bd543d4fb32e11647767301581a5ad54 640w,/images/patterns/content/bridge/bridge-1.5x.png?id=8ef07b78d61cc3830b7e2b783c76c775 960w,/images/patterns/content/bridge/bridge-2x.png?id=1e905ae5742e5cd10a7eb0e3175ef00d 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Bridge デザインパターン" />
</figure>
:::

::: {.section .problem}
##  問題 {#problem}

*[抽]{.cjk}[象]{.cjk}*と*[実]{.cjk}[装]{.cjk}*だって[？]{.chpule2}[
]{.chpuri2}何だか怖そう[？]{.chpule2}[
]{.chpuri2}心配無用[。]{.chpule2}[
]{.chpuri2}ある簡単な例を考えていきましょう[。]{.chpule2}[ ]{.chpuri2}

ええと[、]{.chpule2}[ ]{.chpuri2}幾何学的な形状を表す `Shape`
クラスとそのサブクラスとして `Circle`[
]{.chpule1}[（]{.chpuri1}円[）]{.chpule2}[ ]{.chpuri2}と `Square`[
]{.chpule1}[（]{.chpuri1}正方形[）]{.chpule2}[
]{.chpuri2}があるとします[。]{.chpule2}[
]{.chpuri2}色という要素を表現するために[、]{.chpule2}[
]{.chpuri2}このクラス階層を拡張して[、]{.chpule2}[ ]{.chpuri2}`Red`[
]{.chpule1}[（]{.chpuri1}赤[）]{.chpule2}[ ]{.chpuri2}と `Blue`[
]{.chpule1}[（]{.chpuri1}青[）]{.chpule2}[
]{.chpuri2}の形状クラスを作りたいとします[。]{.chpule2}[
]{.chpuri2}しかしすでに二つのサブクラスがあるため[、]{.chpule2}[
]{.chpuri2}この組み合わせである四つのサブクラスが必要となります[。]{.chpule2}[
]{.chpuri2}`Blue­Circle` とか `Red­Square` です[。]{.chpule2}[ ]{.chpuri2}

<figure class="image">
<img
src="../../images/patterns/diagrams/bridge/problem-jadc1e.png?id=a1ec21a9f546bc0054f4fdd9213f7cf2"
style="aspect-ratio: 1.41;"
srcset="/images/patterns/diagrams/bridge/problem-ja.png?id=a1ec21a9f546bc0054f4fdd9213f7cf2 480w,/images/patterns/diagrams/bridge/problem-ja-1.5x.png?id=a24f6ae98a0ecced40e8f3feeab058d5 720w,/images/patterns/diagrams/bridge/problem-ja-2x.png?id=d696b5e067f716559cba9670a2f63635 960w"
sizes="(max-width: 720px) 100vw, 480px" loading="lazy" width="480"
alt="Bridge パターンの問題" />
<figcaption><p>クラスの組み合わせ数は幾何学的な速さで増加する<span
class="chpule2">。</span><span class="chpuri2"> </span></p></figcaption>
</figure>

クラス階層に形状の種類と色を加えるとそれは指数関数的に増加します[。]{.chpule2}[
]{.chpuri2}たとえば[、]{.chpule2}[
]{.chpuri2}三角形を追加するためには[、]{.chpule2}[
]{.chpuri2}それぞれの色に一つずつ[、]{.chpule2}[
]{.chpuri2}計二つのサブクラスを導入する必要があります[。]{.chpule2}[
]{.chpuri2}その後[、]{.chpule2}[
]{.chpuri2}色を一つ追加するには[、]{.chpule2}[
]{.chpuri2}形状の種類一つにつき一つずつ[、]{.chpule2}[
]{.chpuri2}計三つのサブクラスが必要となります[。]{.chpule2}[ ]{.chpuri2}
:::

::: {.section .solution}
##  解決策 {#solution}

この問題は[、]{.chpule2}[ ]{.chpuri2}形状クラスを[、]{.chpule2}[
]{.chpuri2}形と色という二つの独立した次元で拡張しようとしているために発生します[。]{.chpule2}[
]{.chpuri2}これはクラスの継承に関する非常に一般的な問題です[。]{.chpule2}[
]{.chpuri2}

Bridge パターンでは[、]{.chpule2}[
]{.chpuri2}継承からオブジェクトの合成に切り替えることでこの問題を解決しようとします[。]{.chpule2}[
]{.chpuri2}つまり[、]{.chpule2}[
]{.chpuri2}次元のいずれか一つを別のクラス階層に抽出するということです[。]{.chpule2}[
]{.chpuri2}一つのクラス内にすべての状態と振る舞いを持つ代わりに[、]{.chpule2}[
]{.chpuri2}元のクラスは[、]{.chpule2}[
]{.chpuri2}新しい階層のオブジェクトを参照するようにします[。]{.chpule2}[
]{.chpuri2}

<figure class="image">
<img
src="../../images/patterns/diagrams/bridge/solution-ja9799.png?id=2dba80e7e45c19ae98f86645ae44cb16"
style="aspect-ratio: 2.3;"
srcset="/images/patterns/diagrams/bridge/solution-ja.png?id=2dba80e7e45c19ae98f86645ae44cb16 460w,/images/patterns/diagrams/bridge/solution-ja-1.5x.png?id=7dd75da4b1ea6c60a79477cb71cc4296 690w,/images/patterns/diagrams/bridge/solution-ja-2x.png?id=975887e9aecf4bc6fb185e25772f4fa0 920w"
sizes="(max-width: 720px) 100vw, 460px" loading="lazy" width="460"
alt="Bridge パターンの提案する解決策" />
<figcaption><p>クラス階層の膨張を防ぐために<span
class="chpule2">、</span><span class="chpuri2">
</span>いくつかの関連する階層に変換する<span
class="chpule2">。</span><span class="chpuri2"> </span></p></figcaption>
</figure>

このやり方に従うと[、]{.chpule2}[
]{.chpuri2}色に関するコードは[、]{.chpule2}[ ]{.chpuri2}`Red` と `Blue`
の二つのサブクラスを持つ専用のクラスに抽出します[。]{.chpule2}[
]{.chpuri2}`Shape` クラスには[、]{.chpule2}[
]{.chpuri2}色オブジェクト二つのうち一つへの参照が付きます[。]{.chpule2}[
]{.chpuri2}形状は[、]{.chpule2}[
]{.chpuri2}色に関する作業を色オブジェクトに委任します[。]{.chpule2}[
]{.chpuri2}この参照は[、]{.chpule2}[ ]{.chpuri2}`Shape` クラスと `Color`
クラスのブリッジ＝架け橋として機能します[。]{.chpule2}[
]{.chpuri2}今後は[、]{.chpule2}[
]{.chpuri2}色を追加しても形状階層を変更する必要はなく[、]{.chpule2}[
]{.chpuri2}逆もまた可です[。]{.chpule2}[ ]{.chpuri2}

#### 抽象化と実装

[ ]{.chpule1}[「]{.chpuri1}GoF 本 []{.pop-i}[[
]{.chpule1}[「]{.chpuri1}Gang of Four[[」]{.ls-1}]{.chpule2}[
]{.chpuri2}​[ ]{.chpule1}[（]{.chpuri1}4 人のギャング[）]{.chpule2}[
]{.chpuri2}は[、]{.chpule2}[ ]{.chpuri2}デザインパターンの元となった本[
]{.chpule1}[『]{.chpuri1}[オブジェクト指向における再利用のためのデザインパターン](https://www.amazon.co.jp/-/dp/4797311126)[』]{.chpule2}[
]{.chpuri2}の 4 人の著者のニックネームです[。]{.chpule2}[
]{.chpuri2}]{.pop-content}[」]{.chpule2}[ ]{.chpuri2}には[、]{.chpule2}[
]{.chpuri2}*[抽]{.cjk}[象]{.cjk}[化]{.cjk}*[
]{.chpule1}[（]{.chpuri1}Abstraction[）]{.chpule2}[
]{.chpuri2}と*[実]{.cjk}[装]{.cjk}*[
]{.chpule1}[（]{.chpuri1}Implementation[）]{.chpule2}[
]{.chpuri2}という用語が[、]{.chpule2}[ ]{.chpuri2}Bridge
の定義の中で使われています[。]{.chpule2}[
]{.chpuri2}著者の意見では[、]{.chpule2}[
]{.chpuri2}これらの用語は学術的に響きすぎ[、]{.chpule2}[
]{.chpuri2}このパターンに実際よりも複雑な印象を与えます[。]{.chpule2}[
]{.chpuri2}形状と色の簡単な例を読んだところで[、]{.chpule2}[
]{.chpuri2}GoF 本の怖そううな言葉を解読していきましょう[。]{.chpule2}[
]{.chpuri2}

*[抽]{.cjk}[象]{.cjk}[化]{.cjk}*[
]{.chpule1}[（]{.chpuri1}インターフェースとも[）]{.chpule2}[
]{.chpuri2}は[、]{.chpule2}[
]{.chpuri2}ある項目の高レベル制御層です[。]{.chpule2}[
]{.chpuri2}この層では[、]{.chpule2}[
]{.chpuri2}実際の仕事は行わないことになっています[。]{.chpule2}[
]{.chpuri2}仕事は[、]{.chpule2}[ ]{.chpuri2}*[実]{.cjk}[装]{.cjk}*層[
]{.chpule1}[（]{.chpuri1}プラットフォームとも[）]{.chpule2}[
]{.chpuri2}へ移譲されます[。]{.chpule2}[ ]{.chpuri2}

注意[：]{.chpule2}[ ]{.chpuri2}ここではあるプログラミング言語の
*interface* や *abstract*
についてお話ししているわけではありません[。]{.chpule2}[
]{.chpuri2}これらは同じものではありません[。]{.chpule2}[ ]{.chpuri2}

ここで実際のアプリケーションについての話をすると[、]{.chpule2}[
]{.chpuri2}抽象化はグラフィカル・ユーザー・インターフェース[
]{.chpule1}[（]{.chpuri1}GUI[）]{.ls-05}[、]{.chpule2}[
]{.chpuri2}そして実装は[、]{.chpule2}[ ]{.chpuri2}ユーザーの動作に応じて
GUI 層が呼び出す基盤オペレーティングシステムのコード[
]{.chpule1}[（]{.chpuri1}API[）]{.chpule2}[
]{.chpuri2}と考えることもできます[。]{.chpule2}[ ]{.chpuri2}

一般的に[、]{.chpule2}[ ]{.chpuri2}このようなアプリは[、]{.chpule2}[
]{.chpuri2}二つの独立した方向に拡張することができます[：]{.chpule2}[
]{.chpuri2}

-   いくつかの異なる GUI[
    ]{.chpule1}[（]{.chpuri1}たとえば[、]{.chpule2}[
    ]{.chpuri2}通常の顧客向けに仕立てたり[、]{.chpule2}[
    ]{.chpuri2}管理者向けに仕立てたり[）]{.chpule2}[
    ]{.chpuri2}を提供する[。]{.chpule2}[ ]{.chpuri2}
-   いくつかの異なる API をサポートする[
    ]{.chpule1}[（]{.chpuri1}たとえば[、]{.chpule2}[
    ]{.chpuri2}Windows[、]{.chpule2}[ ]{.chpuri2}Linux[、]{.chpule2}[
    ]{.chpuri2}macOS
    下でアプリを起動を可能とするために[）]{.ls-05}[。]{.chpule2}[
    ]{.chpuri2}

最悪の場合[、]{.chpule2}[
]{.chpuri2}このアプリは大盛りスパゲッティーのように見えるかもしれません[。]{.chpule2}[
]{.chpuri2}様々な API と様々な GUI
を接続する何百もの条件文が[、]{.chpule2}[
]{.chpuri2}コード全体に散らばっています[。]{.chpule2}[ ]{.chpuri2}

<figure class="image">
<img
src="../../images/patterns/content/bridge/bridge-3-jac320.png?id=b51005d7de9f93399c0d14bcf1df72f2"
style="aspect-ratio: 2;"
srcset="/images/patterns/content/bridge/bridge-3-ja.png?id=b51005d7de9f93399c0d14bcf1df72f2 600w,/images/patterns/content/bridge/bridge-3-ja-1.5x.png?id=629cf8ec14ee12b0e867a02335a67006 900w,/images/patterns/content/bridge/bridge-3-ja-2x.png?id=c393fcf281a2b770237ccb7b2375cbad 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="モジュール化されたコードでは変更管理が簡単にできる" />
<figcaption><p>一枚岩のコードベースには<span
class="chpule2">、</span><span class="chpuri2">
</span>簡単な変更を加えることすらかなり困難です<span
class="chpule2">。</span><span class="chpuri2"> </span>なぜなら
<em><span class="cjk">全</span><span class="cjk">体</span><span
class="cjk">の</span><span class="cjk">事</span><span
class="cjk">情</span></em> を非常によく理解しなければならないからです<span
class="chpule2">。</span><span class="chpuri2"> </span>小さく<span
class="chpule2">、</span><span class="chpuri2">
</span>明確に定義されたモジュールに変更を加えるのは<span
class="chpule2">、</span><span class="chpuri2">
</span>はるかに簡単です<span class="chpule2">。</span><span
class="chpuri2"> </span></p></figcaption>
</figure>

特定のインターフェースとプラットフォームの組み合わせに関連するコードを個別のクラスに抽出することで[、]{.chpule2}[
]{.chpuri2}この混乱的状況に秩序をもたらすことができます[。]{.chpule2}[
]{.chpuri2}しかし[、]{.chpule2}[
]{.chpuri2}すぐにこういうクラスが*[多]{.cjk}[数]{.cjk}*あることがわかります[。]{.chpule2}[
]{.chpuri2}新しい GUI を一つ追加したり[、]{.chpule2}[ ]{.chpuri2}別の
API を一つサポートするだけでも[、]{.chpule2}[
]{.chpuri2}さらに多くのクラスを作成する必要があり[、]{.chpule2}[
]{.chpuri2}クラス階層は指数関数的に増加します[。]{.chpule2}[ ]{.chpuri2}

Bridge パターンでこの問題を解決しましょう[。]{.chpule2}[
]{.chpuri2}このパターンに従うと[、]{.chpule2}[
]{.chpuri2}クラスを二つの階層に分割することになります[。]{.chpule2}[
]{.chpuri2}

-   抽象化[：]{.chpule2}[ ]{.chpuri2}アプリの GUI レイヤー
-   実装[：]{.chpule2}[ ]{.chpuri2}オペレーティングシステムの API

<figure class="image">
<img
src="../../images/patterns/content/bridge/bridge-2-jaed75.png?id=67aebf6a776ff79a276925f64984f14c"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/bridge/bridge-2-ja.png?id=67aebf6a776ff79a276925f64984f14c 640w,/images/patterns/content/bridge/bridge-2-ja-1.5x.png?id=8beb8e955196944a8ae44d19b5fb896e 960w,/images/patterns/content/bridge/bridge-2-ja-2x.png?id=1a8caf78c4f47c28925c6e2f9866e476 1280w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="640"
alt="プラットフォーム互換アーキテクチャー" />
<figcaption><p>プラットフォーム互換アプリケーションを構築する方法の一つ<span
class="chpule2">。</span><span class="chpuri2"> </span></p></figcaption>
</figure>

抽象化層のオブジェクトは[、]{.chpule2}[
]{.chpuri2}アプリケーションの外観を制御し[、]{.chpule2}[
]{.chpuri2}実際の作業をリンクされた実装層のオブジェクトに委任します[。]{.chpule2}[
]{.chpuri2}異なる実装は[、]{.chpule2}[
]{.chpuri2}共通のインターフェースに従っている限り[、]{.chpule2}[
]{.chpuri2}交換が可能です[。]{.chpule2}[
]{.chpuri2}これにより[、]{.chpule2}[ ]{.chpuri2}Windows 下でも Linux
下でも同じ GUI が動作するようになります[。]{.chpule2}[ ]{.chpuri2}

その結果[、]{.chpule2}[ ]{.chpuri2}API
関連のクラスに触れることなく[、]{.chpule2}[ ]{.chpuri2}GUI
クラスを変更できます[。]{.chpule2}[ ]{.chpuri2}さらに[、]{.chpule2}[
]{.chpuri2}新たなオペレーティングシステムをサポートするためには[、]{.chpule2}[
]{.chpuri2}実装階層にサブクラスを作成するだけですみます[。]{.chpule2}[
]{.chpuri2}
:::

::::: {.section .structure-container}
##  構造 {#structure}

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/bridge/structure-ja55cd.png?id=be8bf0cbd8785d3b3051e05428017343"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.47;"
srcset="/images/patterns/diagrams/bridge/structure-ja.png?id=be8bf0cbd8785d3b3051e05428017343 560w,/images/patterns/diagrams/bridge/structure-ja-1.5x.png?id=d4f16b3d063bd5b0af22ec0c7c366ca2 840w,/images/patterns/diagrams/bridge/structure-ja-2x.png?id=2293609d2cefc7b45ee9ea9480840e35 1120w"
sizes="(max-width: 720px) 100vw, 560px" loading="lazy" width="560"
alt="Bridge デザインパターン" /><img
src="../../images/patterns/diagrams/bridge/structure-ja-indexed7cb4.png?id=ba145727683f6085f65875d6e23f410c"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.44;"
srcset="/images/patterns/diagrams/bridge/structure-ja-indexed.png?id=ba145727683f6085f65875d6e23f410c 560w,/images/patterns/diagrams/bridge/structure-ja-indexed-1.5x.png?id=9d0023e90af4cbf12f6dcb36dc31f120 840w,/images/patterns/diagrams/bridge/structure-ja-indexed-2x.png?id=1b603552c0bb1a6402c4d74e39079edb 1120w"
sizes="(max-width: 720px) 100vw, 560px" loading="lazy" width="560"
alt="Bridge デザインパターン" />
</figure>
:::

1.  **抽象化層**[ ]{.chpule1}[（]{.chpuri1}Abstraction[）]{.chpule2}[
    ]{.chpuri2}は[、]{.chpule2}[
    ]{.chpuri2}高レベルの制御ロジックを提供します[。]{.chpule2}[
    ]{.chpuri2}実際の低レベルの作業は実装オブジェクトに任されています[。]{.chpule2}[
    ]{.chpuri2}

2.  **実装**[ ]{.chpule1}[（]{.chpuri1}Implementation[）]{.chpule2}[
    ]{.chpuri2}は[、]{.chpule2}[
    ]{.chpuri2}すべての具象実装に共通のインターフェースを宣言します[。]{.chpule2}[
    ]{.chpuri2}抽象化層は[、]{.chpule2}[
    ]{.chpuri2}ここで宣言されたメソッドを介してのみ実装オブジェクトと通信することができます[。]{.chpule2}[
    ]{.chpuri2}

    抽象化は[、]{.chpule2}[
    ]{.chpuri2}実装と同じメソッドを並べただけでもかまいません[。]{.chpule2}[
    ]{.chpuri2}しかし通常[、]{.chpule2}[
    ]{.chpuri2}抽象化は複雑な振る舞いを宣言します[。]{.chpule2}[
    ]{.chpuri2}それらの振る舞いは[、]{.chpule2}[
    ]{.chpuri2}実装によって宣言された多種多様な基本操作を使用します[。]{.chpule2}[
    ]{.chpuri2}

3.  **具象的実装**[ ]{.chpule1}[（]{.chpuri1}Concrete
    Implementation[）]{.chpule2}[ ]{.chpuri2}には[、]{.chpule2}[
    ]{.chpuri2}プラットフォーム固有のコードが含まれています[。]{.chpule2}[
    ]{.chpuri2}

4.  **整った抽象化**[ ]{.chpule1}[（]{.chpuri1}Refined
    Abstraction[）]{.chpule2}[ ]{.chpuri2}は[、]{.chpule2}[
    ]{.chpuri2}制御ロジックの変種を提供します[。]{.chpule2}[
    ]{.chpuri2}親と同様に[、]{.chpule2}[
    ]{.chpuri2}一般的な実装インターフェースを介して各種の異なる実装を使います[。]{.chpule2}[
    ]{.chpuri2}

5.  通常[、]{.chpule2}[ ]{.chpuri2}**クライアント**[
    ]{.chpule1}[（]{.chpuri1}Client[）]{.chpule2}[
    ]{.chpuri2}は[、]{.chpule2}[
    ]{.chpuri2}抽象化層とだけやりとりをします[。]{.chpule2}[
    ]{.chpuri2}しかし[、]{.chpule2}[
    ]{.chpuri2}抽象化層のオブジェクトと実装オブジェクトを結びつけるのは[、]{.chpule2}[
    ]{.chpuri2}クライアントの仕事です[。]{.chpule2}[ ]{.chpuri2}
::::
:::::

::: {.section .pseudocode}
##  擬似コード {#pseudocode}

この例では[、]{.chpule2}[ ]{.chpuri2}**Bridge**
パターンが[、]{.chpule2}[
]{.chpuri2}デバイスとそのリモコンを管理するアプリの一枚岩のコードを分割するのにどう役立つかを説明します[。]{.chpule2}[
]{.chpuri2}`Device` 系のクラスは実装として機能し[、]{.chpule2}[
]{.chpuri2}`Remote` 系のクラスは抽象化層です[。]{.chpule2}[ ]{.chpuri2}

<figure class="image">
<img
src="../../images/patterns/diagrams/bridge/example-jaab39.png?id=89c406a189c45885004d7fa094f616b1"
style="aspect-ratio: 1.33;"
srcset="/images/patterns/diagrams/bridge/example-ja.png?id=89c406a189c45885004d7fa094f616b1 640w,/images/patterns/diagrams/bridge/example-ja-1.5x.png?id=bb870a109edc52920a51b3fc9110e7f4 960w,/images/patterns/diagrams/bridge/example-ja-2x.png?id=19bb8272b4082c5f47c4d047e6cb9967 1280w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="640"
alt="Bridge パターン例の構造" />
<figcaption><p>元のクラス階層は<span class="chpule2">、</span><span
class="chpuri2"> </span>device<span class="chpule1"> </span><span
class="chpuri1">（</span>機器<span class="chpule2">）</span><span
class="chpuri2"> </span>と remote<span class="chpule1"> </span><span
class="chpuri1">（</span>リモコン<span class="chpule2">）</span><span
class="chpuri2"> </span>の二つの部分に分割される<span
class="chpule2">。</span><span class="chpuri2"> </span></p></figcaption>
</figure>

基底のリモコン・クラスは[、]{.chpule2}[
]{.chpuri2}機器オブジェクトへ結びつけるための参照フィールドを宣言しています[。]{.chpule2}[
]{.chpuri2}すべてのリモコンは[、]{.chpule2}[
]{.chpuri2}機器とのやり取りを一般的な機器インターフェースを介して行い[、]{.chpule2}[
]{.chpuri2}同じリモコンで複数の種類の機器をサポートできます[。]{.chpule2}[
]{.chpuri2}

リモコン・クラスの開発は[、]{.chpule2}[
]{.chpuri2}機器クラスと独立して行えます[。]{.chpule2}[
]{.chpuri2}行うべきことは[、]{.chpule2}[
]{.chpuri2}新しいリモコンのサブクラスを作成することだけです[。]{.chpule2}[
]{.chpuri2}たとえば[、]{.chpule2}[
]{.chpuri2}基本的なリモコンには二つのボタンしかありませんが[、]{.chpule2}[
]{.chpuri2}これを拡張して[、]{.chpule2}[
]{.chpuri2}追加の電池やタッチスクリーンなどの機能を追加できます[。]{.chpule2}[
]{.chpuri2}

クライアント・コードは[、]{.chpule2}[
]{.chpuri2}好みの種類のリモコンを特定の機器オブジェクトとリモコンのコンストラクターを介して結びつけます[。]{.chpule2}[
]{.chpuri2}

<figure class="code">
<pre class="code" lang="pseudocode"><code>// 抽象化層は、二つのクラス階層の「Control」関係のインターフェースを定義。
// これは、「実装」階層のオブジェクトへの参照を維持し、実際の作業のすべ
// てをこのオブジェクトに委任。
class RemoteControl is
    protected field device: Device
    constructor RemoteControl(device: Device) is
        this.device = device
    method togglePower() is
        if (device.isEnabled()) then
            device.disable()
        else
            device.enable()
    method volumeDown() is
        device.setVolume(device.getVolume() - 10)
    method volumeUp() is
        device.setVolume(device.getVolume() + 10)
    method channelDown() is
        device.setChannel(device.getChannel() - 1)
    method channelUp() is
        device.setChannel(device.getChannel() + 1)


// 抽象化層のクラスは、Device 系クラスとは独立して拡張可能。
class AdvancedRemoteControl extends RemoteControl is
    method mute() is
        device.setVolume(0)


// 「実装」インターフェースは、全具象実装クラスに共通するメソッドを定義。
// 抽象層インターフェースと一致する必要なし。実際のところ、この二つはまっ
// たく異なってよい。典型的には、実装インターフェースは基礎的な操作を宣言
// し、抽象化層は基礎的操作に基づいて高レベルな操作を定義。
interface Device is
    method isEnabled()
    method enable()
    method disable()
    method getVolume()
    method setVolume(percent)
    method getChannel()
    method setChannel(channel)


// どの機器（Device）も同じインターフェースに準拠。
class Tv implements Device is
    // ……

class Radio implements Device is
    // ……


// クライアント・コードのどこかの様子。
tv = new Tv()
remote = new RemoteControl(tv)
remote.togglePower()

radio = new Radio()
remote = new AdvancedRemoteControl(radio)</code></pre>
</figure>
:::

:::::::::: {.section .applicability-container}
##  適応性 {#applicability}

::::::::: applicability
::: applicability-problem
たとえば[、]{.chpule2}[ ]{.chpuri2}データベースなど[、]{.chpule2}[
]{.chpuri2}機能にちょっとした違い変種がある場合[、]{.chpule2}[
]{.chpuri2}一枚岩のコードを分割して組織するために[、]{.chpule2}[
]{.chpuri2}Bridge パターンを使います[。]{.chpule2}[ ]{.chpuri2}
:::

::: applicability-solution
クラスが増大するにつれ[、]{.chpule2}[
]{.chpuri2}その動作を理解することが難しくなり[、]{.chpule2}[
]{.chpuri2}変更に時間がかかるようになります[。]{.chpule2}[
]{.chpuri2}機能の一つの変種に加えられた変更には[、]{.chpule2}[
]{.chpuri2}クラス全体の変更を要するようになり[、]{.chpule2}[
]{.chpuri2}誤りや重大な副作用への対処を欠くなどの弊害につながります[。]{.chpule2}[
]{.chpuri2}

Bridgeパターンでは[、]{.chpule2}[
]{.chpuri2}一枚岩のクラスをいくつかのクラス階層に分割します[。]{.chpule2}[
]{.chpuri2}その後は[、]{.chpule2}[
]{.chpuri2}独立して各階層のクラスを変更できるようになります[。]{.chpule2}[
]{.chpuri2}この方法により[、]{.chpule2}[
]{.chpuri2}コードの保守が簡素化され[、]{.chpule2}[
]{.chpuri2}既存のコードが動かなくなるリスクを最小限に抑えられます[。]{.chpule2}[
]{.chpuri2}
:::

::: applicability-problem
クラスをいくつかの直交的[
]{.chpule1}[（]{.chpuri1}独立した[）]{.chpule2}[
]{.chpuri2}次元で拡張する必要がある場合[、]{.chpule2}[
]{.chpuri2}このパターンを使用します[。]{.chpule2}[ ]{.chpuri2}
:::

::: applicability-solution
Bridge では[、]{.chpule2}[
]{.chpuri2}各次元に対して個別のクラス階層を抽出することを勧めます[。]{.chpule2}[
]{.chpuri2}元のクラスは[、]{.chpule2}[
]{.chpuri2}すべてを自身で行うのではなく[、]{.chpule2}[
]{.chpuri2}それぞれの階層に属するオブジェクトに関連作業を委任します[。]{.chpule2}[
]{.chpuri2}
:::

::: applicability-problem
実行時に実装を切り替える必要がある場合は[、]{.chpule2}[
]{.chpuri2}Bridge を使用してください[。]{.chpule2}[ ]{.chpuri2}
:::

::: applicability-solution
これは[、]{.chpule2}[ ]{.chpuri2}必ずしも必要ありませんが[、]{.chpule2}[
]{.chpuri2}Bridge パターンでは[、]{.chpule2}[
]{.chpuri2}実装オブジェクトを抽象化層内で置き換えることも許されています[。]{.chpule2}[
]{.chpuri2}新しい値をフィールドに割り当てるのと同じくらい簡単です[。]{.chpule2}[
]{.chpuri2}

ちなみに[、]{.chpule2}[ ]{.chpuri2}この最後の項目のため[、]{.chpule2}[
]{.chpuri2}多くの人々が Bridge を [Strategy](strategy.html)
パターンと混同します[。]{.chpule2}[
]{.chpuri2}覚えておいていただきたことは[、]{.chpule2}[
]{.chpuri2}パターンはクラスに特定の構造を持たせる以上のものであるということです[。]{.chpule2}[
]{.chpuri2}意図と解決すべき問題を伝えるためにも使用します[。]{.chpule2}[
]{.chpuri2}
:::
:::::::::
::::::::::

::: {.section .checklist}
##  実装方法 {#checklist}

1.  クラス内の直交する次元を特定します[。]{.chpule2}[
    ]{.chpuri2}これらの独立した概念は[、]{.chpule2}[
    ]{.chpuri2}たとえば抽象化とプラットフォーム[
    ]{.chpule1}[（]{.chpuri1}OS[）]{.chpule2}[
    ]{.chpuri2}だったり[、]{.chpule2}[
    ]{.chpuri2}ドメインとインフラストラクチャーだったり[、]{.chpule2}[
    ]{.chpuri2}フロントエンドとバックエンドだったり[、]{.chpule2}[
    ]{.chpuri2}インターフェースと実装だったりします[。]{.chpule2}[
    ]{.chpuri2}

2.  クライアントが必要とする操作は何であるかを考え[、]{.chpule2}[
    ]{.chpuri2}抽象化層の基底クラスで定義します[。]{.chpule2}[
    ]{.chpuri2}

3.  すべてのプラットフォームで利用可能な操作は何かを決定します[。]{.chpule2}[
    ]{.chpuri2}抽象化層が必要とするものを一般的な実装インターフェースで宣言します[。]{.chpule2}[
    ]{.chpuri2}

4.  サポートするすべてのプラットフォームに対して具体的な実装クラスを作成します[。]{.chpule2}[
    ]{.chpuri2}すべて実装クラスはプラットフォームの実装インターフェースに従うようにしてください[。]{.chpule2}[
    ]{.chpuri2}

5.  抽象化層のクラス内に[、]{.chpule2}[
    ]{.chpuri2}実装型の参照フィールドを追加します[。]{.chpule2}[
    ]{.chpuri2}抽象化層は[、]{.chpule2}[
    ]{.chpuri2}このフィールドから参照される実装オブジェクトにほとんどの仕事を移譲します[。]{.chpule2}[
    ]{.chpuri2}

6.  高レベルのロジックにいくつかの変種がある場合は[、]{.chpule2}[
    ]{.chpuri2}各変種ごとに抽象化層の基底クラスを拡張して[、]{.chpule2}[
    ]{.chpuri2}より洗練した抽象化層クラスを作成します[。]{.chpule2}[
    ]{.chpuri2}

7.  クライアント・コードは[、]{.chpule2}[
    ]{.chpuri2}抽象化層のクラスのコンストラクターに実装オブジェクトを渡して[、]{.chpule2}[
    ]{.chpuri2}互いを関連づけます[。]{.chpule2}[
    ]{.chpuri2}その後[、]{.chpule2}[
    ]{.chpuri2}クライアントは実装のことは忘れ[、]{.chpule2}[
    ]{.chpuri2}抽象化層のオブジェクトとだけやりとりをします[。]{.chpule2}[
    ]{.chpuri2}
:::

:::::: {.section .pros-cons}
##  長所と短所 {#pros-cons}

::::: row
::: col-sm-6
-    プラットフォーム非依存のクラスやアプリを作成できる[。]{.chpule2}[
    ]{.chpuri2}
-    クライアント・コードは高レベルの抽象化層で動作[。]{.chpule2}[
    ]{.chpuri2}プラットフォームの詳細に左右されない[。]{.chpule2}[
    ]{.chpuri2}
-   
    *[開]{.cjk}[放]{.cjk}[閉]{.cjk}[鎖]{.cjk}[の]{.cjk}[原]{.cjk}[則]{.cjk}*[。]{.chpule2}[
    ]{.chpuri2}新規の抽象化層と実装を互いに独立して導入可[。]{.chpule2}[
    ]{.chpuri2}
-   
    *[単]{.cjk}[一]{.cjk}[責]{.cjk}[任]{.cjk}[の]{.cjk}[原]{.cjk}[則]{.cjk}*[。]{.chpule2}[
    ]{.chpuri2}抽象化層では[、]{.chpule2}[
    ]{.chpuri2}高レベルのロジックに[、]{.chpule2}[
    ]{.chpuri2}実装層では[、]{.chpule2}[
    ]{.chpuri2}プラットフォームの詳細に集中できる[。]{.chpule2}[
    ]{.chpuri2}
:::

::: col-sm-6
-    高度に密着したクラスに本パターンを適用すると[、]{.chpule2}[
    ]{.chpuri2}コードが余計に複雑になる可能性あり[。]{.chpule2}[
    ]{.chpuri2}
:::
:::::
::::::

::: {.section .relations}
##  他のパターンとの関係 {#relations}

-   [Bridge](bridge.html) は通常[、]{.chpule2}[
    ]{.chpuri2}アプリケーションの部分部分を独立して開発できるように[、]{.chpule2}[
    ]{.chpuri2}設計当初から使われます[。]{.chpule2}[
    ]{.chpuri2}一方[、]{.chpule2}[ ]{.chpuri2}[Adapter](adapter.html)
    は[、]{.chpule2}[
    ]{.chpuri2}既存のアプリケーションに対して利用され[、]{.chpule2}[
    ]{.chpuri2}本来は互換性のないクラスとうまく動作させるために使われます[。]{.chpule2}[
    ]{.chpuri2}

-   [Bridge](bridge.html)[、]{.chpule2}[ ]{.chpuri2}
    [State](state.html)[、]{.chpule2}[
    ]{.chpuri2}[Strategy](strategy.html)[
    ]{.chpule1}[（]{.chpuri1}と限られた意味合いでは[、]{.chpule2}[
    ]{.chpuri2}[Adapter](adapter.html) も[）]{.chpule2}[
    ]{.chpuri2}は[、]{.chpule2}[
    ]{.chpuri2}非常に似た構造をしています[。]{.chpule2}[
    ]{.chpuri2}実際のところ[、]{.chpule2}[
    ]{.chpuri2}これらの全てのパターンは[、]{.chpule2}[
    ]{.chpuri2}合成に基づいており[、]{.chpule2}[
    ]{.chpuri2}仕事を他のオブジェクトに委任します[。]{.chpule2}[
    ]{.chpuri2}しかしながら[、]{.chpule2}[
    ]{.chpuri2}違う問題を解決します[。]{.chpule2}[
    ]{.chpuri2}パターンは[、]{.chpule2}[
    ]{.chpuri2}単にコードを特定の方法で構造化するためのレシピではありません[。]{.chpule2}[
    ]{.chpuri2}パターンが解決する問題に関して[、]{.chpule2}[
    ]{.chpuri2}開発者同士がするコミュニケーションの道具でもあります[。]{.chpule2}[
    ]{.chpuri2}

-   [Abstract Factory](abstract-factory.html) は[、]{.chpule2}[
    ]{.chpuri2}[Bridge](bridge.html) と一緒に使用できます[。]{.chpule2}[
    ]{.chpuri2}この組み合わせは[、]{.chpule2}[ ]{.chpuri2}*Bridge*
    によって定義された抽象化層のいくつかが特定の実装としか動作しない場合に便利です[。]{.chpule2}[
    ]{.chpuri2}この場合[、]{.chpule2}[ ]{.chpuri2}*Abstract Factory*
    はこれらの関係をカプセル化し[、]{.chpule2}[
    ]{.chpuri2}クライアント・コードから複雑さを隠すことができます[。]{.chpule2}[
    ]{.chpuri2}

-   [Builder](builder.html) と [Bridge](bridge.html)
    を組み合わせることができます[：]{.chpule2}[
    ]{.chpuri2}ディレクター・クラスは抽象化層の役割を果たし[、]{.chpule2}[
    ]{.chpuri2}ビルダーは実装です[。]{.chpule2}[ ]{.chpuri2}
:::

::: {.section .implementations}
##  コード例 {#implementations}

[![Bridge を C#
で](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](bridge/csharp/example.html "Bridge を C# で"){.prog-lang-link}
[![Bridge を C++
で](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](bridge/cpp/example.html "Bridge を C++ で"){.prog-lang-link}
[![Bridge を Go
で](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](bridge/go/example.html "Bridge を Go で"){.prog-lang-link}
[![Bridge を Java
で](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](bridge/java/example.html "Bridge を Java で"){.prog-lang-link}
[![Bridge を PHP
で](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](bridge/php/example.html "Bridge を PHP で"){.prog-lang-link}
[![Bridge を Python
で](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](bridge/python/example.html "Bridge を Python で"){.prog-lang-link}
[![Bridge を Ruby
で](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](bridge/ruby/example.html "Bridge を Ruby で"){.prog-lang-link}
[![Bridge を Rust
で](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](bridge/rust/example.html "Bridge を Rust で"){.prog-lang-link}
[![Bridge を Swift
で](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](bridge/swift/example.html "Bridge を Swift で"){.prog-lang-link}
[![Bridge を TypeScript
で](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](bridge/typescript/example.html "Bridge を TypeScript で"){.prog-lang-link}
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

[Composite []{.fa .fa-arrow-right}](composite.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### 戻る

[[]{.fa .fa-arrow-left} Adapter](adapter.html){.btn .btn-default
rel="prev"}
:::
:::::::::::::::::::::::::::::::::

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
:::::::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::::::
