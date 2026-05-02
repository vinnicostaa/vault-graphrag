::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [디자인
패턴들](../design-patterns.html){.type} / [생성
패턴](creational-patterns.html){.type}
:::

# 추상 팩토리 패턴 {#추상-팩토리-패턴 .title}

::: aka
다음 이름으로도 불립니다: [Abstract
Factory]{style="display:inline-block"}
:::

::: {.section .intent}
##  의도 {#intent}

**추상 팩토리**는 관련 객체들의 구상 클래스들을 지정하지 않고도 관련
객체들의 모음을 생성할 수 있도록 하는 생성패턴입니다.

<figure class="image">
<img
src="../../images/patterns/content/abstract-factory/abstract-factory-ko8b10.png?id=f806a14f54670f8a500597fec7d85a16"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/abstract-factory/abstract-factory-ko.png?id=f806a14f54670f8a500597fec7d85a16 640w,/images/patterns/content/abstract-factory/abstract-factory-ko-1.5x.png?id=fb53fc467bf1e1cb96f627e57de44518 960w,/images/patterns/content/abstract-factory/abstract-factory-ko-2x.png?id=9ea4ad69ee1473d3c395450b9819be89 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="추상 팩토리 패턴" />
</figure>
:::

::: {.section .problem}
##  문제 {#problem}

예를 들어 당신이 가구 판매장을 위한 프로그램을 만들고 있다고 가정합시다.
당신의 코드는 다음을 나타내는 클래스들로 구성됩니다:

1.  관련 제품들로 형성된 패밀리​(제품군), 예: `Chair`​(의자) +
    `Sofa`​(소파) + `Coffee­Table`​(커피 테이블).

2.  해당 제품군의 여러 가지 변형. 예를 들어 `Chair`​(의자) +
    `Sofa`​(소파) + `Coffee­Table`​(커피 테이블) 같은 제품들은
    `Modern`​(현대식), `Victorian`​(빅토리안), `Art­Deco`​(아르데코 양식)​와
    같은 변형으로 제공됩니다.

<figure class="image">
<img
src="../../images/patterns/diagrams/abstract-factory/problem-koba4d.png?id=75caa0103b449313ce1b9eb894aaef4a"
style="aspect-ratio: 1.4;"
srcset="/images/patterns/diagrams/abstract-factory/problem-ko.png?id=75caa0103b449313ce1b9eb894aaef4a 420w,/images/patterns/diagrams/abstract-factory/problem-ko-1.5x.png?id=afdf6f7da9f1bea567e4999f8b87ae4b 630w,/images/patterns/diagrams/abstract-factory/problem-ko-2x.png?id=a8fbbc596f14e791c3698e8ee96a1ea3 840w"
sizes="(max-width: 720px) 100vw, 420px" loading="lazy" width="420"
alt="제품 패밀리들과 그들의 변형들" />
<figcaption><p>제품 패밀리들과 그들의 변형들</p></figcaption>
</figure>

이제 당신이 새로운 개별 가구 객체를 생성했을 때, 이 객체들이 기존의 같은
패밀리 내에 있는 다른 가구 객체들과 일치하는 변형​(스타일)​을 가지도록 할
방법이 필요합니다. 그 이유는 당신의 고객이 스타일이 일치하지 않는 가구
세트를 받으면 크게 실망할 수 있기 때문입니다.

<figure class="image">
<img
src="../../images/patterns/content/abstract-factory/abstract-factory-comic-1-koae52.png?id=012be436b59d34b9a2b9725a85fd6d5c"
style="aspect-ratio: 2;"
srcset="/images/patterns/content/abstract-factory/abstract-factory-comic-1-ko.png?id=012be436b59d34b9a2b9725a85fd6d5c 600w,/images/patterns/content/abstract-factory/abstract-factory-comic-1-ko-1.5x.png?id=7945135a7767dd4832ed64664a657460 900w,/images/patterns/content/abstract-factory/abstract-factory-comic-1-ko-2x.png?id=2a1a18fd7185e58d7591ba6412ba7ed2 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600" />
<figcaption><p>현대식 스타일의 소파는 빅토리안 스타일의 의자들과 잘
어울리지 않습니다.</p></figcaption>
</figure>

