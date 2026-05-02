::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} /
[デザインパターン](../design-patterns.html){.type} /
[生成に関するパターン](creational-patterns.html){.type}
:::

# Abstract Factory {#abstract-factory .title}

::: aka
別名：[アブストラクト・ファクトリー、]{style="display:inline-block"}[抽象ファクトリー]{style="display:inline-block"}
:::

::: {.section .intent}
##  一言でいうと {#intent}

**Abstract Factory**[
]{.chpule1}[（]{.chpuri1}抽象ファクトリー[、]{.chpule2}[
]{.chpuri2}抽象工場[）]{.chpule2}[ ]{.chpuri2}は[、]{.chpule2}[
]{.chpuri2}生成に関するデザインパターンの一つで[、]{.chpule2}[
]{.chpuri2}関連したオブジェクトの集りを[、]{.chpule2}[
]{.chpuri2}具象クラスを指定することなく生成することを可能とします[。]{.chpule2}[
]{.chpuri2}

<figure class="image">
<img
src="../../images/patterns/content/abstract-factory/abstract-factory-jad1bc.png?id=8c603e530e4940215efb38fbd9ec141a"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/abstract-factory/abstract-factory-ja.png?id=8c603e530e4940215efb38fbd9ec141a 640w,/images/patterns/content/abstract-factory/abstract-factory-ja-1.5x.png?id=c86efb3b728cd52e03aeae9639236877 960w,/images/patterns/content/abstract-factory/abstract-factory-ja-2x.png?id=e7f810fd4f7508c14d19f2d0325edc68 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Abstract Factory パターン" />
</figure>
:::

::: {.section .problem}
##  問題 {#problem}

家具工場シュミレーターを作ることを想像してみてください[。]{.chpule2}[
]{.chpuri2}コードは以下ようなものを表現するクラスで構成されています[：]{.chpule2}[
]{.chpuri2}

1.  関連する製品ファミリー[、]{.chpule2}[
    ]{.chpuri2}たとえば[：]{.chpule2}[ ]{.chpuri2}`Chair`[
    ]{.chpule1}[（]{.chpuri1}椅子[）]{.chpule2}[ ]{.chpuri2}+ `Sofa`[
    ]{.chpule1}[（]{.chpuri1}ソファー[）]{.chpule2}[ ]{.chpuri2}+
    `Coffee­Table`[
    ]{.chpule1}[（]{.chpuri1}コーヒーテーブル[）]{.ls-05}[。]{.chpule2}[
    ]{.chpuri2}

2.  このファミリーの様式[。]{.chpule2}[
    ]{.chpuri2}たとえば[、]{.chpule2}[ ]{.chpuri2}`Chair` + `Sofa` +
    `Coffee­Table` には以下のような様式のものがあります[：]{.chpule2}[
    ]{.chpuri2}`Modern`[
    ]{.chpule1}[（]{.chpuri1}現代風[）]{.ls-05}[、]{.chpule2}[
    ]{.chpuri2}`Victorian`[
    ]{.chpule1}[（]{.chpuri1}ビクトリア様式[）]{.ls-05}[、]{.chpule2}[
    ]{.chpuri2}`Art­Deco`[
    ]{.chpule1}[（]{.chpuri1}アールデコ様式[）]{.ls-05}[。]{.chpule2}[
    ]{.chpuri2}

<figure class="image">
<img
src="../../images/patterns/diagrams/abstract-factory/problem-jacc23.png?id=c1db09fd4ac3faf5214f10fe07de316b"
style="aspect-ratio: 1.4;"
srcset="/images/patterns/diagrams/abstract-factory/problem-ja.png?id=c1db09fd4ac3faf5214f10fe07de316b 420w,/images/patterns/diagrams/abstract-factory/problem-ja-1.5x.png?id=a249dd0b85933bec284b201c17c505cc 630w,/images/patterns/diagrams/abstract-factory/problem-ja-2x.png?id=673ac78f4c51f7a0fd0fc9f173be1c9c 840w"
sizes="(max-width: 720px) 100vw, 420px" loading="lazy" width="420"
alt="製品の集まりと様式" />
<figcaption><p>製品群と様式</p></figcaption>
</figure>

様式の一致する家具を製造するには[、]{.chpule2}[
]{.chpuri2}同じ様式の家具オブジェクトを生成する何らかの方法が必要となります[。]{.chpule2}[
]{.chpuri2}様式の異なる家具が配達されたら[、]{.chpule2}[
]{.chpuri2}客は怒りますよね[。]{.chpule2}[ ]{.chpuri2}

