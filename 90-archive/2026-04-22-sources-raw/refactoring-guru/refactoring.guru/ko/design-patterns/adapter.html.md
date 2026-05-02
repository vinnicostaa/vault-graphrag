:::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [디자인
패턴들](../design-patterns.html){.type} / [구조
패턴](structural-patterns.html){.type}
:::

# 어댑터 패턴 {#어댑터-패턴 .title}

::: aka
다음 이름으로도 불립니다:
[래퍼(Wrapper), ]{style="display:inline-block"}[Adapter]{style="display:inline-block"}
:::

::: {.section .intent}
##  의도 {#intent}

**어댑터**는 호환되지 않는 인터페이스를 가진 객체들이 협업할 수 있도록
하는 구조적 디자인 패턴입니다.

<figure class="image">
<img
src="../../images/patterns/content/adapter/adapter-ko8f09.png?id=4166b1ca5201a660d9ce29ef96a50621"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/adapter/adapter-ko.png?id=4166b1ca5201a660d9ce29ef96a50621 640w,/images/patterns/content/adapter/adapter-ko-1.5x.png?id=39b9b3caf482187ff91bf4e2522eedbc 960w,/images/patterns/content/adapter/adapter-ko-2x.png?id=29fa17e0475c999fc047ed82ed00f655 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="어댑터 디자인 패턴" />
</figure>
:::

::: {.section .problem}
##  문제 {#problem}

주식 시장 모니터링 앱을 만들고 있고, 이 앱은 여러 소스에서 주식 데이터를
XML 형식으로 다운로드한 후 사용자에게 보기 좋은 차트들과 다이어그램들을
표시한다고 상상해 봅시다.

어느 시점에 당신은 타사의 스마트 분석 라이브러리를 통합하여 당신의 앱을
개선하기로 결정했습니다. 그런데 함정이 있습니다: 이 분석 라이브러리는
JSON 형식의 데이터로만 작동한다는 것입니다.

<figure class="image">
<img
src="../../images/patterns/diagrams/adapter/problem-kofe66.png?id=33ecccae252bded5eb70e070ddf28633"
style="aspect-ratio: 2.41;"
srcset="/images/patterns/diagrams/adapter/problem-ko.png?id=33ecccae252bded5eb70e070ddf28633 530w,/images/patterns/diagrams/adapter/problem-ko-1.5x.png?id=8afb05f7980259429d4da2556c10e056 795w,/images/patterns/diagrams/adapter/problem-ko-2x.png?id=88b919e969a568d68fa19cf18c82ab24 1060w"
sizes="(max-width: 720px) 100vw, 530px" loading="lazy" width="530"
alt="스마트 분석 라이브러리와 통합하기 전 앱의 구조" />
<figcaption><p>위 분석 라이브러리는 '있는 그대로' 사용할 수 없습니다.
왜냐하면 당신의 앱과 호환되지 않는 형식의 데이터를 기다리고
있기 때문입니다.</p></figcaption>
</figure>

당신은 이 라이브러리를 XML과 작동하도록 변경할 수 있으나, 그러면
라이브러리에 의존하는 일부 기존 코드가 손상될 수 있습니다. 또 처음부터
타사의 라이브러리 소스 코드에 접근하는 것이 불가능하여 위의 해결 방식을
사용하지 못할 수도 있습니다.
:::

::: {.section .solution}
##  해결책 {#solution}

당신은 *[어]{.cjk}[댑]{.cjk}[터]{.cjk}*를 만들 수 있습니다. 어댑터는 한
객체의 인터페이스를 다른 객체가 이해할 수 있도록 변환하는 특별한
객체입니다.

어댑터는 변환의 복잡성을 숨기기 위하여 객체 중 하나를 래핑​(포장)​합니다.
래핑된 객체는 어댑터를 인식하지도 못합니다. 예를 들어 미터 및 킬로미터
단위로 작동하는 객체를 모든 데이터를 피트 및 마일과 같은 영국식 단위로
변환하는 어댑터로 래핑할 수 있습니다.

어댑터는 데이터를 다양한 형식으로 변환할 수 있을 뿐만 아니라 다른
인터페이스를 가진 객체들이 협업하는 데에도 도움을 줄 수 있으며, 대략
다음과 같이 작동합니다:

