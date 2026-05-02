::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} /
[デザインパターン](../design-patterns.html){.type} /
[構造に関するパターン](structural-patterns.html){.type}
:::

# Decorator {#decorator .title}

::: aka
別名：[Wrapper、]{style="display:inline-block"}[デコレーター、]{style="display:inline-block"}[ラッパー]{style="display:inline-block"}
:::

::: {.section .intent}
##  一言でいうと {#intent}

**Decorator**[ ]{.chpule1}[（]{.chpuri1}デコレーター[、]{.chpule2}[
]{.chpuri2}装飾器[）]{.chpule2}[ ]{.chpuri2}は[、]{.chpule2}[
]{.chpuri2}構造に関するデザインパターンの一つで[、]{.chpule2}[
]{.chpuri2}ある振る舞いを含む特別なラッパー・オブジェクトの中にオブジェクトを配置することで[、]{.chpule2}[
]{.chpuri2}それらのオブジェクトに新しい振る舞いを付け加えます[。]{.chpule2}[
]{.chpuri2}

<figure class="image">
<img
src="../../images/patterns/content/decorator/decoratore8f5.png?id=710c66670c7123e0928d3b3758aea79e"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/decorator/decorator.png?id=710c66670c7123e0928d3b3758aea79e 640w,/images/patterns/content/decorator/decorator-1.5x.png?id=72aafc603a289fc64e028e83e8aa843b 960w,/images/patterns/content/decorator/decorator-2x.png?id=736ab07b1d8920ab2c7a70c9cb1305cc 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Decorator デザインパターン" />
</figure>
:::

::: {.section .problem}
##  問題 {#problem}

プログラムが重要なイベントについてユーザーに通知できる[、]{.chpule2}[
]{.chpuri2}通知ライブラリーを開発しているとします[。]{.chpule2}[
]{.chpuri2}

ライブラリーの初期バージョンは[、]{.chpule2}[
]{.chpuri2}少しのフィールドとコンストラクターと単一の `send`
メソッドからなる `Notifier` クラスを基本にしてました[。]{.chpule2}[
]{.chpuri2}このメソッドは[、]{.chpule2}[
]{.chpuri2}クライアントからメッセージの引数を受け取り[、]{.chpule2}[
]{.chpuri2}コンストラクターを介して通知オブジェクトに渡されたメールアドレスのリストに[、]{.chpule2}[
]{.chpuri2}そのメッセージを送信することができました[。]{.chpule2}[
]{.chpuri2}他社制作のアプリがクライアントで[、]{.chpule2}[
]{.chpuri2}通知オブジェクトを一度だけ構築して設定し[、]{.chpule2}[
]{.chpuri2}何か重要なことが起こるたびにそのオブジェクトを使用することになっていました[。]{.chpule2}[
]{.chpuri2}

<figure class="image">
<img
src="../../images/patterns/diagrams/decorator/problem1-ja292b.png?id=a3e32153fa25e772014e5f3adfed41da"
style="aspect-ratio: 2.45;"
srcset="/images/patterns/diagrams/decorator/problem1-ja.png?id=a3e32153fa25e772014e5f3adfed41da 540w,/images/patterns/diagrams/decorator/problem1-ja-1.5x.png?id=9dc9b74372ca7aa83a802373ba9e7595 810w,/images/patterns/diagrams/decorator/problem1-ja-2x.png?id=154f4a3231ff4d6e833e8e26fbe4886b 1080w"
sizes="(max-width: 720px) 100vw, 540px" loading="lazy" width="540"
alt="Decorator パターン適用前のライブラリーの構造" />
<figcaption><p>プログラムは<span class="chpule2">、</span><span
class="chpuri2"> </span>通知クラスを使用して<span
class="chpule2">、</span><span class="chpuri2">
</span>重要なイベントに関する通知を事前設定されえた固定のメール・アドレスのリストに送信することができた<span
class="chpule2">。</span><span class="chpuri2"> </span></p></figcaption>
</figure>

ある時[、]{.chpule2}[ ]{.chpuri2}ライブラリーのユーザーは[、]{.chpule2}[
]{.chpuri2}電子メール通知以上のものを期待していることに気づきます[。]{.chpule2}[
]{.chpuri2}多数のユーザーは[、]{.chpule2}[
]{.chpuri2}重要事項については[、]{.chpule2}[ ]{.chpuri2}SMS
を受信することを望んでいます[。]{.chpule2}[ ]{.chpuri2}Facebook
で通知を受けたい人もいます[。]{.chpule2}[
]{.chpuri2}そして勿論[、]{.chpule2}[
]{.chpuri2}企業ユーザーは[、]{.chpule2}[ ]{.chpuri2}Slack
の通知を望んでいます[。]{.chpule2}[ ]{.chpuri2}

<figure class="image">
<img
src="../../images/patterns/diagrams/decorator/problem2a2f8.png?id=ba5d5e106ea8c4848d60e230feca9135"
style="aspect-ratio: 2.59;"
srcset="/images/patterns/diagrams/decorator/problem2.png?id=ba5d5e106ea8c4848d60e230feca9135 440w,/images/patterns/diagrams/decorator/problem2-1.5x.png?id=4f15b6208533188dd09e7da7dd6b509a 660w,/images/patterns/diagrams/decorator/problem2-2x.png?id=28b2c8509b4e78c031d728424b876ebc 880w"
sizes="(max-width: 720px) 100vw, 440px" loading="lazy" width="440"
alt="他の通知タイプを実装した後のライブラリーの構造" />
<figcaption><p>通知タイプ一つずつを<span class="chpule2">、</span><span
class="chpuri2"> </span>サブクラスとして実装する<span
class="chpule2">。</span><span class="chpuri2"> </span></p></figcaption>
</figure>