<figure class="image">
<img
src="../../images/patterns/content/abstract-factory/abstract-factory-comic-1-jacbef.png?id=bd741d9d635376ca351acbb0322ecd06"
style="aspect-ratio: 2;"
srcset="/images/patterns/content/abstract-factory/abstract-factory-comic-1-ja.png?id=bd741d9d635376ca351acbb0322ecd06 600w,/images/patterns/content/abstract-factory/abstract-factory-comic-1-ja-1.5x.png?id=e4d10edb4bd81fa10646cc13981d489c 900w,/images/patterns/content/abstract-factory/abstract-factory-comic-1-ja-2x.png?id=8b0023c7e839b10e6457b03be2e8b25b 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600" />
<figcaption><p>現代風ソファーは<span class="chpule2">、</span><span
class="chpuri2"> </span>ビクトリア様式の椅子とは合わない<span
class="chpule2">！</span><span class="chpuri2"> </span></p></figcaption>
</figure>

プログラムに新しい製品や製品群を追加する時[、]{.chpule2}[
]{.chpuri2}既存コードの変更はしたくありません[。]{.chpule2}[
]{.chpuri2}家具メーカーはカタログを常に更新しますが[、]{.chpule2}[
]{.chpuri2}そのたびに中核のコードを変更するのは避けたいところです[。]{.chpule2}[
]{.chpuri2}
:::

::: {.section .solution}
##  解決策 {#solution}

Abstract Factory パターンでは[、]{.chpule2}[
]{.chpuri2}まず[、]{.chpule2}[ ]{.chpuri2}家具ファミリーの個別製品[
]{.chpule1}[（]{.chpuri1}椅子[、]{.chpule2}[
]{.chpuri2}ソファー[、]{.chpule2}[
]{.chpuri2}コーヒーテーブル[）]{.chpule2}[
]{.chpuri2}ごとに[、]{.chpule2}[
]{.chpuri2}明示的にインターフェースを宣言します[。]{.chpule2}[
]{.chpuri2}様式別の製品は[、]{.chpule2}[
]{.chpuri2}これらのインターフェースに従って作成します[。]{.chpule2}[
]{.chpuri2}たとえば[、]{.chpule2}[
]{.chpuri2}椅子の異なる様式に対応する変種[
]{.chpule1}[（]{.chpuri1}バリエーション[）]{.chpule2}[
]{.chpuri2}を作るには[、]{.chpule2}[ ]{.chpuri2}`Chair`
インターフェースの実装を行い[、]{.chpule2}[
]{.chpuri2}コーヒーテーブルの変種を作るには[、]{.chpule2}[
]{.chpuri2}`Coffee­Table` インターフェースを実装する[、]{.chpule2}[
]{.chpuri2}といった具合です[。]{.chpule2}[ ]{.chpuri2}

<figure class="image">
<img
src="../../images/patterns/diagrams/abstract-factory/solution1a1ac.png?id=71f2018d8bb443b9cce90d57110675e3"
style="aspect-ratio: 1.5;"
srcset="/images/patterns/diagrams/abstract-factory/solution1.png?id=71f2018d8bb443b9cce90d57110675e3 420w,/images/patterns/diagrams/abstract-factory/solution1-1.5x.png?id=4366e971c850514cde5d33cb7956de8b 630w,/images/patterns/diagrams/abstract-factory/solution1-2x.png?id=eacec286153e058db9255d26541c0529 840w"
sizes="(max-width: 720px) 100vw, 420px" loading="lazy" width="420"
alt="Chair クラスの階層" />
<figcaption><p>同じオブジェクトの異種は全部<span
class="chpule2">、</span><span class="chpuri2">
</span>単一クラス階層に移動<span class="chpule2">。</span><span
class="chpuri2"> </span></p></figcaption>
</figure>

次に
*[抽]{.cjk}[象]{.cjk}[フ]{.cjk}[ァ]{.cjk}[ク]{.cjk}[ト]{.cjk}[リ]{.cjk}[ー]{.cjk}*
を宣言します[。]{.chpule2}[ ]{.chpuri2}これは[、]{.chpule2}[
]{.chpuri2}製品ファミリー全製品の生成メソッド[
]{.chpule1}[（]{.chpuri1}例[：]{.chpule2}[
]{.chpuri2}`create­Chair`[、]{.chpule2}[
]{.chpuri2}`create­Sofa`[、]{.chpule2}[
]{.chpuri2}`create­Coffee­Table`[）]{.chpule2}[
]{.chpuri2}を並べたインターフェースとなります[。]{.chpule2}[
]{.chpuri2}これらのメソッドは[、]{.chpule2}[
]{.chpuri2}このように抽出した`Chair`[、]{.chpule2}[
]{.chpuri2}`Sofa`[、]{.chpule2}[ ]{.chpuri2}`Coffee­Table`
といったインターフェースで表現された[、]{.chpule2}[ ]{.chpuri2}製品の
**抽象** 型を返す必要があります[。]{.chpule2}[ ]{.chpuri2}

