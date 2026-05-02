:::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} /
[デザインパターン](../design-patterns.html){.type} /
[振る舞いに関するパターン](behavioral-patterns.html){.type}
:::

# Visitor {#visitor .title}

::: aka
別名：[ビジター]{style="display:inline-block"}
:::

::: {.section .intent}
##  一言でいうと {#intent}

**Visitor**[ ]{.chpule1}[（]{.chpuri1}ビジター[、]{.chpule2}[
]{.chpuri2}訪問者[）]{.chpule2}[ ]{.chpuri2}は[、]{.chpule2}[
]{.chpuri2}振る舞いに関するデザインパターンの一つで[、]{.chpule2}[
]{.chpuri2}アルゴリズムをその動作対象となるオブジェクトから切り離します[。]{.chpule2}[
]{.chpuri2}

<figure class="image">
<img
src="../../images/patterns/content/visitor/visitor4a4c.png?id=f36d100188340db7a18854ef7916f972"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/visitor/visitor.png?id=f36d100188340db7a18854ef7916f972 640w,/images/patterns/content/visitor/visitor-1.5x.png?id=751e251363b632b20df856d72d02ee72 960w,/images/patterns/content/visitor/visitor-2x.png?id=2c5d9ab3046d782c19809d3b80650d65 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Visitor デザインパターン" />
</figure>
:::

::: {.section .problem}
##  問題 {#problem}

自分のチームが一つの見事なグラフとして構造化された地理情報を使うアプリを開発していることを想像してみてください[。]{.chpule2}[
]{.chpuri2}グラフの各ノードの表現するものは[、]{.chpule2}[
]{.chpuri2}市のような複雑なものかもしれませんし[、]{.chpule2}[
]{.chpuri2}もっと粒度の細かい[、]{.chpule2}[
]{.chpuri2}産業[、]{.chpule2}[
]{.chpuri2}観光地などかもしれません[。]{.chpule2}[
]{.chpuri2}ノードが表現する実在のものとの間に道路があれば[、]{.chpule2}[
]{.chpuri2}ノードは繋がります[。]{.chpule2}[
]{.chpuri2}内部的には[、]{.chpule2}[
]{.chpuri2}ノードの種類ごとに[、]{.chpule2}[
]{.chpuri2}それ専用のクラスがあり[、]{.chpule2}[
]{.chpuri2}各ノードはオブジェクトです[。]{.chpule2}[ ]{.chpuri2}

<figure class="image">
<img
src="../../images/patterns/diagrams/visitor/problem14338.png?id=e7076532da1e936f3519c63270da8454"
style="aspect-ratio: 2.43;"
srcset="/images/patterns/diagrams/visitor/problem1.png?id=e7076532da1e936f3519c63270da8454 560w,/images/patterns/diagrams/visitor/problem1-1.5x.png?id=250216d3458c38e9855eda96bf71251b 840w,/images/patterns/diagrams/visitor/problem1-2x.png?id=2e5d5143ac55af218754f28761bec17e 1120w"
sizes="(max-width: 720px) 100vw, 560px" loading="lazy" width="560"
alt="グラフの XML へのエクスポート" />
<figcaption><p>グラフの XML へのエクスポート<span
class="chpule2">。</span><span class="chpuri2"> </span></p></figcaption>
</figure>

ある時点で[、]{.chpule2}[ ]{.chpuri2}グラフの XML
形式でのエクスポートの実装を任されました[。]{.chpule2}[
]{.chpuri2}一見して簡単そうな仕事です[。]{.chpule2}[
]{.chpuri2}それぞれのノード・クラスにエクスポート用メソッドを追加し[、]{.chpule2}[
]{.chpuri2}再帰を利用してグラフの各ノードを巡りながら[、]{.chpule2}[
]{.chpuri2}エクスポートのメソッドを実行するだけです[。]{.chpule2}[
]{.chpuri2}この解決策は[、]{.chpule2}[
]{.chpuri2}シンプルでエレガントです[。]{.chpule2}[
]{.chpuri2}多相性のおかげで[、]{.chpule2}[
]{.chpuri2}エクスポート・メソッドを呼ぶコードは[、]{.chpule2}[
]{.chpuri2}ノードの具象クラスと結合していません[。]{.chpule2}[
]{.chpuri2}

不運なことに[、]{.chpule2}[
]{.chpuri2}システム・アーキテクトが既存のノード・クラスの変更の許可を拒否しました[。]{.chpule2}[
]{.chpuri2}彼が言うには[、]{.chpule2}[
]{.chpuri2}コードはすでに本番稼働中なので[、]{.chpule2}[
]{.chpuri2}あなたの変更の中にあるかもしれない不具合によって障害が起きるリスクを負いたくないそうです[。]{.chpule2}[
]{.chpuri2}

