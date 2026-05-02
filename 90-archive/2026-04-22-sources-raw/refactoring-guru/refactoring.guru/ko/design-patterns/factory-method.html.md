::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [디자인
패턴들](../design-patterns.html){.type} / [생성
패턴](creational-patterns.html){.type}
:::

# 팩토리 메서드 패턴 {#팩토리-메서드-패턴 .title}

::: aka
다음 이름으로도 불립니다: [가상
생성자, ]{style="display:inline-block"}[Factory
Method]{style="display:inline-block"}
:::

::: {.section .intent}
##  의도 {#intent}

**팩토리 메서드**는 부모 클래스에서 객체들을 생성할 수 있는 인터페이스를
제공하지만, 자식 클래스들이 생성될 객체들의 유형을 변경할 수 있도록 하는
생성 패턴입니다.

<figure class="image">
<img
src="../../images/patterns/content/factory-method/factory-method-ko2330.png?id=33dd023badd809f01d88f29cf6be43c6"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/factory-method/factory-method-ko.png?id=33dd023badd809f01d88f29cf6be43c6 640w,/images/patterns/content/factory-method/factory-method-ko-1.5x.png?id=9e656b625cf3b10d73a3722e7ef8ac53 960w,/images/patterns/content/factory-method/factory-method-ko-2x.png?id=a0696e3e89b1266b02b7bae93f68c26d 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="팩토리 메서드 패턴" />
</figure>
:::

::: {.section .problem}
##  문제 {#problem}

당신이 물류 관리 앱을 개발하고 있다고 가정합시다. 앱의 첫 번째 버전은
트럭 운송만 처리할 수 있어서 대부분의 코드가 `Truck`​(트럭) 클래스에
있습니다.

또 얼마 후 당신의 앱이 유명해졌으며, 매일 해상 물류 회사들로부터 해상
물류 기능을 앱에 추가해 달라는 요청을 수십 개씩 받기 시작했다고 가정해
봅시다.

<figure class="image">
<img
src="../../images/patterns/diagrams/factory-method/problem1-ko923b.png?id=d54af258065966bbd1afc0329adc5ecc"
style="aspect-ratio: 2.4;"
srcset="/images/patterns/diagrams/factory-method/problem1-ko.png?id=d54af258065966bbd1afc0329adc5ecc 600w,/images/patterns/diagrams/factory-method/problem1-ko-1.5x.png?id=5b9efb16cf7b0721e891bd7ef7e8d784 900w,/images/patterns/diagrams/factory-method/problem1-ko-2x.png?id=85dd10c9b1abfc08a09be5e4090efc09 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="프로그램에 새 운송 클래스를 추가하면 문제가 발생합니다" />
<figcaption><p>나머지 코드가 이미 기존 클래스들에 결합되어 있다면
프로그램에 새 클래스를 추가하는 일은 그리
간단하지 않습니다.</p></figcaption>
</figure>

좋은 소식이죠? 그러나 현재 대부분의 코드는 `Truck` 클래스에 결합되어
있습니다. 앱에 `Ship`​(선박) 클래스를 추가하려면 전체 코드 베이스를
변경해야 합니다. 또한 차후 앱에 다른 유형의 교통수단을 추가하려면 아마도
다시 전체 코드 베이스를 변경해야 할 것입니다.

그러면 결과적으로 많은 조건문이 운송 수단 객체들의 클래스에 따라 앱의
행동을 바꾸는 매우 복잡한 코드가 작성될 것입니다.
:::

::: {.section .solution}
##  해결책 {#solution}

팩토리 메서드 패턴은 (`new` 연산자를 사용한) 객체 생성 직접 호출들을
특별한 *[팩]{.cjk}[토]{.cjk}[리]{.cjk}* 메서드에 대한 호출들로
대체하라고 제안합니다. 걱정하지 마세요: 객체들은 여전히 `new` 연산자를
통해 생성되지만 팩토리 메서드 내에서 호출되고 있습니다. 참고로 팩토리
메서드에서 반환된 객체는 종종 *[제]{.cjk}[품]{.cjk}*이라고도 불립니다.

