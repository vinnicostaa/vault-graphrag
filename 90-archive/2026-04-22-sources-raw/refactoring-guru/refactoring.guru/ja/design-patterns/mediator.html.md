::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} /
[デザインパターン](../design-patterns.html){.type} /
[振る舞いに関するパターン](behavioral-patterns.html){.type}
:::

# Mediator {#mediator .title}

::: aka
別名：[Intermediary、]{style="display:inline-block"}[Controller、]{style="display:inline-block"}[メディエーター、]{style="display:inline-block"}[インターメディアリー、]{style="display:inline-block"}[コントローラー]{style="display:inline-block"}
:::

::: {.section .intent}
##  一言でいうと {#intent}

**Mediator**[ ]{.chpule1}[（]{.chpuri1}メディエーター[、]{.chpule2}[
]{.chpuri2}仲介者[）]{.chpule2}[ ]{.chpuri2}は[、]{.chpule2}[
]{.chpuri2}振る舞いに関するデザインパターンの一つで[、]{.chpule2}[
]{.chpuri2}オブジェクト間の混沌とした依存性を削減します[。]{.chpule2}[
]{.chpuri2}パターンは[、]{.chpule2}[
]{.chpuri2}オブジェクト間の直接の通信を制限し[、]{.chpule2}[
]{.chpuri2}メディエーター・オブジェクトを介してのみの共同作業を強制します[。]{.chpule2}[
]{.chpuri2}

<figure class="image">
<img
src="../../images/patterns/content/mediator/mediator6761.png?id=0264bd857a231b6ea2d0c537c092e698"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/mediator/mediator.png?id=0264bd857a231b6ea2d0c537c092e698 640w,/images/patterns/content/mediator/mediator-1.5x.png?id=b3d5df41892274b5c84282bae8737441 960w,/images/patterns/content/mediator/mediator-2x.png?id=250c2bf72ca1fdee2e6d97ed5a4765f2 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Mediator デザインパターン" />
</figure>
:::

::: {.section .problem}
##  問題 {#problem}

たとえば[、]{.chpule2}[
]{.chpuri2}顧客プロフィールの作成と編集のためのダイアログがあると思ってください[。]{.chpule2}[
]{.chpuri2}これは[、]{.chpule2}[
]{.chpuri2}テキストフィールド[、]{.chpule2}[
]{.chpuri2}チェックボックス[、]{.chpule2}[
]{.chpuri2}ボタンなどの様々なフォーム・コントロールでできています[。]{.chpule2}[
]{.chpuri2}

<figure class="image">
<img
src="../../images/patterns/diagrams/mediator/problem1-ja6068.png?id=7e7ee8c102ecf9c450d3c83711d9a930"
style="aspect-ratio: 2.22;"
srcset="/images/patterns/diagrams/mediator/problem1-ja.png?id=7e7ee8c102ecf9c450d3c83711d9a930 600w,/images/patterns/diagrams/mediator/problem1-ja-1.5x.png?id=9c10e690c99806bc4da0903bbce3aef2 900w,/images/patterns/diagrams/mediator/problem1-ja-2x.png?id=9123fa203a9c2c9e4d3e1d3d157b6dab 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="ユーザー・インターフェースの要素間の混沌とした関係" />
<figcaption><p>アプリケーションの進化につれて<span
class="chpule2">、</span><span class="chpuri2">
</span>ユーザー・インターフェースの要素間の関係は混沌となる<span
class="chpule2">。</span><span class="chpuri2"> </span></p></figcaption>
</figure>

フォーム要素のいくつかは他の要素と相互に関係します[。]{.chpule2}[
]{.chpuri2}たとえば[[、]{.ls-1}]{.chpule2}[ ]{.chpuri2}​[
]{.chpule1}[「]{.chpuri1}犬を飼っています[」]{.chpule2}[
]{.chpuri2}のチェックボックスを選ぶと[、]{.chpule2}[
]{.chpuri2}今まで隠れていた犬の名前を入力するためのテキストフィールドが表示されます[。]{.chpule2}[
]{.chpuri2}別の例として[、]{.chpule2}[
]{.chpuri2}送信ボタンは[、]{.chpule2}[
]{.chpuri2}データを保存する前にすべてのフィールドの値を検証します[。]{.chpule2}[
]{.chpuri2}

<figure class="image">
<img
src="../../images/patterns/diagrams/mediator/problem2a7f7.png?id=072c51eee4dd90c0972866440c87c1c3"
style="aspect-ratio: 3.27;"
srcset="/images/patterns/diagrams/mediator/problem2.png?id=072c51eee4dd90c0972866440c87c1c3 360w,/images/patterns/diagrams/mediator/problem2-1.5x.png?id=b805abc211e60fa567fb114192e24608 540w,/images/patterns/diagrams/mediator/problem2-2x.png?id=d298ec82a47fa2938ed6cf64b7da78c1 720w"
sizes="(max-width: 720px) 100vw, 360px" loading="lazy" width="360"
alt="UI の要素は相互に依存" />
<figcaption><p>要素は他の要素同士と多くのの関係を持つ可能性あり<span
class="chpule2">。</span><span class="chpuri2"> </span>そのため<span
class="chpule2">、</span><span class="chpuri2">
</span>ある要素への変更は他の要素に影響を与えるかもしれない<span
class="chpule2">。</span><span class="chpuri2"> </span></p></figcaption>
</figure>

