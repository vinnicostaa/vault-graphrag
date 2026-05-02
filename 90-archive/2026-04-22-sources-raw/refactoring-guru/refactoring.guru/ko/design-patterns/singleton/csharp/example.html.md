:::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [디자인
패턴들](../../../design-patterns.html){.type} /
[싱글턴](../../singleton.html){.type} / [C#](../../csharp.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![싱글턴](../../../../images/patterns/cards/singleton-minif559.png?id=914e1565dfdf15f240e766163bd303ec){srcset="/images/patterns/cards/singleton-mini-2x.png?id=4a7348083d7719d02709a5c9ff878b9f 2x"}
:::

# C#으로 작성된 **싱글턴** {#c으로-작성된-싱글턴 .pattern-example-title-block-title}
::::

::::::::::::::: pattern-example-body
::: pattern-example-brief
**싱글턴**은 같은 종류의 객체가 하나만 존재하도록 하고 다른 코드의 해당
객체에 대한 단일 접근 지점을 제공하는 생성 디자인 패턴입니다.

싱글턴은 전역 변수들과 거의 같은 장단점을 가지고 있습니다: 매우 편리하나
코드의 모듈성을 깨뜨립니다.

싱글턴에 의존하는 클래스를 다른 콘텍스트에서 사용하려면 싱글턴도 다른
콘텍스트로 전달해야 합니다. 대부분의 경우 이 제한 사항은 유닛 테스트를
생성하는 동안 발생합니다.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ 싱글턴에 대하여 더 자세히 알아보세요 ](../../singleton.html){.btn
.btn-lg .btn-primary}
:::

:::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
내비게이션
:::

