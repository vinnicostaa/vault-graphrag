:::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [디자인
패턴들](../../../design-patterns.html){.type} /
[프로토타입](../../prototype.html){.type} /
[파이썬](../../python.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![프로토타입](../../../../images/patterns/cards/prototype-mini6540.png?id=bc3046bb39ff36574c08d49839fd1c8e){srcset="/images/patterns/cards/prototype-mini-2x.png?id=b871f789a736e7efbb1cd082d2de6398 2x"}
:::

# 파이썬으로 작성된 **프로토타입** {#파이썬으로-작성된-프로토타입 .pattern-example-title-block-title}
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
 [main](#example-0--main-py)
:::

::: en-file
 [Output](#example-0--Output-txt)
:::
::::::::

**복잡도:** []{.fa-stars}

**인기도:** []{.fa-stars}

**사용 사례들:** 프로토타입 패턴은 `copy` 모듈을 통해 파이썬에서 바로
사용할 수 있습니다.

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

#### []{#example-0--main-py .anchor} **main.py:** 개념적인 예시

<figure class="code">
<pre class="code" lang="python"><code>import copy


class SelfReferencingEntity:
    def __init__(self):
        self.parent = None

    def set_parent(self, parent):
        self.parent = parent


class SomeComponent:
    &quot;&quot;&quot;
    Python provides its own interface of Prototype via `copy.copy` and
    `copy.deepcopy` functions. And any class that wants to implement custom
    implementations have to override `__copy__` and `__deepcopy__` member
    functions.
    &quot;&quot;&quot;

    def __init__(self, some_int, some_list_of_objects, some_circular_ref):
        self.some_int = some_int
        self.some_list_of_objects = some_list_of_objects
        self.some_circular_ref = some_circular_ref

    def __copy__(self):
        &quot;&quot;&quot;
        Create a shallow copy. This method will be called whenever someone calls
        `copy.copy` with this object and the returned value is returned as the
        new shallow copy.
        &quot;&quot;&quot;

        # First, let&#39;s create copies of the nested objects.
        some_list_of_objects = copy.copy(self.some_list_of_objects)
        some_circular_ref = copy.copy(self.some_circular_ref)

        # Then, let&#39;s clone the object itself, using the prepared clones of the
        # nested objects.
        new = self.__class__(
            self.some_int, some_list_of_objects, some_circular_ref
        )
        new.__dict__.update(self.__dict__)

        return new

    def __deepcopy__(self, memo=None):
        &quot;&quot;&quot;
        Create a deep copy. This method will be called whenever someone calls
        `copy.deepcopy` with this object and the returned value is returned as
        the new deep copy.

        What is the use of the argument `memo`? Memo is the dictionary that is
        used by the `deepcopy` library to prevent infinite recursive copies in
        instances of circular references. Pass it to all the `deepcopy` calls
        you make in the `__deepcopy__` implementation to prevent infinite
        recursions.
        &quot;&quot;&quot;
        if memo is None:
            memo = {}

        # First, let&#39;s create copies of the nested objects.
        some_list_of_objects = copy.deepcopy(self.some_list_of_objects, memo)
        some_circular_ref = copy.deepcopy(self.some_circular_ref, memo)

        # Then, let&#39;s clone the object itself, using the prepared clones of the
        # nested objects.
        new = self.__class__(
            self.some_int, some_list_of_objects, some_circular_ref
        )
        new.__dict__ = copy.deepcopy(self.__dict__, memo)

        return new


if __name__ == &quot;__main__&quot;:

    list_of_objects = [1, {1, 2, 3}, [1, 2, 3]]
    circular_ref = SelfReferencingEntity()
    component = SomeComponent(23, list_of_objects, circular_ref)
    circular_ref.set_parent(component)

    shallow_copied_component = copy.copy(component)

    # Let&#39;s change the list in shallow_copied_component and see if it changes in
    # component.
    shallow_copied_component.some_list_of_objects.append(&quot;another object&quot;)
    if component.some_list_of_objects[-1] == &quot;another object&quot;:
        print(
            &quot;Adding elements to `shallow_copied_component`&#39;s &quot;
            &quot;some_list_of_objects adds it to `component`&#39;s &quot;
            &quot;some_list_of_objects.&quot;
        )
    else:
        print(
            &quot;Adding elements to `shallow_copied_component`&#39;s &quot;
            &quot;some_list_of_objects doesn&#39;t add it to `component`&#39;s &quot;
            &quot;some_list_of_objects.&quot;
        )

    # Let&#39;s change the set in the list of objects.
    component.some_list_of_objects[1].add(4)
    if 4 in shallow_copied_component.some_list_of_objects[1]:
        print(
            &quot;Changing objects in the `component`&#39;s some_list_of_objects &quot;
            &quot;changes that object in `shallow_copied_component`&#39;s &quot;
            &quot;some_list_of_objects.&quot;
        )
    else:
        print(
            &quot;Changing objects in the `component`&#39;s some_list_of_objects &quot;
            &quot;doesn&#39;t change that object in `shallow_copied_component`&#39;s &quot;
            &quot;some_list_of_objects.&quot;
        )

    deep_copied_component = copy.deepcopy(component)

    # Let&#39;s change the list in deep_copied_component and see if it changes in
    # component.
    deep_copied_component.some_list_of_objects.append(&quot;one more object&quot;)
    if component.some_list_of_objects[-1] == &quot;one more object&quot;:
        print(
            &quot;Adding elements to `deep_copied_component`&#39;s &quot;
            &quot;some_list_of_objects adds it to `component`&#39;s &quot;
            &quot;some_list_of_objects.&quot;
        )
    else:
        print(
            &quot;Adding elements to `deep_copied_component`&#39;s &quot;
            &quot;some_list_of_objects doesn&#39;t add it to `component`&#39;s &quot;
            &quot;some_list_of_objects.&quot;
        )

    # Let&#39;s change the set in the list of objects.
    component.some_list_of_objects[1].add(10)
    if 10 in deep_copied_component.some_list_of_objects[1]:
        print(
            &quot;Changing objects in the `component`&#39;s some_list_of_objects &quot;
            &quot;changes that object in `deep_copied_component`&#39;s &quot;
            &quot;some_list_of_objects.&quot;
        )
    else:
        print(
            &quot;Changing objects in the `component`&#39;s some_list_of_objects &quot;
            &quot;doesn&#39;t change that object in `deep_copied_component`&#39;s &quot;
            &quot;some_list_of_objects.&quot;
        )

    print(
        f&quot;id(deep_copied_component.some_circular_ref.parent): &quot;
        f&quot;{id(deep_copied_component.some_circular_ref.parent)}&quot;
    )
    print(
        f&quot;id(deep_copied_component.some_circular_ref.parent.some_circular_ref.parent): &quot;
        f&quot;{id(deep_copied_component.some_circular_ref.parent.some_circular_ref.parent)}&quot;
    )
    print(
        &quot;^^ This shows that deepcopied objects contain same reference, they &quot;
        &quot;are not cloned repeatedly.&quot;
    )</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** 실행 결과

<figure class="code">
<pre class="code" lang="output"><code>Adding elements to `shallow_copied_component`&#39;s some_list_of_objects adds it to `component`&#39;s some_list_of_objects.
Changing objects in the `component`&#39;s some_list_of_objects changes that object in `shallow_copied_component`&#39;s some_list_of_objects.
Adding elements to `deep_copied_component`&#39;s some_list_of_objects doesn&#39;t add it to `component`&#39;s some_list_of_objects.
Changing objects in the `component`&#39;s some_list_of_objects doesn&#39;t change that object in `deep_copied_component`&#39;s some_list_of_objects.
id(deep_copied_component.some_circular_ref.parent): 4429472784
id(deep_copied_component.some_circular_ref.parent.some_circular_ref.parent): 4429472784
^^ This shows that deepcopied objects contain same reference, they are not cloned repeatedly.</code></pre>
</figure>

::: next
#### 다음을 보세요

[파이썬으로 작성된 싱글턴 []{.fa
.fa-arrow-right}](../../singleton/python/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### 돌아가기

[[]{.fa .fa-arrow-left} 파이썬으로 작성된 팩토리
메서드](../../factory-method/python/example.html){.btn .btn-default
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
[![자바로 작성된
프로토타입](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "자바로 작성된 프로토타입"){.prog-lang-link}
[![PHP로 작성된
프로토타입](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "PHP로 작성된 프로토타입"){.prog-lang-link}
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
