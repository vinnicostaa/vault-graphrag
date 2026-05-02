:::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [디자인
패턴들](../../../design-patterns.html){.type} /
[반복자](../../iterator.html){.type} / [루비](../../ruby.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![반복자](../../../../images/patterns/cards/iterator-minie2c2.png?id=76c28bb48f997b36965983dd2b41f02e){srcset="/images/patterns/cards/iterator-mini-2x.png?id=7d28f66a487066774a743cfceaa40ea4 2x"}
:::

# 루비로 작성된 **반복자** {#루비로-작성된-반복자 .pattern-example-title-block-title}
::::

::::::::::: pattern-example-body
::: pattern-example-brief
**반복자**는 복잡한 데이터 구조의 내부 세부 정보를 노출하지 않고 해당
구조를 차례대로 순회할 수 있도록 하는 행동 디자인 패턴입니다.

반복자 덕분에 클라이언트들은 단일 반복기 인터페이스를 사용하여 유사한
방식으로 다른 컬렉션들의 요소들을 탐색할 수 있습니다.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ 반복자에 대하여 더 자세히 알아보세요 ](../../iterator.html){.btn
.btn-lg .btn-primary}
:::

:::::::: {.sidebar-navigation .with-scroll}
::: en-title
내비게이션
:::

::: en-intro
 [소개](#)
:::

::: en-intro
 [개념적인 예시](#example-0)
:::

::: en-file
 [main](#example-0--main-rb)
:::

::: en-file
 [output](#example-0--output-txt)
:::
::::::::

**복잡도:** []{.fa-stars}

**인기도:** []{.fa-stars}

**사용 예시들:** 이 패턴은 루비 코드에 자주 사용됩니다. 많은
프레임워크들과 라이브러리들은 이 패턴을 컬렉션을 탐색하는 표준 방법을
제공하기 위해 사용합니다.

**식별법:** 반복자는 그의 탐색 메서드들​(예: `next`, `previous` 등)​로
쉽게 인식할 수 있습니다. 또 반복자를 사용하는 클라이언트 코드는 반복자가
순회하는 컬렉션을 직접 접근하지 못할 수도 있습니다.
:::::::::::

[]{#example-0}

## 개념적인 예시 {#example-0-title}

이 예시는 **반복자** 디자인 패턴의 구조를 보여주고 다음 질문에 중점을
둡니다:

-   패턴은 어떤 클래스들로 구성되어 있나요?
-   이 클래스들은 어떤 역할을 하나요?
-   패턴의 요소들은 어떻게 서로 연관되어 있나요?

#### []{#example-0--main-rb .anchor} **main.rb:** 개념적인 예시

<figure class="code">
<pre class="code" lang="ruby"><code>class AlphabeticalOrderIterator
  # In Ruby, the Enumerable mixin provides classes with several traversal and
  # searching methods, and with the ability to sort. The class must provide a
  # method each, which yields successive members of the collection.
  include Enumerable

  # This attribute indicates the traversal direction.
  attr_accessor :reverse
  private :reverse

  # @return [Array]
  attr_accessor :collection
  private :collection

  # @param [Array] collection
  # @param [Boolean] reverse
  def initialize(collection, reverse: false)
    @collection = collection
    @reverse = reverse
  end

  def each(&amp;block)
    return @collection.reverse.each(&amp;block) if reverse

    @collection.each(&amp;block)
  end
end

class WordsCollection
  # @return [Array]
  attr_accessor :collection
  private :collection

  def initialize(collection = [])
    @collection = collection
  end

  # The `iterator` method returns the iterator object itself, by default we
  # return the iterator in ascending order.
  def iterator
    AlphabeticalOrderIterator.new(@collection)
  end

  # @return [AlphabeticalOrderIterator]
  def reverse_iterator
    AlphabeticalOrderIterator.new(@collection, reverse: true)
  end

  # @param [String] item
  def add_item(item)
    @collection &lt;&lt; item
  end
end

# The client code may or may not know about the Concrete Iterator or Collection
# classes, depending on the level of indirection you want to keep in your
# program.
collection = WordsCollection.new
collection.add_item(&#39;First&#39;)
collection.add_item(&#39;Second&#39;)
collection.add_item(&#39;Third&#39;)

puts &#39;Straight traversal:&#39;
collection.iterator.each { |item| puts item }
puts &quot;\n&quot;

puts &#39;Reverse traversal:&#39;
collection.reverse_iterator.each { |item| puts item }</code></pre>
</figure>

#### []{#example-0--output-txt .anchor} **output.txt:** 실행 결과

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

## 다른 언어로 작성된 **반복자**

[![C#으로 작성된
반복자](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "C#으로 작성된 반복자"){.prog-lang-link}
[![C++로 작성된
반복자](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "C++로 작성된 반복자"){.prog-lang-link}
[![Go로 작성된
반복자](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Go로 작성된 반복자"){.prog-lang-link}
[![자바로 작성된
반복자](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "자바로 작성된 반복자"){.prog-lang-link}
[![PHP로 작성된
반복자](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "PHP로 작성된 반복자"){.prog-lang-link}
[![파이썬으로 작성된
반복자](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "파이썬으로 작성된 반복자"){.prog-lang-link}
[![러스트로 작성된
반복자](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "러스트로 작성된 반복자"){.prog-lang-link}
[![스위프트로 작성된
반복자](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "스위프트로 작성된 반복자"){.prog-lang-link}
[![타입스크립트로 작성된
반복자](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "타입스크립트로 작성된 반복자"){.prog-lang-link}
:::::::::::::::

::::::: {.prom .banner-sidebar .banner .banner-striped .banner-examples .banner-removable .banner-removable-patterns data-id="DP: Examples-IDE" creative-id="examples-sidebar-ko" position="sidebar"}
:::::: {.banner-inner style="font-size: 14px; line-height: 21px;"}
::: banner-examples-img
[![](../../../../images/patterns/banners/examples-ide91ca.png?id=3115b4b548fb96b75974e2de8f4f49bc){width="215"
height="158" loading="lazy"
srcset="/images/patterns/banners/examples-ide-2x.png?id=93c007a6d157b97c28eb213f59d716ae 2x"}](../../book.html)
:::

::: banner-examples-fade
[ 예시들을 포함한 저장소](../../book.html){.btn .btn-secondary
.btn-block}
:::

::: {.banner-examples-text style="margin-top: 1rem;"}
전자책 **디자인 패턴에 뛰어들기**를 구매하면 IDE에서 바로 열 수 있는
수십 가지의 상세한 코드 예시들이 포함된 저장소를 사용할 수 있습니다.

[ 더 알아보기...](../../book.html){.btn .btn-secondary .btn-block}
:::
::::::
:::::::
:::::::::::::::::::::
::::::::::::::::::::::
