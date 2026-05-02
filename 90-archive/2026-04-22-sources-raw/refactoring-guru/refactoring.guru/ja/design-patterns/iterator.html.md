::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} /
[デザインパターン](../design-patterns.html){.type} /
[振る舞いに関するパターン](behavioral-patterns.html){.type}
:::

# Iterator {#iterator .title}

::: aka
別名：[イテレーター]{style="display:inline-block"}
:::

::: {.section .intent}
##  一言でいうと {#intent}

**Iterator**[ ]{.chpule1}[（]{.chpuri1}イテレーター[、]{.chpule2}[
]{.chpuri2}反復子[）]{.chpule2}[ ]{.chpuri2}は[、]{.chpule2}[
]{.chpuri2}振る舞いに関するデザインパターンの一つで[、]{.chpule2}[
]{.chpuri2}リスト[、]{.chpule2}[ ]{.chpuri2}スタック[、]{.chpule2}[
]{.chpuri2}ツリーなどの実際のデータ表現を表に出さずにコレクションの要素を探索することができます[。]{.chpule2}[
]{.chpuri2}

<figure class="image">
<img
src="../../images/patterns/content/iterator/iterator-jaf037.png?id=a29a593168960cbd5a2788d2ff4ecd03"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/iterator/iterator-ja.png?id=a29a593168960cbd5a2788d2ff4ecd03 640w,/images/patterns/content/iterator/iterator-ja-1.5x.png?id=920f407d6adb70445f6c1907ab3a5c41 960w,/images/patterns/content/iterator/iterator-ja-2x.png?id=6647db43faac07745e9fe860e0c0b2a7 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Iterator デザインパターン" />
</figure>
:::

::: {.section .problem}
##  問題 {#problem}

コレクションは[、]{.chpule2}[
]{.chpuri2}プログラミングで最も使われるデータ型の一つです[。]{.chpule2}[
]{.chpuri2}とは言っても[、]{.chpule2}[
]{.chpuri2}コレクションは[、]{.chpule2}[
]{.chpuri2}オブジェクトのグループを含むコンテナにすぎません[。]{.chpule2}[
]{.chpuri2}

<figure class="image">
<img
src="../../images/patterns/diagrams/iterator/problem14cac.png?id=52ef2fe2d4920e3fed696c221fe757f2"
style="aspect-ratio: 4.9;"
srcset="/images/patterns/diagrams/iterator/problem1.png?id=52ef2fe2d4920e3fed696c221fe757f2 490w,/images/patterns/diagrams/iterator/problem1-1.5x.png?id=b46e39ade7de292fdcacc9790066c72f 735w,/images/patterns/diagrams/iterator/problem1-2x.png?id=1f4384c3c631be238bfc23d6eb84a0ef 980w"
sizes="(max-width: 720px) 100vw, 490px" loading="lazy" width="490"
alt="いろいろな種類のコレクション" />
<figcaption
aria-hidden="true"><p>いろいろな種類のコレクション</p></figcaption>
</figure>

ほとんどのクレクションは[、]{.chpule2}[
]{.chpuri2}要素を単純なリストにしまいます[。]{.chpule2}[
]{.chpuri2}しかし[、]{.chpule2}[ ]{.chpuri2}いくつかは[、]{.chpule2}[
]{.chpuri2}スタック[、]{.chpule2}[ ]{.chpuri2}ツリー[、]{.chpule2}[
]{.chpuri2}グラフその他の複雑なデータ構造に準拠しています[。]{.chpule2}[
]{.chpuri2}

コレクションの構造にかかわらず[、]{.chpule2}[
]{.chpuri2}他のコードがコレクションの要素を使えるように[、]{.chpule2}[
]{.chpuri2}要素にアクセスする何らかの方法が必要となります[。]{.chpule2}[
]{.chpuri2}同じ要素に何回もアクセスすることなく[、]{.chpule2}[
]{.chpuri2}順番に見ていく方法が必要です[。]{.chpule2}[ ]{.chpuri2}

リストに基づくコレクションであれば[、]{.chpule2}[
]{.chpuri2}これは簡単な仕事のように聞こえるかもしれません[。]{.chpule2}[
]{.chpuri2}全部の要素に対してループを回せばすむ話です[。]{.chpule2}[
]{.chpuri2}でも[、]{.chpule2}[
]{.chpuri2}ツリーのような複雑なデータ構造の場合はどうしますか[？]{.chpule2}[
]{.chpuri2}たとえば[、]{.chpule2}[
]{.chpuri2}あるツリーに対し[、]{.chpule2}[
]{.chpuri2}深さ優先探索で今日は十分だとします[。]{.chpule2}[
]{.chpuri2}翌日になると[、]{.chpule2}[
]{.chpuri2}幅優先探索が必要となるかもしれません[。]{.chpule2}[
]{.chpuri2}そして翌週には[、]{.chpule2}[
]{.chpuri2}ツリーの任意の要素への直接アクセスのような違う要件が出てくるかもしれません[。]{.chpule2}[
]{.chpuri2}