<figure class="image">
<img
src="../../images/patterns/diagrams/abstract-factory/solution24d8e.png?id=53975d6e4714c6f942633a879f7ac571"
style="aspect-ratio: 2;"
srcset="/images/patterns/diagrams/abstract-factory/solution2.png?id=53975d6e4714c6f942633a879f7ac571 640w,/images/patterns/diagrams/abstract-factory/solution2-1.5x.png?id=581a6cdc0dcd5511f1c30e509b1d4a7f 960w,/images/patterns/diagrams/abstract-factory/solution2-2x.png?id=b21557081f75ac7b4110427e89a10378 1280w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="640"
alt="_Factory_ クラスの階層" />
<figcaption><p>個々の具象ファクトリー・クラスは<span
class="chpule2">、</span><span class="chpuri2">
</span>製品の様式に対応する<span class="chpule2">。</span><span
class="chpuri2"> </span></p></figcaption>
</figure>

では製品の様式ごとの変種はどうするか[？]{.chpule2}[
]{.chpuri2}というと[、]{.chpule2}[
]{.chpuri2}製品の様式ごとに[、]{.chpule2}[ ]{.chpuri2}`Abstract­Factory`
を基に別々のファクトリー・クラスを作成します[。]{.chpule2}[
]{.chpuri2}ファクトリーは[、]{.chpule2}[
]{.chpuri2}特定の種類の製品を返すクラスです[。]{.chpule2}[
]{.chpuri2}たとえば[、]{.chpule2}[ ]{.chpuri2}`Modern­Furniture­Factory`[
]{.chpule1}[（]{.chpuri1}現代風家具ファクトリー[）]{.chpule2}[
]{.chpuri2}は[、]{.chpule2}[ ]{.chpuri2}`Modern­Chair`[
]{.chpule1}[（]{.chpuri1}現代風の椅子[）]{.ls-05}[、]{.chpule2}[
]{.chpuri2}`Modern­Sofa`[
]{.chpule1}[（]{.chpuri1}現代風のソファー[）]{.ls-05}[、]{.chpule2}[
]{.chpuri2}そして `Modern­Coffee­Table`[
]{.chpule1}[（]{.chpuri1}現代風のコーヒーテーブル[）]{.chpule2}[
]{.chpuri2}のオブジェクトのみ作ることができます[。]{.chpule2}[
]{.chpuri2}

クライアント側のコードは[、]{.chpule2}[
]{.chpuri2}ファクトリー[、]{.chpule2}[
]{.chpuri2}製品ともそれぞれの抽象インターフェースを通して機能します[。]{.chpule2}[
]{.chpuri2}これにより[、]{.chpule2}[
]{.chpuri2}クライアントのコードに渡すファクトリーの型を変更したり[、]{.chpule2}[
]{.chpuri2}クライアントの受け取る製品の様式を変更しても[、]{.chpule2}[
]{.chpuri2}クライアント側コードは問題なく動作します[。]{.chpule2}[
]{.chpuri2}

<figure class="image">
<img
src="../../images/patterns/content/abstract-factory/abstract-factory-comic-2-ja4d96.png?id=8b7f48a9b44ee167710f566d62cdd2a8"
style="aspect-ratio: 2;"
srcset="/images/patterns/content/abstract-factory/abstract-factory-comic-2-ja.png?id=8b7f48a9b44ee167710f566d62cdd2a8 600w,/images/patterns/content/abstract-factory/abstract-factory-comic-2-ja-1.5x.png?id=79e600899d84a0a88b84e3df52d0259a 900w,/images/patterns/content/abstract-factory/abstract-factory-comic-2-ja-2x.png?id=e2a1837016e42078fe514dbc1e1761b1 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600" />
<figcaption><p>クライアントは<span class="chpule2">、</span><span
class="chpuri2">
</span>ファクトリーの実際のクラスが何かということを気にする必要はない<span
class="chpule2">。</span><span class="chpuri2"> </span></p></figcaption>
</figure>

クライアントはファクトリーに椅子を生成してほしいとします[。]{.chpule2}[
]{.chpuri2}クライアントは[、]{.chpule2}[
]{.chpuri2}ファクトリーのクラスを気にする必要もなく[、]{.chpule2}[
]{.chpuri2}どういう種類の椅子が返ってくるかを気にする必要もありません[。]{.chpule2}[
]{.chpuri2}現代風の椅子でも[、]{.chpule2}[
]{.chpuri2}ビクトリア様式の椅子でも[、]{.chpule2}[ ]{.chpuri2}`Chair`
という抽象インターフェースを使って[、]{.chpule2}[
]{.chpuri2}同じように扱います[。]{.chpule2}[
]{.chpuri2}こういうやり方に従うと[、]{.chpule2}[
]{.chpuri2}クライアントが知っておく必要があることは[、]{.chpule2}[
]{.chpuri2}椅子が何らかの方法で `sit­On`[
]{.chpule1}[（]{.chpuri1}腰掛ける[）]{.chpule2}[
]{.chpuri2}メソッドを実装している[、]{.chpule2}[
]{.chpuri2}ということだけです[。]{.chpule2}[
]{.chpuri2}また[、]{.chpule2}[
]{.chpuri2}どのような様式の椅子が返ってきたとしても[、]{.chpule2}[
]{.chpuri2}同じファクトリー・オブジェクトによって返されるソファーやコーヒーテーブルの様式は常に一致しています[。]{.chpule2}[
]{.chpuri2}

