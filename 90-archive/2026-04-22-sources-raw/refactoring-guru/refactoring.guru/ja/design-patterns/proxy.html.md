::::::::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} /
[デザインパターン](../design-patterns.html){.type} /
[構造に関するパターン](structural-patterns.html){.type}
:::

# Proxy {#proxy .title}

::: aka
別名：[プロキシー]{style="display:inline-block"}
:::

::: {.section .intent}
##  一言でいうと {#intent}

**Proxy**[ ]{.chpule1}[（]{.chpuri1}プロキシー[、]{.chpule2}[
]{.chpuri2}代理[）]{.chpule2}[ ]{.chpuri2}は[、]{.chpule2}[
]{.chpuri2}構造に関するデザインパターンの一つで[、]{.chpule2}[
]{.chpuri2}他のオブジェクトの代理[、]{.chpule2}[
]{.chpuri2}代用を提供します[。]{.chpule2}[
]{.chpuri2}プロキシーは[、]{.chpule2}[
]{.chpuri2}元のオブジェクトへのアクセスを制御し[、]{.chpule2}[
]{.chpuri2}元のオブジェクトへリクエストが行く前か後に別の何かを行うようにすることができます[。]{.chpule2}[
]{.chpuri2}

<figure class="image">
<img
src="../../images/patterns/content/proxy/proxyf258.png?id=efece4647fb11e3f7539291796327666"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/proxy/proxy.png?id=efece4647fb11e3f7539291796327666 640w,/images/patterns/content/proxy/proxy-1.5x.png?id=80a11690fa49172c517470a834289b60 960w,/images/patterns/content/proxy/proxy-2x.png?id=fb3d14e21c210a758d4777f4d93dce09 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Proxy デザインパターン" />
</figure>
:::

::: {.section .problem}
##  問題 {#problem}

いったいなぜオブジェクトへのアクセスを制御したいのかって[？]{.chpule2}[
]{.chpuri2}一例[：]{.chpule2}[
]{.chpuri2}システム資源を大量に消費する巨大なオブジェクトがあるとします[。]{.chpule2}[
]{.chpuri2}それは時々必要になりますが[、]{.chpule2}[
]{.chpuri2}常に必要なわけではありません[。]{.chpule2}[ ]{.chpuri2}

<figure class="image">
<img
src="../../images/patterns/diagrams/proxy/problem-ja5dc6.png?id=6ce48e36bde7247e683d7968a7024abf"
style="aspect-ratio: 3.19;"
srcset="/images/patterns/diagrams/proxy/problem-ja.png?id=6ce48e36bde7247e683d7968a7024abf 510w,/images/patterns/diagrams/proxy/problem-ja-1.5x.png?id=388a4d8eb47eb73c03cc1c5a24ef64c1 765w,/images/patterns/diagrams/proxy/problem-ja-2x.png?id=5e7698e5f0fb5eaa21c174380b56ec7d 1020w"
sizes="(max-width: 720px) 100vw, 510px" loading="lazy" width="510"
alt="Proxy パターンによって解決された問題" />
<figcaption><p>データベースのクエリは本当に遅いことがある<span
class="chpule2">。</span><span class="chpuri2"> </span></p></figcaption>
</figure>

遅延初期化[ ]{.chpule1}[（]{.chpuri1}lazy initialization[）]{.chpule2}[
]{.chpuri2}を実装することもできます[。]{.chpule2}[
]{.chpuri2}つまり[、]{.chpule2}[
]{.chpuri2}オブジェクトを実際に必要な時まで待って作成します[。]{.chpule2}[
]{.chpuri2}すべてのオブジェクトのクライアントが[、]{.chpule2}[
]{.chpuri2}何らかの遅延初期化コードを実行する必要があります[。]{.chpule2}[
]{.chpuri2}残念ながら[、]{.chpule2}[
]{.chpuri2}これにより多くのコードが重複する可能性があります[。]{.chpule2}[
]{.chpuri2}

理想的な世界では[、]{.chpule2}[
]{.chpuri2}このコードをオブジェクトのクラスに直接置きたいところですが[、]{.chpule2}[
]{.chpuri2}それがいつも可能とは限りません[。]{.chpule2}[
]{.chpuri2}たとえば[、]{.chpule2}[
]{.chpuri2}そのクラスはソースコード非公開の外部開発ライブラリーの一部なのかもしれません[。]{.chpule2}[
]{.chpuri2}
:::

