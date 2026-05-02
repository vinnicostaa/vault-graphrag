::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [디자인
패턴들](../design-patterns.html){.type} / [구조
패턴](structural-patterns.html){.type}
:::

# 데코레이터 패턴 {#데코레이터-패턴 .title}

::: aka
다음 이름으로도 불립니다:
[래퍼(Wrapper), ]{style="display:inline-block"}[Decorator]{style="display:inline-block"}
:::

::: {.section .intent}
##  의도 {#intent}

**데코레이터**는 객체들을 새로운 행동들을 포함한 특수 래퍼 객체들 내에
넣어서 위 행동들을 해당 객체들에 연결시키는 구조적 디자인 패턴입니다.

<figure class="image">
<img
src="../../images/patterns/content/decorator/decoratore8f5.png?id=710c66670c7123e0928d3b3758aea79e"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/decorator/decorator.png?id=710c66670c7123e0928d3b3758aea79e 640w,/images/patterns/content/decorator/decorator-1.5x.png?id=72aafc603a289fc64e028e83e8aa843b 960w,/images/patterns/content/decorator/decorator-2x.png?id=736ab07b1d8920ab2c7a70c9cb1305cc 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="데코레이터 디자인 패턴" />
</figure>
:::

::: {.section .problem}
##  문제 {#problem}

당신이 알림 라이브러리를 만들고 있다고 상상해 보세요. 이 알림
라이브러리의 목적은 다른 프로그램들이 사용자들에게 중요한 이벤트들에
대해 알릴 수 있도록 하는 것입니다.

이 라이브러리의 초기 버전은 `Notifier`​(알림자) 클래스를 기반으로 했으며,
이 클래스에는 몇 개의 필드들, 하나의 생성자 그리고 단일 `send`​(전송)
메서드만 있었습니다. 이 메서드는 클라이언트로부터 메시지 인수를 받은 후
그 메세지를 알림자의 생성자를 통해 알림자에게 전달된 이메일 목록으로
보낼 수 있습니다. 또 클라이언트 역할을 한 타사 앱은 알림자 객체를 한번
생성하고 설정한 후 중요한 일이 발생할 때마다 사용하게 되어 있었습니다.

<figure class="image">
<img
src="../../images/patterns/diagrams/decorator/problem1-koc510.png?id=078087cf3f8ef8037771f840272eac66"
style="aspect-ratio: 2.45;"
srcset="/images/patterns/diagrams/decorator/problem1-ko.png?id=078087cf3f8ef8037771f840272eac66 540w,/images/patterns/diagrams/decorator/problem1-ko-1.5x.png?id=2f14f0082b74bdc4d465fd905c32f4ba 810w,/images/patterns/diagrams/decorator/problem1-ko-2x.png?id=5af0ef6856abde6b3f6cb8eebf285ba0 1080w"
sizes="(max-width: 720px) 100vw, 540px" loading="lazy" width="540"
alt="데코레이터 패턴을 적용하기 전의 라이브러리 구조" />
<figcaption><p>프로그램은 알림자 클래스를 사용하여 미리 정의된
이메일들의 집합에 중요한 이벤트에 대한 알림을 보낼
수 있습니다.</p></figcaption>
</figure>

당신은 어느 시점에서 라이브러리 사용자들이 이메일 알림 이상을 기대한다는
것을 알게 됩니다. 그들 중 많은 사용자는 중요한 사안에 대한 SMS 문자
메시지를 받고 싶어 했고, 다른 사용자들은 페이스북 알림을 받고 싶어
했으며 기업 사용자들은 슬랙 알림을 받고 싶어 했습니다.

<figure class="image">
<img
src="../../images/patterns/diagrams/decorator/problem2a2f8.png?id=ba5d5e106ea8c4848d60e230feca9135"
style="aspect-ratio: 2.59;"
srcset="/images/patterns/diagrams/decorator/problem2.png?id=ba5d5e106ea8c4848d60e230feca9135 440w,/images/patterns/diagrams/decorator/problem2-1.5x.png?id=4f15b6208533188dd09e7da7dd6b509a 660w,/images/patterns/diagrams/decorator/problem2-2x.png?id=28b2c8509b4e78c031d728424b876ebc 880w"
sizes="(max-width: 720px) 100vw, 440px" loading="lazy" width="440"
alt="다른 알림 유형을 구현한 후의 라이브러리 구조" />
<figcaption><p>각 알림 유형은 알림자의 자식
클래스로 구현됩니다.</p></figcaption>
</figure>

