::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} /
[デザインパターン](../design-patterns.html){.type} /
[振る舞いに関するパターン](behavioral-patterns.html){.type}
:::

# Template Method {#template-method .title}

::: aka
別名：[テンプレート・メソッド]{style="display:inline-block"}
:::

::: {.section .intent}
##  一言でいうと {#intent}

**Template Method**[
]{.chpule1}[（]{.chpuri1}テンプレート・メソッド[、]{.chpule2}[
]{.chpuri2}鋳型法[）]{.chpule2}[ ]{.chpuri2}は[、]{.chpule2}[
]{.chpuri2}振る舞いに関するデザインパターンの一つで[、]{.chpule2}[
]{.chpuri2}スーパークラス内でアルゴリズムの骨格を定義しておき[、]{.chpule2}[
]{.chpuri2}サブクラスは構造を変えることなくアルゴリズムの特定のステップを上書きします[。]{.chpule2}[
]{.chpuri2}

<figure class="image">
<img
src="../../images/patterns/content/template-method/template-method4c1b.png?id=eee9461742f832814f19612ccf472819"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/template-method/template-method.png?id=eee9461742f832814f19612ccf472819 640w,/images/patterns/content/template-method/template-method-1.5x.png?id=2b4c179aaa02f5c45a59324b904cd130 960w,/images/patterns/content/template-method/template-method-2x.png?id=4e164dc41be4dcfa122628864c2be210 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Template Method デザインパターン" />
</figure>
:::

::: {.section .problem}
##  問題 {#problem}

企業文書を分析するデータマイニングのアプリケーションを作成しているところを想像してみてください[。]{.chpule2}[
]{.chpuri2}ユーザーは[、]{.chpule2}[ ]{.chpuri2}様々な形式[
]{.chpule1}[（]{.chpuri1}PDF[、]{.chpule2}[
]{.chpuri2}DOC[、]{.chpule2}[ ]{.chpuri2}CSV[）]{.chpule2}[
]{.chpuri2}の文書をアプリに送り付け[、]{.chpule2}[
]{.chpuri2}アプリはこれらの文書から何か意味のあるデータを統一的な形式で抽出しようとします[。]{.chpule2}[
]{.chpuri2}

アプリの最初のバージョンは DOC ファイルでのみ動作します[。]{.chpule2}[
]{.chpuri2}次のバージョンでは[、]{.chpule2}[ ]{.chpuri2}CSV
ファイルをサポートすることができました[。]{.chpule2}[ ]{.chpuri2}1
か月後には[、]{.chpule2}[ ]{.chpuri2}PDF
ファイルからデータを抽出する方法をそれに[
]{.chpule1}[「]{.chpuri1}教えました[」]{.ls-05}[。]{.chpule2}[
]{.chpuri2}

<figure class="image">
<img
src="../../images/patterns/diagrams/template-method/problem6793.png?id=60fa4f735c467ac1c9438231a1782807"
style="aspect-ratio: 1.35;"
srcset="/images/patterns/diagrams/template-method/problem.png?id=60fa4f735c467ac1c9438231a1782807 620w,/images/patterns/diagrams/template-method/problem-1.5x.png?id=83fa009f7727bdcc69335b946919f0d8 930w,/images/patterns/diagrams/template-method/problem-2x.png?id=fc8b434afec7b6135043d0d2f48189f0 1240w"
sizes="(max-width: 720px) 100vw, 620px" loading="lazy" width="620"
alt="データマイニングのクラス間の多数の重複したコード" />
<figcaption><p>データ・マイニングのクラス間にある多くの重複したコード<span
class="chpule2">。</span><span class="chpuri2"> </span></p></figcaption>
</figure>

ある時点で[、]{.chpule2}[
]{.chpuri2}三つのクラスすべてに似たようなコードがあることに気がつきました[。]{.chpule2}[
]{.chpuri2}様々なデータ形式を扱うためのコードは[、]{.chpule2}[
]{.chpuri2}すべてのクラスでまったく異なりますが[、]{.chpule2}[
]{.chpuri2}データ処理と解析のためのコードはほぼ同じです[。]{.chpule2}[
]{.chpuri2}アルゴリズムの構造をそのままにして[、]{.chpule2}[
]{.chpuri2}コードの重複を取り除けたらどんなに素晴らしいでしょう[？]{.chpule2}[
]{.chpuri2}

