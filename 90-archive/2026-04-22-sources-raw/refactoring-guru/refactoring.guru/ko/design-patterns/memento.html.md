::::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [디자인
패턴들](../design-patterns.html){.type} / [행동
패턴](behavioral-patterns.html){.type}
:::

# 메멘토 패턴 {#메멘토-패턴 .title}

::: aka
다음 이름으로도 불립니다:
[스냅샷, ]{style="display:inline-block"}[Memento]{style="display:inline-block"}
:::

::: {.section .intent}
##  의도 {#intent}

**메멘토**는 객체의 구현 세부 사항을 공개하지 않으면서 해당 객체의 이전
상태를 저장하고 복원할 수 있게 해주는 행동 디자인 패턴입니다.

<figure class="image">
<img
src="../../images/patterns/content/memento/memento-koa75e.png?id=bc63baae4c8a4997879a888262c75e91"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/memento/memento-ko.png?id=bc63baae4c8a4997879a888262c75e91 640w,/images/patterns/content/memento/memento-ko-1.5x.png?id=908b84657a33afe2b638526b43ddb809 960w,/images/patterns/content/memento/memento-ko-2x.png?id=97700930bf24a3e2936e3be9c7b374ee 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="메멘토 디자인 패턴" />
</figure>
:::

::: {.section .problem}
##  문제 {#problem}

텍스트 편집기 앱을 만들고 있다고 상상해보세요. 당신의 편집기는 간단한
텍스트 편집 외에도 텍스트의 서식 지정, 인라인 이미지들의 삽입 등을 할 수
있습니다.

어느 날 당신은 사용자들이 텍스트에 수행된 모든 작업을 실행 취소할 수
있도록 하기로 했습니다. 이 실행 취소 기능은 수년에 걸쳐 매우
보편화되었기 때문에 오늘날의 사용자들은 모든 앱에 이 기능이 있을
것이라고 가정합니다. 이 기능을 구현하기 위해 직접 접근 방식을 적용하기로
했습니다. 앱은 모든 작업을 수행하기 전에 모든 객체의 상태를 기록해 어떤
스토리지에 저장합니다. 나중에 사용자가 작업을 실행 취소하기로 하면 앱은
기록에서 가장 최신 스냅샷을 가져와 모든 객체의 상태를 복원하는 데
사용합니다.

<figure class="image">
<img
src="../../images/patterns/diagrams/memento/problem1-ko6cf9.png?id=076128ac4b6e81ae02bd52019ee2279d"
style="aspect-ratio: 2.61;"
srcset="/images/patterns/diagrams/memento/problem1-ko.png?id=076128ac4b6e81ae02bd52019ee2279d 470w,/images/patterns/diagrams/memento/problem1-ko-1.5x.png?id=1ce03532ea8a57100c57ea3716a037a6 705w,/images/patterns/diagrams/memento/problem1-ko-2x.png?id=6243d6ade3f4b49252a3fc7d2f4ee456 940w"
sizes="(max-width: 720px) 100vw, 470px" loading="lazy" width="470"
alt="편집기에서 작업 되돌리기" />
<figcaption><p>앱은 작업을 실행하기 전에 객체들의 상태의 스냅샷을
저장하며, 이 스냅샷은 나중에 객체들을 이전 상태로 복원하는 데 사용할
수 있습니다.</p></figcaption>
</figure>

상태 스냅샷들에 대해 생각해 봅시다. 상태 스냅샷은 정확히 어떻게
생성될까요? 아마도 객체의 모든 필드를 살펴본 후 해당 값들을 스토리지에
복사해야 할 것입니다. 그러나 이는 객체의 내용에 대한 액세스 제한이
상당히 완화되어 있는 경우에만 작동할 것입니다. 불행히도, 대부분의 실제
객체들은 모든 중요한 데이터를 비공개 필드에 숨깁니다.

