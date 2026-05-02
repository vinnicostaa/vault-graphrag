::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} /
[デザインパターン](../design-patterns.html){.type} /
[生成に関するパターン](creational-patterns.html){.type}
:::

# Singleton {#singleton .title}

::: aka
別名：[シングルトン]{style="display:inline-block"}
:::

::: {.section .intent}
##  一言でいうと {#intent}

**Singleton**[ ]{.chpule1}[（]{.chpuri1}シングルトン[）]{.chpule2}[
]{.chpuri2}は[、]{.chpule2}[
]{.chpuri2}生成に関するデザインパターンの一つで[、]{.chpule2}[
]{.chpuri2}クラスが一つのインスタンスのみを持つことを保証するとともに[、]{.chpule2}[
]{.chpuri2}このインスタンスへの大域アクセス・ポイントを提供します[。]{.chpule2}[
]{.chpuri2}

<figure class="image">
<img
src="../../images/patterns/content/singleton/singleton4263.png?id=108a0b9b5ea5c4426e0afa4504491d6f"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/singleton/singleton.png?id=108a0b9b5ea5c4426e0afa4504491d6f 640w,/images/patterns/content/singleton/singleton-1.5x.png?id=29490ad6cd1426c63cf5f88243a1701c 960w,/images/patterns/content/singleton/singleton-2x.png?id=accb2cc7594f7a491ce01dddf0d2f876 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Singleton パターン" />
</figure>
:::

::: {.section .problem}
##  問題 {#problem}

Singleton パターンは[、]{.chpule2}[
]{.chpuri2}*[単]{.cjk}[一]{.cjk}[責]{.cjk}[任]{.cjk}[の]{.cjk}[原]{.cjk}[則]{.cjk}*に違反しますが[、]{.chpule2}[
]{.chpuri2}二つの問題を同時に解決します[：]{.chpule2}[ ]{.chpuri2}

1.  **クラスのインスタンスが一つだけであるであることを保証**します[。]{.chpule2}[
    ]{.chpuri2}いったい誰がクラスのインスタンス数を管理したいかですって[？]{.chpule2}[
    ]{.chpuri2}最も一般的な理由は[、]{.chpule2}[
    ]{.chpuri2}データベースやファイルなどの共有資源へのアクセスを制御するためです[。]{.chpule2}[
    ]{.chpuri2}

    こう動作します[：]{.chpule2}[
    ]{.chpuri2}オブジェクトを作ったとしましょう[。]{.chpule2}[
    ]{.chpuri2}しばらくしてもう一つ新しいものを作ることにしました[。]{.chpule2}[
    ]{.chpuri2}できたてのオブジェクトを受け取る代わりに[、]{.chpule2}[
    ]{.chpuri2}すでに作成したオブジェクトを取得することになります[。]{.chpule2}[
    ]{.chpuri2}

    この振る舞いは[、]{.chpule2}[
    ]{.chpuri2}通常のコンストラクターでは実現不能であることに注意してください[。]{.chpule2}[
    ]{.chpuri2}コンストラクターの呼び出しは[、]{.chpule2}[
    ]{.chpuri2}設計上[、]{.chpule2}[
    ]{.chpuri2}常に新規のオブジェクトを返します[。]{.chpule2}[
    ]{.chpuri2}

<figure class="image">
<img
src="../../images/patterns/content/singleton/singleton-comic-1-jaa861.png?id=9b11b1b8ea6acd90a0c56fb201e2a9a9"
style="aspect-ratio: 2;"
srcset="/images/patterns/content/singleton/singleton-comic-1-ja.png?id=9b11b1b8ea6acd90a0c56fb201e2a9a9 600w,/images/patterns/content/singleton/singleton-comic-1-ja-1.5x.png?id=233c71a716f6608adede0618825362c6 900w,/images/patterns/content/singleton/singleton-comic-1-ja-2x.png?id=b81f3b7eb99c9462256af50b45f6a49e 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="オブジェクトへのグローバル・アクセス" />
<figcaption><p>クライアントは<span class="chpule2">、</span><span
class="chpuri2">
</span>同じオブジェクトを使っていることに気づかないかもしれません<span
class="chpule2">。</span><span class="chpuri2"> </span></p></figcaption>
</figure>

