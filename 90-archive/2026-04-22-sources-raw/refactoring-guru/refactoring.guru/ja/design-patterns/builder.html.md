:::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} /
[デザインパターン](../design-patterns.html){.type} /
[生成に関するパターン](creational-patterns.html){.type}
:::

# Builder {#builder .title}

::: aka
別名：[ビルダー]{style="display:inline-block"}
:::

::: {.section .intent}
##  一言でいうと {#intent}

**Builder**[ ]{.chpule1}[（]{.chpuri1}ビルダー[、]{.chpule2}[
]{.chpuri2}建設業者[）]{.chpule2}[ ]{.chpuri2}は[、]{.chpule2}[
]{.chpuri2}複雑なオブジェクトを段階的に構築できる生成に関するデザインパターンです[。]{.chpule2}[
]{.chpuri2}このパターンを使用すると[、]{.chpule2}[
]{.chpuri2}同じ構築コードを使用して異なる型と表現のオブジェクトを生成することが可能です[。]{.chpule2}[
]{.chpuri2}

<figure class="image">
<img
src="../../images/patterns/content/builder/builder-ja7aa8.png?id=8a0ec32cd855b2abd8183768466a112a"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/builder/builder-ja.png?id=8a0ec32cd855b2abd8183768466a112a 640w,/images/patterns/content/builder/builder-ja-1.5x.png?id=0f8905cb36af10404c7caab3776e9dfd 960w,/images/patterns/content/builder/builder-ja-2x.png?id=132d873c31209a77c1f7389110009138 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Builder デザインパターン" />
</figure>
:::

::: {.section .problem}
##  問題 {#problem}

多くのフィールドと入れ子になったオブジェクトからなり[、]{.chpule2}[
]{.chpuri2}面倒な段階的初期化が必要な[、]{.chpule2}[
]{.chpuri2}複雑なオブジェクトを想像してみてください[。]{.chpule2}[
]{.chpuri2}このような初期化コードは通常[、]{.chpule2}[
]{.chpuri2}多くのパラメータを持つ巨大なコンストラクターの中に埋め込まれています[。]{.chpule2}[
]{.chpuri2}最悪の場合[、]{.chpule2}[
]{.chpuri2}初期化コードはクライアント・コードの中のあちこちに分散しています[。]{.chpule2}[
]{.chpuri2}

<figure class="image">
<img
src="../../images/patterns/diagrams/builder/problem16274.png?id=11e715c5c97811f848c48e0f399bb05e"
style="aspect-ratio: 1.71;"
srcset="/images/patterns/diagrams/builder/problem1.png?id=11e715c5c97811f848c48e0f399bb05e 600w,/images/patterns/diagrams/builder/problem1-1.5x.png?id=a46778018c4f0f4bbf2357940c1f5f40 900w,/images/patterns/diagrams/builder/problem1-2x.png?id=02ffbd7ad294600e42aa78989d441b4d 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="多くのサブクラスの存在が別の問題を引き起こします" />
<figcaption><p>オブジェクトの可能なすべての構成に対してサブクラスを作成すると<span
class="chpule2">、</span><span class="chpuri2">
</span>プログラムが過剰に複雑になる可能性がある<span
class="chpule2">。</span><span class="chpuri2"> </span></p></figcaption>
</figure>

たとえば[、]{.chpule2}[ ]{.chpuri2}`House`[
]{.chpule1}[（]{.chpuri1}家[）]{.chpule2}[
]{.chpuri2}オブジェクトをどう作成するか考えてみましょう[。]{.chpule2}[
]{.chpuri2}簡単な家を建てるには[、]{.chpule2}[ ]{.chpuri2}壁を 4
面と床を一つ作り[、]{.chpule2}[
]{.chpuri2}ドアを一つ設置し[、]{.chpule2}[
]{.chpuri2}窓を二つ取り付け[、]{.chpule2}[
]{.chpuri2}屋根を作ればすみそうです[。]{.chpule2}[
]{.chpuri2}しかし[、]{.chpule2}[
]{.chpuri2}もっと大きくて明るい[、]{.chpule2}[
]{.chpuri2}裏庭やその他の設備[
]{.chpule1}[（]{.chpuri1}暖房システム[、]{.chpule2}[
]{.chpuri2}配管[、]{.chpule2}[
]{.chpuri2}電気配線のような[）]{.chpule2}[
]{.chpuri2}のある家を望んでいるとしたら[、]{.chpule2}[
]{.chpuri2}どうしますか[？]{.chpule2}[ ]{.chpuri2}

最も単純な解決策は[、]{.chpule2}[ ]{.chpuri2}基底の `House`
クラスを拡張し[、]{.chpule2}[
]{.chpuri2}あらゆる組み合わせをカバーする多数のサブクラスを作成することです[。]{.chpule2}[
]{.chpuri2}こうすると最終的にはとんでもない数のサブクラスが必要になります[。]{.chpule2}[
]{.chpuri2}ベランダのスタイルのような新しいパラメーターを追加すると[、]{.chpule2}[
]{.chpuri2}この階層はさらに複雑になります[。]{.chpule2}[ ]{.chpuri2}