이 문제는 일단 무시하고, 객체들이 히피족처럼 열린 관계들을 선호해 그들의
상태를 공개했다고 가정해 봅시다. 이렇게 가정하면 일단 위의 문제는
해결되어 원하는 대로 객체들의 상태에 대한 스냅샷들을 생성할 수 있습니다.
하지만 여전히 몇 가지 심각한 문제들이 남아 있습니다. 앞으로 당신이 일부
필드를 추가 또는 제거하거나, 편집기 클래스들을 리팩토링하기로 결정할지도
모르기 때문입니다. 말은 쉬워 보이지만, 그렇게 하려면 영향받은 객체들의
상태를 복사하는 역할을 맡은 클래스들을 변경해야 합니다.

<figure class="image">
<img
src="../../images/patterns/diagrams/memento/problem2-ko3d2c.png?id=c606b8ecb230cbf1759e125367896596"
style="aspect-ratio: 1.43;"
srcset="/images/patterns/diagrams/memento/problem2-ko.png?id=c606b8ecb230cbf1759e125367896596 300w,/images/patterns/diagrams/memento/problem2-ko-1.5x.png?id=515c69e39660de518a9ae770c437b2c3 450w,/images/patterns/diagrams/memento/problem2-ko-2x.png?id=a92a9c3b940f2ef40375cd46a48831da 600w"
sizes="(max-width: 720px) 100vw, 300px" loading="lazy" width="300"
alt="객체의 비공개 상태는 어떻게 복사할까요?" />
<figcaption><p>객체의 비공개 상태는 어떻게 복사할까요?</p></figcaption>
</figure>

그뿐만이 아닙니다. 편집기 상태의 실제 \'스냅샷\'들에 어떤 데이터가
포함되어 있는지 살펴봅시다. 이 안에는 최소한 실제 텍스트, 커서 좌표,
현재 스크롤 위치 등이 포함되어 있을 겁니다. 스냅샷을 만들려면 이러한
값들을 수집한 후 일종의 컨테이너에 넣어야 합니다.

아마도 당신은 이러한 컨테이너 객체들을 기록에 해당하는 어떤 리스트에
많이 저장하게 될 겁니다. 따라서 이 컨테이너들은 결국 한 클래스의
객체들이 될 것입니다. 이 클래스에는 메서드는 거의 없을 테지만, 편집기의
상태를 미러링하는 필드는 많이 있을 겁니다. 다른 객체들이 스냅샷에서
데이터를 읽고 스냅샷에 데이터를 쓸 수 있도록 하려면, 아마도 해당
스냅샷의 필드를 공개해야 할 것입니다. 그러면 편집기의 모든 (비공개 포함)
상태들이 노출될 것이고, 이제 다른 클래스들은 스냅샷 클래스에 발생하는
모든 자그마한 변경에도 영향을 받게 될 것입니다. 편집기의 모든 상태가
노출되지 않았다면 이러한 변경들은 외부 클래스에는 영향을 미치지 않은 채
비공개 필드와 메서드 안에서 변경이 발생하는 것으로 끝났을 겁니다.

이제 교착 상태에 빠진 것 같습니다. 클래스 내부의 세부 정보를 모두
공개하면 클래스가 너무 취약해집니다. 하지만 클래스의 상태에 접근하지
못하게 하면 스냅샷을 생성할 수 없게 됩니다. 그러면 \'실행 취소\'는 대체
어떻게 구현해야 할까요?
:::

::: {.section .solution}
##  해결책 {#solution}

우리가 방금 경험한 모든 문제는 캡슐화의 실패로 인해 발생합니다. 일부
객체들은 원래 해야 할 일보다 더 많은 일들을 수행하려고 합니다. 예를 들어
이러한 객체들은 어떤 작업을 수행하는 데 필요한 데이터를 수집하기 위해
다른 객체들이 실제 작업을 수행하도록 놔두는 대신 그들의 비공개 공간을
침범합니다.

메멘토는 상태 스냅샷들의 생성을 해당 상태의 실제 소유자인
*originator*(오리지네이터) 객체에 위임합니다. 그러면 다른 객체들이
\'외부\'에서 편집기의 상태를 복사하려 시도하는 대신, 자신의 상태에 대해
완전한 접근 권한을 갖는 편집기 클래스가 자체적으로 스냅샷을 생성할 수
있습니다.

