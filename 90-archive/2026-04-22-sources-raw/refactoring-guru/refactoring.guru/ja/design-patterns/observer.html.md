::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} /
[デザインパターン](../design-patterns.html){.type} /
[振る舞いに関するパターン](behavioral-patterns.html){.type}
:::

# Observer {#observer .title}

::: aka
別名：[Event-Subscriber、]{style="display:inline-block"}[Listener、]{style="display:inline-block"}[オブザーバー、]{style="display:inline-block"}[イベント・サブスクライバー、]{style="display:inline-block"}[リスナー]{style="display:inline-block"}
:::

::: {.section .intent}
##  一言でいうと {#intent}

**Observer**[ ]{.chpule1}[（]{.chpuri1}オブザーバー[、]{.chpule2}[
]{.chpuri2}観察者[）]{.chpule2}[ ]{.chpuri2}は[、]{.chpule2}[
]{.chpuri2}振る舞いに関するデザインパターンの一つで[、]{.chpule2}[
]{.chpuri2}サブスクリプション[
]{.chpule1}[（]{.chpuri1}通知申し込み[）]{.chpule2}[
]{.chpuri2}の仕組みを定義することにより[、]{.chpule2}[
]{.chpuri2}観察対象オブジェクトに何かイベントが発生した時[、]{.chpule2}[
]{.chpuri2}そのイベントの観察者である複数のオブジェクトへ通知を行います[。]{.chpule2}[
]{.chpuri2}

<figure class="image">
<img
src="../../images/patterns/content/observer/observerd508.png?id=6088e31e1b0d4a417506a66614dcf065"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/observer/observer.png?id=6088e31e1b0d4a417506a66614dcf065 640w,/images/patterns/content/observer/observer-1.5x.png?id=aa64f9f336354462b57bbff5c09d8269 960w,/images/patterns/content/observer/observer-2x.png?id=d5a83e115528e9fd633f04ad2650f1db 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Observer デザインパターン" />
</figure>
:::

::: {.section .problem}
##  問題 {#problem}

`Customer`[ ]{.chpule1}[（]{.chpuri1}顧客[）]{.chpule2}[ ]{.chpuri2}と
`Store`[ ]{.chpule1}[（]{.chpuri1}店舗[）]{.chpule2}[ ]{.chpuri2}という
2 種類のオブジェクトがあるとします[。]{.chpule2}[
]{.chpuri2}顧客は特定の製品ブランド[
]{.chpule1}[（]{.chpuri1}たとえば[、]{.chpule2}[ ]{.chpuri2}iPhone
の最新モデル[）]{.chpule2}[
]{.chpuri2}に非常に興味を持っていて[、]{.chpule2}[
]{.chpuri2}それは近日中に店舗で発売開始の予定です[。]{.chpule2}[
]{.chpuri2}

客は毎日店舗を訪れ[、]{.chpule2}[
]{.chpuri2}商品があるかどうかを確認することも可能です[。]{.chpule2}[
]{.chpuri2}しかし[、]{.chpule2}[
]{.chpuri2}製品はまだ配送中なので[、]{.chpule2}[
]{.chpuri2}ほとんどの訪問は無駄に終わることになるでしょう[。]{.chpule2}[
]{.chpuri2}

<figure class="image">
<img
src="../../images/patterns/content/observer/observer-comic-1-ja4276.png?id=d8ffe078e167e8c7e51b4ae33b06a4a3"
style="aspect-ratio: 2;"
srcset="/images/patterns/content/observer/observer-comic-1-ja.png?id=d8ffe078e167e8c7e51b4ae33b06a4a3 600w,/images/patterns/content/observer/observer-comic-1-ja-1.5x.png?id=d627e3675a57942c0f36f99629fb8d57 900w,/images/patterns/content/observer/observer-comic-1-ja-2x.png?id=c7e8883e5e1edbfacc2b0a46520ec916 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="店舗への訪問 対 スパム送信" />
<figcaption><p>店舗への訪問 対 スパム送信</p></figcaption>
</figure>