これらのクラスを使用するクライアント側のコードにも一つ問題があります[。]{.chpule2}[
]{.chpuri2}処理オブジェクトのクラスに応じて適切な処理を選ぶための条件文だらけになっている[、]{.chpule2}[
]{.chpuri2}ということです[。]{.chpule2}[
]{.chpuri2}もしも処理クラスの全部が共通のインターフェースか基底クラスを持てれば[、]{.chpule2}[
]{.chpuri2}クライアント・コードの条件分を削除でき[、]{.chpule2}[
]{.chpuri2}処理オブジェクトのメソッドを呼ぶ際に多相性を利用できます[。]{.chpule2}[
]{.chpuri2}
:::

::: {.section .solution}
##  解決策 {#solution}

Template Method パターンでは[、]{.chpule2}[
]{.chpuri2}アルゴリズムを一連のステップに分解し[、]{.chpule2}[
]{.chpuri2}これらのステップをメソッドに書き換え[、]{.chpule2}[
]{.chpuri2}単一の*[テ]{.cjk}[ン]{.cjk}[プ]{.cjk}[レ]{.cjk}[ー]{.cjk}[ト]{.cjk}[・]{.cjk}[メ]{.cjk}[ソ]{.cjk}[ッ]{.cjk}[ド]{.cjk}*からこれらのメソッドを順番に呼び出すようにします[。]{.chpule2}[
]{.chpuri2}*[ス]{.cjk}[テ]{.cjk}[ッ]{.cjk}[プ]{.cjk}*は[、]{.chpule2}[
]{.chpuri2}`abstract` としてもいいですし[、]{.chpule2}[
]{.chpuri2}何らかのデフォルトの実装を用意してもかまいません[。]{.chpule2}[
]{.chpuri2}アルゴリズムを利用するためには[、]{.chpule2}[
]{.chpuri2}クライアントは独自のサブクラスを用意し[、]{.chpule2}[
]{.chpuri2}全抽象メソッドを実装し[、]{.chpule2}[
]{.chpuri2}必要ならば他のメソッド[
]{.chpule1}[（]{.chpuri1}ただしテンプレート・メソッドは除く[）]{.chpule2}[
]{.chpuri2}も上書きする決まりになっています[。]{.chpule2}[ ]{.chpuri2}

データマイニング・アプリでこれがどうなるか見てみましょう[。]{.chpule2}[
]{.chpuri2}三つの解析アルゴリズムすべてに対する基底クラスを作成できます[。]{.chpule2}[
]{.chpuri2}このクラスは[、]{.chpule2}[
]{.chpuri2}様々な文書処理ステップへの一連の呼び出しからなるテンプレート・メソッドを定義します[。]{.chpule2}[
]{.chpuri2}

<figure class="image">
<img
src="../../images/patterns/diagrams/template-method/solution-ja2acb.png?id=7a9848f9797e4a9a0d020eb547397e6a"
style="aspect-ratio: 1.43;"
srcset="/images/patterns/diagrams/template-method/solution-ja.png?id=7a9848f9797e4a9a0d020eb547397e6a 600w,/images/patterns/diagrams/template-method/solution-ja-1.5x.png?id=f10180da6c092368ec0066c6e0dbf792 900w,/images/patterns/diagrams/template-method/solution-ja-2x.png?id=48bed6a870a883599fa9bf135697f290 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="テンプレート・メソッドはアルゴリズムの骨組みを定義する" />
<figcaption><p>Template Method は<span class="chpule2">、</span><span
class="chpuri2"> </span>アルゴリズムをステップに分割し<span
class="chpule2">、</span><span class="chpuri2">
</span>サブクラスがこれらのステップを上書きするが<span
class="chpule2">、</span><span class="chpuri2">
</span>本物のメソッドではない<span class="chpule2">。</span><span
class="chpuri2"> </span></p></figcaption>
</figure>