또, 가구 공급업체들은 카탈로그를 매우 자주 변경하기 때문에, 그들은
새로운 제품 또는 제품군​(패밀리)​을 추가할 때마다 기존 코드를 변경해야
하는 번거로움을 피하고 싶을 것입니다.
:::

::: {.section .solution}
##  해결책 {#solution}

추상 공장 패턴의 첫 번째 방안은 각 제품 패밀리​(제품군)​에 해당하는
개별적인 인터페이스를 명시적으로 선언하는 것입니다. (예: 의자, 소파 또는
커피 테이블). 그다음, 제품의 모든 변형이 위 인터페이스를 따르도록
합니다. 예를 들어, 모든 의자의 변형들은 `Chair`​(의자) 인터페이스를
구현한다; 모든 커피 테이블 변형들은 `Coffee­Table`​(커피 테이블)
인터페이스를 구현한다, 등의 규칙을 명시합니다.

<figure class="image">
<img
src="../../images/patterns/diagrams/abstract-factory/solution1a1ac.png?id=71f2018d8bb443b9cce90d57110675e3"
style="aspect-ratio: 1.5;"
srcset="/images/patterns/diagrams/abstract-factory/solution1.png?id=71f2018d8bb443b9cce90d57110675e3 420w,/images/patterns/diagrams/abstract-factory/solution1-1.5x.png?id=4366e971c850514cde5d33cb7956de8b 630w,/images/patterns/diagrams/abstract-factory/solution1-2x.png?id=eacec286153e058db9255d26541c0529 840w"
sizes="(max-width: 720px) 100vw, 420px" loading="lazy" width="420"
alt="Chairs 클래스 계층" />
<figcaption><p>같은 객체의 모든 변형은 단일 클래스 계층구조로
옮겨져야 합니다.</p></figcaption>
</figure>

추상 공장 패턴의 다음 단계는 *[추]{.cjk}[상]{.cjk}
[팩]{.cjk}[토]{.cjk}[리]{.cjk} [패]{.cjk}[턴]{.cjk}*을 선언하는
것입니다. 추상 공장 패턴은 제품 패밀리 내의 모든 개별 제품들의 생성
메서드들이 목록화되어있는 인터페이스입니다. (예: `create­Chair`​(의자
생성), `create­Sofa`​(소파 생성), `create­Coffee­Table`​(커피 테이블 생성)
등).

<figure class="image">
<img
src="../../images/patterns/diagrams/abstract-factory/solution24d8e.png?id=53975d6e4714c6f942633a879f7ac571"
style="aspect-ratio: 2;"
srcset="/images/patterns/diagrams/abstract-factory/solution2.png?id=53975d6e4714c6f942633a879f7ac571 640w,/images/patterns/diagrams/abstract-factory/solution2-1.5x.png?id=581a6cdc0dcd5511f1c30e509b1d4a7f 960w,/images/patterns/diagrams/abstract-factory/solution2-2x.png?id=b21557081f75ac7b4110427e89a10378 1280w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="640"
alt="_Factories_ 클래스 계층" />
<figcaption><p>각 구상 팩토리는 특정 제품의
변형에 해당합니다.</p></figcaption>
</figure>

다음은 제품 변형을 다룰 차례입니다. 제품 패밀리의 각 변형에 대해
`Abstract­Factory` 추상 팩토리 인터페이스를 기반으로 별도의 팩토리
클래스를 생성합니다. 팩토리는 특정 종류의 제품을 반환하는 클래스입니다.
예를 들어 `Modern­Furniture­Factory`​(현대식 가구 팩토리)​에서는 다음
객체들만 생성할 수 있습니다: `Modern­Chair`​(현대식 의자),
`Modern­Sofa`​(현대식 소파) 및 `Modern­Coffee­Table`​(현대식 커피 테이블).

클라이언트 코드는 자신에 해당하는 추상 인터페이스를 통해 팩토리들과
제품들 모두와 함께 작동해야 합니다. 그래야 클라이언트 코드에 넘기는
팩토리의 종류와 제품 변형들을 클라이언트 코드를 손상하지 않으며
자유자재로 변경할 수 있습니다.

