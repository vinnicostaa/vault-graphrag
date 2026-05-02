::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [디자인
패턴들](../design-patterns.html){.type} / [행동
패턴](behavioral-patterns.html){.type}
:::

# 템플릿 메서드 패턴 {#템플릿-메서드-패턴 .title}

::: aka
다음 이름으로도 불립니다: [Template
Method]{style="display:inline-block"}
:::

::: {.section .intent}
##  의도 {#intent}

**템플릿 메서드**는 부모 클래스에서 알고리즘의 골격을 정의하지만, 해당
알고리즘의 구조를 변경하지 않고 자식 클래스들이 알고리즘의 특정 단계들을
오버라이드​(재정의)​할 수 있도록 하는 행동 디자인 패턴입니다.

<figure class="image">
<img
src="../../images/patterns/content/template-method/template-method4c1b.png?id=eee9461742f832814f19612ccf472819"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/template-method/template-method.png?id=eee9461742f832814f19612ccf472819 640w,/images/patterns/content/template-method/template-method-1.5x.png?id=2b4c179aaa02f5c45a59324b904cd130 960w,/images/patterns/content/template-method/template-method-2x.png?id=4e164dc41be4dcfa122628864c2be210 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="템플릿 메서드 디자인 패턴" />
</figure>
:::

::: {.section .problem}
##  문제 {#problem}

회사 문서들을 분석하는 데이터 마이닝 앱을 만들고 있다고 가정해 봅시다.
사용자들은 앱에 다양한 형식​(PDF, DOC, CSV)​의 문서들을 제공하고 앱은
이러한 문서들에서 일관된 형식으로 의미 있는 데이터를 추출하려고
시도합니다.

앱의 첫 번째 버전은 DOC 파일과만 작동할 수 있었고, 다음 버전에서는 CSV
파일을 지원할 수 있었습니다. 한 달 후, 당신은 앱이 PDF 파일에서 데이터를
추출하도록 \'가르쳤습니다\'.

<figure class="image">
<img
src="../../images/patterns/diagrams/template-method/problem6793.png?id=60fa4f735c467ac1c9438231a1782807"
style="aspect-ratio: 1.35;"
srcset="/images/patterns/diagrams/template-method/problem.png?id=60fa4f735c467ac1c9438231a1782807 620w,/images/patterns/diagrams/template-method/problem-1.5x.png?id=83fa009f7727bdcc69335b946919f0d8 930w,/images/patterns/diagrams/template-method/problem-2x.png?id=fc8b434afec7b6135043d0d2f48189f0 1240w"
sizes="(max-width: 720px) 100vw, 620px" loading="lazy" width="620"
alt="데이터 마이닝 클래스들에는 많은 중복 코드가 포함되어 있었습니다" />
<figcaption><p>데이터 마이닝 클래스들에는 중복 코드가 많이
포함되어 있습니다.</p></figcaption>
</figure>

어느 날 당신은 세 클래스 모두에 유사한 코드가 많다는 것을 알게
되었습니다. 다양한 데이터 형식들을 처리하는 코드는 클래스마다 완전히
다르지만 데이터 처리 및 분석을 위한 코드는 거의 같습니다. 알고리즘
구조는 그대로 두되, 코드 중복은 제거하는 게 좋지 않을까요?

이 클래스들을 사용하는 클라이언트 코드와 관련된 또 다른 문제가
있었습니다. 이 코드에는 작업을 처리하고 있는 클래스에 따라 적절한
행동들을 선택하는 조건문이 많이 있었습니다. 세 처리 클래스에 전부 공통
인터페이스나 공통 기초 클래스가 있었다면, 클라이언트 코드에서 조건문들을
제거하고 처리 객체에 메서드를 호출할 때 다형성을 사용할 수 있었을
겁니다.
:::

::: {.section .solution}
##  해결책 {#solution}

템플릿 메서드 패턴은 알고리즘을 일련의 단계들로 나누고, 이러한 단계들을
메서드들로 변환한 뒤, 단일 *[템]{.cjk}[플]{.cjk}[릿]{.cjk}
[메]{.cjk}[서]{.cjk}[드]{.cjk}* 내부에 이러한 메서드들에 대한 일련의
호출들을 넣으라고 제안합니다. 이러한 단계들은 `abstract`​(추상)​이거나
일부 디폴트​(기본값) 구현을 가질 것입니다. 알고리즘을 사용하기 위해
클라이언트는 자신의 자식 클래스를 제공해야 하고, 모든 추상 단계를
구현해야 하며, 필요하다면 (템플릿 메서드를 제외한) 선택적 단계 중 일부를
오버라이드​(재정의)​해야 합니다.

