::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [디자인
패턴들](../design-patterns.html){.type} / [구조
패턴](structural-patterns.html){.type}
:::

# 퍼사드 패턴 {#퍼사드-패턴 .title}

::: aka
다음 이름으로도 불립니다: [Facade]{style="display:inline-block"}
:::

::: {.section .intent}
##  의도 {#intent}

**퍼사드** 패턴은 라이브러리에 대한, 프레임워크에 대한 또는 다른
클래스들의 복잡한 집합에 대한 단순화된 인터페이스를 제공하는 구조적
디자인 패턴입니다.

<figure class="image">
<img
src="../../images/patterns/content/facade/facade721d.png?id=1f4be17305b6316fbd548edf1937ac3b"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/facade/facade.png?id=1f4be17305b6316fbd548edf1937ac3b 640w,/images/patterns/content/facade/facade-1.5x.png?id=a6af5003b243b59990842d24bdaae2df 960w,/images/patterns/content/facade/facade-2x.png?id=b69fce5943703f5f07c0ba38e3baaed0 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="퍼사드 디자인 패턴" />
</figure>
:::

::: {.section .problem}
##  문제 {#problem}

정교한 라이브러리나 프레임워크에 속하는 광범위한 객체들의 집합으로
당신의 코드를 작동하게 만들어야 한다고 상상해 봅시다. 일반적으로, 당신은
이러한 객체들을 모두 초기화하고, 종속성 관계들을 추적하고, 올바른 순서로
메서드들을 실행하는 등의 작업을 수행해야 합니다.

그 결과 당신의 클래스들의 비즈니스 로직이 타사 클래스들의 구현 세부
사항들과 밀접하게 결합하여 코드를 이해하고 유지 관리하기가 어려워집니다.
:::

::: {.section .solution}
##  해결책 {#solution}

퍼사드는 움직이는 부분이 많이 포함된 복잡한 하위 시스템에 대한 간단한
인터페이스를 제공하는 클래스입니다. 하위 시스템과 직접 작업하는 것과
비교하면 퍼사드는 제한된 기능성을 제공합니다. 하지만 퍼사드에는
클라이언트들이 정말로 중요하게 생각하는 기능들만 포함됩니다.

퍼사드는 당신의 앱을 수십 가지의 기능이 있는 정교한 라이브러리와
통합해야 하지만 그 기능의 극히 일부만을 필요로 할 때 편리합니다.

예를 들어, 고양이가 나오는 짤막하고 재미있는 영상을 소셜 미디어에 올리는
어떤 앱은 전문적인 비디오 변환 라이브러리를 사용할 수 있습니다. 그러나
이 앱이 이 라이브러리로부터 필요로 하는 것은 `encode­(filename, format)`
단일 메서드가 있는 클래스뿐입니다. 이러한 클래스를 만든 후 비디오 변환
라이브러리와 연결하면 당신은 당신의 첫 번째 퍼사드를 소유하게 됩니다.
:::

::: {.section .analogy}
##  실제상황 적용 {#analogy}

<figure class="image">
<img
src="../../images/patterns/diagrams/facade/live-example-ko4878.png?id=f4a50f8659a7b17365f90e95fc365096"
style="aspect-ratio: 2.58;"
srcset="/images/patterns/diagrams/facade/live-example-ko.png?id=f4a50f8659a7b17365f90e95fc365096 490w,/images/patterns/diagrams/facade/live-example-ko-1.5x.png?id=1394739d506380a1b6e174f78251c398 735w,/images/patterns/diagrams/facade/live-example-ko-2x.png?id=ac877ad4312d6d568a88c92e8c5536fa 980w"
sizes="(max-width: 720px) 100vw, 490px" loading="lazy" width="490"
alt="전화 주문 예시" />
<figcaption><p>전화로 주문하기.</p></figcaption>
</figure>

전화로 주문하기 위해 매장에 전화를 걸었을 때 전화를 받는 교환원이 바로
상점의 모든 서비스와 부서에 대한 당신의 퍼사드입니다. 이때 교환원은 주문
시스템, 지불 게이트웨이 및 다양한 배송 서비스에 대한 간단한 음성
인터페이스를 제공합니다.
:::