<figure class="image">
<img
src="../../images/patterns/diagrams/factory-method/solution16ec2.png?id=fc756d2af296b5b4d482e548214d08ef"
style="aspect-ratio: 2.3;"
srcset="/images/patterns/diagrams/factory-method/solution1.png?id=fc756d2af296b5b4d482e548214d08ef 620w,/images/patterns/diagrams/factory-method/solution1-1.5x.png?id=22d3b6bb74e966d1cb3a4d8f380cefa3 930w,/images/patterns/diagrams/factory-method/solution1-2x.png?id=c482b3cd7a4d8dd73b4c8c12dfcaa03c 1240w"
sizes="(max-width: 720px) 100vw, 620px" loading="lazy" width="620"
alt="creator(크리에이터) 클래스들의 구조" />
<figcaption><p>자식 클래스들은 팩토리 메서드가 반환하는 객체들의
클래스를 변경할 수 있습니다.</p></figcaption>
</figure>

얼핏 이러한 변경은 무의미해 보일 수도 있는데, 그 이유는 생성자 호출을
프로그램의 한 부분에서 다른 부분으로 옮겼을 뿐이기 때문입니다. 그러나
위와 같은 변경 덕분에 이제 자식 클래스에서 팩토리 메서드를
오버라이딩하고 그 메서드에 의해 생성되는 제품들의 클래스를 변경할 수
있게 되었습니다.

하지만 약간의 제한이 있긴 합니다. 자식 클래스들은 다른 유형의 제품들을
해당 제품들이 공통 기초 클래스 또는 공통 인터페이스가 있는 경우에만
반환할 수 있습니다. 또 이전에 언급한 모든 제품들에 공통인 `Transport`
인터페이스로 `Logistics` 기초 클래스의 `create­Transport` 팩토리 메서드의
반환 유형을 선언해야 합니다.

<figure class="image">
<img
src="../../images/patterns/diagrams/factory-method/solution2-ko453e.png?id=36e26bcbf4edb396e547573c2d56d894"
style="aspect-ratio: 1.96;"
srcset="/images/patterns/diagrams/factory-method/solution2-ko.png?id=36e26bcbf4edb396e547573c2d56d894 490w,/images/patterns/diagrams/factory-method/solution2-ko-1.5x.png?id=44bf449d4cc0f83eece68f59444e548c 735w,/images/patterns/diagrams/factory-method/solution2-ko-2x.png?id=94374a51d9634cbb74043082e2cf7a2b 980w"
sizes="(max-width: 720px) 100vw, 490px" loading="lazy" width="490"
alt="제품 계층구조의 구조" />
<figcaption><p>모든 제품들은 같은 인터페이스를
따라야 합니다.</p></figcaption>
</figure>

예를 들어 `Truck` 과 `Ship` 클래스들은 모두 `Transport` 인터페이스를
구현해야 하며, 이 인터페이스는 `deliver`​(배달)​라는 메서드를 선언합니다.
그러나 각 클래스는 이 메서드를 다르게 구현합니다. 트럭은 육로로 화물을
배달하고 선박은 해상으로 화물을 배달합니다. `Road­Logistics`​(도로 물류)
클래스에 포함된 팩토리 메서드는 `Truck` 객체들을 반환하는 반면
`Sea­Logistics`​(해운 물류) 클래스에 포함된 팩토리 메서드는 선박 객체들을
반환합니다.

