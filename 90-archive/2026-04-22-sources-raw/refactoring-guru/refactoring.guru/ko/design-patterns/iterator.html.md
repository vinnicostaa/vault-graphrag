::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [디자인
패턴들](../design-patterns.html){.type} / [행동
패턴](behavioral-patterns.html){.type}
:::

# 반복자 패턴 {#반복자-패턴 .title}

::: aka
다음 이름으로도 불립니다: [Iterator]{style="display:inline-block"}
:::

::: {.section .intent}
##  의도 {#intent}

**반복자**는 컬렉션의 요소들의 기본 표현​(리스트, 스택, 트리 등)​을
노출하지 않고 그들을 하나씩 순회할 수 있도록 하는 행동
디자인 패턴입니다.

<figure class="image">
<img
src="../../images/patterns/content/iterator/iterator-kof037.png?id=a29a593168960cbd5a2788d2ff4ecd03"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/iterator/iterator-ko.png?id=a29a593168960cbd5a2788d2ff4ecd03 640w,/images/patterns/content/iterator/iterator-ko-1.5x.png?id=920f407d6adb70445f6c1907ab3a5c41 960w,/images/patterns/content/iterator/iterator-ko-2x.png?id=6647db43faac07745e9fe860e0c0b2a7 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="반복자 디자인 패턴" />
</figure>
:::

::: {.section .problem}
##  문제 {#problem}

컬렉션은 프로그래밍에서 가장 많이 사용되는 데이터 유형 중 하나이긴
하지만, 객체 그룹의 단순한 컨테이너에 불과합니다.

<figure class="image">
<img
src="../../images/patterns/diagrams/iterator/problem14cac.png?id=52ef2fe2d4920e3fed696c221fe757f2"
style="aspect-ratio: 4.9;"
srcset="/images/patterns/diagrams/iterator/problem1.png?id=52ef2fe2d4920e3fed696c221fe757f2 490w,/images/patterns/diagrams/iterator/problem1-1.5x.png?id=b46e39ade7de292fdcacc9790066c72f 735w,/images/patterns/diagrams/iterator/problem1-2x.png?id=1f4384c3c631be238bfc23d6eb84a0ef 980w"
sizes="(max-width: 720px) 100vw, 490px" loading="lazy" width="490"
alt="다양한 유형의 컬렉션들" />
<figcaption><p>다양한 유형들의 컬렉션들.</p></figcaption>
</figure>

대부분의 컬렉션들은 그들의 요소들을 간단한 리스트들에 저장하나, 그중
일부는 스택, 트리, 그래프 및 기타 복잡한 데이터 구조들을 기반으로
합니다.

그러나 컬렉션이 어떻게 구성되어 있는지를 떠나서, 컬렉션은 그 요소들에
접근할 수 있는 어떤 방법을 다른 코드에 제공해야 합니다. 그래야 다른
코드가 이 요소들을 사용할 수 있습니다. 같은 요소에 반복해서 접근하지
않고 컬렉션의 각 요소를 순회하는 방법이 있을 것입니다.

리스트로 된 컬렉션이 있다면 아주 쉽게 해결할 수 있을 겁니다. 모든 요소를
루프 처리하면 되니까요. 하지만 트리처럼 복잡한 데이터 구조의 요소들은
어떻게 순차적으로 순회해야 할까요? 예를 들어 어떤 날은 트리의 깊이를
우선으로 순회하는 것이 적절할지도 모릅니다. 하지만 그다음 날에는 너비를
우선으로 순회해야 할 수도 있습니다. 그리고 그다음 주에는 트리 요소들에
대한 임의 접근 등 다른 방식의 순회가 필요할지도 모릅니다.