이는 표면상 별로 어렵지 않아 보입니다. 당신은 `Notifier`​(알림자)
클래스를 확장하고 추가 알림 메서드들을 새 자식 클래스들에 넣어 이제
클라이언트가 사용자가 원하는 알림 클래스를 인스턴스화하고 모든 추가
알림에 사용하도록 앱을 설계했습니다.

그런데 누군가 당신에게 물었습니다. \'여러 유형의 알림을 한 번에 사용할
수는 없나요? 집에 불이라도 난다면 사용자들은 모든 채널에서 정보를 받고
싶어 할 겁니다.\'

이 문제를 해결하기 위해 당신은 하나의 클래스 내에서 여러 알림 메서드를
합성한 특수 자식 클래스들을 만들었으나, 이 접근 방식은 라이브러리
코드뿐만 아니라 클라이언트 코드도 엄청나게 부풀릴 것이라는 사실이 금세
명백해졌습니다.

<figure class="image">
<img
src="../../images/patterns/diagrams/decorator/problem34033.png?id=f3b3e7a107d870871f2c3167adcb7ccb"
style="aspect-ratio: 1.85;"
srcset="/images/patterns/diagrams/decorator/problem3.png?id=f3b3e7a107d870871f2c3167adcb7ccb 630w,/images/patterns/diagrams/decorator/problem3-1.5x.png?id=f4ef9d367df838c77121e9f84260b09c 945w,/images/patterns/diagrams/decorator/problem3-2x.png?id=abb7a87b521ce97d7661dd9c0b988cc3 1260w"
sizes="(max-width: 720px) 100vw, 630px" loading="lazy" width="630"
alt="클래스 조합들을 생성한 후의 라이브러리 구조" />
<figcaption><p>자식 클래스들의 합성으로 인한 클래스
수의 폭발</p></figcaption>
</figure>

이제 당신은 알림 클래스들의 수가 지나치게 많아지지 않도록 알림
클래스들을 구성하는 다른 방법을 찾아야 합니다.
:::

::: {.section .solution}
##  해결책 {#solution}

객체의 동작을 변경해야 할 때 가장 먼저 고려되는 방법은 클래스의
확장입니다. 그러나 상속에는 당신이 심각하게 주의해야 할 몇 가지 사항들이
있습니다.

-   상속은 정적입니다: 당신은 런타임​(실행시간) 때 기존 객체의 행동을
    변경할 수 없습니다. 당신은 전체 객체를 다른 자식 클래스에서 생성된
    다른 객체로만 바꿀 수 있습니다.
-   자식 클래스는 하나의 부모 클래스만 가질 수 있습니다. 대부분
    언어에서의 상속은 클래스가 동시에 여러 클래스의 행동을 상속하도록
    허용하지 않습니다.

이러한 주의 사항을 극복하는 방법의 하나는 *[상]{.cjk}[속]{.cjk}* 대신
*[집]{.cjk}[합]{.cjk} [관]{.cjk}[계]{.cjk}* 또는
*[합]{.cjk}[성]{.cjk}* []{.pop-i}[*[집]{.cjk}[합]{.cjk}
[관]{.cjk}[계]{.cjk}*: 객체 A는 객체 B를 포함합니다; B는 A 없이 생존할
수 있습니다.\
*[합]{.cjk}[성]{.cjk}*: 객체 A는 객체 B로 구성됩니다; A는 B의 수명
주기를 관리합니다; B는 A 없이 생존할 수 없습니다.]{.pop-content}을
사용하는 것입니다. 두 대안 모두 거의 같은 방식으로 작동합니다: 집합
관계에서는 한 객체가 다른 객체에 대한 *[참]{.cjk}[조]{.cjk}[를]{.cjk}
[갖]{.cjk}[고]{.cjk}* 일부 작업을 위임하는 반면, 상속을 사용하면 객체
자체가 부모 클래스에서 행동을 상속한 후 해당 작업을
*[수]{.cjk}[행]{.cjk}[할]{.cjk} [수]{.cjk}
[있]{.cjk}[습]{.cjk}[니]{.cjk}[다]{.cjk}*.

