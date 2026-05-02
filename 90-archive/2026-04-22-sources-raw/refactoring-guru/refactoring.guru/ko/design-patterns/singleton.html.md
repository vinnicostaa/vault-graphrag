::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [디자인
패턴들](../design-patterns.html){.type} / [생성
패턴](creational-patterns.html){.type}
:::

# 싱글턴 패턴 {#싱글턴-패턴 .title}

::: aka
다음 이름으로도 불립니다: [Singleton]{style="display:inline-block"}
:::

::: {.section .intent}
##  의도 {#intent}

**싱글턴**은 클래스에 인스턴스가 하나만 있도록 하면서 이 인스턴스에 대한
전역 접근​(액세스) 지점을 제공하는 생성 디자인 패턴입니다.

<figure class="image">
<img
src="../../images/patterns/content/singleton/singleton4263.png?id=108a0b9b5ea5c4426e0afa4504491d6f"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/singleton/singleton.png?id=108a0b9b5ea5c4426e0afa4504491d6f 640w,/images/patterns/content/singleton/singleton-1.5x.png?id=29490ad6cd1426c63cf5f88243a1701c 960w,/images/patterns/content/singleton/singleton-2x.png?id=accb2cc7594f7a491ce01dddf0d2f876 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="싱글턴 패턴" />
</figure>
:::

::: {.section .problem}
##  문제 {#problem}

싱글턴 패턴은 한 번에 두 가지의 문제를 동시에 해결함으로써
*[단]{.cjk}[일]{.cjk} [책]{.cjk}[임]{.cjk} [원]{.cjk}[칙]{.cjk}*을
위반합니다.

1.  **클래스에 인스턴스가 하나만 있도록 합니다**. 사람들은 클래스에 있는
    인스턴스 수를 제어하려는 가장 일반적인 이유는 일부 공유 리소스​(예:
    데이터베이스 또는 파일)​에 대한 접근을 제어하기 위함입니다.

    예를 들어 객체를 생성했지만 잠시 후 새 객체를 생성하기로 했다고
    가정해 봅시다. 그러면 새 객체를 생성하는 대신 이미 만든 객체를 받게
    됩니다.

    물론 생성자 호출은 특성상 **반드시** 새 객체를 반환해야 하므로 위
    행동은 일반 생성자로 구현할 수 없습니다.

<figure class="image">
<img
src="../../images/patterns/content/singleton/singleton-comic-1-koeb35.png?id=b0a209b736f252f5457f0e1fde6e19ac"
style="aspect-ratio: 2;"
srcset="/images/patterns/content/singleton/singleton-comic-1-ko.png?id=b0a209b736f252f5457f0e1fde6e19ac 600w,/images/patterns/content/singleton/singleton-comic-1-ko-1.5x.png?id=d31f6a440503bc3f671e773332ee85af 900w,/images/patterns/content/singleton/singleton-comic-1-ko-2x.png?id=93a10e4ad8d01422682886ee0419bb29 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="객체에 대한 전역 접근" />
<figcaption><p>클라이언트들은 항상 같은 객체와 작업하고 있다는 사실을
인식조차 못 할 수 있습니다.</p></figcaption>
</figure>

2.  **해당 인스턴스에 대한 전역 접근 지점을 제공합니다**. 필수 객체들을
    저장하기 위해 전역 변수들을 정의했다고 가정해 봅시다. 이 변수들을
    사용하면 매우 편리할지는 몰라도, 모든 코드가 잠재적으로 해당 변수의
    내용을 덮어쓸 수 있고 그로 인해 앱에 오류가 발생해 충돌할 수
    있으므로 그리 안전한 방법은 아닙니다.

    전역 변수와 마찬가지로 싱글턴 패턴을 사용하면 프로그램의 모든
    곳에서부터 일부 객체에 접근할 수 있습니다. 그러나 이 패턴은 다른
    코드가 해당 인스턴스를 덮어쓰지 못하도록 보호하기도 합니다.

    이 문제에는 또 다른 측면이 있습니다. 당신은 첫 번째 문제를 해결하는
    코드가 프로그램 전체에 흩어져 있는 것을 원하지 않을 것입니다. 특히
    코드의 나머지 부분이 이미 첫 번째 문제를 해결하는 코드에 의존하고
    있다면, 이 코드를 한 클래스 내에 두는 것이 훨씬 좋습니다.

