::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} /
[デザインパターン](../design-patterns.html){.type} /
[振る舞いに関するパターン](behavioral-patterns.html){.type}
:::

# Command {#command .title}

::: aka
別名：[Action、]{style="display:inline-block"}[Transaction、]{style="display:inline-block"}[コマンド、]{style="display:inline-block"}[命令、]{style="display:inline-block"}[アクション、]{style="display:inline-block"}[トランザクション]{style="display:inline-block"}
:::

::: {.section .intent}
##  一言でいうと {#intent}

**Command**[ ]{.chpule1}[（]{.chpuri1}コマンド[、]{.chpule2}[
]{.chpuri2}命令[）]{.chpule2}[ ]{.chpuri2}は[、]{.chpule2}[
]{.chpuri2}振る舞いに関するデザインパターンの一つで[、]{.chpule2}[
]{.chpuri2}リクエストを[、]{.chpule2}[
]{.chpuri2}それに関するすべての情報を含む独立したオブジェクトに転換します[。]{.chpule2}[
]{.chpuri2}この転換により[、]{.chpule2}[
]{.chpuri2}リクエストをメソッドの引数として渡したり[、]{.chpule2}[
]{.chpuri2}リクエストの実行を遅らせたり[、]{.chpule2}[
]{.chpuri2}待ち行列に入れたり[、]{.chpule2}[
]{.chpuri2}取り消し操作を行なうことが可能になります[。]{.chpule2}[
]{.chpuri2}

<figure class="image">
<img
src="../../images/patterns/content/command/command-ja6a3a.png?id=096bf2f09ef54468d846b8d406facd7a"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/command/command-ja.png?id=096bf2f09ef54468d846b8d406facd7a 640w,/images/patterns/content/command/command-ja-1.5x.png?id=8089132b98502a947e7d78df79431246 960w,/images/patterns/content/command/command-ja-2x.png?id=154f7d011add1deca9f82407a40db837 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Command デザインパターン" />
</figure>
:::

::: {.section .problem}
##  問題 {#problem}

新しいテキスト・エディターのアプリを開発しているとします[。]{.chpule2}[
]{.chpuri2}今取り組んでいるのは[、]{.chpule2}[
]{.chpuri2}エディターの様々な操作のためのボタンをたくさん含むツールバーを作成することです[。]{.chpule2}[
]{.chpuri2}ツールバー上のボタンにでも各種ダイアログ上のボタンにでも使用可能な[、]{.chpule2}[
]{.chpuri2}ちょっと気の利いた `Button` クラスができました[。]{.chpule2}[
]{.chpuri2}

<figure class="image">
<img
src="../../images/patterns/diagrams/command/problem163e2.png?id=84189315a0e8d91da262792979005ab4"
style="aspect-ratio: 0.88;"
srcset="/images/patterns/diagrams/command/problem1.png?id=84189315a0e8d91da262792979005ab4 230w,/images/patterns/diagrams/command/problem1-1.5x.png?id=77bf0bc58e5c9c9b8369bc4f8dec457f 345w,/images/patterns/diagrams/command/problem1-2x.png?id=af4c4e9c1d1b4fa2c4104c5f6bb18719 460w"
sizes="(max-width: 720px) 100vw, 230px" loading="lazy" width="230"
alt="Command パターンによって解決された問題" />
<figcaption><p>アプリのすべてのボタンは<span
class="chpule2">、</span><span class="chpuri2">
</span>同じクラスから派生する<span class="chpule2">。</span><span
class="chpuri2"> </span></p></figcaption>
</figure>

これらのボタンはすべて似たように見えますが[、]{.chpule2}[
]{.chpuri2}それぞれ異なることを行うことになっています[。]{.chpule2}[
]{.chpuri2}これらのボタンの様々なクリック処理コードはどこに置けばいいでしょうか[？]{.chpule2}[
]{.chpuri2}最も単純な解決策は[、]{.chpule2}[
]{.chpuri2}ボタンが使用される場所に応じてサブクラスを作成し[、]{.chpule2}[
]{.chpuri2}そこにボタンのクリック時に実行すべきコードを含めるようにすることです[。]{.chpule2}[
]{.chpuri2}多数のサブクラスを作成することになります[。]{.chpule2}[
]{.chpuri2}

<figure class="image">
<img
src="../../images/patterns/diagrams/command/problem2f2f9.png?id=f0e33da1842b3a3ee3b4857de0b6ec93"
style="aspect-ratio: 2.11;"
srcset="/images/patterns/diagrams/command/problem2.png?id=f0e33da1842b3a3ee3b4857de0b6ec93 400w,/images/patterns/diagrams/command/problem2-1.5x.png?id=7ae15e2b07d5d076a878d99c3ed8ac32 600w,/images/patterns/diagrams/command/problem2-2x.png?id=5eea7d0f6247da011150bb63e423f405 800w"
sizes="(max-width: 720px) 100vw, 400px" loading="lazy" width="400"
alt="多数のボタンのサブクラス" />
<figcaption><p>多くのサブクラス<span class="chpule2">。</span><span
class="chpuri2"> </span>はて<span class="chpule2">、</span><span
class="chpuri2"> </span>何か問題ある<span class="chpule2">？</span><span
class="chpuri2"> </span></p></figcaption>
</figure>

やがて[、]{.chpule2}[
]{.chpuri2}あなたはこのやり方には大きな欠陥があることに気づきます[。]{.chpule2}[
]{.chpuri2}まず膨大な数のサブクラスがあります[。]{.chpule2}[
]{.chpuri2}基底の `Button` クラスを変更するたびに[、]{.chpule2}[
]{.chpuri2}サブクラスが壊れるリスクを心配していなのでしたら[、]{.chpule2}[
]{.chpuri2}それでも大丈夫なのですが[。]{.chpule2}[
]{.chpuri2}簡単に言うと[、]{.chpule2}[ ]{.chpuri2}GUI
コードは[、]{.chpule2}[
]{.chpuri2}変更が常のビジネス・ロジックのコードにいつの間にか依存するようになってしまった[、]{.chpule2}[
]{.chpuri2}ということです[。]{.chpule2}[ ]{.chpuri2}

