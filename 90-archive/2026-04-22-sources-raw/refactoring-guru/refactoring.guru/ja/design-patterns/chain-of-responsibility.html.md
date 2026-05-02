::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} /
[デザインパターン](../design-patterns.html){.type} /
[振る舞いに関するパターン](behavioral-patterns.html){.type}
:::

# Chain of Responsibility {#chain-of-responsibility .title}

::: aka
別名：[CoR、]{style="display:inline-block"}[Chain of
Command、]{style="display:inline-block"}[責任のチェーン、]{style="display:inline-block"}[責任の連鎖、]{style="display:inline-block"}[コマンドのチェーン、]{style="display:inline-block"}[コマンドの連鎖]{style="display:inline-block"}
:::

::: {.section .intent}
##  一言でいうと {#intent}

**Chain of Responsibility**[
]{.chpule1}[（]{.chpuri1}責任の連鎖[）]{.chpule2}[
]{.chpuri2}は[、]{.chpule2}[
]{.chpuri2}振る舞いに関するデザインパターンの一つで[、]{.chpule2}[
]{.chpuri2}ハンドラーの連鎖に沿ってリクエストを渡すことができます[。]{.chpule2}[
]{.chpuri2}各ハンドラーは[、]{.chpule2}[
]{.chpuri2}リクエストを受け取ると[、]{.chpule2}[
]{.chpuri2}リクエストを処理するか[、]{.chpule2}[
]{.chpuri2}連鎖内の次のハンドラーに渡すかを決めます[。]{.chpule2}[
]{.chpuri2}

<figure class="image">
<img
src="../../images/patterns/content/chain-of-responsibility/chain-of-responsibilityf2d1.png?id=56c10d0dc712546cc283cfb3fb463458"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/chain-of-responsibility/chain-of-responsibility.png?id=56c10d0dc712546cc283cfb3fb463458 640w,/images/patterns/content/chain-of-responsibility/chain-of-responsibility-1.5x.png?id=770c03ad168fa797ec8ed4556ddf0b5c 960w,/images/patterns/content/chain-of-responsibility/chain-of-responsibility-2x.png?id=cc104b0a00a410f37fb39da80f392b88 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Chain of Responsibility デザインパターン" />
</figure>
:::

::: {.section .problem}
##  問題 {#problem}

オンライン注文システムの開発に取り組んでいるところを想像してみてください[。]{.chpule2}[
]{.chpuri2}認証されたユーザーのみが注文を作成できるように[、]{.chpule2}[
]{.chpuri2}システムへのアクセスを制限したいとします[。]{.chpule2}[
]{.chpuri2}また[、]{.chpule2}[
]{.chpuri2}管理者権限を持つユーザーは[、]{.chpule2}[
]{.chpuri2}すべての注文へ完全にアクセスできる必要があります[。]{.chpule2}[
]{.chpuri2}

少し計画した後[、]{.chpule2}[
]{.chpuri2}これらのチェックは順番に実行されなければならないことに気づきました[。]{.chpule2}[
]{.chpuri2}アプリケーションが[、]{.chpule2}[
]{.chpuri2}ユーザーの資格情報を含むリクエストを受信するたびに[、]{.chpule2}[
]{.chpuri2}システムへのユーザー認証を試みようとすることは可能です[。]{.chpule2}[
]{.chpuri2}しかし[、]{.chpule2}[
]{.chpuri2}それらの資格情報が正しくなく[、]{.chpule2}[
]{.chpuri2}認証に失敗した場合[、]{.chpule2}[
]{.chpuri2}他のチェックを続行する理由はありません[。]{.chpule2}[
]{.chpuri2}

<figure class="image">
<img
src="../../images/patterns/diagrams/chain-of-responsibility/problem1-ja7c2c.png?id=a63d98291f4c4238182125f20adfe2e5"
style="aspect-ratio: 2.5;"
srcset="/images/patterns/diagrams/chain-of-responsibility/problem1-ja.png?id=a63d98291f4c4238182125f20adfe2e5 600w,/images/patterns/diagrams/chain-of-responsibility/problem1-ja-1.5x.png?id=ccf7dea547b4b6d9ec57d2fba5bf3ba1 900w,/images/patterns/diagrams/chain-of-responsibility/problem1-ja-2x.png?id=7294ba592650f40e35893970189b4dcb 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Chain of Responsibility によって解決できる問題" />
<figcaption><p>リクエストは<span class="chpule2">、</span><span
class="chpuri2"> </span>注文システムに到達する前に<span
class="chpule2">、</span><span class="chpuri2">
</span>一連のチェックを通過しなければならない<span
class="chpule2">。</span><span class="chpuri2"> </span></p></figcaption>
</figure>

次の数か月間をかけて[、]{.chpule2}[
]{.chpuri2}さらに以下のような一連のチェックを実装しました[。]{.chpule2}[
]{.chpuri2}

-   同僚の一人が[、]{.chpule2}[
    ]{.chpuri2}生データを直接注文システムに渡すのは危険だと言いました[。]{.chpule2}[
    ]{.chpuri2}リクエスト中のデータをサニタイズするための副次的検証ステップを追加しました[。]{.chpule2}[
    ]{.chpuri2}

-   その後[、]{.chpule2}[
    ]{.chpuri2}システムが力づくのパスワード破り攻撃に対して脆弱であることに誰かが気づきました[。]{.chpule2}[
    ]{.chpuri2}これに対処するために[、]{.chpule2}[ ]{.chpuri2}同じ IP
    アドレスからのリクエストが繰り返し失敗したことを検知するチェックを速やかに追加しました[。]{.chpule2}[
    ]{.chpuri2}

