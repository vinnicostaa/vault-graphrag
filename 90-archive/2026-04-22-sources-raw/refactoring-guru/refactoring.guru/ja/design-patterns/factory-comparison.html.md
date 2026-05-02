:::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::: {.main-content-container .center-content-container}
:::::: {.page .text}
::: breadcrumb
[](../../index.html){.home} /
[デザインパターン](../design-patterns.html){.type} /
[生成に関するパターン](creational-patterns.html){.type}
:::

# ファクトリーの比較 {#ファクトリーの比較 .title}

この記事では[、]{.chpule2}[ ]{.chpuri2}以下を比較し[、]{.chpule2}[
]{.chpuri2}違いを明らかにします[：]{.chpule2}[ ]{.chpuri2}

1.  ファクトリー
2.  生成メソッド
3.  静的生成[ ]{.chpule1}[（]{.chpuri1}つまりファクトリー[）]{.chpule2}[
    ]{.chpuri2}メソッド
4.  単純ファクトリー
5.  [Factory Method](factory-method.html) パターン
6.  [Abstract Factory](abstract-factory.html) パターン

これらの用語は[、]{.chpule2}[
]{.chpuri2}ウェブ上至る所で使われているのが見つけられます[。]{.chpule2}[
]{.chpuri2}似たように見えますが[、]{.chpule2}[
]{.chpuri2}それぞれ違う意味を持っています[。]{.chpule2}[
]{.chpuri2}多くの人々はそのことを知らず[、]{.chpule2}[
]{.chpuri2}混乱や誤解につながっています[。]{.chpule2}[ ]{.chpuri2}

ということで[、]{.chpule2}[ ]{.chpuri2}違いを理解し[、]{.chpule2}[
]{.chpuri2}この問題を一気に片付けてみましょう[。]{.chpule2}[ ]{.chpuri2}

## 1. ファクトリー

**ファクトリー**[ ]{.chpule1}[（]{.chpuri1}工場[）]{.chpule2}[
]{.chpuri2}は[、]{.chpule2}[
]{.chpuri2}何かを生み出すことになっている関数[、]{.chpule2}[
]{.chpuri2}メソッド[、]{.chpule2}[
]{.chpuri2}クラスを指す[、]{.chpule2}[
]{.chpuri2}曖昧な用語です[。]{.chpule2}[
]{.chpuri2}最も一般的な使われ方として[、]{.chpule2}[
]{.chpuri2}ファクトリーはオブジェクトを生み出します[。]{.chpule2}[
]{.chpuri2}しかしファクトリーは[、]{.chpule2}[
]{.chpuri2}ファイルやデータベースのレコードなども生み出すかもしれません[。]{.chpule2}[
]{.chpuri2}

たとえば[、]{.chpule2}[ ]{.chpuri2}くだけた場面では[、]{.chpule2}[
]{.chpuri2}以下のいずれも[
]{.chpule1}[「]{.chpuri1}ファクトリー[」]{.chpule2}[
]{.chpuri2}と呼ばれるかもしれません[。]{.chpule2}[ ]{.chpuri2}

-   プログラムの GUI を作成する関数またはメソッド
-   ユーザーを作成するクラス
-   クラスのコンストラクターをある特定の方法で呼び出す静的メソッド
-   生成に関するデザインパターンの一つ

通常[、]{.chpule2}[ ]{.chpuri2}誰かが[
]{.chpule1}[「]{.chpuri1}ファクトリー[」]{.chpule2}[
]{.chpuri2}という言葉を使った時[、]{.chpule2}[
]{.chpuri2}何を意味しているかは[、]{.chpule2}[
]{.chpuri2}文脈から明らかなはずです[。]{.chpule2}[
]{.chpuri2}でも疑問に思ったら尋ねていてください[。]{.chpule2}[
]{.chpuri2}単にその人がよくわかっていないだけかもしれません[。]{.chpule2}[
]{.chpuri2}

## 2. 生成メソッド