이 패턴은 *[메]{.cjk}[멘]{.cjk}[토]{.cjk}*라는 특수 객체에 객체 상태의
복사본을 저장하라고 제안합니다. 메멘토의 내용에는 메멘토를 생성한 객체를
제외한 다른 어떤 객체도 접근할 수 없습니다. 다른 객체들은 메멘토들과
제한된 인터페이스를 사용해 통신해야 합니다. 이러한 인터페이스는 스냅샷의
메타데이터​(생성 시간, 수행한 작업의 이름, 등)​를 가져올 수 있도록 할 수
있지만, 스냅샷에 포함된 원래 객체의 상태는 가져오지 못합니다.

<figure class="image">
<img
src="../../images/patterns/diagrams/memento/solution-ko6a1a.png?id=2311280692fa69436c28b98667f3bcd1"
style="aspect-ratio: 1.36;"
srcset="/images/patterns/diagrams/memento/solution-ko.png?id=2311280692fa69436c28b98667f3bcd1 610w,/images/patterns/diagrams/memento/solution-ko-1.5x.png?id=265c910a602ada3c852d29fd3b47a295 915w,/images/patterns/diagrams/memento/solution-ko-2x.png?id=548f31bc70c3c47936e44690898a12ae 1220w"
sizes="(max-width: 720px) 100vw, 610px" loading="lazy" width="610"
alt="오리지네이터는 메멘토에 대한 전체 접근 권한이 있지만 케어테이커는 메타데이터에만 접근할 수 있습니다." />
<figcaption><p>오리지네이터는 메멘토에 대한 전체 접근 권한이 있지만
케어테이커​(caretaker)​는 메타데이터에만 접근할
수 있습니다.</p></figcaption>
</figure>

이러한 제한 정책을 사용하면 일반적으로
*[케]{.cjk}[어]{.cjk}[테]{.cjk}[이]{.cjk}[커]{.cjk}*라고 하는 다른
객체들 안에 메멘토들을 저장할 수 있습니다. 케어테이커는 제한된
인터페이스를 통해서만 메멘토와 작업하기 때문에 메멘토 내부에 저장된
상태를 변경할 수 없습니다. 동시에 오리지네이터는 메멘토 내부의 모든
필드에 접근할 수 있으므로 언제든지 자신의 이전 상태를 복원할 수
있습니다.

위의 텍스트 편집기 예시의 경우, 별도의 기록 클래스를 만들어 케어테이커의
역할을 하도록 할 수 있습니다. 케어테이커 내부의 메멘토들의 스택은
편집기가 작업을 실행하려고 할 때마다 계속 늘어날 것입니다. 또 당신은
앱의 UI 내에서 이 스택을 렌더링하여 이전에 수행한 작업들의 기록을
사용자에게 표시할 수도 있습니다.

사용자가 실행 취소를 작동시키면 기록은 스택에서 가장 최근의 메멘토를
가져온 후 편집기에 다시 전달하여 롤백을 요청합니다. 편집기는 메멘토에
대한 완전한 접근 권한이 있으므로 메멘토에서 가져온 값들로 자신의 상태를
변경합니다.
:::

:::::::::: {.section .structure-container}
##  구조 {#structure}

::::::::: structure
#### 중첩된 클래스들에 기반한 구현

이 패턴의 고전적인 구현은 수많은 인기 프로그래밍 언어​(예: C++, C# 및
자바)​에서 사용할 수 있는 중첩 클래스에 대한 지원에 의존합니다.

