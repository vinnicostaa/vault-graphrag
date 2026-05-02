:::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [디자인
패턴들](../design-patterns.html){.type} / [생성
패턴](creational-patterns.html){.type}
:::

# 빌더 패턴 {#빌더-패턴 .title}

::: aka
다음 이름으로도 불립니다: [Builder]{style="display:inline-block"}
:::

::: {.section .intent}
##  의도 {#intent}

**빌더**는 복잡한 객체들을 단계별로 생성할 수 있도록 하는 생성 디자인
패턴입니다. 이 패턴을 사용하면 같은 제작 코드를 사용하여 객체의 다양한
유형들과 표현을 제작할 수 있습니다.

<figure class="image">
<img
src="../../images/patterns/content/builder/builder-koc042.png?id=8cc3bb018eb427cbea0fae25d7ede0b2"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/builder/builder-ko.png?id=8cc3bb018eb427cbea0fae25d7ede0b2 640w,/images/patterns/content/builder/builder-ko-1.5x.png?id=ca94fdaf60f4957ee6efab5238fad81a 960w,/images/patterns/content/builder/builder-ko-2x.png?id=0323d0b27af0e2cb577fd04478c68a23 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="빌더 디자인 패턴" />
</figure>
:::

::: {.section .problem}
##  문제 {#problem}

많은 필드와 중첩된 객체들을 힘들게 단계별로 초기화해야 하는 복잡한
객체를 상상해 보세요. 이러한 초기화 코드는 일반적으로 많은 매개변수가
있는 괴물 같은 생성자 내부에 묻혀 있습니다. 또, 더 최악의 상황에는
클라이언트 코드 전체에 흩어져 있을 수도 있습니다.

<figure class="image">
<img
src="../../images/patterns/diagrams/builder/problem16274.png?id=11e715c5c97811f848c48e0f399bb05e"
style="aspect-ratio: 1.71;"
srcset="/images/patterns/diagrams/builder/problem1.png?id=11e715c5c97811f848c48e0f399bb05e 600w,/images/patterns/diagrams/builder/problem1-1.5x.png?id=a46778018c4f0f4bbf2357940c1f5f40 900w,/images/patterns/diagrams/builder/problem1-2x.png?id=02ffbd7ad294600e42aa78989d441b4d 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="자식 클래스들이 너무 많으면 또 다른 문제를 만듭니다" />
<figcaption><p>당신은 객체의 가능한 모든 설정에 자식 클래스를 만들어
프로그램을 매우 복잡하게 만들 수 있습니다.</p></figcaption>
</figure>

예를 들어 `House`​(집) 객체를 만드는 방법에 대해 생각해 봅시다. 간단한
집을 지으려면 네 개의 벽과 바닥을 만든 후 문도 설치하고 한 쌍의 창문도
맞춘 후 지붕도 만들어야 합니다. 하지만 뒤뜰과 기타 물품​(난방 시스템,
배관 및 전기 배선 등)​이 있는 더 크고 현대적인 집을 원하면 어떻게 해야
할까요?

위 문제의 가장 간단한 해결책은 기초 `House` 클래스를 확장하고 매개변수의
모든 조합을 포함하는 자식 클래스들의 집합을 만드는 것입니다. 그러나
당신은 결국 상당한 수의 자식 클래스를 만들게 될 것입니다. 새로운
매개변수​(예: 현관 스타일)​를 추가할 때마다 이 계층구조는 훨씬 더 복잡해질
것입니다.

자식 클래스들을 늘리지 않는 다른 접근 방식이 있습니다. 기초 `House`
클래스에 `House` 객체를 제어하는 모든 가능한 매개변수를 포함한 거대한
생성자를 만드는 것입니다. 이 접근 방식은 실제로 자식 클래스들의 필요성을
제거하나, 다른 문제를 만들어 냅니다.

<figure class="image">
<img
src="../../images/patterns/diagrams/builder/problem27ab5.png?id=2e91039b6c7d2d2df6ee519983a3b036"
style="aspect-ratio: 1.71;"
srcset="/images/patterns/diagrams/builder/problem2.png?id=2e91039b6c7d2d2df6ee519983a3b036 600w,/images/patterns/diagrams/builder/problem2-1.5x.png?id=3d302cf2762fd04d70cbb91cb54e923c 900w,/images/patterns/diagrams/builder/problem2-2x.png?id=5e7975a91c0e4f4ba960f908cc9c2ea2 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="점층적 생성자" />
<figcaption><p>매개변수가 많은 생성자의 단점은 모든 매개변수가 항상
필요한 것은 아니라는 점입니다.</p></figcaption>
</figure>

