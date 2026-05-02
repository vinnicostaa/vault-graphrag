::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} /
[デザインパターン](../design-patterns.html){.type} /
[生成に関するパターン](creational-patterns.html){.type}
:::

# Factory Method {#factory-method .title}

::: aka
別名：[Virtual
Constructor、]{style="display:inline-block"}[ファクトリー・メソッド、]{style="display:inline-block"}[バーチャル・コンストラクター、]{style="display:inline-block"}[仮想コンストラクター]{style="display:inline-block"}
:::

::: {.section .intent}
##  一言でいうと {#intent}

**Factory Method**[
]{.chpule1}[（]{.chpuri1}ファクトリー・メソッド[）]{.chpule2}[
]{.chpuri2}は[、]{.chpule2}[
]{.chpuri2}生成に関するデザインパターンの一つで[、]{.chpule2}[
]{.chpuri2}スーパークラスでオブジェクトを作成するためのインターフェースが決まっています[。]{.chpule2}[
]{.chpuri2}しかし[、]{.chpule2}[
]{.chpuri2}サブクラスでは作成されるオブジェクトの型を変更することができます[。]{.chpule2}[
]{.chpuri2}

<figure class="image">
<img
src="../../images/patterns/content/factory-method/factory-method-ja8b25.png?id=dd17a49dc710bc5b21dd0d631036fe7a"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/factory-method/factory-method-ja.png?id=dd17a49dc710bc5b21dd0d631036fe7a 640w,/images/patterns/content/factory-method/factory-method-ja-1.5x.png?id=220d1dc24f80752fd890cf2918bd0218 960w,/images/patterns/content/factory-method/factory-method-ja-2x.png?id=756b9597068c985ff11c1c8acea208ad 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Factory Method パターン" />
</figure>
:::

::: {.section .problem}
##  問題 {#problem}

物流管理アプリケーションを作成するとします[。]{.chpule2}[
]{.chpuri2}アプリの最初のバージョンは[、]{.chpule2}[
]{.chpuri2}トラック輸送のみを処理できます[。]{.chpule2}[
]{.chpuri2}コードの大部分は `Truck`[
]{.chpule1}[（]{.chpuri1}トラック[）]{.chpule2}[
]{.chpuri2}クラス内に存在します[。]{.chpule2}[ ]{.chpuri2}

しばらくして[、]{.chpule2}[ ]{.chpuri2}このアプリは[、]{.chpule2}[
]{.chpuri2}かなり人気が出ます[。]{.chpule2}[
]{.chpuri2}海運会社から[、]{.chpule2}[
]{.chpuri2}アプリに海上物流を組み込んでほしいという要望が毎日数十件寄せられるようになりました[。]{.chpule2}[
]{.chpuri2}

<figure class="image">
<img
src="../../images/patterns/diagrams/factory-method/problem1-ja8b0f.png?id=cdca2c7383a6668c9e642c2fa07a5cd1"
style="aspect-ratio: 2.4;"
srcset="/images/patterns/diagrams/factory-method/problem1-ja.png?id=cdca2c7383a6668c9e642c2fa07a5cd1 600w,/images/patterns/diagrams/factory-method/problem1-ja-1.5x.png?id=dc535f87c86177d3042d800402cdfaef 900w,/images/patterns/diagrams/factory-method/problem1-ja-2x.png?id=575b169893b77da4202fd30f8fb034ed 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="新規の運輸クラスのプログラムへの追加による弊害" />
<figcaption><p>コードの大部分がすでに既存のクラスと密に結合されていると<span
class="chpule2">、</span><span class="chpuri2">
</span>新しいクラスのプログラムへの追加はあまり容易ではない<span
class="chpule2">。</span><span class="chpuri2"> </span></p></figcaption>
</figure>

素晴らしいことです[！]{.chpule2}[
]{.chpuri2}でもコードの方はどうしましょう[？]{.chpule2}[
]{.chpuri2}現状では[、]{.chpule2}[
]{.chpuri2}コードのほとんどは[、]{.chpule2}[ ]{.chpuri2}`Truck`
クラスに結合されています[。]{.chpule2}[ ]{.chpuri2}アプリに `Ships`[
]{.chpule1}[（]{.chpuri1}船[）]{.chpule2}[
]{.chpuri2}を追加するには[、]{.chpule2}[
]{.chpuri2}コードの大部分に手を入れる必要があります[。]{.chpule2}[
]{.chpuri2}さらに[、]{.chpule2}[
]{.chpuri2}後でアプリに別の種類の輸送手段を追加することにした場合[、]{.chpule2}[
]{.chpuri2}おそらくこの変更作業すべてを再度行うことになるでしょう[。]{.chpule2}[
]{.chpuri2}

結果として[、]{.chpule2}[
]{.chpuri2}運輸オブジェクトのクラスに応じてアプリの動作を切り替える条件文だらけの[、]{.chpule2}[
]{.chpuri2}かなり厄介なコードができてしまうでしょう[。]{.chpule2}[
]{.chpuri2}
:::

::: {.section .solution}
##  解決策 {#solution}

