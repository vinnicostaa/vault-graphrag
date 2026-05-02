::::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} /
[デザインパターン](../design-patterns.html){.type} /
[振る舞いに関するパターン](behavioral-patterns.html){.type}
:::

# Memento {#memento .title}

::: aka
別名：[Snapshot、]{style="display:inline-block"}[メメント、]{style="display:inline-block"}[スナップショット]{style="display:inline-block"}
:::

::: {.section .intent}
##  一言でいうと {#intent}

**Memento**[ ]{.chpule1}[（]{.chpuri1}メメント[、]{.chpule2}[
]{.chpuri2}形見[）]{.chpule2}[ ]{.chpuri2}は[、]{.chpule2}[
]{.chpuri2}振る舞いに関するデザインパターンの一つで[、]{.chpule2}[
]{.chpuri2}オブジェクトの以前の状態を保存し復元することを[、]{.chpule2}[
]{.chpuri2}実装の詳細を明かさずに行います[。]{.chpule2}[ ]{.chpuri2}

<figure class="image">
<img
src="../../images/patterns/content/memento/memento-jabbc8.png?id=1a1a163e752f1f1c587f05032957746d"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/memento/memento-ja.png?id=1a1a163e752f1f1c587f05032957746d 640w,/images/patterns/content/memento/memento-ja-1.5x.png?id=6bef4278e3f290189a11a111ad0115d5 960w,/images/patterns/content/memento/memento-ja-2x.png?id=d4a724013ff8d2e1691475b519b9f1c1 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Memento デザインパターン" />
</figure>
:::

::: {.section .problem}
##  問題 {#problem}

テキスト・エディターのアプリを作成しているところを想像してみてください[。]{.chpule2}[
]{.chpuri2}このエディターは[、]{.chpule2}[
]{.chpuri2}簡単なテキスト編集に加えて[、]{.chpule2}[
]{.chpuri2}テキストの書式設定[、]{.chpule2}[
]{.chpuri2}インライン画像の挿入などができます[。]{.chpule2}[ ]{.chpuri2}

ある時点で[、]{.chpule2}[
]{.chpuri2}テキストに対して実行された操作をユーザーが元に戻せるようにしようと決めました[。]{.chpule2}[
]{.chpuri2}この機能は長年にわたって非常に一般的になっており[、]{.chpule2}[
]{.chpuri2}最近ではすべてのアプリにあるのが当たり前です[。]{.chpule2}[
]{.chpuri2}実装に関しては[、]{.chpule2}[
]{.chpuri2}直接的な方法を選択しました[。]{.chpule2}[
]{.chpuri2}操作を行う前に[、]{.chpule2}[
]{.chpuri2}アプリはすべてのオブジェクトの状態を記録し[、]{.chpule2}[
]{.chpuri2}どこかに置いておきます[。]{.chpule2}[
]{.chpuri2}その後ユーザーが操作を取り消したいと思ったら[、]{.chpule2}[
]{.chpuri2}アプリは履歴から最新のスナップショットを取得し[、]{.chpule2}[
]{.chpuri2}それを使ってすべてのオブジェクトの状態を元に戻します[。]{.chpule2}[
]{.chpuri2}

<figure class="image">
<img
src="../../images/patterns/diagrams/memento/problem1-jabc04.png?id=853f9a144beceaff82e439cf24004cbb"
style="aspect-ratio: 2.61;"
srcset="/images/patterns/diagrams/memento/problem1-ja.png?id=853f9a144beceaff82e439cf24004cbb 470w,/images/patterns/diagrams/memento/problem1-ja-1.5x.png?id=6ddcc453f57afc5bb68ab5ee66572b8c 705w,/images/patterns/diagrams/memento/problem1-ja-2x.png?id=ae9af152f7d397773079b36835229df5 940w"
sizes="(max-width: 720px) 100vw, 470px" loading="lazy" width="470"
alt="エディター内の操作の取り消し" />
<figcaption><p>操作実行前に<span class="chpule2">、</span><span
class="chpuri2">
</span>アプリはオブジェクトの状態のスナップショットを保存<span
class="chpule2">。</span><span class="chpuri2">
</span>それを使ってオブジェクトを前の状態に復元<span
class="chpule2">。</span><span class="chpuri2"> </span></p></figcaption>
</figure>

状態のスナップショットについて考えてみましょう[。]{.chpule2}[
]{.chpuri2}一体全体[、]{.chpule2}[
]{.chpuri2}どうやってそんなもの作ればいいでしょうか[？]{.chpule2}[
]{.chpuri2}多分[、]{.chpule2}[
]{.chpuri2}オブジェクトの全フィールドを巡って[、]{.chpule2}[
]{.chpuri2}その値を一つずつ記憶装置にコピーする必要があるでしょう[。]{.chpule2}[
]{.chpuri2}でもこの方法は[、]{.chpule2}[
]{.chpuri2}オブジェクトがその内容のアクセス制限について随分と寛容でないと[、]{.chpule2}[
]{.chpuri2}うまくいきません[。]{.chpule2}[
]{.chpuri2}残念ながら[、]{.chpule2}[
]{.chpuri2}ほとんどの現実のオブジェクトは[、]{.chpule2}[
]{.chpuri2}その中身を他のオブジェクトに簡単に覗かれないようになっています[。]{.chpule2}[
]{.chpuri2}重要な全データは[、]{.chpule2}[
]{.chpuri2}非公開フィールドにしまわれています[。]{.chpule2}[ ]{.chpuri2}