もう一つ明らかにしておくべきことがあります[。]{.chpule2}[
]{.chpuri2}クライアントが抽象インターフェースだけに依存しているとすると[、]{.chpule2}[
]{.chpuri2}実際のファクトリー・オブジェクトは何によって作成されるのでしょうか[？]{.chpule2}[
]{.chpuri2}通常[、]{.chpule2}[
]{.chpuri2}アプリケーションは[、]{.chpule2}[
]{.chpuri2}初期化段階でファクトリー・オブジェクトを一つ生成します[。]{.chpule2}[
]{.chpuri2}その少し前に[、]{.chpule2}[
]{.chpuri2}アプリは構成や環境設定を使用してファクトリーの型を選択します[。]{.chpule2}[
]{.chpuri2}
:::

::::: {.section .structure-container}
##  構造 {#structure}

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/abstract-factory/structureb809.png?id=a3112cdd98765406af94595a3c5e7762"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/diagrams/abstract-factory/structure.png?id=a3112cdd98765406af94595a3c5e7762 720w,/images/patterns/diagrams/abstract-factory/structure-1.5x.png?id=d2964e541620d9c087e693bd0e0a0836 1080w,/images/patterns/diagrams/abstract-factory/structure-2x.png?id=c4d3634ec2e74e02a0fe1a83ce9b50f6 1440w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="720"
alt="Abstract Factory デザインパターン" /><img
src="../../images/patterns/diagrams/abstract-factory/structure-indexed028d.png?id=6ae1c99cbd90cf58753c633624fb1a04"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.56;"
srcset="/images/patterns/diagrams/abstract-factory/structure-indexed.png?id=6ae1c99cbd90cf58753c633624fb1a04 700w,/images/patterns/diagrams/abstract-factory/structure-indexed-1.5x.png?id=473be2975c5716162e6969e6532210ac 1050w,/images/patterns/diagrams/abstract-factory/structure-indexed-2x.png?id=cb6d4e1e89826c42966dc7097374f889 1400w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="700"
alt="Abstract Factory デザインパターン" />
</figure>
:::

1.  **抽象製品**[ ]{.chpule1}[（]{.chpuri1}Abstract
    Product[）]{.chpule2}[ ]{.chpuri2}は[、]{.chpule2}[
    ]{.chpuri2}製品ファミリーを構成する個別の関連製品に対するインターフェースを宣言します[。]{.chpule2}[
    ]{.chpuri2}

2.  **具象製品**[ ]{.chpule1}[（]{.chpuri1}Concrete
    Product[）]{.chpule2}[ ]{.chpuri2}は[、]{.chpule2}[
    ]{.chpuri2}抽象製品の種々の変種ごとにグループ化されています[。]{.chpule2}[
    ]{.chpuri2}個々の抽象製品[
    ]{.chpule1}[（]{.chpuri1}椅子とかソファー[）]{.chpule2}[
    ]{.chpuri2}は[、]{.chpule2}[ ]{.chpuri2}全部の変種[
    ]{.chpule1}[（]{.chpuri1}ビクトリア調[、]{.chpule2}[
    ]{.chpuri2}現代風[）]{.chpule2}[
    ]{.chpuri2}において実装されている必要があります[。]{.chpule2}[
    ]{.chpuri2}

3.  **抽象ファクトリー**[ ]{.chpule1}[（]{.chpuri1}Abstract
    Factory[）]{.chpule2}[ ]{.chpuri2}インターフェースは[、]{.chpule2}[
    ]{.chpuri2}抽象製品のそれぞれを生成するメソッドの集合です[。]{.chpule2}[
    ]{.chpuri2}

4.  **具象ファクトリー**[ ]{.chpule1}[（]{.chpuri1}Concrete
    Factory[）]{.chpule2}[ ]{.chpuri2}は[、]{.chpule2}[
    ]{.chpuri2}抽象ファクトリーの生成メソッドを実装します[。]{.chpule2}[
    ]{.chpuri2}個々の具象ファクトリーは[、]{.chpule2}[
    ]{.chpuri2}特定の異種[ ]{.chpule1}[（]{.chpuri1}様式[）]{.chpule2}[
    ]{.chpuri2}に対応しており[、]{.chpule2}[
    ]{.chpuri2}そのような異種の製品のみを作成します[。]{.chpule2}[
    ]{.chpuri2}

