::::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [디자인
패턴들](../design-patterns.html){.type} / [행동
패턴](behavioral-patterns.html){.type}
:::

# 전략 패턴 {#전략-패턴 .title}

::: aka
다음 이름으로도 불립니다: [Strategy]{style="display:inline-block"}
:::

::: {.section .intent}
##  의도 {#intent}

**전략** 패턴은 알고리즘들의 패밀리를 정의하고, 각 패밀리를 별도의
클래스에 넣은 후 그들의 객체들을 상호교환할 수 있도록 하는 행동
디자인 패턴입니다.

<figure class="image">
<img
src="../../images/patterns/content/strategy/strategy4ca6.png?id=379bfba335380500375881a3da6507e0"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/strategy/strategy.png?id=379bfba335380500375881a3da6507e0 640w,/images/patterns/content/strategy/strategy-1.5x.png?id=33ecebc66a9761454f2786a9b3e9bbe5 960w,/images/patterns/content/strategy/strategy-2x.png?id=1cee47d05a76fddf07dce9c67b700748 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="전략 디자인 패턴" />
</figure>
:::

::: {.section .problem}
##  문제 {#problem}

어느 날 당신은 여행자들을 위한 내비 앱을 만들기로 했습니다. 앱의 중심
기능은 사용자들이 어느 도시에서든 빠르게 방향을 잡을 수 있도록 도와주는
아름다운 지도였습니다.

앱에서 가장 많이 요청된 기능 중 하나는 자동 경로 계획 기능이었습니다.
사용자가 주소를 입력하면 지도에 표시된 해당 목적지로 가는 가장 빠른
경로를 볼 수 있는 기능이었죠.

앱의 첫 번째 버전에서는 도로로 된 경로만을 만들 수 있었습니다. 차를 타고
여행하는 사용자들은 만족했습니다. 하지만 모든 사용자가 여가 중에
운전하는 걸 좋아하진 않았습니다. 그래서 그다음 업데이트에서는 도보
경로를 만드는 옵션을 추가했습니다. 바로 그다음에는 사람들이 경로에서
대중교통의 사용을 계획할 수 있도록 옵션을 추가했습니다.

하지만 그것은 시작에 불과했습니다. 나중에는 자전거를 타는 사용자들을
위한 경로를 만들 계획을 세웠습니다. 심지어 그다음에는 도시의 모든 관광
명소들을 지나는 경로를 만들 수 있는 또 다른 옵션을 추가할 계획을
세웠습니다.

<figure class="image">
<img
src="../../images/patterns/diagrams/strategy/problem9b55.png?id=e5ca943e559a979bd31d20e232dc22b6"
style="aspect-ratio: 2.2;"
srcset="/images/patterns/diagrams/strategy/problem.png?id=e5ca943e559a979bd31d20e232dc22b6 330w,/images/patterns/diagrams/strategy/problem-1.5x.png?id=31d1042ffc28056845e45d5c616da2a9 495w,/images/patterns/diagrams/strategy/problem-2x.png?id=3974fb99c97aec525dd0ffcff2e48e78 660w"
sizes="(max-width: 720px) 100vw, 330px" loading="lazy" width="330"
alt="내비게이터의 코드가 매우 복잡해졌습니다." />
<figcaption><p>내비게이터의 코드가 복잡해졌습니다.</p></figcaption>
</figure>

사업적인 측면에서 앱은 성공했지만, 기술적인 문제들이 많은 골칫거리를
야기했습니다. 새 경로 구축 알고리즘을 추가할 때마다 내비게이터의 메인
클래스의 크기가 두 배로 늘어났으며, 어느 시점이 되자 내비 앱은
유지하기가 너무 어려워졌습니다.

간단한 버그를 수정하거나 주행거리 점수를 살짝 조정하기 위해 알고리즘 중
하나를 변경하면 전체 클래스에 영향이 미쳐 이미 작동하는 코드에서 오류가
발생할 가능성이 높아졌습니다.