とりあえずその問題は無視して[、]{.chpule2}[
]{.chpuri2}オブジェクトがヒッピーのように振る舞うと仮定しましょう[。]{.chpule2}[
]{.chpuri2}つまり[、]{.chpule2}[
]{.chpuri2}開かれた関係を好み[、]{.chpule2}[
]{.chpuri2}状態はあけっぴろげです[。]{.chpule2}[
]{.chpuri2}この方法で[、]{.chpule2}[
]{.chpuri2}当座の問題を解決され[、]{.chpule2}[
]{.chpuri2}オブジェクトの状態のスナップショットを好きなように生成できますが[、]{.chpule2}[
]{.chpuri2}それでも深刻な問題があります[。]{.chpule2}[
]{.chpuri2}将来[、]{.chpule2}[
]{.chpuri2}エディターのクラスの一部をリファクタリングするとか[、]{.chpule2}[
]{.chpuri2}フィールドの一部を追加または削除するかもしれない[、]{.chpule2}[
]{.chpuri2}ということです[。]{.chpule2}[
]{.chpuri2}簡単に聞こえますが[、]{.chpule2}[
]{.chpuri2}影響を受けるオブジェクトの状態をコピーするためのクラスも変更する必要が出てきます[。]{.chpule2}[
]{.chpuri2}

<figure class="image">
<img
src="../../images/patterns/diagrams/memento/problem2-jab079.png?id=6da579422e6113924b9278dae914b44b"
style="aspect-ratio: 1.48;"
srcset="/images/patterns/diagrams/memento/problem2-ja.png?id=6da579422e6113924b9278dae914b44b 310w,/images/patterns/diagrams/memento/problem2-ja-1.5x.png?id=283aaac7cc7e8cc5bd66e20b110b042b 465w,/images/patterns/diagrams/memento/problem2-ja-2x.png?id=9a7ab5f478123e15122a81859eee03a5 620w"
sizes="(max-width: 720px) 100vw, 310px" loading="lazy" width="310"
alt="オブジェクトの非公開の状態のコピーをどう作成するか？" />
<figcaption><p>オブジェクトの非公開の状態のコピーをどう作成するか<span
class="chpule2">？</span><span class="chpuri2"> </span></p></figcaption>
</figure>

しかし[、]{.chpule2}[ ]{.chpuri2}まだまだあります[。]{.chpule2}[
]{.chpuri2}エディターの状態の実際の[
]{.chpule1}[「]{.chpuri1}スナップショット[」]{.chpule2}[
]{.chpuri2}を考えてみましょう[。]{.chpule2}[
]{.chpuri2}何が含まれていますか[？]{.chpule2}[
]{.chpuri2}最低でも[、]{.chpule2}[
]{.chpuri2}実際のテキスト[、]{.chpule2}[
]{.chpuri2}カーソル座標[、]{.chpule2}[
]{.chpuri2}現在のスクロール位置などを含める必要があります[。]{.chpule2}[
]{.chpuri2}スナップショットを作成するには[、]{.chpule2}[
]{.chpuri2}これらの値を収集し[、]{.chpule2}[
]{.chpuri2}ある種のコンテナに入れる必要があります[。]{.chpule2}[
]{.chpuri2}

最もありがちなのは[、]{.chpule2}[
]{.chpuri2}多数のコンテナ・オブジェクトの履歴を表すリストに格納することでしょう[。]{.chpule2}[
]{.chpuri2}したがって[、]{.chpule2}[
]{.chpuri2}コンテナはおそらくある一つのクラスのオブジェクトになるでしょう[。]{.chpule2}[
]{.chpuri2}このクラスにはほとんどメソッドがなく[、]{.chpule2}[
]{.chpuri2}エディターの状態を反映する多くのフィールドからなります[。]{.chpule2}[
]{.chpuri2}他のオブジェクトがスナップショットにデータを書き込んだり[、]{.chpule2}[
]{.chpuri2}そこから読み取ったりできるようにするため[、]{.chpule2}[
]{.chpuri2}おそらくフィールドを公開する必要があります[。]{.chpule2}[
]{.chpuri2}エディターの状態が非公開かどうかにかかわらず[、]{.chpule2}[
]{.chpuri2}これらのフィールドは公開されることになります[。]{.chpule2}[
]{.chpuri2}他のクラスはスナップショット・クラスのちょっとした変更に依存するようになります[。]{.chpule2}[
]{.chpuri2}本来ならば非公開フィールドやメソッドで行われる外部クラスに影響を与えることない変更なはずなのに[。]{.chpule2}[
]{.chpuri2}

どうやら袋小路に達したようです[。]{.chpule2}[
]{.chpuri2}クラスの内部の詳細を全公開にして脆弱にするか[、]{.chpule2}[
]{.chpuri2}状態へのアクセスを厳しくしてスナップショット生成を不可能にするか[[。]{.ls-1}]{.chpule2}[
]{.chpuri2}​[ ]{.chpule1}[「]{.chpuri1}取り消す[[」]{.ls-1}]{.chpule2}[
]{.chpuri2}​[ ]{.chpule1}[（]{.chpuri1}Windows
では[[、]{.ls-1}]{.chpule2}[ ]{.chpuri2}​[
]{.chpule1}[「]{.chpuri1}元に戻す[」]{.ls-05}[）]{.chpule2}[
]{.chpuri2}を実装する他の方法はないのでしょうか[？]{.chpule2}[
]{.chpuri2}
:::

::: {.section .solution}
##  解決策 {#solution}

お話した問題のすべては[、]{.chpule2}[
]{.chpuri2}カプセル化の失敗が原因です[。]{.chpule2}[
]{.chpuri2}一部のオブジェクトは想定されている以上のことを行おうとしています[。]{.chpule2}[
]{.chpuri2}ある操作を実行するために必要なデータを収集するために[、]{.chpule2}[
]{.chpuri2}実際の操作の実行を他のオブジェクトにまかせる代わりに[、]{.chpule2}[
]{.chpuri2}それらのオブジェクトのプライバシーを侵害しています[。]{.chpule2}[
]{.chpuri2}

