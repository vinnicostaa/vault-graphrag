:::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [디자인
패턴들](../design-patterns.html){.type} / [생성
패턴](creational-patterns.html){.type}
:::

# 프로토타입 패턴 {#프로토타입-패턴 .title}

::: aka
다음 이름으로도 불립니다:
[클론, ]{style="display:inline-block"}[Prototype]{style="display:inline-block"}
:::

::: {.section .intent}
##  의도 {#intent}

**프로토타입**은 코드를 그들의 클래스들에 의존시키지 않고 기존 객체들을
복사할 수 있도록 하는 생성 디자인 패턴입니다.

<figure class="image">
<img
src="../../images/patterns/content/prototype/prototype3cb9.png?id=e912b1ada20bbf7b2ffc09e93b9fab20"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/prototype/prototype.png?id=e912b1ada20bbf7b2ffc09e93b9fab20 640w,/images/patterns/content/prototype/prototype-1.5x.png?id=a0f76894fb657783b65b9ed899857468 960w,/images/patterns/content/prototype/prototype-2x.png?id=670789c80c8a114e25838ede2da4a881 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="프로토타입 디자인 패턴" />
</figure>
:::

::: {.section .problem}
##  문제 {#problem}

객체가 있고 그 객체의 정확한 복사본을 만들고 싶다고 가정하면, 어떻게
하시겠습니까? 먼저 같은 클래스의 새 객체를 생성해야 합니다. 그런 다음
원본 객체의 모든 필드들을 살펴본 후 해당 값들을 새 객체에 복사해야
합니다.

너무 쉽군요! 하지만 함정이 있습니다. 객체의 필드들 중 일부가 비공개여서
객체 자체의 외부에서 볼 수 없을 수 있으므로 모든 객체를 그런 식으로
복사하지 못합니다.

<figure class="image">
<img
src="../../images/patterns/content/prototype/prototype-comic-1-koc2f5.png?id=f58232c36ce6751e7382bac9c7639179"
style="aspect-ratio: 2;"
srcset="/images/patterns/content/prototype/prototype-comic-1-ko.png?id=f58232c36ce6751e7382bac9c7639179 600w,/images/patterns/content/prototype/prototype-comic-1-ko-1.5x.png?id=a52e1776db2e9059622383d99e112e85 900w,/images/patterns/content/prototype/prototype-comic-1-ko-2x.png?id=749c9237af20dfa2a49ad90d66e67b0e 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="&#39;외부에서&#39; 항목을 복사할 때 무엇이 잘못될 수 있을까요?" />
<figcaption><p>객체를 '외부에서부터' 복사하는 것은 항상 가능하지 <a
href="https://vimeo.com/120816425">않습니다</a>.</p></figcaption>
</figure>

이 직접적인 접근 방식에는 한 가지 문제가 더 있습니다. 객체의 복제본을
생성하려면 객체의 클래스를 알아야 하므로, 당신의 코드가 해당 클래스에
의존하게 된다는 것입니다. 또, 예를 들어 메서드의 매개변수가 일부
인터페이스를 따르는 모든 객체를 수락할 때 당신은 그 객체가 따르는
인터페이스만 알고, 그 객체의 구상 클래스는 알지 못할 수 있습니다.
:::

::: {.section .solution}
##  해결책 {#solution}

프로토타입 패턴은 실제로 복제되는 객체들에 복제 프로세스를 위임합니다.
패턴은 복제를 지원하는 모든 객체에 대한 공통 인터페이스를 선언합니다. 이
인터페이스를 사용하면 코드를 객체의 클래스에 결합하지 않고도 해당 객체를
복제할 수 있습니다. 일반적으로 이러한 인터페이스에는 단일 `clone`
메서드만 포함됩니다.

`clone` 메서드의 구현은 모든 클래스에서 매우 유사합니다. 이 메서드는
현재 클래스의 객체를 만든 후 이전 객체의 모든 필드 값을 새 객체로
전달합니다. 대부분의 프로그래밍 언어는 객체들이 같은 클래스에 속한 다른
객체의 비공개 필드들에 접근​(access) 할 수 있도록 하므로 비공개 필드들을
복사하는 것도 가능합니다.

