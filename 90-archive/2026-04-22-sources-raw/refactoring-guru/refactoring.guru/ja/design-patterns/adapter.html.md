:::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} /
[デザインパターン](../design-patterns.html){.type} /
[構造に関するパターン](structural-patterns.html){.type}
:::

# Adapter {#adapter .title}

::: aka
別名：[Wrapper、]{style="display:inline-block"}[アダプター、]{style="display:inline-block"}[ラッパー]{style="display:inline-block"}
:::

::: {.section .intent}
##  一言でいうと {#intent}

**Adapter**[ ]{.chpule1}[（]{.chpuri1}アダプター[、]{.chpule2}[
]{.chpuri2}適合装置[）]{.chpule2}[ ]{.chpuri2}は[、]{.chpule2}[
]{.chpuri2}構造に関するデザインパターンの一つで[、]{.chpule2}[
]{.chpuri2}非互換なインターフェースのオブジェクト同士の協働を可能とします[。]{.chpule2}[
]{.chpuri2}

<figure class="image">
<img
src="../../images/patterns/content/adapter/adapter-ja76ea.png?id=1d4a040176811ac41a6ac9da77e49708"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/adapter/adapter-ja.png?id=1d4a040176811ac41a6ac9da77e49708 640w,/images/patterns/content/adapter/adapter-ja-1.5x.png?id=137c99696246d12152d60acdfa2e6426 960w,/images/patterns/content/adapter/adapter-ja-2x.png?id=1cb2f05a551b2df54aff55a54cc0eced 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Adapter デザインパターン" />
</figure>
:::

::: {.section .problem}
##  問題 {#problem}

株式市場監視アプリを作成することを想像してみてください[。]{.chpule2}[
]{.chpuri2}このアプリは[、]{.chpule2}[ ]{.chpuri2}複数の情報源から XML
形式の株式データをダウンロードし[、]{.chpule2}[
]{.chpuri2}見栄えの良いグラフや図表をユーザーに表示します[。]{.chpule2}[
]{.chpuri2}

ある時点で[、]{.chpule2}[
]{.chpuri2}外部提供の気の利いた分析ライブラリーを統合してアプリを改善することに決めました[。]{.chpule2}[
]{.chpuri2}ただし[、]{.chpule2}[
]{.chpuri2}一つ問題があります[。]{.chpule2}[
]{.chpuri2}分析ライブラリーは[、]{.chpule2}[ ]{.chpuri2}JSON
形式のデータでのみ機能するということです[。]{.chpule2}[ ]{.chpuri2}

<figure class="image">
<img
src="../../images/patterns/diagrams/adapter/problem-ja3792.png?id=6cea2062034f0e193956ccdbd52c7686"
style="aspect-ratio: 2.41;"
srcset="/images/patterns/diagrams/adapter/problem-ja.png?id=6cea2062034f0e193956ccdbd52c7686 530w,/images/patterns/diagrams/adapter/problem-ja-1.5x.png?id=fab53149b239665a85eb1b4eff131c24 795w,/images/patterns/diagrams/adapter/problem-ja-2x.png?id=1b3ce653b15dfe113008297b3f791db4 1060w"
sizes="(max-width: 720px) 100vw, 530px" loading="lazy" width="530"
alt="分析ライブラリーと統合前のアプリの構造" />
<figcaption><p>分析ライブラリーは<span class="chpule2">、</span><span
class="chpuri2">
</span>自分のアプリとは非互換な形式のデータしか受け付けないので<span
class="chpule2">、</span><span class="chpuri2">
</span>そのままでは使用できない<span class="chpule2">。</span><span
class="chpuri2"> </span></p></figcaption>
</figure>

XML で動作するようにライブラリーを変更することもできます[。]{.chpule2}[
]{.chpuri2}ただそうすると[、]{.chpule2}[
]{.chpuri2}ライブラリーに依存した既存のコードが動かなくなる可能性があります[。]{.chpule2}[
]{.chpuri2}さらに[、]{.chpule2}[
]{.chpuri2}そもそもライブラリーのソースコードが入手できない可能性があり[、]{.chpule2}[
]{.chpuri2}その場合この手法はうまくいきません[。]{.chpule2}[ ]{.chpuri2}
:::

