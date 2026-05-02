::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [디자인
패턴들](../design-patterns.html){.type} / [행동
패턴](behavioral-patterns.html){.type}
:::

# 커맨드 패턴 {#커맨드-패턴 .title}

::: aka
다음 이름으로도 불립니다:
[액션, ]{style="display:inline-block"}[트랜잭션, ]{style="display:inline-block"}[Command]{style="display:inline-block"}
:::

::: {.section .intent}
##  의도 {#intent}

**커맨드**는 요청을 요청에 대한 모든 정보가 포함된 독립실행형 객체로
변환하는 행동 디자인 패턴입니다. 이 변환은 다양한 요청들이 있는
메서드들을 인수화 할 수 있도록 하며, 요청의 실행을 지연 또는 대기열에
넣을 수 있도록 하고, 또 실행 취소할 수 있는 작업을 지원할 수
있도록 합니다.

<figure class="image">
<img
src="../../images/patterns/content/command/command-koc9c1.png?id=5791740c487fa15d9f3539bb143c5cdc"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/command/command-ko.png?id=5791740c487fa15d9f3539bb143c5cdc 640w,/images/patterns/content/command/command-ko-1.5x.png?id=ff486d941b865b341d60aec5a29e0e35 960w,/images/patterns/content/command/command-ko-2x.png?id=3d294dffa18426b0e94157dcfc75f1c4 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="커맨드 디자인 패턴" />
</figure>
:::

::: {.section .problem}
##  문제 {#problem}

당신이 새로운 텍스트 편집기 앱을 개발하고 있다고 상상해 봅시다. 당신이
현재 하는 작업은 편집기의 다양한 작업을 위한 여러 버튼이 있는 도구
모음​(툴바)​을 만드는 것입니다. 당신은 도구 모음의 버튼들과 다양한 대화
상자들의 일반 버튼들에 사용할 수 있는 매우 깔끔한 `Button`​(버튼)
클래스를 만들었습니다.

<figure class="image">
<img
src="../../images/patterns/diagrams/command/problem163e2.png?id=84189315a0e8d91da262792979005ab4"
style="aspect-ratio: 0.88;"
srcset="/images/patterns/diagrams/command/problem1.png?id=84189315a0e8d91da262792979005ab4 230w,/images/patterns/diagrams/command/problem1-1.5x.png?id=77bf0bc58e5c9c9b8369bc4f8dec457f 345w,/images/patterns/diagrams/command/problem1-2x.png?id=af4c4e9c1d1b4fa2c4104c5f6bb18719 460w"
sizes="(max-width: 720px) 100vw, 230px" loading="lazy" width="230"
alt="커맨드 패턴으로 해결된 문제" />
<figcaption><p>앱의 모든 버튼은 같은
클래스에서 파생됩니다.</p></figcaption>
</figure>

이 버튼들은 모두 비슷해 보이지만 각각 다른 기능들을 수행해야 합니다.
그러면 이 버튼들의 다양한 클릭 핸들러들에 대한 코드는 어디에 두겠습니까?
가장 간단한 해결책은 버튼이 사용되는 각 위치에 수많은 자식 클래스들을
만드는 것입니다. 이러한 자식 클래스들에는 버튼 클릭 시 실행되어야 하는
코드가 포함됩니다.

<figure class="image">
<img
src="../../images/patterns/diagrams/command/problem2f2f9.png?id=f0e33da1842b3a3ee3b4857de0b6ec93"
style="aspect-ratio: 2.11;"
srcset="/images/patterns/diagrams/command/problem2.png?id=f0e33da1842b3a3ee3b4857de0b6ec93 400w,/images/patterns/diagrams/command/problem2-1.5x.png?id=7ae15e2b07d5d076a878d99c3ed8ac32 600w,/images/patterns/diagrams/command/problem2-2x.png?id=5eea7d0f6247da011150bb63e423f405 800w"
sizes="(max-width: 720px) 100vw, 400px" loading="lazy" width="400"
alt="많은 버튼 자식 클래스들" />
<figcaption><p>많은 버튼 자식 클래스들이 있습니다. 무엇이 잘못될
수 있을까요?</p></figcaption>
</figure>

머지않아 당신은 이 접근 방식에 심각한 결함이 있음을 깨닫게 됩니다. 일단,
당신은 이제 엄청난 수의 자식 클래스들이 있으며, 기초 `Button` 클래스를
수정할 때마다 이러한 자식 클래스의 코드를 깨뜨릴 위험이 있습니다. 간단히
말해서, 그래픽 사용자 인터페이스 코드는 비즈니스 로직의 불안정한 코드에
어색하게 의존하게 되었습니다.