フォーム要素のコード内にこのロジックを直接実装することで[、]{.chpule2}[
]{.chpuri2}これらの要素のクラスをアプリの他のフォームで再利用するのが非常に困難になります[。]{.chpule2}[
]{.chpuri2}たとえば[、]{.chpule2}[
]{.chpuri2}このチェックボックスのクラスは[、]{.chpule2}[
]{.chpuri2}犬のテキストフィールドに結合されているため[、]{.chpule2}[
]{.chpuri2}別のフォーム内では使用できません[。]{.chpule2}[
]{.chpuri2}プロフィールのフォームのレンダリングをするには[、]{.chpule2}[
]{.chpuri2}関わるすべてのクラスを使用するか[、]{.chpule2}[
]{.chpuri2}あるいは[、]{.chpule2}[
]{.chpuri2}フォームを使うのをあきらめるかしかありません[。]{.chpule2}[
]{.chpuri2}
:::

::: {.section .solution}
##  解決策 {#solution}

Mediator パターンによれば[、]{.chpule2}[
]{.chpuri2}お互いに独立させたいコンポーネント間では直接の通信を断つべきです[。]{.chpule2}[
]{.chpuri2}代わりに[、]{.chpule2}[
]{.chpuri2}これらのコンポーネントは[、]{.chpule2}[
]{.chpuri2}呼び出しを適切なコンポーネントへ方向づけてくれる[、]{.chpule2}[
]{.chpuri2}特別なメディエーター・オブジェクトを呼び出し[、]{.chpule2}[
]{.chpuri2}間接的に共同作業を行う必要があります[。]{.chpule2}[
]{.chpuri2}そうすることにより[、]{.chpule2}[
]{.chpuri2}コンポーネントは多数の同僚のコンポーネントに結合するのではなく[、]{.chpule2}[
]{.chpuri2}一つのメディエーター・クラスにのみ依存します[。]{.chpule2}[
]{.chpuri2}

さきほどのプロフィール編集フォームの例では[、]{.chpule2}[
]{.chpuri2}ダイアログ・クラス自体がメディエーターとして機能するかもしれません[。]{.chpule2}[
]{.chpuri2}おそらく[、]{.chpule2}[
]{.chpuri2}ダイアログ・クラスはその下位要素のすべてをすでに知っているので[、]{.chpule2}[
]{.chpuri2}このクラスに新しい依存関係を導入する必要はありません[。]{.chpule2}[
]{.chpuri2}

<figure class="image">
<img
src="../../images/patterns/diagrams/mediator/solution1-jaf992.png?id=6be316621539f7893b4387c20cc13aba"
style="aspect-ratio: 2.22;"
srcset="/images/patterns/diagrams/mediator/solution1-ja.png?id=6be316621539f7893b4387c20cc13aba 600w,/images/patterns/diagrams/mediator/solution1-ja-1.5x.png?id=36660668904a47ca72a15db5b2ce18b3 900w,/images/patterns/diagrams/mediator/solution1-ja-2x.png?id=f0018706c693696bfe950bc781504b8f 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="UI 要素は、メディエーター・オブジェクトを介して通信すべし。" />
<figcaption><p>UI 要素は<span class="chpule2">、</span><span
class="chpuri2">
</span>メディエーター・オブジェクトを介して間接的に通信すべし<span
class="chpule2">。</span><span class="chpuri2"> </span></p></figcaption>
</figure>

最も重要な変更は[、]{.chpule2}[
]{.chpuri2}実際のフォーム要素に起こります[。]{.chpule2}[
]{.chpuri2}送信ボタンのことを考えてみましょう[。]{.chpule2}[
]{.chpuri2}以前は[、]{.chpule2}[
]{.chpuri2}ユーザーがボタンをクリックするたびに[、]{.chpule2}[
]{.chpuri2}フォームの全要素の値を検証する必要がありました[。]{.chpule2}[
]{.chpuri2}今では[、]{.chpule2}[
]{.chpuri2}その唯一の仕事は[、]{.chpule2}[
]{.chpuri2}クリックについてダイアログに通知することだけです[。]{.chpule2}[
]{.chpuri2}この通知を受信すると[、]{.chpule2}[
]{.chpuri2}ダイアログ自体が検証を実行するか[、]{.chpule2}[
]{.chpuri2}またはその仕事を個々の要素に渡します[。]{.chpule2}[
]{.chpuri2}したがって[、]{.chpule2}[ ]{.chpuri2}ボタンは[、]{.chpule2}[
]{.chpuri2}数十のフォーム要素に結び付けられておらず[、]{.chpule2}[
]{.chpuri2}ダイアログ・クラスだけに依存します[。]{.chpule2}[ ]{.chpuri2}

