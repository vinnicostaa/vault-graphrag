:::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Патерни
проектування](../../../design-patterns.html){.type} /
[Одинак](../../singleton.html){.type} / [Go](../../go.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Одинак](../../../../images/patterns/cards/singleton-minif559.png?id=914e1565dfdf15f240e766163bd303ec){srcset="/images/patterns/cards/singleton-mini-2x.png?id=4a7348083d7719d02709a5c9ff878b9f 2x"}
:::

# **Одинак** на Go {#одинак-на-go .pattern-example-title-block-title}
::::

:::::::::::::::: pattern-example-body
::: pattern-example-brief
**Одинак** --- це породжуючий патерн, який гарантує існування тільки
одного об'єкта певного класу, а також дозволяє дістатися цього об'єкта з
будь-якого місця програми.

Одинак має такі ж переваги та недоліки, що і глобальні змінні. Його
неймовірно зручно використовувати, але він порушує модульність вашого
коду.

Ви не зможете просто взяти і використовувати клас, залежний від одинака,
в іншій програмі. Для цього доведеться емулювати там присутність
одинака. Найчастіше ця проблема проявляється при написанні юніт-тестів.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Детальніше про Одинак ](../../singleton.html){.btn .btn-lg
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
 [single](#example-0--single-go)
:::

::: en-file
 [main](#example-0--main-go)
:::

::: en-file
 [output](#example-0--output-txt)
:::

::: en-intro
 [Інший приклад](#example-1)
:::

::: en-file
 [sync­Once](#example-1--syncOnce-go)
:::

::: en-file
 [main](#example-1--main-go)
:::

::: en-file
 [output](#example-1--output-txt)
:::
:::::::::::::
::::::::::::::::

::: {#nav-tab .nav .nav-tabs .nav-justified role="tablist"}
[Концептуальний приклад](#example-0){#example-0-tab .nav-item .nav-link
.active toggle="tab" role="tab" aria-controls="example-0"
aria-selected="true"} [Інший приклад](#example-1){#example-1-tab
.nav-item .nav-link toggle="tab" role="tab" aria-controls="example-1"
aria-selected="true"}
:::

::::: {#nav-tabContent .pattern-example-tab-content .tab-content}
::: {#example-0 .tab-pane .show .active role="tabpanel" aria-labelledby="example-0-tab" style="padding-top: 24px"}
## Концептуальний приклад {#example-0-title}

Зазвичай, екземпляр одинака створюється під час початкової ініціалізації
структури. Для цього, ми визначаємо для структури метод `getInstance`.
Цей метод буде створювати і повертати екземпляри одинака. Після
створення першого екземпляра, під час кожного виклику методу
`getInstance` буде повертатися саме він.

А що стосовно потоків goroutine? Структура одинака повинна повертати
один і той же екземпляр у випадках, коли різні потоки намагаються
отримати доступ до цього екземпляра. З цієї причини можна легко
помилитися і неправильно реалізувати патерн Одинак. Приклад нижче
показує, як правильно створити Одинака.

Деякі важливі деталі, про які потрібно пам'ятати:

-   На початку потрібна `nil`-перевірка, за допомогою неї ми
    переконуємося, що перший примірник `singleInstance` --- порожній.
    Завдяки цьому ми можемо уникнути ресурсномістких операцій блокування
    під час кожного виклику `getInstance`. Якщо ця перевірка не
    пройдена, тоді поле `singleInstance` вже заповнено.

-   Структура `singleInstance` створюється всередині блокування.

-   Після блокування використовується ще одна `nil`-перевірка. У
    випадках, коли першу перевірку проходить понад одного потоку, друга
    забезпечує створення екземпляра одинака лише одним потоком. Інакше
    всі потоки створювали б свої екземпляри структури одинака.

#### []{#example-0--single-go .anchor} **single.go:** Одинак

<figure class="code">
<pre class="code" lang="go"><code>package main

import (
    &quot;fmt&quot;
    &quot;sync&quot;
)

var lock = &amp;sync.Mutex{}

type single struct {
}

var singleInstance *single

func getInstance() *single {
    if singleInstance == nil {
        lock.Lock()
        defer lock.Unlock()
        if singleInstance == nil {
            fmt.Println(&quot;Creating single instance now.&quot;)
            singleInstance = &amp;single{}
        } else {
            fmt.Println(&quot;Single instance already created.&quot;)
        }
    } else {
        fmt.Println(&quot;Single instance already created.&quot;)
    }

    return singleInstance
}</code></pre>
</figure>

#### []{#example-0--main-go .anchor} **main.go:** Клієнтський код

<figure class="code">
<pre class="code" lang="go"><code>package main

import (
    &quot;fmt&quot;
)

func main() {

    for i := 0; i &lt; 30; i++ {
        go getInstance()
    }

    // Scanln is similar to Scan, but stops scanning at a newline and
    // after the final item there must be a newline or EOF.
    fmt.Scanln()
}</code></pre>
</figure>

#### []{#example-0--output-txt .anchor} **output.txt:** Результат виконання

<figure class="code">
<pre class="code" lang="output"><code>Creating single instance now.
Single instance already created.
Single instance already created.
Single instance already created.
Single instance already created.
Single instance already created.
Single instance already created.
Single instance already created.
Single instance already created.
Single instance already created.
Single instance already created.
Single instance already created.
Single instance already created.
Single instance already created.
Single instance already created.
Single instance already created.
Single instance already created.
Single instance already created.
Single instance already created.
Single instance already created.
Single instance already created.
Single instance already created.
Single instance already created.
Single instance already created.
Single instance already created.
Single instance already created.
Single instance already created.
Single instance already created.
Single instance already created.
Single instance already created.</code></pre>
</figure>
:::

::: {#example-1 .tab-pane role="tabpanel" aria-labelledby="example-1-tab" style="padding-top: 24px"}
## Інший приклад {#example-1-title}

Є і інші методи створення екземпляра одинака в Go:

1.  Функція `init`

Ми можемо створювати екземпляр одинака всередині функції `init`. Це
можливо тільки у тих випадках, коли рання ініціалізація екземпляра не є
проблемою. Функція `init` викликається лише один раз для кожного файлу в
пакеті, тому ми можемо бути впевнені в тому, що буде створений екземпляр
буде єдиним.

2.  `sync.Once`

`Sync.Once` виконає операцію лише один раз. Дивіться код нижче:

#### []{#example-1--syncOnce-go .anchor} **syncOnce.go:** Одинак

<figure class="code">
<pre class="code" lang="go"><code>package main

import (
    &quot;fmt&quot;
    &quot;sync&quot;
)

var once sync.Once

type single struct {
}

var singleInstance *single

func getInstance() *single {
    if singleInstance == nil {
        once.Do(
            func() {
                fmt.Println(&quot;Creating single instance now.&quot;)
                singleInstance = &amp;single{}
            })
    } else {
        fmt.Println(&quot;Single instance already created.&quot;)
    }

    return singleInstance
}</code></pre>
</figure>

#### []{#example-1--main-go .anchor} **main.go:** Клієнтський код

<figure class="code">
<pre class="code" lang="go"><code>package main

import (
    &quot;fmt&quot;
)

func main() {

    for i := 0; i &lt; 30; i++ {
        go getInstance()
    }

    // Scanln is similar to Scan, but stops scanning at a newline and
    // after the final item there must be a newline or EOF.
    fmt.Scanln()
}</code></pre>
</figure>

#### []{#example-1--output-txt .anchor} **output.txt:** Результат виконання

<figure class="code">
<pre class="code" lang="output"><code>Creating single instance now.
Single instance already created.
Single instance already created.</code></pre>
</figure>
:::
:::::

::: {#nav-tab .nav .nav-tabs .nav-tabs-bottom .nav-justified role="tablist"}
[Концептуальний приклад](#example-0){#example-0-tab-bottom .nav-item
.nav-link .active toggle="tab" role="tab" aria-controls="example-0"
aria-selected="true"} [Інший приклад](#example-1){#example-1-tab-bottom
.nav-item .nav-link toggle="tab" role="tab" aria-controls="example-1"
aria-selected="true"}
:::

::: next
#### Читаємо далі

[Адаптер на Go []{.fa
.fa-arrow-right}](../../adapter/go/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Повернутись назад

[[]{.fa .fa-arrow-left} Прототип на
Go](../../prototype/go/example.html){.btn .btn-default rel="prev"}
:::

## **Одинак** іншими мовами програмування

[![Одинак на
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Одинак на C#"){.prog-lang-link}
[![Одинак на
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Одинак на C++"){.prog-lang-link}
[![Одинак на
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Одинак на Java"){.prog-lang-link}
[![Одинак на
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Одинак на PHP"){.prog-lang-link}
[![Одинак на
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Одинак на Python"){.prog-lang-link}
[![Одинак на
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Одинак на Ruby"){.prog-lang-link}
[![Одинак на
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Одинак на Rust"){.prog-lang-link}
[![Одинак на
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Одинак на Swift"){.prog-lang-link}
[![Одинак на
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Одинак на TypeScript"){.prog-lang-link}
:::::::::::::::::::::::::::

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
:::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::
