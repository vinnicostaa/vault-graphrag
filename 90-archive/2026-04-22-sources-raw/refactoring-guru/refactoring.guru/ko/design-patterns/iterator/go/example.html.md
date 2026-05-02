::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [디자인
패턴들](../../../design-patterns.html){.type} /
[반복자](../../iterator.html){.type} / [Go](../../go.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![반복자](../../../../images/patterns/cards/iterator-minie2c2.png?id=76c28bb48f997b36965983dd2b41f02e){srcset="/images/patterns/cards/iterator-mini-2x.png?id=7d28f66a487066774a743cfceaa40ea4 2x"}
:::

# Go로 작성된 **반복자** {#go로-작성된-반복자 .pattern-example-title-block-title}
::::

:::::::::::::::: pattern-example-body
::: pattern-example-brief
**반복자**는 복잡한 데이터 구조의 내부 세부 정보를 노출하지 않고 해당
구조를 차례대로 순회할 수 있도록 하는 행동 디자인 패턴입니다.

반복자 덕분에 클라이언트들은 단일 반복기 인터페이스를 사용하여 유사한
방식으로 다른 컬렉션들의 요소들을 탐색할 수 있습니다.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ 반복자에 대하여 더 자세히 알아보세요 ](../../iterator.html){.btn
.btn-lg .btn-primary}
:::

::::::::::::: {.sidebar-navigation .with-scroll}
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
 [collection](#example-0--collection-go)
:::

::: en-file
 [user­Collection](#example-0--userCollection-go)
:::

::: en-file
 [iterator](#example-0--iterator-go)
:::

::: en-file
 [user­Iterator](#example-0--userIterator-go)
:::

::: en-file
 [user](#example-0--user-go)
:::

::: en-file
 [main](#example-0--main-go)
:::

::: en-file
 [output](#example-0--output-txt)
:::
:::::::::::::
::::::::::::::::

[]{#example-0}

## 개념적인 예시 {#example-0-title}

반복자 패턴의 주요 아이디어는 컬렉션의 순회 논리를 반복자라는 다른
객체로 추출하는 것입니다. 이 반복자는 컬렉션의 형식과 관계없이 해당
컬렉션을 순회하는 일반적인 메서드를 제공합니다.

#### []{#example-0--collection-go .anchor} **collection.go:** 컬렉션

<figure class="code">
<pre class="code" lang="go"><code>package main

type Collection interface {
    createIterator() Iterator
}</code></pre>
</figure>

#### []{#example-0--userCollection-go .anchor} **userCollection.go:** 구상 컬렉션

<figure class="code">
<pre class="code" lang="go"><code>package main

type UserCollection struct {
    users []*User
}

func (u *UserCollection) createIterator() Iterator {
    return &amp;UserIterator{
        users: u.users,
    }
}</code></pre>
</figure>

#### []{#example-0--iterator-go .anchor} **iterator.go:** 반복자

<figure class="code">
<pre class="code" lang="go"><code>package main

type Iterator interface {
    hasNext() bool
    getNext() *User
}</code></pre>
</figure>

#### []{#example-0--userIterator-go .anchor} **userIterator.go:** 구상 반복자

<figure class="code">
<pre class="code" lang="go"><code>package main

type UserIterator struct {
    index int
    users []*User
}

func (u *UserIterator) hasNext() bool {
    if u.index &lt; len(u.users) {
        return true
    }
    return false

}
func (u *UserIterator) getNext() *User {
    if u.hasNext() {
        user := u.users[u.index]
        u.index++
        return user
    }
    return nil
}</code></pre>
</figure>

#### []{#example-0--user-go .anchor} **user.go:** 클라이언트 코드

<figure class="code">
<pre class="code" lang="go"><code>package main

type User struct {
    name string
    age  int
}</code></pre>
</figure>

#### []{#example-0--main-go .anchor} **main.go:** 클라이언트 코드

<figure class="code">
<pre class="code" lang="go"><code>package main

import &quot;fmt&quot;

func main() {

    user1 := &amp;User{
        name: &quot;a&quot;,
        age:  30,
    }
    user2 := &amp;User{
        name: &quot;b&quot;,
        age:  20,
    }

    userCollection := &amp;UserCollection{
        users: []*User{user1, user2},
    }

    iterator := userCollection.createIterator()

    for iterator.hasNext() {
        user := iterator.getNext()
        fmt.Printf(&quot;User is %+v\n&quot;, user)
    }
}</code></pre>
</figure>

#### []{#example-0--output-txt .anchor} **output.txt:** 실행 결과

<figure class="code">
<pre class="code" lang="output"><code>User is &amp;{name:a age:30}
User is &amp;{name:b age:20}</code></pre>
</figure>

::: next
#### 다음을 보세요

[Go로 작성된 중재자 []{.fa
.fa-arrow-right}](../../mediator/go/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### 돌아가기

[[]{.fa .fa-arrow-left} Go로 작성된
커맨드](../../command/go/example.html){.btn .btn-default rel="prev"}
:::

## 다른 언어로 작성된 **반복자**

[![C#으로 작성된
반복자](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "C#으로 작성된 반복자"){.prog-lang-link}
[![C++로 작성된
반복자](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "C++로 작성된 반복자"){.prog-lang-link}
[![자바로 작성된
반복자](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "자바로 작성된 반복자"){.prog-lang-link}
[![PHP로 작성된
반복자](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "PHP로 작성된 반복자"){.prog-lang-link}
[![파이썬으로 작성된
반복자](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "파이썬으로 작성된 반복자"){.prog-lang-link}
[![루비로 작성된
반복자](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "루비로 작성된 반복자"){.prog-lang-link}
[![러스트로 작성된
반복자](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "러스트로 작성된 반복자"){.prog-lang-link}
[![스위프트로 작성된
반복자](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "스위프트로 작성된 반복자"){.prog-lang-link}
[![타입스크립트로 작성된
반복자](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "타입스크립트로 작성된 반복자"){.prog-lang-link}
::::::::::::::::::::::

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
::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::