<figure class="image">
<img
src="../../images/patterns/diagrams/command/problem3-koaaf3.png?id=d55040a179b8fb64d561507ec22eab0b"
style="aspect-ratio: 4.36;"
srcset="/images/patterns/diagrams/command/problem3-ko.png?id=d55040a179b8fb64d561507ec22eab0b 480w,/images/patterns/diagrams/command/problem3-ko-1.5x.png?id=ceac8e2566c685707871495ae76bf532 720w,/images/patterns/diagrams/command/problem3-ko-2x.png?id=bd8444f933cd1d5dc7e3b35647b0faa9 960w"
sizes="(max-width: 720px) 100vw, 480px" loading="lazy" width="480"
alt="여러 클래스가 같은 기능을 구현합니다" />
<figcaption><p>여러 클래스가 같은 기능을 구현합니다.</p></figcaption>
</figure>

그리고 당신이 고려해야 할 최악의 사항은, 텍스트 복사/붙여넣기와 같은
일부 작업은 여러 위치에서 호출될 수 있다는 사실입니다. 예를 들어,
사용자는 무언가를 복사하기 위하여 도구 모음에서 작은 \'복사\' 버튼을
클릭하거나 콘텍스트 메뉴를 통해 무언가를 복사하거나 키보드에서
`Ctrl+C`를 누를 수 있습니다.

당신의 앱에 처음에 하나의 도구 모음만 있었을 때는 다양한 작업의 구현을
버튼의 자식 클래스들에 배치해도 괜찮았습니다. 즉, `Copy­Button` 자식
클래스의 내부에 텍스트를 복사하는 코드가 있어도 괜찮았습니다. 그러나
복사를 할 수 있도록 하는 콘텍스트 메뉴, 바로 가기 및 기타 항목들을
구현하면 당신은 이제 해당 작업의 코드를 많은 클래스에 복제하거나 버튼에
의존하는 메뉴들을 만들어야 하는데, 이것은 오히려 더 나쁜 옵션입니다.
:::

::: {.section .solution}
##  해결책 {#solution}

올바른 소프트웨어 디자인은 종종 *[관]{.cjk}[심]{.cjk}[사]{.cjk}
[분]{.cjk}[리]{.cjk}[의]{.cjk} [원]{.cjk}[칙]{.cjk}*을 기반으로 합니다.
가장 일반적인 예로는 그래픽 사용자 인터페이스용 레이어와 비즈니스 로직용
레이어의 분리입니다. 그래픽 사용자 인터페이스용 레이어는 모든 입력을
캡처하고 화면에 아름다운 그림을 렌더링하고 사용자와 앱이 수행하는 작업의
결과를 나타내는 역할을 합니다. 그러나 달의 행성 궤도를 계산하거나 연간
보고서를 작성하는 것과 같은 중요한 작업을 수행할 때 그래픽 사용자
인터페이스 레이어는 비즈니스 논리의 배경 레이어들에 작업을 위임합니다.

위의 내용은 코드로 다음과 같이 표현될 수 있습니다. 그래픽 사용자
인터페이스 객체가 비즈니스 논리 객체의 메서드를 호출하고 일부 인수를
전달합니다. 위 프로세스는 일반적으로 한 객체가 다른 객체에
*[요]{.cjk}[청]{.cjk}*을 보내는 것이라고 불립니다.

<figure class="image">
<img
src="../../images/patterns/diagrams/command/solution1-koe72e.png?id=80379a2c48102030a47c2c66f3dafe22"
style="aspect-ratio: 2.47;"
srcset="/images/patterns/diagrams/command/solution1-ko.png?id=80379a2c48102030a47c2c66f3dafe22 470w,/images/patterns/diagrams/command/solution1-ko-1.5x.png?id=ed041354f69a4c51d5a3e6da6f8db541 705w,/images/patterns/diagrams/command/solution1-ko-2x.png?id=7e9bb5904e9e4f56f416d62e24df126d 940w"
sizes="(max-width: 720px) 100vw, 470px" loading="lazy" width="470"
alt="그래픽 사용자 인터페이스 객체들은 비즈니스 논리 객체들을 직접 액세스할 수 있습니다" />
<figcaption><p>그래픽 사용자 인터페이스 객체들은 비즈니스 논리 객체들에
직접 접근할 수 있습니다.</p></figcaption>
</figure>