-   同じデータを含むリクエストが繰り返し来た場合[、]{.chpule2}[
    ]{.chpuri2}キャッシュした結果を返せば[、]{.chpule2}[
    ]{.chpuri2}システムのスピードの向上が可能なのではないかと[、]{.chpule2}[
    ]{.chpuri2}別の誰かが提言しました[。]{.chpule2}[
    ]{.chpuri2}そのようなわけで[、]{.chpule2}[
    ]{.chpuri2}適切なキャッシュした応答がない場合にのみ[、]{.chpule2}[
    ]{.chpuri2}リクエストをシステムに通すチェックを追加しました[。]{.chpule2}[
    ]{.chpuri2}

<figure class="image">
<img
src="../../images/patterns/diagrams/chain-of-responsibility/problem2-ja54af.png?id=e47fd6c7fe74cb924a577c6ffb03fff0"
style="aspect-ratio: 1.65;"
srcset="/images/patterns/diagrams/chain-of-responsibility/problem2-ja.png?id=e47fd6c7fe74cb924a577c6ffb03fff0 610w,/images/patterns/diagrams/chain-of-responsibility/problem2-ja-1.5x.png?id=64cc040aa125c62778cbf452fb502303 915w,/images/patterns/diagrams/chain-of-responsibility/problem2-ja-2x.png?id=63c9c359a7482f9a2ff530d4fe3717f7 1220w"
sizes="(max-width: 720px) 100vw, 610px" loading="lazy" width="610"
alt="新規チェック追加のたびに、コードは巨大でゴチャゴチャして醜くなりました。" />
<figcaption><p>コードが大きくなるにつれ<span
class="chpule2">、</span><span class="chpuri2">
</span>それはゴチャゴチャしてきた<span class="chpule2">。</span><span
class="chpuri2"> </span></p></figcaption>
</figure>

元々あまりきれいではなかったチェックのコードは[、]{.chpule2}[
]{.chpuri2}新機能を追加するたびに膨張しました[。]{.chpule2}[
]{.chpuri2}あるチェック項目の変更が[、]{.chpule2}[
]{.chpuri2}他のチェックに影響を与えることもありました[。]{.chpule2}[
]{.chpuri2}さらに悪いことに[、]{.chpule2}[
]{.chpuri2}システムの他のコンポーネントを守るためにチェックを再利用しようとすると[、]{.chpule2}[
]{.chpuri2}それらのコンポーネントはいくつかの[、]{.chpule2}[
]{.chpuri2}しかし全部ではないチェックを必要とするため[、]{.chpule2}[
]{.chpuri2}コードの一部を重複しなければなりませんでした[。]{.chpule2}[
]{.chpuri2}

システムを理解するのが非常に難しくなり[、]{.chpule2}[
]{.chpuri2}保守コストが上がりました[。]{.chpule2}[
]{.chpuri2}コードでしばらく悪戦苦闘した後のある日[、]{.chpule2}[
]{.chpuri2}全体をリファクタリングすることに決めました[。]{.chpule2}[
]{.chpuri2}
:::

::: {.section .solution}
##  解決策 {#solution}

他の多くの振る舞いに関するデザインパターンと同様[、]{.chpule2}[
]{.chpuri2}**Chain of Responsibility** の根幹は[、]{.chpule2}[
]{.chpuri2}特定の振る舞いを
*[ハ]{.cjk}[ン]{.cjk}[ド]{.cjk}[ラ]{.cjk}[ー]{.cjk}*と呼ばれる独立したオブジェクトに転換することです[。]{.chpule2}[
]{.chpuri2}我々の場合[、]{.chpule2}[
]{.chpuri2}各チェックを[、]{.chpule2}[
]{.chpuri2}独自のクラスのチェックを実行するメソッドに抽出する必要があります[。]{.chpule2}[
]{.chpuri2}リクエストは[、]{.chpule2}[
]{.chpuri2}そのデータとともにこのメソッドに引数として渡されます[。]{.chpule2}[
]{.chpuri2}

パターンに従うと[、]{.chpule2}[
]{.chpuri2}これらのハンドラーはリンクされて連鎖となります[。]{.chpule2}[
]{.chpuri2}リンクされた各ハンドラーには[、]{.chpule2}[
]{.chpuri2}連鎖内の次のハンドラーへの参照を格納するためのフィールドがあります[。]{.chpule2}[
]{.chpuri2}リクエストの処理に加えて[、]{.chpule2}[
]{.chpuri2}ハンドラーはリクエストを連鎖にそって次のハンドラーに渡します[。]{.chpule2}[
]{.chpuri2}リクエストは[、]{.chpule2}[
]{.chpuri2}すべてのハンドラーがそれを処理する機会を得るまで[、]{.chpule2}[
]{.chpuri2}連鎖に沿って移動して行きます[。]{.chpule2}[ ]{.chpuri2}

そして一番の特長[：]{.chpule2}[
]{.chpuri2}ハンドラーはリクエストを連鎖上で次に渡さないことを決定し[、]{.chpule2}[
]{.chpuri2}結果としてそれ以降の処理を停止することができます[。]{.chpule2}[
]{.chpuri2}

注文システムの例では[、]{.chpule2}[
]{.chpuri2}ハンドラーが一つずつ処理を行い[、]{.chpule2}[
]{.chpuri2}リクエストを連鎖の先に渡すかどうかを決めます[。]{.chpule2}[
]{.chpuri2}リクエストに適切なデータが含まれていると仮定して[、]{.chpule2}[
]{.chpuri2}すべてのハンドラーは[、]{.chpule2}[
]{.chpuri2}その主要任務を実行することができます[。]{.chpule2}[
]{.chpuri2}それは[、]{.chpule2}[
]{.chpuri2}認証チェックだったりキャッシングかもしれません[。]{.chpule2}[
]{.chpuri2}