또한 팀워크가 비효율적이 되었습니다. 앱 출시 직후 고용된 팀원들은 병합
충돌을 해결하는 데 너무 많은 시간을 할애해야 한다고 불평했습니다. 또
새로운 기능을 구현하려면 거대한 동일 클래스를 변경해야 했는데, 이렇게
바꾼 내용들이 다른 팀원들이 생성한 코드와 충돌하곤 했습니다.
:::

::: {.section .solution}
##  해결책 {#solution}

전략 패턴은 특정 작업을 다양한 방식으로 수행하는 클래스를 선택한 후 모든
알고리즘을 *[전]{.cjk}[략]{.cjk}[들]{.cjk}*(strategies)​이라는 별도의
클래스들로 추출할 것을 제안합니다.

*[콘]{.cjk}[텍]{.cjk}[스]{.cjk}[트]{.cjk}*(context)​라는 원래 클래스에는
전략 중 하나에 대한 참조를 저장하기 위한 필드가 있어야 합니다.
콘텍스트는 작업을 자체적으로 실행하는 대신 연결된 전략 객체에
위임합니다.

콘텍스트는 작업에 적합한 알고리즘을 선택할 책임이 없습니다. 대신
클라이언트가 원하는 전략을 콘텍스트에 전달합니다. 사실, 콘텍스트는
전략들에 대해 많이 알지 못합니다. 콘텍스트는 같은 일반 인터페이스를 통해
모든 전략과 함께 작동하며, 이 일반 인터페이스는 선택된 전략 내에
캡슐화된 알고리즘을 작동시킬 단일 메서드만 노출합니다.

이렇게 하면 콘텍스트가 구상 전략들에 의존하지 않게 되므로 콘텍스트 또는
다른 전략들의 코드를 변경하지 않고도 새 알고리즘들을 추가하거나 기존
알고리즘들을 수정할 수 있습니다.

<figure class="image">
<img
src="../../images/patterns/diagrams/strategy/solution1435.png?id=0813a174b29a2ed5902d321aba28e5fc"
style="aspect-ratio: 2.04;"
srcset="/images/patterns/diagrams/strategy/solution.png?id=0813a174b29a2ed5902d321aba28e5fc 570w,/images/patterns/diagrams/strategy/solution-1.5x.png?id=ce3d4e57f4a2a06ebc96f2607b3d6691 855w,/images/patterns/diagrams/strategy/solution-2x.png?id=66b5ee048ea2ad25c4b20f180ebf94d7 1140w"
sizes="(max-width: 720px) 100vw, 570px" loading="lazy" width="570"
alt="경로 계획 전략들." />
<figcaption><p>경로 계획 전략들.</p></figcaption>
</figure>

당신의 내비 앱에서 각 경로 구축 알고리즘을 단일 `build­Route` 메서드를
사용하여 자체 클래스로 추출할 수 있습니다. 이 메서드는 출발지와 목적지를
받은 후 경로의 체크포인트들의 컬렉션을 반환합니다.

같은 인수가 주어졌더라도 각 경로 구축 클래스는 다른 경로를 구축할 수
있지만 주 내비게이터 클래스는 어떤 알고리즘이 선택되었는지 별로 신경
쓰지 않습니다. 왜냐하면 주 내비게이터 클래스의 주요 작업은 지도에
체크포인트들의 집합을 렌더링하는 것이기 때문입니다. 이 클래스에는 활성
경로 구축 전략을 전환하는 메서드가 있어, 클래스의 클라이언트들이 (예:
사용자 인터페이스의 버튼들) 현재 선택된 경로 구축 행동들을 다른 행동으로
대체할 수 있습니다.
:::

::: {.section .analogy}
##  실제상황 적용 {#analogy}

