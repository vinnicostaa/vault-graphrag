::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} /
[デザインパターン](../design-patterns.html){.type} /
[構造に関するパターン](structural-patterns.html){.type}
:::

# Facade {#facade .title}

::: aka
別名：[ファサード]{style="display:inline-block"}
:::

::: {.section .intent}
##  一言でいうと {#intent}

**Facade**[ ]{.chpule1}[（]{.chpuri1}ファサード[）]{.chpule2}[
]{.chpuri2}は[、]{.chpule2}[
]{.chpuri2}構造に関するデザインパターンの一つで[、]{.chpule2}[
]{.chpuri2}ライブラリー[、]{.chpule2}[
]{.chpuri2}フレームワーク[、]{.chpule2}[
]{.chpuri2}その他のクラスの複雑な組み合わせに対し[、]{.chpule2}[
]{.chpuri2}簡素化されたインターフェースを提供します[。]{.chpule2}[
]{.chpuri2}

<figure class="image">
<img
src="../../images/patterns/content/facade/facade721d.png?id=1f4be17305b6316fbd548edf1937ac3b"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/facade/facade.png?id=1f4be17305b6316fbd548edf1937ac3b 640w,/images/patterns/content/facade/facade-1.5x.png?id=a6af5003b243b59990842d24bdaae2df 960w,/images/patterns/content/facade/facade-2x.png?id=b69fce5943703f5f07c0ba38e3baaed0 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Facade デザインパターン" />
</figure>
:::

::: {.section .problem}
##  問題 {#problem}

高度なライブラリーやフレームワークに属する広範なオブジェクトを自分のコードに組み込む必要があるとします[。]{.chpule2}[
]{.chpuri2}通常は[、]{.chpule2}[
]{.chpuri2}これらすべてのオブジェクトを初期化し[、]{.chpule2}[
]{.chpuri2}依存関係を追跡し[、]{.chpule2}[
]{.chpuri2}正しい順序でメソッドを実行するなどを行う必要があります[。]{.chpule2}[
]{.chpuri2}

その結果[、]{.chpule2}[
]{.chpuri2}自分のコード中のクラスのビジネス上のロジックが[、]{.chpule2}[
]{.chpuri2}外部のクラスの実装の詳細と密に結合されることになり[、]{.chpule2}[
]{.chpuri2}コードの理解と維持が困難になります[。]{.chpule2}[ ]{.chpuri2}
:::

::: {.section .solution}
##  解決策 {#solution}

ファサードは[、]{.chpule2}[
]{.chpuri2}大変入り組んで複雑なサブシステムへの単純なインターフェースを提供するクラスです[。]{.chpule2}[
]{.chpuri2}ファサードは[、]{.chpule2}[
]{.chpuri2}直接サブシステムとやりとりするのに比べると限られた機能しか提供しないかもしれません[。]{.chpule2}[
]{.chpuri2}しかし[、]{.chpule2}[
]{.chpuri2}そこにはクライアントにとって本当に関心のある機能のみが含まれています[。]{.chpule2}[
]{.chpuri2}

とても多くの機能を備えた高級ライブラリーとアプリを統合する必要がありますが[、]{.chpule2}[
]{.chpuri2}そのうちのほんのちょっとの機能しか必要ない場合[、]{.chpule2}[
]{.chpuri2}ファサードが便利です[。]{.chpule2}[ ]{.chpuri2}

たとえば[、]{.chpule2}[
]{.chpuri2}猫の面白い短編ビデオをソーシャルメディアにアップロードするアプリは[、]{.chpule2}[
]{.chpuri2}プロ用のビデオ変換ライブラリーを使用するかもしれません[。]{.chpule2}[
]{.chpuri2}しかし[、]{.chpule2}[
]{.chpuri2}本当に必要なのは[、]{.chpule2}[ ]{.chpuri2}ただ一つのメソッド
`encode­(filename, format)` を持つクラスだけです[。]{.chpule2}[
]{.chpuri2}このようなクラスを作成し[、]{.chpule2}[
]{.chpuri2}ビデオ変換ライブラリーと繋げると[、]{.chpule2}[
]{.chpuri2}最初のファサードができました[。]{.chpule2}[ ]{.chpuri2}
:::

::: {.section .analogy}
##  現実世界でのたとえ {#analogy}

