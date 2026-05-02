:::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} /
[デザインパターン](../design-patterns.html){.type} /
[構造に関するパターン](structural-patterns.html){.type}
:::

# Flyweight {#flyweight .title}

::: aka
別名：[Cache、]{style="display:inline-block"}[フライウィト、]{style="display:inline-block"}[キャッシュ]{style="display:inline-block"}
:::

::: {.section .intent}
##  一言でいうと {#intent}

**Flyweight**[ ]{.chpule1}[（]{.chpuri1}フライウェイト[、]{.chpule2}[
]{.chpuri2}フライ級[）]{.chpule2}[ ]{.chpuri2}は[、]{.chpule2}[
]{.chpuri2}構造に関するデザインパターンの一つで[、]{.chpule2}[
]{.chpuri2}複数のオブジェクト間で共通する部分を各自で持つ代わりに共有することによって[、]{.chpule2}[
]{.chpuri2}利用可能な
RAM により多くのオブジェクトを収められるようにします[。]{.chpule2}[
]{.chpuri2}

<figure class="image">
<img
src="../../images/patterns/content/flyweight/flyweight7be2.png?id=e34fbacb847dd609b5e68aaf252c4db0"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/flyweight/flyweight.png?id=e34fbacb847dd609b5e68aaf252c4db0 640w,/images/patterns/content/flyweight/flyweight-1.5x.png?id=b32df2297473b8a7577e1900e57325ac 960w,/images/patterns/content/flyweight/flyweight-2x.png?id=6a8f17d9550c75c3d648a605c4d31b45 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Flyweight のデザインパターン" />
</figure>
:::

::: {.section .problem}
##  問題 {#problem}

長い仕事の後の娯楽として[、]{.chpule2}[
]{.chpuri2}簡単なビデオゲームを作ることにしました[。]{.chpule2}[
]{.chpuri2}プレーヤーは地図を動き回ってお互い撃ち合います[。]{.chpule2}[
]{.chpuri2}とてもリアルに見えるパーティクル・システムを実装し[、]{.chpule2}[
]{.chpuri2}このゲームのセールスポイントの一つとすることにしました[。]{.chpule2}[
]{.chpuri2}大量の弾丸[、]{.chpule2}[ ]{.chpuri2}ミサイル[、]{.chpule2}[
]{.chpuri2}爆発の破片が[、]{.chpule2}[
]{.chpuri2}マップ上を飛び回り[、]{.chpule2}[
]{.chpuri2}プレーヤーがスリルを味わいます[。]{.chpule2}[ ]{.chpuri2}

プログラミング完成後[、]{.chpule2}[
]{.chpuri2}最後のコミットをプッシュしてゲームをビルドし[、]{.chpule2}[
]{.chpuri2}テストドライブしてもらうために友達に送りました[。]{.chpule2}[
]{.chpuri2}自分のマシン上では完璧に動いていたゲームですが[、]{.chpule2}[
]{.chpuri2}友達は長時間プレーすることができませんでした[。]{.chpule2}[
]{.chpuri2}彼のコンピューター上では[、]{.chpule2}[
]{.chpuri2}ゲームは開始後数分でクラッシュが続きました[。]{.chpule2}[
]{.chpuri2}デバッグ用ログを数時間かけて調べた結果[、]{.chpule2}[
]{.chpuri2}RAM
の容量不足が原因でゲームがクラッシュしたことがわかりました[。]{.chpule2}[
]{.chpuri2}友達のマシンは[、]{.chpule2}[
]{.chpuri2}自分のコンピューターよりもかなり非力なものであることが判明しました[。]{.chpule2}[
]{.chpuri2}彼のマシン上で短時間で問題が起きたのは[、]{.chpule2}[
]{.chpuri2}このためです[。]{.chpule2}[ ]{.chpuri2}

実際の問題は[、]{.chpule2}[
]{.chpuri2}パーティクル・システムに関連していました[。]{.chpule2}[
]{.chpuri2}弾丸[、]{.chpule2}[ ]{.chpuri2}ミサイル[、]{.chpule2}[
]{.chpuri2}破片などのパーティクルは[、]{.chpule2}[
]{.chpuri2}それぞれ大量のデータを含む個別のオブジェクトによって表現されていました[。]{.chpule2}[
]{.chpuri2}ある時点で[、]{.chpule2}[
]{.chpuri2}プレーヤーの画面上の殺戮がクライマックスに達した時[、]{.chpule2}[
]{.chpuri2}新しく作成されたパーティクルが残りの RAM
に収まりきらず[、]{.chpule2}[
]{.chpuri2}そのためプログラムはクラッシュしました[。]{.chpule2}[
]{.chpuri2}

