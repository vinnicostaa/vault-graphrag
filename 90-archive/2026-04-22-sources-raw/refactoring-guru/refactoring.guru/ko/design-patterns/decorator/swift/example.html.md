:::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [디자인
패턴들](../../../design-patterns.html){.type} /
[데코레이터](../../decorator.html){.type} /
[스위프트](../../swift.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![데코레이터](../../../../images/patterns/cards/decorator-mini6adb.png?id=d30458908e315af195cb183bc52dbef9){srcset="/images/patterns/cards/decorator-mini-2x.png?id=3b58e540d7d28523080cad341ed9b2e9 2x"}
:::

# 스위프트로 작성된 **데코레이터** {#스위프트로-작성된-데코레이터 .pattern-example-title-block-title}
::::

:::::::::::::: pattern-example-body
::: pattern-example-brief
**데코레이터**는 구조 패턴이며 새로운 행동들을 특수 래퍼 객체들 내에
넣어서 이러한 행동들을 객체들에 동적으로 추가할 수 있도록 합니다.

데코레이터를 사용하여 객체들을 제한 없이 래핑할 수 있습니다. 왜냐하면
대상 객체들과 데코레이터들은 같은 인터페이스를 따르기 때문입니다. 결과
객체는 모든 래퍼의 스태킹된 행동을 가질 것입니다.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ 데코레이터에 대하여 더 자세히 알아보세요 ](../../decorator.html){.btn
.btn-lg .btn-primary}
:::

::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
내비게이션
:::

