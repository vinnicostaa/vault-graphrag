:::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [디자인
패턴들](../../../design-patterns.html){.type} /
[퍼사드](../../facade.html){.type} /
[타입스크립트](../../typescript.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![퍼사드](../../../../images/patterns/cards/facade-mini14fd.png?id=71ad6fa98b168c11cb3a1a9517dedf78){srcset="/images/patterns/cards/facade-mini-2x.png?id=d4cc6a5d81a31143cc665f7ac1481ac8 2x"}
:::

# 타입스크립트로 작성된 **퍼사드** {#타입스크립트로-작성된-퍼사드 .pattern-example-title-block-title}
::::

::::::::::: pattern-example-body
::: pattern-example-brief
**퍼사드**는 구조 디자인 패턴이며 간단한 (그러나 제한된 인터페이스를
클래스, 라이브러리 또는 프레임워크로 구성된 복잡한 시스템에 제공합니다.

퍼사드는 앱의 전반적인 복잡성을 줄이는 동시에 원치 않는 의존성들을 한
곳으로 옮기는 것을 돕습니다.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ 퍼사드에 대하여 더 자세히 알아보세요 ](../../facade.html){.btn .btn-lg
.btn-primary}
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

**사용 사례들:** 퍼사드 패턴은 일반적으로 타입스크립트 코드로 작성된
앱에서 사용되며 또 복잡한 라이브러리 및 API와 작업할 때 특히 편리합니다.

**식별:** 퍼사드는 인터페이스가 간단하나 대부분의 작업을 다른 클래스들에
위임하는 클래스의 존재여부로 식별할 수 있으며 일반적으로 퍼사드들은
사용하는 객체들의 전체 수명 주기를 관리합니다.
:::::::::::

[]{#example-0}

## 개념적인 예시 {#example-0-title}

이 예시는 **퍼사드** 패턴의 구조를 보여주고 다음 질문에 중점을 둡니다:

-   패턴은 어떤 클래스들로 구성되어 있나요?
-   이 클래스들은 어떤 역할을 하나요?
-   패턴의 요소들은 어떻게 서로 연관되어 있나요?

#### []{#example-0--index-ts .anchor} **index.ts:** 개념적인 예시

<figure class="code">
<pre class="code" lang="typescript"><code>/**
 * The Facade class provides a simple interface to the complex logic of one or
 * several subsystems. The Facade delegates the client requests to the
 * appropriate objects within the subsystem. The Facade is also responsible for
 * managing their lifecycle. All of this shields the client from the undesired
 * complexity of the subsystem.
 */
class Facade {
    protected subsystem1: Subsystem1;

    protected subsystem2: Subsystem2;

    /**
     * Depending on your application&#39;s needs, you can provide the Facade with
     * existing subsystem objects or force the Facade to create them on its own.
     */
    constructor(subsystem1?: Subsystem1, subsystem2?: Subsystem2) {
        this.subsystem1 = subsystem1 || new Subsystem1();
        this.subsystem2 = subsystem2 || new Subsystem2();
    }

    /**
     * The Facade&#39;s methods are convenient shortcuts to the sophisticated
     * functionality of the subsystems. However, clients get only to a fraction
     * of a subsystem&#39;s capabilities.
     */
    public operation(): string {
        let result = &#39;Facade initializes subsystems:\n&#39;;
        result += this.subsystem1.operation1();
        result += this.subsystem2.operation1();
        result += &#39;Facade orders subsystems to perform the action:\n&#39;;
        result += this.subsystem1.operationN();
        result += this.subsystem2.operationZ();

        return result;
    }
}

/**
 * The Subsystem can accept requests either from the facade or client directly.
 * In any case, to the Subsystem, the Facade is yet another client, and it&#39;s not
 * a part of the Subsystem.
 */
class Subsystem1 {
    public operation1(): string {
        return &#39;Subsystem1: Ready!\n&#39;;
    }

    // ...

    public operationN(): string {
        return &#39;Subsystem1: Go!\n&#39;;
    }
}

/**
 * Some facades can work with multiple subsystems at the same time.
 */
class Subsystem2 {
    public operation1(): string {
        return &#39;Subsystem2: Get ready!\n&#39;;
    }

    // ...

    public operationZ(): string {
        return &#39;Subsystem2: Fire!&#39;;
    }
}

/**
 * The client code works with complex subsystems through a simple interface
 * provided by the Facade. When a facade manages the lifecycle of the subsystem,
 * the client might not even know about the existence of the subsystem. This
 * approach lets you keep the complexity under control.
 */
function clientCode(facade: Facade) {
    // ...

    console.log(facade.operation());

    // ...
}

/**
 * The client code may have some of the subsystem&#39;s objects already created. In
 * this case, it might be worthwhile to initialize the Facade with these objects
 * instead of letting the Facade create new instances.
 */
const subsystem1 = new Subsystem1();
const subsystem2 = new Subsystem2();
const facade = new Facade(subsystem1, subsystem2);
clientCode(facade);</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** 실행 결과

<figure class="code">
<pre class="code" lang="output"><code>Facade initializes subsystems:
Subsystem1: Ready!
Subsystem2: Get ready!
Facade orders subsystems to perform the action:
Subsystem1: Go!
Subsystem2: Fire!</code></pre>
</figure>

::: next
#### 다음을 보세요

[타입스크립트로 작성된 플라이웨이트 []{.fa
.fa-arrow-right}](../../flyweight/typescript/example.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### 돌아가기

[[]{.fa .fa-arrow-left} 타입스크립트로 작성된
데코레이터](../../decorator/typescript/example.html){.btn .btn-default
rel="prev"}
:::

## 다른 언어로 작성된 **퍼사드**

[![C#으로 작성된
퍼사드](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "C#으로 작성된 퍼사드"){.prog-lang-link}
[![C++로 작성된
퍼사드](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "C++로 작성된 퍼사드"){.prog-lang-link}
[![Go로 작성된
퍼사드](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Go로 작성된 퍼사드"){.prog-lang-link}
[![자바로 작성된
퍼사드](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "자바로 작성된 퍼사드"){.prog-lang-link}
[![PHP로 작성된
퍼사드](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "PHP로 작성된 퍼사드"){.prog-lang-link}
[![파이썬으로 작성된
퍼사드](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "파이썬으로 작성된 퍼사드"){.prog-lang-link}
[![루비로 작성된
퍼사드](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "루비로 작성된 퍼사드"){.prog-lang-link}
[![러스트로 작성된
퍼사드](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "러스트로 작성된 퍼사드"){.prog-lang-link}
[![스위프트로 작성된
퍼사드](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "스위프트로 작성된 퍼사드"){.prog-lang-link}
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
