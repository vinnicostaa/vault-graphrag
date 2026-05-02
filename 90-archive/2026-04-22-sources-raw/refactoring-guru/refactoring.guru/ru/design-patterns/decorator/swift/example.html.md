::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Паттерны
проектирования](../../../design-patterns.html){.type} /
[Декоратор](../../decorator.html){.type} /
[Swift](../../swift.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Декоратор](../../../../images/patterns/cards/decorator-mini6adb.png?id=d30458908e315af195cb183bc52dbef9){srcset="/images/patterns/cards/decorator-mini-2x.png?id=3b58e540d7d28523080cad341ed9b2e9 2x"}
:::

# **Декоратор** на Swift {#декоратор-на-swift .pattern-example-title-block-title}
::::

:::::::::::::: pattern-example-body
::: pattern-example-brief
**Декоратор** --- это структурный паттерн, который позволяет добавлять
объектам новые поведения на лету, помещая их в объекты-обёртки.

Декоратор позволяет оборачивать объекты бесчисленное количество раз
благодаря тому, что и обёртки, и реальные оборачиваемые объекты имеют
общий интерфейс.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Подробней о паттерне Декоратор ](../../decorator.html){.btn .btn-lg
.btn-primary}
:::

::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
Навигация
:::

::: en-intro
 [Интро](#)
:::

::: en-intro
 [Концептуальный пример](#example-0)
:::

::: en-file
 [Example](#example-0--Example-swift)
:::

::: en-file
 [Output](#example-0--Output-txt)
:::

::: en-intro
 [Пример из реальной жизни](#example-1)
:::

::: en-file
 [Example](#example-1--Example-swift)
:::

::: en-file
 [Output](#example-1--Output-txt)
:::
:::::::::::

**Сложность:** []{.fa-stars}

**Популярность:** []{.fa-stars}

**Применимость:** Паттерн можно часто встретить в Swift-коде, особенно в
коде, работающем с потоками данных.

**Признаки применения паттерна:** Декоратор можно распознать по
создающим методам, которые принимают в параметрах объекты того же
абстрактного типа или интерфейса, что и текущий класс.
::::::::::::::

::::: {style="padding: 1rem; border: solid 4px #e35e5e; border-radius: 1rem; text-align: center"}
::: {style="margin-bottom: 0.5rem;"}
Следующие примеры доступны на [Swift
Playgrounds](https://www.alemohamad.com/playgrounds){style="text-decoration: underline; display: inline-block"
rel="nofollow"}.
:::

::: {style="font-size: 13px;"}
Благодарность [Alejandro
Mohamad](https://www.alemohamad.com/){style="text-decoration: underline;"
rel="nofollow"} за создание версии Playground.
:::
:::::

::: {#nav-tab .nav .nav-tabs .nav-justified role="tablist"}
[Концептуальный пример](#example-0){#example-0-tab .nav-item .nav-link
.active toggle="tab" role="tab" aria-controls="example-0"
aria-selected="true"} [Пример из реальной
жизни](#example-1){#example-1-tab .nav-item .nav-link toggle="tab"
role="tab" aria-controls="example-1" aria-selected="true"}
:::

::::: {#nav-tabContent .pattern-example-tab-content .tab-content}
::: {#example-0 .tab-pane .show .active role="tabpanel" aria-labelledby="example-0-tab" style="padding-top: 24px"}
## Концептуальный пример {#example-0-title}

Этот пример показывает структуру паттерна **Декоратор**, а именно --- из
каких классов он состоит, какие роли эти классы выполняют и как они
взаимодействуют друг с другом.

После ознакомления со структурой, вам будет легче воспринимать второй
пример, который рассматривает реальный случай использования паттерна в
мире Swift.

#### []{#example-0--Example-swift .anchor} **Example.swift:** Пример структуры паттерна

<figure class="code">
<pre class="code" lang="swift"><code>import XCTest

/// Базовый интерфейс Компонента определяет поведение, которое изменяется
/// декораторами.
protocol Component {

    func operation() -&gt; String
}

/// Конкретные Компоненты предоставляют реализации поведения по умолчанию. Может
/// быть несколько вариаций этих классов.
class ConcreteComponent: Component {

    func operation() -&gt; String {
        return &quot;ConcreteComponent&quot;
    }
}

/// Базовый класс Декоратора следует тому же интерфейсу, что и другие
/// компоненты.   Основная цель этого класса - определить интерфейс обёртки для
/// всех конкретных декораторов. Реализация кода обёртки по умолчанию может
/// включать в себя  поле для хранения завёрнутого компонента и средства его
/// инициализации.
class Decorator: Component {

    private var component: Component

    init(_ component: Component) {
        self.component = component
    }

    /// Декоратор делегирует всю работу обёрнутому компоненту.
    func operation() -&gt; String {
        return component.operation()
    }
}

/// Конкретные Декораторы вызывают обёрнутый объект и изменяют его результат
/// некоторым образом.
class ConcreteDecoratorA: Decorator {

    /// Декораторы могут вызывать родительскую реализацию операции,  вместо
    /// того, чтобы вызвать обёрнутый объект напрямую. Такой подход упрощает
    /// расширение классов декораторов.
    override func operation() -&gt; String {
        return &quot;ConcreteDecoratorA(&quot; + super.operation() + &quot;)&quot;
    }
}

/// Декораторы могут выполнять своё поведение до или после вызова обёрнутого
/// объекта.
class ConcreteDecoratorB: Decorator {

    override func operation() -&gt; String {
        return &quot;ConcreteDecoratorB(&quot; + super.operation() + &quot;)&quot;
    }
}

/// Клиентский код работает со всеми объектами, используя интерфейс Компонента.
/// Таким образом, он остаётся независимым от конкретных классов компонентов,  с
/// которыми работает.
class Client {
    // ...
    static func someClientCode(component: Component) {
        print(&quot;Result: &quot; + component.operation())
    }
    // ...
}

/// Давайте посмотрим как всё это будет работать.
class DecoratorConceptual: XCTestCase {

    func testDecoratorConceptual() {
        // Таким образом, клиентский код может поддерживать как простые
        // компоненты...
        print(&quot;Client: I&#39;ve got a simple component&quot;)
        let simple = ConcreteComponent()
        Client.someClientCode(component: simple)

        // ...так и декорированные.
        //
        // Обратите внимание, что декораторы могут обёртывать не только простые
        // компоненты, но и другие декораторы.
        let decorator1 = ConcreteDecoratorA(simple)
        let decorator2 = ConcreteDecoratorB(decorator1)
        print(&quot;\nClient: Now I&#39;ve got a decorated component&quot;)
        Client.someClientCode(component: decorator2)
    }
}</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** Результат выполнения

<figure class="code">
<pre class="code" lang="output"><code>Client: I&#39;ve got a simple component
Result: ConcreteComponent

Client: Now I&#39;ve got a decorated component
Result: ConcreteDecoratorB(ConcreteDecoratorA(ConcreteComponent))</code></pre>
</figure>
:::

::: {#example-1 .tab-pane role="tabpanel" aria-labelledby="example-1-tab" style="padding-top: 24px"}
## Пример из реальной жизни {#example-1-title}

#### []{#example-1--Example-swift .anchor} **Example.swift:** Пример из реальной жизни

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

#### []{#example-1--Output-txt .anchor} **Output.txt:** Результат выполнения

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
[Концептуальный пример](#example-0){#example-0-tab-bottom .nav-item
.nav-link .active toggle="tab" role="tab" aria-controls="example-0"
aria-selected="true"} [Пример из реальной
жизни](#example-1){#example-1-tab-bottom .nav-item .nav-link
toggle="tab" role="tab" aria-controls="example-1" aria-selected="true"}
:::

::: next
#### Читаем дальше

[Фасад на Swift []{.fa
.fa-arrow-right}](../../facade/swift/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Компоновщик на
Swift](../../composite/swift/example.html){.btn .btn-default rel="prev"}
:::

## **Декоратор** на других языках программирования

[![Декоратор на
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Декоратор на C#"){.prog-lang-link}
[![Декоратор на
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Декоратор на C++"){.prog-lang-link}
[![Декоратор на
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Декоратор на Go"){.prog-lang-link}
[![Декоратор на
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Декоратор на Java"){.prog-lang-link}
[![Декоратор на
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Декоратор на PHP"){.prog-lang-link}
[![Декоратор на
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Декоратор на Python"){.prog-lang-link}
[![Декоратор на
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Декоратор на Ruby"){.prog-lang-link}
[![Декоратор на
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Декоратор на Rust"){.prog-lang-link}
[![Декоратор на
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Декоратор на TypeScript"){.prog-lang-link}
::::::::::::::::::::::::::::

::::::: {.prom .banner-sidebar .banner .banner-striped .banner-examples .banner-removable .banner-removable-patterns data-id="DP: Examples-IDE" creative-id="examples-sidebar-ru" position="sidebar"}
:::::: {.banner-inner style="font-size: 14px; line-height: 21px;"}
::: banner-examples-img
[![](../../../../images/patterns/banners/examples-ide91ca.png?id=3115b4b548fb96b75974e2de8f4f49bc){width="215"
height="158" loading="lazy"
srcset="/images/patterns/banners/examples-ide-2x.png?id=93c007a6d157b97c28eb213f59d716ae 2x"}](../../book.html)
:::

::: banner-examples-fade
[ Архив с примерами](../../book.html){.btn .btn-secondary .btn-block}
:::

::: {.banner-examples-text style="margin-top: 1rem;"}
Купи книгу **Погружение в Паттерны** и получи архив с десятками
детальных примеров, которые можно открывать прямо в IDE.

[ Узнать больше...](../../book.html){.btn .btn-secondary .btn-block}
:::
::::::
:::::::
::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::