簡単そうですね[？]{.chpule2}[ ]{.chpuri2}単に `Notifier`
クラスを拡張して[、]{.chpule2}[
]{.chpuri2}新しいサブクラスに追加の通知メソッドをいくつか追加すればいいです[。]{.chpule2}[
]{.chpuri2}クライアントは[、]{.chpule2}[
]{.chpuri2}希望する通知クラスのインスタンスを作り[、]{.chpule2}[
]{.chpuri2}それを以降の全部の通知で使うことになっていました[。]{.chpule2}[
]{.chpuri2}

しかし[、]{.chpule2}[
]{.chpuri2}誰かが理にかなった質問をします[[。]{.ls-1}]{.chpule2}[
]{.chpuri2}​[
]{.chpule1}[「]{.chpuri1}何で一度に複数の通知タイプを使用できないんですか[？]{.chpule2}[
]{.chpuri2}家が火事になったら[、]{.chpule2}[
]{.chpuri2}全チャンネルを通して通知されたいでしょう[。]{.ls-05}[」]{.chpule2}[
]{.chpuri2}

そこで[、]{.chpule2}[ ]{.chpuri2}これを解決しようと[、]{.chpule2}[
]{.chpuri2}複数の通知メソッドを含む新しい特別なサブクラスを作成してみました[。]{.chpule2}[
]{.chpuri2}しかし[、]{.chpule2}[
]{.chpuri2}このやり方では[、]{.chpule2}[
]{.chpuri2}ライブラリーのコードだけでなく[、]{.chpule2}[
]{.chpuri2}クライアントのコードも非常に膨れ上がってしまうことがすぐに明らかになりました[。]{.chpule2}[
]{.chpuri2}

<figure class="image">
<img
src="../../images/patterns/diagrams/decorator/problem34033.png?id=f3b3e7a107d870871f2c3167adcb7ccb"
style="aspect-ratio: 1.85;"
srcset="/images/patterns/diagrams/decorator/problem3.png?id=f3b3e7a107d870871f2c3167adcb7ccb 630w,/images/patterns/diagrams/decorator/problem3-1.5x.png?id=f4ef9d367df838c77121e9f84260b09c 945w,/images/patterns/diagrams/decorator/problem3-2x.png?id=abb7a87b521ce97d7661dd9c0b988cc3 1260w"
sizes="(max-width: 720px) 100vw, 630px" loading="lazy" width="630"
alt="クラスの組み合わせを作成した後のライブラリーの構造" />
<figcaption><p>サブクラスの組み合わせの爆発</p></figcaption>
</figure>

通知機能の構造を作るにあたって[、]{.chpule2}[
]{.chpuri2}サブクラスの数が間違ってギネス記録を破ることのないような[、]{.chpule2}[
]{.chpuri2}別の方法を見つける必要があります[。]{.chpule2}[ ]{.chpuri2}
:::

::: {.section .solution}
##  解決策 {#solution}

オブジェクトの振る舞いを変更する必要がある場合[、]{.chpule2}[
]{.chpuri2}クラス拡張は[、]{.chpule2}[
]{.chpuri2}最初に思い浮かぶことです[。]{.chpule2}[
]{.chpuri2}ただし[、]{.chpule2}[
]{.chpuri2}継承にはいくつかの深刻な注意すべき点があります[。]{.chpule2}[
]{.chpuri2}

-   継承は静的です[。]{.chpule2}[
    ]{.chpuri2}既存のオブジェクトの振る舞いは実行時に変更できません[。]{.chpule2}[
    ]{.chpuri2}オブジェクト全体を別のサブクラスから作成された別のものと置き換えることのみできます[。]{.chpule2}[
    ]{.chpuri2}
-   サブクラスは[、]{.chpule2}[
    ]{.chpuri2}親クラスを一つしか持てません[。]{.chpule2}[
    ]{.chpuri2}ほとんどの言語では[、]{.chpule2}[
    ]{.chpuri2}同時に複数のクラスから振る舞いを継承することはできません[。]{.chpule2}[
    ]{.chpuri2}

このような注意点を克服する方法の一つとしては[、]{.chpule2}[
]{.chpuri2}*[継]{.cjk}[承]{.cjk}* の代わりに*[集]{.cjk}[約]{.cjk}*[
]{.chpule1}[（]{.chpuri1}Aggregation[）]{.chpule2}[
]{.chpuri2}または*[合]{.cjk}[成]{.cjk}* []{.pop-i}[**集約**[：]{.chpule2}[
]{.chpuri2}オブジェクト A はオブジェクト B を含み[、]{.chpule2}[
]{.chpuri2}B は A なしで生存可能[。]{.chpule2}[ ]{.chpuri2}\
**合成**[：]{.chpule2}[ ]{.chpuri2}オブジェクト A はオブジェクト B
から構成され[、]{.chpule2}[ ]{.chpuri2}B
のライフサイクルを管理[。]{.chpule2}[ ]{.chpuri2}B は[、]{.chpule2}[
]{.chpuri2}A なしで存在不可[。]{.chpule2}[ ]{.chpuri2}]{.pop-content}[
]{.chpule1}[（]{.chpuri1}Composition[）]{.chpule2}[
]{.chpuri2}を使用することです[。]{.chpule2}[
]{.chpuri2}どちらの選択肢もほぼ同じように機能します[。]{.chpule2}[
]{.chpuri2}あるオブジェクトが別のオブジェクト参照を
*[一]{.cjk}[つ]{.cjk}[保]{.cjk}[持]{.cjk}*[
]{.chpule1}[（]{.chpuri1}has-a[）]{.chpule2}[
]{.chpuri2}し[、]{.chpule2}[
]{.chpuri2}そのオブジェクトに仕事を委任します[。]{.chpule2}[
]{.chpuri2}継承ではオブジェクト*[自]{.cjk}[身]{.cjk}[が]{.cjk}*[
]{.chpule1}[（]{.chpuri1}is[）]{.chpule2}[
]{.chpuri2}スーパークラスから振る舞いを継承し[、]{.chpule2}[
]{.chpuri2}仕事を行います[。]{.chpule2}[ ]{.chpuri2}

