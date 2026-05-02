:::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [디자인
패턴들](../design-patterns.html){.type} / [구조
패턴](structural-patterns.html){.type}
:::

# 플라이웨이트 패턴 {#플라이웨이트-패턴 .title}

::: aka
다음 이름으로도 불립니다:
[캐시, ]{style="display:inline-block"}[Flyweight]{style="display:inline-block"}
:::

::: {.section .intent}
##  의도 {#intent}

**플라이웨이트**는 각 객체에 모든 데이터를 유지하는 대신 여러 객체들
간에 상태의 공통 부분들을 공유하여 사용할 수 있는 RAM에 더 많은 객체들을
포함할 수 있도록 하는 구조 디자인 패턴입니다.

<figure class="image">
<img
src="../../images/patterns/content/flyweight/flyweight7be2.png?id=e34fbacb847dd609b5e68aaf252c4db0"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/flyweight/flyweight.png?id=e34fbacb847dd609b5e68aaf252c4db0 640w,/images/patterns/content/flyweight/flyweight-1.5x.png?id=b32df2297473b8a7577e1900e57325ac 960w,/images/patterns/content/flyweight/flyweight-2x.png?id=6a8f17d9550c75c3d648a605c4d31b45 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="플라이웨이트 디자인 패턴" />
</figure>
:::

::: {.section .problem}
##  문제 {#problem}

당신은 재미 삼아 플레이어들이 지도를 돌아다니며 서로에게 총을 쏘는
간단한 비디오 게임을 만들기로 했습니다. 당신은 폭발들로 인한 방대한 양의
총알들, 미사일들 및 파편들이 지도 전체를 날아다니는 전율 넘치는 경험을
플레이어들에게 선사하기로 했으며, 이를 선사하기 위해 사실적인 입자
시스템을 구현하기로 했습니다.

당신은 게임을 완성한 후 친구에게 게임을 보냈습니다. 당신의 컴퓨터에서는
게임이 완벽하게 실행되었지만, 당신의 친구는 오랫동안 게임을 즐길 수
없었습니다. 왜냐하면 친구의 컴퓨터에서는 시작 후 고작 몇 분 후에 게임이
계속 충돌했기 때문입니다. 당신이 디버그 로그를 자세히 살펴본 결과,
친구의 컴퓨터의 RAM이 당신의 컴퓨터처럼 충분하지 않아 게임이 충돌했음이
분명해졌습니다.

문제의 원인은 당신의 입자 시스템과 관련이 있었습니다. 각 총알, 미사일
또는 파편과 같은 입자는 많은 데이터를 포함하는 별도의 객체로
표시되었습니다. 플레이어 화면의 대학살이 절정에 이르렀을 때 새로 생성된
입자들을 더 이상 나머지 RAM이 감당하지 못해서 프로그램이 충돌했습니다.

<figure class="image">
<img
src="../../images/patterns/diagrams/flyweight/problem-koc752.png?id=572f292324b5b3acee0d24f9b4e86814"
style="aspect-ratio: 2.54;"
srcset="/images/patterns/diagrams/flyweight/problem-ko.png?id=572f292324b5b3acee0d24f9b4e86814 660w,/images/patterns/diagrams/flyweight/problem-ko-1.5x.png?id=8744b4dfef5533574636e8a1285d6b36 990w,/images/patterns/diagrams/flyweight/problem-ko-2x.png?id=2616561b8edd5992481605b9ce523dfa 1320w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="660"
alt="플라이웨이트 패턴 문제" />
</figure>
:::

::: {.section .solution}
##  해결책 {#solution}

`Particle`​(입자) 클래스를 자세히 살펴보면 `color`​(색상) 및
`sprite`(스프라이트) 필드들이 다른 필드들보다 훨씬 더 많은 메모리를
사용한다는 것을 알 수 있습니다. 더 나쁜 것은 이 두 필드가 모든 입자에
걸쳐 거의 같은 데이터를 저장한다는 것입니다. 예를 들어, 모든 총알은 같은
색상과 스프라이트를 갖습니다.