<figure class="image">
<img
src="../../images/patterns/diagrams/command/problem3-ja354c.png?id=cd2c4b44747a396ab42fcb96e9a97120"
style="aspect-ratio: 4.36;"
srcset="/images/patterns/diagrams/command/problem3-ja.png?id=cd2c4b44747a396ab42fcb96e9a97120 480w,/images/patterns/diagrams/command/problem3-ja-1.5x.png?id=68fc509720f905574c60218b68a21b01 720w,/images/patterns/diagrams/command/problem3-ja-2x.png?id=536cf0c03a3c975309d567dbc09881c8 960w"
sizes="(max-width: 720px) 100vw, 480px" loading="lazy" width="480"
alt="いくつかのクラスが同じ機能を実装" />
<figcaption><p>いくつかのクラスが同じ機能を実装<span
class="chpule2">。</span><span class="chpuri2"> </span></p></figcaption>
</figure>

そしてこれが一番の問題です[。]{.chpule2}[
]{.chpuri2}テキストのコピーや貼り付けなどのいくつかの操作は[、]{.chpule2}[
]{.chpuri2}複数の場所から呼び出される必要があります[。]{.chpule2}[
]{.chpuri2}たとえば[、]{.chpule2}[ ]{.chpuri2}ツールバー上の小さな[
]{.chpule1}[「]{.chpuri1}コピー[」]{.chpule2}[
]{.chpuri2}ボタンをクリックすることも[、]{.chpule2}[
]{.chpuri2}コンテキスト・メニューからコピーをすることも[、]{.chpule2}[
]{.chpuri2}キーボードで `Ctrl+C` を押すこともできます[。]{.chpule2}[
]{.chpuri2}

当初[、]{.chpule2}[
]{.chpuri2}私たちのアプリにはツールバーしかなかった時[、]{.chpule2}[
]{.chpuri2}ボタンのサブクラス内に様々な操作の実装を置くことには何も問題ありませんでした[。]{.chpule2}[
]{.chpuri2}つまり[、]{.chpule2}[ ]{.chpuri2}`Copy­Button`
のサブクラス内にテキストをコピーするためのコードがあっても大丈夫でした[。]{.chpule2}[
]{.chpuri2}しかし次にコンテキスト・メニューやショートカットなどを実装すると[、]{.chpule2}[
]{.chpuri2}多くのクラスで操作のコードを重複させるか[、]{.chpule2}[
]{.chpuri2}メニューをボタンに依存させるというさらに悪い選択肢を選ばざるを得なくなります[。]{.chpule2}[
]{.chpuri2}
:::

::: {.section .solution}
##  解決策 {#solution}

良いソフトウェアの設計は[、]{.chpule2}[
]{.chpuri2}多くの場合[、]{.chpule2}[
]{.chpuri2}*[単]{.cjk}[一]{.cjk}[責]{.cjk}[任]{.cjk}[の]{.cjk}[原]{.cjk}[則]{.cjk}*に基づいており[、]{.chpule2}[
]{.chpuri2}これは通常アプリをいくつかの層に分割するに至ります[。]{.chpule2}[
]{.chpuri2}最も一般的な例としては[、]{.chpule2}[
]{.chpuri2}グラフィカル・ユーザー・インターフェース[
]{.chpule1}[（]{.chpuri1}GUI[）]{.chpule2}[
]{.chpuri2}用のレイヤーと[、]{.chpule2}[
]{.chpuri2}ビジネス・ロジック用のレイヤーがあります[。]{.chpule2}[
]{.chpuri2}GUI 層は[、]{.chpule2}[
]{.chpuri2}画面に美しい絵を描画し[、]{.chpule2}[
]{.chpuri2}あらゆる入力を取り込み[、]{.chpule2}[
]{.chpuri2}ユーザーとアプリが行っていることの結果を表示する役割を担っています[。]{.chpule2}[
]{.chpuri2}しかし[、]{.chpule2}[ ]{.chpuri2}GUI 層は[、]{.chpule2}[
]{.chpuri2}月の軌道の計算とか年次報告書の作成のような重要な仕事をビジネス・ロジック層に委ねます[。]{.chpule2}[
]{.chpuri2}

コード上では[、]{.chpule2}[ ]{.chpuri2}こんな感じです[：]{.chpule2}[
]{.chpuri2}GUI オブジェクトは[、]{.chpule2}[
]{.chpuri2}いくつかの引数を渡して[、]{.chpule2}[
]{.chpuri2}ビジネス・ロジックのオブジェクトのメソッドを呼び出す[。]{.chpule2}[
]{.chpuri2}このことは通常[[、]{.ls-1}]{.chpule2}[ ]{.chpuri2}​[
]{.chpule1}[「]{.chpuri1}あるオブジェクトから別のオブジェクトに*[リ]{.cjk}[ク]{.cjk}[エ]{.cjk}[ス]{.cjk}[ト]{.cjk}*を送る[」]{.chpule2}[
]{.chpuri2}と表現されます[。]{.chpule2}[ ]{.chpuri2}

<figure class="image">
<img
src="../../images/patterns/diagrams/command/solution1-ja0be3.png?id=443156f3fdaf43f8e04571e0130c82c4"
style="aspect-ratio: 2.63;"
srcset="/images/patterns/diagrams/command/solution1-ja.png?id=443156f3fdaf43f8e04571e0130c82c4 500w,/images/patterns/diagrams/command/solution1-ja-1.5x.png?id=43305a61cedd40a5c266e184e35cbe0b 750w,/images/patterns/diagrams/command/solution1-ja-2x.png?id=839d66cfb281da58471e18aa4dff2cdf 1000w"
sizes="(max-width: 720px) 100vw, 500px" loading="lazy" width="500"
alt="GUI 層に、ビジネス層オブジェクトへの直接アクセスを許す" />
<figcaption><p>GUI オブジェクトに<span class="chpule2">、</span><span
class="chpuri2">
</span>ビジネス・ロジックのオブジェクトへの直接アクセスを許す<span
class="chpule2">。</span><span class="chpuri2"> </span></p></figcaption>
</figure>

