::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [디자인
패턴들](../design-patterns.html){.type} / [행동
패턴](behavioral-patterns.html){.type}
:::

# 중재자 패턴 {#중재자-패턴 .title}

::: aka
다음 이름으로도 불립니다:
[중개인, ]{style="display:inline-block"}[컨트롤러, ]{style="display:inline-block"}[Mediator]{style="display:inline-block"}
:::

::: {.section .intent}
##  의도 {#intent}

**중재자**는 객체 간의 혼란스러운 의존 관계들을 줄일 수 있는 행동 디자인
패턴입니다. 이 패턴은 객체 간의 직접 통신을 제한하고 중재자 객체를
통해서만 협력하도록 합니다.

<figure class="image">
<img
src="../../images/patterns/content/mediator/mediator6761.png?id=0264bd857a231b6ea2d0c537c092e698"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/mediator/mediator.png?id=0264bd857a231b6ea2d0c537c092e698 640w,/images/patterns/content/mediator/mediator-1.5x.png?id=b3d5df41892274b5c84282bae8737441 960w,/images/patterns/content/mediator/mediator-2x.png?id=250c2bf72ca1fdee2e6d97ed5a4765f2 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="중재자 디자인 패턴" />
</figure>
:::

::: {.section .problem}
##  문제 {#problem}

고객들의 프로필을 만들고 편집하기 위한 대화 상자가 있다고 가정해 봅시다.
이 대화 상자는 텍스트 필드, 체크 상자, 버튼 등과 같은 다양한 양식
컨트롤들로 구성됩니다.

<figure class="image">
<img
src="../../images/patterns/diagrams/mediator/problem1-ko8eab.png?id=f3a2c91f03d9f5f4081f578efd8eecfe"
style="aspect-ratio: 2.22;"
srcset="/images/patterns/diagrams/mediator/problem1-ko.png?id=f3a2c91f03d9f5f4081f578efd8eecfe 600w,/images/patterns/diagrams/mediator/problem1-ko-1.5x.png?id=15141f9f505a7009932fb23011e834bf 900w,/images/patterns/diagrams/mediator/problem1-ko-2x.png?id=ac400bdacbdd18f6848dbeffc015b345 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="UI 요소 간의 혼란스러운 관계" />
<figcaption><p>앱이 발전함에 따라 사용자 인터페이스 요소 간의 관계가
혼란스러워질 수 있습니다.</p></figcaption>
</figure>

일부 양식 요소들은 다른 요소들과 상호 작용할 수 있습니다. 예를 들어,
\'저는 개가 있습니다\' 확인란을 선택하면 개의 이름을 입력하기 위한
숨겨진 텍스트 필드가 나타날 수 있습니다. 또 다른 예시로 데이터를
저장하기 전에 모든 필드의 값들을 검증해야 하는 제출 버튼이 있습니다.

<figure class="image">
<img
src="../../images/patterns/diagrams/mediator/problem2a7f7.png?id=072c51eee4dd90c0972866440c87c1c3"
style="aspect-ratio: 3.27;"
srcset="/images/patterns/diagrams/mediator/problem2.png?id=072c51eee4dd90c0972866440c87c1c3 360w,/images/patterns/diagrams/mediator/problem2-1.5x.png?id=b805abc211e60fa567fb114192e24608 540w,/images/patterns/diagrams/mediator/problem2-2x.png?id=d298ec82a47fa2938ed6cf64b7da78c1 720w"
sizes="(max-width: 720px) 100vw, 360px" loading="lazy" width="360"
alt="UI의 요소들은 상호 의존적입니다" />
<figcaption><p>요소들은 다른 요소들과 많은 관계를 맺을 수 있습니다.
따라서 일부 요소들을 변경하면 다른 요소들에 영향을 줄
수 있습니다.</p></figcaption>
</figure>

이 논리를 양식 요소들의 코드 내에서 직접 구현하면 이러한 요소들의
클래스들을 앱의 다른 양식들에서 재사용하기가 훨씬 더 어려워집니다. 예를
들어, 다른 양식 내에서는 위에 언급한 개 관련 확인란 클래스를 사용할 수
없습니다. 왜냐하면 기존 클래스가 개의 이름을 입력하기 위한 텍스트 필드와
결합되어 있기 때문입니다. 이 경우 프로필 양식 렌더링과 관련된 클래스들을
전부 사용하거나 아니면 아예 사용하지 말아야 합니다.
:::