이것이 당신의 데이터 마이닝 앱에서 어떻게 작동하는지 봅시다. 세 가지
파싱 알고리즘들 모두를 위한 기초 클래스를 만들 수 있습니다. 이 기초
클래스는 다양한 문서 처리 단계들에 대한 일련의 호출들로 구성된 템플릿
메서드를 정의합니다.

<figure class="image">
<img
src="../../images/patterns/diagrams/template-method/solution-ko965e.png?id=d79b498950b810b2e00ec8dfcd8ea424"
style="aspect-ratio: 1.43;"
srcset="/images/patterns/diagrams/template-method/solution-ko.png?id=d79b498950b810b2e00ec8dfcd8ea424 600w,/images/patterns/diagrams/template-method/solution-ko-1.5x.png?id=06242fea38b36354fdc487bd71e37bc0 900w,/images/patterns/diagrams/template-method/solution-ko-2x.png?id=100524477c04daae219a201c35478ea2 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="템플릿 메서드는 알고리즘의 골격을 정의합니다" />
<figcaption><p>템플릿 메서드는 알고리즘을 단계로 나누어 자식 클래스들이
위에 언급된 실제 메서드를 제외한 이 단계들을 오버라이드할 수
있도록 합니다.</p></figcaption>
</figure>

처음에는 모든 단계를 `abstract`로 선언하여 자식 클래스들이 이러한
메서드들에 대한 자체 구현을 제공하도록 강제할 수 있습니다. 당신의 앱의
경우, 자식 클래스들은 이미 필요한 모든 구현들을 가지고 있으므로, 우리가
해야 할 유일한 일은 메서드들의 시그니처들을 부모 클래스의 메서드들과
일치하도록 조정하는 것입니다.

이제 중복 코드를 제거하기 위해 무엇을 할 수 있는지 봅시다. 파일
열기/닫기 및 데이터 추출/파싱을 위한 코드는 데이터 형식들에 따라
다르므로 해당 메서드들을 건드릴 의미가 없습니다. 그러나 미가공 데이터
분석 및 보고서 작성과 같은 다른 단계들의 구현은 매우 유사하므로 기초
클래스로 끌어올릴 수 있습니다. 그러면 자식 클래스들은 기초 클래스에서 이
코드를 공유할 수 있습니다.

보시다시피 두 가지 유형의 단계들이 있습니다:

-   모든 자식 클래스는 *[추]{.cjk}[상]{.cjk}
    [단]{.cjk}[계]{.cjk}[들]{.cjk}*을 구현해야 합니다.
-   *[선]{.cjk}[택]{.cjk}[적]{.cjk} [단]{.cjk}[계]{.cjk}[들]{.cjk}*에는
    이미 어떤 디폴트​(기본값) 구현이 있지만, 필요한 경우 이를 무시하고
    오버라이드​(재정의) 할 수 있습니다.

*[훅]{.cjk}*이라는 또 다른 유형의 단계가 있습니다. 훅은 몸체가 비어 있는
선택적 단계입니다. 템플릿 메서드는 훅이 오버라이드 되지 않아도
작동합니다. 일반적으로 훅들은 알고리즘의 중요한 단계들의 전 또는 후에
배치되어 자식 클래스들에 알고리즘에 대한 추가 확장 지점들을 제공합니다.
:::

::: {.section .analogy}
##  실제상황 적용 {#analogy}

<figure class="image">
<img
src="../../images/patterns/diagrams/template-method/live-exampleea7a.png?id=2485d52852f87da06c9cc0e2fd257d6a"
style="aspect-ratio: 1.74;"
srcset="/images/patterns/diagrams/template-method/live-example.png?id=2485d52852f87da06c9cc0e2fd257d6a 590w,/images/patterns/diagrams/template-method/live-example-1.5x.png?id=92c5fd9345329da09e3233302158678c 885w,/images/patterns/diagrams/template-method/live-example-2x.png?id=89083a3dcd1fe2b627b9b6e6ff4986dc 1180w"
sizes="(max-width: 720px) 100vw, 590px" loading="lazy" width="590"
alt="대량 주택 건설" />
<figcaption><p>일반적인 건축 계획은 클라이언트의 니즈에 더 잘 부합하도록
약간 변경될 수 있습니다.</p></figcaption>
</figure>

템플릿 메서드 접근 방식은 대량 주택 건설에 사용할 수 있습니다. 표준 주택
건설을 위한 건축 계획에는 잠재적 주택 소유자가 결과 주택의 일부 세부
사항들을 조정할 수 있도록 하는 여러 확장 지점들이 포함될 수 있습니다.

