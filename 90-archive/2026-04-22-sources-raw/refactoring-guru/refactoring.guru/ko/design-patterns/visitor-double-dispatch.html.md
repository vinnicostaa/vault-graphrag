:::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::: {.main-content-container .center-content-container}
:::::: {.page .text}
::: breadcrumb
[](../../index.html){.home} / [디자인
패턴들](../design-patterns.html){.type} / [행동
패턴](behavioral-patterns.html){.type} / [비지터](visitor.html){.type}
:::

# 비지터 패턴과 이중 디스패치 {#비지터-패턴과-이중-디스패치 .title}

다음 기하학적 모양들의 클래스 계층 구조를 살펴보겠습니다. (다음은 실제
코드가 아닌 의사 코드라는 점을 주의하세요).

<figure class="code">
<pre class="code" lang="pseudocode"><code>interface Graphic is
    method draw()

class Shape implements Graphic is
    field id
    method draw()
    // …

class Dot extends Shape is
    field x, y
    method draw()
    // …

class Circle extends Dot is
    field radius
    method draw()
    // …

class Rectangle extends Shape is
    field width, height
    method draw()
    // …

class CompoundGraphic implements Graphic is
    field children: array of Graphic
    method draw()
    // …</code></pre>
</figure>

코드가 제대로 작동하고 있고 앱이 프로덕션 단계에 있다고 가정합시다. 어느
날 당신은 내보내기 기능을 만들기로 했습니다. 내보내기 코드가 위와 같은
클래스에 배치되면 이질적으로 보일 것입니다. 그래서 당신은 이 계층구조의
모든 클래스에 내보내기 기능을 추가하는 대신 모든 내보내기 로직을 포함한
새로운 클래스를 계층구조의 외부에 만들기로 했습니다. 이 클래스는 각
객체의 공개된 상태를 XML 문자열로 내보내는 메서드들을 가져옵니다.

<figure class="code">
<pre class="code" lang="pseudocode"><code>class Exporter is
    method export(s: Shape) is
        print(&quot;Exporting shape&quot;)
    method export(d: Dot)
        print(&quot;Exporting dot&quot;)
    method export(c: Circle)
        print(&quot;Exporting circle&quot;)
    method export(r: Rectangle)
        print(&quot;Exporting rectangle&quot;)
    method export(cs: CompoundGraphic)
        print(&quot;Exporting compound&quot;)</code></pre>
</figure>

코드가 괜찮아 보이네요. 그래도 한 번 시도해 봅시다.

<figure class="code">
<pre class="code" lang="pseudocode"><code>class App() is
    method export(shape: Shape) is
        Exporter exporter = new Exporter()
        exporter.export(shape);

app.export(new Circle());
// Unfortunatelly, this will output &quot;Exporting shape&quot;.</code></pre>
</figure>

아니 잠깐만, 왜 이러지!

## 컴파일러처럼 생각하기

참고로 다음 정보는 대부분의 최신 객체 지향 프로그래밍 언어들​(자바, C#,
PHP 등)​에 해당합니다.

### 늦은/동적 바인딩

당신이 컴파일러라고 가정해 봅시다. 당신은 다음 코드를 컴파일하는 방법을
결정해야 합니다.

<figure class="code">
<pre class="code" lang="pseudocode"><code>method drawShape(shape: Shape) is
    shape.draw();</code></pre>
</figure>

어디 보자\... `Shape`​(모양) 클래스에 `draw`​(그리기) 메서드가 정의되어
있습니다. 그런데 잠시만요, 이 메서드를 오버라이드하는 자식 클래스들도
4개나 있네요. 이 구현들 중 어떤 구현을 호출할지 안전하게 결정할 수
있을까요? 그럴 것 같지 않네요. 이 문제에 대한 해답을 확실히 알 수 있는
유일한 방법은 프로그램을 시작한 후 메서드에 전달된 객체의 클래스를
확인하는 것입니다. 왜냐하면 우리가 알고 있는 가장 확실한 사실은 이
객체가 `draw` 메서드의 구현을 **반드시** 가질 것이라는 사실이기
때문입니다.

따라서 결과 머신 코드는 `shape` 매개변수에 전달된 객체의 클래스를 확인할
것이며 적절한 클래스에서 `draw` 구현을 선택할 것입니다.

이러한 동적 유형 검사를 늦은​(또는 동적) 바인딩이라고 합니다.

-   **늦은**이라고 하는 이유는 컴파일 후 런타임 때 객체와 그의 구현을
    연결하기 때문입니다.