:::: memento-classic
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/memento/structure180c1.png?id=4b4a42363a005b617d4df06689787385"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.87;"
srcset="/images/patterns/diagrams/memento/structure1.png?id=4b4a42363a005b617d4df06689787385 580w,/images/patterns/diagrams/memento/structure1-1.5x.png?id=82cf757f153840c85d27bd63f3f3e133 870w,/images/patterns/diagrams/memento/structure1-2x.png?id=d4e77295e51c2417f22b7abb396d5977 1160w"
sizes="(max-width: 720px) 100vw, 580px" loading="lazy" width="580"
alt="중첩된 클래스들에 기반한 메멘토" /><img
src="../../images/patterns/diagrams/memento/structure1-indexeda7c3.png?id=f79a8356b087ae6b004aec42b787ae2e"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.87;"
srcset="/images/patterns/diagrams/memento/structure1-indexed.png?id=f79a8356b087ae6b004aec42b787ae2e 580w,/images/patterns/diagrams/memento/structure1-indexed-1.5x.png?id=0687dc84dd4c98b4591a70ebd05c4d8e 870w,/images/patterns/diagrams/memento/structure1-indexed-2x.png?id=62fea7bdbc96420568869ea3bd25f6ad 1160w"
sizes="(max-width: 720px) 100vw, 580px" loading="lazy" width="580"
alt="중첩된 클래스들에 기반한 메멘토" />
</figure>
:::
::::

1.  **오리지네이터** 클래스는 자신의 상태에 대한 스냅샷들을 생성할 수
    있으며, 필요시 스냅샷에서 자신의 상태를 복원할 수도 있습니다.

2.  **메멘토**는 오리지네이터의 상태의 스냅샷 역할을 하는 값 객체입니다.
    관행적으로 메멘토는 불변으로 만든 후 생성자를 통해 데이터를 한 번만
    전달합니다.

3.  **케어테이커**는 \'언제\' 그리고 \'왜\' 오리지네이터의 상태를
    캡처해야 하는지 뿐만 아니라 상태가 복원돼야 하는 시기도 알고
    있습니다.

    케어테이커는 메멘토들의 스택을 저장하여 오리지네이터의 기록을 추적할
    수 있습니다. 오리지네이터가 과거로 돌아가야 할 때 케어테이커는 맨
    위의 메멘토를 스택에서 가져온 후 오리지네이터의 복원 메서드에
    전달합니다.

4.  이 구현에서 메멘토 클래스는 오리지네이터 내부에 중첩됩니다. 이것은
    오리지네이터가 메멘토의 필드들과 메서드들이 비공개로 선언된 경우에도
    접근할 수 있도록 합니다. 반면에, 케어테이커는 메멘토의 필드들과
    메서드들에 매우 제한된 접근 권한을 가지므로 메멘토들을 스택에 저장할
    수는 있지만 그들의 상태를 변조할 수는 없습니다.

#### 중간 인터페이스에 기반한 구현

중첩 클래스들을 지원하지 않는 프로그래밍 언어​(예: PHP)​에 적합한 대안적
구현 방식이 있습니다.