**生成メソッド**[ ]{.chpule1}[（]{.chpuri1}creation
method[）]{.chpule2}[ ]{.chpuri2}は[[、]{.ls-1}]{.chpule2}[ ]{.chpuri2}​[
]{.chpule1}[『]{.chpuri1}[パターン指向リファクタリング入門](https://www.amazon.co.jp/-/dp/4822282384)[』]{.chpule2}[
]{.chpuri2}の中で[[、]{.ls-1}]{.chpule2}[ ]{.chpuri2}​[
]{.chpule1}[「]{.chpuri1}オブジェクトを生成するメソッド[」]{.chpule2}[
]{.chpuri2}と定義されています[。]{.chpule2}[
]{.chpuri2}この意味するところはつまり[、]{.chpule2}[ ]{.chpuri2}Factory
Method パターンの結果は[、]{.chpule2}[
]{.chpuri2}生成メソッドですが[、]{.chpule2}[
]{.chpuri2}逆は必ずしも真ではない[、]{.chpule2}[
]{.chpuri2}ということです[。]{.chpule2}[
]{.chpuri2}また[[、]{.ls-1}]{.chpule2}[ ]{.chpuri2}​[
]{.chpule1}[「]{.chpuri1}生成メソッド[」]{.chpule2}[
]{.chpuri2}の用語は[、]{.chpule2}[ ]{.chpuri2}Martin Fowler[
]{.chpule1}[（]{.chpuri1}マーティン・ファウラー[）]{.chpule2}[
]{.chpuri2}氏が[
]{.chpule1}[『]{.chpuri1}[リファクタリング[：]{.chpule2}[
]{.chpuri2}既存のコードを安全に改善する](https://www.amazon.co.jp/-/dp/4274224546)[』]{.chpule2}[
]{.chpuri2}の中で言う[
]{.chpule1}[「]{.chpuri1}ファクトリー・メソッド[」]{.chpule2}[
]{.chpuri2}という用語[、]{.chpule2}[ ]{.chpuri2}Joshua Bloch[
]{.chpule1}[（]{.chpuri1}ジョシュア・ブロック[）]{.chpule2}[
]{.chpuri2}氏が[ ]{.chpule1}[『]{.chpuri1}[Effective
Java](https://www.amazon.com/-/dp/0134685997)[[』]{.ls-1}]{.chpule2}[
]{.chpuri2}​[ ]{.chpule1}[（]{.chpuri1}邦題同[）]{.chpule2}[
]{.chpuri2}の中で言う[
]{.chpule1}[「]{.chpuri1}静的ファクトリー・メソッド[」]{.chpule2}[
]{.chpuri2}という用語と置き換え可能です[。]{.chpule2}[ ]{.chpuri2}

実際のところ[、]{.chpule2}[
]{.chpuri2}生成メソッドはコンストラクター呼び出しのラッパーにすぎません[。]{.chpule2}[
]{.chpuri2}単に意図をより良く言い表せる名前とも言えます[。]{.chpule2}[
]{.chpuri2}一方で[、]{.chpule2}[
]{.chpuri2}コンストラクターの変更からコードを隔離する一助になるかもしれません[。]{.chpule2}[
]{.chpuri2}あるいは新規オブジェクトを作成する代わりに[、]{.chpule2}[
]{.chpuri2}既存のものを返すなどの特定のロジックを含ませることすらできます[。]{.chpule2}[
]{.chpuri2}

新しいオブジェクトを生み出すからという理由だけで[、]{.chpule2}[
]{.chpuri2}多くの人たちがそのようなメソッドのことを[
]{.chpule1}[「]{.chpuri1}ファクトリー・メソッド[」]{.chpule2}[
]{.chpuri2}と呼びます[。]{.chpule2}[
]{.chpuri2}メソッドはオブジェクトを作成し[、]{.chpule2}[
]{.chpuri2}すべての*[フ]{.cjk}[ァ]{.cjk}[ク]{.cjk}[ト]{.cjk}[リ]{.cjk}[ー]{.cjk}*はオブジェクトを作成するので[、]{.chpule2}[
]{.chpuri2}これは明らかに*[フ]{.cjk}[ァ]{.cjk}[ク]{.cjk}[ト]{.cjk}[リ]{.cjk}[ー]{.cjk}[・]{.cjk}[メ]{.cjk}[ソ]{.cjk}[ッ]{.cjk}[ド]{.cjk}*だ[、]{.chpule2}[
]{.chpuri2}という単純な論理に基づいています[。]{.chpule2}[
]{.chpuri2}当然のことながら[、]{.chpule2}[ ]{.chpuri2}本物の [Factory
Method](factory-method.html) パターンのことを話す時に[、]{.chpule2}[
]{.chpuri2}大混乱となります[。]{.chpule2}[ ]{.chpuri2}

