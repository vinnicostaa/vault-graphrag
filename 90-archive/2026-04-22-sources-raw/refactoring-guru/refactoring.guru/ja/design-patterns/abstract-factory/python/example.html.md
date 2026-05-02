:::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} /
[デザインパターン](../../../design-patterns.html){.type} / [Abstract
Factory](../../abstract-factory.html){.type} /
[Python](../../python.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Abstract
Factory](../../../../images/patterns/cards/abstract-factory-minid1c5.png?id=4c3927c446313a38ce77dfee38111e27){srcset="/images/patterns/cards/abstract-factory-mini-2x.png?id=22236aaa65ff52cbde1c713216d52c1f 2x"}
:::

# **Abstract Factory** を Python で {#abstract-factory-を-python-で .pattern-example-title-block-title}
::::

::::::::::: pattern-example-body
::: pattern-example-brief
**Abstract Factory** は[、]{.chpule2}[
]{.chpuri2}生成に関するデザインパターンのひとつで[、]{.chpule2}[
]{.chpuri2}具象クラスを指定することなく[、]{.chpule2}[
]{.chpuri2}プロダクト[ ]{.chpule1}[（]{.chpuri1}訳注[：]{.chpule2}[
]{.chpuri2}本パターンでは[、]{.chpule2}[
]{.chpuri2}生成されるモノのことを一般にプロダクトと呼びます[）]{.chpule2}[
]{.chpuri2}のファミリー全部を生成することを可能とします[。]{.chpule2}[
]{.chpuri2}

Abstract Factory は[、]{.chpule2}[
]{.chpuri2}個々のプロダクト全部を作成するためのインターフェースを定義しますが[、]{.chpule2}[
]{.chpuri2}実際のプロダクト作成の作業は[、]{.chpule2}[
]{.chpuri2}具象クラスに委ねられます[。]{.chpule2}[
]{.chpuri2}ファクトリーの型[
]{.chpule1}[（]{.chpuri1}クラス[）]{.chpule2}[
]{.chpuri2}それぞれは[、]{.chpule2}[
]{.chpuri2}特定のプロダクトの異種に対応します[。]{.chpule2}[ ]{.chpuri2}

クライアント・コードは[、]{.chpule2}[
]{.chpuri2}コンストラクター呼び出し[ ]{.chpule1}[（]{.chpuri1}`new`
演算子[）]{.chpule2}[
]{.chpuri2}で直接プロダクトを作成する代わりにファクトリー・オブジェクトの作成メソッドを呼び出します[。]{.chpule2}[
]{.chpuri2}
ファクトリーはプロダクトの特定の異種に対応しているため[、]{.chpule2}[
]{.chpuri2}すべてのプロダクトには互換性があります[。]{.chpule2}[
]{.chpuri2}

クライアント・コードは[、]{.chpule2}[
]{.chpuri2}その抽象インターフェイスを通じてのみファクトリーやプロダクトとやりとりします[。]{.chpule2}[
]{.chpuri2}このため[、]{.chpule2}[
]{.chpuri2}クライアント・コードはファクトリー・オブジェクトによって作成された任意のプロダクトの異種と動作します[。]{.chpule2}[
]{.chpuri2}プログラマーがやるべきことは[、]{.chpule2}[
]{.chpuri2}新しい具象ファクトリー・クラスを作成し[、]{.chpule2}[
]{.chpuri2}それをクライアント・コードに渡すことです[。]{.chpule2}[
]{.chpuri2}