<figure class="image">
<img
src="../../images/patterns/diagrams/visitor/problem2-ja5b1a.png?id=84d343e913690a1ceb4d6e79080f8bf9"
style="aspect-ratio: 1.92;"
srcset="/images/patterns/diagrams/visitor/problem2-ja.png?id=84d343e913690a1ceb4d6e79080f8bf9 500w,/images/patterns/diagrams/visitor/problem2-ja-1.5x.png?id=0faa73a0c34033b24d231cfb0aec6aad 750w,/images/patterns/diagrams/visitor/problem2-ja-2x.png?id=d9d0190e756c245b682056d2f36099ce 1000w"
sizes="(max-width: 720px) 100vw, 500px" loading="lazy" width="500"
alt="XML エクスポートのメソッドをすべてのノード・クラスに追加する必要があった" />
<figcaption><p>XML エクスポートのメソッドをすべてのノード・クラスに追加する必要があった<span
class="chpule2">。</span><span class="chpuri2">
</span>変更中に不具合がまぎれこむことによるアプリケーション全体の機能停止のリスクあり<span
class="chpule2">。</span><span class="chpuri2"> </span></p></figcaption>
</figure>

それに加えて[、]{.chpule2}[ ]{.chpuri2}XML
エクスポートのコードをノード・クラスに入れるのは[、]{.chpule2}[
]{.chpuri2}理にかなっていることかどうか[、]{.chpule2}[
]{.chpuri2}彼には疑問です[。]{.chpule2}[
]{.chpuri2}これらのクラスの主要任務は[、]{.chpule2}[
]{.chpuri2}地理情報データと機能することです[。]{.chpule2}[
]{.chpuri2}XML エクスポートは[、]{.chpule2}[
]{.chpuri2}そこにはなじみません[。]{.chpule2}[ ]{.chpuri2}

拒否の理由がもう一つありました[。]{.chpule2}[
]{.chpuri2}この機能が実装された後になって[、]{.chpule2}[
]{.chpuri2}マーケティングの部署から[、]{.chpule2}[
]{.chpuri2}他の形式でのエクスポートを追加しろとか[、]{.chpule2}[
]{.chpuri2}他のわけのわからない要求が上がって来る可能性が大です[。]{.chpule2}[
]{.chpuri2}これにより[、]{.chpule2}[
]{.chpuri2}繊細で壊れやすいクラスを再度変更することを強いられることになるでしょう[。]{.chpule2}[
]{.chpuri2}
:::

::: {.section .solution}
##  解決策 {#solution}

Visitor パターンでは[、]{.chpule2}[
]{.chpuri2}新規の振る舞いを既存のクラスに統合するのではなく[、]{.chpule2}[
]{.chpuri2}*[ビ]{.cjk}[ジ]{.cjk}[タ]{.cjk}[ー]{.cjk}*と呼ばれる個別のクラスに置きます[。]{.chpule2}[
]{.chpuri2}その振る舞いを実行したい元のオブジェクトはビジターの引数として渡されます[。]{.chpule2}[
]{.chpuri2}オブジェクトに含まれる必要なデータをそのメソッドがアクセスできることが前提です[。]{.chpule2}[
]{.chpuri2}

さて[、]{.chpule2}[ ]{.chpuri2}ここで[、]{.chpule2}[
]{.chpuri2}もし振る舞いが異なるクラスのオブジェクト上で実行可能だとしたらどうでしょうか[？]{.chpule2}[
]{.chpuri2}たとえば[、]{.chpule2}[ ]{.chpuri2}XML
エクスポートの例では[、]{.chpule2}[
]{.chpuri2}実際の実装はノード・クラスの間で多少異なると思われます[。]{.chpule2}[
]{.chpuri2}したがって[、]{.chpule2}[
]{.chpuri2}ビジター・クラスは[、]{.chpule2}[
]{.chpuri2}メソッドを一つだけではなく[、]{.chpule2}[
]{.chpuri2}いくつかのメソッドの組を定義するかもしれません[。]{.chpule2}[
]{.chpuri2}それぞれのメソッドは異なる型の引数を下記のように取ります[：]{.chpule2}[
]{.chpuri2}

<figure class="code">
<pre class="code" lang="pseudocode"><code>class ExportVisitor implements Visitor is
    method doForCity(City c) { …… }
    method doForIndustry(Industry f) { …… }
    method doForSightSeeing(SightSeeing ss) { …… }
    // ……</code></pre>
</figure>