::: {.section .solution}
##  解決策 {#solution}

Proxy パターンでは[、]{.chpule2}[
]{.chpuri2}元のサービス・オブジェクトと同じインターフェースで新しいプロキシー・クラスを作成します[。]{.chpule2}[
]{.chpuri2}次に[、]{.chpule2}[
]{.chpuri2}アプリを変更して[、]{.chpule2}[
]{.chpuri2}元のオブジェクトのすべてのクライアントに[、]{.chpule2}[
]{.chpuri2}プロキシーのオブジェクトを渡すようにします[。]{.chpule2}[
]{.chpuri2}クライアントからのリクエストを受け取ると[、]{.chpule2}[
]{.chpuri2}プロキシーは実際のサービス・オブジェクトを作成し[、]{.chpule2}[
]{.chpuri2}すべての作業を委任します[。]{.chpule2}[ ]{.chpuri2}

<figure class="image">
<img
src="../../images/patterns/diagrams/proxy/solution-ja0607.png?id=51aae8249647c021e26ae0756c6e1425"
style="aspect-ratio: 3.19;"
srcset="/images/patterns/diagrams/proxy/solution-ja.png?id=51aae8249647c021e26ae0756c6e1425 510w,/images/patterns/diagrams/proxy/solution-ja-1.5x.png?id=0929b5d113099a676cbb96b30d26e805 765w,/images/patterns/diagrams/proxy/solution-ja-2x.png?id=7ed72119166edb60a92f11cba410fc59 1020w"
sizes="(max-width: 720px) 100vw, 510px" loading="lazy" width="510"
alt="Proxy パターンによる解決策" />
<figcaption><p>プロキシーはデータベース・オブジェクトのふりをする<span
class="chpule2">。</span><span class="chpuri2">
</span>遅延初期化やキャッシュ処理を<span class="chpule2">、</span><span
class="chpuri2">
</span>クライアントや実際のデータベース・オブジェクトに知られることなく行える<span
class="chpule2">。</span><span class="chpuri2"> </span></p></figcaption>
</figure>

しかし[、]{.chpule2}[
]{.chpuri2}そんなことをする利点は何でしょう[？]{.chpule2}[
]{.chpuri2}クラスの一次的な仕事の前後に何かを実行する必要がある場合[、]{.chpule2}[
]{.chpuri2}プロキシーは元クラスを変更せずにこれを行うことができます[。]{.chpule2}[
]{.chpuri2}プロキシーは元のクラスと同じインターフェースを実装しているため[、]{.chpule2}[
]{.chpuri2}実際のサービス・オブジェクトを期待しているクライアントに渡すことができます[。]{.chpule2}[
]{.chpuri2}
:::

::: {.section .analogy}
##  現実世界でのたとえ {#analogy}

<figure class="image">
<img
src="../../images/patterns/diagrams/proxy/live-example66da.png?id=a268c57fdaf073ee81cf4dfc7239eae2"
style="aspect-ratio: 2.57;"
srcset="/images/patterns/diagrams/proxy/live-example.png?id=a268c57fdaf073ee81cf4dfc7239eae2 540w,/images/patterns/diagrams/proxy/live-example-1.5x.png?id=4b8ee7f90cf76180004e9f5d37661ee2 810w,/images/patterns/diagrams/proxy/live-example-2x.png?id=8b8083fa1954d2d92ca84a5a5636ead7 1080w"
sizes="(max-width: 720px) 100vw, 540px" loading="lazy" width="540"
alt="クレジットカードは、現金の束のプロキシー" />
<figcaption><p>クレジットカードは<span class="chpule2">、</span><span
class="chpuri2"> </span>まるで現金と同じように<span
class="chpule2">、</span><span class="chpuri2">
</span>支払いに使用可能<span class="chpule2">。</span><span
class="chpuri2"> </span></p></figcaption>
</figure>