복제를 지원하는 객체를
*[프]{.cjk}[로]{.cjk}[토]{.cjk}[타]{.cjk}[입]{.cjk}*이라고 합니다.
당신의 객체들에 수십 개의 필드와 수백 개의 가능한 설정들이 있는 경우
이를 복제하는 것이 서브클래싱의 대안이 될 수 있습니다.

<figure class="image">
<img
src="../../images/patterns/content/prototype/prototype-comic-2-kob213.png?id=64ef8ce56ebb149fdb0cf28bde2d9db6"
style="aspect-ratio: 1.14;"
srcset="/images/patterns/content/prototype/prototype-comic-2-ko.png?id=64ef8ce56ebb149fdb0cf28bde2d9db6 343w,/images/patterns/content/prototype/prototype-comic-2-ko-1.5x.png?id=6cc25c9a034de72727de567979763d60 515w,/images/patterns/content/prototype/prototype-comic-2-ko-2x.png?id=ce624a10f6089d7a11603777b3776e15 687w"
sizes="(max-width: 720px) 100vw, 343px" loading="lazy" width="343"
alt="미리 만들어진 프로토타입들" />
<figcaption><p>미리 만들어진 프로토타입은 서브클래싱의 대안이 될
수 있습니다.</p></figcaption>
</figure>

프로토타이핑은 다음과 같이 작동합니다. 일단, 다양한 방식으로 설정된
객체들의 집합을 만듭니다. 그 후 설정한 것과 비슷한 객체가 필요할 경우
처음부터 새 객체를 생성하는 대신 프로토타입을 복제하면 됩니다.
:::

::: {.section .analogy}
##  실제상황 적용 {#analogy}

실제 산업에서의 프로토타입​(원기)​은 제품의 대량 생산을 시작하기 전에
다양한 테스트를 수행하는 데 사용됩니다. 그러나 프로그래밍의 프로토타입의
경우 프로토타입들은 실제 생산과정에 참여하지 않고 대신 수동적인 역할을
합니다.

<figure class="image">
<img
src="../../images/patterns/content/prototype/prototype-comic-3-kocd37.png?id=1ffad8308001769271945e1b0c31a5f5"
style="aspect-ratio: 2;"
srcset="/images/patterns/content/prototype/prototype-comic-3-ko.png?id=1ffad8308001769271945e1b0c31a5f5 600w,/images/patterns/content/prototype/prototype-comic-3-ko-1.5x.png?id=871c432d9afc8ff35b9bba4d0f528fe2 900w,/images/patterns/content/prototype/prototype-comic-3-ko-2x.png?id=f92e302baa2bdf62fd95efae606800d2 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="세포 분열" />
<figcaption><p>세포의 분열</p></figcaption>
</figure>

산업 프로토타입들은 실제로 자신을 복제하지 않기 때문에, 프로토타입
패턴에 더 가까운 예시는 세포의 유사분열 과정입니다. 유사분열 후에는 한
쌍의 같은 세포가 형성됩니다. 원본 세포는 프로토타입 역할을 하며 복사본을
만드는 데 능동적인 역할을 합니다.
:::

:::::::: {.section .structure-container}
##  구조 {#structure}

::::::: structure
#### 기초 구현

:::: prototype-normal
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/prototype/structure87f6.png?id=088102c5e9785ff45debbbce86f4df81"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.25;"
srcset="/images/patterns/diagrams/prototype/structure.png?id=088102c5e9785ff45debbbce86f4df81 500w,/images/patterns/diagrams/prototype/structure-1.5x.png?id=beec6a1a5242268e10e39f9fdc0b894b 750w,/images/patterns/diagrams/prototype/structure-2x.png?id=ba75079f42f08028ae4cdbda0cfecc26 1000w"
sizes="(max-width: 720px) 100vw, 500px" loading="lazy" width="500"
alt="프로토타입 디자인 패턴의 구조" /><img
src="../../images/patterns/diagrams/prototype/structure-indexed6499.png?id=0e1c809842f5c43aca0541a2eba1f844"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.27;"
srcset="/images/patterns/diagrams/prototype/structure-indexed.png?id=0e1c809842f5c43aca0541a2eba1f844 520w,/images/patterns/diagrams/prototype/structure-indexed-1.5x.png?id=05b072b5b83f5ff1024a7ba448ea9d59 780w,/images/patterns/diagrams/prototype/structure-indexed-2x.png?id=74584ac729c0f6b49d2a97a53cc266ff 1040w"
sizes="(max-width: 720px) 100vw, 520px" loading="lazy" width="520"
alt="프로토타입 디자인 패턴의 구조" />
</figure>
:::
::::