2.  **そのインスタンスへの大域アクセス・ポイント**を提供する[。]{.chpule2}[
    ]{.chpuri2}昔[、]{.chpule2}[
    ]{.chpuri2}重要なオブジェクトを保存するためにあなた[
    ]{.chpule1}[（]{.chpuri1}はい[、]{.chpule2}[
    ]{.chpuri2}実は私です[）]{.chpule2}[
    ]{.chpuri2}が使用していたグローバル変数[、]{.chpule2}[
    ]{.chpuri2}覚えていますか[？]{.chpule2}[
    ]{.chpuri2}非常に便利なのですが[、]{.chpule2}[
    ]{.chpuri2}どんなコードでもそれらの変数の内容を変更でき[、]{.chpule2}[
    ]{.chpuri2}アプリがクラッシュする能性があるため[、]{.chpule2}[
    ]{.chpuri2}非常に危険です[。]{.chpule2}[ ]{.chpuri2}

    Singleton パターンはグローバル変数と同じように[、]{.chpule2}[
    ]{.chpuri2}プログラム内のどこからでもオブジェクトのアクセスを許します[。]{.chpule2}[
    ]{.chpuri2}しかし[、]{.chpule2}[
    ]{.chpuri2}そのインスタンスが他のコードによって変更されるのを防止することもします[。]{.chpule2}[
    ]{.chpuri2}

    この問題には[、]{.chpule2}[
    ]{.chpuri2}もう一つの側面があります[。]{.chpule2}[
    ]{.chpuri2}問題点その 1
    を解くコードをプログラム全体にまき散らかすのを避けるということです[。]{.chpule2}[
    ]{.chpuri2}特に[、]{.chpule2}[
    ]{.chpuri2}あなたのコードの残りの部分がすでにそのクラスに依存している場合[、]{.chpule2}[
    ]{.chpuri2}一つのクラス内にまとめておく方がはるかに良いです[。]{.chpule2}[
    ]{.chpuri2}

現在 Singleton パターンは非常に普及しているため[、]{.chpule2}[
]{.chpuri2}上記の問題点の一つだけを解決する何らかの技法のことが[、]{.chpule2}[
]{.chpuri2}*[シ]{.cjk}[ン]{.cjk}[グ]{.cjk}[ル]{.cjk}[ト]{.cjk}[ン]{.cjk}*と呼ばれることもあります[。]{.chpule2}[
]{.chpuri2}
:::

::: {.section .solution}
##  解決策 {#solution}

Singleton のすべての実装に共通した[、]{.chpule2}[
]{.chpuri2}二つのステップがあります[：]{.chpule2}[ ]{.chpuri2}

-   他のオブジェクトがシングルトンのクラスの `new`
    演算子を使用しないように[、]{.chpule2}[
    ]{.chpuri2}デフォルトのコンストラクターを非公開[
    ]{.chpule1}[（]{.chpuri1}private[）]{.chpule2}[
    ]{.chpuri2}にします[。]{.chpule2}[ ]{.chpuri2}
-   コンストラクターとして機能する静的[
    ]{.chpule1}[（]{.chpuri1}static[）]{.chpule2}[
    ]{.chpuri2}作成メソッドを作成します[。]{.chpule2}[
    ]{.chpuri2}内部では[、]{.chpule2}[
    ]{.chpuri2}このメソッドは[、]{.chpule2}[
    ]{.chpuri2}非公開のコンストラクターを呼び出してオブジェクトを生成し[、]{.chpule2}[
    ]{.chpuri2}静的なフィールドに保存します[。]{.chpule2}[
    ]{.chpuri2}次回以降のこのメソッドへの呼び出しはすべて[、]{.chpule2}[
    ]{.chpuri2}キャッシュされたオブジェクトを返します[。]{.chpule2}[
    ]{.chpuri2}

