:::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} /
[デザインパターン](../../../design-patterns.html){.type} /
[Prototype](../../prototype.html){.type} /
[Python](../../python.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Prototype](../../../../images/patterns/cards/prototype-mini6540.png?id=bc3046bb39ff36574c08d49839fd1c8e){srcset="/images/patterns/cards/prototype-mini-2x.png?id=b871f789a736e7efbb1cd082d2de6398 2x"}
:::

# **Prototype** を Python で {#prototype-を-python-で .pattern-example-title-block-title}
::::

::::::::::: pattern-example-body
::: pattern-example-brief
**Prototype** は[、]{.chpule2}[
]{.chpuri2}生成に関するデザインパターンの一つで[、]{.chpule2}[
]{.chpuri2}特定のクラスに結合することなく[、]{.chpule2}[
]{.chpuri2}オブジェクト[
]{.chpule1}[（]{.chpuri1}たとえ複雑なオブジェクトでも[）]{.chpule2}[
]{.chpuri2}のクローン作成を可能とします[。]{.chpule2}[ ]{.chpuri2}

プロトタイプのクラス全部には[、]{.chpule2}[
]{.chpuri2}共通するインターフェースが必要です[。]{.chpule2}[
]{.chpuri2}これにより[、]{.chpule2}[
]{.chpuri2}具象クラスが不明であってもオブジェクトを複製することが可能となります[。]{.chpule2}[
]{.chpuri2}プロトタイプ・オブジェクトが[、]{.chpule2}[
]{.chpuri2}完全なコピーを生成できるのは[、]{.chpule2}[
]{.chpuri2}同じクラスのオブジェクト同士が非公開フィールドを互いにアクセスできるからです[。]{.chpule2}[
]{.chpuri2}
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Prototype の詳細 ](../../prototype.html){.btn .btn-lg .btn-primary}
:::

:::::::: {.sidebar-navigation .with-scroll}
::: en-title
ナビゲーション
:::

::: en-intro
 [はじめに](#)
:::

::: en-intro
 [概念的な例](#example-0)
:::

::: en-file
 [main](#example-0--main-py)
:::

::: en-file
 [Output](#example-0--Output-txt)
:::
::::::::

**複雑度[：]{.chpule2}[ ]{.chpuri2}**[]{.fa-stars}

**人気度[：]{.chpule2}[ ]{.chpuri2}**[]{.fa-stars}

**使用例[：]{.chpule2}[ ]{.chpuri2}** Prototype
パターンは[、]{.chpule2}[ ]{.chpuri2}Python では[、]{.chpule2}[
]{.chpuri2}`copy` モジュールを使って[、]{.chpule2}[
]{.chpuri2}初めから利用可能です[。]{.chpule2}[ ]{.chpuri2}

**見つけ方[：]{.chpule2}[ ]{.chpuri2}**このパターンは[、]{.chpule2}[
]{.chpuri2}`clone` や `copy`
といったメソッドで容易に識別可能です[。]{.chpule2}[ ]{.chpuri2}
:::::::::::

[]{#example-0}

## 概念的な例 {#example-0-title}

この例は[、]{.chpule2}[ ]{.chpuri2}**Prototype**
デザインパターンの構造を説明するためのものです[。]{.chpule2}[
]{.chpuri2}以下の質問に答えることを目的としています[：]{.chpule2}[
]{.chpuri2}

-   どういうクラスからできているか[？]{.chpule2}[ ]{.chpuri2}
-   それぞれのクラスの役割は[？]{.chpule2}[ ]{.chpuri2}
-   パターンの要素同士はどう関係しているのか[？]{.chpule2}[ ]{.chpuri2}

#### []{#example-0--main-py .anchor} **main.py:** 概念的な例

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

#### []{#example-0--Output-txt .anchor} **Output.txt:** 実行結果

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
#### 次を読む

[Singleton を Python で []{.fa
.fa-arrow-right}](../../singleton/python/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### 戻る

[[]{.fa .fa-arrow-left} Factory Method を Python
で](../../factory-method/python/example.html){.btn .btn-default
rel="prev"}
:::

## 他言語での **Prototype**

[![Prototype を C#
で](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Prototype を C# で"){.prog-lang-link}
[![Prototype を C++
で](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Prototype を C++ で"){.prog-lang-link}
[![Prototype を Go
で](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Prototype を Go で"){.prog-lang-link}
[![Prototype を Java
で](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Prototype を Java で"){.prog-lang-link}
[![Prototype を PHP
で](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Prototype を PHP で"){.prog-lang-link}
[![Prototype を Ruby
で](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Prototype を Ruby で"){.prog-lang-link}
[![Prototype を Rust
で](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Prototype を Rust で"){.prog-lang-link}
[![Prototype を Swift
で](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Prototype を Swift で"){.prog-lang-link}
[![Prototype を TypeScript
で](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Prototype を TypeScript で"){.prog-lang-link}
:::::::::::::::::

::::::: {.prom .banner-sidebar .banner .banner-striped .banner-examples .banner-removable .banner-removable-patterns data-id="DP: Examples-IDE" creative-id="examples-sidebar-ja" position="sidebar"}
:::::: {.banner-inner style="font-size: 14px; line-height: 21px;"}
::: banner-examples-img
[![](../../../../images/patterns/banners/examples-ide91ca.png?id=3115b4b548fb96b75974e2de8f4f49bc){width="215"
height="158" loading="lazy"
srcset="/images/patterns/banners/examples-ide-2x.png?id=93c007a6d157b97c28eb213f59d716ae 2x"}](../../book.html)
:::

::: banner-examples-fade
[ コード例の書庫](../../book.html){.btn .btn-secondary .btn-block}
:::

::: {.banner-examples-text style="margin-top: 1rem;"}
**直撃！デザインパターン**を電子書籍で購入し、IDE
で開くことができる数多くの詳細な例を含む書庫にアクセス。

[ 詳細](../../book.html){.btn .btn-secondary .btn-block}
:::
::::::
:::::::
:::::::::::::::::::::::
::::::::::::::::::::::::
