::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [디자인
패턴들](../design-patterns.html){.type} / [구조
패턴](structural-patterns.html){.type}
:::

# 복합체 패턴 {#복합체-패턴 .title}

::: aka
다음 이름으로도 불립니다: [객체
트리, ]{style="display:inline-block"}[Composite]{style="display:inline-block"}
:::

::: {.section .intent}
##  의도 {#intent}

**복합체** 패턴은 객체들을 트리 구조들로 구성한 후, 이러한 구조들과 개별
객체들처럼 작업할 수 있도록 하는 구조 패턴입니다.

<figure class="image">
<img
src="../../images/patterns/content/composite/compositec299.png?id=73bcf0d94db360b636cd745f710d19db"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/composite/composite.png?id=73bcf0d94db360b636cd745f710d19db 640w,/images/patterns/content/composite/composite-1.5x.png?id=098acddafae70b1ec1cb1b05e833c3ae 960w,/images/patterns/content/composite/composite-2x.png?id=8847e6f8e2cb892ed2229faba83bd1b7 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="복합체 패턴" />
</figure>
:::

::: {.section .problem}
##  문제 {#problem}

복합체 패턴은 앱의 핵심 모델이 트리로 표현될 수 있을 때만 사용하세요.

예를 들어 `제품들`과 `상자들`이라는 두 가지 유형의 객체들이 있다고
가정해 봅시다. `상자`에는 여러 개의 `제품들`과 여러 개의 작은 `상자들`이
포함될 수 있습니다. 이 작은 `상자들`은 또한 일부 `제품들` 또는 더 작은
`상자들`등을 담을 수 있습니다.

이러한 클래스들을 사용하는 주문 시스템을 만들기로 했다고 가정해
보겠습니다. 주문들에는 포장이 없는 단순한 제품들과 제품들로 채워진
상자들 및 다른 상자들이 포함될 수 있습니다. 그러면 그러한 주문의
총가격을 어떻게 계산하시겠습니까?

<figure class="image">
<img
src="../../images/patterns/diagrams/composite/problem-kof5ce.png?id=771acc7452d2994c898d442de7519931"
style="aspect-ratio: 1;"
srcset="/images/patterns/diagrams/composite/problem-ko.png?id=771acc7452d2994c898d442de7519931 370w,/images/patterns/diagrams/composite/problem-ko-1.5x.png?id=4b36473f226fd9d9040decf6e718dcc1 555w,/images/patterns/diagrams/composite/problem-ko-2x.png?id=4e82b616eae4df38449d3cf3197e76fd 740w"
sizes="(max-width: 720px) 100vw, 370px" loading="lazy" width="370"
alt="복잡한 주문의 구조" />
<figcaption><p>하나의 주문은 여러 제품으로 구성될 수 있는데, 이 제품들은
상자에 포장될 수 있으며, 다시 그 상자들이 더 큰 상자에 포장되는 식으로
이어질 수 있습니다. 그러면 그 전체 구조는 거꾸로 된 나무와 같은 모양으로
형성될 것입니다.</p></figcaption>
</figure>

당신은 이 문제에 직접적으로 접근해 볼 수 있습니다: 모든 상자를 푼 후
내부의 모든 제품을 살펴본 다음 가격의 합계를 계산하는 것입니다. 현실
세계에서는 이러한 접근 방법은 가능하나, 프로그램에서 이 작업은 덧셈
루프를 실행하는 것만큼 간단하지 않은데, 그 이유는 덧셈 루프를 실행하기
위해 진행 중인 `제품들` 및 `상자들`의 클래스들, 상자의 중첩 수준 및 기타
복잡한 세부 사항들을 미리 알고 있어야 하기 때문입니다. 이 모든 것이 상자
및 내부 상자들을 열어 포함된 모든 제품 가격의 합계를 계산하는 직접적인
접근 방식을 어렵게 만듭니다.
:::

::: {.section .solution}
##  해결책 {#solution}

복합체 패턴은 총가격을 계산하는 메서드를 선언하는 공통 인터페이스를 통해
`제품들` 및 `상자들` 클래스들과 작업할 것을 제안합니다.

