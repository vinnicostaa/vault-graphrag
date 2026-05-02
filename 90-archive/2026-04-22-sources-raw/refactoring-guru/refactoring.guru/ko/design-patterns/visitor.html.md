:::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [디자인
패턴들](../design-patterns.html){.type} / [행동
패턴](behavioral-patterns.html){.type}
:::

# 비지터 패턴 {#비지터-패턴 .title}

::: aka
다음 이름으로도 불립니다: [Visitor]{style="display:inline-block"}
:::

::: {.section .intent}
##  의도 {#intent}

**비지터**(방문자) 패턴은 알고리즘들을 그들이 작동하는 객체들로부터
분리할 수 있도록 하는 행동 디자인 패턴입니다.

<figure class="image">
<img
src="../../images/patterns/content/visitor/visitor4a4c.png?id=f36d100188340db7a18854ef7916f972"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/visitor/visitor.png?id=f36d100188340db7a18854ef7916f972 640w,/images/patterns/content/visitor/visitor-1.5x.png?id=751e251363b632b20df856d72d02ee72 960w,/images/patterns/content/visitor/visitor-2x.png?id=2c5d9ab3046d782c19809d3b80650d65 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="비지터 디자인 패턴" />
</figure>
:::

::: {.section .problem}
##  문제 {#problem}

당신의 팀이 하나의 거대한 그래프로 구성된 지리 정보를 사용해 작동하는
앱을 개발하고 있다고 가정해 봅시다. 그래프의 각 노드는 도시와 같은
복잡한 객체를 나타낼 수 있지만 산업들, 관광 지역들 등의 더 세부적인
항목들도 나타낼 수 있습니다. 만약에 노드들이 나타내는 실제 객체들 사이에
도로가 있으면 노드들은 서로 연결됩니다. 각 노드 유형은 자체 클래스지만
각 노드는 객체입니다.

<figure class="image">
<img
src="../../images/patterns/diagrams/visitor/problem14338.png?id=e7076532da1e936f3519c63270da8454"
style="aspect-ratio: 2.43;"
srcset="/images/patterns/diagrams/visitor/problem1.png?id=e7076532da1e936f3519c63270da8454 560w,/images/patterns/diagrams/visitor/problem1-1.5x.png?id=250216d3458c38e9855eda96bf71251b 840w,/images/patterns/diagrams/visitor/problem1-2x.png?id=2e5d5143ac55af218754f28761bec17e 1120w"
sizes="(max-width: 720px) 100vw, 560px" loading="lazy" width="560"
alt="그래프를 XML로 내보내기" />
<figcaption><p>그래프를 XML 형식으로 내보내기</p></figcaption>
</figure>

어느 날 당신은 그래프를 XML 형식으로 내보내는 작업을 구현하는 일을
맡았습니다. 처음에는 일이 매우 간단해 보였습니다. 당신은 각 노드
클래스에 내보내기 메서드를 추가한 다음 재귀를 활용하여 그래프의 각
노드에 작업하며 내보내기 메서드를 실행할 계획이었습니다. 해결책은
간단하고 우아했습니다. 다형성 덕분에 내보내기 메서드를 호출하는 코드를
노드들의 구상 클래스들에 결합하지 않았습니다.

불행히도 시스템의 설계자는 기존 노드 클래스들을 변경하는 것을 허용하지
않았습니다. 그는 코드가 이미 프로덕션 단계에 있으며 당신이 제안한 변경
사항들이 오류를 일으킬 수 있으므로 코드가 손상되는 위험을 감수하고 싶지
않다고 말했습니다.

<figure class="image">
<img
src="../../images/patterns/diagrams/visitor/problem2-ko107e.png?id=7308e59f92889a0c813845d33aa0a957"
style="aspect-ratio: 1.92;"
srcset="/images/patterns/diagrams/visitor/problem2-ko.png?id=7308e59f92889a0c813845d33aa0a957 500w,/images/patterns/diagrams/visitor/problem2-ko-1.5x.png?id=cbd07af8744e0c7352b4f6b09c61cdad 750w,/images/patterns/diagrams/visitor/problem2-ko-2x.png?id=e14f9a7401e3c1e573505251d0eeced4 1000w"
sizes="(max-width: 720px) 100vw, 500px" loading="lazy" width="500"
alt="XML 내보내기 메서드는 모든 노드 클래스들에 추가되어야 했습니다" />
<figcaption><p>XML 내보내기 메서드는 모든 노드 클래스에 추가되어야
했으며, 이러한 변경과 함께 버그가 발생하면 전체 앱이 망가질
위험이 있었습니다.</p></figcaption>
</figure>