1.  **프로토타입** 인터페이스는 복제 메서드들을 선언하며, 이 메서드들의
    대부분은 단일 `clone` 메서드입니다.

2.  **구상 프로토타입** 클래스는 복제 메서드를 구현합니다. 원본 객체의
    데이터를 복제본에 복사하는 것 외에도 이 메서드는 복제 프로세스와
    관련된 일부 예외적인 경우들도 처리할 수도 있습니다. (예: 연결된 객체
    복제, 재귀 종속성 풀기).

3.  **클라이언트**는 프로토타입 인터페이스를 따르는 모든 객체의 복사본을
    생성할 수 있습니다.

#### 프로토타입 레지스트리 구현

:::: prototype-pool
::: {.struct-image2 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/prototype/structure-prototype-cache9d19.png?id=609c2af5d14ed55dcbb218a00f98e7d5"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.15;"
srcset="/images/patterns/diagrams/prototype/structure-prototype-cache.png?id=609c2af5d14ed55dcbb218a00f98e7d5 550w,/images/patterns/diagrams/prototype/structure-prototype-cache-1.5x.png?id=8ca0b829185d49c5acbe19966a0659d4 825w,/images/patterns/diagrams/prototype/structure-prototype-cache-2x.png?id=a1e4514bbcc9b10968b856f19b407105 1100w"
sizes="(max-width: 720px) 100vw, 550px" loading="lazy" width="550"
alt="프로토타입 레지스트리" /><img
src="../../images/patterns/diagrams/prototype/structure-prototype-cache-indexed1801.png?id=10a4a84a1a318f59dbc2b806fc936d04"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.12;"
srcset="/images/patterns/diagrams/prototype/structure-prototype-cache-indexed.png?id=10a4a84a1a318f59dbc2b806fc936d04 550w,/images/patterns/diagrams/prototype/structure-prototype-cache-indexed-1.5x.png?id=cb56c95533a4020368c48db9f9e2a37d 825w,/images/patterns/diagrams/prototype/structure-prototype-cache-indexed-2x.png?id=47b99eb7ae51158bdbb31deea4f5e98f 1100w"
sizes="(max-width: 720px) 100vw, 550px" loading="lazy" width="550"
alt="프로토타입 레지스트리" />
</figure>
:::
::::

1.  **프로토타입 레지스트리**는 자주 사용하는 프로토타입들에 쉽게
    접근​(액세스)​하는 방법을 제공합니다. 이 레지스트리는 복사될 준비가 된
    미리 만들어진 객체들의 집합을 저장합니다. 가장 간단한 프로토타입
    레지스트리는 `name → prototype` 해시 맵입니다. 그러나 단순히 이름을
    검색하는 것보다 더 나은 검색 기준이 필요한 경우 훨씬 더 탄탄한
    레지스트리를 구축할 수 있습니다.
:::::::
::::::::

::: {.section .pseudocode}
##  의사코드 {#pseudocode}

아래 예시에서의 **프로토타입** 패턴은 코드를 기하학적 객체들의
클래스들에 결합하지 않고도 해당 객체들의 정확한 복사본을 생성할 수
있도록 합니다.

<figure class="image">
<img
src="../../images/patterns/diagrams/prototype/examplee491.png?id=47bc6c1058cb100b81e675b5ca6bda6c"
style="aspect-ratio: 1.42;"
srcset="/images/patterns/diagrams/prototype/example.png?id=47bc6c1058cb100b81e675b5ca6bda6c 470w,/images/patterns/diagrams/prototype/example-1.5x.png?id=067f3a2415370fadf5f92aadaa12b5e2 705w,/images/patterns/diagrams/prototype/example-2x.png?id=80393e0df17ae8130e5ada832d494949 940w"
sizes="(max-width: 720px) 100vw, 470px" loading="lazy" width="470"
alt="프로토타입 패턴 구조 예시" />
<figcaption><p>클래스 계층구조에 속한 객체 집합의 복제.</p></figcaption>
</figure>

모든 shape(모양) 클래스는 같은 인터페이스를 따르며, 이 인터페이스는 복제
메서드를 제공합니다. 자식 클래스는 자신의 필드 값들을 생성된 객체에
복사하기 전에 부모의 복제 메서드를 호출할 수 있습니다.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// 기초 프로토타입.
abstract class Shape is
    field X: int
    field Y: int
    field color: string

    // 일반 생성자.
    constructor Shape() is
        // …

    // 프로토타입 생성자. 기존 객체의 값들로 새로운 객체가 초기화됩니다.
    constructor Shape(source: Shape) is
        this()
        this.X = source.X
        this.Y = source.Y
        this.color = source.color

    // 복제 작업은 Shape(모양) 자식 클래스 중 하나를 반환합니다.
    abstract method clone():Shape


// 구상 프로토타입. 복제 메서드는 현재 클래스의 생성자를 호출해 현재 객체를
// 생성자의 인수로 전달함으로써 한 번에 새로운 객체를 생성합니다. 생성자에서
// 실제로 모든 것을 복사하게 되면 결과의 일관성이 유지됩니다. 생성자가 새로운
// 객체가 완전히 완성되기 전까지 결과를 반환하지 않아서 어떤 객체도 일부분만 완성된
// 복제본을 참조할 수 없습니다.
class Rectangle extends Shape is
    field width: int
    field height: int

    constructor Rectangle(source: Rectangle) is
        // 부모 클래스에 정의된 비공개 필드들을 복사하려면 부모 생성자 호출이
        // 필요합니다.
        super(source)
        this.width = source.width
        this.height = source.height

    method clone():Shape is
        return new Rectangle(this)


class Circle extends Shape is
    field radius: int

    constructor Circle(source: Circle) is
        super(source)
        this.radius = source.radius

    method clone():Shape is
        return new Circle(this)


// 클라이언트 코드의 어딘가에…
class Application is
    field shapes: array of Shape

    constructor Application() is
        Circle circle = new Circle()
        circle.X = 10
        circle.Y = 10
        circle.radius = 20
        shapes.add(circle)

        Circle anotherCircle = circle.clone()
        shapes.add(anotherCircle)
        // `anotherCircle` 변수에는 `circle` 객체와 똑같은 사본이 포함되어
        // 있습니다.

        Rectangle rectangle = new Rectangle()
        rectangle.width = 10
        rectangle.height = 20
        shapes.add(rectangle)

    method businessLogic() is
        // 프로토타입은 매우 유용합니다! 왜냐하면 프로토타입은 당신이 복사하려는
        // 객체의 유형에 대해 아무것도 몰라도 복사본을 생성할 수 있도록 하기
        // 때문입니다.
        Array shapesCopy = new Array of Shapes.

        // 예를 들어, 우리는 shapes(모양들) 배열의 정확한 요소들을 알지
        // 못하며, 이 요소들이 모양이라는 것만 압니다. 그러나 다형성 덕분에
        // 모양의 `clone`(복제) 메서드를 호출하면 프로그램이 모양의 실제
        // 클래스를 확인하고 해당 클래스에 정의된 적절한 복제 메서드를
        // 실행합니다. 그래서 우리가 단순한 모양 객체들의 집합이 아닌 적절한
        // 복제본들을 얻는 것이죠.
        foreach (s in shapes) do
            shapesCopy.add(s.clone())

        // `shapesCopy`(모양들의 복사본) 배열에는 `shape`(모양) 배열의
        // 자식들과 정확히 일치하는 복사본들이 포함되어 있습니다.</code></pre>
</figure>
:::

:::::::: {.section .applicability-container}
##  적용 {#applicability}

::::::: applicability
::: applicability-problem
프로토타입 패턴은 복사해야 하는 객체들의 구상 클래스들에 코드가 의존하면
안 될 때 사용하세요.
:::

::: applicability-solution
이와 같은 경우는 당신의 코드가 어떤 인터페이스를 통해 타사 코드에서
전달된 객체들과 함께 작동할 때 많이 발생합니다. 이러한 객체들의 구상
클래스들은 알려지지 않았기 때문에 이러한 클래스들에 의존할 수 없습니다.

프로토타입 패턴은 클라이언트 코드에 복제를 지원하는 모든 객체와 작업할
수 있도록 일반 인터페이스를 제공합니다. 이 인터페이스는 클라이언트
코드가 복제하는 객체들의 구상 클래스들에서 클라이언트 코드를
독립시킵니다.
:::

::: applicability-problem
프로토타입 패턴은 각자의 객체를 초기화하는 방식만 다른 자식 클래스들의
수를 줄이고 싶을 때 사용하세요.
:::

::: applicability-solution
사용하기 전에 많은 설정이 필요한 복잡한 클래스가 있다고 가정해 봅시다.
이 클래스를 설정하는 데는 몇 가지 일반적인 방법들이 있으며 설정되어야
하는 클래스의 새로운 인스턴스들의 생성을 담당하는 코드는 당신의 앱에
흩어져 있습니다. 중복을 줄이기 위해 당신은 여러 자식 클래스들을 만들어
모든 공통 설정 코드를 그 클래스들의 생성자들에 넣었습니다. 이렇게 중복
문제는 해결했지만 이제 쓸모없는 자식 클래스들이 많이 생겼습니다.

프로토타입 패턴은 다양한 방식으로 설정된 미리 만들어진 객체들의 집합을
프로토타입들로 사용할 수 있도록 합니다. 일부 설정과 일치하는 자식
클래스를 인스턴스화하는 대신 클라이언트는 간단하게 적절한 프로토타입을
찾아 복제할 수 있습니다.
:::
:::::::
::::::::

::: {.section .checklist}
##  구현방법 {#checklist}

1.  프로토타입 인터페이스를 생성한 후 그 안에 `clone` 메서드를
    선언하세요. 또는 기존 계층 구조가 있는 경우, 이 메서드를 그 계층
    구조의 모든 클래스들에 추가하세요.

2.  프로토타입 클래스는 이 클래스의 객체를 인수로 받아들이는 대체
    생성자를 반드시 정의해야 합니다. 또 생성자는 이 클래스에 정의된 모든
    필드의 값들을 전달된 객체에서 새로 생성된 인스턴스로 복사해야
    합니다. 또 자식 클래스를 변경할 때에는 부모 생성자를 호출하여 부모
    클래스가 부모 클래스의 비공개 필드들의 복제를 처리하도록 해야
    합니다.

    현재 사용 중인 프로그래밍 언어가 메서드 오버로딩을 지원하지 않으면
    별도의 \'프로토타입\' 생성자를 만들 수 없습니다. 따라서 객체의
    데이터를 새로 생성된 복제본에 복사하는 작업은 `clone`​(복제본) 메서드
    내에서 수행되어야 합니다. 그래도 이 코드를 일반적인 생성자에 두는
    것이 더 안전한 이유는 `new` 연산자를 호출한 직후에 생성된 객체는
    완전히 설정된 상태로 반환되기 때문입니다.

3.  복제 메서드는 일반적으로 한 줄로 구성됩니다. 이 줄은 생성자의
    프로토타입 버전으로 `new` 연산자를 실행합니다. 모든 클래스는 복제
    메서드를 명시적으로 오버라이딩한 후 `new` 연산자와 함께 자체 클래스
    이름을 사용해야 합니다. 그렇게 하지 않으면 복제 메서드가 부모
    클래스의 객체를 생성할 수 있습니다.

4.  또, 추가 옵션으로 자주 사용하는 프로토타입들의 카탈로그를 저장할
    중앙 프로토타입 레지스트리를 생성할 수 있습니다.

    레지스트리를 새 팩토리 클래스로 구현하거나 레지스트리를 기초
    프로토타입 클래스에 프로토타입을 가져오기 위한 정적 메서드와 함께
    넣을 수 있습니다. 이 정적 메서드는 클라이언트 코드가 메서드에
    전달하는 검색 기준을 기반으로 프로토타입을 검색해야 합니다. 이때
    검색 기준은 단순한 문자열 태그이거나 복잡한 검색 매개변수들의 집합일
    수 있습니다. 적절한 프로토타입을 찾고 나면, 레지스트리는 이를 복제한
    후 복사본을 클라이언트에 반환해야 합니다.

    마지막으로, 자식 클래스들의 생성자들에 대한 직접 호출들을 프로토타입
    레지스트리의 팩토리 메서드에 대한 호출들로 대체하세요.
:::

:::::: {.section .pros-cons}
##  장단점 {#pros-cons}

::::: row
::: col-sm-6
-    당신은 객체들을 그 구상 클래스들에 결합하지 않고 복제할 수
    있습니다.
-    반복되는 초기화 코드를 제거한 후 그 대신 미리 만들어진
    프로토타입들을 복제하는 방법을 사용할 수 있습니다.
-    복잡한 객체들을 더 쉽게 생성할 수 있습니다.
-    복잡한 객체들에 대한 사전 설정들을 처리할 때 상속 대신 사용할 수
    있는 방법입니다.
:::

::: col-sm-6
-    순환 참조가 있는 복잡한 객체들을 복제하는 것은 매우 까다로울 수
    있습니다.
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

-   [프로토타입](prototype.html)은 [커맨드](command.html) 패턴의
    복사본들을 기록에 저장해야 할 때 도움이 될 수 있습니다.

-   [데코레이터](decorator.html) 및 [복합체](composite.html) 패턴을 많이
    사용하는 디자인들은 [프로토타입](prototype.html)을 사용하면 종종
    이득을 볼 수 있습니다. 프로토타입 패턴을 적용하면 복잡한 구조들을
    처음부터 다시 건축하는 대신 복제할 수 있기 때문입니다.

-   [프로토타입](prototype.html)은 상속을 기반으로 하지 않으므로 상속과
    관련된 단점들이 없습니다. 반면에
    *[프]{.cjk}[로]{.cjk}[토]{.cjk}[타]{.cjk}[입]{.cjk}*은 복제된 객체의
    복잡한 초기화가 필요합니다. [팩토리 메서드](factory-method.html)는
    상속을 기반으로 하지만 초기화 단계가 필요하지 않습니다.

-   때로는 [프로토타입](prototype.html)이 [메멘토](memento.html) 패턴의
    더 간단한 대안이 될 수 있으며, 이 패턴은 상태를 기록에 저장하려는
    객체가 간단하고 외부 리소스에 대한 링크가 없거나 링크들이 있어도
    이들을 재설정하기 쉬운 경우에 작동합니다.

-   [추상 팩토리들](abstract-factory.html), [빌더들](builder.html) 및
    [프로토타입들](prototype.html)은 모두 [싱글턴](singleton.html)으로
    구현할 수 있습니다.
:::

::: {.section .implementations}
##  코드 예시 {#implementations}

[![C#으로 작성된
프로토타입](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](prototype/csharp/example.html "C#으로 작성된 프로토타입"){.prog-lang-link}
[![C++로 작성된
프로토타입](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](prototype/cpp/example.html "C++로 작성된 프로토타입"){.prog-lang-link}
[![Go로 작성된
프로토타입](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](prototype/go/example.html "Go로 작성된 프로토타입"){.prog-lang-link}
[![자바로 작성된
프로토타입](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](prototype/java/example.html "자바로 작성된 프로토타입"){.prog-lang-link}
[![PHP로 작성된
프로토타입](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](prototype/php/example.html "PHP로 작성된 프로토타입"){.prog-lang-link}
[![파이썬으로 작성된
프로토타입](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](prototype/python/example.html "파이썬으로 작성된 프로토타입"){.prog-lang-link}
[![루비로 작성된
프로토타입](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](prototype/ruby/example.html "루비로 작성된 프로토타입"){.prog-lang-link}
[![러스트로 작성된
프로토타입](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](prototype/rust/example.html "러스트로 작성된 프로토타입"){.prog-lang-link}
[![스위프트로 작성된
프로토타입](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](prototype/swift/example.html "스위프트로 작성된 프로토타입"){.prog-lang-link}
[![타입스크립트로 작성된
프로토타입](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](prototype/typescript/example.html "타입스크립트로 작성된 프로토타입"){.prog-lang-link}
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

[싱글턴 []{.fa .fa-arrow-right}](singleton.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### 돌아가기

[[]{.fa .fa-arrow-left} 빌더](builder.html){.btn .btn-default
rel="prev"}
:::
:::::::::::::::::::::::::::::::::::

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
:::::::::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::::::::