이 새로운 접근 방식을 사용하면 연결된 \'도우미\' 객체를 다른 객체로 쉽게
대체하여 런타임 때 컨테이너의 행동을 변경할 수 있습니다. 객체는 여러
클래스의 행동들을 사용할 수 있고, 여러 객체에 대한 참조들이 있으며 이
객체들에 모든 종류의 작업을 위임합니다. 집합 관계/합성은 데코레이터를
포함한 많은 디자인 패턴의 핵심 원칙입니다. 그러면 이제 다시 이 패턴에
대하여 살펴봅시다.

<figure class="image">
<img
src="../../images/patterns/diagrams/decorator/solution1-ko74e2.png?id=00e8b2f10ccf896309f4da842a9aa346"
style="aspect-ratio: 3.44;"
srcset="/images/patterns/diagrams/decorator/solution1-ko.png?id=00e8b2f10ccf896309f4da842a9aa346 550w,/images/patterns/diagrams/decorator/solution1-ko-1.5x.png?id=f1477c77158810bdd277e450551a08f0 825w,/images/patterns/diagrams/decorator/solution1-ko-2x.png?id=c4769f73f2084b0a58eea0d8d3c09ccd 1100w"
sizes="(max-width: 720px) 100vw, 550px" loading="lazy" width="550"
alt="상속과 집합 관계의 차이점." />
<figcaption><p>상속과 집합 관계의 차이점.</p></figcaption>
</figure>

\'래퍼\'는 패턴의 주요 아이디어를 명확하게 표현하는 데코레이터 패턴의
별명입니다. *[래]{.cjk}[퍼]{.cjk}*는 일부 *[대]{.cjk}[상]{.cjk}* 객체와
연결할 수 있는 객체입니다. 래퍼에는 대상 객체와 같은 메서드들의 집합이
포함되어 있으며, 래퍼는 자신이 받는 모든 요청을 대상 객체에 위임합니다.
그러나 래퍼는 이 요청을 대상에 전달하기 전이나 후에 무언가를 수행하여
결과를 변경할 수 있습니다.

그러면 간단한 래퍼는 언제 진정한 데코레이터가 될 수 있을까요? 앞서
언급했듯이 래퍼는 래핑된 객체와 같은 인터페이스를 구현합니다. 그러므로
클라이언트의 관점에서 이러한 객체들은 같습니다. 이제 래퍼의 참조 필드가
해당 인터페이스를 따르는 모든 객체를 받도록 하세요. 이렇게 하면 여러
래퍼로 객체를 포장해서 모든 래퍼들의 합성된 행동들을 객체에 추가할 수
있습니다.

이제 당신의 알림 라이브러리에서 기초 `Notifier` 클래스 내에 있는 간단한
이메일 알림 행동은 그대로 두고 다른 모든 알림 메서드들을 데코레이터로
바꾸어 봅시다.

<figure class="image">
<img
src="../../images/patterns/diagrams/decorator/solution2e4ab.png?id=cbee4a27080ce3a0bf773482613e1347"
style="aspect-ratio: 1.39;"
srcset="/images/patterns/diagrams/decorator/solution2.png?id=cbee4a27080ce3a0bf773482613e1347 640w,/images/patterns/diagrams/decorator/solution2-1.5x.png?id=d2fa0c0b9bf5cba0e38be7aff7e7b7a8 960w,/images/patterns/diagrams/decorator/solution2-2x.png?id=7775f76b94dbd5cd25f711ce81f59262 1280w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="640"
alt="데코레이터 패턴을 사용한 해결책" />
<figcaption><p>다양한 알림 메서드들이
데코레이터가 됩니다.</p></figcaption>
</figure>

클라이언트 코드는 기초 알림자 객체를 클라이언트의 요구사항들과 일치하는
데코레이터들의 집합으로 래핑해야 합니다. 위 결과 객체들은 스택으로
구성됩니다.