クレジットカードは[、]{.chpule2}[
]{.chpuri2}銀行口座のプロキシーであり[、]{.chpule2}[
]{.chpuri2}銀行口座は[、]{.chpule2}[
]{.chpuri2}現金の束のプロキシーです[。]{.chpule2}[
]{.chpuri2}どちらも同じインターフェース[、]{.chpule2}[
]{.chpuri2}つまり支払い方法を実装しています[。]{.chpule2}[
]{.chpuri2}消費者は[、]{.chpule2}[
]{.chpuri2}大量の現金を持ち歩く必要がないので[、]{.chpule2}[
]{.chpuri2}気分が楽です[。]{.chpule2}[
]{.chpuri2}取引先からの入金が店の銀行口座に電子的に追加されるので[、]{.chpule2}[
]{.chpuri2}現金を失ったり[、]{.chpule2}[
]{.chpuri2}銀行に行く途中で強奪されるリスクもなくなり[、]{.chpule2}[
]{.chpuri2}店主も幸せです[。]{.chpule2}[ ]{.chpuri2}
:::

::::: {.section .structure-container}
##  構造 {#structure}

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/proxy/structure7fcd.png?id=f2478a82a84e1a1e512a8414bf1abd1c"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1;"
srcset="/images/patterns/diagrams/proxy/structure.png?id=f2478a82a84e1a1e512a8414bf1abd1c 370w,/images/patterns/diagrams/proxy/structure-1.5x.png?id=0db7bf3c1193b2a1961894349f1e07ad 555w,/images/patterns/diagrams/proxy/structure-2x.png?id=3d54eeca9af4aa373e989a73463539b5 740w"
sizes="(max-width: 720px) 100vw, 370px" loading="lazy" width="370"
alt="Proxy デザインパターンの構造" /><img
src="../../images/patterns/diagrams/proxy/structure-indexed2b2c.png?id=e56d420f31925b3d41348c5036d3cd64"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.05;"
srcset="/images/patterns/diagrams/proxy/structure-indexed.png?id=e56d420f31925b3d41348c5036d3cd64 410w,/images/patterns/diagrams/proxy/structure-indexed-1.5x.png?id=5b24af632e76d81d24eab0b7c0be1a66 615w,/images/patterns/diagrams/proxy/structure-indexed-2x.png?id=ddf2b3b4807e52330c456c59fc52d504 820w"
sizes="(max-width: 720px) 100vw, 410px" loading="lazy" width="410"
alt="Proxy デザインパターンの構造" />
</figure>
:::

1.  **サービス・インターフェース**[ ]{.chpule1}[（]{.chpuri1}Service
    Interface[）]{.chpule2}[ ]{.chpuri2}は[、]{.chpule2}[
    ]{.chpuri2}サービスのインターフェースを宣言します[。]{.chpule2}[
    ]{.chpuri2}プロキシーは[、]{.chpule2}[
    ]{.chpuri2}サービス・オブジェクトのふりをするために[、]{.chpule2}[
    ]{.chpuri2}このインターフェースに沿わなければなりません[。]{.chpule2}[
    ]{.chpuri2}

2.  **サービス**[ ]{.chpule1}[（]{.chpuri1}Service[）]{.chpule2}[
    ]{.chpuri2}は[、]{.chpule2}[
    ]{.chpuri2}何らかの役に立つビジネス・ロジックを提供するクラスです[。]{.chpule2}[
    ]{.chpuri2}

3.  **プロキシー**[ ]{.chpule1}[（]{.chpuri1}Proxy[）]{.chpule2}[
    ]{.chpuri2}クラスには[、]{.chpule2}[
    ]{.chpuri2}サービス・オブジェクトを指す参照フィールドがあります[。]{.chpule2}[
    ]{.chpuri2}プロキシーの行うべき処理[
    ]{.chpule1}[（]{.chpuri1}たとえば[、]{.chpule2}[
    ]{.chpuri2}遅延初期化[、]{.chpule2}[
    ]{.chpuri2}ロギング[、]{.chpule2}[
    ]{.chpuri2}アクセス制御[、]{.chpule2}[
    ]{.chpuri2}キャッシュ処理など[）]{.chpule2}[
    ]{.chpuri2}が完了すると[、]{.chpule2}[
    ]{.chpuri2}プロキシーは[、]{.chpule2}[
    ]{.chpuri2}サービス・オブジェクトにリクエストを渡します[。]{.chpule2}[
    ]{.chpuri2}

    通常[、]{.chpule2}[
    ]{.chpuri2}プロキシーはサービス・オブジェクトのライフサイクルを完全に管理します[。]{.chpule2}[
    ]{.chpuri2}