그러면 이 메서드는 어떻게 작동할까요? 제품의 경우 이 메서드는 단순히
제품 가격을 반환합니다. 상자의 경우, 이 메서드는 상자에 포함된 각 항목을
살펴보고 가격을 확인한 뒤 해당 상자의 총 가격을 반환합니다. 만약 이
항목들 중 하나가 더 작은 상자라면, 메서드는 해당 상자의 모든 내부 구성
요소의 가격이 계산될 때까지 내용물 등을 살펴봅니다. 메서드는 상자를 다룰
때 최종 가격에 포장 비용 같은 약간의 추가 비용도 추가할 수도 있습니다.

<figure class="image">
<img
src="../../images/patterns/content/composite/composite-comic-1-ko2180.png?id=f3206222fb615eec06a6606ac4c8d075"
style="aspect-ratio: 2;"
srcset="/images/patterns/content/composite/composite-comic-1-ko.png?id=f3206222fb615eec06a6606ac4c8d075 600w,/images/patterns/content/composite/composite-comic-1-ko-1.5x.png?id=3cb92e74bf9532de6adc876a92d6c153 900w,/images/patterns/content/composite/composite-comic-1-ko-2x.png?id=44a651179c61e068c05c0c42d31e2705 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="복합체 패턴이 제안하는 해결책" />
<figcaption><p>복합체 패턴은 객체 트리의 모든 컴포넌트들에 대해
재귀적으로 행동을 실행할 수 있도록 합니다.</p></figcaption>
</figure>

이 접근 방식의 가장 큰 이점은 더 이상 트리를 구성하는 객체들의 구상
클래스들에 대해 신경 쓸 필요도, 또 물건이 단순한 제품인지 내용물이 있는
상자인지 알 필요도 없다는 점입니다. 단순히 공통 인터페이스를 통해 모두
같은 방식으로 처리하시면 됩니다. 당신이 메서드를 호출하면 객체들 자체가
요청을 트리 아래로 전달합니다.
:::

::: {.section .analogy}
##  실제상황 적용 {#analogy}

<figure class="image">
<img
src="../../images/patterns/diagrams/composite/live-examplee724.png?id=548a7cec45b493af66e8bfe524a137d3"
style="aspect-ratio: 1.22;"
srcset="/images/patterns/diagrams/composite/live-example.png?id=548a7cec45b493af66e8bfe524a137d3 280w,/images/patterns/diagrams/composite/live-example-1.5x.png?id=2129195f4103a5a4d3440a086ce3dc87 420w,/images/patterns/diagrams/composite/live-example-2x.png?id=b555458f20fc30425ae6dada5da492af 560w"
sizes="(max-width: 720px) 100vw, 280px" loading="lazy" width="280"
alt="군사 구조의 예시" />
<figcaption><p>군대 구조의 예시.</p></figcaption>
</figure>

대부분의 국가에서 군대는 계층구조로 구성되어 있습니다. 군대는 여러
사단으로 구성되며, 사단은 여단의 집합이고, 여단은 소대의 집합이며,
소대는 또 분대로 나누어질 수 있습니다. 마지막으로 분대는 실제 군인들의
작은 집합입니다. 명령들은 계층구조의 최상위에서 내려와 모든 병사가
자신이 수행해야 할 작업을 알게 될 때까지 계층구조의 각 하위 계층으로
전달됩니다.
:::

::::: {.section .structure-container}
##  구조 {#structure}

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/composite/structure-ko1d89.png?id=6ae4e91052a44c5a66cb1c99a665dd51"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 0.82;"
srcset="/images/patterns/diagrams/composite/structure-ko.png?id=6ae4e91052a44c5a66cb1c99a665dd51 360w,/images/patterns/diagrams/composite/structure-ko-1.5x.png?id=85044cba2c6c5f5ea18a940dfd2bb51f 540w,/images/patterns/diagrams/composite/structure-ko-2x.png?id=c38ffaba6274f5fc258c266ef3a6f823 720w"
sizes="(max-width: 720px) 100vw, 360px" loading="lazy" width="360"
alt="복합 디자인 패턴의 구조" /><img
src="../../images/patterns/diagrams/composite/structure-ko-indexed14f8.png?id=f04bdbd0555320e245f5747a3236c1aa"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 0.85;"
srcset="/images/patterns/diagrams/composite/structure-ko-indexed.png?id=f04bdbd0555320e245f5747a3236c1aa 400w,/images/patterns/diagrams/composite/structure-ko-indexed-1.5x.png?id=e87f09de1aef56e437c7cf4a79a8bf54 600w,/images/patterns/diagrams/composite/structure-ko-indexed-2x.png?id=68d681f9c52884850a4da95e47c09807 800w"
sizes="(max-width: 720px) 100vw, 400px" loading="lazy" width="400"
alt="복합 디자인 패턴의 구조" />
</figure>
:::