<figure class="image">
<img
src="../../images/patterns/content/strategy/strategy-comic-1-ko6cd2.png?id=35b630ea7fe8d1d3d6c31a7e1821613d"
style="aspect-ratio: 2.13;"
srcset="/images/patterns/content/strategy/strategy-comic-1-ko.png?id=35b630ea7fe8d1d3d6c31a7e1821613d 640w,/images/patterns/content/strategy/strategy-comic-1-ko-1.5x.png?id=f6a8387a616f7ee722757328518d9d0d 960w,/images/patterns/content/strategy/strategy-comic-1-ko-2x.png?id=cdafffbd7a8b4eb9b6c9ef9002244117 1280w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="640"
alt="다양한 운송 전략들" />
<figcaption><p>공항에 도착하기 위한 다양한 전략들.</p></figcaption>
</figure>

공항에 가야 한다고 상상해 보세요. 당신은 버스를 탈 수도 있고, 택시나
자전거를 탈 수도 있습니다. 이것들이 바로 당신의 운송 전략들입니다.
예산이나 시간 제약 등을 고려하여 이러한 전략 중 하나를 선택할 수
있습니다.
:::

::::: {.section .structure-container}
##  구조 {#structure}

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/strategy/structuree491.png?id=c6aa910c94960f35d100bfca02810ea1"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.19;"
srcset="/images/patterns/diagrams/strategy/structure.png?id=c6aa910c94960f35d100bfca02810ea1 440w,/images/patterns/diagrams/strategy/structure-1.5x.png?id=5b1c6376b06374719dcae95455b068d8 660w,/images/patterns/diagrams/strategy/structure-2x.png?id=5bd791857c3bab419bcf4fa86877439d 880w"
sizes="(max-width: 720px) 100vw, 440px" loading="lazy" width="440"
alt="전략 디자인 패턴의 구조" /><img
src="../../images/patterns/diagrams/strategy/structure-indexed64c7.png?id=ff55c5a6273cf82a5667f3cab5b605c7"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.15;"
srcset="/images/patterns/diagrams/strategy/structure-indexed.png?id=ff55c5a6273cf82a5667f3cab5b605c7 450w,/images/patterns/diagrams/strategy/structure-indexed-1.5x.png?id=3d6ba05b8a74a9fb8e3f88eb9ca1e30f 675w,/images/patterns/diagrams/strategy/structure-indexed-2x.png?id=9f8e2f838f16705775411e2b4616820e 900w"
sizes="(max-width: 720px) 100vw, 450px" loading="lazy" width="450"
alt="전략 디자인 패턴의 구조" />
</figure>
:::

1.  **콘텍스트**는 구상 전략 중 하나에 대한 참조를 유지하고 전략
    인터페이스를 통해서만 이 객체와 통신합니다.

2.  **전략** 인터페이스는 모든 구상 전략에 공통이며, 콘텍스트가 전략을
    실행하는 데 사용하는 메서드를 선언합니다.

3.  **구상 전략들**은 콘텍스트가 사용하는 알고리즘의 다양한 변형들을
    구현합니다.

4.  콘텍스트는 알고리즘을 실행해야 할 때마다 연결된 전략 객체의 실행
    메서드를 호출합니다. 콘텍스트는 알고리즘이 어떻게 실행되는지와
    자신이 어떤 유형의 전략과 함께 작동하는지를 모릅니다.

5.  **클라이언트**는 특정 전략 객체를 만들어 콘텍스트에 전달합니다.
    콘텍스트는 클라이언트들이 런타임에 콘텍스트와 관련된 전략을 대체할
    수 있도록 하는 세터​(setter)​를 노출합니다.
::::
:::::

::: {.section .pseudocode}
##  의사코드 {#pseudocode}

이 예시에서의 콘텍스트는 여러 **전략들**을 사용하여 다양한 산술 연산들을
실행합니다.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// 전략 인터페이스는 어떤 알고리즘의 모든 지원 버전에 공통적인 작업을 선언합니다.
// 콘텍스트는 이 인터페이스를 사용하여 구상 전략들에 의해 정의된 알고리즘을
// 호출합니다.
interface Strategy is
    method execute(a, b)

// 구상 전략들은 기초 전략 인터페이스를 따르면서 알고리즘을 구현합니다. 인터페이스는
// 그들이 콘텍스트에서 상호교환할 수 있게 만듭니다.
class ConcreteStrategyAdd implements Strategy is
    method execute(a, b) is
        return a + b