Factory Method パターンに従うと[、]{.chpule2}[ ]{.chpuri2}`new`
演算子を使用した直接のオブジェクト作成呼び出しを[、]{.chpule2}[
]{.chpuri2}特別な*[フ]{.cjk}[ァ]{.cjk}[ク]{.cjk}[ト]{.cjk}[リ]{.cjk}[ー]{.cjk}*・メソッドへの呼び出しで置き換えます[。]{.chpule2}[
]{.chpuri2}ご心配なく[！]{.chpule2}[
]{.chpuri2}それでもオブジェクトは[、]{.chpule2}[ ]{.chpuri2}`new`
演算子で作成されます[。]{.chpule2}[
]{.chpuri2}ただそれはファクトリー・メソッド内で呼び出されます[。]{.chpule2}[
]{.chpuri2}ファクトリー・メソッドから返されるオブジェクトはよく*[プ]{.cjk}[ロ]{.cjk}[ダ]{.cjk}[ク]{.cjk}[ト]{.cjk}*と呼ばれます[。]{.chpule2}[
]{.chpuri2}

<figure class="image">
<img
src="../../images/patterns/diagrams/factory-method/solution16ec2.png?id=fc756d2af296b5b4d482e548214d08ef"
style="aspect-ratio: 2.3;"
srcset="/images/patterns/diagrams/factory-method/solution1.png?id=fc756d2af296b5b4d482e548214d08ef 620w,/images/patterns/diagrams/factory-method/solution1-1.5x.png?id=22d3b6bb74e966d1cb3a4d8f380cefa3 930w,/images/patterns/diagrams/factory-method/solution1-2x.png?id=c482b3cd7a4d8dd73b4c8c12dfcaa03c 1240w"
sizes="(max-width: 720px) 100vw, 620px" loading="lazy" width="620"
alt="クリエーター・クラスの構造" />
<figcaption><p>サブクラスは<span class="chpule2">、</span><span
class="chpuri2">
</span>ファクトリー・メソッドから返されるオブジェクトのクラスを変更できる<span
class="chpule2">。</span><span class="chpuri2"> </span></p></figcaption>
</figure>

一見すると[、]{.chpule2}[
]{.chpuri2}この変更は無意味に見えるかもしれません[。]{.chpule2}[
]{.chpuri2}コンストラクターの呼び出しをプログラムのある部分から別の部分に移動しただけです[。]{.chpule2}[
]{.chpuri2}しかし[、]{.chpule2}[ ]{.chpuri2}これにより[、]{.chpule2}[
]{.chpuri2}サブクラスのファクトリー・メソッドを上書きさえすれば作成されるプロダクトのクラスの変更が可能というメリットがあります[。]{.chpule2}[
]{.chpuri2}

しかし[、]{.chpule2}[ ]{.chpuri2}わずかな制限があります[：]{.chpule2}[
]{.chpuri2}これらのプロダクトに共通のベースクラスまたはインターフェースがある場合にのみ[、]{.chpule2}[
]{.chpuri2}サブクラスは[、]{.chpule2}[
]{.chpuri2}異なる型のプロダクトを返すことができます[。]{.chpule2}[
]{.chpuri2}また[、]{.chpule2}[
]{.chpuri2}ベースクラス内のファクトリー・メソッドの戻り値の型は[、]{.chpule2}[
]{.chpuri2}このインタフェースとして宣言されている必要があります[。]{.chpule2}[
]{.chpuri2}

<figure class="image">
<img
src="../../images/patterns/diagrams/factory-method/solution2-ja218d.png?id=526049fb709b20777f403a7fe5bf7ef8"
style="aspect-ratio: 1.96;"
srcset="/images/patterns/diagrams/factory-method/solution2-ja.png?id=526049fb709b20777f403a7fe5bf7ef8 490w,/images/patterns/diagrams/factory-method/solution2-ja-1.5x.png?id=eafbb366c2942831df9ac5a9d8cb5306 735w,/images/patterns/diagrams/factory-method/solution2-ja-2x.png?id=a8f8032bcb8d2f118af9b932eb72576d 980w"
sizes="(max-width: 720px) 100vw, 490px" loading="lazy" width="490"
alt="プロダクト階層の構造" />
<figcaption><p>すべてのプロダクトは<span class="chpule2">、</span><span
class="chpuri2"> </span>同じインターフェースに従う必要がある<span
class="chpule2">。</span><span class="chpuri2"> </span></p></figcaption>
</figure>

たとえば[、]{.chpule2}[ ]{.chpuri2}`deliver`[
]{.chpule1}[（]{.chpuri1}配送[）]{.chpule2}[
]{.chpuri2}というメソッドが宣言された `Transport`[
]{.chpule1}[（]{.chpuri1}運送[）]{.chpule2}[
]{.chpuri2}というインターフェースを[、]{.chpule2}[ ]{.chpuri2}`Truck` と
`Ship` の両クラスが実装します[。]{.chpule2}[ ]{.chpuri2}Truck
は陸上で[、]{.chpule2}[ ]{.chpuri2}Ship
は海上で貨物を届けるというように[、]{.chpule2}[
]{.chpuri2}それぞれのクラスでこのメソッドの実装が異なっています[。]{.chpule2}[
]{.chpuri2}`Road­Logistics`[
]{.chpule1}[（]{.chpuri1}陸路物流[）]{.chpule2}[
]{.chpuri2}クラスのファクトリー・メソッドは Truck
オブジェクトを返し[、]{.chpule2}[ ]{.chpuri2}`Sea­Logistics`[
]{.chpule1}[（]{.chpuri1}海上物流[）]{.chpule2}[
]{.chpuri2}クラスのファクトリー・メソッドは Ship
を返します[。]{.chpule2}[ ]{.chpuri2}