一方[、]{.chpule2}[ ]{.chpuri2}店は[、]{.chpule2}[
]{.chpuri2}新しい製品が入荷するたびに[、]{.chpule2}[
]{.chpuri2}すべての顧客に[
]{.chpule1}[（]{.chpuri1}スパムとみなされる可能性がある[）]{.chpule2}[
]{.chpuri2}大量の電子メールを送信することも可能です[。]{.chpule2}[
]{.chpuri2}これで[、]{.chpule2}[
]{.chpuri2}顧客の何割かは[、]{.chpule2}[
]{.chpuri2}度重なる店舗への訪問をせずにすみ[、]{.chpule2}[
]{.chpuri2}助かります[。]{.chpule2}[ ]{.chpuri2}同時に[、]{.chpule2}[
]{.chpuri2}新製品には関心のない顧客を怒らせることになります[。]{.chpule2}[
]{.chpuri2}

対立した状況です[。]{.chpule2}[
]{.chpuri2}顧客が製品の入荷状況を調べるために時間を無駄にするか[、]{.chpule2}[
]{.chpuri2}それとも店が不適切な顧客に通知をして資源を無駄にするか[。]{.chpule2}[
]{.chpuri2}
:::

::: {.section .solution}
##  解決策 {#solution}

関心を惹かれるような状態を持つオブジェクトは[、]{.chpule2}[
]{.chpuri2}しばしば*[サ]{.cjk}[ブ]{.cjk}[ジ]{.cjk}[ェ]{.cjk}[ク]{.cjk}[ト]{.cjk}*[
]{.chpule1}[（]{.chpuri1}subject[、]{.chpule2}[
]{.chpuri2}主体[）]{.chpule2}[
]{.chpuri2}とも呼ばれますが[、]{.chpule2}[
]{.chpuri2}その状態の変化を他のオブジェクトに通知するため[、]{.chpule2}[
]{.chpuri2}我々は[、]{.chpule2}[
]{.chpuri2}*[パ]{.cjk}[ブ]{.cjk}[リ]{.cjk}[ッ]{.cjk}[シ]{.cjk}[ャ]{.cjk}[ー]{.cjk}*[
]{.chpule1}[（]{.chpuri1}publisher[、]{.chpule2}[
]{.chpuri2}発行者[）]{.chpule2}[
]{.chpuri2}と呼ぶことにします[。]{.chpule2}[
]{.chpuri2}パブリッシャーの状態の変化を追いかけたい他のオブジェクトは[、]{.chpule2}[
]{.chpuri2}すべて
*[サ]{.cjk}[ブ]{.cjk}[ス]{.cjk}[ク]{.cjk}[ラ]{.cjk}[イ]{.cjk}[バ]{.cjk}[ー]{.cjk}*[
]{.chpule1}[（]{.chpuri1}subscriber[、]{.chpule2}[
]{.chpuri2}予約購読者[）]{.chpule2}[
]{.chpuri2}と呼ばれます[。]{.chpule2}[ ]{.chpuri2}

Observer パターンでは[、]{.chpule2}[
]{.chpuri2}パブリッシャー・クラスにサブスクリプションの仕組みを追加し[、]{.chpule2}[
]{.chpuri2}個々のオブジェクトがそのパブリッシャーから来る一連のイベントの通知を依頼するか[、]{.chpule2}[
]{.chpuri2}通知を解除することができるようにします[。]{.chpule2}[
]{.chpuri2}えっ[、]{.chpule2}[ ]{.chpuri2}何だか難しそう[？]{.chpule2}[
]{.chpuri2}大丈夫です[！]{.chpule2}[
]{.chpuri2}複雑そうに聞こえますが[、]{.chpule2}[
]{.chpuri2}実はそれほどではありません[。]{.chpule2}[
]{.chpuri2}実際には[、]{.chpule2}[
]{.chpuri2}この仕組みは[[、]{.ls-1}]{.chpule2}[ ]{.chpuri2}​[
]{.chpule1}[（]{.chpuri1}１[）]{.chpule2}[
]{.chpuri2}サブスクライバー・オブジェクトへの参照のリストを格納するための配列フィールドと[[、]{.ls-1}]{.chpule2}[
]{.chpuri2}​[ ]{.chpule1}[（]{.chpuri1}２[）]{.chpule2}[
]{.chpuri2}そのリストへのサブスクライバーの追加と削除をするためのいくつかの公開メソッドで構成されています[。]{.chpule2}[
]{.chpuri2}