<figure class="image">
<img
src="../../images/patterns/diagrams/flyweight/problem-jad517.png?id=f7f63333df57a561d8e00fe8292697fa"
style="aspect-ratio: 2.54;"
srcset="/images/patterns/diagrams/flyweight/problem-ja.png?id=f7f63333df57a561d8e00fe8292697fa 660w,/images/patterns/diagrams/flyweight/problem-ja-1.5x.png?id=dd87efaeb339256db0e69631236f4523 990w,/images/patterns/diagrams/flyweight/problem-ja-2x.png?id=d527997b0cdd910b7b431d643eb83e83 1320w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="660"
alt="Flyweight パターンの問題" />
</figure>
:::

::: {.section .solution}
##  解決策 {#solution}

`Particle` クラスをよく調べてみると[、]{.chpule2}[ ]{.chpuri2}color と
sprite
フィールドが他のフィールドよりもかなり大量にメモリーを消費していることにお気づきかもしれません[。]{.chpule2}[
]{.chpuri2}さらに悪いことには[、]{.chpule2}[
]{.chpuri2}この二つのフィールドには[、]{.chpule2}[
]{.chpuri2}すべてのパーティクルにわたってほぼ同じデータが格納されています[。]{.chpule2}[
]{.chpuri2}たとえば[、]{.chpule2}[
]{.chpuri2}すべての弾丸は同じ色とスプライトを持っています[。]{.chpule2}[
]{.chpuri2}

<figure class="image">
<img
src="../../images/patterns/diagrams/flyweight/solution1-jae96e.png?id=cfaf7847de4517d73632b29db1f893d4"
style="aspect-ratio: 2.06;"
srcset="/images/patterns/diagrams/flyweight/solution1-ja.png?id=cfaf7847de4517d73632b29db1f893d4 640w,/images/patterns/diagrams/flyweight/solution1-ja-1.5x.png?id=9376a83e079feb3898107ad262b7ab18 960w,/images/patterns/diagrams/flyweight/solution1-ja-2x.png?id=61c3dd5729bb72e73ca95cc29f75d256 1280w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="640"
alt="Flyweight パターン解決策" />
</figure>

座標[、]{.chpule2}[ ]{.chpuri2}移動ベクトル[、]{.chpule2}[
]{.chpuri2}速度などのパーティクルの他の状態は[、]{.chpule2}[
]{.chpuri2}それぞれのパーティクルに固有です[。]{.chpule2}[
]{.chpuri2}結局のところ[、]{.chpule2}[
]{.chpuri2}これらのフィールドの値は時間とともに変化します[。]{.chpule2}[
]{.chpuri2}このデータは[、]{.chpule2}[
]{.chpuri2}パーティクルが存在の常に変化する状況を表しているわけですが[、]{.chpule2}[
]{.chpuri2}それぞれのパーティクルの色とスプライトは一定の値を保ちます[。]{.chpule2}[
]{.chpuri2}

このようなオブジェクト内の不変のデータは[、]{.chpule2}[
]{.chpuri2}通常*[内]{.cjk}[因]{.cjk}[的]{.cjk}[状]{.cjk}[態]{.cjk}*[
]{.chpule1}[（]{.chpuri1}intrinsic state[）]{.chpule2}[
]{.chpuri2}と呼ばれます[。]{.chpule2}[
]{.chpuri2}これはオブジェクトの中に存在し[、]{.chpule2}[
]{.chpuri2}他のオブジェクトはこれを読むだけで[、]{.chpule2}[
]{.chpuri2}変更することはできません[。]{.chpule2}[
]{.chpuri2}オブジェクトの残りの状態は[、]{.chpule2}[
]{.chpuri2}しばしば他のオブジェクトによって[
]{.chpule1}[「]{.chpuri1}外側から[」]{.chpule2}[
]{.chpuri2}変更され[、]{.chpule2}[
]{.chpuri2}*[外]{.cjk}[因]{.cjk}[的]{.cjk}[状]{.cjk}[態]{.cjk}*[
]{.chpule1}[（]{.chpuri1}extrinsic state[）]{.chpule2}[
]{.chpuri2}と呼ばれます[。]{.chpule2}[ ]{.chpuri2}

