::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} /
[デザインパターン](../design-patterns.html){.type} /
[振る舞いに関するパターン](behavioral-patterns.html){.type}
:::

# State {#state .title}

::: aka
別名：[ステート]{style="display:inline-block"}
:::

::: {.section .intent}
##  一言でいうと {#intent}

**State**[ ]{.chpule1}[（]{.chpuri1}ステート[、]{.chpule2}[
]{.chpuri2}状態[）]{.chpule2}[ ]{.chpuri2}は[、]{.chpule2}[
]{.chpuri2}振る舞いに関するデザインパターンの一つで[、]{.chpule2}[
]{.chpuri2}オブジェクトの内部状態が変化した時に[、]{.chpule2}[
]{.chpuri2}その挙動を変化させます[。]{.chpule2}[
]{.chpuri2}それは[、]{.chpule2}[
]{.chpuri2}あたかもそのオブジェクトのクラスが変わったかのように見えます[。]{.chpule2}[
]{.chpuri2}

<figure class="image">
<img
src="../../images/patterns/content/state/state-ja7c4a.png?id=27084c79e1db27be96fdd17e0ce25932"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/state/state-ja.png?id=27084c79e1db27be96fdd17e0ce25932 640w,/images/patterns/content/state/state-ja-1.5x.png?id=b3f892276c00f9b0e43117a7331638db 960w,/images/patterns/content/state/state-ja-2x.png?id=1b444e9c1bc600e3404955756b61935e 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="State デザインパターン" />
</figure>
:::

::: {.section .problem}
##  問題 {#problem}