보통 대부분의 매개변수가 사용되지 않아 [생성자 호출들의 코드가 매우
못생겨질 것입니다](../smells/long-parameter-list.html). 예를 들어,
극소수의 집들에만 수영장이 있으므로 수영장과 관련된 매개변수들은
십중팔구 사용되지 않을 것입니다.
:::

::: {.section .solution}
##  해결책 {#solution}

빌더 패턴은 자신의 클래스에서 객체 생성 코드를 추출하여
*builders*(건축업자들)​라는 별도의 객체들로 이동하도록 제안합니다.

<figure class="image">
<img
src="../../images/patterns/diagrams/builder/solution13e5f.png?id=8ce82137f8935998de802cae59e00e11"
style="aspect-ratio: 1.46;"
srcset="/images/patterns/diagrams/builder/solution1.png?id=8ce82137f8935998de802cae59e00e11 410w,/images/patterns/diagrams/builder/solution1-1.5x.png?id=8ab77eb73760a75c35940bae243d43f2 615w,/images/patterns/diagrams/builder/solution1-2x.png?id=a9c2ab02f0b2aca1a7512022194dd113 820w"
sizes="(max-width: 720px) 100vw, 410px" loading="lazy" width="410"
alt="빌더 패턴 적용" />
<figcaption><p>빌더 패턴은 복잡한 객체들을 단계별로 생성할 수 있도록
합니다. 빌더는 제품이 생성되는 동안 다른 객체들이 제품에
접근​(access)​하는 것을 허용하지 않습니다.</p></figcaption>
</figure>

이 패턴은 객체 생성을 일련의 단계들​(`build­Walls`​(벽 건설),
`build­Door`​(문 건설) 등)​로 정리하며, 객체를 생성하고 싶으면 위 단계들을
*builder*(빌더) 객체에 실행하면 됩니다. 또 중요한 점은 모든 단계를
호출할 필요가 없다는 것으로, 객체의 특정 설정을 제작하는 데 필요한
단계들만 호출하면 됩니다.

일부 건축 단계들은 제품의 다양한 표현을 건축해야 하는 경우 다른 구현들이
필요할 수 있습니다. 예를 들어, 오두막의 벽은 나무로 지을 수 있지만
성벽은 돌로 지어야 합니다.

이런 경우 같은 건축 단계들의 집합을 다른 방식으로 구현하는 여러 다른
빌더 클래스를 생성할 수 있으며, 그런 다음 건축 프로세스​(즉, 건축 단계에
대한 순서화된 호출들의 집합)​내에서 이러한 빌더들을 사용하여 다양한
종류의 객체를 생성할 수 있습니다.

<figure class="image">
<img
src="../../images/patterns/content/builder/builder-comic-1-ko6c9a.png?id=988697e68bae7bc26d3cb38ffa44eeb1"
style="aspect-ratio: 2;"
srcset="/images/patterns/content/builder/builder-comic-1-ko.png?id=988697e68bae7bc26d3cb38ffa44eeb1 600w,/images/patterns/content/builder/builder-comic-1-ko-1.5x.png?id=e3829e9afe2e42a7e334f19851adf951 900w,/images/patterns/content/builder/builder-comic-1-ko-2x.png?id=0a67d6792a55a8351513f9a48a09fcec 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600" />
<figcaption><p>다양한 빌더들은 다양한 방식으로 같은
작업을 실행합니다.</p></figcaption>
</figure>

예를 들어, 나무와 유리로 모든 것을 건축하는 건축가, 돌과 철로 모든 것을
건축하는 두 번째 건축가, 금과 다이아몬드로 모든 것을 건축하는 세 번째
건축가가 있다고 상상해 보세요. 이 세 건축가에 대해 같은 단계들의 집합을
호출하면 첫 번째 건축업자에게서부터는 일반 주택을, 두 번째
건축업자에게서부터는 작은 성을, 세 번째 건축업자에게서부터는 궁전을
얻습니다. 그러나 위에 예시된 경우는 건축 단계들을 호출하는 클라이언트
코드가 공통 인터페이스를 사용하여 빌더들과 상호 작용할 수 있는 경우에만
작동합니다.

#### 디렉터 (관리자)

