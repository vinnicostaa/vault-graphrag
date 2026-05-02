::::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} /
[デザインパターン](../design-patterns.html){.type} /
[振る舞いに関するパターン](behavioral-patterns.html){.type}
:::

# Strategy {#strategy .title}

::: aka
別名：[ストラテジー、]{style="display:inline-block"}[戦略]{style="display:inline-block"}
:::

::: {.section .intent}
##  一言でいうと {#intent}

**Strategy**[ ]{.chpule1}[（]{.chpuri1}ストラテジー[、]{.chpule2}[
]{.chpuri2}戦略[）]{.chpule2}[ ]{.chpuri2}は[、]{.chpule2}[
]{.chpuri2}振る舞いに関するデザインパターンの一つで[、]{.chpule2}[
]{.chpuri2}アルゴリズムのファミリーを定義し[、]{.chpule2}[
]{.chpuri2}それぞれのアルゴリズムを別個のクラスとし[、]{.chpule2}[
]{.chpuri2}それらのオブジェクトを交換可能にします[。]{.chpule2}[
]{.chpuri2}

<figure class="image">
<img
src="../../images/patterns/content/strategy/strategy4ca6.png?id=379bfba335380500375881a3da6507e0"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/strategy/strategy.png?id=379bfba335380500375881a3da6507e0 640w,/images/patterns/content/strategy/strategy-1.5x.png?id=33ecebc66a9761454f2786a9b3e9bbe5 960w,/images/patterns/content/strategy/strategy-2x.png?id=1cee47d05a76fddf07dce9c67b700748 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Strategy デザインパターン" />
</figure>
:::

::: {.section .problem}
##  問題 {#problem}

ある日[、]{.chpule2}[
]{.chpuri2}あなたはカジュアルな旅行者のためのナビゲーション・アプリを作成することにしました[。]{.chpule2}[
]{.chpuri2}このアプリは[、]{.chpule2}[
]{.chpuri2}ユーザーがどの都市でも素早くその地に慣れるのに役立つきれいな地図が特徴でした[。]{.chpule2}[
]{.chpuri2}

アプリの最も要望のあった機能の一つは[、]{.chpule2}[
]{.chpuri2}自動経路計画でした[。]{.chpule2}[
]{.chpuri2}ユーザーが住所を入力すれば[、]{.chpule2}[
]{.chpuri2}地図上にその宛先への最速の経路を表示する[、]{.chpule2}[
]{.chpuri2}というものです[。]{.chpule2}[ ]{.chpuri2}

アプリの最初のバージョンでは[、]{.chpule2}[
]{.chpuri2}道路上の経路しか作成できませんでした[。]{.chpule2}[
]{.chpuri2}車で旅行する人々は喜んでいましたが[、]{.chpule2}[
]{.chpuri2}休暇中に皆が運転したいわけではありません[。]{.chpule2}[
]{.chpuri2}そこで[、]{.chpule2}[
]{.chpuri2}次のバージョンでは[、]{.chpule2}[
]{.chpuri2}徒歩の経路を作成するオプションを加えました[。]{.chpule2}[
]{.chpuri2}そしてそのすぐ後に[、]{.chpule2}[
]{.chpuri2}公共交通機関を使用した経路のオプションも追加しました[。]{.chpule2}[
]{.chpuri2}

しかし[、]{.chpule2}[
]{.chpuri2}それは始まりに過ぎませんでした[。]{.chpule2}[
]{.chpuri2}その後[、]{.chpule2}[
]{.chpuri2}サイクリングのための経路作成を計画しました[。]{.chpule2}[
]{.chpuri2}そしてさらに[、]{.chpule2}[
]{.chpuri2}市内のすべての観光スポットを経由して経路を作成するための別のオプションも加えることにしました[。]{.chpule2}[
]{.chpuri2}