コードがこのシングルトン・クラスにアクセスできれば[、]{.chpule2}[
]{.chpuri2}シングルトンの静的メソッドを呼び出すことができます[。]{.chpule2}[
]{.chpuri2}そのため[、]{.chpule2}[
]{.chpuri2}そのメソッドが呼び出されるたびに[、]{.chpule2}[
]{.chpuri2}常に同じオブジェクトが返されます[。]{.chpule2}[ ]{.chpuri2}
:::

::: {.section .analogy}
##  現実世界でのたとえ {#analogy}

政府はシングルトンの優れた例です[。]{.chpule2}[
]{.chpuri2}一つの国には[、]{.chpule2}[
]{.chpuri2}一つの公式な政府しかありません[。]{.chpule2}[
]{.chpuri2}政府を構成する人々がどうであれ[[、]{.ls-1}]{.chpule2}[
]{.chpuri2}​[ ]{.chpule1}[「]{.chpuri1}X 国政府[」]{.chpule2}[
]{.chpuri2}は[、]{.chpule2}[
]{.chpuri2}政権を掌握した人々の集団を指す[、]{.chpule2}[
]{.chpuri2}グローバルなアクセス・ポイントとなります[。]{.chpule2}[
]{.chpuri2}
:::

::::: {.section .structure-container}
##  構造 {#structure}

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/singleton/structure-ja29d5.png?id=d995dfe821710682109dc58fd4ce2dbb"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.48;"
srcset="/images/patterns/diagrams/singleton/structure-ja.png?id=d995dfe821710682109dc58fd4ce2dbb 430w,/images/patterns/diagrams/singleton/structure-ja-1.5x.png?id=605e1083b2a0ec2e403fe304c999781a 645w,/images/patterns/diagrams/singleton/structure-ja-2x.png?id=42fe364b72372d7663a9409922553c87 860w"
sizes="(max-width: 720px) 100vw, 430px" loading="lazy" width="430"
alt="Singleton パターンの構造" /><img
src="../../images/patterns/diagrams/singleton/structure-ja-indexedea74.png?id=d526e1941f11b4d4513dbf20e71c47ef"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.48;"
srcset="/images/patterns/diagrams/singleton/structure-ja-indexed.png?id=d526e1941f11b4d4513dbf20e71c47ef 430w,/images/patterns/diagrams/singleton/structure-ja-indexed-1.5x.png?id=4029d39a183a1edf53f8a4405e21ee33 645w,/images/patterns/diagrams/singleton/structure-ja-indexed-2x.png?id=35111e6210d34073ce730aeafc4b239e 860w"
sizes="(max-width: 720px) 100vw, 430px" loading="lazy" width="430"
alt="Singleton パターンの構造" />
</figure>
:::

1.  **シングルトン**[ ]{.chpule1}[（]{.chpuri1}Singleton[）]{.chpule2}[
    ]{.chpuri2}クラスは[、]{.chpule2}[
    ]{.chpuri2}そのクラスの同じインスタンスを返す静的メソッド
    `get­Instance` を宣言します[。]{.chpule2}[ ]{.chpuri2}

    シングルトンのコンストラクターは[、]{.chpule2}[
    ]{.chpuri2}クライアント・コードから隠れている必要があります[。]{.chpule2}[
    ]{.chpuri2}`get­Instance` メソッドの呼び出しが[、]{.chpule2}[
    ]{.chpuri2}シングルトンのオブジェクトを取得する唯一の方法でなければなりません[。]{.chpule2}[
    ]{.chpuri2}
::::
:::::

::: {.section .pseudocode}
##  擬似コード {#pseudocode}

この例では[、]{.chpule2}[
]{.chpuri2}データベース接続クラスが**シングルトン**として機能します[。]{.chpule2}[
]{.chpuri2}このクラスには[、]{.chpule2}[
]{.chpuri2}公開コンストラクターがありません[。]{.chpule2}[
]{.chpuri2}このクラスのオブジェクトを取得する唯一の方法が[、]{.chpule2}[
]{.chpuri2}`get­Instance`
メソッドを呼び出すことにするためです[。]{.chpule2}[
]{.chpuri2}このメソッドは[、]{.chpule2}[
]{.chpuri2}最初に作成されたオブジェクトをキャッシュし[、]{.chpule2}[
]{.chpuri2}その後の呼び出しでは[、]{.chpule2}[
]{.chpuri2}それを返します[。]{.chpule2}[ ]{.chpuri2}

