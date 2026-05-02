::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [디자인
패턴들](../design-patterns.html){.type} / [행동
패턴](behavioral-patterns.html){.type}
:::

# 옵서버 패턴 {#옵서버-패턴 .title}

::: aka
다음 이름으로도 불립니다: [이벤트
구독자, ]{style="display:inline-block"}[경청자, ]{style="display:inline-block"}[Observer]{style="display:inline-block"}
:::

::: {.section .intent}
##  의도 {#intent}

**옵서버** 패턴은 당신이 여러 객체에 자신이 관찰 중인 객체에 발생하는
모든 이벤트에 대하여 알리는 구독 메커니즘을 정의할 수 있도록 하는 행동
디자인 패턴입니다.

<figure class="image">
<img
src="../../images/patterns/content/observer/observerd508.png?id=6088e31e1b0d4a417506a66614dcf065"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/observer/observer.png?id=6088e31e1b0d4a417506a66614dcf065 640w,/images/patterns/content/observer/observer-1.5x.png?id=aa64f9f336354462b57bbff5c09d8269 960w,/images/patterns/content/observer/observer-2x.png?id=d5a83e115528e9fd633f04ad2650f1db 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="옵서버 디자인 패턴" />
</figure>
:::

::: {.section .problem}
##  문제 {#problem}

`Customer`​(손님) 및 `Store`​(가게)​라는 두 가지 유형의 객체들이 있다고
가정합니다. 손님은 곧 매장에 출시될 특정 브랜드의 제품​(예: 새 아이폰
모델)​에 매우 관심이 있습니다.

손님은 매일 매장을 방문하여 제품 재고를 확인할 수 있으나, 제품이 매장에
아직 운송되는 동안 이러한 방문 대부분은 무의미합니다.

<figure class="image">
<img
src="../../images/patterns/content/observer/observer-comic-1-ko33ac.png?id=57406189c12563bb80b8e54042948298"
style="aspect-ratio: 2;"
srcset="/images/patterns/content/observer/observer-comic-1-ko.png?id=57406189c12563bb80b8e54042948298 600w,/images/patterns/content/observer/observer-comic-1-ko-1.5x.png?id=a75c69402ad226b00069ce0f8c1e68ac 900w,/images/patterns/content/observer/observer-comic-1-ko-2x.png?id=d978646c5bca4f9add1bdd079468ef34 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="매장 방문 vs. 스팸 발송" />
<figcaption><p>매장 방문 vs. 스팸 발송</p></figcaption>
</figure>

반면 매장에서는 새로운 제품이 출시될 때마다 모든 고객에게 스팸으로
간주할 수 있는 수많은 이메일을 보낼 수 있습니다. 이 수많은 이메일은 일부
고객들을 신제품 출시 확인을 위한 잦은 매장 방문으로부터 구출해낼 수
있으나, 동시에 신제품 출시에 관심이 없는 다른 고객들을 화나게 할
것입니다.

여기서 충돌이 발생합니다. 손님들이 신제품 출시 확인을 위해 시간을
낭비하든지, 매장들이 알림을 원하지 않는 고객들에게 신제품 출시를 알리며
자원을 낭비해야 합니다.
:::

::: {.section .solution}
##  해결책 {#solution}

시간이 지나면 변경될 수 있는 중요한 상태를 가진 객체가 있다고
가정해봅시다. 이 객체는 종종 *[주]{.cjk}[제]{.cjk}​(subject)*라고
불립니다. 그러나 위 예시의 경우 이 객체는 자신의 상태에 대한 변경에 대해
다른 객체들에 알림을 보내는 역할도 맡을 것이니 해당 객체를
*[출]{.cjk}[판]{.cjk}[사]{.cjk}*라고 부르겠습니다.

옵서버 패턴은 출판사 클래스에 개별 객체들이 그 출판사로부터 오는
이벤트들의 알림들을 구독 또는 구독 취소할 수 있도록 구독 메커니즘을
추가할 것을 제안합니다. 두려워하지 마세요. 그리 복잡하지 않습니다.
실제로 이 메커니즘은 1) 구독자 객체들에 대한 참조의 리스트를 저장하기
위한 배열 필드와 2) 그 리스트에 구독자들을 추가하거나 제거할 수 있도록
하는 여러 공개된​(public) 메서드들로 구성됩니다.