もう一つのやり方では[、]{.chpule2}[
]{.chpuri2}サブクラスの繁殖は不要です[。]{.chpule2}[ ]{.chpuri2}基底の
`House` クラス中に[、]{.chpule2}[
]{.chpuri2}家オブジェクトに関わるありとあらゆるパラメーターを持った巨大コンストラクターを作ります[。]{.chpule2}[
]{.chpuri2}このやり方では[、]{.chpule2}[
]{.chpuri2}サブクラスの必要性を排除しますが[、]{.chpule2}[
]{.chpuri2}別の問題が発生します[。]{.chpule2}[ ]{.chpuri2}

<figure class="image">
<img
src="../../images/patterns/diagrams/builder/problem27ab5.png?id=2e91039b6c7d2d2df6ee519983a3b036"
style="aspect-ratio: 1.71;"
srcset="/images/patterns/diagrams/builder/problem2.png?id=2e91039b6c7d2d2df6ee519983a3b036 600w,/images/patterns/diagrams/builder/problem2-1.5x.png?id=3d302cf2762fd04d70cbb91cb54e923c 900w,/images/patterns/diagrams/builder/problem2-2x.png?id=5e7975a91c0e4f4ba960f908cc9c2ea2 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="望遠鏡的コンストラクター" />
<figcaption><p>多数のパラメーターを持つコンストラクターには<span
class="chpule2">、</span><span class="chpuri2">
</span>常にすべてのパラメーターは必要ではない<span
class="chpule2">、</span><span class="chpuri2">
</span>という欠点がある<span class="chpule2">。</span><span
class="chpuri2"> </span></p></figcaption>
</figure>

ほとんどの場合[、]{.chpule2}[
]{.chpuri2}大半のパラメータは使われることなく[、]{.chpule2}[
]{.chpuri2}[かなり醜いコンストラクター呼び出し](../smells/long-parameter-list.html)になります[。]{.chpule2}[
]{.chpuri2}たとえば[、]{.chpule2}[
]{.chpuri2}プールがある家はわずかしかないため[、]{.chpule2}[
]{.chpuri2}プールに関連するパラメーターは[、]{.chpule2}[ ]{.chpuri2}9
割の場合[、]{.chpule2}[ ]{.chpuri2}役立たずとなります[。]{.chpule2}[
]{.chpuri2}
:::

::: {.section .solution}
##  解決策 {#solution}

Builder パターンでは[、]{.chpule2}[
]{.chpuri2}オブジェクトの構築コードを自クラスから抽出して[、]{.chpule2}[
]{.chpuri2}*[ビ]{.cjk}[ル]{.cjk}[ダ]{.cjk}[ー]{.cjk}*と呼ばれる別オブジェクトに移動します[。]{.chpule2}[
]{.chpuri2}

<figure class="image">
<img
src="../../images/patterns/diagrams/builder/solution13e5f.png?id=8ce82137f8935998de802cae59e00e11"
style="aspect-ratio: 1.46;"
srcset="/images/patterns/diagrams/builder/solution1.png?id=8ce82137f8935998de802cae59e00e11 410w,/images/patterns/diagrams/builder/solution1-1.5x.png?id=8ab77eb73760a75c35940bae243d43f2 615w,/images/patterns/diagrams/builder/solution1-2x.png?id=a9c2ab02f0b2aca1a7512022194dd113 820w"
sizes="(max-width: 720px) 100vw, 410px" loading="lazy" width="410"
alt="Builder パターンの適用" />
<figcaption><p>Builder パターンでは<span class="chpule2">、</span><span
class="chpuri2"> </span>複雑なオブジェクトを一ステップずつ構築する<span
class="chpule2">。</span><span class="chpuri2"> </span>Builder では<span
class="chpule2">、</span><span class="chpuri2"> </span>プロダクト<span
class="chpule1"> </span><span class="chpuri1">（</span>構築対象物<span
class="chpule2">）</span><span class="chpuri2">
</span>の構築中に他のオブジェクトがプロダクトをアクセスすることは禁止されている<span
class="chpule2">。</span><span class="chpuri2"> </span></p></figcaption>
</figure>

このパターンでは[、]{.chpule2}[ ]{.chpuri2}オブジェクトの構築を
`build­Walls`[ ]{.chpule1}[（]{.chpuri1}壁作成[）]{.ls-05}[、]{.chpule2}[
]{.chpuri2}`build­Door`[ ]{.chpule1}[（]{.chpuri1}ドア作成[）]{.chpule2}[
]{.chpuri2}などの[、]{.chpule2}[
]{.chpuri2}一連のステップに整理します[。]{.chpule2}[
]{.chpuri2}オブジェクトを作成するには[、]{.chpule2}[
]{.chpuri2}ビルダー・オブジェクトに対してこれらのステップを次々に実行します[。]{.chpule2}[
]{.chpuri2}重要なのは[、]{.chpule2}[
]{.chpuri2}すべてのステップを呼び出す必要はない[、]{.chpule2}[
]{.chpuri2}ということです[。]{.chpule2}[
]{.chpuri2}特定の構成のオブジェクトを生成するためには[、]{.chpule2}[
]{.chpuri2}それに必要なステップのみを呼び出せばすみます[。]{.chpule2}[
]{.chpuri2}

様々な種類のプロダクトを構築する必要がある場合[、]{.chpule2}[
]{.chpuri2}構築ステップのいくつかは[、]{.chpule2}[
]{.chpuri2}異なる実装を必要とするかもしれません[。]{.chpule2}[
]{.chpuri2}たとえば[、]{.chpule2}[
]{.chpuri2}小屋の壁は木でできていますが[、]{.chpule2}[
]{.chpuri2}城壁は石である必要があります[。]{.chpule2}[ ]{.chpuri2}

