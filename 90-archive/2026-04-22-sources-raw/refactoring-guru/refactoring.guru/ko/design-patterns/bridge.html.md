:::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [디자인
패턴들](../design-patterns.html){.type} / [구조
패턴](structural-patterns.html){.type}
:::

# 브리지 패턴 {#브리지-패턴 .title}

::: aka
다음 이름으로도 불립니다: [Bridge]{style="display:inline-block"}
:::

::: {.section .intent}
##  의도 {#intent}

**브리지**는 큰 클래스 또는 밀접하게 관련된 클래스들의 집합을 두 개의
개별 계층구조​(추상화 및 구현)​로 나눈 후 각각 독립적으로 개발할 수 있도록
하는 구조 디자인 패턴입니다.

<figure class="image">
<img
src="../../images/patterns/content/bridge/bridge3e59.png?id=bd543d4fb32e11647767301581a5ad54"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/bridge/bridge.png?id=bd543d4fb32e11647767301581a5ad54 640w,/images/patterns/content/bridge/bridge-1.5x.png?id=8ef07b78d61cc3830b7e2b783c76c775 960w,/images/patterns/content/bridge/bridge-2x.png?id=1e905ae5742e5cd10a7eb0e3175ef00d 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="브리지 디자인 패턴" />
</figure>
:::

::: {.section .problem}
##  문제 {#problem}

*[추]{.cjk}[상]{.cjk}[화]{.cjk}?* *[구]{.cjk}[현]{.cjk}?* 어렵게
들리시나요? 진정하세요. 그리고 간단한 예시를 한번 살펴봅시다.

`Circle`​(원) 및 `Square`​(직사각형)​라는 한 쌍의 자식 클래스들이 있는
기하학적 `Shape`​(모양) 클래스가 있다고 가정해 봅시다. 이 클래스 계층
구조를 확장하여 색상을 도입하기 위해 `Red`​(빨간색) 및 `Blue`​(파란색)
모양들의 자식 클래스들을 만들 계획입니다. 그러나 이미 두 개의 자식
클래스가 있으므로 `Blue­Circle`​(파란색 원) 및 `Red­Square`​(빨간색
직사각형)​와 같은 네 가지의 클래스 조합을 만들어야 합니다.

<figure class="image">
<img
src="../../images/patterns/diagrams/bridge/problem-ko85aa.png?id=d6a269defe845caae2181e1568256f16"
style="aspect-ratio: 1.41;"
srcset="/images/patterns/diagrams/bridge/problem-ko.png?id=d6a269defe845caae2181e1568256f16 480w,/images/patterns/diagrams/bridge/problem-ko-1.5x.png?id=a6cf5feedb56c19bfb7bb4f0b9ff06af 720w,/images/patterns/diagrams/bridge/problem-ko-2x.png?id=23b2555eb5caecc9de3404d91e8ce513 960w"
sizes="(max-width: 720px) 100vw, 480px" loading="lazy" width="480"
alt="브리지 패턴 문제" />
<figcaption><p>클래스 조합들의 수는
기하급수적으로 증가합니다.</p></figcaption>
</figure>

새로운 모양 유형들과 색상 유형들을 추가할 때마다 계층 구조는
기하급수적으로 성장합니다. 예를 들어, 삼각형 모양을 추가하려면 각
색상별로 하나씩 두 개의 자식 클래스들을 도입해야 합니다. 그리고 그 후에
또 새 색상을 추가하려면 각 모양 유형별로 하나씩 세 개의 자식 클래스를
만들어야 합니다. 유형들이 많아지면 많아질수록 코드는 점점 복잡해집니다.
:::

::: {.section .solution}
##  해결책 {#solution}

이 문제는 모양과 색상의 두 가지 독립적인 차원에서 모양 클래스들을
확장하려고 하기 때문에 발생합니다. 이것은 클래스 상속과 관련된 매우
일반적인 문제입니다.