<figure class="image">
<img
src="../../images/patterns/diagrams/strategy/problem9b55.png?id=e5ca943e559a979bd31d20e232dc22b6"
style="aspect-ratio: 2.2;"
srcset="/images/patterns/diagrams/strategy/problem.png?id=e5ca943e559a979bd31d20e232dc22b6 330w,/images/patterns/diagrams/strategy/problem-1.5x.png?id=31d1042ffc28056845e45d5c616da2a9 495w,/images/patterns/diagrams/strategy/problem-2x.png?id=3974fb99c97aec525dd0ffcff2e48e78 660w"
sizes="(max-width: 720px) 100vw, 330px" loading="lazy" width="330"
alt="ナビ・アプリのコードは大膨張" />
<figcaption><p>ナビ・アプリのコードは大膨張<span
class="chpule2">。</span><span class="chpuri2"> </span></p></figcaption>
</figure>

ビジネスの観点からは[、]{.chpule2}[
]{.chpuri2}アプリは成功でしたが[、]{.chpule2}[
]{.chpuri2}技術的には[、]{.chpule2}[
]{.chpuri2}頭痛のタネとなりました[。]{.chpule2}[
]{.chpuri2}新しい経路アルゴリズムを追加するたびに[、]{.chpule2}[
]{.chpuri2}ナビゲーターの主要クラスのサイズは倍増しました[。]{.chpule2}[
]{.chpuri2}ある時点で[、]{.chpule2}[
]{.chpuri2}この野獣を飼うのに手を焼くようになりました[。]{.chpule2}[
]{.chpuri2}

バグ修正のためであれ[、]{.chpule2}[
]{.chpuri2}街路スコアの微細な変更であれ[、]{.chpule2}[
]{.chpuri2}アルゴリズムの一つを変更すると[、]{.chpule2}[
]{.chpuri2}クラス全体にその影響が及び[、]{.chpule2}[
]{.chpuri2}問題なく動いていたコードにエラーを引き起こす可能性が増大しました[。]{.chpule2}[
]{.chpuri2}

さらに[、]{.chpule2}[
]{.chpuri2}チームワークは非効率的になりました[。]{.chpule2}[
]{.chpuri2}成功したリリースの直後に採用されたチームメートは[、]{.chpule2}[
]{.chpuri2}マージの競合を解決するのに時間を使い過ぎだと苦情を言います[。]{.chpule2}[
]{.chpuri2}新しい機能を実装するには[、]{.chpule2}[
]{.chpuri2}他の人が生成したコードと競合するため[、]{.chpule2}[
]{.chpuri2}巨大なクラスを変更する必要が出てきます[。]{.chpule2}[
]{.chpuri2}
:::

::: {.section .solution}
##  解決策 {#solution}

Strategy パターンに従うと[、]{.chpule2}[
]{.chpuri2}あることを異なる様々な方法で行うクラスに対し[、]{.chpule2}[
]{.chpuri2}これらのアルゴリズムの一つずつを*[ス]{.cjk}[ト]{.cjk}[ラ]{.cjk}[テ]{.cjk}[ジ]{.cjk}[ー]{.cjk}*[
]{.chpule1}[（]{.chpuri1}戦略[）]{.chpule2}[
]{.chpuri2}と呼ばれる別々のクラスに抽出します[。]{.chpule2}[ ]{.chpuri2}

*[コ]{.cjk}[ン]{.cjk}[テ]{.cjk}[キ]{.cjk}[ス]{.cjk}[ト]{.cjk}*と呼ばれる元のクラスは[、]{.chpule2}[
]{.chpuri2}数ある戦略のいずれか一つへの参照を格納するためのフィールドを持たなければなりません[。]{.chpule2}[
]{.chpuri2}コンテキストは[、]{.chpule2}[
]{.chpuri2}自分で作業を行う代わりに[、]{.chpule2}[
]{.chpuri2}リンクされたストラテジー・オブジェクトに作業を委任します[。]{.chpule2}[
]{.chpuri2}