<figure class="image">
<img
src="../../images/patterns/diagrams/factory-method/solution3-ja458d.png?id=77439d4f3ddb8d63d460b21349179e5f"
style="aspect-ratio: 1.83;"
srcset="/images/patterns/diagrams/factory-method/solution3-ja.png?id=77439d4f3ddb8d63d460b21349179e5f 640w,/images/patterns/diagrams/factory-method/solution3-ja-1.5x.png?id=9bf33aafead34f2916b72a934a352f20 960w,/images/patterns/diagrams/factory-method/solution3-ja-2x.png?id=af8562b45487168d7b84ccb3e4f1cf42 1280w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="640"
alt="Factory Method パターン適用後のコードの構造" />
<figcaption><p>すべてのプロダクト・クラスが共通のインターフェースを実装している限り<span
class="chpule2">、</span><span class="chpuri2">
</span>どのクラスのオブジェクトでも問題なくクライアント・コードに渡すことができる<span
class="chpule2">。</span><span class="chpuri2"> </span></p></figcaption>
</figure>

ファクトリー・メソッドを使用するコード[
]{.chpule1}[（]{.chpuri1}*[ク]{.cjk}[ラ]{.cjk}[イ]{.cjk}[ア]{.cjk}[ン]{.cjk}[ト]{.cjk}*コードとよく呼ばれる[）]{.chpule2}[
]{.chpuri2}からは[、]{.chpule2}[
]{.chpuri2}様々なサブクラスが返す実際のプロダクトの間に違いは見られません[。]{.chpule2}[
]{.chpuri2}クライアントはすべての製品を抽象的な `Transport`
として扱います[。]{.chpule2}[ ]{.chpuri2}クライアントは[、]{.chpule2}[
]{.chpuri2}すべての `Transport` オブジェクトが `deliver`
メソッドを持っていることは知っていますが[、]{.chpule2}[
]{.chpuri2}それが厳密にどのように振る舞うかは[、]{.chpule2}[
]{.chpuri2}クライアントにとっては重要なことではありません[。]{.chpule2}[
]{.chpuri2}
:::

::::: {.section .structure-container}
##  構造 {#structure}

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/factory-method/structure78c3.png?id=4cba0803f42517cfe8548c9bc7dc4c9b"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.74;"
srcset="/images/patterns/diagrams/factory-method/structure.png?id=4cba0803f42517cfe8548c9bc7dc4c9b 660w,/images/patterns/diagrams/factory-method/structure-1.5x.png?id=ece8300890afc14494770a6b6d370428 990w,/images/patterns/diagrams/factory-method/structure-2x.png?id=9ea3aa8a47f8be22e12e523c15b448fd 1320w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="660"
alt="Factory Method パターンの構造" /><img
src="../../images/patterns/diagrams/factory-method/structure-indexed6136.png?id=4c603207859ca1f939b17b60a3a2e9e0"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.74;"
srcset="/images/patterns/diagrams/factory-method/structure-indexed.png?id=4c603207859ca1f939b17b60a3a2e9e0 660w,/images/patterns/diagrams/factory-method/structure-indexed-1.5x.png?id=626b6d7f577e1c265cce244678b2cf7f 990w,/images/patterns/diagrams/factory-method/structure-indexed-2x.png?id=c794e4f2d05013fb176464a1d1a5d7ab 1320w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="660"
alt="Factory Method パターンの構造" />
</figure>
:::

1.  **プロダクト**[ ]{.chpule1}[（]{.chpuri1}Product[）]{.chpule2}[
    ]{.chpuri2}は[、]{.chpule2}[
    ]{.chpuri2}クリエーターとそのサブクラスによって生成されるすべてのオブジェクトに共通なインターフェースを宣言します[。]{.chpule2}[
    ]{.chpuri2}

2.  **具象プロダクト**[ ]{.chpule1}[（]{.chpuri1}Concrete
    Product[）]{.chpule2}[ ]{.chpuri2}は[、]{.chpule2}[
    ]{.chpuri2}プロダクトのインターフェースの種々の異なる実装です[。]{.chpule2}[
    ]{.chpuri2}

3.  **クリエーター**[ ]{.chpule1}[（]{.chpuri1}Creator[）]{.chpule2}[
    ]{.chpuri2}クラスは[、]{.chpule2}[
    ]{.chpuri2}新しいプロダクトのオブジェクトを返すファクトリー・メソッドを宣言します[。]{.chpule2}[
    ]{.chpuri2}このメソッドの戻り値の型がプロダクトのインターフェースと一致していることが要点です[。]{.chpule2}[
    ]{.chpuri2}

    ファクトリー・メソッドを `abstract` と宣言して[、]{.chpule2}[
    ]{.chpuri2}すべてのサブクラスに独自のメソッドの実装を強制することができます[。]{.chpule2}[
    ]{.chpuri2}代わりの方法としては[、]{.chpule2}[
    ]{.chpuri2}基底クラスのファクトリー・メソッドで[、]{.chpule2}[
    ]{.chpuri2}何らかのデフォルトのプロダクト型を返すようにもできます[。]{.chpule2}[
    ]{.chpuri2}

    その名前にもかかわらず[、]{.chpule2}[
    ]{.chpuri2}プロダクトの作成はクリエーターの主要任務では**ありません**[。]{.chpule2}[
    ]{.chpuri2}通常[、]{.chpule2}[
    ]{.chpuri2}クリエーター・クラスはすでにプロダクトに関連するいくつかの中核となるビジネス・ロジックを持っています[。]{.chpule2}[
    ]{.chpuri2}ファクトリー・メソッドは[、]{.chpule2}[
    ]{.chpuri2}このロジックを具象クラスから分離するのに役立ちます[。]{.chpule2}[
    ]{.chpuri2}比喩を使うとこういうことです[：]{.chpule2}[
    ]{.chpuri2}大規模ソフトウェア開発会社にはプログラマーのための研修部門があるが[、]{.chpule2}[
    ]{.chpuri2}会社全体の主な機能は[、]{.chpule2}[
    ]{.chpuri2}コードを書くことであって[、]{.chpule2}[
    ]{.chpuri2}プログラマーを育成することではない[。]{.chpule2}[
    ]{.chpuri2}