<figure class="image">
<img
src="../../images/patterns/diagrams/decorator/solution3-ko2972.png?id=7d788c466c110e59952178d9f0c33f1a"
style="aspect-ratio: 1.03;"
srcset="/images/patterns/diagrams/decorator/solution3-ko.png?id=7d788c466c110e59952178d9f0c33f1a 300w,/images/patterns/diagrams/decorator/solution3-ko-1.5x.png?id=ea46a19308414719d17603f82e7f6b8e 450w,/images/patterns/diagrams/decorator/solution3-ko-2x.png?id=09657c63fd5ed0ddafd11b3fa234439a 600w"
sizes="(max-width: 720px) 100vw, 300px" loading="lazy" width="300"
alt="앱들은 알림 데코레이터들의 복잡한 스택들을 설정할 수 있습니다" />
<figcaption><p>앱들은 알림 데코레이터들의 복잡한 스택들을 설정할
수 있습니다.</p></figcaption>
</figure>

스택의 마지막 데코레이터는 실제로 클라이언트와 작업하는 객체입니다. 모든
데코레이터들은 기초 알림자와 같은 인터페이스를 구현하므로 나머지
클라이언트 코드는 자신이 \'순수한\' 알림자 객체와 작동하든 데코레이터로
장식된 알림자 객체와 함께 작동하든 상관하지 않습니다.

메시지 형식 지정 또는 수신자 리스트 작성과 같은 다른 행동들에도 같은
접근 방식을 적용할 수 있습니다. 클라이언트는 객체가 다른 객체들과 같은
인터페이스를 따르는 한 객체를 모든 사용자 지정 데코레이터로 장식할 수
있습니다.
:::

::: {.section .analogy}
##  실제상황 적용 {#analogy}

<figure class="image">
<img
src="../../images/patterns/content/decorator/decorator-comic-1cbff.png?id=80d95baacbfb91f5bcdbdc7814b0c64d"
style="aspect-ratio: 2;"
srcset="/images/patterns/content/decorator/decorator-comic-1.png?id=80d95baacbfb91f5bcdbdc7814b0c64d 600w,/images/patterns/content/decorator/decorator-comic-1-1.5x.png?id=490ef9754d7554c2046b69f6f213c6da 900w,/images/patterns/content/decorator/decorator-comic-1-2x.png?id=ba869f621b6e0ea173fdc2b535fc7eed 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="데코레이터 패턴 예시" />
<figcaption><p>여러 벌의 옷을 입으면 복합 효과를 얻을
수 있습니다.</p></figcaption>
</figure>

옷을 입는 것은 데코레이터 패턴을 사용하는 예입니다. 당신은 추울 때
스웨터로 몸을 감쌉니다. 스웨터를 입어도 춥다면 위에 재킷을 입고, 또 비가
오면 비옷을 입습니다. 이 모든 옷은 기초 행동을 \'확장\'하지만, 당신의
일부가 아니기에 필요하지 않을 때마다 옷을 쉽게 벗을 수 있습니다.
:::

::::: {.section .structure-container}
##  구조 {#structure}

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/decorator/structure128f.png?id=8c95d894aecce5315cc1b12093a7ea0c"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 0.92;"
srcset="/images/patterns/diagrams/decorator/structure.png?id=8c95d894aecce5315cc1b12093a7ea0c 480w,/images/patterns/diagrams/decorator/structure-1.5x.png?id=4b2cd91d4483d9e3bba725f0e45d281d 720w,/images/patterns/diagrams/decorator/structure-2x.png?id=3cfa1f10417a4ef0c12580bc4a63b80d 960w"
sizes="(max-width: 720px) 100vw, 480px" loading="lazy" width="480"
alt="데코레이터 디자인 패턴 구조" /><img
src="../../images/patterns/diagrams/decorator/structure-indexeda93c.png?id=09401b230a58f2249e4c9a1195d485a0"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 0.98;"
srcset="/images/patterns/diagrams/decorator/structure-indexed.png?id=09401b230a58f2249e4c9a1195d485a0 500w,/images/patterns/diagrams/decorator/structure-indexed-1.5x.png?id=dccc54182965078ccb4cfdeee41acbe5 750w,/images/patterns/diagrams/decorator/structure-indexed-2x.png?id=2733e7d0e322bfb2f150ccf8a878dbf6 1000w"
sizes="(max-width: 720px) 100vw, 500px" loading="lazy" width="500"
alt="데코레이터 디자인 패턴 구조" />
</figure>
:::