::: en-intro
 [소개](#)
:::

::: en-intro
 [개념적인 예시](#example-0)
:::

::: en-file
 [Example](#example-0--Example-swift)
:::

::: en-file
 [Output](#example-0--Output-txt)
:::

::: en-intro
 [실제 사례 예시](#example-1)
:::

::: en-file
 [Example](#example-1--Example-swift)
:::

::: en-file
 [Output](#example-1--Output-txt)
:::
:::::::::::

**복잡도:** []{.fa-stars}

**인기도:** []{.fa-stars}

**사용 예시들:** 데코레이터는 스위프트 코드, 특히 스트림과 관련된
코드에서 꽤 표준적입니다.

**식별:** 데코레이터는 같은 클래스의 객체 또는 인터페이스를 현재
클래스로 수락하는 생성 메서드들 또는 생성자들로 인식할 수 있습니다.
::::::::::::::

::: {#nav-tab .nav .nav-tabs .nav-justified role="tablist"}
[개념적인 예시](#example-0){#example-0-tab .nav-item .nav-link .active
toggle="tab" role="tab" aria-controls="example-0" aria-selected="true"}
[실제 사례 예시](#example-1){#example-1-tab .nav-item .nav-link
toggle="tab" role="tab" aria-controls="example-1" aria-selected="true"}
:::

::::: {#nav-tabContent .pattern-example-tab-content .tab-content}
::: {#example-0 .tab-pane .show .active role="tabpanel" aria-labelledby="example-0-tab" style="padding-top: 24px"}
## 개념적인 예시 {#example-0-title}

이 예시는 **데코레이터** 패턴의 구조를 보여주고 다음 질문에 중점을
둡니다:

-   패턴은 어떤 클래스들로 구성되어 있나요?
-   이 클래스들은 어떤 역할을 하나요?
-   패턴의 요소들은 어떻게 서로 연관되어 있나요?

이 패턴의 구조를 배우면 실제 스위프트 사용 사례를 기반으로 하는 다음
예시를 더욱 쉽게 이해할 수 있을 것입니다.

#### []{#example-0--Example-swift .anchor} **Example.swift:** 개념적인 예시

<figure class="code">
<pre class="code" lang="swift"><code>import XCTest

/// The base Component interface defines operations that can be altered by
/// decorators.
protocol Component {

    func operation() -&gt; String
}

/// Concrete Components provide default implementations of the operations. There
/// might be several variations of these classes.
class ConcreteComponent: Component {

    func operation() -&gt; String {
        return &quot;ConcreteComponent&quot;
    }
}

/// The base Decorator class follows the same interface as the other components.
/// The primary purpose of this class is to define the wrapping interface for
/// all concrete decorators. The default implementation of the wrapping code
/// might include a field for storing a wrapped component and the means to
/// initialize it.
class Decorator: Component {

    private var component: Component

    init(_ component: Component) {
        self.component = component
    }

    /// The Decorator delegates all work to the wrapped component.
    func operation() -&gt; String {
        return component.operation()
    }
}

/// Concrete Decorators call the wrapped object and alter its result in some
/// way.
class ConcreteDecoratorA: Decorator {

    /// Decorators may call parent implementation of the operation, instead of
    /// calling the wrapped object directly. This approach simplifies extension
    /// of decorator classes.
    override func operation() -&gt; String {
        return &quot;ConcreteDecoratorA(&quot; + super.operation() + &quot;)&quot;
    }
}

/// Decorators can execute their behavior either before or after the call to a
/// wrapped object.
class ConcreteDecoratorB: Decorator {

    override func operation() -&gt; String {
        return &quot;ConcreteDecoratorB(&quot; + super.operation() + &quot;)&quot;
    }
}

/// The client code works with all objects using the Component interface. This
/// way it can stay independent of the concrete classes of components it works
/// with.
class Client {
    // ...
    static func someClientCode(component: Component) {
        print(&quot;Result: &quot; + component.operation())
    }
    // ...
}

/// Let&#39;s see how it all works together.
class DecoratorConceptual: XCTestCase {

    func testDecoratorConceptual() {
        // This way the client code can support both simple components...
        print(&quot;Client: I&#39;ve got a simple component&quot;)
        let simple = ConcreteComponent()
        Client.someClientCode(component: simple)

        // ...as well as decorated ones.
        //
        // Note how decorators can wrap not only simple components but the other
        // decorators as well.
        let decorator1 = ConcreteDecoratorA(simple)
        let decorator2 = ConcreteDecoratorB(decorator1)
        print(&quot;\nClient: Now I&#39;ve got a decorated component&quot;)
        Client.someClientCode(component: decorator2)
    }
}</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** 실행 결과

<figure class="code">
<pre class="code" lang="output"><code>Client: I&#39;ve got a simple component
Result: ConcreteComponent

Client: Now I&#39;ve got a decorated component
Result: ConcreteDecoratorB(ConcreteDecoratorA(ConcreteComponent))</code></pre>
</figure>
:::

::: {#example-1 .tab-pane role="tabpanel" aria-labelledby="example-1-tab" style="padding-top: 24px"}
## 실제 사례 예시 {#example-1-title}

#### []{#example-1--Example-swift .anchor} **Example.swift:** 실제 사례 예시

<figure class="code">
<pre class="code" lang="swift"><code>import UIKit
import XCTest


protocol ImageEditor: CustomStringConvertible {

    func apply() -&gt; UIImage
}

class ImageDecorator: ImageEditor {

    private var editor: ImageEditor

    required init(_ editor: ImageEditor) {
        self.editor = editor
    }

    func apply() -&gt; UIImage {
        print(editor.description + &quot; applies changes&quot;)
        return editor.apply()
    }

    var description: String {
        return &quot;ImageDecorator&quot;
    }
}

extension UIImage: ImageEditor {

    func apply() -&gt; UIImage {
        return self
    }

    open override var description: String {
        return &quot;Image&quot;
    }
}



class BaseFilter: ImageDecorator {

    fileprivate var filter: CIFilter?

    init(editor: ImageEditor, filterName: String) {
        self.filter = CIFilter(name: filterName)
        super.init(editor)
    }

    required init(_ editor: ImageEditor) {
        super.init(editor)
    }

    override func apply() -&gt; UIImage {

        let image = super.apply()
        let context = CIContext(options: nil)

        filter?.setValue(CIImage(image: image), forKey: kCIInputImageKey)

        guard let output = filter?.outputImage else { return image }
        guard let coreImage = context.createCGImage(output, from: output.extent) else {
            return image
        }
        return UIImage(cgImage: coreImage)
    }

    override var description: String {
        return &quot;BaseFilter&quot;
    }
}

class BlurFilter: BaseFilter {

    required init(_ editor: ImageEditor) {
        super.init(editor: editor, filterName: &quot;CIGaussianBlur&quot;)
    }

    func update(radius: Double) {
        filter?.setValue(radius, forKey: &quot;inputRadius&quot;)
    }

    override var description: String {
        return &quot;BlurFilter&quot;
    }
}

class ColorFilter: BaseFilter {

    required init(_ editor: ImageEditor) {
        super.init(editor: editor, filterName: &quot;CIColorControls&quot;)
    }

    func update(saturation: Double) {
        filter?.setValue(saturation, forKey: &quot;inputSaturation&quot;)
    }

    func update(brightness: Double) {
        filter?.setValue(brightness, forKey: &quot;inputBrightness&quot;)
    }

    func update(contrast: Double) {
        filter?.setValue(contrast, forKey: &quot;inputContrast&quot;)
    }

    override var description: String {
        return &quot;ColorFilter&quot;
    }
}

class Resizer: ImageDecorator {

    private var xScale: CGFloat = 0
    private var yScale: CGFloat = 0
    private var hasAlpha = false

    convenience init(_ editor: ImageEditor, xScale: CGFloat = 0, yScale: CGFloat = 0, hasAlpha: Bool = false) {
        self.init(editor)
        self.xScale = xScale
        self.yScale = yScale
        self.hasAlpha = hasAlpha
    }

    required init(_ editor: ImageEditor) {
        super.init(editor)
    }

    override func apply() -&gt; UIImage {

        let image = super.apply()

        let size = image.size.applying(CGAffineTransform(scaleX: xScale, y: yScale))

        UIGraphicsBeginImageContextWithOptions(size, !hasAlpha, UIScreen.main.scale)
        image.draw(in: CGRect(origin: .zero, size: size))

        let scaledImage = UIGraphicsGetImageFromCurrentImageContext()
        UIGraphicsEndImageContext()

        return scaledImage ?? image
    }

    override var description: String {
        return &quot;Resizer&quot;
    }
}


class DecoratorRealWorld: XCTestCase {

    func testDecoratorRealWorld() {

        let image = loadImage()

        print(&quot;Client: set up an editors stack&quot;)
        let resizer = Resizer(image, xScale: 0.2, yScale: 0.2)

        let blurFilter = BlurFilter(resizer)
        blurFilter.update(radius: 2)

        let colorFilter = ColorFilter(blurFilter)
        colorFilter.update(contrast: 0.53)
        colorFilter.update(brightness: 0.12)
        colorFilter.update(saturation: 4)

        clientCode(editor: colorFilter)
    }

    func clientCode(editor: ImageEditor) {
        let image = editor.apply()
        /// Note. You can stop an execution in Xcode to see an image preview.
        print(&quot;Client: all changes have been applied for \(image)&quot;)
    }
}

private extension DecoratorRealWorld {

    func loadImage() -&gt; UIImage {

        let urlString = &quot;https:// refactoring.guru/images/content-public/logos/logo-new-3x.png&quot;

        /// Note:
        /// Do not download images the following way in a production code.

        guard let url = URL(string: urlString) else {
            fatalError(&quot;Please enter a valid URL&quot;)
        }

        guard let data = try? Data(contentsOf: url) else {
            fatalError(&quot;Cannot load an image&quot;)
        }

        guard let image = UIImage(data: data) else {
            fatalError(&quot;Cannot create an image from data&quot;)
        }
        return image
    }
}</code></pre>
</figure>

#### []{#example-1--Output-txt .anchor} **Output.txt:** 실행 결과

<figure class="code">
<pre class="code" lang="output"><code>Client: set up an editors stack

BlurFilter applies changes
Resizer applies changes
Image applies changes

Client: all changes have been applied for Image</code></pre>
</figure>
:::
:::::

::: {#nav-tab .nav .nav-tabs .nav-tabs-bottom .nav-justified role="tablist"}
[개념적인 예시](#example-0){#example-0-tab-bottom .nav-item .nav-link
.active toggle="tab" role="tab" aria-controls="example-0"
aria-selected="true"} [실제 사례 예시](#example-1){#example-1-tab-bottom
.nav-item .nav-link toggle="tab" role="tab" aria-controls="example-1"
aria-selected="true"}
:::

::: next
#### 다음을 보세요

[스위프트로 작성된 퍼사드 []{.fa
.fa-arrow-right}](../../facade/swift/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### 돌아가기

[[]{.fa .fa-arrow-left} 스위프트로 작성된
복합체](../../composite/swift/example.html){.btn .btn-default
rel="prev"}
:::

## 다른 언어로 작성된 **데코레이터**

[![C#으로 작성된
데코레이터](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "C#으로 작성된 데코레이터"){.prog-lang-link}
[![C++로 작성된
데코레이터](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "C++로 작성된 데코레이터"){.prog-lang-link}
[![Go로 작성된
데코레이터](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Go로 작성된 데코레이터"){.prog-lang-link}
[![자바로 작성된
데코레이터](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "자바로 작성된 데코레이터"){.prog-lang-link}
[![PHP로 작성된
데코레이터](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "PHP로 작성된 데코레이터"){.prog-lang-link}
[![파이썬으로 작성된
데코레이터](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "파이썬으로 작성된 데코레이터"){.prog-lang-link}
[![루비로 작성된
데코레이터](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "루비로 작성된 데코레이터"){.prog-lang-link}
[![러스트로 작성된
데코레이터](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "러스트로 작성된 데코레이터"){.prog-lang-link}
[![타입스크립트로 작성된
데코레이터](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "타입스크립트로 작성된 데코레이터"){.prog-lang-link}
:::::::::::::::::::::::::

::::::: {.prom .banner-sidebar .banner .banner-striped .banner-examples .banner-removable .banner-removable-patterns data-id="DP: Examples-IDE" creative-id="examples-sidebar-ko" position="sidebar"}
:::::: {.banner-inner style="font-size: 14px; line-height: 21px;"}
::: banner-examples-img
[![](../../../../images/patterns/banners/examples-ide91ca.png?id=3115b4b548fb96b75974e2de8f4f49bc){width="215"
height="158" loading="lazy"
srcset="/images/patterns/banners/examples-ide-2x.png?id=93c007a6d157b97c28eb213f59d716ae 2x"}](../../book.html)
:::

::: banner-examples-fade
[ 예시들을 포함한 저장소](../../book.html){.btn .btn-secondary
.btn-block}
:::

::: {.banner-examples-text style="margin-top: 1rem;"}
전자책 **디자인 패턴에 뛰어들기**를 구매하면 IDE에서 바로 열 수 있는
수십 가지의 상세한 코드 예시들이 포함된 저장소를 사용할 수 있습니다.

[ 더 알아보기...](../../book.html){.btn .btn-secondary .btn-block}
:::
::::::
:::::::
:::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::