1.  어댑터는 기존에 있던 객체 중 하나와 호환되는 인터페이스를 받습니다.
2.  이 인터페이스를 사용하면 기존 객체는 어댑터의 메서드들을 안전하게
    호출할 수 있습니다.
3.  호출을 수신하면 어댑터는 이 요청을 두 번째 객체에 해당 객체가
    예상하는 형식과 순서대로 전달합니다.

때로는 양방향으로 호출을 변환할 수 있는 양방향 어댑터를 만드는 것도
가능합니다.

<figure class="image">
<img
src="../../images/patterns/diagrams/adapter/solution-ko749e.png?id=73504c03a6e85f8b6182ad1701232d16"
style="aspect-ratio: 1.51;"
srcset="/images/patterns/diagrams/adapter/solution-ko.png?id=73504c03a6e85f8b6182ad1701232d16 530w,/images/patterns/diagrams/adapter/solution-ko-1.5x.png?id=d130018e5c4c599b44fe4ca87c033f76 795w,/images/patterns/diagrams/adapter/solution-ko-2x.png?id=c421fe77927474734b25b3a3713e2963 1060w"
sizes="(max-width: 720px) 100vw, 530px" loading="lazy" width="530"
alt="어댑터의 해결책" />
</figure>

다시 당신의 주식 시장 앱을 살펴봅시다. 당신은 형식이 호환되지 않는
문제를 해결하기 위해 당신의 코드와 직접 작동하는 분석 라이브러리의 모든
클래스에 대한 XML-\>JSON 변환 어댑터를 만듭니다. 그 후 이러한 어댑터들을
통해서만 해당 라이브러리와 통신하도록 코드를 조정합니다. 어댑터는 호출을
받으면 들어오는 XML 데이터를 JSON 구조로 변환한 후 해당 호출을 래핑된
분석 객체의 적절한 메서드들에 전달합니다.
:::

::: {.section .analogy}
##  실제상황 적용 {#analogy}

<figure class="image">
<img
src="../../images/patterns/content/adapter/adapter-comic-1-ko3f87.png?id=e840abb1b88879b7e023c77f12316310"
style="aspect-ratio: 1.78;"
srcset="/images/patterns/content/adapter/adapter-comic-1-ko.png?id=e840abb1b88879b7e023c77f12316310 533w,/images/patterns/content/adapter/adapter-comic-1-ko-1.5x.png?id=5785575c826ff54cc7cf9c077faf3011 800w,/images/patterns/content/adapter/adapter-comic-1-ko-2x.png?id=43504156d482348f047932a912a600c6 1067w"
sizes="(max-width: 720px) 100vw, 533px" loading="lazy" width="533"
alt="어댑터 패턴 예시" />
<figcaption><p>해외여행 전과 후의 서류가방.</p></figcaption>
</figure>

미국에서 유럽으로 처음 여행을 가서 노트북을 충전하려고 하면 깜짝
놀랄지도 모릅니다. 전원 플러그와 소켓은 국가마다 표준이 달라 미국
플러그가 독일 소켓에 맞지 않을 수 있기 때문입니다. 이 문제는 미국식
소켓과 유럽식 플러그가 있는 전원 플러그 어댑터를 사용하면 해결할 수
있습니다.
:::

:::::::: {.section .structure-container}
##  구조 {#structure}

::::::: structure
#### 객체 어댑터

이 구현은 객체 합성 원칙을 사용합니다. 어댑터는 한 객체의 인터페이스를
구현하고 다른 객체는 래핑합니다. 위 합성은 모든 인기 있는 프로그래밍
언어로 구현할 수 있습니다.