4.  **クライアント**[ ]{.chpule1}[（]{.chpuri1}Client[）]{.chpule2}[
    ]{.chpuri2}は[、]{.chpule2}[
    ]{.chpuri2}同じインターフェースを介してサービスとプロキシーの両方で動作します[。]{.chpule2}[
    ]{.chpuri2}こうすることで[、]{.chpule2}[
    ]{.chpuri2}サービス・オブジェクトを期待するどんなコードにでもプロキシーを渡すことができます[。]{.chpule2}[
    ]{.chpuri2}
::::
:::::

::: {.section .pseudocode}
##  擬似コード {#pseudocode}

この例では[、]{.chpule2}[ ]{.chpuri2}どのように **Proxy**
パターンが[、]{.chpule2}[ ]{.chpuri2}外部作成の YouTube
統合ライブラリーに遅延初期化とキャッシグを導入するのに役立つかを説明します[。]{.chpule2}[
]{.chpuri2}

<figure class="image">
<img
src="../../images/patterns/diagrams/proxy/example4e80.png?id=ddb1402479562b9e2c776933cc861bed"
style="aspect-ratio: 1.23;"
srcset="/images/patterns/diagrams/proxy/example.png?id=ddb1402479562b9e2c776933cc861bed 490w,/images/patterns/diagrams/proxy/example-1.5x.png?id=9b7608e1ef46a52d4ca1d7d89986e5b0 735w,/images/patterns/diagrams/proxy/example-2x.png?id=40f22d12d183b9285a380e4a072efb16 980w"
sizes="(max-width: 720px) 100vw, 490px" loading="lazy" width="490"
alt="Proxy パターンの例の構造" />
<figcaption><p>プロキシーを使用してサービスの結果をキャッシュ</p></figcaption>
</figure>

ライブラリーは[、]{.chpule2}[
]{.chpuri2}ビデオのダウンロードをするクラスを提供します[。]{.chpule2}[
]{.chpuri2}しかしながらそれは[、]{.chpule2}[
]{.chpuri2}かなり非効率です[。]{.chpule2}[
]{.chpuri2}もしクライアント・アプリケーションが同じビデオを複数回要求すると[、]{.chpule2}[
]{.chpuri2}ライブラリーは[、]{.chpule2}[
]{.chpuri2}それを何回もダウンロードします[。]{.chpule2}[
]{.chpuri2}ファイルをキャッシングして再利用をすることはしません[。]{.chpule2}[
]{.chpuri2}

プロキシー・クラスは元のダウンロード・クラスと同じインターフェースを実装し[、]{.chpule2}[
]{.chpuri2}すべての作業を委任します[。]{.chpule2}[
]{.chpuri2}ただし[、]{.chpule2}[
]{.chpuri2}ダウンロードしたファイルを記録し[、]{.chpule2}[
]{.chpuri2}アプリが同じ動画を複数回リクエストした時は[、]{.chpule2}[
]{.chpuri2}キャッシュされた結果を返します[。]{.chpule2}[ ]{.chpuri2}

<figure class="code">
<pre class="code" lang="pseudocode"><code>// リモート・サービスのインターフェース。
interface ThirdPartyYouTubeLib is
    method listVideos()
    method getVideoInfo(id)
    method downloadVideo(id)

// サービス・コネクターの具体的な実装。このクラスのメソッドは、YouTube か
// ら情報を要求できる。リクエストの処理速度は、ユーザーのインターネット接
// 続状況と YoutTube 側の状況に依存。同時にいくつものリクエストがあると、
// 仮に全部が同じ情報のリクエストだったとしても、アプリケーションは減速す
// る。
class ThirdPartyYouTubeClass implements ThirdPartyYouTubeLib is
    method listVideos() is
        // YouTube に API リクエストを送信。

    method getVideoInfo(id) is
        // ビデオに関するメタデータの取得。

    method downloadVideo(id) is
        // YouTube からビデオ・ファイルをダウンロード。