4.  **具象クリエーター**[ ]{.chpule1}[（]{.chpuri1}Concrete
    Creator[）]{.chpule2}[ ]{.chpuri2}は[、]{.chpule2}[
    ]{.chpuri2}異なる型のプロダクトを返すように[、]{.chpule2}[
    ]{.chpuri2}基底クラスのファクトリー・メソッドを上書きします[。]{.chpule2}[
    ]{.chpuri2}

    ファクトリー・メソッドは[、]{.chpule2}[
    ]{.chpuri2}常に新しいインスタンスを**作成**する必要はないことに注意してください[。]{.chpule2}[
    ]{.chpuri2}キャッシュ[、]{.chpule2}[
    ]{.chpuri2}オブジェクト・プール[、]{.chpule2}[
    ]{.chpuri2}その他の方法で既存のオブジェクトを返してもかまいません[。]{.chpule2}[
    ]{.chpuri2}
::::
:::::

::: {.section .pseudocode}
##  擬似コード {#pseudocode}

この例では[、]{.chpule2}[ ]{.chpuri2}クライアント・コードを具体的な UI
クラスと結合することなく[、]{.chpule2}[
]{.chpuri2}プラットフォーム互換な UI 部品を作成するために[、]{.chpule2}[
]{.chpuri2}**Factory Method** を適用する方法を説明します[。]{.chpule2}[
]{.chpuri2}

<figure class="image">
<img
src="../../images/patterns/diagrams/factory-method/examplec59f.png?id=67db9a5cb817913444efcb1c067c9835"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/diagrams/factory-method/example.png?id=67db9a5cb817913444efcb1c067c9835 640w,/images/patterns/diagrams/factory-method/example-1.5x.png?id=921d97276e5e2fd8e64609c9cfa021e7 960w,/images/patterns/diagrams/factory-method/example-2x.png?id=a2470830778e318263155000dbdc5870 1280w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="640"
alt="Factory Method パターン例の構造" />
<figcaption><p>プラットフォーム互換ダイアログの例</p></figcaption>
</figure>

基底クラスである `Dialog`[
]{.chpule1}[（]{.chpuri1}ダイアログ[）]{.chpule2}[
]{.chpuri2}クラスは[、]{.chpule2}[
]{.chpuri2}ウィンドウを描画するために異なる UI
部品を使用します[。]{.chpule2}[
]{.chpuri2}オペレーティングシステムよって[、]{.chpule2}[
]{.chpuri2}これらの UI
部品は少し異なって見えるかもしれませんが[、]{.chpule2}[
]{.chpuri2}それでも一貫して動作する必要があります[。]{.chpule2}[
]{.chpuri2}Windows 上でのボタンは[、]{.chpule2}[ ]{.chpuri2}Linux
上でもボタンです[。]{.chpule2}[ ]{.chpuri2}

Factory Method を適用すると[、]{.chpule2}[
]{.chpuri2}オペレーティングシステムごとにダイアログのロジックを書き直す必要はありません[。]{.chpule2}[
]{.chpuri2}ダイアログの基底クラス内でボタンを生成するファクトリー・メソッドを宣言しておけば[、]{.chpule2}[
]{.chpuri2}ファクトリー・メソッドから Windows
形式のボタンを返すようにした[、]{.chpule2}[
]{.chpuri2}ダイアログのサブクラスを後で作成できます[。]{.chpule2}[
]{.chpuri2}サブクラスは基底クラスからダイアログのコードの大部分を継承しますが[、]{.chpule2}[
]{.chpuri2}ファクトリー・メソッドのおかげで[、]{.chpule2}[
]{.chpuri2}Windows のボタンをスクリーン上に描画できます[。]{.chpule2}[
]{.chpuri2}

このパターンが機能するためには[、]{.chpule2}[
]{.chpuri2}ダイアログの基底クラスは抽象的ボタン[
]{.chpule1}[（]{.chpuri1}すべての具象的ボタンが従う基底クラスかインターフェース[）]{.chpule2}[
]{.chpuri2}と機能する必要があります[。]{.chpule2}[
]{.chpuri2}こうしておけば[、]{.chpule2}[
]{.chpuri2}どのような種類のボタンであっても[、]{.chpule2}[
]{.chpuri2}ダイアログのコードは機能し続けます[。]{.chpule2}[ ]{.chpuri2}