コンテキストは[、]{.chpule2}[
]{.chpuri2}その仕事に適切なアルゴリズムを選択する責任を負いません[。]{.chpule2}[
]{.chpuri2}代わりに[、]{.chpule2}[
]{.chpuri2}クライアントは希望するストラテジーをコンテキストに渡します[。]{.chpule2}[
]{.chpuri2}実際のところ[、]{.chpule2}[
]{.chpuri2}コンテキストはストラテジーについての知識を持ちません[。]{.chpule2}[
]{.chpuri2}コンテキストは[、]{.chpule2}[
]{.chpuri2}すべてのストラテジーと[、]{.chpule2}[
]{.chpuri2}同じ汎用インターフェースを通してやりとりします[。]{.chpule2}[
]{.chpuri2}インターフェースは[、]{.chpule2}[
]{.chpuri2}選ばれたストラテジーに内包されたアルゴリズムを使用するメソッド一つだけでできています[。]{.chpule2}[
]{.chpuri2}

こうすると[、]{.chpule2}[
]{.chpuri2}コンテキストは具体的なストラテジーからは独立したものとなり[、]{.chpule2}[
]{.chpuri2}コンテキストや他のストラテジーのコードを変更することなく[、]{.chpule2}[
]{.chpuri2}新しいアルゴリズムを追加したり[、]{.chpule2}[
]{.chpuri2}既存のものを変更できるようになります[。]{.chpule2}[
]{.chpuri2}

<figure class="image">
<img
src="../../images/patterns/diagrams/strategy/solution1435.png?id=0813a174b29a2ed5902d321aba28e5fc"
style="aspect-ratio: 2.04;"
srcset="/images/patterns/diagrams/strategy/solution.png?id=0813a174b29a2ed5902d321aba28e5fc 570w,/images/patterns/diagrams/strategy/solution-1.5x.png?id=ce3d4e57f4a2a06ebc96f2607b3d6691 855w,/images/patterns/diagrams/strategy/solution-2x.png?id=66b5ee048ea2ad25c4b20f180ebf94d7 1140w"
sizes="(max-width: 720px) 100vw, 570px" loading="lazy" width="570"
alt="経路計画戦略 " />
<figcaption><p>経路計画アプリ</p></figcaption>
</figure>

我々のナビゲーション・アプリでは[、]{.chpule2}[
]{.chpuri2}それぞれのアルゴリズムを抽出して `build­Route`
メソッド一つだけを持つ固有のクラスを作ります[。]{.chpule2}[
]{.chpuri2}このメソッドは[、]{.chpule2}[
]{.chpuri2}出発地と目的地を受け取り[、]{.chpule2}[
]{.chpuri2}経路上の主要箇所のコレクションを返します[。]{.chpule2}[
]{.chpuri2}

同じ引数を与えても[、]{.chpule2}[
]{.chpuri2}各経路計画クラスは違う経路を作り上げるかもしれません[。]{.chpule2}[
]{.chpuri2}ナビの主クラスは[、]{.chpule2}[
]{.chpuri2}どのアルゴリズムが選択されたかについては[、]{.chpule2}[
]{.chpuri2}まったく感知しません[。]{.chpule2}[
]{.chpuri2}なぜなら[、]{.chpule2}[
]{.chpuri2}経由地を地図上に描画することがその一番の仕事だからです[。]{.chpule2}[
]{.chpuri2}クライアントが UI
上のボタンなどにより経路選択の振る舞いを他のものと入れ替えれらるよう[、]{.chpule2}[
]{.chpuri2}現在有効なストラテジーを切り替えるためのメソッドがクラスにはあります[。]{.chpule2}[
]{.chpuri2}
:::

::: {.section .analogy}
##  現実世界でのたとえ {#analogy}

<figure class="image">
<img
src="../../images/patterns/content/strategy/strategy-comic-1-ja9216.png?id=3d11af77b687bf940faabb8f8c8f4e43"
style="aspect-ratio: 2.13;"
srcset="/images/patterns/content/strategy/strategy-comic-1-ja.png?id=3d11af77b687bf940faabb8f8c8f4e43 640w,/images/patterns/content/strategy/strategy-comic-1-ja-1.5x.png?id=51dd9726ab890df9b3443c3928c3bf43 960w,/images/patterns/content/strategy/strategy-comic-1-ja-2x.png?id=ec599b0c7ca280a3d910b148bb8f0680 1280w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="640"
alt="各種輸送戦略" />
<figcaption><p>空港に到達するための様々な戦略<span
class="chpule2">。</span><span class="chpuri2"> </span></p></figcaption>
</figure>

