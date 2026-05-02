::::::::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [디자인
패턴들](../design-patterns.html){.type} / [구조
패턴](structural-patterns.html){.type}
:::

# 프록시 패턴 {#프록시-패턴 .title}

::: aka
다음 이름으로도 불립니다: [Proxy]{style="display:inline-block"}
:::

::: {.section .intent}
##  의도 {#intent}

**프록시**는 다른 객체에 대한 대체 또는 자리표시자를 제공할 수 있는 구조
디자인 패턴입니다. 프록시는 원래 객체에 대한 접근을 제어하므로, 당신의
요청이 원래 객체에 전달되기 전 또는 후에 무언가를 수행할 수
있도록 합니다.

<figure class="image">
<img
src="../../images/patterns/content/proxy/proxyf258.png?id=efece4647fb11e3f7539291796327666"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/proxy/proxy.png?id=efece4647fb11e3f7539291796327666 640w,/images/patterns/content/proxy/proxy-1.5x.png?id=80a11690fa49172c517470a834289b60 960w,/images/patterns/content/proxy/proxy-2x.png?id=fb3d14e21c210a758d4777f4d93dce09 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="프록시 디자인 패턴" />
</figure>
:::

::: {.section .problem}
##  문제 {#problem}

객체에 대한 접근을 제한하는 이유는 무엇일까요? 이 질문에 답하기 위하여
방대한 양의 시스템 자원을 소비하는 거대한 객체가 있다고 가정합시다. 이
객체는 필요할 때가 있기는 하지만, 항상 필요한 것은 아닙니다.

<figure class="image">
<img
src="../../images/patterns/diagrams/proxy/problem-ko91e2.png?id=ab75180b50cdbd12af4a718a3b7e9c2f"
style="aspect-ratio: 3.19;"
srcset="/images/patterns/diagrams/proxy/problem-ko.png?id=ab75180b50cdbd12af4a718a3b7e9c2f 510w,/images/patterns/diagrams/proxy/problem-ko-1.5x.png?id=0cb7a133e2f071e5b0daec4d0a673aa4 765w,/images/patterns/diagrams/proxy/problem-ko-2x.png?id=468e42a38e757912261af8aa1c5cdd3c 1020w"
sizes="(max-width: 720px) 100vw, 510px" loading="lazy" width="510"
alt="프록시 패턴으로 해결된 문제" />
<figcaption><p>데이터베이스 쿼리들은 정말 느릴
수 있습니다.</p></figcaption>
</figure>

당신은 실제로 필요할 때만 이 객체를 만들어서 지연된 초기화를 구현할 수
있습니다. 그러면 객체의 모든 클라이언트들은 어떤 지연된 초기화 코드를
실행해야 하는데, 불행히도 이것은 아마도 많은 코드 중복을 초래할
것입니다.

이상적인 상황에서는 이 코드를 객체의 클래스에 직접 넣을 수 있겠지만,
그게 항상 가능한 것은 아닙니다. 예를 들어 그 클래스가 폐쇄된 타사
라이브러리의 일부일 수 있습니다.
:::

::: {.section .solution}
##  해결책 {#solution}

프록시 패턴은 원래 서비스 객체와 같은 인터페이스로 새 프록시 클래스를
생성하라고 제안합니다. 그러면 프록시 객체를 원래 객체의 모든
클라이언트들에 전달하도록 앱을 업데이트할 수 있습니다. 클라이언트로부터
요청을 받으면 이 프록시는 실제 서비스 객체를 생성하고 모든 작업을 이
객체에 위임합니다.

<figure class="image">
<img
src="../../images/patterns/diagrams/proxy/solution-ko3554.png?id=1c4cfc760f804ebf75f97a967fa2e562"
style="aspect-ratio: 3.19;"
srcset="/images/patterns/diagrams/proxy/solution-ko.png?id=1c4cfc760f804ebf75f97a967fa2e562 510w,/images/patterns/diagrams/proxy/solution-ko-1.5x.png?id=fdc68dbcb02a96fc508f0ab58b1bc51f 765w,/images/patterns/diagrams/proxy/solution-ko-2x.png?id=ccbd651d1d6d2f9e1b7fd36ea58641e9 1020w"
sizes="(max-width: 720px) 100vw, 510px" loading="lazy" width="510"
alt="프록시 패턴을 사용한 해결책" />
<figcaption><p>프록시는 데이터베이스 객체로 자신을 변장합니다. 프록시는
지연된 초기화 및 결괏값 캐싱을 클라이언트와 실제 데이터베이스 객체가
알지 못하는 상태에서 처리할 수 있습니다.</p></figcaption>
</figure>