최근에는 싱글턴 패턴이 워낙 대중화되어 패턴이 나열된 문제 중 한 가지만
해결하더라도 그것을 *[싱]{.cjk}[글]{.cjk}[턴]{.cjk}*이라고 부를 수
있습니다.
:::

::: {.section .solution}
##  해결책 {#solution}

싱글턴의 모든 구현은 공통적으로 다음의 두 단계를 갖습니다.

-   다른 객체들이 싱글턴 클래스와 함께 `new` 연산자를 사용하지 못하도록
    디폴트 생성자를 비공개로 설정하세요.
-   생성자 역할을 하는 정적 생성 메서드를 만드세요. 내부적으로 이
    메서드는 객체를 만들기 위하여 비공개 생성자를 호출한 후 객체를 정적
    필드에 저장합니다. 이 메서드에 대한 그다음 호출들은 모두 캐시된
    객체를 반환합니다.

당신의 코드가 싱글턴 클래스에 접근할 수 있는 경우, 이 코드는 싱글턴의
정적 메서드를 호출할 수 있습니다. 따라서 해당 메서드가 호출될 때마다
항상 같은 객체가 반환됩니다.
:::

::: {.section .analogy}
##  실제상황 적용 {#analogy}

정부는 싱글턴 패턴의 훌륭한 예입니다. 국가는 하나의 공식 정부만 가질 수
있습니다. 그리고 \'X의 정부\'라는 명칭은 정부를 구성하는 개인들의 신원과
관계없이 정부 책임자들의 그룹을 식별하는 글로벌 접근 지점입니다.
:::

::::: {.section .structure-container}
##  구조 {#structure}

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/singleton/structure-ko9a59.png?id=a228111a05555964862529d7d401f599"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.48;"
srcset="/images/patterns/diagrams/singleton/structure-ko.png?id=a228111a05555964862529d7d401f599 430w,/images/patterns/diagrams/singleton/structure-ko-1.5x.png?id=4db1f7b969c4b4b6d001bfdee8fb839e 645w,/images/patterns/diagrams/singleton/structure-ko-2x.png?id=5072149009fd8706089fa9942ea6a69a 860w"
sizes="(max-width: 720px) 100vw, 430px" loading="lazy" width="430"
alt="싱글턴 패턴 구조" /><img
src="../../images/patterns/diagrams/singleton/structure-ko-indexed5026.png?id=b0740e0b6f79e0fe23320da552093ae7"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.48;"
srcset="/images/patterns/diagrams/singleton/structure-ko-indexed.png?id=b0740e0b6f79e0fe23320da552093ae7 430w,/images/patterns/diagrams/singleton/structure-ko-indexed-1.5x.png?id=1aa93465c0b281ca14d9bedd3b33da10 645w,/images/patterns/diagrams/singleton/structure-ko-indexed-2x.png?id=a1588dfe16e1f0f77f933af09d66a0a5 860w"
sizes="(max-width: 720px) 100vw, 430px" loading="lazy" width="430"
alt="싱글턴 패턴 구조" />
</figure>
:::

1.  **싱글턴** 클래스는 정적 메서드 `get­Instance`를 선언합니다. 이
    메서드는 자체 클래스의 같은 인스턴스를 반환합니다.

    싱글턴의 생성자는 항상 클라이언트 코드에서부터 숨겨져야 합니다.
    `get­Instance` 메서드를 호출하는 것이 Singleton 객체를 가져올 수 있는
    유일한 방법이어야 합니다.
::::
:::::

::: {.section .pseudocode}
##  의사코드 {#pseudocode}