<figure class="image">
<img
src="../../images/patterns/diagrams/chain-of-responsibility/solution1-ja0899.png?id=2fdecacf488ed7f11d2f124f0230fe49"
style="aspect-ratio: 4;"
srcset="/images/patterns/diagrams/chain-of-responsibility/solution1-ja.png?id=2fdecacf488ed7f11d2f124f0230fe49 640w,/images/patterns/diagrams/chain-of-responsibility/solution1-ja-1.5x.png?id=0c25cf1f7236156327c46ffb35b9fc0e 960w,/images/patterns/diagrams/chain-of-responsibility/solution1-ja-2x.png?id=cc9b0e82d27964c273e6b82acdae7518 1280w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="640"
alt="ハンドラーは、列をなして連鎖を作る。" />
<figcaption><p>ハンドラーは<span class="chpule2">、</span><span
class="chpuri2"> </span>列をなして連鎖を作る<span
class="chpule2">。</span><span class="chpuri2"> </span></p></figcaption>
</figure>

しかし[、]{.chpule2}[ ]{.chpuri2}少々異なった[
]{.chpule1}[（]{.chpuri1}そしてもう少し規範に沿った[）]{.chpule2}[
]{.chpuri2}やり方があります[。]{.chpule2}[
]{.chpuri2}ハンドラーはリクエストを受け取るとそれを自分で処理できるかどうかを決めます[。]{.chpule2}[
]{.chpuri2}もし処理可能であれば[、]{.chpule2}[
]{.chpuri2}これ以上リクエストを先に渡しません[。]{.chpule2}[
]{.chpuri2}ですから[、]{.chpule2}[
]{.chpuri2}リクエストは[、]{.chpule2}[
]{.chpuri2}一つのハンドラーで処理されるか[、]{.chpule2}[
]{.chpuri2}まったくされないかのどちらかです[。]{.chpule2}[
]{.chpuri2}このやり方は[、]{.chpule2}[
]{.chpuri2}グラフィカル・ユーザー・インターフェース内上の積み重なった要素のイベント処理で非常に一般的です[。]{.chpule2}[
]{.chpuri2}

たとえば[、]{.chpule2}[
]{.chpuri2}ユーザーがボタンをクリックすると[、]{.chpule2}[
]{.chpuri2}ボタンから始まる GUI
要素の連鎖を通してイベントが伝播し[[、]{.ls-1}]{.chpule2}[ ]{.chpuri2}​[
]{.chpule1}[（]{.chpuri1}フォームやパネルのような[）]{.chpule2}[
]{.chpuri2}コンテナに行き[、]{.chpule2}[
]{.chpuri2}最後にメインのアプリケーション・ウィンドウに到達します[。]{.chpule2}[
]{.chpuri2}イベントは連鎖内で最初に処理可能な要素によって処理されます[。]{.chpule2}[
]{.chpuri2}この例のもう一つ注目すべき点は[、]{.chpule2}[
]{.chpuri2}連鎖は常にオブジェクト・ツリーから抽出できる[、]{.chpule2}[
]{.chpuri2}ということです[。]{.chpule2}[ ]{.chpuri2}

<figure class="image">
<img
src="../../images/patterns/diagrams/chain-of-responsibility/solution2-jab9d8.png?id=824d0e9706459ad965e47c84f032aaf7"
style="aspect-ratio: 1.73;"
srcset="/images/patterns/diagrams/chain-of-responsibility/solution2-ja.png?id=824d0e9706459ad965e47c84f032aaf7 520w,/images/patterns/diagrams/chain-of-responsibility/solution2-ja-1.5x.png?id=6e6c3ef45a4105cd00d4fe0b25f25cd8 780w,/images/patterns/diagrams/chain-of-responsibility/solution2-ja-2x.png?id=f8f1c3caa430564e730e2925fd2697c0 1040w"
sizes="(max-width: 720px) 100vw, 520px" loading="lazy" width="520"
alt="連鎖はオブジェクト・ツリーの枝から形成可能。" />
<figcaption><p>連鎖はオブジェクト・ツリーの枝から形成可能<span
class="chpule2">。</span><span class="chpuri2"> </span></p></figcaption>
</figure>

すべてのハンドラー・クラスが同じインターフェースを実装するということが大変重要です[。]{.chpule2}[
]{.chpuri2}それぞれの具象ハンドラーは[、]{.chpule2}[
]{.chpuri2}次のハンドラーが[、]{.chpule2}[ ]{.chpuri2}`execute`
メソッドを持っている[、]{.chpule2}[
]{.chpuri2}ということだけ気にかけています[。]{.chpule2}[
]{.chpuri2}これにより[、]{.chpule2}[
]{.chpuri2}コードを具象クラスに結合することなく[、]{.chpule2}[
]{.chpuri2}様々なハンドラーを使用した連鎖を実行時に構成することができます[。]{.chpule2}[
]{.chpuri2}
:::

::: {.section .analogy}
##  現実世界でのたとえ {#analogy}

<figure class="image">
<img
src="../../images/patterns/content/chain-of-responsibility/chain-of-responsibility-comic-1-ja6f96.png?id=898cdf5c391ee1420776fef451e8f538"
style="aspect-ratio: 2;"
srcset="/images/patterns/content/chain-of-responsibility/chain-of-responsibility-comic-1-ja.png?id=898cdf5c391ee1420776fef451e8f538 600w,/images/patterns/content/chain-of-responsibility/chain-of-responsibility-comic-1-ja-1.5x.png?id=a4e61b3c25ff96167e699a0c9f19761d 900w,/images/patterns/content/chain-of-responsibility/chain-of-responsibility-comic-1-ja-2x.png?id=5737f12d280f217e22f5bca078adb183 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="技術サポートと話すのは大変" />
<figcaption><p>技術サポートへの通話は<span
class="chpule2">、</span><span class="chpuri2">
</span>複数の担当者に回されるかもしれない<span
class="chpule2">。</span><span class="chpuri2"> </span></p></figcaption>
</figure>