커맨드 패턴은 그래픽 사용자 인터페이스 객체들이 이러한 요청을 직접
보내서는 안된다고 합니다. 대신 모든 요청 세부 정보들​(예: 호출되는 객체,
메서드 이름 및 인수 리스트)​을 요청을 작동시키는 단일 메서드를 가진
별도의 커맨드 클래스로 추출하라고 제안합니다.

커맨드 객체들은 다양한 그래픽 사용자 인터페이스 객체들과 비즈니스 논리
객체들 간의 링크 역할을 합니다. 이제부터 그래픽 사용자 인터페이스 객체는
어떤 비즈니스 논리 객체가 요청을 받을지와 이 요청이 어떻게 처리할지에
대하여 알 필요가 없습니다. 그래픽 사용자 인터페이스 객체는 커맨드를
작동시킬 뿐이며, 그렇게 작동된 커맨드는 모든 세부 사항을 처리합니다.

<figure class="image">
<img
src="../../images/patterns/diagrams/command/solution2-koc316.png?id=bbc838103015bf141177e4046a564d42"
style="aspect-ratio: 2.89;"
srcset="/images/patterns/diagrams/command/solution2-ko.png?id=bbc838103015bf141177e4046a564d42 550w,/images/patterns/diagrams/command/solution2-ko-1.5x.png?id=7fcdb18490592b88a7200e13373ebf6f 825w,/images/patterns/diagrams/command/solution2-ko-2x.png?id=16be0f239e15aae4dc8a8432abe47312 1100w"
sizes="(max-width: 720px) 100vw, 550px" loading="lazy" width="550"
alt="커맨드를 통해 비즈니스 논리 레이어를 액세스합니다." />
<figcaption><p>커맨드를 통해 비즈니스 논리
레이어를 접근합니다.</p></figcaption>
</figure>

이제 다음 단계는 당신의 커맨드들이 같은 인터페이스를 구현하도록 하는
것입니다. 일반적으로 커맨드는 매개 변수들을 받지 않는 단일 실행
메서드만을 가집니다. 이 인터페이스는 다양한 커맨드들을 커맨드들의 구상
클래스들과 결합하지 않고 같은 요청 발신자와 사용할 수 있게 해줍니다.
이제 당신은 발신자에 연결된 커맨드 객체들을 전환할 수 있으며, 그렇게
하여 런타임에 발신자의 행동을 변경할 수 있습니다.

당신은 요청 매개변수들이 빠져있다는 점을 눈치채셨을 것입니다. 그래픽
사용자 인터페이스 객체가 비즈니스 레이어 객체에 일부 매개변수들을
제공했을 수 있습니다. 커맨드 실행 메서드에 매개변수들이 없는데, 그러면
어떻게 요청의 세부 정보를 수신자에게 전달할 수 있을까요? 커맨드를 이러한
데이터로 미리 설정해놓거나, 이 데이터를 자체적으로 가져올 수 있도록 해야
합니다.

<figure class="image">
<img
src="../../images/patterns/diagrams/command/solution3-kof690.png?id=5c362a3600f0261ba00bd334a3bd7e9c"
style="aspect-ratio: 1.83;"
srcset="/images/patterns/diagrams/command/solution3-ko.png?id=5c362a3600f0261ba00bd334a3bd7e9c 440w,/images/patterns/diagrams/command/solution3-ko-1.5x.png?id=36a515dee0977c97d0b6dac986e6e48f 660w,/images/patterns/diagrams/command/solution3-ko-2x.png?id=38ef66fe04f933ca39f05cfd5e5209b6 880w"
sizes="(max-width: 720px) 100vw, 440px" loading="lazy" width="440"
alt="그래픽 사용자 인터페이스 객체들은 작업을 커맨드들에 위임합니다" />
<figcaption><p>그래픽 사용자 인터페이스 객체들은 작업을
커맨드들에 위임합니다.</p></figcaption>
</figure>

다시 당신의 텍스트 편집기를 살펴봅시다. 커맨드 패턴을 적용한 후에는 더
이상 다양한 클릭 행동들을 구현하기 위한 여러 버튼 자식 클래스들이
필요하지 않습니다. 기초 `Button` 클래스에 커맨드 객체에 대한 참조를
저장하는 단일 필드를 넣은 후 이 버튼이 클릭 될 때 그 커맨드를 시행하도록
하면 됩니다.

이제 가능한 모든 작업에 대해 많은 커맨드 클래스들을 구현하고 이
클래스들을 버튼의 의도된 동작에 따라 특정 버튼들과 연결해야 합니다.