Command パターンに従うと[、]{.chpule2}[ ]{.chpuri2}GUI
オブジェクトは[、]{.chpule2}[
]{.chpuri2}このようなリクエストを直接送るべきではありません[。]{.chpule2}[
]{.chpuri2}代わりに[、]{.chpule2}[
]{.chpuri2}呼び出し対象のオブジェクト[、]{.chpule2}[
]{.chpuri2}メソッド名[、]{.chpule2}[
]{.chpuri2}引数などのリクエストの詳細を *Command*
クラスに抽出します[。]{.chpule2}[
]{.chpuri2}このクラスには[、]{.chpule2}[
]{.chpuri2}リクエストの引き金となるようなメソッドが一つだけあります[。]{.chpule2}[
]{.chpuri2}

コマンドオ・ブジェクトは[、]{.chpule2}[ ]{.chpuri2}様々な GUI
とビジネス・ロジックのオブジェクトとの間のリンクとして機能します[。]{.chpule2}[
]{.chpuri2}これ以降[、]{.chpule2}[ ]{.chpuri2}GUI
オブジェクトは[、]{.chpule2}[
]{.chpuri2}どのビジネス・ロジックのオブジェクトがリクエストを受け取り[、]{.chpule2}[
]{.chpuri2}どう処理するのかを知る必要はありません[。]{.chpule2}[
]{.chpuri2}GUI オブジェクトが[、]{.chpule2}[
]{.chpuri2}コマンドの引き金を引くだけで[、]{.chpule2}[
]{.chpuri2}細かいことはコマンドが処理します[。]{.chpule2}[ ]{.chpuri2}

<figure class="image">
<img
src="../../images/patterns/diagrams/command/solution2-ja141e.png?id=45e146d8acb61829a66d03d72b8b61f9"
style="aspect-ratio: 3;"
srcset="/images/patterns/diagrams/command/solution2-ja.png?id=45e146d8acb61829a66d03d72b8b61f9 570w,/images/patterns/diagrams/command/solution2-ja-1.5x.png?id=98846ffeebf0ea549d8cbfd9c33dafc7 855w,/images/patterns/diagrams/command/solution2-ja-2x.png?id=87f1c58b6694a5668074a1347813ac0c 1140w"
sizes="(max-width: 720px) 100vw, 570px" loading="lazy" width="570"
alt="コマンドを介してビジネス・ロジック層にアクセス。" />
<figcaption><p>コマンドを介してビジネス・ロジック層にアクセス<span
class="chpule2">。</span><span class="chpuri2"> </span></p></figcaption>
</figure>

次のステップは[、]{.chpule2}[
]{.chpuri2}コマンドが同じインターフェースを実装するようにすることです[。]{.chpule2}[
]{.chpuri2}このインターフェースは[、]{.chpule2}[
]{.chpuri2}通常はパラメーターなしの単一の実行メソッドを持っています[。]{.chpule2}[
]{.chpuri2}このおかげで[、]{.chpule2}[
]{.chpuri2}コマンドの特定の具象クラスに密結合せずに[、]{.chpule2}[
]{.chpuri2}様々なコマンドを使用できるようになります[。]{.chpule2}[
]{.chpuri2}ボーナスとして[、]{.chpule2}[
]{.chpuri2}送信オブジェクトにリンクされているコマンド・オブジェクトを変更できるので[、]{.chpule2}[
]{.chpuri2}送信オブジェクトの振る舞いを動的に変更するのと同じ効果があります[。]{.chpule2}[
]{.chpuri2}

一つ欠けているものがあるのにお気づきですか[？]{.chpule2}[
]{.chpuri2}リクエストのパラメーターです[。]{.chpule2}[ ]{.chpuri2}ある
GUI オブジェクトは[、]{.chpule2}[
]{.chpuri2}ビジネス層のオブジェクトに何らかのパラメーターを渡していたかもしれません[。]{.chpule2}[
]{.chpuri2}コマンドの実行メソッドには引数がありません[。]{.chpule2}[
]{.chpuri2}どうやってリクエストの細かいことを受け手に渡せばいいのでしょう[？]{.chpule2}[
]{.chpuri2}このため[、]{.chpule2}[ ]{.chpuri2}コマンドは[、]{.chpule2}[
]{.chpuri2}データを事前構成しておくか[、]{.chpule2}[
]{.chpuri2}構成を自前で取得する能力を持っている必要があります[。]{.chpule2}[
]{.chpuri2}

<figure class="image">
<img
src="../../images/patterns/diagrams/command/solution3-ja8545.png?id=15d6ebc34ff5e3853af786fb2e8fd503"
style="aspect-ratio: 1.83;"
srcset="/images/patterns/diagrams/command/solution3-ja.png?id=15d6ebc34ff5e3853af786fb2e8fd503 440w,/images/patterns/diagrams/command/solution3-ja-1.5x.png?id=834af6859357cd2d7272bfd0fa375e86 660w,/images/patterns/diagrams/command/solution3-ja-2x.png?id=bcedb957a3d9b6b426a6526a16d1d705 880w"
sizes="(max-width: 720px) 100vw, 440px" loading="lazy" width="440"
alt="GUI オブジェクトは作業をコマンドに委任" />
<figcaption><p>GUI オブジェクトは作業をコマンドに委任<span
class="chpule2">。</span><span class="chpuri2"> </span></p></figcaption>
</figure>

テキスト・エディターの話に戻りましょう[。]{.chpule2}[ ]{.chpuri2}Command
パターンを適用した後[、]{.chpule2}[
]{.chpuri2}様々なクリック動作を実装するためのボタンのサブクラスはすべて不要となりました[。]{.chpule2}[
]{.chpuri2}基底クラスである `Button`
クラスにコマンド・オブジェクトへの参照を格納するフィールド一つを追加し[、]{.chpule2}[
]{.chpuri2}クリックに応じてボタンがそのコマンドを実行するだけで十分です[。]{.chpule2}[
]{.chpuri2}

すべての可能な操作に対してコマンドのクラスを多数実装し[、]{.chpule2}[
]{.chpuri2}ボタンの意図した振る舞いに応じて特定のボタンとリンクします[。]{.chpule2}[
]{.chpuri2}

メニュー[、]{.chpule2}[ ]{.chpuri2}ショートカット[、]{.chpule2}[
]{.chpuri2}ダイアログ全体等[、]{.chpule2}[ ]{.chpuri2}他の GUI
要素も同様に実装できます[。]{.chpule2}[ ]{.chpuri2}ユーザーがある GUI
要素を操作すると[、]{.chpule2}[
]{.chpuri2}それにリンクされたコマンドが実行されます[。]{.chpule2}[
]{.chpuri2}多分もうお分かりかと思いますが[、]{.chpule2}[
]{.chpuri2}同じ操作に関する GUI
要素は同じコマンドにリンクされ[、]{.chpule2}[
]{.chpuri2}コードの重複を防ぎます[。]{.chpule2}[ ]{.chpuri2}

