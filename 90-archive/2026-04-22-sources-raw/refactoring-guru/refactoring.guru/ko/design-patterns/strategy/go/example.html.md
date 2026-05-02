::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [디자인
패턴들](../../../design-patterns.html){.type} /
[전략](../../strategy.html){.type} / [Go](../../go.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![전략](../../../../images/patterns/cards/strategy-mini28f6.png?id=d38abee4fb6f2aed909d262bdadca936){srcset="/images/patterns/cards/strategy-mini-2x.png?id=f4e6608561f8e5d18be6927d4620ad29 2x"}
:::

# Go로 작성된 **전략** {#go로-작성된-전략 .pattern-example-title-block-title}
::::

:::::::::::::::: pattern-example-body
::: pattern-example-brief
**전략**은 행동들의 객체들을 객체들로 변환하며 이들이 원래 콘텍스트 객체
내에서 상호 교환이 가능하게 만드는 행동 디자인 패턴입니다.

원래 객체는 콘텍스트라고 불리며 전략 객체에 대한 참조를 포함합니다.
콘텍스트는 행동의 실행을 연결된 전략 객체에 위임합니다. 콘텍스트가
작업을 수행하는 방식을 변경하기 위하여 다른 객체들은 현재 연결된 전략
객체를 다른 전략 객체와 대체할 수 있습니다.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ 전략에 대하여 더 자세히 알아보세요 ](../../strategy.html){.btn .btn-lg
.btn-primary}
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
 [eviction­Algo](#example-0--evictionAlgo-go)
:::

::: en-file
 [fifo](#example-0--fifo-go)
:::

::: en-file
 [lru](#example-0--lru-go)
:::

::: en-file
 [lfu](#example-0--lfu-go)
:::

::: en-file
 [cache](#example-0--cache-go)
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

당신이 인메모리 캐싱을 생성하고 있다고 가정해 봅시다. 이것은 메모리 내에
있으므로 크기가 제한되며 최대 크기에 도달할 때마다 공간을 확보하기 위해
일부 항목들을 제거해야 합니다. 위 사건은 여러 알고리즘을 통해 발생할 수
있으며 다음은 일부 인기 있는 알고리즘들입니다:

-   LRU 알고리즘: LRU는 Least Recently Used의 약자이며 해당 알고리즘은
    가장 최근에 사용된 항목을 제거합니다.
-   FIFO 알고리즘: FIFO는 First In First Out의 약자이며 해당 알고리즘은
    가장 처음에 생성된 항목을 제거합니다.
-   LFU 알고리즘: LFU는 Least Frequently Used의 약자이며 해당 알고리즘은
    가장 적게 사용된 항목을 제거합니다.

이제 우리가 해결해야 할 문제는 런타임에 알고리즘을 변경할 수 있도록 캐시
클래스를 이러한 알고리즘에서 분리하는 방법입니다. 또한 새 알고리즘이
추가될 때 캐시 클래스가 변경되지 않아야 합니다.

이러한 상황에서 전략 패턴이 유용해집니다. 전략 패턴은 각 알고리즘에 해당
알고리즘의 클래스가 있는 알고리즘 패밀리를 만들 것을 제안합니다. 이러한
각 클래스는 같은 인터페이스를 따르므로 패밀리 내에서 알고리즘을 교환할
수 있습니다. 일반적인 인터페이스 이름이 `eviction­Algo`라고 가정해
보겠습니다.

이제 우리의 주 캐시 클래스는 evictionAlgo 인터페이스를 포함합니다. 모든
유형의 eviction 알고리즘들을 자체적으로 구현하는 대신 해당 캐시 클래스는
실행을 evictionAlgo 인터페이스에 위임할것입니다. evictionAlgo는
인터페이스이기 때문에 우리는 캐시 클래스를 변경하지 않고 런타임에
알고리즘을 LRU, FIFO, 또는 LFU로 변경할 수 있습니다.

#### []{#example-0--evictionAlgo-go .anchor} **evictionAlgo.go:** 전략 인터페이스

<figure class="code">
<pre class="code" lang="go"><code>package main

type EvictionAlgo interface {
    evict(c *Cache)
}</code></pre>
</figure>

#### []{#example-0--fifo-go .anchor} **fifo.go:** 구상 전략

<figure class="code">
<pre class="code" lang="go"><code>package main

import &quot;fmt&quot;

type Fifo struct {
}

func (l *Fifo) evict(c *Cache) {
    fmt.Println(&quot;Evicting by fifo strtegy&quot;)
}</code></pre>
</figure>

#### []{#example-0--lru-go .anchor} **lru.go:** 구상 전략

<figure class="code">
<pre class="code" lang="go"><code>package main

import &quot;fmt&quot;

type Lru struct {
}

func (l *Lru) evict(c *Cache) {
    fmt.Println(&quot;Evicting by lru strtegy&quot;)
}</code></pre>
</figure>

#### []{#example-0--lfu-go .anchor} **lfu.go:** 구상 전략

<figure class="code">
<pre class="code" lang="go"><code>package main

import &quot;fmt&quot;

type Lfu struct {
}

func (l *Lfu) evict(c *Cache) {
    fmt.Println(&quot;Evicting by lfu strtegy&quot;)
}</code></pre>
</figure>

#### []{#example-0--cache-go .anchor} **cache.go:** 콘텍스트

<figure class="code">
<pre class="code" lang="go"><code>package main

type Cache struct {
    storage      map[string]string
    evictionAlgo EvictionAlgo
    capacity     int
    maxCapacity  int
}

func initCache(e EvictionAlgo) *Cache {
    storage := make(map[string]string)
    return &amp;Cache{
        storage:      storage,
        evictionAlgo: e,
        capacity:     0,
        maxCapacity:  2,
    }
}

func (c *Cache) setEvictionAlgo(e EvictionAlgo) {
    c.evictionAlgo = e
}

func (c *Cache) add(key, value string) {
    if c.capacity == c.maxCapacity {
        c.evict()
    }
    c.capacity++
    c.storage[key] = value
}

func (c *Cache) get(key string) {
    delete(c.storage, key)
}

func (c *Cache) evict() {
    c.evictionAlgo.evict(c)
    c.capacity--
}</code></pre>
</figure>

#### []{#example-0--main-go .anchor} **main.go:** 클라이언트 코드

<figure class="code">
<pre class="code" lang="go"><code>package main

func main() {
    lfu := &amp;Lfu{}
    cache := initCache(lfu)

    cache.add(&quot;a&quot;, &quot;1&quot;)
    cache.add(&quot;b&quot;, &quot;2&quot;)

    cache.add(&quot;c&quot;, &quot;3&quot;)

    lru := &amp;Lru{}
    cache.setEvictionAlgo(lru)

    cache.add(&quot;d&quot;, &quot;4&quot;)

    fifo := &amp;Fifo{}
    cache.setEvictionAlgo(fifo)

    cache.add(&quot;e&quot;, &quot;5&quot;)

}</code></pre>
</figure>

#### []{#example-0--output-txt .anchor} **output.txt:** 실행 결과

<figure class="code">
<pre class="code" lang="output"><code>Evicting by lfu strtegy
Evicting by lru strtegy
Evicting by fifo strtegy</code></pre>
</figure>

::: next
#### 다음을 보세요

[Go로 작성된 템플릿 메서드 []{.fa
.fa-arrow-right}](../../template-method/go/example.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### 돌아가기

[[]{.fa .fa-arrow-left} Go로 작성된
상태](../../state/go/example.html){.btn .btn-default rel="prev"}
:::

## 다른 언어로 작성된 **전략**

[![C#으로 작성된
전략](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "C#으로 작성된 전략"){.prog-lang-link}
[![C++로 작성된
전략](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "C++로 작성된 전략"){.prog-lang-link}
[![자바로 작성된
전략](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "자바로 작성된 전략"){.prog-lang-link}
[![PHP로 작성된
전략](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "PHP로 작성된 전략"){.prog-lang-link}
[![파이썬으로 작성된
전략](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "파이썬으로 작성된 전략"){.prog-lang-link}
[![루비로 작성된
전략](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "루비로 작성된 전략"){.prog-lang-link}
[![러스트로 작성된
전략](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "러스트로 작성된 전략"){.prog-lang-link}
[![스위프트로 작성된
전략](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "스위프트로 작성된 전략"){.prog-lang-link}
[![타입스크립트로 작성된
전략](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "타입스크립트로 작성된 전략"){.prog-lang-link}
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