この新しいやり方では[、]{.chpule2}[ ]{.chpuri2}リンクされた[
]{.chpule1}[「]{.chpuri1}ヘルパー[」]{.chpule2}[
]{.chpuri2}オブジェクトを簡単に別のオブジェクトに置き換えることができ[、]{.chpule2}[
]{.chpuri2}実行時のコンテナの振る舞いを変更できす[。]{.chpule2}[
]{.chpuri2}オブジェクト一つから複数のオブジェクトへの参照を持ち[、]{.chpule2}[
]{.chpuri2}いろいろな種類の作業を委任することにより[、]{.chpule2}[
]{.chpuri2}様々なクラスの振る舞いを利用できます[。]{.chpule2}[
]{.chpuri2}集約と合成は[、]{.chpule2}[ ]{.chpuri2}Decorator
を含む多くのデザインパターンの背後にある重要な原則です[。]{.chpule2}[
]{.chpuri2}というところで[、]{.chpule2}[
]{.chpuri2}パターンの議論に戻りましょう[。]{.chpule2}[ ]{.chpuri2}

<figure class="image">
<img
src="../../images/patterns/diagrams/decorator/solution1-ja5628.png?id=01ae1d8c8aa8f04d51e0a2e65a3c274a"
style="aspect-ratio: 3.44;"
srcset="/images/patterns/diagrams/decorator/solution1-ja.png?id=01ae1d8c8aa8f04d51e0a2e65a3c274a 550w,/images/patterns/diagrams/decorator/solution1-ja-1.5x.png?id=203bacb202c5d0c8d96f290e4e36aabe 825w,/images/patterns/diagrams/decorator/solution1-ja-2x.png?id=bc22ac351f883dcf4f0ee42eb1dec405 1100w"
sizes="(max-width: 720px) 100vw, 550px" loading="lazy" width="550"
alt="継承 対 集約" />
<figcaption><p>継承 対 集約</p></figcaption>
</figure>

[ ]{.chpule1}[「]{.chpuri1}Wrapper[[」]{.ls-1}]{.chpule2}[ ]{.chpuri2}​[
]{.chpule1}[（]{.chpuri1}ラッパー[、]{.chpule2}[
]{.chpuri2}包装[）]{.chpule2}[ ]{.chpuri2}は[、]{.chpule2}[
]{.chpuri2}Decorator パターンのもう一つのニックネームです[。]{.chpule2}[
]{.chpuri2}このパターンの中心となるアイデアを明確に表現しています[。]{.chpule2}[
]{.chpuri2}*[ラ]{.cjk}[ッ]{.cjk}[パ]{.cjk}[ー]{.cjk}* は[、]{.chpule2}[
]{.chpuri2}何らかの
*[タ]{.cjk}[ー]{.cjk}[ゲ]{.cjk}[ッ]{.cjk}[ト]{.cjk}*オブジェクトにリンクできるオブジェクトです[。]{.chpule2}[
]{.chpuri2}ラッパーはターゲットと同じメソッドを含み[、]{.chpule2}[
]{.chpuri2}受け取ったすべてのリクエストをターゲットに委任します[。]{.chpule2}[
]{.chpuri2}しかし[、]{.chpule2}[
]{.chpuri2}ラッパーはリクエストをターゲットに渡す前や後に[、]{.chpule2}[
]{.chpuri2}何らかのことを行い結果を操作することができます[。]{.chpule2}[
]{.chpuri2}

単純なラッパーはいつ真のデコレーターになるのでしょう[？]{.chpule2}[
]{.chpuri2}さっき述べたように[、]{.chpule2}[
]{.chpuri2}ラッパーはラップされたオブジェクトと同じインターフェースを実装しています[。]{.chpule2}[
]{.chpuri2}そのため[、]{.chpule2}[
]{.chpuri2}クライアントの観点からすると[、]{.chpule2}[
]{.chpuri2}これらのオブジェクトは同一です[。]{.chpule2}[
]{.chpuri2}ラッパーの参照フィールドが[、]{.chpule2}[
]{.chpuri2}そのインターフェースに沿ったあらゆるオブジェクトを受け入れるようにしてください[。]{.chpule2}[
]{.chpuri2}オブジェクトを複数のラッパーでカバーし[、]{.chpule2}[
]{.chpuri2}すべてのラッパーの振る舞いの組み合わせを追加できるようになります[。]{.chpule2}[
]{.chpuri2}

先ほどの通知の例では[、]{.chpule2}[
]{.chpuri2}電子メール通知の振る舞いは[、]{.chpule2}[
]{.chpuri2}`Notifier` 基底クラスに残し[、]{.chpule2}[
]{.chpuri2}他のすべての通知方法をデコレーターに変えてみましょう[。]{.chpule2}[
]{.chpuri2}