<figure class="image">
<img
src="../../images/patterns/content/abstract-factory/abstract-factory-comic-2-ko45bd.png?id=eea89e0057cf51dd787f76492a7abbec"
style="aspect-ratio: 2;"
srcset="/images/patterns/content/abstract-factory/abstract-factory-comic-2-ko.png?id=eea89e0057cf51dd787f76492a7abbec 600w,/images/patterns/content/abstract-factory/abstract-factory-comic-2-ko-1.5x.png?id=8639c693b6458507bd07e53707e7faf7 900w,/images/patterns/content/abstract-factory/abstract-factory-comic-2-ko-2x.png?id=c5629d16fa64377bc55773b5bdd2e75f 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600" />
<figcaption><p>클라이언트들은 함께 작업하는 팩토리의 구상 클래스에 대해
신경을 쓰지 않아도 돼야 합니다.</p></figcaption>
</figure>

클라이언트가 팩토리에 의자를 주문했다고 가정해 봅시다. 클라이언트는
팩토리의 클래스들을 알 필요가 없으며, 팩토리가 어떤 변형의 의자를
생성할지 전혀 신경을 쓰지 않습니다. 클라이언트는 추상 `Chair`​(의자)
인터페이스를 사용하여, 현대식 의자이든 빅토리아식 의자이든 종류에
상관없이 모든 의자를 항상 동일한 방식으로 주문하며, 그가 의자에 대해
아는 유일한 사실은 제품이 `sit­On`​(앉을 수 있다) 메서드를 구현한다는
것뿐입니다. 그러나, 생성된 의자의 변형은 항상 같은 팩토리 객체에서
생성된 소파 또는 커피 테이블의 변형과 같을 것입니다.

여기에서 명확히 짚고 넘어가야 할 점이 있습니다. 클라이언트가 추상
인터페이스에만 노출된다면 실제 팩토리 객체를 생성하는 것은 무엇일까요?
일반적으로 프로그램은 초기화 단계에서 구상 팩토리 객체를 생성합니다. 그
직전에 프로그램은 환경 또는 구성 설정에 따라 팩토리 유형을 선택해야
합니다.
:::

::::: {.section .structure-container}
##  구조 {#structure}

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/abstract-factory/structureb809.png?id=a3112cdd98765406af94595a3c5e7762"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/diagrams/abstract-factory/structure.png?id=a3112cdd98765406af94595a3c5e7762 720w,/images/patterns/diagrams/abstract-factory/structure-1.5x.png?id=d2964e541620d9c087e693bd0e0a0836 1080w,/images/patterns/diagrams/abstract-factory/structure-2x.png?id=c4d3634ec2e74e02a0fe1a83ce9b50f6 1440w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="720"
alt="추상 팩토리 디자인 패턴" /><img
src="../../images/patterns/diagrams/abstract-factory/structure-indexed028d.png?id=6ae1c99cbd90cf58753c633624fb1a04"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.56;"
srcset="/images/patterns/diagrams/abstract-factory/structure-indexed.png?id=6ae1c99cbd90cf58753c633624fb1a04 700w,/images/patterns/diagrams/abstract-factory/structure-indexed-1.5x.png?id=473be2975c5716162e6969e6532210ac 1050w,/images/patterns/diagrams/abstract-factory/structure-indexed-2x.png?id=cb6d4e1e89826c42966dc7097374f889 1400w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="700"
alt="추상 팩토리 디자인 패턴" />
</figure>
:::

1.  **추상 제품들**은 제품 패밀리를 구성하는 개별 연관 제품들의 집합에
    대한 인터페이스들을 선언합니다.

2.  **구상 제품들**은 변형들로 그룹화된 추상 제품들의 다양한
    구현들입니다. 각 추상 제품​(의자/소파)​은 주어진 모든
    변형​(빅토리안/현대식)​에 구현되어야 합니다.

3.  **추상 팩토리** 인터페이스는 각각의 추상 제품들을 생성하기 위한 여러
    메서드들의 집합을 선언합니다.