class ConcreteStrategySubtract implements Strategy is
    method execute(a, b) is
        return a - b

class ConcreteStrategyMultiply implements Strategy is
    method execute(a, b) is
        return a * b

// 콘텍스트는 클라이언트들이 관심을 갖는 인터페이스를 정의합니다.
class Context is
    // 콘텍스트는 전략 객체 중 하나에 대한 참조를 유지합니다. 콘텍스트는 전략의
    // 구상 클래스를 알지 못하며, 전략 인터페이스를 통해 모든 전략과 함께
    // 작동해야 합니다.
    private strategy: Strategy

    // 일반적으로 콘텍스트는 생성자를 통해 전략을 수락하고 런타임에 전략이 전환될
    // 수 있도록 세터도 제공합니다.
    method setStrategy(Strategy strategy) is
        this.strategy = strategy

    // 콘텍스트는 자체적으로 여러 버전의 알고리즘을 구현하는 대신 일부 작업을 전략
    // 객체에 위임합니다.
    method executeStrategy(int a, int b) is
        return strategy.execute(a, b)


// 클라이언트 코드는 구상 전략을 선택하고 콘텍스트에 전달합니다. 클라이언트는 올바른
// 선택을 하기 위해 전략 간의 차이점을 알고 있어야 합니다.
class ExampleApplication is
    method main() is
        Create context object.

        Read first number.
        Read last number.
        Read the desired action from user input.

        if (action == addition) then
            context.setStrategy(new ConcreteStrategyAdd())

        if (action == subtraction) then
            context.setStrategy(new ConcreteStrategySubtract())

        if (action == multiplication) then
            context.setStrategy(new ConcreteStrategyMultiply())

        result = context.executeStrategy(First number, Second number)

        Print result.</code></pre>
</figure>
:::

:::::::::::: {.section .applicability-container}
##  적용 {#applicability}

::::::::::: applicability
::: applicability-problem
전략 패턴은 객체 내에서 한 알고리즘의 다양한 변형들을 사용하고 싶을 때,
그리고 런타임 중에 한 알고리즘에서 다른 알고리즘으로 전환하고 싶을 때
사용하세요.
:::

::: applicability-solution
또 전략 패턴은 객체의 행동들을 특정 하위 행동들을 다양한 방식으로 수행할
수 있는 다른 하위 객체들과 연관시켜 객체의 행동들을 런타임에 간접적으로
변경할 수 있게 해줍니다.
:::

::: applicability-problem
전략 패턴은 일부 행동을 실행하는 방식에서만 차이가 있는 유사한
클래스들이 많은 경우에 사용하세요.
:::

::: applicability-solution
전략 패턴은 다양한 행동들을 별도의 클래스 계층구조로 추출하고 원래
클래스들을 하나로 결합하여 중복 코드를 줄일 수 있게 해줍니다.
:::

::: applicability-problem
전략 패턴을 사용하여 클래스의 비즈니스 로직을 해당 로직의 콘텍스트에서
그리 중요하지 않을지도 모르는 알고리즘들의 구현 세부 사항들로부터
고립하세요.
:::

::: applicability-solution
전략 패턴은 코드의 나머지 부분에서 해당 코드, 내부 데이터, 그리고 다양한
알고리즘들의 의존 관계들을 고립시킬 수 있습니다. 다양한 클라이언트들이
알고리즘들을 실행하고 런타임에 전환하기 위한 간단한 인터페이스를
얻습니다.
:::

::: applicability-problem
이 패턴은 같은 알고리즘의 다른 변형들 사이를 전환하는 거대한 조건문이
당신의 클래스에 있는 경우에 사용하세요.
:::

::: applicability-solution
전략 패턴을 사용하면 모든 알고리즘을 같은 인터페이스를 구현하는 별도의
클래스들로 추출하여 이러한 조건문을 제거할 수 있습니다. 원래 객체는
알고리즘의 모든 변형들을 구현하는 대신 이러한 객체들 중 하나에 실행을
위임합니다.
:::
:::::::::::
::::::::::::