::: {.section .solution}
##  해결책 {#solution}

중재자 패턴은 서로 독립적으로 작동해야 하는 컴포넌트 간의 모든 직접
통신을 중단한 후, 대신 이러한 컴포넌트들은 호출들을 적절한 컴포넌트들로
리다이렉션하는 특수 중재자 객체를 호출하여 간접적으로 협력하게 하라고
제안합니다. 그러면 컴포넌트들은 수십 개의 동료 컴포넌트들과 결합되는
대신 단일 중재자 클래스에만 의존합니다.

위 프로필 편집 양식 예시에서는 대화 상자 클래스 자체가 중재자 역할을 할
수 있습니다. 아마도 대화 상자 클래스는 이미 자신의 모든 하위 요소들을
인식하고 있으므로 새로운 의존관계들을 도입할 필요가 없을 것입니다.

<figure class="image">
<img
src="../../images/patterns/diagrams/mediator/solution1-ko1800.png?id=cd6e9a97ae9e9f8d8896ba1587945bfc"
style="aspect-ratio: 2.22;"
srcset="/images/patterns/diagrams/mediator/solution1-ko.png?id=cd6e9a97ae9e9f8d8896ba1587945bfc 600w,/images/patterns/diagrams/mediator/solution1-ko-1.5x.png?id=0cbd6022e20e064ec68b03fe1f801053 900w,/images/patterns/diagrams/mediator/solution1-ko-2x.png?id=4939b88241f68be398795bb552f02ca1 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="UI 요소들은 중재자를 통해 통신해야 합니다." />
<figcaption><p>UI 요소들은 다른 UI 요소들과 중재자 객체를 통해
간접적으로 통신해야 합니다.</p></figcaption>
</figure>

가장 중요한 변경들은 실제 양식 요소들에 적용됩니다. 제출 버튼을 살펴보면
이전에는 사용자가 이 버튼을 클릭할 때마다 버튼은 모든 개별 양식 요소들의
값들을 검증해야 했습니다. 이제 제출 버튼이 해야 할 유일한 일은 클릭을
대화 상자에 알리는 것 하나입니다. 이 알림을 받으면 대화 상자는 스스로
검증을 수행하거나 개별 요소들에게 작업을 전달합니다. 따라서 버튼은 여러
개의 양식 요소들에 연결되는 대신 대화 상자 클래스에만 의존하게 됩니다.

여기서 더 나아가 모든 유형의 대화 상자에서 공통 인터페이스를 추출하여
의존성을 더욱 느슨하게 만들 수 있습니다. 이 인터페이스는 모든 양식
요소가 해당 요소들에 발생하는 일​(이벤트)​들을 대화 상자에 알리는 데
사용할 수 있는 알림 메서드를 선언합니다. 이렇게 하면 제출 버튼은 이제
해당 인터페이스를 구현하는 모든 대화 상자들과 작업할 수 있습니다.

그렇게 하면, 중재자 패턴은 단일 중재자 객체 내부의 다양한 객체 간의
복잡한 관계망을 캡슐화할 수 있도록 합니다. 클래스의 의존관계들이
적을수록 해당 클래스를 수정, 확장 또는 재사용하기가 더 쉬워집니다.
:::

::: {.section .analogy}
##  실제상황 적용 {#analogy}

<figure class="image">
<img
src="../../images/patterns/diagrams/mediator/live-example3f2a.png?id=aa1de3cb7b63aa623e63578a1e43399a"
style="aspect-ratio: 1.85;"
srcset="/images/patterns/diagrams/mediator/live-example.png?id=aa1de3cb7b63aa623e63578a1e43399a 370w,/images/patterns/diagrams/mediator/live-example-1.5x.png?id=315109aa1099cc6e7c06057fa139881c 555w,/images/patterns/diagrams/mediator/live-example-2x.png?id=fd55a9f9d8cf49effa223555c7369504 740w"
sizes="(max-width: 720px) 100vw, 370px" loading="lazy" width="370"
alt="항공 교통 관제탑" />
<figcaption><p>항공기 조종사들은 다음에 누가 비행기를 착륙시킬지를
결정할 때 서로 직접 대화하지 않습니다. 모든 통신은 비행기 관제탑을
통해 이루어집니다.</p></figcaption>
</figure>