브리지 패턴은 상속에서 객체 합성으로 전환하여 이 문제를 해결하려고
시도합니다. 이것이 의미하는 바는 차원 중 하나를 별도의 클래스 계층구조로
추출하여 원래 클래스들이 한 클래스 내에서 모든 상태와 행동들을 갖는 대신
새 계층구조의 객체를 참조하도록 한다는 것입니다.

<figure class="image">
<img
src="../../images/patterns/diagrams/bridge/solution-ko3d6e.png?id=5e4d726b4474819783501d71f7ce96f9"
style="aspect-ratio: 2.3;"
srcset="/images/patterns/diagrams/bridge/solution-ko.png?id=5e4d726b4474819783501d71f7ce96f9 460w,/images/patterns/diagrams/bridge/solution-ko-1.5x.png?id=fd7777928f8d8417971031e551124ad7 690w,/images/patterns/diagrams/bridge/solution-ko-2x.png?id=6b9f6d4c51c2c8d07d95cc4553816203 920w"
sizes="(max-width: 720px) 100vw, 460px" loading="lazy" width="460"
alt="브리지 패턴이 제시하는 해결책" />
<figcaption><p>클래스 계층구조의 기하급수적인 성장을 방지하기 위하여
그것을 여러 관련 계층구조들로 변환할 수 있습니다.</p></figcaption>
</figure>

이 접근 방식을 따르면, 색상 관련 코드를 `Red` 및 `Blue`라는 두 개의 자식
클래스들이 있는 자체 클래스로 추출할 수 있습니다. 그런 다음 `Shape`
클래스는 색상 객체들 중 하나를 가리키는 참조 필드를 받습니다. 이제
모양은 연결된 색상 객체에 모든 색상 관련 작업을 위임할 수 있습니다. 이
참조는 `Shape` 및 `Color` 클래스들 사이의 브리지​(다리) 역할을 할
것입니다. 이제부터 새 색상들을 추가할 때 모양 계층구조를 변경할 필요가
없으며 그 반대의 경우도 마찬가지입니다.

#### 추상화와 구현

GoF의 디자인 패턴 []{.pop-i}[\'Gang of Four\'(사인조)​는 디자인 패턴에
관한 1994년 원조 책의 4명의 저자에게 주어진 별명입니다: [
]{.chpule1}[『]{.chpuri1}[GoF의 디자인 패턴: 재사용성을 지닌 객체지향
소프트웨어의 핵심
요소](http://www.kyobobook.co.kr/product/detailViewKor.laf?mallGb=KOR&ejkGb=KOR&barcode=9788945072146)[』]{.chpule2}[
]{.chpuri2}]{.pop-content}은 브리지 패턴 정의의 일부로
*[추]{.cjk}[상]{.cjk}[화]{.cjk}* 및 *[구]{.cjk}[현]{.cjk}*이라는
용어들을 소개합니다. 저는 위 용어들이 너무 학문적이라 그로 인해 패턴이
실제보다 더 복잡하게 들린다고 생각합니다. 그러면 GoF의 책의 난해한
용어들 뒤에 숨겨진 의미를 모양과 색상이 있는 간단한 예를 통해 해독해
보겠습니다.

*[추]{.cjk}[상]{.cjk}[화]{.cjk}*(*[인]{.cjk}[터]{.cjk}[페]{.cjk}[이]{.cjk}[스]{.cjk}*라고도
함)​는 일부 개체​(entity)​에 대한 상위 수준의 제어 레이어입니다. 이
레이어는 자체적으로 실제 작업을 수행해서는 안 되며, 작업들을
*[구]{.cjk}[현]{.cjk}* 레이어​(*[플]{.cjk}[랫]{.cjk}[폼]{.cjk}*이라고도
함)​에 위임해야 합니다.