State パターンは[、]{.chpule2}[
]{.chpuri2}**有限オートマトン** []{.pop-i}[Finite-State
Machine[、]{.chpule2}[ ]{.chpuri2}有限状態機械[、]{.chpule2}[
]{.chpuri2}ステートマシンとも[：]{.chpule2}[
]{.chpuri2}[https://refactoring.guru/ja/fsm](https://ja.wikipedia.org/wiki/有限オートマトン)]{.pop-content}の概念と密接に関連しています[。]{.chpule2}[
]{.chpuri2}

<figure class="image">
<img
src="../../images/patterns/diagrams/state/problem10244.png?id=503968745461a0970d1fbb4362dc186f"
style="aspect-ratio: 1.45;"
srcset="/images/patterns/diagrams/state/problem1.png?id=503968745461a0970d1fbb4362dc186f 320w,/images/patterns/diagrams/state/problem1-1.5x.png?id=47c7ca068eceaa9896d320690e6f3368 480w,/images/patterns/diagrams/state/problem1-2x.png?id=ae03c2233939eace11d44925ddeb912d 640w"
sizes="(max-width: 720px) 100vw, 320px" loading="lazy" width="320"
alt="有限状態機械" />
<figcaption aria-hidden="true"><p>有限状態機械</p></figcaption>
</figure>

基本的な考えとしては[、]{.chpule2}[
]{.chpuri2}いかなる時点でも[、]{.chpule2}[
]{.chpuri2}プログラムが取ることのできる*[有]{.cjk}[限]{.cjk}*個の*[状]{.cjk}[態]{.cjk}*がある[、]{.chpule2}[
]{.chpuri2}ということです[。]{.chpule2}[
]{.chpuri2}ある固有の状態では[、]{.chpule2}[
]{.chpuri2}プログラムは異なる振る舞いを見せ[、]{.chpule2}[
]{.chpuri2}一つの状態から別の状態に瞬時に切り替え可能です[。]{.chpule2}[
]{.chpuri2}しかし[、]{.chpule2}[
]{.chpuri2}現在に状態によっては[、]{.chpule2}[
]{.chpuri2}プログラムは特定の別の状態に移行できるかもしれませんし[、]{.chpule2}[
]{.chpuri2}移行できないかもしれません[。]{.chpule2}[
]{.chpuri2}この移行の規則は[、]{.chpule2}[
]{.chpuri2}*[遷]{.cjk}[移]{.cjk}*と呼ばれ[、]{.chpule2}[
]{.chpuri2}有限個であり[、]{.chpule2}[
]{.chpuri2}予め決められています[。]{.chpule2}[ ]{.chpuri2}

このやり方をオブジェクトに当てはめることもできます[。]{.chpule2}[
]{.chpuri2}`Document`[
]{.chpule1}[（]{.chpuri1}ドキュメント[）]{.chpule2}[
]{.chpuri2}クラスが一つあると仮定してみてください[。]{.chpule2}[
]{.chpuri2}ドキュメントは[、]{.chpule2}[
]{.chpuri2}次の三つの状態を取ることができます[：]{.chpule2}[
]{.chpuri2}`Draft`[
]{.chpule1}[（]{.chpuri1}下書き[）]{.ls-05}[、]{.chpule2}[
]{.chpuri2}`Moderation`[
]{.chpule1}[（]{.chpuri1}考査中[）]{.ls-05}[、]{.chpule2}[
]{.chpuri2}`Published`[
]{.chpule1}[（]{.chpuri1}発行済み[）]{.ls-05}[。]{.chpule2}[
]{.chpuri2}ドキュメントの ` publish` メソッドは[、]{.chpule2}[
]{.chpuri2}それぞれの状態でちょっとずつ違って機能します[。]{.chpule2}[
]{.chpuri2}

-   `Draft` では[、]{.chpule2}[ ]{.chpuri2}ドキュメントを Moderation
    に移行します[。]{.chpule2}[ ]{.chpuri2}
-   `Moderation` では[、]{.chpule2}[
    ]{.chpuri2}現在のユーザーが管理者である場合にのみ[、]{.chpule2}[
    ]{.chpuri2}ドキュメントを公開にします[。]{.chpule2}[ ]{.chpuri2}
-   `Published` では[、]{.chpule2}[
    ]{.chpuri2}まったく何もしません[。]{.chpule2}[ ]{.chpuri2}

<figure class="image">
<img
src="../../images/patterns/diagrams/state/problem2-ja0c39.png?id=70328727e934648994de819f6813e20f"
style="aspect-ratio: 1.25;"
srcset="/images/patterns/diagrams/state/problem2-ja.png?id=70328727e934648994de819f6813e20f 550w,/images/patterns/diagrams/state/problem2-ja-1.5x.png?id=481ad7bc6ea39856a7f946d5048fdcca 825w,/images/patterns/diagrams/state/problem2-ja-2x.png?id=0f8c7539d0bc30d44cf38c5d1444e812 1100w"
sizes="(max-width: 720px) 100vw, 550px" loading="lazy" width="550"
alt="ドキュメント・オブジェクトの可能な状態" />
<figcaption><p>ドキュメント・オブジェクトが取り得る状態と遷移<span
class="chpule2">。</span><span class="chpuri2"> </span></p></figcaption>
</figure>

状態機械は通常[、]{.chpule2}[
]{.chpuri2}オブジェクトの現在の状態に応じて適切な動作を選択する多くの条件付き演算子[
]{.chpule1}[（]{.chpuri1}`if` または `switch`[）]{.chpule2}[
]{.chpuri2}で実装されます[。]{.chpule2}[ ]{.chpuri2}通常[、]{.chpule2}[
]{.chpuri2}この[ ]{.chpule1}[「]{.chpuri1}状態[」]{.chpule2}[
]{.chpuri2}はオブジェクトのフィールドの値の集合です[。]{.chpule2}[
]{.chpuri2}仮に今まで有限状態機械なんて聞いたことがないとしても[、]{.chpule2}[
]{.chpuri2}おそらく少なくとも 1
回は状態を実装したことがあるはずです[。]{.chpule2}[
]{.chpuri2}次のコードの構造に[、]{.chpule2}[
]{.chpuri2}何か見覚えないですか[？]{.chpule2}[ ]{.chpuri2}

<figure class="code">
<pre class="code" lang="pseudocode"><code>class Document is
    field state: string
    // ……
    method publish() is
        switch (state)
            &quot;draft&quot;:
                state = &quot;moderation&quot;
                break
            &quot;moderation&quot;:
                if (currentUser.role == &quot;admin&quot;)
                    state = &quot;published&quot;
                break
            &quot;published&quot;:
                // 何もしない
                break
    // ……</code></pre>
</figure>

条件文に基づく状態機械の最大の弱点が[、]{.chpule2}[
]{.chpuri2}`Document`
クラスに新しい状態と状態依存の動作とを次々に追加し始めると露呈します[。]{.chpule2}[
]{.chpuri2}ほとんどのメソッドは[、]{.chpule2}[
]{.chpuri2}現在の状態に従ってメソッドの適切な動作を選択する退屈な条件文から成ります[。]{.chpule2}[
]{.chpuri2}遷移ロジックを変更するためには[、]{.chpule2}[
]{.chpuri2}すべてのメソッドで状態条件を変更する必要があり[、]{.chpule2}[
]{.chpuri2}コードを保守が非常に困難になります[。]{.chpule2}[ ]{.chpuri2}

プロジェクトが進展するにつれて問題は大きくなりがちです[。]{.chpule2}[
]{.chpuri2}設計段階ですべての可能な状態や遷移を予測することは非常に困難です[。]{.chpule2}[
]{.chpuri2}したがって[、]{.chpule2}[
]{.chpuri2}限られた条件文で作られた無駄のない状態機械は[、]{.chpule2}[
]{.chpuri2}時間の経過とともに膨張した化け物になりがちです[。]{.chpule2}[
]{.chpuri2}
:::

::: {.section .solution}
##  解決策 {#solution}

State パターンに従うと[、]{.chpule2}[
]{.chpuri2}オブジェクトのすべての可能な状態に対して新しいクラス[
]{.chpule1}[（]{.chpuri1}訳註[：]{.chpule2}[
]{.chpuri2}一つの状態ごとに一つのクラス[）]{.chpule2}[
]{.chpuri2}を作成し[、]{.chpule2}[
]{.chpuri2}状態固有の振る舞いをこれらのクラスに抽出します[。]{.chpule2}[
]{.chpuri2}

すべての振る舞いを単独で実装する代わりに[、]{.chpule2}[
]{.chpuri2}*[コ]{.cjk}[ン]{.cjk}[テ]{.cjk}[キ]{.cjk}[ス]{.cjk}[ト]{.cjk}*[
]{.chpule1}[（]{.chpuri1}context[、]{.chpule2}[
]{.chpuri2}前後関係[）]{.chpule2}[
]{.chpuri2}と呼ばれる元々のオブジェクトが[、]{.chpule2}[
]{.chpuri2}現在の状態を表す状態オブジェクトの一つへの参照を持ち[、]{.chpule2}[
]{.chpuri2}そのオブジェクトにすべての状態関連の作業を委任します[。]{.chpule2}[
]{.chpuri2}

<figure class="image">
<img
src="../../images/patterns/diagrams/state/solution-jaaa78.png?id=f95e3275b5dd313dfb0bb3eacaa5a7ec"
style="aspect-ratio: 1.53;"
srcset="/images/patterns/diagrams/state/solution-ja.png?id=f95e3275b5dd313dfb0bb3eacaa5a7ec 490w,/images/patterns/diagrams/state/solution-ja-1.5x.png?id=6320a73a05e70e0b92607ee204f15e9c 735w,/images/patterns/diagrams/state/solution-ja-2x.png?id=6d06e3b589293ec1b44ada1c38e382cf 980w"
sizes="(max-width: 720px) 100vw, 490px" loading="lazy" width="490"
alt="ドキュメントは、その作業を状態オブジェクトに委任" />
<figcaption><p>ドキュメントは<span class="chpule2">、</span><span
class="chpuri2"> </span>その作業を状態オブジェクトに委任<span
class="chpule2">。</span><span class="chpuri2"> </span></p></figcaption>
</figure>

コンテキストを別の状態に遷移するには[、]{.chpule2}[
]{.chpuri2}現在有効な状態オブジェクトを[、]{.chpule2}[
]{.chpuri2}新しい状態を表す別のオブジェクトで置き換えます[。]{.chpule2}[
]{.chpuri2}これは[、]{.chpule2}[ ]{.chpuri2}すべての[
]{.chpule1}[（]{.chpuri1}具象[）]{.chpule2}[
]{.chpuri2}ステート・クラスが同じインターフェースに従っていて[、]{.chpule2}[
]{.chpuri2}コンテキスト自体がそのインターフェースを介し[、]{.chpule2}[
]{.chpuri2}これらのオブジェクトとやりとりする場合にのみ可能です[。]{.chpule2}[
]{.chpuri2}

この構造は[、]{.chpule2}[ ]{.chpuri2}[Strategy](strategy.html)
パターンに似ているように見えますが[、]{.chpule2}[
]{.chpuri2}重要な違いが一つあります[。]{.chpule2}[ ]{.chpuri2}State
パターンでは[、]{.chpule2}[
]{.chpuri2}特定の状態同士はお互いを知っており[、]{.chpule2}[
]{.chpuri2}ある状態から別の状態への遷移を開始しますが[、]{.chpule2}[
]{.chpuri2}ストラテジーは[、]{.chpule2}[
]{.chpuri2}お互いを知っていることはまずありません[。]{.chpule2}[
]{.chpuri2}
:::

::: {.section .analogy}
##  現実世界でのたとえ {#analogy}

スマートフォーンのボタンとスイッチは[、]{.chpule2}[
]{.chpuri2}現在の機器の状態によって異なる振る舞いをします[。]{.chpule2}[
]{.chpuri2}

-   携帯電話がロックされてない時は[、]{.chpule2}[
    ]{.chpuri2}ボタンを押すと様々な機能が実行されます[。]{.chpule2}[
    ]{.chpuri2}
-   電話がロックされていると[、]{.chpule2}[
    ]{.chpuri2}どのボタンを押してもロック解除画面が表示されます[。]{.chpule2}[
    ]{.chpuri2}
-   携帯電話の充電量が少ないと[、]{.chpule2}[
    ]{.chpuri2}どのボタンを押しても充電画面が表示されます[。]{.chpule2}[
    ]{.chpuri2}
:::

::::: {.section .structure-container}
##  構造 {#structure}

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/state/structure-jaf2fa.png?id=42cf6f603b53ba137a06a1baec09298d"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.32;"
srcset="/images/patterns/diagrams/state/structure-ja.png?id=42cf6f603b53ba137a06a1baec09298d 540w,/images/patterns/diagrams/state/structure-ja-1.5x.png?id=faa7e834db4164c6917b9a388c9cef30 810w,/images/patterns/diagrams/state/structure-ja-2x.png?id=da6f1d843e4d8be6409f8bdeade95ce9 1080w"
sizes="(max-width: 720px) 100vw, 540px" loading="lazy" width="540"
alt="State デザインパターンの構造" /><img
src="../../images/patterns/diagrams/state/structure-ja-indexedb4d1.png?id=8cb9c379f8acb15292a66f5da4f8444c"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.26;"
srcset="/images/patterns/diagrams/state/structure-ja-indexed.png?id=8cb9c379f8acb15292a66f5da4f8444c 540w,/images/patterns/diagrams/state/structure-ja-indexed-1.5x.png?id=73918a764ad03862603fb5f451e2608a 810w,/images/patterns/diagrams/state/structure-ja-indexed-2x.png?id=cd7fb5819ed71f4583c4257aa70c243a 1080w"
sizes="(max-width: 720px) 100vw, 540px" loading="lazy" width="540"
alt="State デザインパターンの構造" />
</figure>
:::

1.  **コンテキスト**[ ]{.chpule1}[（]{.chpuri1}Context[）]{.chpule2}[
    ]{.chpuri2}は[、]{.chpule2}[
    ]{.chpuri2}複数個ある具象状態クラスのオブジェクトのいずれか一つへ参照を保存し[、]{.chpule2}[
    ]{.chpuri2}状態に固有の作業は[、]{.chpule2}[
    ]{.chpuri2}すべてそのオブジェクトに委任されます[。]{.chpule2}[
    ]{.chpuri2}コンテキストは[、]{.chpule2}[
    ]{.chpuri2}状態インターフェースを介して状態オブジェクトと通信します[。]{.chpule2}[
    ]{.chpuri2}コンテキストには[、]{.chpule2}[
    ]{.chpuri2}新しい状態オブジェクトを渡すための公開の setter
    があります[。]{.chpule2}[ ]{.chpuri2}

2.  **ステート**[ ]{.chpule1}[（]{.chpuri1}State[）]{.chpule2}[
    ]{.chpuri2}インターフェースは状態固有のメソッドを宣言します[。]{.chpule2}[
    ]{.chpuri2}これらのメソッドはすべての具象ステートに意味のあるものでなければなりません[。]{.chpule2}[
    ]{.chpuri2}いくつかの状態では絶対に呼ばれない無駄なメソッドがないようにしたいからです[。]{.chpule2}[
    ]{.chpuri2}

3.  **具象ステート**[ ]{.chpule1}[（]{.chpuri1}Concrete
    State[、]{.chpule2}[ ]{.chpuri2}具象状態[）]{.chpule2}[
    ]{.chpuri2}は[、]{.chpule2}[
    ]{.chpuri2}状態固有のメソッドに対して独自の実装を行います[。]{.chpule2}[
    ]{.chpuri2}似たようなコードが複数の状態で重複することを避けるために[、]{.chpule2}[
    ]{.chpuri2}共通の振る舞いを内包した中間層の抽象クラスを作ることもできます[。]{.chpule2}[
    ]{.chpuri2}

    ステート・オブジェクトは[、]{.chpule2}[
    ]{.chpuri2}コンテキスト・オブジェクトへの逆参照を格納するようにもできます[。]{.chpule2}[
    ]{.chpuri2}この参照を通して[、]{.chpule2}[
    ]{.chpuri2}ステートはコンテキスト・オブジェクトから必要な情報を取得したり[、]{.chpule2}[
    ]{.chpuri2}状態遷移を開始したりできます[。]{.chpule2}[ ]{.chpuri2}

4.  コンテキストと具象ステートの両方とも[、]{.chpule2}[
    ]{.chpuri2}コンテキストの次の状態の設定が可能で[、]{.chpule2}[
    ]{.chpuri2}コンテキストにリンクされたステート・オブジェクトを置き換えることにより実際の状態遷移を行います[。]{.chpule2}[
    ]{.chpuri2}
::::
:::::

::: {.section .pseudocode}
##  擬似コード {#pseudocode}

この例では[、]{.chpule2}[ ]{.chpuri2}**State**
パターンにより[、]{.chpule2}[
]{.chpuri2}メディア・プレーヤーの現在の再生状態に応じて[、]{.chpule2}[
]{.chpuri2}同じコントロール[
]{.chpule1}[（]{.chpuri1}訳注[：]{.chpule2}[ ]{.chpuri2}UI
のボタンなど[）]{.chpule2}[
]{.chpuri2}に異なる振る舞いをさせます[。]{.chpule2}[ ]{.chpuri2}

<figure class="image">
<img
src="../../images/patterns/diagrams/state/example1dc9.png?id=1ffdb109b3ebb85d223b9d1651d2034c"
style="aspect-ratio: 1.64;"
srcset="/images/patterns/diagrams/state/example.png?id=1ffdb109b3ebb85d223b9d1651d2034c 590w,/images/patterns/diagrams/state/example-1.5x.png?id=a9ff56d0a139530fa103d496513dfa06 885w,/images/patterns/diagrams/state/example-2x.png?id=cd81e3ffb8aba5883983a59c111b805f 1180w"
sizes="(max-width: 720px) 100vw, 590px" loading="lazy" width="590"
alt="State パターン例の構造" />
<figcaption><p>オブジェクトの振る舞いを状態オブジェクトで変更する例<span
class="chpule2">。</span><span class="chpuri2"> </span></p></figcaption>
</figure>

プレーヤーの主オブジェクトは[、]{.chpule2}[
]{.chpuri2}プレーヤーのほとんどの作業を実行する状態オブジェクトのどれか一つに常にリンクされています[。]{.chpule2}[
]{.chpuri2}ユーザーの行う操作によっては[、]{.chpule2}[
]{.chpuri2}プレーヤーの現在の状態オブジェクトを別のものに置き換え[、]{.chpule2}[
]{.chpuri2}プレーヤーがユーザーとの対話方法を変更します[。]{.chpule2}[
]{.chpuri2}

<figure class="code">
<pre class="code" lang="pseudocode"><code>// AudioPlayer クラスはコンテキストとして機能する。また、オーディオ・プ
// レーヤーの現在の状態に対応する状態クラスのインスタンスへの参照も維持す
// る。
class AudioPlayer is
    field state: State
    field UI, volume, playlist, currentSong

    constructor AudioPlayer() is
        this.state = new ReadyState(this)

        // コンテキストは、ユーザー入力の処理をステート・オブジェクトに委
        // 任。状態＝ステートによって、入力を違うように扱うため、結果は現
        // 在有効な状態に当然依存する。
        UI = new UserInterface()
        UI.lockButton.onClick(this.clickLock)
        UI.playButton.onClick(this.clickPlay)
        UI.nextButton.onClick(this.clickNext)
        UI.prevButton.onClick(this.clickPrevious)

    // 他のオブジェクトがオーディオ・プレーヤーの現在有効な状態を変更する
    // ことを可能とするメソッド。
    method changeState(state: State) is
        this.state = state

    // UI メソッドは、現在有効なステートに実行を委任。
    method clickLock() is
        state.clickLock()
    method clickPlay() is
        state.clickPlay()
    method clickNext() is
        state.clickNext()
    method clickPrevious() is
        state.clickPrevious()

    // ステートは、コンテキスト上のサービス・メソッドをいくつか呼び出すか
    // もしれない。
    method startPlayback() is
        // ……
    method stopPlayback() is
        // ……
    method nextSong() is
        // ……
    method previousSong() is
        // ……
    method fastForward(time) is
        // ……
    method rewind(time) is
        // ……


// ステートの基底クラスは、全部の具象ステートが実装すべきメソッドを宣言し、
// ステートと関連するコンテキスト・オブジェクトに対する逆参照を提供する。
// ステートは、コンテキストを別の状態に遷移するために逆参照を使用できる。
abstract class State is
    protected field player: AudioPlayer

    // コンテキストは、ステートのコンストラクターに渡される。これは、もし
    // 必要ならば、ステートが有用なコンテキスト・データを取得する助けとな
    // る。
    constructor State(player) is
        this.player = player

    abstract method clickLock()
    abstract method clickPlay()
    abstract method clickNext()
    abstract method clickPrevious()


// 具象ステートは、コンテキストの状態に結びついた種々の振る舞いを実装。
class LockedState extends State is

    // ユーザーがロックされていたプレーヤーのロックを解除すると、二つの状
    // 態のうちの一つになる。
    method clickLock() is
        if (player.playing)
            player.changeState(new PlayingState(player))
        else
            player.changeState(new ReadyState(player))

    method clickPlay() is
        // ロックされているので何もしない。

    method clickNext() is
        // ロックされているので何もしない。

    method clickPrevious() is
        // ロックされているので何もしない。


// コンテキスト中の状態遷移の引き金ともなり得る。
class ReadyState extends State is
    method clickLock() is
        player.changeState(new LockedState(player))

    method clickPlay() is
        player.startPlayback()
        player.changeState(new PlayingState(player))

    method clickNext() is
        player.nextSong()

    method clickPrevious() is
        player.previousSong()


class PlayingState extends State is
    method clickLock() is
        player.changeState(new LockedState(player))

    method clickPlay() is
        player.stopPlayback()
        player.changeState(new ReadyState(player))

    method clickNext() is
        if (event.doubleclick)
            player.nextSong()
        else
            player.fastForward(5)

    method clickPrevious() is
        if (event.doubleclick)
            player.previous()
        else
            player.rewind(5)</code></pre>
</figure>
:::

:::::::::: {.section .applicability-container}
##  適応性 {#applicability}

::::::::: applicability
::: applicability-problem
現在の状態に応じて異なる振る舞いをするオブジェクトがあり[、]{.chpule2}[
]{.chpuri2}状態数が多く[、]{.chpule2}[
]{.chpuri2}状態固有のコードの変更が頻繁な場合に[、]{.chpule2}[
]{.chpuri2}State パターンを使用します[。]{.chpule2}[ ]{.chpuri2}
:::

::: applicability-solution
このパターンに従うには[、]{.chpule2}[
]{.chpuri2}すべての状態固有のコードを抽出し[、]{.chpule2}[
]{.chpuri2}複数の個別のクラスに入れます[。]{.chpule2}[
]{.chpuri2}その結果[、]{.chpule2}[
]{.chpuri2}新しい状態を追加したり[、]{.chpule2}[
]{.chpuri2}既存の状態を変更したりすることが独立して可能になり[、]{.chpule2}[
]{.chpuri2}保守コストを削減できます[。]{.chpule2}[ ]{.chpuri2}
:::

::: applicability-problem
クラスのフィールドの現在の値に応じて振る舞いを変えるために[、]{.chpule2}[
]{.chpuri2}クラスが膨大な数の条件文であふれている場合に[、]{.chpule2}[
]{.chpuri2}このパターンを使用します[。]{.chpule2}[ ]{.chpuri2}
:::

::: applicability-solution
State パターンでは[、]{.chpule2}[
]{.chpuri2}これらの条件文の分岐を[、]{.chpule2}[
]{.chpuri2}対応する状態クラスのメソッドとして抽出します[。]{.chpule2}[
]{.chpuri2}これを行う際には[、]{.chpule2}[
]{.chpuri2}主クラス中の状態固有コードに関連した一時的なフィールドやヘルパー・メソッドを一掃することもできます[。]{.chpule2}[
]{.chpuri2}
:::

::: applicability-problem
似たような複数の状態間や条件文に基づく状態機械の状態遷移に多数の重複したコードがある場合[、]{.chpule2}[
]{.chpuri2}State パターンを使用します[。]{.chpule2}[ ]{.chpuri2}
:::

::: applicability-solution
State パターンを使用すると[、]{.chpule2}[
]{.chpuri2}状態クラスの階層を作成し[、]{.chpule2}[
]{.chpuri2}共通のコードを基底抽象クラスに抽出して重複を減らすことができます[。]{.chpule2}[
]{.chpuri2}
:::
:::::::::
::::::::::

::: {.section .checklist}
##  実装方法 {#checklist}

1.  コンテキストとして適切なクラスを決定します[。]{.chpule2}[
    ]{.chpuri2}状態依存のコードがある既存のクラスがそれかもしれません[。]{.chpule2}[
    ]{.chpuri2}もし状態固有のコードが複数のクラスに分散している場合は[、]{.chpule2}[
    ]{.chpuri2}新規クラスが必要となります[。]{.chpule2}[ ]{.chpuri2}

2.  状態インターフェースを宣言します[。]{.chpule2}[
    ]{.chpuri2}コンテキスト内で宣言されたメソッド全部を反映させることもできますが[、]{.chpule2}[
    ]{.chpuri2}ステート固有の振る舞いを持つものだけに絞ることを目指してください[。]{.chpule2}[
    ]{.chpuri2}

3.  実際の状態ごとに[、]{.chpule2}[
    ]{.chpuri2}状態インターフェースを実装するクラスを作成します[。]{.chpule2}[
    ]{.chpuri2}次に[、]{.chpule2}[
    ]{.chpuri2}コンテキストのメソッドを見渡し[、]{.chpule2}[
    ]{.chpuri2}その状態に関連するすべてのコードを新しく作成したクラスに抽出します[。]{.chpule2}[
    ]{.chpuri2}

    コードを状態クラスに移動する際に[、]{.chpule2}[
    ]{.chpuri2}それがコンテキストの非公開メンバーに依存していることを発見するかもしれません[。]{.chpule2}[
    ]{.chpuri2}これにはいくつかの回避策があります[：]{.chpule2}[
    ]{.chpuri2}

    -   これらのフィールドまたはメソッドを公開にする[。]{.chpule2}[
        ]{.chpuri2}
    -   抽出中の振る舞いをコンテキスト中の公開メソッドとし[、]{.chpule2}[
        ]{.chpuri2}状態クラスから呼び出す[。]{.chpule2}[
        ]{.chpuri2}醜いが[、]{.chpule2}[
        ]{.chpuri2}即効性があり[、]{.chpule2}[
        ]{.chpuri2}後で修正可能[。]{.chpule2}[ ]{.chpuri2}
    -   ステート・クラスをコンテキスト・クラスにネスト[。]{.chpule2}[
        ]{.chpuri2}ただし[、]{.chpule2}[
        ]{.chpuri2}プログラミング言語がクラスのネストをサポートしている場合に限定[。]{.chpule2}[
        ]{.chpuri2}

4.  コンテキスト・クラスに[、]{.chpule2}[
    ]{.chpuri2}状態インターフェースの型の参照フィールドと[、]{.chpule2}[
    ]{.chpuri2}そのフィールドの値を変更できる公開 setter
    を追加します[。]{.chpule2}[ ]{.chpuri2}

5.  コンテキストのメソッドを再度一つずつ検証し[、]{.chpule2}[
    ]{.chpuri2}空[ ]{.chpule1}[（]{.chpuri1}から[）]{.chpule2}[
    ]{.chpuri2}の状態条件文を状態オブジェクトに対する対応したメソッドの呼び出しと置き換えます[。]{.chpule2}[
    ]{.chpuri2}

6.  コンテキストの状態を切り替えるには[、]{.chpule2}[
    ]{.chpuri2}状態クラスのどれか一つのインスタンスを作成し[、]{.chpule2}[
    ]{.chpuri2}コンテキストに渡します[。]{.chpule2}[
    ]{.chpuri2}これは[、]{.chpule2}[
    ]{.chpuri2}コンテキスト自身の内部でも[、]{.chpule2}[
    ]{.chpuri2}種々の状態内部[、]{.chpule2}[
    ]{.chpuri2}あるいはクライアント内でもできます[。]{.chpule2}[
    ]{.chpuri2}どこで行うにせよ[、]{.chpule2}[
    ]{.chpuri2}クラスは[、]{.chpule2}[
    ]{.chpuri2}それがインスタンスを作成した具象状態クラスに依存するようになります[。]{.chpule2}[
    ]{.chpuri2}
:::

:::::: {.section .pros-cons}
##  長所と短所 {#pros-cons}

::::: row
::: col-sm-6
-   
    *[単]{.cjk}[一]{.cjk}[責]{.cjk}[任]{.cjk}[の]{.cjk}[原]{.cjk}[則]{.cjk}*[。]{.chpule2}[
    ]{.chpuri2}特定の状態にまつわるコードを別クラスにして整理[。]{.chpule2}[
    ]{.chpuri2}
-   
    *[開]{.cjk}[放]{.cjk}[閉]{.cjk}[鎖]{.cjk}[の]{.cjk}[原]{.cjk}[則]{.cjk}*[。]{.chpule2}[
    ]{.chpuri2}既存の状態クラスやコンテキストを変更せずに新しい状態を導入[。]{.chpule2}[
    ]{.chpuri2}
-    状態機械にある[、]{.chpule2}[
    ]{.chpuri2}かさばった条件分を削減し[、]{.chpule2}[
    ]{.chpuri2}コンテキストのコードを簡素化[。]{.chpule2}[ ]{.chpuri2}
:::

::: col-sm-6
-    状態機械に数個しか状態がない[、]{.chpule2}[
    ]{.chpuri2}またはめったに変更がない場合[、]{.chpule2}[
    ]{.chpuri2}このパターンの適用は過剰[。]{.chpule2}[ ]{.chpuri2}
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

[![State を C#
で](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](state/csharp/example.html "State を C# で"){.prog-lang-link}
[![State を C++
で](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](state/cpp/example.html "State を C++ で"){.prog-lang-link}
[![State を Go
で](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](state/go/example.html "State を Go で"){.prog-lang-link}
[![State を Java
で](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](state/java/example.html "State を Java で"){.prog-lang-link}
[![State を PHP
で](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](state/php/example.html "State を PHP で"){.prog-lang-link}
[![State を Python
で](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](state/python/example.html "State を Python で"){.prog-lang-link}
[![State を Ruby
で](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](state/ruby/example.html "State を Ruby で"){.prog-lang-link}
[![State を Rust
で](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](state/rust/example.html "State を Rust で"){.prog-lang-link}
[![State を Swift
で](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](state/swift/example.html "State を Swift で"){.prog-lang-link}
[![State を TypeScript
で](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](state/typescript/example.html "State を TypeScript で"){.prog-lang-link}
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

[Strategy []{.fa .fa-arrow-right}](strategy.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### 戻る

[[]{.fa .fa-arrow-left} Observer](observer.html){.btn .btn-default
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