これをさらに進めて[、]{.chpule2}[
]{.chpuri2}すべての種類のダイアログに共通なインターフェースを抽出すれば[、]{.chpule2}[
]{.chpuri2}依存関係をさらに緩めることができます[。]{.chpule2}[
]{.chpuri2}インターフェースは[、]{.chpule2}[
]{.chpuri2}フォーム要素に発生したイベントをダイアログに通知するのに全フォーム要素が使える通知メソッドを宣言します[。]{.chpule2}[
]{.chpuri2}したがって[、]{.chpule2}[
]{.chpuri2}送信ボタンは[、]{.chpule2}[
]{.chpuri2}そのインターフェースを実装するどんなダイアログとでも動作できるようになります[。]{.chpule2}[
]{.chpuri2}

こういうふうにして[、]{.chpule2}[ ]{.chpuri2}Mediator
パターンにより[、]{.chpule2}[
]{.chpuri2}メディエーター・オブジェクト内のいろいろなオブジェクト間の複雑な関係を内に包むことができます[。]{.chpule2}[
]{.chpuri2}
:::

::: {.section .analogy}
##  現実世界でのたとえ {#analogy}

<figure class="image">
<img
src="../../images/patterns/diagrams/mediator/live-example3f2a.png?id=aa1de3cb7b63aa623e63578a1e43399a"
style="aspect-ratio: 1.85;"
srcset="/images/patterns/diagrams/mediator/live-example.png?id=aa1de3cb7b63aa623e63578a1e43399a 370w,/images/patterns/diagrams/mediator/live-example-1.5x.png?id=315109aa1099cc6e7c06057fa139881c 555w,/images/patterns/diagrams/mediator/live-example-2x.png?id=fd55a9f9d8cf49effa223555c7369504 740w"
sizes="(max-width: 720px) 100vw, 370px" loading="lazy" width="370"
alt="航空交通管制塔" />
<figcaption><p>航空機のパイロットは<span class="chpule2">、</span><span
class="chpuri2"> </span>次に誰が着陸するかを決める時に<span
class="chpule2">、</span><span class="chpuri2">
</span>互いに直接話すことはしない<span class="chpule2">。</span><span
class="chpuri2"> </span>すべての通信は管制塔を通して行われる<span
class="chpule2">。</span><span class="chpuri2"> </span></p></figcaption>
</figure>

空港に接近したり[、]{.chpule2}[
]{.chpuri2}空港から出発する航空機のパイロットは[、]{.chpule2}[
]{.chpuri2}互いに直接連絡を取ることはしません[。]{.chpule2}[
]{.chpuri2}代わりに[、]{.chpule2}[
]{.chpuri2}滑走路近くのどこかの高い塔に座っている航空交通管制官と話します[。]{.chpule2}[
]{.chpuri2}航空交通管制官がいなければ[、]{.chpule2}[
]{.chpuri2}パイロットたちは[、]{.chpule2}[
]{.chpuri2}空港近くのあらゆる飛行機を気にして[、]{.chpule2}[
]{.chpuri2}他の何十人ものパイロットたちと着陸の優先順位について話し合うことになります[。]{.chpule2}[
]{.chpuri2}おそらくそれは飛行機の墜落率を急増させることになるでしょう[。]{.chpule2}[
]{.chpuri2}

その管制塔は飛行全部を管理するわけではありません[。]{.chpule2}[
]{.chpuri2}管制塔は[、]{.chpule2}[
]{.chpuri2}飛行の末端での制限を守らせるためにあります[。]{.chpule2}[
]{.chpuri2}その辺では[、]{.chpule2}[
]{.chpuri2}パイロット一人でさばききれないだけの数の関係者が存在するからです[。]{.chpule2}[
]{.chpuri2}
:::

::::: {.section .structure-container}
##  構造 {#structure}

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/mediator/structure71f0.png?id=1f2accc7820ecfe9665b6d30cbc0bc61"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.21;"
srcset="/images/patterns/diagrams/mediator/structure.png?id=1f2accc7820ecfe9665b6d30cbc0bc61 520w,/images/patterns/diagrams/mediator/structure-1.5x.png?id=958b373815bf6a56cd9b38763ed01dce 780w,/images/patterns/diagrams/mediator/structure-2x.png?id=5191daa1c9d4caa36e38af3c5b7d1522 1040w"
sizes="(max-width: 720px) 100vw, 520px" loading="lazy" width="520"
alt="Mediator デザインパターンの構造" /><img
src="../../images/patterns/diagrams/mediator/structure-indexed5302.png?id=a82d4cf1b92a4f72af32f231ffd21131"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.21;"
srcset="/images/patterns/diagrams/mediator/structure-indexed.png?id=a82d4cf1b92a4f72af32f231ffd21131 520w,/images/patterns/diagrams/mediator/structure-indexed-1.5x.png?id=81d4f842ae5157a3cb09f8f3c05159dd 780w,/images/patterns/diagrams/mediator/structure-indexed-2x.png?id=88722da01a5c82b0452078c9339ca497 1040w"
sizes="(max-width: 720px) 100vw, 520px" loading="lazy" width="520"
alt="Mediator デザインパターンの構造" />
</figure>
:::

