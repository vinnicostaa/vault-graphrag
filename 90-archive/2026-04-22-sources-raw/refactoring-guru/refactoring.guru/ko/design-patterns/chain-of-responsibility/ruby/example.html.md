:::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [디자인
패턴들](../../../design-patterns.html){.type} / [책임
연쇄](../../chain-of-responsibility.html){.type} /
[루비](../../ruby.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![책임
연쇄](../../../../images/patterns/cards/chain-of-responsibility-mini6d42.png?id=36d85eba8d14986f053123de17aac7a7){srcset="/images/patterns/cards/chain-of-responsibility-mini-2x.png?id=8c81f7069e51259b2443801b91135f7f 2x"}
:::

# 루비로 작성된 **책임 연쇄** {#루비로-작성된-책임-연쇄 .pattern-example-title-block-title}
::::

::::::::::: pattern-example-body
::: pattern-example-brief
**책임 연쇄** 패턴은 핸들러 중 하나가 요청을 처리할 때까지 핸들러들의
체인​(사슬)​을 따라 요청을 전달할 수 있게 해주는 행동 디자인 패턴입니다.

이 패턴은 발신자 클래스를 수신자들의 구상 클래스들에 연결하지 않고도
여러 객체가 요청을 처리할 수 있도록 합니다. 체인은 표준 핸들러
인터페이스를 따르는 모든 핸들러와 런타임 때 동적으로 구성될 수 있습니다.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ 책임 연쇄에 대하여 더 자세히 알아보세요
](../../chain-of-responsibility.html){.btn .btn-lg .btn-primary}
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

**사용 예시들:** 책임 연쇄 패턴은 루비 코드에 매우 일반적이며, 당신의
코드가 필터, 이벤터 체인 등과 같은 객체 체인과 함께 작동할 때 특히
유용합니다.

**식별:** 패턴의 모든 객체는 공통 인터페이스를 따르며, 다른 객체들의
같은 메서드들을 간접적으로 호출하는 한 객체 그룹의 행동 메서드들이
있습니다.
:::::::::::

[]{#example-0}

## 개념적인 예시 {#example-0-title}

이 예시는 **책임 연쇄** 패턴의 구조를 보여주고 다음 질문에 중점을
둡니다:

-   패턴은 어떤 클래스들로 구성되어 있나요?
-   이 클래스들은 어떤 역할을 하나요?
-   패턴의 요소들은 어떻게 서로 연관되어 있나요?

#### []{#example-0--main-rb .anchor} **main.rb:** 개념적인 예시

<figure class="code">
<pre class="code" lang="ruby"><code># The Handler interface declares a method for building the chain of handlers. It
# also declares a method for executing a request.
class Handler
  # @abstract
  #
  # @param [Handler] handler
  def next_handler=(handler)
    raise NotImplementedError, &quot;#{self.class} has not implemented method &#39;#{__method__}&#39;&quot;
  end

  # @abstract
  #
  # @param [String] request
  #
  # @return [String, nil]
  def handle(request)
    raise NotImplementedError, &quot;#{self.class} has not implemented method &#39;#{__method__}&#39;&quot;
  end
end

# The default chaining behavior can be implemented inside a base handler class.
class AbstractHandler &lt; Handler
  # @return [Handler]
  attr_writer :next_handler

  # @param [Handler] handler
  #
  # @return [Handler]
  def next_handler(handler)
    @next_handler = handler
    # Returning a handler from here will let us link handlers in a convenient
    # way like this:
    # monkey.next_handler(squirrel).next_handler(dog)
    handler
  end

  # @abstract
  #
  # @param [String] request
  #
  # @return [String, nil]
  def handle(request)
    return @next_handler.handle(request) if @next_handler

    nil
  end
end

# All Concrete Handlers either handle a request or pass it to the next handler
# in the chain.
class MonkeyHandler &lt; AbstractHandler
  # @param [String] request
  #
  # @return [String, nil]
  def handle(request)
    if request == &#39;Banana&#39;
      &quot;Monkey: I&#39;ll eat the #{request}&quot;
    else
      super(request)
    end
  end
end

class SquirrelHandler &lt; AbstractHandler
  # @param [String] request
  #
  # @return [String, nil]
  def handle(request)
    if request == &#39;Nut&#39;
      &quot;Squirrel: I&#39;ll eat the #{request}&quot;
    else
      super(request)
    end
  end
end

class DogHandler &lt; AbstractHandler
  # @param [String] request
  #
  # @return [String, nil]
  def handle(request)
    if request == &#39;MeatBall&#39;
      &quot;Dog: I&#39;ll eat the #{request}&quot;
    else
      super(request)
    end
  end
end

# The client code is usually suited to work with a single handler. In most
# cases, it is not even aware that the handler is part of a chain.
def client_code(handler)
  [&#39;Nut&#39;, &#39;Banana&#39;, &#39;Cup of coffee&#39;].each do |food|
    puts &quot;\nClient: Who wants a #{food}?&quot;
    result = handler.handle(food)
    if result
      print &quot;  #{result}&quot;
    else
      print &quot;  #{food} was left untouched.&quot;
    end
  end
end

monkey = MonkeyHandler.new
squirrel = SquirrelHandler.new
dog = DogHandler.new

monkey.next_handler(squirrel).next_handler(dog)

# The client should be able to send a request to any handler, not just the first
# one in the chain.
puts &#39;Chain: Monkey &gt; Squirrel &gt; Dog&#39;
client_code(monkey)
puts &quot;\n\n&quot;

puts &#39;Subchain: Squirrel &gt; Dog&#39;
client_code(squirrel)</code></pre>
</figure>

#### []{#example-0--output-txt .anchor} **output.txt:** 실행 결과

<figure class="code">
<pre class="code" lang="output"><code>Chain: Monkey &gt; Squirrel &gt; Dog

Client: Who wants a Nut?
  Squirrel: I&#39;ll eat the Nut
Client: Who wants a Banana?
  Monkey: I&#39;ll eat the Banana
Client: Who wants a Cup of coffee?
  Cup of coffee was left untouched.

Subchain: Squirrel &gt; Dog

Client: Who wants a Nut?
  Squirrel: I&#39;ll eat the Nut
Client: Who wants a Banana?
  Banana was left untouched.
Client: Who wants a Cup of coffee?
  Cup of coffee was left untouched.</code></pre>
</figure>

## 다른 언어로 작성된 **책임 연쇄**

[![C#으로 작성된 책임
연쇄](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "C#으로 작성된 책임 연쇄"){.prog-lang-link}
[![C++로 작성된 책임
연쇄](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "C++로 작성된 책임 연쇄"){.prog-lang-link}
[![Go로 작성된 책임
연쇄](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Go로 작성된 책임 연쇄"){.prog-lang-link}
[![자바로 작성된 책임
연쇄](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "자바로 작성된 책임 연쇄"){.prog-lang-link}
[![PHP로 작성된 책임
연쇄](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "PHP로 작성된 책임 연쇄"){.prog-lang-link}
[![파이썬으로 작성된 책임
연쇄](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "파이썬으로 작성된 책임 연쇄"){.prog-lang-link}
[![러스트로 작성된 책임
연쇄](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "러스트로 작성된 책임 연쇄"){.prog-lang-link}
[![스위프트로 작성된 책임
연쇄](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "스위프트로 작성된 책임 연쇄"){.prog-lang-link}
[![타입스크립트로 작성된 책임
연쇄](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "타입스크립트로 작성된 책임 연쇄"){.prog-lang-link}
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