:::: adapter-object
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/adapter/structure-object-adaptera83b.png?id=33dffbe3aece294162440c7ddd3d5d4f"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.81;"
srcset="/images/patterns/diagrams/adapter/structure-object-adapter.png?id=33dffbe3aece294162440c7ddd3d5d4f 580w,/images/patterns/diagrams/adapter/structure-object-adapter-1.5x.png?id=c1b8a87b74fc4ce5639212fe19ee06fe 870w,/images/patterns/diagrams/adapter/structure-object-adapter-2x.png?id=03e8052e168c962d6bc369bbb13b0945 1160w"
sizes="(max-width: 720px) 100vw, 580px" loading="lazy" width="580"
alt="어댑터 디자인 패턴의 구조 (객체 어댑터)" /><img
src="../../images/patterns/diagrams/adapter/structure-object-adapter-indexedd6ac.png?id=a20b311948b361a058097e5bcdbf067a"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.76;"
srcset="/images/patterns/diagrams/adapter/structure-object-adapter-indexed.png?id=a20b311948b361a058097e5bcdbf067a 600w,/images/patterns/diagrams/adapter/structure-object-adapter-indexed-1.5x.png?id=72a1c8fde064c4915379fb34a1a66881 900w,/images/patterns/diagrams/adapter/structure-object-adapter-indexed-2x.png?id=759771515f08d74d53cf4fe500f814a3 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="어댑터 디자인 패턴의 구조 (객체 어댑터)" />
</figure>
:::
::::

1.  **클라이언트**는 프로그램의 기존 비즈니스 로직을 포함하는
    클래스입니다.

2.  **클라이언트 인터페이스**는 다른 클래스들이 클라이언트 코드와 공동
    작업할 수 있도록 따라야 하는 프로토콜을 뜻합니다.

3.  **서비스**는 일반적으로 타사 또는 레거시의 유용한 클래스를 뜻합니다.
    클라이언트는 서비스 클래스를 직접 사용할 수 없습니다. 왜냐하면
    서비스 클래스는 호환되지 않는 인터페이스를 가지고 있기 때문입니다.

4.  **어댑터**는 클라이언트와 서비스 양쪽에서 작동할 수 있는 클래스로,
    서비스 객체를 래핑하는 동안 클라이언트 인터페이스를 구현합니다.
    어댑터는 어댑터 인터페이스를 통해 클라이언트로부터 호출들을 수신한
    후 이 호출을 래핑된 서비스 객체가 이해할 수 있는 형식의 호출들로
    변환합니다.

5.  클라이언트 코드는 클라이언트 인터페이스를 통해 어댑터와 작동하는 한
    구상 어댑터 클래스와 결합하지 않습니다. 덕분에 기존 클라이언트
    코드를 손상하지 않고 새로운 유형의 어댑터들을 프로그램에 도입할 수
    있습니다. 이것은 서비스 클래스의 인터페이스가 변경되거나 교체될 때
    유용할 수 있습니다: 클라이언트 코드를 변경하지 않은 채 새 어댑터
    클래스를 생성할 수 있으니까요.

#### 클래스 어댑터

이 구현은 상속을 사용하며, 어댑터는 동시에 두 객체의 인터페이스를
상속합니다. 이 방식은 C++ 와 같이 다중 상속을 지원하는 프로그래밍
언어에서만 구현할 수 있습니다.

