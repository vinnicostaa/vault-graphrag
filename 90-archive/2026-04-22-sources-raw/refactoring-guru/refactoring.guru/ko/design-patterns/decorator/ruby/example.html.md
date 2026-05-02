:::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [디자인
패턴들](../../../design-patterns.html){.type} /
[데코레이터](../../decorator.html){.type} /
[루비](../../ruby.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![데코레이터](../../../../images/patterns/cards/decorator-mini6adb.png?id=d30458908e315af195cb183bc52dbef9){srcset="/images/patterns/cards/decorator-mini-2x.png?id=3b58e540d7d28523080cad341ed9b2e9 2x"}
:::

# 루비로 작성된 **데코레이터** {#루비로-작성된-데코레이터 .pattern-example-title-block-title}
::::

::::::::::: pattern-example-body
::: pattern-example-brief
**데코레이터**는 구조 패턴이며 새로운 행동들을 특수 래퍼 객체들 내에
넣어서 이러한 행동들을 객체들에 동적으로 추가할 수 있도록 합니다.

데코레이터를 사용하여 객체들을 제한 없이 래핑할 수 있습니다. 왜냐하면
대상 객체들과 데코레이터들은 같은 인터페이스를 따르기 때문입니다. 결과
객체는 모든 래퍼의 스태킹된 행동을 가질 것입니다.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ 데코레이터에 대하여 더 자세히 알아보세요 ](../../decorator.html){.btn
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

**사용 예시들:** 데코레이터는 루비 코드, 특히 스트림과 관련된 코드에서
꽤 표준적입니다.

**식별:** 데코레이터는 같은 클래스의 객체 또는 인터페이스를 현재
클래스로 수락하는 생성 메서드들 또는 생성자들로 인식할 수 있습니다.
:::::::::::

[]{#example-0}

## 개념적인 예시 {#example-0-title}

이 예시는 **데코레이터** 패턴의 구조를 보여주고 다음 질문에 중점을
둡니다:

-   패턴은 어떤 클래스들로 구성되어 있나요?
-   이 클래스들은 어떤 역할을 하나요?
-   패턴의 요소들은 어떻게 서로 연관되어 있나요?

#### []{#example-0--main-rb .anchor} **main.rb:** 개념적인 예시

<figure class="code">
<pre class="code" lang="ruby"><code># The base Component interface defines operations that can be altered by
# decorators.
class Component
  # @return [String]
  def operation
    raise NotImplementedError, &quot;#{self.class} has not implemented method &#39;#{__method__}&#39;&quot;
  end
end

# Concrete Components provide default implementations of the operations. There
# might be several variations of these classes.
class ConcreteComponent &lt; Component
  # @return [String]
  def operation
    &#39;ConcreteComponent&#39;
  end
end

# The base Decorator class follows the same interface as the other components.
# The primary purpose of this class is to define the wrapping interface for all
# concrete decorators. The default implementation of the wrapping code might
# include a field for storing a wrapped component and the means to initialize
# it.
class Decorator &lt; Component
  attr_accessor :component

  # @param [Component] component
  def initialize(component)
    @component = component
  end

  # The Decorator delegates all work to the wrapped component.
  def operation
    @component.operation
  end
end

# Concrete Decorators call the wrapped object and alter its result in some way.
class ConcreteDecoratorA &lt; Decorator
  # Decorators may call parent implementation of the operation, instead of
  # calling the wrapped object directly. This approach simplifies extension of
  # decorator classes.
  def operation
    &quot;ConcreteDecoratorA(#{@component.operation})&quot;
  end
end

# Decorators can execute their behavior either before or after the call to a
# wrapped object.
class ConcreteDecoratorB &lt; Decorator
  # @return [String]
  def operation
    &quot;ConcreteDecoratorB(#{@component.operation})&quot;
  end
end

# The client code works with all objects using the Component interface. This way
# it can stay independent of the concrete classes of components it works with.
def client_code(component)
  # ...

  print &quot;RESULT: #{component.operation}&quot;

  # ...
end

# This way the client code can support both simple components...
simple = ConcreteComponent.new
puts &#39;Client: I\&#39;ve got a simple component:&#39;
client_code(simple)
puts &quot;\n\n&quot;

# ...as well as decorated ones.
#
# Note how decorators can wrap not only simple components but the other
# decorators as well.
decorator1 = ConcreteDecoratorA.new(simple)
decorator2 = ConcreteDecoratorB.new(decorator1)
puts &#39;Client: Now I\&#39;ve got a decorated component:&#39;
client_code(decorator2)</code></pre>
</figure>

#### []{#example-0--output-txt .anchor} **output.txt:** 실행 결과

<figure class="code">
<pre class="code" lang="output"><code>Client: I&#39;ve got a simple component:
RESULT: ConcreteComponent

Client: Now I&#39;ve got a decorated component:
RESULT: ConcreteDecoratorB(ConcreteDecoratorA(ConcreteComponent))</code></pre>
</figure>

## 다른 언어로 작성된 **데코레이터**

[![C#으로 작성된
데코레이터](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "C#으로 작성된 데코레이터"){.prog-lang-link}
[![C++로 작성된
데코레이터](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "C++로 작성된 데코레이터"){.prog-lang-link}
[![Go로 작성된
데코레이터](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Go로 작성된 데코레이터"){.prog-lang-link}
[![자바로 작성된
데코레이터](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "자바로 작성된 데코레이터"){.prog-lang-link}
[![PHP로 작성된
데코레이터](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "PHP로 작성된 데코레이터"){.prog-lang-link}
[![파이썬으로 작성된
데코레이터](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "파이썬으로 작성된 데코레이터"){.prog-lang-link}
[![러스트로 작성된
데코레이터](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "러스트로 작성된 데코레이터"){.prog-lang-link}
[![스위프트로 작성된
데코레이터](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "스위프트로 작성된 데코레이터"){.prog-lang-link}
[![타입스크립트로 작성된
데코레이터](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "타입스크립트로 작성된 데코레이터"){.prog-lang-link}
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