완성된 집이 다른 집들과 조금씩 다르도록 각 건축 단계​(예: 기초 쌓기,
골조공사, 벽 쌓기, 수도 및 전기 배선 설치 등)​는 약간씩 변경될 수
있습니다.
:::

::::: {.section .structure-container}
##  구조 {#structure}

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/template-method/structure1c3b.png?id=924692f994bff6578d8408d90f6fc459"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 0.89;"
srcset="/images/patterns/diagrams/template-method/structure.png?id=924692f994bff6578d8408d90f6fc459 340w,/images/patterns/diagrams/template-method/structure-1.5x.png?id=613d78ad8ec08536ec70f4e0976b5a1a 510w,/images/patterns/diagrams/template-method/structure-2x.png?id=25082d6d6a76f51c6b64d8aeeaffdbb5 680w"
sizes="(max-width: 720px) 100vw, 340px" loading="lazy" width="340"
alt="템플릿 메서드 디자인 패턴 구조" /><img
src="../../images/patterns/diagrams/template-method/structure-indexedb94b.png?id=4ced6107519bc66710d2f05c0f4097a1"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 0.92;"
srcset="/images/patterns/diagrams/template-method/structure-indexed.png?id=4ced6107519bc66710d2f05c0f4097a1 350w,/images/patterns/diagrams/template-method/structure-indexed-1.5x.png?id=58e3a9092786c824eb738a7771702093 525w,/images/patterns/diagrams/template-method/structure-indexed-2x.png?id=86f28789cdcc5a4c415d6a1100de56fc 700w"
sizes="(max-width: 720px) 100vw, 350px" loading="lazy" width="350"
alt="템플릿 메서드 디자인 패턴 구조" />
</figure>
:::

1.  **추상 클래스**는 알고리즘의 단계들의 역할을 하는 메서드들을
    선언하며, 이러한 메서드를 특정 순서로 호출하는 실제 템플릿 메서드도
    선언합니다. 단계들은 `abstract`로 선언되거나 일부 디폴트 구현을
    갖습니다.

2.  **구상 클래스들**은 모든 단계들을 오버라이드할 수 있지만 템플릿
    메서드 자체는 오버라이드 할 수 없습니다.
::::
:::::

::: {.section .pseudocode}
##  의사코드 {#pseudocode}

이 예시에서 **템플릿 메서드** 패턴은 간단한 전략 비디오 게임의 인공
지능의 다양한 브랜치들에 대한 \'골격\'을 제공합니다.

<figure class="image">
<img
src="../../images/patterns/diagrams/template-method/example32c6.png?id=c0ce5cc8070925a1cd345fac6afa16b6"
style="aspect-ratio: 1;"
srcset="/images/patterns/diagrams/template-method/example.png?id=c0ce5cc8070925a1cd345fac6afa16b6 430w,/images/patterns/diagrams/template-method/example-1.5x.png?id=d85c6b81c900f46d55688406170600bc 645w,/images/patterns/diagrams/template-method/example-2x.png?id=d8b309659c4b722237ec97733da935bd 860w"
sizes="(max-width: 720px) 100vw, 430px" loading="lazy" width="430"
alt="템플릿 메서드 패턴 구조 예시" />
<figcaption><p>간단한 비디오 게임의 인공지능 클래스들.</p></figcaption>
</figure>

게임의 모든 종족은 거의 같은 유형의 유닛들과 건물들을 가지고 있습니다.
따라서 다양한 종족에 대해 같은 인공지능 구조를 재사용하면서 일부 세부
사항들은 오버라이드할 수 있습니다. 이 접근 방식을 사용하면 오크들의
인공지능을 오버라이드하여 그들을 더 공격적으로 만들고, 같은 방식으로
인간들을 방어 지향적으로 만들고 몬스터들은 아무것도 건설할 수 없도록
만들 수 있습니다. 게임에 새 종족을 추가하려면 새 인공지능 자식 클래스를
만들고 기초 인공지능 클래스에 선언된 디폴트 메서드들을 오버라이드해야
합니다.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// 추상 클래스는 템플릿 메서드를 정의합니다. 이 메서드는 일반적으로 원시 작업을
// 추상화하기 위해 호출로 구성된 어떤 알고리즘의 골격을 포함합니다. 구상 자식
// 클래스들은 이러한 작업을 구현하지만 템플릿 메서드 자체는 그대로 둡니다.
class GameAI is
    // 템플릿 메서드는 알고리즘의 골격을 정의합니다.
    method turn() is
        collectResources()
        buildStructures()
        buildUnits()
        attack()

    // 일부 단계들은 기초 클래스에서 바로 구현될 수 있습니다.
    method collectResources() is
        foreach (s in this.builtStructures) do
            s.collect()

    // 그리고 그중 일부는 추상으로 정의될 수 있습니다.
    abstract method buildStructures()
    abstract method buildUnits()

    // 한 클래스에는 여러 템플릿 메서드가 있을 수 있습니다.
    method attack() is
        enemy = closestEnemy()
        if (enemy == null)
            sendScouts(map.center)
        else
            sendWarriors(enemy.position)

    abstract method sendScouts(position)
    abstract method sendWarriors(position)

