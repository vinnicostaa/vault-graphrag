::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [디자인
패턴들](../design-patterns.html){.type} / [행동
패턴](behavioral-patterns.html){.type}
:::

# 책임 연쇄 패턴 {#책임-연쇄-패턴 .title}

::: aka
다음 이름으로도 불립니다: [CoR, ]{style="display:inline-block"}[커맨드
사슬, ]{style="display:inline-block"}[Chain of
Responsibility]{style="display:inline-block"}
:::

::: {.section .intent}
##  의도 {#intent}

**책임 연쇄** 패턴은 핸들러들의 체인​(사슬)​을 따라 요청을 전달할 수 있게
해주는 행동 디자인 패턴입니다. 각 핸들러는 요청을 받으면 요청을 처리할지
아니면 체인의 다음 핸들러로 전달할지를 결정합니다.

<figure class="image">
<img
src="../../images/patterns/content/chain-of-responsibility/chain-of-responsibilityf2d1.png?id=56c10d0dc712546cc283cfb3fb463458"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/chain-of-responsibility/chain-of-responsibility.png?id=56c10d0dc712546cc283cfb3fb463458 640w,/images/patterns/content/chain-of-responsibility/chain-of-responsibility-1.5x.png?id=770c03ad168fa797ec8ed4556ddf0b5c 960w,/images/patterns/content/chain-of-responsibility/chain-of-responsibility-2x.png?id=cc104b0a00a410f37fb39da80f392b88 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="책임 연쇄 패턴" />
</figure>
:::

::: {.section .problem}
##  문제 {#problem}

당신이 온라인 주문 시스템을 개발하고 있다고 가정해봅시다. 당신은 인증된
사용자들만 주문을 생성할 수 있도록 시스템에 대한 접근을 제한하려고
합니다. 또 관리 권한이 있는 사용자들에게는 모든 주문에 대한 전체 접근
권한을 부여하려고 합니다.

당신은 약간의 설계 후에 이러한 검사들은 차례대로 수행해야 한다는 사실을
깨달았습니다. 당신의 앱은 사용자들의 자격 증명이 포함된 요청을 받을
때마다 시스템에 대해 사용자 인증을 시도할 수 있습니다. 그러나 이러한
자격 증명이 올바르지 않아서 인증에 실패하면 다른 검사들을 진행할 이유가
없습니다.

<figure class="image">
<img
src="../../images/patterns/diagrams/chain-of-responsibility/problem1-ko2c4b.png?id=8344cbf5f2cda8d3cf2ffb29fcafdcf7"
style="aspect-ratio: 2.5;"
srcset="/images/patterns/diagrams/chain-of-responsibility/problem1-ko.png?id=8344cbf5f2cda8d3cf2ffb29fcafdcf7 600w,/images/patterns/diagrams/chain-of-responsibility/problem1-ko-1.5x.png?id=d517ccd2bf67870c4e8ab0946e8ec4b9 900w,/images/patterns/diagrams/chain-of-responsibility/problem1-ko-2x.png?id=3c121f18651118d1f87703b80b7a6717 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="문제, 책임 연쇄 패턴에 의해 해결되다" />
<figcaption><p>요청은 주문 시스템 자체가 처리할 수 있기 전에 일련의
검사들을 통과해야 합니다.</p></figcaption>
</figure>

다음 몇 달 동안 당신은 이러한 순차 검사들을 몇 가지 더 구현했습니다.

-   동료 중 한 명이 검증되지 않은 데이터를 주문 시스템에 직접 전달하는
    것은 안전하지 않다고 제안했습니다. 그래서 당신은 요청 내의 데이터를
    정제​(sanitize)​하는 추가 유효성 검사 단계를 추가했습니다.

-   나중에 누군가가 시스템이 무차별 대입 공격에 취약하다는 사실을
    발견했으며, 이러한 공격을 방어하기 위해 같은 IP 주소에서 오는
    반복적으로 실패한 요청을 걸러내는 검사를 즉시 추가했습니다.

-   또 다른 누군가는 같은 데이터가 포함된 반복 요청에 대해 캐시된 결과를
    반환하여 시스템 속도를 높일 수 있다고 제안했고, 당신은 적절한 캐시
    응답이 없는 경우에만 요청이 시스템으로 전달되도록 하는 또 다른
    검사를 추가했습니다.