<figure class="image">
<img
src="../../images/patterns/diagrams/iterator/problem29f71.png?id=f9c1a746c787320291c85fdc2a314192"
style="aspect-ratio: 3.75;"
srcset="/images/patterns/diagrams/iterator/problem2.png?id=f9c1a746c787320291c85fdc2a314192 600w,/images/patterns/diagrams/iterator/problem2-1.5x.png?id=7a003d020e789773e0c833d7b1df00e6 900w,/images/patterns/diagrams/iterator/problem2-2x.png?id=656b13c109d4d4960ea1f9fa993bae4c 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="様々な探索アルゴリズム" />
<figcaption><p>同じコレクションでも<span class="chpule2">、</span><span
class="chpuri2"> </span>違った方法で探索可能<span
class="chpule2">。</span><span class="chpuri2"> </span></p></figcaption>
</figure>

コレクションにいくつもの探索アルゴリズムを追加していくと[、]{.chpule2}[
]{.chpuri2}効率的なデータの貯蔵という本来の目的が次第にぼやけてきます[。]{.chpule2}[
]{.chpuri2}また[、]{.chpule2}[
]{.chpuri2}ある種のアルゴリズムは[、]{.chpule2}[
]{.chpuri2}特定の用途向けにできているため[、]{.chpule2}[
]{.chpuri2}そのようなアルゴリズムを一般的なコレクションに含めるというのは[、]{.chpule2}[
]{.chpuri2}ちょっと変な話です[。]{.chpule2}[ ]{.chpuri2}

一方[、]{.chpule2}[
]{.chpuri2}多くのコレクションと共に動作することになっているクライアント・コードにとっては[、]{.chpule2}[
]{.chpuri2}要素がどう格納されているかということは[、]{.chpule2}[
]{.chpuri2}どうでもいい話です[。]{.chpule2}[
]{.chpuri2}しかし[、]{.chpule2}[
]{.chpuri2}コレクションによって要素へのアクセス方法が異なるため[、]{.chpule2}[
]{.chpuri2}特定のコレクションのクラスに結合したコードを書くしかありません[。]{.chpule2}[
]{.chpuri2}
:::

::: {.section .solution}
##  解決策 {#solution}

Iterator パターンの基本的な考え方は[、]{.chpule2}[
]{.chpuri2}コレクションに対する探索の振る舞いを*[イ]{.cjk}[テ]{.cjk}[レ]{.cjk}[ー]{.cjk}[タ]{.cjk}[ー]{.cjk}*[
]{.chpule1}[（]{.chpuri1}反復子とも[）]{.chpule2}[
]{.chpuri2}と呼ばれるオブジェクトへ抽出することです[。]{.chpule2}[
]{.chpuri2}

<figure class="image">
<img
src="../../images/patterns/diagrams/iterator/solution19739.png?id=2f5fbcce6099d8ea09b2fbb83e3e7059"
style="aspect-ratio: 0.85;"
srcset="/images/patterns/diagrams/iterator/solution1.png?id=2f5fbcce6099d8ea09b2fbb83e3e7059 400w,/images/patterns/diagrams/iterator/solution1-1.5x.png?id=68612372ede6e5029403d38b381fdc05 600w,/images/patterns/diagrams/iterator/solution1-2x.png?id=dcebcad4c197bfa5f25f680382d0e5ac 800w"
sizes="(max-width: 720px) 100vw, 400px" loading="lazy" width="400"
alt="イテレーターは様々な探索アルゴリズムを実装" />
<figcaption><p>イテレーターは様々な探索アルゴリズムを実装する<span
class="chpule2">。</span><span class="chpuri2">
</span>複数のイテレーター・オブジェクトは<span
class="chpule2">、</span><span class="chpuri2">
</span>同じコレクションを同時に探索可能<span
class="chpule2">。</span><span class="chpuri2"> </span></p></figcaption>
</figure>

アルゴリズム自体を実装することに加えて[、]{.chpule2}[
]{.chpuri2}イテレーター・オブジェクトは[、]{.chpule2}[
]{.chpuri2}現在位置[、]{.chpule2}[
]{.chpuri2}残数など詳細を一箇所にまとめます[。]{.chpule2}[
]{.chpuri2}このため[、]{.chpule2}[
]{.chpuri2}同一コレクション上を複数のイテレーターが[、]{.chpule2}[
]{.chpuri2}同時に[、]{.chpule2}[
]{.chpuri2}お互いに独立して[、]{.chpule2}[
]{.chpuri2}作業することができます[。]{.chpule2}[ ]{.chpuri2}