<figure class="image">
<img
src="../../images/patterns/diagrams/factory-method/solution3-kob8e1.png?id=bae3e95fdeaaf8b3dd34466a7d18cc2b"
style="aspect-ratio: 1.83;"
srcset="/images/patterns/diagrams/factory-method/solution3-ko.png?id=bae3e95fdeaaf8b3dd34466a7d18cc2b 640w,/images/patterns/diagrams/factory-method/solution3-ko-1.5x.png?id=99fdbd9d1b61191e5f56aac9fd868d0b 960w,/images/patterns/diagrams/factory-method/solution3-ko-2x.png?id=c43f3fbdfbaaf415ce8eafa3cac34ce7 1280w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="640"
alt="팩토리 메서드 패턴을 적용한 코드의 구조" />
<figcaption><p>모든 제품 클래스들이 공통 인터페이스를 구현하는 한, 제품
클래스들의 객체들을 손상하지 않고 클라이언트 코드를 통과시킬
수 있습니다.</p></figcaption>
</figure>

팩토리 메서드를 사용하는 코드를 종종
*[클]{.cjk}[라]{.cjk}[이]{.cjk}[언]{.cjk}[트]{.cjk}* 코드라고 부르며,
클라이언트 코드는 다양한 자식 클래스들에서 실제로 반환되는 여러 제품
간의 차이에 대해 알지 못합니다. 클라이언트 코드는 모든 제품을 추상
`Transport`​(운송체계)​로 간주합니다. 클라이언트는 모든 `Transport`
객체들이 `deliver`​(배달) 메서드를 가져야 한다는 사실을 잘 알고 있지만,
이 메서드가 정확히 어떻게 작동하는지는 클라이언트에게 중요하지 않습니다.
:::

::::: {.section .structure-container}
##  구조 {#structure}

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/factory-method/structure78c3.png?id=4cba0803f42517cfe8548c9bc7dc4c9b"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.74;"
srcset="/images/patterns/diagrams/factory-method/structure.png?id=4cba0803f42517cfe8548c9bc7dc4c9b 660w,/images/patterns/diagrams/factory-method/structure-1.5x.png?id=ece8300890afc14494770a6b6d370428 990w,/images/patterns/diagrams/factory-method/structure-2x.png?id=9ea3aa8a47f8be22e12e523c15b448fd 1320w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="660"
alt="팩토리 메서드 패턴 구조" /><img
src="../../images/patterns/diagrams/factory-method/structure-indexed6136.png?id=4c603207859ca1f939b17b60a3a2e9e0"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.74;"
srcset="/images/patterns/diagrams/factory-method/structure-indexed.png?id=4c603207859ca1f939b17b60a3a2e9e0 660w,/images/patterns/diagrams/factory-method/structure-indexed-1.5x.png?id=626b6d7f577e1c265cce244678b2cf7f 990w,/images/patterns/diagrams/factory-method/structure-indexed-2x.png?id=c794e4f2d05013fb176464a1d1a5d7ab 1320w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="660"
alt="팩토리 메서드 패턴 구조" />
</figure>
:::

1.  **제품**은 인터페이스를 선언합니다. 인터페이스는 생성자와 자식
    클래스들이 생성할 수 있는 모든 객체에 공통입니다.

2.  **구상 제품들**은 제품 인터페이스의 다양한 구현들입니다.

3.  **크리에이터​(Creator)** 클래스는 새로운 제품 객체들을 반환하는
    팩토리 메서드를 선언합니다. 중요한 점은 이 팩토리 메서드의 반환
    유형이 제품 인터페이스와 일치해야 한다는 것입니다.

    당신은 팩토리 메서드를 `abstract`​(추상)​로 선언하여 모든 자식
    클래스들이 각각 이 메서드의 자체 버전들을 구현하도록 강제할 수
    있으며, 또 대안적으로 기초 팩토리 메서드가 디폴트​(기본값) 제품
    유형을 반환하도록 만들 수도 있습니다.

    크리에이터라는 이름에도 불구하고 크리에이터의 주책임은 제품을
    생성하는 것이 **아닙니다.** 일반적으로 크리에이터 클래스에는 이미
    제품과 관련된 핵심 비즈니스 로직이 있으며, 팩토리 메서드는 이 로직을
    구상 제품 클래스들로부터 디커플링​(분리) 하는 데 도움을 줄 뿐입니다.
    대규모 소프트웨어 개발사에 비유해 보자면, 이 회사는 프로그래머들을
    위한 교육 부서가 있을 수 있으나, 회사의 주 임무는 프로그래머를
    교육하는 것이 아니라 코드를 작성하는 것입니다.