// 帯域幅を節約するために、リクエストの結果をキャッシュし、しばらくの間保
// 存することが可能。しかし、そのようなコードを直接サービス・クラスに入れ
// ることは不可能な場合あり。たとえば、外部作成のライブラリーの一部として
// 提供されていたり、クラスが final と定義されていたりする場合。サービス・
// クラスと同じインターフェースを実装する新しいプロキシー・クラスを作り、
// キャッシュのためのコードを追加するのは、このため。本当のリクエストを
// 送信する時にだけ、サービス・オブジェクトにリクエストを委任する。
class CachedYouTubeClass implements ThirdPartyYouTubeLib is
    private field service: ThirdPartyYouTubeLib
    private field listCache, videoCache
    field needReset

    constructor CachedYouTubeClass(service: ThirdPartyYouTubeLib) is
        this.service = service

    method listVideos() is
        if (listCache == null || needReset)
            listCache = service.listVideos()
        return listCache

    method getVideoInfo(id) is
        if (videoCache == null || needReset)
            videoCache = service.getVideoInfo(id)
        return videoCache

    method downloadVideo(id) is
        if (!downloadExists(id) || needReset)
            service.downloadVideo(id)

// 以前はサービス・オブジェクトと直接機能していた GUI のクラスは、あるイ
// ンターフェースを通してサービス・オブジェクトと機能する限り変更不要。本
// 物のサービス・オブジェクトに代わってプロキシー・オブジェクトを安全に渡
// すことが可能。これは、両方のクラスが同じインターフェースを実装するため。
class YouTubeManager is
    protected field service: ThirdPartyYouTubeLib

    constructor YouTubeManager(service: ThirdPartyYouTubeLib) is
        this.service = service

    method renderVideoPage(id) is
        info = service.getVideoInfo(id)
        // ビデオ・ページを描画。

    method renderListPanel() is
        list = service.listVideos()
        // ビデオのサムネールをリスト表示。

    method reactOnUserInput() is
        renderVideoPage()
        renderListPanel()

// アプリケーションは、プロキシーをその場で構成可能。
class Application is
    method init() is
        aYouTubeService = new ThirdPartyYouTubeClass()
        aYouTubeProxy = new CachedYouTubeClass(aYouTubeService)
        manager = new YouTubeManager(aYouTubeProxy)
        manager.reactOnUserInput()</code></pre>
</figure>
:::

:::::::::::::::: {.section .applicability-container}
##  適応性 {#applicability}

::::::::::::::: applicability
Proxy パターンの用途はたくさんあります[。]{.chpule2}[
]{.chpuri2}最もよく使われる用途を見てみましょう[。]{.chpule2}[
]{.chpuri2}

::: applicability-problem
遅延初期化[
]{.chpule1}[（]{.chpuri1}仮想プロキシー[）]{.ls-05}[。]{.chpule2}[
]{.chpuri2}たまにしか必要のないヘビー級のサービス・オブジェクトが常に使用可能となっている場合です[。]{.chpule2}[
]{.chpuri2}
:::

::: applicability-solution
アプリが起動した時にオブジェクトを作成する代わりに[、]{.chpule2}[
]{.chpuri2}オブジェクトの初期化を実際に必要な時まで遅らせることができます[。]{.chpule2}[
]{.chpuri2}
:::

::: applicability-problem
アクセス・コントロール[
]{.chpule1}[（]{.chpuri1}保護プロキシー[）]{.ls-05}[。]{.chpule2}[
]{.chpuri2}これは[、]{.chpule2}[
]{.chpuri2}特定のクライアントだけがサービス・オブジェクトを使用できるようにしたい場合です[。]{.chpule2}[
]{.chpuri2}たとえば[、]{.chpule2}[
]{.chpuri2}対象オブジェクトがオペレーティングシステムの基幹部分で[、]{.chpule2}[
]{.chpuri2}クライアントが様々な起動アプリケーション[
]{.chpule1}[（]{.chpuri1}悪意のあるものを含む[）]{.chpule2}[
]{.chpuri2}である場合[。]{.chpule2}[ ]{.chpuri2}
:::