新しい何らかのハードウェア部品を購入し[、]{.chpule2}[
]{.chpuri2}コンピュータに取り付けたとします[。]{.chpule2}[
]{.chpuri2}あなたはオタクなので[、]{.chpule2}[
]{.chpuri2}コンピュータにはいくつかのオペレーティングシステムがインストールされています[。]{.chpule2}[
]{.chpuri2}ハードウェアがすべての OS
でサポートされているかどうかを確認するため[、]{.chpule2}[
]{.chpuri2}全部ブートしようと試みます[。]{.chpule2}[ ]{.chpuri2}Windows
は自動的にハードウェアを検出し[、]{.chpule2}[
]{.chpuri2}有効化しました[。]{.chpule2}[
]{.chpuri2}ところが[、]{.chpule2}[ ]{.chpuri2}愛する Linuxは
新しいハードウェアと一緒に動くことを拒否します[。]{.chpule2}[
]{.chpuri2}あまり希望はもてませんが[、]{.chpule2}[
]{.chpuri2}ダメモトで[、]{.chpule2}[
]{.chpuri2}箱に書いてある技術サポートの番号に電話してみることにしました[。]{.chpule2}[
]{.chpuri2}

最初に耳にするのは[、]{.chpule2}[
]{.chpuri2}音声自動応答装置のロボットの声です[。]{.chpule2}[
]{.chpuri2}様々な問題に対する 9
つの一般的な解決策を提示しますが[、]{.chpule2}[
]{.chpuri2}どれも自分の事例にあてはまりません[。]{.chpule2}[
]{.chpuri2}しばらくすると[、]{.chpule2}[
]{.chpuri2}ロボットは人間の担当者に接続します[。]{.chpule2}[ ]{.chpuri2}

残念ながら[、]{.chpule2}[
]{.chpuri2}担当者はちっとも役に立つ提言ができません[。]{.chpule2}[
]{.chpuri2}マニュアルに書いてあることの一部を引用し続け[、]{.chpule2}[
]{.chpuri2}あなたの意見に耳を傾けようとしません[[。]{.ls-1}]{.chpule2}[
]{.chpuri2}​[
]{.chpule1}[「]{.chpuri1}コンピューターの電源を一旦オフにしてからオンにしてみましたか？[」]{.chpule2}[
]{.chpuri2}というセリフを 10 回聞いた後で[、]{.chpule2}[
]{.chpuri2}適切なエンジニアと話をしたいと要求します[。]{.chpule2}[
]{.chpuri2}

最終的に[、]{.chpule2}[ ]{.chpuri2}その担当者は[、]{.chpule2}[
]{.chpuri2}エンジニアの一人に内線転送します[。]{.chpule2}[
]{.chpuri2}雑居ビルの暗い地下室の孤独なサーバー・ルームに座りながら[、]{.chpule2}[
]{.chpuri2}生きた人間との会話をずっと待ち望んでいたエンジニアは[、]{.chpule2}[
]{.chpuri2}新しいハードウェアに適切なドライバーをどこからダウンロードし[、]{.chpule2}[
]{.chpuri2}どう Linux
にインストールすればいいかを教えてくれました[。]{.chpule2}[
]{.chpuri2}ついに[、]{.chpule2}[
]{.chpuri2}解決策にありつけた[！]{.chpule2}[
]{.chpuri2}喜びに満ちて通話を切ることができました[。]{.chpule2}[
]{.chpuri2}
:::

::::: {.section .structure-container}
##  構造 {#structure}

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/chain-of-responsibility/structure03c9.png?id=848f0fc8dca57a44974d63f8181f5406"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 0.93;"
srcset="/images/patterns/diagrams/chain-of-responsibility/structure.png?id=848f0fc8dca57a44974d63f8181f5406 380w,/images/patterns/diagrams/chain-of-responsibility/structure-1.5x.png?id=49dfbae38f146af7319f80dce9ea7e2a 570w,/images/patterns/diagrams/chain-of-responsibility/structure-2x.png?id=bb837faaac88e7f2a16f751d0beaa201 760w"
sizes="(max-width: 720px) 100vw, 380px" loading="lazy" width="380"
alt="Chain of Responsibility デザインパターンの構造" /><img
src="../../images/patterns/diagrams/chain-of-responsibility/structure-indexedfcb6.png?id=e13a5bf44f9ca47299223116af77cbef"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 0.88;"
srcset="/images/patterns/diagrams/chain-of-responsibility/structure-indexed.png?id=e13a5bf44f9ca47299223116af77cbef 380w,/images/patterns/diagrams/chain-of-responsibility/structure-indexed-1.5x.png?id=ca660e2cb697b512aadc07fdd3e109cd 570w,/images/patterns/diagrams/chain-of-responsibility/structure-indexed-2x.png?id=4f27e2c48e635f45a78472d707a8df3c 760w"
sizes="(max-width: 720px) 100vw, 380px" loading="lazy" width="380"
alt="Chain of Responsibility デザインパターンの構造" />
</figure>
:::