<figure class="image">
<img
src="../../images/patterns/diagrams/chain-of-responsibility/problem2-ko9376.png?id=fbf849f75f6e06cc39dcbdcb26b369b6"
style="aspect-ratio: 1.65;"
srcset="/images/patterns/diagrams/chain-of-responsibility/problem2-ko.png?id=fbf849f75f6e06cc39dcbdcb26b369b6 610w,/images/patterns/diagrams/chain-of-responsibility/problem2-ko-1.5x.png?id=79d983f4c8c83ef30534cfff7e731e93 915w,/images/patterns/diagrams/chain-of-responsibility/problem2-ko-2x.png?id=1c8aeab6ceee85b6bb4d10a9470febf8 1220w"
sizes="(max-width: 720px) 100vw, 610px" loading="lazy" width="610"
alt="새로운 검사를 추가할 때마다 코드가 더 커지고 더 복잡해지고 더 추악해졌습니다" />
<figcaption><p>코드가 커질수록 더 복잡해졌습니다.</p></figcaption>
</figure>

이미 엉망진창이었던 검사 코드는 당신이 새로운 기능을 추가할 때마다 더욱
크게 부풀어 올랐습니다. 하나의 검사 코드를 바꾸면 다른 검사 코드가
영향을 받기도 했습니다. 더 심각한 문제는, 시스템의 다른 컴포넌트들을
보호하기 위해 검사를 재사용하려고 할 때 해당 컴포넌트들에 일부 코드를
복제해야 했다는 것입니다. 왜냐하면 컴포넌트들이 필요로 한 것은 검사의
일부였지, 모든 검사는 아니었기 때문입니다.

당신의 시스템은 이해하기가 매우 어려웠고 유지 관리 비용이 많이 들었으며,
당신은 프로그램 전체를 리팩토링하기로 할 때까지 한동안 코드와
씨름했습니다.
:::

::: {.section .solution}
##  해결책 {#solution}

다른 여러 행동 디자인 패턴들과 마찬가지로 **책임 연쇄** 패턴은 특정
행동들을 *[핸]{.cjk}[들]{.cjk}[러]{.cjk}*라는 독립 실행형 객체들로
변환합니다. 당신의 앱의 경우 각 검사는 검사를 수행하는 단일 메서드가
있는 자체 클래스로 추출되어야 합니다. 이제 요청은 데이터와 함께 이
메서드에 인수로 전달됩니다.

이 패턴은 이러한 핸들러들을 체인으로 연결하도록 제안합니다. 연결된 각
핸들러에는 체인의 다음 핸들러에 대한 참조를 저장하기 위한 필드가
있습니다. 요청을 처리하는 것 외에도 핸들러들은 체인을 따라 요청을 더
멀리 전달하며, 이 요청은 모든 핸들러가 요청을 처리할 기회를 가질 때까지
체인을 따라 이동합니다.

가장 좋은 부분은 핸들러가 요청을 체인 아래로 더 이상 전달하지 않고 추가
처리를 사실상 중지하는 결정을 내릴 수 있다는 것입니다.

당신의 주문 관리 시스템에서는 하나의 핸들러가 주문 처리를 수행한 다음
요청을 체인 아래로 더 전달할지를 결정합니다. 요청에 올바른 데이터가
포함되어 있다고 가정하면 모든 핸들러들은 인증 확인이든 캐싱이든 그들의
주 행동들을 실행할 수 있습니다.

<figure class="image">
<img
src="../../images/patterns/diagrams/chain-of-responsibility/solution1-koa709.png?id=7d44cebc959187615a965791aecee3df"
style="aspect-ratio: 4;"
srcset="/images/patterns/diagrams/chain-of-responsibility/solution1-ko.png?id=7d44cebc959187615a965791aecee3df 640w,/images/patterns/diagrams/chain-of-responsibility/solution1-ko-1.5x.png?id=985d7c5c1e9879dc59acad9aa7d424b9 960w,/images/patterns/diagrams/chain-of-responsibility/solution1-ko-2x.png?id=d36782ad64bf8aa8369e185a36869ec4 1280w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="640"
alt="핸들러들이 하나씩 줄지어 체인을 형성합니다." />
<figcaption><p>핸들러들이 하나씩 줄지어
체인을 형성합니다.</p></figcaption>
</figure>