참고로 지금 우리가 이야기하는 것은 당신이 선호하는 프로그래밍 언어의
*[인]{.cjk}[터]{.cjk}[페]{.cjk}[이]{.cjk}[스]{.cjk}[들]{.cjk}*이나
*[추]{.cjk}[상]{.cjk} [클]{.cjk}[래]{.cjk}[스]{.cjk}[들]{.cjk}*이
아닙니다. 이것들은 같은 것이 아닙니다.

실제 앱을 예로 들면 추상화는 그래픽 사용자 인터페이스이며 구현은 그래픽
사용자 인터페이스 레이어가 사용자와 상호작용하여 그 결과로 호출하는 배경
운영 체제 코드​(API)​입니다.

일반적으로 이러한 앱은 두 가지 독립적인 방향으로 확장할 수 있습니다.

-   다른 여러 가지의 그래픽 사용자 인터페이스를 가진다 (예: 일반 고객
    또는 관리자용으로 맞춘 인터페이스들).
-   여러 다른 API들을 지원한다 (예: 맥, 리눅스 및 윈도우에서 앱을 실행할
    수 있는 API들).

최악의 경우 이 앱은 수백 개의 조건문들이 코드 전체에 다양한 API와 다양한
유형의 그래픽 사용자 인터페이스들을 연결한 거대한 스파게티 코드 그릇처럼
형성됩니다.

<figure class="image">
<img
src="../../images/patterns/content/bridge/bridge-3-ko315a.png?id=30aba648afd595b26c336f43970d9bfe"
style="aspect-ratio: 2;"
srcset="/images/patterns/content/bridge/bridge-3-ko.png?id=30aba648afd595b26c336f43970d9bfe 600w,/images/patterns/content/bridge/bridge-3-ko-1.5x.png?id=ac5a8eb6028204f96d9627124336c374 900w,/images/patterns/content/bridge/bridge-3-ko-2x.png?id=47e92c3a9f001e5e3d7a22c8e1216d09 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="모듈성 코드는 변경 관리가 훨씬 쉽습니다" />
<figcaption><p><em><span class="cjk">전</span><span
class="cjk">체</span></em>를 잘 이해해야 하므로 모놀리식 코드베이스는
간단한 변경을 하는 것조차 매우 어렵습니다. 반면에 잘 정의된 작은
모듈들을 변경하는 것이 훨씬 쉽습니다.</p></figcaption>
</figure>

당신은 특정 인터페이스-플랫폼 조합들과 관련된 코드를 별도의 클래스들로
추출하여 이 복잡함에 질서를 부여할 수 있으나, 곧 이러한 클래스들이
*[많]{.cjk}[이]{.cjk}* 있다는 것을 알게 될 것입니다. 새로운 그래픽
사용자 인터페이스를 추가하거나 다른 API를 지원하려면 점점 더 많은
클래스를 생성해야 하므로 클래스 계층구조는 기하급수적으로 성장할
것입니다.

브리지 패턴으로 이 문제를 해결해 봅시다. 브리지 패턴은 클래스들을 두
개의 계층구조로 분리하라고 제안합니다:

-   추상화: 앱의 그래픽 사용자 인터페이스 레이어.
-   구현: 운영 체제의 API.

<figure class="image">
<img
src="../../images/patterns/content/bridge/bridge-2-ko586a.png?id=46cfc1de98dbc4ede4edacea48755e6f"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/bridge/bridge-2-ko.png?id=46cfc1de98dbc4ede4edacea48755e6f 640w,/images/patterns/content/bridge/bridge-2-ko-1.5x.png?id=a02fa04e5ca65e95a250fa6e758d8d1f 960w,/images/patterns/content/bridge/bridge-2-ko-2x.png?id=73952dd12aa132149b9595f27320ccd3 1280w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="640"
alt="크로스 플랫폼 아키텍처" />
<figcaption><p>크로스 플랫폼 앱을 구성하는 방법
중 하나.</p></figcaption>
</figure>