1.  **컴포넌트** 인터페이스는 트리의 단순 요소들과 복잡한 요소들 모두에
    공통적인 작업을 설명합니다.

2.  **잎**은 트리의 기본 요소이며 하위요소가 없습니다.

    일반적으로 잎 컴포넌트들은 작업을 위임할 하위요소가 없어서 대부분의
    실제 작업들을 수행합니다.

3.  **컨테이너**(일명 *[복]{.cjk}[합]{.cjk}[체]{.cjk}*)​는 하위 요소들​(잎
    또는 기타 컨테이너)​이 있는 요소입니다. 컨테이너는 자녀들의 구상
    클래스들을 알지 못하며, 컴포넌트 인터페이스를 통해서만 모든 하위
    요소들과 함께 작동합니다.

    요청을 전달받으면 컨테이너는 작업을 하위 요소들에 위임하고 중간
    결과들을 처리한 다음 최종 결과들을 클라이언트에 반환합니다.

4.  **클라이언트**는 컴포넌트 인터페이스를 통해 모든 요소들과
    작동합니다. 그 결과 클라이언트는 트리의 단순 요소들 또는 복잡한
    요소들 모두에 대해 같은 방식으로 작업할 수 있습니다.
::::
:::::

::: {.section .pseudocode}
##  의사코드 {#pseudocode}

이 예시에서는 **복합체** 패턴이 어떻게 그래픽 편집기에서 기하학적 모양
쌓기를 구현할 수 있도록 하는지를 살펴보겠습니다.

<figure class="image">
<img
src="../../images/patterns/diagrams/composite/example7ea7.png?id=98ba81d07c979038dd2ebf3c83a2e19f"
style="aspect-ratio: 0.84;"
srcset="/images/patterns/diagrams/composite/example.png?id=98ba81d07c979038dd2ebf3c83a2e19f 370w,/images/patterns/diagrams/composite/example-1.5x.png?id=ae008efd4146628cd7303925d4ce90be 555w,/images/patterns/diagrams/composite/example-2x.png?id=d21edef39d3792e8a4c6736727ac7305 740w"
sizes="(max-width: 720px) 100vw, 370px" loading="lazy" width="370"
alt="복합체 패턴 구조 예시" />
<figcaption><p>기하학적 모양 편집기 예시.</p></figcaption>
</figure>

`Compound­Graphic` 클래스는 다른 복합 모양들을 포함한 모든 하위 모양으로
구성될 수 있는 컨테이너입니다. 복합 모양은 단순 모양과 같은 메서드들을
보유하나, 자체적으로 무언가를 수행하는 대신 복합 모양은 모든 자녀에게
재귀적으로 요청을 전달하고 이의 결과를 \'요약\'합니다.

클라이언트 코드는 모든 모양 클래스들에 공통인 단일 인터페이스를 통해
모든 모양과 함께 작동합니다. 따라서 클라이언트는 자신이 단순 모양과
작업하는지 복합 모양과 작업하는지를 알지 못합니다. 또 클라이언트는 매우
복잡한 객체 구조들을 형성하는 구상 클래스들에 결합하지 않고도 해당
구조들과 작업할 수 있습니다.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// 컴포넌트 인터페이스는 합성 관계의 단순 객체와 복잡한 객체 모두를 위한 공통
// 작업들을 선언합니다.
interface Graphic is
    method move(x, y)
    method draw()

// 잎 클래스는 합성 관계의 최종 객체들을 나타냅니다. 잎 객체는 하위 객체들을 가질
// 수 없습니다. 일반적으로 실제 작업을 수행하는 것은 잎 객체들이며, 복합체 객체들은
// 오로지 자신의 하위 컴포넌트에만 작업을 위임합니다.
class Dot implements Graphic is
    field x, y

    constructor Dot(x, y) { ... }

    method move(x, y) is
        this.x += x, this.y += y

    method draw() is
        // X와 Y에 점을 그립니다.

// 모든 컴포넌트 클래스들은 다른 컴포넌트들을 확장할 수 있습니다.
class Circle extends Dot is
    field radius

    constructor Circle(x, y, radius) { ... }

    method draw() is
        // X와 Y에 반지름이 R인 원을 그립니다.