5.  具象ファクトリーは[、]{.chpule2}[
    ]{.chpuri2}具象製品をインスタンス化しますが[、]{.chpule2}[
    ]{.chpuri2}生成メソッドのシグネチャーは[、]{.chpule2}[
    ]{.chpuri2}*[抽]{.cjk}[象]{.cjk}*
    製品である必要があります[。]{.chpule2}[
    ]{.chpuri2}そうすることで[、]{.chpule2}[
    ]{.chpuri2}クライアント側コードは[、]{.chpule2}[
    ]{.chpuri2}あるファクトリーから返される特定の変種の製品に縛られることがなくなります[。]{.chpule2}[
    ]{.chpuri2}抽象インターフェースによって通信する限り[、]{.chpule2}[
    ]{.chpuri2}**クライアント**[
    ]{.chpule1}[（]{.chpuri1}Client[）]{.chpule2}[
    ]{.chpuri2}は[、]{.chpule2}[
    ]{.chpuri2}いかなる具象ファクトリーや製品変種も扱うことができます[。]{.chpule2}[
    ]{.chpuri2}
::::
:::::

::: {.section .pseudocode}
##  擬似コード {#pseudocode}

ここでは[、]{.chpule2}[ ]{.chpuri2}**Abstract Factory**
パターンをどう使用すれば[、]{.chpule2}[ ]{.chpuri2}クライアントを具象 UI
クラスに結合することなくプラットフォーム互換の UI
部品の生成に利用できるかを説明します[。]{.chpule2}[
]{.chpuri2}結合されてないにもかかわらず[、]{.chpule2}[
]{.chpuri2}選択したオペレーティングシステムに適した UI
部品が生成されます[。]{.chpule2}[ ]{.chpuri2}

<figure class="image">
<img
src="../../images/patterns/diagrams/abstract-factory/exampled62f.png?id=5928a61d18bf00b047463471c599100a"
style="aspect-ratio: 1.42;"
srcset="/images/patterns/diagrams/abstract-factory/example.png?id=5928a61d18bf00b047463471c599100a 640w,/images/patterns/diagrams/abstract-factory/example-1.5x.png?id=baee3bac793ec97e0ec91c49d9382e7d 960w,/images/patterns/diagrams/abstract-factory/example-2x.png?id=eb5127b1d6486f6fad73be2d5e444449 1280w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="640"
alt="Abstract Factory パターン例のクラス図" />
<figcaption><p>プラットフォーム互換 UI クラスの例<span
class="chpule2">。</span><span class="chpuri2"> </span></p></figcaption>
</figure>

プラットフォーム互換アプリケーションでは同一 UI 部品は[、]{.chpule2}[
]{.chpuri2}オペレーティングシステムにより若干見た目に違いはありますが[、]{.chpule2}[
]{.chpuri2}似たように振る舞うことになります[。]{.chpule2}[
]{.chpuri2}もっと言うと[、]{.chpule2}[ ]{.chpuri2}UI
部品を既存のオペレーティングシステムのスタイルに一致させることは[、]{.chpule2}[
]{.chpuri2}プログラマーであるあなたの責任です[。]{.chpule2}[
]{.chpuri2}Windows で走行している時に macOS
コントロールをレンダリングするのは避けたいものです[。]{.chpule2}[
]{.chpuri2}

Abstract Factory インターフェースは[、]{.chpule2}[
]{.chpuri2}クライアント側コードが種々の UI
部品を作り出すために使用できる生成メソッドを宣言します[。]{.chpule2}[
]{.chpuri2}各種オペレーティングシステムに対応した具象クラスを用意し[、]{.chpule2}[
]{.chpuri2}それぞれのオペレーティングシステムに一致した UI
部品を生成します[。]{.chpule2}[ ]{.chpuri2}

仕組みはこうです[：]{.chpule2}[
]{.chpuri2}アプリケーションは起動時に現在のオペレーティングシステムを検出します[。]{.chpule2}[
]{.chpuri2}アプリはこの情報に基づいて[、]{.chpule2}[
]{.chpuri2}オペレーティングシステムに適したファクトリー・オブジェクトを生成します[。]{.chpule2}[
]{.chpuri2}残りのコードは[、]{.chpule2}[
]{.chpuri2}このファクトリーを使用して UI 部品を生成します[。]{.chpule2}[
]{.chpuri2}これで[、]{.chpule2}[
]{.chpuri2}間違った部品が作成されるのを防ぐことができます[。]{.chpule2}[
]{.chpuri2}

この手法を使うと[、]{.chpule2}[
]{.chpuri2}クライアント側コードが[、]{.chpule2}[
]{.chpuri2}ファクトリーや UI
部品の具象クラスやに依存するのを防ぐことができます[。]{.chpule2}[
]{.chpuri2}そのためには[、]{.chpule2}[
]{.chpuri2}これらのオブジェクトが抽象インターフェースを使用して行われる必要があります[。]{.chpule2}[
]{.chpuri2}もう一つのメリットとして[、]{.chpule2}[
]{.chpuri2}将来新しいファクトリーや UI 部品が追加されても[、]{.chpule2}[
]{.chpuri2}同じクライアント側コードが使えます[。]{.chpule2}[ ]{.chpuri2}