// 구상 클래스들은 기초 클래스의 모든 추상 작업을 구현해야 합니다. 하지만 템플릿
// 메서드 자체를 오버라이드해서는 안 됩니다.
class OrcsAI extends GameAI is
    method buildStructures() is
        if (there are some resources) then
            // 농장들, 막사들, 그리고 요새들을 차례로 건설하세요.

    method buildUnits() is
        if (there are plenty of resources) then
            if (there are no scouts)
                // 잡역인을 생성한 후 정찰병 그룹에 추가하세요.
            else
                // 하급 병사를 생성한 후 전사 그룹에 추가하세요.

    // …

    method sendScouts(position) is
        if (scouts.length &gt; 0) then
            // 정찰병들을 위치로 보내세요.

    method sendWarriors(position) is
        if (warriors.length &gt; 5) then
            // 전사들을 위치로 보내세요.

// 자식 클래스들은 디폴트 구현을 가진 일부 작업을 오버라이드할 수 있습니다.
class MonstersAI extends GameAI is
    method collectResources() is
        // 몬스터들은 자원을 모으지 않습니다.

    method buildStructures() is
        // 몬스터들은 건물을 짓지 않습니다.

    method buildUnits() is
        // 몬스터들은 유닛들을 생성하지 않습니다.</code></pre>
</figure>
:::

:::::::: {.section .applicability-container}
##  적용 {#applicability}

::::::: applicability
::: applicability-problem
템플릿 메서드 패턴은 클라이언트들이 알고리즘의 특정 단계들만 확장할 수
있도록 하고 싶을 때, 그러나 전체 알고리즘이나 알고리즘 구조는 확장하지
못하도록 하려고 할 때 사용하세요.
:::

::: applicability-solution
템플릿 메서드는 모놀리식 알고리즘을 일련의 개별 단계들로 전환할 수
있도록 합니다. 이 알고리즘은 부모 클래스에서 정의된 구조를 그대로
유지하면서 자식 클래스들에 의해 쉽게 확장될 수 있습니다.
:::

::: applicability-problem
이 패턴은 약간의 차이가 있지만 거의 같은 알고리즘들을 포함하는 여러
클래스가 있는 경우에 사용하세요. 결과적으로 알고리즘이 변경되면 모든
클래스를 수정해야 할 수도 있습니다.
:::

::: applicability-solution
이러한 알고리즘을 템플릿 메서드로 전환하면 유사한 구현들이 있는 단계들을
부모 클래스로 끌어올릴 수 있으며, 그로 인해 코드 중복을 제거할 수
있습니다. 자식 클래스 중 서로 코드가 다른 부분들은 자식 클래스들에
남겨놓을 수 있습니다.
:::
:::::::
::::::::

::: {.section .checklist}
##  구현방법 {#checklist}

1.  대상 알고리즘을 분석하여 여러 단계로 나눌 수 있는지 확인하세요. 어떤
    단계들이 모든 자식 클래스에 공통인지 또 어떤 단계들이 항상
    고유한지를 고려하세요.

2.  추상 기초 클래스를 만들고 알고리즘의 단계들을 표현하는 템플릿
    메서드와 추상 메서드들의 집합을 선언하세요. 해당하는 단계들을
    실행하여 템플릿 메서드에서 알고리즘의 구조의 윤곽을 잡으세요. 템플릿
    메서드를 `final`로 만들어 자식 클래스들이 메서드를 오버라이드하지
    못하도록 하는 것을 고려하세요.

3.  모든 단계가 추상적이어도 괜찮습니다. 그러나 일부 단계들에는 디폴트
    구현이 있는 것이 도움이 될 수 있습니다. 자식 클래스들은 이러한
    디폴트 메서드들을 구현할 필요가 없습니다.

4.  알고리즘의 중요한 단계들 사이에 훅들을 추가하는 것을 고려하세요.