<figure class="image">
<img
src="../../images/patterns/diagrams/observer/solution1-ko20d4.png?id=5df3ff2a5b9f8660433b0a7a8685287e"
style="aspect-ratio: 2.61;"
srcset="/images/patterns/diagrams/observer/solution1-ko.png?id=5df3ff2a5b9f8660433b0a7a8685287e 470w,/images/patterns/diagrams/observer/solution1-ko-1.5x.png?id=f33ccafb1e5eed5b5441892a044d2da0 705w,/images/patterns/diagrams/observer/solution1-ko-2x.png?id=edc1fd0ab3a09ab11f54075047cf55ea 940w"
sizes="(max-width: 720px) 100vw, 470px" loading="lazy" width="470"
alt="구독 메커니즘" />
<figcaption><p>구독 메커니즘을 통해 개별 객체들이 이벤트 알림들을 구독할
수 있습니다.</p></figcaption>
</figure>

이제 출판사에 중요한 이벤트가 발생할 때마다 구독자 리스트를 참조한 후
그들의 객체들에 있는 특정 알림 메서드를 호출합니다.

실제 앱에는 같은 출판사 클래스의 이벤트들을 추적하는 데 관심이 있는 수십
개의 서로 다른 구독자 클래스들이 있을 수 있습니다. 당신은 출판사를
이러한 모든 클래스에 결합하고 싶지 않을 것입니다. 게다가 당신은 당신의
출판사 클래스가 다른 사람들에 의해 사용되어야 한다면 이러한 구독자
클래스 중 일부는 미리 알지 못할 수도 있습니다.

그러므로 모든 구독자가 같은 인터페이스를 구현하고 출판사가 오직 그
인터페이스를 통해서만 구독자들과 통신하는 것이 매우 중요합니다. 이
인터페이스는 출판사가 알림과 어떤 콘텍스트 데이터를 전달하는 데 사용할
수 있는 매개변수들의 집합과 알림 메서드를 선언해야 합니다.

<figure class="image">
<img
src="../../images/patterns/diagrams/observer/solution2-ko7bf8.png?id=c91ea0905f2ea3ebdce959d975b19ab8"
style="aspect-ratio: 1.24;"
srcset="/images/patterns/diagrams/observer/solution2-ko.png?id=c91ea0905f2ea3ebdce959d975b19ab8 460w,/images/patterns/diagrams/observer/solution2-ko-1.5x.png?id=5dce26ac1a8bf97ea7ee92bde4957316 690w,/images/patterns/diagrams/observer/solution2-ko-2x.png?id=7d3644a085bd64940f80d4270bf7804b 920w"
sizes="(max-width: 720px) 100vw, 460px" loading="lazy" width="460"
alt="알림 메서드들" />
<figcaption><p>출판사는 특정 알림 메서드를 구독자들의 객체들에서부터
호출하여 그들에게 알림을 보냅니다.</p></figcaption>
</figure>

당신의 앱에 여러 유형의 출판사가 있고 이들을 구독자들과 호환되도록
하려면 당신은 더 나아가 모든 출판사가 같은 인터페이스를 따르도록 할 수
있습니다. 이 인터페이스는 몇 가지 구독 메서드들만 설명하면 됩니다. 이
인터페이스를 통해 구독자들은 출판자들의 상태들을 그들의 구상 클래스들과
결합하지 않고 관찰할 수 있습니다.
:::

::: {.section .analogy}
##  실제상황 적용 {#analogy}

<figure class="image">
<img
src="../../images/patterns/content/observer/observer-comic-2-ko263b.png?id=736738665a481b788167371b913db20c"
style="aspect-ratio: 2;"
srcset="/images/patterns/content/observer/observer-comic-2-ko.png?id=736738665a481b788167371b913db20c 600w,/images/patterns/content/observer/observer-comic-2-ko-1.5x.png?id=df0014870c79c6736c29200c7eb4b80a 900w,/images/patterns/content/observer/observer-comic-2-ko-2x.png?id=eacabaea88928e4e07c790436ecda4a7 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="잡지 및 신문 구독" />
<figcaption><p>잡지 및 신문 구독.</p></figcaption>
</figure>

당신이 신문이나 잡지를 구독한다면 다음 호가 있는지 확인하러 가게에 갈
필요가 없습니다. 대신 출판사가 발행 직후나 사전에 새 발행물을 구독자의
우편함으로 직접 보냅니다.