추상화 객체는 앱의 드러나는 모습을 제어하고 연결된 구현 객체에 실제
작업들을 위임합니다. 서로 다른 구현들은 공통 인터페이스를 따르는 한 상호
호환이 가능하며, 이에 따라 같은 그래픽 사용자 인터페이스는 리눅스와
윈도우에 동시에 작동할 수 있습니다.

따라서 당신은 API 관련 클래스들을 건드리지 않고 그래픽 사용자 인터페이스
클래스들을 변경할 수 있습니다. 그리고 다른 운영 체제에 대한 지원을
추가하려면 구현 계층구조 내에 자식 클래스를 생성하기만 하면 됩니다.
:::

::::: {.section .structure-container}
##  구조 {#structure}

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/bridge/structure-koc258.png?id=3a6aa3114517cd410257fadb0754713a"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.47;"
srcset="/images/patterns/diagrams/bridge/structure-ko.png?id=3a6aa3114517cd410257fadb0754713a 560w,/images/patterns/diagrams/bridge/structure-ko-1.5x.png?id=4aed0876dfe72c07764857894ae36c84 840w,/images/patterns/diagrams/bridge/structure-ko-2x.png?id=e14227ad22486d0981c23b8709556383 1120w"
sizes="(max-width: 720px) 100vw, 560px" loading="lazy" width="560"
alt="브리지 디자인 패턴" /><img
src="../../images/patterns/diagrams/bridge/structure-ko-indexed9d88.png?id=b829023128a479e00dd63903ba10ba76"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.44;"
srcset="/images/patterns/diagrams/bridge/structure-ko-indexed.png?id=b829023128a479e00dd63903ba10ba76 560w,/images/patterns/diagrams/bridge/structure-ko-indexed-1.5x.png?id=8edd6abccd21116fbb3658c2c1d5a0c8 840w,/images/patterns/diagrams/bridge/structure-ko-indexed-2x.png?id=699ef288f0db217ac2f38c1ffa5b8df2 1120w"
sizes="(max-width: 720px) 100vw, 560px" loading="lazy" width="560"
alt="브리지 디자인 패턴" />
</figure>
:::

1.  **추상화**는 상위 수준의 제어 논리를 제공하며, 구현 객체에 의존해
    실제 하위 수준 작업들을 수행합니다.

2.  **구현**은 모든 구상 구현들에 공통적인 인터페이스를 선언하며,
    추상화는 여기에 선언된 메서드들을 통해서만 구현 객체와 소통할 수
    있습니다.

    추상화는 구현과 같은 메서드들을 나열할 수 있지만 보통은 구현이
    선언한 다양한 원시 작업들에 의존하는 몇 가지 복잡한 행동들을
    선언합니다.

3.  **구상 구현들**에는 플랫폼별 맞춤형 코드가 포함됩니다.

4.  **정제된 추상화들**은 제어 논리의 변형들을 제공합니다. 그들은 그들의
    부모처럼 일반 구현 인터페이스를 통해 다른 구현들과 작업합니다.

5.  일반적으로 **클라이언트**는 추상화와 작업하는데만 관심이 있습니다.
    그러나 추상화 객체를 구현 객체들 중 하나와 연결하는 것도
    클라이언트의 역할입니다.
::::
:::::

::: {.section .pseudocode}
##  의사코드 {#pseudocode}

이 예시는 **브리지** 패턴이 기기와 리모컨을 관리하는 앱의 모놀리식
코드를 나누는 데 어떻게 도움이 되는지 보여줍니다. `Device` 클래스들은
구현의 역할을 하는 반면, `Remote`클래스들은 추상화 역할을 합니다.

<figure class="image">
<img
src="../../images/patterns/diagrams/bridge/example-koc0c4.png?id=ce912a29894524a057d024e168c682f9"
style="aspect-ratio: 1.33;"
srcset="/images/patterns/diagrams/bridge/example-ko.png?id=ce912a29894524a057d024e168c682f9 640w,/images/patterns/diagrams/bridge/example-ko-1.5x.png?id=ab9781a5029c1135457099ae98bef754 960w,/images/patterns/diagrams/bridge/example-ko-2x.png?id=03c202d70eb36402d0d108f2a6bb7736 1280w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="640"
alt="브리지 패턴 구조 예시" />
<figcaption><p>원조 클래스의 계층구조는 두 부분으로 나뉩니다:
장치들과 리모컨들.</p></figcaption>
</figure>