::: en-intro
 [소개](#)
:::

::: en-intro
 [기본 싱글턴](#example-0)
:::

::: en-file
 [Program](#example-0--Program-cs)
:::

::: en-file
 [Output](#example-0--Output-txt)
:::

::: en-intro
 [스레드로부터 안전한 싱글턴](#example-1)
:::

::: en-file
 [Program](#example-1--Program-cs)
:::

::: en-file
 [Output](#example-1--Output-txt)
:::

::: en-intro
 [더 많은 정보를 원하시나요?](#example-2)
:::
::::::::::::

**복잡도:** []{.fa-stars}

**인기도:** []{.fa-stars}

**사용 사례들:** 많은 개발자는 싱클턴을 안티패턴으로 간주합니다. 그래서
C# 코드에서의 사용이 감소하고 있습니다.

**식별:** 싱글턴은 같은 캐싱 된 객체를 반환하는 정적 생성 메서드로
식별될 수 있습니다.
:::::::::::::::

::: {#nav-tab .nav .nav-tabs .nav-justified role="tablist"}
[기본 싱글턴](#example-0){#example-0-tab .nav-item .nav-link .active
toggle="tab" role="tab" aria-controls="example-0" aria-selected="true"}
[스레드로부터 안전한 싱글턴](#example-1){#example-1-tab .nav-item
.nav-link toggle="tab" role="tab" aria-controls="example-1"
aria-selected="true"} [더 많은 정보를
원하시나요?](#example-2){#example-2-tab .nav-item .nav-link toggle="tab"
role="tab" aria-controls="example-2" aria-selected="true"}
:::

:::::: {#nav-tabContent .pattern-example-tab-content .tab-content}
::: {#example-0 .tab-pane .show .active role="tabpanel" aria-labelledby="example-0-tab" style="padding-top: 24px"}
## 기본 싱글턴 {#example-0-title}

조잡한 싱글턴을 구현하는 것은 매우 쉽습니다. 생성자를 숨기고 정적 생성
메서드를 구현하기만 하면 됩니다.

같은 클래스는 다중 스레드 환경에서 잘못 작동합니다. 여러 스레드가 생성
메서드를 동시에 호출할 수 있으며 싱글턴 클래스의 여러 인스턴스를 가져올
수 있기 때문입니다.

#### []{#example-0--Program-cs .anchor} **Program.cs:** 개념적인 예시

<figure class="code">
<pre class="code" lang="csharp"><code>using System;

namespace RefactoringGuru.DesignPatterns.Singleton.Conceptual.NonThreadSafe
{
    // The Singleton class defines the `GetInstance` method that serves as an
    // alternative to constructor and lets clients access the same instance of
    // this class over and over.

    // EN : The Singleton should always be a &#39;sealed&#39; class to prevent class
    // inheritance through external classes and also through nested classes.
    public sealed class Singleton
    {
        // The Singleton&#39;s constructor should always be private to prevent
        // direct construction calls with the `new` operator.
        private Singleton() { }

        // The Singleton&#39;s instance is stored in a static field. There there are
        // multiple ways to initialize this field, all of them have various pros
        // and cons. In this example we&#39;ll show the simplest of these ways,
        // which, however, doesn&#39;t work really well in multithreaded program.
        private static Singleton _instance;

        // This is the static method that controls the access to the singleton
        // instance. On the first run, it creates a singleton object and places
        // it into the static field. On subsequent runs, it returns the client
        // existing object stored in the static field.
        public static Singleton GetInstance()
        {
            if (_instance == null)
            {
                _instance = new Singleton();
            }
            return _instance;
        }

        // Finally, any singleton should define some business logic, which can
        // be executed on its instance.
        public void someBusinessLogic()
        {
            // ...
        }
    }

    class Program
    {
        static void Main(string[] args)
        {
            // The client code.
            Singleton s1 = Singleton.GetInstance();
            Singleton s2 = Singleton.GetInstance();

            if (s1 == s2)
            {
                Console.WriteLine(&quot;Singleton works, both variables contain the same instance.&quot;);
            }
            else
            {
                Console.WriteLine(&quot;Singleton failed, variables contain different instances.&quot;);
            }
        }
    }
}</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** 실행 결과

<figure class="code">
<pre class="code" lang="output"><code>Singleton works, both variables contain the same instance.</code></pre>
</figure>
:::

::: {#example-1 .tab-pane role="tabpanel" aria-labelledby="example-1-tab" style="padding-top: 24px"}
## 스레드로부터 안전한 싱글턴 {#example-1-title}

이 문제를 해결하려면 싱글턴 객체를 처음 생성하는 동안 스레드들을
동기화해야 합니다.

#### []{#example-1--Program-cs .anchor} **Program.cs:** 개념적인 예시

<figure class="code">
<pre class="code" lang="csharp"><code>using System;
using System.Threading;

namespace Singleton
{
    // This Singleton implementation is called &quot;double check lock&quot;. It is safe
    // in multithreaded environment and provides lazy initialization for the
    // Singleton object.
    class Singleton
    {
        private Singleton() { }

        private static Singleton _instance;

        // We now have a lock object that will be used to synchronize threads
        // during first access to the Singleton.
        private static readonly object _lock = new object();

        public static Singleton GetInstance(string value)
        {
            // This conditional is needed to prevent threads stumbling over the
            // lock once the instance is ready.
            if (_instance == null)
            {
                // Now, imagine that the program has just been launched. Since
                // there&#39;s no Singleton instance yet, multiple threads can
                // simultaneously pass the previous conditional and reach this
                // point almost at the same time. The first of them will acquire
                // lock and will proceed further, while the rest will wait here.
                lock (_lock)
                {
                    // The first thread to acquire the lock, reaches this
                    // conditional, goes inside and creates the Singleton
                    // instance. Once it leaves the lock block, a thread that
                    // might have been waiting for the lock release may then
                    // enter this section. But since the Singleton field is
                    // already initialized, the thread won&#39;t create a new
                    // object.
                    if (_instance == null)
                    {
                        _instance = new Singleton();
                        _instance.Value = value;
                    }
                }
            }
            return _instance;
        }

        // We&#39;ll use this property to prove that our Singleton really works.
        public string Value { get; set; }
    }

    class Program
    {
        static void Main(string[] args)
        {
            // The client code.
            
            Console.WriteLine(
                &quot;{0}\n{1}\n\n{2}\n&quot;,
                &quot;If you see the same value, then singleton was reused (yay!)&quot;,
                &quot;If you see different values, then 2 singletons were created (booo!!)&quot;,
                &quot;RESULT:&quot;
            );
            
            Thread process1 = new Thread(() =&gt;
            {
                TestSingleton(&quot;FOO&quot;);
            });
            Thread process2 = new Thread(() =&gt;
            {
                TestSingleton(&quot;BAR&quot;);
            });
            
            process1.Start();
            process2.Start();
            
            process1.Join();
            process2.Join();
        }
        
        public static void TestSingleton(string value)
        {
            Singleton singleton = Singleton.GetInstance(value);
            Console.WriteLine(singleton.Value);
        } 
    }
}</code></pre>
</figure>

#### []{#example-1--Output-txt .anchor} **Output.txt:** 실행 결과

<figure class="code">
<pre class="code" lang="output"><code>FOO
FOO</code></pre>
</figure>
:::

::: {#example-2 .tab-pane role="tabpanel" aria-labelledby="example-2-tab" style="padding-top: 24px"}
## 더 많은 정보를 원하시나요? {#example-2-title}

C#에는 싱글턴 패턴의 더 많은 변형이 있습니다. 자세한 내용을 알아보시려면
이 기사를 참조하세요:

[C# 깊이 탐구하기: 싱글턴의
적용](https://csharpindepth.com/Articles/Singleton)
:::
::::::

::: {#nav-tab .nav .nav-tabs .nav-tabs-bottom .nav-justified role="tablist"}
[기본 싱글턴](#example-0){#example-0-tab-bottom .nav-item .nav-link
.active toggle="tab" role="tab" aria-controls="example-0"
aria-selected="true"} [스레드로부터 안전한
싱글턴](#example-1){#example-1-tab-bottom .nav-item .nav-link
toggle="tab" role="tab" aria-controls="example-1" aria-selected="true"}
[더 많은 정보를 원하시나요?](#example-2){#example-2-tab-bottom .nav-item
.nav-link toggle="tab" role="tab" aria-controls="example-2"
aria-selected="true"}
:::

::: next
#### 다음을 보세요

[C#으로 작성된 어댑터 []{.fa
.fa-arrow-right}](../../adapter/csharp/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### 돌아가기

[[]{.fa .fa-arrow-left} C#으로 작성된
프로토타입](../../prototype/csharp/example.html){.btn .btn-default
rel="prev"}
:::

## 다른 언어로 작성된 **싱글턴**

[![C++로 작성된
싱글턴](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "C++로 작성된 싱글턴"){.prog-lang-link}
[![Go로 작성된
싱글턴](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Go로 작성된 싱글턴"){.prog-lang-link}
[![자바로 작성된
싱글턴](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "자바로 작성된 싱글턴"){.prog-lang-link}
[![PHP로 작성된
싱글턴](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "PHP로 작성된 싱글턴"){.prog-lang-link}
[![파이썬으로 작성된
싱글턴](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "파이썬으로 작성된 싱글턴"){.prog-lang-link}
[![루비로 작성된
싱글턴](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "루비로 작성된 싱글턴"){.prog-lang-link}
[![러스트로 작성된
싱글턴](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "러스트로 작성된 싱글턴"){.prog-lang-link}
[![스위프트로 작성된
싱글턴](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "스위프트로 작성된 싱글턴"){.prog-lang-link}
[![타입스크립트로 작성된
싱글턴](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "타입스크립트로 작성된 싱글턴"){.prog-lang-link}
:::::::::::::::::::::::::::

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
:::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::