1.  **コンポーネント**[
    ]{.chpule1}[（]{.chpuri1}Component[）]{.chpule2}[
    ]{.chpuri2}は[、]{.chpule2}[
    ]{.chpuri2}何らかのビジネス・ロジックを含んだ種々のクラスです[。]{.chpule2}[
    ]{.chpuri2}各コンポーネントは[、]{.chpule2}[
    ]{.chpuri2}メディエーター・インターフェースの型で宣言されたメディエーターへの参照を持っています[。]{.chpule2}[
    ]{.chpuri2}コンポーネントは[、]{.chpule2}[
    ]{.chpuri2}メディエーターの実際のクラスが何かは知りません[。]{.chpule2}[
    ]{.chpuri2}そのため[、]{.chpule2}[
    ]{.chpuri2}別のメディエーターにリンクすることで[、]{.chpule2}[
    ]{.chpuri2}他のプログラムのコンポーネントを再利用することができます[。]{.chpule2}[
    ]{.chpuri2}

2.  **メディエーター**[ ]{.chpule1}[（]{.chpuri1}Mediator[）]{.chpule2}[
    ]{.chpuri2}インターフェースは[、]{.chpule2}[
    ]{.chpuri2}コンポーネントとの通信するメソッドを宣言します[。]{.chpule2}[
    ]{.chpuri2}通常は[、]{.chpule2}[
    ]{.chpuri2}通知メソッド一つだけからなります[。]{.chpule2}[
    ]{.chpuri2}コンポーネントは[、]{.chpule2}[
    ]{.chpuri2}このメソッドの引数として任意のコンテキストを渡すことができます[。]{.chpule2}[
    ]{.chpuri2}それはオブジェクト自身でもかまいませんが[、]{.chpule2}[
    ]{.chpuri2}受け手のコンポーネントと送り手のクラスが結合しないような方法に限ります[。]{.chpule2}[
    ]{.chpuri2}

3.  **具象メディエーター**[ ]{.chpule1}[（]{.chpuri1}Concrete
    Mediator[）]{.chpule2}[ ]{.chpuri2}は[、]{.chpule2}[
    ]{.chpuri2}様々なコンポーネント間の関係を内に隠します[。]{.chpule2}[
    ]{.chpuri2}具象メディエーターはしばしば[、]{.chpule2}[
    ]{.chpuri2}管理するすべてのコンポーネントへの参照を保持し[、]{.chpule2}[
    ]{.chpuri2}時にはそのライフサイクルを管理することもあります[。]{.chpule2}[
    ]{.chpuri2}

4.  コンポーネント[ ]{.chpule1}[（]{.chpuri1}Component[）]{.chpule2}[
    ]{.chpuri2}は他のコンポーネントの存在を知る必要はありません[。]{.chpule2}[
    ]{.chpuri2}コンポーネント内で[、]{.chpule2}[
    ]{.chpuri2}コンポーネントに対して[、]{.chpule2}[
    ]{.chpuri2}何か重要なことが発生した場合は[、]{.chpule2}[
    ]{.chpuri2}メディエーターにのみ通知する必要があります[。]{.chpule2}[
    ]{.chpuri2}メディエーターは通知を受け取ると[、]{.chpule2}[
    ]{.chpuri2}送り手を簡単に識別でき[、]{.chpule2}[
    ]{.chpuri2}どのコンポーネントの引き金を引けばいいかを決めるには[、]{.chpule2}[
    ]{.chpuri2}その情報だけで十分かもしれません[。]{.chpule2}[
    ]{.chpuri2}

    コンポーネントの観点からすると[、]{.chpule2}[
    ]{.chpuri2}すべては[、]{.chpule2}[
    ]{.chpuri2}完全なブラックボックスのように見えます[。]{.chpule2}[
    ]{.chpuri2}送り手は誰がそのリクエストを処理することになるのかはわからず[、]{.chpule2}[
    ]{.chpuri2}受け手は誰が最初にリクエストを送信したのかを知りません[。]{.chpule2}[
    ]{.chpuri2}
::::
:::::

::: {.section .pseudocode}
##  擬似コード {#pseudocode}

この例では[、]{.chpule2}[ ]{.chpuri2}**Mediator**
パターンを使用して[、]{.chpule2}[ ]{.chpuri2}ボタン[、]{.chpule2}[
]{.chpuri2}チェックボックス[、]{.chpule2}[
]{.chpuri2}テキストラベルなど[、]{.chpule2}[ ]{.chpuri2}様々な UI
クラス間の相互依存関係を解消しています[。]{.chpule2}[ ]{.chpuri2}