::: {.section .checklist}
##  구현방법 {#checklist}

1.  콘텍스트 클래스에서 자주 변경되는 알고리즘을 식별하세요. 런타임에
    같은 알고리즘의 변형을 선택한 후 실행하는 거대한 조건문일 수도
    있습니다.

2.  알고리즘의 모든 변형에 공통인 전략 인터페이스를 선언하세요.

3.  하나씩 모든 알고리즘을 자체 클래스들로 추출하세요. 그들은 모두 전략
    인터페이스를 구현해야 합니다.

4.  콘텍스트 클래스에서 전략 객체에 대한 참조를 저장하기 위한 필드를
    추가한 후, 해당 필드의 값을 대체하기 위한 세터를 제공하세요.
    콘텍스트는 전략 인터페이스를 통해서만 전략 객체와 작동해야 합니다.
    콘텍스트는 인터페이스를 정의할 수 있으며, 이 인터페이스는 전략이
    콘텍스트의 데이터에 접근할 수 있도록 합니다.

5.  콘텍스트의 클라이언트들은 콘텍스트를 적절한 전략과 연관시켜야
    합니다. 이러한 전략은 클라이언트들이 기대하는 콘텍스트가 주 작업을
    수행하는 방식과 일치해야 합니다.
:::

:::::: {.section .pros-cons}
##  장단점 {#pros-cons}

::::: row
::: col-sm-6
-    런타임에 한 객체 내부에서 사용되는 알고리즘들을 교환할 수 있습니다.
-    알고리즘을 사용하는 코드에서 알고리즘의 구현 세부 정보들을 고립할
    수 있습니다.
-    상속을 합성으로 대체할 수 있습니다.
-    *[개]{.cjk}[방]{.cjk}/[폐]{.cjk}[쇄]{.cjk} [원]{.cjk}[칙]{.cjk}*.
    콘텍스트를 변경하지 않고도 새로운 전략들을 도입할 수 있습니다.
:::

::: col-sm-6
-    알고리즘이 몇 개밖에 되지 않고 거의 변하지 않는다면, 패턴과 함께
    사용되는 새로운 클래스들과 인터페이스들로 프로그램을 지나치게
    복잡하게 만들 이유가 없습니다.
-    클라이언트들은 적절한 전략을 선택할 수 있도록 전략 간의 차이점들을
    알고 있어야 합니다.
-    현대의 많은 프로그래밍 언어에는 익명 함수들의 집합 내에서
    알고리즘의 다양한 버전들을 구현할 수 있는 함수형 지원이 있으며,
    클래스들과 인터페이스들을 추가하여 코드의 부피를 늘리지 않으면서도
    전략 객체를 사용했을 때와 똑같이 이러한 함수들을 사용할 수 있습니다.
:::
:::::
::::::

::: {.section .relations}
##  다른 패턴과의 관계 {#relations}

-   [브리지](bridge.html), [상태](state.html), [전략](strategy.html)
    패턴은 매우 유사한 구조로 되어 있으며, [어댑터](adapter.html) 패턴도
    이들과 어느 정도 유사한 구조로 되어 있습니다. 위 모든 패턴은 다른
    객체에 작업을 위임하는 합성을 기반으로 합니다. 하지만 이 패턴들은
    모두 다른 문제들을 해결합니다. 패턴은 특정 방식으로 코드의 구조를
    짜는 레시피에 불과하지 않습니다. 왜냐하면 패턴은 해결하는 문제를
    다른 개발자들에게 전달할 수도 있기 때문입니다.