<figure class="image">
<img
src="../../images/patterns/diagrams/observer/solution1-ja29af.png?id=53c12c06fc4b1ec7f53d7989063acaf5"
style="aspect-ratio: 2.61;"
srcset="/images/patterns/diagrams/observer/solution1-ja.png?id=53c12c06fc4b1ec7f53d7989063acaf5 470w,/images/patterns/diagrams/observer/solution1-ja-1.5x.png?id=6ce3d3f94cd9d30cb980364c4272b396 705w,/images/patterns/diagrams/observer/solution1-ja-2x.png?id=4c1a646982573133a7b9f895ef331205 940w"
sizes="(max-width: 720px) 100vw, 470px" loading="lazy" width="470"
alt="サブスクリプションの仕組み" />
<figcaption><p>通知申し込みの仕組みにより<span
class="chpule2">、</span><span class="chpuri2">
</span>個々のオブジェクトはイベントの通知を申し込める<span
class="chpule2">。</span><span class="chpuri2"> </span></p></figcaption>
</figure>

こうしておけば[、]{.chpule2}[
]{.chpuri2}重要なイベントがパブリッシャーに発生するたびに[、]{.chpule2}[
]{.chpuri2}そのイベントはサブスクライバーに渡され[、]{.chpule2}[
]{.chpuri2}そのオブジェクトの特定の通知メソッドを呼び出します[。]{.chpule2}[
]{.chpuri2}

実際のアプリには[、]{.chpule2}[
]{.chpuri2}同じパブリッシャー・クラスのイベントを追跡したい数十のサブスクライバー・クラスがあるかもしれません[。]{.chpule2}[
]{.chpuri2}パブリッシャーがそれらのクラスに密に結合されるのは避けなければなりません[。]{.chpule2}[
]{.chpuri2}また[、]{.chpule2}[
]{.chpuri2}パブリッシャークラスが第三者によって使用されることになっている場合は[、]{.chpule2}[
]{.chpuri2}事前にそのようなクラスについては知りようがありません[。]{.chpule2}[
]{.chpuri2}

そのため[、]{.chpule2}[
]{.chpuri2}すべてのサブスクライバーが同じインターフェースを実装し[、]{.chpule2}[
]{.chpuri2}パブリッシャーはサブスクライバーとそのインターフェースを介してのみ通信することが肝心です[。]{.chpule2}[
]{.chpuri2}このインターフェースは[、]{.chpule2}[
]{.chpuri2}パブリッシャーが通知と共に状況に関するデータを渡すために使用できる一連のパラメーターとともに[、]{.chpule2}[
]{.chpuri2}通知メソッドを宣言する必要があります[。]{.chpule2}[
]{.chpuri2}

<figure class="image">
<img
src="../../images/patterns/diagrams/observer/solution2-jad3a8.png?id=97fb5aaaa4fd2e951163125ccc70c95c"
style="aspect-ratio: 1.24;"
srcset="/images/patterns/diagrams/observer/solution2-ja.png?id=97fb5aaaa4fd2e951163125ccc70c95c 460w,/images/patterns/diagrams/observer/solution2-ja-1.5x.png?id=36fa66de4a22e13b299ca04c7dbd9d2e 690w,/images/patterns/diagrams/observer/solution2-ja-2x.png?id=7e5a534dfd0f9e2181735a74b912d026 920w"
sizes="(max-width: 720px) 100vw, 460px" loading="lazy" width="460"
alt="通知メソッド" />
<figcaption><p>パブリッシャーは<span class="chpule2">、</span><span
class="chpuri2">
</span>サブスクライバー・オブジェクトの特定の通知メソッドを呼び出して通知<span
class="chpule2">。</span><span class="chpuri2"> </span></p></figcaption>
</figure>

アプリに複数の異なる種類のパブリッシャーがあり[、]{.chpule2}[
]{.chpuri2}サブスクライバーにすべてのパブリッシャーと互換性を持たせたい場合は[、]{.chpule2}[
]{.chpuri2}さらに一歩進んで[、]{.chpule2}[
]{.chpuri2}すべてのパブリッシャーが同じインターフェースに従うようにさせます[。]{.chpule2}[
]{.chpuri2}このインターフェースには[、]{.chpule2}[
]{.chpuri2}少数のサブスクリプション用メソッドがあるだけで大丈夫です[。]{.chpule2}[
]{.chpuri2}このインターフェースにより[、]{.chpule2}[
]{.chpuri2}サブスクライバーは具体的なクラスと結合することなく複数のパブリッシャーの状態を観察できます[。]{.chpule2}[
]{.chpuri2}
:::