通常[、]{.chpule2}[
]{.chpuri2}イテレーターにはコレクションの要素を取得するための主要なメソッドが一つあります[。]{.chpule2}[
]{.chpuri2}クライアントは[、]{.chpule2}[
]{.chpuri2}このメソッドが何も返さなくなるまで[、]{.chpule2}[
]{.chpuri2}このメソッドを実行し続けることができます[。]{.chpule2}[
]{.chpuri2}何も返さないということは[、]{.chpule2}[
]{.chpuri2}イテレーターが全要素を探索し終えたということを意味します[。]{.chpule2}[
]{.chpuri2}

すべてのイテレーターは同じインターフェースを実装する必要があります[。]{.chpule2}[
]{.chpuri2}これにより[、]{.chpule2}[
]{.chpuri2}クライアント・コードは[、]{.chpule2}[
]{.chpuri2}イテレーターが適切である限り[、]{.chpule2}[
]{.chpuri2}いかなるコレクション型とも[、]{.chpule2}[
]{.chpuri2}つまりいかなる探索アルゴリズムとも[、]{.chpule2}[
]{.chpuri2}互換性を維持できます[。]{.chpule2}[
]{.chpuri2}コレクションを探索する特別な方法が必要な場合は[、]{.chpule2}[
]{.chpuri2}コレクションやクライアントには手を加えずに[、]{.chpule2}[
]{.chpuri2}新しいイテレーターのクラスを作成するだけですみます[。]{.chpule2}[
]{.chpuri2}
:::

::: {.section .analogy}
##  現実世界でのたとえ {#analogy}

<figure class="image">
<img
src="../../images/patterns/content/iterator/iterator-comic-1-jaa668.png?id=80e45de33270bdbe4058148b7f4aa689"
style="aspect-ratio: 2.13;"
srcset="/images/patterns/content/iterator/iterator-comic-1-ja.png?id=80e45de33270bdbe4058148b7f4aa689 640w,/images/patterns/content/iterator/iterator-comic-1-ja-1.5x.png?id=7e021159ff2778adb4ea1350f98cd3e6 960w,/images/patterns/content/iterator/iterator-comic-1-ja-2x.png?id=ab65c89d790ab176804ffaea3520e288 1280w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="640"
alt="ローマをうろうろする様々な方法" />
<figcaption><p>ローマをうろうろする様々な方法<span
class="chpule2">。</span><span class="chpuri2"> </span></p></figcaption>
</figure>

あなたは[、]{.chpule2}[ ]{.chpuri2}数日間ローマを訪問し[、]{.chpule2}[
]{.chpuri2}その主な観光スポットすべてを訪問する予定です[。]{.chpule2}[
]{.chpuri2}しかし[、]{.chpule2}[
]{.chpuri2}到着してからぐるぐると円を描いて歩き回ると[、]{.chpule2}[
]{.chpuri2}多くの時間を無駄にしてしまい[、]{.chpule2}[
]{.chpuri2}コロッセオすら見つけられないかもしれません[。]{.chpule2}[
]{.chpuri2}

一方[、]{.chpule2}[
]{.chpuri2}ナビゲーションのためにスマートフォンの仮想ガイド・アプリを購入することもできます[。]{.chpule2}[
]{.chpuri2}それは賢くてしかも安く[、]{.chpule2}[
]{.chpuri2}自分の興味のある場所に好きなだけ長時間留まることを許してくれます[。]{.chpule2}[
]{.chpuri2}

第三の選択肢としては[、]{.chpule2}[
]{.chpuri2}旅行予算のいくらかを使って[、]{.chpule2}[
]{.chpuri2}自宅の裏庭のように市内を熟知している地元のガイドを雇うことです[。]{.chpule2}[
]{.chpuri2}ガイドは[、]{.chpule2}[
]{.chpuri2}あなたの好みに合わせてツアーを調整して名所を案内し[、]{.chpule2}[
]{.chpuri2}歴史や逸話を話してくれます[。]{.chpule2}[
]{.chpuri2}旅をもっと楽しくしてくれますが[、]{.chpule2}[
]{.chpuri2}残念ながら値段が張ります[。]{.chpule2}[ ]{.chpuri2}