한편, 약간 다른 조금 더 정식적인 접근 방법이 있습니다. 이 방식에서는
핸들러가 요청을 받으면 핸들러는 요청을 처리할 수 있는지를 판단합니다.
처리가 가능한 경우, 핸들러는 이 요청을 더 이상 전달하지 않습니다. 따라서
요청을 처리하는 핸들러는 하나뿐이거나 아무 핸들러도 요청을 처리하지
않습니다. 이 접근 방식은 그래픽 사용자 인터페이스 내에서 요소들의
스택에서 이벤트들을 처리할 때 매우 일반적입니다.

예를 들어, 사용자가 버튼을 클릭하면 결과 이벤트는 그래픽 사용자
인터페이스 요소 체인을 통해 전파됩니다. 이 체인은 버튼으로 시작하여 해당
컨테이너들​(예: 양식 또는 패널)​을 따라 이동한 후 메인 애플리케이션 창으로
끝납니다. 또 이 이벤트는 그를 처리할 수 있는 체인의 첫 번째 요소에 의해
처리됩니다. 이 예가 주목할 만한 이유는 체인이 항상 객체 트리에서 추출될
수 있음을 보여주기 때문입니다.

<figure class="image">
<img
src="../../images/patterns/diagrams/chain-of-responsibility/solution2-koaba1.png?id=3100bb0dc7a555045daca3168b16d447"
style="aspect-ratio: 1.73;"
srcset="/images/patterns/diagrams/chain-of-responsibility/solution2-ko.png?id=3100bb0dc7a555045daca3168b16d447 520w,/images/patterns/diagrams/chain-of-responsibility/solution2-ko-1.5x.png?id=fe1c02f650bac176c804753ad1295228 780w,/images/patterns/diagrams/chain-of-responsibility/solution2-ko-2x.png?id=a046b0c919f5b079294e2e2437f9cbff 1040w"
sizes="(max-width: 720px) 100vw, 520px" loading="lazy" width="520"
alt="체인은 객체 트리의 가지에서 형성될 수 있습니다" />
<figcaption><p>체인은 객체 트리의 가지에서부터 형성될
수 있습니다.</p></figcaption>
</figure>

모든 핸들러 클래스들이 같은 인터페이스를 구현하는 것은 매우 중요합니다.
각 구상 핸들러는 `execute` 메서드가 있는 다음 핸들러에만 신경을 써야
합니다. 이렇게 하면 다양한 핸들러들을 사용하여 코드를 핸들러들의 구상
클래스들에 결합하지 않고도 런타임에 체인들을 구성할 수 있습니다.
:::

::: {.section .analogy}
##  실제상황 적용 {#analogy}

<figure class="image">
<img
src="../../images/patterns/content/chain-of-responsibility/chain-of-responsibility-comic-1-ko39f8.png?id=4446817e1e3108a57a1f3a9f113f06b4"
style="aspect-ratio: 2;"
srcset="/images/patterns/content/chain-of-responsibility/chain-of-responsibility-comic-1-ko.png?id=4446817e1e3108a57a1f3a9f113f06b4 600w,/images/patterns/content/chain-of-responsibility/chain-of-responsibility-comic-1-ko-1.5x.png?id=c9618c6f307d01b1a3a6ac1999974bdf 900w,/images/patterns/content/chain-of-responsibility/chain-of-responsibility-comic-1-ko-2x.png?id=308d05c4340d04c7174f2fddb4259636 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="기술 지원팀과의 대화는 어려울 수 있습니다" />
<figcaption><p>기술 지원 부서로의 전화는 여러 교환원을 거쳐 이루어질
수 있습니다.</p></figcaption>
</figure>

당신은 당신의 컴퓨터에 새 하드웨어를 구매하여 설치했습니다. 당신은
컴퓨터 괴짜이기 때문에 당신의 컴퓨터에는 여러 운영 체제가 설치되어
있습니다. 당신은 이 하드웨어가 지원되는지 확인하기 위해 모든 운영
체제들의 부팅을 시도합니다. 윈도우는 하드웨어를 자동으로 감지하고
활성화합니다. 그러나 당신이 애지중지하는 리눅스 운영 체제는 새
하드웨어와 작업하는 것을 거부합니다. 당신은 약간의 희망을 품고 상자에
적힌 기술 지원 전화번호로 전화하기로 합니다.