しかし[、]{.chpule2}[
]{.chpuri2}これらのメソッドはいったいどう呼べばいいのでしょう[？]{.chpule2}[
]{.chpuri2}特にグラフ全体が対象の場合は[？]{.chpule2}[
]{.chpuri2}これらのメソッドは異なるシグネチャーを持っているので[、]{.chpule2}[
]{.chpuri2}多相性は利用できません[。]{.chpule2}[
]{.chpuri2}与えられたオブジェクトのプロセスが可能な[、]{.chpule2}[
]{.chpuri2}適切なビジター・メソッドを選ぶためには[、]{.chpule2}[
]{.chpuri2}そのクラスをチェックする必要があります[。]{.chpule2}[
]{.chpuri2}まるで悪夢のように聞こえますね[？]{.chpule2}[ ]{.chpuri2}

<figure class="code">
<pre class="code" lang="pseudocode"><code>foreach (Node node in graph)
    if (node instanceof City)
        exportVisitor.doForCity((City) node)
    if (node instanceof Industry)
        exportVisitor.doForIndustry((Industry) node)
    // ……
}</code></pre>
</figure>

メソッドの多重定義を使えばいいでしょ[？]{.chpule2}[
]{.chpuri2}と皆さんは思うかもしれません[。]{.chpule2}[
]{.chpuri2}つまり[、]{.chpule2}[
]{.chpuri2}パラメーターの組み合わせが異うメソッドに同じ名前をつけるわけです[。]{.chpule2}[
]{.chpuri2}残念ながら[、]{.chpule2}[
]{.chpuri2}仮にプログラミング言語がそれをサポートしていたとしても[
]{.chpule1}[（]{.chpuri1}Java や C#
はサポートしています[）]{.ls-05}[、]{.chpule2}[
]{.chpuri2}我々の助けにはなりません[。]{.chpule2}[
]{.chpuri2}ノード・オブジェクトの正確なクラスが前もってわからないので[、]{.chpule2}[
]{.chpuri2}多重定義の仕組みを使って[、]{.chpule2}[
]{.chpuri2}実行すべき正しいメソッドを判定することができないからです[。]{.chpule2}[
]{.chpuri2}基底の `Node`
クラスのオブジェクトを取るメソッドが使われることになってしまいます[。]{.chpule2}[
]{.chpuri2}

でも[、]{.chpule2}[ ]{.chpuri2}Visitor パターンは[、]{.chpule2}[
]{.chpuri2}この問題に次のように対処します[。]{.chpule2}[
]{.chpuri2}[二重ディスパッチ](visitor-double-dispatch.html)という技を使用することにより[、]{.chpule2}[
]{.chpuri2}入り組んだ条件文なしで[、]{.chpule2}[
]{.chpuri2}オブジェクトに適切なメソッドを呼べるようにします[。]{.chpule2}[
]{.chpuri2}クライアントに適切な版のメソッドを選ばせる代わりに[、]{.chpule2}[
]{.chpuri2}その任務を[、]{.chpule2}[
]{.chpuri2}ビジターに引数として渡されるオジェクトに任せてはどうでしょう[？]{.chpule2}[
]{.chpuri2}オブジェクトは自分のクラスのことは分かっていますから[、]{.chpule2}[
]{.chpuri2}ビジターの適切なメソッドを選ぶことは[、]{.chpule2}[
]{.chpuri2}難なくできます[。]{.chpule2}[
]{.chpuri2}オブジェクトは[、]{.chpule2}[ ]{.chpuri2}ビジターを[
]{.chpule1}[「]{.chpuri1}受け入れ[[」]{.ls-1}]{.chpule2}[ ]{.chpuri2}​[
]{.chpule1}[（]{.chpuri1}accept[）]{.ls-05}[、]{.chpule2}[
]{.chpuri2}どの訪問メソッドが実行されるべきかをビジターに伝えます[。]{.chpule2}[
]{.chpuri2}

<figure class="code">
<pre class="code" lang="pseudocode"><code>// クライアントのコード
foreach (Node node in graph)
    node.accept(exportVisitor)

// 市
class City is
    method accept(Visitor v) is
        v.doForCity(this)
    // ……

// 産業
class Industry is
    method accept(Visitor v) is
        v.doForIndustry(this)
    // ……</code></pre>
</figure>

白状します[。]{.chpule2}[
]{.chpuri2}結局のところノード・クラスの変更は必要でした[。]{.chpule2}[
]{.chpuri2}しかし[、]{.chpule2}[
]{.chpuri2}変更は些細なもので[、]{.chpule2}[
]{.chpuri2}再度のコード変更なしに[、]{.chpule2}[
]{.chpuri2}更なる振る舞いの追加が可能となります[。]{.chpule2}[
]{.chpuri2}