1.  **ハンドラー**[ ]{.chpule1}[（]{.chpuri1}Handler[）]{.chpule2}[
    ]{.chpuri2}は[、]{.chpule2}[
    ]{.chpuri2}すべての具象ハンドラーに共通したインタフェースを宣言します[。]{.chpule2}[
    ]{.chpuri2}通常は[、]{.chpule2}[
    ]{.chpuri2}リクエストを処理するためのメソッド一つだけを含んでいますが[、]{.chpule2}[
    ]{.chpuri2}時には連鎖上の次のハンドラーを設定するための別のメソッドを含んでいることもあります[。]{.chpule2}[
    ]{.chpuri2}

2.  **基底ハンドラー**[ ]{.chpule1}[（]{.chpuri1}Base
    Handler[）]{.chpule2}[ ]{.chpuri2}は[、]{.chpule2}[
    ]{.chpuri2}ある場合とない場合があります[。]{.chpule2}[
    ]{.chpuri2}すべてのハンドラー・クラスに共通する定型のコードを入れることができます[。]{.chpule2}[
    ]{.chpuri2}

    通常[、]{.chpule2}[
    ]{.chpuri2}このクラスには次のハンドラーへの参照を格納するためのフィールドが定義されています[。]{.chpule2}[
    ]{.chpuri2}クライアントは[、]{.chpule2}[
    ]{.chpuri2}ハンドラーを[、]{.chpule2}[
    ]{.chpuri2}その手前のハンドラーのコンストラクターか setter
    に渡すことで[、]{.chpule2}[
    ]{.chpuri2}連鎖を構築できます[。]{.chpule2}[
    ]{.chpuri2}クラスは[、]{.chpule2}[
    ]{.chpuri2}次のハンドラーがあることを確認した後[、]{.chpule2}[
    ]{.chpuri2}実行を次に移すといった[、]{.chpule2}[
    ]{.chpuri2}デフォルトの処理動作を実装することもできます[。]{.chpule2}[
    ]{.chpuri2}

3.  **具象ハンドラー**[ ]{.chpule1}[（]{.chpuri1}Concrete
    Handler[）]{.chpule2}[
    ]{.chpuri2}にはリクエストを処理するための実際のコードが含まれています[。]{.chpule2}[
    ]{.chpuri2}リクエストを受け取ると[、]{.chpule2}[
    ]{.chpuri2}各ハンドラーはそれを処理するかどうか[、]{.chpule2}[
    ]{.chpuri2}さらには連鎖に沿ってそれを渡すかどうかを決める必要があります[。]{.chpule2}[
    ]{.chpuri2}

    ハンドラーは通常[、]{.chpule2}[
    ]{.chpuri2}自己完結型かつ不変であり[、]{.chpule2}[
    ]{.chpuri2}コンストラクターを通して必要なすべてのデータを一度だけ受け入れます[。]{.chpule2}[
    ]{.chpuri2}

4.  **クライアント**[ ]{.chpule1}[（]{.chpuri1}Client[）]{.chpule2}[
    ]{.chpuri2}は[、]{.chpule2}[
    ]{.chpuri2}一度だけ連鎖を構成するかもしれませんし[、]{.chpule2}[
    ]{.chpuri2}動的に構成するかもしれません[。]{.chpule2}[
    ]{.chpuri2}これは[、]{.chpule2}[
    ]{.chpuri2}アプリケーションのロジックによります[。]{.chpule2}[
    ]{.chpuri2}リクエストは連鎖内のいずれのハンドラーにも送ることができます[。]{.chpule2}[
    ]{.chpuri2}最初のハンドラーとは限りません[。]{.chpule2}[ ]{.chpuri2}
::::
:::::

::: {.section .pseudocode}
##  擬似コード {#pseudocode}

この例では[、]{.chpule2}[ ]{.chpuri2}**Chain of Responsibility**
パターンを使い[、]{.chpule2}[ ]{.chpuri2}アクティブな GUI
要素の状況依存ヘルプ情報を表示します[。]{.chpule2}[ ]{.chpuri2}

<figure class="image">
<img
src="../../images/patterns/diagrams/chain-of-responsibility/example-ja0b7f.png?id=880446946cf4371263ff620119f01bb7"
style="aspect-ratio: 1.09;"
srcset="/images/patterns/diagrams/chain-of-responsibility/example-ja.png?id=880446946cf4371263ff620119f01bb7 610w,/images/patterns/diagrams/chain-of-responsibility/example-ja-1.5x.png?id=5c8d32462602112ae5294492be63dcc7 915w,/images/patterns/diagrams/chain-of-responsibility/example-ja-2x.png?id=0a15da4104418e91908187b789d1fae0 1220w"
sizes="(max-width: 720px) 100vw, 610px" loading="lazy" width="610"
alt="Chain of Responsibility デザインパターンの例の構造" />
<figcaption><p>GUI のクラスは Composite
パターンに従って構築されていて<span class="chpule2">、</span><span
class="chpuri2"> </span>各要素は<span class="chpule2">、</span><span
class="chpuri2"> </span>そのコンテナ要素にリンクされている<span
class="chpule2">。</span><span class="chpuri2"> </span>いつでも<span
class="chpule2">、</span><span class="chpuri2">
</span>要素自体から始まり<span class="chpule2">、</span><span
class="chpuri2"> </span>それを含むすべてのコンテナ要素を通過する
GUI 要素の連鎖が構築可能<span class="chpule2">。</span><span
class="chpuri2"> </span></p></figcaption>
</figure>