空港に行くことを想像してみてください[。]{.chpule2}[
]{.chpuri2}バスに乗ることもできまし[、]{.chpule2}[
]{.chpuri2}タクシーを頼むこともできますし[、]{.chpule2}[
]{.chpuri2}自転車でも行けます[。]{.chpule2}[
]{.chpuri2}これらが[、]{.chpule2}[
]{.chpuri2}輸送上の戦略です[。]{.chpule2}[
]{.chpuri2}予算や時間の制限などの要因を勘案して[、]{.chpule2}[
]{.chpuri2}どれを選ぶかを決めます[。]{.chpule2}[ ]{.chpuri2}
:::

::::: {.section .structure-container}
##  構造 {#structure}

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/strategy/structuree491.png?id=c6aa910c94960f35d100bfca02810ea1"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.19;"
srcset="/images/patterns/diagrams/strategy/structure.png?id=c6aa910c94960f35d100bfca02810ea1 440w,/images/patterns/diagrams/strategy/structure-1.5x.png?id=5b1c6376b06374719dcae95455b068d8 660w,/images/patterns/diagrams/strategy/structure-2x.png?id=5bd791857c3bab419bcf4fa86877439d 880w"
sizes="(max-width: 720px) 100vw, 440px" loading="lazy" width="440"
alt="Strategy デザインパターンの構造" /><img
src="../../images/patterns/diagrams/strategy/structure-indexed64c7.png?id=ff55c5a6273cf82a5667f3cab5b605c7"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.15;"
srcset="/images/patterns/diagrams/strategy/structure-indexed.png?id=ff55c5a6273cf82a5667f3cab5b605c7 450w,/images/patterns/diagrams/strategy/structure-indexed-1.5x.png?id=3d6ba05b8a74a9fb8e3f88eb9ca1e30f 675w,/images/patterns/diagrams/strategy/structure-indexed-2x.png?id=9f8e2f838f16705775411e2b4616820e 900w"
sizes="(max-width: 720px) 100vw, 450px" loading="lazy" width="450"
alt="Strategy デザインパターンの構造" />
</figure>
:::

1.  **コンテキスト**[ ]{.chpule1}[（]{.chpuri1}Context[）]{.chpule2}[
    ]{.chpuri2}は[、]{.chpule2}[
    ]{.chpuri2}複数の具象ストラテジーのいずれか一つへの参照を維持し[、]{.chpule2}[
    ]{.chpuri2}ストラテジー・インターフェースのみを介してこのオブジェクトと通信します[。]{.chpule2}[
    ]{.chpuri2}

2.  **ストラテジー**[ ]{.chpule1}[（]{.chpuri1}Strategy[）]{.chpule2}[
    ]{.chpuri2}インターフェースは[、]{.chpule2}[
    ]{.chpuri2}すべての具象ストラテジーに共通です[。]{.chpule2}[
    ]{.chpuri2}コンテキストがストラテジーを実行するために使用するメソッドを宣言します[。]{.chpule2}[
    ]{.chpuri2}

3.  **具象ストラテジー**[ ]{.chpule1}[（]{.chpuri1}Concrete
    Strategy[）]{.chpule2}[ ]{.chpuri2}は[、]{.chpule2}[
    ]{.chpuri2}コンテキストが使用するアルゴリズムを種々の違う方法で実装します[。]{.chpule2}[
    ]{.chpuri2}

4.  アルゴリズムの実行が必要となるたびに[、]{.chpule2}[
    ]{.chpuri2}コンテキストはリンク先ストラテジー・オブジェクトの実行メソッドを呼びます[。]{.chpule2}[
    ]{.chpuri2}コンテキストは[、]{.chpule2}[
    ]{.chpuri2}どのような種類のストラテジーを相手に動作するのか[、]{.chpule2}[
    ]{.chpuri2}どのようにアルゴリズムが実行されるかは認知しません[。]{.chpule2}[
    ]{.chpuri2}