Memento パターンでは[、]{.chpule2}[
]{.chpuri2}状態のスナップショットの作成を[、]{.chpule2}[
]{.chpuri2}状態の実際の所有者である
*[オ]{.cjk}[リ]{.cjk}[ジ]{.cjk}[ネ]{.cjk}[ー]{.cjk}[タ]{.cjk}[ー]{.cjk}*[
]{.chpule1}[（]{.chpuri1}Originator[、]{.chpule2}[
]{.chpuri2}発起人[）]{.chpule2}[ ]{.chpuri2}に任せます[。]{.chpule2}[
]{.chpuri2}したがって[、]{.chpule2}[ ]{.chpuri2}他のオブジェクトが[
]{.chpule1}[「]{.chpuri1}外部から[」]{.chpule2}[
]{.chpuri2}エディターの状態をコピーしようとする代わりに[、]{.chpule2}[
]{.chpuri2}自身の状態へ完全にアクセスできる[、]{.chpule2}[
]{.chpuri2}エディターのクラスそのものがスナップショットを作成します[。]{.chpule2}[
]{.chpuri2}

パターンによると[、]{.chpule2}[
]{.chpuri2}オブジェクトの状態のコピーは[、]{.chpule2}[
]{.chpuri2}*[メ]{.cjk}[メ]{.cjk}[ン]{.cjk}[ト]{.cjk}*[
]{.chpule1}[（]{.chpuri1}Memento[）]{.chpule2}[
]{.chpuri2}という特別なオブジェクトに格納します[。]{.chpule2}[
]{.chpuri2}メメントの内容は[、]{.chpule2}[
]{.chpuri2}いかなる他のオブジェクトからもアクセスできません[。]{.chpule2}[
]{.chpuri2}他のオブジェクトは[、]{.chpule2}[
]{.chpuri2}スナップショットのメタデータ[
]{.chpule1}[（]{.chpuri1}作成時刻[、]{.chpule2}[
]{.chpuri2}実行した操作の名前など[）]{.chpule2}[
]{.chpuri2}の取得を許す限られたインターフェースを使用してメメントと通信する必要があります[。]{.chpule2}[
]{.chpuri2}スナップショットに含まれる元のオブジェクトの状態の取得は許されていません[。]{.chpule2}[
]{.chpuri2}

<figure class="image">
<img
src="../../images/patterns/diagrams/memento/solution-ja650b.png?id=7ee9f3461a9c8428e66320aad2be07fc"
style="aspect-ratio: 1.36;"
srcset="/images/patterns/diagrams/memento/solution-ja.png?id=7ee9f3461a9c8428e66320aad2be07fc 610w,/images/patterns/diagrams/memento/solution-ja-1.5x.png?id=234a41bd24a453ebd7541879bdf2f639 915w,/images/patterns/diagrams/memento/solution-ja-2x.png?id=f5a46ed5518ae931ec2731be4a04761a 1220w"
sizes="(max-width: 720px) 100vw, 610px" loading="lazy" width="610"
alt="オリジネーターはメメントへの完全なアクセス権を持つが、ケアテーカーはメタデータにのみアクセス可。" />
<figcaption><p>オリジネーターはメメントへの完全なアクセス権を持つが<span
class="chpule2">、</span><span class="chpuri2">
</span>ケアテーカーはメタデータにのみアクセス可<span
class="chpule2">。</span><span class="chpuri2"> </span></p></figcaption>
</figure>

このような制限されたポリシーにより[、]{.chpule2}[
]{.chpuri2}メメントは[、]{.chpule2}[ ]{.chpuri2}通常[、]{.chpule2}[
]{.chpuri2}*[ケ]{.cjk}[ア]{.cjk}[テ]{.cjk}[ー]{.cjk}[カ]{.cjk}[ー]{.cjk}*[
]{.chpule1}[（]{.chpuri1}Caretaker[）]{.chpule2}[
]{.chpuri2}と呼ばれる他のオブジェクトに保存します[。]{.chpule2}[
]{.chpuri2}ケアテーカーは限られたインターフェースのみでメメントとやりとりをするので[、]{.chpule2}[
]{.chpuri2}メメントの中に保存されている状態を改竄かいざんすることはできません[。]{.chpule2}[
]{.chpuri2}同時に[、]{.chpule2}[
]{.chpuri2}オリジネーターは[、]{.chpule2}[
]{.chpuri2}メメント内のすべてのフィールドにアクセスすることができ[、]{.chpule2}[
]{.chpuri2}意のままに以前の状態を復元することができます[。]{.chpule2}[
]{.chpuri2}

テキストエディターの例では[、]{.chpule2}[
]{.chpuri2}ケアーテーカーとして機能する個別の履歴クラスを作成できます[。]{.chpule2}[
]{.chpuri2}ケアーテーカー内に格納されたメメントのスタックは[、]{.chpule2}[
]{.chpuri2}一つの操作が行われる直前に成長します[。]{.chpule2}[
]{.chpuri2}アプリの UI 内でこのスタックの内容を描画し[、]{.chpule2}[
]{.chpuri2}以前に実行された操作の履歴をユーザーに見せることすらできます[。]{.chpule2}[
]{.chpuri2}