<figure class="image">
<img
src="../../images/patterns/diagrams/iterator/problem29f71.png?id=f9c1a746c787320291c85fdc2a314192"
style="aspect-ratio: 3.75;"
srcset="/images/patterns/diagrams/iterator/problem2.png?id=f9c1a746c787320291c85fdc2a314192 600w,/images/patterns/diagrams/iterator/problem2-1.5x.png?id=7a003d020e789773e0c833d7b1df00e6 900w,/images/patterns/diagrams/iterator/problem2-2x.png?id=656b13c109d4d4960ea1f9fa993bae4c 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="다양한 순회 알고리즘들" />
<figcaption><p>같은 컬렉션을 여러 가지 방법들로 순회할
수 있습니다.</p></figcaption>
</figure>

현재 컬렉션의 주요 책임은 효율적인 데이터 저장이나, 컬렉션에 더 많은
순회 알고리즘들을 추가할수록 컬렉션의 주요 책임이 무엇인지 점점
명확해지지 않게 됩니다. 또한 일부 알고리즘들은 특정 앱에 맞게 조정되었을
수 있으므로 일반적인 컬렉션 클래스에 이들을 포함하는 것은 이상할 수
있습니다.

반면에 다양한 컬렉션들과 작동해야 하는 클라이언트 코드는 자신들의 요소가
어떻게 저장되는지 관심을 두지 않습니다. 하지만 컬렉션마다 그 요소들에
접근할 수 있도록 허용하는 방법이 다르므로, 당신은 코드를 특정한 컬렉션
클래스에 결합할 수밖에 없습니다.
:::

::: {.section .solution}
##  해결책 {#solution}

반복자 패턴의 주 아이디어는 컬렉션의 순회 동작을
*[반]{.cjk}[복]{.cjk}[자]{.cjk}*라는 별도의 객체로 추출하는 것입니다.

<figure class="image">
<img
src="../../images/patterns/diagrams/iterator/solution19739.png?id=2f5fbcce6099d8ea09b2fbb83e3e7059"
style="aspect-ratio: 0.85;"
srcset="/images/patterns/diagrams/iterator/solution1.png?id=2f5fbcce6099d8ea09b2fbb83e3e7059 400w,/images/patterns/diagrams/iterator/solution1-1.5x.png?id=68612372ede6e5029403d38b381fdc05 600w,/images/patterns/diagrams/iterator/solution1-2x.png?id=dcebcad4c197bfa5f25f680382d0e5ac 800w"
sizes="(max-width: 720px) 100vw, 400px" loading="lazy" width="400"
alt="반복자들은 다양한 순회 알고리즘들을 구현합니다" />
<figcaption><p>반복자들은 다양한 순회 알고리즘들을 구현합니다. 여러
반복자 객체들이 동시에 같은 컬렉션을 순회할
수 있습니다.</p></figcaption>
</figure>

반복자 객체는 알고리즘 자체를 구현하는 것 외에도 모든 순회 세부
정보들​(예: 현재 위치 및 남은 요소들의 수)​을 캡슐화하며, 이 때문에 여러
반복자들이 서로 독립적으로 동시에 같은 컬렉션을 통과할 수 있습니다.

일반적으로 반복자들은 컬렉션의 요소들을 가져오기 위한 하나의 주 메서드를
제공합니다. 클라이언트는 이 메서드를 더 이상 아무것도 반환하지 않을
때까지 계속 실행할 수 있습니다. 이는 반복자가 모든 요소를 순회했음을
의미합니다.

모든 반복자들은 같은 인터페이스를 구현해야 합니다. 이렇게 하면 적절한
반복자가 있는 한 클라이언트 코드는 모든 컬렉션 유형들 및 순회
알고리즘들과 호환됩니다. 컬렉션을 순회하는 특별한 방법이 필요하면
컬렉션이나 클라이언트를 변경할 필요 없이 새 반복자 클래스를 만들기만
하면 됩니다.
:::

::: {.section .analogy}
##  실제상황 적용 {#analogy}

