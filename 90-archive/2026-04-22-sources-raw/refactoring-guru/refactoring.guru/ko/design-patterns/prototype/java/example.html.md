:::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [디자인
패턴들](../../../design-patterns.html){.type} /
[프로토타입](../../prototype.html){.type} /
[자바](../../java.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![프로토타입](../../../../images/patterns/cards/prototype-mini6540.png?id=bc3046bb39ff36574c08d49839fd1c8e){srcset="/images/patterns/cards/prototype-mini-2x.png?id=b871f789a736e7efbb1cd082d2de6398 2x"}
:::

# 자바로 작성된 **프로토타입** {#자바로-작성된-프로토타입 .pattern-example-title-block-title}
::::

::::::::::::::::::::::: pattern-example-body
::: pattern-example-brief
**프로토타입**은 객체들​(복잡한 객체 포함)​을 그의 특정 클래스들에
결합하지 않고 복제할 수 있도록 하는 생성 디자인 패턴입니다.

모든 프로토타입 클래스들은 객체들의 구상 클래스들을 알 수 없는 경우에도
해당 객체들을 복사할 수 있도록 하는 공통 인터페이스가 있어야 합니다.
프로토타입 객체들은 전체 복사본들을 생성할 수 있습니다. 왜냐하면 같은
클래스의 객체들은 서로의 비공개 필드들에 접근할 수 있기 때문입니다.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ 프로토타입에 대하여 더 자세히 알아보세요 ](../../prototype.html){.btn
.btn-lg .btn-primary}
:::

:::::::::::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
내비게이션
:::