::: {.section .analogy}
##  現実世界でのたとえ {#analogy}

<figure class="image">
<img
src="../../images/patterns/content/observer/observer-comic-2-ja9687.png?id=2cb615053554d19105626059ee8a5a6d"
style="aspect-ratio: 2;"
srcset="/images/patterns/content/observer/observer-comic-2-ja.png?id=2cb615053554d19105626059ee8a5a6d 600w,/images/patterns/content/observer/observer-comic-2-ja-1.5x.png?id=10bc99a22a34077c3ebf626a61dd2ac6 900w,/images/patterns/content/observer/observer-comic-2-ja-2x.png?id=403268b74b83253bfec1bd8cf1454211 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="雑誌と新聞の購読" />
<figcaption><p>雑誌と新聞の購読<span class="chpule2">。</span><span
class="chpuri2"> </span></p></figcaption>
</figure>

新聞や雑誌を購読したら[、]{.chpule2}[
]{.chpuri2}次の号があるかどうかを確認するために店に行く必要はありません[。]{.chpule2}[
]{.chpuri2}代わりに[、]{.chpule2}[ ]{.chpuri2}出版社は出版直後に[
]{.chpule1}[（]{.chpuri1}時には事前に[）]{.chpule2}[
]{.chpuri2}郵便受けに直接新しい号を配達します[。]{.chpule2}[ ]{.chpuri2}

出版社は購読者のリストを維持しており[、]{.chpule2}[
]{.chpuri2}どの雑誌に興味があるのかを知っています[。]{.chpule2}[
]{.chpuri2}購読者は[、]{.chpule2}[
]{.chpuri2}出版社に新しい雑誌を送るのを停止してもらいたい時にはいつでもリストから抜けることができます[。]{.chpule2}[
]{.chpuri2}
:::

::::: {.section .structure-container}
##  構造 {#structure}

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/observer/structurea121.png?id=365b7e2b8fbecc8948f34b9f8f16f33c"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.97;"
srcset="/images/patterns/diagrams/observer/structure.png?id=365b7e2b8fbecc8948f34b9f8f16f33c 610w,/images/patterns/diagrams/observer/structure-1.5x.png?id=7f23a48c9a2d789844f912d3a0f289e6 915w,/images/patterns/diagrams/observer/structure-2x.png?id=228af9bded4d6ee6daf43a0e23cca9ff 1220w"
sizes="(max-width: 720px) 100vw, 610px" loading="lazy" width="610"
alt="Observer デザインパターンの構造" /><img
src="../../images/patterns/diagrams/observer/structure-indexed8afd.png?id=2ca2c123503ede860740af2a22bc4b4d"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.88;"
srcset="/images/patterns/diagrams/observer/structure-indexed.png?id=2ca2c123503ede860740af2a22bc4b4d 620w,/images/patterns/diagrams/observer/structure-indexed-1.5x.png?id=20b9e7765e83ca75514c202968d9e9fa 930w,/images/patterns/diagrams/observer/structure-indexed-2x.png?id=910eec855bc41f05199e510494078926 1240w"
sizes="(max-width: 720px) 100vw, 620px" loading="lazy" width="620"
alt="Observer デザインパターンの構造" />
</figure>
:::

1.  **パブリッシャー**[
    ]{.chpule1}[（]{.chpuri1}Publisher[）]{.chpule2}[
    ]{.chpuri2}は[、]{.chpule2}[
    ]{.chpuri2}他のオブジェクトが関心を持つイベントを発行します[。]{.chpule2}[
    ]{.chpuri2}パブリッシャーがその状態を変えた時や何らかの行為を行なった時に[、]{.chpule2}[
    ]{.chpuri2}このようなイベントが発生します[。]{.chpule2}[
    ]{.chpuri2}パブリッシャーには[、]{.chpule2}[
    ]{.chpuri2}サブスクリプションの仕組みがあり[、]{.chpule2}[
    ]{.chpuri2}新規サブスクライバーの参加や現サブスクラーバーの参加を取り消すことができます[。]{.chpule2}[
    ]{.chpuri2}

2.  新しいイベントが発生すると[、]{.chpule2}[
    ]{.chpuri2}パブリッシャーはサブスクリプション・リストにアクセスし[、]{.chpule2}[
    ]{.chpuri2}各サブスクライバー・オブジェクトのサブスクライバー・インターフェースで宣言された通知メソッドを呼び出します[。]{.chpule2}[
    ]{.chpuri2}