::: {.section .solution}
##  解決策 {#solution}

*[ア]{.cjk}[ダ]{.cjk}[プ]{.cjk}[タ]{.cjk}[ー]{.cjk}*
を作成します[。]{.chpule2}[ ]{.chpuri2}これは[、]{.chpule2}[
]{.chpuri2}あるオブジェクトのインターフェースを他のオブジェクトが理解できるように変換する特殊なオブジェクトです[。]{.chpule2}[
]{.chpuri2}

アダプターは[、]{.chpule2}[
]{.chpuri2}オブジェクトをラップして[、]{.chpule2}[
]{.chpuri2}舞台裏で行われる変換の詳細を隠蔽します[。]{.chpule2}[
]{.chpuri2}ラップされたオブジェクトは[、]{.chpule2}[
]{.chpuri2}アダプター内で動作しているということすら認識していません[。]{.chpule2}[
]{.chpuri2}たとえば[、]{.chpule2}[
]{.chpuri2}メートルとキロメートルで動作するオブジェクトを[、]{.chpule2}[
]{.chpuri2}すべてのデータをフィートやマイルに変換するアダプターでラップすることができます[。]{.chpule2}[
]{.chpuri2}

アダプターは[、]{.chpule2}[
]{.chpuri2}データの形式変換だけだけではなく[、]{.chpule2}[
]{.chpuri2}異なるインターフェースのオブジェクト同士の協働を可能とします[。]{.chpule2}[
]{.chpuri2}仕組みは次のとおりです[。]{.chpule2}[ ]{.chpuri2}

1.  アダプターは[、]{.chpule2}[
    ]{.chpuri2}既存オブジェクトの一つと互換なインターフェースを実装します[。]{.chpule2}[
    ]{.chpuri2}
2.  このインターフェースを使用して[、]{.chpule2}[
    ]{.chpuri2}既存オブジェクトはアダプターのメソッドを安全に呼び出すことができます[。]{.chpule2}[
    ]{.chpuri2}
3.  呼び出されると[、]{.chpule2}[
    ]{.chpuri2}アダプターはリクエストを二つ目のオブジェクトに渡します[。]{.chpule2}[
    ]{.chpuri2}ただし[、]{.chpule2}[
    ]{.chpuri2}二つ目のオブジェクトが期待する形式で渡します[。]{.chpule2}[
    ]{.chpuri2}

時には[、]{.chpule2}[
]{.chpuri2}呼び出しを双方向に変換できる双方向アダプターを作成することも可能です[。]{.chpule2}[
]{.chpuri2}

<figure class="image">
<img
src="../../images/patterns/diagrams/adapter/solution-ja4dbc.png?id=1022dd0f6e8530da0f952e391c5fae75"
style="aspect-ratio: 1.51;"
srcset="/images/patterns/diagrams/adapter/solution-ja.png?id=1022dd0f6e8530da0f952e391c5fae75 530w,/images/patterns/diagrams/adapter/solution-ja-1.5x.png?id=82d7f2f96b25ccde3988dd81325e3b6a 795w,/images/patterns/diagrams/adapter/solution-ja-2x.png?id=82c25c19a012dc64874c06cf30fb6046 1060w"
sizes="(max-width: 720px) 100vw, 530px" loading="lazy" width="530"
alt="Adapter の解決策" />
</figure>

株式市場アプリの話に戻りましょう[。]{.chpule2}[
]{.chpuri2}形式非互換の問題を解決するために[、]{.chpule2}[
]{.chpuri2}コードが直接接触する分析ライブラリーのすべてのクラスに対して
XML から JSON へのアダプターを作成します[。]{.chpule2}[
]{.chpuri2}次に[、]{.chpule2}[
]{.chpuri2}これらのアダプターを介してのみライブラリーーと通信するようにコードを調整します[。]{.chpule2}[
]{.chpuri2}アダプターは呼び出しを受けると[、]{.chpule2}[ ]{.chpuri2}着信
XML データを JSON に変換し[、]{.chpule2}[
]{.chpuri2}ラップされた分析オブジェクトの適切なメソッドを呼び出します[。]{.chpule2}[
]{.chpuri2}
:::