// 복합체 클래스는 자식이 있을 수 있는 복잡한 컴포넌트들을 나타냅니다. 복합체
// 객체들은 일반적으로 실제 작업을 자식들에 위임한 다음 결과를 &#39;합산&#39;합니다.
class CompoundGraphic implements Graphic is
    field children: array of Graphic

    // 복합체 객체는 자식 리스트에 단순한 또는 복잡한 다른 컴포넌트들을 추가하거나
    // 제거할 수 있습니다.
    method add(child: Graphic) is
        // 하나의 자식을 자식들의 배열에 추가합니다.

    method remove(child: Graphic) is
        // 하나의 자식을 자식들의 배열에서 제거합니다.

    method move(x, y) is
        foreach (child in children) do
            child.move(x, y)

    // 복합체는 특정 방식으로 기본 논리를 실행합니다. 복합체는 모든 자식을
    // 재귀적으로 순회하여 결과들을 수집하고 요약합니다. 복합체의 자식들이 이러한
    // 호출들을 자신의 자식들 등으로 전달하기 때문에 결과적으로 전체 객체 트리를
    // 순회하게 됩니다.
    method draw() is
        // 1. 각 자식 컴포넌트에 대해:
        //     - 컴포넌트를 그리세요.
        //     - 경계 사각형을 업데이트하세요.
        // 2. 경계 좌표를 사용하여 점선 직사각형을 그리세요.


// 클라이언트 코드는 기초 인터페이스를 통해 모든 컴포넌트와 함께 작동합니다. 그래야
// 클라이언트 코드가 단순한 잎 컴포넌트들과 복잡한 복합체들을 지원할 수 있습니다.
class ImageEditor is
    field all: CompoundGraphic

    method load() is
        all = new CompoundGraphic()
        all.add(new Dot(1, 2))
        all.add(new Circle(5, 3, 10))
        // …

    // 선택한 컴포넌트들을 하나의 복잡한 복합체 컴포넌트로 합성합니다.
    method groupSelected(components: array of Graphic) is
        group = new CompoundGraphic()
        foreach (component in components) do
            group.add(component)
            all.remove(component)
        all.add(group)
        // 모든 컴포넌트가 그려질 것입니다.
        all.draw()</code></pre>
</figure>
:::

:::::::: {.section .applicability-container}
##  적용 {#applicability}

::::::: applicability
::: applicability-problem
복합체 패턴은 나무와 같은 객체 구조를 구현해야 할 때 사용하세요.
:::

::: applicability-solution
복합체 패턴은 공통 인터페이스를 공유하는 두 가지 기본 요소 유형들인 단순
잎들과 복합 컨테이너들을 제공합니다. 컨테이너는 잎들과 다른 컨테이너들로
구성될 수 있으며, 이를 통해 나무와 유사한 중첩된 재귀 객체 구조를 구성할
수 있습니다.
:::

::: applicability-problem
이 패턴은 클라이언트 코드가 단순 요소들과 복합 요소들을 모두 균일하게
처리하도록 하고 싶을 때 사용하세요.
:::

::: applicability-solution
복합체 패턴에 의해 정의된 모든 요소들은 공통 인터페이스를 공유하며, 이
인터페이스를 사용하면 클라이언트는 작업하는 객체들의 구상 클래스에 대해
걱정할 필요가 없습니다.
:::
:::::::
::::::::

::: {.section .checklist}
##  구현방법 {#checklist}

1.  앱의 핵심 모델이 트리 구조로 표현될 수 있는지 확인하세요. 그 후 앱을
    단순 요소와 컨테이너로 분해하세요. 컨테이너는 다른 컨테이너들과 단순
    요소들을 모두 포함할 수 있어야 합니다.

2.  컴포넌트 인터페이스를 선언하세요. 이 인터페이스 내에는 복잡하고
    단순한 컴포넌트 모두에 적합한 메서드들의 리스트를 포함하세요.

3.  단순 요소들을 나타내는 잎 클래스를 생성하세요. 하나의 프로그램에는
    여러 개의 서로 다른 잎 클래스들이 있을 수 있습니다.

4.  복잡한 요소들을 나타내는 컨테이너 클래스를 만든 후, 이 클래스에 하위
    요소들에 대한 참조를 저장하기 위한 배열 필드를 제공하세요. 배열은
    잎과 컨테이너를 모두 저장할 수 있어야 하므로 컴포넌트 인터페이스
    유형으로 선언하셔야 합니다.

    컴포넌트 인터페이스의 메서드들을 구현하는 동안 컨테이너는 대부분
    작업을 하위 요소들에 위임해야 한다는 점을 기억하세요.