::: applicability-solution
プロキシーは[、]{.chpule2}[
]{.chpuri2}クライアントの資格情報が何らかの条件に一致する場合にのみ[、]{.chpule2}[
]{.chpuri2}リクエストをサービス・オブジェクトに渡すことができます[。]{.chpule2}[
]{.chpuri2}
:::

::: applicability-problem
リモート・サービスのローカル実行[
]{.chpule1}[（]{.chpuri1}リモート・プロキシー[）]{.ls-05}[。]{.chpule2}[
]{.chpuri2}これは[、]{.chpule2}[
]{.chpuri2}サービス・オブジェクトがリモート・サーバー上にある場合です[。]{.chpule2}[
]{.chpuri2}
:::

::: applicability-solution
この場合[、]{.chpule2}[ ]{.chpuri2}プロキシーは[、]{.chpule2}[
]{.chpuri2}ネットワークを介してクライアントのリクエストを渡します[。]{.chpule2}[
]{.chpuri2}プロキシーがネットワークに関わる面倒で厄介な処理を実行します[。]{.chpule2}[
]{.chpuri2}
:::

::: applicability-problem
ロギング・リクエスト[
]{.chpule1}[（]{.chpuri1}ロギング・プロキシー[）]{.ls-05}[。]{.chpule2}[
]{.chpuri2}これは[、]{.chpule2}[
]{.chpuri2}サービス・オブジェクトへのリクエストの履歴を残したい場合です[。]{.chpule2}[
]{.chpuri2}
:::

::: applicability-solution
プロキシーは[、]{.chpule2}[
]{.chpuri2}各リクエストをサービスに渡す前にログに記録できます[。]{.chpule2}[
]{.chpuri2}
:::

::: applicability-problem
リクエスト結果のキャッシュ[
]{.chpule1}[（]{.chpuri1}キャッシング・プロキシー[）]{.ls-05}[。]{.chpule2}[
]{.chpuri2}これは[、]{.chpule2}[
]{.chpuri2}クライアントのリクエストの結果をキャッシュし[、]{.chpule2}[
]{.chpuri2}このキャシュのライフサイクルを管理する必要がある場合です[。]{.chpule2}[
]{.chpuri2}これは特に[、]{.chpule2}[
]{.chpuri2}リクエストの結果が非常に大きい場合に役立ちます[。]{.chpule2}[
]{.chpuri2}
:::

::: applicability-solution
プロキシーは[、]{.chpule2}[ ]{.chpuri2}定期的になされ[、]{.chpule2}[
]{.chpuri2}同じ結果に至るリクエストのキャッシュを実装できます[。]{.chpule2}[
]{.chpuri2}プロキシーはリクエストのパラメーターをキャッシュのキーとして使用するかもしれません[。]{.chpule2}[
]{.chpuri2}
:::

::: applicability-problem
スマート参照[。]{.chpule2}[ ]{.chpuri2}これは[、]{.chpule2}[
]{.chpuri2}重量級オブジェクトを[、]{.chpule2}[
]{.chpuri2}それを使うクライアントがなくなったら破棄できるようにしたい場合に重宝します[。]{.chpule2}[
]{.chpuri2}
:::

::: applicability-solution
プロキシーは[、]{.chpule2}[
]{.chpuri2}サービス・オブジェクトの参照をしたクライアントとその結果を記録することができます[。]{.chpule2}[
]{.chpuri2}時々プロキシーは[、]{.chpule2}[
]{.chpuri2}クライアント一つずつに対して[、]{.chpule2}[
]{.chpuri2}まだ活動中かどうかを調べます[。]{.chpule2}[
]{.chpuri2}クライアントのリストが空になったら[、]{.chpule2}[
]{.chpuri2}プロキシーはサービス・オブジェクトを破棄し[、]{.chpule2}[
]{.chpuri2}そのシステム資源を解放するかもしれません[。]{.chpule2}[
]{.chpuri2}

プロキシーは[、]{.chpule2}[
]{.chpuri2}クライアントがサービス・オブジェクトを変更したかどうかを追跡することもできます[。]{.chpule2}[
]{.chpuri2}その後[、]{.chpule2}[
]{.chpuri2}変更されていないオブジェクトは他のクライアントによって再利用されるかもしれません[。]{.chpule2}[
]{.chpuri2}
:::
:::::::::::::::
::::::::::::::::