또 시스템 설계자는 노드 클래스들 내에 XML 내보내기 코드를 넣는 것이
적절한지에 대한 의문을 제기했습니다. 이 클래스들의 주 작업은 지리
데이터를 처리하는 것이므로, XML 내보내기 동작은 그곳에서 이상하게 보일
것이라고 했습니다.

시스템 설계자의 거절에는 또 다른 이유도 있었습니다. 위 기능이 구현된
후에도 마케팅 부서의 누군가가 데이터를 다른 형식으로 내보낼 수 있는 기능
또는 다른 기능을 요청할 가능성이 있으며, 그러면 당신은 다시 이 망가지기
쉬운 클래스들을 다시 한번 변경해야 한다는 것이었습니다.
:::

::: {.section .solution}
##  해결책 {#solution}

비지터 패턴은 당신이 새로운 행동을 기존 클래스들에 통합하는 대신
*visitor*(방문자)​라는 별도의 클래스에 배치할 것을 제안합니다. 이제
행동을 수행해야 했던 원래 객체는 visitor의 메서드 중 하나에 인수로
전달됩니다. 그러면 메서드는 원래 객체 내에 포함된 모든 필요한 데이터에
접근할 수 있습니다.

이제 그 행동이 다른 클래스들의 객체들에 대해 실행될 수 있다면 어떨까요?
예를 들어 XML 내보내기의 경우 실제 구현은 다양한 노드 클래스들에서
약간씩 다를 수 있습니다. 따라서 비지터 클래스는 단일 메서드를 정의하는
대신 다음과 같이 메서드의 집합을 정의하여 각 메서드가 다른 유형의 인수를
받을 수 있도록 합니다:

<figure class="code">
<pre class="code" lang="pseudocode"><code>class ExportVisitor implements Visitor is
    method doForCity(City c) { ... }
    method doForIndustry(Industry f) { ... }
    method doForSightSeeing(SightSeeing ss) { ... }
    // …</code></pre>
</figure>

그러나 우리는 이러한 메서드들을 정확히 어떻게 호출할까요? 특히 전체
그래프를 다룰 때 말입니다. 이 메서드들은 시그니처들이 다르므로 다형성을
사용할 수 없습니다. 주어진 객체를 처리할 수 있는 적절한 비지터 메서드를
선택하려면 먼저 그 클래스를 확인해야 합니다. 너무 복잡하지 않나요?

<figure class="code">
<pre class="code" lang="pseudocode"><code>foreach (Node node in graph)
    if (node instanceof City)
        exportVisitor.doForCity((City) node)
    if (node instanceof Industry)
        exportVisitor.doForIndustry((Industry) node)
    // …
}</code></pre>
</figure>

여기서 당신은 메서드 오버로딩을 사용하는 게 어떻겠냐고 제안할지도
모릅니다. 메서드 오버로딩은 다른 매개변수들의 집합들을 지원하더라도 모든
메서드에 같은 이름을 지정하는 방식입니다. 당신이 사용하는 프로그래밍
언어가 자바나 C#처럼 메서드 오버로딩을 지원한다고 가정하더라도, 그건
우리에겐 도움이 되지 않을 겁니다. 노드 객체의 정확한 클래스를 사전에 알
수 없으므로, 오버로딩 메커니즘은 실행해야 할 올바른 메서드가 무엇인지
판단할 수 없고, 따라서 디폴트​(기본값)​로 기초 `Node` 클래스의 객체를 받는
메서드를 선택하게 됩니다.

그러나 비지터 패턴에서는 이 문제를 [더블
디스패치](visitor-double-dispatch.html)라는 방법을 사용하여 해결합니다.
이 방법은 번거로운 조건문 없이 객체에 적절한 메서드를 실행하는 것을
돕습니다. 클라이언트가 호출할 메서드의 적절한 버전을 선택하도록 하는
대신 이 선택권을 비지터에게 인수로 전달되는 객체에게 위임합니다. 이러한
객체들은 자신의 클래스들을 알고 있으므로 비지터에 대한 적합한 메서드를
더 쉽게 선택할 수 있습니다. 그들은 비지터를 \'수락\'하고 어떤 비지터
메서드가 실행되어야 하는지 알려줍니다.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// Client code
foreach (Node node in graph)
    node.accept(exportVisitor)