<figure class="code">
<pre class="code" lang="pseudocode"><code>// Database クラスには、getInstance メソッドがあり、クライアントがプロ
// グラム全体を通してデータベース接続の同じインスタンスにアクセスすること
// を可能とする。
class Database is
    // シングルトンのインスタンスを格納するためのフィールドは static と
    // 宣言されるべき。
    private static field instance: Database

    // new 演算子を使った直接の構築を防ぐため、シングルトンのコンストラク
    // ターは、private と宣言されるべき。
    private constructor Database() is
        // データベース・サーバーへの実際の接続等の初期化コード。
        // ……

    // シングルトン・インスタンスへのアクセスを管理するための静的メソッド。
    public static method getInstance() is
        if (Database.instance == null) then
            acquireThreadLock() and then
                // ロックのリリースを待っている間に他のスレッドにより本
                // インスタンスが初期化されていないことの確認。
                if (Database.instance == null) then
                    Database.instance = new Database()
        return Database.instance

    // 最後に、シングルトンはインスタンス上で実行できる何らかのビジネス・
    // ロジックを定義する必要あり。
    public method query(sql) is
        // たとえば、アプリのすべてのデータベース・クエリはこのメソッドを
        // 通して行う。したがって、ここにスロットリングやキャッシュ処理の
        // ためのロジックを置くことが可能。
        // ……

class Application is
    method main() is
        Database foo = Database.getInstance()
        foo.query(&quot;SELECT ……&quot;)
        // ……
        Database bar = Database.getInstance()
        bar.query(&quot;SELECT ……&quot;)
        // 変数 bar は、変数 foo と同じオブジェクトを保持。</code></pre>
</figure>
:::

:::::::: {.section .applicability-container}
##  適応性 {#applicability}

::::::: applicability
::: applicability-problem
プログラム内のあるクラスがすべてのクライアントで利用可能な単一のインスタンスを持つ必要がある場合に[、]{.chpule2}[
]{.chpuri2}Singleton パターンを使用します[。]{.chpule2}[
]{.chpuri2}たとえば[、]{.chpule2}[
]{.chpuri2}プログラムの異なる部分で共有されている単一のデータベース・オブジェクトです[。]{.chpule2}[
]{.chpuri2}
:::

::: applicability-solution
Singleton パターンは[、]{.chpule2}[
]{.chpuri2}特別な作成メソッドを除いて[、]{.chpule2}[
]{.chpuri2}クラスのオブジェクトを作成する他のすべての方法を無効にします[。]{.chpule2}[
]{.chpuri2}このメソッドは[、]{.chpule2}[
]{.chpuri2}新しいオブジェクトを作成するか[、]{.chpule2}[
]{.chpuri2}すでに作成されている場合は既存のオブジェクトを返します[。]{.chpule2}[
]{.chpuri2}
:::

::: applicability-problem
グローバル変数をより厳密に管理する必要がある場合は[、]{.chpule2}[
]{.chpuri2}Singleton パターンを使用してください[。]{.chpule2}[
]{.chpuri2}
:::

::: applicability-solution
グローバル変数とは異なり[、]{.chpule2}[ ]{.chpuri2}Singleton
パターンはクラスのインスタンスが一つだけであることを保証します[。]{.chpule2}[
]{.chpuri2}Singleton クラス自体を除くいかなるものも[、]{.chpule2}[
]{.chpuri2}キャッシュされたインスタンスを置き換えることはできません[。]{.chpule2}[
]{.chpuri2}

この制限を調整して[、]{.chpule2}[
]{.chpuri2}任意個のシングルトンのインスタンスを作成するようにすることは[、]{.chpule2}[
]{.chpuri2}いつでも可能です[。]{.chpule2}[
]{.chpuri2}変更が必要なコードは[、]{.chpule2}[ ]{.chpuri2}`get­Instance`
メソッドの本体だけです[。]{.chpule2}[ ]{.chpuri2}
:::
:::::::
::::::::