これらのオプション[、]{.chpule2}[ ]{.chpuri2}つまり[、]{.chpule2}[
]{.chpuri2}カンを頼りに適当な方向へ行く[、]{.chpule2}[
]{.chpuri2}スマホのアプリ[、]{.chpule2}[
]{.chpuri2}人間のガイド[、]{.chpule2}[ ]{.chpuri2}は[、]{.chpule2}[
]{.chpuri2}ローマにある数多い観光名所のコレクションに対するイタレーターと言えます[。]{.chpule2}[
]{.chpuri2}
:::

::::: {.section .structure-container}
##  構造 {#structure}

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/iterator/structure5247.png?id=35ea851f8f6bbe51d79eb91e6e6519d0"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.12;"
srcset="/images/patterns/diagrams/iterator/structure.png?id=35ea851f8f6bbe51d79eb91e6e6519d0 480w,/images/patterns/diagrams/iterator/structure-1.5x.png?id=19d4c2130ab2e3bd718f87e956fcd873 720w,/images/patterns/diagrams/iterator/structure-2x.png?id=b7b4708d3f279dd046eb1692f1cba710 960w"
sizes="(max-width: 720px) 100vw, 480px" loading="lazy" width="480"
alt="Iterator デザインパターンの構造" /><img
src="../../images/patterns/diagrams/iterator/structure-indexed3472.png?id=7bc28907ff6b480db6635a93ebaa10ff"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.33;"
srcset="/images/patterns/diagrams/iterator/structure-indexed.png?id=7bc28907ff6b480db6635a93ebaa10ff 520w,/images/patterns/diagrams/iterator/structure-indexed-1.5x.png?id=32fde4c4c1cbfbe07aa6e716e5ac2346 780w,/images/patterns/diagrams/iterator/structure-indexed-2x.png?id=d809428b2262e2013972fe69d2434473 1040w"
sizes="(max-width: 720px) 100vw, 520px" loading="lazy" width="520"
alt="Iterator デザインパターンの構造" />
</figure>
:::

1.  **イテレーター**[ ]{.chpule1}[（]{.chpuri1}Iterator[）]{.chpule2}[
    ]{.chpuri2}インターフェースは[、]{.chpule2}[
    ]{.chpuri2}コレクションを探索するために必要な操作[、]{.chpule2}[
    ]{.chpuri2}つまり[、]{.chpule2}[
    ]{.chpuri2}次要素の獲得[、]{.chpule2}[
    ]{.chpuri2}現在位置の取得[、]{.chpule2}[
    ]{.chpuri2}探索のリセットなどを宣言します[。]{.chpule2}[ ]{.chpuri2}

2.  **具象イテレーター**[ ]{.chpule1}[（]{.chpuri1}Concrete
    Iterator[）]{.chpule2}[ ]{.chpuri2}は[、]{.chpule2}[
    ]{.chpuri2}コレクションを探索するための特定のアルゴリズムを実装します[。]{.chpule2}[
    ]{.chpuri2}イテレーター・オブジェクトは[、]{.chpule2}[
    ]{.chpuri2}探索の進行状況を独自に把握する必要があります[。]{.chpule2}[
    ]{.chpuri2}これにより[、]{.chpule2}[
    ]{.chpuri2}複数のイテレーターが同じコレクションを互いに独立して探索することができます[。]{.chpule2}[
    ]{.chpuri2}

3.  **コレクション**[ ]{.chpule1}[（]{.chpuri1}Iterable
    Collection[）]{.chpule2}[
    ]{.chpuri2}インターフェースは[、]{.chpule2}[
    ]{.chpuri2}コレクションと互換なイテレーターを取得するための一つまたは複数のメソッドを宣言します[。]{.chpule2}[
    ]{.chpuri2}具象コレクションが様々な種類のイテレーターを返せられるように[、]{.chpule2}[
    ]{.chpuri2}メソッドの戻り値型はイテレーター・インターフェースとして宣言されなければいけないことに注意してください[。]{.chpule2}[
    ]{.chpuri2}

4.  **具象コレクション**[ ]{.chpule1}[（]{.chpuri1}Concrete
    Collection[）]{.chpule2}[ ]{.chpuri2}は[、]{.chpule2}[
    ]{.chpuri2}クライアントから要求があるたびに[、]{.chpule2}[
    ]{.chpuri2}特定の具象イテレーター・クラスの新インスタンスを返します[。]{.chpule2}[
    ]{.chpuri2}コレクションの残りのコードはどこにあるのだろう[、]{.chpule2}[
    ]{.chpuri2}と思ってませんか[？]{.chpule2}[
    ]{.chpuri2}大丈夫[、]{.chpule2}[
    ]{.chpuri2}同じクラスにあるはずです[。]{.chpule2}[
    ]{.chpuri2}このような細いことは実際のパターンにとっては重要ではないので[、]{.chpule2}[
    ]{.chpuri2}省略しているだけです[。]{.chpule2}[ ]{.chpuri2}