3.  **サブスクライバー**[
    ]{.chpule1}[（]{.chpuri1}Subscriber[）]{.chpule2}[
    ]{.chpuri2}インターフェースは通知用インターフェースを宣言します[。]{.chpule2}[
    ]{.chpuri2}ほとんどの場合[、]{.chpule2}[ ]{.chpuri2}`update`
    メソッド一つだけがあります[。]{.chpule2}[
    ]{.chpuri2}このメソッドには[、]{.chpule2}[
    ]{.chpuri2}更新時にパブリッシャーがイベントの詳細情報を渡せるようにいくつかパラメータを持たせることもできます[。]{.chpule2}[
    ]{.chpuri2}

4.  **具象サブスクライバー**[ ]{.chpule1}[（]{.chpuri1}Concrete
    Subscribers[）]{.chpule2}[ ]{.chpuri2}は[、]{.chpule2}[
    ]{.chpuri2}パブリッシャーが発行した通知に応じて何らかのことを行います[。]{.chpule2}[
    ]{.chpuri2}これらのクラスは[、]{.chpule2}[
    ]{.chpuri2}パブリッシャーが具体的なクラスに結合されずにすむように[、]{.chpule2}[
    ]{.chpuri2}すべて同じインターフェースを実装しなければなりません[。]{.chpule2}[
    ]{.chpuri2}

5.  通常[、]{.chpule2}[ ]{.chpuri2}サブスクライバーは[、]{.chpule2}[
    ]{.chpuri2}更新を正しく処理するために周辺情報を必要とします[。]{.chpule2}[
    ]{.chpuri2}このため[、]{.chpule2}[
    ]{.chpuri2}パブリッシャーは[、]{.chpule2}[
    ]{.chpuri2}通知メソッドの引数として周辺データを渡すことがよくあります[。]{.chpule2}[
    ]{.chpuri2}パブリッシャーは引数として自分自身を渡すことができ[、]{.chpule2}[
    ]{.chpuri2}サブスクライバーは必要なデータを直接取得することができます[。]{.chpule2}[
    ]{.chpuri2}

6.  **Client**[ ]{.chpule1}[（]{.chpuri1}クライアント[）]{.chpule2}[
    ]{.chpuri2}は[、]{.chpule2}[
    ]{.chpuri2}パブリッシャーとサブスクライバーのオブジェクトを別々に作成し[、]{.chpule2}[
    ]{.chpuri2}パブリッシャーの更新に対してサブスクライバーを登録します[。]{.chpule2}[
    ]{.chpuri2}
::::
:::::

::: {.section .pseudocode}
##  擬似コード {#pseudocode}

この例では[、]{.chpule2}[ ]{.chpuri2}**Observer**
パターンを適応して[、]{.chpule2}[
]{.chpuri2}テキスト・エディター・オブジェクトがその状態の変化について他のサービス・オブジェクトに通知します[。]{.chpule2}[
]{.chpuri2}

<figure class="image">
<img
src="../../images/patterns/diagrams/observer/example04a2.png?id=6d0603ab5a00e4463b81d9639cd746a2"
style="aspect-ratio: 1.19;"
srcset="/images/patterns/diagrams/observer/example.png?id=6d0603ab5a00e4463b81d9639cd746a2 510w,/images/patterns/diagrams/observer/example-1.5x.png?id=67adccd1030a38dd263bfa09867f8dbe 765w,/images/patterns/diagrams/observer/example-2x.png?id=e2838e1562325e485fc7c2828a8ca445 1020w"
sizes="(max-width: 720px) 100vw, 510px" loading="lazy" width="510"
alt="Observer パターン例の構造" />
<figcaption><p>オブジェクトに起きるイベントを他オブジェクトに通知<span
class="chpule2">。</span><span class="chpuri2"> </span></p></figcaption>
</figure>

サブスクライバーのリストは常に変化します[。]{.chpule2}[
]{.chpuri2}アプリの望ましい動作に従い[、]{.chpule2}[
]{.chpuri2}オブジェクトは[、]{.chpule2}[
]{.chpuri2}通知の開始と停止が動的に可能です[。]{.chpule2}[ ]{.chpuri2}