<figure class="image">
<img
src="../../images/patterns/content/iterator/iterator-comic-1-koe8f8.png?id=2563d015bb4cec16706340e5ebc3091e"
style="aspect-ratio: 2.13;"
srcset="/images/patterns/content/iterator/iterator-comic-1-ko.png?id=2563d015bb4cec16706340e5ebc3091e 640w,/images/patterns/content/iterator/iterator-comic-1-ko-1.5x.png?id=a8303cefcd86d694928ed425fb07f156 960w,/images/patterns/content/iterator/iterator-comic-1-ko-2x.png?id=c414af88adb1950b24261d64a4608515 1280w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="640"
alt="도보로 로마를 탐험하는 다양한 방법들" />
<figcaption><p>도보로 로마를 탐험하는 다양한 방법들</p></figcaption>
</figure>

당신은 며칠 동안 로마를 방문하고 주요 명소들을 모두 방문할 계획을
세웠습니다. 그러나 막상 도착한 후 당신은 콜로세움조차 찾지 못하고
제자리를 맴도는 데 많은 시간을 허비할 수 있습니다.

위 사례의 대안으로 스마트폰용 가상 가이드 앱을 구매하여 내비게이션에
사용할 수 있습니다. 이 앱은 똑똑하고 저렴하며 실제 가이드와 달리 원하는
만큼 흥미로운 장소에 머물 수 있도록 합니다.

세 번째 대안은 조금 더 비싸더라도 도시를 잘 아는 현지 가이드를 고용하는
것입니다. 현지 가이드는 당신의 취향에 맞게 여행을 조정하고 모든 명소를
보여주며 흥미진진한 이야기들을 많이 알려줄 수 있습니다. 이 대안을
선택하면 재미는 있겠지만, 더 많은 비용이 들게 될 것입니다.

이 모든 대안들은 (예: 당신이 무작위로 생각해 낸 방향들, 스마트폰
내비게이터, 현지 가이드) 로마의 많은 관광명소에 대해 반복자들로
작동합니다.
:::

::::: {.section .structure-container}
##  구조 {#structure}

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/iterator/structure5247.png?id=35ea851f8f6bbe51d79eb91e6e6519d0"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.12;"
srcset="/images/patterns/diagrams/iterator/structure.png?id=35ea851f8f6bbe51d79eb91e6e6519d0 480w,/images/patterns/diagrams/iterator/structure-1.5x.png?id=19d4c2130ab2e3bd718f87e956fcd873 720w,/images/patterns/diagrams/iterator/structure-2x.png?id=b7b4708d3f279dd046eb1692f1cba710 960w"
sizes="(max-width: 720px) 100vw, 480px" loading="lazy" width="480"
alt="반복자 디자인 패턴 구조" /><img
src="../../images/patterns/diagrams/iterator/structure-indexed3472.png?id=7bc28907ff6b480db6635a93ebaa10ff"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.33;"
srcset="/images/patterns/diagrams/iterator/structure-indexed.png?id=7bc28907ff6b480db6635a93ebaa10ff 520w,/images/patterns/diagrams/iterator/structure-indexed-1.5x.png?id=32fde4c4c1cbfbe07aa6e716e5ac2346 780w,/images/patterns/diagrams/iterator/structure-indexed-2x.png?id=d809428b2262e2013972fe69d2434473 1040w"
sizes="(max-width: 720px) 100vw, 520px" loading="lazy" width="520"
alt="반복자 디자인 패턴 구조" />
</figure>
:::

1.  **반복자** 인터페이스는 컬렉션의 순회에 필요한 작업들​(예: 다음 요소
    가져오기, 현재 위치 가져오기, 반복자 다시 시작 등)​을 선언합니다.

2.  **구상 반복자들**은 컬렉션 순회를 위한 특정 알고리즘들을 구현합니다.
    반복자 객체는 순회의 진행 상황을 자체적으로 추적해야 합니다. 이는
    여러 반복자들이 같은 컬렉션을 서로 독립적으로 순회할 수 있도록
    합니다.