<figure class="image">
<img
src="../../images/patterns/diagrams/decorator/solution2e4ab.png?id=cbee4a27080ce3a0bf773482613e1347"
style="aspect-ratio: 1.39;"
srcset="/images/patterns/diagrams/decorator/solution2.png?id=cbee4a27080ce3a0bf773482613e1347 640w,/images/patterns/diagrams/decorator/solution2-1.5x.png?id=d2fa0c0b9bf5cba0e38be7aff7e7b7a8 960w,/images/patterns/diagrams/decorator/solution2-2x.png?id=7775f76b94dbd5cd25f711ce81f59262 1280w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="640"
alt="Decorator パターンを使った解決策" />
<figcaption><p>様々な通知メソッドがデコレーターになる<span
class="chpule2">。</span><span class="chpuri2"> </span></p></figcaption>
</figure>

クライアント・コードは[、]{.chpule2}[
]{.chpuri2}クライアントの設定に一致するデコレーターの組で[、]{.chpule2}[
]{.chpuri2}基本的通知ブジェクトをラップする必要があります[。]{.chpule2}[
]{.chpuri2}結果として得られるオブジェクトは積み重ねの構造をしています[。]{.chpule2}[
]{.chpuri2}

<figure class="image">
<img
src="../../images/patterns/diagrams/decorator/solution3-jadd04.png?id=00cbe8d2c5c3381cd4106a699df66147"
style="aspect-ratio: 1.03;"
srcset="/images/patterns/diagrams/decorator/solution3-ja.png?id=00cbe8d2c5c3381cd4106a699df66147 300w,/images/patterns/diagrams/decorator/solution3-ja-1.5x.png?id=583cb85b8b90eecb558062bbc96ad2ea 450w,/images/patterns/diagrams/decorator/solution3-ja-2x.png?id=a94a300f0fc66cd98a9a006b9cb6405a 600w"
sizes="(max-width: 720px) 100vw, 300px" loading="lazy" width="300"
alt="アプリは、通知デコレーターの複雑な積み重ねのように構成するかもしれない。" />
<figcaption><p>アプリは<span class="chpule2">、</span><span
class="chpuri2">
</span>通知デコレーターの複雑な積み重ねのように構成するかもしれない<span
class="chpule2">。</span><span class="chpuri2"> </span></p></figcaption>
</figure>

積み重ねの中の最後にあるデコレーターが[、]{.chpule2}[
]{.chpuri2}クライアントが実際にやりとりするオブジェクトです[。]{.chpule2}[
]{.chpuri2}すべてのデコレーターが[、]{.chpule2}[
]{.chpuri2}通知の基底クラスと同じインターフェースを実装しているため[、]{.chpule2}[
]{.chpuri2}クライアント・コードの残りの部分は[、]{.chpule2}[
]{.chpuri2}相手が[ ]{.chpule1}[「]{.chpuri1}純粋な[」]{.chpule2}[
]{.chpuri2}通知オブジェクトなのか[、]{.chpule2}[
]{.chpuri2}デコレーターなのかは[、]{.chpule2}[
]{.chpuri2}気にしません[。]{.chpule2}[ ]{.chpuri2}

同じやり方を[、]{.chpule2}[
]{.chpuri2}メッセージの組み立てや[、]{.chpule2}[
]{.chpuri2}受信者リストの作成など[、]{.chpule2}[
]{.chpuri2}他の振る舞いにも適用できます[。]{.chpule2}[
]{.chpuri2}クライアントは[、]{.chpule2}[
]{.chpuri2}同じインターフェースに沿っている限り[、]{.chpule2}[
]{.chpuri2}いかなる特製デコレーターを使ってでもオブジェクトを飾ることができます[。]{.chpule2}[
]{.chpuri2}
:::

::: {.section .analogy}
##  現実世界でのたとえ {#analogy}

<figure class="image">
<img
src="../../images/patterns/content/decorator/decorator-comic-1cbff.png?id=80d95baacbfb91f5bcdbdc7814b0c64d"
style="aspect-ratio: 2;"
srcset="/images/patterns/content/decorator/decorator-comic-1.png?id=80d95baacbfb91f5bcdbdc7814b0c64d 600w,/images/patterns/content/decorator/decorator-comic-1-1.5x.png?id=490ef9754d7554c2046b69f6f213c6da 900w,/images/patterns/content/decorator/decorator-comic-1-2x.png?id=ba869f621b6e0ea173fdc2b535fc7eed 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Decorator パターンの例" />
<figcaption><p>衣類を重ね着して<span class="chpule2">、</span><span
class="chpuri2"> </span>組み合わせの効果を得る<span
class="chpule2">。</span><span class="chpuri2"> </span></p></figcaption>
</figure>

服を着ることはデコレーター使用の例です[。]{.chpule2}[
]{.chpuri2}寒い時は自分をセーターに包み込み込みます[。]{.chpule2}[
]{.chpuri2}セーターだけではまだ寒い場合[、]{.chpule2}[
]{.chpuri2}上にジャケットを着ることができます[。]{.chpule2}[
]{.chpuri2}雨が降っていれば[、]{.chpule2}[
]{.chpuri2}レインコートを着られます[。]{.chpule2}[
]{.chpuri2}これらの衣服のすべてがあなたの基本的な振る舞いを[
]{.chpule1}[「]{.chpuri1}拡張[」]{.chpule2}[
]{.chpuri2}しますが[、]{.chpule2}[
]{.chpuri2}それはあなたの一部ではありません[。]{.chpule2}[
]{.chpuri2}必要がなくなればいつでも簡単に服を脱ぐことができます[。]{.chpule2}[
]{.chpuri2}
:::