::: {.section .checklist}
##  実装方法 {#checklist}

1.  クラスにシングルトンのインスタンスを格納する非公開[
    ]{.chpule1}[（]{.chpuri1}private[）]{.chpule2}[ ]{.chpuri2}の静的[
    ]{.chpule1}[（]{.chpuri1}static[）]{.chpule2}[
    ]{.chpuri2}フィールドを追加します[。]{.chpule2}[ ]{.chpuri2}

2.  シングルトンのインスタンスを取得するための公開[
    ]{.chpule1}[（]{.chpuri1}public[）]{.chpule2}[ ]{.chpuri2}静的[
    ]{.chpule1}[（]{.chpuri1}static[）]{.chpule2}[
    ]{.chpuri2}作成用メソッドを宣言します[。]{.chpule2}[ ]{.chpuri2}

3.  静的メソッド内に[、]{.chpule2}[
    ]{.chpuri2}遅延初期化コードを実装します[。]{.chpule2}[
    ]{.chpuri2}初回の呼び出し時だけ新規オブジェクトを作成し[、]{.chpule2}[
    ]{.chpuri2}それを静的フィールドに格納します[。]{.chpule2}[
    ]{.chpuri2}その後の呼び出しでは[、]{.chpule2}[
    ]{.chpuri2}メソッドは常にそのインスタンスを返すようにします[。]{.chpule2}[
    ]{.chpuri2}

4.  コンストラクターを非公開[
    ]{.chpule1}[（]{.chpuri1}private[）]{.chpule2}[
    ]{.chpuri2}とします[。]{.chpule2}[
    ]{.chpuri2}クラス中の静的メソッドからはこのコンストラクターを呼べますが[、]{.chpule2}[
    ]{.chpuri2}他のオブジェクトからは呼べません[。]{.chpule2}[
    ]{.chpuri2}

5.  クライアントのコード中のコンストラクター呼び出しのすべてを静的生成用メソッドの呼び出しと置き換えます[。]{.chpule2}[
    ]{.chpuri2}
:::

:::::: {.section .pros-cons}
##  長所と短所 {#pros-cons}

::::: row
::: col-sm-6
-    クラスにはインスタンスが一つしかないことが保証可能[。]{.chpule2}[
    ]{.chpuri2}
-    そのインスタンスへの大域アクセス・ポイントが得られる[。]{.chpule2}[
    ]{.chpuri2}
-    シングルトンのオブジェクトは[、]{.chpule2}[
    ]{.chpuri2}初回要求時にのみ初期化[。]{.chpule2}[ ]{.chpuri2}
:::

::: col-sm-6
-   
    *[単]{.cjk}[一]{.cjk}[責]{.cjk}[任]{.cjk}[の]{.cjk}[原]{.cjk}[則]{.cjk}*に違反[。]{.chpule2}[
    ]{.chpuri2}このパターンは[、]{.chpule2}[
    ]{.chpuri2}二つの問題点を同時に解決しようとするため[。]{.chpule2}[
    ]{.chpuri2}
-    Singleton パターンは[、]{.chpule2}[
    ]{.chpuri2}設計上の欠陥を隠蔽[。]{.chpule2}[
    ]{.chpuri2}たとえば[、]{.chpule2}[
    ]{.chpuri2}プログラム中のコンポーネント同士が互いの詳細を知りすぎるなど[。]{.chpule2}[
    ]{.chpuri2}
-    このパターンの使用にあたっては[、]{.chpule2}[
    ]{.chpuri2}マルチスレッド環境において[、]{.chpule2}[
    ]{.chpuri2}複数のスレッドがシングルトン・オブジェクトを複数回生成しないように特別な処理が必要[。]{.chpule2}[
    ]{.chpuri2}