1.  **컴포넌트**는 래퍼들과 래핑된 객체들 모두에 대한 공통 인터페이스를
    선언합니다.

2.  **구상 컴포넌트**는 래핑되는 객체들의 클래스이며, 그는 기본 행동들을
    정의하고 해당 기본 행동들은 데코레이터들이 변경할 수 있습니다.

3.  **기초 데코레이터** 클래스에는 래핑된 객체를 참조하기 위한 필드가
    있습니다. 필드의 유형은 구상 컴포넌트들과 구상 데코레이터들을 모두
    포함할 수 있도록 컴포넌트 인터페이스로 선언되어야 합니다. 그 후 기초
    데코레이터는 모든 작업들을 래핑된 객체에 위임합니다.

4.  **구상 데코레이터들**은 컴포넌트들에 동적으로 추가될 수 있는 추가
    행동들을 정의합니다. 그들은 기초 데코레이터의 메서드를
    오버라이드​(재정의)​하고 해당 행동을 부모 메서드를 호출하기 전이나
    후에 실행합니다.

5.  **클라이언트**는 아래에 언급한 데코레이터들이 컴포넌트 인터페이스를
    통해 모든 객체와 작동하는 한 컴포넌트들을 여러 계층의 데코레이터들로
    래핑할 수 있습니다.
::::
:::::

::: {.section .pseudocode}
##  의사코드 {#pseudocode}

이 예시에서 **데코레이터** 패턴은 민감한 데이터를 해당 데이터를 실제로
사용하는 코드와 별도로 압축하고 암호화할 수 있도록 합니다.

<figure class="image">
<img
src="../../images/patterns/diagrams/decorator/example42cb.png?id=eec9dc488f00c85f50e764323baa723e"
style="aspect-ratio: 0.98;"
srcset="/images/patterns/diagrams/decorator/example.png?id=eec9dc488f00c85f50e764323baa723e 540w,/images/patterns/diagrams/decorator/example-1.5x.png?id=40ccaba4f5a8e6a2ebac697f04b9f10a 810w,/images/patterns/diagrams/decorator/example-2x.png?id=4891323a27d5601a174eec366187d833 1080w"
sizes="(max-width: 720px) 100vw, 540px" loading="lazy" width="540"
alt="데코레이터 패턴 구조 예시" />
<figcaption><p>암호화 및 압축화 데코레이터들의 예.</p></figcaption>
</figure>

이 애플리케이션은 데이터 소스 객체를 한 쌍의 데코레이터로 래핑합니다. 두
래퍼 모두 디스크에 데이터를 쓰고 읽는 방식들을 변경합니다.

-   데이터가 **디스크에 기록**되기 직전에 데코레이터들은 데이터를
    암호화하고 압축합니다. 원래 클래스는 위 변경 사항에 대해 알지 못한
    채 암호화되고 보호된 데이터를 파일에 씁니다.

-   데이터는 **디스크에서 읽힌 직후** 같은 데코레이터들을 거쳐 가며, 이
    데코레이터들은 데이터의 압축을 풀고 디코딩합니다.

데코레이터와 데이터 소스 클래스는 같은 인터페이스를 구현하므로
클라이언트 코드에서 모두 상호 교환이 가능합니다.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// 컴포넌트 인터페이스는 데코레이터들이 변경할 수 있는 작업들을 정의합니다.
interface DataSource is
    method writeData(data)
    method readData():data

// 구상 컴포넌트들은 작업들에 대한 디폴트 구현들을 제공합니다. 프로그램에는 이러한
// 클래스들의 여러 변형이 있을 수 있습니다.
class FileDataSource implements DataSource is
    constructor FileDataSource(filename) { ... }

    method writeData(data) is
        // 파일에 데이터를 씁니다.

    method readData():data is
        // 파일에서 데이터를 읽으세요.