::::: {.section .structure-container}
##  構造 {#structure}

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/decorator/structure128f.png?id=8c95d894aecce5315cc1b12093a7ea0c"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 0.92;"
srcset="/images/patterns/diagrams/decorator/structure.png?id=8c95d894aecce5315cc1b12093a7ea0c 480w,/images/patterns/diagrams/decorator/structure-1.5x.png?id=4b2cd91d4483d9e3bba725f0e45d281d 720w,/images/patterns/diagrams/decorator/structure-2x.png?id=3cfa1f10417a4ef0c12580bc4a63b80d 960w"
sizes="(max-width: 720px) 100vw, 480px" loading="lazy" width="480"
alt="Decorator デザインパターンの構造" /><img
src="../../images/patterns/diagrams/decorator/structure-indexeda93c.png?id=09401b230a58f2249e4c9a1195d485a0"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 0.98;"
srcset="/images/patterns/diagrams/decorator/structure-indexed.png?id=09401b230a58f2249e4c9a1195d485a0 500w,/images/patterns/diagrams/decorator/structure-indexed-1.5x.png?id=dccc54182965078ccb4cfdeee41acbe5 750w,/images/patterns/diagrams/decorator/structure-indexed-2x.png?id=2733e7d0e322bfb2f150ccf8a878dbf6 1000w"
sizes="(max-width: 720px) 100vw, 500px" loading="lazy" width="500"
alt="Decorator デザインパターンの構造" />
</figure>
:::

1.  **コンポーネント**[
    ]{.chpule1}[（]{.chpuri1}Component[）]{.chpule2}[
    ]{.chpuri2}は[、]{.chpule2}[
    ]{.chpuri2}ラッパーとラップされるオブジェクトとの共通インターフェースを宣言します[。]{.chpule2}[
    ]{.chpuri2}

2.  **具象コンポーネント**[ ]{.chpule1}[（]{.chpuri1}Concrete
    Component[）]{.chpule2}[ ]{.chpuri2}は[、]{.chpule2}[
    ]{.chpuri2}ラップされるオブジェクトのクラスで[、]{.chpule2}[
    ]{.chpuri2}デコレーターによって変更できる基本的な振る舞いを定義します[。]{.chpule2}[
    ]{.chpuri2}

3.  **基底デコレーター**[ ]{.chpule1}[（]{.chpuri1}Base
    Decorator[）]{.chpule2}[ ]{.chpuri2}クラスには[、]{.chpule2}[
    ]{.chpuri2}ラップされたオブジェクトを参照するためのフィールドがあります[。]{.chpule2}[
    ]{.chpuri2}フィールドの型は[、]{.chpule2}[
    ]{.chpuri2}具象コンポーネントとデコレーターの両方を含むことができるように[、]{.chpule2}[
    ]{.chpuri2}コンポーネント・インターフェースとして宣言する必要があります[。]{.chpule2}[
    ]{.chpuri2}基底デコレーターは[、]{.chpule2}[
    ]{.chpuri2}すべての操作をラップされたオブジェクトに委任します[。]{.chpule2}[
    ]{.chpuri2}

4.  **具象デコレーター**[ ]{.chpule1}[（]{.chpuri1}Concrete
    Decorator[）]{.chpule2}[ ]{.chpuri2}は[、]{.chpule2}[
    ]{.chpuri2}コンポーネントに動的に追加可能な振る舞いを定義します[。]{.chpule2}[
    ]{.chpuri2}具象デコレーターは[、]{.chpule2}[
    ]{.chpuri2}基底デコレーターのメソッドを上書きし[、]{.chpule2}[
    ]{.chpuri2}親メソッドを呼び出す前または後に追加の振る舞いのコードを実行します[。]{.chpule2}[
    ]{.chpuri2}

5.  **クライアント**[ ]{.chpule1}[（]{.chpuri1}Client[）]{.chpule2}[
    ]{.chpuri2}は[、]{.chpule2}[
    ]{.chpuri2}コンポーネント・インターフェースを介してすべてのオブジェクトとやりとりする限り[、]{.chpule2}[
    ]{.chpuri2}コンポーネントを複数のデコレーターの層でラップできます[。]{.chpule2}[
    ]{.chpuri2}
::::
:::::

::: {.section .pseudocode}
##  擬似コード {#pseudocode}

この例では[、]{.chpule2}[ ]{.chpuri2}**Decorator**
パターンを適応して[、]{.chpule2}[
]{.chpuri2}機密データの圧縮と暗号化を[、]{.chpule2}[
]{.chpuri2}このデータを実際に使うコードから独立して行います[。]{.chpule2}[
]{.chpuri2}

<figure class="image">
<img
src="../../images/patterns/diagrams/decorator/example42cb.png?id=eec9dc488f00c85f50e764323baa723e"
style="aspect-ratio: 0.98;"
srcset="/images/patterns/diagrams/decorator/example.png?id=eec9dc488f00c85f50e764323baa723e 540w,/images/patterns/diagrams/decorator/example-1.5x.png?id=40ccaba4f5a8e6a2ebac697f04b9f10a 810w,/images/patterns/diagrams/decorator/example-2x.png?id=4891323a27d5601a174eec366187d833 1080w"
sizes="(max-width: 720px) 100vw, 540px" loading="lazy" width="540"
alt="Decorator パターン例の構造" />
<figcaption><p>暗号化と圧縮のデコレーターの例</p></figcaption>
</figure>

アプリケーションは[、]{.chpule2}[
]{.chpuri2}元データのオブジェクトを二つのデコレーターで包み込みます[。]{.chpule2}[
]{.chpuri2}どちらのラッパーも[、]{.chpule2}[
]{.chpuri2}データの書き込み方法とディスクからの読み取り方を以下のように変更します[：]{.chpule2}[
]{.chpuri2}