5.  알고리즘의 각 변형에서 새로운 구상 자식 클래스를 생성하세요. 새로운
    구상 자식 클래스는 모든 추상 단계들을
    *[반]{.cjk}[드]{.cjk}[시]{.cjk}* 구현해야 하지만 일부 선택 단계를
    *[오]{.cjk}[버]{.cjk}[라]{.cjk}[이]{.cjk}[드]{.cjk}[할]{.cjk}
    [수]{.cjk}[도]{.cjk}* 있습니다.
:::

:::::: {.section .pros-cons}
##  장단점 {#pros-cons}

::::: row
::: col-sm-6
-    클라이언트들이 대규모 알고리즘의 특정 부분만 오버라이드하도록 하여
    그들이 알고리즘의 다른 부분에 발생하는 변경에 영향을 덜 받도록 할 수
    있습니다.
-    중복 코드를 부모 클래스로 가져올 수 있습니다.
:::

::: col-sm-6
-    일부 클라이언트들은 알고리즘의 제공된 골격에 의해 제한될 수
    있습니다.
-    당신은 자식 클래스를 통해 디폴트 단계 구현을 억제하여
    *[리]{.cjk}[스]{.cjk}[코]{.cjk}[프]{.cjk} [치]{.cjk}[환]{.cjk}
    [원]{.cjk}[칙]{.cjk}*을 위반할 수 있습니다.
-    템플릿 메서드들은 단계들이 더 많을수록 유지가 더 어려운 경향이
    있습니다.
:::
:::::
::::::

::: {.section .relations}
##  다른 패턴과의 관계 {#relations}

-   [팩토리 메서드](factory-method.html)는 [템플릿
    메서드](template-method.html)의 특수화라고 생각할 수 있습니다.
    동시에 대규모 *[템]{.cjk}[플]{.cjk}[릿]{.cjk}
    [메]{.cjk}[서]{.cjk}[드]{.cjk}*의 한 단계의 역할을
    *[팩]{.cjk}[토]{.cjk}[리]{.cjk} [메]{.cjk}[서]{.cjk}[드]{.cjk}*가 할
    수 있습니다.

-   [템플릿 메서드](template-method.html)는 상속을 기반으로 합니다. 이
    메서드는 자식 클래스들에서 알고리즘의 부분들을 확장하여 변경할 수
    있도록 합니다. [전략](strategy.html) 패턴은 합성을 기반으로 합니다:
    당신은 객체 행동의 일부분들을 이러한 행동에 해당하는 다양한 전략들을
    제공하여 변경할 수 있습니다. *[템]{.cjk}[플]{.cjk}[릿]{.cjk}
    [메]{.cjk}[서]{.cjk}[드]{.cjk}*는 클래스 수준에서 작동하므로
    정적입니다. *[전]{.cjk}[략]{.cjk} [패]{.cjk}[턴]{.cjk}[은]{.cjk}*은
    객체 수준에서 작동하므로 런타임에 행동들을 전환할 수 있도록 합니다.
:::

::: {.section .implementations}
##  코드 예시 {#implementations}

[![C#으로 작성된 템플릿
메서드](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](template-method/csharp/example.html "C#으로 작성된 템플릿 메서드"){.prog-lang-link}
[![C++로 작성된 템플릿
메서드](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](template-method/cpp/example.html "C++로 작성된 템플릿 메서드"){.prog-lang-link}
[![Go로 작성된 템플릿
메서드](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](template-method/go/example.html "Go로 작성된 템플릿 메서드"){.prog-lang-link}
[![자바로 작성된 템플릿
메서드](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](template-method/java/example.html "자바로 작성된 템플릿 메서드"){.prog-lang-link}
[![PHP로 작성된 템플릿
메서드](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](template-method/php/example.html "PHP로 작성된 템플릿 메서드"){.prog-lang-link}
[![파이썬으로 작성된 템플릿
메서드](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](template-method/python/example.html "파이썬으로 작성된 템플릿 메서드"){.prog-lang-link}
[![루비로 작성된 템플릿
메서드](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](template-method/ruby/example.html "루비로 작성된 템플릿 메서드"){.prog-lang-link}
[![러스트로 작성된 템플릿
메서드](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](template-method/rust/example.html "러스트로 작성된 템플릿 메서드"){.prog-lang-link}
[![스위프트로 작성된 템플릿
메서드](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](template-method/swift/example.html "스위프트로 작성된 템플릿 메서드"){.prog-lang-link}
[![타입스크립트로 작성된 템플릿
메서드](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](template-method/typescript/example.html "타입스크립트로 작성된 템플릿 메서드"){.prog-lang-link}
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

[비지터 []{.fa .fa-arrow-right}](visitor.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### 돌아가기

[[]{.fa .fa-arrow-left} 전략](strategy.html){.btn .btn-default
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