출판사는 구독자 리스트를 유지 관리하고 구독자들이 어떤 잡지에 관심
있는지 알고 있습니다. 출판사가 새로운 잡지의 발행호들를 보내는 것을
중단시키고 싶다면 구독자들은 언제든지 이 리스트를 떠날 수 있습니다.
:::

::::: {.section .structure-container}
##  구조 {#structure}

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/observer/structurea121.png?id=365b7e2b8fbecc8948f34b9f8f16f33c"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.97;"
srcset="/images/patterns/diagrams/observer/structure.png?id=365b7e2b8fbecc8948f34b9f8f16f33c 610w,/images/patterns/diagrams/observer/structure-1.5x.png?id=7f23a48c9a2d789844f912d3a0f289e6 915w,/images/patterns/diagrams/observer/structure-2x.png?id=228af9bded4d6ee6daf43a0e23cca9ff 1220w"
sizes="(max-width: 720px) 100vw, 610px" loading="lazy" width="610"
alt="옵서버 디자인 패턴 구조" /><img
src="../../images/patterns/diagrams/observer/structure-indexed8afd.png?id=2ca2c123503ede860740af2a22bc4b4d"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.88;"
srcset="/images/patterns/diagrams/observer/structure-indexed.png?id=2ca2c123503ede860740af2a22bc4b4d 620w,/images/patterns/diagrams/observer/structure-indexed-1.5x.png?id=20b9e7765e83ca75514c202968d9e9fa 930w,/images/patterns/diagrams/observer/structure-indexed-2x.png?id=910eec855bc41f05199e510494078926 1240w"
sizes="(max-width: 720px) 100vw, 620px" loading="lazy" width="620"
alt="옵서버 디자인 패턴 구조" />
</figure>
:::

1.  **출판사**는 다른 객체들에 관심 이벤트들을 발행합니다. 이러한
    이벤트들은 출판사가 상태를 전환하거나 어떤 행동들을 실행할 때
    발생합니다. 출판사들에는 구독 인프라가 포함되어 있으며, 이 인프라는
    현재 구독자들이 리스트를 떠나고 새 구독자들이 리스트에 가입할 수
    있도록 합니다.

2.  새 이벤트가 발생하면 출판사는 구독자 리스트를 살펴본 후 각 구독자
    객체의 구독자 인터페이스에 선언된 알림 메서드를 호출합니다.

3.  이 **구독자** 인터페이스는 알림 인터페이스를 선언하며 대부분의 경우
    단일 `update` 메서드로 구성됩니다. 이 메서드에는 출판사가 업데이트와
    함께 어떤 이벤트의 세부 정보들을 전달할 수 있도록 하는 여러
    매개변수가 있을 수 있습니다.

4.  **구상 구독자들**은 출판사가 보낸 알림들에 대한 응답으로 몇 가지
    작업을 수행합니다. 이러한 모든 클래스는 출판사가 구상 클래스들과
    결합하지 않도록 같은 인터페이스를 구현해야 합니다.

5.  일반적으로 구독자들은 업데이트를 올바르게 처리하기 위해 콘텍스트
    정보가 어느 정도 필요로 합니다. 그러므로 출판사들은 종종 콘텍스트
    데이터를 알림 메서드의 인수들로 전달합니다. 출판사는 자신을 인수로
    전달할 수 있으며, 구독자가 필요한 데이터를 직접 가져오도록 합니다.

6.  **클라이언트**는 출판사 및 구독자 객체들을 별도로 생성한 후
    구독자들을 출판사 업데이트에 등록합니다.
::::
:::::

::: {.section .pseudocode}
##  의사코드 {#pseudocode}

이 예시에서 **옵서버 패턴**은 텍스트 편집기 객체가 다른 서비스 객체들에
자신의 상태 변경에 대해 알릴 수 있도록 합니다.