最初は[、]{.chpule2}[ ]{.chpuri2}すべてのステップを `abstract`
と宣言し[、]{.chpule2}[
]{.chpuri2}サブクラスにこれらのメソッドの独自の実装を強いるようにできます[。]{.chpule2}[
]{.chpuri2}我々の例題では[、]{.chpule2}[
]{.chpuri2}サブクラスに必要な実装がすでにすべてあるので[、]{.chpule2}[
]{.chpuri2}残されたことは[、]{.chpule2}[
]{.chpuri2}必要に応じてスーパークラスのメソッドに合うようにメソッドのシグネチャーを調整することぐらいです[。]{.chpule2}[
]{.chpuri2}

ここで[、]{.chpule2}[
]{.chpuri2}重複したコードを取り除くために何ができるか考えてみましょう[。]{.chpule2}[
]{.chpuri2}ファイルを開けたり閉じたりするためのコードや[、]{.chpule2}[
]{.chpuri2}データの抽出・解析に関わるコードは[、]{.chpule2}[
]{.chpuri2}データ形式により異なるため[、]{.chpule2}[
]{.chpuri2}この辺のコードはいじる必要ないみたいですね[。]{.chpule2}[
]{.chpuri2}しかし[、]{.chpule2}[
]{.chpuri2}生データの分析やレポートの作成などの他のステップの実装はサブクラス間で非常に類似しているので[、]{.chpule2}[
]{.chpuri2}基底クラスに移動すれば[、]{.chpule2}[
]{.chpuri2}サブクラスから共有できそうです[。]{.chpule2}[ ]{.chpuri2}

ステップには二つの種類があります[：]{.chpule2}[ ]{.chpuri2}

-   *[抽]{.cjk}[象]{.cjk}[ス]{.cjk}[テ]{.cjk}[ッ]{.cjk}[プ]{.cjk}*は[、]{.chpule2}[
    ]{.chpuri2}全サブクラスで実装が必要[。]{.chpule2}[ ]{.chpuri2}
-   *[オ]{.cjk}[プ]{.cjk}[シ]{.cjk}[ョ]{.cjk}[ン]{.cjk}[の]{.cjk}[ス]{.cjk}[テ]{.cjk}[ッ]{.cjk}[プ]{.cjk}*は[、]{.chpule2}[
    ]{.chpuri2}何らかのデフォルト実装があり[、]{.chpule2}[
    ]{.chpuri2}必要な場合だけ上書き可能[。]{.chpule2}[ ]{.chpuri2}

実はもう一つ*[フ]{.cjk}[ッ]{.cjk}[ク]{.cjk}*という種類のステップがあります[。]{.chpule2}[
]{.chpuri2}フックは[、]{.chpule2}[
]{.chpuri2}空のデフォルト実装のオプションのステップです[。]{.chpule2}[
]{.chpuri2}フックが上書きされてなくても[、]{.chpule2}[
]{.chpuri2}テンプレート・メソッドは動きます[。]{.chpule2}[
]{.chpuri2}フックは通常アルゴリズムの重要なステップの前後に置かれ[、]{.chpule2}[
]{.chpuri2}サブクラスにアルゴリズムの更なる拡張箇所を提供します[。]{.chpule2}[
]{.chpuri2}
:::

::: {.section .analogy}
##  現実世界でのたとえ {#analogy}

<figure class="image">
<img
src="../../images/patterns/diagrams/template-method/live-exampleea7a.png?id=2485d52852f87da06c9cc0e2fd257d6a"
style="aspect-ratio: 1.74;"
srcset="/images/patterns/diagrams/template-method/live-example.png?id=2485d52852f87da06c9cc0e2fd257d6a 590w,/images/patterns/diagrams/template-method/live-example-1.5x.png?id=92c5fd9345329da09e3233302158678c 885w,/images/patterns/diagrams/template-method/live-example-2x.png?id=89083a3dcd1fe2b627b9b6e6ff4986dc 1180w"
sizes="(max-width: 720px) 100vw, 590px" loading="lazy" width="590"
alt="大規模住宅建設" />
<figcaption><p>典型的な建築プランは<span class="chpule2">、</span><span
class="chpuri2">
</span>顧客の要望に合わせてわずかに変更することが可能<span
class="chpule2">。</span><span class="chpuri2"> </span></p></figcaption>
</figure>