<figure class="image">
<img
src="../../images/patterns/diagrams/mediator/example576a.png?id=3151c153533e816e226be0ef977211e8"
style="aspect-ratio: 1.24;"
srcset="/images/patterns/diagrams/mediator/example.png?id=3151c153533e816e226be0ef977211e8 560w,/images/patterns/diagrams/mediator/example-1.5x.png?id=cf441780d96f3306ac54d6809f85b87d 840w,/images/patterns/diagrams/mediator/example-2x.png?id=02064e5a7c4f065f806747e1b04ac1b0 1120w"
sizes="(max-width: 720px) 100vw, 560px" loading="lazy" width="560"
alt="Mediator パターン例の構造" />
<figcaption><p>UI ダイアログ・クラスの構造</p></figcaption>
</figure>

ユーザーに起動された要素は[、]{.chpule2}[
]{.chpuri2}他の要素と直接通信するのがよさそうに見えても[、]{.chpule2}[
]{.chpuri2}そうしません[。]{.chpule2}[
]{.chpuri2}代わりに[、]{.chpule2}[
]{.chpuri2}要素はそのメディエーターにイベントについて通知するだけです[。]{.chpule2}[
]{.chpuri2}周辺情報も[、]{.chpule2}[
]{.chpuri2}その通知とともに渡します[。]{.chpule2}[ ]{.chpuri2}

この例では[、]{.chpule2}[
]{.chpuri2}認証ダイアログ全体がメディエーターとして機能します[。]{.chpule2}[
]{.chpuri2}メディエーターは[、]{.chpule2}[
]{.chpuri2}具象要素同士が連携するやり方を知っており[、]{.chpule2}[
]{.chpuri2}間接的な通信を主導します[。]{.chpule2}[
]{.chpuri2}イベントに関する通知を受信すると[、]{.chpule2}[
]{.chpuri2}どの要素がイベントに対応すべきかをダイアログが決定し[、]{.chpule2}[
]{.chpuri2}そこへ呼び出しをまわします[。]{.chpule2}[ ]{.chpuri2}

<figure class="code">
<pre class="code" lang="pseudocode"><code>// メディエーター・インターフェースは、様々なイベントに関してコンポーネン
// トがメディエーターに通知するために使用するメソッドを宣言。メディエー
// ターは、これらのイベントに反応し、実行を他のコンポーネントに渡すことが
// できる。
interface Mediator is
    method notify(sender: Component, event: string)


// 具象メディエーター・クラス。個々のコンポーネント間の複雑に絡み合った関
// 係は解きほぐされて、メディエーターへ移動。
class AuthenticationDialog implements Mediator is
    private field title: string
    private field loginOrRegisterChkBx: Checkbox
    private field loginUsername, loginPassword: Textbox
    private field registrationUsername, registrationPassword,
                  registrationEmail: Textbox
    private field okBtn, cancelBtn: Button

    constructor AuthenticationDialog() is
        // すべてのコンポーネント・オブジェクトを作成。その際、現在のメ
        // ディエーターをコンストラクターに渡し、リンクを確立する。

    // ひとつのコンポーネントに何かが起きると、それはメディエーターに通知
    // される。通知を受け取った後、メディエーターは自分で何らかのことを行
    // うか、リクエストを他のコンポーネントに渡す。
    method notify(sender, event) is
        if (sender == loginOrRegisterChkBx and event == &quot;check&quot;)
            if (loginOrRegisterChkBx.checked)
                title = &quot;Log in&quot;
                // 1. ログイン・フォームのコンポーネントを表示。
                // 2. 登録フォームのコンポーネントを隠す。
            else
                title = &quot;Register&quot;
                // 1. 登録フォームのコンポーネントを表示。
                // 2. ログイン・フォームのコンポーネントを隠す。

        if (sender == okBtn &amp;&amp; event == &quot;click&quot;)
            if (loginOrRegister.checked)
                // ログイン資格情報を使用してユーザーを探すことを試みる。
                if (!found)
                    // ログイン・フィールドの上にエラーメッセージを表示。
            else
                // 1. 登録フィールドからのデータを使用してユーザー・ア
                // カウントを作成。
                // 2. そのユーザーをログイン。
                // ……


// コンポーネントは、メディエーター・オブジェクトとメディエーター・イン
// ターフェースを介して通信を行う。このため、同じコンポーネントのクラスを
// 違うメディエーター・オブジェクトと結びつけることによって、違う状況下で
// 使用することが可能。
class Component is
    field dialog: Mediator

    constructor Component(dialog) is
        this.dialog = dialog

    method click() is
        dialog.notify(this, &quot;click&quot;)

    method keypress() is
        dialog.notify(this, &quot;keypress&quot;)

// 具象コンポーネント同士は話をしない。唯一の通信手段は、メディエーターへ
// 通知を送ること。
class Button extends Component is
    // ……

class Textbox extends Component is
    // ……

class Checkbox extends Component is
    method check() is
        dialog.notify(this, &quot;check&quot;)
    // ……</code></pre>
</figure>
:::

:::::::::: {.section .applicability-container}
##  適応性 {#applicability}

