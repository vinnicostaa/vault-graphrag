:::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [디자인
패턴들](../../../design-patterns.html){.type} /
[플라이웨이트](../../flyweight.html){.type} /
[파이썬](../../python.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![플라이웨이트](../../../../images/patterns/cards/flyweight-minie3d8.png?id=422ca8d2f90614dce810a8812c626698){srcset="/images/patterns/cards/flyweight-mini-2x.png?id=857ca676fff84317bb0dab76abfce08e 2x"}
:::

# 파이썬으로 작성된 **플라이웨이트** {#파이썬으로-작성된-플라이웨이트 .pattern-example-title-block-title}
::::

::::::::::: pattern-example-body
::: pattern-example-brief
**플라이웨이트**는 구조 패턴이며 프로그램들이 객체들의 메모리 소비를
낮게 유지하여 방대한 양의 객체들을 지원할 수 있도록 합니다.

이 패턴은 여러 객체 사이의 객체 상태를 공유하여 위를 달성합니다. 다르게
설명하자면 플라이웨이트는 다른 객체들이 공통으로 사용하는 데이터를
캐싱하여 RAM을 절약합니다.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ 플라이웨이트에 대하여 더 자세히 알아보세요
](../../flyweight.html){.btn .btn-lg .btn-primary}
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
 [main](#example-0--main-py)
:::

::: en-file
 [Output](#example-0--Output-txt)
:::
::::::::

**복잡도:** []{.fa-stars}

**인기도:** []{.fa-stars}

**사용 사례들:** 플라이웨이트 패턴의 유일한 목적은 메모리 섭취를
최소화하는 것입니다. 당신의 프로그램이 RAM 부족으로 문제를 겪지 않는다면
당분간 이 패턴을 무시할 수 있습니다.

**식별:** 플라이웨이트는 새로운 객체들 대신 캐싱 된 객체들을 반환하는
생성 메서드의 유무로 식별될 수 있습니다.
:::::::::::

[]{#example-0}

## 개념적인 예시 {#example-0-title}

이 예시는 **플라이웨이트**의 구조를 보여주고 다음 질문에 중점을 둡니다:

-   패턴은 어떤 클래스들로 구성되어 있나요?
-   이 클래스들은 어떤 역할을 하나요?
-   패턴의 요소들은 어떻게 서로 연관되어 있나요?

#### []{#example-0--main-py .anchor} **main.py:** 개념적인 예시

<figure class="code">
<pre class="code" lang="python"><code>import json
from typing import Dict


class Flyweight():
    &quot;&quot;&quot;
    The Flyweight stores a common portion of the state (also called intrinsic
    state) that belongs to multiple real business entities. The Flyweight
    accepts the rest of the state (extrinsic state, unique for each entity) via
    its method parameters.
    &quot;&quot;&quot;

    def __init__(self, shared_state: str) -&gt; None:
        self._shared_state = shared_state

    def operation(self, unique_state: str) -&gt; None:
        s = json.dumps(self._shared_state)
        u = json.dumps(unique_state)
        print(f&quot;Flyweight: Displaying shared ({s}) and unique ({u}) state.&quot;, end=&quot;&quot;)


class FlyweightFactory():
    &quot;&quot;&quot;
    The Flyweight Factory creates and manages the Flyweight objects. It ensures
    that flyweights are shared correctly. When the client requests a flyweight,
    the factory either returns an existing instance or creates a new one, if it
    doesn&#39;t exist yet.
    &quot;&quot;&quot;

    _flyweights: Dict[str, Flyweight] = {}

    def __init__(self, initial_flyweights: Dict) -&gt; None:
        for state in initial_flyweights:
            self._flyweights[self.get_key(state)] = Flyweight(state)

    def get_key(self, state: Dict) -&gt; str:
        &quot;&quot;&quot;
        Returns a Flyweight&#39;s string hash for a given state.
        &quot;&quot;&quot;

        return &quot;_&quot;.join(sorted(state))

    def get_flyweight(self, shared_state: Dict) -&gt; Flyweight:
        &quot;&quot;&quot;
        Returns an existing Flyweight with a given state or creates a new one.
        &quot;&quot;&quot;

        key = self.get_key(shared_state)

        if not self._flyweights.get(key):
            print(&quot;FlyweightFactory: Can&#39;t find a flyweight, creating new one.&quot;)
            self._flyweights[key] = Flyweight(shared_state)
        else:
            print(&quot;FlyweightFactory: Reusing existing flyweight.&quot;)

        return self._flyweights[key]

    def list_flyweights(self) -&gt; None:
        count = len(self._flyweights)
        print(f&quot;FlyweightFactory: I have {count} flyweights:&quot;)
        print(&quot;\n&quot;.join(map(str, self._flyweights.keys())), end=&quot;&quot;)


def add_car_to_police_database(
    factory: FlyweightFactory, plates: str, owner: str,
    brand: str, model: str, color: str
) -&gt; None:
    print(&quot;\n\nClient: Adding a car to database.&quot;)
    flyweight = factory.get_flyweight([brand, model, color])
    # The client code either stores or calculates extrinsic state and passes it
    # to the flyweight&#39;s methods.
    flyweight.operation([plates, owner])


if __name__ == &quot;__main__&quot;:
    &quot;&quot;&quot;
    The client code usually creates a bunch of pre-populated flyweights in the
    initialization stage of the application.
    &quot;&quot;&quot;

    factory = FlyweightFactory([
        [&quot;Chevrolet&quot;, &quot;Camaro2018&quot;, &quot;pink&quot;],
        [&quot;Mercedes Benz&quot;, &quot;C300&quot;, &quot;black&quot;],
        [&quot;Mercedes Benz&quot;, &quot;C500&quot;, &quot;red&quot;],
        [&quot;BMW&quot;, &quot;M5&quot;, &quot;red&quot;],
        [&quot;BMW&quot;, &quot;X6&quot;, &quot;white&quot;],
    ])

    factory.list_flyweights()

    add_car_to_police_database(
        factory, &quot;CL234IR&quot;, &quot;James Doe&quot;, &quot;BMW&quot;, &quot;M5&quot;, &quot;red&quot;)

    add_car_to_police_database(
        factory, &quot;CL234IR&quot;, &quot;James Doe&quot;, &quot;BMW&quot;, &quot;X1&quot;, &quot;red&quot;)

    print(&quot;\n&quot;)

    factory.list_flyweights()</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** 실행 결과

<figure class="code">
<pre class="code" lang="output"><code>FlyweightFactory: I have 5 flyweights:
Camaro2018_Chevrolet_pink
C300_Mercedes Benz_black
C500_Mercedes Benz_red
BMW_M5_red
BMW_X6_white

Client: Adding a car to database.
FlyweightFactory: Reusing existing flyweight.
Flyweight: Displaying shared ([&quot;BMW&quot;, &quot;M5&quot;, &quot;red&quot;]) and unique ([&quot;CL234IR&quot;, &quot;James Doe&quot;]) state.

Client: Adding a car to database.
FlyweightFactory: Can&#39;t find a flyweight, creating new one.
Flyweight: Displaying shared ([&quot;BMW&quot;, &quot;X1&quot;, &quot;red&quot;]) and unique ([&quot;CL234IR&quot;, &quot;James Doe&quot;]) state.

FlyweightFactory: I have 6 flyweights:
Camaro2018_Chevrolet_pink
C300_Mercedes Benz_black
C500_Mercedes Benz_red
BMW_M5_red
BMW_X6_white
BMW_X1_red</code></pre>
</figure>

::: next
#### 다음을 보세요

[파이썬으로 작성된 프록시 []{.fa
.fa-arrow-right}](../../proxy/python/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### 돌아가기

[[]{.fa .fa-arrow-left} 파이썬으로 작성된
퍼사드](../../facade/python/example.html){.btn .btn-default rel="prev"}
:::

## 다른 언어로 작성된 **플라이웨이트**

[![C#으로 작성된
플라이웨이트](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "C#으로 작성된 플라이웨이트"){.prog-lang-link}
[![C++로 작성된
플라이웨이트](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "C++로 작성된 플라이웨이트"){.prog-lang-link}
[![Go로 작성된
플라이웨이트](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Go로 작성된 플라이웨이트"){.prog-lang-link}
[![자바로 작성된
플라이웨이트](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "자바로 작성된 플라이웨이트"){.prog-lang-link}
[![PHP로 작성된
플라이웨이트](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "PHP로 작성된 플라이웨이트"){.prog-lang-link}
[![루비로 작성된
플라이웨이트](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "루비로 작성된 플라이웨이트"){.prog-lang-link}
[![러스트로 작성된
플라이웨이트](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "러스트로 작성된 플라이웨이트"){.prog-lang-link}
[![스위프트로 작성된
플라이웨이트](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "스위프트로 작성된 플라이웨이트"){.prog-lang-link}
[![타입스크립트로 작성된
플라이웨이트](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "타입스크립트로 작성된 플라이웨이트"){.prog-lang-link}
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