// City
class City is
    method accept(Visitor v) is
        v.doForCity(this)
    // …

// Industry
class Industry is
    method accept(Visitor v) is
        v.doForIndustry(this)
    // …</code></pre>
</figure>

죄송합니다. 결국 노드 클래스들을 변경해야 했습니다. 그러나 최소한 변경
사항들은 사소했으며, 이제 코드를 다시 변경하지 않고도 다른 행동들을
추가할 수 있습니다.

이제 모든 비지터에 대한 공통 인터페이스를 추출하면 기존의 모든 노드가
당신이 앱에 도입하는 모든 비지터와 함께 작동할 수 있습니다. 노드와
관련된 새로운 행동을 도입하려면 새 비지터 클래스를 구현하기만 하면
됩니다.
:::

::: {.section .analogy}
##  실제상황 적용 {#analogy}

<figure class="image">
<img
src="../../images/patterns/content/visitor/visitor-comic-17f02.png?id=7ee4fa8800f7c4df4e1aa3b1aca2b7f1"
style="aspect-ratio: 2;"
srcset="/images/patterns/content/visitor/visitor-comic-1.png?id=7ee4fa8800f7c4df4e1aa3b1aca2b7f1 600w,/images/patterns/content/visitor/visitor-comic-1-1.5x.png?id=c9cadd73d25cc63fce94312f3c82bfee 900w,/images/patterns/content/visitor/visitor-comic-1-2x.png?id=439032451eb49ebbcb5257f25ecee790 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="보험 대리인" />
<figcaption><p>좋은 보험 대리인은 항상 다양한 유형의 조직들에 적절한
보험을 판매할 준비가 되어 있습니다.</p></figcaption>
</figure>

새로운 고객을 확보하고 싶어 하는 노련한 보험 대리인을 상상해 봅시다.
그는 근방의 모든 건물을 방문하여 만나는 모든 사람에게 보험을 판매하려고
합니다. 그는 방문한 건물에 있는 회사 또는 조직의 유형에 따라 맞춤형 전문
보험 정책들을 제공할 수 있습니다.

-   주거용 건물을 방문할 때는 의료 보험을 판매합니다.
-   은행을 방문할 때는 도난 보험을 판매합니다.
-   커피숍을 방문할 때는 화재 및 홍수 보험을 판매합니다.
:::

::::: {.section .structure-container}
##  구조 {#structure}

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/visitor/structure-koea36.png?id=8445ad6187fdd0457c929bb4040458af"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 0.96;"
srcset="/images/patterns/diagrams/visitor/structure-ko.png?id=8445ad6187fdd0457c929bb4040458af 520w,/images/patterns/diagrams/visitor/structure-ko-1.5x.png?id=43fc6b6eb77a470f018541cb777335ea 780w,/images/patterns/diagrams/visitor/structure-ko-2x.png?id=df91d9e12406f606c918bb89f4bdf8ee 1040w"
sizes="(max-width: 720px) 100vw, 520px" loading="lazy" width="520"
alt="비지터 디자인 패턴의 구조" /><img
src="../../images/patterns/diagrams/visitor/structure-ko-indexedfdda.png?id=3ca199bed081a7d750a1e164551ece3f"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 0.93;"
srcset="/images/patterns/diagrams/visitor/structure-ko-indexed.png?id=3ca199bed081a7d750a1e164551ece3f 520w,/images/patterns/diagrams/visitor/structure-ko-indexed-1.5x.png?id=50a8838bcde44f9ad5be5257fad3070b 780w,/images/patterns/diagrams/visitor/structure-ko-indexed-2x.png?id=5ab3fec06b7f6865278f6a7f1ee9e6f4 1040w"
sizes="(max-width: 720px) 100vw, 520px" loading="lazy" width="520"
alt="비지터 디자인 패턴의 구조" />
</figure>
:::

1.  **비지터** 인터페이스는 객체 구조의 구상 요소들을 인수들로 사용할 수
    있는 비지터 메서드들의 집합을 선언합니다. 이러한 메서드들은 (앱이
    오버로딩을 지원하는 언어로 작성된 경우) 같은 이름을 가질 수 있지만
    그들의 매개변수들의 유형은 달라야 합니다.

2.  각 **구상 비지터**는 다양한 구상 요소 클래스들에 맞춤으로 작성된
    같은 행동들의 여러 버전을 구현합니다.

