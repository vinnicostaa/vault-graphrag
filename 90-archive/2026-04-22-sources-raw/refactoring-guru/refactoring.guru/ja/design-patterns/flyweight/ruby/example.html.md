:::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} /
[デザインパターン](../../../design-patterns.html){.type} /
[Flyweight](../../flyweight.html){.type} /
[Ruby](../../ruby.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Flyweight](../../../../images/patterns/cards/flyweight-minie3d8.png?id=422ca8d2f90614dce810a8812c626698){srcset="/images/patterns/cards/flyweight-mini-2x.png?id=857ca676fff84317bb0dab76abfce08e 2x"}
:::

# **Flyweight** を Ruby で {#flyweight-を-ruby-で .pattern-example-title-block-title}
::::

::::::::::: pattern-example-body
::: pattern-example-brief
**Flyweight** は[、]{.chpule2}[
]{.chpuri2}構造に関するデザインパターンの一つで[、]{.chpule2}[
]{.chpuri2}メモリー消費量を低く抑えることで[、]{.chpule2}[
]{.chpuri2}プログラムが膨大な数のオブジェクトを支えることができるようにします[。]{.chpule2}[
]{.chpuri2}

複数のオブジェクト間でオブジェクトの状態の一部を共有することにより[、]{.chpule2}[
]{.chpuri2}これを実現します[。]{.chpule2}[
]{.chpuri2}つまり[、]{.chpule2}[ ]{.chpuri2}Flyweight は[、]{.chpule2}[
]{.chpuri2}異なるオブジェクトによって使われる同じデータをキャッシュすることにより[、]{.chpule2}[
]{.chpuri2}RAM を節約します[。]{.chpule2}[ ]{.chpuri2}
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Flyweight の詳細 ](../../flyweight.html){.btn .btn-lg .btn-primary}
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

**使用例[：]{.chpule2}[ ]{.chpuri2}** Flyweight
の目的はただ一つです[：]{.chpule2}[
]{.chpuri2}メモリー摂取を最小にする[。]{.chpule2}[
]{.chpuri2}もしご自分のプログラムが RAM
不足で困っていない場合は[、]{.chpule2}[
]{.chpuri2}このパターンのことはしばらく忘れて大丈夫です[。]{.chpule2}[
]{.chpuri2}

**見つけ方[：]{.chpule2}[ ]{.chpuri2}** Flyweight は[、]{.chpule2}[
]{.chpuri2}新規オブジェクトの代わりにキャッシュされたオブジェクトを返す生成メソッドの存在により識別できます[。]{.chpule2}[
]{.chpuri2}
:::::::::::