さて[、]{.chpule2}[
]{.chpuri2}全部のビジターにとっての共通インターフェースを抽出さえすれば[、]{.chpule2}[
]{.chpuri2}既存の全ノードは[、]{.chpule2}[
]{.chpuri2}アプリに導入されるいかなるビジターとでも対応できるようになります[。]{.chpule2}[
]{.chpuri2}ノードに関する新規の振る舞いを導入する立場になった場合[、]{.chpule2}[
]{.chpuri2}やるべきことは新規のビジター・クラスの実装だけです[。]{.chpule2}[
]{.chpuri2}
:::

::: {.section .analogy}
##  現実世界でのたとえ {#analogy}

<figure class="image">
<img
src="../../images/patterns/content/visitor/visitor-comic-17f02.png?id=7ee4fa8800f7c4df4e1aa3b1aca2b7f1"
style="aspect-ratio: 2;"
srcset="/images/patterns/content/visitor/visitor-comic-1.png?id=7ee4fa8800f7c4df4e1aa3b1aca2b7f1 600w,/images/patterns/content/visitor/visitor-comic-1-1.5x.png?id=c9cadd73d25cc63fce94312f3c82bfee 900w,/images/patterns/content/visitor/visitor-comic-1-2x.png?id=439032451eb49ebbcb5257f25ecee790 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="保険代理店の営業担当者" />
<figcaption><p>優れた保険の外交員は<span class="chpule2">、</span><span
class="chpuri2">
</span>様々な種類の組織に異なるポリシーを提供する用意がいつでもできている<span
class="chpule2">。</span><span class="chpuri2"> </span></p></figcaption>
</figure>

新規顧客を獲得したいと思っている経験豊富な保険の外交員を想像してみてください[。]{.chpule2}[
]{.chpuri2}彼は近所のすべてのビルを訪問し[、]{.chpule2}[
]{.chpuri2}会う人ごとに保険を販売しようとします[。]{.chpule2}[
]{.chpuri2}ビルに入っている組織の種類に応じて[、]{.chpule2}[
]{.chpuri2}それに特化した保険を提供します[：]{.chpule2}[ ]{.chpuri2}

-   住宅ならば[、]{.chpule2}[
    ]{.chpuri2}医療保険を販売します[。]{.chpule2}[ ]{.chpuri2}
-   銀行ならば[、]{.chpule2}[
    ]{.chpuri2}盗難保険を販売します[。]{.chpule2}[ ]{.chpuri2}
-   喫茶店ならば[、]{.chpule2}[
    ]{.chpuri2}火災・洪水保険を販売します[。]{.chpule2}[ ]{.chpuri2}
:::

::::: {.section .structure-container}
##  構造 {#structure}

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/visitor/structure-ja2daf.png?id=a69ebd1946bf8fd8caf35c9e2a5cc653"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 0.96;"
srcset="/images/patterns/diagrams/visitor/structure-ja.png?id=a69ebd1946bf8fd8caf35c9e2a5cc653 520w,/images/patterns/diagrams/visitor/structure-ja-1.5x.png?id=34672dfc5b4d15d22dac7d788aab7c5c 780w,/images/patterns/diagrams/visitor/structure-ja-2x.png?id=a42447d1c9cd889dc9d669e9af5e851f 1040w"
sizes="(max-width: 720px) 100vw, 520px" loading="lazy" width="520"
alt="Visitor デザインパターンの構造" /><img
src="../../images/patterns/diagrams/visitor/structure-ja-indexed58f4.png?id=56c375d72794b149b3fa61587aeea6eb"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 0.93;"
srcset="/images/patterns/diagrams/visitor/structure-ja-indexed.png?id=56c375d72794b149b3fa61587aeea6eb 520w,/images/patterns/diagrams/visitor/structure-ja-indexed-1.5x.png?id=15cc41db0e46ff2be67e58038348d038 780w,/images/patterns/diagrams/visitor/structure-ja-indexed-2x.png?id=15079a2fe4dedfcb329db0dfe7e8e1f8 1040w"
sizes="(max-width: 720px) 100vw, 520px" loading="lazy" width="520"
alt="Visitor デザインパターンの構造" />
</figure>
:::

1.  **ビジター**[ ]{.chpule1}[（]{.chpuri1}Visitor[）]{.chpule2}[
    ]{.chpuri2}インターフェースは[、]{.chpule2}[
    ]{.chpuri2}オブジェクト構造の具象要素を引数として取る一連の訪問メソッドを宣言します[。]{.chpule2}[
    ]{.chpuri2}多重定義をサポートする言語でプログラムが書かれている場合は[、]{.chpule2}[
    ]{.chpuri2}これらのメソッドの名前は同じでかまいません[。]{.chpule2}[
    ]{.chpuri2}しかしパラメータの種類は違わなければなりません[。]{.chpule2}[
    ]{.chpuri2}

