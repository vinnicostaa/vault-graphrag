:::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} /
[デザインパターン](../../../design-patterns.html){.type} / [Factory
Method](../../factory-method.html){.type} /
[Ruby](../../ruby.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Factory
Method](../../../../images/patterns/cards/factory-method-minid7f6.png?id=72619e9527893374b98a5913779ac167){srcset="/images/patterns/cards/factory-method-mini-2x.png?id=fa9d4a8d61a67cc3822e52b9daf69dad 2x"}
:::

# **Factory Method** を Ruby で {#factory-method-を-ruby-で .pattern-example-title-block-title}
::::

::::::::::: pattern-example-body
::: pattern-example-brief
**Factory Method** は[、]{.chpule2}[
]{.chpuri2}生成に関するデザインパターンの一つで[、]{.chpule2}[
]{.chpuri2}具象クラスを指定することなく[、]{.chpule2}[
]{.chpuri2}プロダクト[ ]{.chpule1}[（]{.chpuri1}訳注[：]{.chpule2}[
]{.chpuri2}本パターンでは[、]{.chpule2}[
]{.chpuri2}生成されるモノのことを一般にプロダクトと呼びます[）]{.chpule2}[
]{.chpuri2}のオブジェクトを生成することを可能とします[。]{.chpule2}[
]{.chpuri2}

Factory Method では[、]{.chpule2}[
]{.chpuri2}オブジェクトの生成において[、]{.chpule2}[
]{.chpuri2}直接のコンストラクター呼び出し[
]{.chpule1}[（]{.chpuri1}`new` 演算子[）]{.chpule2}[
]{.chpuri2}代わりに使用すべきメソッドを定義します[。]{.chpule2}[
]{.chpuri2}サブクラスにおいてこのメソッドを上書きすることにより[、]{.chpule2}[
]{.chpuri2}生成されるオブジェクトのクラスを変更します[。]{.chpule2}[
]{.chpuri2}