<figure class="image">
<img
src="../../images/patterns/diagrams/facade/live-example-jaa931.png?id=6e22261746fea8da67e403a7fb5f0286"
style="aspect-ratio: 2.58;"
srcset="/images/patterns/diagrams/facade/live-example-ja.png?id=6e22261746fea8da67e403a7fb5f0286 490w,/images/patterns/diagrams/facade/live-example-ja-1.5x.png?id=d558ccc77797bf7e72ccb8f0fa4c4b26 735w,/images/patterns/diagrams/facade/live-example-ja-2x.png?id=da83a7d5a15cd238db03a02863da81af 980w"
sizes="(max-width: 720px) 100vw, 490px" loading="lazy" width="490"
alt="電話注文を受ける例" />
<figcaption><p>電話で注文</p></figcaption>
</figure>

店に電話して注文をする時[、]{.chpule2}[
]{.chpuri2}電話オペレーターは[、]{.chpule2}[
]{.chpuri2}すべてのサービスやデパートに対するファサードです[。]{.chpule2}[
]{.chpuri2}オペレーターは[、]{.chpule2}[
]{.chpuri2}注文システム[、]{.chpule2}[
]{.chpuri2}支払いゲートウェイ[、]{.chpule2}[
]{.chpuri2}配達サービスに対して簡単な音声インターフェースを提供します[。]{.chpule2}[
]{.chpuri2}
:::

::::: {.section .structure-container}
##  構造 {#structure}

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/facade/structure2883.png?id=258401362234ac77a2aaf1cde62339e7"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.47;"
srcset="/images/patterns/diagrams/facade/structure.png?id=258401362234ac77a2aaf1cde62339e7 560w,/images/patterns/diagrams/facade/structure-1.5x.png?id=98d134de0c368d382909ba162ec21767 840w,/images/patterns/diagrams/facade/structure-2x.png?id=528ca429555bce293b7c3bd90954e097 1120w"
sizes="(max-width: 720px) 100vw, 560px" loading="lazy" width="560"
alt="Facade デザインパターンの構造" /><img
src="../../images/patterns/diagrams/facade/structure-indexeddb1c.png?id=2da06d6b850701ea15cf72f9d2642fb8"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.58;"
srcset="/images/patterns/diagrams/facade/structure-indexed.png?id=2da06d6b850701ea15cf72f9d2642fb8 600w,/images/patterns/diagrams/facade/structure-indexed-1.5x.png?id=fad7d667f4d8d9a7d522748bcd37dfde 900w,/images/patterns/diagrams/facade/structure-indexed-2x.png?id=4d181bcf1df5d58c533e6c92461e9571 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Facade デザインパターンの構造" />
</figure>
:::

1.  **ファサード**[ ]{.chpule1}[（]{.chpuri1}Facade[）]{.chpule2}[
    ]{.chpuri2}は[、]{.chpule2}[
    ]{.chpuri2}サブシステムの機能の特定の部分への便利なアクセスを提供します[。]{.chpule2}[
    ]{.chpuri2}クライアントからのリクエストをどこにどう送り[、]{.chpule2}[
    ]{.chpuri2}どういう細かい操作をすればいいのかを知っています[。]{.chpule2}[
    ]{.chpuri2}

2.  **追加のファサード**[ ]{.chpule1}[（]{.chpuri1}Additional
    Facade[）]{.chpule2}[ ]{.chpuri2}クラスを作成すると[、]{.chpule2}[
    ]{.chpuri2}関係ない機能によって単一のファサードが汚染され[、]{.chpule2}[
    ]{.chpuri2}さらに複雑な構造のクラスがまた一つできてしまうのを防止することができます[。]{.chpule2}[
    ]{.chpuri2}追加のファサードは[、]{.chpule2}[
    ]{.chpuri2}クライアントからでも[、]{.chpule2}[
    ]{.chpuri2}他のファサードからでも使用できます[。]{.chpule2}[
    ]{.chpuri2}

3.  **複雑なサブシステム**は[、]{.chpule2}[
    ]{.chpuri2}何十もの様々なオブジェクトで構成されています[。]{.chpule2}[
    ]{.chpuri2}それらを使って何か意味のあることをするためには[、]{.chpule2}[
    ]{.chpuri2}サブシステムの実装の詳細について熟知する必要があります[。]{.chpule2}[
    ]{.chpuri2}たとえば[、]{.chpule2}[
    ]{.chpuri2}オブジェクトを正しい順序で初期化し[、]{.chpule2}[
    ]{.chpuri2}適切な形式のデータを提供する必要があります[。]{.chpule2}[
    ]{.chpuri2}

    サブシステム内のクラスは[、]{.chpule2}[
    ]{.chpuri2}ファサードの存在を認識していません[。]{.chpule2}[
    ]{.chpuri2}クラスはシステム内で動作し[、]{.chpule2}[
    ]{.chpuri2}お互いに直接連携して機能します[。]{.chpule2}[ ]{.chpuri2}