5.  **クライアント**[ ]{.chpule1}[（]{.chpuri1}Client[）]{.chpule2}[
    ]{.chpuri2}は[、]{.chpule2}[
    ]{.chpuri2}コレクションとイテレーターの両方と[、]{.chpule2}[
    ]{.chpuri2}それぞれのインターフェースを介してやりとりをします[。]{.chpule2}[
    ]{.chpuri2}これにより[、]{.chpule2}[
    ]{.chpuri2}クライアントは具体的なクラスに結合されることなく[、]{.chpule2}[
    ]{.chpuri2}同じクライアント・コードで様々なコレクションやイテレーターを使用することができます[。]{.chpule2}[
    ]{.chpuri2}

    多くの場合[、]{.chpule2}[
    ]{.chpuri2}クライアントは自分でイテレーターを作成することはせず[、]{.chpule2}[
    ]{.chpuri2}コレクションから取得します[。]{.chpule2}[
    ]{.chpuri2}しかし特別な場合[、]{.chpule2}[
    ]{.chpuri2}クライアントはイテレーターを直接作成できます[。]{.chpule2}[
    ]{.chpuri2}クライアントが専用の特殊イテレーターを定義した場合などです[。]{.chpule2}[
    ]{.chpuri2}
::::
:::::

::: {.section .pseudocode}
##  擬似コード {#pseudocode}

この例では[、]{.chpule2}[ ]{.chpuri2}**Iterator**
パターンを使って[、]{.chpule2}[ ]{.chpuri2}Facebook
のソーシャル・グラフへのアクセスを内包した特殊なコレクションを探索します[。]{.chpule2}[
]{.chpuri2}コレクションは[、]{.chpule2}[
]{.chpuri2}様々な方法でプロフィールを探索するイテレーターをいくつか提供します[。]{.chpule2}[
]{.chpuri2}

<figure class="image">
<img
src="../../images/patterns/diagrams/iterator/examplee0a2.png?id=f2a24ef3787bf80ed450709240506ff2"
style="aspect-ratio: 0.95;"
srcset="/images/patterns/diagrams/iterator/example.png?id=f2a24ef3787bf80ed450709240506ff2 600w,/images/patterns/diagrams/iterator/example-1.5x.png?id=cab0e1459ffc3721579a3e7d6a4e9022 900w,/images/patterns/diagrams/iterator/example-2x.png?id=73c3e55f75ca0250bd84e8a1d8cc88d2 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Iterator パターン例の構造" />
<figcaption><p>ソーシャル・プロフィールの反復をする例<span
class="chpule2">。</span><span class="chpuri2"> </span></p></figcaption>
</figure>

`create­Friends­Iterator` の返す[
]{.chpule1}[「]{.chpuri1}友達[」]{.chpule2}[
]{.chpuri2}イテレーター[、]{.chpule2}[ ]{.chpuri2}は[、]{.chpule2}[
]{.chpuri2}特定のプロフィールにある友達を渡り歩くために使用できます[。]{.chpule2}[
]{.chpuri2}`create­Coworker­Iterator` の返す[
]{.chpule1}[「]{.chpuri1}同僚[」]{.chpule2}[
]{.chpuri2}のイテレーターは[、]{.chpule2}[
]{.chpuri2}同じ会社で働いていない友人は飛ばすことを除いて[、]{.chpule2}[
]{.chpuri2}同じことをします[。]{.chpule2}[
]{.chpuri2}両方のイテレーターは共通インターフェースを実装しており[、]{.chpule2}[
]{.chpuri2}認証や REST
リクエストの送信などの実装状の詳細にかかわらず[、]{.chpule2}[
]{.chpuri2}クライアントはプロフィールを獲得できます[。]{.chpule2}[
]{.chpuri2}

クライアント・コードは[、]{.chpule2}[
]{.chpuri2}コレクションとイテレーターとインターフェースを介してのみやりとりするため[、]{.chpule2}[
]{.chpuri2}具体的なクラスに結合されていません[。]{.chpule2}[
]{.chpuri2}アプリを新しいソーシャル・ネットワークに接続すると決めたら[、]{.chpule2}[
]{.chpuri2}既存のコードを変更する必要はなく[、]{.chpule2}[
]{.chpuri2}新しいコレクションとイテレーター・クラスを作成するだけですみます[。]{.chpule2}[
]{.chpuri2}

<figure class="code">
<pre class="code" lang="pseudocode"><code>// コレクション・インターフェースは、イテレーターを生成するためのファクト
// リー・メソッドを宣言しなければならい。プログラムに異なる種類の反復があ
// る場合は、複数のメソッドを宣言可能。
interface SocialNetwork is
    method createFriendsIterator(profileId):ProfileIterator
    method createCoworkersIterator(profileId):ProfileIterator