3.  **컬렉션** 인터페이스는 컬렉션과 호환되는 반복자들을 가져오기 위한
    하나 이상의 메서드들을 선언합니다. 참고로 메서드들의 반환 유형은
    반복자 인터페이스의 유형으로 선언되어야 합니다. 그래야 구상
    컬렉션들이 다양한 유형의 반복자들을 반환할 수 있기 때문입니다.

4.  **구상 컬렉션들**은 클라이언트가 요청할 때마다 특정 구상 반복자
    클래스의 새 인스턴스들을 반환합니다. 당신은 컬렉션의 나머지 코드가
    어디에 있는지 궁금하실 수도 있습니다. 걱정하지 마세요, 같은 클래스에
    있을 것입니다. 나머지 코드가 어디에 있는지와 같은 세부 사항들은 실제
    패턴에 중요하지 않으므로 생략하기로 했습니다.

5.  **클라이언트**는 반복자들과 컬렉션들의 인터페이스를 통해 그들과 함께
    작동합니다. 이렇게 하면 클라이언트가 구상 클래스들에 결합하지
    않으므로 같은 클라이언트 코드로 다양한 컬렉션들과 반복자들을 사용할
    수 있도록 합니다.

    일반적으로 클라이언트들은 자체적으로 반복자들을 생성하지 않고 대신
    컬렉션들에서 가져옵니다. 그러나 어떤 경우에는 (예를 들어
    클라이언트가 자체 특수 반복자를 정의할 때) 클라이언트가 반복자를
    직접 만들 수 있습니다.
::::
:::::

::: {.section .pseudocode}
##  의사코드 {#pseudocode}

이 예시에서 **반복자** 패턴은 페이스북의 소셜 그래프에 대한 접근을
캡슐화하는 특별한 종류의 컬렉션을 순회하는 데 사용합니다. 이 컬렉션은
다양한 방식으로 프로필들을 순회할 수 있는 여러 반복자들을 제공합니다.

<figure class="image">
<img
src="../../images/patterns/diagrams/iterator/examplee0a2.png?id=f2a24ef3787bf80ed450709240506ff2"
style="aspect-ratio: 0.95;"
srcset="/images/patterns/diagrams/iterator/example.png?id=f2a24ef3787bf80ed450709240506ff2 600w,/images/patterns/diagrams/iterator/example-1.5x.png?id=cab0e1459ffc3721579a3e7d6a4e9022 900w,/images/patterns/diagrams/iterator/example-2x.png?id=73c3e55f75ca0250bd84e8a1d8cc88d2 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="반복자 패턴 구조 예시" />
<figcaption><p>소셜 프로필들의 순회 예시.</p></figcaption>
</figure>

`Friends`​(친구들) 반복자는 주어진 프로필의 친구들을 탐색하는 데 사용될
수 있습니다. `Colleagues`​(동료들) 반복자는 프로필 주인과 같은 회사에서
일하지 않는 친구들을 제외한 후 같은 작업을 수행합니다. 두 반복자 모두
인증 및 REST 요청 전송과 같은 구현 세부 사항들을 자세히 살펴보지 않고도
클라이언트들이 프로필들을 가져올 수 있도록 하는 공통 인터페이스를
구현합니다.

클라이언트 코드는 인터페이스들을 통해서만 컬렉션들 및 반복자들과
작업하기 때문에 구상 클래스들과 결합하지 않습니다. 앱을 새로운 소셜
네트워크에 연결하기로 했다면 기존 코드를 변경하지 않고 새로운 컬렉션 및
반복자 클래스들을 제공하기만 하면 됩니다.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// 컬렉션 인터페이스는 반복자들을 생성하기 위한 팩토리 메서드를 선언해야 합니다.
// 프로그램에서 사용할 수 있는 다양한 종류의 순회가 있는 경우 여러 메서드를 선언할
// 수 있습니다.
interface SocialNetwork is
    method createFriendsIterator(profileId):ProfileIterator
    method createCoworkersIterator(profileId):ProfileIterator


