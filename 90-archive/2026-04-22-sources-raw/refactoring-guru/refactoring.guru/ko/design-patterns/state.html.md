::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [디자인
패턴들](../design-patterns.html){.type} / [행동
패턴](behavioral-patterns.html){.type}
:::

# 상태 패턴 {#상태-패턴 .title}

::: aka
다음 이름으로도 불립니다: [State]{style="display:inline-block"}
:::

::: {.section .intent}
##  의도 {#intent}

**상태** 패턴은 객체의 내부 상태가 변경될 때 해당 객체가 그의 행동을
변경할 수 있도록 하는 행동 디자인 패턴입니다. 객체가 행동을 변경할 때
객체가 클래스를 변경한 것처럼 보일 수 있습니다.

<figure class="image">
<img
src="../../images/patterns/content/state/state-ko9d76.png?id=d0ebc2efe76959288257a989faebb978"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/state/state-ko.png?id=d0ebc2efe76959288257a989faebb978 640w,/images/patterns/content/state/state-ko-1.5x.png?id=52b7803ab36c874a18c1e8942648520c 960w,/images/patterns/content/state/state-ko-2x.png?id=29bf5543c2e0818f7e6217f79beeb433 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="상태 디자인 패턴" />
</figure>
:::

::: {.section .problem}
##  문제 {#problem}