<figure class="image">
<img
src="../../images/patterns/diagrams/flyweight/solution1-ko5598.png?id=8736baf093d68831d33835efd83f59f1"
style="aspect-ratio: 2.06;"
srcset="/images/patterns/diagrams/flyweight/solution1-ko.png?id=8736baf093d68831d33835efd83f59f1 640w,/images/patterns/diagrams/flyweight/solution1-ko-1.5x.png?id=2bfbab4c0dc7f9b358cd4aa48f9aab89 960w,/images/patterns/diagrams/flyweight/solution1-ko-2x.png?id=30f5e51bae11e0f064bbb905c43a8715 1280w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="640"
alt="플라이웨이트 패턴 해결책" />
</figure>

좌표, 이동 벡터 및 속도와 같은 입자 상태의 다른 부분들은 각 입자마다
고유하며, 이러한 필드들의 값은 시간이 지남에 따라 변경됩니다. 이
데이터는 입자의 계속 변화하는 콘텍스트를 나타내나, 반면 색상과
스프라이트는 각 입자마다 일정하게 유지됩니다.

객체의 이러한 상수 데이터를 일반적으로 *[고]{.cjk}[유]{.cjk}[한]{.cjk}
[상]{.cjk}[태]{.cjk}*라고 합니다. 이 데이터는 객체 안에서 삽니다. 다른
객체들은 이 데이터를 읽을 수만 있고 변경할 수는 없습니다. 종종 다른
객체들에 의해 \'외부에서\' 변경되는 객체의 나머지 상태를
*[공]{.cjk}[유]{.cjk}[한]{.cjk} [상]{.cjk}[태]{.cjk}*라고 합니다.

플라이웨이트 패턴은 객체 내부에 공유한 상태의 저장을 중단하고, 대신 이
상태를 이 상태에 의존하는 특정 메서드들에 전달할 것을 제안합니다. 고유한
상태만 객체 내에 유지되므로 해당 고유한 상태는 콘텍스트가 다른 곳에서
재사용할 수 있습니다. 이러한 객체들은 공유한 상태보다 변형이 훨씬 적은
고유한 상태에서만 달라지므로 훨씬 더 적은 수의 객체만 있으면 됩니다.

<figure class="image">
<img
src="../../images/patterns/diagrams/flyweight/solution3-ko4e14.png?id=518a4d27e38553f19e02adf737daec12"
style="aspect-ratio: 0.76;"
srcset="/images/patterns/diagrams/flyweight/solution3-ko.png?id=518a4d27e38553f19e02adf737daec12 424w,/images/patterns/diagrams/flyweight/solution3-ko-1.5x.png?id=2cf84ff9c547493dd5d5fc90b1dbd2a8 636w,/images/patterns/diagrams/flyweight/solution3-ko-2x.png?id=5003b23e27ce9b6b73514b0e7c69e8c9 848w"
sizes="(max-width: 720px) 100vw, 424px" loading="lazy" width="424"
alt="플라이웨이트 패턴 해결책" />
</figure>

이제 당신의 게임을 다시 살펴봅시다. 입자 클래스에서 공유한 상태를
추출했다고 가정하면 총알, 미사일, 파편의 세 가지 다른 객체만으로도
게임의 모든 입자를 충분히 나타낼 수 있습니다. 지금쯤 짐작하셨겠지만
고유한 상태만 저장하는 객체를 플라이웨이트라고 합니다.

#### 공유한 상태 스토리지

공유한 상태는 어디로 이동할까요? 일부 클래스가 이 상태를 여전히 저장하고
있는 거겠죠? 대부분의 경우 공유한 상태는 패턴을 적용하기 전에 객체들을
집합시키는 컨테이너 객체로 이동됩니다.

당신의 게임에서 이것은 `particle` 필드에 모든 입자를 저장하는 주요
`Game` 객체입니다. 공유한 상태를 이 클래스로 이동하려면 개별 입자의
좌표, 벡터 및 속도를 저장하기 위한 여러 배열 필드들을 생성해야 합니다.
거기서 끝이 아닙니다. 입자를 나타내는 특정 플라이웨이트에 대한 참조를
저장하려면 다른 배열이 필요합니다. 이러한 배열들은 같은 인덱스를
사용하여 입자의 모든 데이터에 액세스할 수 있도록 동기화되어야 합니다.