4.  **구상 크리에이터**들은 기초 팩토리 메서드를 오버라이드​(재정의)​하여
    다른 유형의 제품을 반환하게 하도록 합니다.

    참고로 팩토리 메서드는 항상 새로운 인스턴스들을 **생성**해야 할
    필요가 없습니다. 팩토리 메서드는 기존 객체들을 캐시, 객체 풀 또는
    다른 소스로부터 반환할 수 있습니다.
::::
:::::

::: {.section .pseudocode}
##  의사코드 {#pseudocode}

아래 예시는 어떻게 **팩토리 메서드**가 클라이언트 코드를 구상 UI
클래스들과 결합하지 않고도 크로스 플랫폼 UI 요소들을 생성할 수 있는지를
보여줍니다.

<figure class="image">
<img
src="../../images/patterns/diagrams/factory-method/examplec59f.png?id=67db9a5cb817913444efcb1c067c9835"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/diagrams/factory-method/example.png?id=67db9a5cb817913444efcb1c067c9835 640w,/images/patterns/diagrams/factory-method/example-1.5x.png?id=921d97276e5e2fd8e64609c9cfa021e7 960w,/images/patterns/diagrams/factory-method/example-2x.png?id=a2470830778e318263155000dbdc5870 1280w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="640"
alt="팩토리 메서드 패턴 구조 예시" />
<figcaption><p>크로스 플랫폼 다이얼로그​(대화
상자) 예시.</p></figcaption>
</figure>

기초 `Dialog`​(대화 상자) 클래스는 여러 UI 요소들을 사용하여 대화 상자를
렌더링합니다. 다양한 운영 체제에서 이러한 요소들은 약간씩 다르게 보일 수
있지만 여전히 일관되게 작동해야 합니다. 예를 들어 윈도우에서의 버튼은
리눅스에서도 여전히 버튼이어야 합니다.

팩토리 메서드가 적용되면, 당신은 대화 상자 로직을 각 운영 체제를 위하여
반복해서 재작성할 필요가 없습니다. 기초 `Dialog` 클래스 내에서 버튼을
생성하는 팩토리 메서드를 선언하면 나중에 팩토리 메서드에서 윈도우 유형의
버튼들을 반환하는 `Dialog` 자식 클래스를 생성할 수 있습니다. 그 후 이
자식 클래스는 기초 클래스로부터 `Dialog`의 코드 대부분을 상속받으나,
팩토리 메서드 덕분에 윈도우 유형의 버튼들도 렌더링할 수 있습니다.

이 패턴이 작동하려면 기초 `Dialog` 클래스가 추상 버튼들과 함께 작동해야
합니다. (참고로 추상 버튼은 모든 구상 버튼들이 따르는 인터페이스 또는
기초 클래스입니다). 이렇게 해야 다이얼로그 코드가 버튼 유형에 관계없이
작동합니다.