5.  **クライアント**[ ]{.chpule1}[（]{.chpuri1}Client[）]{.chpule2}[
    ]{.chpuri2}は[、]{.chpule2}[
    ]{.chpuri2}特定のストラテジーのオブジェクトを作成し[、]{.chpule2}[
    ]{.chpuri2}コンテキストに渡します[。]{.chpule2}[
    ]{.chpuri2}コンテキストは[、]{.chpule2}[ ]{.chpuri2}setter
    を公開しており[、]{.chpule2}[
    ]{.chpuri2}クライアントは実行時にそれを使ってストラテジーを置き換えます[。]{.chpule2}[
    ]{.chpuri2}
::::
:::::

::: {.section .pseudocode}
##  擬似コード {#pseudocode}

この例では[、]{.chpule2}[ ]{.chpuri2}コンテキストは[、]{.chpule2}[
]{.chpuri2}様々な算術演算を実行するために複数の**ストラテジー**を使用します[。]{.chpule2}[
]{.chpuri2}

<figure class="code">
<pre class="code" lang="pseudocode"><code>// ストラテジー・インターフェースは、あるアルゴリズムのサポートされている
// すべてのバージョンのアルゴリズムに共通する操作を宣言。コンテキストは、
// 具象ストラテジーで定義されたアルゴリズムを呼ぶのにこのインターフェース
// を使用。
interface Strategy is
    method execute(a, b)

// 具象ストラテジーは、基底ストラテジー・インターフェースに従いながらアル
// ゴリズムを実装する。インターフェースのために、具象ストラテジーはコンテ
// キスト中で交換可能となる。
class ConcreteStrategyAdd implements Strategy is
    method execute(a, b) is
        return a + b

class ConcreteStrategySubtract implements Strategy is
    method execute(a, b) is
        return a - b

class ConcreteStrategyMultiply implements Strategy is
    method execute(a, b) is
        return a * b

// コンテキストは、異なるストラテジーを使って、少し異なるビジネス・ロジッ
// クを実装する。
class Context is
    // コンテキストは、ストラテジー・オブジェクトの一つへの参照を維持する。
    // コンテキストは、ストラテジーの具象クラスを知らない。ストラテジー・
    // インターフェースを通して、全てのストラテジーと共に動作可能なはず。
    private strategy: Strategy

    // 通常コンテキストはコンストラクターを通してストラテジーを受け付ける。
    // そして実行時に切り替えができるよう、setter も提供する。
    method setStrategy(Strategy strategy) is
        this.strategy = strategy

    // コンテキストは、複数個のバージョンのアルゴリズムを自前で持つ代わり
    // に、作業の一部をストラテジー・オブジェクトに委任する。
    method executeStrategy(int a, int b) is
        return strategy.execute(a, b)


// クライアント・コードは具象ストラテジーを一つ選び、コンテキストに渡す。
// クライアントが、正しい選択をするためには、ストラテジー間の違いを知って
// おく必要あり。
class ExampleApplication is
    method main() is
        Create context object.

        Read first number.
        Read last number.
        Read the desired action from user input.

        if (action == addition) then
            context.setStrategy(new ConcreteStrategyAdd())

        if (action == subtraction) then
            context.setStrategy(new ConcreteStrategySubtract())

        if (action == multiplication) then
            context.setStrategy(new ConcreteStrategyMultiply())

        result = context.executeStrategy(First number, Second number)

        Print result.</code></pre>
</figure>
:::

:::::::::::: {.section .applicability-container}
##  適応性 {#applicability}

::::::::::: applicability
::: applicability-problem
オブジェクト内でアルゴリズムのいろいろ異なる変種の使用を望み[、]{.chpule2}[
]{.chpuri2}あるアルゴリズムから別のアルゴリズムに実行時に切り替えられるようにするには[、]{.chpule2}[
]{.chpuri2}Strategy パターンを使用します[。]{.chpule2}[ ]{.chpuri2}
:::

::: applicability-solution
Strategy パターンでは[、]{.chpule2}[
]{.chpuri2}オブジェクトの振る舞いを実行時に間接的に切り替えることが可能です[。]{.chpule2}[
]{.chpuri2}それは[、]{.chpule2}[
]{.chpuri2}仕事の一部を異なる方法で実行するオブジェクトと結びつけることによって行います[。]{.chpule2}[
]{.chpuri2}
:::