この場合[、]{.chpule2}[
]{.chpuri2}同じ一連の構築ステップを異なる方法で実装した複数のビルダー・クラスを作成することができます[。]{.chpule2}[
]{.chpuri2}次に[、]{.chpule2}[
]{.chpuri2}異なる種類のオブジェクトを作り出すためには[、]{.chpule2}[
]{.chpuri2}これらのビルダーを構築プロセス[
]{.chpule1}[（]{.chpuri1}一連の決まった順番でのステップの呼び出し[）]{.chpule2}[
]{.chpuri2}で使用します[。]{.chpule2}[ ]{.chpuri2}

<figure class="image">
<img
src="../../images/patterns/content/builder/builder-comic-1-ja5704.png?id=d0559cb71e91063f422f4f62e95bbead"
style="aspect-ratio: 2;"
srcset="/images/patterns/content/builder/builder-comic-1-ja.png?id=d0559cb71e91063f422f4f62e95bbead 600w,/images/patterns/content/builder/builder-comic-1-ja-1.5x.png?id=7d9306a3c8635e1ba29926a5b0c8e96a 900w,/images/patterns/content/builder/builder-comic-1-ja-2x.png?id=e5b649542babe50f5a08c26d6492d654 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600" />
<figcaption><p>ビルダーは<span class="chpule2">、</span><span
class="chpuri2"> </span>同じタスクを異なる方法で実行する<span
class="chpule2">。</span><span class="chpuri2"> </span></p></figcaption>
</figure>

たとえば[、]{.chpule2}[
]{.chpuri2}木やガラスからすべてを作るビルダーを想像してみてください[。]{.chpule2}[
]{.chpuri2}二つ目のビルダーは[、]{.chpule2}[
]{.chpuri2}石と鉄を使い[、]{.chpule2}[
]{.chpuri2}三つ目は[、]{.chpule2}[
]{.chpuri2}金とダイヤモンドを使います[。]{.chpule2}[
]{.chpuri2}同じステップを呼び出すと[、]{.chpule2}[
]{.chpuri2}最初のビルダーから通常の家が手に入ります[。]{.chpule2}[
]{.chpuri2}2 番目のビルダーからは[、]{.chpule2}[
]{.chpuri2}小さな城[、]{.chpule2}[ ]{.chpuri2}3
番目のビルダーからは[、]{.chpule2}[
]{.chpuri2}宮殿が手に入ります[。]{.chpule2}[
]{.chpuri2}しかし[、]{.chpule2}[
]{.chpuri2}このことを可能にするためには[、]{.chpule2}[
]{.chpuri2}構築ステップを呼び出すクライアント・コードが共通のインターフェースを使用してビルダーとやりとりする必要があります[。]{.chpule2}[
]{.chpuri2}

#### ディレクター

この考えをさらに進めて[、]{.chpule2}[
]{.chpuri2}プロダクト構築に使用する一連の構築ステップへの呼び出しを*[デ]{.cjk}[ィ]{.cjk}[レ]{.cjk}[ク]{.cjk}[タ]{.cjk}[ー]{.cjk}*と呼ばれる別クラスに抽出することができます[。]{.chpule2}[
]{.chpuri2}ディレクター・クラスは構築ステップを実行する順番を定義し[、]{.chpule2}[
]{.chpuri2}ビルダーはそのステップの実装を提供します[。]{.chpule2}[
]{.chpuri2}

<figure class="image">
<img
src="../../images/patterns/content/builder/builder-comic-2-ja947d.png?id=12a03811e7c6ae09e66f8a5e2d702992"
style="aspect-ratio: 1.14;"
srcset="/images/patterns/content/builder/builder-comic-2-ja.png?id=12a03811e7c6ae09e66f8a5e2d702992 343w,/images/patterns/content/builder/builder-comic-2-ja-1.5x.png?id=dc2a837fb5a8d2c159c5aab9afc1e078 515w,/images/patterns/content/builder/builder-comic-2-ja-2x.png?id=5769e5238bb8ed32d4189cc7968158e8 687w"
sizes="(max-width: 720px) 100vw, 343px" loading="lazy" width="343" />
<figcaption><p>ディレクターは<span class="chpule2">、</span><span
class="chpuri2"> </span>機能するプロダクトを取得するために<span
class="chpule2">、</span><span class="chpuri2">
</span>どの構築ステップを実行するべきかを知っている<span
class="chpule2">。</span><span class="chpuri2"> </span></p></figcaption>
</figure>

ディレクター・クラスは厳密には必要ありません[。]{.chpule2}[
]{.chpuri2}クライアント・コードから直接[、]{.chpule2}[
]{.chpuri2}特定の順序で構築ステップを呼び出せばすみます[。]{.chpule2}[
]{.chpuri2}しかし[、]{.chpule2}[
]{.chpuri2}ディレクター・クラスは[、]{.chpule2}[
]{.chpuri2}再利用のために[、]{.chpule2}[
]{.chpuri2}様々なよく使う構築ステップの一連の呼び出しを置いておくには[、]{.chpule2}[
]{.chpuri2}良い場所かもしれません[。]{.chpule2}[ ]{.chpuri2}