공항 관제 구역으로 들어오거나 그곳을 떠나는 항공기의 조종사들은 서로
직접 통신하지 않습니다. 대신, 그들은 높은 타워에 앉아서 일하는 항공 교통
관제사와 통신합니다. 항공 교통 관제사가 없다면 조종사들은 공항 근처의
모든 비행기의 존재 여부를 인식하고 수십 명의 다른 조종사들로 구성된
위원회와 착륙 우선순위를 논의해야 합니다. 그러면 비행기 충돌 횟수는
아마도 하늘로 치솟을 것입니다.

관제탑은 전체 비행을 관할하지 않습니다. 다만 관련되는 비행기의 수가
조종사에게는 너무 많을 수 있기에 공항 터미널 지역에서만 제약들을
강제하기 위해 존재합니다.
:::

::::: {.section .structure-container}
##  구조 {#structure}

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/mediator/structure71f0.png?id=1f2accc7820ecfe9665b6d30cbc0bc61"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.21;"
srcset="/images/patterns/diagrams/mediator/structure.png?id=1f2accc7820ecfe9665b6d30cbc0bc61 520w,/images/patterns/diagrams/mediator/structure-1.5x.png?id=958b373815bf6a56cd9b38763ed01dce 780w,/images/patterns/diagrams/mediator/structure-2x.png?id=5191daa1c9d4caa36e38af3c5b7d1522 1040w"
sizes="(max-width: 720px) 100vw, 520px" loading="lazy" width="520"
alt="중재자 디자인 패턴 구조" /><img
src="../../images/patterns/diagrams/mediator/structure-indexed5302.png?id=a82d4cf1b92a4f72af32f231ffd21131"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.21;"
srcset="/images/patterns/diagrams/mediator/structure-indexed.png?id=a82d4cf1b92a4f72af32f231ffd21131 520w,/images/patterns/diagrams/mediator/structure-indexed-1.5x.png?id=81d4f842ae5157a3cb09f8f3c05159dd 780w,/images/patterns/diagrams/mediator/structure-indexed-2x.png?id=88722da01a5c82b0452078c9339ca497 1040w"
sizes="(max-width: 720px) 100vw, 520px" loading="lazy" width="520"
alt="중재자 디자인 패턴 구조" />
</figure>
:::

1.  **컴포넌트들**은 어떤 비즈니스 로직을 포함한 다양한 클래스들입니다.
    각 컴포넌트에는 중재자에 대한 참조가 있는데, 이 중재자는 중재자
    인터페이스의 유형으로 선언됩니다. 컴포넌트는 중재자의 실제 클래스를
    인식하지 못하므로 컴포넌트를 다른 중재자에 연결하여 다른
    프로그램에서 재사용할 수 있습니다.

2.  **중재자** 인터페이스는 일반적으로 단일 알림 메서드만을 포함하는
    컴포넌트들과의 통신 메서드들을 선언합니다. 컴포넌트들은 자체
    객체들을 포함하여 모든 콘텍스트를 이 메서드의 인수로 전달할 수
    있지만 이는 수신자 컴포넌트와 발송자 클래스 간의 결합이 발생하지
    않는 방식으로만 가능합니다.

3.  **구상 중재자들**은 다양한 컴포넌트 간의 관계를 캡슐화합니다. 구상
    중재자들은 자신이 관리하는 모든 컴포넌트에 대한 참조를 유지하고
    때로는 그들의 수명 주기를 관리하기도 합니다.

4.  컴포넌트들은 다른 컴포넌트들을 인식하지 않아야 합니다. 컴포넌트
    내에서 또는 컴포넌트에 중요한 일이 발생하면, 컴포넌트는 이를
    중재자에게만 알려야 합니다. 중재자는 알림을 받으면 발송자를 쉽게
    식별할 수 있으며, 이는 응답으로 어떤 컴포넌트가 작동되어야 하는지
    결정하기에 충분할 수 있습니다.

    컴포넌트의 관점에서는 모든 것들이 블랙박스들​(기능은 알지만, 작동
    원리를 이해할 수 없는 복잡한 기계나 시스템)​처럼 보입니다. 발송자는
    누가 요청을 처리할지 모르고, 수신자는 누가 처음에 요청을 보냈는지를
    모릅니다.