물론, 위 접근 방식을 다른 UI 요소들에도 적용할 수 있으나, 대화 상자에
새로운 팩토리 메서드를 추가할 때마다 이 프로그램은 [추상
팩토리](abstract-factory.html) 패턴에 더 가까워집니다. 걱정하지 마세요.
추상 팩토리 패턴에 대해서는 나중에 설명하겠습니다.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// 크리에이터 클래스는 제품 클래스의 객체를 반환해야 하는 팩토리 메서드를
// 선언합니다. 크리에이터의 자식 클래스들은 일반적으로 이 메서드의 구현을
// 제공합니다.
class Dialog is
    // 크리에이터는 팩토리 메서드의 일부 디폴트 구현을 제공할 수도 있습니다.
    abstract method createButton():Button

    // 크리에이터의 주 업무는 제품을 생성하는 것이 아닙니다. 크리에이터는
    // 일반적으로 팩토리 메서드에서 반환된 제품 객체에 의존하는 어떤 핵심
    // 비즈니스 로직을 포함합니다. 자식 클래스들은 팩토리 메서드를 오버라이드 한
    // 후 해당 메서드에서 다른 유형의 제품을 반환하여 해당 비즈니스 로직을
    // 간접적으로 변경할 수 있습니다.
    method render() is
        // 팩토리 메서드를 호출하여 제품 객체를 생성하세요.
        Button okButton = createButton()
        // 이제 제품을 사용하세요.
        okButton.onClick(closeDialog)
        okButton.render()


// 구상 크리에이터들은 결과 제품들의 유형을 변경하기 위해 팩토리 메서드를
// 오버라이드합니다.
class WindowsDialog extends Dialog is
    method createButton():Button is
        return new WindowsButton()

class WebDialog extends Dialog is
    method createButton():Button is
        return new HTMLButton()


// 제품 인터페이스는 모든 구상 제품들이 구현해야 하는 작업들을 선언합니다.
interface Button is
    method render()
    method onClick(f)

// 구상 제품들은 제품 인터페이스의 다양한 구현을 제공합니다.
class WindowsButton implements Button is
    method render(a, b) is
        // 버튼을 윈도우 스타일로 렌더링하세요.
    method onClick(f) is
        // 네이티브 운영체제 클릭 이벤트를 바인딩하세요.

class HTMLButton implements Button is
    method render(a, b) is
        // 버튼의 HTML 표현을 반환하세요.
    method onClick(f) is
        // 웹 브라우저 클릭 이벤트를 바인딩하세요.


class Application is
    field dialog: Dialog

    // 앱은 현재 설정 또는 환경 설정에 따라 크리에이터의 유형을 선택합니다.
    method initialize() is
        config = readApplicationConfigFile()

        if (config.OS == &quot;Windows&quot;) then
            dialog = new WindowsDialog()
        else if (config.OS == &quot;Web&quot;) then
            dialog = new WebDialog()
        else
            throw new Exception(&quot;Error! Unknown operating system.&quot;)

    // 클라이언트 코드는 비록 구상 크리에이터의 기초 인터페이스를 통하는 것이긴
    // 하지만 구상 크리에이터의 인스턴스와 함께 작동합니다. 클라이언트가
    // 크리에이터의 기초 인터페이스를 통해 크리에이터와 계속 작업하는 한 모든
    // 크리에이터의 자식 클래스를 클라이언트에 전달할 수 있습니다.
    method main() is
        this.initialize()
        dialog.render()</code></pre>
</figure>
:::

:::::::::: {.section .applicability-container}
##  적용 {#applicability}

::::::::: applicability
::: applicability-problem
팩토리 메서드는 당신의 코드가 함께 작동해야 하는 객체들의 정확한
유형들과 의존관계들을 미리 모르는 경우 사용하세요.
:::

::: applicability-solution
팩토리 메서드는 제품 생성 코드를 제품을 실제로 사용하는 코드와
분리합니다. 그러면 제품 생성자 코드를 나머지 코드와는 독립적으로
확장하기 쉬워집니다.

예를 들어, 앱에 새로운 제품을 추가하려면 당신은 새로운 크리에이터 자식
클래스를 생성한 후 해당 클래스 내부의 팩토리 메서드를
오버라이딩​(재정의)​하기만 하면 됩니다.
:::

::: applicability-problem
팩토리 메서드는 당신의 라이브러리 또는 프레임워크의 사용자들에게 내부
컴포넌트들을 확장하는 방법을 제공하고 싶을 때 사용하세요.
:::