-   **동적**이라고 하는 이유는 각 새로운 객체가 다른 구현에 연결되어야
    할 필요가 있을 수 있기 때문입니다.

### 이른/정적 바인딩

이제 다음 코드를 \'컴파일\' 합시다.

<figure class="code">
<pre class="code" lang="pseudocode"><code>method exportShape(shape: Shape) is
    Exporter exporter = new Exporter()
    exporter.export(shape);</code></pre>
</figure>

두 번째 줄에서는 모든 것이 명확합니다. `Exporter`​(내보내는 자) 클래스에
맞춤형 생성자가 없으므로 우리는 단순히 객체를 인스턴스화합니다.
`export`​(내보내기) 호출은 어떤가요? `Exporter`에는 같은 이름을 가진
5개의 메서드가 있으며 모두 다른 유형의 매개변수들을 가지고 있습니다.
그러면 어느 메서드를 호출해야 할까요? 아무래도 여기에서도 동적 바인딩이
필요할 것 같습니다.

하지만 또 다른 문제가 있습니다. `Exporter` 클래스에 적절한 `export`
메서드가 없는 모양 클래스가 있으면 어떻게 될까요? 예를 들어, `타원` 객체
같은 경우 말입니다. 컴파일러는 오버라이딩된 메서드들과 대조를 이루어
적절하게 오버로드된 메서드가 존재한다고 보장할 수 없습니다. 이로 인해
컴파일러가 허용할 수 없는 모호한 상황이 발생합니다.

따라서 컴파일러 개발자들은 이러한 문제들을 예방하기 위해 오버로드된
메서드들에 대해 이른​(또는 정적) 바인딩을 사용합니다.

-   **이른**이라고 하는 이유는 프로그램이 실행하기 전에 컴파일 때
    바인딩이 시행되기 때문입니다.
-   **정적**이라고 하는 이유는 런타임 때 이것이 변경될 수 없기
    때문입니다.

다시 위의 예시를 살펴봅시다. 우리는 들어오는 인수가 `Shape` 계층 구조의
구성원일 것이라고 확신합니다. `Shape` 클래스 또는 그 자식 클래스 중
하나의 구성원일 것입니다. 또 `Exporter` 클래스에는 `Shape` 클래스를
지원하는 내보내기의 기초 구현이 있다는 사실도 알고 있습니다. (예:
`export(s: Shape)`).

이것이 모호한 상황을 피하며 주어진 코드에 안전하게 연결할 수 있는 유일한
구현입니다. 그러면 우리가 `Rectangle` 객체를 `export­Shape`에
전달하더라도 exporter는 여전히 `export(s: Shape)` 메서드를 호출합니다.

## 이중 디스패치

**이중 디스패치**는 오버로딩된 메서드와 함께 동적 바인딩을 사용할 수
있는 요령이며, 수행 방법은 다음과 같습니다.

<figure class="code">
<pre class="code" lang="pseudocode"><code>class Visitor is
    method visit(s: Shape) is
        print(&quot;Visited shape&quot;)
    method visit(d: Dot)
        print(&quot;Visited dot&quot;)

interface Graphic is
    method accept(v: Visitor)

class Shape implements Graphic is
    method accept(v: Visitor)
        // Compiler knows for sure that `this` is a `Shape`.
        // Which means that the `visit(s: Shape)` can be safely called.
        v.visit(this)

class Dot extends Shape is
    method accept(v: Visitor)
        // Compiler knows that `this` is a `Dot`.
        // Which means that the `visit(s: Dot)` can be safely called.
        v.visit(this)


Visitor v = new Visitor();
Graphic g = new Dot();

// The `accept` method is overriden, not overloaded. Compiler binds it
// dynamically. Therefore the `accept` will be executed on a class that
// corresponds to an object calling a method (in our case, the `Dot` class).
g.accept(v);

// Output: &quot;Visited dot&quot;</code></pre>
</figure>

### 결론

[Visitor](visitor.html) 패턴은 이중 디스패치 원칙을 기반으로 하지만
그것이 주목적은 아닙니다. 비지터 패턴은 전체 클래스 계층에 이러한
클래스들의 기존 코드를 변경하지 않고 \'외부\' 작업들을 추가할 수 있도록
합니다.

::: next
#### 다음을 보세요

[C#으로 작성된 디자인 패턴들 []{.fa .fa-arrow-right}](csharp.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### 돌아가기

[[]{.fa .fa-arrow-left} 비지터](visitor.html){.btn .btn-default
rel="prev"}
:::
::::::
:::::::
::::::::