3.  **요소** 인터페이스는 비지터를 \'수락\'하는 메서드를 선언합니다. 이
    메서드에는 비지터 인터페이스 유형으로 선언된 하나의 매개변수가
    있어야 합니다.

4.  각 **구상 요소**는 반드시 수락 메서드를 구현해야 합니다. 이 메서드의
    목적은 호출을 현재 요소 클래스에 해당하는 적절한 비지터 메서드로
    리다이렉트하는 것입니다. 기초 요소 클래스가 이 메서드를 구현하더라도
    모든 자식 클래스들은 여전히 자신들의 클래스들 내에서 이 메서드를
    오버라이드해야 하며 비지터 객체에 적절한 메서드를 호출해야 합니다.

5.  **클라이언트**는 일반적으로 컬렉션 또는 기타 복잡한 객체​(예:
    [복합체](composite.html) 트리)​를 나타냅니다. 일반적으로
    클라이언트들은 해당 컬렉션의 객체들과 어떠한 추상 인터페이스를 통해
    작업하기 때문에 모든 구상 요소 클래스들을 인식하지 못합니다.
::::
:::::

::: {.section .pseudocode}
##  의사코드 {#pseudocode}

이 예시에서의 **비지터** 패턴은 기하학적 모양들의 클래스 계층구조에 XML
내보내기 지원을 추가합니다.

<figure class="image">
<img
src="../../images/patterns/diagrams/visitor/example9405.png?id=d66acd1b9096c47db17ab3bb82b54a59"
style="aspect-ratio: 1.14;"
srcset="/images/patterns/diagrams/visitor/example.png?id=d66acd1b9096c47db17ab3bb82b54a59 560w,/images/patterns/diagrams/visitor/example-1.5x.png?id=534425a20f1128cb81f418cbee30cba8 840w,/images/patterns/diagrams/visitor/example-2x.png?id=f44438b98f13fcb50898baefad67ffff 1120w"
sizes="(max-width: 720px) 100vw, 560px" loading="lazy" width="560"
alt="비지터 패턴 구조 예시" />
<figcaption><p>비지터 객체를 통해 다양한 유형의 객체들을 XML
형식으로 내보내기.</p></figcaption>
</figure>

<figure class="code">
<pre class="code" lang="pseudocode"><code>// 요소 인터페이스는 기초 방문자 인터페이스를 인수로 받는 `accept` 메서드를
// 선언합니다.
interface Shape is
    method move(x, y)
    method draw()
    method accept(v: Visitor)

// 각 구상 요소 클래스는 요소의 클래스에 해당하는 비지터의 메서드를 호출하는
// 방식으로 `accept` 메서드를 구현해야 합니다.
class Dot implements Shape is
    // …

    // 참고로 우리는 현재 클래스 이름과 일치하는 `visitDot`를 호출하고
    // 있습니다. 그래야 비지터가 함께 작업하는 요소의 클래스를 알 수 있습니다.
    method accept(v: Visitor) is
        v.visitDot(this)

class Circle implements Shape is
    // …
    method accept(v: Visitor) is
        v.visitCircle(this)

class Rectangle implements Shape is
    // …
    method accept(v: Visitor) is
        v.visitRectangle(this)

class CompoundShape implements Shape is
    // …
    method accept(v: Visitor) is
        v.visitCompoundShape(this)


// 비지터 인터페이스는 요소 클래스들에 해당하는 방문 메서드들의 집합을 선언합니다.
// 방문 메서드의 시그니처를 통해 비지터는 처리 중인 요소의 정확한 클래스를 식별할
// 수 있습니다.
interface Visitor is
    method visitDot(d: Dot)
    method visitCircle(c: Circle)
    method visitRectangle(r: Rectangle)
    method visitCompoundShape(cs: CompoundShape)

// 구상 비지터는 모든 구상 요소 클래스와 작동할 수 있는 같은 알고리즘의 여러 버전을
// 구현합니다.
//
// 비지터 패턴은 복합체 트리와 같은 복잡한 객체 구조와 함께 사용할 때 가장 큰
// 이득을 볼 수 있습니다. 그러면 비지터의 메서드들을 구조의 다양한 객체 위에서
// 실행하는 동안 알고리즘의 어떤 중간 상태를 저장하는 것이 도움이 될 수 있습니다.
class XMLExportVisitor implements Visitor is
    method visitDot(d: Dot) is
        // 점의 아이디와 중심 좌표를 내보냅니다.

    method visitCircle(c: Circle) is
        // 원의 아이디, 중심 좌표 및 반지름을 내보냅니다.

    method visitRectangle(r: Rectangle) is
        // 사각형의 아이디, 왼쪽 상단 좌표, 너비 및 높이를 내보냅니다.

    method visitCompoundShape(cs: CompoundShape) is
        // 모양의 아이디와 그 자식들의 아이디 리스트를 내보냅니다.