<figure class="image">
<img
src="../../images/patterns/diagrams/flyweight/solution2-koc311.png?id=65b480543ba6c454157be874b8041b56"
style="aspect-ratio: 1.94;"
srcset="/images/patterns/diagrams/flyweight/solution2-ko.png?id=65b480543ba6c454157be874b8041b56 640w,/images/patterns/diagrams/flyweight/solution2-ko-1.5x.png?id=77b13064f069624a9e6dff63f7147600 960w,/images/patterns/diagrams/flyweight/solution2-ko-2x.png?id=26bb7331def6a35fcc1e2c2ba7dc817c 1280w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="640"
alt="플라이웨이트 패턴 해결책" />
</figure>

이보다 더 훌륭한 해결책은 플라이웨이트 객체에 대한 참조와 함께 공유된
상태를 저장할 별도의 콘텍스트 클래스를 만드는 것입니다. 이 접근 방식을
사용하려면 컨테이너 클래스에 단일 배열만 있으면 됩니다.

잠시만요! 처음에 그랬던 것처럼 이런 콘텍스트 객체들이 많이 있어야 하지
않나요? 맞습니다. 그러나 이제는 이러한 객체들이 이전보다 훨씬 작습니다.
가장 메모리를 많이 사용하는 필드들이 고작 몇 개의 플라이웨이트 객체들로
이동되었습니다. 이제 하나의 커다란 플라이웨이트 객체를 몇천 개의 작은
콘텍스트 객체들이 재사용할 수 있으며, 더 이상 커다란 플라이웨이트 객체의
데이터의 천 개의 복사본을 저장할 필요가 없습니다.

#### 플라이웨이트와 불변성

같은 플라이웨이트 객체가 다른 콘텍스트들에서 사용될 수 있으므로 해당
플라이웨이트 객체의 상태를 수정할 수 없는지 확인해야 합니다.
플라이웨이트는 생성자 매개변수들을 통해 상태를 한 번만 초기화해야
합니다. 또 setter 또는 public 필드들을 다른 객체들에 노출해서는 안
됩니다.

#### 플라이웨이트 팩토리

다양한 플라이웨이트들에 보다 편리하게 액세스하기 위해 기존 플라이웨이트
객체들의 풀을 관리하는 팩토리 메서드를 생성할 수 있습니다. 이 메서드는
클라이언트에서 원하는 플라이웨이트의 고유한 상태를 받아들이고 이 상태와
일치하는 기존 플라이웨이트 객체를 찾고 발견되면 반환합니다. 그렇지
않으면 새 플라이웨이트를 생성하여 풀에 추가합니다.

이 메서드를 배치할 수 있는 몇 가지 옵션이 있습니다. 그중 가장 확실한
장소는 플라이웨이트 컨테이너입니다. 대안으로 당신은 새 팩토리 클래스를
생성할 수 있고, 또 팩토리 메서드를 정적으로 만들고 실제 플라이웨이트
클래스에 넣을 수 있습니다.
:::

::::: {.section .structure-container}
##  구조 {#structure}

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/flyweight/structure7b00.png?id=c1e7e1748f957a4792822f902bc1d420"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.64;"
srcset="/images/patterns/diagrams/flyweight/structure.png?id=c1e7e1748f957a4792822f902bc1d420 640w,/images/patterns/diagrams/flyweight/structure-1.5x.png?id=34cf08f6c2b09dcd1c521c7512cc52b6 960w,/images/patterns/diagrams/flyweight/structure-2x.png?id=a7c8347421bde16435fc90a706f5dd34 1280w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="640"
alt="플라이웨이트 디자인 패턴 구조" /><img
src="../../images/patterns/diagrams/flyweight/structure-indexed086a.png?id=aa490792baa26b04464dacbc1f4a19cd"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.54;"
srcset="/images/patterns/diagrams/flyweight/structure-indexed.png?id=aa490792baa26b04464dacbc1f4a19cd 630w,/images/patterns/diagrams/flyweight/structure-indexed-1.5x.png?id=b22a5eab6aacbbd03c66c34802b240ca 945w,/images/patterns/diagrams/flyweight/structure-indexed-2x.png?id=205e2f7d08b4ac0695f445a9db8989c4 1260w"
sizes="(max-width: 720px) 100vw, 630px" loading="lazy" width="630"
alt="플라이웨이트 디자인 패턴 구조" />
</figure>
:::

1.  플라이웨이트 패턴은 단지 최적화에 불과합니다. 이 패턴을 적용하기
    전에 프로그램이 동시에 메모리에 유사한 객체들을 대량으로 보유하는
    것과 관련된 RAM 소비 문제가 있는지 확인하시고 이 문제가 다른 의미
    있는 방법으로 해결될 수 없는지도 확인하세요.