더 나아가 제품을 생성하는 데 사용하는 빌더 단계들에 대한 일련의 호출을
*[디]{.cjk}[렉]{.cjk}[터]{.cjk} ([관]{.cjk}[리]{.cjk}[자]{.cjk})*라는
별도의 클래스로 추출할 수 있습니다. 디렉터 클래스는 제작 단계들을
실행하는 순서를 정의하는 반면 빌더는 이러한 단계들에 대한 구현을
제공합니다.

<figure class="image">
<img
src="../../images/patterns/content/builder/builder-comic-2-ko44c2.png?id=7c52497b2b701bce83af07f3f5ba8895"
style="aspect-ratio: 1.14;"
srcset="/images/patterns/content/builder/builder-comic-2-ko.png?id=7c52497b2b701bce83af07f3f5ba8895 343w,/images/patterns/content/builder/builder-comic-2-ko-1.5x.png?id=d4065be88ea3f4fe03ade0843ebb24e9 515w,/images/patterns/content/builder/builder-comic-2-ko-2x.png?id=e69c84869dfc53646024e7316f1f4cdf 687w"
sizes="(max-width: 720px) 100vw, 343px" loading="lazy" width="343" />
<figcaption><p>디렉터는 작동하는 제품을 얻기 위하여 어떤 건축 단계들을
실행해야 하는지 알고 있습니다.</p></figcaption>
</figure>

프로그램에 디렉터 클래스를 포함하는 것은 필수사항은 아닙니다. 당신은
언제든지 클라이언트 코드에서 생성 단계들을 직접 특정 순서로 호출할 수
있습니다. 그러나 디렉터 클래스는 다양한 생성 루틴들을 배치하여 프로그램
전체에서 재사용할 수 있는 좋은 장소가 될 수 있습니다.

또한 디렉터 클래스는 클라이언트 코드에서 제품 생성의 세부 정보를 완전히
숨깁니다. 클라이언트는 빌더를 디렉터와 연관시키고 디렉터와 생성을 시행한
후 빌더로부터 결과를 얻기만 하면 됩니다.
:::

::::: {.section .structure-container}
##  구조 {#structure}

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/builder/structure05e4.png?id=fe9e23559923ea0657aa5fe75efef333"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 0.79;"
srcset="/images/patterns/diagrams/builder/structure.png?id=fe9e23559923ea0657aa5fe75efef333 460w,/images/patterns/diagrams/builder/structure-1.5x.png?id=91ea8cd3137b403542c23abf63938f00 690w,/images/patterns/diagrams/builder/structure-2x.png?id=dca1b1508e23c266cbedc80ffb84311a 920w"
sizes="(max-width: 720px) 100vw, 460px" loading="lazy" width="460"
alt="빌더 디자인 패턴 구조" /><img
src="../../images/patterns/diagrams/builder/structure-indexedadac.png?id=44b3d763ce91dbada5d8394ef777437f"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 0.81;"
srcset="/images/patterns/diagrams/builder/structure-indexed.png?id=44b3d763ce91dbada5d8394ef777437f 470w,/images/patterns/diagrams/builder/structure-indexed-1.5x.png?id=d57a6ff9342ea31736ea98e5066e0748 705w,/images/patterns/diagrams/builder/structure-indexed-2x.png?id=153eed9a51784cbe00d0ca8b3d6b268d 940w"
sizes="(max-width: 720px) 100vw, 470px" loading="lazy" width="470"
alt="빌더 디자인 패턴 구조" />
</figure>
:::

1.  **빌더** 인터페이스는 모든 유형의 빌더들에 공통적인 제품 생성
    단계들을 선언합니다.

2.  **구상 빌더**들은 생성 단계들의 다양한 구현을 제공합니다. 또 구상
    빌더들은 공통 인터페이스를 따르지 않는 제품들도 생산할 수 있습니다.

3.  **제품들**은 그 결과로 나온 객체들입니다. 다른 빌더들에 의해 생성된
    제품들은 같은 클래스 계층구조 또는 인터페이스에 속할 필요가
    없습니다.

4.  **디렉터** 클래스는 생성 단계들을 호출하는 순서를 정의하므로
    제품들의 특정 설정을 만들고 재사용할 수 있습니다.