Flyweight パターンに従うと[、]{.chpule2}[
]{.chpuri2}外因的状態をオブジェクト内部に持つことはやめるべきです[。]{.chpule2}[
]{.chpuri2}代わりに[、]{.chpule2}[ ]{.chpuri2}この状態は[、]{.chpule2}[
]{.chpuri2}依存する特定のメソッドに渡します[。]{.chpule2}[
]{.chpuri2}内因的状態だけをオブジェクトに保管し[、]{.chpule2}[
]{.chpuri2}違った状況で再利用します[。]{.chpule2}[
]{.chpuri2}内因的状態は[、]{.chpule2}[
]{.chpuri2}外因的態に比べて種類が少ないので[、]{.chpule2}[
]{.chpuri2}結果[、]{.chpule2}[
]{.chpuri2}このオブジェクトは少い個数ですみます[。]{.chpule2}[
]{.chpuri2}

<figure class="image">
<img
src="../../images/patterns/diagrams/flyweight/solution3-ja28ca.png?id=d43f341e16e6a355f7188181036b1e09"
style="aspect-ratio: 0.76;"
srcset="/images/patterns/diagrams/flyweight/solution3-ja.png?id=d43f341e16e6a355f7188181036b1e09 424w,/images/patterns/diagrams/flyweight/solution3-ja-1.5x.png?id=f0de44d12f6c1b1ce3b1070515be63da 636w,/images/patterns/diagrams/flyweight/solution3-ja-2x.png?id=f9ccf75d4711c963e2f4d8d9d54baca9 848w"
sizes="(max-width: 720px) 100vw, 424px" loading="lazy" width="424"
alt="Flyweight パターン解決策" />
</figure>

さて[、]{.chpule2}[ ]{.chpuri2}ゲームの話に戻りましょう[。]{.chpule2}[
]{.chpuri2}パーティクル・クラスから外因的状態を抽出したところ[、]{.chpule2}[
]{.chpuri2}ゲームのすべてのパーティクルを表現するためには[、]{.chpule2}[
]{.chpuri2}弾丸[、]{.chpule2}[ ]{.chpuri2}ミサイル[、]{.chpule2}[
]{.chpuri2}破片のたった 3
個のオブジェクトで十分だということがわかったとします[。]{.chpule2}[
]{.chpuri2}もうお気づきかもしれませんが[、]{.chpule2}[
]{.chpuri2}内因的状態を格納するオブジェクトは[、]{.chpule2}[
]{.chpuri2}フライウェイトと呼ばれます[。]{.chpule2}[ ]{.chpuri2}

#### 外因的状態の記憶場所

外因的な状態はどこへ移動すればいいでしょうか[？]{.chpule2}[
]{.chpuri2}どこかのクラスにそれを保管する必要がありますよね[？]{.chpule2}[
]{.chpuri2}ほとんどの場合[、]{.chpule2}[
]{.chpuri2}パターン適用前にオブジェクトが集約されているコンテナのオブジェクトに移動します[。]{.chpule2}[
]{.chpuri2}

我々の例では[、]{.chpule2}[ ]{.chpuri2}主要項目である `Game`
オブジェクトの `particles`
フィールドにすべてのパーティクルを格納します[。]{.chpule2}[
]{.chpuri2}このクラスに外因的状態を移動するため[、]{.chpule2}[
]{.chpuri2}個々のパーティクルの座標[、]{.chpule2}[
]{.chpuri2}ベクトル[、]{.chpule2}[
]{.chpuri2}速度を格納するための配列フィールドをいくつか作成する必要があります[。]{.chpule2}[
]{.chpuri2}しかし[、]{.chpule2}[
]{.chpuri2}それだけではありません[。]{.chpule2}[
]{.chpuri2}それぞれのパーティクルを表す特定のフライウェイトへの参照を格納するために[、]{.chpule2}[
]{.chpuri2}別の配列が必要となります[。]{.chpule2}[
]{.chpuri2}同じインデックスを使用してパーティクルのすべてのデータにアクセスできるように[、]{.chpule2}[
]{.chpuri2}これらの配列は同期されていなければなりません[。]{.chpule2}[
]{.chpuri2}

<figure class="image">
<img
src="../../images/patterns/diagrams/flyweight/solution2-jab368.png?id=ad6e5320cd387e00aff0bbac55bf8e36"
style="aspect-ratio: 1.94;"
srcset="/images/patterns/diagrams/flyweight/solution2-ja.png?id=ad6e5320cd387e00aff0bbac55bf8e36 640w,/images/patterns/diagrams/flyweight/solution2-ja-1.5x.png?id=217fd54c83bdfe65917415438a33abec 960w,/images/patterns/diagrams/flyweight/solution2-ja-2x.png?id=82e9fdb42cd6ce75e06b0e911fe12178 1280w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="640"
alt="Flyweight パターン解決策" />
</figure>

