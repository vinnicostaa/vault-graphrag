::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Паттерны
проектирования](../../../design-patterns.html){.type} /
[Снимок](../../memento.html){.type} / [Go](../../go.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Снимок](../../../../images/patterns/cards/memento-mini723d.png?id=8b2ea4dc2c5d15775a654808cc9de099){srcset="/images/patterns/cards/memento-mini-2x.png?id=1d7cba189261dd84b11369a6838b1055 2x"}
:::

# **Снимок** на Go {#снимок-на-go .pattern-example-title-block-title}
::::

:::::::::::::: pattern-example-body
::: pattern-example-brief
**Снимок** --- это поведенческий паттерн, позволяющий делать снимки
внутреннего состояния объектов, а затем восстанавливать их.

При этом Снимок не раскрывает подробностей реализации объектов, и клиент
не имеет доступа к защищённой информации объекта.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Подробней о паттерне Снимок ](../../memento.html){.btn .btn-lg
.btn-primary}
:::

::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
Навигация
:::

::: en-intro
 [Интро](#)
:::

::: en-intro
 [Концептуальный пример](#example-0)
:::

::: en-file
 [originator](#example-0--originator-go)
:::

::: en-file
 [memento](#example-0--memento-go)
:::

::: en-file
 [caretaker](#example-0--caretaker-go)
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

## Концептуальный пример {#example-0-title}

Паттерн Снимок позволяет нам сохранять «снимки» состояния объекта. В
дальнейшем эти снимки можно использовать, чтобы вернуть его в прежнее
состояние. Это весьма удобно в случаях, когда для объекта нужно
реализовать операции «отменить/повторить».

#### []{#example-0--originator-go .anchor} **originator.go:** Создатель

<figure class="code">
<pre class="code" lang="go"><code>package main

type Originator struct {
    state string
}

func (e *Originator) createMemento() *Memento {
    return &amp;Memento{state: e.state}
}

func (e *Originator) restoreMemento(m *Memento) {
    e.state = m.getSavedState()
}

func (e *Originator) setState(state string) {
    e.state = state
}

func (e *Originator) getState() string {
    return e.state
}</code></pre>
</figure>

#### []{#example-0--memento-go .anchor} **memento.go:** Снимок

<figure class="code">
<pre class="code" lang="go"><code>package main

type Memento struct {
    state string
}

func (m *Memento) getSavedState() string {
    return m.state
}</code></pre>
</figure>

#### []{#example-0--caretaker-go .anchor} **caretaker.go:** Опекун

<figure class="code">
<pre class="code" lang="go"><code>package main

type Caretaker struct {
    mementoArray []*Memento
}

func (c *Caretaker) addMemento(m *Memento) {
    c.mementoArray = append(c.mementoArray, m)
}

func (c *Caretaker) getMemento(index int) *Memento {
    return c.mementoArray[index]
}</code></pre>
</figure>

#### []{#example-0--main-go .anchor} **main.go:** Клиентский код

<figure class="code">
<pre class="code" lang="go"><code>package main

import &quot;fmt&quot;

func main() {

    caretaker := &amp;Caretaker{
        mementoArray: make([]*Memento, 0),
    }

    originator := &amp;Originator{
        state: &quot;A&quot;,
    }

    fmt.Printf(&quot;Originator Current State: %s\n&quot;, originator.getState())
    caretaker.addMemento(originator.createMemento())

    originator.setState(&quot;B&quot;)
    fmt.Printf(&quot;Originator Current State: %s\n&quot;, originator.getState())
    caretaker.addMemento(originator.createMemento())

    originator.setState(&quot;C&quot;)
    fmt.Printf(&quot;Originator Current State: %s\n&quot;, originator.getState())
    caretaker.addMemento(originator.createMemento())

    originator.restoreMemento(caretaker.getMemento(1))
    fmt.Printf(&quot;Restored to State: %s\n&quot;, originator.getState())

    originator.restoreMemento(caretaker.getMemento(0))
    fmt.Printf(&quot;Restored to State: %s\n&quot;, originator.getState())

}</code></pre>
</figure>

#### []{#example-0--output-txt .anchor} **output.txt:** Результат выполнения

<figure class="code">
<pre class="code" lang="output"><code>originator Current State: A
originator Current State: B
originator Current State: C
Restored to State: B
Restored to State: A</code></pre>
</figure>

::: next
#### Читаем дальше

[Наблюдатель на Go []{.fa
.fa-arrow-right}](../../observer/go/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Посредник на
Go](../../mediator/go/example.html){.btn .btn-default rel="prev"}
:::

## **Снимок** на других языках программирования

[![Снимок на
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Снимок на C#"){.prog-lang-link}
[![Снимок на
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Снимок на C++"){.prog-lang-link}
[![Снимок на
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Снимок на Java"){.prog-lang-link}
[![Снимок на
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Снимок на PHP"){.prog-lang-link}
[![Снимок на
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Снимок на Python"){.prog-lang-link}
[![Снимок на
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Снимок на Ruby"){.prog-lang-link}
[![Снимок на
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Снимок на Rust"){.prog-lang-link}
[![Снимок на
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Снимок на Swift"){.prog-lang-link}
[![Снимок на
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Снимок на TypeScript"){.prog-lang-link}
::::::::::::::::::::

::::::: {.prom .banner-sidebar .banner .banner-striped .banner-examples .banner-removable .banner-removable-patterns data-id="DP: Examples-IDE" creative-id="examples-sidebar-ru" position="sidebar"}
:::::: {.banner-inner style="font-size: 14px; line-height: 21px;"}
::: banner-examples-img
[![](../../../../images/patterns/banners/examples-ide91ca.png?id=3115b4b548fb96b75974e2de8f4f49bc){width="215"
height="158" loading="lazy"
srcset="/images/patterns/banners/examples-ide-2x.png?id=93c007a6d157b97c28eb213f59d716ae 2x"}](../../book.html)
:::

::: banner-examples-fade
[ Архив с примерами](../../book.html){.btn .btn-secondary .btn-block}
:::

::: {.banner-examples-text style="margin-top: 1rem;"}
Купи книгу **Погружение в Паттерны** и получи архив с десятками
детальных примеров, которые можно открывать прямо в IDE.

[ Узнать больше...](../../book.html){.btn .btn-secondary .btn-block}
:::
::::::
:::::::
::::::::::::::::::::::::::
:::::::::::::::::::::::::::