// 기초 데코레이터 클래스는 다른 컴포넌트들과 같은 인터페이스를 따릅니다. 이
// 클래스의 주목적은 모든 구상 데코레이터에 대한 래핑 인터페이스를 정의하는
// 것입니다. 래핑 코드의 디폴트 구현에는 래핑된 컴포넌트를 저장하기 위한 필드와
// 이를 초기화하는 수단들이 포함될 수 있습니다.
class DataSourceDecorator implements DataSource is
    protected field wrappee: DataSource

    constructor DataSourceDecorator(source: DataSource) is
        wrappee = source

    // 기초 데코레이터는 단순히 모든 작업을 래핑된 컴포넌트에 위임합니다. 구상
    // 데코레이터들에는 추가 행동들이 추가될 수 있습니다.
    method writeData(data) is
        wrappee.writeData(data)

    // 구상 데코레이터들은 래핑된 객체를 직접 호출하는 대신 부모의 작업 구현을
    // 호출할 수 있습니다. 이 접근 방식은 데코레이터 클래스들의 확장을
    // 단순화합니다.
    method readData():data is
        return wrappee.readData()

// 구상 데코레이터는 래핑된 객체에 메서드를 호출해야 하지만 결과에 자기 것을 뭔가를
// 추가할 수 있습니다. 데코레이터들은 래핑된 객체에 대한 호출 전 또는 후에 추가된
// 행동을 실행할 수 있습니다.
class EncryptionDecorator extends DataSourceDecorator is
    method writeData(data) is
        // 1. 전달된 데이터를 암호화합니다.
        // 2. 암호화된 데이터를 wrapee(래핑된)​의 writeData(데이터 쓰기)
        // 메서드에 전달합니다.

    method readData():data is
        // 1. wrapee(래핑된)​의 readData 메서드에서 데이터를 가져옵니다.
        // 2. 암호화되어 있다면 암호 해독을 시도하세요.
        // 3. 결과를 반환하세요.

// 여러 계층의 데코레이터들로 객체들을 래핑할 수 있습니다.
class CompressionDecorator extends DataSourceDecorator is
    method writeData(data) is
        // 1. 전달된 데이터를 압축하세요.
        // 2. 압축된 데이터를 wrapee(래핑된)​의 writeData 메서드에
        // 전달하세요.

    method readData():data is
        // 1. wrappee(래핑된)​의 readData(데이터 읽기) 메서드에서 데이터를
        // 가져오세요.
        // 2. 압축되어 있으면 압축을 풀어보세요.
        // 3. 결과를 반환하세요.


// 옵션 1. 데코레이터 조립의 간단한 예시.
class Application is
    method dumbUsageExample() is
        source = new FileDataSource(&quot;somefile.dat&quot;)
        source.writeData(salaryRecords)
        // 대상 파일은 일반 데이터로 작성되었습니다.

        source = new CompressionDecorator(source)
        source.writeData(salaryRecords)
        // 대상 파일은 압축된 데이터로 작성되었습니다.

        source = new EncryptionDecorator(source)
        // 이제 소스 변수에 다음이 포함됩니다. 암호화 &gt; 압축 &gt; 파일 데이터 소스
        source.writeData(salaryRecords)
        // 파일은 압축 및 암호화된 데이터로 작성되었습니다.


// 옵션 2. 외부 데이터 소스를 사용하는 클라이언트 코드. SalaryManager 객체들은
// 데이터 저장 세부 사항들을 알지도 못하고 신경 쓰지도 않습니다. 이들은 앱 설정
// 프로그램에서 받은 사전 구성된 데이터 소스로 작업합니다.
class SalaryManager is
    field source: DataSource

    constructor SalaryManager(source: DataSource) { ... }

    method load() is
        return source.readData()

    method save() is
        source.writeData(salaryRecords)
    // …다른 유용한 메서드들…


// 앱은 설정 또는 환경에 따라 런타임 때 다양한 데코레이터 스택들을 조합할 수
// 있습니다.
class ApplicationConfigurator is
    method configurationExample() is
        source = new FileDataSource(&quot;salary.dat&quot;)
        if (enabledEncryption)
            source = new EncryptionDecorator(source)
        if (enabledCompression)
            source = new CompressionDecorator(source)

        logger = new SalaryManager(source)
        salary = logger.load()
    // …</code></pre>
</figure>
:::

:::::::: {.section .applicability-container}
##  적용 {#applicability}