가장 먼저 들리는 것은 자동 응답기의 로봇 음성입니다. 이 음성은 다양한
문제에 대한 9가지 인기 있는 솔루션을 제안하지만, 그 중 어느 것도 당신의
문제와 관련이 없습니다. 잠시 후 로봇이 당신을 실제 교환원에게
연결합니다.

그러나 이 교환원도 별로 도움이 될만한 제안을 하지 않습니다. 그는 당신의
문제를 경청하지 않은 채 사용자 설명서에서 발췌한 긴 문장을 계속
인용합니다. \'컴퓨터를 껐다가 다시 켜 보셨습니까?\' 같은 별 쓸모없는
문구를 10번 이상 들은 후, 당신은 적절한 엔지니어와 연결해 줄 것을
요구합니다.

결국 드디어 교환원은 어둡고 외로운 지하 서버실에서 인간적인 접촉을
갈망했던 엔지니어 중 한 명에게 전화를 연결합니다. 이 엔지니어는 새
하드웨어에 적합한 드라이브를 어디에서 다운받아야 하는지와 리눅스에
설치하는 방법 등을 알려줍니다. 드디어 문제가 해결되었군요! 당신은 기쁜
마음으로 통화를 종료합니다.
:::

::::: {.section .structure-container}
##  구조 {#structure}

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/chain-of-responsibility/structure03c9.png?id=848f0fc8dca57a44974d63f8181f5406"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 0.93;"
srcset="/images/patterns/diagrams/chain-of-responsibility/structure.png?id=848f0fc8dca57a44974d63f8181f5406 380w,/images/patterns/diagrams/chain-of-responsibility/structure-1.5x.png?id=49dfbae38f146af7319f80dce9ea7e2a 570w,/images/patterns/diagrams/chain-of-responsibility/structure-2x.png?id=bb837faaac88e7f2a16f751d0beaa201 760w"
sizes="(max-width: 720px) 100vw, 380px" loading="lazy" width="380"
alt="책임 연쇄 디자인 패턴 구조" /><img
src="../../images/patterns/diagrams/chain-of-responsibility/structure-indexedfcb6.png?id=e13a5bf44f9ca47299223116af77cbef"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 0.88;"
srcset="/images/patterns/diagrams/chain-of-responsibility/structure-indexed.png?id=e13a5bf44f9ca47299223116af77cbef 380w,/images/patterns/diagrams/chain-of-responsibility/structure-indexed-1.5x.png?id=ca660e2cb697b512aadc07fdd3e109cd 570w,/images/patterns/diagrams/chain-of-responsibility/structure-indexed-2x.png?id=4f27e2c48e635f45a78472d707a8df3c 760w"
sizes="(max-width: 720px) 100vw, 380px" loading="lazy" width="380"
alt="책임 연쇄 디자인 패턴 구조" />
</figure>
:::

1.  **핸들러**는 모든 구상 핸들러에 공통적인 인터페이스를 선언합니다.
    일반적으로 여기에는 요청을 처리하기 위한 단일 메서드만 포함되지만
    때로는 체인의 다음 핸들러를 세팅하기 위한 다른 메서드가 있을 수도
    있습니다.

2.  **기초 핸들러**는 선택적 클래스이며 여기에 모든 핸들러 클래스들에
    공통적인 상용구 코드를 넣을 수 있습니다.

    일반적으로 이 클래스는 다음 핸들러에 대한 참조를 저장하기 위한
    필드를 정의합니다. 클라이언트들은 핸들러를 이전 핸들러의 생성자 또는
    세터​(setter)​에 해당 핸들러를 전달하여 체인을 구축할 수 있습니다. 또
    클래스는 디폴트 핸들러 행동을 구현할 수도 있습니다. 즉, 다음
    핸들러의 존재 여부를 확인한 후 다음 핸들러로 실행을 넘길 수
    있습니다.