5.  **클라이언트**는 빌더 객체들 중 하나를 디렉터와 연결해야 합니다.
    일반적으로 위 연결은 디렉터 생성자의 매개변수들을 통해 한 번만
    수행되며, 그 후 디렉터는 모든 추가 생성에 해당 빌더 객체들을
    사용합니다. 그러나 클라이언트가 빌더 객체를 디렉터의 프로덕션
    메서드에 전달할 때를 위한 대안적 접근 방식이 있습니다. 이 경우
    디렉터와 함께 무언가를 만들 때마다 다른 빌더를 사용할 수 있습니다.
::::
:::::

::: {.section .pseudocode}
##  의사코드 {#pseudocode}

아래 **빌더** 패턴 예시는 자동차와 같은 다양한 유형의 제품들을 생성할 때
동일한 객체 생성 코드를 재사용하고 그에 해당하는 매뉴얼을 만드는 방법을
보여줍니다.

<figure class="image">
<img
src="../../images/patterns/diagrams/builder/example-ko2952.png?id=1b837f980205cffaf5220c41642254ad"
style="aspect-ratio: 0.85;"
srcset="/images/patterns/diagrams/builder/example-ko.png?id=1b837f980205cffaf5220c41642254ad 500w,/images/patterns/diagrams/builder/example-ko-1.5x.png?id=65063090ba3795f6f4f30aa23ce34430 750w,/images/patterns/diagrams/builder/example-ko-2x.png?id=13219fb8281c4e71c578561e218e7075 1000w"
sizes="(max-width: 720px) 100vw, 500px" loading="lazy" width="500"
alt="빌더 디자인 패턴 구조 예시" />
<figcaption><p>자동차들의 단계별 생성과 해당 자동차 모델들에 맞는 사용자
설명서들의 예시.</p></figcaption>
</figure>

자동차는 수백 가지 방법으로 생성될 수 있는 복잡한 객체입니다. 거대한
생성자로 `Car`​(자동차) 클래스를 부풀리는 대신, 우리는 자동차 조립 코드를
별도의 자동차 빌더 클래스로 추출했습니다. 이 클래스에는 자동차의 다양한
부분들을 설정하기 위한 메서드들의 집합이 있습니다.

클라이언트 코드가 미세하게 조정된 특별한 자동차 모델을 조립해야 하는
경우 빌더와 직접 작동할 수 있습니다. 반면에 클라이언트는 조립을 디렉터
클래스로 위임할 수 있으며, 이 디렉터 클래스는 빌더를 사용하여 가장 인기
있는 자동차 모델들을 조립하는 방법들을 알고 있습니다.

모든 자동차에는 사용설명서가 필요합니다. 사용설명서에는 해당 자동차의
모든 기능이 설명되어 있으므로 설명서의 세부 사항은 모델마다 다릅니다.
그러므로 실제 자동차들과 그들의 해당 사용설명서들 모두에 기존 제작
프로세스들을 재사용하는 것이 합리적입니다. 물론 사용설명서를 만드는 것은
자동차를 만드는 것과 같지 않기 때문에 사용설명서 작성을 전문으로 하는 또
다른 빌더 클래스를 제공해야 합니다. 이 클래스는 자동차 제작 관련 형제
클래스와 같은 제작 메서드들을 구현하지만, 자동차 부품들을 제작하는 대신
설명합니다. 이러한 빌더들을 같은 디렉터 객체에 전달하여 자동차 또는
설명서를 제작할 수 있습니다.

마지막 부분은 결과 객체를 가져오는 것입니다. 금속으로 만들어진 자동차와
종이 사용설명서는 관련은 있지만 여전히 매우 다른 사물들입니다. 디렉터를
구상 제품 클래스들에 결합하지 않고 디렉터에 결과를 가져오는 메서드를
배치할 수는 없습니다. 따라서 우리는 작업을 수행한 빌더로부터 생성의
결과를 얻습니다.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// 빌더 패턴을 사용하는 것은 제품에 매우 복잡하고 광범위한 설정이 필요한 경우에만
// 의미가 있습니다. 다음 두 제품은 공통 인터페이스는 없지만 관련되어 있습니다.
class Car is
    // 자동차에는 GPS, 트립 컴퓨터 및 몇 개의 좌석이 있을 수 있습니다. 다른
    // 모델의 자동차​(스포츠카, SUV, 오픈카)​에는 다른 기능들이 설치되거나
    // 활성화되어 있을 수 있습니다.

class Manual is
    // 각 자동차에는 자동차의 설정에 해당하는, 모든 기능을 설명하는 사용 설명서가
    // 있어야 합니다.


