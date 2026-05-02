:::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::: {.main-content-container .center-content-container}
:::::: {.page .text}
::: breadcrumb
[](../../index.html){.home} / [디자인
패턴들](../design-patterns.html){.type} / [생성
패턴](creational-patterns.html){.type}
:::

# 팩토리 비교 {#팩토리-비교 .title}

이 글에서는 다음 항목들의 차이점을 설명합니다.

1.  팩토리
2.  생성 메서드
3.  정적 생성 (또는 팩토리) 메서드
4.  단순 팩토리
5.  [팩토리 메서드](factory-method.html) 패턴
6.  [추상 팩토리](abstract-factory.html) 패턴

인터넷에서 이러한 용어들에 대한 참고 문서를 찾을 수 있습니다. 용어들은
비슷해 보여도 모두 다른 의미를 갖고 있습니다. 많은 사람들이 이러한
사실을 알지 못해 용어를 혼동하고 오해합니다.

따라서 이러한 용어들의 올바른 정의와 차이점들을 확실히 짚고 넘어가
봅시다.

## 1. 팩토리

**팩토리**는 무언가를 생산해야 하는 함수, 메서드 또는 클래스를 나타내는
모호한 용어입니다. 가장 일반적으로 팩토리들은 객체들을 생성합니다.
하지만 파일, 데이터베이스 기록 등을 생성할 수도 있습니다.

가령, 다음 항목들은 모두 \'공장\'이라고 부를 수 있습니다.

-   프로그램의 그래픽 사용자 인터페이스를 생성하는 함수 또는 메서드
-   사용자들을 생성하는 클래스
-   특정 방식으로 클래스 생성자를 호출하는 정적 메서드
-   생성 디자인 패턴 중 하나

일반적으로 누군가가 \'팩토리\'라는 단어를 사용할 때 이 팩토리의 정확한
의미는 문맥상 명확해야 합니다. 하지만 의미가 확실하지 않다면 그냥
물어보세요. 글쓴이가 팩토리가 무엇인지 몰랐을 수도 있습니다.

## 2. 생성 메서드