結果として[、]{.chpule2}[ ]{.chpuri2}コマンドは[、]{.chpule2}[
]{.chpuri2}GUI
とビジネス・ロジック層との結合を減らす便利な中間層となります[。]{.chpule2}[
]{.chpuri2}そしてこれは[、]{.chpule2}[ ]{.chpuri2}Command
パターンがもたらすメリットのほんの一部に過ぎません[！]{.chpule2}[
]{.chpuri2}
:::

::: {.section .analogy}
##  現実世界でのたとえ {#analogy}

<figure class="image">
<img
src="../../images/patterns/content/command/command-comic-16e6f.png?id=551df832f445080976f3116e0dc120c9"
style="aspect-ratio: 2;"
srcset="/images/patterns/content/command/command-comic-1.png?id=551df832f445080976f3116e0dc120c9 600w,/images/patterns/content/command/command-comic-1-1.5x.png?id=82dc5c38ce372ed4dfd4a37c04faeb72 900w,/images/patterns/content/command/command-comic-1-2x.png?id=47b3c00b2cfbda7157a1ed9da6ce2948 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="レストランで注文票を作成" />
<figcaption><p>レストランで注文票を作成<span
class="chpule2">。</span><span class="chpuri2"> </span></p></figcaption>
</figure>

結構長い時間かけて街中を歩き回った後[、]{.chpule2}[
]{.chpuri2}やっと素敵なレストランの窓際のテーブルにありつけました[。]{.chpule2}[
]{.chpuri2}フレンドリーなウェイターがやってきて[、]{.chpule2}[
]{.chpuri2}素早く注文を取り[、]{.chpule2}[
]{.chpuri2}紙の上に書き留めます[。]{.chpule2}[
]{.chpuri2}ウェイターはキッチンへ行き[、]{.chpule2}[
]{.chpuri2}壁に注文票を貼り付けます[。]{.chpule2}[
]{.chpuri2}しばらくすると[、]{.chpule2}[
]{.chpuri2}注文票はシェフのところにたどり着き[、]{.chpule2}[
]{.chpuri2}シェフはそれを読み[、]{.chpule2}[
]{.chpuri2}その通りに調理します[。]{.chpule2}[
]{.chpuri2}料理人は注文票と一緒にトレイに食事を置きます[。]{.chpule2}[
]{.chpuri2}ウェイターはトレイを見つけ[、]{.chpule2}[
]{.chpuri2}注文通りにできていることを確認してから[、]{.chpule2}[
]{.chpuri2}すべてをテーブルに運びます[。]{.chpule2}[ ]{.chpuri2}

紙の注文票は[、]{.chpule2}[
]{.chpuri2}コマンドの働きをしています[。]{.chpule2}[
]{.chpuri2}それは[、]{.chpule2}[
]{.chpuri2}シェフが注文の処理ができるまで[、]{.chpule2}[
]{.chpuri2}待ち行列に入ります[。]{.chpule2}[
]{.chpuri2}注文票には[、]{.chpule2}[
]{.chpuri2}調理に必要な関連情報がすべて含まれています[。]{.chpule2}[
]{.chpuri2}シェフは[、]{.chpule2}[
]{.chpuri2}客の注文が何であったかを明らかにするために[、]{.chpule2}[
]{.chpuri2}あちこち走り回ることなく[、]{.chpule2}[
]{.chpuri2}すぐに調理を開始できます[。]{.chpule2}[ ]{.chpuri2}
:::

::::: {.section .structure-container}
##  構造 {#structure}

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/command/structure44ec.png?id=1cd7833638f4c43630f4a84017d31195"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.7;"
srcset="/images/patterns/diagrams/command/structure.png?id=1cd7833638f4c43630f4a84017d31195 630w,/images/patterns/diagrams/command/structure-1.5x.png?id=6e5b68277c7747b22266e408891d5841 945w,/images/patterns/diagrams/command/structure-2x.png?id=1dfaa84354ffe49ef7ad46ce069482d2 1260w"
sizes="(max-width: 720px) 100vw, 630px" loading="lazy" width="630"
alt="Command デザインパターンの構造" /><img
src="../../images/patterns/diagrams/command/structure-indexed49b8.png?id=95529d7282dc7bc1c5bc443423b1cf4f"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.64;"
srcset="/images/patterns/diagrams/command/structure-indexed.png?id=95529d7282dc7bc1c5bc443423b1cf4f 640w,/images/patterns/diagrams/command/structure-indexed-1.5x.png?id=29d6c5c7a06da2747ed756c0ddad6169 960w,/images/patterns/diagrams/command/structure-indexed-2x.png?id=e4cc286a44768c7d060af33497da7df6 1280w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="640"
alt="Command デザインパターンの構造" />
</figure>
:::

1.  **送り手**[ ]{.chpule1}[（]{.chpuri1}Sender[）]{.chpule2}[
    ]{.chpuri2}クラス[、]{.chpule2}[
    ]{.chpuri2}別名*[イ]{.cjk}[ン]{.cjk}[ボ]{.cjk}[ー]{.cjk}[カ]{.cjk}[ー]{.cjk}*[
    ]{.chpule1}[（]{.chpuri1}Invoker[）]{.chpule2}[
    ]{.chpuri2}は[、]{.chpule2}[
    ]{.chpuri2}リクエストの開始を担当します[。]{.chpule2}[
    ]{.chpuri2}このクラスには[、]{.chpule2}[
    ]{.chpuri2}コマンドオ・ブジェクトへの参照を保存するためのフィールドが必要となります[。]{.chpule2}[
    ]{.chpuri2}送り手は[、]{.chpule2}[
    ]{.chpuri2}リクエストを受け手に直接送る代わりに[、]{.chpule2}[
    ]{.chpuri2}コマンドの引き金を引きます[。]{.chpule2}[
    ]{.chpuri2}送り手にはコマンド・オブジェクトの作成の責任がないことに注目してください[。]{.chpule2}[
    ]{.chpuri2}通常は[、]{.chpule2}[
    ]{.chpuri2}クライアントからコンストラクターを介して事前に作成されたコマンドを取得します[。]{.chpule2}[
    ]{.chpuri2}