2.  それぞれの**具象ビジター**[ ]{.chpule1}[（]{.chpuri1}Concrete
    Visitor[）]{.chpule2}[ ]{.chpuri2}は[、]{.chpule2}[
    ]{.chpuri2}異なる具体的要素クラスに合わせて調整され[、]{.chpule2}[
    ]{.chpuri2}同じ動作をするいくつかのバージョンを実装しています[。]{.chpule2}[
    ]{.chpuri2}

3.  **要素**[ ]{.chpule1}[（]{.chpuri1}Element[）]{.chpule2}[
    ]{.chpuri2}インターフェースは[、]{.chpule2}[
    ]{.chpuri2}ビジターを受け入れる[
    ]{.chpule1}[（]{.chpuri1}accept[）]{.chpule2}[
    ]{.chpuri2}ためのメソッドを宣言します[。]{.chpule2}[
    ]{.chpuri2}このメソッドは[、]{.chpule2}[
    ]{.chpuri2}ビジター・インターフェースの型で宣言されたパラメータ一つを取る必要があります[。]{.chpule2}[
    ]{.chpuri2}

4.  各**具象要素**[ ]{.chpule1}[（]{.chpuri1}Concrete
    Element[）]{.chpule2}[ ]{.chpuri2}は[、]{.chpule2}[
    ]{.chpuri2}ビジターを受け入れるためのメソッドを実装する必要があります[。]{.chpule2}[
    ]{.chpuri2}このメソッドの目的は[、]{.chpule2}[
    ]{.chpuri2}現在の要素クラスに対応する適切なビジター・メソッドを呼び出すようにすることです[。]{.chpule2}[
    ]{.chpuri2}仮に基底要素クラスがこのメソッドを実装していても[、]{.chpule2}[
    ]{.chpuri2}すべてのサブクラスは[、]{.chpule2}[
    ]{.chpuri2}このメソッドを上書きし[、]{.chpule2}[
    ]{.chpuri2}ビジター・オブジェクト上の適切なメソッドを呼び出す必要があります[。]{.chpule2}[
    ]{.chpuri2}

5.  **クライアント**[ ]{.chpule1}[（]{.chpuri1}Client[）]{.chpule2}[
    ]{.chpuri2}は通常は[、]{.chpule2}[
    ]{.chpuri2}コレクションまたはその他の複雑なオブジェクト[
    ]{.chpule1}[（]{.chpuri1}たとえば[、]{.chpule2}[
    ]{.chpuri2}[コンポジット](composite.html)ツリー[）]{.chpule2}[
    ]{.chpuri2}を表します[。]{.chpule2}[ ]{.chpuri2}通常[、]{.chpule2}[
    ]{.chpuri2}クライアントは具象要素クラスをすべて認識していません[。]{.chpule2}[
    ]{.chpuri2}そのコレクションのオブジェクトと抽象的なインターフェースを通してやりとりするからです[。]{.chpule2}[
    ]{.chpuri2}
::::
:::::

::: {.section .pseudocode}
##  擬似コード {#pseudocode}

この例では[、]{.chpule2}[ ]{.chpuri2}**Visitor**
パターンにより[、]{.chpule2}[ ]{.chpuri2}幾何学的形状のクラス階層に XML
エクスポートのサポートを追加します[。]{.chpule2}[ ]{.chpuri2}

<figure class="image">
<img
src="../../images/patterns/diagrams/visitor/example9405.png?id=d66acd1b9096c47db17ab3bb82b54a59"
style="aspect-ratio: 1.14;"
srcset="/images/patterns/diagrams/visitor/example.png?id=d66acd1b9096c47db17ab3bb82b54a59 560w,/images/patterns/diagrams/visitor/example-1.5x.png?id=534425a20f1128cb81f418cbee30cba8 840w,/images/patterns/diagrams/visitor/example-2x.png?id=f44438b98f13fcb50898baefad67ffff 1120w"
sizes="(max-width: 720px) 100vw, 560px" loading="lazy" width="560"
alt="Visitor パターン例の構造" />
<figcaption><p>ビジター・オブジェクトを介して様々な種類のオブジェクトを
XML 形式でエクスポート<span class="chpule2">。</span><span
class="chpuri2"> </span></p></figcaption>
</figure>

<figure class="code">
<pre class="code" lang="pseudocode"><code>// 要素のインターフェースは、accept メソッドを宣言。これは、基底ビジター・
// インターフェースを引数として取る。
interface Shape is
    method move(x, y)
    method draw()
    method accept(v: Visitor)