::::::: applicability
::: applicability-problem
데코레이터 패턴은 이 객체들을 사용하는 코드를 훼손하지 않으면서 런타임에
추가 행동들을 객체들에 할당할 수 있어야 할 때 사용하세요.
:::

::: applicability-solution
데코레이터는 비즈니스 로직을 계층으로 구성하고, 각 계층에 데코레이터를
생성하고 런타임에 이 로직의 다양한 조합들로 객체들을 구성할 수 있도록
합니다. 이러한 모든 객체가 공통 인터페이스를 따르기 때문에 클라이언트
코드는 해당 모든 객체를 같은 방식으로 다룰 수 있습니다.
:::

::: applicability-problem
이 패턴은 상속을 사용하여 객체의 행동을 확장하는 것이 어색하거나
불가능할 때 사용하세요.
:::

::: applicability-solution
많은 프로그래밍 언어에는 클래스의 추가 확장을 방지하는 데 사용할 수 있는
`final` 키워드가 있습니다. Final 클래스의 경우 기존 행동들을 재사용할 수
있는 유일한 방법은 데코레이터 패턴을 사용하여 클래스를 자체 래퍼로
래핑하는 것입니다.
:::
:::::::
::::::::

::: {.section .checklist}
##  구현방법 {#checklist}

1.  당신의 비즈니스 도메인이 여러 선택적 계층으로 감싸진 기본 컴포넌트로
    표시될 수 있는지 확인하세요.

2.  기본 컴포넌트와 선택적 계층들 양쪽에 공통적인 메서드들이 무엇인지
    파악하세요. 그곳에 컴포넌트 인터페이스를 만들고 해당 메서드들을
    선언하세요.

3.  구상 컴포넌트 클래스를 만든 후 그 안에 기초 행동들을 정의하세요.

4.  기초 데코레이터 클래스를 만드세요. 이 클래스에는 래핑된 객체에 대한
    참조를 저장하기 위한 필드가 있어야 합니다. 이 필드는 데코레이터들 및
    구상 컴포넌트들과의 연결을 허용하기 위하여 컴포넌트 인터페이스
    유형으로 선언하셔야 합니다. 기초 데코레이터는 모든 작업을 래핑된
    객체에 위임해야 합니다.

5.  모든 클래스들이 컴포넌트 인터페이스를 구현하도록 하세요.

6.  기초 데코레이터를 확장하여 구상 데코레이터들을 생성하세요. 구상
    데코레이터는 항상 부모 메서드 호출 전 또는 후에 행동들을 실행해야
    합니다. (부모 메서드는 항상 래핑된 객체에 작업을 위임합니다).

7.  데코레이터들을 만들고 이러한 데코레이터들을 클라이언트가 필요로 하는
    방식으로 구성하는 일은 반드시 클라이언트 코드가 맡아야 합니다.
:::

:::::: {.section .pros-cons}
##  장단점 {#pros-cons}

::::: row
::: col-sm-6
-    새 자식 클래스를 만들지 않고도 객체의 행동을 확장할 수 있습니다.
-    런타임에 객체들에서부터 책임들을 추가하거나 제거할 수 있습니다.
-    객체를 여러 데코레이터로 래핑하여 여러 행동들을 합성할 수 있습니다.
-    *[단]{.cjk}[일]{.cjk} [책]{.cjk}[임]{.cjk} [원]{.cjk}[칙]{.cjk}*.
    다양한 행동들의 여러 변형들을 구현하는 모놀리식 클래스를 여러 개의
    작은 클래스들로 나눌 수 있습니다.
:::

::: col-sm-6
-    래퍼들의 스택에서 특정 래퍼를 제거하기가 어렵습니다.
-    데코레이터의 행동이 데코레이터 스택 내의 순서에 의존하지 않는
    방식으로 데코레이터를 구현하기가 어렵습니다.
-    계층들의 초기 설정 코드가 보기 흉할 수 있습니다.
:::
:::::
::::::

::: {.section .relations}
##  다른 패턴과의 관계 {#relations}