::: {.section .checklist}
##  実装方法 {#checklist}

1.  既存のサービス・インターフェースがない場合は[、]{.chpule2}[
    ]{.chpuri2}プロキシーとサービス・オブジェクトを交換可能にするために[、]{.chpule2}[
    ]{.chpuri2}インターフェースを作成します[。]{.chpule2}[
    ]{.chpuri2}サービス・クラスからインターフェースを抽出することは[、]{.chpule2}[
    ]{.chpuri2}必ずしも可能ではありません[。]{.chpule2}[
    ]{.chpuri2}というのは[、]{.chpule2}[
    ]{.chpuri2}そのサービスを使用しているすべてのクライアントを変更する必要が出てくるからです[。]{.chpule2}[
    ]{.chpuri2}次善の策としては[、]{.chpule2}[
    ]{.chpuri2}プロキシーをサービス・クラスのサブクラスにすることです[。]{.chpule2}[
    ]{.chpuri2}これによりサービスのインターフェースを継承します[。]{.chpule2}[
    ]{.chpuri2}

2.  プロキシー・クラスを作成します[。]{.chpule2}[
    ]{.chpuri2}サービスへの参照を保存するためのフィールドを持っている必要があります[。]{.chpule2}[
    ]{.chpuri2}通常[、]{.chpule2}[
    ]{.chpuri2}プロキシーは[、]{.chpule2}[
    ]{.chpuri2}サービスを作成し[、]{.chpule2}[
    ]{.chpuri2}その後のライフサイクルを生涯に渡り管理します[。]{.chpule2}[
    ]{.chpuri2}稀にですが[、]{.chpule2}[
    ]{.chpuri2}クライアントはコンストラクターを介してサービスをプロキシーに渡すことがあります[。]{.chpule2}[
    ]{.chpuri2}

3.  目的に応じて[、]{.chpule2}[
    ]{.chpuri2}プロキシーにいくつかのメソッドを実装します[。]{.chpule2}[
    ]{.chpuri2}ほとんどの場合[、]{.chpule2}[
    ]{.chpuri2}何らかの作業を行った後[、]{.chpule2}[
    ]{.chpuri2}プロキシーは仕事をサービス・オブジェクトに委任します[。]{.chpule2}[
    ]{.chpuri2}

4.  生成メソッドを作成することを検討してください[。]{.chpule2}[
    ]{.chpuri2}それは[、]{.chpule2}[
    ]{.chpuri2}クライアントにプロキシーを渡すべきか[、]{.chpule2}[
    ]{.chpuri2}本物のサービスを渡すべきかを決めます[。]{.chpule2}[
    ]{.chpuri2}プロキシー・クラス中の静的メソッドとして実装することも[、]{.chpule2}[
    ]{.chpuri2}一人前のファクトリー・メソッドとして実装することもできます[。]{.chpule2}[
    ]{.chpuri2}

5.  サービス・オブジェクトの遅延初期化の実装を検討してください[。]{.chpule2}[
    ]{.chpuri2}
:::

:::::: {.section .pros-cons}
##  長所と短所 {#pros-cons}

::::: row
::: col-sm-6
-    クライアントに知られずにサービス・オブジェクトを制御可能
    [。]{.chpule2}[ ]{.chpuri2}
-    クライアントが関知しないのであれば[、]{.chpule2}[
    ]{.chpuri2}サービス・オブジェクトのライフサイクルの管理が可能[。]{.chpule2}[
    ]{.chpuri2}
-    サービス・オブジェクトの準備中[、]{.chpule2}[
    ]{.chpuri2}不足中でも[、]{.chpule2}[
    ]{.chpuri2}プロキシーは機能[。]{.chpule2}[ ]{.chpuri2}
-   
    *[開]{.cjk}[放]{.cjk}[閉]{.cjk}[鎖]{.cjk}[の]{.cjk}[原]{.cjk}[則]{.cjk}*[。]{.chpule2}[
    ]{.chpuri2}サービスやクライアントを変更せずとも新規プロキシーを導入可能[。]{.chpule2}[
    ]{.chpuri2}