메뉴, 단축키 또는 대화 상자와 같은 다른 그래픽 사용자 인터페이스
요소들도 같은 방식으로 구현할 수 있습니다. 이들은 사용자가 그래픽 사용자
인터페이스 요소와 상호 작용할 때 실행되는 커맨드에 연결될 것입니다.
지금쯤 짐작하셨겠지만 같은 작업과 관련된 요소들은 같은 커맨드들에
연결되어 코드 중복을 방지할 것입니다.

결과적으로 커맨드들은 그래픽 사용자 인터페이스 레이어와 비즈니스 로직
레이어 간의 결합도를 줄이는 편리한 중간 레이어들이 됩니다. 그리고 이것은
커맨드 패턴이 제공할 수 있는 이점의 극히 일부에 불과합니다!
:::

::: {.section .analogy}
##  실제상황 적용 {#analogy}

<figure class="image">
<img
src="../../images/patterns/content/command/command-comic-16e6f.png?id=551df832f445080976f3116e0dc120c9"
style="aspect-ratio: 2;"
srcset="/images/patterns/content/command/command-comic-1.png?id=551df832f445080976f3116e0dc120c9 600w,/images/patterns/content/command/command-comic-1-1.5x.png?id=82dc5c38ce372ed4dfd4a37c04faeb72 900w,/images/patterns/content/command/command-comic-1-2x.png?id=47b3c00b2cfbda7157a1ed9da6ce2948 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="레스토랑에서 주문하기" />
<figcaption><p>레스토랑에서 주문하기.</p></figcaption>
</figure>

당신은 도시를 한참 걷다가 멋진 레스토랑에 도착하여 창가 테이블에
앉습니다. 친절한 웨이터가 다가와 신속하게 당신의 주문을 받아 종이에
적습니다. 웨이터는 부엌으로 가서 주문을 벽에 붙입니다. 잠시 후
요리사에게 주문이 전달되고 요리사는 주문을 읽고 그에 따라 음식을
요리합니다. 요리사는 주문과 함께 식사를 트레이에 놓습니다. 웨이터는
트레이를 발견한 후 당신이 주문한 대로 식사가 요리되었는지 확인하고
완성된 주문을 당신의 테이블로 가져옵니다.

종이에 적힌 주문은 커맨드 역할을 합니다. 이 주문은 요리사가 요리할
준비가 될 때까지 대기열에 남아 있습니다. 주문에는 식사를 요리하는 데
필요한 모든 관련 정보가 포함되어 있습니다. 이를 통해 요리사는 당신에게서
주문 세부 사항을 직접 전달받는 대신 바로 요리를 시작할 수 있습니다.
:::

::::: {.section .structure-container}
##  구조 {#structure}

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/command/structure44ec.png?id=1cd7833638f4c43630f4a84017d31195"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.7;"
srcset="/images/patterns/diagrams/command/structure.png?id=1cd7833638f4c43630f4a84017d31195 630w,/images/patterns/diagrams/command/structure-1.5x.png?id=6e5b68277c7747b22266e408891d5841 945w,/images/patterns/diagrams/command/structure-2x.png?id=1dfaa84354ffe49ef7ad46ce069482d2 1260w"
sizes="(max-width: 720px) 100vw, 630px" loading="lazy" width="630"
alt="커맨드 디자인 패턴의 구조" /><img
src="../../images/patterns/diagrams/command/structure-indexed49b8.png?id=95529d7282dc7bc1c5bc443423b1cf4f"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.64;"
srcset="/images/patterns/diagrams/command/structure-indexed.png?id=95529d7282dc7bc1c5bc443423b1cf4f 640w,/images/patterns/diagrams/command/structure-indexed-1.5x.png?id=29d6c5c7a06da2747ed756c0ddad6169 960w,/images/patterns/diagrams/command/structure-indexed-2x.png?id=e4cc286a44768c7d060af33497da7df6 1280w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="640"
alt="커맨드 디자인 패턴의 구조" />
</figure>
:::

1.  **발송자** 클래스​(*invoker*라고도 함)​는 요청들을 시작하는 역할을
    합니다. 이 클래스에는 커맨드 객체에 대한 참조를 저장하기 위한 필드가
    있어야 합니다. 발송자는 요청을 수신자에게 직접 보내는 대신 해당
    커맨드를 작동시킵니다. 참고로 발송자는 커맨드 객체를 생성할 책임이
    없으며 일반적으로 생성자를 통해 클라이언트로부터 미리 생성된
    커맨드를 받습니다.