::: applicability-problem
一部の振る舞いだけが異なり[、]{.chpule2}[
]{.chpuri2}あとは同じようなクラスがたくさんある場合[、]{.chpule2}[
]{.chpuri2}Strategy を使用します[。]{.chpule2}[ ]{.chpuri2}
:::

::: applicability-solution
Strategy パターンを使用すると[、]{.chpule2}[
]{.chpuri2}様々な振る舞いを個別のクラス階層に抽出し[、]{.chpule2}[
]{.chpuri2}元のクラスを一つにまとめ[、]{.chpule2}[
]{.chpuri2}重複したコードを削減できます[。]{.chpule2}[ ]{.chpuri2}
:::

::: applicability-problem
クラスのビジネス・ロジックを[、]{.chpule2}[
]{.chpuri2}ロジック全体からするとあまり重要ではないアルゴリズムの実装の詳細から分離します[。]{.chpule2}[
]{.chpuri2}
:::

::: applicability-solution
Strategy パターンを使用すると[、]{.chpule2}[
]{.chpuri2}コード[、]{.chpule2}[ ]{.chpuri2}内部データ[、]{.chpule2}[
]{.chpuri2}様々なアルゴリズムの依存関係をコードの残りの部分から分離できます[。]{.chpule2}[
]{.chpuri2}クライアントはアルゴリズムを実行し[、]{.chpule2}[
]{.chpuri2}実行時にそれらを切り替える単純なインターフェースが使えます[。]{.chpule2}[
]{.chpuri2}
:::

::: applicability-problem
クラスに[、]{.chpule2}[
]{.chpuri2}同じアルゴリズムの変種を切り替えるための多数の条件文の塊が見受けられる場合[、]{.chpule2}[
]{.chpuri2}このパターンを使用します[。]{.chpule2}[ ]{.chpuri2}
:::

::: applicability-solution
Strategy パターンを適用することにより[、]{.chpule2}[
]{.chpuri2}アルゴリズムを[、]{.chpule2}[
]{.chpuri2}同一インターフェースを実装する個別のクラスに抽出し[、]{.chpule2}[
]{.chpuri2}条件文を排除できます[。]{.chpule2}[
]{.chpuri2}元のオブジェクトは[、]{.chpule2}[
]{.chpuri2}アルゴリズムの多くの変種をその中で実装するのに代わり[、]{.chpule2}[
]{.chpuri2}これらのオブジェクトの一つに実行を委任します[。]{.chpule2}[
]{.chpuri2}
:::
:::::::::::
::::::::::::

::: {.section .checklist}
##  実装方法 {#checklist}

1.  コンテキスト・クラス内で[、]{.chpule2}[
    ]{.chpuri2}頻繁に変更されそうなアルゴリズムを見つけてください[。]{.chpule2}[
    ]{.chpuri2}同じアルゴリズムの変種の切り替えを実行するための条件分の塊がその目印になるかもしれません[。]{.chpule2}[
    ]{.chpuri2}

2.  アルゴリズムの変種間で共通となるストラテジー・インターフェースを宣言します[。]{.chpule2}[
    ]{.chpuri2}

3.  アルゴリズムの変種全部を一つずつ個別のクラスに抽出します[。]{.chpule2}[
    ]{.chpuri2}クラスはストラテジー・インターフェースを実装するようにします[。]{.chpule2}[
    ]{.chpuri2}

4.  コンテキスト・クラス中に[、]{.chpule2}[
    ]{.chpuri2}ストラテジー・オブジェクトへの参照を格納するためのフィールドを追加します[。]{.chpule2}[
    ]{.chpuri2}そのフィールドの値を置き換えるための setter
    セッターを書き加えてください[。]{.chpule2}[
    ]{.chpuri2}コンテキストはストラテジー・インターフェースを介してのみストラテジー・オブジェクトを使用する必要があります[。]{.chpule2}[
    ]{.chpuri2}コンテキストは[、]{.chpule2}[
    ]{.chpuri2}ストラテジーがデータにアクセスできるようにするためのインターフェースを定義してもかまいません[。]{.chpule2}[
    ]{.chpuri2}