::: {.section .analogy}
##  現実世界でのたとえ {#analogy}

<figure class="image">
<img
src="../../images/patterns/content/adapter/adapter-comic-1-jaf7b2.png?id=3a71208ff4ee01f4e4029cc84482f859"
style="aspect-ratio: 1.78;"
srcset="/images/patterns/content/adapter/adapter-comic-1-ja.png?id=3a71208ff4ee01f4e4029cc84482f859 533w,/images/patterns/content/adapter/adapter-comic-1-ja-1.5x.png?id=07dbbf683cfe5e3c5c96da66ef5ebb3a 800w,/images/patterns/content/adapter/adapter-comic-1-ja-2x.png?id=f2384cc3ffe1e893a581e81f3b817459 1067w"
sizes="(max-width: 720px) 100vw, 533px" loading="lazy" width="533"
alt="Adapter パターンの例" />
<figcaption><p>海外旅行前と後のスーツケース</p></figcaption>
</figure>

米国からヨーロッパに初めて旅行する時[、]{.chpule2}[
]{.chpuri2}ラップトップを充電しようとすると驚くかもしれません[。]{.chpule2}[
]{.chpuri2}電源プラグとソケットの規格は国によって異なるのです[。]{.chpule2}[
]{.chpuri2}そのため[、]{.chpule2}[
]{.chpuri2}米国のプラグはドイツのソケットに入りません[。]{.chpule2}[
]{.chpuri2}この問題は[、]{.chpule2}[
]{.chpuri2}アメリカ式ソケットとヨーロッパ式プラグを備えた電源プラグ・アダプターを使用することで解決できます[。]{.chpule2}[
]{.chpuri2}
:::

:::::::: {.section .structure-container}
##  構造 {#structure}

::::::: structure
#### オブジェクト・アダプター

この実装では[、]{.chpule2}[
]{.chpuri2}オブジェクトの合成原理を使用します[。]{.chpule2}[
]{.chpuri2}アダプターは一方のオブジェクトのインタフェースを実装し[、]{.chpule2}[
]{.chpuri2}もう一方のオブジェクトをラップします[。]{.chpule2}[
]{.chpuri2}これは[、]{.chpule2}[
]{.chpuri2}一般的に普及しているすべてのプログラミング言語で実装可能です[。]{.chpule2}[
]{.chpuri2}

:::: adapter-object
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/adapter/structure-object-adaptera83b.png?id=33dffbe3aece294162440c7ddd3d5d4f"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.81;"
srcset="/images/patterns/diagrams/adapter/structure-object-adapter.png?id=33dffbe3aece294162440c7ddd3d5d4f 580w,/images/patterns/diagrams/adapter/structure-object-adapter-1.5x.png?id=c1b8a87b74fc4ce5639212fe19ee06fe 870w,/images/patterns/diagrams/adapter/structure-object-adapter-2x.png?id=03e8052e168c962d6bc369bbb13b0945 1160w"
sizes="(max-width: 720px) 100vw, 580px" loading="lazy" width="580"
alt="Adapter デザインパターンの構造（オブジェクト・アダプター）" /><img
src="../../images/patterns/diagrams/adapter/structure-object-adapter-indexedd6ac.png?id=a20b311948b361a058097e5bcdbf067a"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.76;"
srcset="/images/patterns/diagrams/adapter/structure-object-adapter-indexed.png?id=a20b311948b361a058097e5bcdbf067a 600w,/images/patterns/diagrams/adapter/structure-object-adapter-indexed-1.5x.png?id=72a1c8fde064c4915379fb34a1a66881 900w,/images/patterns/diagrams/adapter/structure-object-adapter-indexed-2x.png?id=759771515f08d74d53cf4fe500f814a3 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Adapter デザインパターンの構造（オブジェクト・アダプター）" />
</figure>
:::
::::