그러나 이것들은 무슨 소용이 있는지 궁금하실 것입니다. 당신이 클래스의
메인 로직 이전이나 이후에 무언가를 실행해야 하는 경우 프록시는 해당
클래스를 변경하지 않고도 이 무언가를 수행할 수 있도록 합니다. 프록시는
원래 클래스와 같은 인터페이스를 구현하므로 실제 서비스 객체를 기대하는
모든 클라이언트에 전달될 수 있습니다.
:::

::: {.section .analogy}
##  실제상황 적용 {#analogy}

<figure class="image">
<img
src="../../images/patterns/diagrams/proxy/live-example66da.png?id=a268c57fdaf073ee81cf4dfc7239eae2"
style="aspect-ratio: 2.57;"
srcset="/images/patterns/diagrams/proxy/live-example.png?id=a268c57fdaf073ee81cf4dfc7239eae2 540w,/images/patterns/diagrams/proxy/live-example-1.5x.png?id=4b8ee7f90cf76180004e9f5d37661ee2 810w,/images/patterns/diagrams/proxy/live-example-2x.png?id=8b8083fa1954d2d92ca84a5a5636ead7 1080w"
sizes="(max-width: 720px) 100vw, 540px" loading="lazy" width="540"
alt="신용 카드는 현금 묶음의 프록시입니다." />
<figcaption><p>신용 카드는 현금과 마찬가지로 결제에 사용할
수 있습니다.</p></figcaption>
</figure>

신용 카드는 은행 계좌의 프록시이며, 은행 계좌는 현금의 프록시입니다. 둘
다 같은 인터페이스를 구현하며 둘 다 결제에 사용될 수 있습니다. 신용
카드를 사용하는 소비자는 많은 현금을 가지고 다닐 필요가 없어서 기분이
좋습니다. 또한 상점 주인은 거래 수입을 은행에 가는 길에 강도를 당하거나
잃어버릴 위험 없이 계좌에 전자적으로 입금이 되기 때문에 기분이 좋습니다.
:::

::::: {.section .structure-container}
##  구조 {#structure}

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/proxy/structure7fcd.png?id=f2478a82a84e1a1e512a8414bf1abd1c"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1;"
srcset="/images/patterns/diagrams/proxy/structure.png?id=f2478a82a84e1a1e512a8414bf1abd1c 370w,/images/patterns/diagrams/proxy/structure-1.5x.png?id=0db7bf3c1193b2a1961894349f1e07ad 555w,/images/patterns/diagrams/proxy/structure-2x.png?id=3d54eeca9af4aa373e989a73463539b5 740w"
sizes="(max-width: 720px) 100vw, 370px" loading="lazy" width="370"
alt="프록시 디자인 패턴의 구조" /><img
src="../../images/patterns/diagrams/proxy/structure-indexed2b2c.png?id=e56d420f31925b3d41348c5036d3cd64"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.05;"
srcset="/images/patterns/diagrams/proxy/structure-indexed.png?id=e56d420f31925b3d41348c5036d3cd64 410w,/images/patterns/diagrams/proxy/structure-indexed-1.5x.png?id=5b24af632e76d81d24eab0b7c0be1a66 615w,/images/patterns/diagrams/proxy/structure-indexed-2x.png?id=ddf2b3b4807e52330c456c59fc52d504 820w"
sizes="(max-width: 720px) 100vw, 410px" loading="lazy" width="410"
alt="프록시 디자인 패턴의 구조" />
</figure>
:::

1.  **서비스 인터페이스**는 서비스의 인터페이스를 선언합니다. 프록시가
    서비스 객체로 위장할 수 있으려면 이 인터페이스를 따라야 합니다.

2.  **서비스**는 어떤 유용한 비즈니스 로직을 제공하는 클래스입니다.

3.  **프록시** 클래스에는 서비스 객체를 가리키는 참조 필드가 있습니다.
    프록시가 요청의 처리​(예: 초기화 지연, 로깅, 액세스 제어, 캐싱 등)​를
    완료하면, 그 후 처리된 요청을 서비스 객체에 전달합니다.

    일반적으로 프록시들은 서비스 객체들의 전체 수명 주기를 관리합니다.

4.  **클라이언트**는 같은 인터페이스를 통해 서비스들 및 프록시들과 함께
    작동해야 합니다. 그러면 서비스 객체를 기대하는 모든 코드에 프록시를
    전달할 수 있기 때문입니다.
::::
:::::