// 具象コレクションは、それが返す具象イテレーター・クラスと結合される。し
// かし、これらのメソッドのシグネチャーがイテレーター・インターフェースを
// 返すようになっているため、クライアントは具象イテレーターとは結合されな
// い。
class Facebook implements SocialNetwork is
    // …… コレクションのコードの大部分はここに ……

    // イテレーター作成コード。
    method createFriendsIterator(profileId) is
        return new FacebookIterator(this, profileId, &quot;friends&quot;)
    method createCoworkersIterator(profileId) is
        return new FacebookIterator(this, profileId, &quot;coworkers&quot;)


// すべてのイテレーターに共通のインターフェース。
interface ProfileIterator is
    method getNext():Profile
    method hasMore():bool


// 具象イタレーター・クラス。
class FacebookIterator implements ProfileIterator is
    // イテレーターは、探索対象のコレクションへの参照を必要とする。
    private field facebook: Facebook
    private field profileId, type: string

    // イテレーター・オブジェクトは、他のイテレーターとは独立してコレク
    // ションを探索する。そのため、探索の状態を保存する必要あり。
    private field currentPosition
    private field cache: array of Profile

    constructor FacebookIterator(facebook, profileId, type) is
        this.facebook = facebook
        this.profileId = profileId
        this.type = type

    private method lazyInit() is
        if (cache == null)
            cache = facebook.socialGraphRequest(profileId, type)

    // 各具象イテレーターのクラスは、共通イテレーター・インターフェースの
    // 独自の実装を行う。
    method getNext() is
        if (hasMore())
            result = cache[currentPosition]
            currentPosition++
            return result

    method hasMore() is
        lazyInit()
        return currentPosition &lt; cache.length


// もうひとつ役に立つトリック：クライアント・クラスにコレクション全体への
// アクセスを与える代わりに、イテレーターを渡すことができる。こうすると、
// コレクションそのものをクライアントに露出する必要がなくなる。
//
// さらなる利点：実行時に異なるイテレーターを渡すことで、クライアントのコ
// レクションに対する動作を変更できる。これは、クライアント・コードが具象
// イテレーター・クラスと結合されていないために可能。
class SocialSpammer is
    method send(iterator: ProfileIterator, message: string) is
        while (iterator.hasMore())
            profile = iterator.getNext()
            System.sendEmail(profile.getEmail(), message)


// アプリケーションのクラスはコレクションとイテレーターを設定し、それらを
// クライアント・コードに渡す。
class Application is
    field network: SocialNetwork
    field spammer: SocialSpammer

    method config() is
        if working with Facebook
            this.network = new Facebook()
        if working with LinkedIn
            this.network = new LinkedIn()
        this.spammer = new SocialSpammer()

    method sendSpamToFriends(profile) is
        iterator = network.createFriendsIterator(profile.getId())
        spammer.send(iterator, &quot;Very important message&quot;)

    method sendSpamToCoworkers(profile) is
        iterator = network.createCoworkersIterator(profile.getId())
        spammer.send(iterator, &quot;Very important message&quot;)</code></pre>
</figure>
:::

:::::::::: {.section .applicability-container}
##  適応性 {#applicability}

::::::::: applicability
::: applicability-problem
コレクションが複雑なデータ構造をしており[、]{.chpule2}[
]{.chpuri2}その複雑さを[
]{.chpule1}[（]{.chpuri1}利便性やセキュリティー上の理由から[）]{.chpule2}[
]{.chpuri2}クライアントから隠したい場合[、]{.chpule2}[
]{.chpuri2}Iterator パターンを使用します[。]{.chpule2}[ ]{.chpuri2}
:::

::: applicability-solution
イテレーターは[、]{.chpule2}[
]{.chpuri2}複雑なデータ構造に対する作業の詳細を包み隠し[、]{.chpule2}[
]{.chpuri2}クライアントがいくつかの簡単なメソッドでコレクションの要素にアクセスできるようにします[。]{.chpule2}[
]{.chpuri2}このやり方はクライアントに対して便利なだけでなく[、]{.chpule2}[
]{.chpuri2}クライアントがコレクションと直接やりとりできたとしたら起きるかもしれない不注意[、]{.chpule2}[
]{.chpuri2}あるいは悪意による行為からコレクションを保護します[。]{.chpule2}[
]{.chpuri2}
:::

::: applicability-problem
アプリ全体の探索用コードの重複を減らすために[、]{.chpule2}[
]{.chpuri2}このパターンを使用します[。]{.chpule2}[ ]{.chpuri2}
:::