이 예에서 데이터베이스의 연결 클래스는 **싱글턴**의 역할을 합니다. 이
클래스에는 공개된 생성자가 없으므로 해당 클래스의 객체를 가져오는 유일한
방법은 `get­Instance` 메서드를 호출하는 것입니다. 이 메서드는 처음 생성된
객체를 캐시 한 후 모든 후속 호출들에서 해당 객체를 반환합니다.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// 데이터베이스 클래스는 클라이언트들이 프로그램 전체에서 데이터베이스 연결의 같은
// 인스턴스에 접근할 수 있도록 해주는 `getInstance`(인스턴스 가져오기) 메서드를
// 정의합니다.
class Database is
    // 싱글턴 인스턴스를 저장하기 위한 필드는 정적으로 선언되어야 합니다.
    private static field instance: Database

    // 싱글턴의 생성자는 `new` 연산자를 사용한 직접 생성 호출들을 방지하기 위해
    // 항상 비공개여야 합니다.
    private constructor Database() is
        // 데이터베이스 서버에 대한 실제 연결과 같은 일부 초기화 코드.

    // 싱글턴 인스턴스로의 접근을 제어하는 정적 메서드.
    public static method getInstance() is
        if (Database.instance == null) then
            acquireThreadLock() and then
                // 이 스레드가 잠금 해제를 기다리는 동안 인스턴스가 다른
                // 스레드에 의해 초기화되지 않았는지 확인하세요.
                if (Database.instance == null) then
                    Database.instance = new Database()
        return Database.instance

    // 마지막으로 모든 싱글턴은 해당 로직의 인스턴스에서 실행할 수 있는 비즈니스
    // 로직을 정의해야 합니다.
    public method query(sql) is
        // 예를 들어 앱의 모든 데이터베이스 쿼리들은 이 메서드를 거칩니다. 따라서
        // 여기에 스로틀링 또는 캐싱 논리를 배치할 수 있습니다.
        // …

class Application is
    method main() is
        Database foo = Database.getInstance()
        foo.query(&quot;SELECT ...&quot;)
        // …
        Database bar = Database.getInstance()
        bar.query(&quot;SELECT ...&quot;)
        // 변수 `bar`는 변수 `foo`와 같은 객체를 포함할 것입니다.</code></pre>
</figure>
:::

:::::::: {.section .applicability-container}
##  적용 {#applicability}

::::::: applicability
::: applicability-problem
싱글턴 패턴은 당신 프로그램의 클래스에 모든 클라이언트가 사용할 수 있는
단일 인스턴스만 있어야 할 때 사용하세요. 예를 들자면 프로그램의 다른
부분들에서 공유되는 단일 데이터베이스 객체처럼 말입니다.
:::

::: applicability-solution
싱글턴 패턴은 특별 생성 메서드를 제외하고는 클래스의 객체들을 생성할 수
있는 모든 다른 수단들을 비활성화합니다. 이 메서드는 새 객체를 생성하거나
객체가 이미 생성되었으면 기존 객체를 반환합니다.
:::

::: applicability-problem
싱글턴 패턴은 전역 변수들을 더 엄격하게 제어해야 할 때 사용하세요.
:::

::: applicability-solution
전역 변수들과 달리 싱글턴 패턴은 클래스의 인스턴스가 하나만 있도록
보장해 줍니다. 캐시 된 인스턴스는 싱글턴 클래스 자체를 제외하고는 그
어떤 것과도 대체될 수 없습니다.

참고로 이 제한은 언제든 조정할 수 있고 원하는 수만큼의 싱글턴 인스턴스
생성을 허용할 수 있습니다. 그러기 위해서 변경해야 하는 코드의 유일한
부분은 `get­Instance` 메서드의 본문입니다.
:::
:::::::
::::::::

::: {.section .checklist}
##  구현방법 {#checklist}

1.  싱글턴 인스턴스의 저장을 위해 클래스에 비공개 정적 필드를
    추가하세요.

2.  싱글턴 인스턴스를 가져오기 위한 공개된 정적 생성 메서드를
    선언하세요.

3.  정적 메서드 내에서 \'지연된 초기화\'를 구현하세요. 그러면 이것은 첫
    번째 호출에서 새 객체를 만든 후 그 객체를 정적 필드에 넣을 것입니다.
    이 메서드는 모든 후속 호출들에서 항상 해당 인스턴스를 반환해야
    합니다.

4.  클래스의 생성자를 비공개로 만드세요. 그러면 클래스의 정적 메서드는
    여전히 생성자를 호출할 수 있지만 다른 객체들은 호출할 수 없을
    것입니다.