勿論このやり方は[、]{.chpule2}[ ]{.chpuri2}他の UI
部品にも適用できます[。]{.chpule2}[ ]{.chpuri2}ただし[、]{.chpule2}[
]{.chpuri2}ダイアログに新しいファクトリ・メソッドを追加するたびに[、]{.chpule2}[
]{.chpuri2}[Abstract Factory](abstract-factory.html)[
]{.chpule1}[（]{.chpuri1}抽象ファクトリー[）]{.chpule2}[
]{.chpuri2}に近いものとなっていきます[。]{.chpule2}[
]{.chpuri2}でも大丈夫です[。]{.chpule2}[
]{.chpuri2}このパターンについては後で説明します[。]{.chpule2}[
]{.chpuri2}

<figure class="code">
<pre class="code" lang="pseudocode"><code>// クリエーター・クラスは、ファクトリー・メソッドを宣言。これは、プロダク
// トのクラスのオブジェクトを返さなければならない。クリエーターのサブクラ
// スは通常このメソッドを実装する。
class Dialog is
    // クリエーターは、また、ファクトリー・メソッドの何らかのデフォルトの
    // 実装を提供してもよい。
    abstract method createButton():Button

    // その名前にもかかわらず、プロダクトの作成はクリエーターの主要任務で
    // はない。通常、クリエーター・クラスにはすでにファクトリー・メソッド
    // から返されるプロダクト・オブジェクトに依存した何らかの中核となるビ
    // ジネス・ロジックが存在している。サブクラスは、ファクトリー・メソッ
    // ドを上書きし、異なる種類のプロダクトを返すようにすることで、間接的
    // にビジネス・ロジックを変更可能。
    method render() is
        // プロダクト・オブジェクトを作成するためにファクトリー・メソッド
        // を呼ぶ。
        Button okButton = createButton()
        // そして今、そのプロダクトを使用。
        okButton.onClick(closeDialog)
        okButton.render()


// 具象クリエーターはファクトリー・メソッドを上書きして、結果のプロダクト
// の型を変更。
class WindowsDialog extends Dialog is
    method createButton():Button is
        return new WindowsButton()

class WebDialog extends Dialog is
    method createButton():Button is
        return new HTMLButton()


// プロダクト・インターフェースは、全具象プロダクトが実装しなければならな
// い操作を宣言。
interface Button is
    method render()
    method onClick(f)

// 具象プロダクトは、プロダクト・インターフェースのさまざまな実装を行う。
class WindowsButton implements Button is
    method render(a, b) is
        // ボタンを Windows 風に描画。
    method onClick(f) is
        // OS 本来のクリック・イベントと結びつける。

class HTMLButton implements Button is
    method render(a, b) is
        // ボタンを HTML で表現したものを返す。
    method onClick(f) is
        // ウェブブラウザーのクリック・イベントと結びつける。


class Application is
    field dialog: Dialog

    // アプリケーションは、現在の構成または環境設定に依存して、クリエー
    // ターの型を決定。
    method initialize() is
        config = readApplicationConfigFile()

        if (config.OS == &quot;Windows&quot;) then
            dialog = new WindowsDialog()
        else if (config.OS == &quot;Web&quot;) then
            dialog = new WebDialog()
        else
            throw new Exception(&quot;Error! Unknown operating system.&quot;)

    // クライアント・コードは、基底インターフェースを通してではあるが、具
    // 象クリエーターのインスタンスに対して動作する。クライアントが基底イ
    // ンターフェースを介してクリエーターと作業を続けている限り、いかなる
    // クリエーターのサブクラスでも渡すことが可能。
    method main() is
        this.initialize()
        dialog.render()</code></pre>
</figure>
:::

:::::::::: {.section .applicability-container}
##  適応性 {#applicability}

::::::::: applicability
::: applicability-problem
コードが機能する対象のオブジェクトの正確な型と依存関係が前もってわからない場合に[、]{.chpule2}[
]{.chpuri2}Factory Method を使用します[。]{.chpule2}[ ]{.chpuri2}
:::

::: applicability-solution
Factory Method では[、]{.chpule2}[
]{.chpuri2}プロダクト作成のコードを実際にプロダクトを使用するコードから分離します[。]{.chpule2}[
]{.chpuri2}したがって[、]{.chpule2}[
]{.chpuri2}プロダクト作成のコードの拡張が残りのコードから独立して簡単に行えます[。]{.chpule2}[
]{.chpuri2}

たとえば[、]{.chpule2}[
]{.chpuri2}アプリに新しいプロダクトの型を追加するには[、]{.chpule2}[
]{.chpuri2}新しい作成クラスのサブクラスを作成し[、]{.chpule2}[
]{.chpuri2}その中のファクトリー・メソッドを上書きするだけですみます[。]{.chpule2}[
]{.chpuri2}
:::

::: applicability-problem
自分の書いたライブラリーやフレームワークのユーザーに内部のコンポーネントを拡張する方法を提供したい場合[、]{.chpule2}[
]{.chpuri2}Factory Method を使用します[。]{.chpule2}[ ]{.chpuri2}
:::

::: applicability-solution
ライブラリーやフレームワークのデフォルト動作を拡張する最も簡単な方法は[、]{.chpule2}[
]{.chpuri2}おそらく継承です[。]{.chpule2}[
]{.chpuri2}しかし[、]{.chpule2}[
]{.chpuri2}フレームワークは[、]{.chpule2}[
]{.chpuri2}標準コンポーネントの代わりにサブクラスを使用すべきだということをどう認識するのでしょうか[？]{.chpule2}[
]{.chpuri2}