<figure class="image">
<img
src="../../images/patterns/diagrams/observer/example04a2.png?id=6d0603ab5a00e4463b81d9639cd746a2"
style="aspect-ratio: 1.19;"
srcset="/images/patterns/diagrams/observer/example.png?id=6d0603ab5a00e4463b81d9639cd746a2 510w,/images/patterns/diagrams/observer/example-1.5x.png?id=67adccd1030a38dd263bfa09867f8dbe 765w,/images/patterns/diagrams/observer/example-2x.png?id=e2838e1562325e485fc7c2828a8ca445 1020w"
sizes="(max-width: 720px) 100vw, 510px" loading="lazy" width="510"
alt="옵서버 패턴 구조 예시" />
<figcaption><p>다른 객체들에 발생하는 이벤트에 대해
객체들에 알립니다.</p></figcaption>
</figure>

구독자 리스트는 동적으로 컴파일됩니다. 당신이 앱이 원하는 행동에 따라
객체들은 런타임 때 알림들을 받는 것을 시작하거나 중단할 수 있습니다.

이 구현에서 편집기 클래스는 자체적으로 구독 리스트를 유지 관리하지
않습니다. 편집기 클래스는 이 작업을 해당 작업을 전담하는 특수 도우미
객체에 위임합니다. 이 객체를 중앙 집중식 이벤트 디스패처 역할을 하도록
업그레이드하여 모든 객체가 출판사 역할을 하도록 할 수 있습니다.

앱에 새 구독자들을 추가할 때 기존 출판사 클래스들이 같은 인터페이스를
통해 모든 구독자와 작업하는 한 기존 출판사 클래스들은 변경할 필요가
없습니다.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// 기초 출판사 클래스에는 구독 관리 코드 및 알림 메서드들이 포함됩니다.
class EventManager is
    private field listeners: hash map of event types and listeners

    method subscribe(eventType, listener) is
        listeners.add(eventType, listener)

    method unsubscribe(eventType, listener) is
        listeners.remove(eventType, listener)

    method notify(eventType, data) is
        foreach (listener in listeners.of(eventType)) do
            listener.update(data)

// 구상 출판사는 일부 구독자에게 흥미로운 실제 비즈니스 논리를 포함합니다. 우리는
// 이 클래스를 기초 출판사로부터 파생시킬 수 있습니다. 그러나 이는 현실에서 항상
// 가능하지 않습니다. 왜냐하면 구상 클래스가 이미 자식 클래스일 수 있기
// 때문입니다. 이 경우 여기에서 했던 것처럼 합성 관계 속으로 구독 논리를 덧붙여
// 넣을 수 있습니다.
class Editor is
    public field events: EventManager
    private field file: File

    constructor Editor() is
        events = new EventManager()

    // 비즈니스 로직의 메서드들은 구독자들에게 변경 사항에 대해 알릴 수 있습니다.
    method openFile(path) is
        this.file = new File(path)
        events.notify(&quot;open&quot;, file.name)

    method saveFile() is
        file.write()
        events.notify(&quot;save&quot;, file.name)

    // …


// 여기 구독자 인터페이스가 있습니다. 사용 중인 프로그래밍 언어가 함수형 타입을
// 지원하는 경우 전체 구독자 계층구조를 함수들의 집합으로 바꿀 수 있습니다.
interface EventListener is
    method update(filename)