Template Method のやり方は[、]{.chpule2}[
]{.chpuri2}大規模住宅建設に使うことができます[。]{.chpule2}[
]{.chpuri2}標準的な家を建てるための建築プランに[、]{.chpule2}[
]{.chpuri2}購買予定者が細かい部分を調整できるよう[、]{.chpule2}[
]{.chpuri2}いくつかの変更可能箇所を含めることができます[。]{.chpule2}[
]{.chpuri2}

たとえば[、]{.chpule2}[ ]{.chpuri2}基礎工事[、]{.chpule2}[
]{.chpuri2}骨組み[、]{.chpule2}[ ]{.chpuri2}壁面の設置[、]{.chpule2}[
]{.chpuri2}下水の設置[、]{.chpule2}[
]{.chpuri2}水道の配管[、]{.chpule2}[
]{.chpuri2}電気の配線などを少し変えて[、]{.chpule2}[
]{.chpuri2}他とちょっと違った家を建てることができます[。]{.chpule2}[
]{.chpuri2}
:::

::::: {.section .structure-container}
##  構造 {#structure}

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/template-method/structure1c3b.png?id=924692f994bff6578d8408d90f6fc459"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 0.89;"
srcset="/images/patterns/diagrams/template-method/structure.png?id=924692f994bff6578d8408d90f6fc459 340w,/images/patterns/diagrams/template-method/structure-1.5x.png?id=613d78ad8ec08536ec70f4e0976b5a1a 510w,/images/patterns/diagrams/template-method/structure-2x.png?id=25082d6d6a76f51c6b64d8aeeaffdbb5 680w"
sizes="(max-width: 720px) 100vw, 340px" loading="lazy" width="340"
alt="Template Method デザインパターンの構造" /><img
src="../../images/patterns/diagrams/template-method/structure-indexedb94b.png?id=4ced6107519bc66710d2f05c0f4097a1"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 0.92;"
srcset="/images/patterns/diagrams/template-method/structure-indexed.png?id=4ced6107519bc66710d2f05c0f4097a1 350w,/images/patterns/diagrams/template-method/structure-indexed-1.5x.png?id=58e3a9092786c824eb738a7771702093 525w,/images/patterns/diagrams/template-method/structure-indexed-2x.png?id=86f28789cdcc5a4c415d6a1100de56fc 700w"
sizes="(max-width: 720px) 100vw, 350px" loading="lazy" width="350"
alt="Template Method デザインパターンの構造" />
</figure>
:::

1.  **抽象クラス**[ ]{.chpule1}[（]{.chpuri1}Abstract
    Class[）]{.chpule2}[ ]{.chpuri2}は[、]{.chpule2}[
    ]{.chpuri2}アルゴリズムの各ステップに対応するメソッドと[、]{.chpule2}[
    ]{.chpuri2}これらのメソッドを特定の順序で呼び出す実際のテンプレート・メソッドを宣言します[。]{.chpule2}[
    ]{.chpuri2}ステップは[、]{.chpule2}[ ]{.chpuri2}`abstract`
    と宣言されるか[、]{.chpule2}[
    ]{.chpuri2}何らかのデフォルトの実装を持つかのどちらかです[。]{.chpule2}[
    ]{.chpuri2}

2.  **具象クラス**[ ]{.chpule1}[（]{.chpuri1}Concrete
    Class[）]{.chpule2}[ ]{.chpuri2}は[、]{.chpule2}[
    ]{.chpuri2}テンプレート・メソッドを除くすべてのステップを上書きできます[。]{.chpule2}[
    ]{.chpuri2}
::::
:::::

::: {.section .pseudocode}
##  擬似コード {#pseudocode}

この例では[、]{.chpule2}[ ]{.chpuri2}**Template Method**
パターンが[、]{.chpule2}[
]{.chpuri2}簡単な戦略ビデオ・ゲームで[、]{.chpule2}[
]{.chpuri2}人工知能の様々な戦略に対する[
]{.chpule1}[「]{.chpuri1}骨組み[」]{.chpule2}[
]{.chpuri2}を提供します[。]{.chpule2}[ ]{.chpuri2}