アプリケーションの GUI は通常[、]{.chpule2}[
]{.chpuri2}オブジェクト・ツリーの構造をしています[。]{.chpule2}[
]{.chpuri2}たとえば[、]{.chpule2}[
]{.chpuri2}アプリのメイン・ウィンドウを描画する `Dialog`
クラスはオブジェクト・ツリーの根本となります[。]{.chpule2}[
]{.chpuri2}ダイアログは `Panel` を複数含み[、]{.chpule2}[
]{.chpuri2}それはさらに他のパネルや[、]{.chpule2}[ ]{.chpuri2}`Buttons`
や `Text­Fields`
などの単純な低レベルの要素を含んでいるかもしれません[。]{.chpule2}[
]{.chpuri2}

単純なコンポーネントは[、]{.chpule2}[
]{.chpuri2}それに割り当てられたヘルプテキストがある限り[、]{.chpule2}[
]{.chpuri2}簡単な状況依存のツールチップを表示できます[。]{.chpule2}[
]{.chpuri2}しかし[、]{.chpule2}[
]{.chpuri2}もっと複雑なコンポーネントは[、]{.chpule2}[
]{.chpuri2}マニュアルの抜粋を表示するとか[、]{.chpule2}[
]{.chpuri2}ブラウザーでページを開くとか[、]{.chpule2}[
]{.chpuri2}ヘルプを表示する独自の方法を定義します[。]{.chpule2}[
]{.chpuri2}

<figure class="image">
<img
src="../../images/patterns/diagrams/chain-of-responsibility/example2-ja0ae4.png?id=c2282607da7c311389a36330cb44c899"
style="aspect-ratio: 0.81;"
srcset="/images/patterns/diagrams/chain-of-responsibility/example2-ja.png?id=c2282607da7c311389a36330cb44c899 250w,/images/patterns/diagrams/chain-of-responsibility/example2-ja-1.5x.png?id=08c56217dc66084a766363ff5f3bace9 375w,/images/patterns/diagrams/chain-of-responsibility/example2-ja-2x.png?id=491283ec0413005ab170ca166b838a7c 500w"
sizes="(max-width: 720px) 100vw, 250px" loading="lazy" width="250"
alt="Chain of Responsibility 例の構造" />
<figcaption><p>ヘルプ・リクエストが
GUI オブジェクトをたどって行く様子<span class="chpule2">。</span><span
class="chpuri2"> </span></p></figcaption>
</figure>

ユーザーがある GUI 要素にマウス・カーソルをあててから `F1`
キーを押すと[、]{.chpule2}[
]{.chpuri2}アプリケーションはポインターの下のコンポーネントを検出し[、]{.chpule2}[
]{.chpuri2}それにヘルプ・リクエストを送ります[。]{.chpule2}[
]{.chpuri2}リクエストは[、]{.chpule2}[
]{.chpuri2}情報を表示できる要素に到達するまで[、]{.chpule2}[
]{.chpuri2}各要素のコンテナを通って泡のように上昇して行きます[。]{.chpule2}[
]{.chpuri2}

<figure class="code">
<pre class="code" lang="pseudocode"><code>// ハンドラーのインターフェースは、リクエストを実行するためのメソッドを宣
// 言。
interface ComponentWithContextualHelp is
    method showHelp()


// 単純なコンポーネントのための基底クラス。
abstract class Component implements ComponentWithContextualHelp is
    field tooltipText: string

    // コンポーネントのコンテナは、ハンドラーの連鎖の中で次へのリンクとし
    // て機能。
    protected field container: Container

    // コンポーネントは、ヘルプ・テキストが指定されている場合は、ツール
    // チップを表示。そうでない場合は、呼び出しをコンテナに（もしあれば）
    // 転送。
    method showHelp() is
        if (tooltipText != null)
            // ツールチップを表示。
        else
            container.showHelp()


// コンテナは、単純なコンポーネントと他のコンポーネントの両方を子として含
// めることができる。連鎖関係はここで発生する。クラスは、showHelp の振る
// 舞いを親から引き継ぐ。
abstract class Container extends Component is
    protected field children: array of Component

    method add(child) is
        children.add(child)
        child.container = this


// 基本コンポーネントに対しては、デフォルトのヘルプの実装で十分かもしれな
// い。
class Button extends Component is
    // ……

// しかし、複雑なコンポーネントの場合は、デフォルトの実装を上書きするかも
// しれない。ヘルプ文が新規の方法で提供できない場合、コンポーネントは常に
// 基底クラスの実装を呼ぶことができる（Component クラス参照）。
class Panel extends Container is
    field modalHelpText: string

    method showHelp() is
        if (modalHelpText != null)
            // ヘルプ文をモーダル・ウィンドウで表示。
        else
            super.showHelp()

// ……同上……
class Dialog extends Container is
    field wikiPageURL: string

    method showHelp() is
        if (wikiPageURL != null)
            // Wiki のヘルプ・ページを開く。
        else
            super.showHelp()


// クライアント・コード。
class Application is
    // 連鎖の構成の方法はアプリによって異なる。
    method createUI() is
        dialog = new Dialog(&quot;Budget Reports&quot;)
        dialog.wikiPageURL = &quot;http://……&quot;
        panel = new Panel(0, 0, 400, 800)
        panel.modalHelpText = &quot;This panel does……&quot;
        ok = new Button(250, 760, 50, 20, &quot;OK&quot;)
        ok.tooltipText = &quot;This is an OK button that……&quot;
        cancel = new Button(320, 760, 50, 20, &quot;Cancel&quot;)
        // ……
        panel.add(ok)
        panel.add(cancel)
        dialog.add(panel)

    // 何が起きるか考えてみてください。
    method onF1KeyPress() is
        component = this.getComponentAtMouseCoords()
        component.showHelp()</code></pre>
</figure>
:::

:::::::::: {.section .applicability-container}
##  適応性 {#applicability}