5.  클라이언트 코드를 살펴보며 싱글턴의 생성자에 대한 모든 직접 호출들을
    싱글턴의 정적 생성 메서드에 대한 호출로 바꾸세요.
:::

:::::: {.section .pros-cons}
##  장단점 {#pros-cons}

::::: row
::: col-sm-6
-    클래스가 하나의 인스턴트만 갖는다는 것을 확신할 수 있습니다.
-    이 인스턴스에 대한 전역 접근 지점을 얻습니다.
-    싱글턴 객체는 처음 요청될 때만 초기화됩니다.
:::

::: col-sm-6
-    *[단]{.cjk}[일]{.cjk} [책]{.cjk}[임]{.cjk} [원]{.cjk}[칙]{.cjk}*을
    위반합니다. 이 패턴은 한 번에 두 가지의 문제를 동시에 해결합니다.
-    또 싱글턴 패턴은 잘못된 디자인​(예를 들어 프로그램의 컴포넌트들이
    서로에 대해 너무 많이 알고 있는 경우)​을 가릴 수 있습니다.
-    그리고 이 패턴은 다중 스레드 환경에서 여러 스레드가 싱글턴 객체를
    여러 번 생성하지 않도록 특별한 처리가 필요합니다.
-    싱글턴의 클라이언트 코드를 유닛 테스트하기 어려울 수 있습니다. 그
    이유는 많은 테스트 프레임워크들이 모의 객체들을 생성할 때 상속에
    의존하기 때문입니다. 싱글턴 클래스의 생성자는 비공개이고 대부분
    언어에서 정적 메서드를 오버라이딩하는 것이 불가능하므로 싱글턴의
    한계를 극복할 수 있는 창의적인 방법을 생각해야 합니다. 아니면 그냥
    테스트를 작성하지 말거나 싱글턴 패턴을 사용하지 않으면 됩니다.
:::
:::::
::::::

::: {.section .relations}
##  다른 패턴과의 관계 {#relations}

-   대부분의 경우 하나의 퍼사드 객체만 있어도 충분하므로
    [퍼사드](facade.html) 패턴의 클래스는 종종
    [싱글턴](singleton.html)으로 변환될 수 있습니다.

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

-   [추상 팩토리들](abstract-factory.html), [빌더들](builder.html) 및
    [프로토타입들](prototype.html)은 모두 [싱글턴](singleton.html)으로
    구현할 수 있습니다.
:::

::: {.section .implementations}
##  코드 예시 {#implementations}

[![C#으로 작성된
싱글턴](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](singleton/csharp/example.html "C#으로 작성된 싱글턴"){.prog-lang-link}
[![C++로 작성된
싱글턴](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](singleton/cpp/example.html "C++로 작성된 싱글턴"){.prog-lang-link}
[![Go로 작성된
싱글턴](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](singleton/go/example.html "Go로 작성된 싱글턴"){.prog-lang-link}
[![자바로 작성된
싱글턴](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](singleton/java/example.html "자바로 작성된 싱글턴"){.prog-lang-link}
[![PHP로 작성된
싱글턴](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](singleton/php/example.html "PHP로 작성된 싱글턴"){.prog-lang-link}
[![파이썬으로 작성된
싱글턴](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](singleton/python/example.html "파이썬으로 작성된 싱글턴"){.prog-lang-link}
[![루비로 작성된
싱글턴](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](singleton/ruby/example.html "루비로 작성된 싱글턴"){.prog-lang-link}
[![러스트로 작성된
싱글턴](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](singleton/rust/example.html "러스트로 작성된 싱글턴"){.prog-lang-link}
[![스위프트로 작성된
싱글턴](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](singleton/swift/example.html "스위프트로 작성된 싱글턴"){.prog-lang-link}
[![타입스크립트로 작성된
싱글턴](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](singleton/typescript/example.html "타입스크립트로 작성된 싱글턴"){.prog-lang-link}
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

[구조 패턴 []{.fa .fa-arrow-right}](structural-patterns.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### 돌아가기

[[]{.fa .fa-arrow-left} 프로토타입](prototype.html){.btn .btn-default
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