<figure class="image">
<img
src="../../images/patterns/diagrams/template-method/example32c6.png?id=c0ce5cc8070925a1cd345fac6afa16b6"
style="aspect-ratio: 1;"
srcset="/images/patterns/diagrams/template-method/example.png?id=c0ce5cc8070925a1cd345fac6afa16b6 430w,/images/patterns/diagrams/template-method/example-1.5x.png?id=d85c6b81c900f46d55688406170600bc 645w,/images/patterns/diagrams/template-method/example-2x.png?id=d8b309659c4b722237ec97733da935bd 860w"
sizes="(max-width: 720px) 100vw, 430px" loading="lazy" width="430"
alt="Template Method パターン例の構造" />
<figcaption><p>単純なビデオ・ゲームの AI クラスたち</p></figcaption>
</figure>

ゲーム内のすべての種族は[、]{.chpule2}[
]{.chpuri2}ほぼ同じ種類の部隊と建物を所有しています[。]{.chpule2}[
]{.chpuri2}そのため[、]{.chpule2}[ ]{.chpuri2}同じ AI
の構造を様々な種族用に再利用できます[。]{.chpule2}[
]{.chpuri2}いくつかの細かいことを上書きするだけです[。]{.chpule2}[
]{.chpuri2}このやり方で[、]{.chpule2}[ ]{.chpuri2}オーク[
]{.chpule1}[（]{.chpuri1}Orc[）]{.chpule2}[ ]{.chpuri2}の AI
を上書きして[、]{.chpule2}[ ]{.chpuri2}より攻撃的にする[、]{.chpule2}[
]{.chpuri2}人類を防衛指向にする[、]{.chpule2}[
]{.chpuri2}モンスターは何も建設できないようにする[、]{.chpule2}[
]{.chpuri2}などができます[。]{.chpule2}[
]{.chpuri2}新しい種族をゲームに追加するには[、]{.chpule2}[
]{.chpuri2}新規の AI のサブクラスを作成し[、]{.chpule2}[ ]{.chpuri2}基底
AI クラスで宣言されたデフォルトのメソッドを上書きします[。]{.chpule2}[
]{.chpuri2}

<figure class="code">
<pre class="code" lang="pseudocode"><code>// この抽象クラスは、何らかのアルゴリズムの骨組みを含むテンプレート・メ
// ソッドを一つ定義する。このメソッドは、通常は abstract と宣言された原始
// 操作メソッドに対する呼び出しから構成される。具象サブクラスは、これらの
// 原始操作を実装するが、テンプレート・メソッドはそのままにする。
class GameAI is
    // テンプレート・メソッドはアルゴリズムの骨組みを定義する。
    method turn() is
        collectResources()
        buildStructures()
        buildUnits()
        attack()

    // ステップのいくつかは、基底クラスで定義されているかもしれない。
    method collectResources() is
        foreach (s in this.builtStructures) do
            s.collect()

    // そしていくつかは、abstract と定義されているかもしれない。
    abstract method buildStructures()
    abstract method buildUnits()

    // クラスには、複数のテンプレート・メソッドがあってもよい。
    method attack() is
        enemy = closestEnemy()
        if (enemy == null)
            sendScouts(map.center)
        else
            sendWarriors(enemy.position)

    abstract method sendScouts(position)
    abstract method sendWarriors(position)

// 具象クラスは、基底クラス中の全ての抽象操作（メソッド）を実装しなければ
// ならないが、テンプレート・メソッドを上書きしてはならない。
class OrcsAI extends GameAI is
    method buildStructures() is
        if (there are some resources) then
            // 農場を建設し、兵舎を建設し、拠点を築く。

    method buildUnits() is
        if (there are plenty of resources) then
            if (there are no scouts)
                // ピオンを作り、それを偵察隊グループに追加。
            else
                // グラントを作り、それを戦闘員グループに追加。

    // ……

    method sendScouts(position) is
        if (scouts.length &gt; 0) then
            // 偵察隊を配置につける。

    method sendWarriors(position) is
        if (warriors.length &gt; 5) then
            // 兵隊を配置につける。