4.  **구상 팩토리들**은 추상 팩토리의 생성 메서드들을 구현합니다. 각
    구상 팩토리는 제품들의 특정 변형들에 해당하며 해당 특정 변형들만
    생성합니다.

5.  구상 팩토리들은 구상 제품들을 인스턴스화하나, 그 제품들의 생성
    메서드들의 시그니처들은 그에 해당하는 *[추]{.cjk}[상]{.cjk}*
    제품들을 반환해야 합니다. 그래야 팩토리를 사용하는 클라이언트 코드가
    팩토리에서 받은 제품의 특정 변형과 결합되지 않습니다.
    **클라이언트**는 추상 인터페이스를 통해 팩토리/제품 변형의 객체들과
    소통하는 한 그 어떤 구상 팩토리/제품 변형과 작업할 수 있습니다.
::::
:::::

::: {.section .pseudocode}
##  의사코드 {#pseudocode}

이 예시는 추상 팩토리 패턴이 크로스 플랫폼 사용자 인터페이스​(UI)
요소들을 생성하는 방법을 보여줍니다. 이 방법으로 요소들을 생성하면
클라이언트 코드는 구상 UI 클래스들과 결합하지 않으며, 모든 생성된
요소들은 선택된 운영 체제에 맞게 생성됩니다.

<figure class="image">
<img
src="../../images/patterns/diagrams/abstract-factory/exampled62f.png?id=5928a61d18bf00b047463471c599100a"
style="aspect-ratio: 1.42;"
srcset="/images/patterns/diagrams/abstract-factory/example.png?id=5928a61d18bf00b047463471c599100a 640w,/images/patterns/diagrams/abstract-factory/example-1.5x.png?id=baee3bac793ec97e0ec91c49d9382e7d 960w,/images/patterns/diagrams/abstract-factory/example-2x.png?id=eb5127b1d6486f6fad73be2d5e444449 1280w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="640"
alt="추상 팩토리 패턴 예시의 클래스 다이어그램" />
<figcaption><p>크로스 플랫폼 사용자 인터페이스
클래스 예시.</p></figcaption>
</figure>

UI 요소들은 크로스 플랫폼 애플리케이션 내에서는 유사하게 동작할 것으로
예상되나, 다른 운영 체제 내에서는 약간씩 다르게 보일 것으로 예상됩니다.
또한 UI 요소들이 해당하는 운영 체제의 스타일과 일치하는지 확인하는 것도
당신의 책임입니다. 왜냐하면 당신은 당신의 프로그램이 윈도우에서 실행될
때 매킨토시 컨트롤을 렌더링하는 것을 원하지 않을 것이기 때문입니다.

추상 팩토리 인터페이스는 일련의 생성 메서드들을 선언하며, 이 메서드들은
클라이언트 코드가 다양한 유형들의 UI 요소들을 생성하는 데 사용될 수
있습니다. 구상 팩토리는 특정 운영 체제에 해당하고 해당 특정 운영체제에
일치하는 UI 요소들을 생성합니다.

작동 방식은 다음과 같습니다. 앱이 시작될 때 현재 소속된 운영 체제의
유형을 확인합니다. 그 후 앱은 이 정보를 사용하여 운영 체제와 일치하는
클래스에서 팩토리 객체를 생성합니다. 나머지 코드는 이 팩토리 객체를
사용하여 UI 요소들을 만듭니다. 이렇게 하면 잘못된 요소들이 생성되는 것을
방지할 수 있습니다.

위 방법을 사용하면 클라이언트 코드가 객체의 추상 인터페이스를 통해
작업하는 한 팩토리들과 UI 요소들의 구상 클래스들에 의존하지 않게 됩니다.
또 위 방법은 클라이언트 코드가 당신이 나중에 추가할 수 있는 다른
팩토리들과 UI 요소들을 지원할 수 있도록 합니다.