3.  **구상 핸들러들**에는 요청을 처리하기 위한 실제 코드가 포함되어
    있습니다. 각 핸들러는 요청을 받으면 이 요청을 처리할지와 함께 체인을
    따라 전달할지를 결정해야 합니다.

    핸들러들은 일반적으로 자체 포함형이고 불변하며, 생성자를 통해 필요한
    모든 데이터를 한 번만 받습니다.

4.  **클라이언트**는 앱의 논리에 따라 체인들을 한 번만 구성하거나
    동적으로 구성할 수 있습니다. 참고로 요청은 체인의 모든 핸들러에 보낼
    수 있으며, 꼭 첫 번째 핸들러일 필요는 없습니다.
::::
:::::

::: {.section .pseudocode}
##  의사코드 {#pseudocode}

이 예에서 **책임 연쇄** 패턴은 활성 그래픽 사용자 인터페이스 요소에 대한
상황별 도움말 정보를 표시하는 역할을 합니다.

<figure class="image">
<img
src="../../images/patterns/diagrams/chain-of-responsibility/example-kof847.png?id=a2841a183f6f0b76f5a01aaa511fa6e0"
style="aspect-ratio: 1.09;"
srcset="/images/patterns/diagrams/chain-of-responsibility/example-ko.png?id=a2841a183f6f0b76f5a01aaa511fa6e0 610w,/images/patterns/diagrams/chain-of-responsibility/example-ko-1.5x.png?id=e331eb3f0e54fdfdbe6ae7a56ed0e2f6 915w,/images/patterns/diagrams/chain-of-responsibility/example-ko-2x.png?id=31af9b7c4e3a9e84d5816f4648abc75f 1220w"
sizes="(max-width: 720px) 100vw, 610px" loading="lazy" width="610"
alt="책임 연쇄 패턴 구조 예시" />
<figcaption><p>그래픽 사용자 인터페이스 클래스들은 복합체 패턴으로
빌드되고, 각 요소는 그 요소의 컨테이너 요소에 연결됩니다. 또 언제든지
요소 자체에서 시작하여 그 요소의 모든 컨테이너 요소를 통과하는 요소
체인을 구축할 수 있습니다.</p></figcaption>
</figure>

앱의 그래픽 사용자 인터페이스의 구조는 일반적으로 객체 트리로
구성됩니다. 예를 들어 앱의 기본 창을 렌더링하는 `Dialog`​(대화 상자)
클래스는 객체 트리의 뿌리​(root)​가 됩니다. `Dialog`에는
`Panels`​(패널들)​가 포함되어 있으며, 여기에는 다른 패널들이나
`Buttons`​(버튼들) 및 `Text­Fields`​(문자 필드들)​와 같은 단순한 하위 설계
요소들이 포함될 수 있습니다.

간단한 컴포넌트는 컴포넌트에 어떤 도움말 텍스트가 할당되어 있는 한
상황에 맞는 짧은 도구 도움말들을 표시할 수 있습니다. 그러나 더 복잡한
컴포넌트들은 (예를 들어 설명서에서 발췌한 내용을 표시하거나 브라우저에서
웹페이지를 여는 것과 같은 상황에 맞는 도움말을 표시하는 컴포넌트들)
상황별 도움말을 나타내기 위한 그들의 고유한 방법들을 정의할 수 있습니다.

<figure class="image">
<img
src="../../images/patterns/diagrams/chain-of-responsibility/example2-ko10ec.png?id=6f4ebeb751f897e41e77126282a6d7e4"
style="aspect-ratio: 0.81;"
srcset="/images/patterns/diagrams/chain-of-responsibility/example2-ko.png?id=6f4ebeb751f897e41e77126282a6d7e4 250w,/images/patterns/diagrams/chain-of-responsibility/example2-ko-1.5x.png?id=f2c98c6ca06079109976938b649505cc 375w,/images/patterns/diagrams/chain-of-responsibility/example2-ko-2x.png?id=2cbd5b794e503721ba951a5ab10c4da5 500w"
sizes="(max-width: 720px) 100vw, 250px" loading="lazy" width="250"
alt="책임 연쇄 패턴 구조 예시" />
<figcaption><p>도움말 요청이 그래픽 사용자 인터페이스 객체들을 가로질러
이동하는 방법입니다.</p></figcaption>
</figure>