::: en-intro
 [소개](#)
:::

::: en-intro
 [그래픽 모양들의 복사](#example-0)
:::

::: en-file
 shapes
:::

:::: en-inside
::: en-file
  [Shape](#example-0--shapes-Shape-java)
:::
::::

:::: en-inside
::: en-file
  [Circle](#example-0--shapes-Circle-java)
:::
::::

:::: en-inside
::: en-file
  [Rectangle](#example-0--shapes-Rectangle-java)
:::
::::

::: en-file
 [Demo](#example-0--Demo-java)
:::

::: en-file
 [Output­Demo](#example-0--OutputDemo-txt)
:::

::: en-file
 cache
:::

:::: en-inside
::: en-file
  [Bundled­Shape­Cache](#example-0--cache-BundledShapeCache-java)
:::
::::

::: en-file
 [Demo](#example-0--Demo-java)
:::

::: en-file
 [Output­Demo](#example-0--OutputDemo-txt)
:::
::::::::::::::::::::

**복잡도:** []{.fa-stars}

**인기도:** []{.fa-stars}

**사용 사례들:** 프로토타입 패턴은 `Cloneable` 인터페이스를 통해
자바에서 바로 사용할 수 있습니다.

모든 클래스는 이 인터페이스를 구현하여 복제 가능해질 수 있습니다.

-   [`java.lang.Object#clone()`](http://docs.oracle.com/javase/8/docs/api/java/lang/Object.html#clone--)
    (클래스는
    [`java.lang.Cloneable`](http://docs.oracle.com/javase/8/docs/api/java/lang/Cloneable.html)
    인터페이스를 구현해야 합니다)

**식별:** 프로토타입은 `clone` 또는 `copy` 등의 메서드들의 유무로 식별할
수 있습니다.
:::::::::::::::::::::::

[]{#example-0}

## 그래픽 모양들의 복사 {#example-0-title}

표준 `Cloneable` 인터페이스 없이 프로토타입 패턴이 어떻게 구현될 수
있는지 살펴봅시다.

### []{#example-0--shapes .anchor} **shapes:** 모양 목록 {#shapes-모양-목록 .example-title}

#### []{#example-0--shapes-Shape-java .anchor} **shapes/Shape.java:** 공통 모양 인터페이스

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.prototype.example.shapes;

import java.util.Objects;

public abstract class Shape {
    public int x;
    public int y;
    public String color;

    public Shape() {
    }

    public Shape(Shape target) {
        if (target != null) {
            this.x = target.x;
            this.y = target.y;
            this.color = target.color;
        }
    }

    public abstract Shape clone();

    @Override
    public boolean equals(Object object2) {
        if (!(object2 instanceof Shape)) return false;
        Shape shape2 = (Shape) object2;
        return shape2.x == x &amp;&amp; shape2.y == y &amp;&amp; Objects.equals(shape2.color, color);
    }
}</code></pre>
</figure>

#### []{#example-0--shapes-Circle-java .anchor} **shapes/Circle.java:** 간단한 모양

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.prototype.example.shapes;

public class Circle extends Shape {
    public int radius;

    public Circle() {
    }

    public Circle(Circle target) {
        super(target);
        if (target != null) {
            this.radius = target.radius;
        }
    }

    @Override
    public Shape clone() {
        return new Circle(this);
    }

    @Override
    public boolean equals(Object object2) {
        if (!(object2 instanceof Circle) || !super.equals(object2)) return false;
        Circle shape2 = (Circle) object2;
        return shape2.radius == radius;
    }
}</code></pre>
</figure>

#### []{#example-0--shapes-Rectangle-java .anchor} **shapes/Rectangle.java:** 또 다른 모양

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.prototype.example.shapes;

public class Rectangle extends Shape {
    public int width;
    public int height;

    public Rectangle() {
    }

    public Rectangle(Rectangle target) {
        super(target);
        if (target != null) {
            this.width = target.width;
            this.height = target.height;
        }
    }

    @Override
    public Shape clone() {
        return new Rectangle(this);
    }

    @Override
    public boolean equals(Object object2) {
        if (!(object2 instanceof Rectangle) || !super.equals(object2)) return false;
        Rectangle shape2 = (Rectangle) object2;
        return shape2.width == width &amp;&amp; shape2.height == height;
    }
}</code></pre>
</figure>

#### []{#example-0--Demo-java .anchor} **Demo.java:** 복제 예시

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.prototype.example;

import refactoring_guru.prototype.example.shapes.Circle;
import refactoring_guru.prototype.example.shapes.Rectangle;
import refactoring_guru.prototype.example.shapes.Shape;

import java.util.ArrayList;
import java.util.List;

public class Demo {
    public static void main(String[] args) {
        List&lt;Shape&gt; shapes = new ArrayList&lt;&gt;();
        List&lt;Shape&gt; shapesCopy = new ArrayList&lt;&gt;();

        Circle circle = new Circle();
        circle.x = 10;
        circle.y = 20;
        circle.radius = 15;
        circle.color = &quot;red&quot;;
        shapes.add(circle);

        Circle anotherCircle = (Circle) circle.clone();
        shapes.add(anotherCircle);

        Rectangle rectangle = new Rectangle();
        rectangle.width = 10;
        rectangle.height = 20;
        rectangle.color = &quot;blue&quot;;
        shapes.add(rectangle);

        cloneAndCompare(shapes, shapesCopy);
    }

    private static void cloneAndCompare(List&lt;Shape&gt; shapes, List&lt;Shape&gt; shapesCopy) {
        for (Shape shape : shapes) {
            shapesCopy.add(shape.clone());
        }

        for (int i = 0; i &lt; shapes.size(); i++) {
            if (shapes.get(i) != shapesCopy.get(i)) {
                System.out.println(i + &quot;: Shapes are different objects (yay!)&quot;);
                if (shapes.get(i).equals(shapesCopy.get(i))) {
                    System.out.println(i + &quot;: And they are identical (yay!)&quot;);
                } else {
                    System.out.println(i + &quot;: But they are not identical (booo!)&quot;);
                }
            } else {
                System.out.println(i + &quot;: Shape objects are the same (booo!)&quot;);
            }
        }
    }
}</code></pre>
</figure>

#### []{#example-0--OutputDemo-txt .anchor} **OutputDemo.txt:** 실행 결과

<figure class="code">
<pre class="code" lang="output"><code>0: Shapes are different objects (yay!)
0: And they are identical (yay!)
1: Shapes are different objects (yay!)
1: And they are identical (yay!)
2: Shapes are different objects (yay!)
2: And they are identical (yay!)</code></pre>
</figure>

### 프로토타입 레지스트리

당신은 미리 정의된 프로토타입 객체들의 집합을 포함한 중앙 집중식
프로토타입 레지스트리​(또는 팩토리)​를 구현할 수 있습니다. 그러면 당신은
이름 또는 다른 매개변수들을 전달하여 팩토리로부터 새로운 객체들을 가져올
수 있습니다. 팩토리는 적절한 프로토타입을 검색한 후 복제 후 복사본을
반환할 것입니다.

### []{#example-0--cache .anchor} **cache** {#cache .example-title}

#### []{#example-0--cache-BundledShapeCache-java .anchor} **cache/BundledShapeCache.java:** 프로토타입 팩토리

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.prototype.caching.cache;

import refactoring_guru.prototype.example.shapes.Circle;
import refactoring_guru.prototype.example.shapes.Rectangle;
import refactoring_guru.prototype.example.shapes.Shape;

import java.util.HashMap;
import java.util.Map;

public class BundledShapeCache {
    private Map&lt;String, Shape&gt; cache = new HashMap&lt;&gt;();

    public BundledShapeCache() {
        Circle circle = new Circle();
        circle.x = 5;
        circle.y = 7;
        circle.radius = 45;
        circle.color = &quot;Green&quot;;

        Rectangle rectangle = new Rectangle();
        rectangle.x = 6;
        rectangle.y = 9;
        rectangle.width = 8;
        rectangle.height = 10;
        rectangle.color = &quot;Blue&quot;;

        cache.put(&quot;Big green circle&quot;, circle);
        cache.put(&quot;Medium blue rectangle&quot;, rectangle);
    }

    public Shape put(String key, Shape shape) {
        cache.put(key, shape);
        return shape;
    }

    public Shape get(String key) {
        return cache.get(key).clone();
    }
}</code></pre>
</figure>

#### []{#example-0--Demo-java .anchor} **Demo.java:** 복제 예시

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.prototype.caching;

import refactoring_guru.prototype.caching.cache.BundledShapeCache;
import refactoring_guru.prototype.example.shapes.Shape;

public class Demo {
    public static void main(String[] args) {
        BundledShapeCache cache = new BundledShapeCache();

        Shape shape1 = cache.get(&quot;Big green circle&quot;);
        Shape shape2 = cache.get(&quot;Medium blue rectangle&quot;);
        Shape shape3 = cache.get(&quot;Medium blue rectangle&quot;);

        if (shape1 != shape2 &amp;&amp; !shape1.equals(shape2)) {
            System.out.println(&quot;Big green circle != Medium blue rectangle (yay!)&quot;);
        } else {
            System.out.println(&quot;Big green circle == Medium blue rectangle (booo!)&quot;);
        }

        if (shape2 != shape3) {
            System.out.println(&quot;Medium blue rectangles are two different objects (yay!)&quot;);
            if (shape2.equals(shape3)) {
                System.out.println(&quot;And they are identical (yay!)&quot;);
            } else {
                System.out.println(&quot;But they are not identical (booo!)&quot;);
            }
        } else {
            System.out.println(&quot;Rectangle objects are the same (booo!)&quot;);
        }
    }
}</code></pre>
</figure>

#### []{#example-0--OutputDemo-txt .anchor} **OutputDemo.txt:** 실행 결과

<figure class="code">
<pre class="code" lang="output"><code>Big green circle != Medium blue rectangle (yay!)
Medium blue rectangles are two different objects (yay!)
And they are identical (yay!)</code></pre>
</figure>

::: next
#### 다음을 보세요

[자바로 작성된 싱글턴 []{.fa
.fa-arrow-right}](../../singleton/java/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### 돌아가기

[[]{.fa .fa-arrow-left} 자바로 작성된 팩토리
메서드](../../factory-method/java/example.html){.btn .btn-default
rel="prev"}
:::

## 다른 언어로 작성된 **프로토타입**

[![C#으로 작성된
프로토타입](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "C#으로 작성된 프로토타입"){.prog-lang-link}
[![C++로 작성된
프로토타입](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "C++로 작성된 프로토타입"){.prog-lang-link}
[![Go로 작성된
프로토타입](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Go로 작성된 프로토타입"){.prog-lang-link}
[![PHP로 작성된
프로토타입](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "PHP로 작성된 프로토타입"){.prog-lang-link}
[![파이썬으로 작성된
프로토타입](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "파이썬으로 작성된 프로토타입"){.prog-lang-link}
[![루비로 작성된
프로토타입](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "루비로 작성된 프로토타입"){.prog-lang-link}
[![러스트로 작성된
프로토타입](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "러스트로 작성된 프로토타입"){.prog-lang-link}
[![스위프트로 작성된
프로토타입](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "스위프트로 작성된 프로토타입"){.prog-lang-link}
[![타입스크립트로 작성된
프로토타입](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "타입스크립트로 작성된 프로토타입"){.prog-lang-link}
:::::::::::::::::::::::::::::

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
:::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::