よりエレガントな解決策は[、]{.chpule2}[ ]{.chpuri2}別にコンテキスト[
]{.chpule1}[（]{.chpuri1}状況管理[）]{.chpule2}[
]{.chpuri2}クラスを作り[、]{.chpule2}[
]{.chpuri2}そこにフライウェイト・オブジェクトへの参照と共に外因的状態を格納することです[。]{.chpule2}[
]{.chpuri2}このやり方では[、]{.chpule2}[
]{.chpuri2}コンテナ・クラス内には配列が一つだけあればすみます[。]{.chpule2}[
]{.chpuri2}

えっ[、]{.chpule2}[ ]{.chpuri2}ちょっと待って[！]{.chpule2}[
]{.chpuri2}最初と同じくらい多数のコンテキスト・オブジェクトを持つ必要があるんじゃない[？]{.chpule2}[
]{.chpuri2}技術的にはそうです[。]{.chpule2}[
]{.chpuri2}しかし実の所[、]{.chpule2}[
]{.chpuri2}これらのオブジェクトは前よりずっと小さいのです[。]{.chpule2}[
]{.chpuri2}最もメモリを消費するフィールドは[、]{.chpule2}[
]{.chpuri2}わずか数個しかないフライウェイト・オブジェクトに移動されました[。]{.chpule2}[
]{.chpuri2}1000 個の小さなコンテクスト・オブジェクトが 1000
個のデータのコピーを保管する代わりに[、]{.chpule2}[ ]{.chpuri2}重い[
]{.chpule1}[「]{.chpuri1}フライウェイト[」]{.chpule2}[
]{.chpuri2}のオブジェクト 1 個を再利用できます[。]{.chpule2}[
]{.chpuri2}

#### フライウェイトと不変性

同じフライウェイト・オブジェクトを違ったコンテキストから使うので[、]{.chpule2}[
]{.chpuri2}その状態は変更できないようにする必要があります[。]{.chpule2}[
]{.chpuri2}フライウェイトは[、]{.chpule2}[
]{.chpuri2}コンストラクターのパラメーターを介して一度だけ状態の初期化を行います[。]{.chpule2}[
]{.chpuri2}よそのオブジェクトに setter
や公開フィールドを見せないようにします[。]{.chpule2}[ ]{.chpuri2}

#### フライウェイト・ファクトリー

様々なフライウェイト・オブジェクトへのアクセスをより便利にするために[、]{.chpule2}[
]{.chpuri2}既存のフライウェイト・オブジェクトのプールを管理するファクトリー・メソッドを作成することもできます[。]{.chpule2}[
]{.chpuri2}このメソッドは[、]{.chpule2}[
]{.chpuri2}クライアントから所望のフライウェイトの内因的状態を受け取り[、]{.chpule2}[
]{.chpuri2}この状態に一致する既存のフライウェイト・オブジェクトを探し[、]{.chpule2}[
]{.chpuri2}見つかったらそれを返します[。]{.chpule2}[
]{.chpuri2}見つからなかった場合は[、]{.chpule2}[
]{.chpuri2}新しいフライウェイト・オブジェクトを作成し[、]{.chpule2}[
]{.chpuri2}プールに追加します[。]{.chpule2}[ ]{.chpuri2}

このメソッドをどこに置くかについては[、]{.chpule2}[
]{.chpuri2}いくつか選択肢があります[。]{.chpule2}[
]{.chpuri2}一番わかりやすい場所は[、]{.chpule2}[
]{.chpuri2}フライウェイトのコンテナです[。]{.chpule2}[
]{.chpuri2}あるいは[、]{.chpule2}[
]{.chpuri2}新しいファクトリー・クラスを作成することもできます[。]{.chpule2}[
]{.chpuri2}または[、]{.chpule2}[
]{.chpuri2}ファクトリー・メソッドを静的[
]{.chpule1}[（]{.chpuri1}static[）]{.chpule2}[
]{.chpuri2}にし[、]{.chpule2}[
]{.chpuri2}それを実際のフライウェイト・クラスの中に置くこともできます[。]{.chpule2}[
]{.chpuri2}
:::

::::: {.section .structure-container}
##  構造 {#structure}

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/flyweight/structure7b00.png?id=c1e7e1748f957a4792822f902bc1d420"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.64;"
srcset="/images/patterns/diagrams/flyweight/structure.png?id=c1e7e1748f957a4792822f902bc1d420 640w,/images/patterns/diagrams/flyweight/structure-1.5x.png?id=34cf08f6c2b09dcd1c521c7512cc52b6 960w,/images/patterns/diagrams/flyweight/structure-2x.png?id=a7c8347421bde16435fc90a706f5dd34 1280w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="640"
alt="Flyweight デザインパターンの構造" /><img
src="../../images/patterns/diagrams/flyweight/structure-indexed086a.png?id=aa490792baa26b04464dacbc1f4a19cd"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.54;"
srcset="/images/patterns/diagrams/flyweight/structure-indexed.png?id=aa490792baa26b04464dacbc1f4a19cd 630w,/images/patterns/diagrams/flyweight/structure-indexed-1.5x.png?id=b22a5eab6aacbbd03c66c34802b240ca 945w,/images/patterns/diagrams/flyweight/structure-indexed-2x.png?id=205e2f7d08b4ac0695f445a9db8989c4 1260w"
sizes="(max-width: 720px) 100vw, 630px" loading="lazy" width="630"
alt="Flyweight デザインパターンの構造" />
</figure>
:::