解決策は[、]{.chpule2}[
]{.chpuri2}フレームワーク中に散らばった[、]{.chpule2}[
]{.chpuri2}コンポーネント構築コードを削減し[、]{.chpule2}[
]{.chpuri2}単一のファクトリー・メソッドに集めることです[。]{.chpule2}[
]{.chpuri2}こうすれば[、]{.chpule2}[
]{.chpuri2}コンポーネント自体を拡張することに加えて[、]{.chpule2}[
]{.chpuri2}誰でもこのメソッドを上書きできます[。]{.chpule2}[ ]{.chpuri2}

それでは[、]{.chpule2}[
]{.chpuri2}それがどううまくいくのか見てみましょう[。]{.chpule2}[
]{.chpuri2}あるオープン・ソースの UI
フレームワークを使ってアプリを書くことを想像してみてください[。]{.chpule2}[
]{.chpuri2}アプリには丸いボタンが必要ですが[、]{.chpule2}[
]{.chpuri2}フレームワークは四角いボタンしか提供していません[。]{.chpule2}[
]{.chpuri2}そこで[、]{.chpule2}[ ]{.chpuri2}標準の `Button`
クラスを拡張して[、]{.chpule2}[ ]{.chpuri2}輝かしい `Round­Button`
サブクラスを作成します[。]{.chpule2}[ ]{.chpuri2}しかし[、]{.chpule2}[
]{.chpuri2}ここで[、]{.chpule2}[
]{.chpuri2}デフォルトのボタンの代わりに新しいボタンのサブクラスを使用するように[、]{.chpule2}[
]{.chpuri2}メインの `UIFramework`
クラスに伝える必要が出てきます[。]{.chpule2}[ ]{.chpuri2}

そのためには[、]{.chpule2}[ ]{.chpuri2}フレームワークの基底クラスから
`UIWith­Round­Buttons`[ ]{.chpule1}[（]{.chpuri1}丸いボタンで
UI[）]{.chpule2}[ ]{.chpuri2}というサブクラスを作り[、]{.chpule2}[
]{.chpuri2}その `create­Button` メソッドを上書きします[。]{.chpule2}[
]{.chpuri2}このメソッドは基底クラスでは `Button`
オブジェクトを返しますが[、]{.chpule2}[ ]{.chpuri2}サブクラスでは
`Round­Button` オブジェクトを返すようにします[。]{.chpule2}[
]{.chpuri2}ここで[、]{.chpule2}[ ]{.chpuri2}`UIFramework` の代わりに
`UIWith­Round­Buttons` クラスを使用してください[。]{.chpule2}[
]{.chpuri2}それだけです[！]{.chpule2}[ ]{.chpuri2}
:::

::: applicability-problem
毎回再構築する代わりに[、]{.chpule2}[
]{.chpuri2}既存オブジェクトを再利用してシステム資源を節約したい場合に[、]{.chpule2}[
]{.chpuri2}Factory Method を使用します[。]{.chpule2}[ ]{.chpuri2}
:::

::: applicability-solution
データベース接続[、]{.chpule2}[
]{.chpuri2}ファイルシステム[、]{.chpule2}[
]{.chpuri2}ネットワーク資源等[、]{.chpule2}[
]{.chpuri2}資源を大量に消費するオブジェクトを扱う場合に[、]{.chpule2}[
]{.chpuri2}このような状況によく直面します[。]{.chpule2}[ ]{.chpuri2}

オブジェクトの再利用のために何をしなければいけないか考えてみましよう[。]{.chpule2}[
]{.chpuri2}

1.  まず[、]{.chpule2}[
    ]{.chpuri2}作成されたすべてのオブジェクトを追跡するための記録場所を作成する必要があります[。]{.chpule2}[
    ]{.chpuri2}
2.  誰かがオブジェクトを要求してきたら[、]{.chpule2}[
    ]{.chpuri2}プログラムはそのプール[
    ]{.chpule1}[（]{.chpuri1}蓄積地[）]{.chpule2}[
    ]{.chpuri2}内の空きオブジェクトを探す必要があります[。]{.chpule2}[
    ]{.chpuri2}
3.  そして[、]{.chpule2}[
    ]{.chpuri2}クライアントのコードに対して見つかったオブジェクトを返します[。]{.chpule2}[
    ]{.chpuri2}
4.  再使用可能なオブジェクトがない場合[、]{.chpule2}[
    ]{.chpuri2}プログラムは新しいオブジェクトを作成し[、]{.chpule2}[
    ]{.chpuri2}そしてプールに追加する必要があります[。]{.chpule2}[
    ]{.chpuri2}

これは結構な量のコードになります[！]{.chpule2}[
]{.chpuri2}重複したコードでプログラムを汚染しないように[、]{.chpule2}[
]{.chpuri2}このコードは一箇所に納めるべきです[。]{.chpule2}[ ]{.chpuri2}

おそらく[、]{.chpule2}[
]{.chpuri2}このコードを置くべき最も明白で便利な場所は[、]{.chpule2}[
]{.chpuri2}オブジェクトの再利用をしたいクラスのコンストラクター内です[。]{.chpule2}[
]{.chpuri2}しかし[、]{.chpule2}[
]{.chpuri2}コンストラクターは定義により常に
**新規オブジェクト**を返さなければなりません[。]{.chpule2}[
]{.chpuri2}既存インスタンスを返すことはできません[。]{.chpule2}[
]{.chpuri2}