사용자가 요소에 마우스 커서를 놓고 `F1` 키를 누르면 앱은 포인터 아래에
있는 컴포넌트를 감지하고 그에게 도움 요청을 보냅니다. 이 요청은 도움말
정보를 표시할 수 있는 요소에 도달할 때까지 모든 요소의 컨테이너를
통과하며 올라갑니다.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// 핸들러 인터페이스는 요청을 실행하기 위한 메서드를 선언합니다.
interface ComponentWithContextualHelp is
    method showHelp()


// 간단한 컴포넌트들의 기초 클래스.
abstract class Component implements ComponentWithContextualHelp is
    field tooltipText: string

    // 컴포넌트의 컨테이너는 핸들러 체인의 다음 링크 역할을 합니다.
    protected field container: Container

    // 컴포넌트는 도움말 텍스트가 할당되었을 때 도구 설명을 표시합니다. 그렇지
    // 않으면 컨테이너가 있는 경우 호출을 해당 컨테이너로 전달합니다.
    method showHelp() is
        if (tooltipText != null)
            // 도구 설명 표시하기.
        else
            container.showHelp()


// 컨테이너는 간단한 컴포넌트들과 다른 컨테이너들을 자식으로 포함할 수 있습니다.
// 여기에서 체인 관계들이 설립됩니다. 이 클래스는 부모로부터 showHelp 행동을
// 상속합니다.
abstract class Container extends Component is
    protected field children: array of Component

    method add(child) is
        children.add(child)
        child.container = this


// 원시적인 컴포넌트들은 디폴트 도움말 구현으로 괜찮을 수 있습니다…
class Button extends Component is
    // …

// 그러나 복잡한 컴포넌트들은 기초 구현을 오버라이드할 수 있습니다. 도움말 텍스트를
// 새로운 방식으로 제공할 수 없는 경우 컴포넌트는 언제든지 기초 구현을 호출할 수
// 있습니다. (컴포넌트 클래스 참조).
class Panel extends Container is
    field modalHelpText: string

    method showHelp() is
        if (modalHelpText != null)
            // 도움말 텍스트와 함께 모달 창을 표시합니다.
        else
            super.showHelp()

// …위와 같음…
class Dialog extends Container is
    field wikiPageURL: string

    method showHelp() is
        if (wikiPageURL != null)
            // 위키 도움말 페이지를 엽니다.
        else
            super.showHelp()


// 클라이언트 코드
class Application is
    // 모든 앱은 체인을 다르게 설정합니다.
    method createUI() is
        dialog = new Dialog(&quot;Budget Reports&quot;)
        dialog.wikiPageURL = &quot;http://...&quot;
        panel = new Panel(0, 0, 400, 800)
        panel.modalHelpText = &quot;This panel does...&quot;
        ok = new Button(250, 760, 50, 20, &quot;OK&quot;)
        ok.tooltipText = &quot;This is an OK button that...&quot;
        cancel = new Button(320, 760, 50, 20, &quot;Cancel&quot;)
        // …
        panel.add(ok)
        panel.add(cancel)
        dialog.add(panel)

    // 여기에서 무슨 일이 일어날지 상상해 보세요.
    method onF1KeyPress() is
        component = this.getComponentAtMouseCoords()
        component.showHelp()</code></pre>
</figure>
:::

:::::::::: {.section .applicability-container}
##  적용 {#applicability}

::::::::: applicability
::: applicability-problem
책임 연쇄 패턴은 당신의 프로그램이 다양한 방식으로 다양한 종류의
요청들을 처리할 것으로 예상되지만 정확한 요청 유형들과 순서들을 미리 알
수 없는 경우에 사용하세요.
:::

::: applicability-solution
이 패턴은 당신이 여러 핸들러를 하나의 체인으로 연결할 수 있도록 해주고,
또 요청을 받으면 바로 각 핸들러에게 이 요청을 처리할 수 있는지
질문합니다. 이렇게 해야 모든 핸들러들은 요청을 처리할 기회를 얻습니다.
:::

::: applicability-problem
이 패턴은 특정 순서로 여러 핸들러를 실행해야 할 때 사용하세요.
:::