::::::::: applicability
::: applicability-problem
クラスが他の多くのクラスに密に結合しているため変更が難しい時に[、]{.chpule2}[
]{.chpuri2}Mediator パターンを適用してください[。]{.chpule2}[
]{.chpuri2}
:::

::: applicability-solution
このパターンは[、]{.chpule2}[
]{.chpuri2}クラス間の関係のすべてを別のクラスに抽出します[。]{.chpule2}[
]{.chpuri2}特定コンポーネントへの変更を[、]{.chpule2}[
]{.chpuri2}残りのコンポーネントから隔離します[。]{.chpule2}[ ]{.chpuri2}
:::

::: applicability-problem
他コンポーネントへの依存度が高いため[、]{.chpule2}[
]{.chpuri2}違うプログラムのコンポーネントを再利用できない時[、]{.chpule2}[
]{.chpuri2}このパターンを使ってください[。]{.chpule2}[ ]{.chpuri2}
:::

::: applicability-solution
Mediator を適用すると[、]{.chpule2}[
]{.chpuri2}個々のコンポーネントは他コンポーネントの存在に気づかなくなります[。]{.chpule2}[
]{.chpuri2}間接的ではありますが[、]{.chpule2}[
]{.chpuri2}コンポーネント同士はメディエーター・オブジェクトを通してまだ通信することができます[。]{.chpule2}[
]{.chpuri2}別のアプリケーションでコンポーネントを再利用するには[、]{.chpule2}[
]{.chpuri2}新しいメディエーター・クラスを作成する必要があります[。]{.chpule2}[
]{.chpuri2}
:::

::: applicability-problem
様々な状況で基本的な振る舞いを再利用するだけのために[、]{.chpule2}[
]{.chpuri2}とんでもない数のコンポーネントのサブクラスを作成していることに気づいたら[、]{.chpule2}[
]{.chpuri2}Mediator を使ってください[。]{.chpule2}[ ]{.chpuri2}
:::

::: applicability-solution
コンポーネント間のすべての関係がメディエーターに含まれているため[、]{.chpule2}[
]{.chpuri2}コンポーネント自体は変更せず[、]{.chpule2}[
]{.chpuri2}新しいメディエーターを導入するたけで[、]{.chpule2}[
]{.chpuri2}これらのコンポーネントが協力して機能するためのまったく新しい方法を簡単に定義できます[。]{.chpule2}[
]{.chpuri2}
:::
:::::::::
::::::::::

::: {.section .checklist}
##  実装方法 {#checklist}

1.  独立性の強化の恩恵を受けそうな密に結合されたクラスのグループを特定します[
    ]{.chpule1}[（]{.chpuri1}例[：]{.chpule2}[
    ]{.chpuri2}保守を容易にする[。]{.chpule2}[
    ]{.chpuri2}クラス再利用の簡易化[）]{.ls-05}[。]{.chpule2}[
    ]{.chpuri2}

2.  メディエーター・インターフェースを宣言し[、]{.chpule2}[
    ]{.chpuri2}メディエーターと様々なコンポーネント間の望ましい通信プロトコルを記述します[。]{.chpule2}[
    ]{.chpuri2}多くの場合[、]{.chpule2}[
    ]{.chpuri2}コンポーネントから通知を受け取るメソッド一つで十分です[。]{.chpule2}[
    ]{.chpuri2}

    異なる状況でコンポーネント・クラスを再利用したい場合[、]{.chpule2}[
    ]{.chpuri2}このインターフェースは[、]{.chpule2}[
    ]{.chpuri2}極めて重要な役を果たします[。]{.chpule2}[
    ]{.chpuri2}コンポーネントが一般的インターフェースを介してメディエーターと動作する限り[、]{.chpule2}[
    ]{.chpuri2}コンポーネントを異なる実装のメディエーターとリンクさせることができます[。]{.chpule2}[
    ]{.chpuri2}

3.  具象メディエーター・クラスを実装します[。]{.chpule2}[
    ]{.chpuri2}このクラスの管理するすべてのコンポーネントへの参照を格納するようにしておくと[、]{.chpule2}[
    ]{.chpuri2}メディエーターのメソッドからどのコンポーネントでも呼ぶことができ[、]{.chpule2}[
    ]{.chpuri2}便利です[。]{.chpule2}[ ]{.chpuri2}

4.  さらに進んで[、]{.chpule2}[
    ]{.chpuri2}コンポーネント・オブジェクトの作成と破壊をメディエーターにやらせるようにもできます[。]{.chpule2}[
    ]{.chpuri2}こうすると[、]{.chpule2}[
    ]{.chpuri2}メディエーターは[、]{.chpule2}[
    ]{.chpuri2}[ファクトリー](abstract-factory.html)や[ファサード](facade.html)に類似してくるかもしれません[。]{.chpule2}[
    ]{.chpuri2}