というわけで[、]{.chpule2}[
]{.chpuri2}新規オブジェクト作成も既存オブジェクトの再利用もできる通常メソッドが必要となります[。]{.chpule2}[
]{.chpuri2}それ[、]{.chpule2}[
]{.chpuri2}ファクトリー・メソッドのように聞こえますね[。]{.chpule2}[
]{.chpuri2}
:::
:::::::::
::::::::::

::: {.section .checklist}
##  実装方法 {#checklist}

1.  すべてのプロダクトが同じインターフェースに従うようにします[。]{.chpule2}[
    ]{.chpuri2}このインターフェースでは[、]{.chpule2}[
    ]{.chpuri2}すべてのプロダクトにとって意味のあるメソッドを宣言する必要があります[。]{.chpule2}[
    ]{.chpuri2}

2.  クリエーター・クラス内に空のファクトリー・メソッドを追加します[。]{.chpule2}[
    ]{.chpuri2}メソッドの戻り値の型は[、]{.chpule2}[
    ]{.chpuri2}共通のプロダクト・インターフェースと一致する必要があります[。]{.chpule2}[
    ]{.chpuri2}

3.  クリエーターのコード内で[、]{.chpule2}[
    ]{.chpuri2}プロダクトのコンストラクターの参照を全部探し出します[。]{.chpule2}[
    ]{.chpuri2}プロダクト作成コードをファクトリー・メソッドに抽出しながら[、]{.chpule2}[
    ]{.chpuri2}一つずつ[、]{.chpule2}[
    ]{.chpuri2}コンストラクター呼び出しをファクトリー・メソッドへの呼び出しで置き換えていきます[。]{.chpule2}[
    ]{.chpuri2}

    ファクトリー・メソッドに[、]{.chpule2}[
    ]{.chpuri2}プロダクトの戻り値の型を決めるためのパラメーターを一時的に追加する必要があるかもしれません[。]{.chpule2}[
    ]{.chpuri2}

    この時点では[、]{.chpule2}[
    ]{.chpuri2}ファクトリー・メソッドのコードはあまりきれいなものではないかもしれません[。]{.chpule2}[
    ]{.chpuri2}どのプロダクトのクラスをインスタンス化するかを選択するかを決める巨大な
    `switch` 文の塊になっているかもしれません[。]{.chpule2}[
    ]{.chpuri2}しかし[、]{.chpule2}[
    ]{.chpuri2}これはすぐ修正するので[、]{.chpule2}[
    ]{.chpuri2}心配は無用です[。]{.chpule2}[ ]{.chpuri2}

4.  次に[、]{.chpule2}[
    ]{.chpuri2}ファクトリー・メソッドに並んでいるプロダクトの型ごとに[、]{.chpule2}[
    ]{.chpuri2}クリエーターのサブクラスを作成します[。]{.chpule2}[
    ]{.chpuri2}基底クラスのファクトリー・メソッドの構築コードの該当部分を抽出して[、]{.chpule2}[
    ]{.chpuri2}それでサブクラスのファクトリー・メソッドを上書きします[。]{.chpule2}[
    ]{.chpuri2}

5.  プロダクトの型が多すぎて[、]{.chpule2}[
    ]{.chpuri2}すべての型に応じたサブクラスを作成することがあまり現実的ではない場合は[、]{.chpule2}[
    ]{.chpuri2}基底クラスに追加した制御パラメーターを再利用できます[。]{.chpule2}[
    ]{.chpuri2}

    たとえば[、]{.chpule2}[
    ]{.chpuri2}次のようなクラスの階層があるとします[：]{.chpule2}[
    ]{.chpuri2}基底クラスである `Mail`[
    ]{.chpule1}[（]{.chpuri1}郵便[）]{.chpule2}[
    ]{.chpuri2}クラスとそのサブクラス[、]{.chpule2}[
    ]{.chpuri2}`Air­Mail`[ ]{.chpule1}[（]{.chpuri1}航空便[）]{.chpule2}[
    ]{.chpuri2}と `Ground­Mail`[
    ]{.chpule1}[（]{.chpuri1}陸送便[）]{.ls-05}[、]{.chpule2}[
    ]{.chpuri2}`Transport`[ ]{.chpule1}[（]{.chpuri1}運輸[）]{.chpule2}[
    ]{.chpuri2}クラスとして `Plane`[
    ]{.chpule1}[（]{.chpuri1}飛行機[）]{.ls-05}[、]{.chpule2}[
    ]{.chpuri2}`Truck`[
    ]{.chpule1}[（]{.chpuri1}トラック[）]{.ls-05}[、]{.chpule2}[
    ]{.chpuri2}`Train`[
    ]{.chpule1}[（]{.chpuri1}鉄道[）]{.ls-05}[。]{.chpule2}[
    ]{.chpuri2}`Air­Mail` クラスは `Plane`
    オブジェクトのみを使用しますが[、]{.chpule2}[
    ]{.chpuri2}`Ground­Mail` は `Truck` と `Train`
    両方のオブジェクトを扱うことができます[。]{.chpule2}[
    ]{.chpuri2}両方のケースを扱うために新しいサブクラス[、]{.chpule2}[
    ]{.chpuri2}たとえば `Train­Mail`[
    ]{.chpule1}[（]{.chpuri1}鉄道便[）]{.chpule2}[
    ]{.chpuri2}を作ることもできます[。]{.chpule2}[
    ]{.chpuri2}が[、]{.chpule2}[
    ]{.chpuri2}もう一つ選択肢としては[、]{.chpule2}[
    ]{.chpuri2}クライアント・コードが `Ground­Mail`
    クラスのファクトリー・メソッドを引数として渡して[、]{.chpule2}[
    ]{.chpuri2}どちらのプロダクトを受け取りたいかを伝えるようにすることです[。]{.chpule2}[
    ]{.chpuri2}

