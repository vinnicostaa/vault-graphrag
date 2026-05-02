:::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [디자인
패턴들](../../../design-patterns.html){.type} /
[어댑터](../../adapter.html){.type} / [루비](../../ruby.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![어댑터](../../../../images/patterns/cards/adapter-minid866.png?id=b2ee4f681fb589be5a0685b94692aebb){srcset="/images/patterns/cards/adapter-mini-2x.png?id=8274d99afbbe9c63bfbfd0d68ceeffc7 2x"}
:::

# 루비로 작성된 **어댑터** {#루비로-작성된-어댑터 .pattern-example-title-block-title}
::::

::::::::::: pattern-example-body
::: pattern-example-brief
**어댑터**는 구조 디자인 패턴이며, 호환되지 않는 객체들이 협업할 수
있도록 합니다.

어댑터는 두 객체 사이의 래퍼 역할을 합니다. 하나의 객체에 대한 호출을
캐치하고 두 번째 객체가 인식할 수 있는 형식과 인터페이스로 변환합니다.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ 어댑터에 대하여 더 자세히 알아보세요 ](../../adapter.html){.btn
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

**사용 예시들:** 어댑터 패턴은 루비 코드에 자주 사용됩니다. 특히 일부
레거시 코드를 기반으로 하는 시스템에서 매우 자주 사용됩니다. 이러한 경우
어댑터는 레거시 코드가 현대식 클래스들과 함께 작동하도록 합니다.

**식별:** 어댑터는 다른 추상/인터페이스 유형의 인스턴스를 받는 생성자의
존재여부로 인식할 수 있습니다. 어댑터가 그의 메서드들에 대한 호출을
수신하면, 어댑터는 매개변수들을 적절한 형식으로 변환한 다음 해당 호출을
래핑 된 객체의 하나 또는 여러 메서드들에 전달합니다.
:::::::::::

[]{#example-0}

## 개념적인 예시 {#example-0-title}

이 예시는 **어댑터** 디자인 패턴의 구조를 보여주고 다음 질문에 중점을
둡니다:

-   패턴은 어떤 클래스들로 구성되어 있나요?
-   이 클래스들은 어떤 역할을 하나요?
-   패턴의 요소들은 어떻게 서로 연관되어 있나요?

#### []{#example-0--main-rb .anchor} **main.rb:** 개념적인 예시

<figure class="code">
<pre class="code" lang="ruby"><code># The Target defines the domain-specific interface used by the client code.
class Target
  # @return [String]
  def request
    &#39;Target: The default target\&#39;s behavior.&#39;
  end
end

# The Adaptee contains some useful behavior, but its interface is incompatible
# with the existing client code. The Adaptee needs some adaptation before the
# client code can use it.
class Adaptee
  # @return [String]
  def specific_request
    &#39;.eetpadA eht fo roivaheb laicepS&#39;
  end
end

# The Adapter makes the Adaptee&#39;s interface compatible with the Target&#39;s
# interface.
class Adapter &lt; Target
  # @param [Adaptee] adaptee
  def initialize(adaptee)
    @adaptee = adaptee
  end

  def request
    &quot;Adapter: (TRANSLATED) #{@adaptee.specific_request.reverse!}&quot;
  end
end

# The client code supports all classes that follow the Target interface.
def client_code(target)
  print target.request
end

puts &#39;Client: I can work just fine with the Target objects:&#39;
target = Target.new
client_code(target)
puts &quot;\n\n&quot;

adaptee = Adaptee.new
puts &#39;Client: The Adaptee class has a weird interface. See, I don\&#39;t understand it:&#39;
puts &quot;Adaptee: #{adaptee.specific_request}&quot;
puts &quot;\n&quot;

puts &#39;Client: But I can work with it via the Adapter:&#39;
adapter = Adapter.new(adaptee)
client_code(adapter)</code></pre>
</figure>

#### []{#example-0--output-txt .anchor} **output.txt:** 실행 결과

<figure class="code">
<pre class="code" lang="output"><code>Client: I can work just fine with the Target objects:
Target: The default target&#39;s behavior.

Client: The Adaptee class has a weird interface. See, I don&#39;t understand it:
Adaptee: .eetpadA eht fo roivaheb laicepS

Client: But I can work with it via the Adapter:
Adapter: (TRANSLATED) Special behavior of the Adaptee.</code></pre>
</figure>

## 다른 언어로 작성된 **어댑터**

[![C#으로 작성된
어댑터](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "C#으로 작성된 어댑터"){.prog-lang-link}
[![C++로 작성된
어댑터](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "C++로 작성된 어댑터"){.prog-lang-link}
[![Go로 작성된
어댑터](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Go로 작성된 어댑터"){.prog-lang-link}
[![자바로 작성된
어댑터](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "자바로 작성된 어댑터"){.prog-lang-link}
[![PHP로 작성된
어댑터](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "PHP로 작성된 어댑터"){.prog-lang-link}
[![파이썬으로 작성된
어댑터](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "파이썬으로 작성된 어댑터"){.prog-lang-link}
[![러스트로 작성된
어댑터](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "러스트로 작성된 어댑터"){.prog-lang-link}
[![스위프트로 작성된
어댑터](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "스위프트로 작성된 어댑터"){.prog-lang-link}
[![타입스크립트로 작성된
어댑터](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "타입스크립트로 작성된 어댑터"){.prog-lang-link}
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