// 各具象要素クラスは、accept メソッドを実装しなければならない。実装は、
// 要素のクラスに対応したビジターのメソッドを呼び出すようなものでなければ
// ならない。
class Dot implements Shape is
    // ……

    // ここでは現クラスを受け入れる visitDot を呼び出していることに注意。
    // このようにしてビジターに、動作の対象となる要素のクラスを知らせる。
    method accept(v: Visitor) is
        v.visitDot(this)

class Circle implements Shape is
    // ……
    method accept(v: Visitor) is
        v.visitCircle(this)

class Rectangle implements Shape is
    // ……
    method accept(v: Visitor) is
        v.visitRectangle(this)

class CompoundShape implements Shape is
    // ……
    method accept(v: Visitor) is
        v.visitCompoundShape(this)


// Visitor インターフェースは、要素のクラスに応じた訪問メソッドの集まりを
// 宣言。訪問メソッドのシグネチャーから、ビジターは処理対象の要素の正確な
// クラスを特定可能。
interface Visitor is
    method visitDot(d: Dot)
    method visitCircle(c: Circle)
    method visitRectangle(r: Rectangle)
    method visitCompoundShape(cs: CompoundShape)

// ビジターの具象クラスは、同じアルゴリズムのいくつか異なるバージョンを実
// 装し、全具象要素クラスと動作可能。
//
// Composite ツリー等、複雑なオブジェクト構造に対して用いると、Visitor
// パターンの利点を最大限に活用可能。この場合、構造中の種々のオブジェクト
// に対してビジター・メソッドの実行中に、アルゴリズムの中間状態を保存する
// ことが役に立つ場合あり。
class XMLExportVisitor implements Visitor is
    method visitDot(d: Dot) is
        // 点の ID と中心座標をエクスポート。

    method visitCircle(c: Circle) is
        // 円の ID と中心座標と半径をエクスポート。

    method visitRectangle(r: Rectangle) is
        // 長方形の ID と左上の座標と幅と高さをエクスポート。

    method visitCompoundShape(cs: CompoundShape) is
        // 形状の ID と子の ID のリストをエクスポート。


// クライアント・コードは、具象クラスを把握することなく、一連の要素に対し
// てビジター操作を実行可能。accept 操作は、ビジター・オブジェクト内の適
// 切な操作への呼び出しとなる。
class Application is
    field allShapes: array of Shapes

    method export() is
        exportVisitor = new XMLExportVisitor()

        foreach (shape in allShapes) do
            shape.accept(exportVisitor)</code></pre>
</figure>

もし[、]{.chpule2}[ ]{.chpuri2}この例でなぜ `accept`
メソッドが必要なんだろうと思われた方[。]{.chpule2}[
]{.chpuri2}私の記事[ビジターと二重ディスパッチ](visitor-double-dispatch.html)
がその疑問に答えます[。]{.chpule2}[ ]{.chpuri2}
:::

:::::::::: {.section .applicability-container}
##  適応性 {#applicability}

::::::::: applicability
::: applicability-problem
複雑なオブジェクト構造[ ]{.chpule1}[（]{.chpuri1}例[：]{.chpule2}[
]{.chpuri2}オブジェクト・ツリー[）]{.chpule2}[
]{.chpuri2}のすべての要素に対してある操作を実行する必要がある場合は[、]{.chpule2}[
]{.chpuri2}Visitor を使用します[。]{.chpule2}[ ]{.chpuri2}
:::

::: applicability-solution
Visitor パターンを使用すると[、]{.chpule2}[
]{.chpuri2}一つの操作を異なるクラスのオブジェクトに対して実行することができます[。]{.chpule2}[
]{.chpuri2}これは[、]{.chpule2}[
]{.chpuri2}ビジター・オブジェクトが[、]{.chpule2}[
]{.chpuri2}対象クラスに応じてやや異なる同一操作の実装をすることでかなえられます[。]{.chpule2}[
]{.chpuri2}
:::

::: applicability-problem
補助的な振る舞いのビジネス・ロジックを整理するために Visitor
を使用します[。]{.chpule2}[ ]{.chpuri2}
:::

::: applicability-solution
このパターンは[、]{.chpule2}[
]{.chpuri2}アプリの主要クラスがその主要任務に集中できるようにします[。]{.chpule2}[
]{.chpuri2}これは[、]{.chpule2}[
]{.chpuri2}他の付帯的な振る舞いをビジター・クラスに抽出することにより行います[。]{.chpule2}[
]{.chpuri2}
:::

::: applicability-problem
ある振る舞いが[、]{.chpule2}[
]{.chpuri2}クラス階層の中のあるクラスでは意味があるが[、]{.chpule2}[
]{.chpuri2}他のクラスでは意味がない場合に[、]{.chpule2}[
]{.chpuri2}このクラスを使用します[。]{.chpule2}[ ]{.chpuri2}
:::