2.  **플라이웨이트** 클래스에는 여러 객체들 간에 공유할 수 있는 원래
    객체의 상태의 부분이 포함됩니다. 같은 플라이웨이트 객체가 다양한
    콘텍스트에서 사용될 수 있습니다. 플라이웨이트 내부에 저장된 상태를
    *[고]{.cjk}[유]{.cjk}[한]{.cjk}*(intrinsic) 상태라고 하며,
    플라이웨이트의 메서드에 전달된 상태를
    *[공]{.cjk}[유]{.cjk}[한]{.cjk}*(extrinsic) 상태라고 합니다.

3.  **콘텍스트** 클래스는 공유한 상태를 포함하며, 이 상태는 모든 원본
    객체들에서 고유합니다. 콘텍스트가 플라이웨이트 객체 중 하나와 쌍을
    이루면 원래 객체의 전체 상태를 나타냅니다.

4.  일반적으로 원래 객체의 행동은 플라이웨이트 클래스에 남아 있습니다.
    이 경우 플라이웨이트의 메서드의 호출자는 공유한 상태의 적절한
    부분들을 메서드의 매개변수들에 전달해야 합니다. 반면에, 행동은
    콘텍스트 클래스로 이동할 수 있으며, 이 클래스는 연결된
    플라이웨이트를 단순히 데이터 객체로 사용할 것입니다.

5.  **클라이언트**는 플라이웨이트들의 공유된 상태를 저장하거나
    계산합니다. 클라이언트의 관점에서 플라이웨이트는 일부 콘텍스트
    데이터를 그의 메서드들의 매개변수들에 전달하여 런타임에 설정될 수
    있는 템플릿 객체입니다.

6.  **플라이웨이트 팩토리**는 기존 플라이웨이트들의 풀을 관리합니다. 이
    팩토리로 인해 클라이언트들은 플라이웨이트들을 직접 만들지 않는 대신
    원하는 플라이웨이트의 고유한 상태의 일부를 전달하여 공장을
    호출합니다. 팩토리는 이전에 생성된 플라이웨이트들을 살펴보고 검색
    기준과 일치하는 기존 플라이웨이트를 반환하거나 기준에 맞는
    플라이웨이트가 발견되지 않으면 새로 생성합니다.
::::
:::::

::: {.section .pseudocode}
##  의사코드 {#pseudocode}

이 예시에서 **플라이웨이트** 패턴은 캔버스에 수백만 개의 나무 객체들을
렌더링할 때 메모리 사용량을 줄이는 데 도움을 줍니다.

<figure class="image">
<img
src="../../images/patterns/diagrams/flyweight/example5053.png?id=0818d078c1a79f373e96397f37b7ee06"
style="aspect-ratio: 1.54;"
srcset="/images/patterns/diagrams/flyweight/example.png?id=0818d078c1a79f373e96397f37b7ee06 540w,/images/patterns/diagrams/flyweight/example-1.5x.png?id=449fa5906e277c134870c07e33dd4b62 810w,/images/patterns/diagrams/flyweight/example-2x.png?id=9423640fe3688a64201389b6e7aa1f48 1080w"
sizes="(max-width: 720px) 100vw, 540px" loading="lazy" width="540"
alt="플라이웨이트 패턴 예시" />
</figure>

이 패턴은 주요 `Tree`​(나무) 클래스에서 반복되는 고유한 상태를 추출하여
`Tree­Type`​(나무 종류) 플라이웨이트 클래스로 이동합니다.

같은 데이터를 여러 객체에 저장하는 대신 이제 몇 개의 플라이웨이트
객체들에 보관되고 콘텍스트 역할을 하는 적절한 `Tree` 객체들에
연결됩니다. 클라이언트 코드는 플라이웨이트 팩토리를 사용하여 새 `Tree`
객체들을 생성합니다. 이 팩토리는 올바른 객체를 검색하고 필요한 경우
재사용하는 작업의 복잡성을 캡슐화합니다.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// 플라이웨이트 클래스는 트리의 상태 일부를 포함합니다. 이러한 필드는 각 특정
// 트리에 대해 고유한 값들을 저장합니다. 예를 들어 여기에서는 트리 좌표들을 찾을
// 수 없을 것입니다. 그러나 많은 트리들이 공유하는 질감들과 색상들은 찾을 수 있을
// 것입니다. 이 데이터는 일반적으로 크기 때문에 각 트리 개체에 보관하면 많은
// 메모리를 낭비하게 됩니다. 대신 질감, 색상 및 기타 반복되는 데이터를 많은 개별
// 트리 객체들이 참조할 수 있는 별도의 객체로 추출할 수 있습니다.
class TreeType is
    field name
    field color
    field texture
    constructor TreeType(name, color, texture) { ... }
    method draw(canvas, x, y) is
        // 1. 주어진 유형, 색상 및 질감의 비트맵을 만드세요.
        // 2. 비트맵을 캔버스의 X 및 Y 좌표에 그리세요.