:::: memento-empty-interface
::: {.struct-image2 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/memento/structure21ec9.png?id=fcff71cb648389be2e302fbe55e2f847"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/diagrams/memento/structure2.png?id=fcff71cb648389be2e302fbe55e2f847 560w,/images/patterns/diagrams/memento/structure2-1.5x.png?id=5c05495a7ca6545fcc57f70ea7ee904a 840w,/images/patterns/diagrams/memento/structure2-2x.png?id=aa7fb5d0f622d4344a2cb590f437f8c8 1120w"
sizes="(max-width: 720px) 100vw, 560px" loading="lazy" width="560"
alt="중첩 클래스들이 없는 메멘토" /><img
src="../../images/patterns/diagrams/memento/structure2-indexede045.png?id=2c98b4f64b03f2a30e159de31ca9f718"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.6;"
srcset="/images/patterns/diagrams/memento/structure2-indexed.png?id=2c98b4f64b03f2a30e159de31ca9f718 560w,/images/patterns/diagrams/memento/structure2-indexed-1.5x.png?id=1ba6e0f22bb613f3f1dcf86850c3b604 840w,/images/patterns/diagrams/memento/structure2-indexed-2x.png?id=2fb637daef1110dfa89f15b2d4627894 1120w"
sizes="(max-width: 720px) 100vw, 560px" loading="lazy" width="560"
alt="중첩 클래스들이 없는 메멘토" />
</figure>
:::
::::

1.  중첩 클래스들이 없는 경우, 당신은 케어테이커들이 명시적으로 선언된
    중개 인터페이스를 통해서만 메멘토와 작업할 수 있는 규칙을 만들어
    메멘토의 필드들에 대한 접근을 제한할 수 있습니다. 이 인터페이스는
    메멘토의 메타데이터와 관련된 메서드들만 선언합니다.

2.  반면에 오리지네이터들은 메멘토 객체와 직접 작업하여 메멘토 클래스에
    선언된 필드들과 메서드들에 접근할 수 있습니다. 이 접근 방식의 단점은
    메멘토의 모든 구성원을 공개​(public)​로 선언해야 한다는 것입니다.

#### 더 엄격한 캡슐화를 사용한 구현

또 다른 구현이 있는데, 이 구현은 당신이 다른 클래스들이 오리지네이터의
상태를 메멘토를 통해 접근할 가능성을 완전히 제거하고자 할 때 유용합니다.

:::: memento-safe
::: {.struct-image3 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/memento/structure3f138.png?id=6c3ef6a916be8c8ec6d6659d19a6a79f"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.74;"
srcset="/images/patterns/diagrams/memento/structure3.png?id=6c3ef6a916be8c8ec6d6659d19a6a79f 590w,/images/patterns/diagrams/memento/structure3-1.5x.png?id=9bb6d9dd5567bc71d9e93efc9183ef3a 885w,/images/patterns/diagrams/memento/structure3-2x.png?id=988c37f92059457153d26ba3458d371e 1180w"
sizes="(max-width: 720px) 100vw, 590px" loading="lazy" width="590"
alt="엄격한 캡슐화를 사용한 메멘토" /><img
src="../../images/patterns/diagrams/memento/structure3-indexedadf8.png?id=17e84b0ef89a41bb3fb844c8d7a445ad"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.74;"
srcset="/images/patterns/diagrams/memento/structure3-indexed.png?id=17e84b0ef89a41bb3fb844c8d7a445ad 590w,/images/patterns/diagrams/memento/structure3-indexed-1.5x.png?id=c121c75333433b70b9a67b75e536214c 885w,/images/patterns/diagrams/memento/structure3-indexed-2x.png?id=fef9ae2a0151c105976075aafb8939dd 1180w"
sizes="(max-width: 720px) 100vw, 590px" loading="lazy" width="590"
alt="엄격한 캡슐화를 사용한 메멘토" />
</figure>
:::
::::

1.  이 구현 방식을 사용하면 여러 유형의 오리지네이터들과 메멘토들을
    보유할 수 있습니다. 각 오리지네이터는 그에 상응하는 메멘토 클래스와
    함께 작동합니다. 오리지네이터들과 메멘토들은 자신의 상태를
    누구에게도 노출하지 않습니다.

2.  케어테이커들은 이제 메멘토들에 저장된 상태의 변경에 명시적인 제한을
    받습니다. 또 케어테이커 클래스는 복원 메서드가 이제 메멘토 클래스에
    정의되어 있으므로 오리지네이터에게서 독립됩니다.

3.  각 메멘토는 그것을 생성한 오리지네이터와 연결됩니다. 오리지네이터는
    자신의 상태 값들과 함께 자신을 메멘토의 생성자에 전달합니다. 이러한
    클래스 간의 긴밀한 관계 덕분에 메멘토는, 오리지네이터가 적절한
    세터들을 정의했을 경우, 자신의 오리지네이터의 상태를 복원할 수
    있습니다.
:::::::::
::::::::::

::: {.section .pseudocode}
##  의사코드 {#pseudocode}

이 예시에서는 메멘토를 커맨드 패턴과 함께 사용하여 복잡한 텍스트
편집기의 상태의 스냅샷들을 저장하고 필요할 때 스냅샷들로부터 이전 상태를
복원할 수 있도록 합니다.

<figure class="image">
<img
src="../../images/patterns/diagrams/memento/example4526.png?id=fb2196b065f374a1c2a64a0943463760"
style="aspect-ratio: 4.29;"
srcset="/images/patterns/diagrams/memento/example.png?id=fb2196b065f374a1c2a64a0943463760 600w,/images/patterns/diagrams/memento/example-1.5x.png?id=174b686f918a2c49e2545d5573c52d42 900w,/images/patterns/diagrams/memento/example-2x.png?id=41a73f3cc22bc3dd180f53e6968974d4 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="메멘토 구조 예시" />
<figcaption><p>텍스트 편집기 상태에 대한 스냅샷 저장.</p></figcaption>
</figure>

커맨드 객체들은 케어테이커 역할을 합니다. 이 객체들은 커맨드들과 관련된
작업들을 실행하기 전에 편집기의 메멘토를 가져옵니다. 사용자가 가장 최근
커맨드를 실행 취소하려고 하면 편집기는 해당 커맨드에 저장된 메멘토를
사용하여 자신을 이전 상태로 되돌릴 수 있습니다.

메멘토 클래스는 공개된 필드들, 게터​(getter)​들 또는 세터​(setter)​들을
선언하지 않습니다. 따라서 어떤 객체도 자신의 내용을 변경할 수 없습니다.
메멘토들은 자신을 만든 편집기 객체에 연결됩니다. 이것은 메멘토가
데이터를 연결된 편집기 객체의 세터들을 통해 전달하여 해당 편집기의
상태를 복원할 수 있도록 합니다. 메멘토들은 특정 편집자 객체들에 연결되어
있으므로 당신은 당신의 앱이 중앙 집중식 실행 취소 스택을 사용하여 여러
독립 편집기 창을 지원하도록 할 수 있습니다.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// 오리지네이터는 시간이 지남에 따라 변경될 수 있는 어떤 중요한 데이터를
// 보유합니다. 또한 자신의 상태를 메멘토 내부에 저장하는 메서드와 해당 상태를
// 메멘토로부터 복원하는 또 다른 메서드를 정의합니다.
class Editor is
    private field text, curX, curY, selectionWidth

    method setText(text) is
        this.text = text

    method setCursor(x, y) is
        this.curX = x
        this.curY = y

    method setSelectionWidth(width) is
        this.selectionWidth = width

    // 현재 상태를 메멘토 내부에 저장합니다.
    method createSnapshot():Snapshot is
        // 메멘토는 불변 객체입니다. 이 때문에 오리지네이터는 자신의 상태를
        // 메멘토의 생성자 매개변수들에 전달합니다.
        return new Snapshot(this, text, curX, curY, selectionWidth)

// 메멘토 클래스는 편집기의 이전 상태를 저장합니다.
class Snapshot is
    private field editor: Editor
    private field text, curX, curY, selectionWidth

    constructor Snapshot(editor, text, curX, curY, selectionWidth) is
        this.editor = editor
        this.text = text
        this.curX = x
        this.curY = y
        this.selectionWidth = selectionWidth

    // 어느 시점에 메멘토 객체를 사용하여 편집기의 이전 상태를 복원할 수
    // 있습니다.
    method restore() is
        editor.setText(text)
        editor.setCursor(curX, curY)
        editor.setSelectionWidth(selectionWidth)

// 커맨드 객체는 케어테이커 역할을 할 수 있습니다. 그러면 커맨드는 오리지네이터의
// 상태를 변경하기 직전에 메멘토를 얻습니다. 실행 취소가 요청되면 커맨드는
// 메멘토에서 오리지네이터의 상태를 복원합니다.
class Command is
    private field backup: Snapshot

    method makeBackup() is
        backup = editor.createSnapshot()

    method undo() is
        if (backup != null)
            backup.restore()
    // …</code></pre>
</figure>
:::

:::::::: {.section .applicability-container}
##  적용 {#applicability}

::::::: applicability
::: applicability-problem
메멘토는 객체의 이전 상태를 복원할 수 있도록 객체의 상태의 스냅샷들을
생성하려는 경우에 사용하세요.
:::

::: applicability-solution
메멘토는 비공개 필드들을 포함하여 객체의 상태의 전체 복사본들을 만들 수
있도록 하고 이 복사본들을 객체와 별도로 저장할 수 있도록 합니다.
대부분의 개발자는 이 패턴을 \'실행 취소\'의 사용과 관련지어 기억하지만,
트랜잭션들을 처리할 때​(즉, 오류 발생 시 작업을 롤백해야 할 때)​도 필수
불가결한 패턴입니다.
:::

::: applicability-problem
이 패턴은 또 객체의 필드들/게터들/세터들을 직접 접근하는 것이 해당
객체의 캡슐화를 위반할 때 사용하세요.
:::

::: applicability-solution
메멘토는 객체가 스스로 자신의 상태의 스냅샷의 생성을 담당하게 합니다.
다른 객체는 스냅샷을 읽을 수 없으므로 원래 객체의 상태 데이터는
안전합니다.
:::
:::::::
::::::::

::: {.section .checklist}
##  구현방법 {#checklist}

1.  어떤 클래스가 오리지네이터의 역할을 할 것인지 결정하세요. 프로그램이
    이 유형의 중심 객체를 사용하는지 아니면 여러 개의 작은 객체들을
    사용하는지 아는 것이 중요합니다.

2.  메멘토 클래스를 만드세요. 하나씩 오리지네이터 클래스 내부에 선언된
    필드들을 미러링하는 필드들의 집합을 선언하세요.

3.  메멘토 클래스를 변경할 수 없도록 하세요. 메멘토는 생성자를 통해
    데이터를 한 번만 받아야 하며, 그 클래스에는 세터들이 없어야 합니다.

4.  사용하고 있는 프로그래밍 언어가 중첩 클래스를 지원하면 오리지네이터
    내부에 메멘토를 중첩하세요. 그렇지 않은 경우, 메멘토 클래스에서 빈
    인터페이스를 추출한 후 다른 모든 객체가 메멘토를 참조하는 데
    사용하도록 하세요. 인터페이스에 일부 메타데이터 작업을 추가할 수
    있지만 오리지네이터의 상태를 노출해서는 안 됩니다.

5.  오리지네이터 클래스에 메멘토들을 생성하는 메서드를 추가하세요.
    오리지네이터는 자신의 상태를 메멘토의 생성자의 하나 또는 여러
    인수들을 통해 메멘토에게 전달해야 합니다.

    이 메서드의 반환 유형은 (이전 단계에서 추출했다고 가정했을 때) 이전
    단계에서 추출한 인터페이스의 유형이어야 합니다. 메멘토 생성 메서드는
    메멘토 클래스와 직접 작동해야 합니다.

6.  오리지네이터의 클래스에 자신의 상태를 복원하는 메서드를 추가하세요.
    이 메서드는 메멘토 객체를 인수로 받아들여야 합니다. 이전 단계에서
    인터페이스를 추출했다면 이 인터페이스의 유형을 매개변수의 유형으로
    지정하세요. 이 경우, 들어오는 객체를 메멘토 클래스에 타입캐스팅해야
    합니다. 왜냐하면 오리지네이터에게 이 객체에 대한 완전한 접근 권한이
    필요하기 때문입니다.

7.  케어테이커는 커맨드 객체든, 기록이든, 아니면 완전히 다른 무언가를
    나타낼 때 새로운 메멘토들을 오리지네이터로부터 언제 요청해야 하는지,
    이 메멘토들을 어떻게 저장하고, 언제 특정 메멘토로부터 오리지네이터를
    복원해야 하는지를 알아야 합니다.

8.  케어테이커들과 오리지네이터들 간의 연결은 메멘토 클래스로 이동시킬
    수 있습니다. 이 경우, 각 메멘토는 자신을 생성한 오리지네이터와
    연결되어야 합니다. 복원 메서드도 메멘토 클래스로 이동할 수 있습니다.
    그러나 이 모든 것은 메멘토 클래스가 오리지네이터에 중첩되거나
    오리지네이터 클래스가 메멘토 클래스의 상태를 오버라이드하기에 충분한
    세터들을 제공하는 경우에만 의미가 있습니다.
:::

:::::: {.section .pros-cons}
##  장단점 {#pros-cons}

::::: row
::: col-sm-6
-    캡슐화를 위반하지 않고 객체의 상태의 스냅샷들을 생성할 수 있습니다.
-    당신은 케어테이커가 오리지네이터의 상태의 기록을 유지하도록 하여
    오리지네이터의 코드를 단순화할 수 있습니다.
:::

::: col-sm-6
-    클라이언트들이 메멘토들을 너무 자주 생성하면 앱이 많은 RAM을 소모할
    수 있습니다.
-    케어테이커들은 더 이상 쓸모없는 메멘토들을 파괴할 수 있도록
    오리지네이터의 수명주기를 추적해야 합니다.
-    PHP, 파이썬 및 JavaScript와 같은 대부분의 동적 프로그래밍
    언어에서는 메멘토 내의 상태가 그대로 유지된다고 보장할 수 없습니다.
:::
:::::
::::::

::: {.section .relations}
##  다른 패턴과의 관계 {#relations}

-   당신은 \'실행 취소\'를 구현할 때 [커맨드](command.html)와
    [메멘토](memento.html) 패턴을 함께 사용할 수 있습니다. 그러면
    커맨드들은 대상 객체에 대해 다양한 작업을 수행하는 역할을 맡습니다.
    반면, 메멘토들은 커맨드가 실행되기 직전에 해당 객체의 상태를
    저장합니다.

-   [메멘토](memento.html) 패턴을 [반복자](iterator.html) 패턴과 함께
    사용하여 현재 순회 상태를 포착하고 필요한 경우 롤백할 수 있습니다.

-   때로는 [프로토타입](prototype.html)이 [메멘토](memento.html) 패턴의
    더 간단한 대안이 될 수 있으며, 이 패턴은 상태를 기록에 저장하려는
    객체가 간단하고 외부 리소스에 대한 링크가 없거나 링크들이 있어도
    이들을 재설정하기 쉬운 경우에 작동합니다.
:::

::: {.section .implementations}
##  코드 예시 {#implementations}

[![C#으로 작성된
메멘토](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](memento/csharp/example.html "C#으로 작성된 메멘토"){.prog-lang-link}
[![C++로 작성된
메멘토](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](memento/cpp/example.html "C++로 작성된 메멘토"){.prog-lang-link}
[![Go로 작성된
메멘토](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](memento/go/example.html "Go로 작성된 메멘토"){.prog-lang-link}
[![자바로 작성된
메멘토](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](memento/java/example.html "자바로 작성된 메멘토"){.prog-lang-link}
[![PHP로 작성된
메멘토](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](memento/php/example.html "PHP로 작성된 메멘토"){.prog-lang-link}
[![파이썬으로 작성된
메멘토](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](memento/python/example.html "파이썬으로 작성된 메멘토"){.prog-lang-link}
[![루비로 작성된
메멘토](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](memento/ruby/example.html "루비로 작성된 메멘토"){.prog-lang-link}
[![러스트로 작성된
메멘토](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](memento/rust/example.html "러스트로 작성된 메멘토"){.prog-lang-link}
[![스위프트로 작성된
메멘토](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](memento/swift/example.html "스위프트로 작성된 메멘토"){.prog-lang-link}
[![타입스크립트로 작성된
메멘토](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](memento/typescript/example.html "타입스크립트로 작성된 메멘토"){.prog-lang-link}
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

[옵서버 []{.fa .fa-arrow-right}](observer.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### 돌아가기

[[]{.fa .fa-arrow-left} 중재자](mediator.html){.btn .btn-default
rel="prev"}
:::
::::::::::::::::::::::::::::::::::::

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
::::::::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::::::::