// サブクラスでは、デフォルト実装付きの操作のいくつかを上書きすることもで
// きる。
class MonstersAI extends GameAI is
    method collectResources() is
        // モンスターは資源を集めない。

    method buildStructures() is
        // モンスターは構造物を建設しない。

    method buildUnits() is
        // モンスターは部隊を編成しない。</code></pre>
</figure>
:::

:::::::: {.section .applicability-container}
##  適応性 {#applicability}

::::::: applicability
::: applicability-problem
クライアントにアルゴリズムの特定のステップのみを拡張させたいが全体のアルゴリズムや構造は変えたくない場合に[、]{.chpule2}[
]{.chpuri2}Template Method パターンを使用します[。]{.chpule2}[
]{.chpuri2}
:::

::: applicability-solution
Template メソッドを使用すると[、]{.chpule2}[
]{.chpuri2}一枚岩のアルゴリズムを[、]{.chpule2}[
]{.chpuri2}一連のステップに分割し[、]{.chpule2}[
]{.chpuri2}サブクラスで簡単に拡張できるようにできます[。]{.chpule2}[
]{.chpuri2}スーパークラスで定義された構造は[、]{.chpule2}[
]{.chpuri2}そのまま維持されます[。]{.chpule2}[ ]{.chpuri2}
:::

::: applicability-problem
少数の些細な違いを除いてほとんど同じアルゴリズムのクラスがいくつかある場合[、]{.chpule2}[
]{.chpuri2}このパターンを使用します[。]{.chpule2}[
]{.chpuri2}結果として[、]{.chpule2}[
]{.chpuri2}アルゴリズムを変更する場合は[、]{.chpule2}[
]{.chpuri2}すべてのクラスを変更する必要があるかもしれません[。]{.chpule2}[
]{.chpuri2}
:::

::: applicability-solution
アルゴリズムをテンプレート・メソッドに転換する時に[、]{.chpule2}[
]{.chpuri2}類似の実装を含むステップをスーパークラスに移動して[、]{.chpule2}[
]{.chpuri2}コードの重複を削除できます[。]{.chpule2}[
]{.chpuri2}サブクラス間で異なるコードはサブクラスに留めます[。]{.chpule2}[
]{.chpuri2}
:::
:::::::
::::::::

::: {.section .checklist}
##  実装方法 {#checklist}

1.  対象となるアルゴリズムを分析し[、]{.chpule2}[
    ]{.chpuri2}それをステップに分割できるかどうかを検討します[。]{.chpule2}[
    ]{.chpuri2}どのステップがサブクラス間で共通で[、]{.chpule2}[
    ]{.chpuri2}どのステップが常にサブクラスに固有かを考察します[。]{.chpule2}[
    ]{.chpuri2}

2.  抽象基底クラスを作成し[、]{.chpule2}[
    ]{.chpuri2}テンプレート・メソッドと[、]{.chpule2}[
    ]{.chpuri2}アルゴリズムの各ステップに対応する抽象メソッドの組を宣言します[。]{.chpule2}[
    ]{.chpuri2}アルゴリズムの輪郭は[、]{.chpule2}[
    ]{.chpuri2}ステップを実行するテンプレート・メソッドに描かれています[。]{.chpule2}[
    ]{.chpuri2}テンプレート・メソッドがサブクラスで上書きされるのを防ぐために[、]{.chpule2}[
    ]{.chpuri2}テンプレート メソッドを `final`
    と宣言することを検討してください[。]{.chpule2}[ ]{.chpuri2}

3.  全部のステップが抽象メソッドとなってしまったとしても[、]{.chpule2}[
    ]{.chpuri2}大丈夫です[。]{.chpule2}[
    ]{.chpuri2}いくつかのメソッドにはデフォルトの実装があった方が有益かもしれません[。]{.chpule2}[
    ]{.chpuri2}サブクラスがそれらのメソッドを実装する必要がなくなります[。]{.chpule2}[
    ]{.chpuri2}

4.  アルゴリズムの重要なステップの間にフックを追加することを検討してください[。]{.chpule2}[
    ]{.chpuri2}