5.  コンテキストのクライアントは[、]{.chpule2}[
    ]{.chpuri2}コンテキストの主な任務を実行するのに適切と思われるストラテジーとコンテキストを関連付ける必要があります[。]{.chpule2}[
    ]{.chpuri2}
:::

:::::: {.section .pros-cons}
##  長所と短所 {#pros-cons}

::::: row
::: col-sm-6
-   
    実行時にオブジェクト内で使用されるアルゴリズムを交換可能[。]{.chpule2}[
    ]{.chpuri2}
-    アルゴリズムの実装の詳細を[、]{.chpule2}[
    ]{.chpuri2}それを使用するコードから分離可能[。]{.chpule2}[
    ]{.chpuri2}
-    継承を合成で置き換え可能[。]{.chpule2}[ ]{.chpuri2}
-   
    *[開]{.cjk}[放]{.cjk}[閉]{.cjk}[鎖]{.cjk}[の]{.cjk}[原]{.cjk}[則]{.cjk}*[。]{.chpule2}[
    ]{.chpuri2}コンテキストの変更の必要なしに新規ストラテジーを導入可能[。]{.chpule2}[
    ]{.chpuri2}
:::

::: col-sm-6
-    ごく少数のアルゴリズムしかなく[、]{.chpule2}[
    ]{.chpuri2}それらがほとんど変更されなければ[、]{.chpule2}[
    ]{.chpuri2}このパターン導入で必要となる新規クラスやインターフェースでプログラムを必要以上に複雑化させる理由はない[。]{.chpule2}[
    ]{.chpuri2}
-    クライアントは[、]{.chpule2}[
    ]{.chpuri2}適切なストラテジーを選択するにあたり[、]{.chpule2}[
    ]{.chpuri2}ストラテジー間の違いを認識する必要あり[。]{.chpule2}[
    ]{.chpuri2}
-   
    多くの近代的なプログラミング言語は関数型をサポートしており[、]{.chpule2}[
    ]{.chpuri2}アルゴリズムの異種を匿名関数の集合中で実装可能[。]{.chpule2}[
    ]{.chpuri2}余計なクラスやインターフェースでプログラムを膨らませることなく[、]{.chpule2}[
    ]{.chpuri2}ストラテジー・オブジェクトとまったく同じようにこれらの関数が使用可能[。]{.chpule2}[
    ]{.chpuri2}
:::
:::::
::::::

::: {.section .relations}
##  他のパターンとの関係 {#relations}