その結果[、]{.chpule2}[ ]{.chpuri2}アプリに UI
部品の変種を追加しても[、]{.chpule2}[
]{.chpuri2}クライアント側コードには手を加える必要がなくなります[。]{.chpule2}[
]{.chpuri2}新規部品に対応した新しいファクトリークラスを書き[、]{.chpule2}[
]{.chpuri2}そのクラスを選択できるように初期化コードを少し変えるだけですみます[。]{.chpule2}[
]{.chpuri2}

<figure class="code">
<pre class="code" lang="pseudocode"><code>// 抽象ファクトリー・インターフェースは、異なる抽象的なプロダクトを返す一
// 連のメソッドを宣言。これらの製品はファミリーと呼ばれ、ある高レベルな
// テーマや概念に関連する。通常同一のファミリーのプロダクト同士は、協力し
// て動作可能。プロダクトのファミリーにはいくつかの異種があり、ある異種の
// プロダクトは別の異種のプロダクトとの互換性を欠く。
interface GUIFactory is
    method createButton():Button
    method createCheckbox():Checkbox


// 具象ファクトリーは、同一の異種に属するプロダクトのファミリーを生成する。
// ファクトリーにより、生成されるプロダクト同士が互換であることが保証さ
// れる。具象ファクトリーのメソッドのシグネチャーは、抽象プロダクトの返却
// を示唆しているが、メソッド内部では具象プロダクトのインスタンスが生成さ
// れる。
class WinFactory implements GUIFactory is
    method createButton():Button is
        return new WinButton()
    method createCheckbox():Checkbox is
        return new WinCheckbox()

// 各具象ファクトリーは、プロダクトの異種に対応。
class MacFactory implements GUIFactory is
    method createButton():Button is
        return new MacButton()
    method createCheckbox():Checkbox is
        return new MacCheckbox()


// プロダクト・ファミリー中の個別のプロダクトは、それぞれその基底インター
// フェースを持つ。プロダクトの全異種は、このインターフェースを実装しなけ
// ればならない。
interface Button is
    method paint()

// 具象プロダクトは、対応する具象ファクトリーが生成する。
class WinButton implements Button is
    method paint() is
        // ボタンを Windows 風に描画。

class MacButton implements Button is
    method paint() is
        // ボタンを macOS 風に描画。

// これは、もう一つのプロダクトの基底インターフェース。全プロダクトは互い
// にやりとりできるが、同一の異種のプロダクト同士間でのみ適切な動作が可能。
interface Checkbox is
    method paint()

class WinCheckbox implements Checkbox is
    method paint() is
        // チェックボックスを Windows 風に描画。

class MacCheckbox implements Checkbox is
    method paint() is
        // チェックボックスを macOS 風に描画。


// クライアント・コードは、ファクトリーやプロダクトと、抽象型、つまり、
// GUIFactory、Button、Checkbox を介してのみ動作する。これにより、ど
// んなファクトリーやプロダクトのサブクラスでもクライアント・コードに機能
// を損なわずに渡すことが可能。
class Application is
    private field factory: GUIFactory
    private field button: Button
    constructor Application(factory: GUIFactory) is
        this.factory = factory
    method createUI() is
        this.button = factory.createButton()
    method paint() is
        button.paint()


// アプリケーションは、現在の構成や環境設定に応じてファクトリーの型を選び、
// 実行時に（通常は初期化処理中に）生成。
class ApplicationConfigurator is
    method main() is
        config = readApplicationConfigFile()

        if (config.OS == &quot;Windows&quot;) then
            factory = new WinFactory()
        else if (config.OS == &quot;Mac&quot;) then
            factory = new MacFactory()
        else
            throw new Exception(&quot;Error! Unknown operating system.&quot;)

        Application app = new Application(factory)</code></pre>
</figure>
:::

:::::::: {.section .applicability-container}
##  適応性 {#applicability}

::::::: applicability
::: applicability-problem
関連する製品の集まりである様々な変種に対応したいが[、]{.chpule2}[
]{.chpuri2}製品の具象クラス[
]{.chpule1}[（]{.chpuri1}それは設計段階では未知かもしれません[）]{.chpule2}[
]{.chpuri2}に依存させたくない場合に[、]{.chpule2}[ ]{.chpuri2}Abstract
Factory を使用します[。]{.chpule2}[ ]{.chpuri2}あるいは[、]{.chpule2}[
]{.chpuri2}単に将来の拡張に備えるために使用することもできます[。]{.chpule2}[
]{.chpuri2}
:::

::: applicability-solution
Abstract Factory では[、]{.chpule2}[
]{.chpuri2}製品ファミリーの各クラスのオブジェクトを作成するインターフェースを使用します[。]{.chpule2}[
]{.chpuri2}このインターフェースによってオブジェクトを生成する限り[、]{.chpule2}[
]{.chpuri2}すでにアプリで作成した製品と合わない間違った変種の製品が作成されることを心配することはありません[。]{.chpule2}[
]{.chpuri2}
:::