::::
:::::

::: {.section .pseudocode}
##  의사코드 {#pseudocode}

이 예시에서 **중재자** 패턴은 버튼들, 확인란들 및 텍스트 레이블들과 같은
다양한 UI 클래스 간의 상호 의존성을 제거하는 데 도움이 됩니다.

<figure class="image">
<img
src="../../images/patterns/diagrams/mediator/example576a.png?id=3151c153533e816e226be0ef977211e8"
style="aspect-ratio: 1.24;"
srcset="/images/patterns/diagrams/mediator/example.png?id=3151c153533e816e226be0ef977211e8 560w,/images/patterns/diagrams/mediator/example-1.5x.png?id=cf441780d96f3306ac54d6809f85b87d 840w,/images/patterns/diagrams/mediator/example-2x.png?id=02064e5a7c4f065f806747e1b04ac1b0 1120w"
sizes="(max-width: 720px) 100vw, 560px" loading="lazy" width="560"
alt="중재자 패턴의 구조 예시" />
<figcaption><p>UI 대화 상자 클래스들의 구조.</p></figcaption>
</figure>

사용자에 의해 작동된 요소는 다른 요소들과 직접 통신하지 않습니다. 대신
이 요소는 중재자에게 이 이벤트​(사건)​에 대해 알리고 중재자에게 해당
알림과 함께 콘텍스트 정보를 전달합니다.

이 예시에서는 인증 대화 상자 전체가 중재자의 역할을 합니다. 이것은 구상
요소들이 어떻게 협력해야 하는지 알고 있으며 그들의 간접적인 의사소통을
촉진합니다. 이벤트에 대한 알림을 받으면 대화 상자는 이벤트를 처리해야
하는 요소를 결정하고 그 결정에 따라 호출을 리다이렉션합니다.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// 중재자 인터페이스는 컴포넌트들에서 사용하는 메서드를 선언하여 다양한 이벤트를
// 중재자에게 알립니다. 중재자는 이러한 이벤트에 반응해 실행을 다른 컴포넌트들에게
// 전달할 수 있습니다.
interface Mediator is
    method notify(sender: Component, event: string)


// 구상 중재자 클래스. 개별 컴포넌트들의 얽히고설킨 연결들이 풀리고 중재자로
// 이동되었습니다.
class AuthenticationDialog implements Mediator is
    private field title: string
    private field loginOrRegisterChkBx: Checkbox
    private field loginUsername, loginPassword: Textbox
    private field registrationUsername, registrationPassword,
                  registrationEmail: Textbox
    private field okBtn, cancelBtn: Button

    constructor AuthenticationDialog() is
        // 연결을 설립하기 위해 현재 중재자를 컴포넌트 객체들의 생성자들에
        // 전달하여 모든 컴포넌트 객체들을 생성하세요.

    // 컴포넌트에 무슨 일어나면, 중재자에게 알립니다. 알림을 받으면 중재자는
    // 자체적으로 알림을 처리하거나 요청을 다른 컴포넌트에 전달할 수 있습니다.
    method notify(sender, event) is
        if (sender == loginOrRegisterChkBx and event == &quot;check&quot;)
            if (loginOrRegisterChkBx.checked)
                title = &quot;Log in&quot;
                // 1. 로그인 양식 컴포넌트들을 표시하세요.
                // 2. 등록 양식 컴포넌트들을 표시하세요.
            else
                title = &quot;Register&quot;
                // 1. 등록 양식 컴포넌트들을 표시하세요.
                // 2. 로그인 양식 컴포넌트들을 숨기세요.

        if (sender == okBtn &amp;&amp; event == &quot;click&quot;)
            if (loginOrRegister.checked)
                // 로그인 자격 증명을 사용하여 사용자를 찾아보세요.
                if (!found)
                    // 로그인 필드 위에 오류 메시지를 표시하세요.
            else
                // 1. 등록 필드의 데이터를 사용하여 사용자 계정을 만드세요.
                // 2. 해당 사용자를 로그인시키세요.
                // …