2.  **コマンド**[ ]{.chpule1}[（]{.chpuri1}Command[）]{.chpule2}[
    ]{.chpuri2}インターフェースは通常[、]{.chpule2}[
    ]{.chpuri2}コマンドを実行するためのメソッドを一つ宣言します[。]{.chpule2}[
    ]{.chpuri2}

3.  **具象コマンド**[ ]{.chpule1}[（]{.chpuri1}Concrete
    Command[）]{.chpule2}[
    ]{.chpuri2}は様々な種類のリクエストを実装します[。]{.chpule2}[
    ]{.chpuri2}具象コマンドは[、]{.chpule2}[
    ]{.chpuri2}仕事を独立して自身で行うべきではなく[、]{.chpule2}[
    ]{.chpuri2}ビジネス・ロジックのオブジェクトのどれかに仕事を渡すべきです[。]{.chpule2}[
    ]{.chpuri2}しかし[、]{.chpule2}[
    ]{.chpuri2}コードの簡素化のためにこの二つのクラスの統合は可能です[。]{.chpule2}[
    ]{.chpuri2}

    受け手のオブジェクトでメソッドの実行に必要なパラメーターは[、]{.chpule2}[
    ]{.chpuri2}具象コマンドのフィールドとして宣言することができます[。]{.chpule2}[
    ]{.chpuri2}これらのフィールドの初期化をコンストラクター内だけで許可するようにすれば[、]{.chpule2}[
    ]{.chpuri2}コマンド・オブジェクトは不変となります[。]{.chpule2}[
    ]{.chpuri2}

4.  **受け手**[ ]{.chpule1}[（]{.chpuri1}Receiver[）]{.chpule2}[
    ]{.chpuri2}クラスには[、]{.chpule2}[
    ]{.chpuri2}何らかのビジネス・ロジックが含まれています[。]{.chpule2}[
    ]{.chpuri2}ほとんどどんなオブジェクトでも受け手になれます[。]{.chpule2}[
    ]{.chpuri2}ほとんどのコマンドは[、]{.chpule2}[
    ]{.chpuri2}リクエストを受け手に渡す詳細を扱い[、]{.chpule2}[
    ]{.chpuri2}実際の作業は受け手が行います[。]{.chpule2}[ ]{.chpuri2}

5.  **クライアント**[ ]{.chpule1}[（]{.chpuri1}Client[）]{.chpule2}[
    ]{.chpuri2}は[、]{.chpule2}[
    ]{.chpuri2}具象コマンド・オブジェクトの作成および構成を行います[。]{.chpule2}[
    ]{.chpuri2}クライアントは[、]{.chpule2}[
    ]{.chpuri2}受け手インスタンスを含むすべてのリクエスト・パラメータをコマンドのコンストラクターに渡す必要があります[。]{.chpule2}[
    ]{.chpuri2}その後[、]{.chpule2}[
    ]{.chpuri2}作成されたコマンドは[、]{.chpule2}[
    ]{.chpuri2}一つ以上の送り手に関連付けられます[。]{.chpule2}[
    ]{.chpuri2}
::::
:::::

::: {.section .pseudocode}
##  擬似コード {#pseudocode}

この例では[、]{.chpule2}[ ]{.chpuri2}**Command**
パターンが[、]{.chpule2}[
]{.chpuri2}実行された操作の履歴を記録し[、]{.chpule2}[
]{.chpuri2}必要に応じて操作の取り消しを可能にするために役立っています[。]{.chpule2}[
]{.chpuri2}

<figure class="image">
<img
src="../../images/patterns/diagrams/command/example31bf.png?id=1f42c8395fe54d0e409026b91881e2a0"
style="aspect-ratio: 1.07;"
srcset="/images/patterns/diagrams/command/example.png?id=1f42c8395fe54d0e409026b91881e2a0 640w,/images/patterns/diagrams/command/example-1.5x.png?id=435055a78a82c005eebb1bef869998ae 960w,/images/patterns/diagrams/command/example-2x.png?id=ed44dfd9b02eccf1ae7e2714d018ed36 1280w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="640"
alt="Command パターン例の構造" />
<figcaption><p>テキスト・エディターの取り消し可能な操作<span
class="chpule2">。</span><span class="chpuri2"> </span></p></figcaption>
</figure>

エディターの状態の変更に至るコマンド[
]{.chpule1}[（]{.chpuri1}例[：]{.chpule2}[
]{.chpuri2}切り取りとペースト[）]{.chpule2}[
]{.chpuri2}は[、]{.chpule2}[
]{.chpuri2}コマンドに関連する操作を実行する前に[、]{.chpule2}[
]{.chpuri2}エディターの状態のバックアップ用コピーを作成します[。]{.chpule2}[
]{.chpuri2}コマンド実行後に[、]{.chpule2}[
]{.chpuri2}コマンド・オブジェクトはコマンド履歴[
]{.chpule1}[（]{.chpuri1}コマンド・オブジェクトのスタック[）]{.chpule2}[
]{.chpuri2}に[、]{.chpule2}[
]{.chpuri2}その時点でエディターの状態のバックアップ用コピーとともに置かれます[。]{.chpule2}[
]{.chpuri2}その後[、]{.chpule2}[
]{.chpuri2}ユーザーが操作を取り消す必要がある場合は[、]{.chpule2}[
]{.chpuri2}アプリは履歴から最新のコマンドを取り出し[、]{.chpule2}[
]{.chpuri2}それに関連したエディターの状態のバックアップを読み込んで[、]{.chpule2}[
]{.chpuri2}復元を行います[。]{.chpule2}[ ]{.chpuri2}

クライアント・コード[ ]{.chpule1}[（]{.chpuri1}GUI 要素[、]{.chpule2}[
]{.chpuri2}コマンド履歴など[）]{.chpule2}[ ]{.chpuri2}は[、]{.chpule2}[
]{.chpuri2}コマンド・インターフェースを介してコマンドと連携協働するため[、]{.chpule2}[
]{.chpuri2}どんなコマンドの具象クラスとも連携していません[。]{.chpule2}[
]{.chpuri2}このやり方では[、]{.chpule2}[
]{.chpuri2}既存のコードを壊すことなく[、]{.chpule2}[
]{.chpuri2}新しいコマンドをアプリに導入できます[。]{.chpule2}[
]{.chpuri2}