// 빌더 인터페이스는 제품 객체들의 다른 부분들을 만드는 메서드들을 지정합니다.
interface Builder is
    method reset()
    method setSeats(...)
    method setEngine(...)
    method setTripComputer(...)
    method setGPS(...)

// 구상 빌더 클래스들은 빌더 인터페이스를 따르고 빌드 단계들의 특정 구현들을
// 제공합니다. 당신의 프로그램에는 각기 다르게 구현된 여러 가지 빌더 변형들이 있을
// 수 있습니다.
class CarBuilder implements Builder is
    private field car:Car

    // 새로운 빌더 인스턴스에는 인스턴스가 추가적인 조립과정에서 사용하는 빈 제품
    // 객체가 포함되어야 합니다.
    constructor CarBuilder() is
        this.reset()

    // reset 메서드는 구축 중인 객체를 지웁니다.
    method reset() is
        this.car = new Car()

    // 모든 생성 단계들은 같은 제품의 인스턴스와 작동합니다.
    method setSeats(...) is
        // 차량의 좌석 수를 설정하세요.

    method setEngine(...) is
        // 해당 엔진을 설치하세요.

    method setTripComputer(...) is
        // 트립 컴퓨터를 설치하세요.

    method setGPS(...) is
        // GPS를 설치하세요.

    // 구상 빌더들은 결과들을 가져오기 위한 자체 메서드들을 제공해야 합니다.
    // 왜냐하면 다양한 유형의 빌더들은 모두 같은 인터페이스를 따르지 않는 완전히
    // 다른 제품들을 생성할 수 있기 때문입니다. 따라서 이러한 메서드는 빌더
    // 인터페이스에서 선언할 수 없습니다. 적어도 이는 정적 타입 언어에서는
    // 불가능합니다.
    //
    // 최종 결과를 클라이언트에 반환한 후 일반적으로 빌더 인스턴스는 다른 제품
    // 생산을 시작할 준비가 되어 있을 것이라고 예상됩니다. 이것이
    // `getProduct` 메서드의 본문 끝에서 reset 메서드를 호출하는 것이
    // 일반적인 관행인 이유입니다. 하지만 반드시 이렇게 해야 하는 것은
    // 아니라서, 빌더가 클라이언트 코드로부터 명시적으로 reset 호출을 받을
    // 때까지 이전 결과를 삭제하지 않고 기다리게 만들 수 있습니다.
    method getProduct():Car is
        product = this.car
        this.reset()
        return product

// 다른 생성 패턴과 달리 빌더를 사용하면 공통 인터페이스를 따르지 않는 제품들을
// 생성할 수 있습니다.
class CarManualBuilder implements Builder is
    private field manual:Manual

    constructor CarManualBuilder() is
        this.reset()

    method reset() is
        this.manual = new Manual()

    method setSeats(...) is
        // 자동차 좌석의 기능들을 문서화하세요.

    method setEngine(...) is
        // 엔진 사용 지침을 추가하세요.

    method setTripComputer(...) is
        // 트립 컴퓨터 사용 지침을 추가하세요.

    method setGPS(...) is
        // GPS 사용 지침을 추가하세요.

    method getProduct():Manual is
        // 매뉴얼을 반환하고 빌더를 초기화하세요.


// 디렉터는 특정 순서로 생성 단계들을 실행하는 책임만 있습니다. 이것은 특정 순서나
// 설정에 따라 제품들을 생성할 때 유용합니다. 엄밀히 말하면, 클라이언트가 빌더들을
// 직접 제어할 수 있으므로 디렉터 클래스는 선택 사항입니다.
class Director is
    // 디렉터는 클라이언트 코드가 전달하는 모든 빌더 인스턴스와 함께 작동합니다.
    // 그러면 클라이언트 코드는 새로 조립된 제품의 최종 유형을 변경할 수
    // 있습니다. 디렉터는 같은 생성 단계들을 사용하여 여러 제품 변형들을 생성할
    // 수 있습니다.
    method constructSportsCar(builder: Builder) is
        builder.reset()
        builder.setSeats(2)
        builder.setEngine(new SportEngine())
        builder.setTripComputer(true)
        builder.setGPS(true)

    method constructSUV(builder: Builder) is
        // …