이것의 결과로 새로운 변형의 UI 요소를 앱에 추가할 때마다 클라이언트
코드를 수정할 필요가 없어집니다. 이 요소들을 생성하는 새로운 팩토리
클래스를 만든 후 앱의 초기화 코드를 그 팩토리 클래스를 선택하도록 약간
수정하기만 하면 됩니다.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// 추상 팩토리 인터페이스는 다른 추상 제품들을 반환하는 메서드들의 집합을
// 선언합니다. 이러한 제품들을 패밀리라고 하며 이들은 상위 수준의 주제 또는
// 개념으로 연결됩니다. 한 가족의 제품들은 일반적으로 서로 협력할 수 있습니다.
// 제품들의 패밀리​(제품군)​에는 여러 변형이 있을 수 있지만 한 변형의 제품들은 다른
// 변형의 제품들과 호환되지 않습니다.
interface GUIFactory is
    method createButton():Button
    method createCheckbox():Checkbox


// 구상 팩토리들은 단일 변형에 속하는 제품들의 패밀리​(제품군)​을 생성합니다. 이
// 팩토리는 결과 제품들의 호환을 보장합니다. 구상 팩토리 메서드의 시그니처들은 추상
// 제품을 반환하는 반면, 메서드 내부에서는 구상 제품이 인스턴스화됩니다.
class WinFactory implements GUIFactory is
    method createButton():Button is
        return new WinButton()
    method createCheckbox():Checkbox is
        return new WinCheckbox()

// 각 구상 팩토리에는 해당하는 제품 변형이 있습니다.
class MacFactory implements GUIFactory is
    method createButton():Button is
        return new MacButton()
    method createCheckbox():Checkbox is
        return new MacCheckbox()


// 제품 패밀리의 각 개별 제품에는 기초 인터페이스가 있어야 합니다. 이 제품의 모든
// 변형은 이 인터페이스를 구현해야 합니다.
interface Button is
    method paint()

// 구상 제품들은 해당하는 구상 팩토리에서 생성됩니다.
class WinButton implements Button is
    method paint() is
        // 버튼을 윈도우 스타일로 렌더링하세요.

class MacButton implements Button is
    method paint() is
        // 버튼을 맥 스타일로 렌더링하세요.

// 다음은 다른 제품의 기초 인터페이스입니다. 모든 제품은 상호 작용할 수 있지만 같은
// 구상 변형의 제품들 사이에서만 적절한 상호 작용이 가능합니다.
interface Checkbox is
    method paint()

class WinCheckbox implements Checkbox is
    method paint() is
        // 윈도우 스타일의 확인란을 렌더링하세요.

class MacCheckbox implements Checkbox is
    method paint() is
        // 맥 스타일의 확인란을 렌더링하세요.


// 클라이언트 코드는 GUIFactory, Button 및 Checkbox와 같은 추상 유형을
// 통해서만 팩토리들 및 제품들과 작동하며, 이는 클라이언트 코드를 손상하지 않고
// 클라이언트 코드에 모든 팩토리 또는 하위 클래스를 전달할 수 있게 해줍니다.
class Application is
    private field factory: GUIFactory
    private field button: Button
    constructor Application(factory: GUIFactory) is
        this.factory = factory
    method createUI() is
        this.button = factory.createButton()
    method paint() is
        button.paint()


// 앱은 현재 설정 또는 환경 설정에 따라 팩토리 유형을 선택한 후 팩토리를 런타임
// 때​(일반적으로는 초기화 단계에서) 생성합니다.
class ApplicationConfigurator is
    method main() is
        config = readApplicationConfigFile()

        if (config.OS == &quot;Windows&quot;) then
            factory = new WinFactory()
        else if (config.OS == &quot;Mac&quot;) then
            factory = new MacFactory()
        else
            throw new Exception(&quot;Error! Unknown operating system.&quot;)

        Application app = new Application(factory)</code></pre>
</figure>
:::

:::::::: {.section .applicability-container}
##  적용 {#applicability}

::::::: applicability
::: applicability-problem
추상 팩토리는 당신의 코드가 관련된 제품군의 다양한 패밀리들과 작동해야
하지만 해당 제품들의 구상 클래스들에 의존하고 싶지 않을 때 사용하세요.
왜냐하면 이러한 클래스들은 당신에게 미리 알려지지 않았을 수 있으며, 그
때문에 향후 확장성​(extensibility)​을 허용하기를 원할 수 있기 때문입니다.
:::