// 컴포넌트들은 중재자 인터페이스를 사용하여 중재자와 통신합니다. 덕분에 컴포넌트들을
// 다른 중재자 객체들과 연결하여 다른 콘텍스트에서 같은 컴포넌트들을 사용할 수
// 있습니다.
class Component is
    field dialog: Mediator

    constructor Component(dialog) is
        this.dialog = dialog

    method click() is
        dialog.notify(this, &quot;click&quot;)

    method keypress() is
        dialog.notify(this, &quot;keypress&quot;)

// 구상 컴포넌트들은 서로 통신하지 않습니다. 그들은 하나의 통신 채널만 가지고
// 있으며, 이 채널을 통해 중재자에게 알림들을 보냅니다.
class Button extends Component is
    // …

class Textbox extends Component is
    // …

class Checkbox extends Component is
    method check() is
        dialog.notify(this, &quot;check&quot;)
    // …</code></pre>
</figure>
:::

:::::::::: {.section .applicability-container}
##  적용 {#applicability}

::::::::: applicability
::: applicability-problem
중재자 패턴은 일부 클래스들이 다른 클래스들과 단단하게 결합하여 변경하기
어려울 때 사용하세요.
:::

::: applicability-solution
중재자 패턴을 사용하면 특정 컴포넌트에 대한 모든 변경을 나머지
컴포넌트들로부터 고립하며 클래스 간의 모든 관계들을 별도의 클래스로
추출할 수 있습니다.
:::

::: applicability-problem
이 패턴은 타 컴포넌트들에 너무 의존하기 때문에 다른 프로그램에서
컴포넌트를 재사용할 수 없는 경우 사용하세요.
:::

::: applicability-solution
중재자 패턴을 적용하면, 그 후 개별 컴포넌트들은 다른 컴포넌트들을
인식하지 못합니다. 또, 비록 간접적이기는 하지만 컴포넌트들은 중재자
객체를 통해 여전히 서로 통신할 수 있습니다. 다른 앱에서 컴포넌트를
재사용하려면 그 앱에 새 중재자 클래스를 제공해야 합니다.
:::

::: applicability-problem
중재자 패턴은 몇 가지 기본 행동을 다양한 콘텍스트들에서 재사용하기 위해
수많은 컴포넌트 자식 클래스들을 만들고 있는 스스로를 발견했을 때
사용하세요.
:::

::: applicability-solution
컴포넌트들 간의 모든 관계들이 중재자 내에 포함되어 있으므로 컴포넌트들
자체를 변경할 필요 없이 새 중재자 클래스들을 도입하여 이러한
컴포넌트들이 협업할 수 있는 완전히 새로운 방법들을 쉽게 정의할 수
있습니다.
:::
:::::::::
::::::::::

::: {.section .checklist}
##  구현방법 {#checklist}

1.  더 독립적으로 만들었을 때 클래스의 유지 관리가 더 쉬워지거나
    재사용이 더 간편해지는 등의 이점이 있는 단단히 결합된 클래스들을
    식별하세요.

2.  중재자 인터페이스를 선언하고 중재자와 다양한 컴포넌트 간의 원하는
    통신 프로토콜을 설명하세요. 대부분의 경우 컴포넌트들에서 알림을
    수신하는 단일 메서드로 충분합니다.

    이 인터페이스는 다른 콘텍스트들에서 컴포넌트 클래스들을 재사용하고자
    할 때 매우 중요합니다. 컴포넌트가 일반 인터페이스를 통해 중재자와
    함께 작동하는 한 컴포넌트를 중재자의 다른 구현과 연결할 수 있습니다.

3.  구상 중재자 클래스를 구현하세요. 모든 컴포넌트에 대한 참조를 중재자
    내부에 저장하는 것을 고려해보세요. 그렇게 하면 중재자의 메서드에서
    어떤 컴포넌트라도 호출할 수 있습니다.

4.  더 나아가 중재자가 컴포넌트 객체들의 생성 및 파괴를 담당하도록 할 수
    있으며, 그러면 중재자는 [팩토리](abstract-factory.html) 또는
    [퍼사드](facade.html)와 유사할 수 있습니다.