1.  Flyweight パターンは最適化にすぎません[。]{.chpule2}[
    ]{.chpuri2}適用前[、]{.chpule2}[
    ]{.chpuri2}プログラムに[、]{.chpule2}[
    ]{.chpuri2}メモリ内に大量の同じようなオブジェクトが同時に存在することに起因する
    RAM 消費問題があることを確認してください[。]{.chpule2}[
    ]{.chpuri2}この問題が別の方法では解決できないことを確認してください[。]{.chpule2}[
    ]{.chpuri2}

2.  **フライウェイト**[
    ]{.chpule1}[（]{.chpuri1}Flyweight[）]{.chpule2}[
    ]{.chpuri2}クラスは[、]{.chpule2}[
    ]{.chpuri2}元のオブジェクトの状態のうち[、]{.chpule2}[
    ]{.chpuri2}複数のオブジェクト間で共有できる部分を含んでいます[。]{.chpule2}[
    ]{.chpuri2}同じフライウェイトオブジェクトを多くの異なるコンテキストで使用することができます[。]{.chpule2}[
    ]{.chpuri2}フライウェイト内部に格納されている状態は*[内]{.cjk}[因]{.cjk}[的]{.cjk}*[、]{.chpule2}[
    ]{.chpuri2}フライウェイトのメソッドに渡される状態は*[外]{.cjk}[因]{.cjk}[的]{.cjk}*と呼ばれます[。]{.chpule2}[
    ]{.chpuri2}

3.  **コンテキスト**[ ]{.chpule1}[（]{.chpuri1}Context[）]{.chpule2}[
    ]{.chpuri2}クラスには[、]{.chpule2}[
    ]{.chpuri2}すべての元のオブジェクトで一意の[、]{.chpule2}[
    ]{.chpuri2}外因的状態が含まれています[。]{.chpule2}[
    ]{.chpuri2}コンテキストは[、]{.chpule2}[
    ]{.chpuri2}フライウェイト・オブジェクトのいずれかと組になることによって[、]{.chpule2}[
    ]{.chpuri2}元のオブジェクトの状態を完全に表現します[。]{.chpule2}[
    ]{.chpuri2}

4.  通常[、]{.chpule2}[
    ]{.chpuri2}元のオブジェクトの振る舞いは[、]{.chpule2}[
    ]{.chpuri2}フライウェイト・クラスに残ります[。]{.chpule2}[
    ]{.chpuri2}この場合[、]{.chpule2}[
    ]{.chpuri2}フライウェイトのメソッドを呼び出す際に[、]{.chpule2}[
    ]{.chpuri2}メソッドのパラメーターに外因的状態に関する適切な情報を渡す必要があります[。]{.chpule2}[
    ]{.chpuri2}あるいは[、]{.chpule2}[
    ]{.chpuri2}この振る舞いはコンテキスト・クラスに移動することもできます[。]{.chpule2}[
    ]{.chpuri2}この場合[、]{.chpule2}[
    ]{.chpuri2}リンクされたフライウェイトは[、]{.chpule2}[
    ]{.chpuri2}単なるデータ・オブジェクトとして使用することになります[。]{.chpule2}[
    ]{.chpuri2}

5.  **クライアント**[ ]{.chpule1}[（]{.chpuri1}Client[）]{.chpule2}[
    ]{.chpuri2}は[、]{.chpule2}[
    ]{.chpuri2}フライウェイトの外因的状態を算出するか保存します[。]{.chpule2}[
    ]{.chpuri2}クライアントの視点からすると[、]{.chpule2}[
    ]{.chpuri2}フライウェイトは[、]{.chpule2}[
    ]{.chpuri2}コンテキスト・データをメソッドのパラメーターに渡すことで実行時に設定可能なテンプレート・オブジェクトです[。]{.chpule2}[
    ]{.chpuri2}