> もし各種ファクトリー系のパターンやコンセプトの違いで迷った場合は[、]{.chpule2}[
> ]{.chpuri2}[ファクトリーの比較](../../factory-comparison.html)
> をご覧ください[。]{.chpule2}[ ]{.chpuri2}
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Factory Method の詳細 ](../../factory-method.html){.btn .btn-lg
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
 [main](#example-0--main-rb)
:::

::: en-file
 [output](#example-0--output-txt)
:::
::::::::

**複雑度[：]{.chpule2}[ ]{.chpuri2}**[]{.fa-stars}

**人気度[：]{.chpule2}[ ]{.chpuri2}**[]{.fa-stars}

**使用例[：]{.chpule2}[ ]{.chpuri2}** Factory Method
パターンは[、]{.chpule2}[ ]{.chpuri2}Ruby
コードでは広く使われます[。]{.chpule2}[
]{.chpuri2}コードに高度の柔軟性を持たせたい時にとても役に立ちます[。]{.chpule2}[
]{.chpuri2}

**見つけ方[：]{.chpule2}[ ]{.chpuri2}**
具象クラスで具象オブジェクトを作成し[、]{.chpule2}[
]{.chpuri2}それを抽象型またはインターフェースのオブジェクトとして返すような生成メソッドの存在により[、]{.chpule2}[
]{.chpuri2}Factory Method を識別できます[。]{.chpule2}[ ]{.chpuri2}
:::::::::::

[]{#example-0}

## 概念的な例 {#example-0-title}

この例は[、]{.chpule2}[ ]{.chpuri2}**Factory Method**
デザインパターンの構造を説明するためのものです[。]{.chpule2}[
]{.chpuri2}以下の質問に答えることを目的としています[：]{.chpule2}[
]{.chpuri2}

-   どういうクラスからできているか[？]{.chpule2}[ ]{.chpuri2}
-   それぞれのクラスの役割は[？]{.chpule2}[ ]{.chpuri2}
-   パターンの要素同士はどう関係しているのか[？]{.chpule2}[ ]{.chpuri2}

#### []{#example-0--main-rb .anchor} **main.rb:** 概念的な例

<figure class="code">
<pre class="code" lang="ruby"><code># The Creator class declares the factory method that is supposed to return an
# object of a Product class. The Creator&#39;s subclasses usually provide the
# implementation of this method.
class Creator
  # Note that the Creator may also provide some default implementation of the
  # factory method.
  def factory_method
    raise NotImplementedError, &quot;#{self.class} has not implemented method &#39;#{__method__}&#39;&quot;
  end

  # Also note that, despite its name, the Creator&#39;s primary responsibility is
  # not creating products. Usually, it contains some core business logic that
  # relies on Product objects, returned by the factory method. Subclasses can
  # indirectly change that business logic by overriding the factory method and
  # returning a different type of product from it.
  def some_operation
    # Call the factory method to create a Product object.
    product = factory_method

    # Now, use the product.
    &quot;Creator: The same creator&#39;s code has just worked with #{product.operation}&quot;
  end
end

# Concrete Creators override the factory method in order to change the resulting
# product&#39;s type.
class ConcreteCreator1 &lt; Creator
  # Note that the signature of the method still uses the abstract product type,
  # even though the concrete product is actually returned from the method. This
  # way the Creator can stay independent of concrete product classes.
  def factory_method
    ConcreteProduct1.new
  end
end

class ConcreteCreator2 &lt; Creator
  # @return [ConcreteProduct2]
  def factory_method
    ConcreteProduct2.new
  end
end

# The Product interface declares the operations that all concrete products must
# implement.
class Product
  # return [String]
  def operation
    raise NotImplementedError, &quot;#{self.class} has not implemented method &#39;#{__method__}&#39;&quot;
  end
end

# Concrete Products provide various implementations of the Product interface.
class ConcreteProduct1 &lt; Product
  # @return [String]
  def operation
    &#39;{Result of the ConcreteProduct1}&#39;
  end
end

class ConcreteProduct2 &lt; Product
  # @return [String]
  def operation
    &#39;{Result of the ConcreteProduct2}&#39;
  end
end

# The client code works with an instance of a concrete creator, albeit through
# its base interface. As long as the client keeps working with the creator via
# the base interface, you can pass it any creator&#39;s subclass.
def client_code(creator)
  print &quot;Client: I&#39;m not aware of the creator&#39;s class, but it still works.\n&quot;\
        &quot;#{creator.some_operation}&quot;
end

puts &#39;App: Launched with the ConcreteCreator1.&#39;
client_code(ConcreteCreator1.new)
puts &quot;\n\n&quot;

puts &#39;App: Launched with the ConcreteCreator2.&#39;
client_code(ConcreteCreator2.new)</code></pre>
</figure>

#### []{#example-0--output-txt .anchor} **output.txt:** 実行結果

<figure class="code">
<pre class="code" lang="output"><code>App: Launched with the ConcreteCreator1.
Client: I&#39;m not aware of the creator&#39;s class, but it still works.
Creator: The same creator&#39;s code has just worked with {Result of the ConcreteProduct1}

App: Launched with the ConcreteCreator2.
Client: I&#39;m not aware of the creator&#39;s class, but it still works.
Creator: The same creator&#39;s code has just worked with {Result of the ConcreteProduct2}</code></pre>
</figure>

## 他言語での **Factory Method**

[![Factory Method を C#
で](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Factory Method を C# で"){.prog-lang-link}
[![Factory Method を C++
で](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Factory Method を C++ で"){.prog-lang-link}
[![Factory Method を Go
で](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Factory Method を Go で"){.prog-lang-link}
[![Factory Method を Java
で](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Factory Method を Java で"){.prog-lang-link}
[![Factory Method を PHP
で](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Factory Method を PHP で"){.prog-lang-link}
[![Factory Method を Python
で](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Factory Method を Python で"){.prog-lang-link}
[![Factory Method を Rust
で](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Factory Method を Rust で"){.prog-lang-link}
[![Factory Method を Swift
で](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Factory Method を Swift で"){.prog-lang-link}
[![Factory Method を TypeScript
で](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Factory Method を TypeScript で"){.prog-lang-link}
:::::::::::::::

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
:::::::::::::::::::::
::::::::::::::::::::::