6.  すべての抽出作業の後[、]{.chpule2}[
    ]{.chpuri2}基底クラスのファクトリー・メソッドが空になった場合は[、]{.chpule2}[
    ]{.chpuri2}そのクラスを抽象クラスとすることができます[。]{.chpule2}[
    ]{.chpuri2}何か残った場合は[、]{.chpule2}[
    ]{.chpuri2}それをメソッドのデフォルト動作とすることができます[。]{.chpule2}[
    ]{.chpuri2}
:::

:::::: {.section .pros-cons}
##  長所と短所 {#pros-cons}

::::: row
::: col-sm-6
-    クリエーターと具象プロダクトとの密な結合を回避[。]{.chpule2}[
    ]{.chpuri2}
-   
    *[単]{.cjk}[一]{.cjk}[責]{.cjk}[任]{.cjk}[の]{.cjk}[原]{.cjk}[則]{.cjk}*[。]{.chpule2}[
    ]{.chpuri2}プロダクト作成コードがプログラム中の一箇所にまとめられ[、]{.chpule2}[
    ]{.chpuri2}保守が容易[。]{.chpule2}[ ]{.chpuri2}
-   
    *[開]{.cjk}[放]{.cjk}[閉]{.cjk}[鎖]{.cjk}[の]{.cjk}[原]{.cjk}[則]{.cjk}*[。]{.chpule2}[
    ]{.chpuri2}プロダクトの新しい型をプログラムに導入しても[、]{.chpule2}[
    ]{.chpuri2}既存のクライアント・コードの機能に影響しない[。]{.chpule2}[
    ]{.chpuri2}
:::

::: col-sm-6
-    本パターンの適用では[、]{.chpule2}[
    ]{.chpuri2}多数の新規サブクラス導入の必要があり[、]{.chpule2}[
    ]{.chpuri2}コードの複雑化の恐れあり[。]{.chpule2}[
    ]{.chpuri2}既存クリエーター・クラスの階層にこのパターンを適用する場合[、]{.chpule2}[
    ]{.chpuri2}最善の結果が得られる[。]{.chpule2}[ ]{.chpuri2}
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

-   [Factory Method](factory-method.html) を [Iterator](iterator.html)
    と一緒に使って[、]{.chpule2}[
    ]{.chpuri2}コレクションのサブクラスが[、]{.chpule2}[
    ]{.chpuri2}コレクションと互換な[、]{.chpule2}[
    ]{.chpuri2}異なる型のイテレーターを返すようにできます[。]{.chpule2}[
    ]{.chpuri2}

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

-   [Factory Method](factory-method.html) は[、]{.chpule2}[
    ]{.chpuri2}[Template Method](template-method.html)
    の特別な場合です[。]{.chpule2}[ ]{.chpuri2}同時に[、]{.chpule2}[
    ]{.chpuri2}*Factory Method* は[、]{.chpule2}[ ]{.chpuri2}大きな
    *Template Method*
    の一つのステップとして使うこともできます[。]{.chpule2}[ ]{.chpuri2}
:::

::: {.section .implementations}
##  コード例 {#implementations}

[![Factory Method を C#
で](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](factory-method/csharp/example.html "Factory Method を C# で"){.prog-lang-link}
[![Factory Method を C++
で](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](factory-method/cpp/example.html "Factory Method を C++ で"){.prog-lang-link}
[![Factory Method を Go
で](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](factory-method/go/example.html "Factory Method を Go で"){.prog-lang-link}
[![Factory Method を Java
で](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](factory-method/java/example.html "Factory Method を Java で"){.prog-lang-link}
[![Factory Method を PHP
で](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](factory-method/php/example.html "Factory Method を PHP で"){.prog-lang-link}
[![Factory Method を Python
で](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](factory-method/python/example.html "Factory Method を Python で"){.prog-lang-link}
[![Factory Method を Ruby
で](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](factory-method/ruby/example.html "Factory Method を Ruby で"){.prog-lang-link}
[![Factory Method を Rust
で](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](factory-method/rust/example.html "Factory Method を Rust で"){.prog-lang-link}
[![Factory Method を Swift
で](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](factory-method/swift/example.html "Factory Method を Swift で"){.prog-lang-link}
[![Factory Method を TypeScript
で](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](factory-method/typescript/example.html "Factory Method を TypeScript で"){.prog-lang-link}
:::

::: {.section .extras}
##  追記 {#extras}

-   多種のファクトリー・パターンとその概念の違いがはっきりしない場合は[、]{.chpule2}[
    ]{.chpuri2}[ファクトリー比較](factory-comparison.html)を参照してください[。]{.chpule2}[
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

[Abstract Factory []{.fa .fa-arrow-right}](abstract-factory.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### 戻る

[[]{.fa
.fa-arrow-left} 生成に関するパターン](creational-patterns.html){.btn
.btn-default rel="prev"}
:::
::::::::::::::::::::::::::::::::::

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
::::::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::::::