> もし各種ファクトリー系のパターンやコンセプトの違いで迷った場合は[、]{.chpule2}[
> ]{.chpuri2}[ファクトリーの比較](../../factory-comparison.html)
> をご覧ください[。]{.chpule2}[ ]{.chpuri2}
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Abstract Factory の詳細 ](../../abstract-factory.html){.btn .btn-lg
.btn-primary}
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
 [main](#example-0--main-py)
:::

::: en-file
 [Output](#example-0--Output-txt)
:::
::::::::

**複雑度[：]{.chpule2}[ ]{.chpuri2}**[]{.fa-stars}

**人気度[：]{.chpule2}[ ]{.chpuri2}**[]{.fa-stars}

**使用例[：]{.chpule2}[ ]{.chpuri2}** Abstract Factory
パターンは[、]{.chpule2}[ ]{.chpuri2}Python
コードではよく見かけます[。]{.chpule2}[
]{.chpuri2}多くのフレームワークやライブラリーが[、]{.chpule2}[
]{.chpuri2}その標準コンポーネントを拡張したりカスタマイズするためにこのパターンを使います[。]{.chpule2}[
]{.chpuri2}

**見つけ方[：]{.chpule2}[ ]{.chpuri2}**このパターンは[、]{.chpule2}[
]{.chpuri2}ファクトリー・オブジェクトを返すメソッドに注目すれば[、]{.chpule2}[
]{.chpuri2}簡単に見つけられます[。]{.chpule2}[
]{.chpuri2}そしてファクトリーを使ってサブコンポーネントが作成されます[。]{.chpule2}[
]{.chpuri2}
:::::::::::

[]{#example-0}

## 概念的な例 {#example-0-title}

この例は[、]{.chpule2}[ ]{.chpuri2}**Abstract Factory**
デザインパターンの構造を説明するためのものです[。]{.chpule2}[
]{.chpuri2}以下の質問に答えることを目的としています[：]{.chpule2}[
]{.chpuri2}

-   どういうクラスからできているか[？]{.chpule2}[ ]{.chpuri2}
-   それぞれのクラスの役割は[？]{.chpule2}[ ]{.chpuri2}
-   パターンの要素同士はどう関係しているのか[？]{.chpule2}[ ]{.chpuri2}

#### []{#example-0--main-py .anchor} **main.py:** 概念的な例

<figure class="code">
<pre class="code" lang="python"><code>from __future__ import annotations
from abc import ABC, abstractmethod


class AbstractFactory(ABC):
    &quot;&quot;&quot;
    The Abstract Factory interface declares a set of methods that return
    different abstract products. These products are called a family and are
    related by a high-level theme or concept. Products of one family are usually
    able to collaborate among themselves. A family of products may have several
    variants, but the products of one variant are incompatible with products of
    another.
    &quot;&quot;&quot;
    @abstractmethod
    def create_product_a(self) -&gt; AbstractProductA:
        pass

    @abstractmethod
    def create_product_b(self) -&gt; AbstractProductB:
        pass


class ConcreteFactory1(AbstractFactory):
    &quot;&quot;&quot;
    Concrete Factories produce a family of products that belong to a single
    variant. The factory guarantees that resulting products are compatible. Note
    that signatures of the Concrete Factory&#39;s methods return an abstract
    product, while inside the method a concrete product is instantiated.
    &quot;&quot;&quot;

    def create_product_a(self) -&gt; AbstractProductA:
        return ConcreteProductA1()

    def create_product_b(self) -&gt; AbstractProductB:
        return ConcreteProductB1()


class ConcreteFactory2(AbstractFactory):
    &quot;&quot;&quot;
    Each Concrete Factory has a corresponding product variant.
    &quot;&quot;&quot;

    def create_product_a(self) -&gt; AbstractProductA:
        return ConcreteProductA2()

    def create_product_b(self) -&gt; AbstractProductB:
        return ConcreteProductB2()


class AbstractProductA(ABC):
    &quot;&quot;&quot;
    Each distinct product of a product family should have a base interface. All
    variants of the product must implement this interface.
    &quot;&quot;&quot;

    @abstractmethod
    def useful_function_a(self) -&gt; str:
        pass


&quot;&quot;&quot;
Concrete Products are created by corresponding Concrete Factories.
&quot;&quot;&quot;


class ConcreteProductA1(AbstractProductA):
    def useful_function_a(self) -&gt; str:
        return &quot;The result of the product A1.&quot;


class ConcreteProductA2(AbstractProductA):
    def useful_function_a(self) -&gt; str:
        return &quot;The result of the product A2.&quot;


class AbstractProductB(ABC):
    &quot;&quot;&quot;
    Here&#39;s the the base interface of another product. All products can interact
    with each other, but proper interaction is possible only between products of
    the same concrete variant.
    &quot;&quot;&quot;
    @abstractmethod
    def useful_function_b(self) -&gt; None:
        &quot;&quot;&quot;
        Product B is able to do its own thing...
        &quot;&quot;&quot;
        pass

    @abstractmethod
    def another_useful_function_b(self, collaborator: AbstractProductA) -&gt; None:
        &quot;&quot;&quot;
        ...but it also can collaborate with the ProductA.

        The Abstract Factory makes sure that all products it creates are of the
        same variant and thus, compatible.
        &quot;&quot;&quot;
        pass


&quot;&quot;&quot;
Concrete Products are created by corresponding Concrete Factories.
&quot;&quot;&quot;


class ConcreteProductB1(AbstractProductB):
    def useful_function_b(self) -&gt; str:
        return &quot;The result of the product B1.&quot;

    &quot;&quot;&quot;
    The variant, Product B1, is only able to work correctly with the variant,
    Product A1. Nevertheless, it accepts any instance of AbstractProductA as an
    argument.
    &quot;&quot;&quot;

    def another_useful_function_b(self, collaborator: AbstractProductA) -&gt; str:
        result = collaborator.useful_function_a()
        return f&quot;The result of the B1 collaborating with the ({result})&quot;


class ConcreteProductB2(AbstractProductB):
    def useful_function_b(self) -&gt; str:
        return &quot;The result of the product B2.&quot;

    def another_useful_function_b(self, collaborator: AbstractProductA):
        &quot;&quot;&quot;
        The variant, Product B2, is only able to work correctly with the
        variant, Product A2. Nevertheless, it accepts any instance of
        AbstractProductA as an argument.
        &quot;&quot;&quot;
        result = collaborator.useful_function_a()
        return f&quot;The result of the B2 collaborating with the ({result})&quot;


def client_code(factory: AbstractFactory) -&gt; None:
    &quot;&quot;&quot;
    The client code works with factories and products only through abstract
    types: AbstractFactory and AbstractProduct. This lets you pass any factory
    or product subclass to the client code without breaking it.
    &quot;&quot;&quot;
    product_a = factory.create_product_a()
    product_b = factory.create_product_b()

    print(f&quot;{product_b.useful_function_b()}&quot;)
    print(f&quot;{product_b.another_useful_function_b(product_a)}&quot;, end=&quot;&quot;)


if __name__ == &quot;__main__&quot;:
    &quot;&quot;&quot;
    The client code can work with any concrete factory class.
    &quot;&quot;&quot;
    print(&quot;Client: Testing client code with the first factory type:&quot;)
    client_code(ConcreteFactory1())

    print(&quot;\n&quot;)

    print(&quot;Client: Testing the same client code with the second factory type:&quot;)
    client_code(ConcreteFactory2())</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** 実行結果

<figure class="code">
<pre class="code" lang="output"><code>Client: Testing client code with the first factory type:
The result of the product B1.
The result of the B1 collaborating with the (The result of the product A1.)

Client: Testing the same client code with the second factory type:
The result of the product B2.
The result of the B2 collaborating with the (The result of the product A2.)</code></pre>
</figure>

::: next
#### 次を読む

[Builder を Python で []{.fa
.fa-arrow-right}](../../builder/python/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### 戻る

[[]{.fa .fa-arrow-left} デザインパターンを Python
で](../../python.html){.btn .btn-default rel="prev"}
:::

## 他言語での **Abstract Factory**

[![Abstract Factory を C#
で](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Abstract Factory を C# で"){.prog-lang-link}
[![Abstract Factory を C++
で](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Abstract Factory を C++ で"){.prog-lang-link}
[![Abstract Factory を Go
で](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Abstract Factory を Go で"){.prog-lang-link}
[![Abstract Factory を Java
で](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Abstract Factory を Java で"){.prog-lang-link}
[![Abstract Factory を PHP
で](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Abstract Factory を PHP で"){.prog-lang-link}
[![Abstract Factory を Ruby
で](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Abstract Factory を Ruby で"){.prog-lang-link}
[![Abstract Factory を Rust
で](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Abstract Factory を Rust で"){.prog-lang-link}
[![Abstract Factory を Swift
で](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Abstract Factory を Swift で"){.prog-lang-link}
[![Abstract Factory を TypeScript
で](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Abstract Factory を TypeScript で"){.prog-lang-link}
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