5.  컴포넌트들은 중재자 객체에 대한 참조를 저장해야 합니다. 이 연결은
    일반적으로 컴포넌트의 생성자에서 설정되며 중재자 객체가 인수로
    전달됩니다.

6.  다른 컴포넌트들의 메서드 대신 중재자의 알림 메서드를 호출하도록
    컴포넌트의 코드를 변경하세요. 그 후 다른 컴포넌트들을 호출하는 것과
    관련된 코드를 중재자 클래스 안으로 추출하세요. 이 코드는 중재자가
    해당 컴포넌트에서 알림들을 받을 때마다 실행하세요.
:::

:::::: {.section .pros-cons}
##  장단점 {#pros-cons}

::::: row
::: col-sm-6
-    *[단]{.cjk}[일]{.cjk} [책]{.cjk}[임]{.cjk} [원]{.cjk}[칙]{.cjk}*.
    다양한 컴포넌트 간의 통신을 한곳으로 추출하여 코드를 이해하고 유지
    관리하기 쉽게 만들 수 있습니다.
-    *[개]{.cjk}[방]{.cjk}/[폐]{.cjk}[쇄]{.cjk} [원]{.cjk}[칙]{.cjk}*.
    실제 컴포넌트들을 변경하지 않고도 새로운 중재자들을 도입할 수
    있습니다.
-    프로그램의 다양한 컴포넌트 간의 결합도를 줄일 수 있습니다.
-    개별 컴포넌트들을 더 쉽게 재사용할 수 있습니다.
:::

::: col-sm-6
-    중재자는 [전지전능한
    객체](https://en.wikipedia.org/wiki/God_object)로 발전할지도
    모릅니다.
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

-   [중재자](mediator.html)와 [퍼사드](facade.html) 패턴은 비슷한 역할을
    합니다. 둘 다 밀접하게 결합된 많은 클래스 간의 협업을 구성하려고
    합니다.

    -   *[퍼]{.cjk}[사]{.cjk}[드]{.cjk}* 패턴은 객체들의 하위 시스템에
        대한 단순화된 인터페이스를 정의하지만 새로운 기능을 도입하지는
        않습니다. 하위 시스템 자체는 퍼사드를 인식하지 못하며, 하위
        시스템 내의 객체들은 서로 직접 통신할 수 있습니다.
    -   *[중]{.cjk}[재]{.cjk}[자]{.cjk}*는 시스템 컴포넌트 간의 통신을
        중앙 집중화합니다. 컴포넌트들은 중재자 객체에 대해서만 알며 서로
        직접 통신하지 않습니다.

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
중재자](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](mediator/csharp/example.html "C#으로 작성된 중재자"){.prog-lang-link}
[![C++로 작성된
중재자](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](mediator/cpp/example.html "C++로 작성된 중재자"){.prog-lang-link}
[![Go로 작성된
중재자](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](mediator/go/example.html "Go로 작성된 중재자"){.prog-lang-link}
[![자바로 작성된
중재자](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](mediator/java/example.html "자바로 작성된 중재자"){.prog-lang-link}
[![PHP로 작성된
중재자](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](mediator/php/example.html "PHP로 작성된 중재자"){.prog-lang-link}
[![파이썬으로 작성된
중재자](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](mediator/python/example.html "파이썬으로 작성된 중재자"){.prog-lang-link}
[![루비로 작성된
중재자](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](mediator/ruby/example.html "루비로 작성된 중재자"){.prog-lang-link}
[![러스트로 작성된
중재자](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](mediator/rust/example.html "러스트로 작성된 중재자"){.prog-lang-link}
[![스위프트로 작성된
중재자](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](mediator/swift/example.html "스위프트로 작성된 중재자"){.prog-lang-link}
[![타입스크립트로 작성된
중재자](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](mediator/typescript/example.html "타입스크립트로 작성된 중재자"){.prog-lang-link}
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

[메멘토 []{.fa .fa-arrow-right}](memento.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### 돌아가기

[[]{.fa .fa-arrow-left} 반복자](iterator.html){.btn .btn-default
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