// 클라이언트 코드는 요소의 구상 클래스들을 파악하지 않고도 모든 요소 집합 위에서
// 비지터의 작업들을 실행할 수 있습니다. `accept` 작업은 비지터 객체의 적절한
// 작업으로 호출을 전달합니다.
class Application is
    field allShapes: array of Shapes

    method export() is
        exportVisitor = new XMLExportVisitor()

        foreach (shape in allShapes) do
            shape.accept(exportVisitor)</code></pre>
</figure>

이 예시에서 `accept` 메서드가 필요한 이유가 궁금하다면 제 설명글
[Visitor and Double Dispatch](visitor-double-dispatch.html)에서 이
주제를 자세히 다루고 있습니다.
:::

:::::::::: {.section .applicability-container}
##  적용 {#applicability}

::::::::: applicability
::: applicability-problem
비지터 객체는 복잡한 객체 구조​(예: 객체 트리)​의 모든 요소에 대해 작업을
수행해야 할 때 사용하세요.
:::

::: applicability-solution
비지터 패턴은 비지터 객체가 모든 대상 클래스들에 해당하는 같은 작업의
여러 변형들을 구현하도록 함으로써 다양한 클래스들을 가진 여러 객체의
집합에 작업을 실행할 수 있도록 해줍니다.
:::

::: applicability-problem
비지터 패턴을 사용하여 보조 행동들의 비즈니스 로직을 정리하세요.
:::

::: applicability-solution
이 패턴은 앱의 주 클래스들의 주 작업들을 제외한 모든 다른 행동들을
비지터 클래스들의 집합으로 추출함으로써 그들이 주 작업에 더 집중하도록
만들 수 있게 해줍니다.
:::

::: applicability-problem
이 패턴은 행동이 클래스 계층구조의 일부 클래스들에서만 의미가 있고 다른
클래스들에서는 의미가 없을 때 사용하세요.
:::

::: applicability-solution
이 행동을 별도의 비지터 클래스로 추출한 후 관련 클래스들의 객체들을
수락하는 비지터 메서드들만 구현하고 나머지는 비워둡니다.
:::
:::::::::
::::::::::

::: {.section .checklist}
##  구현방법 {#checklist}

1.  프로그램에 존재하는 각 구상 요소 클래스당 하나씩 \'비지터​(방문)\'
    메서드를 만들고 이 메서드들의 집합으로 비지터 인터페이스를
    선언하세요.

2.  요소 인터페이스를 선언하세요. 기존 요소 클래스 계층구조와 작업하는
    경우 계층구조의 기초 클래스에 추상 수락 메서드를 추가하세요. 이
    메서드는 비지터 객체를 인수로 받아들여야 합니다.

3.  모든 구상 요소 클래스들에서 수락 메서드들을 구현하세요. 이러한
    메서드들은 단순히 비지터 메서드에 대한 호출을 들어오는 비지터 객체에
    리다이렉트해야 합니다. 이 들어오는 비지터 객체는 현재 요소의
    클래스와 일치합니다.

4.  요소 클래스들은 비지터 인터페이스를 통해서만 비지터와 작동해야
    합니다. 그러나 비지터들은 비지터 메서드들의 매개변수 유형들로 참조된
    모든 구상 요소 클래스들에 대해 알고 있어야 합니다.

5.  요소 계층구조 내에서 구현할 수 없는 각 행동의 경우, 새로운 구상
    비지터 클래스를 만들고 모든 비지터 메서드들을 구현하세요.

    비지터가 요소 클래스의 일부 비공개 필드들 또는 메서드들에 접근해야
    할 상황이 발생할 수 있습니다. 이럴 때 이러한 필드들 또는 메서드들을
    공개하여 요소의 캡슐화를 위반하거나, 비지터 클래스를 요소 클래스에
    중첩할 수 있습니다. 중첩 옵션의 경우 중첩 클래스들을 지원하는
    프로그래밍 언어를 사용할 때만 가능합니다.