상태 패턴은 **유한 상태 기계** []{.pop-i}[유한 상태 기계:
[https://refactoring.guru/ko/fsm](https://ko.wikipedia.org/wiki/%EC%9C%A0%ED%95%9C_%EC%83%81%ED%83%9C_%EA%B8%B0%EA%B3%84)]{.pop-content}
개념과 밀접하게 관련되어 있습니다.

<figure class="image">
<img
src="../../images/patterns/diagrams/state/problem10244.png?id=503968745461a0970d1fbb4362dc186f"
style="aspect-ratio: 1.45;"
srcset="/images/patterns/diagrams/state/problem1.png?id=503968745461a0970d1fbb4362dc186f 320w,/images/patterns/diagrams/state/problem1-1.5x.png?id=47c7ca068eceaa9896d320690e6f3368 480w,/images/patterns/diagrams/state/problem1-2x.png?id=ae03c2233939eace11d44925ddeb912d 640w"
sizes="(max-width: 720px) 100vw, 320px" loading="lazy" width="320"
alt="유한 상태 기계" />
<figcaption><p>유한 상태 기계</p></figcaption>
</figure>

이 패턴의 주요 개념은 모든 주어진 순간에 프로그램이 속해 있을 수 있는
*[상]{.cjk}[태]{.cjk}[들]{.cjk}*의 수는 *[유]{.cjk}[한]{.cjk}*하다는
것입니다. 어떤 고유한 상태 내에서든 프로그램은 다르게 행동하며, 한
상태에서 다른 상태로 즉시 전환될 수 있습니다. 하지만 현재의 상태에 따라
프로그램은 특정 다른 상태로 전환되거나 전환되지 않을 수 있습니다. 이러한
전환 규칙들을 *[천]{.cjk}[이]{.cjk}*(transition)​라고도 하는데, 이러한
규칙들 또한 유한하고 미리 결정되어 있습니다.

이 접근 방식을 객체들에 적용할 수도 있습니다. `Document`​(문서) 클래스가
있다고 상상해보세요. 문서는 `Draft`​(초안), `Moderation`​(검토) 및
`Published`​(출판됨)​의 세 가지 상태 중 하나일 수 있습니다. 문서의
`publish`​(출판하기) 메서드는 각 상태에서 약간씩 다르게 작동합니다.

-   `Draft`에서는 문서를 검토 상태로 이동합니다.
-   `Moderation`에서는 문서를 공개하나, 현재 사용자가 관리자인 경우에만
    공개합니다.
-   `Published`에서는 아무 작업도 수행하지 않습니다.

<figure class="image">
<img
src="../../images/patterns/diagrams/state/problem2-koe1eb.png?id=67229d5a249b90dce5e7a9a5cda23320"
style="aspect-ratio: 1.25;"
srcset="/images/patterns/diagrams/state/problem2-ko.png?id=67229d5a249b90dce5e7a9a5cda23320 550w,/images/patterns/diagrams/state/problem2-ko-1.5x.png?id=9090fabede6682b20718074f29cf9b43 825w,/images/patterns/diagrams/state/problem2-ko-2x.png?id=beda0d55d8df9593435bc63afa1166d5 1100w"
sizes="(max-width: 720px) 100vw, 550px" loading="lazy" width="550"
alt="문서 객체의 가능한 상태들" />
<figcaption><p>문서 객체의 가능한 상태들
및 천이​(transition)​들.</p></figcaption>
</figure>

상태 머신들은 일반적으로 객체의 상태에 따라 적절한 행동들을 선택하는
많은 조건문​(`if` 또는 `switch`)​으로 구현됩니다. 일반적으로 이 \'상태\'는
객체의 필드들의 값들의 집합일 뿐입니다. 당신은 유한 상태 머신에 대해
들어본 적이 없더라도 적어도 한 번은 상태를 구현해봤을 것입니다. 다음
코드 구조가 익숙해 보이나요?

<figure class="code">
<pre class="code" lang="pseudocode"><code>class Document is
    field state: string
    // …
    method publish() is
        switch (state)
            &quot;draft&quot;:
                state = &quot;moderation&quot;
                break
            &quot;moderation&quot;:
                if (currentUser.role == &quot;admin&quot;)
                    state = &quot;published&quot;
                break
            &quot;published&quot;:
                // Do nothing.
                break
    // …</code></pre>
</figure>

조건문들에 기반한 상태 머신의 가장 큰 약점은 `Document` 클래스에
상태들과 상태에 의존하는 행동들을 추가할수록 분명해집니다. 그러면 현재
상태에 따라 메서드의 적절한 행동을 선택하는 거대한 조건문들이 대부분의
메서드들에 포함될 것입니다. 이와 같은 코드는 유지 관리하기가 매우
어렵습니다. 왜냐하면 천이 논리를 변경하려면 모든 메서드들의 상태
조건문들을 변경해야 할 수 있기 때문입니다.

이 문제는 프로젝트가 개발되면서 더 복잡해지는 경향이 있습니다. 설계
단계에서 가능한 모든 상태와 천이를 예측하는 것은 매우 어렵습니다.
따라서, 제한된 조건문들의 집합으로 구축되어 간단명료했던 상태 머신이
시간이 지남에 따라 부풀려져 엉망이 될 수 있습니다.
:::

::: {.section .solution}
##  해결책 {#solution}

상태 패턴은 객체의 모든 가능한 상태들에 대해 새 클래스들을 만들고 모든
상태별 행동들을 이러한 클래스들로 추출할 것을 제안합니다.

*[콘]{.cjk}[텍]{.cjk}[스]{.cjk}[트]{.cjk}*라는 원래 객체는 모든 행동을
자체적으로 구현하는 대신 현재 상태를 나타내는 상태 객체 중 하나에 대한
참조를 저장하고 모든 상태와 관련된 작업을 그 객체에 위임합니다.

<figure class="image">
<img
src="../../images/patterns/diagrams/state/solution-ko5049.png?id=b985d382d2f983defad1a0539623e548"
style="aspect-ratio: 1.53;"
srcset="/images/patterns/diagrams/state/solution-ko.png?id=b985d382d2f983defad1a0539623e548 490w,/images/patterns/diagrams/state/solution-ko-1.5x.png?id=08c693b119f2cc4adc5bd5f4d2e9f4de 735w,/images/patterns/diagrams/state/solution-ko-2x.png?id=3e091dc143bd3327909dee75a184b259 980w"
sizes="(max-width: 720px) 100vw, 490px" loading="lazy" width="490"
alt="문서는 상태 객체에 작업을 위임합니다" />
<figcaption><p>문서는 상태 객체에 작업을 위임합니다.</p></figcaption>
</figure>

콘텍스트를 다른 상태로 전환하려면 활성 상태 객체를 새 상태를 나타내는
다른 객체로 바꾸세요. 이것은 모든 상태 클래스들이 같은 인터페이스를
따르고 콘텍스트 자체가 이 인터페이스를 통해 객체들과 작동할 때만
가능합니다.

이 구조는 [전략](strategy.html) 패턴과 비슷해 보이지만 한 가지 중요한
차이점이 있습니다. 상태 패턴에서의 특정 상태들은 서로를 인식하고 한
상태에서 다른 상태로 천이를 시작할 수 있지만 전략들은 거의 대부분 서로에
대해 알지 못한다는 것입니다.
:::

::: {.section .analogy}
##  실제상황 적용 {#analogy}

스마트폰의 버튼들과 스위치들은 장치의 현재 상태에 따라 다르게
행동합니다.

-   스마트폰이 잠금 해제된 상태에서 버튼들을 누르면 다양한 함수들이
    실행됩니다.
-   스마트폰이 잠긴 상태에서 아무 버튼이나 누르면 항상 잠금 해제 화면이
    나타납니다.
-   스마트폰의 충전량이 적을 때 아무 버튼이나 누르면 충전 화면이
    나타납니다.
:::

::::: {.section .structure-container}
##  구조 {#structure}

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/state/structure-ko56c1.png?id=e187e16690b5ff7aab5df7c31908af8a"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.32;"
srcset="/images/patterns/diagrams/state/structure-ko.png?id=e187e16690b5ff7aab5df7c31908af8a 540w,/images/patterns/diagrams/state/structure-ko-1.5x.png?id=7dfeabd24b59e0bd41dc5b43b2deb658 810w,/images/patterns/diagrams/state/structure-ko-2x.png?id=a2c8905c3d6b638b2d5be06fd636313c 1080w"
sizes="(max-width: 720px) 100vw, 540px" loading="lazy" width="540"
alt="상태 디자인 패턴 구조" /><img
src="../../images/patterns/diagrams/state/structure-ko-indexed1f7a.png?id=31e0cc2656bd7d7fe2df59016c51c6ef"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.26;"
srcset="/images/patterns/diagrams/state/structure-ko-indexed.png?id=31e0cc2656bd7d7fe2df59016c51c6ef 540w,/images/patterns/diagrams/state/structure-ko-indexed-1.5x.png?id=a33a71c77e247fd9a781f6412ac28669 810w,/images/patterns/diagrams/state/structure-ko-indexed-2x.png?id=62ccdfd26146e7729325a383a7a91bb8 1080w"
sizes="(max-width: 720px) 100vw, 540px" loading="lazy" width="540"
alt="상태 디자인 패턴 구조" />
</figure>
:::

1.  **콘텍스트**는 구상 상태 객체 중 하나에 대한 참조를 저장하고 모든
    상태별 작업을 그곳에 위임합니다. 콘텍스트는 상태 인터페이스를 통해
    상태 객체와 통신하며, 새로운 상태 객체를 전달하기 위한
    세터​(setter)​를 노출합니다.

2.  **상태** 인터페이스는 상태별 메서드들을 선언합니다. 이러한
    메서드들은 모든 구상 상태에서 유효해야 합니다. 왜냐하면 당신은 결코
    호출될 일 없는 쓸모없는 메서드들이 일부 상태 내에 존재하는 것은
    원하지 않을 것이기 때문입니다.

3.  **구상 상태들**은 상태별 메서드들에 대한 자체적인 구현을 제공합니다.
    여러 상태에서 유사한 코드의 중복을 피하기 위하여 어떤 공통 행동을
    캡슐화하는 중간 추상 클래스들을 제공할 수 있습니다.

    상태 객체들은 콘텍스트 객체에 대한 역참조를 저장할 수 있습니다. 이
    참조를 통해 상태는 콘텍스트 객체에서 모든 필요한 정보를 가져올 수
    있고 상태 천이를 시작할 수 있습니다.

4.  콘텍스트와 구상 상태들 모두 콘텍스트의 다음 상태를 설정할 수 있으며,
    콘텍스트에 연결된 상태 객체를 교체하여 실제 상태 천이를 수행할 수
    있습니다.
::::
:::::

::: {.section .pseudocode}
##  의사코드 {#pseudocode}

이 예시에서 **상태** 패턴을 사용하면 현재 재생 상태에 따라 미디어
플레이어의 같은 컨트롤들이 다르게 행동합니다.

<figure class="image">
<img
src="../../images/patterns/diagrams/state/example1dc9.png?id=1ffdb109b3ebb85d223b9d1651d2034c"
style="aspect-ratio: 1.64;"
srcset="/images/patterns/diagrams/state/example.png?id=1ffdb109b3ebb85d223b9d1651d2034c 590w,/images/patterns/diagrams/state/example-1.5x.png?id=a9ff56d0a139530fa103d496513dfa06 885w,/images/patterns/diagrams/state/example-2x.png?id=cd81e3ffb8aba5883983a59c111b805f 1180w"
sizes="(max-width: 720px) 100vw, 590px" loading="lazy" width="590"
alt="상태 패턴 구조 예시" />
<figcaption><p>상태 객체들을 사용하여 객체 행동을
변경하는 예시.</p></figcaption>
</figure>

플레이어의 주 객체는 항상 상태 객체에 연결되며, 이 상태 객체는
플레이어를 위해 대부분 작업을 수행합니다. 일부 작업들은 플레이어의 현재
상태 객체를 다른 객체로 대체하여 플레이어가 사용자 상호 작용에 반응하는
방식을 변경합니다.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// AudioPlayer(오디오 플레이어) 클래스는 콘텍스트 역할을 합니다. 이 클래스는 또
// 오디오 플레이어의 현재 상태를 나타내는 상태 클래스 중 하나의 인스턴스에 대한
// 참조를 유지합니다.
class AudioPlayer is
    field state: State
    field UI, volume, playlist, currentSong

    constructor AudioPlayer() is
        this.state = new ReadyState(this)

        // 콘텍스트는 사용자 입력 처리를 상태 객체에 위임합니다. 당연히 결과는
        // 현재 활성화된 상태에 따라 달라집니다. 왜냐하면 각 상태는 입력을
        // 다르게 처리할 수 있기 때문입니다.
        UI = new UserInterface()
        UI.lockButton.onClick(this.clickLock)
        UI.playButton.onClick(this.clickPlay)
        UI.nextButton.onClick(this.clickNext)
        UI.prevButton.onClick(this.clickPrevious)

    // 다른 객체들은 오디오 플레이어의 활성 상태를 전환할 수 있어야 합니다.
    method changeState(state: State) is
        this.state = state

    // 사용자 인터페이스 메서드들은 실행을 활성 상태에 위임합니다.
    method clickLock() is
        state.clickLock()
    method clickPlay() is
        state.clickPlay()
    method clickNext() is
        state.clickNext()
    method clickPrevious() is
        state.clickPrevious()

    // 상태는 콘텍스트에 일부 서비스 메서드들을 호출할 수 있습니다.
    method startPlayback() is
        // …
    method stopPlayback() is
        // …
    method nextSong() is
        // …
    method previousSong() is
        // …
    method fastForward(time) is
        // …
    method rewind(time) is
        // …


// 기초 상태 클래스는 모든 구상 상태들이 구현해야 하는 메서드들을 선언하고 상태와
// 연결된 콘텍스트 객체에 대한 역참조도 제공합니다. 상태는 역참조를 사용하여
// 콘텍스트를 다른 상태로 천이할 수 있습니다.
abstract class State is
    protected field player: AudioPlayer

    // 콘텍스트는 상태 생성자를 통해 자신을 전달합니다. 이는 필요한 경우 상태가
    // 유용한 콘텍스트 데이터를 가져오는 데 도움이 될 수 있습니다.
    constructor State(player) is
        this.player = player

    abstract method clickLock()
    abstract method clickPlay()
    abstract method clickNext()
    abstract method clickPrevious()


// 구상 상태들은 콘텍스트의 상태와 연관된 다양한 행동들을 구현합니다.
class LockedState extends State is

    // 잠긴 플레이어의 잠금을 해제하면 플레이어가 두 가지 상태 중 하나를 택할 수
    // 있습니다.
    method clickLock() is
        if (player.playing)
            player.changeState(new PlayingState(player))
        else
            player.changeState(new ReadyState(player))

    method clickPlay() is
        // 잠금 상태: 아무것도 하지 않는다.

    method clickNext() is
        // 잠금 상태: 아무것도 하지 않는다.

    method clickPrevious() is
        // 잠금 상태: 아무것도 하지 않는다.


// 콘텍스트에서 상태 천이를 실행시킬 수도 있습니다.
class ReadyState extends State is
    method clickLock() is
        player.changeState(new LockedState(player))

    method clickPlay() is
        player.startPlayback()
        player.changeState(new PlayingState(player))

    method clickNext() is
        player.nextSong()

    method clickPrevious() is
        player.previousSong()


class PlayingState extends State is
    method clickLock() is
        player.changeState(new LockedState(player))

    method clickPlay() is
        player.stopPlayback()
        player.changeState(new ReadyState(player))

    method clickNext() is
        if (event.doubleclick)
            player.nextSong()
        else
            player.fastForward(5)

    method clickPrevious() is
        if (event.doubleclick)
            player.previous()
        else
            player.rewind(5)</code></pre>
</figure>
:::

:::::::::: {.section .applicability-container}
##  적용 {#applicability}

::::::::: applicability
::: applicability-problem
상태 패턴은 현재 상태에 따라 다르게 행동하는 객체가 있을 때, 상태들의
수가 많을 때, 그리고 상태별 코드가 자주 변경될 때 사용하세요.
:::

::: applicability-solution
이 패턴은 모든 상태별 코드를 서로 다른 클래스들의 집합으로 추출하도록
제안합니다. 그렇게 하면 새로운 상태들을 추가하거나 기존 상태들을 서로
독립적으로 변경할 수 있어서 유지 관리 비용을 절감할 수 있습니다.
:::

::: applicability-problem
이 패턴은 당신이 클래스 필드들의 현재 값들에 따라 클래스가 행동하는
방식을 변경하는 거대한 조건문들로 오염된 클래스가 있을 때 사용하세요.
:::

::: applicability-solution
상태 패턴은 당신이 이러한 조건문들의 브랜치들을 해당 상태 클래스들의
메서드들로 추출할 수 있도록 합니다. 그렇게 하는 동안 당신은 당신의 주
클래스의 상태별 코드와 관련된 임시 필드들과 도우미 메서드들을 정리할
수도 있습니다.
:::

::: applicability-problem
상태 패턴은 유사한 상태들에 중복 코드와 조건문-기반 상태 머신의 천이가
많을 때 사용하세요.
:::

::: applicability-solution
상태 패턴은 당신이 상태 클래스들의 계층구조들을 구성할 수 있도록 하며 또
공통 코드를 추상 기초 클래스들에 추출하여 중복을 줄일 수 있도록 합니다.
:::
:::::::::
::::::::::

::: {.section .checklist}
##  구현방법 {#checklist}

1.  어떤 클래스가 콘텍스트로 작동할지 결정하세요. 이는 상태에 의존하는
    코드가 이미 있는 기존 클래스일 수 있고, 상태별 코드가 여러 클래스에
    분산된 경우 새로운 클래스일 수도 있습니다.

2.  상태 인터페이스를 선언하세요. 콘텍스트에 선언된 모든 메서드들을
    미러링할 수 있어도 상태별 동작을 포함할 수 있는 메서드들만 목표로
    설정하세요.

3.  모든 실제 상태에 대해 상태 인터페이스에서 파생된 클래스를 만드세요.
    그런 다음 콘텍스트의 메서드들을 살펴보고 당신의 새로 생성된 클래스에
    상태와 관련된 모든 코드를 추출하세요.

    당신이 코드를 상태 클래스로 옮기는 동안 코드가 콘텍스트의 비공개
    멤버들​(필드와 메서드)​에 의존한다는 사실을 발견할 수 있습니다. 그럴
    때 몇 가지 해결 방법이 있습니다.

    -   이 필드들 또는 메서드들을 공개된​(public) 상태로 전환하세요.
    -   당신이 추출하는 행동을 콘텍스트의 공개된 메서드로 전환하고 상태
        클래스에서 호출하세요. 이 방법은 보기 흉하지만 빠르며 나중에
        언제든지 고칠 수 있습니다.
    -   사용 중인 프로그래밍 언어가 중첩 클래스들을 지원하는 경우에 한해
        상태 클래스들을 콘텍스트 클래스에 중첩하세요.

4.  콘텍스트 클래스에서 상태 인터페이스 유형의 참조 필드와 필드의 값을
    오버라이드할 수 있는 공개된 세터​(setter)​를 추가하세요.

5.  콘텍스트의 메서드를 다시 살펴보고 빈 상태 조건문들을 상태 객체의
    해당하는 메서드들에 대한 호출들로 바꾸세요.

6.  콘텍스트의 상태를 전환하려면 상태 클래스 중 하나의 인스턴스를 만든
    후 콘텍스트에 전달하세요. 당신은 이 작업을 콘텍스트 자체에서, 다양한
    상태들에서, 또는 클라이언트에서 수행할 수 있습니다. 이 작업이
    수행되는 곳마다 클래스는 클래스가 인스턴스화하는 구상 상태 클래스에
    의존하게 됩니다.
:::

:::::: {.section .pros-cons}
##  장단점 {#pros-cons}

::::: row
::: col-sm-6
-    *[단]{.cjk}[일]{.cjk} [책]{.cjk}[임]{.cjk} [원]{.cjk}[칙]{.cjk}*.
    특정 상태들과 관련된 코드를 별도의 클래스들로 구성하세요.
-    *[개]{.cjk}[방]{.cjk}/[폐]{.cjk}[쇄]{.cjk} [원]{.cjk}[칙]{.cjk}*.
    기존 상태 클래스들 또는 콘텍스트를 변경하지 않고 새로운 상태들을
    도입하세요.
-    거대한 상태 머신 조건문들을 제거하여 콘텍스트의 코드를
    단순화하세요.
:::

::: col-sm-6
-    상태 머신에 몇 가지 상태만 있거나 머신이 거의 변경되지 않을 때 상태
    패턴을 적용하는 것은 과도할 수 있습니다.
:::
:::::
::::::

::: {.section .relations}
##  다른 패턴과의 관계 {#relations}

-   [브리지](bridge.html), [상태](state.html), [전략](strategy.html)
    패턴은 매우 유사한 구조로 되어 있으며, [어댑터](adapter.html) 패턴도
    이들과 어느 정도 유사한 구조로 되어 있습니다. 위 모든 패턴은 다른
    객체에 작업을 위임하는 합성을 기반으로 합니다. 하지만 이 패턴들은
    모두 다른 문제들을 해결합니다. 패턴은 특정 방식으로 코드의 구조를
    짜는 레시피에 불과하지 않습니다. 왜냐하면 패턴은 해결하는 문제를
    다른 개발자들에게 전달할 수도 있기 때문입니다.

-   [상태](state.html)는 [전략](strategy.html)의 확장으로 간주할 수
    있습니다. 두 패턴 모두 합성을 기반으로 합니다. 그들은 어떤 작업을
    도우미 객체들에 전달하여 콘텍스트의 행동을 바꿉니다.
    *[전]{.cjk}[략]{.cjk} [패]{.cjk}[턴]{.cjk}*은 이러한 객체들을 완전히
    독립적으로 만들어 서로를 인식하지 못하도록 만듭니다. 그러나
    *[상]{.cjk}[태]{.cjk}*는 구상 상태들 사이의 의존 관계들을 제한하지
    않으므로 그들이 콘텍스트의 상태를 마음대로 변경할 수 있도록 합니다.
:::

::: {.section .implementations}
##  코드 예시 {#implementations}

[![C#으로 작성된
상태](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](state/csharp/example.html "C#으로 작성된 상태"){.prog-lang-link}
[![C++로 작성된
상태](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](state/cpp/example.html "C++로 작성된 상태"){.prog-lang-link}
[![Go로 작성된
상태](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](state/go/example.html "Go로 작성된 상태"){.prog-lang-link}
[![자바로 작성된
상태](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](state/java/example.html "자바로 작성된 상태"){.prog-lang-link}
[![PHP로 작성된
상태](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](state/php/example.html "PHP로 작성된 상태"){.prog-lang-link}
[![파이썬으로 작성된
상태](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](state/python/example.html "파이썬으로 작성된 상태"){.prog-lang-link}
[![루비로 작성된
상태](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](state/ruby/example.html "루비로 작성된 상태"){.prog-lang-link}
[![러스트로 작성된
상태](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](state/rust/example.html "러스트로 작성된 상태"){.prog-lang-link}
[![스위프트로 작성된
상태](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](state/swift/example.html "스위프트로 작성된 상태"){.prog-lang-link}
[![타입스크립트로 작성된
상태](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](state/typescript/example.html "타입스크립트로 작성된 상태"){.prog-lang-link}
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

[전략 []{.fa .fa-arrow-right}](strategy.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### 돌아가기

[[]{.fa .fa-arrow-left} 옵서버](observer.html){.btn .btn-default
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