::: applicability-solution
この振る舞いを別個のビジター・クラスに抽出し[、]{.chpule2}[
]{.chpuri2}訪問メソッドのうち関連クラスのオブジェクトを受け入れるものだけ実装し[、]{.chpule2}[
]{.chpuri2}他は空のままにします[。]{.chpule2}[ ]{.chpuri2}
:::
:::::::::
::::::::::

::: {.section .checklist}
##  実装方法 {#checklist}

1.  プログラム中の各具象要素クラスごとに[[、]{.ls-1}]{.chpule2}[
    ]{.chpuri2}​[ ]{.chpule1}[「]{.chpuri1}訪問[」]{.chpule2}[
    ]{.chpuri2}メソッドの組からなるビジター・インターフェースを宣言します[。]{.chpule2}[
    ]{.chpuri2}

2.  要素インターフェースを宣言します[。]{.chpule2}[
    ]{.chpuri2}もし既存の要素クラスの階層があるのであれば[、]{.chpule2}[
    ]{.chpuri2}抽象[
    ]{.chpule1}[「]{.chpuri1}受け入れ[[」]{.ls-1}]{.chpule2}[
    ]{.chpuri2}​[ ]{.chpule1}[（]{.chpuri1}accept[）]{.chpule2}[
    ]{.chpuri2}メソッドを階層の基底クラスに追加します[。]{.chpule2}[
    ]{.chpuri2}このメソッドは[、]{.chpule2}[
    ]{.chpuri2}ビジター・オブジェクトを引数として取ります[。]{.chpule2}[
    ]{.chpuri2}