기초 리모컨 클래스는 이 클래스를 장치 객체와 연결하는 참조 필드를
선언합니다. 모든 리모컨은 일반 장치 인터페이스를 통해 장치들과
작동하므로 같은 리모컨이 여러 장치 유형을 지원할 수 있습니다.

장치 클래스들과 독립적으로 리모컨 클래스들을 개발할 수 있으며, 필요한
것은 새로운 리모컨 자식 클래스를 만드는 것뿐입니다. 예를 들어 기초
리모컨에는 버튼이 두 개뿐일 수 있지만, 추가 터치스크린과 추가 배터리
같은 기능들도 가지도록 확장할 수 있습니다.

클라이언트 코드는 `Remote`의 생성자를 통해 원하는 유형의 리모컨을 특정
장치 객체와 연결합니다.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// &#39;추상화&#39;는 두 클래스 계층구조의 &#39;제어&#39; 부분에 대한 인터페이스를 정의하며,
// 이것은 &#39;구현&#39; 계층구조의 객체에 대한 참조를 유지하고 모든 실제 작업을 이
// 객체에 위임합니다.
class RemoteControl is
    protected field device: Device
    constructor RemoteControl(device: Device) is
        this.device = device
    method togglePower() is
        if (device.isEnabled()) then
            device.disable()
        else
            device.enable()
    method volumeDown() is
        device.setVolume(device.getVolume() - 10)
    method volumeUp() is
        device.setVolume(device.getVolume() + 10)
    method channelDown() is
        device.setChannel(device.getChannel() - 1)
    method channelUp() is
        device.setChannel(device.getChannel() + 1)


// 이제 추상화 계층구조로부터 클래스들을 장치 클래스들과 독립적으로 확장할 수
// 있습니다.
class AdvancedRemoteControl extends RemoteControl is
    method mute() is
        device.setVolume(0)


// &#39;구현&#39; 인터페이스는 모든 구상 구현 클래스들에 공통적인 메서드를 선언하며, 이는
// 추상화의 인터페이스와 일치할 필요가 없습니다. 실제로 두 인터페이스는 완전히 다를
// 수 있습니다. 일반적으로 구현 인터페이스는 원시​(primitive) 작업들만 제공하는
// 반면 추상화는 이러한 원시 작업들을 기반으로 더 상위 수준의 작업들을 정의합니다.
interface Device is
    method isEnabled()
    method enable()
    method disable()
    method getVolume()
    method setVolume(percent)
    method getChannel()
    method setChannel(channel)


// 모든 장치는 같은 인터페이스를 따릅니다.
class Tv implements Device is
    // …

class Radio implements Device is
    // …


// 클라이언트 코드 어딘가에…
tv = new Tv()
remote = new RemoteControl(tv)
remote.togglePower()

radio = new Radio()
remote = new AdvancedRemoteControl(radio)</code></pre>
</figure>
:::

:::::::::: {.section .applicability-container}
##  적용 {#applicability}

::::::::: applicability
::: applicability-problem
브리지 패턴은 당신이 어떤 기능의 여러 변형을 가진 모놀리식 클래스를
나누고 정돈하려 할 때 사용하세요. (예: 클래스가 다양한 데이터베이스
서버들과 작동할 수 있는 경우).
:::

::: applicability-solution
클래스가 성장할수록 그 작동 방식을 파악하기가 더 어려워지고 해당
클래스를 변경하는 데 더더욱 오랜 시간이 걸립니다. 클래스 기능의 여러
변형 중 하나를 변경하려면 클래스 전체에 걸쳐 여러 가지 변경을 수행해야
할 수 있으며, 이를 수행 중 개발자들은 종종 실수하거나 일부 중요한
부작용들을 해결하지 않기도 합니다.