この実装では[、]{.chpule2}[
]{.chpuri2}エディター・クラスは[、]{.chpule2}[
]{.chpuri2}サブスクリプション・リストを自身で管理しません[。]{.chpule2}[
]{.chpuri2}その目的のために作られた特別なヘルパー・オブジェクトにその仕事を委任します[。]{.chpule2}[
]{.chpuri2}そのオブジェクトを改善して[、]{.chpule2}[
]{.chpuri2}中央イベント・ディスパッチャーとして機能するようにもできます[。]{.chpule2}[
]{.chpuri2}そうすると[、]{.chpule2}[
]{.chpuri2}任意のオブジェクトがパブリッシャーとして機能できます[。]{.chpule2}[
]{.chpuri2}

プログラムに新しいサブスクライバーを追加する時[、]{.chpule2}[
]{.chpuri2}パブリッシャーがサブスクラーバーと同一インターフェースを通してやりとりする限り[、]{.chpule2}[
]{.chpuri2}既存のパブリッシャー・クラスの変更は必要ありません[。]{.chpule2}[
]{.chpuri2}

<figure class="code">
<pre class="code" lang="pseudocode"><code>// 基底パブリッシャー・クラスには、サブスクリプション管理用のコードと通知
// メソッドが含まれている。
class EventManager is
    private field listeners: hash map of event types and listeners

    method subscribe(eventType, listener) is
        listeners.add(eventType, listener)

    method unsubscribe(eventType, listener) is
        listeners.remove(eventType, listener)

    method notify(eventType, data) is
        foreach (listener in listeners.of(eventType)) do
            listener.update(data)

// 具象パブリッシャーは、サブスクライバーの一部にとって興味深い本当のビジ
// ネス・ロジックを含む。このクラスは、基底クラスから派生することも可能だ
// が、現実の世界ではそれは常に可能ではない。具象パブリッシャーがすでにサ
// ブクラスである場合があるからだ。その場合は、ここで行っているように、合
// 成を使ってサブスクリプションのロジックにパッチをあてられる。
class Editor is
    public field events: EventManager
    private field file: File

    constructor Editor() is
        events = new EventManager()

    // ビジネス・ロジックのメソッドは、変更に関してサブスクラーバーに通知
    // 可能。
    method openFile(path) is
        this.file = new File(path)
        events.notify(&quot;open&quot;, file.name)

    method saveFile() is
        file.write()
        events.notify(&quot;save&quot;, file.name)

    // ……


// これがサブスクライバーのインターフェース。お使いのプログラミング言語が
// 関数型をサポートしているのなら、サブスクラーバー階層を全部、関数の組で
// 置き換え可能。
interface EventListener is
    method update(filename)