6.  **フライウェイト・ファクトリー**[ ]{.chpule1}[（]{.chpuri1}Flyweight
    Factory[）]{.chpule2}[ ]{.chpuri2}は[、]{.chpule2}[
    ]{.chpuri2}既存のフライウェイトのプールを管理します[。]{.chpule2}[
    ]{.chpuri2}ファクトリーがあるため[、]{.chpule2}[
    ]{.chpuri2}クライアントはフライウェイトを直接作成しません[。]{.chpule2}[
    ]{.chpuri2}代わりに[、]{.chpule2}[
    ]{.chpuri2}ファクトリーを呼び出し[、]{.chpule2}[
    ]{.chpuri2}望むフライウェイトの内因的状態を指定する情報を渡します[。]{.chpule2}[
    ]{.chpuri2}ファクトリーは[、]{.chpule2}[
    ]{.chpuri2}以前に作成されたフライウェイトを見渡して[、]{.chpule2}[
    ]{.chpuri2}検索条件に一致する既存のものを返すか[、]{.chpule2}[
    ]{.chpuri2}何も見つからない場合は新しいものを作成します[。]{.chpule2}[
    ]{.chpuri2}
::::
:::::

::: {.section .pseudocode}
##  擬似コード {#pseudocode}

この例では[、]{.chpule2}[ ]{.chpuri2}**Flyweight**
パターンを適用して[、]{.chpule2}[
]{.chpuri2}何百万ものツリー・オブジェクトをキャンバスに描画する際のメモリー使用量を削減します[。]{.chpule2}[
]{.chpuri2}

<figure class="image">
<img
src="../../images/patterns/diagrams/flyweight/example5053.png?id=0818d078c1a79f373e96397f37b7ee06"
style="aspect-ratio: 1.54;"
srcset="/images/patterns/diagrams/flyweight/example.png?id=0818d078c1a79f373e96397f37b7ee06 540w,/images/patterns/diagrams/flyweight/example-1.5x.png?id=449fa5906e277c134870c07e33dd4b62 810w,/images/patterns/diagrams/flyweight/example-2x.png?id=9423640fe3688a64201389b6e7aa1f48 1080w"
sizes="(max-width: 720px) 100vw, 540px" loading="lazy" width="540"
alt="Flyweight パターンの例" />
</figure>

パターンを適用して[、]{.chpule2}[ ]{.chpuri2}主要データである `Tree`
クラスに繰り返し現れる内因的状態を抽出し[、]{.chpule2}[
]{.chpuri2}フライウェイト・クラスの `Tree­Type`
に移動します[。]{.chpule2}[ ]{.chpuri2}

ここでは[、]{.chpule2}[
]{.chpuri2}同じデータを複数のオブジェクトに格納する代わりに[、]{.chpule2}[
]{.chpuri2}ほんの数個のフライウェイト・オブジェクトに格納し[、]{.chpule2}[
]{.chpuri2}コンテキストとして機能する[、]{.chpule2}[ ]{.chpuri2}適切な
`Tree` オブジェクトにリンクします[。]{.chpule2}[
]{.chpuri2}クライアント・コードはフライウェイト・ファクトリーを使用して新しいツリー・オブジェクトを作成します[。]{.chpule2}[
]{.chpuri2}このファクトリーは[、]{.chpule2}[
]{.chpuri2}正しいオブジェクトを探し[、]{.chpule2}[
]{.chpuri2}必要に応じて再利用する複雑な仕組みを隠蔽します[。]{.chpule2}[
]{.chpuri2}

<figure class="code">
<pre class="code" lang="pseudocode"><code>// フライウェイト・クラスはツリーの状態の一部を含む。これらのフィールドに
// は、それぞれのツリーに固有の値を格納する。たとえば、ここにはツリーの座
// 標はみつからない。しかし多くのツリーの間で共有されている質感と色はここ
// に含む。このデータは通常巨大であるため、各ツリー・オブジェクトに保存す
// るとメモリーを大量に無駄にすることになる。その代わりに、質感、色、など
// の繰り返すデータを別個のオブジェクトに抽出し、多数の各ツリー・オブジェ
// クトから参照することができる。
class TreeType is
    field name
    field color
    field texture
    constructor TreeType(name, color, texture) { …… }
    method draw(canvas, x, y) is
        // 1. 指定の種類、色、質感のビットマップを作成。
        // 2. キャンバス上、X と Y の座標にビットマップを描画。

// フライウェイトのファクトリーは、既存のフライウェイトを再利用するか、新
// しいオブジェクトを作成するかを決定。
class TreeFactory is
    static field treeTypes: collection of tree types
    static method getTreeType(name, color, texture) is
        type = treeTypes.find(name, color, texture)
        if (type == null)
            type = new TreeType(name, color, texture)
            treeTypes.add(type)
        return type