::: applicability-solution
상속​(inheritance)​은 아마도 라이브러리나 프레임워크의 디폴트 행동을
확장하는 가장 쉬운 방법일 것입니다. 그러나 프레임워크는 표준 컴포넌트
대신 당신의 자식 클래스를 사용해야 한다는 것을 어떻게 인식할까요?

해결책은 일단 프레임워크 전체에서 컴포넌트들을 생성하는 코드를 단일
팩토리 메서드로 줄인 후, 누구나 이 팩토리 메서드를 오버라이드 할 수
있도록 하는 것입니다.

그러면 해결책의 예시를 한번 살펴봅시다. 당신이 오픈 소스 UI 프레임워크를
사용하여 앱을 작성하고 있고, 당신이 개발 중인 앱에는 둥근 버튼들이
필요한데 프레임워크는 사각형 버튼만 제공한다고 가정합시다. 또 표준
`Button`​(버튼) 클래스는 `Round­Button`​(둥근 버튼) 자식 클래스로
확장했지만, 이제 메인 `UIFramework`​(사용자 인터페이스 프레임워크)
클래스에 디폴트 클래스 대신 새로운 `Round­Button`​(둥근 버튼) 자식
클래스를 사용하라고 지시해야 한다고 가정해 봅시다.

이를 달성하려면 기초 프레임워크 클래스에서 자식 클래스
`UIWith­Round­Buttons`를 만들어서 기초 프레임워크 클래스의 `create­Button`
메서드를 오버라이딩​(재정의)​합니다. 이 메서드는 기초 클래스에 `Button`
객체들을 반환하지만, 당신은 당신의 자식 클래스가 `Round­Button` 객체들을
반환하도록 만듭니다. 이제 `UIFramework` 클래스 대신 `UIWith­Round­Buttons`
클래스를 사용하면 끝입니다!
:::

::: applicability-problem
팩토리 메서드는 기존 객체들을 매번 재구축하는 대신 이들을 재사용하여
시스템 리소스를 절약하고 싶을 때 사용하세요.
:::

::: applicability-solution
이러한 요구 사항은 데이터베이스 연결, 파일 시스템 및 네트워크처럼 시스템
자원을 많이 사용하는 대규모 객체들을 처리할 때 자주 발생합니다.

기존 객체를 재사용하려면 무엇을 해야 하는지 한번 생각해 봅시다.

1.  먼저 생성된 모든 객체를 추적하기 위해 일부 스토리지를 생성해야
    합니다.
2.  누군가가 객체를 요청하면 프로그램은 해당 풀 내에서 유휴​(free) 객체를
    찾아야 합니다. 그 후[...]{.chpule2}[ ]{.chpuri2}
3.  [...]{.chpule2}[ ]{.chpuri2}이 객체를 클라이언트 코드에 반환해야
    합니다.
4.  유휴​(free) 객체가 없으면, 프로그램은 새로운 객체를 생성해야 합니다.
    (그리고 풀에 이 객체를 추가해야 합니다).

이것은 정말로 많은 양의 코드입니다! 그리고 프로그램을 중복 코드로
오염시키지 않도록 이 많은 양의 코드를 모두 한곳에 넣어야 합니다.

아마도 이 코드를 배치할 수 있는 가장 확실하고 편리한 위치는 우리가
재사용하려는 객체들의 클래스의 생성자일 것입니다. 그러나 생성자는 특성상
항상 **새로운 객체들을** 반환해야 하며, 기존 인스턴트를 반환할 수는
없습니다.

따라서 새 객체들을 생성하고 기존 객체를 재사용할 수 있는 일반적인
메서드가 필요합니다. 이 설명, 꼭 팩토리 메서드처럼 들리지 않나요?
:::
:::::::::
::::::::::

::: {.section .checklist}
##  구현방법 {#checklist}

1.  모든 제품이 같은 인터페이스를 따르도록 하세요. 이 인터페이스는 모든
    제품에서 의미가 있는 메서드들을 선언해야 합니다.