2.  **커맨드** 인터페이스는 일반적으로 커맨드를 실행하기 위한 단일
    메서드만을 선언합니다.

3.  **구상 커맨드**들은 다양한 유형의 요청을 구현합니다. 구상 커맨드는
    자체적으로 작업을 수행해서는 안 되며, 대신 비즈니스 논리 객체 중
    하나에 호출을 전달해야 합니다. 그러나 코드를 단순화하기 위해 이러한
    클래스들은 병합될 수 있습니다.

    수신 객체에서 메서드를 실행하는 데 필요한 매개 변수들은 구상
    커맨드의 필드들로 선언할 수 있습니다. 생성자를 통해서만 이러한
    필드들의 초기화를 허용함으로써 커맨드 객체들을 불변으로 만들 수
    있습니다.

4.  **수신자** 클래스에는 일부 비즈니스 로직이 포함되어 있습니다. 거의
    모든 객체는 수신자 역할을 할 수 있습니다. 대부분의 커맨드들은 요청이
    수신자에게 전달되는 방법에 대한 세부 정보만 처리하는 반면 수신자
    자체는 실제 작업을 수행합니다.

5.  **클라이언트**는 구상 커맨드 객체들을 만들고 설정합니다.
    클라이언트는 수신자 인스턴스를 포함한 모든 요청 매개변수들을
    커맨드의 생성자로 전달해야 하며 그렇게 만들어진 커맨드는 하나 또는
    여러 발송자와 연관될 수 있습니다.
::::
:::::

::: {.section .pseudocode}
##  의사코드 {#pseudocode}

이 예시에서 **커맨드** 패턴은 실행된 작업의 기록을 추적하는 데 도움을
주며 필요한 경우 작업을 되돌릴 수 있도록 합니다.

<figure class="image">
<img
src="../../images/patterns/diagrams/command/example31bf.png?id=1f42c8395fe54d0e409026b91881e2a0"
style="aspect-ratio: 1.07;"
srcset="/images/patterns/diagrams/command/example.png?id=1f42c8395fe54d0e409026b91881e2a0 640w,/images/patterns/diagrams/command/example-1.5x.png?id=435055a78a82c005eebb1bef869998ae 960w,/images/patterns/diagrams/command/example-2x.png?id=ed44dfd9b02eccf1ae7e2714d018ed36 1280w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="640"
alt="커맨드 패턴의 구조 예시" />
<figcaption><p>텍스트 편집기에서 실행 취소될 수
있는 작업들.</p></figcaption>
</figure>

잘라내기 및 붙여넣기 등의 편집기 상태를 변경하는 커맨드들은 커맨드와
관련된 작업을 실행하기 전에 편집기 상태의 백업 복사본을 만듭니다. 어떤
커맨드가 실행되고 나면, 그 커맨드는 해당 지점에서 편집기 상태의 백업
복사본과 함께 커맨드 기록​(커맨드 객체들의 스택)​에 배치됩니다. 나중에
사용자가 작업을 되돌려야 하는 경우 앱은 기록에서 가장 최근 커맨드를
가져와 편집기 상태의 관련 백업을 읽은 후 작업을 되돌릴 수 있습니다.

클라이언트 코드​(그래픽 사용자 인터페이스 요소들, 커맨드 기록 등)​는
커맨드 인터페이스를 통해 커맨드와 함께 작동하기 때문에 구상 커맨드
클래스들에 결합하지 않습니다. 이러한 접근 방식을 사용하면 기존 코드를
손상하지 않고 앱에 새 커맨드들을 도입할 수 있습니다.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// 기초 커맨드 클래스는 모든 구상 커맨드에 대한 공통 인터페이스를 정의합니다.
abstract class Command is
    protected field app: Application
    protected field editor: Editor
    protected field backup: text

    constructor Command(app: Application, editor: Editor) is
        this.app = app
        this.editor = editor

    // 편집기의 상태에 대한 백업을 만드세요.
    method saveBackup() is
        backup = editor.text

    // 편집기의 상태를 복원하세요.
    method undo() is
        editor.text = backup

    // 실행 메서드는 모든 구상 커맨드들이 자체 구현을 제공하도록 강제하기 위해
    // 추상으로 선언됩니다. 이 메서드는 커맨드가 편집기의 상태를 변경하는지에 따라
    // 진실 또는 거짓을 반환해야 합니다.
    abstract method execute()