::: {.section .pseudocode}
##  의사코드 {#pseudocode}

이 예시는 **프록시** 패턴이 제삼자 유튜브 통합 라이브러리에 지연된
초기화 및 캐싱을 도입하는 데 어떻게 도움이 되는지 보여줍니다.

<figure class="image">
<img
src="../../images/patterns/diagrams/proxy/example4e80.png?id=ddb1402479562b9e2c776933cc861bed"
style="aspect-ratio: 1.23;"
srcset="/images/patterns/diagrams/proxy/example.png?id=ddb1402479562b9e2c776933cc861bed 490w,/images/patterns/diagrams/proxy/example-1.5x.png?id=9b7608e1ef46a52d4ca1d7d89986e5b0 735w,/images/patterns/diagrams/proxy/example-2x.png?id=40f22d12d183b9285a380e4a072efb16 980w"
sizes="(max-width: 720px) 100vw, 490px" loading="lazy" width="490"
alt="프록시 패턴의 구조 예시" />
<figcaption><p>프록시를 사용하여 서비스의
결과를 캐싱합니다.</p></figcaption>
</figure>

이 라이브러리는 비디오 다운로드 클래스를 제공하나 매우 비효율적입니다.
왜냐하면 클라이언트 앱이 같은 비디오를 여러 번 요청하면 라이브러리는
처음 다운로드한 파일을 캐싱하고 재사용하는 대신 계속해서 같은 비디오를
다운로드하기 때문입니다.

프록시 클래스는 원래 다운로더와 같은 인터페이스를 구현하고 이 다운로더에
모든 작업을 위임하나, 앱이 같은 비디오를 두 번 이상 요청하면 이미
다운로드한 파일을 추적한 후 캐시 된 결과를 반환합니다.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// 원격 서비스의 인터페이스.
interface ThirdPartyYouTubeLib is
    method listVideos()
    method getVideoInfo(id)
    method downloadVideo(id)

// 서비스 연결자의 구상 구현. 이 클래스의 메서드들은 유튜브에서 정보를 요청할 수
// 있습니다. 해당 요청의 속도는 사용자와 유튜브의 인터넷 연결 속도에 따라 다를
// 것입니다. 앱이 많은 요청을 동시에 처리하면 속도가 느려질 것입니다. 이는
// 요청들이 모두 같은 정보를 요청하더라도 마찬가지입니다.
class ThirdPartyYouTubeClass implements ThirdPartyYouTubeLib is
    method listVideos() is
        // 유튜브에 API 요청을 보냅니다.

    method getVideoInfo(id) is
        // 어떤 비디오에 대한 메타데이터를 가져옵니다.

    method downloadVideo(id) is
        // 유튜브에서 동영상 파일을 다운로드합니다.

// 일부 대역폭을 절약하기 위해 요청 결과를 캐시하고 일정 기간 보관할 수 있습니다.
// 그러나 이러한 코드를 서비스 클래스에 직접 넣는 것은 불가능할 수 있습니다. 예를
// 들어, 타사 라이브러리의 일부로 제공되었거나 `final`로 정의된 경우에는 말이죠.
// 서비스 클래스와 같은 인터페이스를 구현하는 새 프록시 클래스에 캐싱 코드를 넣는
// 이유가 바로 그 때문입니다. 이 클래스는 실제 요청을 보내야 하는 경우에만 서비스
// 객체에 위임합니다.
class CachedYouTubeClass implements ThirdPartyYouTubeLib is
    private field service: ThirdPartyYouTubeLib
    private field listCache, videoCache
    field needReset

    constructor CachedYouTubeClass(service: ThirdPartyYouTubeLib) is
        this.service = service

    method listVideos() is
        if (listCache == null || needReset)
            listCache = service.listVideos()
        return listCache

    method getVideoInfo(id) is
        if (videoCache == null || needReset)
            videoCache = service.getVideoInfo(id)
        return videoCache

    method downloadVideo(id) is
        if (!downloadExists(id) || needReset)
            service.downloadVideo(id)

// 서비스 객체와 직접 작업하던 그래픽 사용자 인터페이스 클래스는 서비스 객체와
// 인터페이스를 통해 작업하는 한 변경되지 않습니다. 둘 다 같은 인터페이스를
// 구현하므로 실제 서비스 객체 대신 프록시 객체를 안전하게 전달할 수 있습니다.
class YouTubeManager is
    protected field service: ThirdPartyYouTubeLib

    constructor YouTubeManager(service: ThirdPartyYouTubeLib) is
        this.service = service

    method renderVideoPage(id) is
        info = service.getVideoInfo(id)
        // 비디오 페이지를 렌더링하세요.

    method renderListPanel() is
        list = service.listVideos()
        // 비디오 섬네일 리스트를 렌더링하세요.

    method reactOnUserInput() is
        renderVideoPage()
        renderListPanel()