::::: {.section .structure-container}
##  구조 {#structure}

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/facade/structure2883.png?id=258401362234ac77a2aaf1cde62339e7"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.47;"
srcset="/images/patterns/diagrams/facade/structure.png?id=258401362234ac77a2aaf1cde62339e7 560w,/images/patterns/diagrams/facade/structure-1.5x.png?id=98d134de0c368d382909ba162ec21767 840w,/images/patterns/diagrams/facade/structure-2x.png?id=528ca429555bce293b7c3bd90954e097 1120w"
sizes="(max-width: 720px) 100vw, 560px" loading="lazy" width="560"
alt="퍼사드 디자인 패턴 구조" /><img
src="../../images/patterns/diagrams/facade/structure-indexeddb1c.png?id=2da06d6b850701ea15cf72f9d2642fb8"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.58;"
srcset="/images/patterns/diagrams/facade/structure-indexed.png?id=2da06d6b850701ea15cf72f9d2642fb8 600w,/images/patterns/diagrams/facade/structure-indexed-1.5x.png?id=fad7d667f4d8d9a7d522748bcd37dfde 900w,/images/patterns/diagrams/facade/structure-indexed-2x.png?id=4d181bcf1df5d58c533e6c92461e9571 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="퍼사드 디자인 패턴 구조" />
</figure>
:::

1.  **퍼사드** 패턴을 사용하면 하위 시스템 기능들의 특정 부분에 편리하게
    접근할 수 있습니다. 또 퍼사드는 클라이언트의 요청을 어디로 보내야
    하는지와 움직이는 모든 부품을 어떻게 작동해야 하는지를 알고
    있습니다.

2.  **추가적인 퍼사드** 클래스를 생성하여 하나의 퍼사드를 관련 없는
    기능들로 오염시켜 복잡한 구조로 만드는 것을 방지할 수 있습니다. 추가
    퍼사드들은 클라이언트들과 다른 퍼사드들 모두에 사용할 수 있습니다.

3.  **복잡한 하위 시스템**은 수십 개의 다양한 객체들로 구성됩니다. 이
    모든 객체가 의미 있는 작업을 수행하도록 하려면, 하위 시스템의
    세부적인 구현 정보를 깊이 있게 살펴야 합니다. 예를 들어 올바른
    순서로 객체들을 초기화하고 그들에게 적절한 형식의 데이터를 제공하는
    등의 작업을 수행해야 합니다.

    하위 시스템 클래스들은 퍼사드의 존재를 인식하지 못합니다. 이들은
    시스템 내에서 작동하며, 매개체 없이 직접 서로와 작업합니다.

4.  **클라이언트**는 하위 시스템 객체들을 직접 호출하는 대신 퍼사드를
    사용합니다.
::::
:::::

::: {.section .pseudocode}
##  의사코드 {#pseudocode}

이 예시에서 **퍼사드** 패턴은 복잡한 비디오 변환 프레임워크와의
상호작용을 단순화합니다.

<figure class="image">
<img
src="../../images/patterns/diagrams/facade/example16bc.png?id=2249d134e3ff83819dfc19032f02eced"
style="aspect-ratio: 1.5;"
srcset="/images/patterns/diagrams/facade/example.png?id=2249d134e3ff83819dfc19032f02eced 570w,/images/patterns/diagrams/facade/example-1.5x.png?id=034ecbe0f3cc108f4ae49d05d1c77dbe 855w,/images/patterns/diagrams/facade/example-2x.png?id=f2c846d74041626c923ff3e8919b68a9 1140w"
sizes="(max-width: 720px) 100vw, 570px" loading="lazy" width="570"
alt="퍼사드 패턴 구조 예시" />
<figcaption><p>단일 퍼사드 클래스 내에서 여러 종속 관계들을
격리하는 예시.</p></figcaption>
</figure>

당신의 코드가 수십 개의 프레임워크 클래스들과 직접 작업하도록 하는 대신,
해당 기능들을 캡슐화하여 코드의 나머지 부분으로부터 숨기는 퍼사드
클래스를 만듭니다. 이러한 구조를 이용하면 나중에 프레임워크를
업그레이드하거나 다른 프레임워크로 교체할 때 들어갈 노력을 최소한으로
줄일 수 있습니다. 앱에서 바꾸어야 할 부분이 퍼사드의 메서드들의 구현뿐일
것이기 때문입니다.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// 이것들은 복잡한 타사 비디오 변환 프레임워크 클래스의 일부입니다. 해당 프레임워크
// 코드는 우리가 제어할 수 없기 때문에 단순화할 수 없습니다.