4.  **クライアント**は[、]{.chpule2}[
    ]{.chpuri2}サブシステムのオブジェクトを直接呼び出す代わりにファサードを使用します[。]{.chpule2}[
    ]{.chpuri2}
::::
:::::

::: {.section .pseudocode}
##  擬似コード {#pseudocode}

この例では[、]{.chpule2}[ ]{.chpuri2}**Facade**
パターンを使い[、]{.chpule2}[
]{.chpuri2}複雑なビデオ変換フレームワークとのやりとりを簡素化します[。]{.chpule2}[
]{.chpuri2}

<figure class="image">
<img
src="../../images/patterns/diagrams/facade/example16bc.png?id=2249d134e3ff83819dfc19032f02eced"
style="aspect-ratio: 1.5;"
srcset="/images/patterns/diagrams/facade/example.png?id=2249d134e3ff83819dfc19032f02eced 570w,/images/patterns/diagrams/facade/example-1.5x.png?id=034ecbe0f3cc108f4ae49d05d1c77dbe 855w,/images/patterns/diagrams/facade/example-2x.png?id=f2c846d74041626c923ff3e8919b68a9 1140w"
sizes="(max-width: 720px) 100vw, 570px" loading="lazy" width="570"
alt="Facade パターン例の構造" />
<figcaption><p>単一のファサード・クラス内で複数の依存関係を分離する例</p></figcaption>
</figure>

フレームワークの何十ものクラスと直接やりとりするコードを書く代わりに[、]{.chpule2}[
]{.chpuri2}ファサードのクラスを作り[、]{.chpule2}[
]{.chpuri2}機能をカプセル化してコードの残りの部分から隠します[。]{.chpule2}[
]{.chpuri2}こうしておくと[、]{.chpule2}[
]{.chpuri2}フレームワークの将来のバージョンへのアップグレードや[、]{.chpule2}[
]{.chpuri2}他のものと置き換える作業の労力を最小に抑えることができます[。]{.chpule2}[
]{.chpuri2}アプリケーション内で変更が必要なのは[、]{.chpule2}[
]{.chpuri2}ファサードのメソッドの実装だけだからです[。]{.chpule2}[
]{.chpuri2}

<figure class="code">
<pre class="code" lang="pseudocode"><code>// 以下は、外部作成のビデオ変換フレームワークのクラス。自分たちのものでは
// ないので、単純化できない。

class VideoFile
// ……

class OggCompressionCodec
// ……

class MPEG4CompressionCodec
// ……

class CodecFactory
// ……

class BitrateReader
// ……

class AudioMixer
// ……


// フレームワークの複雑さを隠して、簡素なインターフェースとするためにファ
// サード・クラスを作成。機能性とシンプルさとの間の取引。
class VideoConverter is
    method convert(filename, format):File is
        file = new VideoFile(filename)
        sourceCodec = (new CodecFactory).extract(file)
        if (format == &quot;mp4&quot;)
            destinationCodec = new MPEG4CompressionCodec()
        else
            destinationCodec = new OggCompressionCodec()
        buffer = BitrateReader.read(filename, sourceCodec)
        result = BitrateReader.convert(buffer, destinationCodec)
        result = (new AudioMixer()).fix(result)
        return new File(result)

// アプリケーションのクラスは、複雑なフレームワークが提供する何億ものクラ
// スに依存しない。フレームワークを違うものに切り替える場合は、ファサード・
// クラスの書き換えだけで済む。
class Application is
    method main() is
        convertor = new VideoConverter()
        mp4 = convertor.convert(&quot;funny-cats-video.ogg&quot;, &quot;mp4&quot;)
        mp4.save()</code></pre>
</figure>
:::

:::::::: {.section .applicability-container}
##  適応性 {#applicability}

::::::: applicability
::: applicability-problem
複雑なサブシステムへの限定はされているが簡潔なインターフェースが必要な場合に[、]{.chpule2}[
]{.chpuri2}Facade パターンを使用します[。]{.chpule2}[ ]{.chpuri2}
:::

::: applicability-solution
多くの場合[、]{.chpule2}[
]{.chpuri2}サブシステムは時間の経過とともにより複雑になります[。]{.chpule2}[
]{.chpuri2}いろいろなデザインパターンを適用しても[、]{.chpule2}[
]{.chpuri2}クラス数が増える傾向があります[。]{.chpule2}[
]{.chpuri2}サブシステムは[、]{.chpule2}[
]{.chpuri2}様々な状況で柔軟で再利用しやすくなるかもしれません[。]{.chpule2}[
]{.chpuri2}しかしクライアントが行わなければならない構成と定型コードの量は増加し続けます[。]{.chpule2}[
]{.chpuri2}Facade は[、]{.chpule2}[
]{.chpuri2}ほとんどのクライアントが最も使用するサブシステム機能への近道を提供することによって[、]{.chpule2}[
]{.chpuri2}この問題を解決しようとします[。]{.chpule2}[ ]{.chpuri2}
:::