// 앱은 언제든지 프록시를 설정할 수 있습니다.
class Application is
    method init() is
        aYouTubeService = new ThirdPartyYouTubeClass()
        aYouTubeProxy = new CachedYouTubeClass(aYouTubeService)
        manager = new YouTubeManager(aYouTubeProxy)
        manager.reactOnUserInput()</code></pre>
</figure>
:::

:::::::::::::::: {.section .applicability-container}
##  적용 {#applicability}

::::::::::::::: applicability
프록시 패턴을 활용하는 방법들은 수십 가지가 있으며, 패턴이 가장 많이
사용되는 용도들을 살펴보겠습니다.

::: applicability-problem
지연된 초기화​(가상 프록시). 이것은 어쩌다 필요한 무거운 서비스 객체가
항상 가동되어 있어 시스템 자원들을 낭비하는 경우입니다.
:::

::: applicability-solution
앱이 시작될 때 객체를 생성하는 대신, 객체 초기화를 실제로 초기화가
필요한 시점까지 지연할 수 있습니다.
:::

::: applicability-problem
접근 제어 (보호 프록시). 당신이 특정 클라이언트들만 서비스 객체를 사용할
수 있도록 하려는 경우에 사용할 수 있습니다. 예를 들어 당신의 객체들이
운영 체제의 중요한 부분이고 클라이언트들이 다양한 실행된 응용
프로그램​(악의적인 응용 프로그램 포함)​인 경우입니다.
:::

::: applicability-solution
이 프록시는 클라이언트의 자격 증명이 어떤 정해진 기준과 일치하는
경우에만 서비스 객체에 요청을 전달할 수 있습니다.
:::

::: applicability-problem
원격 서비스의 로컬 실행 (원격 프록시). 서비스 객체가 원격 서버에 있는
경우입니다.
:::

::: applicability-solution
이 경우 프록시는 네트워크를 통해 클라이언트 요청을 전달하여 네트워크와의
작업의 모든 복잡한 세부 사항을 처리합니다.
:::

::: applicability-problem
요청들의 로깅​(로깅 프록시). 서비스 객체에 대한 요청들의 기록을
유지하려는 경우입니다.
:::

::: applicability-solution
프록시는 각 요청을 서비스에 전달하기 전에 로깅​(기록)​할 수 있습니다.
:::

::: applicability-problem
요청 결과들의 캐싱​(캐싱 프록시). 이것은 클라이언트 요청들의 결과들을
캐시하고 이 캐시들의 수명 주기를 관리해야 할 때, 특히 결과들이 상당히 큰
경우에 사용됩니다.
:::

::: applicability-solution
프록시는 항상 같은 결과를 생성하는 반복 요청들에 대해 캐싱을 구현할 수
있습니다. 프록시는 요청들의 매개변수들을 캐시 키들로 사용할 수 있습니다.
:::

::: applicability-problem
스마트 참조. 이것은 사용하는 클라이언트들이 없어 거대한 객체를 해제할 수
있어야 할 때 사용됩니다.
:::

::: applicability-solution
프록시는 서비스 객체 또는 그 결과에 대한 참조를 얻은 클라이언트들을
추적할 수 있습니다. 때때로 프록시는 클라이언트들을 점검하여
클라이언트들이 여전히 활성 상태인지를 확인할 수 있습니다. 클라이언트
리스트가 비어 있으면 프록시는 해당 서비스 객체를 닫고 그에 해당하는
시스템 자원을 확보할 수 있습니다.

또 프록시는 클라이언트가 서비스 객체를 수정했는지도 추적할 수 있으며,
변경되지 않은 객체는 다른 클라이언트들이 재사용할 수 있습니다.
:::
:::::::::::::::
::::::::::::::::

::: {.section .checklist}
##  구현방법 {#checklist}

1.  기존 서비스 인터페이스가 없는 경우, 서비스 인터페이스를 하나
    생성하여 프록시와 서비스 객체 간의 상호 교환을 가능하게 만드세요.
    서비스 클래스에서 인터페이스를 추출하는 것이 항상 가능한 것은
    아닙니다. 왜냐하면 그 인터페이스를 사용하려면 서비스의 모든
    클라이언트를 변경해야 하기 때문입니다. 대신 프록시를 서비스 클래스의
    자식 클래스로 만들 수 있으며, 이렇게 하면 서비스의 인터페이스를
    상속하게 할 수 있습니다.

