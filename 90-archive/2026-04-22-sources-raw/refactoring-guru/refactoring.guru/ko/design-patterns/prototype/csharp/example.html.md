:::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [디자인
패턴들](../../../design-patterns.html){.type} /
[프로토타입](../../prototype.html){.type} /
[C#](../../csharp.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![프로토타입](../../../../images/patterns/cards/prototype-mini6540.png?id=bc3046bb39ff36574c08d49839fd1c8e){srcset="/images/patterns/cards/prototype-mini-2x.png?id=b871f789a736e7efbb1cd082d2de6398 2x"}
:::

# C#으로 작성된 **프로토타입** {#c으로-작성된-프로토타입 .pattern-example-title-block-title}
::::

::::::::::: pattern-example-body
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

:::::::: {.sidebar-navigation .with-scroll}
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
 [Program](#example-0--Program-cs)
:::

::: en-file
 [Output](#example-0--Output-txt)
:::
::::::::

**복잡도:** []{.fa-stars}

**인기도:** []{.fa-stars}

**사용 사례들:** 프로토타입 패턴은 `ICloneable` 인터페이스를 통해 C#에서
바로 사용할 수 있습니다.

**식별:** 프로토타입은 `clone` 또는 `copy` 등의 메서드들의 유무로 식별할
수 있습니다.
:::::::::::

[]{#example-0}

## 개념적인 예시 {#example-0-title}

이 예시는 **프로토타입** 패턴의 구조를 보여주고 다음 질문에 중점을
둡니다:

-   패턴은 어떤 클래스들로 구성되어 있나요?
-   이 클래스들은 어떤 역할을 하나요?
-   패턴의 요소들은 어떻게 서로 연관되어 있나요?

#### []{#example-0--Program-cs .anchor} **Program.cs:** 개념적인 예시

<figure class="code">
<pre class="code" lang="csharp"><code>using System;

namespace RefactoringGuru.DesignPatterns.Prototype.Conceptual
{
    public class Person
    {
        public int Age;
        public DateTime BirthDate;
        public string Name;
        public IdInfo IdInfo;

        public Person ShallowCopy()
        {
            return (Person) this.MemberwiseClone();
        }

        public Person DeepCopy()
        {
            Person clone = (Person) this.MemberwiseClone();
            clone.IdInfo = new IdInfo(IdInfo.IdNumber);
            clone.Name = String.Copy(Name);
            return clone;
        }
    }

    public class IdInfo
    {
        public int IdNumber;

        public IdInfo(int idNumber)
        {
            this.IdNumber = idNumber;
        }
    }

    class Program
    {
        static void Main(string[] args)
        {
            Person p1 = new Person();
            p1.Age = 42;
            p1.BirthDate = Convert.ToDateTime(&quot;1977-01-01&quot;);
            p1.Name = &quot;Jack Daniels&quot;;
            p1.IdInfo = new IdInfo(666);

            // Perform a shallow copy of p1 and assign it to p2.
            Person p2 = p1.ShallowCopy();
            // Make a deep copy of p1 and assign it to p3.
            Person p3 = p1.DeepCopy();

            // Display values of p1, p2 and p3.
            Console.WriteLine(&quot;Original values of p1, p2, p3:&quot;);
            Console.WriteLine(&quot;   p1 instance values: &quot;);
            DisplayValues(p1);
            Console.WriteLine(&quot;   p2 instance values:&quot;);
            DisplayValues(p2);
            Console.WriteLine(&quot;   p3 instance values:&quot;);
            DisplayValues(p3);

            // Change the value of p1 properties and display the values of p1,
            // p2 and p3.
            p1.Age = 32;
            p1.BirthDate = Convert.ToDateTime(&quot;1900-01-01&quot;);
            p1.Name = &quot;Frank&quot;;
            p1.IdInfo.IdNumber = 7878;
            Console.WriteLine(&quot;\nValues of p1, p2 and p3 after changes to p1:&quot;);
            Console.WriteLine(&quot;   p1 instance values: &quot;);
            DisplayValues(p1);
            Console.WriteLine(&quot;   p2 instance values (reference values have changed):&quot;);
            DisplayValues(p2);
            Console.WriteLine(&quot;   p3 instance values (everything was kept the same):&quot;);
            DisplayValues(p3);
        }

        public static void DisplayValues(Person p)
        {
            Console.WriteLine(&quot;      Name: {0:s}, Age: {1:d}, BirthDate: {2:MM/dd/yy}&quot;,
                p.Name, p.Age, p.BirthDate);
            Console.WriteLine(&quot;      ID#: {0:d}&quot;, p.IdInfo.IdNumber);
        }
    }
}</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** 실행 결과

<figure class="code">
<pre class="code" lang="output"><code>Original values of p1, p2, p3:
   p1 instance values: 
      Name: Jack Daniels, Age: 42, BirthDate: 01/01/77
      ID#: 666
   p2 instance values:
      Name: Jack Daniels, Age: 42, BirthDate: 01/01/77
      ID#: 666
   p3 instance values:
      Name: Jack Daniels, Age: 42, BirthDate: 01/01/77
      ID#: 666

Values of p1, p2 and p3 after changes to p1:
   p1 instance values: 
      Name: Frank, Age: 32, BirthDate: 01/01/00
      ID#: 7878
   p2 instance values (reference values have changed):
      Name: Jack Daniels, Age: 42, BirthDate: 01/01/77
      ID#: 7878
   p3 instance values (everything was kept the same):
      Name: Jack Daniels, Age: 42, BirthDate: 01/01/77
      ID#: 666</code></pre>
</figure>

::: next
#### 다음을 보세요

[C#으로 작성된 싱글턴 []{.fa
.fa-arrow-right}](../../singleton/csharp/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### 돌아가기

[[]{.fa .fa-arrow-left} C#으로 작성된 팩토리
메서드](../../factory-method/csharp/example.html){.btn .btn-default
rel="prev"}
:::

## 다른 언어로 작성된 **프로토타입**

[![C++로 작성된
프로토타입](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "C++로 작성된 프로토타입"){.prog-lang-link}
[![Go로 작성된
프로토타입](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Go로 작성된 프로토타입"){.prog-lang-link}
[![자바로 작성된
프로토타입](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "자바로 작성된 프로토타입"){.prog-lang-link}
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
:::::::::::::::::

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
:::::::::::::::::::::::
::::::::::::::::::::::::