::: applicability-problem
一連の[ファクトリー・メソッド](factory-method.html)からなるクラスがあり[、]{.chpule2}[
]{.chpuri2}それが一次責任をあいまいにしてしまう場合に[、]{.chpule2}[
]{.chpuri2}Abstract Factory の使用を検討してください[。]{.chpule2}[
]{.chpuri2}
:::

::: applicability-solution
よく設計されたプログラムでは[、]{.chpule2}[
]{.chpuri2}*[そ]{.cjk}[れ]{.cjk}[ぞ]{.cjk}[れ]{.cjk}[の]{.cjk}[ク]{.cjk}[ラ]{.cjk}[ス]{.cjk}[は]{.cjk}[単]{.cjk}[一]{.cjk}[の]{.cjk}[事]{.cjk}[項]{.cjk}[に]{.cjk}[つ]{.cjk}[い]{.cjk}[て]{.cjk}[の]{.cjk}[み]{.cjk}[責]{.cjk}[任]{.cjk}[を]{.cjk}[持]{.cjk}[ち]{.cjk}[ま]{.cjk}[す]{.cjk}*[。]{.chpule2}[
]{.chpuri2}ある一つのクラスが複数の製品の型を扱う場合[、]{.chpule2}[
]{.chpuri2}ファクトリー・メソッドを抽出して独立したファクトリー・クラスを作るか[、]{.chpule2}[
]{.chpuri2}完全な Abstract Factory
の実装を行うことをお勧めします[。]{.chpule2}[ ]{.chpuri2}
:::
:::::::
::::::::

::: {.section .checklist}
##  実装方法 {#checklist}

1.  明確に区別できる製品の型と[、]{.chpule2}[
    ]{.chpuri2}製品型の変種の表を作ります[。]{.chpule2}[ ]{.chpuri2}

2.  全部の製品型について[、]{.chpule2}[
    ]{.chpuri2}抽象製品インターフェースを宣言します[。]{.chpule2}[
    ]{.chpuri2}そして[、]{.chpule2}[
    ]{.chpuri2}全インターフェースを実装した製品の具象クラスを作ります[。]{.chpule2}[
    ]{.chpuri2}

3.  全抽象製品型に対する生成メソッドを含んだ抽象ファクトリー・インターフェースを宣言します[。]{.chpule2}[
    ]{.chpuri2}

4.  各変種ごとに[、]{.chpule2}[
    ]{.chpuri2}具象ファクトリー・クラスを実装します[。]{.chpule2}[
    ]{.chpuri2}

5.  アプリのどこかに[、]{.chpule2}[
    ]{.chpuri2}ファクトリー初期化コードを追加します[。]{.chpule2}[
    ]{.chpuri2}そこでアプリケーションの構成か環境設定に従って具象ファクトリー・クラスのインスタンスを作成します[。]{.chpule2}[
    ]{.chpuri2}製品を構築するすべてのクラスに[、]{.chpule2}[
    ]{.chpuri2}このファクトリー・オブジェクトを渡します[。]{.chpule2}[
    ]{.chpuri2}

6.  コードをスキャンして[、]{.chpule2}[
    ]{.chpuri2}製品クラスのコンストラクターへの直接の呼び出しを探します[。]{.chpule2}[
    ]{.chpuri2}これらの呼び出しを[、]{.chpule2}[
    ]{.chpuri2}ファクトリー・オブジェクトに対する適切な作成メソッドの呼び出しと置き換えます[。]{.chpule2}[
    ]{.chpuri2}
:::

:::::: {.section .pros-cons}
##  長所と短所 {#pros-cons}

::::: row
::: col-sm-6
-    ファクトリーから得られる製品同士は[、]{.chpule2}[
    ]{.chpuri2}互換であることが保証される[。]{.chpule2}[ ]{.chpuri2}
-    具象製品とクライアント側コードの密結合を防止できる[。]{.chpule2}[
    ]{.chpuri2}
-   
    *[単]{.cjk}[一]{.cjk}[責]{.cjk}[任]{.cjk}[の]{.cjk}[原]{.cjk}[則]{.cjk}*[。]{.chpule2}[
    ]{.chpuri2}製品作成コードが一箇所にまとめられ[、]{.chpule2}[
    ]{.chpuri2}保守が容易になる[。]{.chpule2}[ ]{.chpuri2}
-   
    *[開]{.cjk}[放]{.cjk}[閉]{.cjk}[鎖]{.cjk}[の]{.cjk}[原]{.cjk}[則]{.cjk}*[。]{.chpule2}[
    ]{.chpuri2}製品の新しい変種を導入しても[、]{.chpule2}[
    ]{.chpuri2}既存クライアント側コードは動作する[。]{.chpule2}[
    ]{.chpuri2}