3.  すべての具象要素クラスで[、]{.chpule2}[
    ]{.chpuri2}受け入れメソッド(複数あるかもしれません[）]{.chpule2}[
    ]{.chpuri2}を実装します[。]{.chpule2}[
    ]{.chpuri2}これらのメソッドは[、]{.chpule2}[
    ]{.chpuri2}単に呼び出しを[、]{.chpule2}[
    ]{.chpuri2}現要素のクラスと一致する[、]{.chpule2}[
    ]{.chpuri2}渡された訪問オブジェクトの訪問メソッドの呼び出しに換えます[。]{.chpule2}[
    ]{.chpuri2}

4.  要素クラスは[、]{.chpule2}[
    ]{.chpuri2}ビジター・インターフェースを通してのみ[、]{.chpule2}[
    ]{.chpuri2}ビジターと関わるべきです[。]{.chpule2}[
    ]{.chpuri2}ビジターは[、]{.chpule2}[ ]{.chpuri2}逆に[、]{.chpule2}[
    ]{.chpuri2}訪問メソッドのパラメーター型として参照されるすべての具象要素クラスを知っておく必要があります[。]{.chpule2}[
    ]{.chpuri2}

5.  要素の階層の中で実装できない振る舞いの一つごとに[、]{.chpule2}[
    ]{.chpuri2}新しい具象ビジター・クラスを作成し[、]{.chpule2}[
    ]{.chpuri2}訪問メソッドを全部実装します[。]{.chpule2}[ ]{.chpuri2}

    ビジターが要素クラスの非公開メンバーにアクセスする必要がある状況に遭遇するかもしれません[。]{.chpule2}[
    ]{.chpuri2}この場合[、]{.chpule2}[
    ]{.chpuri2}要素のカプセル化の目標に反してこれらのフィールドやメソッドを公開するか[、]{.chpule2}[
    ]{.chpuri2}ビジター・クラスを要素クラス中にネストすることができます[。]{.chpule2}[
    ]{.chpuri2}後者は[、]{.chpule2}[
    ]{.chpuri2}幸運にも使用しているプログラミング言語がクラスのネストをサポートする場合にのみ可能です[。]{.chpule2}[
    ]{.chpuri2}

6.  クライアントはビジター・オブジェクトを複数作成し[、]{.chpule2}[
    ]{.chpuri2}それらを要素の[
    ]{.chpule1}[「]{.chpuri1}受け入れ[」]{.chpule2}[
    ]{.chpuri2}メソッドに渡します[。]{.chpule2}[ ]{.chpuri2}
:::

:::::: {.section .pros-cons}
##  長所と短所 {#pros-cons}

::::: row
::: col-sm-6
-   
    *[開]{.cjk}[放]{.cjk}[閉]{.cjk}[鎖]{.cjk}[の]{.cjk}[原]{.cjk}[則]{.cjk}*[。]{.chpule2}[
    ]{.chpuri2}異なるクラスのオブジェクトと動作可能な新規の振る舞いを[、]{.chpule2}[
    ]{.chpuri2}これらのクラスの変更なしに導入可能[。]{.chpule2}[
    ]{.chpuri2}
-   
    *[単]{.cjk}[一]{.cjk}[責]{.cjk}[任]{.cjk}[の]{.cjk}[原]{.cjk}[則]{.cjk}*[。]{.chpule2}[
    ]{.chpuri2}同じ振る舞いの複数のバージョンを同一クラスに移動可能[。]{.chpule2}[
    ]{.chpuri2}
-    ビジター・オブジェクトは[、]{.chpule2}[
    ]{.chpuri2}様々なオブジェクトと動作しながら[、]{.chpule2}[
    ]{.chpuri2}有用な情報を蓄積可能[。]{.chpule2}[
    ]{.chpuri2}これはオブジェクト・ツリーのような複雑なオブジェクト構造を探索し[、]{.chpule2}[
    ]{.chpuri2}この構造の各オブジェクトにビジターを適用したい場合に便利[。]{.chpule2}[
    ]{.chpuri2}
:::

::: col-sm-6
-    要素階層にクラスを追加または削除するたびに[、]{.chpule2}[
    ]{.chpuri2}すべてのビジターを更新する必要あり[。]{.chpule2}[
    ]{.chpuri2}
-    ビジターは[、]{.chpule2}[
    ]{.chpuri2}作業する対象要素の非公開フィールドとメソッドへのアクセスができない可能性あり[。]{.chpule2}[
    ]{.chpuri2}
:::
:::::
::::::

::: {.section .relations}
##  他のパターンとの関係 {#relations}

-   [Visitor](visitor.html) を [Command](command.html)
    パターンのより強力なものとして扱うことができます[。]{.chpule2}[
    ]{.chpuri2}そのオブジェクトは[、]{.chpule2}[
    ]{.chpuri2}異なるクラスの様々なオブジェクトに対して操作を実行できます[。]{.chpule2}[
    ]{.chpuri2}

-   [Visitor](visitor.html) を使用して[、]{.chpule2}[
    ]{.chpuri2}[Composite](composite.html)
    ツリー全体に対して一つの操作を実行できます[。]{.chpule2}[
    ]{.chpuri2}

-   複雑なデータ構造を探索し[、]{.chpule2}[
    ]{.chpuri2}その要素に対してある操作を実行するために[、]{.chpule2}[
    ]{.chpuri2}[Visitor](visitor.html) を [Iterator](iterator.html)
    と一緒に使用することができます[。]{.chpule2}[
    ]{.chpuri2}要素のクラスが全部異なっていてもかまいません[。]{.chpule2}[
    ]{.chpuri2}
:::

::: {.section .implementations}
##  コード例 {#implementations}

[![Visitor を C#
で](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](visitor/csharp/example.html "Visitor を C# で"){.prog-lang-link}
[![Visitor を C++
で](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](visitor/cpp/example.html "Visitor を C++ で"){.prog-lang-link}
[![Visitor を Go
で](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](visitor/go/example.html "Visitor を Go で"){.prog-lang-link}
[![Visitor を Java
で](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](visitor/java/example.html "Visitor を Java で"){.prog-lang-link}
[![Visitor を PHP
で](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](visitor/php/example.html "Visitor を PHP で"){.prog-lang-link}
[![Visitor を Python
で](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](visitor/python/example.html "Visitor を Python で"){.prog-lang-link}
[![Visitor を Ruby
で](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](visitor/ruby/example.html "Visitor を Ruby で"){.prog-lang-link}
[![Visitor を Rust
で](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](visitor/rust/example.html "Visitor を Rust で"){.prog-lang-link}
[![Visitor を Swift
で](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](visitor/swift/example.html "Visitor を Swift で"){.prog-lang-link}
[![Visitor を TypeScript
で](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](visitor/typescript/example.html "Visitor を TypeScript で"){.prog-lang-link}
:::

::: {.section .extras}
##  追記 {#extras}

-   どうして Visitor
    パターンを単純にメソッド多重定義で置き換えられないのだろうと[、]{.chpule2}[
    ]{.chpuri2}困惑していますか[？]{.chpule2}[
    ]{.chpuri2}厄介な詳細については[、]{.chpule2}[
    ]{.chpuri2}私の記事[、]{.chpule2}[
    ]{.chpuri2}[ビジターと二重ディスパッチ](visitor-double-dispatch.html)をご一読ください[。]{.chpule2}[
    ]{.chpuri2}
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

[ビジターと二重ディスパッチ []{.fa
.fa-arrow-right}](visitor-double-dispatch.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### 戻る

[[]{.fa .fa-arrow-left} Template Method](template-method.html){.btn
.btn-default rel="prev"}
:::
:::::::::::::::::::::::::::::::::::

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
:::::::::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::::::::