1.  **クライアント**[ ]{.chpule1}[（]{.chpuri1}Client[）]{.chpule2}[
    ]{.chpuri2}は[、]{.chpule2}[
    ]{.chpuri2}既存のビジネス・ロジックを含んだクラスです[。]{.chpule2}[
    ]{.chpuri2}

2.  **クライアント・インターフェース**[ ]{.chpule1}[（]{.chpuri1}Client
    Interface[）]{.chpule2}[ ]{.chpuri2}は[、]{.chpule2}[
    ]{.chpuri2}クライアント側コードと協働するためには従わなければいけないプロトコル＝決まり事を記述しています[。]{.chpule2}[
    ]{.chpuri2}

3.  **サービス**[ ]{.chpule1}[（]{.chpuri1}Service[）]{.chpule2}[
    ]{.chpuri2}は[、]{.chpule2}[
    ]{.chpuri2}外部から提供されるか[、]{.chpule2}[
    ]{.chpuri2}昔から使われてきた[、]{.chpule2}[
    ]{.chpuri2}ある役に立つクラスです[。]{.chpule2}[
    ]{.chpuri2}このクラスは[、]{.chpule2}[
    ]{.chpuri2}非互換のインターフェースのため[、]{.chpule2}[
    ]{.chpuri2}クライアントから直接使うことができません[。]{.chpule2}[
    ]{.chpuri2}

4.  **アダプター**[ ]{.chpule1}[（]{.chpuri1}Adapter[）]{.chpule2}[
    ]{.chpuri2}は[、]{.chpule2}[
    ]{.chpuri2}クライアントとサービスの両方と機能できるコードです[：]{.chpule2}[
    ]{.chpuri2}クライアント・インターフェースを実装すると同時にサービス・オブジェクトをラップします[。]{.chpule2}[
    ]{.chpuri2}アダプターは[、]{.chpule2}[
    ]{.chpuri2}アダプターのインターフェースを介してクライアントから呼び出され[、]{.chpule2}[
    ]{.chpuri2}それをラップされたサービス・オブジェクトが理解できる形式に変換して呼び出します[。]{.chpule2}[
    ]{.chpuri2}

5.  クライアント側コードは[、]{.chpule2}[
    ]{.chpuri2}クライアント・インタフェースを介してアダプターとやりとりする限り[、]{.chpule2}[
    ]{.chpuri2}具象アダプター・クラスと結合されることはありません[。]{.chpule2}[
    ]{.chpuri2}このおかげで[、]{.chpule2}[
    ]{.chpuri2}新しい種類のアダプターをプログラムに導入しても既存のクライアント側コードは問題なく動作します[。]{.chpule2}[
    ]{.chpuri2}これは[、]{.chpule2}[
    ]{.chpuri2}サービス・クラスのインターフェースを変更したり置き換えたりする際に便利です[。]{.chpule2}[
    ]{.chpuri2}クライアント側コードを変更することなく[、]{.chpule2}[
    ]{.chpuri2}新しいアダプター・クラスを作成することができます[。]{.chpule2}[
    ]{.chpuri2}

#### クラス・アダプター

この実装では継承を利用します[。]{.chpule2}[
]{.chpuri2}アダプターは[、]{.chpule2}[
]{.chpuri2}両方のオブジェクトのインターフェースを同時に継承します[。]{.chpule2}[
]{.chpuri2}この方法は[、]{.chpule2}[ ]{.chpuri2}C++
のような多重継承をサポートするプログラミング言語でのみ実装可能であることにご注意ください[。]{.chpule2}[
]{.chpuri2}