::: applicability-problem
サブシステムを多層構造にするために[、]{.chpule2}[
]{.chpuri2}ファサードを使用します[。]{.chpule2}[ ]{.chpuri2}
:::

::: applicability-solution
サブシステムの各レベルへのエントリー・ポイントを定義するために[、]{.chpule2}[
]{.chpuri2}ファサードを作成します[。]{.chpule2}[
]{.chpuri2}ファサードを通してのみやりとりをすることで[、]{.chpule2}[
]{.chpuri2}サブシステム間の結合度を減らすことができます[。]{.chpule2}[
]{.chpuri2}

ビデオ変換フレームワークの話に戻りましょう[。]{.chpule2}[
]{.chpuri2}これは[、]{.chpule2}[
]{.chpuri2}ビデオと音声の二つの層に分割することができます[。]{.chpule2}[
]{.chpuri2}各層で一つのファサードを作成し[、]{.chpule2}[
]{.chpuri2}各レイヤーのクラスをそれらのファサードを介してのみ相互に通信するようにします[。]{.chpule2}[
]{.chpuri2}このやり方は[、]{.chpule2}[
]{.chpuri2}[Mediator](mediator.html)
パターンによく似ています[。]{.chpule2}[ ]{.chpuri2}
:::
:::::::
::::::::

::: {.section .checklist}
##  実装方法 {#checklist}

1.  既存のサブシステムよりも簡単なインターフェースを提供することが可能かどうかを確認します[。]{.chpule2}[
    ]{.chpuri2}このインターフェースにより[、]{.chpule2}[
    ]{.chpuri2}クライアント・コードを多くのサブシステムのクラスから独立させられれば[、]{.chpule2}[
    ]{.chpuri2}うまい方向に向いています[。]{.chpule2}[ ]{.chpuri2}

2.  このインターフェースを宣言し[、]{.chpule2}[
    ]{.chpuri2}新ファサード・クラスとして定義します[。]{.chpule2}[
    ]{.chpuri2}クライアントからの呼び出しは[、]{.chpule2}[
    ]{.chpuri2}サブシステムの適切なオブジェクトに再送されます[。]{.chpule2}[
    ]{.chpuri2}クライアント・コードがすでにそれを行ってなければ[、]{.chpule2}[
    ]{.chpuri2}サブシステムの初期化やそれ以降のライフサイクル管理は[、]{.chpule2}[
    ]{.chpuri2}ファサードの責任です[。]{.chpule2}[ ]{.chpuri2}

3.  このパターンを最大限に活用するためには[、]{.chpule2}[
    ]{.chpuri2}すべてのクライアント・コードがファサード経由でサブシステムと通信するようにします[。]{.chpule2}[
    ]{.chpuri2}これで[、]{.chpule2}[
    ]{.chpuri2}クライアント・コードはサブシステムのコードの変更から保護されます[。]{.chpule2}[
    ]{.chpuri2}たとえば[、]{.chpule2}[
    ]{.chpuri2}サブシステムが新しいバージョンにアップグレードされた場合[、]{.chpule2}[
    ]{.chpuri2}ファサードのコードを変更するだけですみます[。]{.chpule2}[
    ]{.chpuri2}

4.  ファサードが[過大](../smells/large-class.html)になった場合は[、]{.chpule2}[
    ]{.chpuri2}ファサードの振る舞いの一部を抽出して[、]{.chpule2}[
    ]{.chpuri2}新規の精製ファサード・クラス作成を検討してみてください[。]{.chpule2}[
    ]{.chpuri2}
:::

:::::: {.section .pros-cons}
##  長所と短所 {#pros-cons}

::::: row
::: col-sm-6
-    サブシステムの複雑さからコードを分離可能[。]{.chpule2}[ ]{.chpuri2}
:::

::: col-sm-6
-    ファサードが[、]{.chpule2}[
    ]{.chpuri2}アプリのすべてのクラスに結合された[神オブジェクト](https://en.wikipedia.org/wiki/God_object)になる可能性あり[。]{.chpule2}[
    ]{.chpuri2}
:::
:::::
::::::

::: {.section .relations}
##  他のパターンとの関係 {#relations}

