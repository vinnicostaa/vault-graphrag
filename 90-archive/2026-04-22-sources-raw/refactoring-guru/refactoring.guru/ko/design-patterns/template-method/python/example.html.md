:::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [디자인
패턴들](../../../design-patterns.html){.type} / [템플릿
메서드](../../template-method.html){.type} /
[파이썬](../../python.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![템플릿
메서드](../../../../images/patterns/cards/template-method-mini56e3.png?id=9f200248d88026d8e79d0f3dae411ab4){srcset="/images/patterns/cards/template-method-mini-2x.png?id=178bf56e39b3a1f548dd636076209c98 2x"}
:::

# 파이썬으로 작성된 **템플릿 메서드** {#파이썬으로-작성된-템플릿-메서드 .pattern-example-title-block-title}
::::

::::::::::: pattern-example-body
::: pattern-example-brief
**템플릿 메서드**는 기초 클래스에서 알고리즘의 골격을 정의할 수 있도록
하는 행동 디자인 패턴입니다. 또 이 패턴은 자식 클래스들이 전체
알고리즘의 구조를 변경하지 않고도 기본 알고리즘의 단계들을 오버라이드할
수 있도록 합니다.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ 템플릿 메서드에 대하여 더 자세히 알아보세요
](../../template-method.html){.btn .btn-lg .btn-primary}
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
 [main](#example-0--main-py)
:::

::: en-file
 [Output](#example-0--Output-txt)
:::
::::::::

**복잡도:** []{.fa-stars}

**인기도:** []{.fa-stars}

**사용 사례들:** 템플릿 메서드 패턴은 파이썬 프레임워크들에서 매우
일반적이며 개발자들은 종종 이 패턴을 사용하여 프레임워크 사용자들에게
상속을 사용하여 표준 기능들을 확장하는 간단한 수단을 제공합니다.

**식별:** 템플릿 메서드는 기초 클래스에 추상적이거나 비어 있는 다른 여러
메서드들을 호출하는 메서드가 있습니다.
:::::::::::

[]{#example-0}

## 개념적인 예시 {#example-0-title}

이 예시는 **템플릿 메서드**의 구조를 보여주고 다음 질문에 중점을 둡니다:

-   패턴은 어떤 클래스들로 구성되어 있나요?
-   이 클래스들은 어떤 역할을 하나요?
-   패턴의 요소들은 어떻게 서로 연관되어 있나요?

#### []{#example-0--main-py .anchor} **main.py:** 개념적인 예시

<figure class="code">
<pre class="code" lang="python"><code>from abc import ABC, abstractmethod


class AbstractClass(ABC):
    &quot;&quot;&quot;
    The Abstract Class defines a template method that contains a skeleton of
    some algorithm, composed of calls to (usually) abstract primitive
    operations.

    Concrete subclasses should implement these operations, but leave the
    template method itself intact.
    &quot;&quot;&quot;

    def template_method(self) -&gt; None:
        &quot;&quot;&quot;
        The template method defines the skeleton of an algorithm.
        &quot;&quot;&quot;

        self.base_operation1()
        self.required_operations1()
        self.base_operation2()
        self.hook1()
        self.required_operations2()
        self.base_operation3()
        self.hook2()

    # These operations already have implementations.

    def base_operation1(self) -&gt; None:
        print(&quot;AbstractClass says: I am doing the bulk of the work&quot;)

    def base_operation2(self) -&gt; None:
        print(&quot;AbstractClass says: But I let subclasses override some operations&quot;)

    def base_operation3(self) -&gt; None:
        print(&quot;AbstractClass says: But I am doing the bulk of the work anyway&quot;)

    # These operations have to be implemented in subclasses.

    @abstractmethod
    def required_operations1(self) -&gt; None:
        pass

    @abstractmethod
    def required_operations2(self) -&gt; None:
        pass

    # These are &quot;hooks.&quot; Subclasses may override them, but it&#39;s not mandatory
    # since the hooks already have default (but empty) implementation. Hooks
    # provide additional extension points in some crucial places of the
    # algorithm.

    def hook1(self) -&gt; None:
        pass

    def hook2(self) -&gt; None:
        pass


class ConcreteClass1(AbstractClass):
    &quot;&quot;&quot;
    Concrete classes have to implement all abstract operations of the base
    class. They can also override some operations with a default implementation.
    &quot;&quot;&quot;

    def required_operations1(self) -&gt; None:
        print(&quot;ConcreteClass1 says: Implemented Operation1&quot;)

    def required_operations2(self) -&gt; None:
        print(&quot;ConcreteClass1 says: Implemented Operation2&quot;)


class ConcreteClass2(AbstractClass):
    &quot;&quot;&quot;
    Usually, concrete classes override only a fraction of base class&#39;
    operations.
    &quot;&quot;&quot;

    def required_operations1(self) -&gt; None:
        print(&quot;ConcreteClass2 says: Implemented Operation1&quot;)

    def required_operations2(self) -&gt; None:
        print(&quot;ConcreteClass2 says: Implemented Operation2&quot;)

    def hook1(self) -&gt; None:
        print(&quot;ConcreteClass2 says: Overridden Hook1&quot;)


def client_code(abstract_class: AbstractClass) -&gt; None:
    &quot;&quot;&quot;
    The client code calls the template method to execute the algorithm. Client
    code does not have to know the concrete class of an object it works with, as
    long as it works with objects through the interface of their base class.
    &quot;&quot;&quot;

    # ...
    abstract_class.template_method()
    # ...


if __name__ == &quot;__main__&quot;:
    print(&quot;Same client code can work with different subclasses:&quot;)
    client_code(ConcreteClass1())
    print(&quot;&quot;)

    print(&quot;Same client code can work with different subclasses:&quot;)
    client_code(ConcreteClass2())</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** 실행 결과

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
#### 다음을 보세요

[파이썬으로 작성된 비지터 []{.fa
.fa-arrow-right}](../../visitor/python/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### 돌아가기

[[]{.fa .fa-arrow-left} 파이썬으로 작성된
전략](../../strategy/python/example.html){.btn .btn-default rel="prev"}
:::

## 다른 언어로 작성된 **템플릿 메서드**

[![C#으로 작성된 템플릿
메서드](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "C#으로 작성된 템플릿 메서드"){.prog-lang-link}
[![C++로 작성된 템플릿
메서드](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "C++로 작성된 템플릿 메서드"){.prog-lang-link}
[![Go로 작성된 템플릿
메서드](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Go로 작성된 템플릿 메서드"){.prog-lang-link}
[![자바로 작성된 템플릿
메서드](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "자바로 작성된 템플릿 메서드"){.prog-lang-link}
[![PHP로 작성된 템플릿
메서드](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "PHP로 작성된 템플릿 메서드"){.prog-lang-link}
[![루비로 작성된 템플릿
메서드](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "루비로 작성된 템플릿 메서드"){.prog-lang-link}
[![러스트로 작성된 템플릿
메서드](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "러스트로 작성된 템플릿 메서드"){.prog-lang-link}
[![스위프트로 작성된 템플릿
메서드](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "스위프트로 작성된 템플릿 메서드"){.prog-lang-link}
[![타입스크립트로 작성된 템플릿
메서드](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "타입스크립트로 작성된 템플릿 메서드"){.prog-lang-link}
:::::::::::::::::

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
:::::::::::::::::::::::
::::::::::::::::::::::::
