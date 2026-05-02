:::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} /
[デザインパターン](../../../design-patterns.html){.type} /
[Iterator](../../iterator.html){.type} /
[TypeScript](../../typescript.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Iterator](../../../../images/patterns/cards/iterator-minie2c2.png?id=76c28bb48f997b36965983dd2b41f02e){srcset="/images/patterns/cards/iterator-mini-2x.png?id=7d28f66a487066774a743cfceaa40ea4 2x"}
:::

# **Iterator** を TypeScript で {#iterator-を-typescript-で .pattern-example-title-block-title}
::::

::::::::::: pattern-example-body
::: pattern-example-brief
**Iterator** は[、]{.chpule2}[
]{.chpuri2}振る舞いに関するデザインパターンの一つで[、]{.chpule2}[
]{.chpuri2}複雑なデータ構造の内部の詳細を公開することなく[、]{.chpule2}[
]{.chpuri2}順次横断的に探索することを可能とします[。]{.chpule2}[
]{.chpuri2}

Iterator のおかげで[、]{.chpule2}[
]{.chpuri2}クライアントは[、]{.chpule2}[
]{.chpuri2}異なるコレクション上の要素の探索を[、]{.chpule2}[
]{.chpuri2}単一のイテレーター・インターフェースを使用して同様の方法で行えます[。]{.chpule2}[
]{.chpuri2}
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Iterator の詳細 ](../../iterator.html){.btn .btn-lg .btn-primary}
:::

:::::::: {.sidebar-navigation .with-scroll}
::: en-title
ナビゲーション
:::

