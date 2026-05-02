::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Патерни
проектування](../../../design-patterns.html){.type} /
[Стратегія](../../strategy.html){.type} / [Go](../../go.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Стратегія](../../../../images/patterns/cards/strategy-mini28f6.png?id=d38abee4fb6f2aed909d262bdadca936){srcset="/images/patterns/cards/strategy-mini-2x.png?id=f4e6608561f8e5d18be6927d4620ad29 2x"}
:::

# **Стратегія** на Go {#стратегія-на-go .pattern-example-title-block-title}
::::

:::::::::::::::: pattern-example-body
::: pattern-example-brief
**Стратегія** --- це поведінковий патерн, який виносить набір алгоритмів
у власні класи і робить їх взаємозамінними.

Інші об'єкти містять посилання на об'єкт-стратегію та делегують їй
роботу. Програма може підмінити цей об'єкт іншим, якщо потрібен інший
спосіб вирішення завдання.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Детальніше про Стратегія ](../../strategy.html){.btn .btn-lg
.btn-primary}
:::

::::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
Навігація
:::

::: en-intro
 [Інтро](#)
:::

::: en-intro
 [Концептуальний приклад](#example-0)
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

## Концептуальний приклад {#example-0-title}

Уявіть, що ви розробляєте «In-Memory-Cache». Оскільки він розташований
всередині пам'яті, його розмір обмежений. Щойно він вщент заповниться,
якісь записи доведеться прибрати, аби звільнити простір. Цю функцію
можна реалізувати завдяки декільком алгоритмам, найпопулярніші серед
них:

-   Найбільш давно використовувалися (Least Recently Used --- LRU):
    прибирає запис, який використовувався найбільш давно.
-   «Першим прийшов, першим пішов» (First In, First Out --- FIFO):
    прибирає запис, який був створений раніше за інші.
-   Найменш часто використовувалися (Least Frequently Used --- LFU):
    прибирає запис, який використовувався найменш часто.

Проблема полягає в тому, щоб відокремити кеш від цих алгоритмів для
можливості їх заміни «на ходу». Крім цього, клас кешу мусить не
змінюватися при додаванні нового алгоритму.

У такій ситуації нам допоможе патерн Стратегія. Він передбачає створення
сімейства алгоритмів, кожен з яких має свій клас. Всі класи застосовують
однаковий інтерфейс, що робить алгоритми взаємозамінними всередині цього
сімейства. Назвемо цей загальний інтерфейс `evictionAlgo`.

Тепер основний клас нашого кеша включатиме у себе `evictionAlgo`.
Замість прямої реалізації всіх типів алгоритмів витіснення всередині
самого себе, наш клас передаватиме їх в `evictionAlgo`. Оскільки це
інтерфейс, ми можемо безпосередньо під час виконання програми змінювати
алгоритм на LRU, FIFO, LFU без змін у класі кеша.

#### []{#example-0--evictionAlgo-go .anchor} **evictionAlgo.go:** Інтерфейс стратегії

<figure class="code">
<pre class="code" lang="go"><code>package main

type EvictionAlgo interface {
    evict(c *Cache)
}</code></pre>
</figure>

#### []{#example-0--fifo-go .anchor} **fifo.go:** Конкретна стратегія

<figure class="code">
<pre class="code" lang="go"><code>package main

import &quot;fmt&quot;

type Fifo struct {
}

func (l *Fifo) evict(c *Cache) {
    fmt.Println(&quot;Evicting by fifo strtegy&quot;)
}</code></pre>
</figure>

#### []{#example-0--lru-go .anchor} **lru.go:** Конкретна стратегія

<figure class="code">
<pre class="code" lang="go"><code>package main

import &quot;fmt&quot;

type Lru struct {
}

func (l *Lru) evict(c *Cache) {
    fmt.Println(&quot;Evicting by lru strtegy&quot;)
}</code></pre>
</figure>

#### []{#example-0--lfu-go .anchor} **lfu.go:** Конкретна стратегія

<figure class="code">
<pre class="code" lang="go"><code>package main

import &quot;fmt&quot;

type Lfu struct {
}

func (l *Lfu) evict(c *Cache) {
    fmt.Println(&quot;Evicting by lfu strtegy&quot;)
}</code></pre>
</figure>

#### []{#example-0--cache-go .anchor} **cache.go:** Контекст

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

#### []{#example-0--main-go .anchor} **main.go:** Клієнтський код

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

#### []{#example-0--output-txt .anchor} **output.txt:** Результат виконання

<figure class="code">
<pre class="code" lang="output"><code>Evicting by lfu strtegy
Evicting by lru strtegy
Evicting by fifo strtegy</code></pre>
</figure>

::: next
#### Читаємо далі

[Шаблонний метод на Go []{.fa
.fa-arrow-right}](../../template-method/go/example.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Повернутись назад

[[]{.fa .fa-arrow-left} Стан на Go](../../state/go/example.html){.btn
.btn-default rel="prev"}
:::

## **Стратегія** іншими мовами програмування

[![Стратегія на
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Стратегія на C#"){.prog-lang-link}
[![Стратегія на
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Стратегія на C++"){.prog-lang-link}
[![Стратегія на
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Стратегія на Java"){.prog-lang-link}
[![Стратегія на
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Стратегія на PHP"){.prog-lang-link}
[![Стратегія на
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Стратегія на Python"){.prog-lang-link}
[![Стратегія на
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Стратегія на Ruby"){.prog-lang-link}
[![Стратегія на
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Стратегія на Rust"){.prog-lang-link}
[![Стратегія на
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Стратегія на Swift"){.prog-lang-link}
[![Стратегія на
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Стратегія на TypeScript"){.prog-lang-link}
::::::::::::::::::::::

::::::: {.prom .banner-sidebar .banner .banner-striped .banner-examples .banner-removable .banner-removable-patterns data-id="DP: Examples-IDE" creative-id="examples-sidebar-uk" position="sidebar"}
:::::: {.banner-inner style="font-size: 14px; line-height: 21px;"}
::: banner-examples-img
[![](../../../../images/patterns/banners/examples-ide91ca.png?id=3115b4b548fb96b75974e2de8f4f49bc){width="215"
height="158" loading="lazy"
srcset="/images/patterns/banners/examples-ide-2x.png?id=93c007a6d157b97c28eb213f59d716ae 2x"}](../../book.html)
:::

::: banner-examples-fade
[ Архів з прикладами](../../book.html){.btn .btn-secondary .btn-block}
:::

::: {.banner-examples-text style="margin-top: 1rem;"}
Придбай книгу **Занурення в Патерни** та отримай архів з десятками
детальних прикладів, які можна відкривати прямо в IDE.

[ Дізнатися більше...](../../book.html){.btn .btn-secondary .btn-block}
:::
::::::
:::::::
::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::