また[、]{.chpule2}[
]{.chpuri2}ディレクター・クラスを使用すると[、]{.chpule2}[
]{.chpuri2}プロダクト構築の詳細をクライアント・コードから完全に隠蔽できます[。]{.chpule2}[
]{.chpuri2}クライアントはビルダーをディレクターに関連付け[、]{.chpule2}[
]{.chpuri2}ディレクターを通して構築を開始し[、]{.chpule2}[
]{.chpuri2}ビルダーから結果を取得するだけです[。]{.chpule2}[ ]{.chpuri2}
:::

::::: {.section .structure-container}
##  構造 {#structure}

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/builder/structure05e4.png?id=fe9e23559923ea0657aa5fe75efef333"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 0.79;"
srcset="/images/patterns/diagrams/builder/structure.png?id=fe9e23559923ea0657aa5fe75efef333 460w,/images/patterns/diagrams/builder/structure-1.5x.png?id=91ea8cd3137b403542c23abf63938f00 690w,/images/patterns/diagrams/builder/structure-2x.png?id=dca1b1508e23c266cbedc80ffb84311a 920w"
sizes="(max-width: 720px) 100vw, 460px" loading="lazy" width="460"
alt="Builder デザインパターンの構造" /><img
src="../../images/patterns/diagrams/builder/structure-indexedadac.png?id=44b3d763ce91dbada5d8394ef777437f"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 0.81;"
srcset="/images/patterns/diagrams/builder/structure-indexed.png?id=44b3d763ce91dbada5d8394ef777437f 470w,/images/patterns/diagrams/builder/structure-indexed-1.5x.png?id=d57a6ff9342ea31736ea98e5066e0748 705w,/images/patterns/diagrams/builder/structure-indexed-2x.png?id=153eed9a51784cbe00d0ca8b3d6b268d 940w"
sizes="(max-width: 720px) 100vw, 470px" loading="lazy" width="470"
alt="Builder デザインパターンの構造" />
</figure>
:::

1.  **ビルダー**[ ]{.chpule1}[（]{.chpuri1}Builder[）]{.chpule2}[
    ]{.chpuri2}インターフェースは[、]{.chpule2}[
    ]{.chpuri2}すべての型のビルダーに共通するプロダクト構築の様々なステップを宣言します[。]{.chpule2}[
    ]{.chpuri2}

2.  **具象ビルダー**[ ]{.chpule1}[（]{.chpuri1}Concrete
    Builder[）]{.chpule2}[ ]{.chpuri2}は[、]{.chpule2}[
    ]{.chpuri2}様々な構築ステップの実装を行います[。]{.chpule2}[
    ]{.chpuri2}具象ビルダーは[、]{.chpule2}[
    ]{.chpuri2}共通インターフェース外のプロダクトの産出も可能です[。]{.chpule2}[
    ]{.chpuri2}

3.  **プロダクト**[ ]{.chpule1}[（]{.chpuri1}Product[）]{.chpule2}[
    ]{.chpuri2}は[、]{.chpule2}[
    ]{.chpuri2}結果のオブジェクトです[。]{.chpule2}[
    ]{.chpuri2}異なるビルダーによって構築された製品は[、]{.chpule2}[
    ]{.chpuri2}同じクラス階層やインターフェースに属する必要はありません[。]{.chpule2}[
    ]{.chpuri2}

4.  **ディレクター**[ ]{.chpule1}[（]{.chpuri1}Director[）]{.chpule2}[
    ]{.chpuri2}クラスは[、]{.chpule2}[
    ]{.chpuri2}構築ステップを呼び出す順序を定義し[、]{.chpule2}[
    ]{.chpuri2}プロダクトの特定の設定を作成して再利用できます[。]{.chpule2}[
    ]{.chpuri2}

5.  **クライアント**[ ]{.chpule1}[（]{.chpuri1}Client[）]{.chpule2}[
    ]{.chpuri2}は[、]{.chpule2}[
    ]{.chpuri2}ビルダー・オブジェクトの一つをディレクターに関連付ける必要があります[。]{.chpule2}[
    ]{.chpuri2}通常[、]{.chpule2}[
    ]{.chpuri2}ディレクターのコンストラクターのパラメータを介して 1
    回だけ実行されます[。]{.chpule2}[ ]{.chpuri2}その後[、]{.chpule2}[
    ]{.chpuri2}ディレクターはそのビルダー・オブジェクトをその後のすべての構築に使用します[。]{.chpule2}[
    ]{.chpuri2}しかし[、]{.chpule2}[
    ]{.chpuri2}クライアントがビルダー・オブジェクトをディレクターの産出メソッドに渡すタイミングに関しては[、]{.chpule2}[
    ]{.chpuri2}別のやり方もあります[。]{.chpule2}[
    ]{.chpuri2}この場合[、]{.chpule2}[
    ]{.chpuri2}ディレクターが何かの産出をするごとに[、]{.chpule2}[
    ]{.chpuri2}別のビルダーを使用できます[。]{.chpule2}[ ]{.chpuri2}
::::
:::::

::: {.section .pseudocode}
##  擬似コード {#pseudocode}

この **Builder** パターンの適用例では[、]{.chpule2}[
]{.chpuri2}異なる型のプロダクトの構築に際して同じオブジェクト構築コードを再利用する方法を説明します[。]{.chpule2}[
]{.chpuri2}車の構築とその車のマニュアルの作成を行います[。]{.chpule2}[
]{.chpuri2}