::: en-intro
 [はじめに](#)
:::

::: en-intro
 [概念的な例](#example-0)
:::

::: en-file
 [index](#example-0--index-ts)
:::

::: en-file
 [Output](#example-0--Output-txt)
:::
::::::::

**複雑度[：]{.chpule2}[ ]{.chpuri2}**[]{.fa-stars}

**人気度[：]{.chpule2}[ ]{.chpuri2}**[]{.fa-stars}

**使用例[：]{.chpule2}[ ]{.chpuri2}** このパターンは[、]{.chpule2}[
]{.chpuri2}TypeScript コードではよく見かけます[。]{.chpule2}[
]{.chpuri2}多くのフレームワークやライブラリーがこれを使用してコレクション上の探索の標準的方法を提供します[。]{.chpule2}[
]{.chpuri2}

**見つけ方[：]{.chpule2}[ ]{.chpuri2}** Iterator は[、]{.chpule2}[
]{.chpuri2}`next` や `previous`
などの操舵そうだ用メソッドの存在から簡単に識別できます[。]{.chpule2}[
]{.chpuri2}イテレーターを使ったクライアント・コードには[、]{.chpule2}[
]{.chpuri2}探索対象のコレクションへの直接のアクセスがないかもしれません[。]{.chpule2}[
]{.chpuri2}
:::::::::::

[]{#example-0}

## 概念的な例 {#example-0-title}

この例は[、]{.chpule2}[ ]{.chpuri2}**Iterator**
デザインパターンの構造を説明するためのものです[。]{.chpule2}[
]{.chpuri2}以下の質問に答えることを目的としています[：]{.chpule2}[
]{.chpuri2}

-   どういうクラスからできているか[？]{.chpule2}[ ]{.chpuri2}
-   それぞれのクラスの役割は[？]{.chpule2}[ ]{.chpuri2}
-   パターンの要素同士はどう関係しているのか[？]{.chpule2}[ ]{.chpuri2}

#### []{#example-0--index-ts .anchor} **index.ts:** 概念的な例

<figure class="code">
<pre class="code" lang="typescript"><code>/**
 * Iterator Design Pattern
 *
 * Intent: Lets you traverse elements of a collection without exposing its
 * underlying representation (list, stack, tree, etc.).
 */

interface Iterator&lt;T&gt; {
    // Return the current element.
    current(): T;

    // Return the current element and move forward to next element.
    next(): T;

    // Return the key of the current element.
    key(): number;

    // Checks if current position is valid.
    valid(): boolean;

    // Rewind the Iterator to the first element.
    rewind(): void;
}

interface Aggregator {
    // Retrieve an external iterator.
    getIterator(): Iterator&lt;string&gt;;
}

/**
 * Concrete Iterators implement various traversal algorithms. These classes
 * store the current traversal position at all times.
 */

class AlphabeticalOrderIterator implements Iterator&lt;string&gt; {
    private collection: WordsCollection;

    /**
     * Stores the current traversal position. An iterator may have a lot of
     * other fields for storing iteration state, especially when it is supposed
     * to work with a particular kind of collection.
     */
    private position: number = 0;

    /**
     * This variable indicates the traversal direction.
     */
    private reverse: boolean = false;

    constructor(collection: WordsCollection, reverse: boolean = false) {
        this.collection = collection;
        this.reverse = reverse;

        if (reverse) {
            this.position = collection.getCount() - 1;
        }
    }

    public rewind() {
        this.position = this.reverse ?
            this.collection.getCount() - 1 :
            0;
    }

    public current(): string {
        return this.collection.getItems()[this.position];
    }

    public key(): number {
        return this.position;
    }

    public next(): string {
        const item = this.collection.getItems()[this.position];
        this.position += this.reverse ? -1 : 1;
        return item;
    }

    public valid(): boolean {
        if (this.reverse) {
            return this.position &gt;= 0;
        }

        return this.position &lt; this.collection.getCount();
    }
}

/**
 * Concrete Collections provide one or several methods for retrieving fresh
 * iterator instances, compatible with the collection class.
 */
class WordsCollection implements Aggregator {
    private items: string[] = [];

    public getItems(): string[] {
        return this.items;
    }

    public getCount(): number {
        return this.items.length;
    }

    public addItem(item: string): void {
        this.items.push(item);
    }

    public getIterator(): Iterator&lt;string&gt; {
        return new AlphabeticalOrderIterator(this);
    }

    public getReverseIterator(): Iterator&lt;string&gt; {
        return new AlphabeticalOrderIterator(this, true);
    }
}

/**
 * The client code may or may not know about the Concrete Iterator or Collection
 * classes, depending on the level of indirection you want to keep in your
 * program.
 */
const collection = new WordsCollection();
collection.addItem(&#39;First&#39;);
collection.addItem(&#39;Second&#39;);
collection.addItem(&#39;Third&#39;);

const iterator = collection.getIterator();

console.log(&#39;Straight traversal:&#39;);
while (iterator.valid()) {
    console.log(iterator.next());
}

console.log(&#39;&#39;);
console.log(&#39;Reverse traversal:&#39;);
const reverseIterator = collection.getReverseIterator();
while (reverseIterator.valid()) {
    console.log(reverseIterator.next());
}</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** 実行結果

<figure class="code">
<pre class="code" lang="output"><code>Straight traversal:
First
Second
Third

Reverse traversal:
Third
Second
First</code></pre>
</figure>

::: next
#### 次を読む

[Mediator を TypeScript で []{.fa
.fa-arrow-right}](../../mediator/typescript/example.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### 戻る

[[]{.fa .fa-arrow-left} Command を TypeScript
で](../../command/typescript/example.html){.btn .btn-default rel="prev"}
:::

## 他言語での **Iterator**

[![Iterator を C#
で](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Iterator を C# で"){.prog-lang-link}
[![Iterator を C++
で](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Iterator を C++ で"){.prog-lang-link}
[![Iterator を Go
で](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Iterator を Go で"){.prog-lang-link}
[![Iterator を Java
で](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Iterator を Java で"){.prog-lang-link}
[![Iterator を PHP
で](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Iterator を PHP で"){.prog-lang-link}
[![Iterator を Python
で](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Iterator を Python で"){.prog-lang-link}
[![Iterator を Ruby
で](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Iterator を Ruby で"){.prog-lang-link}
[![Iterator を Rust
で](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Iterator を Rust で"){.prog-lang-link}
[![Iterator を Swift
で](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Iterator を Swift で"){.prog-lang-link}
:::::::::::::::::

::::::: {.prom .banner-sidebar .banner .banner-striped .banner-examples .banner-removable .banner-removable-patterns data-id="DP: Examples-IDE" creative-id="examples-sidebar-ja" position="sidebar"}
:::::: {.banner-inner style="font-size: 14px; line-height: 21px;"}
::: banner-examples-img
[![](../../../../images/patterns/banners/examples-ide91ca.png?id=3115b4b548fb96b75974e2de8f4f49bc){width="215"
height="158" loading="lazy"
srcset="/images/patterns/banners/examples-ide-2x.png?id=93c007a6d157b97c28eb213f59d716ae 2x"}](../../book.html)
:::

::: banner-examples-fade
[ コード例の書庫](../../book.html){.btn .btn-secondary .btn-block}
:::

::: {.banner-examples-text style="margin-top: 1rem;"}
**直撃！デザインパターン**を電子書籍で購入し、IDE
で開くことができる数多くの詳細な例を含む書庫にアクセス。

[ 詳細](../../book.html){.btn .btn-secondary .btn-block}
:::
::::::
:::::::
:::::::::::::::::::::::
::::::::::::::::::::::::