-   データを**ディスクに書き込む**直前に[、]{.chpule2}[
    ]{.chpuri2}デコレーターが暗号化と圧縮を行います[。]{.chpule2}[
    ]{.chpuri2}元のクラスは[、]{.chpule2}[
    ]{.chpuri2}暗号化され安全になったデータを[、]{.chpule2}[
    ]{.chpuri2}そのような変更が行われたことを知らずに[、]{.chpule2}[
    ]{.chpuri2}ファイルに書き込みます[。]{.chpule2}[ ]{.chpuri2}

-   データを**ディスクから読み取った**直後に[、]{.chpule2}[
    ]{.chpuri2}データは同じ二つのデコレーターを通過し[、]{.chpule2}[
    ]{.chpuri2}解凍と復号が行われます[。]{.chpule2}[ ]{.chpuri2}

デコレーターとデータ・ソース・クラスは[、]{.chpule2}[
]{.chpuri2}同じインターフェースを実装しているため[、]{.chpule2}[
]{.chpuri2}クライアント・コード内で入れ替えが可能です[。]{.chpule2}[
]{.chpuri2}

<figure class="code">
<pre class="code" lang="pseudocode"><code>// コンポーネントのインターフェースは、デコレーターが変更できる操作を定義。
interface DataSource is
    method writeData(data)
    method readData():data

// 具象コンポーネントは、操作のデフォルトの実装を行う。プログラムには、こ
// れらのクラスのいくつかの異種があるかもしれない。
class FileDataSource implements DataSource is
    constructor FileDataSource(filename) { …… }

    method writeData(data) is
        // データをファイルに書く。

    method readData():data is
        // データをファイルから読む。

// 基底デコレーター・クラスは、他のコンポーネントと同じインターフェースに
// 従う。このクラスの一次的目的は、すべての具象デコレーターのラップ用イン
// ターフェースを定義すること。ラッパー・コードのデフォルトの実装には、
// ラップされたコンポーネントを格納するためのフィールドとそれを初期化する
// 方法が含まれているかもしれない。
class DataSourceDecorator implements DataSource is
    protected field wrappee: DataSource

    constructor DataSourceDecorator(source: DataSource) is
        wrappee = source

    // 基底デコレーターは、ラップされたコンポーネントにすべての作業を単純
    // に委任。追加の振る舞いは具象デコレーターに追加可能。
    method writeData(data) is
        wrappee.writeData(data)

    // 具象デコレーターはラップされたオブジェクトを直接呼び出す代わりに、
    // 親の実装を呼び出すこともできる。このやり方は、デコレーター・クラス
    // の拡張を簡単に行える。
    method readData():data is
        return wrappee.readData()

// 具象デコレーターはラップされたオブジェクトのメソッドを呼び出さなければ
// ならないが、結果に何かを付け足すことは許される。デコレーターは、ラップ
// されたオブジェクトへの呼び出しの前か後に追加の振る舞いを実行可能。
class EncryptionDecorator extends DataSourceDecorator is
    method writeData(data) is
        // 1. 渡されたデータを暗号化。
        // 2. ラップされたオブジェクトの writeData メソッドに暗号化され
        // たデータを渡す。

    method readData():data is
        // 1. ラップされたオブジェクトの readData メソッドからデータを
        // 取得。
        // 2. 暗号化されている場合は復号を試みる。
        // 3. 結果を返す。

// 複数の層のデコレーターでオブジェクトをラップ可能。
class CompressionDecorator extends DataSourceDecorator is
    method writeData(data) is
        // 1. 渡されたデータを圧縮。
        // 2. ラップされたオブジェクトの writeData メソッドに圧縮した
        // データを渡す。

    method readData():data is
        // 1. ラップされたオブジェクトの readData メソッドからデータを
        // 取得。
        // 2. 圧縮されている場合は、解凍を試みる。
        // 3. 結果を返す。


// オプション 1. デコレーターの組み合わせの単純な例。
class Application is
    method dumbUsageExample() is
        source = new FileDataSource(&quot;somefile.dat&quot;)
        source.writeData(salaryRecords)
        // データはそのままの形で目的のファイルに書き込まれた。

        source = new CompressionDecorator(source)
        source.writeData(salaryRecords)
        // データは圧縮されて目的のファイルに書き込まれた。

        source = new EncryptionDecorator(source)
        // この時点で source 変数に含まれているのは：
        // 暗号化 &gt; 圧縮 &gt; FileDataSource
        source.writeData(salaryRecords)
        // 圧縮、暗号化されたデータがファイルに書き込まれた。


// オプション 2. クライアント・コードは外部データ・ソースを使用。SalaryManager
// オブジェクトは、データ格納方法の詳細を知らず、関心もない。事前に決められ、
// アプリケーション設定コードから受け取ったデータソースに対して作業を行
// う。
class SalaryManager is
    field source: DataSource

    constructor SalaryManager(source: DataSource) { …… }

    method load() is
        return source.readData()

    method save() is
        source.writeData(salaryRecords)
    // 他の便利なメソッドが続く……


// アプリは設定や環境に従い、他のデコレーターの組み合わせを実行時に作成可
// 能。
class ApplicationConfigurator is
    method configurationExample() is
        source = new FileDataSource(&quot;salary.dat&quot;)
        if (enabledEncryption)
            source = new EncryptionDecorator(source)
        if (enabledCompression)
            source = new CompressionDecorator(source)

        logger = new SalaryManager(source)
        salary = logger.load()
    // ……</code></pre>
</figure>
:::

:::::::: {.section .applicability-container}
##  適応性 {#applicability}

::::::: applicability
::: applicability-problem
オブジェクトに[、]{.chpule2}[
]{.chpuri2}それを使うコードを変更せずに[、]{.chpule2}[
]{.chpuri2}追加の振る舞いを実行時に与える必要がある場合に[、]{.chpule2}[
]{.chpuri2}Decorator パターンを使用します[。]{.chpule2}[ ]{.chpuri2}
:::