ユーザーが[ ]{.chpule1}[「]{.chpuri1}取り消す[」]{.chpule2}[
]{.chpuri2}を実行すると[、]{.chpule2}[
]{.chpuri2}履歴オブジェクトがスタックから最新のメメントを拾って来て[、]{.chpule2}[
]{.chpuri2}エディターに渡し[、]{.chpule2}[
]{.chpuri2}ロールバックを要求します[。]{.chpule2}[
]{.chpuri2}エディターは[、]{.chpule2}[
]{.chpuri2}メメントへの完全なアクセス権を持っているので[、]{.chpule2}[
]{.chpuri2}自分の状態をメメントから取得した値に変更します[。]{.chpule2}[
]{.chpuri2}
:::

:::::::::: {.section .structure-container}
##  構造 {#structure}

::::::::: structure
#### ネストされたクラスに基づく実装

このパターンの古典的な実装は[、]{.chpule2}[
]{.chpuri2}C++[、]{.chpule2}[ ]{.chpuri2}C#[、]{.chpule2}[
]{.chpuri2}Java
など多くの普及しているプログラミング言語で利用可能な[、]{.chpule2}[
]{.chpuri2}ネストされた[
]{.chpule1}[（]{.chpuri1}入れ子の[）]{.chpule2}[
]{.chpuri2}クラスのサポートに依存しています[。]{.chpule2}[ ]{.chpuri2}

:::: memento-classic
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/memento/structure180c1.png?id=4b4a42363a005b617d4df06689787385"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.87;"
srcset="/images/patterns/diagrams/memento/structure1.png?id=4b4a42363a005b617d4df06689787385 580w,/images/patterns/diagrams/memento/structure1-1.5x.png?id=82cf757f153840c85d27bd63f3f3e133 870w,/images/patterns/diagrams/memento/structure1-2x.png?id=d4e77295e51c2417f22b7abb396d5977 1160w"
sizes="(max-width: 720px) 100vw, 580px" loading="lazy" width="580"
alt="ネストしたクラスによる Memento の実装" /><img
src="../../images/patterns/diagrams/memento/structure1-indexeda7c3.png?id=f79a8356b087ae6b004aec42b787ae2e"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.87;"
srcset="/images/patterns/diagrams/memento/structure1-indexed.png?id=f79a8356b087ae6b004aec42b787ae2e 580w,/images/patterns/diagrams/memento/structure1-indexed-1.5x.png?id=0687dc84dd4c98b4591a70ebd05c4d8e 870w,/images/patterns/diagrams/memento/structure1-indexed-2x.png?id=62fea7bdbc96420568869ea3bd25f6ad 1160w"
sizes="(max-width: 720px) 100vw, 580px" loading="lazy" width="580"
alt="ネストしたクラスによる Memento の実装" />
</figure>
:::
::::

1.  **オリジネーター**[
    ]{.chpule1}[（]{.chpuri1}Originator[）]{.chpule2}[
    ]{.chpuri2}クラスは[、]{.chpule2}[
    ]{.chpuri2}自身の状態のスナップショットを作成することと[、]{.chpule2}[
    ]{.chpuri2}必要に応じてスナップショットから状態を復元することができます[。]{.chpule2}[
    ]{.chpuri2}

2.  **メメント**[ ]{.chpule1}[（]{.chpuri1}Memento[）]{.chpule2}[
    ]{.chpuri2}は[、]{.chpule2}[
    ]{.chpuri2}オリジネーターの状態のスナップショットとして機能する値オブジェクトです[。]{.chpule2}[
    ]{.chpuri2}メメント・オブジェクトは変更不可とし[、]{.chpule2}[
    ]{.chpuri2}コンストラクターを介して一度だけデータを渡すことが[、]{.chpule2}[
    ]{.chpuri2}よく行われるやり方です[。]{.chpule2}[ ]{.chpuri2}

3.  **Caretaker** は[、]{.chpule2}[ ]{.chpuri2}いつ[、]{.chpule2}[
    ]{.chpuri2}どうして[、]{.chpule2}[
    ]{.chpuri2}オリジネーターの状態を獲得すべきかを知っているだけでなく[、]{.chpule2}[
    ]{.chpuri2}いつ状態を復元すべきかも知っています[。]{.chpule2}[
    ]{.chpuri2}

    ケアテーカーは[、]{.chpule2}[
    ]{.chpuri2}メメントのスタックを保存することによって[、]{.chpule2}[
    ]{.chpuri2}オリジネーターの履歴を追跡することができます[。]{.chpule2}[
    ]{.chpuri2}オリジネーターが履歴をさかのぼる必要のある時は[、]{.chpule2}[
    ]{.chpuri2}スタックから一番上のメメントを取り出し[、]{.chpule2}[
    ]{.chpuri2}オリジネーターの復元メソッドに渡します[。]{.chpule2}[
    ]{.chpuri2}

4.  この実装では[、]{.chpule2}[
    ]{.chpuri2}メメントのクラスはオリジネーターの中にネストされます[。]{.chpule2}[
    ]{.chpuri2}これにより[、]{.chpule2}[
    ]{.chpuri2}オリジネーターは非公開と宣言されているメメントのフィールドとメソッドにアクセスできます[。]{.chpule2}[
    ]{.chpuri2}一方[、]{.chpule2}[
    ]{.chpuri2}ケアテーカーからのメメントのフィールドやメソッドへのアクセスは非常に制限されており[、]{.chpule2}[
    ]{.chpuri2}スタックにメメントを収めることはできますが[、]{.chpule2}[
    ]{.chpuri2}状態の改竄はできません[。]{.chpule2}[ ]{.chpuri2}

#### 中間的なインターフェースに基づく実装

クラスのネストをサポートしない言語[
]{.chpule1}[（]{.chpuri1}おい[、]{.chpule2}[
]{.chpuri2}php[、]{.chpule2}[ ]{.chpuri2}お前のことだよ[）]{.chpule2}[
]{.chpuri2}に適した[、]{.chpule2}[
]{.chpuri2}代替実装方法があります[。]{.chpule2}[ ]{.chpuri2}