브리지 패턴을 사용하면 모놀리식 클래스를 여러 클래스 계층구조로 나눌 수
있습니다. 그런 다음 각 계층구조의 클래스들을 다른 계층구조들에 있는
클래스들과는 독립적으로 변경할 수 있습니다. 이 접근 방식은 코드의
유지관리를 단순화하고 기존 코드가 손상될 위험을 최소화합니다.
:::

::: applicability-problem
이 패턴은 여러 직교​(독립) 차원에서 클래스를 확장해야 할 때 사용하세요.
:::

::: applicability-solution
브리지 패턴은 각 차원에 대해 별도의 클래스 계층구조를 추출할 것을
제안합니다. 원래 클래스는 모든 작업을 자체적으로 수행하는 대신 추출된
계층구조들에 속한 객체들에 관련 작업들을 위임합니다.
:::

::: applicability-problem
브리지 패턴은 런타임​(실행시간)​에 구현을 전환할 수 있어야 할 때에
사용하세요.
:::

::: applicability-solution
선택 사항이지만 브리지 패턴을 사용하면 추상화 내부의 구현 객체를 바꿀 수
있으며, 그렇게 하려면 필드에 새 값을 할당하기만 하면 됩니다.

위 항목은 많은 사람이 브리지와 [전략](strategy.html) 패턴을 혼동하는
주된 이유입니다. 패턴은 클래스의 구조를 설계하는 특정 방법 그 이상의
것이라는 것을 기억하세요. 패턴은 제기되고 있는 문제 및 의도에 대하여도
소통할 수 있습니다.
:::
:::::::::
::::::::::

::: {.section .checklist}
##  구현방법 {#checklist}

1.  클래스에서 직교 차원들을 식별하세요. 이러한 독립적인 개념들은
    추상화/플랫폼, 도메인/인프라, 프런트엔드/백엔드 또는 인터페이스/구현
    등일 수 있습니다.

2.  클라이언트가 필요로 하는 작업들을 확인한 후 기초 추상 클래스에서
    정의하세요.

3.  모든 플랫폼들에 제공되어야 하는 작업들을 결정하세요. 그 후 추상화에
    필요한 작업들을 일반 구현 인터페이스에서 선언하세요.

4.  도메인의 모든 플랫폼에 대해 구상 구현 클래스들을 생성하되 이
    클래스들 모두가 구현 인터페이스를 따르도록 하세요.

5.  추상화 클래스 내에서 구현 유형에 대한 참조 필드를 추가하세요.
    추상화는 대부분 작업들을 위 필드에서 참조되는 구현 객체에
    위임합니다.

6.  상위 수준 논리의 변형들이 여러 개 있는 경우 기초 추상화 클래스를
    확장하여 각 변형에 대해 정제된 추상화들을 만드세요.

7.  클라이언트 코드는 구현 객체를 추상화의 생성자에 전달하여 이 객체를
    그 생성자에 연관시켜야 합니다. 그 후에 클라이언트는 구현을 잊어버린
    후 추상화 객체와만 작업할 수 있습니다.
:::

:::::: {.section .pros-cons}
##  장단점 {#pros-cons}

::::: row
::: col-sm-6
-    플랫폼 독립적인 클래스들과 앱들을 만들 수 있습니다.
-    클라이언트 코드는 상위 수준의 추상화를 통해 작동하며, 플랫폼 세부
    정보에 노출되지 않습니다.
-    *[개]{.cjk}[방]{.cjk}/[폐]{.cjk}[쇄]{.cjk} [원]{.cjk}[칙]{.cjk}*.
    새로운 추상화들과 구현들을 상호 독립적으로 도입할 수 있습니다.