// 구상 커맨드들은 여기에 배치됩니다.
class CopyCommand extends Command is
    // 복사 커맨드는 편집기의 상태를 변경하지 않으므로 기록에 저장되지 않습니다.
    method execute() is
        app.clipboard = editor.getSelection()
        return false

class CutCommand extends Command is
    // cut 커맨드는 편집기의 상태를 변경하므로 기록에 반드시 저장되어야 하며,
    // 메서드가 true를 반환하는 한 저장됩니다.
    method execute() is
        saveBackup()
        app.clipboard = editor.getSelection()
        editor.deleteSelection()
        return true

class PasteCommand extends Command is
    method execute() is
        saveBackup()
        editor.replaceSelection(app.clipboard)
        return true

// 실행취소 작업도 커맨드입니다.
class UndoCommand extends Command is
    method execute() is
        app.undo()
        return false


// 글로벌 커맨드 기록도 스택일 뿐입니다.
class CommandHistory is
    private field history: array of Command

    // 후입 …
    method push(c: Command) is
        // 커맨드를 기록 배열의 끝으로 푸시하세요.

    // … 선출
    method pop():Command is
        // 기록에서 가장 최근 명령을 가져오세요.


// 편집기 클래스에는 실제 텍스트 편집 기능이 있습니다. 이는 수신기의 역할을
// 합니다. 모든 커맨드들은 결국 편집기의 메서드들에 실행을 위임합니다.
class Editor is
    field text: string

    method getSelection() is
        // 선택된 텍스트를 반환하세요.

    method deleteSelection() is
        // 선택된 텍스트를 삭제하세요.

    method replaceSelection(text) is
        // 현재 위치에 클립보드의 내용을 삽입하세요.


// 앱 클래스는 객체 관계들을 설정하며 발신자 역할을 합니다. 이것은 무언가를
// 수행해야 할 때 커맨드 객체를 만들고 실행합니다.
class Application is
    field clipboard: string
    field editors: array of Editors
    field activeEditor: Editor
    field history: CommandHistory

    // 사용자 인터페이스 객체들에 커맨드들을 할당하는 코드는 다음과 같을 수
    // 있습니다.
    method createUI() is
        // …
        copy = function() { executeCommand(
            new CopyCommand(this, activeEditor)) }
        copyButton.setCommand(copy)
        shortcuts.onKeyPress(&quot;Ctrl+C&quot;, copy)

        cut = function() { executeCommand(
            new CutCommand(this, activeEditor)) }
        cutButton.setCommand(cut)
        shortcuts.onKeyPress(&quot;Ctrl+X&quot;, cut)

        paste = function() { executeCommand(
            new PasteCommand(this, activeEditor)) }
        pasteButton.setCommand(paste)
        shortcuts.onKeyPress(&quot;Ctrl+V&quot;, paste)

        undo = function() { executeCommand(
            new UndoCommand(this, activeEditor)) }
        undoButton.setCommand(undo)
        shortcuts.onKeyPress(&quot;Ctrl+Z&quot;, undo)

    // 커맨드를 실행하여 기록에 추가해야 하는지 확인하세요.
    method executeCommand(command) is
        if (command.execute())
            history.push(command)

    // 기록에서 가장 최근의 커맨드를 가져와서 그의 실행 취소 메서드를 실행하세요.
    // 참고로 우리는 이 커맨드의 클래스를 알지 못한다는 사실에 유념하세요.
    // 하지만 몰라도 상관없는데, 그 이유는 커맨드가 자신의 작업을 실행 취소하는
    // 법을 알기 때문입니다.
    method undo() is
        command = history.pop()
        if (command != null)
            command.undo()</code></pre>
</figure>
:::

:::::::::: {.section .applicability-container}
##  적용 {#applicability}

::::::::: applicability
::: applicability-problem
작업들로 객체를 매개변수화하려는 경우 커맨드 패턴을 사용하세요.
:::

::: applicability-solution
커맨드 패턴은 특정 메서드 호출을 독립실행형 객체로 전환할 수 있습니다.
이 변경을 통해 당신은 이제 커맨드들을 메서드 인수들로 전달하고, 이들을
다른 객체들의 내부에 저장하고, 런타임에 연결된 커맨드를 전환하는 등의
여러 흥미로운 작업을 진행할 수 있습니다.

예를 들어 당신은 콘텍스트 메뉴​(상황에 맞는 메뉴)​와 같은 그래픽 사용자
인터페이스 컴포넌트를 개발 중이고, 앱의 사용자들이 최종 사용자가 하나의
항목을 클릭했을 때 작업이 시작되는 메뉴 항목들을 설정할 수 있도록 만들고
싶어 합니다.
:::

