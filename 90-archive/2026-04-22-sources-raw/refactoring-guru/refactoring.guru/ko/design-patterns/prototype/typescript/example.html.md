:::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [디자인
패턴들](../../../design-patterns.html){.type} /
[프로토타입](../../prototype.html){.type} /
[타입스크립트](../../typescript.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![프로토타입](../../../../images/patterns/cards/prototype-mini6540.png?id=bc3046bb39ff36574c08d49839fd1c8e){srcset="/images/patterns/cards/prototype-mini-2x.png?id=b871f789a736e7efbb1cd082d2de6398 2x"}
:::

# 타입스크립트로 작성된 **프로토타입** {#타입스크립트로-작성된-프로토타입 .pattern-example-title-block-title}
::::

::::::::::: pattern-example-body
::: pattern-example-brief
**프로토타입**은 객체들​(복잡한 객체 포함)​을 그의 특정 클래스들에
결합하지 않고 복제할 수 있도록 하는 생성 디자인 패턴입니다.

모든 프로토타입 클래스들은 객체들의 구상 클래스들을 알 수 없는 경우에도
해당 객체들을 복사할 수 있도록 하는 공통 인터페이스가 있어야 합니다.
프로토타입 객체들은 전체 복사본들을 생성할 수 있습니다. 왜냐하면 같은
클래스의 객체들은 서로의 비공개 필드들에 접근할 수 있기 때문입니다.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ 프로토타입에 대하여 더 자세히 알아보세요 ](../../prototype.html){.btn
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
 [index](#example-0--index-ts)
:::

::: en-file
 [Output](#example-0--Output-txt)
:::
::::::::

**복잡도:** []{.fa-stars}

**인기도:** []{.fa-stars}

**사용 사례들:** 프로토타입 패턴은 자바스크립트 고유의 `Object.assign()`
메서드를 통해 자바스크립트 코드에서 바로 사용할 수 있습니다.

**식별:** 프로토타입은 `clone` 또는 `copy` 등의 메서드들의 유무로 식별할
수 있습니다.
:::::::::::

[]{#example-0}

## 개념적인 예시 {#example-0-title}

이 예시는 **프로토타입** 디자인 패턴의 구조를 보여주고 다음 질문에
중점을 둡니다:

-   패턴은 어떤 클래스들로 구성되어 있나요?
-   이 클래스들은 어떤 역할을 하나요?
-   패턴의 요소들은 어떻게 서로 연관되어 있나요?

#### []{#example-0--index-ts .anchor} **index.ts:** 개념적인 예시

<figure class="code">
<pre class="code" lang="typescript"><code>/**
 * The example class that has cloning ability. We&#39;ll see how the values of field
 * with different types will be cloned.
 */
class Prototype {
    public primitive: any;
    public component: object;
    public circularReference: ComponentWithBackReference;

    public clone(): this {
        const clone = Object.create(this);

        clone.component = Object.create(this.component);

        // Cloning an object that has a nested object with backreference
        // requires special treatment. After the cloning is completed, the
        // nested object should point to the cloned object, instead of the
        // original object. Spread operator can be handy for this case.
        clone.circularReference = new ComponentWithBackReference(clone);

        return clone;
    }
}

class ComponentWithBackReference {
    public prototype;

    constructor(prototype: Prototype) {
        this.prototype = prototype;
    }
}

/**
 * The client code.
 */
function clientCode() {
    const p1 = new Prototype();
    p1.primitive = 245;
    p1.component = new Date();
    p1.circularReference = new ComponentWithBackReference(p1);

    const p2 = p1.clone();
    if (p1.primitive === p2.primitive) {
        console.log(&#39;Primitive field values have been carried over to a clone. Yay!&#39;);
    } else {
        console.log(&#39;Primitive field values have not been copied. Booo!&#39;);
    }
    if (p1.component === p2.component) {
        console.log(&#39;Simple component has not been cloned. Booo!&#39;);
    } else {
        console.log(&#39;Simple component has been cloned. Yay!&#39;);
    }

    if (p1.circularReference === p2.circularReference) {
        console.log(&#39;Component with back reference has not been cloned. Booo!&#39;);
    } else {
        console.log(&#39;Component with back reference has been cloned. Yay!&#39;);
    }

    if (p1.circularReference.prototype === p2.circularReference.prototype) {
        console.log(&#39;Component with back reference is linked to original object. Booo!&#39;);
    } else {
        console.log(&#39;Component with back reference is linked to the clone. Yay!&#39;);
    }
}

clientCode();</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** 실행 결과

<figure class="code">
<pre class="code" lang="output"><code>Primitive field values have been carried over to a clone. Yay!
Simple component has been cloned. Yay!
Component with back reference has been cloned. Yay!
Component with back reference is linked to the clone. Yay!</code></pre>
</figure>

::: next
#### 다음을 보세요

[타입스크립트로 작성된 싱글턴 []{.fa
.fa-arrow-right}](../../singleton/typescript/example.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### 돌아가기

[[]{.fa .fa-arrow-left} 타입스크립트로 작성된 팩토리
메서드](../../factory-method/typescript/example.html){.btn .btn-default
rel="prev"}
:::

## 다른 언어로 작성된 **프로토타입**

[![C#으로 작성된
프로토타입](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "C#으로 작성된 프로토타입"){.prog-lang-link}
[![C++로 작성된
프로토타입](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "C++로 작성된 프로토타입"){.prog-lang-link}
[![Go로 작성된
프로토타입](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Go로 작성된 프로토타입"){.prog-lang-link}
[![자바로 작성된
프로토타입](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "자바로 작성된 프로토타입"){.prog-lang-link}
[![PHP로 작성된
프로토타입](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "PHP로 작성된 프로토타입"){.prog-lang-link}
[![파이썬으로 작성된
프로토타입](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "파이썬으로 작성된 프로토타입"){.prog-lang-link}
[![루비로 작성된
프로토타입](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "루비로 작성된 프로토타입"){.prog-lang-link}
[![러스트로 작성된
프로토타입](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "러스트로 작성된 프로토타입"){.prog-lang-link}
[![스위프트로 작성된
프로토타입](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "스위프트로 작성된 프로토타입"){.prog-lang-link}
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