class VideoFile
// …

class OggCompressionCodec
// …

class MPEG4CompressionCodec
// …

class CodecFactory
// …

class BitrateReader
// …

class AudioMixer
// …


// 퍼사드 클래스를 만들어 프레임워크의 복잡성을 간단한 인터페이스 뒤에 숨길 수
// 있습니다. 기능성과 단순함을 상호보완하는 것이죠.
class VideoConverter is
    method convert(filename, format):File is
        file = new VideoFile(filename)
        sourceCodec = (new CodecFactory).extract(file)
        if (format == &quot;mp4&quot;)
            destinationCodec = new MPEG4CompressionCodec()
        else
            destinationCodec = new OggCompressionCodec()
        buffer = BitrateReader.read(filename, sourceCodec)
        result = BitrateReader.convert(buffer, destinationCodec)
        result = (new AudioMixer()).fix(result)
        return new File(result)

// 애플리케이션 클래스들은 복잡한 프레임워크에서 제공하는 수많은 클래스에 의존하지
// 않습니다. 또한 프레임워크의 전환을 결정한 경우에는 퍼사드 클래스만 다시 작성하면
// 됩니다.
class Application is
    method main() is
        convertor = new VideoConverter()
        mp4 = convertor.convert(&quot;funny-cats-video.ogg&quot;, &quot;mp4&quot;)
        mp4.save()</code></pre>
</figure>
:::

:::::::: {.section .applicability-container}
##  적용 {#applicability}

::::::: applicability
::: applicability-problem
퍼사드 패턴은 당신이 복잡한 하위 시스템에 대한 제한적이지만 간단한
인터페이스가 필요할 때 사용하세요.
:::

::: applicability-solution
하위 시스템은 시간이 지날수록 더 복잡해지곤 합니다. 디자인 패턴들을
적용하더라도 보통은 생성되는 클래스들이 점점 더 많아지게 됩니다. 하위
시스템은 더 유연해지고 더 많은 다양한 상황에서 재사용할 수 있도록 변경될
수도 있지만, 해당 시스템이 클라이언트에게 요구하는 설정 및 상용구 코드의
양은 점점 더 많아질 것입니다. 이 문제를 해결하기 위해 퍼사드는 대부분의
클라이언트 요건에 부합하면서 하위 시스템에서 가장 많이 사용되는 기능들로
향하는 지름길을 제공합니다.
:::

::: applicability-problem
퍼사드 패턴은 하위 시스템을 계층들로 구성하려는 경우 사용하세요.
:::

::: applicability-solution
하위시스템의 각 계층에 대한 진입점을 정의하기 위해 퍼사드 패턴들을
생성하세요. 당신은 여러 하위시스템이 퍼사드 패턴들을 통해서만 통신하도록
함으로써 해당 하위시스템 간의 결합도를 줄일 수 있습니다.

예를 들어 당신의 비디오 변환 프레임워크는 비디오 또는 오디오 관련의 두
가지 계층으로 나뉠 수 있습니다. 각 계층에 대해 퍼사드를 만든 다음 각
계층의 클래스들이 해당 퍼사드들을 통해 서로 통신하도록 할 수 있습니다.
이러한 접근 방식은 [중재자](mediator.html) 패턴과 매우 유사합니다.
:::
:::::::
::::::::

::: {.section .checklist}
##  구현방법 {#checklist}

1.  기존 하위시스템이 이미 제공하고 있는 것보다 더 간단한 인터페이스를
    제공하는 것이 가능한지 확인하세요. 이 인터페이스가 클라이언트 코드를
    하위시스템의 여러 클래스로부터 독립시킨다면 제대로 하고 계신 겁니다.

2.  새 퍼사드 패턴 클래스에서 이 인터페이스를 선언하고 구현하세요. 이
    퍼사드는 클라이언트 코드의 호출들을 하위 시스템의 적절한 객체들로
    리다이렉션해야 합니다. 이 퍼사드는 클라이언트가 아래 작업을 이미
    수행하지 않는 한 하위시스템을 초기화하고 추가 수명 주기를 관리하는
    작업의 책임을 맡아야 합니다.