<figure class="image">
<img
src="../../images/patterns/diagrams/builder/example-jaee54.png?id=0e711905efe8cb66c391f50c961c9e12"
style="aspect-ratio: 0.85;"
srcset="/images/patterns/diagrams/builder/example-ja.png?id=0e711905efe8cb66c391f50c961c9e12 500w,/images/patterns/diagrams/builder/example-ja-1.5x.png?id=269cc211c7eabb0e0af4dd6502e6c62f 750w,/images/patterns/diagrams/builder/example-ja-2x.png?id=b851784e4c029df39390462bb006591b 1000w"
sizes="(max-width: 720px) 100vw, 500px" loading="lazy" width="500"
alt="Builder パターン例の構造" />
<figcaption><p>車のステップごとの構築と<span
class="chpule2">、</span><span class="chpuri2">
</span>その車のモデルのユーザーガイドの構築の例<span
class="chpule2">。</span><span class="chpuri2"> </span></p></figcaption>
</figure>

車は 100 以上の方法で作れる複雑な物体です[。]{.chpule2}[
]{.chpuri2}巨大なコンストラクターで `Car`
クラスを複雑怪奇なものにする代わりに[、]{.chpule2}[
]{.chpuri2}車生産のコードを[、]{.chpule2}[
]{.chpuri2}独立した車のビルダー・クラスに抽出しました[。]{.chpule2}[
]{.chpuri2}このクラスには[、]{.chpule2}[
]{.chpuri2}車の様々なパーツを構成するための一連のメソッドがあります[。]{.chpule2}[
]{.chpuri2}

クライアントのコードが特注モデルの車を組み立てる必要がある場合は[、]{.chpule2}[
]{.chpuri2}ビルダーを直接使うことができます[。]{.chpule2}[
]{.chpuri2}一方[、]{.chpule2}[ ]{.chpuri2}クライアントは[、]{.chpule2}[
]{.chpuri2}組み立てをディレクター・クラスに委任することもできます[。]{.chpule2}[
]{.chpuri2}ディレクターは[、]{.chpule2}[
]{.chpuri2}最も人気のあるモデルの車をビルダーを使って作る方法を知っています[。]{.chpule2}[
]{.chpuri2}

ショックを受けるかもしれませんが[、]{.chpule2}[
]{.chpuri2}すべての車にはマニュアルが必要です[[。]{.ls-1}]{.chpule2}[
]{.chpuri2}​[
]{.chpule1}[（]{.chpuri1}本当に読む人いるのか？[）]{.chpule2}[
]{.chpuri2}マニュアルは[、]{.chpule2}[
]{.chpuri2}車のすべての機能を説明するため[、]{.chpule2}[
]{.chpuri2}マニュアルの詳細はモデルごとに異なります[。]{.chpule2}[
]{.chpuri2}だからこそ[、]{.chpule2}[
]{.chpuri2}既存の構築手順を本物の車とそのマニュアルの作成の両方に再利用することは理にかなっています[。]{.chpule2}[
]{.chpuri2}勿論マニュアルを作るのは車を作るのと同じではありません[。]{.chpule2}[
]{.chpuri2}マニュアルの作成に特化したもう一つのビルダー・クラスが必要となります[。]{.chpule2}[
]{.chpuri2}このクラスは車を作るクラスと兄弟関係にあり[、]{.chpule2}[
]{.chpuri2}同じ構築メソッドの組を実装しますが[、]{.chpule2}[
]{.chpuri2}車の部品を作る代わりにそれらを記述します[。]{.chpule2}[
]{.chpuri2}これらのビルダーを同じディレクトリー・オブジェクトに渡すことで[、]{.chpule2}[
]{.chpuri2}車かマニュアルのどちらかを作ることができます[。]{.chpule2}[
]{.chpuri2}

最後の部分は[、]{.chpule2}[
]{.chpuri2}結果のオブジェクトの取り出しです[。]{.chpule2}[
]{.chpuri2}鉄でできた車と紙のマニュアルは[、]{.chpule2}[
]{.chpuri2}関連してはいますが[、]{.chpule2}[
]{.chpuri2}とても異なるものです[。]{.chpule2}[
]{.chpuri2}ディレクターに結果を取り出すメソッドを追加するためには[、]{.chpule2}[
]{.chpuri2}ディレクターをプロダクトの具象クラスに結合する必要が出てきます[。]{.chpule2}[
]{.chpuri2}そこで[
]{.chpule1}[（]{.chpuri1}結合を不要とするため[）]{.ls-05}[、]{.chpule2}[
]{.chpuri2}実際の作業を行ったビルダーから[、]{.chpule2}[
]{.chpuri2}構築結果を受け取ります[。]{.chpule2}[ ]{.chpuri2}

<figure class="code">
<pre class="code" lang="pseudocode"><code>// Builder パターンの使用は、製品が非常に複雑で細かな構成が必要な場合にの
// み意味がある。以下の二つのプロダクトは、共通のインターフェースは持たな
// いが、関連している。
class Car is
    // 車には、GPS とトリップコンピューターといくつかの座席がある。異なる
    // モデルの車（スポーツカー、SUV、オープンカー）には、異なる機能が搭
    // 載されている。

class Manual is
    // 車ごとに、その車の構成に従ったマニュアルがあり、全機能を説明する。


// ビルダー・インターフェースは、プロダクト・オブジェクトの異なる部品を作
// 成するメソッドを指定。
interface Builder is
    method reset()
    method setSeats(……)
    method setEngine(……)
    method setTripComputer(……)
    method setGPS(……)