<figure class="code">
<pre class="code" lang="pseudocode"><code>// コマンドの基底クラスは、すべての具象コマンドに共通なインターフェースを
// 定義。
abstract class Command is
    protected field app: Application
    protected field editor: Editor
    protected field backup: text

    constructor Command(app: Application, editor: Editor) is
        this.app = app
        this.editor = editor

    // エディターの状態のバックアップを作成。
    method saveBackup() is
        backup = editor.text

    // エディターの状態を復元。
    method undo() is
        editor.text = backup

    // 実行メソッドを abstract と宣言することにより、すべての具象コマン
    // ドが専用の実装を行うことを強制。メソッドは、コマンドがエディターの
    // 状態を変更したかどうかに応じて、true または false を返す。
    abstract method execute()


// 具象コマンドはここで定義。
class CopyCommand extends Command is
    // コピー・コマンドはエディターの状態を変更しないので、履歴には保存し
    // ない。
    method execute() is
        app.clipboard = editor.getSelection()
        return false

class CutCommand extends Command is
    // 切り取りコマンドはエディターの状態を変更するので、履歴に保存する必
    // 要あり。メソッドが true を返す限り、保存される。
    method execute() is
        saveBackup()
        app.clipboard = editor.getSelection()
        editor.deleteSelection()
        return true

class PasteCommand extends Command is
    method execute() is
        saveBackup()
        editor.replaceSelection(app.clipboard)
        return true

// 取り消し操作もコマンドの一つ。
class UndoCommand extends Command is
    method execute() is
        app.undo()
        return false


// 大域コマンド履歴は、単なるスタック。
class CommandHistory is
    private field history: array of Command

    // 最後に入れたものを……
    method push(c: Command) is
        // コマンドを履歴配列の後方にプッシュ。

    // ……最初に取り出す。
    method pop():Command is
        // 履歴から最新のコマンドを取得。


// エディター・クラスは、実際のエディターの操作を保持。受け手の役を果たし、
// 全コマンドは、実行をエディターのメソッドに委任。
class Editor is
    field text: string

    method getSelection() is
        // 選択されたテキストを返す。

    method deleteSelection() is
        // 選択されたテキストを削除。

    method replaceSelection(text) is
        // クリップボードの内容を現在の位置に貼り付ける。


// アプリケーション・クラスは、オブジェクト同士の関係を設定する。何かを行
// う必要がある時に送り手として機能し、コマンド・オブジェクトを作成して実
// 行。
class Application is
    field clipboard: string
    field editors: array of Editors
    field activeEditor: Editor
    field history: CommandHistory

    // UI のオブジェクトにコマンドを割り当てるコードは、こんな感じ。
    method createUI() is
        // ……
        copy = function() { executeCommand(
            new CopyCommand(this, activeEditor)) }
        copyButton.setCommand(copy)
        shortcuts.onKeyPress(&quot;Ctrl+C&quot;, copy)

        cut = function() { executeCommand(
            new CutCommand(this, activeEditor)) }
        cutButton.setCommand(cut)
        shortcuts.onKeyPress(&quot;Ctrl+X&quot;, cut)

        paste = function() { executeCommand(
            new PasteCommand(this, activeEditor)) }
        pasteButton.setCommand(paste)
        shortcuts.onKeyPress(&quot;Ctrl+V&quot;, paste)

        undo = function() { executeCommand(
            new UndoCommand(this, activeEditor)) }
        undoButton.setCommand(undo)
        shortcuts.onKeyPress(&quot;Ctrl+Z&quot;, undo)

    // コマンドを実行し、それが履歴に追加されたかどうかをチェック。
    method executeCommand(command) is
        if (command.execute())
            history.push(command)

    // 履歴から最新のコマンドを取り、取り消しメソッドを実行。そのコマンド
    // のクラスが不明であることに注意。コマンドはそれ自身の操作を取り消す
    // 方法を周知しているため、知る必要なし。
    method undo() is
        command = history.pop()
        if (command != null)
            command.undo()</code></pre>
</figure>
:::

:::::::::: {.section .applicability-container}
##  適応性 {#applicability}

::::::::: applicability
::: applicability-problem
操作をオブジェクトを使いパラメーターとして扱うためには[、]{.chpule2}[
]{.chpuri2}Command パターンを使用します[。]{.chpule2}[ ]{.chpuri2}
:::

::: applicability-solution
Command パターンは[、]{.chpule2}[
]{.chpuri2}特定のメソッドの呼び出しを独立したオブジェクトに変えることができます[。]{.chpule2}[
]{.chpuri2}この変更により[、]{.chpule2}[
]{.chpuri2}以下のような多くの興味深い用途が考えられます[：]{.chpule2}[
]{.chpuri2}コマンドをメソッドの引数として渡す[。]{.chpule2}[
]{.chpuri2}他のオブジェクトに格納する[。]{.chpule2}[
]{.chpuri2}リンクされたコマンドを実行時に切り替えるなど[。]{.chpule2}[
]{.chpuri2}

例を示します[。]{.chpule2}[ ]{.chpuri2}コンテキストメニューのような GUI
コンポーネントを開発しているとします[。]{.chpule2}[
]{.chpuri2}そして[、]{.chpule2}[
]{.chpuri2}エンド・ユーザーがメニュー項目をクリックした時に[、]{.chpule2}[
]{.chpuri2}どのような操作が起動されるかの設定を管理者ができるようにしたいとします[。]{.chpule2}[
]{.chpuri2}
:::

::: applicability-problem
操作を待ち行列に入れたり[、]{.chpule2}[
]{.chpuri2}実行をスケジュールしたり[、]{.chpule2}[
]{.chpuri2}リモートで実行したい場合は[、]{.chpule2}[ ]{.chpuri2}Command
パターンを使用します[。]{.chpule2}[ ]{.chpuri2}
:::