6.  클라이언트는 비지터 객체들을 만들고 \'수락\' 메서드들을 통해
    그것들을 요소들에 전달해야 합니다.
:::

:::::: {.section .pros-cons}
##  장단점 {#pros-cons}

::::: row
::: col-sm-6
-    *[개]{.cjk}[방]{.cjk}/[폐]{.cjk}[쇄]{.cjk} [원]{.cjk}[칙]{.cjk}*.
    당신은 다른 클래스를 변경하지 않으면서 해당 클래스의 객체와 작동할
    수 있는 새로운 행동을 도입할 수 있습니다.
-    *[단]{.cjk}[일]{.cjk} [책]{.cjk}[임]{.cjk} [원]{.cjk}[칙]{.cjk}*.
    같은 행동의 여러 버전을 같은 클래스로 이동할 수 있습니다.
-    비지터 객체는 다양한 객체들과 작업하면서 유용한 정보를 축적할 수
    있습니다. 이것은 객체 트리와 같은 복잡한 객체 구조를 순회하여 이
    구조의 각 객체에 비지터 패턴을 적용하려는 경우에 유용할 수 있습니다.
:::

::: col-sm-6
-    당신은 클래스가 요소 계층구조에 추가되거나 제거될 때마다 모든
    비지터를 업데이트해야 합니다.
-    비지터들은 함께 작업해야 하는 요소들의 비공개 필드들 및 메서드들에
    접근하기 위해 필요한 권한이 부족할 수 있습니다.
:::
:::::
::::::

::: {.section .relations}
##  다른 패턴과의 관계 {#relations}

-   [비지터](visitor.html) 패턴은 [커맨드](command.html) 패턴의 강력한
    버전으로 취급할 수 있습니다. 비지터 패턴의 객체들은 다른 클래스들의
    다양한 객체에 대한 작업을 실행할 수 있습니다.

-   당신은 [비지터](visitor.html) 패턴을 사용하여
    [복합체](composite.html) 패턴 트리 전체를 대상으로 작업을 수행할 수
    있습니다.

-   [비지터](visitor.html) 패턴과 [반복자](iterator.html) 패턴을 함께
    사용해 복잡한 데이터 구조를 순회하여 해당 구조의 요소들의 클래스들이
    모두 다르더라도 이러한 요소들에 대해 어떤 작업을 실행할 수 있습니다.
:::

::: {.section .implementations}
##  코드 예시 {#implementations}

[![C#으로 작성된
비지터](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](visitor/csharp/example.html "C#으로 작성된 비지터"){.prog-lang-link}
[![C++로 작성된
비지터](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](visitor/cpp/example.html "C++로 작성된 비지터"){.prog-lang-link}
[![Go로 작성된
비지터](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](visitor/go/example.html "Go로 작성된 비지터"){.prog-lang-link}
[![자바로 작성된
비지터](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](visitor/java/example.html "자바로 작성된 비지터"){.prog-lang-link}
[![PHP로 작성된
비지터](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](visitor/php/example.html "PHP로 작성된 비지터"){.prog-lang-link}
[![파이썬으로 작성된
비지터](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](visitor/python/example.html "파이썬으로 작성된 비지터"){.prog-lang-link}
[![루비로 작성된
비지터](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](visitor/ruby/example.html "루비로 작성된 비지터"){.prog-lang-link}
[![러스트로 작성된
비지터](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](visitor/rust/example.html "러스트로 작성된 비지터"){.prog-lang-link}
[![스위프트로 작성된
비지터](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](visitor/swift/example.html "스위프트로 작성된 비지터"){.prog-lang-link}
[![타입스크립트로 작성된
비지터](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](visitor/typescript/example.html "타입스크립트로 작성된 비지터"){.prog-lang-link}
:::

::: {.section .extras}
##  추가 콘텐츠 {#extras}

-   비지터 패턴을 메서드 오버로딩으로 대체할 수 없는 이유가
    궁금하십니까? 제 설명글 [Visitor and Double
    Dispatch](visitor-double-dispatch.html)에서 이 문제의 불쾌한 세부
    사항들에 대해 더 자세히 다루었습니다.
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

[비지터 패턴과 이중 디스패치 []{.fa
.fa-arrow-right}](visitor-double-dispatch.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### 돌아가기

[[]{.fa .fa-arrow-left} 템플릿 메서드](template-method.html){.btn
.btn-default rel="prev"}
:::
:::::::::::::::::::::::::::::::::::

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
:::::::::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::::::::