::: applicability-solution
추상 팩토리는 제품 패밀리의 각 클래스에서부터 객체들을 생성할 수 있는
인터페이스를 제공합니다. 위 인터페이스를 통해 코드가 객체들을 생성하는
한 당신은 당신의 앱에서 이미 생성된 제품들과 일치하지 않는 잘못된 제품
변형을 생성하지 않을지 걱정할 필요가 없습니다.
:::

::: applicability-problem
코드에 클래스가 있고, 이 클래스의 [팩토리
메서드들](factory-method.html)의 집합의 기본 책임이 뚜렷하지 않을 때
추상 팩토리 구현을 고려하세요.
:::

::: applicability-solution
잘 설계된 프로그램에서는 *[각]{.cjk}
[클]{.cjk}[래]{.cjk}[스]{.cjk}[는]{.cjk} [하]{.cjk}[나]{.cjk}[의]{.cjk}
[책]{.cjk}[임]{.cjk}[만]{.cjk}
[가]{.cjk}[집]{.cjk}[니]{.cjk}[다]{.cjk}*. 클래스가 여러 제품 유형을
상대할 경우, 클래스의 팩토리 메서드들을 독립실행형 팩토리 클래스 또는
완전한 추상 팩토리 구현으로 추출할 가치가 있을 수 있습니다.
:::
:::::::
::::::::

::: {.section .checklist}
##  구현방법 {#checklist}

1.  고유한 제품 유형들 대 변형 제품들을 나타내는 매트릭스를 매핑하세요.

2.  모든 제품 변형들에 대한 추상 제품 인터페이스들을 선언하세요. 그 후
    모든 구상 제품 클래스들이 위 인터페이스들을 구현하도록 하세요.

3.  추상 팩토리 인터페이스를 모든 추상 제품들에 대한 생성 메서드들의
    집합과 함께 선언하세요.

4.  각 제품 변형에 대해 각각 하나의 구상 팩토리 클래스 집합을
    구현하세요.

5.  앱 어딘가에 팩토리 초기화 코드를 생성하세요. 초기화 코드는 앱 설정
    또는 현재 환경에 따라 구상 팩토리 클래스 중 하나를 인스턴스화해야
    합니다. 이 팩토리 객체를 제품을 생성하는 모든 클래스들에 전달하세요.

6.  코드를 검토해서 제품 생성자에 대한 모든 직접 호출을 찾으세요. 이
    호출들을 팩토리 객체에 대한 적절한 생성 메서드에 대한 호출들로
    교체하세요.
:::

:::::: {.section .pros-cons}
##  장단점 {#pros-cons}

::::: row
::: col-sm-6
-    팩토리에서 생성되는 제품들의 상호 호환을 보장할 수 있습니다.
-    구상 제품들과 클라이언트 코드 사이의 단단한 결합을 피할 수
    있습니다.
-    *[단]{.cjk}[일]{.cjk} [책]{.cjk}[임]{.cjk} [원]{.cjk}[칙]{.cjk}*.
    제품 생성 코드를 한 곳으로 추출하여 코드를 더 쉽게 유지보수할 수
    있습니다.
-    *[개]{.cjk}[방]{.cjk}/[폐]{.cjk}[쇄]{.cjk} [원]{.cjk}[칙]{.cjk}*.
    기존 클라이언트 코드를 훼손하지 않고 제품의 새로운 변형들을 생성할
    수 있습니다.
:::

::: col-sm-6
-    패턴과 함께 새로운 인터페이스들과 클래스들이 많이 도입되기 때문에
    코드가 필요 이상으로 복잡해질 수 있습니다.
:::
:::::
::::::

::: {.section .relations}
##  다른 패턴과의 관계 {#relations}

-   많은 디자인은 복잡성이 낮고 자식 클래스들을 통해 더 많은
    커스터마이징이 가능한 [팩토리 메서드](factory-method.html)로 시작해
    더 유연하면서도 더 복잡한 [추상 팩토리](abstract-factory.html),
    [프로토타입](prototype.html) 또는 [빌더](builder.html) 패턴으로
    발전해 나갑니다.