:::

::: col-sm-6
-    パターンの使用に伴い[、]{.chpule2}[
    ]{.chpuri2}多数の新規インターフェースやクラスが導入され[、]{.chpule2}[
    ]{.chpuri2}コードが必要以上に複雑になる可能性あり[。]{.chpule2}[
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

-   [Builder](builder.html) は[、]{.chpule2}[
    ]{.chpuri2}複雑なオブジェクトを段階的に構築することに重点を置いています[。]{.chpule2}[
    ]{.chpuri2}[Abstract Factory](abstract-factory.html)
    は[、]{.chpule2}[
    ]{.chpuri2}関連するオブジェクトの集団を作成することに特化しています[。]{.chpule2}[
    ]{.chpuri2}*Abstract Factory*
    がすぐにプロダクトを返すのに対して[、]{.chpule2}[
    ]{.chpuri2}*Builder* ではプロダクトの取得前に[、]{.chpule2}[
    ]{.chpuri2}いくつかの追加の構築のステップを踏まなければなりません[。]{.chpule2}[
    ]{.chpuri2}

-   [Abstract Factory](abstract-factory.html) クラスは[、]{.chpule2}[
    ]{.chpuri2}多くの場合 [Factory Methods](factory-method.html)
    の集まりですが[、]{.chpule2}[ ]{.chpuri2}[Prototype](prototype.html)
    を使ってメソッドを書くこともできます[。]{.chpule2}[ ]{.chpuri2}

-   サブシステムがオブジェクトを作成する方法をクライアントから隠蔽することだけが目的なら[、]{.chpule2}[
    ]{.chpuri2}[Abstract Factory](abstract-factory.html) を
    [Facade](facade.html) の代わりに使えます[。]{.chpule2}[ ]{.chpuri2}

-   [Abstract Factory](abstract-factory.html) は[、]{.chpule2}[
    ]{.chpuri2}[Bridge](bridge.html) と一緒に使用できます[。]{.chpule2}[
    ]{.chpuri2}この組み合わせは[、]{.chpule2}[ ]{.chpuri2}*Bridge*
    によって定義された抽象化層のいくつかが特定の実装としか動作しない場合に便利です[。]{.chpule2}[
    ]{.chpuri2}この場合[、]{.chpule2}[ ]{.chpuri2}*Abstract Factory*
    はこれらの関係をカプセル化し[、]{.chpule2}[
    ]{.chpuri2}クライアント・コードから複雑さを隠すことができます[。]{.chpule2}[
    ]{.chpuri2}

-   [Abstract Factories](abstract-factory.html)[、]{.chpule2}[
    ]{.chpuri2}[Builders](builder.html)[、]{.chpule2}[
    ]{.chpuri2}[Prototypes](prototype.html) はどれも
    [Singletons](singleton.html) で実装可能です[。]{.chpule2}[
    ]{.chpuri2}
:::

::: {.section .implementations}
##  コード例 {#implementations}

[![Abstract Factory を C#
で](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](abstract-factory/csharp/example.html "Abstract Factory を C# で"){.prog-lang-link}
[![Abstract Factory を C++
で](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](abstract-factory/cpp/example.html "Abstract Factory を C++ で"){.prog-lang-link}
[![Abstract Factory を Go
で](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](abstract-factory/go/example.html "Abstract Factory を Go で"){.prog-lang-link}
[![Abstract Factory を Java
で](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](abstract-factory/java/example.html "Abstract Factory を Java で"){.prog-lang-link}
[![Abstract Factory を PHP
で](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](abstract-factory/php/example.html "Abstract Factory を PHP で"){.prog-lang-link}
[![Abstract Factory を Python
で](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](abstract-factory/python/example.html "Abstract Factory を Python で"){.prog-lang-link}
[![Abstract Factory を Ruby
で](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](abstract-factory/ruby/example.html "Abstract Factory を Ruby で"){.prog-lang-link}
[![Abstract Factory を Rust
で](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](abstract-factory/rust/example.html "Abstract Factory を Rust で"){.prog-lang-link}
[![Abstract Factory を Swift
で](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](abstract-factory/swift/example.html "Abstract Factory を Swift で"){.prog-lang-link}
[![Abstract Factory を TypeScript
で](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](abstract-factory/typescript/example.html "Abstract Factory を TypeScript で"){.prog-lang-link}
:::

::: {.section .extras}
##  追記 {#extras}

-   多種のファクトリー・パターンの比較と概念については[ファクトリー比較](factory-comparison.html)を参照してください[。]{.chpule2}[
    ]{.chpuri2}
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

[ファクトリーの比較 []{.fa
.fa-arrow-right}](factory-comparison.html){.btn .btn-primary rel="next"}
:::

::: prev
#### 戻る

[[]{.fa .fa-arrow-left} Factory Method](factory-method.html){.btn
.btn-default rel="prev"}
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