2.  프록시 클래스를 만드세요. 이 클래스에는 서비스에 대한 참조를
    저장하기 위한 필드가 있어야 합니다. 일반적으로 프록시들은 서비스들의
    전체 수명 주기를 생성하고 관리합니다. 또 드물지만, 클라이언트가
    서비스를 프록시의 생성자에 전달하는 방식으로 서비스가 프록시에
    전달되기도 합니다.

3.  목적에 따라 프록시 메서드들을 구현하세요. 대부분의 경우 프록시는
    일부 작업을 수행한 후에 그 작업을 서비스 객체에 위임해야 합니다.

4.  클라이언트가 프록시를 받을지 실제 서비스를 받을지를 결정하는 생성
    메서드를 도입하는 것을 고려하세요. 이 메서드는 프록시 클래스의
    간단한 정적 메서드이거나 완전한 팩토리 메서드일 수도 있습니다.

5.  서비스 객체에 대해 지연된 초기화 구현을 고려하세요.
:::

:::::: {.section .pros-cons}
##  장단점 {#pros-cons}

::::: row
::: col-sm-6
-    클라이언트들이 알지 못하는 상태에서 서비스 객체를 제어할 수
    있습니다.
-    클라이언트들이 신경 쓰지 않을 때 서비스 객체의 수명 주기를 관리할
    수 있습니다.
-    프록시는 서비스 객체가 준비되지 않았거나 사용할 수 없는 경우에도
    작동합니다.
-    *[개]{.cjk}[방]{.cjk}/[폐]{.cjk}[쇄]{.cjk} [원]{.cjk}[칙]{.cjk}*.
    서비스나 클라이언트들을 변경하지 않고도 새 프록시들을 도입할 수
    있습니다.
:::

::: col-sm-6
-    새로운 클래스들을 많이 도입해야 하므로 코드가 복잡해질 수 있습니다.
-    서비스의 응답이 늦어질 수 있습니다.
:::
:::::
::::::

::: {.section .relations}
##  다른 패턴과의 관계 {#relations}

-   [어댑터](adapter.html)는 다른 인터페이스를, [프록시](proxy.html)는
    같은 인터페이스를, [데코레이터](decorator.html)는 향상된
    인터페이스를 래핑된 객체에 제공합니다.

-   [퍼사드](facade.html) 패턴은 복잡한 객체 또는 시스템을 보호하고
    자체적으로 초기화한다는 점에서 [프록시](proxy.html)와 유사합니다.
    *[퍼]{.cjk}[사]{.cjk}[드]{.cjk}* 패턴과 달리
    *[프]{.cjk}[록]{.cjk}[시]{.cjk}*는 자신의 서비스 객체와 같은
    인터페이스를 가지므로 이들은 서로 상호 교환이 가능합니다.

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
프록시](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](proxy/csharp/example.html "C#으로 작성된 프록시"){.prog-lang-link}
[![C++로 작성된
프록시](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](proxy/cpp/example.html "C++로 작성된 프록시"){.prog-lang-link}
[![Go로 작성된
프록시](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](proxy/go/example.html "Go로 작성된 프록시"){.prog-lang-link}
[![자바로 작성된
프록시](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](proxy/java/example.html "자바로 작성된 프록시"){.prog-lang-link}
[![PHP로 작성된
프록시](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](proxy/php/example.html "PHP로 작성된 프록시"){.prog-lang-link}
[![파이썬으로 작성된
프록시](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](proxy/python/example.html "파이썬으로 작성된 프록시"){.prog-lang-link}
[![루비로 작성된
프록시](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](proxy/ruby/example.html "루비로 작성된 프록시"){.prog-lang-link}
[![러스트로 작성된
프록시](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](proxy/rust/example.html "러스트로 작성된 프록시"){.prog-lang-link}
[![스위프트로 작성된
프록시](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](proxy/swift/example.html "스위프트로 작성된 프록시"){.prog-lang-link}
[![타입스크립트로 작성된
프록시](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](proxy/typescript/example.html "타입스크립트로 작성된 프록시"){.prog-lang-link}
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

[행동 패턴 []{.fa .fa-arrow-right}](behavioral-patterns.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### 돌아가기

[[]{.fa .fa-arrow-left} 플라이웨이트](flyweight.html){.btn .btn-default
rel="prev"}
:::
::::::::::::::::::::::::::::::::::::::::

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
::::::::::::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::::::::::::