:::: memento-empty-interface
::: {.struct-image2 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/memento/structure21ec9.png?id=fcff71cb648389be2e302fbe55e2f847"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/diagrams/memento/structure2.png?id=fcff71cb648389be2e302fbe55e2f847 560w,/images/patterns/diagrams/memento/structure2-1.5x.png?id=5c05495a7ca6545fcc57f70ea7ee904a 840w,/images/patterns/diagrams/memento/structure2-2x.png?id=aa7fb5d0f622d4344a2cb590f437f8c8 1120w"
sizes="(max-width: 720px) 100vw, 560px" loading="lazy" width="560"
alt="クラスのネストなしの Memento" /><img
src="../../images/patterns/diagrams/memento/structure2-indexede045.png?id=2c98b4f64b03f2a30e159de31ca9f718"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.6;"
srcset="/images/patterns/diagrams/memento/structure2-indexed.png?id=2c98b4f64b03f2a30e159de31ca9f718 560w,/images/patterns/diagrams/memento/structure2-indexed-1.5x.png?id=1ba6e0f22bb613f3f1dcf86850c3b604 840w,/images/patterns/diagrams/memento/structure2-indexed-2x.png?id=2fb637daef1110dfa89f15b2d4627894 1120w"
sizes="(max-width: 720px) 100vw, 560px" loading="lazy" width="560"
alt="クラスのネストなしの Memento" />
</figure>
:::
::::

1.  クラスのネストができない場合[、]{.chpule2}[
    ]{.chpuri2}ケアテーカーは明示的に宣言された中間的インターフェースを通してのみメメントにアクセスできる[、]{.chpule2}[
    ]{.chpuri2}という決まりごとを作ることにより[、]{.chpule2}[
    ]{.chpuri2}メメントのフィールドへのアクセスの制限が可能です[。]{.chpule2}[
    ]{.chpuri2}このインターフェースは[、]{.chpule2}[
    ]{.chpuri2}メメントのメタデータに関連するメソッドだけを宣言します[。]{.chpule2}[
    ]{.chpuri2}

2.  一方[、]{.chpule2}[ ]{.chpuri2}オリジネーターは[、]{.chpule2}[
    ]{.chpuri2}メメント・クラスで宣言されたフィールドやメソッドに直接アクセスして[、]{.chpule2}[
    ]{.chpuri2}メメント・オブジェクトを扱うことができます[。]{.chpule2}[
    ]{.chpuri2}このやり方の弱点は[、]{.chpule2}[
    ]{.chpuri2}メメントのすべてのメンバーを公開と宣言する必要があるということです[。]{.chpule2}[
    ]{.chpuri2}

#### さらに厳格なカプセル化による実装

メメントを介して他のクラスがオリジネーターの状態にアクセスできるかもしれないわずかな可能性も許容できない場合に役に立つ[、]{.chpule2}[
]{.chpuri2}もう一つの実装方法があります[。]{.chpule2}[ ]{.chpuri2}

:::: memento-safe
::: {.struct-image3 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/memento/structure3f138.png?id=6c3ef6a916be8c8ec6d6659d19a6a79f"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.74;"
srcset="/images/patterns/diagrams/memento/structure3.png?id=6c3ef6a916be8c8ec6d6659d19a6a79f 590w,/images/patterns/diagrams/memento/structure3-1.5x.png?id=9bb6d9dd5567bc71d9e93efc9183ef3a 885w,/images/patterns/diagrams/memento/structure3-2x.png?id=988c37f92059457153d26ba3458d371e 1180w"
sizes="(max-width: 720px) 100vw, 590px" loading="lazy" width="590"
alt="厳格なカプセル化による実装" /><img
src="../../images/patterns/diagrams/memento/structure3-indexedadf8.png?id=17e84b0ef89a41bb3fb844c8d7a445ad"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.74;"
srcset="/images/patterns/diagrams/memento/structure3-indexed.png?id=17e84b0ef89a41bb3fb844c8d7a445ad 590w,/images/patterns/diagrams/memento/structure3-indexed-1.5x.png?id=c121c75333433b70b9a67b75e536214c 885w,/images/patterns/diagrams/memento/structure3-indexed-2x.png?id=fef9ae2a0151c105976075aafb8939dd 1180w"
sizes="(max-width: 720px) 100vw, 590px" loading="lazy" width="590"
alt="厳格なカプセル化による実装" />
</figure>
:::
::::

1.  この実装では[、]{.chpule2}[
    ]{.chpuri2}複数の種類のオリジネーターとメメントの扱いが可能です[。]{.chpule2}[
    ]{.chpuri2}それぞれのオリジネーターは[、]{.chpule2}[
    ]{.chpuri2}対応するメメント・クラスと一緒に機能します[。]{.chpule2}[
    ]{.chpuri2}オリジネーターもメメントも状態を誰にも漏らしません[。]{.chpule2}[
    ]{.chpuri2}

2.  ケアテーカーがメメント内の状態を変更することは[、]{.chpule2}[
    ]{.chpuri2}今や明示的に制限されています[。]{.chpule2}[
    ]{.chpuri2}さらに[、]{.chpule2}[
    ]{.chpuri2}復元メソッドはメメント・クラス内で定義されているため[、]{.chpule2}[
    ]{.chpuri2}ケアテーカー・クラスは[、]{.chpule2}[
    ]{.chpuri2}オリジネーターから独立しています[。]{.chpule2}[
    ]{.chpuri2}

3.  それぞれのメメントは[、]{.chpule2}[
    ]{.chpuri2}それを生成したオリジネーターにリンクされます[。]{.chpule2}[
    ]{.chpuri2}オリジネーターは[、]{.chpule2}[
    ]{.chpuri2}自身を[、]{.chpule2}[
    ]{.chpuri2}その状態の値とともにメメントのコンストラクターに渡します[。]{.chpule2}[
    ]{.chpuri2}これらのクラス間の緊密な連携により[、]{.chpule2}[
    ]{.chpuri2}メメントはオリジネーターの状態を復活させることができます[。]{.chpule2}[
    ]{.chpuri2}ただし[、]{.chpule2}[ ]{.chpuri2}後者が適切な setter
    を定義していることが条件です[。]{.chpule2}[ ]{.chpuri2}
:::::::::
::::::::::

::: {.section .pseudocode}
##  擬似コード {#pseudocode}

この例では[、]{.chpule2}[
]{.chpuri2}複雑なテキスト・エディターの状態のスナップショットを記録し[、]{.chpule2}[
]{.chpuri2}必要な時にこれらのスナップショットから以前の状態を復元するために[、]{.chpule2}[
]{.chpuri2}Memento パターンを [Command](command.html)
パターンと一緒に活用しています[。]{.chpule2}[ ]{.chpuri2}

<figure class="image">
<img
src="../../images/patterns/diagrams/memento/example4526.png?id=fb2196b065f374a1c2a64a0943463760"
style="aspect-ratio: 4.29;"
srcset="/images/patterns/diagrams/memento/example.png?id=fb2196b065f374a1c2a64a0943463760 600w,/images/patterns/diagrams/memento/example-1.5x.png?id=174b686f918a2c49e2545d5573c52d42 900w,/images/patterns/diagrams/memento/example-2x.png?id=41a73f3cc22bc3dd180f53e6968974d4 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Memento 例の構造" />
<figcaption><p>テキストエディターの状態のスナップショットを保存する<span
class="chpule2">。</span><span class="chpuri2"> </span></p></figcaption>
</figure>

コマンド・オブジェクトがケアテーカーとして機能します[。]{.chpule2}[
]{.chpuri2}コマンド・オブジェクトは[、]{.chpule2}[
]{.chpuri2}コマンドに関連した実作業を実行する前に[、]{.chpule2}[
]{.chpuri2}エディターのメメントを獲得します[。]{.chpule2}[
]{.chpuri2}ユーザーが最新のコマンドを取り消そうとする時[、]{.chpule2}[
]{.chpuri2}エディターはそのコマンドに保存されたメメントを使って[、]{.chpule2}[
]{.chpuri2}自身を前の状態に戻すことができます[。]{.chpule2}[ ]{.chpuri2}

メメント・クラスは[、]{.chpule2}[ ]{.chpuri2}公開フィールドも setter も
getter も宣言しません[。]{.chpule2}[ ]{.chpuri2}そのため[、]{.chpule2}[
]{.chpuri2}いかなるオブジェクトも[、]{.chpule2}[
]{.chpuri2}メメントの内容を変更することはできません[。]{.chpule2}[
]{.chpuri2}メメントは[、]{.chpule2}[
]{.chpuri2}それを構築したエディター・オブジェクトとリンクされています[。]{.chpule2}[
]{.chpuri2}リンクされたエディターの setter
メソッドを介して[、]{.chpule2}[
]{.chpuri2}メメントはエディターの状態を復元することができます[。]{.chpule2}[
]{.chpuri2}メメントは[、]{.chpule2}[
]{.chpuri2}特定のエディター・オブジェクトにリンクされているため[、]{.chpule2}[
]{.chpuri2}アプリは[、]{.chpule2}[
]{.chpuri2}複数の独立したエディター・ウィンドウを[、]{.chpule2}[
]{.chpuri2}一元化された取り消し用スタックでサポートできます[。]{.chpule2}[
]{.chpuri2}

<figure class="code">
<pre class="code" lang="pseudocode"><code>// オリジネーターは、時間とともに変わる可能性のある重要なデータを保持。ま
// た、その状態をメメント内に保存するためのメソッドと、そこから状態を復元
// するための別のメソッドを定義。
class Editor is
    private field text, curX, curY, selectionWidth

    method setText(text) is
        this.text = text

    method setCursor(x, y) is
        this.curX = x
        this.curY = y

    method setSelectionWidth(width) is
        this.selectionWidth = width

    // 現在の状態をメメントに保存。
    method createSnapshot():Snapshot is
        // メメントは不変オブジェクト。そのため、オリジネーターはその状態
        // をメメントのコンストラクターにパラメーターとして渡す。
        return new Snapshot(this, text, curX, curY, selectionWidth)

// メメント・クラスは、エディターの過去の状態を蓄積。
class Snapshot is
    private field editor: Editor
    private field text, curX, curY, selectionWidth

    constructor Snapshot(editor, text, curX, curY, selectionWidth) is
        this.editor = editor
        this.text = text
        this.curX = x
        this.curY = y
        this.selectionWidth = selectionWidth

    // ある時点で、メメント・オブジェクトを使ってエディターの以前の状態を
    // 復元できる。
    method restore() is
        editor.setText(text)
        editor.setCursor(curX, curY)
        editor.setSelectionWidth(selectionWidth)

// コマンド・オブジェクトは、ケアテーカーとして機能可能。その場合、コマン
// ドはオリジネーターの状態を変更する直前にメメントを取得。取り消し操作の
// 要求が来ると、オリジネーターの状態をメメントから復元。
class Command is
    private field backup: Snapshot

    method makeBackup() is
        backup = editor.createSnapshot()

    method undo() is
        if (backup != null)
            backup.restore()
    // ……</code></pre>
</figure>
:::

:::::::: {.section .applicability-container}
##  適応性 {#applicability}

::::::: applicability
::: applicability-problem
オブジェクトの以前の状態を復元するためにオブジェクトの状態のスナップショットを生成したい場合に
Memento パターンを使用します[。]{.chpule2}[ ]{.chpuri2}
:::

::: applicability-solution
Memento パターンを使用すると[、]{.chpule2}[
]{.chpuri2}非公開フィールドを含むオブジェクトの状態の完全なコピーを作成し[、]{.chpule2}[
]{.chpuri2}オブジェクトから分離して保存できます[。]{.chpule2}[
]{.chpuri2}多くの人たちは[
]{.chpule1}[「]{.chpuri1}取り消す[」]{.chpule2}[
]{.chpuri2}コマンドの使用例のおかげでこのパターンを覚えていますが[、]{.chpule2}[
]{.chpuri2}このパターンは[、]{.chpule2}[ ]{.chpuri2}トランザクション[
]{.chpule1}[（]{.chpuri1}つまりエラー発生時の操作のロールバックが必要な場合[）]{.chpule2}[
]{.chpuri2}を扱う際にも不可欠です[。]{.chpule2}[ ]{.chpuri2}
:::

::: applicability-problem
オブジェクトのフィールドや getter や setter
への直接アクセスがカプセル化に違反する場合は[、]{.chpule2}[
]{.chpuri2}このパターンを使用します[。]{.chpule2}[ ]{.chpuri2}
:::

::: applicability-solution
Memento では[、]{.chpule2}[
]{.chpuri2}オブジェクト自体が状態のスナップショットを作成する責任を負います[。]{.chpule2}[
]{.chpuri2}他のいかなるオブジェクトもスナップショットを読み取ることができません[。]{.chpule2}[
]{.chpuri2}元のオブジェクトの状態データは安全で堅牢に守られます[。]{.chpule2}[
]{.chpuri2}
:::
:::::::
::::::::

::: {.section .checklist}
##  実装方法 {#checklist}

1.  どのクラスがオリジネーターの役割を果たすのかを決定します[。]{.chpule2}[
    ]{.chpuri2}プログラムが[、]{.chpule2}[
    ]{.chpuri2}この型のオブジェクト 1
    個を中心として使うのか[、]{.chpule2}[
    ]{.chpuri2}それとも複数の小さなオブジェクトを使うのかを知ることが重要です[。]{.chpule2}[
    ]{.chpuri2}

2.  メメント・クラスを作成します[。]{.chpule2}[
    ]{.chpuri2}オリジネーター内で宣言されたフィールドに対応するフィールドを一つずつ宣言していきます[。]{.chpule2}[
    ]{.chpuri2}

3.  メメントのクラスを不変とします[。]{.chpule2}[
    ]{.chpuri2}メメントは[、]{.chpule2}[
    ]{.chpuri2}コンストラクターを介してのみデータを受け取ります[。]{.chpule2}[
    ]{.chpuri2}クラスに setter があっってはなりません[。]{.chpule2}[
    ]{.chpuri2}

4.  プログラミング言語がネストされたクラスをサポートしている場合は[、]{.chpule2}[
    ]{.chpuri2}メメントは[、]{.chpule2}[
    ]{.chpuri2}オリジネーターの内部の入れ子クラスにします[。]{.chpule2}[
    ]{.chpuri2}そうでない場合は[、]{.chpule2}[ ]{.chpuri2}空[
    ]{.chpule1}[（]{.chpuri1}から[）]{.chpule2}[
    ]{.chpuri2}のインターフェースを作成し[、]{.chpule2}[
    ]{.chpuri2}メメント・クラスがそれを実装するようにし[、]{.chpule2}[
    ]{.chpuri2}他のすべてのオブジェクトはメメント・クラスではなく[、]{.chpule2}[
    ]{.chpuri2}インターフェースを参照するようにします[。]{.chpule2}[
    ]{.chpuri2}インターフェースにいくつかのメタデータに対するメソッドを追加するのはかまいませんが[、]{.chpule2}[
    ]{.chpuri2}オリジネーターの状態を露呈するものは避けてください[。]{.chpule2}[
    ]{.chpuri2}

5.  オリジネーター・クラスにメメントを生成するメソッドを追加します[。]{.chpule2}[
    ]{.chpuri2}オリジネーターは[、]{.chpule2}[
    ]{.chpuri2}メメントのコンストラクターの 1
    個または複数の引数を介してその状態を渡します[。]{.chpule2}[
    ]{.chpuri2}

    メソッドの戻り値の型は[、]{.chpule2}[ ]{.chpuri2}前段階で抽出した[
    ]{.chpule1}[（]{.chpuri1}抽出したとして[）]{.chpule2}[
    ]{.chpuri2}インターフェースとします[。]{.chpule2}[
    ]{.chpuri2}内部では[、]{.chpule2}[
    ]{.chpuri2}メメントの作成メソッドは
    メメント・クラスと直接やりとりをします[。]{.chpule2}[ ]{.chpuri2}

6.  オリジネーターの状態を復元するメソッドを一つ[、]{.chpule2}[
    ]{.chpuri2}クラスに追加します[。]{.chpule2}[
    ]{.chpuri2}それは[、]{.chpule2}[
    ]{.chpuri2}メメント・オブジェクトを引数として取るようにします[。]{.chpule2}[
    ]{.chpuri2}前段階でインターフェースを抽出した場合は[、]{.chpule2}[
    ]{.chpuri2}それをパラメーターの型とします[。]{.chpule2}[
    ]{.chpuri2}この場合[、]{.chpule2}[
    ]{.chpuri2}受け取るオブジェクトをメメント・クラスに型変換[
    ]{.chpule1}[（]{.chpuri1}キャスト[）]{.chpule2}[
    ]{.chpuri2}する必要があります[。]{.chpule2}[
    ]{.chpuri2}オリジネーターはそのオブジェクトへの完全アクセスが必要だからです[。]{.chpule2}[
    ]{.chpuri2}

7.  ケアテーカー[
    ]{.chpule1}[（]{.chpuri1}コマンド・オブジェクトかもしれませんし[、]{.chpule2}[
    ]{.chpuri2}履歴かもしれませんし[、]{.chpule2}[
    ]{.chpuri2}まったく違うものかもしれません[）]{.chpule2}[
    ]{.chpuri2}は[、]{.chpule2}[
    ]{.chpuri2}オリジネーターにいつ新規メメントを要求するべきか[、]{.chpule2}[
    ]{.chpuri2}それをどう保存するのか[、]{.chpule2}[
    ]{.chpuri2}そしていつ特定のメメントでオリジネーターを復元するべきか[、]{.chpule2}[
    ]{.chpuri2}を知っている必要があります[。]{.chpule2}[ ]{.chpuri2}

8.  ケアテーカーとオリジネーター間のリンクを[、]{.chpule2}[
    ]{.chpuri2}メメントに置いておく場合もあります[。]{.chpule2}[
    ]{.chpuri2}その場合は[、]{.chpule2}[
    ]{.chpuri2}それぞれのメメントは[、]{.chpule2}[
    ]{.chpuri2}それを作成したオリジネーターと接続されている必要があります[。]{.chpule2}[
    ]{.chpuri2}復元メソッドをメメント・クラスに移動する場合もあります[。]{.chpule2}[
    ]{.chpuri2}しかしこれが意味をなすのは[、]{.chpule2}[
    ]{.chpuri2}メメント・クラスがオリジネーターの中にネストされているか[、]{.chpule2}[
    ]{.chpuri2}オリジネーターにその状態を変更させるに十分な setter
    の組がある場合に限ります[。]{.chpule2}[ ]{.chpuri2}
:::

:::::: {.section .pros-cons}
##  長所と短所 {#pros-cons}

::::: row
::: col-sm-6
-    カプセル化に違反することなく[、]{.chpule2}[
    ]{.chpuri2}オブジェクトの状態のスナップショットを生成可能[。]{.chpule2}[
    ]{.chpuri2}
-   
    ケアテーカーにオリジネーターの状態の履歴の保全を任せることで[、]{.chpule2}[
    ]{.chpuri2}オリジネーターのコードを簡素化可能[。]{.chpule2}[
    ]{.chpuri2}
:::

::: col-sm-6
-    クライアントによるメメント過剰作成により[、]{.chpule2}[
    ]{.chpuri2}アプリが大量の RAM を消費する可能性[。]{.chpule2}[
    ]{.chpuri2}
-    不要となったメメントを廃棄するためには[、]{.chpule2}[
    ]{.chpuri2}ケアテーカーがオリジネーターのライフサイクルを把握する必要あり[。]{.chpule2}[
    ]{.chpuri2}
-    PHP[、]{.chpule2}[ ]{.chpuri2}Python[、]{.chpule2}[
    ]{.chpuri2}JavaScript
    などのほとんどの動的プログラミング言語では[、]{.chpule2}[
    ]{.chpuri2}メメント内部状態の不変性の保証不可[。]{.chpule2}[
    ]{.chpuri2}
:::
:::::
::::::

::: {.section .relations}
##  他のパターンとの関係 {#relations}

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

-   現在の反復状態を獲得し[、]{.chpule2}[
    ]{.chpuri2}必要に応じてロールバックするために[、]{.chpule2}[
    ]{.chpuri2}[Memento](memento.html) を [Iterator](iterator.html)
    と一緒に使用できます[。]{.chpule2}[ ]{.chpuri2}

-   場合によっては[、]{.chpule2}[ ]{.chpuri2}[Prototype](prototype.html)
    を [Memento](memento.html)
    の代わりに使用した方が簡単な場合があります[。]{.chpule2}[
    ]{.chpuri2}状態の履歴を保存したいオブジェクトが比較的単純で[、]{.chpule2}[
    ]{.chpuri2}他の外部リソースへのリンクを持たないか簡単に再現できる場合に[、]{.chpule2}[
    ]{.chpuri2}この方法が使えます[。]{.chpule2}[ ]{.chpuri2}
:::

::: {.section .implementations}
##  コード例 {#implementations}

[![Memento を C#
で](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](memento/csharp/example.html "Memento を C# で"){.prog-lang-link}
[![Memento を C++
で](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](memento/cpp/example.html "Memento を C++ で"){.prog-lang-link}
[![Memento を Go
で](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](memento/go/example.html "Memento を Go で"){.prog-lang-link}
[![Memento を Java
で](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](memento/java/example.html "Memento を Java で"){.prog-lang-link}
[![Memento を PHP
で](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](memento/php/example.html "Memento を PHP で"){.prog-lang-link}
[![Memento を Python
で](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](memento/python/example.html "Memento を Python で"){.prog-lang-link}
[![Memento を Ruby
で](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](memento/ruby/example.html "Memento を Ruby で"){.prog-lang-link}
[![Memento を Rust
で](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](memento/rust/example.html "Memento を Rust で"){.prog-lang-link}
[![Memento を Swift
で](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](memento/swift/example.html "Memento を Swift で"){.prog-lang-link}
[![Memento を TypeScript
で](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](memento/typescript/example.html "Memento を TypeScript で"){.prog-lang-link}
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

[Observer []{.fa .fa-arrow-right}](observer.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### 戻る

[[]{.fa .fa-arrow-left} Mediator](mediator.html){.btn .btn-default
rel="prev"}
:::
::::::::::::::::::::::::::::::::::::

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
::::::::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::::::::