// 플라이웨이트 팩토리는 기존 플라이웨이트를 재사용할지 아니면 새로운 객체를
// 생성할지를 결정합니다.
class TreeFactory is
    static field treeTypes: collection of tree types
    static method getTreeType(name, color, texture) is
        type = treeTypes.find(name, color, texture)
        if (type == null)
            type = new TreeType(name, color, texture)
            treeTypes.add(type)
        return type

// 콘텍스트 객체는 트리 상태의 공유된 부분을 포함합니다. 이러한 부분들은 두 개의
// 정수로 된 좌표와 하나의 참조 필드만 참조하여 크기가 작기 때문에 하나의 앱이
// 이런 부분을 수십억 개씩 만들 수 있습니다.
class Tree is
    field x,y
    field type: TreeType
    constructor Tree(x, y, type) { ... }
    method draw(canvas) is
        type.draw(canvas, this.x, this.y)

// Tree 및 Forest 클래스들은 플라이웨이트의 클라이언트들이며 Tree 클래스를 더
// 이상 개발할 계획이 없으면 이 둘을 병합할 수 있습니다.
class Forest is
    field trees: collection of Trees

    method plantTree(x, y, name, color, texture) is
        type = TreeFactory.getTreeType(name, color, texture)
        tree = new Tree(x, y, type)
        trees.add(tree)

    method draw(canvas) is
        foreach (tree in trees) do
            tree.draw(canvas)</code></pre>
</figure>
:::

:::::: {.section .applicability-container}
##  적용 {#applicability}

::::: applicability
::: applicability-problem
플라이웨이트 패턴은 당신의 프로그램이 많은 수의 객체들을 지원해야 해서
사용할 수 있는 RAM을 거의 다 사용했을 때만 사용하세요.
:::

::: applicability-solution
이 패턴 적용의 혜택은 패턴을 사용하는 방법과 위치에 따라 크게 달라지며,
다음과 같은 경우에 가장 유용합니다.

-   앱이 수많은 유사 객체들을 생성해야 할 때
-   이것이 대상 장치에서 사용할 수 있는 모든 RAM을 소모할 때
-   이 객체들에 여러 중복 상태들이 포함되어 있으며, 이 상태들이 추출된
    후 객체 간에 공유될 수 있을 때
:::
:::::
::::::

::: {.section .checklist}
##  구현방법 {#checklist}

1.  플라이웨이트가 될 클래스의 필드들을 두 부분으로 나누세요.

    -   고유한 상태: 많은 객체에 걸쳐 복제된 변경되지 않는 데이터를
        포함하는 필드들
    -   공유한 상태: 각 객체에 고유한 콘텍스트 데이터를 포함하는 필드들

2.  클래스의 고유한 상태를 나타내는 필드들은 그대로 두되 변경될 수
    없도록 하세요. 이 필드들은 생성자 내부에서만 초깃값들을 가져와야
    합니다.

3.  공유한 상태의 필드들을 사용하는 메서드들을 살펴보세요. 메서드에
    사용된 각 필드에 대해 새 매개변수를 도입하고 필드 대신 사용하세요.

4.  옵션으로, 플라이웨이트들의 풀을 관리하기 위한 팩토리 클래스를
    생성하세요. 이 클래스는 새 플라이웨이트를 만들기 전에 기존
    플라이웨이트의 존재 여부를 확인해야 합니다. 팩토리가 설치되면 고객은
    팩토리를 통해서만 플라이웨이트를 요청해야 합니다. 그들은 팩토리에
    플라이웨이트의 고유한 상태를 전달하여 원하는 플라이웨이트를 설명해야
    합니다.