5.  마지막으로 컨테이너에서 자식 요소들을 추가 및 제거하는 메서드들을
    정의하세요.

    이러한 작업은 컴포넌트 인터페이스에서 선언할 수 있으나, 그리하면 잎
    클래스의 메서드들이 비어 있을 것이므로
    *[인]{.cjk}[터]{.cjk}[페]{.cjk}[이]{.cjk}[스]{.cjk}
    [분]{.cjk}[리]{.cjk} [원]{.cjk}[칙]{.cjk}*을 위반할 것입니다. 그러나
    클라이언트는 트리를 구성할 때도 포함해서 모든 요소를 동등하게 처리할
    수 있을 것입니다.
:::

:::::: {.section .pros-cons}
##  장단점 {#pros-cons}

::::: row
::: col-sm-6
-    다형성과 재귀를 당신에 유리하게 사용해 복잡한 트리 구조들과 더
    편리하게 작업할 수 있습니다.
-    *[개]{.cjk}[방]{.cjk}/[폐]{.cjk}[쇄]{.cjk} [원]{.cjk}[칙]{.cjk}*.
    객체 트리와 작동하는 기존 코드를 훼손하지 않고 앱에 새로운 요소
    유형들을 도입할 수 있습니다.
:::

::: col-sm-6
-    기능이 너무 다른 클래스들에는 공통 인터페이스를 제공하기 어려울 수
    있으며, 어떤 경우에는 컴포넌트 인터페이스를 과도하게 일반화해야 하여
    이해하기 어렵게 만들 수 있습니다.
:::
:::::
::::::

::: {.section .relations}
##  다른 패턴과의 관계 {#relations}

-   당신은 복잡한 [복합체](composite.html) 패턴 트리를 생성할 때
    [빌더](builder.html)를 사용할 수 있습니다. 왜냐하면 빌더의 생성
    단계들을 재귀적으로 작동하도록 프로그래밍할 수 있기 때문입니다.

-   [책임 연쇄](chain-of-responsibility.html) 패턴은 종종
    [복합체](composite.html) 패턴과 함께 사용됩니다. 그러면 잎
    컴포넌트가 요청을 받으면 해당 요청을 모든 부모 컴포넌트들의 체인을
    통해 객체 트리의 뿌리​(root)​까지 전달할 수 있습니다.

-   당신은 [반복자들](iterator.html)을 사용하여 [복합체](composite.html)
    패턴 트리들을 순회할 수 있습니다.

-   당신은 [비지터](visitor.html) 패턴을 사용하여
    [복합체](composite.html) 패턴 트리 전체를 대상으로 작업을 수행할 수
    있습니다.

-   RAM을 절약하기 위하여 [복합체](composite.html) 패턴 트리의 공유된 잎
    노드들을 [플라이웨이트들](flyweight.html)로 구현할 수 있습니다.

-   [복합체](composite.html) 패턴 및 [데코레이터](decorator.html)는 둘
    다 구조 다이어그램이 유사합니다. 왜냐하면 둘 다 재귀적인 합성에
    의존하여 하나 또는 불특정 다수의 객체들을 정리하기 때문입니다.

    *[데]{.cjk}[코]{.cjk}[레]{.cjk}[이]{.cjk}[터]{.cjk}*는
    *[복]{.cjk}[합]{.cjk}[체]{.cjk} [패]{.cjk}[턴]{.cjk}*과 비슷하지만,
    자식 컴포넌트가 하나만 있습니다. 또 다른 중요한 차이점은
    *[데]{.cjk}[코]{.cjk}[레]{.cjk}[이]{.cjk}[터]{.cjk}*는 래핑된 객체에
    추가 책임들을 추가하는 반면 *[복]{.cjk}[합]{.cjk}[체]{.cjk}
    [패]{.cjk}[턴]{.cjk}*은 자신의 자식들의 결과를 \'요약\'하기만
    합니다.

    그러나 패턴들은 서로 협력할 수도 있습니다:
    *[데]{.cjk}[코]{.cjk}[레]{.cjk}[이]{.cjk}[터]{.cjk}*를 사용하여
    *[복]{.cjk}[합]{.cjk}[체]{.cjk} [패]{.cjk}[턴]{.cjk}* 트리의 특정
    객체의 행동을 확장할 수 있습니다.