::: applicability-problem
커맨드 패턴은 작업들의 실행을 예약하거나, 작업들을 대기열에 넣거나
작업들을 원격으로 실행하려는 경우에 사용하세요.
:::

::: applicability-solution
다른 여느 객체와 마찬가지로 커맨드는 직렬화될 수 있습니다. 직렬화는
파일이나 데이터베이스에 쉽게 쓸 수 있는 문자열로 변환하는 행위입니다.
나중에 이 문자열은 초기 커맨드 객체로 복원될 수 있습니다. 따라서
커맨드의 실행을 지연하고 예약할 수 있습니다. 또 같은 방식으로 네트워크를
통해 커맨드를 대기열에 추가하거나 로그​(기록) 하거나 전송할 수 있습니다.
:::

::: applicability-problem
커맨드 패턴은 되돌릴 수 있는 작업을 구현하려고 할 때 사용하세요.
:::

::: applicability-solution
실행 취소/다시 실행을 구현하는 방법에는 여러 가지가 있지만, 커맨드
패턴이 아마도 가장 많이 사용되는 패턴일 것입니다.

작업을 되돌리려면 수행한 작업의 기록을 구현해야 합니다. 커맨드 기록은 앱
상태의 관련 백업들과 함께 실행된 모든 커맨드 객체들을 포함하는
스택입니다.

이 메서드에는 두 가지 단점이 있습니다. 첫째, 앱 일부가 비공개일 수
있으므로 앱의 상태를 저장하는 것이 쉽지 않습니다. 이 문제는
[메멘토](memento.html) 패턴으로 완화할 수 있습니다.

둘째, 상태 백업들은 상당히 많은 RAM을 소모할 수 있습니다. 따라서 때로는
대안적 구현에 의존할 수 있습니다. 예를 들어 과거 상태를 복원하는 대신
커맨드가 작업을 역으로 수행할 수 있습니다. 하지만 역으로 작업을
수행하는데도 대가가 있습니다. 구현하기 어렵거나 심지어 불가능할 수도
있습니다.
:::
:::::::::
::::::::::

::: {.section .checklist}
##  구현방법 {#checklist}

1.  단일 실행 메서드로 커맨드 인터페이스를 선언하세요.

2.  요청들을 커맨드 인터페이스를 구현하는 구상 커맨드 클래스들로
    추출하기 시작하세요. 각 클래스에는 실제 수신자 객체에 대한 참조와
    함께 요청 인수들을 저장하기 위한 필드들의 집합이 있어야 합니다.
    이러한 모든 값은 커맨드의 생성자를 통해 초기화되어야 합니다.

3.  *[발]{.cjk}[송]{.cjk}[자]{.cjk}* 역할을 할 클래스들을 식별하세요.
    이러한 클래스들에 커맨드들을 저장하기 위한 필드들을 추가하세요.
    발송자들은 커맨드 인터페이스를 통해서만 커맨드들과 통신해야 합니다.
    발송자들은 일반적으로 자체적으로 커맨드 객체들을 생성하지 않고
    클라이언트 코드에서 가져옵니다.

4.  수신자에게 직접 요청을 보내는 대신 커맨드를 실행하도록 발송자들을
    변경하세요.

5.  클라이언트는 다음 순서로 객체들을 초기화해야 합니다.

    -   수신자들을 만드세요.
    -   커맨드들을 만들고 필요한 경우 수신자들과 연관시키세요.
    -   발송자들을 만들고 특정 커맨드들과 연관시키세요.
:::

:::::: {.section .pros-cons}
##  장단점 {#pros-cons}

::::: row
::: col-sm-6
-    *[단]{.cjk}[일]{.cjk} [책]{.cjk}[임]{.cjk} [원]{.cjk}[칙]{.cjk}*.
    작업을 호출하는 클래스들을 이러한 작업을 수행하는 클래스들로부터
    분리할 수 있습니다.
-    *[개]{.cjk}[방]{.cjk}/[폐]{.cjk}[쇄]{.cjk} [원]{.cjk}[칙]{.cjk}*.
    기존 클라이언트 코드를 손상하지 않고 앱에 새 커맨드들을 도입할 수
    있습니다.
-    실행 취소/다시 실행을 구현할 수 있습니다.
-    작업들의 지연된 실행을 구현할 수 있습니다.
-    간단한 커맨드들의 집합을 복잡한 커맨드로 조합할 수 있습니다.
:::