3.  패턴을 최대한 활용하려면 모든 클라이언트 코드가 퍼사드 패턴을
    통해서만 하위시스템과 통신하도록 하세요. 그러면 이제 클라이언트
    코드는 하위시스템 코드의 변경 사항들로부터 보호됩니다. 예를 들어,
    하위시스템이 새 버전으로 업그레이드되면 당신은 퍼사드의 코드만
    수정하면 됩니다.

4.  퍼사드가 [너무 커지면](../smells/large-class.html) 행동들 일부를
    새롭고 정제된 퍼사드 클래스로 추출하는 것을 고려하세요.
:::

:::::: {.section .pros-cons}
##  장단점 {#pros-cons}

::::: row
::: col-sm-6
-    복잡한 하위 시스템에서 코드를 별도로 분리할 수 있습니다.
:::

::: col-sm-6
-    퍼사드는 앱의 모든 클래스에 결합된 [전지전능한
    객체](https://en.wikipedia.org/wiki/God_object)가 될 수 있습니다.
:::
:::::
::::::

::: {.section .relations}
##  다른 패턴과의 관계 {#relations}

-   [퍼사드](facade.html)는 기존 객체들을 위한 새 인터페이스를 정의하는
    반면 [어댑터](adapter.html)는 기존의 인터페이스를 사용할 수 있게
    만들려고 노력합니다. 또 *[어]{.cjk}[댑]{.cjk}[터]{.cjk}*는
    일반적으로 하나의 객체만 래핑하는 반면
    *[퍼]{.cjk}[사]{.cjk}[드]{.cjk}*는 많은 객체의 하위시스템과 함께
    작동합니다.

-   [추상 팩토리](abstract-factory.html)는 하위시스템 객체들이
    클라이언트 코드에서 생성되는 방식만 숨기고 싶을 때
    [퍼사드](facade.html) 대신 사용할 수 있습니다.

-   [플라이웨이트](flyweight.html)는 작은 객체들을 많이 만드는 방법을
    보여 주는 반면 [퍼사드](facade.html) 패턴은 전체 하위 시스템을
    나타내는 단일 객체를 만드는 방법을 보여 줍니다.

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

-   대부분의 경우 하나의 퍼사드 객체만 있어도 충분하므로
    [퍼사드](facade.html) 패턴의 클래스는 종종
    [싱글턴](singleton.html)으로 변환될 수 있습니다.

-   [퍼사드](facade.html) 패턴은 복잡한 객체 또는 시스템을 보호하고
    자체적으로 초기화한다는 점에서 [프록시](proxy.html)와 유사합니다.
    *[퍼]{.cjk}[사]{.cjk}[드]{.cjk}* 패턴과 달리
    *[프]{.cjk}[록]{.cjk}[시]{.cjk}*는 자신의 서비스 객체와 같은
    인터페이스를 가지므로 이들은 서로 상호 교환이 가능합니다.
:::

::: {.section .implementations}
##  코드 예시 {#implementations}

[![C#으로 작성된
퍼사드](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](facade/csharp/example.html "C#으로 작성된 퍼사드"){.prog-lang-link}
[![C++로 작성된
퍼사드](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](facade/cpp/example.html "C++로 작성된 퍼사드"){.prog-lang-link}
[![Go로 작성된
퍼사드](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](facade/go/example.html "Go로 작성된 퍼사드"){.prog-lang-link}
[![자바로 작성된
퍼사드](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](facade/java/example.html "자바로 작성된 퍼사드"){.prog-lang-link}
[![PHP로 작성된
퍼사드](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](facade/php/example.html "PHP로 작성된 퍼사드"){.prog-lang-link}
[![파이썬으로 작성된
퍼사드](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](facade/python/example.html "파이썬으로 작성된 퍼사드"){.prog-lang-link}
[![루비로 작성된
퍼사드](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](facade/ruby/example.html "루비로 작성된 퍼사드"){.prog-lang-link}
[![러스트로 작성된
퍼사드](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](facade/rust/example.html "러스트로 작성된 퍼사드"){.prog-lang-link}
[![스위프트로 작성된
퍼사드](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](facade/swift/example.html "스위프트로 작성된 퍼사드"){.prog-lang-link}
[![타입스크립트로 작성된
퍼사드](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](facade/typescript/example.html "타입스크립트로 작성된 퍼사드"){.prog-lang-link}
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

[플라이웨이트 []{.fa .fa-arrow-right}](flyweight.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### 돌아가기

[[]{.fa .fa-arrow-left} 데코레이터](decorator.html){.btn .btn-default
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