// コンテキスト・オブジェクトにはツリー状態の外因的な部分が含まれる。これ
// は二つの整数座標と一つの参照フィールドからできていて、とても小さいため、
// アプリケーションはこれを数十億個作成することも可能。
class Tree is
    field x,y
    field type: TreeType
    constructor Tree(x, y, type) { …… }
    method draw(canvas) is
        type.draw(canvas, this.x, this.y)

// Tree クラスと Forest クラスはフライウェイトのクライアント。これ以上ツ
// リー・クラスを開発する予定がなければ、一緒にしてもよい。
class Forest is
    field trees: collection of Trees

    method plantTree(x, y, name, color, texture) is
        type = TreeFactory.getTreeType(name, color, texture)
        tree = new Tree(x, y, type)
        trees.add(tree)

    method draw(canvas) is
        foreach (tree in trees) do
            tree.draw(canvas)</code></pre>
</figure>
:::

:::::: {.section .applicability-container}
##  適応性 {#applicability}

::::: applicability
::: applicability-problem
Flyweight パターンは[、]{.chpule2}[ ]{.chpuri2}使用可能な RAM
にほとんど収まらりきらない膨大な数のオブジェクトをプログラムがサポートする必要がある場合にのみ使用します[。]{.chpule2}[
]{.chpuri2}
:::

::: applicability-solution
このパターンが有効かどうかは[、]{.chpule2}[
]{.chpuri2}パターンの使用方法と使用場所に大きく依存します[。]{.chpule2}[
]{.chpuri2}以下の場合に最も有効です[：]{.chpule2}[ ]{.chpuri2}

-   アプリケーションが[、]{.chpule2}[
    ]{.chpuri2}膨大な数の類似したオブジェクトを生成する必要がある[。]{.chpule2}[
    ]{.chpuri2}
-   これが対象機器上の利用可能な RAM をすべて枯渇する[。]{.chpule2}[
    ]{.chpuri2}
-   オブジェクトには重複した状態が含まれており[、]{.chpule2}[
    ]{.chpuri2}それを複数オブジェクト間で抽出し共有することが可能[。]{.chpule2}[
    ]{.chpuri2}
:::
:::::
::::::

::: {.section .checklist}
##  実装方法 {#checklist}

1.  フライウェイトになるクラスのフィールドを二つの部分に分割します[。]{.chpule2}[
    ]{.chpuri2}

    -   内因的状態[：]{.chpule2}[
        ]{.chpuri2}多くのオブジェクト間で繰り返し現れる不変のデータを含むフィールド
    -   外因的状態[：]{.chpule2}[
        ]{.chpuri2}オブジェクトに固有のコンテキスト・データを含むフィールド

2.  内因的状態を表すフィールドはクラス内に残し変更不可とします[。]{.chpule2}[
    ]{.chpuri2}コンストラクターの初期値を取る必要があります[。]{.chpule2}[
    ]{.chpuri2}

3.  外因的状態のフィールドを使用するメソッドを一つずつ調べます[。]{.chpule2}[
    ]{.chpuri2}メソッドに新しいパラメーターを導入し[、]{.chpule2}[
    ]{.chpuri2}フィールドの代わりに使用します[。]{.chpule2}[ ]{.chpuri2}

4.  必要に応じて[、]{.chpule2}[
    ]{.chpuri2}フライウェイトのプールを管理するファクトリー・クラスを作成します[。]{.chpule2}[
    ]{.chpuri2}それは[、]{.chpule2}[
    ]{.chpuri2}新しいフライウェイトを作成する前に[、]{.chpule2}[
    ]{.chpuri2}既存のものがないことを確認します[。]{.chpule2}[
    ]{.chpuri2}ファクトリーができたところで[、]{.chpule2}[
    ]{.chpuri2}クライアントは[、]{.chpule2}[
    ]{.chpuri2}ファクトリーを通してのみフライウェイトを要求しなければなりません[。]{.chpule2}[
    ]{.chpuri2}どのようなフライウェイトが欲しいかを知らせるために[、]{.chpule2}[
    ]{.chpuri2}ファクトリーに内因的状態を渡します[。]{.chpule2}[
    ]{.chpuri2}

5.  フライウェイト・オブジェクトのメソッドを呼び出せるようにするため[、]{.chpule2}[
    ]{.chpuri2}クライアントは外因的状態[
    ]{.chpule1}[（]{.chpuri1}コンテキスト[）]{.chpule2}[
    ]{.chpuri2}の値を保管するか計算する必要があります[。]{.chpule2}[
    ]{.chpuri2}これを容易にするため[、]{.chpule2}[
    ]{.chpuri2}外因的状態とフライウェイト参照フィールドを別のコンテキスト・クラスに移動することもできます[。]{.chpule2}[
    ]{.chpuri2}