::: applicability-solution
自明ではない反復アルゴリズムのコードは非常にかさばる傾向があります[。]{.chpule2}[
]{.chpuri2}アプリのビジネス・ロジック内に配置すると[、]{.chpule2}[
]{.chpuri2}元のコードの責任を曖昧にし[、]{.chpule2}[
]{.chpuri2}保守性を低下させる可能性があります[。]{.chpule2}[
]{.chpuri2}探索コードを指定されたイテレーターに移動することにより[、]{.chpule2}[
]{.chpuri2}アプリケーションのコードがより無駄がなくクリーンになります[。]{.chpule2}[
]{.chpuri2}
:::

::: applicability-problem
コードが異なるデータ構造を探索できるようにしたい場合や[、]{.chpule2}[
]{.chpuri2}これらの構造体の型が事前にわからない場合に[、]{.chpule2}[
]{.chpuri2}Iterator を使用します[。]{.chpule2}[ ]{.chpuri2}
:::

::: applicability-solution
このパターンはコレクションとイテレーターの両方に対していくつかの汎用インターフェースを提供します[。]{.chpule2}[
]{.chpuri2}コードがこれらのインターフェースを使用していれば[、]{.chpule2}[
]{.chpuri2}これらのインターフェースを実装する様々な種類のコレクションやイテレーターを渡しても問題なく動作します[。]{.chpule2}[
]{.chpuri2}
:::
:::::::::
::::::::::

::: {.section .checklist}
##  実装方法 {#checklist}

1.  イテレーター・インターフェースを宣言します[。]{.chpule2}[
    ]{.chpuri2}少なくとも[、]{.chpule2}[
    ]{.chpuri2}コレクションから次の要素を獲得するためのメソッドを持つ必要があります[。]{.chpule2}[
    ]{.chpuri2}しかし[、]{.chpule2}[
    ]{.chpuri2}利便性を考え[、]{.chpule2}[
    ]{.chpuri2}前要素の獲得[、]{.chpule2}[
    ]{.chpuri2}現在位置の確認[、]{.chpule2}[
    ]{.chpuri2}探索終了チェックなど[、]{.chpule2}[
    ]{.chpuri2}いくつかの他のメソッドを追加することもできます[。]{.chpule2}[
    ]{.chpuri2}

2.  コレクションのインターフェースを宣言し[、]{.chpule2}[
    ]{.chpuri2}イテレーター取得メソッドを記述します[。]{.chpule2}[
    ]{.chpuri2}戻り値の型はイテレーター・インターフェースにする必要があります[。]{.chpule2}[
    ]{.chpuri2}いくつかの異なるイテレーターを作る予定の場合は[、]{.chpule2}[
    ]{.chpuri2}似たようなメソッドをいくつか宣言してもかまいません[。]{.chpule2}[
    ]{.chpuri2}

3.  イテレーターで探索したいコレクションに対して具象イテレーター・クラスを実装します[。]{.chpule2}[
    ]{.chpuri2}イテレーター・オブジェクトはコレクションのインスタンス一つとリンクされている必要があります[。]{.chpule2}[
    ]{.chpuri2}通常[、]{.chpule2}[
    ]{.chpuri2}このリンクはイテレーターのコンストラクターを介して確立されます[。]{.chpule2}[
    ]{.chpuri2}

4.  コレクション・インターフェースを実装してコレクション・クラスを作成します[。]{.chpule2}[
    ]{.chpuri2}基本的な考え方としては[、]{.chpule2}[
    ]{.chpuri2}特定のコレクション・クラス用に調整されたイテレーターを作成するための近道をクライアントに提供するということです[。]{.chpule2}[
    ]{.chpuri2}コレクション・オブジェクトは[、]{.chpule2}[
    ]{.chpuri2}イテレーターのコンストラクターに自身を渡して[、]{.chpule2}[
    ]{.chpuri2}リンクを確立する必要があります[。]{.chpule2}[ ]{.chpuri2}

5.  クライアント・コードを見渡して[、]{.chpule2}[
    ]{.chpuri2}すべてのコレクションの探索コードをイテレーターの使用に置き換えます[。]{.chpule2}[
    ]{.chpuri2}クライアントは[、]{.chpule2}[
    ]{.chpuri2}コレクション要素を反復する必要があるたびに[、]{.chpule2}[
    ]{.chpuri2}新しいイテレーター・オブジェクトを獲得します[。]{.chpule2}[
    ]{.chpuri2}
:::

:::::: {.section .pros-cons}
##  長所と短所 {#pros-cons}