-   [데코레이터](decorator.html) 및 [복합체](composite.html) 패턴을 많이
    사용하는 디자인들은 [프로토타입](prototype.html)을 사용하면 종종
    이득을 볼 수 있습니다. 프로토타입 패턴을 적용하면 복잡한 구조들을
    처음부터 다시 건축하는 대신 복제할 수 있기 때문입니다.
:::

::: {.section .implementations}
##  코드 예시 {#implementations}

[![C#으로 작성된
복합체](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](composite/csharp/example.html "C#으로 작성된 복합체"){.prog-lang-link}
[![C++로 작성된
복합체](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](composite/cpp/example.html "C++로 작성된 복합체"){.prog-lang-link}
[![Go로 작성된
복합체](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](composite/go/example.html "Go로 작성된 복합체"){.prog-lang-link}
[![자바로 작성된
복합체](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](composite/java/example.html "자바로 작성된 복합체"){.prog-lang-link}
[![PHP로 작성된
복합체](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](composite/php/example.html "PHP로 작성된 복합체"){.prog-lang-link}
[![파이썬으로 작성된
복합체](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](composite/python/example.html "파이썬으로 작성된 복합체"){.prog-lang-link}
[![루비로 작성된
복합체](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](composite/ruby/example.html "루비로 작성된 복합체"){.prog-lang-link}
[![러스트로 작성된
복합체](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](composite/rust/example.html "러스트로 작성된 복합체"){.prog-lang-link}
[![스위프트로 작성된
복합체](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](composite/swift/example.html "스위프트로 작성된 복합체"){.prog-lang-link}
[![타입스크립트로 작성된
복합체](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](composite/typescript/example.html "타입스크립트로 작성된 복합체"){.prog-lang-link}
:::

:::::: {#book-promo .banner-set2}
::::: {.prom .banner-content .banner-bg .banner-striped .banner-with-image data-id="DP: 1: Support our free website and own the eBook!" creative-id="standard-ko" position="content_bottom"}
::: banner-image
[![](../../images/patterns/banners/patterns-book-banner-3a29e.png?id=7d445df13c80287beaab234b4f3b698c){width="200"
height="200" loading="lazy"
srcset="/images/patterns/banners/patterns-book-banner-3-2x.png?id=0cc3f77ab421d1a5c02ee46488231c3a 2x"}](book.html)
:::

::: banner-text
### 전자책을 구매하여 당사의 무료 웹사이트를 지원하세요! {#전자책을-구매하여-당사의-무료-웹사이트를-지원하세요 .title}

-   22가지 디자인 패턴과 8가지 원칙들이 깊이 있게 설명됩니다.
-   잘 구성된 읽기 쉽고 전문 용어를 되도록 적게 사용한 페이지들이
    409개가 있습니다.
-   225개의 유용하고 명확한 삽화들과 다이어그램들이 있습니다.
-   11개의 언어로 된 코드 예시들이 있는 저장소.
-   모든 장치가 지원합니다: PDF/EPUB/MOBI/KFX 형식들.

[ 더 알아보기...](book.html){.btn .btn-secondary}
:::
:::::
::::::

::: next
#### 다음을 보세요

[데코레이터 []{.fa .fa-arrow-right}](decorator.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### 돌아가기

[[]{.fa .fa-arrow-left} 브리지](bridge.html){.btn .btn-default
rel="prev"}
:::
::::::::::::::::::::::::::::::::

::::::: {.prom .banner-sidebar .banner-removable .banner-removable-patterns data-id="DP: Sidebar" creative-id="standard-sidebar-ko" position="sidebar"}
:::::: banner-inner
:::: image3d-book-right
::: {.image3d-cover style="background: #0b3752;"}
[![](../../images/patterns/book/web-cover-ko50c5.png?id=e2403179cb38e78e35b925e860f6cda5){width="250"
height="375" loading="lazy"
srcset="/images/patterns/book/web-cover-ko-2x.png?id=fa8228428370e6499acb5727e591db59 2x"}](book.html)
:::
::::

::: {style="margin-top: 1rem"}
이 글은 전자책 **디자인 패턴에\
뛰어들기**의 일부입니다.

[ 더 알아보기...](book.html){.btn .btn-secondary .btn-block}
:::
::::::
:::::::
::::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::::