// 具象ビルダー・クラスはビルダー・インターフェースに従い、構築ステップの
// 特定の実装を行う。プログラムには、いくつかの異種のビルダーがあるかもし
// れない。
class CarBuilder implements Builder is
    private field car:Car

    // できたてのビルダーのインスタンスは、今後の組み立てに使用される、空
    // のプロダクト・オブジェクトがある。
    constructor CarBuilder() is
        this.reset()

    // リセット・メソッドは、構築されるオブジェクトを一掃する。
    method reset() is
        this.car = new Car()

    // 全生産ステップは、同一のプロダクト・インスタンスに対して機能。
    method setSeats(……) is
        // 座席数を指定。

    method setEngine(……) is
        // 指定のエンジンを搭載。

    method setTripComputer(……) is
        // トリップコンピュータを取り付け。

    method setGPS(……) is
        // GPS を取り付け。

    // 具象ビルダーは、結果を取得するための独自のメソッドを提供することに
    // なっている。ビルダーによっては、まったく異なるプロダクトを生成する
    // 可能性があり、同じインターフェースに従えないため。したがって、結果
    // 取得のためのメソッドは、ビルダー・インターフェース（少なくとも静的
    // 型付けプログラミング言語）では宣言できない。
    //
    // 通常、最終結果をクライアントに返した後、ビルダーのインスタンスは別
    // のプロダクトの構築を開始する準備ができているものと期待される。その
    // ため、普通はプロダクト取得メソッドの中でリセット・メソッドを呼び出
    // す。 ただし、これは必須ではない。クライアント・コードからの明示的
    // なリセット呼び出しを待ってから今までの結果を廃棄してもよい。
    method getProduct():Car is
        product = this.car
        this.reset()
        return product

// 他の生成に関するパターンと異なり、Builder では、共通インターフェースに
// 従わないプロダクトを構築可能。
class CarManualBuilder implements Builder is
    private field manual:Manual

    constructor CarManualBuilder() is
        this.reset()

    method reset() is
        this.manual = new Manual()

    method setSeats(……) is
        // 車の座席の特徴に関する記述。

    method setEngine(……) is
        // エンジンに関する説明を追加。

    method setTripComputer(……) is
        // トリップコンピューターに関する説明を追加。

    method setGPS(……) is
        // GPS に関する説明を追加。

    method getProduct():Manual is
        // マニュアルを返し、ビルダーをリセット。


// ディレクターは、構築ステップを特定の順番で実行することのみの責任を持つ。
// プロダクトを特定の順番や構成で作り出す時に便利。クライアントはビル
// ダーを直接管理できるので、ディレクター・クラスは厳密には必須でない。
class Director is
    // ディレクターは、クライアント・コードが渡すどんなビルダー・インスタ
    // ンスとでも機能する。このため、クライアント・コードは、新規に組み立
    // てられたプロダクトの最終的な型を変更可能。ディレクターは、同一の
    // 構築ステップを使っていくつものプロダクトの異種を構築可能。
    method constructSportsCar(builder: Builder) is
        builder.reset()
        builder.setSeats(2)
        builder.setEngine(new SportEngine())
        builder.setTripComputer(true)
        builder.setGPS(true)

    method constructSUV(builder: Builder) is
        // ……


// クライアント・コードはビルダー・オブジェクトを作成し、それをディレク
// ターに渡し、構築プロセスを開始。最終結果はビルダーオブジェクトから取得。
class Application is

    method makeCar() is
        director = new Director()

        CarBuilder builder = new CarBuilder()
        director.constructSportsCar(builder)
        Car car = builder.getProduct()

        CarManualBuilder builder = new CarManualBuilder()
        director.constructSportsCar(builder)

        // ディレクターは具象ビルダーのクラスとプロダクトを認識していない
        // ため、最終製品はビルダー・オブジェクトから取得することが多い。
        Manual manual = builder.getProduct()</code></pre>
</figure>
:::

:::::::::: {.section .applicability-container}
##  適応性 {#applicability}

::::::::: applicability
::: applicability-problem
[ ]{.chpule1}[「]{.chpuri1}望遠鏡的な[」]{.chpule2}[
]{.chpuri2}コンストラクターを避けるために[、]{.chpule2}[
]{.chpuri2}Builder パターンを使用します[。]{.chpule2}[ ]{.chpuri2}
:::

::: applicability-solution
たとえば[、]{.chpule2}[ ]{.chpuri2}10
個の省略可能パラメータを持つコンストラクターがあるとします[。]{.chpule2}[
]{.chpuri2}このような難物を呼び出すことは非常に不便です[。]{.chpule2}[
]{.chpuri2}したがって[、]{.chpule2}[
]{.chpuri2}コンストラクタ－を多重定義し[、]{.chpule2}[
]{.chpuri2}少数パラメータの短縮版を作成します[。]{.chpule2}[
]{.chpuri2}これらのコンストラクターは[、]{.chpule2}[
]{.chpuri2}省略されたパラメーターの代わりにデフォルト値を元々のコンストラクターに渡します[。]{.chpule2}[
]{.chpuri2}