::::: row
::: col-sm-6
-   
    *[単]{.cjk}[一]{.cjk}[責]{.cjk}[任]{.cjk}[の]{.cjk}[原]{.cjk}[則]{.cjk}*[。]{.chpule2}[
    ]{.chpuri2}かさばる探索アルゴリズムを別クラスに抽出することにより[、]{.chpule2}[
    ]{.chpuri2}クライアント・コードとコレクションの整理整頓が可能[。]{.chpule2}[
    ]{.chpuri2}
-   
    *[開]{.cjk}[放]{.cjk}[閉]{.cjk}[鎖]{.cjk}[の]{.cjk}[原]{.cjk}[則]{.cjk}*[。]{.chpule2}[
    ]{.chpuri2}新しい種類のコレクションとイテレーターを実装し[、]{.chpule2}[
    ]{.chpuri2}既存のコードに問題なく渡すことが可能[。]{.chpule2}[
    ]{.chpuri2}
-   
    各イテレーター・オブジェクトが自身の反復状態を持っているため[、]{.chpule2}[
    ]{.chpuri2}同じコレクションを平行して反復可能[。]{.chpule2}[
    ]{.chpuri2}
-    同じ理由で[、]{.chpule2}[
    ]{.chpuri2}反復を一時中断して必要な時に再開することが可能[。]{.chpule2}[
    ]{.chpuri2}
:::

::: col-sm-6
-    アプリが単純なコレクションとのみ動作する場合[、]{.chpule2}[
    ]{.chpuri2}このパターンの適用は行き過ぎの可能性[。]{.chpule2}[
    ]{.chpuri2}
-    特定の特殊なコレクションの場合[、]{.chpule2}[
    ]{.chpuri2}イテレーターの使用は[、]{.chpule2}[
    ]{.chpuri2}要素を直接見て行くよりも非効率な可能性[。]{.chpule2}[
    ]{.chpuri2}
:::
:::::
::::::

::: {.section .relations}
##  他のパターンとの関係 {#relations}

-   [Iterators](iterator.html) を使用して [Composite](composite.html)
    ツリーを探索することができます[。]{.chpule2}[ ]{.chpuri2}

-   [Factory Method](factory-method.html) を [Iterator](iterator.html)
    と一緒に使って[、]{.chpule2}[
    ]{.chpuri2}コレクションのサブクラスが[、]{.chpule2}[
    ]{.chpuri2}コレクションと互換な[、]{.chpule2}[
    ]{.chpuri2}異なる型のイテレーターを返すようにできます[。]{.chpule2}[
    ]{.chpuri2}

-   現在の反復状態を獲得し[、]{.chpule2}[
    ]{.chpuri2}必要に応じてロールバックするために[、]{.chpule2}[
    ]{.chpuri2}[Memento](memento.html) を [Iterator](iterator.html)
    と一緒に使用できます[。]{.chpule2}[ ]{.chpuri2}

-   複雑なデータ構造を探索し[、]{.chpule2}[
    ]{.chpuri2}その要素に対してある操作を実行するために[、]{.chpule2}[
    ]{.chpuri2}[Visitor](visitor.html) を [Iterator](iterator.html)
    と一緒に使用することができます[。]{.chpule2}[
    ]{.chpuri2}要素のクラスが全部異なっていてもかまいません[。]{.chpule2}[
    ]{.chpuri2}
:::

::: {.section .implementations}
##  コード例 {#implementations}

[![Iterator を C#
で](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](iterator/csharp/example.html "Iterator を C# で"){.prog-lang-link}
[![Iterator を C++
で](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](iterator/cpp/example.html "Iterator を C++ で"){.prog-lang-link}
[![Iterator を Go
で](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](iterator/go/example.html "Iterator を Go で"){.prog-lang-link}
[![Iterator を Java
で](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](iterator/java/example.html "Iterator を Java で"){.prog-lang-link}
[![Iterator を PHP
で](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](iterator/php/example.html "Iterator を PHP で"){.prog-lang-link}
[![Iterator を Python
で](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](iterator/python/example.html "Iterator を Python で"){.prog-lang-link}
[![Iterator を Ruby
で](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](iterator/ruby/example.html "Iterator を Ruby で"){.prog-lang-link}
[![Iterator を Rust
で](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](iterator/rust/example.html "Iterator を Rust で"){.prog-lang-link}
[![Iterator を Swift
で](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](iterator/swift/example.html "Iterator を Swift で"){.prog-lang-link}
[![Iterator を TypeScript
で](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](iterator/typescript/example.html "Iterator を TypeScript で"){.prog-lang-link}
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

[Mediator []{.fa .fa-arrow-right}](mediator.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### 戻る

[[]{.fa .fa-arrow-left} Command](command.html){.btn .btn-default
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