2.  크리에이터 클래스 내부에 빈 팩토리 메서드를 추가하세요. 이 메서드의
    반환 유형은 공통 제품 인터페이스와 일치해야 합니다.

3.  크리에이터의 코드에서 제품 생성자들에 대한 모든 참조를 찾으세요. 이
    참조들을 하나씩 팩토리 메소드에 대한 호출로 교체하면서 제품 생성
    코드를 팩토리 메서드로 추출하세요.

    반환된 제품의 유형을 제어하기 위해 팩토리 메서드에 임시 매개변수를
    추가해야 할 수도 있습니다.

    이 시점에서 팩토리 메서드의 코드는 꽤 복잡할 수 있습니다. 예를 들어
    인스턴트화할 제품 클래스를 선택하는 큰 `switch` 문장이 있을 수
    있습니다. 하지만 걱정하지 마세요, 곧 이 문제를 해결할 테니까요.

4.  이제 팩토리 메서드에 나열된 각 제품 유형에 대한 크리에이터 자식
    클래스들의 집합을 생성한 후, 자식 클래스들에서 팩토리 메서드를
    오버라이딩하고 기초 메서드에서 생성자 코드의 적절한 부분들을
    추출하세요.

5.  제품 유형이 너무 많아 모든 제품에 대하여 자식 클래스들을 만드는 것이
    합리적이지 않을 경우, 자식 클래스들의 기초 클래스의 제어 매개변수를
    재사용할 수 있습니다.

    예를 들어, 다음과 같은 클래스 계층구조가 있다고 상상해 보세요.
    `Mail`​(우편) 기초 클래스의 자식 클래스들은 `Air­Mail`​(항공우편)​과
    `Ground­Mail`​(지상우편)​이며, `Transport`​(운송수단) 클래스의 자식
    클래스들은 `Plane`​(비행기), `Truck`​(트럭), 그리고
    `Train`​(기차)​입니다. `Air­Mail`​(항공우편) 클래스는 `Plane`​(비행기)
    객체만 사용하지만, `Ground­Mail`​(지상우편)​은 `Truck` 과 `Train`
    객체들 모두 사용할 수 있습니다. 이 두 가지 경우를 모두 처리하기 위해
    새 자식 클래스​(예: `Train­Mail`​(기차우편))​를 만들 수도 있으나, 다른
    방법도 있습니다. 클라이언트 코드가 받으려는 제품을 제어하기 위해
    `Ground­Mail` 클래스의 팩토리 메서드에 전달인자​(argument)​를 전달하는
    방법입니다.

6.  추출이 모두 끝난 후 기초 팩토리 메서드가 비어 있으면, 해당 팩토리
    메서드를 추상화할 수 있습니다. 팩토리 메서드가 비어 있지 않으면,
    나머지를 그 메서드의 디폴트 행동으로 만들 수 있습니다.
:::

:::::: {.section .pros-cons}
##  장단점 {#pros-cons}

::::: row
::: col-sm-6
-    크리에이터와 구상 제품들이 단단하게 결합되지 않도록 할 수 있습니다.
-    *[단]{.cjk}[일]{.cjk} [책]{.cjk}[임]{.cjk} [원]{.cjk}[칙]{.cjk}*.
    제품 생성 코드를 프로그램의 한 위치로 이동하여 코드를 더 쉽게
    유지관리할 수 있습니다.
-    *[개]{.cjk}[방]{.cjk}/[폐]{.cjk}[쇄]{.cjk} [원]{.cjk}[칙]{.cjk}*.
    당신은 기존 클라이언트 코드를 훼손하지 않고 새로운 유형의 제품들을
    프로그램에 도입할 수 있습니다.
:::

::: col-sm-6
-    패턴을 구현하기 위해 많은 새로운 자식 클래스들을 도입해야 하므로
    코드가 더 복잡해질 수 있습니다. 가장 좋은 방법은 크리에이터
    클래스들의 기존 계층구조에 패턴을 도입하는 것입니다.
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