5.  コンポーネントはメディエーター・オブジェクトへの参照を格納すべきです[。]{.chpule2}[
    ]{.chpuri2}この接続は通常[、]{.chpule2}[
    ]{.chpuri2}コンポーネントのコンストラクター内で確立され[、]{.chpule2}[
    ]{.chpuri2}コンストラクターにはメディエーター・オブジェクトが引数として渡されます[。]{.chpule2}[
    ]{.chpuri2}

6.  コンポーネントのコードを変更して[、]{.chpule2}[
    ]{.chpuri2}他のコンポーネントのメソッドの代わりに[、]{.chpule2}[
    ]{.chpuri2}メディエーターの通知メソッドを呼び出すように
    します[。]{.chpule2}[
    ]{.chpuri2}他のコンポーネントを呼び出すようなコードを抽出して[、]{.chpule2}[
    ]{.chpuri2}メディエーター・クラスに入れます[。]{.chpule2}[
    ]{.chpuri2}メディエーターがそのコンポーネントから通知を受けるたびに[、]{.chpule2}[
    ]{.chpuri2}このコードを実行します[。]{.chpule2}[ ]{.chpuri2}
:::

:::::: {.section .pros-cons}
##  長所と短所 {#pros-cons}

::::: row
::: col-sm-6
-   
    *[単]{.cjk}[一]{.cjk}[責]{.cjk}[任]{.cjk}[の]{.cjk}[原]{.cjk}[則]{.cjk}*[。]{.chpule2}[
    ]{.chpuri2}様々なコンポーネント間の通信を一箇所に抽出することで[、]{.chpule2}[
    ]{.chpuri2}理解と保守を容易にする[。]{.chpule2}[ ]{.chpuri2}
-   
    *[開]{.cjk}[放]{.cjk}[閉]{.cjk}[鎖]{.cjk}[の]{.cjk}[原]{.cjk}[則]{.cjk}*[。]{.chpule2}[
    ]{.chpuri2}実際のクライアントの変更の必要なしに新規メディエーターを導入可能[。]{.chpule2}[
    ]{.chpuri2}
-   
    プログラムの様々なコンポーネント間の結合度の削減が可能[。]{.chpule2}[
    ]{.chpuri2}
-    個々のコンポーネントの再利用が容易に可能[。]{.chpule2}[ ]{.chpuri2}
:::

::: col-sm-6
-    時間が経つにつれて[、]{.chpule2}[
    ]{.chpuri2}メディエーターは[神オブジェクト](https://en.wikipedia.org/wiki/God_object)に進化するかもしれない[。]{.chpule2}[
    ]{.chpuri2}
:::
:::::
::::::

::: {.section .relations}
##  他のパターンとの関係 {#relations}

-   [Chain of Responsibility](chain-of-responsibility.html) と
    [Command](command.html) と [Mediator](mediator.html) と
    [Observer](observer.html) は[、]{.chpule2}[
    ]{.chpuri2}リクエストの送り手と受け手を接続する様々な方法を示します[：]{.chpule2}[
    ]{.chpuri2}

    -   *Chain of Responsibility* は[、]{.chpule2}[
        ]{.chpuri2}潜在的受け手の動的な連鎖に沿って[、]{.chpule2}[
        ]{.chpuri2}どれか一つが処理するまで[、]{.chpule2}[
        ]{.chpuri2}リクエストを順番に渡します[。]{.chpule2}[ ]{.chpuri2}
    -   *Command* は[、]{.chpule2}[
        ]{.chpuri2}送り手と受け手との間で単方向の接続を確立します[。]{.chpule2}[
        ]{.chpuri2}
    -   *Mediator* は[、]{.chpule2}[
        ]{.chpuri2}送り手と受け手の間の直接の接続を削除し[、]{.chpule2}[
        ]{.chpuri2}メディエーター・オブジェクトを介しての間接的通信を強制します[。]{.chpule2}[
        ]{.chpuri2}
    -   *Observer* では[、]{.chpule2}[
        ]{.chpuri2}受け手が動的にリクエストの受信申し込みをしたり[、]{.chpule2}[
        ]{.chpuri2}申し込み取り消しをしたりできます[。]{.chpule2}[
        ]{.chpuri2}

-   [Facade](facade.html) と [Mediator](mediator.html) は[、]{.chpule2}[
    ]{.chpuri2}多くの密に結合されたクラス間の協力関係を組織するという似た目的があります[。]{.chpule2}[
    ]{.chpuri2}

    -   *Facade* は[、]{.chpule2}[
        ]{.chpuri2}オブジェクトのサブシステムへの簡略化されたインターフェースを定義しますが[、]{.chpule2}[
        ]{.chpuri2}新しい機能が導入されるわけではありません[。]{.chpule2}[
        ]{.chpuri2}サブシステム自体はファサードを認識しません[。]{.chpule2}[
        ]{.chpuri2}サブシステム内のオブジェクト同士は直接やりとりします[。]{.chpule2}[
        ]{.chpuri2}
    -   *Mediator* は[、]{.chpule2}[
        ]{.chpuri2}システムのコンポーネント間の通信を一元化します[。]{.chpule2}[
        ]{.chpuri2}コンポーネントはメディエーター・オブジェクトについてのみ知っており[、]{.chpule2}[
        ]{.chpuri2}直接やりとりしません[。]{.chpule2}[ ]{.chpuri2}