-   [빌더](builder.html)는 복잡한 객체들을 단계별로 생성하는 데 중점을
    둡니다. [추상 팩토리](abstract-factory.html)는 관련된 객체들의
    패밀리들을 생성하는 데 중점을 둡니다. *[추]{.cjk}[상]{.cjk}
    [팩]{.cjk}[토]{.cjk}[리]{.cjk}*는 제품을 즉시 반환하지만
    *[빌]{.cjk}[더]{.cjk}*는 제품을 가져오기 전에 당신이 몇 가지 추가
    생성 단계들을 실행할 수 있도록 합니다.

-   [추상 팩토리](abstract-factory.html) 클래스들은 [팩토리
    메서드](factory-method.html)들의 집합을 기반으로 하는 경우가
    많습니다. 그러나 당신은 또한 [프로토타입](prototype.html)을 사용하여
    [추상 팩토리](abstract-factory.html)의 구상 클래스들의 생성
    메서드들을 구현할 수도 있습니다.

-   [추상 팩토리](abstract-factory.html)는 하위시스템 객체들이
    클라이언트 코드에서 생성되는 방식만 숨기고 싶을 때
    [퍼사드](facade.html) 대신 사용할 수 있습니다.

-   당신은 [추상 팩토리](abstract-factory.html)를
    [브리지](bridge.html)와 함께 사용할 수 있습니다. 이 조합은
    *[브]{.cjk}[리]{.cjk}[지]{.cjk}*에 의해 정의된 어떤 추상화들이 특정
    구현들과만 작동할 수 있을 때 유용합니다. 이런 경우에
    *[추]{.cjk}[상]{.cjk} [팩]{.cjk}[토]{.cjk}[리]{.cjk}*는 이러한
    관계들을 캡슐화하고 클라이언트 코드에서부터 복잡성을 숨길 수
    있습니다.

-   [추상 팩토리들](abstract-factory.html), [빌더들](builder.html) 및
    [프로토타입들](prototype.html)은 모두 [싱글턴](singleton.html)으로
    구현할 수 있습니다.
:::

::: {.section .implementations}
##  코드 예시 {#implementations}

[![C#으로 작성된 추상
팩토리](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](abstract-factory/csharp/example.html "C#으로 작성된 추상 팩토리"){.prog-lang-link}
[![C++로 작성된 추상
팩토리](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](abstract-factory/cpp/example.html "C++로 작성된 추상 팩토리"){.prog-lang-link}
[![Go로 작성된 추상
팩토리](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](abstract-factory/go/example.html "Go로 작성된 추상 팩토리"){.prog-lang-link}
[![자바로 작성된 추상
팩토리](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](abstract-factory/java/example.html "자바로 작성된 추상 팩토리"){.prog-lang-link}
[![PHP로 작성된 추상
팩토리](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](abstract-factory/php/example.html "PHP로 작성된 추상 팩토리"){.prog-lang-link}
[![파이썬으로 작성된 추상
팩토리](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](abstract-factory/python/example.html "파이썬으로 작성된 추상 팩토리"){.prog-lang-link}
[![루비로 작성된 추상
팩토리](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](abstract-factory/ruby/example.html "루비로 작성된 추상 팩토리"){.prog-lang-link}
[![러스트로 작성된 추상
팩토리](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](abstract-factory/rust/example.html "러스트로 작성된 추상 팩토리"){.prog-lang-link}
[![스위프트로 작성된 추상
팩토리](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](abstract-factory/swift/example.html "스위프트로 작성된 추상 팩토리"){.prog-lang-link}
[![타입스크립트로 작성된 추상
팩토리](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](abstract-factory/typescript/example.html "타입스크립트로 작성된 추상 팩토리"){.prog-lang-link}
:::

::: {.section .extras}
##  추가 콘텐츠 {#extras}

-   [팩토리 비교](factory-comparison.html)를 참고하세요.
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

[팩토리 비교 []{.fa .fa-arrow-right}](factory-comparison.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### 돌아가기

[[]{.fa .fa-arrow-left} 팩토리 메서드](factory-method.html){.btn
.btn-default rel="prev"}
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