// 具象サブスクライバーは、帰属するパブリッシャーから発行された更新通知に
// 反応。
class LoggingListener implements EventListener is
    private field log: File
    private field message: string

    constructor LoggingListener(log_filename, message) is
        this.log = new File(log_filename)
        this.message = message

    method update(filename) is
        log.write(replace(&#39;%s&#39;,filename,message))

class EmailAlertsListener implements EventListener is
    private field email: string
    private field message: string

    constructor EmailAlertsListener(email, message) is
        this.email = email
        this.message = message

    method update(filename) is
        system.email(email, replace(&#39;%s&#39;,filename,message))


// アプリケーションは、実行時にパブリッシャーとサブスクライバーを設定でき
// る。
class Application is
    method config() is
        editor = new Editor()

        logger = new LoggingListener(
            &quot;/path/to/log.txt&quot;,
            &quot;Someone has opened the file: %s&quot;)
        editor.events.subscribe(&quot;open&quot;, logger)

        emailAlerts = new EmailAlertsListener(
            &quot;admin@example.com&quot;,
            &quot;Someone has changed the file: %s&quot;)
        editor.events.subscribe(&quot;save&quot;, emailAlerts)</code></pre>
</figure>
:::

:::::::: {.section .applicability-container}
##  適応性 {#applicability}

::::::: applicability
::: applicability-problem
オブジェクトの状態の変更が他のオブジェクトの変更を必要とし[、]{.chpule2}[
]{.chpuri2}実際に影響を受けるオブジェクトの集りを事前に知ることができないか[、]{.chpule2}[
]{.chpuri2}影響を受けるオブジェクトが動的に変化する場合に[、]{.chpule2}[
]{.chpuri2}Observer パターンを使用します[。]{.chpule2}[ ]{.chpuri2}
:::

::: applicability-solution
グラフィカル・ユーザー・インターフェースのクラスを扱う場合[、]{.chpule2}[
]{.chpuri2}このような問題によく遭遇します[。]{.chpule2}[
]{.chpuri2}たとえば[、]{.chpule2}[
]{.chpuri2}特化したボタンのクラスを作成したとすると[、]{.chpule2}[
]{.chpuri2}ユーザーがボタンを押すたびに起動する特別なコードをクライアントが差し込めるようにしたい場合です[。]{.chpule2}[
]{.chpuri2}

Observer パターンでは[、]{.chpule2}[
]{.chpuri2}サブスクリプション・インターフェースを実装したどんなオブジェクトでもパブリッシャー・オブジェクトのイベント通知を申し込めます[。]{.chpule2}[
]{.chpuri2}開発したボタンに通知申し込みの仕組みを加えることで[、]{.chpule2}[
]{.chpuri2}サブスクライバー・クラスを通して[、]{.chpule2}[
]{.chpuri2}特別なコードをクライアントが差し込めるようにできます[。]{.chpule2}[
]{.chpuri2}
:::

::: applicability-problem
アプリ内の一部のオブジェクトが他のオブジェクトを限定された期間だけ[、]{.chpule2}[
]{.chpuri2}あるいは特別な場合にだけ監視しなければならない時[、]{.chpule2}[
]{.chpuri2}このパターンを使用してください[。]{.chpule2}[ ]{.chpuri2}
:::

::: applicability-solution
サブスクリプション・リストは動的で[、]{.chpule2}[
]{.chpuri2}サブスクライバーは必要な時にいつでも参加または退去できます[。]{.chpule2}[
]{.chpuri2}
:::
:::::::
::::::::

::: {.section .checklist}
##  実装方法 {#checklist}

1.  ビジネス・ロジックを眺めて[、]{.chpule2}[
    ]{.chpuri2}これを二つの部分に分けます[。]{.chpule2}[
    ]{.chpuri2}他のコードから独立した中核機能はパブリッシャーとして機能し[、]{.chpule2}[
    ]{.chpuri2}残りはサブスクライバー・クラスの集合となります[。]{.chpule2}[
    ]{.chpuri2}

2.  サブスクライバー・インターフェースを宣言します[。]{.chpule2}[
    ]{.chpuri2}最低でも[、]{.chpule2}[ ]{.chpuri2}`update`
    メソッドを一つ宣言します[。]{.chpule2}[ ]{.chpuri2}

3.  パブリッシャー・インターフェースを宣言し[、]{.chpule2}[
    ]{.chpuri2}サブスクライバー・オブジェクトをリストに追加と削除を行うメソッドのペアを書き入れます[。]{.chpule2}[
    ]{.chpuri2}パブリッシャーはサブスクライバーとサブスクライバー・インターフェースを通してのみやりとりすることをお忘れなく[。]{.chpule2}[
    ]{.chpuri2}

4.  どこに実際のサブスクリプション・リストを置くかを決め[、]{.chpule2}[
    ]{.chpuri2}サブスクリプション用メソッドを実装します[。]{.chpule2}[
    ]{.chpuri2}このコードは[、]{.chpule2}[
    ]{.chpuri2}通常すべてのパブリッシャー間で同じなので[、]{.chpule2}[
    ]{.chpuri2}これを置くのに自明な場所は[、]{.chpule2}[
    ]{.chpuri2}パブリッシャー・インターフェースから直接派生した抽象クラスです[。]{.chpule2}[
    ]{.chpuri2}具象クラスはそのクラスを拡張し[、]{.chpule2}[
    ]{.chpuri2}サブスクリプションの振る舞いを継承します[。]{.chpule2}[
    ]{.chpuri2}

    しかしながら[、]{.chpule2}[
    ]{.chpuri2}既存のクラス階層にこのパターンを適応する場合は[、]{.chpule2}[
    ]{.chpuri2}合成に基づく方法を検討してください[。]{.chpule2}[
    ]{.chpuri2}つまり[、]{.chpule2}[
    ]{.chpuri2}サブスクリプションのロジックを個別のオブジェクトに置き[、]{.chpule2}[
    ]{.chpuri2}実際の全パブリッシャーがそれを使うようにします[。]{.chpule2}[
    ]{.chpuri2}

5.  パブリッシャーの具象クラスを作成します[。]{.chpule2}[
    ]{.chpuri2}何か重要なことがパブリッシャーの中で起きるたびに[、]{.chpule2}[
    ]{.chpuri2}サブスクライバー全部に通知しなければいけません[。]{.chpule2}[
    ]{.chpuri2}

6.  具象サブスクライバー・クラスの中に更新通知メソッドを実装します[。]{.chpule2}[
    ]{.chpuri2}大多数のサブスクライバーは[、]{.chpule2}[
    ]{.chpuri2}イベントについて何らかの周辺情報を必要とします[。]{.chpule2}[
    ]{.chpuri2}それは[、]{.chpule2}[
    ]{.chpuri2}通知メソッドの引数として渡すことができます[。]{.chpule2}[
    ]{.chpuri2}

    しかし[、]{.chpule2}[
    ]{.chpuri2}もう一つのやり方があります[。]{.chpule2}[
    ]{.chpuri2}通知を受けたらすぐ[、]{.chpule2}[
    ]{.chpuri2}サブスクライバーはそのデータを通知から直接取り込むことができます[。]{.chpule2}[
    ]{.chpuri2}この場合[、]{.chpule2}[
    ]{.chpuri2}パブリッシャーはそれ自身を更新メソッドを通して渡します[。]{.chpule2}[
    ]{.chpuri2}少し柔軟性に欠けますが[、]{.chpule2}[
    ]{.chpuri2}コンストラクターを通して[、]{.chpule2}[
    ]{.chpuri2}パブリッシャーをサブスクライバーと永久にリンクすることもできます[。]{.chpule2}[
    ]{.chpuri2}

7.  クライアントは[、]{.chpule2}[
    ]{.chpuri2}必要なすべてのサブスクライバーを作成し[、]{.chpule2}[
    ]{.chpuri2}それらを適切なパブリッシャーに登録する必要があります[。]{.chpule2}[
    ]{.chpuri2}
:::

:::::: {.section .pros-cons}
##  長所と短所 {#pros-cons}

::::: row
::: col-sm-6
-   
    *[開]{.cjk}[放]{.cjk}[閉]{.cjk}[鎖]{.cjk}[の]{.cjk}[原]{.cjk}[則]{.cjk}*[。]{.chpule2}[
    ]{.chpuri2}パブリッシャーのコードを変更することなく[、]{.chpule2}[
    ]{.chpuri2}新しいサブスクライバー・クラスを導入可能[[。]{.ls-1}]{.chpule2}[
    ]{.chpuri2}​[
    ]{.chpule1}[（]{.chpuri1}パブリッシャー・インターフェースがある場合は[、]{.chpule2}[
    ]{.chpuri2}逆も可[。]{.ls-05}[）]{.chpule2}[ ]{.chpuri2}
-    実行時にオブジェクト間の関係を確立できます[。]{.chpule2}[
    ]{.chpuri2}
:::

::: col-sm-6
-    サブスクライバーへの通知の順番はランダムです[。]{.chpule2}[
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

[![Observer を C#
で](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](observer/csharp/example.html "Observer を C# で"){.prog-lang-link}
[![Observer を C++
で](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](observer/cpp/example.html "Observer を C++ で"){.prog-lang-link}
[![Observer を Go
で](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](observer/go/example.html "Observer を Go で"){.prog-lang-link}
[![Observer を Java
で](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](observer/java/example.html "Observer を Java で"){.prog-lang-link}
[![Observer を PHP
で](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](observer/php/example.html "Observer を PHP で"){.prog-lang-link}
[![Observer を Python
で](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](observer/python/example.html "Observer を Python で"){.prog-lang-link}
[![Observer を Ruby
で](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](observer/ruby/example.html "Observer を Ruby で"){.prog-lang-link}
[![Observer を Rust
で](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](observer/rust/example.html "Observer を Rust で"){.prog-lang-link}
[![Observer を Swift
で](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](observer/swift/example.html "Observer を Swift で"){.prog-lang-link}
[![Observer を TypeScript
で](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](observer/typescript/example.html "Observer を TypeScript で"){.prog-lang-link}
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

[State []{.fa .fa-arrow-right}](state.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### 戻る

[[]{.fa .fa-arrow-left} Memento](memento.html){.btn .btn-default
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