// 각 구상 컬렉션은 자신이 반환하는 구상 반복자 클래스들의 집합에 연결됩니다.
// 하지만 이러한 메서드들의 시그니처는 반복자 인터페이스를 반환하기 때문에
// 클라이언트는 연결되지 않습니다.
class Facebook implements SocialNetwork is
    // …컬렉션 코드의 대부분은 여기에 포함되어야 합니다…

    // 반복자 생성 코드.
    method createFriendsIterator(profileId) is
        return new FacebookIterator(this, profileId, &quot;friends&quot;)
    method createCoworkersIterator(profileId) is
        return new FacebookIterator(this, profileId, &quot;coworkers&quot;)


// 모든 반복자에 대한 공통 인터페이스.
interface ProfileIterator is
    method getNext():Profile
    method hasMore():bool


// 구상 반복자 클래스.
class FacebookIterator implements ProfileIterator is
    // 반복자는 순회하는 컬렉션에 대한 참조가 필요합니다.
    private field facebook: Facebook
    private field profileId, type: string

    // 반복자 객체는 다른 반복자들과 별도로 컬렉션을 순회합니다. 따라서 반복자
    // 상태를 저장해야 합니다.
    private field currentPosition
    private field cache: array of Profile

    constructor FacebookIterator(facebook, profileId, type) is
        this.facebook = facebook
        this.profileId = profileId
        this.type = type

    private method lazyInit() is
        if (cache == null)
            cache = facebook.socialGraphRequest(profileId, type)

    // 각 구상 반복자 클래스는 공통 반복자 인터페이스를 자체적으로 구현합니다.
    method getNext() is
        if (hasMore())
            result = cache[currentPosition]
            currentPosition++
            return result

    method hasMore() is
        lazyInit()
        return currentPosition &lt; cache.length


// 유용한 요령이 하나 더 있습니다. 전체 컬렉션에 대한 접근 권한을 클라이언트
// 클래스에 부여하는 대신 반복자를 클라이언트 클래스에 전달하는 것입니다. 그러면
// 컬렉션이 클라이언트에 노출되지 않습니다.
//
// 장점도 하나 더 있습니다. 클라이언트에 다른 반복자를 전달하여 런타임 때
// 클라이언트가 컬렉션과 작동하는 방식을 변경할 수 있습니다. 이것은 클라이언트
// 코드가 구상 반복자 클래스들에 결합되어 있지 않기 때문에 가능합니다.
class SocialSpammer is
    method send(iterator: ProfileIterator, message: string) is
        while (iterator.hasMore())
            profile = iterator.getNext()
            System.sendEmail(profile.getEmail(), message)


// 앱 클래스는 컬렉션들과 반복자들을 설정한 다음 클라이언트 코드에 전달합니다.
class Application is
    field network: SocialNetwork
    field spammer: SocialSpammer

    method config() is
        if working with Facebook
            this.network = new Facebook()
        if working with LinkedIn
            this.network = new LinkedIn()
        this.spammer = new SocialSpammer()

    method sendSpamToFriends(profile) is
        iterator = network.createFriendsIterator(profile.getId())
        spammer.send(iterator, &quot;Very important message&quot;)

    method sendSpamToCoworkers(profile) is
        iterator = network.createCoworkersIterator(profile.getId())
        spammer.send(iterator, &quot;Very important message&quot;)</code></pre>
</figure>
:::

:::::::::: {.section .applicability-container}
##  적용 {#applicability}

::::::::: applicability
::: applicability-problem
반복자 패턴은 당신의 컬렉션이 내부에 복잡한 데이터 구조가 있지만 이
구조의 복잡성을 보안이나 편의상의 이유로 클라이언트들로부터 숨기고 싶을
때 사용하세요.
:::