下記の例では[、]{.chpule2}[ ]{.chpuri2}`next` は[、]{.chpule2}[
]{.chpuri2}生成メソッドです[。]{.chpule2}[ ]{.chpuri2}

<figure class="code">
<pre class="code" lang="php"><code>class Number {
    private $value;

    public function __construct($value) {
        $this-&gt;value = $value;
    }

    public function next() {
        return new Number ($this-&gt;value + 1);
    }
}</code></pre>
</figure>

## 3. 静的生成メソッド

**静的生成メソッド**[ ]{.chpule1}[（]{.chpuri1}static creation
method[）]{.chpule2}[ ]{.chpuri2}は[、]{.chpule2}[ ]{.chpuri2}`static`
と宣言された生成メソッドです[。]{.chpule2}[
]{.chpuri2}言い換えると[、]{.chpule2}[
]{.chpuri2}クラスに対して呼び出すことができ[、]{.chpule2}[
]{.chpuri2}呼び出しに際してオブジェクトを必要としません[。]{.chpule2}[
]{.chpuri2}

この種のメソッドのことを誰かが[
]{.chpule1}[「]{.chpuri1}静的ファクトリー・メソッド[」]{.chpule2}[
]{.chpuri2}と呼んだからと言って驚かないでください[。]{.chpule2}[
]{.chpuri2}ただの習慣です[。]{.chpule2}[ ]{.chpuri2}[Factory
Method](factory-method.html) は[、]{.chpule2}[
]{.chpuri2}継承に頼ったデザインパターンです[。]{.chpule2}[
]{.chpuri2}それを `static` としてしまうと[、]{.chpule2}[
]{.chpuri2}サブクラスではもう拡張できず[、]{.chpule2}[
]{.chpuri2}このパターンの目的に反したものとなります[。]{.chpule2}[
]{.chpuri2}

静的生成メソッドが新規オブジェクトを返すなら[、]{.chpule2}[
]{.chpuri2}コンストラクターの代替となります[。]{.chpule2}[ ]{.chpuri2}

以下の場合に便利かもしれません[：]{.chpule2}[ ]{.chpuri2}

-   異なる目的のために異なるコンストラクターがいくつか必要だが[、]{.chpule2}[
    ]{.chpuri2}コンストラクターのシグネチャーが一致してしまう時[。]{.chpule2}[
    ]{.chpuri2}たとえば[、]{.chpule2}[ ]{.chpuri2}`Random­(int max)` と
    `Random­(int min)` の両方を持つことは[、]{.chpule2}[
    ]{.chpuri2}Java[、]{.chpule2}[ ]{.chpuri2}C++[、]{.chpule2}[
    ]{.chpuri2}C#[、]{.chpule2}[
    ]{.chpuri2}そして他の多くの言語では不可能です[。]{.chpule2}[
    ]{.chpuri2}この問題点の回避方法として一番よく使われる方法は[、]{.chpule2}[
    ]{.chpuri2}デフォルト・コンストラクターを呼んで[、]{.chpule2}[
    ]{.chpuri2}その後適切な値を設定する静的メソッドをいくつか作成することです[。]{.chpule2}[
    ]{.chpuri2}

-   新規オブジェクトを作成する代わりに[、]{.chpule2}[
    ]{.chpuri2}既存のオブジェクトを再利用したい時[[。]{.ls-1}]{.chpule2}[
    ]{.chpuri2}​[
    ]{.chpule1}[（]{.chpuri1}\[[Singleton](singleton.html)\]
    パターンをご覧ください[。]{.ls-05}[）]{.chpule2}[
    ]{.chpuri2}ほとんどのプログラミング言語では[、]{.chpule2}[
    ]{.chpuri2}コンストラクターはクラスの新インスタンスを返さなければなりません[。]{.chpule2}[
    ]{.chpuri2}静的作成メソッドは[、]{.chpule2}[
    ]{.chpuri2}この制限の回避策です[。]{.chpule2}[
    ]{.chpuri2}静的メソッド内部では[、]{.chpule2}[
    ]{.chpuri2}コンストラクターを呼び出して新鮮なインスタンスを作成すべきか[、]{.chpule2}[
    ]{.chpuri2}キャッシュにある既存オブジェクトを返すべきかを[、]{.chpule2}[
    ]{.chpuri2}自分のコードが決めます[。]{.chpule2}[ ]{.chpuri2}

