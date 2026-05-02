::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [디자인
패턴들](../../../design-patterns.html){.type} /
[프로토타입](../../prototype.html){.type} / [Go](../../go.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![프로토타입](../../../../images/patterns/cards/prototype-mini6540.png?id=bc3046bb39ff36574c08d49839fd1c8e){srcset="/images/patterns/cards/prototype-mini-2x.png?id=b871f789a736e7efbb1cd082d2de6398 2x"}
:::

# Go로 작성된 **프로토타입** {#go로-작성된-프로토타입 .pattern-example-title-block-title}
::::

:::::::::::::: pattern-example-body
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
 [inode](#example-0--inode-go)
:::

::: en-file
 [file](#example-0--file-go)
:::

::: en-file
 [folder](#example-0--folder-go)
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

운영 체제의 파일 시스템을 기반으로 한 예시를 사용하여 프로토타입 패턴을
연구해 봅시다. 운영 체제의 파일 시스템은 재귀적입니다: 폴더에는 파일과
폴더들이 포함되며, 내부 폴더에도 파일과 폴더 등이 포함될 수 있습니다.

각 파일과 폴더는 `inode` 인터페이스로 나타낼 수 있습니다. `inode`
인터페이스도 `clone` 함수가 있습니다.

`파일` 및 `폴더` 구조체​(struct)​들은 모두 `inode`유형에 속하므로 `print`
및 `clone` 함수들을 구현합니다. 또한 `파일`과 `폴더` 모두에서 `복제`
함수를 확인하세요. 두 유형 모두에 있는 `복제` 함수는 해당 파일 또는
폴더의 복사본을 반환합니다. 복제하는 동안 이름 필드에 키워드
\'\_clone\'을 추가합니다.

#### []{#example-0--inode-go .anchor} **inode.go:** 프로토타입 인터페이스

<figure class="code">
<pre class="code" lang="go"><code>package main

type Inode interface {
    print(string)
    clone() Inode
}</code></pre>
</figure>

#### []{#example-0--file-go .anchor} **file.go:** 구상 프로토타입

<figure class="code">
<pre class="code" lang="go"><code>package main

import &quot;fmt&quot;

type File struct {
    name string
}

func (f *File) print(indentation string) {
    fmt.Println(indentation + f.name)
}

func (f *File) clone() Inode {
    return &amp;File{name: f.name + &quot;_clone&quot;}
}</code></pre>
</figure>

#### []{#example-0--folder-go .anchor} **folder.go:** 구상 프로토타입

<figure class="code">
<pre class="code" lang="go"><code>package main

import &quot;fmt&quot;

type Folder struct {
    children []Inode
    name     string
}

func (f *Folder) print(indentation string) {
    fmt.Println(indentation + f.name)
    for _, i := range f.children {
        i.print(indentation + indentation)
    }
}

func (f *Folder) clone() Inode {
    cloneFolder := &amp;Folder{name: f.name + &quot;_clone&quot;}
    var tempChildren []Inode
    for _, i := range f.children {
        copy := i.clone()
        tempChildren = append(tempChildren, copy)
    }
    cloneFolder.children = tempChildren
    return cloneFolder
}</code></pre>
</figure>

#### []{#example-0--main-go .anchor} **main.go:** 클라이언트 코드

<figure class="code">
<pre class="code" lang="go"><code>package main

import &quot;fmt&quot;

func main() {
    file1 := &amp;File{name: &quot;File1&quot;}
    file2 := &amp;File{name: &quot;File2&quot;}
    file3 := &amp;File{name: &quot;File3&quot;}

    folder1 := &amp;Folder{
        children: []Inode{file1},
        name:     &quot;Folder1&quot;,
    }

    folder2 := &amp;Folder{
        children: []Inode{folder1, file2, file3},
        name:     &quot;Folder2&quot;,
    }
    fmt.Println(&quot;\nPrinting hierarchy for Folder2&quot;)
    folder2.print(&quot;  &quot;)

    cloneFolder := folder2.clone()
    fmt.Println(&quot;\nPrinting hierarchy for clone Folder&quot;)
    cloneFolder.print(&quot;  &quot;)
}</code></pre>
</figure>

#### []{#example-0--output-txt .anchor} **output.txt:** 실행 결과

<figure class="code">
<pre class="code" lang="output"><code>Printing hierarchy for Folder2
  Folder2
    Folder1
        File1
    File2
    File3

Printing hierarchy for clone Folder
  Folder2_clone
    Folder1_clone
        File1_clone
    File2_clone
    File3_clone</code></pre>
</figure>

::: next
#### 다음을 보세요

[Go로 작성된 싱글턴 []{.fa
.fa-arrow-right}](../../singleton/go/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### 돌아가기

[[]{.fa .fa-arrow-left} Go로 작성된 팩토리
메서드](../../factory-method/go/example.html){.btn .btn-default
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