:::: adapter-class
::: {.struct-image2 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/adapter/structure-class-adapterf043.png?id=e1c60240508146ed3b98ac562cc8e510"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.72;"
srcset="/images/patterns/diagrams/adapter/structure-class-adapter.png?id=e1c60240508146ed3b98ac562cc8e510 550w,/images/patterns/diagrams/adapter/structure-class-adapter-1.5x.png?id=299a79bdfa10ac53213ba02408255430 825w,/images/patterns/diagrams/adapter/structure-class-adapter-2x.png?id=ddca3e3e4d972b7c58207daba8d24866 1100w"
sizes="(max-width: 720px) 100vw, 550px" loading="lazy" width="550"
alt="어댑터 디자인 패턴 (클래스 어댑터)" /><img
src="../../images/patterns/diagrams/adapter/structure-class-adapter-indexedd30e.png?id=250b5c485a7dfba7c16b89a9201538fb"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.72;"
srcset="/images/patterns/diagrams/adapter/structure-class-adapter-indexed.png?id=250b5c485a7dfba7c16b89a9201538fb 550w,/images/patterns/diagrams/adapter/structure-class-adapter-indexed-1.5x.png?id=b56d736f8076d34b1896de0a2b22a219 825w,/images/patterns/diagrams/adapter/structure-class-adapter-indexed-2x.png?id=9ae1182ef2a34d2ea65f4b4f94a4019e 1100w"
sizes="(max-width: 720px) 100vw, 550px" loading="lazy" width="550"
alt="어댑터 디자인 패턴 (클래스 어댑터)" />
</figure>
:::
::::

1.  **클래스 어댑터**는 객체를 래핑할 필요가 없습니다. 그 이유는
    클라이언트와 서비스 양쪽에서 행동들을 상속받기 때문입니다. 위의
    어댑테이션​(적용)​은 오버라이딩된 메서드 내에서 발생합니다. 위
    어댑터는 기존 클라이언트 클래스 대신 사용할 수 있습니다.
:::::::
::::::::

::: {.section .pseudocode}
##  의사코드 {#pseudocode}

이 **어댑터** 패턴은 서로 맞지 않는 정사각형 못과 둥근 구멍이라는
고전적인 예시를 기초로 합니다.

<figure class="image">
<img
src="../../images/patterns/diagrams/adapter/examplef4c9.png?id=9d2b6857ce256f2c669383ce4df3d0aa"
style="aspect-ratio: 1.68;"
srcset="/images/patterns/diagrams/adapter/example.png?id=9d2b6857ce256f2c669383ce4df3d0aa 640w,/images/patterns/diagrams/adapter/example-1.5x.png?id=76e68b9cea3b371e465e81587e57e5d9 960w,/images/patterns/diagrams/adapter/example-2x.png?id=0ac62d1bc151e8ce6abad8e8502756cf 1280w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="640"
alt="어댑터 패턴 예시 구조" />
<figcaption><p>둥근 구멍에 정사각형 못을 맞춰 넣기.</p></figcaption>
</figure>

어댑터는 정사각형 지름의 절반​(즉, 사각형 못을 수용할 수 있는 가장 작은
원의 반지름)​을 반지름으로 가진 둥근 못인 척 합니다.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// RoundHole(둥근 구멍) 및 RoundPeg(둥근 못)​라는 호환되는 인터페이스들이 있는
// 두 개의 클래스가 있다고 가정해봅시다.
class RoundHole is
    constructor RoundHole(radius) { ... }

    method getRadius() is
        // 구멍의 반지름을 반환하세요.

    method fits(peg: RoundPeg) is
        return this.getRadius() &gt;= peg.getRadius()

class RoundPeg is
    constructor RoundPeg(radius) { ... }

    method getRadius() is
        // 못의 반지름을 반환하세요.


// 그러나 SquarePeg(직사각형 못)​라는 호환되지 않는 클래스가 있습니다.
class SquarePeg is
    constructor SquarePeg(width) { ... }

    method getWidth() is
        // 직사각형 못의 너비를 반환하세요.


// 어댑터 클래스를 사용하면 정사각형 못을 둥근 구멍에 맞출 수 있습니다. 어댑터
// 객체들은 RoundPeg(둥근 못) 클래스를 확장해 둥근 못들처럼 작동하게 해줍니다.
class SquarePegAdapter extends RoundPeg is
    // 실제로 어댑터에는 SquarePeg(정사각형 못) 클래스의 인스턴스가 포함되어
    // 있습니다.
    private field peg: SquarePeg

    constructor SquarePegAdapter(peg: SquarePeg) is
        this.peg = peg

    method getRadius() is
        // 어댑터는 이것이 어댑터가 실제로 감싸는 정사각형 못에 맞는 반지름을
        // 가진 원형 못인 것처럼 가장합니다.
        return peg.getWidth() * Math.sqrt(2) / 2


// 클라이언트 코드 어딘가에…
hole = new RoundHole(5)
rpeg = new RoundPeg(5)
hole.fits(rpeg) // 참

small_sqpeg = new SquarePeg(5)
large_sqpeg = new SquarePeg(10)
hole.fits(small_sqpeg) // 이것은 컴파일되지 않습니다​(호환되지 않는 유형)

small_sqpeg_adapter = new SquarePegAdapter(small_sqpeg)
large_sqpeg_adapter = new SquarePegAdapter(large_sqpeg)
hole.fits(small_sqpeg_adapter) // 참
hole.fits(large_sqpeg_adapter) // 거짓</code></pre>
</figure>
:::

:::::::: {.section .applicability-container}
##  적용 {#applicability}

::::::: applicability
::: applicability-problem
어댑터 클래스는 기존 클래스를 사용하고 싶지만 그 인터페이스가 나머지
코드와 호환되지 않을 때 사용하세요.
:::

::: applicability-solution
어댑터 패턴은 당신의 코드와 레거시 클래스, 타사 클래스 또는 특이한
인터페이스가 있는 다른 클래스 간의 변환기 역할을 하는 중간 레이어
클래스를 만들 수 있도록 합니다.
:::

::: applicability-problem
이 패턴은 부모 클래스에 추가할 수 없는 어떤 공통 기능들이 없는 여러 기존
자식 클래스들을 재사용하려는 경우에 사용하세요.
:::

::: applicability-solution
각 자식 클래스를 확장한 후 누락된 기능들을 새 자식 클래스들에 넣을 수
있습니다. 하지만 해당 코드를 모든 새 클래스들에 복제해야 하며, 그건 정말
[나쁜 냄새가 나는 코드](../smells/duplicate-code.html)일 것입니다.

이보다 훨씬 더 깔끔한 해결책은 누락된 기능을 어댑터 클래스에 넣는
것입니다. 그 후 어댑터 내부에 누락된 기능이 있는 객체들을 래핑하면
필요한 기능들을 동적으로 얻을 것입니다. 이 해결책이 작동하려면 대상
클래스들에는 반드시 공통 인터페이스가 있어야 하며 어댑터의 필드는 해당
인터페이스를 따라야 합니다. 위 접근 방식은 [데코레이터](decorator.html)
패턴과 매우 유사합니다.
:::
:::::::
::::::::

::: {.section .checklist}
##  구현방법 {#checklist}

1.  호환되지 않는 인터페이스가 있는 클래스가 최소 두 개 이상 있는지
    확인하세요:

    -   당신이 변경할 수 없는 유용한 *[서]{.cjk}[비]{.cjk}[스]{.cjk}*
        클래스가 있습니다. (종종 타사 코드, 레거시 코드 또는 기존
        의존성이 많은 코드).
    -   위 서비스 클래스를 사용하여 이득을 얻을 수 있는 하나 또는 여러
        개의 *[클]{.cjk}[라]{.cjk}[이]{.cjk}[언]{.cjk}[트]{.cjk}*
        클래스들이 있습니다.

2.  클라이언트 인터페이스를 선언하고 클라이언트들이 서비스와 통신하는
    방법을 기술하세요.

3.  어댑터 클래스를 생성한 후 클라이언트 인터페이스를 따르게 하세요.
    일단은 모든 메서드들을 비워 두세요.

4.  서비스 객체에 참조를 저장하기 위하여 어댑터 클래스에 필드를
    추가하세요. 일반적으로 사용되는 방법은 생성자를 통해 이 필드를
    초기화하는 것이지만, 때때로 어댑터의 메서드들을 호출할 때는 이
    필드를 어댑터에 전달하는 것이 더 편리하기도 합니다.

5.  클라이언트 인터페이스의 모든 메서드를 어댑터 클래스에서 하나씩
    구현하세요. 어댑터는 인터페이스 또는 데이터 형식 변환만 처리해야
    하며, 실제 작업의 대부분을 서비스 객체에 위임해야 합니다.

6.  클라이언트들은 클라이언트 인터페이스를 통해 어댑터를 사용해야
    합니다. 이렇게 하면 클라이언트 코드에 영향을 주지 않고 어댑터들을
    변경하거나 확장할 수 있습니다.
:::

:::::: {.section .pros-cons}
##  장단점 {#pros-cons}

::::: row
::: col-sm-6
-    *[단]{.cjk}[일]{.cjk} [책]{.cjk}[임]{.cjk} [원]{.cjk}[칙]{.cjk}*.
    프로그램의 기본 비즈니스 로직에서 인터페이스 또는 데이터 변환 코드를
    분리할 수 있습니다.
-    *[개]{.cjk}[방]{.cjk}/[폐]{.cjk}[쇄]{.cjk} [원]{.cjk}[칙]{.cjk}*.
    클라이언트 코드가 클라이언트 인터페이스를 통해 어댑터와 작동하는 한,
    기존의 클라이언트 코드를 손상시키지 않고 새로운 유형의 어댑터들을
    프로그램에 도입할 수 있습니다.
:::

::: col-sm-6
-    다수의 새로운 인터페이스와 클래스들을 도입해야 하므로 코드의
    전반적인 복잡성이 증가합니다. 때로는 코드의 나머지 부분과 작동하도록
    서비스 클래스를 변경하는 것이 더 간단합니다.
:::
:::::
::::::

::: {.section .relations}
##  다른 패턴과의 관계 {#relations}

-   [브리지](bridge.html)는 일반적으로 사전에 설계되며, 앱의 다양한
    부분을 독립적으로 개발할 수 있도록 합니다. 반면에
    [어댑터](adapter.html)는 일반적으로 기존 앱과 사용되어 원래 호환되지
    않던 일부 클래스들이 서로 잘 작동하도록 합니다.

-   [어댑터](adapter.html)는 기존 객체의 인터페이스를 변경하는 반면
    [데코레이터](decorator.html)는 객체를 해당 객체의 인터페이스를
    변경하지 않고 향상합니다. 또한
    *[데]{.cjk}[코]{.cjk}[레]{.cjk}[이]{.cjk}[터]{.cjk}*는
    *[어]{.cjk}[댑]{.cjk}[터]{.cjk}*를 사용할 때는 불가능한 재귀적
    합성을 지원합니다.

-   [어댑터](adapter.html)는 다른 인터페이스를, [프록시](proxy.html)는
    같은 인터페이스를, [데코레이터](decorator.html)는 향상된
    인터페이스를 래핑된 객체에 제공합니다.

-   [퍼사드](facade.html)는 기존 객체들을 위한 새 인터페이스를 정의하는
    반면 [어댑터](adapter.html)는 기존의 인터페이스를 사용할 수 있게
    만들려고 노력합니다. 또 *[어]{.cjk}[댑]{.cjk}[터]{.cjk}*는
    일반적으로 하나의 객체만 래핑하는 반면
    *[퍼]{.cjk}[사]{.cjk}[드]{.cjk}*는 많은 객체의 하위시스템과 함께
    작동합니다.

-   [브리지](bridge.html), [상태](state.html), [전략](strategy.html)
    패턴은 매우 유사한 구조로 되어 있으며, [어댑터](adapter.html) 패턴도
    이들과 어느 정도 유사한 구조로 되어 있습니다. 위 모든 패턴은 다른
    객체에 작업을 위임하는 합성을 기반으로 합니다. 하지만 이 패턴들은
    모두 다른 문제들을 해결합니다. 패턴은 특정 방식으로 코드의 구조를
    짜는 레시피에 불과하지 않습니다. 왜냐하면 패턴은 해결하는 문제를
    다른 개발자들에게 전달할 수도 있기 때문입니다.
:::

::: {.section .implementations}
##  코드 예시 {#implementations}

[![C#으로 작성된
어댑터](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](adapter/csharp/example.html "C#으로 작성된 어댑터"){.prog-lang-link}
[![C++로 작성된
어댑터](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](adapter/cpp/example.html "C++로 작성된 어댑터"){.prog-lang-link}
[![Go로 작성된
어댑터](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](adapter/go/example.html "Go로 작성된 어댑터"){.prog-lang-link}
[![자바로 작성된
어댑터](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](adapter/java/example.html "자바로 작성된 어댑터"){.prog-lang-link}
[![PHP로 작성된
어댑터](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](adapter/php/example.html "PHP로 작성된 어댑터"){.prog-lang-link}
[![파이썬으로 작성된
어댑터](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](adapter/python/example.html "파이썬으로 작성된 어댑터"){.prog-lang-link}
[![루비로 작성된
어댑터](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](adapter/ruby/example.html "루비로 작성된 어댑터"){.prog-lang-link}
[![러스트로 작성된
어댑터](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](adapter/rust/example.html "러스트로 작성된 어댑터"){.prog-lang-link}
[![스위프트로 작성된
어댑터](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](adapter/swift/example.html "스위프트로 작성된 어댑터"){.prog-lang-link}
[![타입스크립트로 작성된
어댑터](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](adapter/typescript/example.html "타입스크립트로 작성된 어댑터"){.prog-lang-link}
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

[브리지 []{.fa .fa-arrow-right}](bridge.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### 돌아가기

[[]{.fa .fa-arrow-left} 구조 패턴](structural-patterns.html){.btn
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