::: applicability-solution
他のオブジェクト同様[、]{.chpule2}[ ]{.chpuri2}コマンドもシリアライズ[
]{.chpule1}[（]{.chpuri1}直列化[）]{.chpule2}[
]{.chpuri2}することができ[、]{.chpule2}[
]{.chpuri2}文字列に変換してファイルやデータベースに簡単に書き込めます[。]{.chpule2}[
]{.chpuri2}その後[、]{.chpule2}[
]{.chpuri2}文字列を元のコマンド・オブジェクトに復元できます[。]{.chpule2}[
]{.chpuri2}したがって[、]{.chpule2}[
]{.chpuri2}コマンドの実行を遅延し[、]{.chpule2}[
]{.chpuri2}後ほどスケジュールに従って実行することができます[。]{.chpule2}[
]{.chpuri2}しかし[、]{.chpule2}[
]{.chpuri2}それだけではありません[！]{.chpule2}[
]{.chpuri2}同様にして[、]{.chpule2}[
]{.chpuri2}待ち行列に入れたり[、]{.chpule2}[
]{.chpuri2}ログに記録したり[、]{.chpule2}[
]{.chpuri2}ネットワークを通してコマンドを送信したりもできます[。]{.chpule2}[
]{.chpuri2}
:::

::: applicability-problem
取り消し可能な操作の実装にも[、]{.chpule2}[ ]{.chpuri2}Command
パターンが使えます[。]{.chpule2}[ ]{.chpuri2}
:::

::: applicability-solution
取り消しと再実行[ ]{.chpule1}[（]{.chpuri1}undo/redo[）]{.chpule2}[
]{.chpuri2}の実装方法にはいろいろありますが[、]{.chpule2}[
]{.chpuri2}Command
パターンはおそらく最もよく使われている方法です[。]{.chpule2}[
]{.chpuri2}

操作の取り消しを可能とするには[、]{.chpule2}[
]{.chpuri2}実行された操作履歴の実装が必要となります[。]{.chpule2}[
]{.chpuri2}コマンド履歴は[、]{.chpule2}[
]{.chpuri2}実行されたすべてのコマンド・オブジェクトとそれに関連するアプリケーションの状態のバックアップを含んだスタックです[。]{.chpule2}[
]{.chpuri2}

このメソッドには二つの欠点があります[。]{.chpule2}[
]{.chpuri2}最初に[、]{.chpule2}[
]{.chpuri2}アプリケーションの状態の保存は[、]{.chpule2}[
]{.chpuri2}その一部が非公開である可能性があるので[、]{.chpule2}[
]{.chpuri2}それほど簡単ではありません[。]{.chpule2}[
]{.chpuri2}この問題は [Memento](memento.html) パターンで[、]{.chpule2}[
]{.chpuri2}軽減できます[。]{.chpule2}[ ]{.chpuri2}

二番目は[、]{.chpule2}[ ]{.chpuri2}状態バックアップはかなり多くの RAM
を消費するかもしれないということです[。]{.chpule2}[
]{.chpuri2}したがって[、]{.chpule2}[ ]{.chpuri2}時には[、]{.chpule2}[
]{.chpuri2}過去の状態を復元する代わりに[、]{.chpule2}[
]{.chpuri2}コマンドは逆操作を実行する[、]{.chpule2}[
]{.chpuri2}という代替実装に頼る必要があります[。]{.chpule2}[
]{.chpuri2}逆操作にも問題があります[。]{.chpule2}[
]{.chpuri2}実装が結果的に極めて困難であったり[、]{.chpule2}[
]{.chpuri2}不可能であるという結論に至る可能性があります[。]{.chpule2}[
]{.chpuri2}
:::
:::::::::
::::::::::

::: {.section .checklist}
##  実装方法 {#checklist}

1.  実行メソッドを一つだけ持つコマンド・インターフェースを宣言します[。]{.chpule2}[
    ]{.chpuri2}

2.  コマンド・インターフェースを実装する具象コマンド・クラスにリクエストを抽出します[。]{.chpule2}[
    ]{.chpuri2}各クラスは[、]{.chpule2}[
    ]{.chpuri2}リクエストの引数と[、]{.chpule2}[
    ]{.chpuri2}実際の受け取りオブジェクトへの参照を格納するためのフィールドをいくつか持っている必要があります[。]{.chpule2}[
    ]{.chpuri2}これら値はすべて[、]{.chpule2}[
    ]{.chpuri2}コマンドのコンストラクターを通して初期化されなければなりません[。]{.chpule2}[
    ]{.chpuri2}

3.  *[送]{.cjk}[り]{.cjk}[手]{.cjk}*として機能するクラスをいくつか探し出します[。]{.chpule2}[
    ]{.chpuri2}これらのクラスにコマンドを保存するためのフィールドを追加します[。]{.chpule2}[
    ]{.chpuri2}送り手は[、]{.chpule2}[
    ]{.chpuri2}コマンド・インターフェースを通してのみコマンドと通信する必要があります[。]{.chpule2}[
    ]{.chpuri2}送り手は通常[、]{.chpule2}[
    ]{.chpuri2}自身でコマンド・オブジェクトを作成することはせず[、]{.chpule2}[
    ]{.chpuri2}クライアント・コードから取得します[。]{.chpule2}[
    ]{.chpuri2}

4.  送り手を変更して[、]{.chpule2}[
    ]{.chpuri2}直接リクエストを受け手に送る代わりに[、]{.chpule2}[
    ]{.chpuri2}コマンドを実行するようにします[。]{.chpule2}[ ]{.chpuri2}

5.  クライアントは以下の順序でオブジェクトを初期化します[：]{.chpule2}[
    ]{.chpuri2}

    -   受け手を作成[。]{.chpule2}[ ]{.chpuri2}
    -   コマンドを作成し[、]{.chpule2}[
        ]{.chpuri2}必要に応じて受け手と関連付ける[。]{.chpule2}[
        ]{.chpuri2}
    -   送り手を作成し[、]{.chpule2}[
        ]{.chpuri2}特定のコマンドに関連付ける[。]{.chpule2}[ ]{.chpuri2}
:::

:::::: {.section .pros-cons}
##  長所と短所 {#pros-cons}

::::: row
::: col-sm-6
-   
    *[単]{.cjk}[一]{.cjk}[責]{.cjk}[任]{.cjk}[の]{.cjk}[原]{.cjk}[則]{.cjk}*[。]{.chpule2}[
    ]{.chpuri2}処理を起動するクラスを[、]{.chpule2}[
    ]{.chpuri2}実際に処理をするクラスから分離可能[。]{.chpule2}[
    ]{.chpuri2}