::::::::: applicability
::: applicability-problem
プログラムが様々な種類のリクエストを様々な方法で処理しなければいけないが[、]{.chpule2}[
]{.chpuri2}正確にどういうリクエストがどういう順番で来るかが前もって予想できない場合[、]{.chpule2}[
]{.chpuri2}Chain of Responsibility を適用します[。]{.chpule2}[
]{.chpuri2}
:::

::: applicability-solution
このパターンにより[、]{.chpule2}[
]{.chpuri2}複数のハンドラーを一つの連鎖にリンクすることができます[。]{.chpule2}[
]{.chpuri2}リクエストを受け取ると各ハンドラーに処理できるかどうかを[
]{.chpule1}[「]{.chpuri1}問い合わせ[」]{.chpule2}[
]{.chpuri2}ます[。]{.chpule2}[ ]{.chpuri2}これにより[、]{.chpule2}[
]{.chpuri2}すべてのハンドラーはリクエストを処理する機会を得ることができます[。]{.chpule2}[
]{.chpuri2}
:::

::: applicability-problem
特定の順序で複数のハンドラーを実行することが必須な場合に[、]{.chpule2}[
]{.chpuri2}パターンを使用します[。]{.chpule2}[ ]{.chpuri2}
:::

::: applicability-solution
連鎖内のハンドラーを好きな順番にリンクできるので[、]{.chpule2}[
]{.chpuri2}すべてのリクエストは計画通りに連鎖を通過します[。]{.chpule2}[
]{.chpuri2}
:::

::: applicability-problem
ハンドラの組み合わせと順序が実行時に変更されることが想定されている場合に[、]{.chpule2}[
]{.chpuri2}CoR パターンを使用します[。]{.chpule2}[ ]{.chpuri2}
:::

::: applicability-solution
ハンドラー・クラス内の参照フィールド用の setter
を提供すれば[、]{.chpule2}[
]{.chpuri2}ハンドラーを動的に挿入[、]{.chpule2}[
]{.chpuri2}削除[、]{.chpule2}[
]{.chpuri2}順番変更することが可能となります[。]{.chpule2}[ ]{.chpuri2}
:::
:::::::::
::::::::::

::: {.section .checklist}
##  実装方法 {#checklist}

1.  ハンドラー・インターフェースを宣言し[、]{.chpule2}[
    ]{.chpuri2}そこでリクエストを処理するメソッドのシグネチャーを記述します[。]{.chpule2}[
    ]{.chpuri2}

    クライアントがリクエスト・データをどうにメソッドに渡すかを決めます[。]{.chpule2}[
    ]{.chpuri2}最も柔軟な方法は[、]{.chpule2}[
    ]{.chpuri2}リクエストをオブジェクトに変換し[、]{.chpule2}[
    ]{.chpuri2}それを処理メソッドに引数として渡すことです[。]{.chpule2}[
    ]{.chpuri2}

2.  具象ハンドラー内同士の定形コードの重複を排除するために[、]{.chpule2}[
    ]{.chpuri2}ハンドラー・インターフェースから派生した抽象基底クラスを作成する価値があるかもしれません[。]{.chpule2}[
    ]{.chpuri2}

    このクラスには[、]{.chpule2}[
    ]{.chpuri2}連鎖内の次のハンドラーへの参照を格納するためのフィールドが必要です[。]{.chpule2}[
    ]{.chpuri2}クラスを変更不可にすることを検討してください[。]{.chpule2}[
    ]{.chpuri2}ただし[、]{.chpule2}[
    ]{.chpuri2}実行時に連鎖を変更する計画がある場合は[、]{.chpule2}[
    ]{.chpuri2}参照フィールドの値を変更する setter
    を定義する必要があります[。]{.chpule2}[ ]{.chpuri2}

    次のオブジェクトがある限りそこへリクエストを転送することを[、]{.chpule2}[
    ]{.chpuri2}処理メソッドの便利なデフォルトとして実装することもできます[。]{.chpule2}[
    ]{.chpuri2}

3.  ハンドラーの具象サブクラスの作成と[、]{.chpule2}[
    ]{.chpuri2}その処理メソッドを一つずつ実装していきます[。]{.chpule2}[
    ]{.chpuri2}各ハンドラーは[、]{.chpule2}[
    ]{.chpuri2}リクエストを受け取る時に以下の二つの事項を決定をする必要があります[：]{.chpule2}[
    ]{.chpuri2}

    -   リクエストを処理すべきか[。]{.chpule2}[ ]{.chpuri2}
    -   連鎖に沿ってリクエストを渡すべきか[。]{.chpule2}[ ]{.chpuri2}

4.  クライアントは[、]{.chpule2}[
    ]{.chpuri2}それ自身で連鎖を組み立てるかもしれませんし[、]{.chpule2}[
    ]{.chpuri2}または他のオブジェクトから事前構築済みの連鎖を受け取ることもできます[。]{.chpule2}[
    ]{.chpuri2}後者の場合[、]{.chpule2}[
    ]{.chpuri2}構成または環境設定に従って連鎖を構築するためのファクトリー・クラスをいくつか実装する必要があります[。]{.chpule2}[
    ]{.chpuri2}

5.  クライアントは[、]{.chpule2}[
    ]{.chpuri2}最初のハンドラーに限らず[、]{.chpule2}[
    ]{.chpuri2}連鎖内のどのハンドラーからでも[、]{.chpule2}[
    ]{.chpuri2}処理を開始できます[。]{.chpule2}[
    ]{.chpuri2}リクエストは[、]{.chpule2}[
    ]{.chpuri2}あるハンドラーが次に渡すことを拒否するか[、]{.chpule2}[
    ]{.chpuri2}あるいは連鎖の最後に到達するまで[、]{.chpule2}[
    ]{.chpuri2}連鎖沿いに渡されます[。]{.chpule2}[ ]{.chpuri2}

