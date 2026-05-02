::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [디자인
패턴들](../../../design-patterns.html){.type} /
[복합체](../../composite.html){.type} / [Go](../../go.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![복합체](../../../../images/patterns/cards/composite-minib543.png?id=a369d98d18b417f255d04568fd0131b8){srcset="/images/patterns/cards/composite-mini-2x.png?id=3f7f811fefeb0b64f6774746eb42af09 2x"}
:::

# Go로 작성된 **복합체** {#go로-작성된-복합체 .pattern-example-title-block-title}
::::

:::::::::::::: pattern-example-body
::: pattern-example-brief
**복합체** 패턴은 객체들을 트리 구조들로 구성한 후, 이러한 구조들을 개별
객체들처럼 다룰 수 있도록 하는 구조 패턴입니다.

복합체는 트리 구조를 생성해야 하는 대부분 문제에 대해 인기 있는
해결책입니다. 전체 트리 구조에 대해 재귀적으로 메서드들을 실행하고
결과를 요약하는 기능은 복합체의 훌륭한 기능 중 하나입니다.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ 복합체에 대하여 더 자세히 알아보세요 ](../../composite.html){.btn
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
 [component](#example-0--component-go)
:::

::: en-file
 [folder](#example-0--folder-go)
:::

::: en-file
 [file](#example-0--file-go)
:::

::: en-file
 [main](#example-0--main-go)
:::

::: en-file
 [output](#example-0--output-txt)
:::
:::::::::::
::::::::::::::

[]{#example-0}

## 개념적인 예시 {#example-0-title}

운영 체제의 파일 시스템을 예시로 들어 복합체 패턴을 이해해 봅시다.
예시의 파일 시스템에는 파일과 폴더라는 두 가지 유형의 객체가 있으며
때로는 파일과 폴더를 같은 방식으로 취급해야 하는 경우가 있는데 이럴 때
복합체 패턴이 유용합니다.

파일 시스템에서 특정 키워드에 대한 검색을 실행해야 한다고 상상해 보세요.
이 검색 작업은 파일과 폴더 모두에 적용됩니다. 파일의 경우 파일 내용만
살펴보나 폴더의 경우 검색된 키워드를 찾기 위해 해당 폴더의 모든 파일을
탐색합니다.

#### []{#example-0--component-go .anchor} **component.go:** 컴포넌트 인터페이스

<figure class="code">
<pre class="code" lang="go"><code>package main

type Component interface {
    search(string)
}</code></pre>
</figure>

#### []{#example-0--folder-go .anchor} **folder.go:** 복합체

<figure class="code">
<pre class="code" lang="go"><code>package main

import &quot;fmt&quot;

type Folder struct {
    components []Component
    name       string
}

func (f *Folder) search(keyword string) {
    fmt.Printf(&quot;Serching recursively for keyword %s in folder %s\n&quot;, keyword, f.name)
    for _, composite := range f.components {
        composite.search(keyword)
    }
}

func (f *Folder) add(c Component) {
    f.components = append(f.components, c)
}</code></pre>
</figure>

#### []{#example-0--file-go .anchor} **file.go:** 잎

<figure class="code">
<pre class="code" lang="go"><code>package main

import &quot;fmt&quot;

type File struct {
    name string
}

func (f *File) search(keyword string) {
    fmt.Printf(&quot;Searching for keyword %s in file %s\n&quot;, keyword, f.name)
}

func (f *File) getName() string {
    return f.name
}</code></pre>
</figure>

#### []{#example-0--main-go .anchor} **main.go:** 클라이언트 코드

<figure class="code">
<pre class="code" lang="go"><code>package main

func main() {
    file1 := &amp;File{name: &quot;File1&quot;}
    file2 := &amp;File{name: &quot;File2&quot;}
    file3 := &amp;File{name: &quot;File3&quot;}

    folder1 := &amp;Folder{
        name: &quot;Folder1&quot;,
    }

    folder1.add(file1)

    folder2 := &amp;Folder{
        name: &quot;Folder2&quot;,
    }
    folder2.add(file2)
    folder2.add(file3)
    folder2.add(folder1)

    folder2.search(&quot;rose&quot;)
}</code></pre>
</figure>

#### []{#example-0--output-txt .anchor} **output.txt:** 실행 결과

<figure class="code">
<pre class="code" lang="output"><code>Serching recursively for keyword rose in folder Folder2
Searching for keyword rose in file File2
Searching for keyword rose in file File3
Serching recursively for keyword rose in folder Folder1
Searching for keyword rose in file File1</code></pre>
</figure>

::: next
#### 다음을 보세요

[Go로 작성된 데코레이터 []{.fa
.fa-arrow-right}](../../decorator/go/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### 돌아가기

[[]{.fa .fa-arrow-left} Go로 작성된
브리지](../../bridge/go/example.html){.btn .btn-default rel="prev"}
:::

## 다른 언어로 작성된 **복합체**

[![C#으로 작성된
복합체](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "C#으로 작성된 복합체"){.prog-lang-link}
[![C++로 작성된
복합체](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "C++로 작성된 복합체"){.prog-lang-link}
[![자바로 작성된
복합체](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "자바로 작성된 복합체"){.prog-lang-link}
[![PHP로 작성된
복합체](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "PHP로 작성된 복합체"){.prog-lang-link}
[![파이썬으로 작성된
복합체](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "파이썬으로 작성된 복합체"){.prog-lang-link}
[![루비로 작성된
복합체](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "루비로 작성된 복합체"){.prog-lang-link}
[![러스트로 작성된
복합체](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "러스트로 작성된 복합체"){.prog-lang-link}
[![스위프트로 작성된
복합체](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "스위프트로 작성된 복합체"){.prog-lang-link}
[![타입스크립트로 작성된
복합체](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "타입스크립트로 작성된 복합체"){.prog-lang-link}
::::::::::::::::::::

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
::::::::::::::::::::::::::
:::::::::::::::::::::::::::