5.  클라이언트는 플라이웨이트 객체들의 메서드들을 호출할 수 있도록
    공유한 상태의 값들​(콘텍스트)​을 저장하거나 계산해야 합니다. 편의상,
    플라이웨이트를 참조하는 필드와 공유한 상태는 별도의 콘텍스트
    클래스로 이동할 수 있습니다.
:::

:::::: {.section .pros-cons}
##  장단점 {#pros-cons}

::::: row
::: col-sm-6
-    당신의 프로그램에 유사한 객체들이 많다고 가정하면 많은 RAM을 절약할
    수 있습니다.
:::

::: col-sm-6
-    누군가가 플라이웨이트 메서드를 호출할 때마다 콘텍스트 데이터의
    일부를 다시 계산해야 한다면 당신은 CPU 주기 대신 RAM을 절약하고 있는
    것일지도 모릅니다.
-    코드가 복잡해지므로 새로운 팀원들은 왜 개체​(entity)​의 상태가 그런
    식으로 분리되었는지 항상 궁금해할 것입니다.
:::
:::::
::::::

::: {.section .relations}
##  다른 패턴과의 관계 {#relations}

-   RAM을 절약하기 위하여 [복합체](composite.html) 패턴 트리의 공유된 잎
    노드들을 [플라이웨이트들](flyweight.html)로 구현할 수 있습니다.

-   [플라이웨이트](flyweight.html)는 작은 객체들을 많이 만드는 방법을
    보여 주는 반면 [퍼사드](facade.html) 패턴은 전체 하위 시스템을
    나타내는 단일 객체를 만드는 방법을 보여 줍니다.

-   만약 객체들의 공유된 상태들을 단 하나의 플라이웨이트 객체로 줄일 수
    있다면 [플라이웨이트](flyweight.html)는 [싱글턴](singleton.html)과
    유사해질 수 있습니다. 그러나 이 패턴들에는 두 가지 근본적인 차이점이
    있습니다:

    1.  싱글턴은 인스턴스가 하나만 있어야 합니다. 반면에
        *[플]{.cjk}[라]{.cjk}[이]{.cjk}[웨]{.cjk}[이]{.cjk}[트]{.cjk}*
        클래스는 여러 고유한 상태를 가진 여러 인스턴스를 포함할 수
        있습니다.
    2.  *[싱]{.cjk}[글]{.cjk}[턴]{.cjk}* 객체는 변할 수 있습니다
        (mutable). 플라이웨이트 객체들은 변할 수 없습니다 (immutable).
:::

::: {.section .implementations}
##  코드 예시 {#implementations}

[![C#으로 작성된
플라이웨이트](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](flyweight/csharp/example.html "C#으로 작성된 플라이웨이트"){.prog-lang-link}
[![C++로 작성된
플라이웨이트](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](flyweight/cpp/example.html "C++로 작성된 플라이웨이트"){.prog-lang-link}
[![Go로 작성된
플라이웨이트](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](flyweight/go/example.html "Go로 작성된 플라이웨이트"){.prog-lang-link}
[![자바로 작성된
플라이웨이트](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](flyweight/java/example.html "자바로 작성된 플라이웨이트"){.prog-lang-link}
[![PHP로 작성된
플라이웨이트](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](flyweight/php/example.html "PHP로 작성된 플라이웨이트"){.prog-lang-link}
[![파이썬으로 작성된
플라이웨이트](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](flyweight/python/example.html "파이썬으로 작성된 플라이웨이트"){.prog-lang-link}
[![루비로 작성된
플라이웨이트](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](flyweight/ruby/example.html "루비로 작성된 플라이웨이트"){.prog-lang-link}
[![러스트로 작성된
플라이웨이트](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](flyweight/rust/example.html "러스트로 작성된 플라이웨이트"){.prog-lang-link}
[![스위프트로 작성된
플라이웨이트](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](flyweight/swift/example.html "스위프트로 작성된 플라이웨이트"){.prog-lang-link}
[![타입스크립트로 작성된
플라이웨이트](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](flyweight/typescript/example.html "타입스크립트로 작성된 플라이웨이트"){.prog-lang-link}
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

[프록시 []{.fa .fa-arrow-right}](proxy.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### 돌아가기

[[]{.fa .fa-arrow-left} 퍼사드](facade.html){.btn .btn-default
rel="prev"}
:::
:::::::::::::::::::::::::::::

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
:::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::