// 클라이언트 코드는 빌더 객체를 만든 후 이를 디렉터에게 전달한 다음 생성
// 프로세스를 시작합니다. 최종 결과는 빌더 객체에서 가져옵니다.
class Application is

    method makeCar() is
        director = new Director()

        CarBuilder builder = new CarBuilder()
        director.constructSportsCar(builder)
        Car car = builder.getProduct()

        CarManualBuilder builder = new CarManualBuilder()
        director.constructSportsCar(builder)

        // 디렉터는 구상 빌더들 및 제품들에 의존하지 않고 인식하지 못하기 때문에
        // 최종 제품은 종종 빌더 객체에서 가져옵니다.
        Manual manual = builder.getProduct()</code></pre>
</figure>
:::

:::::::::: {.section .applicability-container}
##  적용 {#applicability}

::::::::: applicability
::: applicability-problem
\'점층적 생성자\'를 제거하기 위하여 빌더 패턴을 사용하세요.
:::

::: applicability-solution
10개의 선택적 매개변수가 있는 생성자가 있다고 가정합니다. 이렇게 복잡한
생성자를 호출하는 것은 매우 불편합니다. 따라서 이 생성자를 오버로드하고
더 적은 수의 매개변수들을 사용하는 더 짧은 생성자 버전들을 여러 개
만듭니다. 이러한 생성자들은 여전히 주 생성자를 참조하며, 생략된
매개변수들에 일부 기본값들을 전달합니다.