6.  連鎖は動的に変化するので[、]{.chpule2}[
    ]{.chpuri2}クライアントは次のような状況に対処できる必要があります[：]{.chpule2}[
    ]{.chpuri2}

    -   連鎖には[、]{.chpule2}[
        ]{.chpuri2}リンクが一つしかない[。]{.chpule2}[ ]{.chpuri2}
    -   一部のリクエストは連鎖の最後に到達しないかもしれない[。]{.chpule2}[
        ]{.chpuri2}
    -   他のリクエストは処理されないまま[、]{.chpule2}[
        ]{.chpuri2}連鎖の最後に到達するかもしれない[。]{.chpule2}[
        ]{.chpuri2}
:::

:::::: {.section .pros-cons}
##  長所と短所 {#pros-cons}

::::: row
::: col-sm-6
-    リクエスト処理の順序を制御できる[。]{.chpule2}[ ]{.chpuri2}
-   
    *[単]{.cjk}[一]{.cjk}[責]{.cjk}[任]{.cjk}[の]{.cjk}[原]{.cjk}[則]{.cjk}*[。]{.chpule2}[
    ]{.chpuri2}処理を起動するクラスを[、]{.chpule2}[
    ]{.chpuri2}実際に処理をするクラスから分離可能[。]{.chpule2}[
    ]{.chpuri2}
-   
    *[開]{.cjk}[放]{.cjk}[閉]{.cjk}[鎖]{.cjk}[の]{.cjk}[原]{.cjk}[則]{.cjk}*[。]{.chpule2}[
    ]{.chpuri2}新規ハンドラーをアプリに導入しても[、]{.chpule2}[
    ]{.chpuri2}既存クライアント側コードは動作する[。]{.chpule2}[
    ]{.chpuri2}
:::

::: col-sm-6
-    一部のリクエストが未処理で終わる可能性[。]{.chpule2}[ ]{.chpuri2}
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

-   [Chain of Responsibility](chain-of-responsibility.html)
    は[、]{.chpule2}[ ]{.chpuri2}よく [Composite](composite.html)
    と一緒に使われます[。]{.chpule2}[ ]{.chpuri2}この場合[、]{.chpule2}[
    ]{.chpuri2}リーフ[ ]{.chpule1}[（]{.chpuri1}末端[）]{.chpule2}[
    ]{.chpuri2}のコンポーネントがリクエストを受ける時[、]{.chpule2}[
    ]{.chpuri2}リクエストは[、]{.chpule2}[
    ]{.chpuri2}全部の親コンポーネントからオブジェクト・ツリーのルート[
    ]{.chpule1}[（]{.chpuri1}根[）]{.chpule2}[
    ]{.chpuri2}までを通るかもしれません[。]{.chpule2}[ ]{.chpuri2}

-   [Chain of Responsibility](chain-of-responsibility.html)
    のハンドラーは[、]{.chpule2}[ ]{.chpuri2}[Commands](command.html)
    で実装可能です[。]{.chpule2}[ ]{.chpuri2}この場合[、]{.chpule2}[
    ]{.chpuri2}リクエストに代表される同一のコンテキスト・オブジェクトに対して多くの異なる処理を実行できます[。]{.chpule2}[
    ]{.chpuri2}

    しかしもう一つのやり方は[、]{.chpule2}[
    ]{.chpuri2}リクエスト自身を*[コ]{.cjk}[マ]{.cjk}[ン]{.cjk}[ド]{.cjk}*・オブジェクトとすることです[。]{.chpule2}[
    ]{.chpuri2}この場合[、]{.chpule2}[
    ]{.chpuri2}連鎖にリンクされた異なる一連のコンテキスト中で同じ処理を実行できます[。]{.chpule2}[
    ]{.chpuri2}

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
:::

::: {.section .implementations}
##  コード例 {#implementations}

[![Chain of Responsibility を C#
で](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](chain-of-responsibility/csharp/example.html "Chain of Responsibility を C# で"){.prog-lang-link}
[![Chain of Responsibility を C++
で](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](chain-of-responsibility/cpp/example.html "Chain of Responsibility を C++ で"){.prog-lang-link}
[![Chain of Responsibility を Go
で](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](chain-of-responsibility/go/example.html "Chain of Responsibility を Go で"){.prog-lang-link}
[![Chain of Responsibility を Java
で](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](chain-of-responsibility/java/example.html "Chain of Responsibility を Java で"){.prog-lang-link}
[![Chain of Responsibility を PHP
で](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](chain-of-responsibility/php/example.html "Chain of Responsibility を PHP で"){.prog-lang-link}
[![Chain of Responsibility を Python
で](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](chain-of-responsibility/python/example.html "Chain of Responsibility を Python で"){.prog-lang-link}
[![Chain of Responsibility を Ruby
で](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](chain-of-responsibility/ruby/example.html "Chain of Responsibility を Ruby で"){.prog-lang-link}
[![Chain of Responsibility を Rust
で](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](chain-of-responsibility/rust/example.html "Chain of Responsibility を Rust で"){.prog-lang-link}
[![Chain of Responsibility を Swift
で](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](chain-of-responsibility/swift/example.html "Chain of Responsibility を Swift で"){.prog-lang-link}
[![Chain of Responsibility を TypeScript
で](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](chain-of-responsibility/typescript/example.html "Chain of Responsibility を TypeScript で"){.prog-lang-link}
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

[Command []{.fa .fa-arrow-right}](command.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### 戻る

[[]{.fa
.fa-arrow-left} 振る舞いに関するパターン](behavioral-patterns.html){.btn
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