-   [추상 팩토리](abstract-factory.html) 클래스들은 [팩토리
    메서드](factory-method.html)들의 집합을 기반으로 하는 경우가
    많습니다. 그러나 당신은 또한 [프로토타입](prototype.html)을 사용하여
    [추상 팩토리](abstract-factory.html)의 구상 클래스들의 생성
    메서드들을 구현할 수도 있습니다.

-   [팩토리 메서드](factory-method.html)를 [반복자](iterator.html)와
    함께 사용하여 컬렉션 자식 클래스들이 해당 컬렉션들과 호환되는 다양한
    유형의 반복자들을 반환하도록 할 수 있습니다.

-   [프로토타입](prototype.html)은 상속을 기반으로 하지 않으므로 상속과
    관련된 단점들이 없습니다. 반면에
    *[프]{.cjk}[로]{.cjk}[토]{.cjk}[타]{.cjk}[입]{.cjk}*은 복제된 객체의
    복잡한 초기화가 필요합니다. [팩토리 메서드](factory-method.html)는
    상속을 기반으로 하지만 초기화 단계가 필요하지 않습니다.

-   [팩토리 메서드](factory-method.html)는 [템플릿
    메서드](template-method.html)의 특수화라고 생각할 수 있습니다.
    동시에 대규모 *[템]{.cjk}[플]{.cjk}[릿]{.cjk}
    [메]{.cjk}[서]{.cjk}[드]{.cjk}*의 한 단계의 역할을
    *[팩]{.cjk}[토]{.cjk}[리]{.cjk} [메]{.cjk}[서]{.cjk}[드]{.cjk}*가 할
    수 있습니다.
:::

::: {.section .implementations}
##  코드 예시 {#implementations}

[![C#으로 작성된 팩토리
메서드](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](factory-method/csharp/example.html "C#으로 작성된 팩토리 메서드"){.prog-lang-link}
[![C++로 작성된 팩토리
메서드](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](factory-method/cpp/example.html "C++로 작성된 팩토리 메서드"){.prog-lang-link}
[![Go로 작성된 팩토리
메서드](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](factory-method/go/example.html "Go로 작성된 팩토리 메서드"){.prog-lang-link}
[![자바로 작성된 팩토리
메서드](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](factory-method/java/example.html "자바로 작성된 팩토리 메서드"){.prog-lang-link}
[![PHP로 작성된 팩토리
메서드](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](factory-method/php/example.html "PHP로 작성된 팩토리 메서드"){.prog-lang-link}
[![파이썬으로 작성된 팩토리
메서드](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](factory-method/python/example.html "파이썬으로 작성된 팩토리 메서드"){.prog-lang-link}
[![루비로 작성된 팩토리
메서드](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](factory-method/ruby/example.html "루비로 작성된 팩토리 메서드"){.prog-lang-link}
[![러스트로 작성된 팩토리
메서드](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](factory-method/rust/example.html "러스트로 작성된 팩토리 메서드"){.prog-lang-link}
[![스위프트로 작성된 팩토리
메서드](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](factory-method/swift/example.html "스위프트로 작성된 팩토리 메서드"){.prog-lang-link}
[![타입스크립트로 작성된 팩토리
메서드](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](factory-method/typescript/example.html "타입스크립트로 작성된 팩토리 메서드"){.prog-lang-link}
:::

::: {.section .extras}
##  추가 콘텐츠 {#extras}

-   다양한 팩토리 패턴들과 개념들의 차이점에 대하여 더 자세히
    알아보시려면 [팩토리 비교](factory-comparison.html)를 읽어보세요.
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

[추상 팩토리 []{.fa .fa-arrow-right}](abstract-factory.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### 돌아가기

[[]{.fa .fa-arrow-left} 생성 패턴](creational-patterns.html){.btn
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