// 구상 구독자들은 자신이 연결된 출판사가 발행한 업데이트에 반응합니다.
class LoggingListener implements EventListener is
    private field log: File
    private field message: string

    constructor LoggingListener(log_filename, message) is
        this.log = new File(log_filename)
        this.message = message

    method update(filename) is
        log.write(replace(&#39;%s&#39;,filename,message))

class EmailAlertsListener implements EventListener is
    private field email: string
    private field message: string

    constructor EmailAlertsListener(email, message) is
        this.email = email
        this.message = message

    method update(filename) is
        system.email(email, replace(&#39;%s&#39;,filename,message))


// 앱은 런타임에 출판사들과 구독자들을 설정할 수 있습니다.
class Application is
    method config() is
        editor = new Editor()

        logger = new LoggingListener(
            &quot;/path/to/log.txt&quot;,
            &quot;Someone has opened the file: %s&quot;)
        editor.events.subscribe(&quot;open&quot;, logger)

        emailAlerts = new EmailAlertsListener(
            &quot;admin@example.com&quot;,
            &quot;Someone has changed the file: %s&quot;)
        editor.events.subscribe(&quot;save&quot;, emailAlerts)</code></pre>
</figure>
:::

:::::::: {.section .applicability-container}
##  적용 {#applicability}

::::::: applicability
::: applicability-problem
옵서버 패턴은 한 객체의 상태가 변경되어 다른 객체들을 변경해야 할
필요성이 생겼을 때, 그리고 실제 객체 집합들을 미리 알 수 없거나 이러한
집합들이 동적으로 변경될 때 사용하세요.
:::

::: applicability-solution
이런 문제는 GUI 클래스와 작업할 때 자주 경험할 수 있습니다. 예를 들어
당신이 사용자 정의 버튼 클래스들을 생성했고, 이제 클라이언트들이 사용자
정의 코드를 버튼에 연결하여 사용자가 버튼을 누를 때마다 실행되도록 하고
싶다고 가정합시다.

옵서버 패턴은 구독자 인터페이스를 구현하는 모든 객체가 출판사 객체의
이벤트 알림들에 구독할 수 있도록 합니다. 당신은 버튼에 구독 메커니즘을
추가할 수 있으며, 클라이언트들이 사용자 정의 구독자 클래스들을 통해
사용자 정의 코드를 연결하도록 할 수 있습니다.
:::

::: applicability-problem
이 패턴은 앱의 일부 객체들이 제한된 시간 동안 또는 특정 경우에만 다른
객체들을 관찰해야 할 때 사용하세요.
:::

::: applicability-solution
구독 리스트는 동적이므로 구독자들은 필요할 때마다 리스트에 가입하거나
탈퇴할 수 있습니다.
:::
:::::::
::::::::

::: {.section .checklist}
##  구현방법 {#checklist}

1.  당신의 앱의 비즈니스 로직을 살펴보고 두 부분으로 나누세요. 핵심
    기능들은 다른 코드와 독립적이며 출판사 역할을 합니다. 나머지는
    구독자 클래스들의 집합으로 바뀝니다.

2.  구독자 인터페이스를 선언하세요. 이 인터페이스는 최소한 하나의
    `update` 메서드를 선언해야 합니다.

3.  출판사 인터페이스를 선언하고 구독자 객체를 구독자 리스트에 추가 및
    제거하는 한 쌍의 메서드에 대해 기술하세요. 출판사들은 구독자
    인터페이스를 통해서만 구독자들과 작업해야 합니다.

4.  구독 메서드들의 구현과 실제 구독 리스트를 어디에 배치할지
    결정하세요. 일반적으로 모든 유형의 출판사에서 이 코드는 실질적으로
    유사하므로 출판사 인터페이스에서 직접 파생된 추상 클래스에 코드를
    넣는 것이 가장 적합합니다. 구상 출판사들은 이 클래스를 확장하여 해당
    클래스의 구독 행동을 상속합니다.

    그러나 기존 클래스 계층구조에 패턴을 적용하는 경우 합성에 기반한
    접근 방식을 고려하세요. 구독 로직을 별도의 객체에 넣고 모든 실제
    출판사들이 이를 사용하도록 하세요.

5.  구상 출판사 클래스들을 만드세요. 출판사 내부에서 중요한 일이 발생할
    때마다 모든 구독자에게 알림을 전달해야 합니다.

6.  구상 구독자 클래스들에서 업데이트 알림 메서드들을 구현하세요.
    대부분의 구독자는 이벤트에 대한 일부 콘텍스트 데이터가 필요하며, 이
    데이터는 알림 메서드의 인수로 전달될 수 있습니다.

    그러나 다른 옵션이 있습니다. 알림을 받으면 구독자들이 알림에서 직접
    모든 데이터를 가져오도록 하는 것입니다. 이 경우 출판사는 업데이트
    메서드를 통해 자신을 전달해야 합니다. 유연성이 보다 떨어지는 옵션은
    생성자를 통해 출판자를 구독자에 영구적으로 연결하는 것입니다.

7.  클라이언트는 필요한 모든 구독자를 생성하고 적절한 출판사들과
    등록시켜야 합니다.
:::

:::::: {.section .pros-cons}
##  장단점 {#pros-cons}

::::: row
::: col-sm-6
-    *[개]{.cjk}[방]{.cjk}/[폐]{.cjk}[쇄]{.cjk} [원]{.cjk}[칙]{.cjk}*.
    출판사의 코드를 변경하지 않고도 새 구독자 클래스들을 도입할 수
    있습니다. (출판사 인터페이스가 있는 경우 그 반대로 구독자의
    클래스들을 변경하지 않고 새 출판사 클래스들을 도입하는 것 역시
    가능합니다).
-    런타임에 객체 간의 관계들을 형성할 수 있습니다.
:::

::: col-sm-6
-    구독자들은 무작위로 알림을 받습니다.
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

-   [중재자](mediator.html)와 [옵서버](observer.html) 패턴의 차이는 종종
    애매합니다. 대부분의 경우 두 패턴 중 하나를 구현할 수 있으나, 때로는
    두 패턴을 동시에 적용할 수 있습니다. 이것이 어떻게 가능한지
    살펴보겠습니다.

    *[중]{.cjk}[재]{.cjk}[자]{.cjk}*의 주목적은 시스템 컴포넌트들의 집합
    간의 상호 의존성을 제거하는 것입니다. 그러면 이러한 컴포넌트들은
    대신 단일 중재자 객체에 의존하게 됩니다.
    *[옵]{.cjk}[서]{.cjk}[버]{.cjk} [패]{.cjk}[턴]{.cjk}[의]{.cjk}*의
    목적은 객체들 사이에 단방향 연결을 설정하는 것으로, 여기서 일부
    객체는 다른 객체의 종속자 역할을 합니다.

    *[옵]{.cjk}[서]{.cjk}[버]{.cjk} [패]{.cjk}[턴]{.cjk}*에 의존하는
    *[중]{.cjk}[재]{.cjk}[자]{.cjk}* 패턴의 인기 있는 구현이 있습니다.
    중재자 객체는 출판사의 역할을 맡고, 컴포넌트들은 중재자의 이벤트들을
    구독 및 구독 취소하는 구독자들의 역할을 맡습니다.
    *[중]{.cjk}[재]{.cjk}[자]{.cjk}*가 이러한 방식으로 구현되면
    *[옵]{.cjk}[서]{.cjk}[버]{.cjk} [패]{.cjk}[턴]{.cjk}*과 매우
    유사하게 보일 수 있습니다.

    만약 혼란스러우시다면 중재자 패턴을 다른 방법들로 구현할 수 있음을
    기억하세요. 예를 들어 모든 컴포넌트를 영구적으로 같은 중재자 객체에
    연결하는 방법이 있습니다. 이 구현은 *[옵]{.cjk}[서]{.cjk}[버]{.cjk}*
    패턴과 유사하지 않겠지만 여전히 중재자 패턴의 인스턴스일 것입니다.

    이제 모든 컴포넌트가 출판사가 되어 서로 간의 동적 연결을 허용하는
    프로그램을 상상해 보세요. 중앙화된 중재자 객체는 없고 분산된
    옵서버들의 집합만 있을 것입니다.
:::

::: {.section .implementations}
##  코드 예시 {#implementations}

[![C#으로 작성된
옵서버](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](observer/csharp/example.html "C#으로 작성된 옵서버"){.prog-lang-link}
[![C++로 작성된
옵서버](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](observer/cpp/example.html "C++로 작성된 옵서버"){.prog-lang-link}
[![Go로 작성된
옵서버](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](observer/go/example.html "Go로 작성된 옵서버"){.prog-lang-link}
[![자바로 작성된
옵서버](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](observer/java/example.html "자바로 작성된 옵서버"){.prog-lang-link}
[![PHP로 작성된
옵서버](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](observer/php/example.html "PHP로 작성된 옵서버"){.prog-lang-link}
[![파이썬으로 작성된
옵서버](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](observer/python/example.html "파이썬으로 작성된 옵서버"){.prog-lang-link}
[![루비로 작성된
옵서버](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](observer/ruby/example.html "루비로 작성된 옵서버"){.prog-lang-link}
[![러스트로 작성된
옵서버](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](observer/rust/example.html "러스트로 작성된 옵서버"){.prog-lang-link}
[![스위프트로 작성된
옵서버](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](observer/swift/example.html "스위프트로 작성된 옵서버"){.prog-lang-link}
[![타입스크립트로 작성된
옵서버](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](observer/typescript/example.html "타입스크립트로 작성된 옵서버"){.prog-lang-link}
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

[상태 []{.fa .fa-arrow-right}](state.html){.btn .btn-primary rel="next"}
:::

::: prev
#### 돌아가기

[[]{.fa .fa-arrow-left} 메멘토](memento.html){.btn .btn-default
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
