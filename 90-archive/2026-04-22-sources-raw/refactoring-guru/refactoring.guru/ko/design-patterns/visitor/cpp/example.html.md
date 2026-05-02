:::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [디자인
패턴들](../../../design-patterns.html){.type} /
[비지터](../../visitor.html){.type} / [C++](../../cpp.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![비지터](../../../../images/patterns/cards/visitor-mini0a1c.png?id=854a35a62963bec1d75eab996918989b){srcset="/images/patterns/cards/visitor-mini-2x.png?id=9b87f3f3b772f434b28a25876829b504 2x"}
:::

# C++로 작성된 **비지터** {#c로-작성된-비지터 .pattern-example-title-block-title}
::::

::::::::::: pattern-example-body
::: pattern-example-brief
**비지터**는 기존 코드를 변경하지 않고 기존 클래스 계층구조에 새로운
행동들을 추가할 수 있도록 하는 행동 디자인 패턴입니다.

> 제 설명글 \[비지터와 이중 디스패치\]{비지터와 이중 디스패치}에서 왜
> 단순히 비지터들을 메서드 오버로딩으로 대체할 수 없는지 알아보세요.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ 비지터에 대하여 더 자세히 알아보세요 ](../../visitor.html){.btn
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
 [main](#example-0--main-cc)
:::

::: en-file
 [Output](#example-0--Output-txt)
:::
::::::::

**복잡도:** []{.fa-stars}

**인기도:** []{.fa-stars}

**사용 사례들:** 비지터는 복잡하고 적용 범위가 좁기 때문에 매우 일반적인
패턴이 아닙니다.
:::::::::::

[]{#example-0}

## 개념적인 예시 {#example-0-title}

이 예시는 **비지터** 패턴의 구조를 보여주고 다음 질문에 중점을 둡니다:

-   패턴은 어떤 클래스들로 구성되어 있나요?
-   이 클래스들은 어떤 역할을 하나요?
-   패턴의 요소들은 어떻게 서로 연관되어 있나요?

#### []{#example-0--main-cc .anchor} **main.cc:** 개념적인 예시

<figure class="code">
<pre class="code" lang="cpp"><code>/**
 * The Visitor Interface declares a set of visiting methods that correspond to
 * component classes. The signature of a visiting method allows the visitor to
 * identify the exact class of the component that it&#39;s dealing with.
 */
class ConcreteComponentA;
class ConcreteComponentB;

class Visitor {
 public:
  virtual void VisitConcreteComponentA(const ConcreteComponentA *element) const = 0;
  virtual void VisitConcreteComponentB(const ConcreteComponentB *element) const = 0;
};

/**
 * The Component interface declares an `accept` method that should take the base
 * visitor interface as an argument.
 */

class Component {
 public:
  virtual ~Component() {}
  virtual void Accept(Visitor *visitor) const = 0;
};

/**
 * Each Concrete Component must implement the `Accept` method in such a way that
 * it calls the visitor&#39;s method corresponding to the component&#39;s class.
 */
class ConcreteComponentA : public Component {
  /**
   * Note that we&#39;re calling `visitConcreteComponentA`, which matches the
   * current class name. This way we let the visitor know the class of the
   * component it works with.
   */
 public:
  void Accept(Visitor *visitor) const override {
    visitor-&gt;VisitConcreteComponentA(this);
  }
  /**
   * Concrete Components may have special methods that don&#39;t exist in their base
   * class or interface. The Visitor is still able to use these methods since
   * it&#39;s aware of the component&#39;s concrete class.
   */
  std::string ExclusiveMethodOfConcreteComponentA() const {
    return &quot;A&quot;;
  }
};

class ConcreteComponentB : public Component {
  /**
   * Same here: visitConcreteComponentB =&gt; ConcreteComponentB
   */
 public:
  void Accept(Visitor *visitor) const override {
    visitor-&gt;VisitConcreteComponentB(this);
  }
  std::string SpecialMethodOfConcreteComponentB() const {
    return &quot;B&quot;;
  }
};

/**
 * Concrete Visitors implement several versions of the same algorithm, which can
 * work with all concrete component classes.
 *
 * You can experience the biggest benefit of the Visitor pattern when using it
 * with a complex object structure, such as a Composite tree. In this case, it
 * might be helpful to store some intermediate state of the algorithm while
 * executing visitor&#39;s methods over various objects of the structure.
 */
class ConcreteVisitor1 : public Visitor {
 public:
  void VisitConcreteComponentA(const ConcreteComponentA *element) const override {
    std::cout &lt;&lt; element-&gt;ExclusiveMethodOfConcreteComponentA() &lt;&lt; &quot; + ConcreteVisitor1\n&quot;;
  }

  void VisitConcreteComponentB(const ConcreteComponentB *element) const override {
    std::cout &lt;&lt; element-&gt;SpecialMethodOfConcreteComponentB() &lt;&lt; &quot; + ConcreteVisitor1\n&quot;;
  }
};

class ConcreteVisitor2 : public Visitor {
 public:
  void VisitConcreteComponentA(const ConcreteComponentA *element) const override {
    std::cout &lt;&lt; element-&gt;ExclusiveMethodOfConcreteComponentA() &lt;&lt; &quot; + ConcreteVisitor2\n&quot;;
  }
  void VisitConcreteComponentB(const ConcreteComponentB *element) const override {
    std::cout &lt;&lt; element-&gt;SpecialMethodOfConcreteComponentB() &lt;&lt; &quot; + ConcreteVisitor2\n&quot;;
  }
};
/**
 * The client code can run visitor operations over any set of elements without
 * figuring out their concrete classes. The accept operation directs a call to
 * the appropriate operation in the visitor object.
 */
void ClientCode(std::array&lt;const Component *, 2&gt; components, Visitor *visitor) {
  // ...
  for (const Component *comp : components) {
    comp-&gt;Accept(visitor);
  }
  // ...
}

int main() {
  std::array&lt;const Component *, 2&gt; components = {new ConcreteComponentA, new ConcreteComponentB};
  std::cout &lt;&lt; &quot;The client code works with all visitors via the base Visitor interface:\n&quot;;
  ConcreteVisitor1 *visitor1 = new ConcreteVisitor1;
  ClientCode(components, visitor1);
  std::cout &lt;&lt; &quot;\n&quot;;
  std::cout &lt;&lt; &quot;It allows the same client code to work with different types of visitors:\n&quot;;
  ConcreteVisitor2 *visitor2 = new ConcreteVisitor2;
  ClientCode(components, visitor2);

  for (const Component *comp : components) {
    delete comp;
  }
  delete visitor1;
  delete visitor2;

  return 0;
}</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** 실행 결과

<figure class="code">
<pre class="code" lang="output"><code>The client code works with all visitors via the base Visitor interface:
A + ConcreteVisitor1
B + ConcreteVisitor1

It allows the same client code to work with different types of visitors:
A + ConcreteVisitor2
B + ConcreteVisitor2</code></pre>
</figure>

::: next
#### 다음을 보세요

[Go로 작성된 디자인 패턴들 []{.fa .fa-arrow-right}](../../go.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### 돌아가기

[[]{.fa .fa-arrow-left} C++로 작성된 템플릿
메서드](../../template-method/cpp/example.html){.btn .btn-default
rel="prev"}
:::

## 다른 언어로 작성된 **비지터**

[![C#으로 작성된
비지터](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "C#으로 작성된 비지터"){.prog-lang-link}
[![Go로 작성된
비지터](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Go로 작성된 비지터"){.prog-lang-link}
[![자바로 작성된
비지터](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "자바로 작성된 비지터"){.prog-lang-link}
[![PHP로 작성된
비지터](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "PHP로 작성된 비지터"){.prog-lang-link}
[![파이썬으로 작성된
비지터](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "파이썬으로 작성된 비지터"){.prog-lang-link}
[![루비로 작성된
비지터](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "루비로 작성된 비지터"){.prog-lang-link}
[![러스트로 작성된
비지터](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "러스트로 작성된 비지터"){.prog-lang-link}
[![스위프트로 작성된
비지터](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "스위프트로 작성된 비지터"){.prog-lang-link}
[![타입스크립트로 작성된
비지터](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "타입스크립트로 작성된 비지터"){.prog-lang-link}
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