::: applicability-solution
당신은 체인의 핸들러들을 원하는 순서로 연결할 수 있으므로 모든 요청은
정확히 당신이 계획한 대로 체인을 통과합니다.
:::

::: applicability-problem
책임 연쇄 패턴은 핸들러들의 집합과 그들의 순서가 런타임에 변경되어야 할
때 사용하세요.
:::

::: applicability-solution
당신이 핸들러 클래스들 내부의 참조 필드에 세터들을 제공하면, 핸들러들을
동적으로 삽입, 제거 또는 재정렬할 수 있을 것입니다.
:::
:::::::::
::::::::::

::: {.section .checklist}
##  구현방법 {#checklist}

1.  핸들러 인터페이스를 선언하고 요청을 처리하는 메서드의 시그니처를
    설명하세요.

    클라이언트가 요청 데이터를 메서드에 전달하는 방법을 결정하세요. 가장
    유연한 방법은 요청을 객체로 변환하여 처리 메서드에 인수로 전달하는
    것입니다.

2.  구상 핸들러들에서 중복된 상용구 코드를 제거하려면 핸들러
    인터페이스에서 파생된 추상 기초 핸들러 클래스를 만드는 것도 고려해볼
    만합니다.

    이 클래스에는 체인의 다음 핸들러에 대한 참조를 저장하기 위한 필드가
    있어야 합니다. 이 클래스를 불변으로 만드는 것을 고려하세요. 그러나
    런타임에 체인들을 수정할 계획이라면 참조 필드의 값을 변경하기 위한
    세터​(setter)​를 정의해야 합니다.

    당신은 또 처리​(핸들링) 메서드를 위한 편리한 디폴트​(기본값) 행동을
    구현할 수 있으며, 이 행동은 남아있는 객체가 없을 때까지 요청을 다음
    객체로 넘기는 것입니다. 구상 핸들러들은 부모 메서드를 호출하여 이
    행동을 사용할 수 있습니다.

3.  하나씩 구상 핸들러 자식 클래스들을 만들고 그들의 처리 메서드들을
    구현하세요. 각 핸들러는 요청을 받았을 때 두 가지 결정을 내려야
    합니다:

    -   요청을 처리할지의 여부.
    -   체인을 따라 요청을 전달할지의 여부.

4.  클라이언트는 자체적으로 체인을 조립하거나 다른 객체들에서부터 미리
    구축된 체인을 받을 수 있습니다. 후자의 경우 설정 또는 환경 설정에
    따라 체인들을 구축하기 위해 일부 공장 클래스들을 구현해야 합니다.

5.  클라이언트는 첫 번째 핸들러뿐만 아니라 체인의 모든 핸들러를 활성화할
    수 있습니다. 요청은 어떤 핸들러가 더 이상의 전달을 거부하거나 요청이
    체인 끝에 도달할 때까지 체인을 따라 전달됩니다.

6.  체인의 동적 특성으로 인해 클라이언트는 다음 상황들을 처리할 준비가
    되어 있어야 합니다:

    -   체인은 단일 링크로 구성될 수 있습니다.
    -   일부 요청들은 체인 끝에 도달하지 못할 수 있습니다.
    -   다른 요청들은 처리되지 않은 상태로 체인의 끝에 도달할 수
        있습니다.
:::

:::::: {.section .pros-cons}
##  장단점 {#pros-cons}

::::: row
::: col-sm-6
-    요청의 처리 순서를 제어할 수 있습니다.
-    *[단]{.cjk}[일]{.cjk} [책]{.cjk}[임]{.cjk} [원]{.cjk}[칙]{.cjk}*.
    당신은 작업을 호출하는 클래스들을 작업을 수행하는 클래스들과 분리할
    수 있습니다.
-    *[개]{.cjk}[방]{.cjk}/[폐]{.cjk}[쇄]{.cjk} [원]{.cjk}[칙]{.cjk}*.
    기존 클라이언트 코드를 손상하지 않고 앱에 새 핸들러들을 도입할 수
    있습니다.
:::

::: col-sm-6
-    일부 요청들은 처리되지 않을 수 있습니다.
:::
:::::
::::::

::: {.section .relations}
##  다른 패턴과의 관계 {#relations}