[패턴을 활용한
리팩토링](http://www.kyobobook.co.kr/product/detailViewKor.laf?mallGb=KOR&ejkGb=KOR&barcode=9788991268920)이라는
책은 생성 메서드를 \'객체들을 생성하는 메서드\'로 정의합니다. 즉, 팩토리
메서드 패턴을 적용한 후의 모든 결과는 \'생성 메서드\'이지만 그 역이
반드시 성립하지는 않음을 뜻합니다. 또한 저자 마틴 파울러가
[리팩토링](http://www.kyobobook.co.kr/product/detailViewKor.laf?mallGb=KOR&ejkGb=KOR&barcode=9791162242742)에서
사용하는 \'팩토리 메서드\'라는 용어와 \'조슈아 블로크\'가 [이펙티브
자바](http://www.kyobobook.co.kr/product/detailViewKor.laf?ejkGb=KOR&mallGb=KOR&barcode=9788966262281&orderClick=LAG&Kc=)에서
사용하는 \'정적 팩토리 메서드\'라는 용어를 \'생성 메서드\'로 대체할 수
있다는 것을 의미하기도 합니다.

실제로 생성 메서드는 생성자 호출을 둘러싼 래퍼일 뿐이며, 당신의 의도를
더 잘 표현하는 이름을 가질 수도 있습니다. 반면에 생성 메서드는 당신의
코드를 생성자에 대한 변경 사항으로부터 분리하는 데 도움이 될 수
있습니다. 새로 객체를 만드는 대신 기존 객체들을 반환하는 특정 논리를
포함할 수도 있습니다.

많은 사람은 이러한 생성 메서드를 무언가를 생성한다는 이유만으로 \'팩토리
메서드\'라고 부릅니다. 그들의 논리는 간단합니다. 이러한 생성 메서드는
객체들을 생성한다: 모든 *[팩]{.cjk}[토]{.cjk}[리]{.cjk}*는 객체들을
생성한다: 따라서 이 메서드는 *[팩]{.cjk}[토]{.cjk}[리]{.cjk}
[메]{.cjk}[서]{.cjk}[드]{.cjk}*여야만 한다. 애석하게도 이러한 이유로
사람들은 생성 메서드를 실제 [팩토리 메서드](factory-method.html) 패턴과
혼동합니다.

다음 예시에서 `next`가 생성 메서드입니다.

<figure class="code">
<pre class="code" lang="php"><code>class Number {
    private $value;

    public function __construct($value) {
        $this-&gt;value = $value;
    }

    public function next() {
        return new Number ($this-&gt;value + 1);
    }
}</code></pre>
</figure>

## 3. 정적 생성 메서드

**정적 생성 메서드**는 `정적`으로 선언된 생성 메서드입니다. 즉, 이
메서드는 클래스를 통해 호출될 수 있으며 호출하기 위해 객체를 만들 필요가
없습니다. 참고로 특정한 경우에는 객체 없이 클래스를 사용할 수 있습니다.
(예: 클래스의 정적 메서드들에 접근할 때).

누군가가 이러한 메서드에 \'정적 팩토리 메서드\'와 같은 이름을 붙여도
혼동하지 마세요. 잘못된 습관일 뿐입니다. [팩토리
메서드](factory-method.html)는 상속에 의존하는 디자인 패턴입니다. 따라서
팩토리 메서드를 `static`​(정적)​으로 만들면 더 이상 자식 클래스들에서
확장할 수 없으므로 패턴의 목적에 어긋납니다.

정적 생성 메서드가 새 객체들을 반환하면 대체 생성자가 됩니다.

이것은 다음과 같은 경우에 유용할 수 있습니다.

-   목적은 다르지만, 시그니처가 일치하는 여러 다른 생성자들이 있어야 할
    때 유용합니다. 예를 들어 자바, C++, C# 및 기타 많은 언어에서는
    `Random­(int max)` 및 `Random­(int min)`을 모두 갖는 것이 불가능하며,
    이 문제를 해결하는 가장 인기 있는 방법은 디폴트 생성자를 호출하는
    여러 정적 메서드들을 생성한 후 나중에 적절한 값들을 설정하는
    것입니다.

-   또 새 객체들을 인스턴스화하는 대신 기존 객체를 재사용하려고 할 때
    유용합니다 ([싱글턴](singleton.html) 패턴 참조). 대부분의 프로그래밍
    언어에서 생성자들은 새 클래스 인스턴스들을 반환해야 하나, 정적 생성
    메서드를 사용하여 이 제한을 우회할 수 있습니다. 당신의 코드는 정적
    메서드 내에서 생성자를 호출하여 새 인스턴스를 만들지 아니면 어떤
    캐시에서 기존 객체를 반환할지 결정할 수 있습니다.

다음 예시에서 `load` 메서드는 정적 생성 메서드이며, 데이터베이스에서
사용자들을 가져오는 편리한 방법을 제공합니다.

<figure class="code">
<pre class="code" lang="php"><code>class User {
    private $id, $name, $email, $phone;

    public function __construct($id, $name, $email, $phone) {
        $this-&gt;id = $id;
        $this-&gt;name = $name;
        $this-&gt;email = $email;
        $this-&gt;phone = $phone;
    }

    public static function load($id) {
        list($id, $name, $email, $phone) = DB::load_data(&#39;users&#39;, &#39;id&#39;, &#39;name&#39;, &#39;email&#39;, &#39;phone&#39;);
        $user = new User($id, $name, $email, $phone);
        return $user;
    }
}</code></pre>
</figure>

## 4. *[단]{.cjk}[순]{.cjk} [팩]{.cjk}[토]{.cjk}[리]{.cjk}* 패턴

**단순 팩토리 패턴** []{.pop-i}[[Head First Design
Patterns](https://www.amazon.co.jp/-/dp/4873119766)에서
정의된]{.pop-content}은 메서드 매개변수들을 기반으로 어떤 제품 클래스를
인스턴스화한 후 반환할지를 결정하는 커다란 조건문이 있는 하나의 생성
메서드를 가진 클래스를 설명합니다.

사람들은 일반적으로 *[단]{.cjk}[순]{.cjk}
[팩]{.cjk}[토]{.cjk}[리]{.cjk}*를 일반 *[팩]{.cjk}[토]{.cjk}[리]{.cjk}*
또는 생성 디자인 패턴 중 하나와 혼동합니다. 대부분의 경우 단순 팩토리는
[팩토리 메서드](factory-method.html) 또는 [추상
팩토리](abstract-factory.html) 패턴을 도입하는 중간 단계입니다.

단순 팩토리는 일반적으로 단일 클래스의 단일 메서드로 표현됩니다. 시간이
지나면 이 메서드는 매우 커질 수 있으며, 그럴 때 메서드의 일부를 자식
클래스들로 추출할 수 있습니다. 이런 과정을 몇 번 반복하다 보면 코드
전체가 고전적인 *[팩]{.cjk}[토]{.cjk}[리]{.cjk}
[메]{.cjk}[서]{.cjk}[드]{.cjk}* 패턴으로 바뀌는 것을 발견할 수 있습니다.

그건 그렇고, 참고로 단순 팩토리를 `abstract`로 선언해도 팩토리가
마법처럼 *[추]{.cjk}[상]{.cjk} [팩]{.cjk}[토]{.cjk}[리]{.cjk}* 패턴이
되지는 않습니다.

다음은 *[단]{.cjk}[순]{.cjk} [팩]{.cjk}[토]{.cjk}[리]{.cjk}*의
예시입니다.

<figure class="code">
<pre class="code" lang="php"><code>class UserFactory {
    public static function create($type) {
        switch ($type) {
            case &#39;user&#39;: return new User();
            case &#39;customer&#39;: return new Customer();
            case &#39;admin&#39;: return new Admin();
            default:
                throw new Exception(&#39;Wrong user type passed.&#39;);
        }
    }
}</code></pre>
</figure>

## 5. *[팩]{.cjk}[토]{.cjk}[리]{.cjk} [메]{.cjk}[서]{.cjk}[드]{.cjk}* 패턴

**팩토리 메서드** []{.pop-i}[사인조가 쓴 책 [GOF의 디자인 패턴:
재사용성을 지닌 객체지향 소프트웨어의 핵심
요소](http://www.kyobobook.co.kr/product/detailViewKor.laf?mallGb=KOR&ejkGb=KOR&barcode=9788945072146)
에서 정의됨.]{.pop-content}는 객체 생성을 위한 인터페이스를 제공하며
자식 클래스들이 생성될 객체의 유형을 변경할 수 있도록 하는 생성 디자인
패턴입니다.

만약 기초 클래스에 생성 메서드가 있고 이를 확장하는 자식 클래스들이
있으면, 이는 팩토리 메서드일 가능성이 있습니다.

<figure class="code">
<pre class="code" lang="php"><code>abstract class Department {
    public abstract function createEmployee($id);

    public function fire($id) {
        $employee = $this-&gt;createEmployee($id);
        $employee-&gt;paySalary();
        $employee-&gt;dismiss();
    }
}

class ITDepartment extends Department {
    public function createEmployee($id) {
        return new Programmer($id);
    }
}

class AccountingDepartment extends Department {
    public function createEmployee($id) {
        return new Accountant($id);
    }
}</code></pre>
</figure>

## 6. *[추]{.cjk}[상]{.cjk} [팩]{.cjk}[토]{.cjk}[리]{.cjk}* 패턴

**추상 팩토리** []{.pop-i}[[GoF
책](http://www.kyobobook.co.kr/product/detailViewKor.laf?mallGb=KOR&ejkGb=KOR&barcode=9788945072146)에서
정의됨.]{.pop-content}는 그와 관련된 또는 그에 의존하는 객체들의 구상
클래스들을 지정하지 않고 이 객체들의 패밀리를 생성할 수 있도록 하는 생성
패턴입니다.

\'객체들의 패밀리\'란 무엇일까요? 예를 들어 다음 클래스들의 집합을
살펴봅시다. `교통수단` + `엔진` + `제어장치`. 이 집합에는 다음과 같은
여러 변형이 있을 수 있습니다.

1.  `자동차` + `연소엔진` + `핸들`
2.  `비행기` + `제트엔진` + `요크`

당신의 프로그램이 제품들의 패밀리들과 함께 작동하지 않으면 추상 팩토리는
필요하지 않습니다.

다시 강조하지만, 많은 사람이 *[추]{.cjk}[상]{.cjk}
[팩]{.cjk}[토]{.cjk}[리]{.cjk}* 패턴을 `abstract`로 선언된 단순 팩토리
클래스와 헷갈리곤 합니다. 혼동하지 마세요!

### 결론

다양한 용어의 차이점을 알았으니 이제 다음 디자인 패턴을 새롭게
살펴보세요.

-   [팩토리 메서드](factory-method.html)
-   [추상 팩토리](abstract-factory.html)

::: next
#### 다음을 보세요

[빌더 []{.fa .fa-arrow-right}](builder.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### 돌아가기

[[]{.fa .fa-arrow-left} 추상 팩토리](abstract-factory.html){.btn
.btn-default rel="prev"}
:::
::::::
:::::::
::::::::