[]{#example-0}

## 概念的な例 {#example-0-title}

この例は[、]{.chpule2}[ ]{.chpuri2}**Flyweight**
デザインパターンの構造を説明するためのものです[。]{.chpule2}[
]{.chpuri2}以下の質問に答えることを目的としています[：]{.chpule2}[
]{.chpuri2}

-   どういうクラスからできているか[？]{.chpule2}[ ]{.chpuri2}
-   それぞれのクラスの役割は[？]{.chpule2}[ ]{.chpuri2}
-   パターンの要素同士はどう関係しているのか[？]{.chpule2}[ ]{.chpuri2}

#### []{#example-0--main-rb .anchor} **main.rb:** 概念的な例

<figure class="code">
<pre class="code" lang="ruby"><code>require &#39;json&#39;

# The Flyweight stores a common portion of the state (also called intrinsic
# state) that belongs to multiple real business entities. The Flyweight accepts
# the rest of the state (extrinsic state, unique for each entity) via its method
# parameters.
class Flyweight
  # @param [String] shared_state
  def initialize(shared_state)
    @shared_state = shared_state
  end

  # @param [String] unique_state
  def operation(unique_state)
    s = @shared_state.to_json
    u = unique_state.to_json
    print &quot;Flyweight: Displaying shared (#{s}) and unique (#{u}) state.&quot;
  end
end

# The Flyweight Factory creates and manages the Flyweight objects. It ensures
# that flyweights are shared correctly. When the client requests a flyweight,
# the factory either returns an existing instance or creates a new one, if it
# doesn&#39;t exist yet.
class FlyweightFactory
  # @param [Hash] initial_flyweights
  def initialize(initial_flyweights)
    @flyweights = {}
    initial_flyweights.each do |state|
      @flyweights[get_key(state)] = Flyweight.new(state)
    end
  end

  # Returns a Flyweight&#39;s string hash for a given state.
  def get_key(state)
    state.sort.join(&#39;_&#39;)
  end

  # Returns an existing Flyweight with a given state or creates a new one.
  def get_flyweight(shared_state)
    key = get_key(shared_state)

    if !@flyweights.key?(key)
      puts &quot;FlyweightFactory: Can&#39;t find a flyweight, creating new one.&quot;
      @flyweights[key] = Flyweight.new(shared_state)
    else
      puts &#39;FlyweightFactory: Reusing existing flyweight.&#39;
    end

    @flyweights[key]
  end

  def list_flyweights
    puts &quot;FlyweightFactory: I have #{@flyweights.size} flyweights:&quot;
    print @flyweights.keys.join(&quot;\n&quot;)
  end
end

# @param [FlyweightFactory] factory
# @param [String] plates
# @param [String] owner
# @param [String] brand
# @param [String] model
# @param [String] color
def add_car_to_police_database(factory, plates, owner, brand, model, color)
  puts &quot;\n\nClient: Adding a car to database.&quot;
  flyweight = factory.get_flyweight([brand, model, color])
  # The client code either stores or calculates extrinsic state and passes it to
  # the flyweight&#39;s methods.
  flyweight.operation([plates, owner])
end

# The client code usually creates a bunch of pre-populated flyweights in the
# initialization stage of the application.

factory = FlyweightFactory.new([
                                 %w[Chevrolet Camaro2018 pink],
                                 [&#39;Mercedes Benz&#39;, &#39;C300&#39;, &#39;black&#39;],
                                 [&#39;Mercedes Benz&#39;, &#39;C500&#39;, &#39;red&#39;],
                                 %w[BMW M5 red],
                                 %w[BMW X6 white]
                               ])

factory.list_flyweights

add_car_to_police_database(factory, &#39;CL234IR&#39;, &#39;James Doe&#39;, &#39;BMW&#39;, &#39;M5&#39;, &#39;red&#39;)

add_car_to_police_database(factory, &#39;CL234IR&#39;, &#39;James Doe&#39;, &#39;BMW&#39;, &#39;X1&#39;, &#39;red&#39;)

puts &quot;\n\n&quot;

factory.list_flyweights</code></pre>
</figure>

#### []{#example-0--output-txt .anchor} **output.txt:** 実行結果

<figure class="code">
<pre class="code" lang="output"><code>FlyweightFactory: I have 5 flyweights:
Camaro2018_Chevrolet_pink
C300_Mercedes Benz_black
C500_Mercedes Benz_red
BMW_M5_red
BMW_X6_white

Client: Adding a car to database.
FlyweightFactory: Reusing existing flyweight.
Flyweight: Displaying shared ([&quot;BMW&quot;,&quot;M5&quot;,&quot;red&quot;]) and unique ([&quot;CL234IR&quot;,&quot;James Doe&quot;]) state.

Client: Adding a car to database.
FlyweightFactory: Can&#39;t find a flyweight, creating new one.
Flyweight: Displaying shared ([&quot;BMW&quot;,&quot;X1&quot;,&quot;red&quot;]) and unique ([&quot;CL234IR&quot;,&quot;James Doe&quot;]) state.

FlyweightFactory: I have 6 flyweights:
Camaro2018_Chevrolet_pink
C300_Mercedes Benz_black
C500_Mercedes Benz_red
BMW_M5_red
BMW_X6_white
BMW_X1_red</code></pre>
</figure>

## 他言語での **Flyweight**

[![Flyweight を C#
で](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Flyweight を C# で"){.prog-lang-link}
[![Flyweight を C++
で](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Flyweight を C++ で"){.prog-lang-link}
[![Flyweight を Go
で](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Flyweight を Go で"){.prog-lang-link}
[![Flyweight を Java
で](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Flyweight を Java で"){.prog-lang-link}
[![Flyweight を PHP
で](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Flyweight を PHP で"){.prog-lang-link}
[![Flyweight を Python
で](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Flyweight を Python で"){.prog-lang-link}
[![Flyweight を Rust
で](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Flyweight を Rust で"){.prog-lang-link}
[![Flyweight を Swift
で](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Flyweight を Swift で"){.prog-lang-link}
[![Flyweight を TypeScript
で](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Flyweight を TypeScript で"){.prog-lang-link}
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