-   [커맨드](command.html), [중재자](mediator.html),
    [옵서버](observer.html) 및 [책임 연쇄](chain-of-responsibility.html)
    패턴은 요청의 발신자와 수신자를 연결하는 다양한 방법을 다룹니다.

    -   *[책]{.cjk}[임]{.cjk} [연]{.cjk}[쇄]{.cjk}* 패턴은 잠재적
        수신자의 동적 체인을 따라 수신자 중 하나에 의해 요청이 처리될
        때까지 요청을 순차적으로 전달합니다.
    -   *[커]{.cjk}[맨]{.cjk}[드]{.cjk}* 패턴은 발신자와 수신자 간의
        단방향 연결을 설립합니다.
    -   *[중]{.cjk}[재]{.cjk}[자]{.cjk}* 패턴은 발신자와 수신자 간의
        직접 연결을 제거하여 그들이 중재자 객체를 통해 간접적으로
        통신하도록 강제합니다.
    -   *[옵]{.cjk}[서]{.cjk}[버]{.cjk}* 패턴은 수신자들이 요청들의
        수신을 동적으로 구독 및 구독 취소할 수 있도록 합니다.

-   [책임 연쇄](chain-of-responsibility.html) 패턴은 종종
    [복합체](composite.html) 패턴과 함께 사용됩니다. 그러면 잎
    컴포넌트가 요청을 받으면 해당 요청을 모든 부모 컴포넌트들의 체인을
    통해 객체 트리의 뿌리​(root)​까지 전달할 수 있습니다.

-   [책임 연쇄](chain-of-responsibility.html) 패턴의 핸들러들은
    [커맨드](command.html)로 구현할 수 있습니다. 그러면 당신은 많은
    다양한 작업을 같은 콘텍스트 객체에 대해 실행할 수 있으며, 해당
    콘텍스트 객체는 요청의 역할을 합니다. 여기에서의 요청은 처리
    메서드의 매개변수를 의미합니다.

    그러나 요청 자체가 *[커]{.cjk}[맨]{.cjk}[드]{.cjk}* 객체인 다른 접근
    방식이 있습니다. 이 접근 방식을 사용하면 당신은 같은 작업을 체인에
    연결된 일련의 서로 다른 콘텍스트들에서 실행할 수 있습니다.

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
:::

::: {.section .implementations}
##  코드 예시 {#implementations}

[![C#으로 작성된 책임
연쇄](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](chain-of-responsibility/csharp/example.html "C#으로 작성된 책임 연쇄"){.prog-lang-link}
[![C++로 작성된 책임
연쇄](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](chain-of-responsibility/cpp/example.html "C++로 작성된 책임 연쇄"){.prog-lang-link}
[![Go로 작성된 책임
연쇄](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](chain-of-responsibility/go/example.html "Go로 작성된 책임 연쇄"){.prog-lang-link}
[![자바로 작성된 책임
연쇄](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](chain-of-responsibility/java/example.html "자바로 작성된 책임 연쇄"){.prog-lang-link}
[![PHP로 작성된 책임
연쇄](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](chain-of-responsibility/php/example.html "PHP로 작성된 책임 연쇄"){.prog-lang-link}
[![파이썬으로 작성된 책임
연쇄](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](chain-of-responsibility/python/example.html "파이썬으로 작성된 책임 연쇄"){.prog-lang-link}
[![루비로 작성된 책임
연쇄](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](chain-of-responsibility/ruby/example.html "루비로 작성된 책임 연쇄"){.prog-lang-link}
[![러스트로 작성된 책임
연쇄](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](chain-of-responsibility/rust/example.html "러스트로 작성된 책임 연쇄"){.prog-lang-link}
[![스위프트로 작성된 책임
연쇄](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](chain-of-responsibility/swift/example.html "스위프트로 작성된 책임 연쇄"){.prog-lang-link}
[![타입스크립트로 작성된 책임
연쇄](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](chain-of-responsibility/typescript/example.html "타입스크립트로 작성된 책임 연쇄"){.prog-lang-link}
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

[커맨드 []{.fa .fa-arrow-right}](command.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### 돌아가기

[[]{.fa .fa-arrow-left} 행동 패턴](behavioral-patterns.html){.btn
.btn-default rel="prev"}
:::
::::::::::::::::::::::::::::::::::

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
::::::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::::::