::: applicability-solution
Decorator では[、]{.chpule2}[
]{.chpuri2}ビジネス上のロジックを層構造にし[、]{.chpule2}[
]{.chpuri2}各層ごとにデコレーターを一つ作成し[、]{.chpule2}[
]{.chpuri2}これを実行時に組み合わせます[。]{.chpule2}[
]{.chpuri2}クライアントのコードは[、]{.chpule2}[
]{.chpuri2}これらのオブジェクトを同じ方法で扱えます[。]{.chpule2}[
]{.chpuri2}どれも共通のインターフェースに沿っているからです[。]{.chpule2}[
]{.chpuri2}
:::

::: applicability-problem
継承を使ってオブジェクトの振る舞いを拡張することが簡単にできない場合や不可能な場合に[、]{.chpule2}[
]{.chpuri2}このパターンを使用してください[。]{.chpule2}[ ]{.chpuri2}
:::

::: applicability-solution
多くのプログラミング言語には[、]{.chpule2}[
]{.chpuri2}クラスの拡張を防止する `final`
キーワードが用意されています[。]{.chpule2}[ ]{.chpuri2}final
クラスの場合[、]{.chpule2}[
]{.chpuri2}既存の振る舞いを再利用する唯一の方法は[、]{.chpule2}[
]{.chpuri2}Decorator パターンを使い[、]{.chpule2}[
]{.chpuri2}そのクラスを独自のラッパーで包み込むことです[。]{.chpule2}[
]{.chpuri2}
:::
:::::::
::::::::

::: {.section .checklist}
##  実装方法 {#checklist}

1.  プログラムの処理対象が[、]{.chpule2}[
    ]{.chpuri2}一つの主要コンポーネントと[、]{.chpule2}[
    ]{.chpuri2}それに追加可能な複数の層として表現可能なことを確認してください[。]{.chpule2}[
    ]{.chpuri2}

2.  主要コンポーネントと追加の層に共通なメソッドは何かを考え出してください[。]{.chpule2}[
    ]{.chpuri2}そのようなメソッドを列挙した共通インターフェースを宣言します[。]{.chpule2}[
    ]{.chpuri2}

3.  具象コンポーネント・クラスを作成し[、]{.chpule2}[
    ]{.chpuri2}その中で基本動作を定義します[。]{.chpule2}[ ]{.chpuri2}

4.  デコレーターの基底クラスを作成します[。]{.chpule2}[
    ]{.chpuri2}ラップされたオブジェクトへの参照を記憶するためのフィールドが必要となります[。]{.chpule2}[
    ]{.chpuri2}このフィールドの型は[、]{.chpule2}[
    ]{.chpuri2}具象コンポーネントとデコレーターのどちらにでもリンクできるよう[、]{.chpule2}[
    ]{.chpuri2}コンポーネント・インターフェースとして宣言される必要があります[。]{.chpule2}[
    ]{.chpuri2}基底デコレーターは[、]{.chpule2}[
    ]{.chpuri2}すべての操作をラップされたオブジェクトに委任します[。]{.chpule2}[
    ]{.chpuri2}

5.  すべてのクラスがコンポーネント・インターフェースを実装していることを確認してください[。]{.chpule2}[
    ]{.chpuri2}

6.  デコレーターの基底クラスを拡張して[、]{.chpule2}[
    ]{.chpuri2}具象デコレーターを作成します[。]{.chpule2}[
    ]{.chpuri2}具象デコレーターは[、]{.chpule2}[
    ]{.chpuri2}親のメソッド[
    ]{.chpule1}[（]{.chpuri1}親メソッドは常にラップされたオブジェクトに委任[）]{.chpule2}[
    ]{.chpuri2}を呼ぶ前か呼んだ後に[、]{.chpule2}[
    ]{.chpuri2}それ自身の追加の振る舞いを実行します[。]{.chpule2}[
    ]{.chpuri2}

7.  デコレーターを作成し[、]{.chpule2}[
    ]{.chpuri2}クライアントの必要に応じてそれを組み合わせるのは[、]{.chpule2}[
    ]{.chpuri2}クライアント・コードの責任となります[。]{.chpule2}[
    ]{.chpuri2}
:::

:::::: {.section .pros-cons}
##  長所と短所 {#pros-cons}

::::: row
::: col-sm-6
-   
    新しいサブクラスを作成せずにオブジェクトの振る舞いの拡張可能[。]{.chpule2}[
    ]{.chpuri2}
-    オブジェクトへの責任の追加・削除が実行時に可能[。]{.chpule2}[
    ]{.chpuri2}
-    オブジェクトを複数のデコレーターに包み込み[、]{.chpule2}[
    ]{.chpuri2}振る舞いを組み合わせることが可能[。]{.chpule2}[
    ]{.chpuri2}
-   
    *[単]{.cjk}[一]{.cjk}[責]{.cjk}[任]{.cjk}[の]{.cjk}[原]{.cjk}[則]{.cjk}*[。]{.chpule2}[
    ]{.chpuri2}いくつもの可能なすべての振る舞いを実装した一枚岩のクラスを[、]{.chpule2}[
    ]{.chpuri2}いくつかの小さなクラスに分割可能[。]{.chpule2}[
    ]{.chpuri2}
:::

::: col-sm-6
-    ラッパーの積み重ねから特定のラッパーの削除は困難[。]{.chpule2}[
    ]{.chpuri2}
-   
    振る舞いがデコレーターの積み重ねの順序に依存しないようにデコレーターを実装することは困難[。]{.chpule2}[
    ]{.chpuri2}