5.  アルゴリズムの異種ごとに[、]{.chpule2}[
    ]{.chpuri2}新しい具象サブクラスを作成します[。]{.chpule2}[
    ]{.chpuri2}サブクラスは[、]{.chpule2}[
    ]{.chpuri2}抽象ステップのすべてを実装*[し]{.cjk}[な]{.cjk}[け]{.cjk}[れ]{.cjk}[ば]{.cjk}[な]{.cjk}[ら]{.cjk}[な]{.cjk}[い]{.cjk}*ですが[、]{.chpule2}[
    ]{.chpuri2}任意のステップは上書き*[し]{.cjk}[て]{.cjk}[も]{.cjk}[よ]{.cjk}[い]{.cjk}*です[。]{.chpule2}[
    ]{.chpuri2}
:::

:::::: {.section .pros-cons}
##  長所と短所 {#pros-cons}

::::: row
::: col-sm-6
-    クライアントは[、]{.chpule2}[
    ]{.chpuri2}大きなアルゴリズムの特定の部分だけを上書き可能[。]{.chpule2}[
    ]{.chpuri2}アルゴリズムの他部分への変更の影響を受けにくい[。]{.chpule2}[
    ]{.chpuri2}
-    重複コードをスーパークラスに取り入れ可能[。]{.chpule2}[ ]{.chpuri2}
:::

::: col-sm-6
-    いくつかのクライアントにとっては[、]{.chpule2}[
    ]{.chpuri2}与えられたアルゴリズムの骨組みは[、]{.chpule2}[
    ]{.chpuri2}そのロジックを表現するために不十分である可能性[。]{.chpule2}[
    ]{.chpuri2}
-    サブクラスでステップのデフォルト実装を抑制することは[、]{.chpule2}[
    ]{.chpuri2}*[リ]{.cjk}[ス]{.cjk}[コ]{.cjk}[フ]{.cjk}[の]{.cjk}[置]{.cjk}[換]{.cjk}[原]{.cjk}[則]{.cjk}*違反の可能性[。]{.chpule2}[
    ]{.chpuri2}
-    ステップ数が多いほど[、]{.chpule2}[
    ]{.chpuri2}テンプレート・メソッドの保守が困難となりがち[。]{.chpule2}[
    ]{.chpuri2}
:::
:::::
::::::

::: {.section .relations}
##  他のパターンとの関係 {#relations}

-   [Factory Method](factory-method.html) は[、]{.chpule2}[
    ]{.chpuri2}[Template Method](template-method.html)
    の特別な場合です[。]{.chpule2}[ ]{.chpuri2}同時に[、]{.chpule2}[
    ]{.chpuri2}*Factory Method* は[、]{.chpule2}[ ]{.chpuri2}大きな
    *Template Method*
    の一つのステップとして使うこともできます[。]{.chpule2}[ ]{.chpuri2}

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
:::

::: {.section .implementations}
##  コード例 {#implementations}

[![Template Method を C#
で](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](template-method/csharp/example.html "Template Method を C# で"){.prog-lang-link}
[![Template Method を C++
で](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](template-method/cpp/example.html "Template Method を C++ で"){.prog-lang-link}
[![Template Method を Go
で](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](template-method/go/example.html "Template Method を Go で"){.prog-lang-link}
[![Template Method を Java
で](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](template-method/java/example.html "Template Method を Java で"){.prog-lang-link}
[![Template Method を PHP
で](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](template-method/php/example.html "Template Method を PHP で"){.prog-lang-link}
[![Template Method を Python
で](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](template-method/python/example.html "Template Method を Python で"){.prog-lang-link}
[![Template Method を Ruby
で](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](template-method/ruby/example.html "Template Method を Ruby で"){.prog-lang-link}
[![Template Method を Rust
で](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](template-method/rust/example.html "Template Method を Rust で"){.prog-lang-link}
[![Template Method を Swift
で](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](template-method/swift/example.html "Template Method を Swift で"){.prog-lang-link}
[![Template Method を TypeScript
で](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](template-method/typescript/example.html "Template Method を TypeScript で"){.prog-lang-link}
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

[Visitor []{.fa .fa-arrow-right}](visitor.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### 戻る

[[]{.fa .fa-arrow-left} Strategy](strategy.html){.btn .btn-default
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