:::

::: col-sm-6
-    多くの新しいクラスを導入する必要のため[、]{.chpule2}[
    ]{.chpuri2}コードがより複雑になる可能性あり[。]{.chpule2}[
    ]{.chpuri2}
-    サービスからの応答遅延の可能性[。]{.chpule2}[ ]{.chpuri2}
:::
:::::
::::::

::: {.section .relations}
##  他のパターンとの関係 {#relations}

-   [Adapter](adapter.html)
    はラップされたオブジェクトに対しては異なるインターフェースを提供し[、]{.chpule2}[
    ]{.chpuri2}[Proxy](proxy.html)
    は同じインターフェースを提供し[、]{.chpule2}[
    ]{.chpuri2}[Decorator](decorator.html)
    は強化したインターフェースを提供します[。]{.chpule2}[ ]{.chpuri2}

-   [Facade](facade.html) は[、]{.chpule2}[
    ]{.chpuri2}どちらも複雑な実体の緩衝材として機能し[、]{.chpule2}[
    ]{.chpuri2}初期化も行うという意味で[、]{.chpule2}[
    ]{.chpuri2}[Proxy](proxy.html) と似てます[。]{.chpule2}[
    ]{.chpuri2}*Facade* と異なり[、]{.chpule2}[ ]{.chpuri2}*Proxy*
    はそのサービス・オブジェクトと同じインターフェースを持ち[、]{.chpule2}[
    ]{.chpuri2}両者は交換可能です[。]{.chpule2}[ ]{.chpuri2}

-   [Decorator](decorator.html) と [Proxy](proxy.html)
    とは似たような構造をしていますが[、]{.chpule2}[
    ]{.chpuri2}その意図は非常に異なります[。]{.chpule2}[
    ]{.chpuri2}どちらのパターンも[、]{.chpule2}[ ]{.chpuri2}合成[
    ]{.chpule1}[（]{.chpuri1}コンポジション[）]{.chpule2}[
    ]{.chpuri2}の原則に基づいており[、]{.chpule2}[
    ]{.chpuri2}あるオブジェクトが仕事の一部を別のオブジェクトに委任します[。]{.chpule2}[
    ]{.chpuri2}違いは[、]{.chpule2}[ ]{.chpuri2}*Proxy*
    は通常そのサービス・オブジェクトのライフサイクルの管理を行うのに対し[、]{.chpule2}[
    ]{.chpuri2}*Decorators* では[、]{.chpule2}[
    ]{.chpuri2}常にクライアントが管理するという点です[。]{.chpule2}[
    ]{.chpuri2}
:::

::: {.section .implementations}
##  コード例 {#implementations}

[![Proxy を C#
で](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](proxy/csharp/example.html "Proxy を C# で"){.prog-lang-link}
[![Proxy を C++
で](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](proxy/cpp/example.html "Proxy を C++ で"){.prog-lang-link}
[![Proxy を Go
で](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](proxy/go/example.html "Proxy を Go で"){.prog-lang-link}
[![Proxy を Java
で](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](proxy/java/example.html "Proxy を Java で"){.prog-lang-link}
[![Proxy を PHP
で](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](proxy/php/example.html "Proxy を PHP で"){.prog-lang-link}
[![Proxy を Python
で](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](proxy/python/example.html "Proxy を Python で"){.prog-lang-link}
[![Proxy を Ruby
で](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](proxy/ruby/example.html "Proxy を Ruby で"){.prog-lang-link}
[![Proxy を Rust
で](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](proxy/rust/example.html "Proxy を Rust で"){.prog-lang-link}
[![Proxy を Swift
で](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](proxy/swift/example.html "Proxy を Swift で"){.prog-lang-link}
[![Proxy を TypeScript
で](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](proxy/typescript/example.html "Proxy を TypeScript で"){.prog-lang-link}
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

[振る舞いに関するパターン []{.fa
.fa-arrow-right}](behavioral-patterns.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### 戻る

[[]{.fa .fa-arrow-left} Flyweight](flyweight.html){.btn .btn-default
rel="prev"}
:::
::::::::::::::::::::::::::::::::::::::::

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
::::::::::::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::::::::::::