-   [Bridge](bridge.html)[、]{.chpule2}[ ]{.chpuri2}
    [State](state.html)[、]{.chpule2}[
    ]{.chpuri2}[Strategy](strategy.html)[
    ]{.chpule1}[（]{.chpuri1}と限られた意味合いでは[、]{.chpule2}[
    ]{.chpuri2}[Adapter](adapter.html) も[）]{.chpule2}[
    ]{.chpuri2}は[、]{.chpule2}[
    ]{.chpuri2}非常に似た構造をしています[。]{.chpule2}[
    ]{.chpuri2}実際のところ[、]{.chpule2}[
    ]{.chpuri2}これらの全てのパターンは[、]{.chpule2}[
    ]{.chpuri2}合成に基づいており[、]{.chpule2}[
    ]{.chpuri2}仕事を他のオブジェクトに委任します[。]{.chpule2}[
    ]{.chpuri2}しかしながら[、]{.chpule2}[
    ]{.chpuri2}違う問題を解決します[。]{.chpule2}[
    ]{.chpuri2}パターンは[、]{.chpule2}[
    ]{.chpuri2}単にコードを特定の方法で構造化するためのレシピではありません[。]{.chpule2}[
    ]{.chpuri2}パターンが解決する問題に関して[、]{.chpule2}[
    ]{.chpuri2}開発者同士がするコミュニケーションの道具でもあります[。]{.chpule2}[
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

-   [Decorator](decorator.html)
    がオブジェクトの表層を変えるのに対し[、]{.chpule2}[
    ]{.chpuri2}[Strategy](strategy.html) は中身を変えます[。]{.chpule2}[
    ]{.chpuri2}

-   [Template Method](template-method.html) は[、]{.chpule2}[
    ]{.chpuri2}継承に基づくもので[、]{.chpule2}[
    ]{.chpuri2}サブクラスでその一部を拡張することによって[、]{.chpule2}[
    ]{.chpuri2}アルゴリズムを部分的に変更できます[。]{.chpule2}[
    ]{.chpuri2}[Strategy](strategy.html) は[、]{.chpule2}[
    ]{.chpuri2}合成に基き[、]{.chpule2}[
    ]{.chpuri2}オブジェクトの振る舞いを[、]{.chpule2}[
    ]{.chpuri2}新しい振る舞いに関連した異なるストラテジーを与えることにより変更します[。]{.chpule2}[
    ]{.chpuri2}*Template Method* は[、]{.chpule2}[
    ]{.chpuri2}クラスのレベルで機能するので[、]{.chpule2}[
    ]{.chpuri2}静的です[。]{.chpule2}[ ]{.chpuri2}*Strategy*
    はオブジェクトのレベルで機能するため[、]{.chpule2}[
    ]{.chpuri2}実行時に動作を切り替えることができます[。]{.chpule2}[
    ]{.chpuri2}

-   [State](state.html) は[、]{.chpule2}[
    ]{.chpuri2}[Strategy](strategy.html)
    の拡張と考えることができます[。]{.chpule2}[
    ]{.chpuri2}どちらのパターンも合成[
    ]{.chpule1}[（]{.chpuri1}コンポジション[）]{.chpule2}[
    ]{.chpuri2}に基づいており[、]{.chpule2}[
    ]{.chpuri2}コンテキストの振る舞いの変更を[、]{.chpule2}[
    ]{.chpuri2}ヘルパー・オブジェクトに仕事の一部を委任することにより行います[。]{.chpule2}[
    ]{.chpuri2}*Strategy* では[、]{.chpule2}[
    ]{.chpuri2}これらのオブジェクトは完全に独立しており[、]{.chpule2}[
    ]{.chpuri2}互いを意識しません[。]{.chpule2}[
    ]{.chpuri2}しかし[、]{.chpule2}[ ]{.chpuri2}*State*
    では[、]{.chpule2}[
    ]{.chpuri2}具象状態間の依存関係を制限せず[、]{.chpule2}[
    ]{.chpuri2}コンテキストの状態を自由に変更できます[。]{.chpule2}[
    ]{.chpuri2}
:::

::: {.section .implementations}
##  コード例 {#implementations}

[![Strategy を C#
で](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](strategy/csharp/example.html "Strategy を C# で"){.prog-lang-link}
[![Strategy を C++
で](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](strategy/cpp/example.html "Strategy を C++ で"){.prog-lang-link}
[![Strategy を Go
で](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](strategy/go/example.html "Strategy を Go で"){.prog-lang-link}
[![Strategy を Java
で](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](strategy/java/example.html "Strategy を Java で"){.prog-lang-link}
[![Strategy を PHP
で](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](strategy/php/example.html "Strategy を PHP で"){.prog-lang-link}
[![Strategy を Python
で](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](strategy/python/example.html "Strategy を Python で"){.prog-lang-link}
[![Strategy を Ruby
で](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](strategy/ruby/example.html "Strategy を Ruby で"){.prog-lang-link}
[![Strategy を Rust
で](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](strategy/rust/example.html "Strategy を Rust で"){.prog-lang-link}
[![Strategy を Swift
で](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](strategy/swift/example.html "Strategy を Swift で"){.prog-lang-link}
[![Strategy を TypeScript
で](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](strategy/typescript/example.html "Strategy を TypeScript で"){.prog-lang-link}
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

[Template Method []{.fa .fa-arrow-right}](template-method.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### 戻る

[[]{.fa .fa-arrow-left} State](state.html){.btn .btn-default rel="prev"}
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