-   
    多くのテスト・フレームワークがモック・オブジェクトの生成において継承に依存しているため[、]{.chpule2}[
    ]{.chpuri2}シングルトンのクライアント・コードは[、]{.chpule2}[
    ]{.chpuri2}ユニット・テストが困難[。]{.chpule2}[
    ]{.chpuri2}シングルトンのクラスのコンストラクターは非公開のため[、]{.chpule2}[
    ]{.chpuri2}ほとんどの言語で静的メソッドを上書きすることが不可能[。]{.chpule2}[
    ]{.chpuri2}シングルトンのモックを行うには[、]{.chpule2}[
    ]{.chpuri2}巧妙な方法を考える必要あり[。]{.chpule2}[
    ]{.chpuri2}またはテストを放棄[。]{.chpule2}[ ]{.chpuri2}または
    Singleton パターンの使用をあきらめる[。]{.chpule2}[ ]{.chpuri2}
:::
:::::
::::::

::: {.section .relations}
##  他のパターンとの関係 {#relations}

-   多くの場合[、]{.chpule2}[
    ]{.chpuri2}ファサード・オブジェクトは一つだけあれば十分なので[、]{.chpule2}[
    ]{.chpuri2}[Facade](facade.html) はしばしば
    [Singleton](singleton.html) に変換可能です[。]{.chpule2}[
    ]{.chpuri2}

-   [Flyweight](flyweight.html) で[、]{.chpule2}[
    ]{.chpuri2}共有状態の全部を一つのフライウェイト・オブジェクトに何らかの方法で削減できた場合[、]{.chpule2}[
    ]{.chpuri2}それは [Singleton](singleton.html)
    に似たものになります[。]{.chpule2}[ ]{.chpuri2}しかし[、]{.chpule2}[
    ]{.chpuri2}この二つのパターンには[、]{.chpule2}[
    ]{.chpuri2}根本的な違いが二箇所あります[。]{.chpule2}[ ]{.chpuri2}

    1.  Singleton のインスタンスは一つだけですが[、]{.chpule2}[
        ]{.chpuri2}*Flyweight* クラスは[、]{.chpule2}[
        ]{.chpuri2}異なる内因的状態を持つ複数のインスタンスがある可能性があります[。]{.chpule2}[
        ]{.chpuri2}
    2.  *Singleton*
        オブジェクトは変更可能かもしれませんが[、]{.chpule2}[
        ]{.chpuri2}Flyweight のオブジェクトは不変です[。]{.chpule2}[
        ]{.chpuri2}

-   [Abstract Factories](abstract-factory.html)[、]{.chpule2}[
    ]{.chpuri2}[Builders](builder.html)[、]{.chpule2}[
    ]{.chpuri2}[Prototypes](prototype.html) はどれも
    [Singletons](singleton.html) で実装可能です[。]{.chpule2}[
    ]{.chpuri2}
:::

::: {.section .implementations}
##  コード例 {#implementations}

[![Singleton を C#
で](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](singleton/csharp/example.html "Singleton を C# で"){.prog-lang-link}
[![Singleton を C++
で](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](singleton/cpp/example.html "Singleton を C++ で"){.prog-lang-link}
[![Singleton を Go
で](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](singleton/go/example.html "Singleton を Go で"){.prog-lang-link}
[![Singleton を Java
で](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](singleton/java/example.html "Singleton を Java で"){.prog-lang-link}
[![Singleton を PHP
で](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](singleton/php/example.html "Singleton を PHP で"){.prog-lang-link}
[![Singleton を Python
で](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](singleton/python/example.html "Singleton を Python で"){.prog-lang-link}
[![Singleton を Ruby
で](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](singleton/ruby/example.html "Singleton を Ruby で"){.prog-lang-link}
[![Singleton を Rust
で](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](singleton/rust/example.html "Singleton を Rust で"){.prog-lang-link}
[![Singleton を Swift
で](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](singleton/swift/example.html "Singleton を Swift で"){.prog-lang-link}
[![Singleton を TypeScript
で](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](singleton/typescript/example.html "Singleton を TypeScript で"){.prog-lang-link}
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

[構造に関するパターン []{.fa
.fa-arrow-right}](structural-patterns.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### 戻る

[[]{.fa .fa-arrow-left} Prototype](prototype.html){.btn .btn-default
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