<figure class="code">
<pre class="code" lang="java"><code>class Pizza {
    Pizza(int size) { ... }
    Pizza(int size, boolean cheese) { ... }
    Pizza(int size, boolean cheese, boolean pepperoni) { ... }
    // …</code></pre>
<figcaption><p>이렇게 복잡한 생성자를 만드는 것은 C#나 자바와 같이
메서드 오버로딩을 지원하는 언어들에서만 가능합니다.</p></figcaption>
</figure>

빌더 패턴을 사용하면 실제로 필요한 단계들만 사용하여 단계별로 객체들을
생성할 수 있으며, 패턴을 구현한 후에는 더 이상 수십 개의 매개변수를
생성자에 집어넣을 필요가 없습니다.
:::

::: applicability-problem
빌더 패턴은 당신의 코드가 일부 제품의 다른 표현들​(예: 석조 및 목조
주택들)​을 생성할 수 있도록 하고 싶을 때 사용하세요.
:::

::: applicability-solution
빌더 패턴은 제품의 다양한 표현의 생성 과정이 세부 사항만 다른 유사한
단계를 포함할 때 적용할 수 있습니다.

기초 빌더 인터페이스는 가능한 모든 생성 단계들을 정의하고 구상 빌더들은
이러한 단계들을 구현하여 제품의 여러 표현을 생성합니다. 또 한편 디렉터
클래스는 건설 순서를 안내합니다.
:::

::: applicability-problem
빌더를 사용하여 [복합체](composite.html) 트리들 또는 기타 복잡한
객체들을 생성하세요.
:::

::: applicability-solution
빌더 패턴을 사용하면 제품들을 단계별로 생성할 수 있으며, 또 최종 제품을
손상하지 않고 일부 단계들의 실행을 연기할 수 있습니다. 그리고 재귀적으로
단계들을 호출할 수도 있는데, 이는 객체 트리를 구축해야 할 때 매우
유용합니다.

빌더는 생성 단계들을 수행하는 동안 미완성 제품을 노출하지 않으며, 이는
클라이언트 코드가 불완전한 결과를 가져오는 것을 방지합니다.
:::
:::::::::
::::::::::

::: {.section .checklist}
##  구현방법 {#checklist}

1.  사용할 수 있는 모든 제품 표현을 생성하기 위한 공통 생성 단계들을
    명확하게 정의할 수 있는지 확인하세요. 그렇게 하지 못하면, 패턴
    구현을 진행할 수 없습니다.

2.  기초 빌더 인터페이스에서 이 단계를 선언하세요.

3.  각 제품 표현에 대한 구상 빌더 클래스를 만들고 해당 생성 단계들을
    구현하세요.

    생성 결과를 가져오는 메서드를 구현하는 것을 잊지 마세요. 빌더
    인터페이스 내에서 이 메서드를 선언할 수 없는 이유는 다양한 빌더들이
    공통 인터페이스가 없는 제품들을 생성할 수 있기 때문입니다. 따라서
    이러한 메서드의 반환 유형이 무엇인지 알 수 없습니다. 그러나 단일
    계층구조의 제품들을 처리하는 경우 생성 결과를 가져오는 메서드를 기초
    인터페이스에 안전하게 추가할 수 있습니다.

4.  디렉터 클래스를 만드는 것에 대해 생각해보세요. 같은 빌더 객체를
    사용하여 제품을 제작하는 다양한 방법을 캡슐화할 수 있습니다.

5.  클라이언트 코드는 빌더 객체들과 디렉터 객체들을 모두 생성합니다.
    제작이 시작되기 전에 클라이언트는 빌더 객체를 디렉터에게 전달해야
    합니다. 일반적으로 클라이언트는 디렉터의 클래스 생성자의
    매개변수들을 통해 위 작업을 한 번만 수행합니다. 그 후 디렉터는 모든
    추가 제작에서 빌더 객체를 사용합니다. 또 대안적인 접근 방식이
    있는데, 이 방식은 빌더가 디렉터의 특정 제품 제작 메서드에 전달되는
    것입니다.

6.  모든 제품이 같은 인터페이스를 따르는 경우에만 디렉터로부터 직접 생성
    결과를 얻을 수 있습니다. 그렇지 않으면 클라이언트는 빌더에서 결과를
    가져와야 합니다.
:::

:::::: {.section .pros-cons}
##  장단점 {#pros-cons}

::::: row
::: col-sm-6
-    객체들을 단계별로 생성하거나 생성 단계들을 연기하거나 재귀적으로
    단계들을 실행할 수 있습니다.
-    제품들의 다양한 표현을 만들 때 같은 생성 코드를 재사용할 수
    있습니다.
-    *[단]{.cjk}[일]{.cjk} [책]{.cjk}[임]{.cjk} [원]{.cjk}[칙]{.cjk}*.
    제품의 비즈니스 로직에서 복잡한 생성 코드를 고립시킬 수 있습니다.
:::

::: col-sm-6
-    패턴이 여러 개의 새 클래스들을 생성해야 하므로 코드의 전반적인
    복잡성이 증가합니다.
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

-   당신은 복잡한 [복합체](composite.html) 패턴 트리를 생성할 때
    [빌더](builder.html)를 사용할 수 있습니다. 왜냐하면 빌더의 생성
    단계들을 재귀적으로 작동하도록 프로그래밍할 수 있기 때문입니다.

-   [빌더](builder.html)를 [브리지](bridge.html)와 조합할 수 있습니다.
    디렉터 클래스는 추상화의 역할을 하고 다양한 빌더들은 구현의 역할을
    합니다.

-   [추상 팩토리들](abstract-factory.html), [빌더들](builder.html) 및
    [프로토타입들](prototype.html)은 모두 [싱글턴](singleton.html)으로
    구현할 수 있습니다.
:::

::: {.section .implementations}
##  코드 예시 {#implementations}

[![C#으로 작성된
빌더](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](builder/csharp/example.html "C#으로 작성된 빌더"){.prog-lang-link}
[![C++로 작성된
빌더](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](builder/cpp/example.html "C++로 작성된 빌더"){.prog-lang-link}
[![Go로 작성된
빌더](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](builder/go/example.html "Go로 작성된 빌더"){.prog-lang-link}
[![자바로 작성된
빌더](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](builder/java/example.html "자바로 작성된 빌더"){.prog-lang-link}
[![PHP로 작성된
빌더](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](builder/php/example.html "PHP로 작성된 빌더"){.prog-lang-link}
[![파이썬으로 작성된
빌더](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](builder/python/example.html "파이썬으로 작성된 빌더"){.prog-lang-link}
[![루비로 작성된
빌더](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](builder/ruby/example.html "루비로 작성된 빌더"){.prog-lang-link}
[![러스트로 작성된
빌더](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](builder/rust/example.html "러스트로 작성된 빌더"){.prog-lang-link}
[![스위프트로 작성된
빌더](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](builder/swift/example.html "스위프트로 작성된 빌더"){.prog-lang-link}
[![타입스크립트로 작성된
빌더](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](builder/typescript/example.html "타입스크립트로 작성된 빌더"){.prog-lang-link}
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

[프로토타입 []{.fa .fa-arrow-right}](prototype.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### 돌아가기

[[]{.fa .fa-arrow-left} 팩토리 비교](factory-comparison.html){.btn
.btn-default rel="prev"}
:::
:::::::::::::::::::::::::::::::::

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
:::::::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::::::