-   [Mediator](mediator.html) と [Observer](observer.html)
    との違いは[、]{.chpule2}[
    ]{.chpuri2}理解に苦しむことがあります[。]{.chpule2}[
    ]{.chpuri2}ほとんどの場合[、]{.chpule2}[
    ]{.chpuri2}これらのパターンのいずれかを実装すればいですが[、]{.chpule2}[
    ]{.chpuri2}両方を同時に適用することもできます[。]{.chpule2}[
    ]{.chpuri2}どうすればそれができるか見てみましょう[。]{.chpule2}[
    ]{.chpuri2}

    *Mediator* の主目的は[、]{.chpule2}[
    ]{.chpuri2}システム構成コンポーネント間の相互依存をなくすことです[。]{.chpule2}[
    ]{.chpuri2}その代わりに[、]{.chpule2}[
    ]{.chpuri2}これらのコンポーネントは単一のメディエーター・オブジェクトに依存するようになります[。]{.chpule2}[
    ]{.chpuri2}*Observer* の主目的は[、]{.chpule2}[
    ]{.chpuri2}オブジェクト間の動的な単方向の接続を確立することにあり[、]{.chpule2}[
    ]{.chpuri2}そこではあるオブジェクトが他のオブジェクトの部下として動作します[。]{.chpule2}[
    ]{.chpuri2}

    *Observer* に依存した[、]{.chpule2}[ ]{.chpuri2}*Mediator*
    パターンの有名な実装方法があります[。]{.chpule2}[
    ]{.chpuri2}メディエーター・オブジェクトがパブリッシャーとしての役割を果たし[、]{.chpule2}[
    ]{.chpuri2}他のコンポーネントがサブスクライバーとしてメディエーターのイベントに通知依頼をしたり依頼解除をします[。]{.chpule2}[
    ]{.chpuri2}このように *Mediator* を実装すると[、]{.chpule2}[
    ]{.chpuri2}*Observer* と非常に似たよう見えます[。]{.chpule2}[
    ]{.chpuri2}

    混乱した時は[、]{.chpule2}[ ]{.chpuri2}Mediator
    パターンは[、]{.chpule2}[
    ]{.chpuri2}他の方法でも実装できるということを忘れないでください[。]{.chpule2}[
    ]{.chpuri2}たとえば[、]{.chpule2}[
    ]{.chpuri2}同一のメディエーター・オブジェクトにすべてのコンポーネントを恒久的にリンクできます[。]{.chpule2}[
    ]{.chpuri2}この実装方法は[、]{.chpule2}[ ]{.chpuri2}*Observer*
    には似ていませんが[、]{.chpule2}[ ]{.chpuri2}Mediator
    パターンの適用例の一つと言えます[。]{.chpule2}[ ]{.chpuri2}

    ここで[、]{.chpule2}[
    ]{.chpuri2}プログラム中のすべてのコンポーネントがパブリッシャーとなり[、]{.chpule2}[
    ]{.chpuri2}お互いに動的な接続が許された状況を想像してみてください[。]{.chpule2}[
    ]{.chpuri2}ここでは[、]{.chpule2}[
    ]{.chpuri2}中心となるメディエーター・オブジェクトはなく[、]{.chpule2}[
    ]{.chpuri2}分散されたオブザーバーの集団があるだけです[。]{.chpule2}[
    ]{.chpuri2}
:::

::: {.section .implementations}
##  コード例 {#implementations}

[![Mediator を C#
で](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](mediator/csharp/example.html "Mediator を C# で"){.prog-lang-link}
[![Mediator を C++
で](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](mediator/cpp/example.html "Mediator を C++ で"){.prog-lang-link}
[![Mediator を Go
で](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](mediator/go/example.html "Mediator を Go で"){.prog-lang-link}
[![Mediator を Java
で](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](mediator/java/example.html "Mediator を Java で"){.prog-lang-link}
[![Mediator を PHP
で](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](mediator/php/example.html "Mediator を PHP で"){.prog-lang-link}
[![Mediator を Python
で](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](mediator/python/example.html "Mediator を Python で"){.prog-lang-link}
[![Mediator を Ruby
で](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](mediator/ruby/example.html "Mediator を Ruby で"){.prog-lang-link}
[![Mediator を Rust
で](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](mediator/rust/example.html "Mediator を Rust で"){.prog-lang-link}
[![Mediator を Swift
で](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](mediator/swift/example.html "Mediator を Swift で"){.prog-lang-link}
[![Mediator を TypeScript
で](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](mediator/typescript/example.html "Mediator を TypeScript で"){.prog-lang-link}
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

[Memento []{.fa .fa-arrow-right}](memento.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### 戻る

[[]{.fa .fa-arrow-left} Iterator](iterator.html){.btn .btn-default
rel="prev"}
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