-   [어댑터](adapter.html)는 기존 객체의 인터페이스를 변경하는 반면
    [데코레이터](decorator.html)는 객체를 해당 객체의 인터페이스를
    변경하지 않고 향상합니다. 또한
    *[데]{.cjk}[코]{.cjk}[레]{.cjk}[이]{.cjk}[터]{.cjk}*는
    *[어]{.cjk}[댑]{.cjk}[터]{.cjk}*를 사용할 때는 불가능한 재귀적
    합성을 지원합니다.

-   [어댑터](adapter.html)는 다른 인터페이스를, [프록시](proxy.html)는
    같은 인터페이스를, [데코레이터](decorator.html)는 향상된
    인터페이스를 래핑된 객체에 제공합니다.

-   [책임 연쇄](chain-of-responsibility.html) 패턴과
    [데코레이터](decorator.html)는 클래스 구조가 매우 유사합니다. 두
    패턴 모두 실행을 일련의 객체들을 통해 전달할 때 재귀적인 합성에
    의존하나, 몇 가지 결정적인 차이점이 있습니다.

    *[책]{.cjk}[임]{.cjk} [연]{.cjk}[쇄]{.cjk}* 패턴 핸들러들은 서로
    독립적으로 임의의 작업을 실행할 수 있으며, 또한 해당 요청을 언제든지
    더 이상 전달하지 않을 수 있습니다. 반면에 다양한
    *[데]{.cjk}[코]{.cjk}[레]{.cjk}[이]{.cjk}[터]{.cjk}[들]{.cjk}*은
    객체의 행동을 확장하며 동시에 이러한 행동을 기초 인터페이스와
    일관되게 유지할 수 있습니다. 또한 데코레이터들은 요청의 흐름을
    중단할 수 없습니다.

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

-   [데코레이터](decorator.html)는 객체의 피부를 변경할 수 있고
    [전략](strategy.html) 패턴은 객체의 내장을 변경할 수 있다고 비유할
    수 있습니다.

-   [데코레이터](decorator.html)와 [프록시](proxy.html)의 구조는
    비슷하나 이들의 의도는 매우 다릅니다. 두 패턴 모두 한 객체가 일부
    작업을 다른 객체에 위임해야 하는 합성 원칙을 기반으로 합니다. 이 두
    패턴의 차이점은 *[프]{.cjk}[록]{.cjk}[시]{.cjk}*는 일반적으로
    자체적으로 자신의 서비스 객체의 수명 주기를 관리하는 반면
    *[데]{.cjk}[코]{.cjk}[레]{.cjk}[이]{.cjk}[터]{.cjk}*의 합성은 항상
    클라이언트에 의해 제어된다는 점입니다.
:::

::: {.section .implementations}
##  코드 예시 {#implementations}

[![C#으로 작성된
데코레이터](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](decorator/csharp/example.html "C#으로 작성된 데코레이터"){.prog-lang-link}
[![C++로 작성된
데코레이터](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](decorator/cpp/example.html "C++로 작성된 데코레이터"){.prog-lang-link}
[![Go로 작성된
데코레이터](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](decorator/go/example.html "Go로 작성된 데코레이터"){.prog-lang-link}
[![자바로 작성된
데코레이터](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](decorator/java/example.html "자바로 작성된 데코레이터"){.prog-lang-link}
[![PHP로 작성된
데코레이터](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](decorator/php/example.html "PHP로 작성된 데코레이터"){.prog-lang-link}
[![파이썬으로 작성된
데코레이터](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](decorator/python/example.html "파이썬으로 작성된 데코레이터"){.prog-lang-link}
[![루비로 작성된
데코레이터](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](decorator/ruby/example.html "루비로 작성된 데코레이터"){.prog-lang-link}
[![러스트로 작성된
데코레이터](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](decorator/rust/example.html "러스트로 작성된 데코레이터"){.prog-lang-link}
[![스위프트로 작성된
데코레이터](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](decorator/swift/example.html "스위프트로 작성된 데코레이터"){.prog-lang-link}
[![타입스크립트로 작성된
데코레이터](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](decorator/typescript/example.html "타입스크립트로 작성된 데코레이터"){.prog-lang-link}
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

[퍼사드 []{.fa .fa-arrow-right}](facade.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### 돌아가기

[[]{.fa .fa-arrow-left} 복합체](composite.html){.btn .btn-default
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