::: applicability-solution
반복자는 복잡한 데이터 구조와 작업 시의 세부 사항을 캡슐화하여
클라이언트에 컬렉션 요소들에 접근할 수 있는 몇 가지 간단한 메서드들을
제공합니다. 이 접근 방식은 클라이언트에게 매우 편리합니다. 또
클라이언트가 컬렉션과 직접 작동할 때 클라이언트가 수행할 수 있는
부주의하거나 악의적인 행동들로부터 컬렉션을 보호합니다.
:::

::: applicability-problem
반복자 패턴을 사용하여 당신의 앱 전체에서 순회 코드의 중복을 줄이세요.
:::

::: applicability-solution
사소하지 않은 순회 알고리즘들의 코드는 부피가 매우 큰 경향이 있습니다.
이 코드들이 앱의 비즈니스 로직 내에 배치되면 원래 코드의 책임이 무엇인지
모호해질 수 있으며 코드의 유지관리가 더 어려워질 수 있습니다. 순회
코드를 지정된 반복자들로 이동하면 당신의 앱의 코드가 더 간결하고
깔끔해질 수 있습니다.
:::

::: applicability-problem
반복자 패턴은 코드가 다른 데이터 구조들을 순회할 수 있기를 원할 때 또는
이러한 구조들의 유형을 미리 알 수 없을 때 사용하세요.
:::

::: applicability-solution
이 패턴은 컬렉션들과 반복자들 모두에 몇 개의 일반 인터페이스들을
제공합니다. 당신의 코드가 이러한 인터페이스들을 사용한다는 점을 고려할
때, 이러한 인터페이스들은 그들을 구현하는 다양한 컬렉션들 및 반복자들을
전달받아도 여전히 작동합니다.
:::
:::::::::
::::::::::

::: {.section .checklist}
##  구현방법 {#checklist}

1.  반복자 인터페이스를 선언하세요. 이 인터페이스는 최소한 컬렉션에서
    다음 요소를 가져오는 메서드가 있어야 하며, 또 편의를 위해 여기에 몇
    가지 다른 메서드들도 (예: 전 요소를 가져오는, 현재 위치를 추적하는,
    그리고 반복자의 순회의 끝을 확인하는 메서드들) 추가할 수 있습니다.

2.  컬렉션 인터페이스를 선언하고 반복자를 가져오는 메서드를 설명하세요.
    컬렉션 인터페이스의 반환 유형은 반복자 인터페이스의 유형과 같아야
    합니다. 뚜렷하게 다른 여러 개의 반복자들의 그룹을 가질 계획이라면
    유사한 메서드들을 선언할 수 있습니다.

3.  반복자들이 순회하게 할 수 있도록 하고 싶은 컬렉션들에 대한 구상
    반복자 클래스들을 구현하세요. 반복자 객체는 단일 컬렉션 인스턴스와
    반드시 연결되어야 합니다. 일반적으로 이러한 연결은 반복자의 생성자를
    통해 맺어집니다.

4.  당신의 컬렉션 클래스들에서 컬렉션 인터페이스를 구현하세요. 그 주된
    목적은 클라이언트에 특정 컬렉션 클래스에 맞게 조정된 반복자들을
    생성하기 위한 바로 가기를 제공하는 것입니다. 컬렉션 객체는 반복자의
    생성자에 자신을 전달해야 하며 이 둘 사이에 연결을 맺어야 합니다.

5.  클라이언트 코드를 살펴보면서 반복자들을 사용하여 모든 컬렉션 순회
    코드들을 교체하세요. 클라이언트는 컬렉션 요소들을 순회해야 할 때마다
    새 반복자 객체를 가져옵니다.
:::

:::::: {.section .pros-cons}
##  장단점 {#pros-cons}