:::: adapter-class
::: {.struct-image2 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/adapter/structure-class-adapterf043.png?id=e1c60240508146ed3b98ac562cc8e510"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.72;"
srcset="/images/patterns/diagrams/adapter/structure-class-adapter.png?id=e1c60240508146ed3b98ac562cc8e510 550w,/images/patterns/diagrams/adapter/structure-class-adapter-1.5x.png?id=299a79bdfa10ac53213ba02408255430 825w,/images/patterns/diagrams/adapter/structure-class-adapter-2x.png?id=ddca3e3e4d972b7c58207daba8d24866 1100w"
sizes="(max-width: 720px) 100vw, 550px" loading="lazy" width="550"
alt="Adapter デザインパターンの構造（クラス・アダプター）" /><img
src="../../images/patterns/diagrams/adapter/structure-class-adapter-indexedd30e.png?id=250b5c485a7dfba7c16b89a9201538fb"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.72;"
srcset="/images/patterns/diagrams/adapter/structure-class-adapter-indexed.png?id=250b5c485a7dfba7c16b89a9201538fb 550w,/images/patterns/diagrams/adapter/structure-class-adapter-indexed-1.5x.png?id=b56d736f8076d34b1896de0a2b22a219 825w,/images/patterns/diagrams/adapter/structure-class-adapter-indexed-2x.png?id=9ae1182ef2a34d2ea65f4b4f94a4019e 1100w"
sizes="(max-width: 720px) 100vw, 550px" loading="lazy" width="550"
alt="Adapter デザインパターンの構造（クラス・アダプター）" />
</figure>
:::
::::

1.  **クラス・アダプター** では[、]{.chpule2}[
    ]{.chpuri2}オブジェクトをラップする必要はありません[。]{.chpule2}[
    ]{.chpuri2}クライアントとサービスの両方から振る舞いを継承するからです[。]{.chpule2}[
    ]{.chpuri2}適合[ ]{.chpule1}[（]{.chpuri1}変換[）]{.chpule2}[
    ]{.chpuri2}は[、]{.chpule2}[
    ]{.chpuri2}上書きされたメソッドの間で行われます[。]{.chpule2}[
    ]{.chpuri2}完成したアダプターは[、]{.chpule2}[
    ]{.chpuri2}既存のクライアント側クラスの代わりに使うことができます[。]{.chpule2}[
    ]{.chpuri2}
:::::::
::::::::

::: {.section .pseudocode}
##  擬似コード {#pseudocode}

この **Adapter** パターンの使用例は[、]{.chpule2}[
]{.chpuri2}矛盾の典型である[
]{.chpule1}[「]{.chpuri1}丸い穴に四角いくい[[」]{.ls-1}]{.chpule2}[
]{.chpuri2}​[ ]{.chpule1}[（]{.chpuri1}訳注[：]{.chpule2}[
]{.chpuri2}合わないものを無理に一緒にすることを表現する英語の慣用句[）]{.chpule2}[
]{.chpuri2}に基づいています[。]{.chpule2}[ ]{.chpuri2}

<figure class="image">
<img
src="../../images/patterns/diagrams/adapter/examplef4c9.png?id=9d2b6857ce256f2c669383ce4df3d0aa"
style="aspect-ratio: 1.68;"
srcset="/images/patterns/diagrams/adapter/example.png?id=9d2b6857ce256f2c669383ce4df3d0aa 640w,/images/patterns/diagrams/adapter/example-1.5x.png?id=76e68b9cea3b371e465e81587e57e5d9 960w,/images/patterns/diagrams/adapter/example-2x.png?id=0ac62d1bc151e8ce6abad8e8502756cf 1280w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="640"
alt="Adapter パターン例の構造" />
<figcaption><p>四角いくいを丸い穴に適応中</p></figcaption>
</figure>

アダプターは[、]{.chpule2}[ ]{.chpuri2}半径が四角の対角線の半分の長さ[
]{.chpule1}[（]{.chpuri1}言い換えると[、]{.chpule2}[
]{.chpuri2}四角いくいを含むことのできる最小の円の半径[）]{.chpule2}[
]{.chpuri2}である丸いくいのふりをします[。]{.chpule2}[ ]{.chpuri2}

<figure class="code">
<pre class="code" lang="pseudocode"><code>// 互換なインターフェースを持つ二つのクラス RoundHole と RoundPeg を考
// 察。
class RoundHole is
    constructor RoundHole(radius) { …… }

    method getRadius() is
        // 穴の半径を返す。

    method fits(peg: RoundPeg) is
        return this.getRadius() &gt;= peg.getRadius()

class RoundPeg is
    constructor RoundPeg(radius) { …… }

    method getRadius() is
        // くいの半径を返す。


// しかし、SquarePeg は、非互換なクラス。
class SquarePeg is
    constructor SquarePeg(width) { …… }

    method getWidth() is
        // 四角いくいの幅を返す。


// アダプター・クラスを使えば、四角いくいを丸い穴に合わせられる。RoundPeg
// クラスを拡張してアダプター・オブジェクトが丸いくいとして機能。
class SquarePegAdapter extends RoundPeg is
    // 実際には、アダプターは SquarePeg クラスのインスタンスを保持。
    private field peg: SquarePeg

    constructor SquarePegAdapter(peg: SquarePeg) is
        this.peg = peg

    method getRadius() is
        // アダプターは、それが内包する四角いくいが入る半径の丸い杭のふり
        // をする。
        return peg.getWidth() * Math.sqrt(2) / 2


// クライアント・コードのどこかの様子。
hole = new RoundHole(5)
rpeg = new RoundPeg(5)
hole.fits(rpeg) // true

small_sqpeg = new SquarePeg(5)
large_sqpeg = new SquarePeg(10)
hole.fits(small_sqpeg) // これはコンパイル不可（非互換な型）

small_sqpeg_adapter = new SquarePegAdapter(small_sqpeg)
large_sqpeg_adapter = new SquarePegAdapter(large_sqpeg)
hole.fits(small_sqpeg_adapter) // true
hole.fits(large_sqpeg_adapter) // false</code></pre>
</figure>
:::

:::::::: {.section .applicability-container}
##  適応性 {#applicability}

::::::: applicability
::: applicability-problem
Adapter クラスは[、]{.chpule2}[
]{.chpuri2}既存のクラスを使用したいが[、]{.chpule2}[
]{.chpuri2}そのインターフェースが自分のコードの他の部分と互換性がない場合に使用します[。]{.chpule2}[
]{.chpuri2}
:::

::: applicability-solution
Adapter パターンでは[、]{.chpule2}[
]{.chpuri2}自分のクラスと[、]{.chpule2}[
]{.chpuri2}昔から引き継がれてきたクラスや外部提供のクラスや奇妙なインターフェースのクラスとの間で翻訳者として機能する中間層のクラスを作成します[。]{.chpule2}[
]{.chpuri2}
:::

::: applicability-problem
既存のいくつかのサブクラスを再利用したいが[、]{.chpule2}[
]{.chpuri2}スーパークラスに追加できる共通機能が欠けている場合に[、]{.chpule2}[
]{.chpuri2}このパターンを使用します[。]{.chpule2}[ ]{.chpuri2}
:::

::: applicability-solution
それぞれのサブクラスを拡張して[、]{.chpule2}[
]{.chpuri2}新しい子クラスに欠けている機能を追加する[、]{.chpule2}[
]{.chpuri2}ということも可能です[。]{.chpule2}[
]{.chpuri2}しかし[、]{.chpule2}[
]{.chpuri2}すべての新規クラスにでコードの複製が必要となります[。]{.chpule2}[
]{.chpuri2}これは[、]{.chpule2}[
]{.chpuri2}[何か怪しげな匂い](../smells/duplicate-code.html)がします[。]{.chpule2}[
]{.chpuri2}

もっとエレガントな解決策は[、]{.chpule2}[
]{.chpuri2}足りない機能をアダプター・クラスに入れることです[。]{.chpule2}[
]{.chpuri2}そして[、]{.chpule2}[
]{.chpuri2}機能の欠落したオブジェクトをアダプター・クラスでラップし[、]{.chpule2}[
]{.chpuri2}必要な機能を動的に追加します[。]{.chpule2}[
]{.chpuri2}これを実現するためには[、]{.chpule2}[
]{.chpuri2}対象となるクラスは共通のインターフェースを持ち[、]{.chpule2}[
]{.chpuri2}アダプターのフィールドがそのインターフェースに従う必要があります[。]{.chpule2}[
]{.chpuri2}この手法は[、]{.chpule2}[
]{.chpuri2}[Decorator](decorator.html)
パターンに非常によく似ています[。]{.chpule2}[ ]{.chpuri2}
:::
:::::::
::::::::

::: {.section .checklist}
##  実装方法 {#checklist}

1.  最低二つの非互換なクラスがあることを確認してください[：]{.chpule2}[
    ]{.chpuri2}

    -   有益だが変更不能な*[サ]{.cjk}[ー]{.cjk}[ビ]{.cjk}[ス]{.cjk}*クラス[[。]{.ls-1}]{.chpule2}[
        ]{.chpuri2}​[ ]{.chpule1}[（]{.chpuri1}多くの場合[、]{.chpule2}[
        ]{.chpuri2}外部提供だから[、]{.chpule2}[
        ]{.chpuri2}過去から引き継がれたコードだから[、]{.chpule2}[
        ]{.chpuri2}あるは[、]{.chpule2}[
        ]{.chpuri2}依存する要素の数が多すぎるためといった理由で変更できない[。]{.ls-05}[）]{.chpule2}[
        ]{.chpuri2}
    -   サービス・クラスを利用したい 1 個以上の
        *[ク]{.cjk}[ラ]{.cjk}[イ]{.cjk}[ア]{.cjk}[ン]{.cjk}[ト]{.cjk}*・クラス[。]{.chpule2}[
        ]{.chpuri2}

2.  クライアント・インタフェースを宣言し[、]{.chpule2}[
    ]{.chpuri2}クライアントとサービスとの情報伝達の方法を記述する[。]{.chpule2}[
    ]{.chpuri2}

3.  クライアント・インターフェースに従うアダプター・クラスを作成[。]{.chpule2}[
    ]{.chpuri2}メソッドはとりあえず空のままにする[。]{.chpule2}[
    ]{.chpuri2}

4.  アダプター・クラスに[、]{.chpule2}[
    ]{.chpuri2}サービス・オブジェクトへの参照を格納するためのフィールドを追加します[。]{.chpule2}[
    ]{.chpuri2}このフィールドは[、]{.chpule2}[
    ]{.chpuri2}コンストラクタで初期化するのが一般的な方法ですが[、]{.chpule2}[
    ]{.chpuri2}アダプターのメソッドの呼び出し時に渡す方が便利な場合もあります[。]{.chpule2}[
    ]{.chpuri2}

5.  クライアント・インタフェースの全メソッドをアダプター・クラスで一つずつ実装していってください[。]{.chpule2}[
    ]{.chpuri2}アダプターは[、]{.chpule2}[
    ]{.chpuri2}実際の仕事のほとんどはサービス・オブジェクトに委ね[、]{.chpule2}[
    ]{.chpuri2}インターフェースやデータ形式の変換だけを行うようにします[。]{.chpule2}[
    ]{.chpuri2}

6.  クライアントは[、]{.chpule2}[
    ]{.chpuri2}クライアント・インタフェースを介してアダプターを使用する必要があります[。]{.chpule2}[
    ]{.chpuri2}そうすることで[、]{.chpule2}[
    ]{.chpuri2}クライアントのコードに影響を与えることなくアダプターの変更や拡張が可能となります[。]{.chpule2}[
    ]{.chpuri2}
:::

:::::: {.section .pros-cons}
##  長所と短所 {#pros-cons}

::::: row
::: col-sm-6
-   
    *[単]{.cjk}[一]{.cjk}[責]{.cjk}[任]{.cjk}[の]{.cjk}[原]{.cjk}[則]{.cjk}*[。]{.chpule2}[
    ]{.chpuri2}インターフェースやデータ変換のコードを[、]{.chpule2}[
    ]{.chpuri2}プログラムの主要なビジネス・ロジックから分離可能[。]{.chpule2}[
    ]{.chpuri2}
-   
    *[開]{.cjk}[放]{.cjk}[閉]{.cjk}[鎖]{.cjk}[の]{.cjk}[原]{.cjk}[則]{.cjk}*[。]{.chpule2}[
    ]{.chpuri2}クライアント・インターフェースを介してアダプターと連携する限り[、]{.chpule2}[
    ]{.chpuri2}既存のクライアント側コードを壊すことなく[、]{.chpule2}[
    ]{.chpuri2}新しい種類のアダプターをプログラムに追加可能[。]{.chpule2}[
    ]{.chpuri2}
:::

::: col-sm-6
-   
    一連の新規のインターフェースとクラスを追加する必要があるため[、]{.chpule2}[
    ]{.chpuri2}全体的なコードの複雑性が増加[。]{.chpule2}[
    ]{.chpuri2}コードの他の部分と一致するようにサービス・クラスを書き直す方が単純でいい場合あり[。]{.chpule2}[
    ]{.chpuri2}
:::
:::::
::::::

::: {.section .relations}
##  他のパターンとの関係 {#relations}

-   [Bridge](bridge.html) は通常[、]{.chpule2}[
    ]{.chpuri2}アプリケーションの部分部分を独立して開発できるように[、]{.chpule2}[
    ]{.chpuri2}設計当初から使われます[。]{.chpule2}[
    ]{.chpuri2}一方[、]{.chpule2}[ ]{.chpuri2}[Adapter](adapter.html)
    は[、]{.chpule2}[
    ]{.chpuri2}既存のアプリケーションに対して利用され[、]{.chpule2}[
    ]{.chpuri2}本来は互換性のないクラスとうまく動作させるために使われます[。]{.chpule2}[
    ]{.chpuri2}

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

-   [Facade](facade.html)
    が既存のオブジェクトに対して新しいインターフェースを定義するのに対し[、]{.chpule2}[
    ]{.chpuri2}[Adapter](adapter.html)
    は既存のインターフェースを使えるようにしようとするものです[。]{.chpule2}[
    ]{.chpuri2}*Adapter*
    は通常一つのオブジェクトだけを包み込みますが[、]{.chpule2}[
    ]{.chpuri2}*Facade*
    は複数のオブジェクトからなるサブシステム全体を相手にします[。]{.chpule2}[
    ]{.chpuri2}

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
:::

::: {.section .implementations}
##  コード例 {#implementations}

[![Adapter を C#
で](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](adapter/csharp/example.html "Adapter を C# で"){.prog-lang-link}
[![Adapter を C++
で](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](adapter/cpp/example.html "Adapter を C++ で"){.prog-lang-link}
[![Adapter を Go
で](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](adapter/go/example.html "Adapter を Go で"){.prog-lang-link}
[![Adapter を Java
で](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](adapter/java/example.html "Adapter を Java で"){.prog-lang-link}
[![Adapter を PHP
で](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](adapter/php/example.html "Adapter を PHP で"){.prog-lang-link}
[![Adapter を Python
で](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](adapter/python/example.html "Adapter を Python で"){.prog-lang-link}
[![Adapter を Ruby
で](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](adapter/ruby/example.html "Adapter を Ruby で"){.prog-lang-link}
[![Adapter を Rust
で](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](adapter/rust/example.html "Adapter を Rust で"){.prog-lang-link}
[![Adapter を Swift
で](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](adapter/swift/example.html "Adapter を Swift で"){.prog-lang-link}
[![Adapter を TypeScript
で](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](adapter/typescript/example.html "Adapter を TypeScript で"){.prog-lang-link}
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

[Bridge []{.fa .fa-arrow-right}](bridge.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### 戻る

[[]{.fa
.fa-arrow-left} 構造に関するパターン](structural-patterns.html){.btn
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