-   
    *[開]{.cjk}[放]{.cjk}[閉]{.cjk}[鎖]{.cjk}[の]{.cjk}[原]{.cjk}[則]{.cjk}*[。]{.chpule2}[
    ]{.chpuri2}新規コマンドをアプリに導入しても[、]{.chpule2}[
    ]{.chpuri2}既存クライアント側コードは問題なく動作する[。]{.chpule2}[
    ]{.chpuri2}
-    取り消しと再実行を実装可能[。]{.chpule2}[ ]{.chpuri2}
-    操作の遅延実行を実装可能[。]{.chpule2}[ ]{.chpuri2}
-    単純なコマンドを束ねて複雑なコマンドの作成可能[。]{.chpule2}[
    ]{.chpuri2}
:::

::: col-sm-6
-    送り手と受け手の間で新しい層を導入するため[、]{.chpule2}[
    ]{.chpuri2}コードが複雑化する可能性[。]{.chpule2}[ ]{.chpuri2}
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
    のハンドラーは[、]{.chpule2}[ ]{.chpuri2}[Commands](command.html)
    で実装可能です[。]{.chpule2}[ ]{.chpuri2}この場合[、]{.chpule2}[
    ]{.chpuri2}リクエストに代表される同一のコンテキスト・オブジェクトに対して多くの異なる処理を実行できます[。]{.chpule2}[
    ]{.chpuri2}

    しかしもう一つのやり方は[、]{.chpule2}[
    ]{.chpuri2}リクエスト自身を*[コ]{.cjk}[マ]{.cjk}[ン]{.cjk}[ド]{.cjk}*・オブジェクトとすることです[。]{.chpule2}[
    ]{.chpuri2}この場合[、]{.chpule2}[
    ]{.chpuri2}連鎖にリンクされた異なる一連のコンテキスト中で同じ処理を実行できます[。]{.chpule2}[
    ]{.chpuri2}

-   [Command](command.html) と [Memento](memento.html)
    とを一緒に使用して[[、]{.ls-1}]{.chpule2}[ ]{.chpuri2}​[
    ]{.chpule1}[「]{.chpuri1}取り消す[」]{.chpule2}[
    ]{.chpuri2}を実装可能です[。]{.chpule2}[
    ]{.chpuri2}この場合[、]{.chpule2}[
    ]{.chpuri2}コマンドは[、]{.chpule2}[
    ]{.chpuri2}一つのターゲット・オブジェクトに対して異なる操作を実行する責任を負い[、]{.chpule2}[
    ]{.chpuri2}メメントは[、]{.chpule2}[
    ]{.chpuri2}コマンド実行前にオブジェクトの状態を保存します[。]{.chpule2}[
    ]{.chpuri2}

-   [Command](command.html) と [Strategy](strategy.html)
    は[、]{.chpule2}[
    ]{.chpuri2}オブジェクトを何らかの操作でパラメーター化できるため[、]{.chpule2}[
    ]{.chpuri2}似たように見えます[。]{.chpule2}[
    ]{.chpuri2}しかし[、]{.chpule2}[
    ]{.chpuri2}この二つはまったく異なる意図を持っています[。]{.chpule2}[
    ]{.chpuri2}

    -   *Command* を使用して[、]{.chpule2}[
        ]{.chpuri2}任意の操作をオブジェクトに変換できます[。]{.chpule2}[
        ]{.chpuri2}操作のパラメーターは[、]{.chpule2}[
        ]{.chpuri2}そのオブジェクトのフィールドになります[。]{.chpule2}[
        ]{.chpuri2}変換により[、]{.chpule2}[
        ]{.chpuri2}操作の実行を延期したり[、]{.chpule2}[
        ]{.chpuri2}キューに入れたり[、]{.chpule2}[
        ]{.chpuri2}コマンド履歴を保存したり[、]{.chpule2}[
        ]{.chpuri2}遠隔サービスにコマンドを送信したりできます[。]{.chpule2}[
        ]{.chpuri2}

    -   一方[、]{.chpule2}[ ]{.chpuri2}*Strategy* は通常[、]{.chpule2}[
        ]{.chpuri2}同じことを行う異なる方法に関するものです[。]{.chpule2}[
        ]{.chpuri2}単一のコンテキスト・クラス内でアルゴリズムを入れ替えることができます[。]{.chpule2}[
        ]{.chpuri2}

-   [Prototype](prototype.html) は[、]{.chpule2}[
    ]{.chpuri2}[Commands](command.html)
    のコピーを履歴に保存する必要がある場合に役立ちます[。]{.chpule2}[
    ]{.chpuri2}

-   [Visitor](visitor.html) を [Command](command.html)
    パターンのより強力なものとして扱うことができます[。]{.chpule2}[
    ]{.chpuri2}そのオブジェクトは[、]{.chpule2}[
    ]{.chpuri2}異なるクラスの様々なオブジェクトに対して操作を実行できます[。]{.chpule2}[
    ]{.chpuri2}
:::

::: {.section .implementations}
##  コード例 {#implementations}

[![Command を C#
で](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](command/csharp/example.html "Command を C# で"){.prog-lang-link}
[![Command を C++
で](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](command/cpp/example.html "Command を C++ で"){.prog-lang-link}
[![Command を Go
で](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](command/go/example.html "Command を Go で"){.prog-lang-link}
[![Command を Java
で](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](command/java/example.html "Command を Java で"){.prog-lang-link}
[![Command を PHP
で](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](command/php/example.html "Command を PHP で"){.prog-lang-link}
[![Command を Python
で](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](command/python/example.html "Command を Python で"){.prog-lang-link}
[![Command を Ruby
で](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](command/ruby/example.html "Command を Ruby で"){.prog-lang-link}
[![Command を Rust
で](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](command/rust/example.html "Command を Rust で"){.prog-lang-link}
[![Command を Swift
で](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](command/swift/example.html "Command を Swift で"){.prog-lang-link}
[![Command を TypeScript
で](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](command/typescript/example.html "Command を TypeScript で"){.prog-lang-link}
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

[Iterator []{.fa .fa-arrow-right}](iterator.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### 戻る

[[]{.fa .fa-arrow-left} Chain of
Responsibility](chain-of-responsibility.html){.btn .btn-default
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
