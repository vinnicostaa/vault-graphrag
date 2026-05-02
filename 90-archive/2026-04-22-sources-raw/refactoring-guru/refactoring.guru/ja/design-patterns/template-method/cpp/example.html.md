:::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} /
[デザインパターン](../../../design-patterns.html){.type} / [Template
Method](../../template-method.html){.type} /
[C++](../../cpp.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Template
Method](../../../../images/patterns/cards/template-method-mini56e3.png?id=9f200248d88026d8e79d0f3dae411ab4){srcset="/images/patterns/cards/template-method-mini-2x.png?id=178bf56e39b3a1f548dd636076209c98 2x"}
:::

# **Template Method** を C++ で {#template-method-を-c-で .pattern-example-title-block-title}
::::

::::::::::: pattern-example-body
::: pattern-example-brief
**Template Method** は[、]{.chpule2}[
]{.chpuri2}振る舞いに関するデザインパターンの一つで[、]{.chpule2}[
]{.chpuri2}アルゴリズムの骨組みを基底クラスで定義し[、]{.chpule2}[
]{.chpuri2}サブクラスではアルゴリズムの全体的な構造は残したまま[、]{.chpule2}[
]{.chpuri2}ステップを上書きします[。]{.chpule2}[ ]{.chpuri2}
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Template Method の詳細 ](../../template-method.html){.btn .btn-lg
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
 [main](#example-0--main-cc)
:::

::: en-file
 [Output](#example-0--Output-txt)
:::
::::::::

**複雑度[：]{.chpule2}[ ]{.chpuri2}**[]{.fa-stars}

**人気度[：]{.chpule2}[ ]{.chpuri2}**[]{.fa-stars}

**使用例[：]{.chpule2}[ ]{.chpuri2}** Template Method
パターンは[、]{.chpule2}[ ]{.chpuri2}C++
コードではよく見かけます[。]{.chpule2}[
]{.chpuri2}フレームワークの利用者に継承を用いて標準機能を拡張する単純な方法を提供するために[、]{.chpule2}[
]{.chpuri2}開発者がよく利用します[。]{.chpule2}[ ]{.chpuri2}

**見つけ方[：]{.chpule2}[
]{.chpuri2}**基底クラスにあるメソッドの一つが[、]{.chpule2}[
]{.chpuri2}他の抽象または空くうのメソッドをたくさん呼んでいる場合[、]{.chpule2}[
]{.chpuri2}Template Method を識別できます[。]{.chpule2}[ ]{.chpuri2}
:::::::::::

[]{#example-0}

## 概念的な例 {#example-0-title}

この例は[、]{.chpule2}[ ]{.chpuri2}**Template Method**
デザインパターンの構造を説明するためのものです[。]{.chpule2}[
]{.chpuri2}以下の質問に答えることを目的としています[：]{.chpule2}[
]{.chpuri2}

-   どういうクラスからできているか[？]{.chpule2}[ ]{.chpuri2}
-   それぞれのクラスの役割は[？]{.chpule2}[ ]{.chpuri2}
-   パターンの要素同士はどう関係しているのか[？]{.chpule2}[ ]{.chpuri2}

#### []{#example-0--main-cc .anchor} **main.cc:** 概念的な例

<figure class="code">
<pre class="code" lang="cpp"><code>/**
 * The Abstract Class defines a template method that contains a skeleton of some
 * algorithm, composed of calls to (usually) abstract primitive operations.
 *
 * Concrete subclasses should implement these operations, but leave the template
 * method itself intact.
 */
class AbstractClass {
  /**
   * The template method defines the skeleton of an algorithm.
   */
 public:
  void TemplateMethod() const {
    this-&gt;BaseOperation1();
    this-&gt;RequiredOperations1();
    this-&gt;BaseOperation2();
    this-&gt;Hook1();
    this-&gt;RequiredOperation2();
    this-&gt;BaseOperation3();
    this-&gt;Hook2();
  }
  /**
   * These operations already have implementations.
   */
 protected:
  void BaseOperation1() const {
    std::cout &lt;&lt; &quot;AbstractClass says: I am doing the bulk of the work\n&quot;;
  }
  void BaseOperation2() const {
    std::cout &lt;&lt; &quot;AbstractClass says: But I let subclasses override some operations\n&quot;;
  }
  void BaseOperation3() const {
    std::cout &lt;&lt; &quot;AbstractClass says: But I am doing the bulk of the work anyway\n&quot;;
  }
  /**
   * These operations have to be implemented in subclasses.
   */
  virtual void RequiredOperations1() const = 0;
  virtual void RequiredOperation2() const = 0;
  /**
   * These are &quot;hooks.&quot; Subclasses may override them, but it&#39;s not mandatory
   * since the hooks already have default (but empty) implementation. Hooks
   * provide additional extension points in some crucial places of the
   * algorithm.
   */
  virtual void Hook1() const {}
  virtual void Hook2() const {}
};
/**
 * Concrete classes have to implement all abstract operations of the base class.
 * They can also override some operations with a default implementation.
 */
class ConcreteClass1 : public AbstractClass {
 protected:
  void RequiredOperations1() const override {
    std::cout &lt;&lt; &quot;ConcreteClass1 says: Implemented Operation1\n&quot;;
  }
  void RequiredOperation2() const override {
    std::cout &lt;&lt; &quot;ConcreteClass1 says: Implemented Operation2\n&quot;;
  }
};
/**
 * Usually, concrete classes override only a fraction of base class&#39; operations.
 */
class ConcreteClass2 : public AbstractClass {
 protected:
  void RequiredOperations1() const override {
    std::cout &lt;&lt; &quot;ConcreteClass2 says: Implemented Operation1\n&quot;;
  }
  void RequiredOperation2() const override {
    std::cout &lt;&lt; &quot;ConcreteClass2 says: Implemented Operation2\n&quot;;
  }
  void Hook1() const override {
    std::cout &lt;&lt; &quot;ConcreteClass2 says: Overridden Hook1\n&quot;;
  }
};
/**
 * The client code calls the template method to execute the algorithm. Client
 * code does not have to know the concrete class of an object it works with, as
 * long as it works with objects through the interface of their base class.
 */
void ClientCode(AbstractClass *class_) {
  // ...
  class_-&gt;TemplateMethod();
  // ...
}

int main() {
  std::cout &lt;&lt; &quot;Same client code can work with different subclasses:\n&quot;;
  ConcreteClass1 *concreteClass1 = new ConcreteClass1;
  ClientCode(concreteClass1);
  std::cout &lt;&lt; &quot;\n&quot;;
  std::cout &lt;&lt; &quot;Same client code can work with different subclasses:\n&quot;;
  ConcreteClass2 *concreteClass2 = new ConcreteClass2;
  ClientCode(concreteClass2);
  delete concreteClass1;
  delete concreteClass2;
  return 0;
}</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** 実行結果

<figure class="code">
<pre class="code" lang="output"><code>Same client code can work with different subclasses:
AbstractClass says: I am doing the bulk of the work
ConcreteClass1 says: Implemented Operation1
AbstractClass says: But I let subclasses override some operations
ConcreteClass1 says: Implemented Operation2
AbstractClass says: But I am doing the bulk of the work anyway

Same client code can work with different subclasses:
AbstractClass says: I am doing the bulk of the work
ConcreteClass2 says: Implemented Operation1
AbstractClass says: But I let subclasses override some operations
ConcreteClass2 says: Overridden Hook1
ConcreteClass2 says: Implemented Operation2
AbstractClass says: But I am doing the bulk of the work anyway</code></pre>
</figure>

::: next
#### 次を読む

[Visitor を C++ で []{.fa
.fa-arrow-right}](../../visitor/cpp/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### 戻る

[[]{.fa .fa-arrow-left} Strategy を C++
で](../../strategy/cpp/example.html){.btn .btn-default rel="prev"}
:::

## 他言語での **Template Method**

[![Template Method を C#
で](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Template Method を C# で"){.prog-lang-link}
[![Template Method を Go
で](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Template Method を Go で"){.prog-lang-link}
[![Template Method を Java
で](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Template Method を Java で"){.prog-lang-link}
[![Template Method を PHP
で](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Template Method を PHP で"){.prog-lang-link}
[![Template Method を Python
で](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Template Method を Python で"){.prog-lang-link}
[![Template Method を Ruby
で](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Template Method を Ruby で"){.prog-lang-link}
[![Template Method を Rust
で](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Template Method を Rust で"){.prog-lang-link}
[![Template Method を Swift
で](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Template Method を Swift で"){.prog-lang-link}
[![Template Method を TypeScript
で](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Template Method を TypeScript で"){.prog-lang-link}
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