<figure class="code">
<pre class="code" lang="java"><code>class Pizza {
    Pizza(int size) { …… }
    Pizza(int size, boolean cheese) { …… }
    Pizza(int size, boolean cheese, boolean pepperoni) { …… }
    // ……</code></pre>
<figcaption><p>このような化け物を作成することは<span
class="chpule2">、</span><span class="chpuri2"> </span>C# や
Java のようなメソッドの多重定義をサポートする言語でのみ可能です<span
class="chpule2">。</span><span class="chpuri2"> </span></p></figcaption>
</figure>

Builder パターンで[、]{.chpule2}[
]{.chpuri2}必要なステップのみを使用して[、]{.chpule2}[
]{.chpuri2}オブジェクトを段階的に作成できます[。]{.chpule2}[
]{.chpuri2}このパターンを適用すれば[、]{.chpule2}[
]{.chpuri2}コンストラクターに何十個ものパラメーターを詰め込む必要がなくなります[。]{.chpule2}[
]{.chpuri2}
:::

::: applicability-problem
ご自分のコードで異なる表現のプロダクト[
]{.chpule1}[（]{.chpuri1}石や木製の家など[）]{.chpule2}[
]{.chpuri2}を作成したい場合に[、]{.chpule2}[ ]{.chpuri2}Builder
パターンを使ってください[。]{.chpule2}[ ]{.chpuri2}
:::

::: applicability-solution
Builder パターンは[、]{.chpule2}[
]{.chpuri2}製品の様々な表現の構築において[、]{.chpule2}[
]{.chpuri2}詳細のみが異なる同様のステップが含まれる場合に適用できます[。]{.chpule2}[
]{.chpuri2}

基底のビルダー・インターフェースは[、]{.chpule2}[
]{.chpuri2}すべての可能な構築ステップを定義し[、]{.chpule2}[
]{.chpuri2}具象ビルダーは[、]{.chpule2}[
]{.chpuri2}プロダクトの特定の表現を構築するために[、]{.chpule2}[
]{.chpuri2}これらのステップを実装します[。]{.chpule2}[
]{.chpuri2}一方[、]{.chpule2}[
]{.chpuri2}ディレクター・クラスは[、]{.chpule2}[
]{.chpuri2}構築の順序を手引きします[。]{.chpule2}[ ]{.chpuri2}
:::

::: applicability-problem
[Composite](composite.html)
ツリーなどの複雑なオブジェクトの構築に[、]{.chpule2}[ ]{.chpuri2}Builder
を適用してください[。]{.chpule2}[ ]{.chpuri2}
:::

::: applicability-solution
Builder パターンでは[、]{.chpule2}[
]{.chpuri2}プロダクトを段階的に構築します[。]{.chpule2}[
]{.chpuri2}いくつかのステップの実行を遅らせても[、]{.chpule2}[
]{.chpuri2}最終的プロダクトは[、]{.chpule2}[
]{.chpuri2}問題なく機能します[。]{.chpule2}[
]{.chpuri2}ステップを再帰的に呼び出すことも可能で[、]{.chpule2}[
]{.chpuri2}オブジェクト・ツリーの構築時に便利です[。]{.chpule2}[
]{.chpuri2}

ビルダーは[、]{.chpule2}[
]{.chpuri2}構築ステップの実行中の未完成のプロダクトを外部に公開しません[。]{.chpule2}[
]{.chpuri2}これにより[、]{.chpule2}[
]{.chpuri2}クライアントのコードが不完全な結果を取得することを防ぎます[。]{.chpule2}[
]{.chpuri2}
:::
:::::::::
::::::::::

::: {.section .checklist}
##  実装方法 {#checklist}

1.  利用できるすべてのプロダクトの表現[
    ]{.chpule1}[（]{.chpuri1}種類[）]{.chpule2}[
    ]{.chpuri2}を構築するために必要な共通の構築ステップが[、]{.chpule2}[
    ]{.chpuri2}明確に定義可能であることを確認してください[。]{.chpule2}[
    ]{.chpuri2}そうでなければ[、]{.chpule2}[
    ]{.chpuri2}パターンの実装に進むことはできません[。]{.chpule2}[
    ]{.chpuri2}

2.  これらのステップを[、]{.chpule2}[
    ]{.chpuri2}共通のビルダー・インターフェースで宣言します[。]{.chpule2}[
    ]{.chpuri2}

3.  プロダクトの表現ごとに具象ビルダー・クラスを作成し[、]{.chpule2}[
    ]{.chpuri2}その構築ステップを実装します[。]{.chpule2}[ ]{.chpuri2}

    構築の結果を取得するためのメソッドの実装をお忘れなく[。]{.chpule2}[
    ]{.chpuri2}このメソッドがビルダーのインターフェース内で宣言できない理由は[、]{.chpule2}[
    ]{.chpuri2}ビルダーによっては共通のインターフェースを持たないプロダクトを構築する可能性があるためです[。]{.chpule2}[
    ]{.chpuri2}したがって[、]{.chpule2}[
    ]{.chpuri2}メソッドの戻り値の型がわかりません[。]{.chpule2}[
    ]{.chpuri2}ただし[、]{.chpule2}[
    ]{.chpuri2}単一の階層にあるプロダクトを扱う場合は[、]{.chpule2}[
    ]{.chpuri2}結果取得用メソッドを安全に共通インターフェースに追加することが可能です[。]{.chpule2}[
    ]{.chpuri2}

4.  ディレクター・クラスの作成を検討してみてください[。]{.chpule2}[
    ]{.chpuri2}同じビルダーのオブジェクトを使用して[、]{.chpule2}[
    ]{.chpuri2}プロダクトを構築するための様々な方法を一箇所にまとめることが可能かもしれません[。]{.chpule2}[
    ]{.chpuri2}

5.  クライアント・コードで[、]{.chpule2}[
    ]{.chpuri2}ビルダーとディレクターの両方のオブジェクトを作成します[。]{.chpule2}[
    ]{.chpuri2}構築開始前に[、]{.chpule2}[
    ]{.chpuri2}クライアントはビルダーのオブジェクトをディレクターに渡す必要があります[。]{.chpule2}[
    ]{.chpuri2}通常[、]{.chpule2}[
    ]{.chpuri2}クライアントはディレクターのコンストラクターのパラメータを介して一度だけこれを行います[。]{.chpule2}[
    ]{.chpuri2}ディレクターは[、]{.chpule2}[
    ]{.chpuri2}そのビルダー・オブジェクトをその後のすべての構築で使用します[。]{.chpule2}[
    ]{.chpuri2}それに代わるやり方として[、]{.chpule2}[
    ]{.chpuri2}ディレクターの特定プロダクト用構築メソッドにビルダーを直接渡すということもできます[。]{.chpule2}[
    ]{.chpuri2}

6.  すべてのプロダクトが同じインターフェースに従っている場合は[、]{.chpule2}[
    ]{.chpuri2}構築結果をディレクターから直接取得できます[。]{.chpule2}[
    ]{.chpuri2}そうでない場合は[、]{.chpule2}[
    ]{.chpuri2}クライアントはビルダーから結果を取得する必要します[。]{.chpule2}[
    ]{.chpuri2}
:::

:::::: {.section .pros-cons}
##  長所と短所 {#pros-cons}

::::: row
::: col-sm-6
-    段階的にオブジェクトを作成したり[、]{.chpule2}[
    ]{.chpuri2}構築ステップを遅延させたり[、]{.chpule2}[
    ]{.chpuri2}再帰的にステップを実行することが可能[。]{.chpule2}[
    ]{.chpuri2}
-    プロダクトの様々な表現の作成に際して[、]{.chpule2}[
    ]{.chpuri2}同じ構築コードの再利用が可能[。]{.chpule2}[ ]{.chpuri2}
-   
    *[単]{.cjk}[一]{.cjk}[責]{.cjk}[任]{.cjk}[の]{.cjk}[原]{.cjk}[則]{.cjk}*[。]{.chpule2}[
    ]{.chpuri2}複雑な構築用コードを[、]{.chpule2}[
    ]{.chpuri2}プロダクトのビジネス・ロジックから分離可能[。]{.chpule2}[
    ]{.chpuri2}
:::

::: col-sm-6
-    本パターンでは[、]{.chpule2}[
    ]{.chpuri2}複数の新規クラス作成の必要があるため[、]{.chpule2}[
    ]{.chpuri2}コードの全体的な複雑さが増加[。]{.chpule2}[ ]{.chpuri2}
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

-   [Builder](builder.html) は[、]{.chpule2}[ ]{.chpuri2}複雑な
    [Composite](composite.html) ツリー作成に使用できます[。]{.chpule2}[
    ]{.chpuri2}構築ステップを再帰的に行なうように
    プログラムします[。]{.chpule2}[ ]{.chpuri2}

-   [Builder](builder.html) と [Bridge](bridge.html)
    を組み合わせることができます[：]{.chpule2}[
    ]{.chpuri2}ディレクター・クラスは抽象化層の役割を果たし[、]{.chpule2}[
    ]{.chpuri2}ビルダーは実装です[。]{.chpule2}[ ]{.chpuri2}

-   [Abstract Factories](abstract-factory.html)[、]{.chpule2}[
    ]{.chpuri2}[Builders](builder.html)[、]{.chpule2}[
    ]{.chpuri2}[Prototypes](prototype.html) はどれも
    [Singletons](singleton.html) で実装可能です[。]{.chpule2}[
    ]{.chpuri2}
:::

::: {.section .implementations}
##  コード例 {#implementations}

[![Builder を C#
で](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](builder/csharp/example.html "Builder を C# で"){.prog-lang-link}
[![Builder を C++
で](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](builder/cpp/example.html "Builder を C++ で"){.prog-lang-link}
[![Builder を Go
で](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](builder/go/example.html "Builder を Go で"){.prog-lang-link}
[![Builder を Java
で](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](builder/java/example.html "Builder を Java で"){.prog-lang-link}
[![Builder を PHP
で](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](builder/php/example.html "Builder を PHP で"){.prog-lang-link}
[![Builder を Python
で](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](builder/python/example.html "Builder を Python で"){.prog-lang-link}
[![Builder を Ruby
で](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](builder/ruby/example.html "Builder を Ruby で"){.prog-lang-link}
[![Builder を Rust
で](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](builder/rust/example.html "Builder を Rust で"){.prog-lang-link}
[![Builder を Swift
で](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](builder/swift/example.html "Builder を Swift で"){.prog-lang-link}
[![Builder を TypeScript
で](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](builder/typescript/example.html "Builder を TypeScript で"){.prog-lang-link}
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

[Prototype []{.fa .fa-arrow-right}](prototype.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### 戻る

[[]{.fa
.fa-arrow-left} ファクトリーの比較](factory-comparison.html){.btn
.btn-default rel="prev"}
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