::: col-sm-6
-    발송자와 수신자 사이에 완전히 새로운 레이어를 도입하기 때문에
    코드가 더 복잡해질 수 있습니다.
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

-   [책임 연쇄](chain-of-responsibility.html) 패턴의 핸들러들은
    [커맨드](command.html)로 구현할 수 있습니다. 그러면 당신은 많은
    다양한 작업을 같은 콘텍스트 객체에 대해 실행할 수 있으며, 해당
    콘텍스트 객체는 요청의 역할을 합니다. 여기에서의 요청은 처리
    메서드의 매개변수를 의미합니다.

    그러나 요청 자체가 *[커]{.cjk}[맨]{.cjk}[드]{.cjk}* 객체인 다른 접근
    방식이 있습니다. 이 접근 방식을 사용하면 당신은 같은 작업을 체인에
    연결된 일련의 서로 다른 콘텍스트들에서 실행할 수 있습니다.

-   당신은 \'실행 취소\'를 구현할 때 [커맨드](command.html)와
    [메멘토](memento.html) 패턴을 함께 사용할 수 있습니다. 그러면
    커맨드들은 대상 객체에 대해 다양한 작업을 수행하는 역할을 맡습니다.
    반면, 메멘토들은 커맨드가 실행되기 직전에 해당 객체의 상태를
    저장합니다.

-   [커맨드](command.html)와 [전략](strategy.html) 패턴은 비슷해 보일 수
    있습니다. 왜냐하면 둘 다 어떤 작업으로 객체를 매개변수화하는 데
    사용할 수 있기 때문입니다. 그러나 이 둘의 의도는 매우 다릅니다.

    -   당신은 *[커]{.cjk}[맨]{.cjk}[드]{.cjk}*를 사용하여 모든 작업을
        객체로 변환할 수 있습니다. 작업의 매개변수들은 해당 객체의
        필드들이 됩니다. 이 변환은 작업의 실행을 연기하고, 해당 작업을
        대기열에 넣고, 커맨드들의 기록을 저장한 후 해당 커맨드들을 원격
        서비스에 보내는 등의 작업을 가능하게 합니다.

    -   반면에 *[전]{.cjk}[략]{.cjk} [패]{.cjk}[턴]{.cjk}*은 일반적으로
        같은 작업을 수행하는 다양한 방법을 설명하므로 단일 콘텍스트
        클래스 내에서 이러한 알고리즘들을 교환할 수 있도록 합니다.

-   [프로토타입](prototype.html)은 [커맨드](command.html) 패턴의
    복사본들을 기록에 저장해야 할 때 도움이 될 수 있습니다.

-   [비지터](visitor.html) 패턴은 [커맨드](command.html) 패턴의 강력한
    버전으로 취급할 수 있습니다. 비지터 패턴의 객체들은 다른 클래스들의
    다양한 객체에 대한 작업을 실행할 수 있습니다.
:::

::: {.section .implementations}
##  코드 예시 {#implementations}

[![C#으로 작성된
커맨드](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](command/csharp/example.html "C#으로 작성된 커맨드"){.prog-lang-link}
[![C++로 작성된
커맨드](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](command/cpp/example.html "C++로 작성된 커맨드"){.prog-lang-link}
[![Go로 작성된
커맨드](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](command/go/example.html "Go로 작성된 커맨드"){.prog-lang-link}
[![자바로 작성된
커맨드](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](command/java/example.html "자바로 작성된 커맨드"){.prog-lang-link}
[![PHP로 작성된
커맨드](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](command/php/example.html "PHP로 작성된 커맨드"){.prog-lang-link}
[![파이썬으로 작성된
커맨드](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](command/python/example.html "파이썬으로 작성된 커맨드"){.prog-lang-link}
[![루비로 작성된
커맨드](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](command/ruby/example.html "루비로 작성된 커맨드"){.prog-lang-link}
[![러스트로 작성된
커맨드](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](command/rust/example.html "러스트로 작성된 커맨드"){.prog-lang-link}
[![스위프트로 작성된
커맨드](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](command/swift/example.html "스위프트로 작성된 커맨드"){.prog-lang-link}
[![타입스크립트로 작성된
커맨드](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](command/typescript/example.html "타입스크립트로 작성된 커맨드"){.prog-lang-link}
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

[반복자 []{.fa .fa-arrow-right}](iterator.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### 돌아가기

[[]{.fa .fa-arrow-left} 책임 연쇄](chain-of-responsibility.html){.btn
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