-   [커맨드](command.html)와 [전략](strategy.html) 패턴은 비슷해 보일 수
    있습니다. 왜냐하면 둘 다 어떤 작업으로 객체를 매개변수화하는 데
    사용할 수 있기 때문입니다. 그러나 이 둘의 의도는 매우 다릅니다.

    -   당신은 *[커]{.cjk}[맨]{.cjk}[드]{.cjk}*를 사용하여 모든 작업을
        객체로 변환할 수 있습니다. 작업의 매개변수들은 해당 객체의
        필드들이 됩니다. 이 변환은 작업의 실행을 연기하고, 해당 작업을
        대기열에 넣고, 커맨드들의 기록을 저장한 후 해당 커맨드들을 원격
        서비스에 보내는 등의 작업을 가능하게 합니다.

    -   반면에 *[전]{.cjk}[략]{.cjk} [패]{.cjk}[턴]{.cjk}*은 일반적으로
        같은 작업을 수행하는 다양한 방법을 설명하므로 단일 콘텍스트
        클래스 내에서 이러한 알고리즘들을 교환할 수 있도록 합니다.

-   [데코레이터](decorator.html)는 객체의 피부를 변경할 수 있고
    [전략](strategy.html) 패턴은 객체의 내장을 변경할 수 있다고 비유할
    수 있습니다.

-   [템플릿 메서드](template-method.html)는 상속을 기반으로 합니다. 이
    메서드는 자식 클래스들에서 알고리즘의 부분들을 확장하여 변경할 수
    있도록 합니다. [전략](strategy.html) 패턴은 합성을 기반으로 합니다:
    당신은 객체 행동의 일부분들을 이러한 행동에 해당하는 다양한 전략들을
    제공하여 변경할 수 있습니다. *[템]{.cjk}[플]{.cjk}[릿]{.cjk}
    [메]{.cjk}[서]{.cjk}[드]{.cjk}*는 클래스 수준에서 작동하므로
    정적입니다. *[전]{.cjk}[략]{.cjk} [패]{.cjk}[턴]{.cjk}[은]{.cjk}*은
    객체 수준에서 작동하므로 런타임에 행동들을 전환할 수 있도록 합니다.

-   [상태](state.html)는 [전략](strategy.html)의 확장으로 간주할 수
    있습니다. 두 패턴 모두 합성을 기반으로 합니다. 그들은 어떤 작업을
    도우미 객체들에 전달하여 콘텍스트의 행동을 바꿉니다.
    *[전]{.cjk}[략]{.cjk} [패]{.cjk}[턴]{.cjk}*은 이러한 객체들을 완전히
    독립적으로 만들어 서로를 인식하지 못하도록 만듭니다. 그러나
    *[상]{.cjk}[태]{.cjk}*는 구상 상태들 사이의 의존 관계들을 제한하지
    않으므로 그들이 콘텍스트의 상태를 마음대로 변경할 수 있도록 합니다.
:::

::: {.section .implementations}
##  코드 예시 {#implementations}

[![C#으로 작성된
전략](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](strategy/csharp/example.html "C#으로 작성된 전략"){.prog-lang-link}
[![C++로 작성된
전략](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](strategy/cpp/example.html "C++로 작성된 전략"){.prog-lang-link}
[![Go로 작성된
전략](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](strategy/go/example.html "Go로 작성된 전략"){.prog-lang-link}
[![자바로 작성된
전략](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](strategy/java/example.html "자바로 작성된 전략"){.prog-lang-link}
[![PHP로 작성된
전략](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](strategy/php/example.html "PHP로 작성된 전략"){.prog-lang-link}
[![파이썬으로 작성된
전략](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](strategy/python/example.html "파이썬으로 작성된 전략"){.prog-lang-link}
[![루비로 작성된
전략](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](strategy/ruby/example.html "루비로 작성된 전략"){.prog-lang-link}
[![러스트로 작성된
전략](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](strategy/rust/example.html "러스트로 작성된 전략"){.prog-lang-link}
[![스위프트로 작성된
전략](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](strategy/swift/example.html "스위프트로 작성된 전략"){.prog-lang-link}
[![타입스크립트로 작성된
전략](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](strategy/typescript/example.html "타입스크립트로 작성된 전략"){.prog-lang-link}
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

[템플릿 메서드 []{.fa .fa-arrow-right}](template-method.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### 돌아가기

[[]{.fa .fa-arrow-left} 상태](state.html){.btn .btn-default rel="prev"}
:::
::::::::::::::::::::::::::::::::::::

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
::::::::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::::::::