-   [Facade](facade.html)
    が既存のオブジェクトに対して新しいインターフェースを定義するのに対し[、]{.chpule2}[
    ]{.chpuri2}[Adapter](adapter.html)
    は既存のインターフェースを使えるようにしようとするものです[。]{.chpule2}[
    ]{.chpuri2}*Adapter*
    は通常一つのオブジェクトだけを包み込みますが[、]{.chpule2}[
    ]{.chpuri2}*Facade*
    は複数のオブジェクトからなるサブシステム全体を相手にします[。]{.chpule2}[
    ]{.chpuri2}

-   サブシステムがオブジェクトを作成する方法をクライアントから隠蔽することだけが目的なら[、]{.chpule2}[
    ]{.chpuri2}[Abstract Factory](abstract-factory.html) を
    [Facade](facade.html) の代わりに使えます[。]{.chpule2}[ ]{.chpuri2}

-   [Flyweight](flyweight.html)
    は多くの小さなオブジェクトを作る方法についてですが[、]{.chpule2}[
    ]{.chpuri2}[Facade](facade.html)
    はサブシステム全体を表す単一のオブジェクトを作る方法に関してです[。]{.chpule2}[
    ]{.chpuri2}

-   [Facade](facade.html) と [Mediator](mediator.html) は[、]{.chpule2}[
    ]{.chpuri2}多くの密に結合されたクラス間の協力関係を組織するという似た目的があります[。]{.chpule2}[
    ]{.chpuri2}

    -   *Facade* は[、]{.chpule2}[
        ]{.chpuri2}オブジェクトのサブシステムへの簡略化されたインターフェースを定義しますが[、]{.chpule2}[
        ]{.chpuri2}新しい機能が導入されるわけではありません[。]{.chpule2}[
        ]{.chpuri2}サブシステム自体はファサードを認識しません[。]{.chpule2}[
        ]{.chpuri2}サブシステム内のオブジェクト同士は直接やりとりします[。]{.chpule2}[
        ]{.chpuri2}
    -   *Mediator* は[、]{.chpule2}[
        ]{.chpuri2}システムのコンポーネント間の通信を一元化します[。]{.chpule2}[
        ]{.chpuri2}コンポーネントはメディエーター・オブジェクトについてのみ知っており[、]{.chpule2}[
        ]{.chpuri2}直接やりとりしません[。]{.chpule2}[ ]{.chpuri2}

-   多くの場合[、]{.chpule2}[
    ]{.chpuri2}ファサード・オブジェクトは一つだけあれば十分なので[、]{.chpule2}[
    ]{.chpuri2}[Facade](facade.html) はしばしば
    [Singleton](singleton.html) に変換可能です[。]{.chpule2}[
    ]{.chpuri2}

-   [Facade](facade.html) は[、]{.chpule2}[
    ]{.chpuri2}どちらも複雑な実体の緩衝材として機能し[、]{.chpule2}[
    ]{.chpuri2}初期化も行うという意味で[、]{.chpule2}[
    ]{.chpuri2}[Proxy](proxy.html) と似てます[。]{.chpule2}[
    ]{.chpuri2}*Facade* と異なり[、]{.chpule2}[ ]{.chpuri2}*Proxy*
    はそのサービス・オブジェクトと同じインターフェースを持ち[、]{.chpule2}[
    ]{.chpuri2}両者は交換可能です[。]{.chpule2}[ ]{.chpuri2}
:::

::: {.section .implementations}
##  コード例 {#implementations}

[![Facade を C#
で](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](facade/csharp/example.html "Facade を C# で"){.prog-lang-link}
[![Facade を C++
で](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](facade/cpp/example.html "Facade を C++ で"){.prog-lang-link}
[![Facade を Go
で](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](facade/go/example.html "Facade を Go で"){.prog-lang-link}
[![Facade を Java
で](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](facade/java/example.html "Facade を Java で"){.prog-lang-link}
[![Facade を PHP
で](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](facade/php/example.html "Facade を PHP で"){.prog-lang-link}
[![Facade を Python
で](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](facade/python/example.html "Facade を Python で"){.prog-lang-link}
[![Facade を Ruby
で](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](facade/ruby/example.html "Facade を Ruby で"){.prog-lang-link}
[![Facade を Rust
で](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](facade/rust/example.html "Facade を Rust で"){.prog-lang-link}
[![Facade を Swift
で](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](facade/swift/example.html "Facade を Swift で"){.prog-lang-link}
[![Facade を TypeScript
で](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](facade/typescript/example.html "Facade を TypeScript で"){.prog-lang-link}
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

[Flyweight []{.fa .fa-arrow-right}](flyweight.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### 戻る

[[]{.fa .fa-arrow-left} Decorator](decorator.html){.btn .btn-default
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