:::

:::::: {.section .pros-cons}
##  長所と短所 {#pros-cons}

::::: row
::: col-sm-6
-    プログラムに類似オブジェクトが山のようにある状況下で[、]{.chpule2}[
    ]{.chpuri2}RAM を大量に節約可能[。]{.chpule2}[ ]{.chpuri2}
:::

::: col-sm-6
-   
    フライウェイトのメソッドを呼び出すたびにコンテキスト・データの一部を再計算する必要がある場合[、]{.chpule2}[
    ]{.chpuri2}RAM 使用量減少と引き換えに CPU
    使用率増加の可能性[。]{.chpule2}[ ]{.chpuri2}
-    コードが大幅に煩雑化[。]{.chpule2}[
    ]{.chpuri2}新しいチーム・メンバーは[、]{.chpule2}[
    ]{.chpuri2}あるエンティティーの状態がどうして別けられているのか[、]{.chpule2}[
    ]{.chpuri2}といつも疑問に思うことになる[。]{.chpule2}[ ]{.chpuri2}
:::
:::::
::::::

::: {.section .relations}
##  他のパターンとの関係 {#relations}

-   RAM を節約するために[、]{.chpule2}[
    ]{.chpuri2}[Composite](composite.html) ツリーの共有リーフ・ノードを
    [Flyweights](flyweight.html) として実装できます[。]{.chpule2}[
    ]{.chpuri2}

-   [Flyweight](flyweight.html)
    は多くの小さなオブジェクトを作る方法についてですが[、]{.chpule2}[
    ]{.chpuri2}[Facade](facade.html)
    はサブシステム全体を表す単一のオブジェクトを作る方法に関してです[。]{.chpule2}[
    ]{.chpuri2}

-   [Flyweight](flyweight.html) で[、]{.chpule2}[
    ]{.chpuri2}共有状態の全部を一つのフライウェイト・オブジェクトに何らかの方法で削減できた場合[、]{.chpule2}[
    ]{.chpuri2}それは [Singleton](singleton.html)
    に似たものになります[。]{.chpule2}[ ]{.chpuri2}しかし[、]{.chpule2}[
    ]{.chpuri2}この二つのパターンには[、]{.chpule2}[
    ]{.chpuri2}根本的な違いが二箇所あります[。]{.chpule2}[ ]{.chpuri2}

    1.  Singleton のインスタンスは一つだけですが[、]{.chpule2}[
        ]{.chpuri2}*Flyweight* クラスは[、]{.chpule2}[
        ]{.chpuri2}異なる内因的状態を持つ複数のインスタンスがある可能性があります[。]{.chpule2}[
        ]{.chpuri2}
    2.  *Singleton*
        オブジェクトは変更可能かもしれませんが[、]{.chpule2}[
        ]{.chpuri2}Flyweight のオブジェクトは不変です[。]{.chpule2}[
        ]{.chpuri2}
:::

::: {.section .implementations}
##  コード例 {#implementations}

[![Flyweight を C#
で](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](flyweight/csharp/example.html "Flyweight を C# で"){.prog-lang-link}
[![Flyweight を C++
で](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](flyweight/cpp/example.html "Flyweight を C++ で"){.prog-lang-link}
[![Flyweight を Go
で](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](flyweight/go/example.html "Flyweight を Go で"){.prog-lang-link}
[![Flyweight を Java
で](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](flyweight/java/example.html "Flyweight を Java で"){.prog-lang-link}
[![Flyweight を PHP
で](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](flyweight/php/example.html "Flyweight を PHP で"){.prog-lang-link}
[![Flyweight を Python
で](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](flyweight/python/example.html "Flyweight を Python で"){.prog-lang-link}
[![Flyweight を Ruby
で](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](flyweight/ruby/example.html "Flyweight を Ruby で"){.prog-lang-link}
[![Flyweight を Rust
で](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](flyweight/rust/example.html "Flyweight を Rust で"){.prog-lang-link}
[![Flyweight を Swift
で](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](flyweight/swift/example.html "Flyweight を Swift で"){.prog-lang-link}
[![Flyweight を TypeScript
で](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](flyweight/typescript/example.html "Flyweight を TypeScript で"){.prog-lang-link}
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

[Proxy []{.fa .fa-arrow-right}](proxy.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### 戻る

[[]{.fa .fa-arrow-left} Facade](facade.html){.btn .btn-default
rel="prev"}
:::
:::::::::::::::::::::::::::::

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
:::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::