下記の例では[、]{.chpule2}[ ]{.chpuri2}`load` メソッドは[、]{.chpule2}[
]{.chpuri2}静的生成メソッドで[、]{.chpule2}[
]{.chpuri2}ユーザーをデータベースから取得する便利な方法を提供します[。]{.chpule2}[
]{.chpuri2}

<figure class="code">
<pre class="code" lang="php"><code>class User {
    private $id, $name, $email, $phone;

    public function __construct($id, $name, $email, $phone) {
        $this-&gt;id = $id;
        $this-&gt;name = $name;
        $this-&gt;email = $email;
        $this-&gt;phone = $phone;
    }

    public static function load($id) {
        list($id, $name, $email, $phone) = DB::load_data(&#39;users&#39;, &#39;id&#39;, &#39;name&#39;, &#39;email&#39;, &#39;phone&#39;);
        $user = new User($id, $name, $email, $phone);
        return $user;
    }
}</code></pre>
</figure>

## 4. *[単]{.cjk}[純]{.cjk}[フ]{.cjk}[ァ]{.cjk}[ク]{.cjk}[ト]{.cjk}[リ]{.cjk}[ー]{.cjk}*・パターン

**単純ファクトリー**[ ]{.chpule1}[（]{.chpuri1}simple
factory[）]{.chpule2}[ ]{.chpuri2}パターン []{.pop-i}[書籍[
]{.chpule1}[『]{.chpuri1}[Head First
デザインパターン](https://www.amazon.co.jp/-/dp/4873119766)[』]{.chpule2}[
]{.chpuri2}中で定義]{.pop-content}は[、]{.chpule2}[
]{.chpuri2}巨大な条件分岐のついた生成メソッドを持つクラスのことです[。]{.chpule2}[
]{.chpuri2}メソッドのパラメーターに基づき[、]{.chpule2}[
]{.chpuri2}どのプロダクトのクラスをインスタンス化すべきかを決め[、]{.chpule2}[
]{.chpuri2}生成し[、]{.chpule2}[ ]{.chpuri2}返します[。]{.chpule2}[
]{.chpuri2}

人々は通常[、]{.chpule2}[
]{.chpuri2}*[単]{.cjk}[純]{.cjk}[フ]{.cjk}[ァ]{.cjk}[ク]{.cjk}[ト]{.cjk}[リ]{.cjk}[ー]{.cjk}*を一般的なファクトリーか[、]{.chpule2}[
]{.chpuri2}あるいは生成に関するデザインパターンのうちの一つと勘違いします[。]{.chpule2}[
]{.chpuri2}大抵の場合[、]{.chpule2}[
]{.chpuri2}単純ファクトリーは[、]{.chpule2}[ ]{.chpuri2}[Factory
Method](factory-method.html) パターンか [Abstract
Factory](abstract-factory.html)
パターンを導入する時の中間ステップです[。]{.chpule2}[ ]{.chpuri2}

単純ファクトリーは[、]{.chpule2}[
]{.chpuri2}一つのクラス中の一つのメソッドとして表現されます[。]{.chpule2}[
]{.chpuri2}時間が経過するにつれて[、]{.chpule2}[
]{.chpuri2}このメソッドは大きくなりすぎ[、]{.chpule2}[
]{.chpuri2}メソッドの一部をサブクラスに抽出すべきかどうか検討を始めます[。]{.chpule2}[
]{.chpuri2}そして抽出を何回か繰り返すと[、]{.chpule2}[
]{.chpuri2}典型的な *Factory Method*
パターンになっていることに気が付くでしょう[。]{.chpule2}[ ]{.chpuri2}

ちなみに[、]{.chpule2}[ ]{.chpuri2}単純ファクトリーを `abstract`
と宣言さえすれば[、]{.chpule2}[ ]{.chpuri2}魔法のように *Abstract
Factory* パターンになるわけではありません[。]{.chpule2}[ ]{.chpuri2}

単純ファクトリーの例です[：]{.chpule2}[ ]{.chpuri2}

<figure class="code">
<pre class="code" lang="php"><code>class UserFactory {
    public static function create($type) {
        switch ($type) {
            case &#39;user&#39;: return new User();
            case &#39;customer&#39;: return new Customer();
            case &#39;admin&#39;: return new Admin();
            default:
                throw new Exception(&#39;Wrong user type passed.&#39;);
        }
    }
}</code></pre>
</figure>

## 5. *Factory Method* パターン

**Factory Method** []{.pop-i}[GoF 本 [
]{.chpule1}[『]{.chpuri1}[オブジェクト指向における再利用のためのデザインパターン](https://www.amazon.co.jp/-/dp/4797311126)[』]{.chpule2}[
]{.chpuri2}中で定義]{.pop-content} は[、]{.chpule2}[
]{.chpuri2}生成に関するデザインパターンの一つで[、]{.chpule2}[
]{.chpuri2}生成するオブジェクトのインターフェースを宣言し[、]{.chpule2}[
]{.chpuri2}サブクラスで作成されるオブジェクトの型を自由に変更できるようにします[。]{.chpule2}[
]{.chpuri2}

もし基底クラスに生成メソッドがあり[、]{.chpule2}[
]{.chpuri2}それをサブクラスで拡張していたら[、]{.chpule2}[
]{.chpuri2}Factory Method が使われているのかもしれません[。]{.chpule2}[
]{.chpuri2}

<figure class="code">
<pre class="code" lang="php"><code>abstract class Department {
    public abstract function createEmployee($id);

    public function fire($id) {
        $employee = $this-&gt;createEmployee($id);
        $employee-&gt;paySalary();
        $employee-&gt;dismiss();
    }
}

class ITDepartment extends Department {
    public function createEmployee($id) {
        return new Programmer($id);
    }
}

class AccountingDepartment extends Department {
    public function createEmployee($id) {
        return new Accountant($id);
    }
}</code></pre>
</figure>

## 6. *Abstract Factory* パターン

**Abstract Factory** []{.pop-i}[これも[GoF
本](https://www.amazon.co.jp/-/dp/4797311126)中で定義]{.pop-content}
は[、]{.chpule2}[
]{.chpuri2}生成に関するデザインパターンの一つで[、]{.chpule2}[
]{.chpuri2}関連あるいは依存関係にあるオブジェクトのファミリーを[、]{.chpule2}[
]{.chpuri2}具象クラスを指定せずに可能にします[。]{.chpule2}[ ]{.chpuri2}

[ ]{.chpule1}[「]{.chpuri1}オブジェトのファミリー[」]{.chpule2}[
]{.chpuri2}って何だ[、]{.chpule2}[ ]{.chpuri2}ですって[？]{.chpule2}[
]{.chpuri2}たとえば[、]{.chpule2}[
]{.chpuri2}このクラスの集まりを考えてみてください[：]{.chpule2}[
]{.chpuri2}`輸送手段`＋`エンジン`＋`制御方法`[。]{.chpule2}[
]{.chpuri2}これらには[、]{.chpule2}[
]{.chpuri2}以下のような[、]{.chpule2}[
]{.chpuri2}変種があるかもしれません[：]{.chpule2}[ ]{.chpuri2}

1.  `車`＋`燃焼エンジン`＋`ハンドル`
2.  `飛行機`＋`ジェットエンジン`＋`操縦桿`

あなたのプログラムにプロダクトのファミリーのようなものがないのであれば[、]{.chpule2}[
]{.chpuri2}Abstract Factory を使う必要はありません[。]{.chpule2}[
]{.chpuri2}

繰り返しますが[、]{.chpule2}[ ]{.chpuri2}多くの人が *Abstract Factory*
のことを[、]{.chpule2}[ ]{.chpuri2}`abstract`
と宣言された単純ファクトリーと勘違いしています[。]{.chpule2}[
]{.chpuri2}十分注意してください[。]{.chpule2}[ ]{.chpuri2}

### 後書き

違いがわかったところで[、]{.chpule2}[
]{.chpuri2}これらのデザインパターンについて改めて再読してみてください[：]{.chpule2}[
]{.chpuri2}

-   [Factory Method](factory-method.html)
-   [Abstract Factory](abstract-factory.html)

::: next
#### 次を読む

[Builder []{.fa .fa-arrow-right}](builder.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### 戻る

[[]{.fa .fa-arrow-left} Abstract Factory](abstract-factory.html){.btn
.btn-default rel="prev"}
:::
::::::
:::::::
::::::::