-    デコレーター層初期設定コードは[、]{.chpule2}[
    ]{.chpuri2}きれいとはいえない[。]{.chpule2}[ ]{.chpuri2}
:::
:::::
::::::

::: {.section .relations}
##  他のパターンとの関係 {#relations}

-   [Adapter](adapter.html)
    は既存のオブジェクトのインターフェースを変更するのに対し[、]{.chpule2}[
    ]{.chpuri2}[Decorator](decorator.html)
    はインターフェースの変更なしにオブジェクトを強力にします[。]{.chpule2}[
    ]{.chpuri2}さらに[、]{.chpule2}[ ]{.chpuri2}*Decorator*
    は[、]{.chpule2}[
    ]{.chpuri2}再帰的な合成をサポートします[。]{.chpule2}[
    ]{.chpuri2}これは[、]{.chpule2}[ ]{.chpuri2}*Adapter*
    を使用する時には不可能です[。]{.chpule2}[ ]{.chpuri2}

-   [Adapter](adapter.html)
    はラップされたオブジェクトに対しては異なるインターフェースを提供し[、]{.chpule2}[
    ]{.chpuri2}[Proxy](proxy.html)
    は同じインターフェースを提供し[、]{.chpule2}[
    ]{.chpuri2}[Decorator](decorator.html)
    は強化したインターフェースを提供します[。]{.chpule2}[ ]{.chpuri2}

-   [Chain of Responsibility](chain-of-responsibility.html) と
    [Decorator](decorator.html)
    とは非常によく似た階層構造をしています[。]{.chpule2}[
    ]{.chpuri2}両パターンとも[、]{.chpule2}[
    ]{.chpuri2}一連のオブジェクトを通して実行を渡すために[、]{.chpule2}[
    ]{.chpuri2}再起的合成に依存します[。]{.chpule2}[
    ]{.chpuri2}しかしながら[、]{.chpule2}[
    ]{.chpuri2}いくつかの重要な違いがあります[。]{.chpule2}[ ]{.chpuri2}

    *CoR* ハンドラーは[、]{.chpule2}[
    ]{.chpuri2}互いに独立して任意の処理を実行可能です[。]{.chpule2}[
    ]{.chpuri2}また[、]{.chpule2}[
    ]{.chpuri2}任意の時点でそれ以上リクエストを渡すのを止めることもできます[。]{.chpule2}[
    ]{.chpuri2}一方[、]{.chpule2}[ ]{.chpuri2}様々な *Decorator*
    では[、]{.chpule2}[
    ]{.chpuri2}オブジェクトの振る舞いの拡張をする時[、]{.chpule2}[
    ]{.chpuri2}基底インターフェースとの一貫性を保つ必要があります[。]{.chpule2}[
    ]{.chpuri2}さらに[、]{.chpule2}[
    ]{.chpuri2}デコレーターはリクエストの流れを断ち切ることは許されていません[。]{.chpule2}[
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

-   [Decorator](decorator.html)
    がオブジェクトの表層を変えるのに対し[、]{.chpule2}[
    ]{.chpuri2}[Strategy](strategy.html) は中身を変えます[。]{.chpule2}[
    ]{.chpuri2}

-   [Decorator](decorator.html) と [Proxy](proxy.html)
    とは似たような構造をしていますが[、]{.chpule2}[
    ]{.chpuri2}その意図は非常に異なります[。]{.chpule2}[
    ]{.chpuri2}どちらのパターンも[、]{.chpule2}[ ]{.chpuri2}合成[
    ]{.chpule1}[（]{.chpuri1}コンポジション[）]{.chpule2}[
    ]{.chpuri2}の原則に基づいており[、]{.chpule2}[
    ]{.chpuri2}あるオブジェクトが仕事の一部を別のオブジェクトに委任します[。]{.chpule2}[
    ]{.chpuri2}違いは[、]{.chpule2}[ ]{.chpuri2}*Proxy*
    は通常そのサービス・オブジェクトのライフサイクルの管理を行うのに対し[、]{.chpule2}[
    ]{.chpuri2}*Decorators* では[、]{.chpule2}[
    ]{.chpuri2}常にクライアントが管理するという点です[。]{.chpule2}[
    ]{.chpuri2}
:::

::: {.section .implementations}
##  コード例 {#implementations}

[![Decorator を C#
で](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](decorator/csharp/example.html "Decorator を C# で"){.prog-lang-link}
[![Decorator を C++
で](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](decorator/cpp/example.html "Decorator を C++ で"){.prog-lang-link}
[![Decorator を Go
で](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](decorator/go/example.html "Decorator を Go で"){.prog-lang-link}
[![Decorator を Java
で](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](decorator/java/example.html "Decorator を Java で"){.prog-lang-link}
[![Decorator を PHP
で](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](decorator/php/example.html "Decorator を PHP で"){.prog-lang-link}
[![Decorator を Python
で](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](decorator/python/example.html "Decorator を Python で"){.prog-lang-link}
[![Decorator を Ruby
で](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](decorator/ruby/example.html "Decorator を Ruby で"){.prog-lang-link}
[![Decorator を Rust
で](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](decorator/rust/example.html "Decorator を Rust で"){.prog-lang-link}
[![Decorator を Swift
で](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](decorator/swift/example.html "Decorator を Swift で"){.prog-lang-link}
[![Decorator を TypeScript
で](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](decorator/typescript/example.html "Decorator を TypeScript で"){.prog-lang-link}
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

[Facade []{.fa .fa-arrow-right}](facade.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### 戻る

[[]{.fa .fa-arrow-left} Composite](composite.html){.btn .btn-default
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