-    *[단]{.cjk}[일]{.cjk} [책]{.cjk}[임]{.cjk} [원]{.cjk}[칙]{.cjk}*.
    추상화의 상위 수준 논리와 구현의 플랫폼 세부 정보에 집중할 수
    있습니다.
:::

::: col-sm-6
-    결합도가 높은 클래스에 패턴을 적용하여 코드를 더 복잡하게 만들 수
    있습니다.
:::
:::::
::::::

::: {.section .relations}
##  다른 패턴과의 관계 {#relations}

-   [브리지](bridge.html)는 일반적으로 사전에 설계되며, 앱의 다양한
    부분을 독립적으로 개발할 수 있도록 합니다. 반면에
    [어댑터](adapter.html)는 일반적으로 기존 앱과 사용되어 원래 호환되지
    않던 일부 클래스들이 서로 잘 작동하도록 합니다.

-   [브리지](bridge.html), [상태](state.html), [전략](strategy.html)
    패턴은 매우 유사한 구조로 되어 있으며, [어댑터](adapter.html) 패턴도
    이들과 어느 정도 유사한 구조로 되어 있습니다. 위 모든 패턴은 다른
    객체에 작업을 위임하는 합성을 기반으로 합니다. 하지만 이 패턴들은
    모두 다른 문제들을 해결합니다. 패턴은 특정 방식으로 코드의 구조를
    짜는 레시피에 불과하지 않습니다. 왜냐하면 패턴은 해결하는 문제를
    다른 개발자들에게 전달할 수도 있기 때문입니다.

-   당신은 [추상 팩토리](abstract-factory.html)를
    [브리지](bridge.html)와 함께 사용할 수 있습니다. 이 조합은
    *[브]{.cjk}[리]{.cjk}[지]{.cjk}*에 의해 정의된 어떤 추상화들이 특정
    구현들과만 작동할 수 있을 때 유용합니다. 이런 경우에
    *[추]{.cjk}[상]{.cjk} [팩]{.cjk}[토]{.cjk}[리]{.cjk}*는 이러한
    관계들을 캡슐화하고 클라이언트 코드에서부터 복잡성을 숨길 수
    있습니다.

-   [빌더](builder.html)를 [브리지](bridge.html)와 조합할 수 있습니다.
    디렉터 클래스는 추상화의 역할을 하고 다양한 빌더들은 구현의 역할을
    합니다.
:::

::: {.section .implementations}
##  코드 예시 {#implementations}

[![C#으로 작성된
브리지](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](bridge/csharp/example.html "C#으로 작성된 브리지"){.prog-lang-link}
[![C++로 작성된
브리지](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](bridge/cpp/example.html "C++로 작성된 브리지"){.prog-lang-link}
[![Go로 작성된
브리지](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](bridge/go/example.html "Go로 작성된 브리지"){.prog-lang-link}
[![자바로 작성된
브리지](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](bridge/java/example.html "자바로 작성된 브리지"){.prog-lang-link}
[![PHP로 작성된
브리지](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](bridge/php/example.html "PHP로 작성된 브리지"){.prog-lang-link}
[![파이썬으로 작성된
브리지](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](bridge/python/example.html "파이썬으로 작성된 브리지"){.prog-lang-link}
[![루비로 작성된
브리지](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](bridge/ruby/example.html "루비로 작성된 브리지"){.prog-lang-link}
[![러스트로 작성된
브리지](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](bridge/rust/example.html "러스트로 작성된 브리지"){.prog-lang-link}
[![스위프트로 작성된
브리지](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](bridge/swift/example.html "스위프트로 작성된 브리지"){.prog-lang-link}
[![타입스크립트로 작성된
브리지](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](bridge/typescript/example.html "타입스크립트로 작성된 브리지"){.prog-lang-link}
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

[복합체 []{.fa .fa-arrow-right}](composite.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### 돌아가기

[[]{.fa .fa-arrow-left} 어댑터](adapter.html){.btn .btn-default
rel="prev"}
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