::::: row
::: col-sm-6
-    *[단]{.cjk}[일]{.cjk} [책]{.cjk}[임]{.cjk} [원]{.cjk}[칙]{.cjk}*.
    부피가 큰 순회 알고리즘들을 별도의 클래스들로 추출하여 클라이언트
    코드와 컬렉션들을 정돈할 수 있습니다.
-    *[개]{.cjk}[방]{.cjk}/[폐]{.cjk}[쇄]{.cjk} [원]{.cjk}[칙]{.cjk}*.
    새로운 유형의 컬렉션들과 반복자들을 구현할 수 있으며 이들을 아무것도
    훼손하지 않은 체 기존의 코드에 전달할 수 있습니다.
-    당신은 이제 같은 컬렉션을 병렬로 순회할 수 있습니다. 왜냐하면 각
    반복자 객체에는 자신의 고유한 순회 상태가 포함되어 있기 때문입니다.
-    같은 이유로 당신은 순회를 지연하고 필요할 때 계속할 수 있습니다.
:::

::: col-sm-6
-    당신의 앱이 단순한 컬렉션들과만 작동하는 경우 반복자 패턴을
    적용하는 것은 과도할 수 있습니다.
-    반복자를 사용하는 것은 일부 특수 컬렉션들의 요소들을 직접 탐색하는
    것보다 덜 효율적일 수 있습니다.
:::
:::::
::::::

::: {.section .relations}
##  다른 패턴과의 관계 {#relations}

-   당신은 [반복자들](iterator.html)을 사용하여 [복합체](composite.html)
    패턴 트리들을 순회할 수 있습니다.

-   [팩토리 메서드](factory-method.html)를 [반복자](iterator.html)와
    함께 사용하여 컬렉션 자식 클래스들이 해당 컬렉션들과 호환되는 다양한
    유형의 반복자들을 반환하도록 할 수 있습니다.

-   [메멘토](memento.html) 패턴을 [반복자](iterator.html) 패턴과 함께
    사용하여 현재 순회 상태를 포착하고 필요한 경우 롤백할 수 있습니다.

-   [비지터](visitor.html) 패턴과 [반복자](iterator.html) 패턴을 함께
    사용해 복잡한 데이터 구조를 순회하여 해당 구조의 요소들의 클래스들이
    모두 다르더라도 이러한 요소들에 대해 어떤 작업을 실행할 수 있습니다.
:::

::: {.section .implementations}
##  코드 예시 {#implementations}

[![C#으로 작성된
반복자](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](iterator/csharp/example.html "C#으로 작성된 반복자"){.prog-lang-link}
[![C++로 작성된
반복자](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](iterator/cpp/example.html "C++로 작성된 반복자"){.prog-lang-link}
[![Go로 작성된
반복자](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](iterator/go/example.html "Go로 작성된 반복자"){.prog-lang-link}
[![자바로 작성된
반복자](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](iterator/java/example.html "자바로 작성된 반복자"){.prog-lang-link}
[![PHP로 작성된
반복자](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](iterator/php/example.html "PHP로 작성된 반복자"){.prog-lang-link}
[![파이썬으로 작성된
반복자](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](iterator/python/example.html "파이썬으로 작성된 반복자"){.prog-lang-link}
[![루비로 작성된
반복자](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](iterator/ruby/example.html "루비로 작성된 반복자"){.prog-lang-link}
[![러스트로 작성된
반복자](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](iterator/rust/example.html "러스트로 작성된 반복자"){.prog-lang-link}
[![스위프트로 작성된
반복자](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](iterator/swift/example.html "스위프트로 작성된 반복자"){.prog-lang-link}
[![타입스크립트로 작성된
반복자](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](iterator/typescript/example.html "타입스크립트로 작성된 반복자"){.prog-lang-link}
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

[중재자 []{.fa .fa-arrow-right}](mediator.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### 돌아가기

[[]{.fa .fa-arrow-left} 커맨드](command.html){.btn .btn-default
rel="prev"}
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
