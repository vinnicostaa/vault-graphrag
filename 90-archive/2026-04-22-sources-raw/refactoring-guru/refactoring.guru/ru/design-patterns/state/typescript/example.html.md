:::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Паттерны
проектирования](../../../design-patterns.html){.type} /
[Состояние](../../state.html){.type} /
[TypeScript](../../typescript.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Состояние](../../../../images/patterns/cards/state-mini63fb.png?id=f4018837e0641d1dade756b6678fd4ee){srcset="/images/patterns/cards/state-mini-2x.png?id=7e24398b27a43c7bd286fc0ea54d2a35 2x"}
:::

# **Состояние** на TypeScript {#состояние-на-typescript .pattern-example-title-block-title}
::::

::::::::::: pattern-example-body
::: pattern-example-brief
**Состояние** --- это поведенческий паттерн, позволяющий динамически
изменять поведение объекта при смене его состояния.

Поведения, зависящие от состояния, переезжают в отдельные классы.
Первоначальный класс хранит ссылку на один из таких объектов-состояний и
делегирует ему работу.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Подробней о паттерне Состояние ](../../state.html){.btn .btn-lg
.btn-primary}
:::

:::::::: {.sidebar-navigation .with-scroll}
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
 [index](#example-0--index-ts)
:::

::: en-file
 [Output](#example-0--Output-txt)
:::
::::::::

**Сложность:** []{.fa-stars}

**Популярность:** []{.fa-stars}

**Применимость:** Паттерн Состояние часто используют в TypeScript для
превращения в объекты громоздких стейт-машин, построенных на операторах
`switch`.

**Признаки применения паттерна:** Методы класса делегируют работу одному
вложенному объекту.
:::::::::::

[]{#example-0}

## Концептуальный пример {#example-0-title}

Этот пример показывает структуру паттерна **Состояние**, а именно --- из
каких классов он состоит, какие роли эти классы выполняют и как они
взаимодействуют друг с другом.

#### []{#example-0--index-ts .anchor} **index.ts:** Пример структуры паттерна

<figure class="code">
<pre class="code" lang="typescript"><code>/**
 * Контекст определяет интерфейс, представляющий интерес для клиентов. Он также
 * хранит ссылку на экземпляр подкласса Состояния, который отображает текущее
 * состояние Контекста.
 */
class Context {
    /**
     * @type {State} Ссылка на текущее состояние Контекста.
     */
    private state: State;

    constructor(state: State) {
        this.transitionTo(state);
    }

    /**
     * Контекст позволяет изменять объект Состояния во время выполнения.
     */
    public transitionTo(state: State): void {
        console.log(`Context: Transition to ${(&lt;any&gt;state).constructor.name}.`);
        this.state = state;
        this.state.setContext(this);
    }

    /**
     * Контекст делегирует часть своего поведения текущему объекту Состояния.
     */
    public request1(): void {
        this.state.handle1();
    }

    public request2(): void {
        this.state.handle2();
    }
}

/**
 * Базовый класс Состояния объявляет методы, которые должны реализовать все
 * Конкретные Состояния, а также предоставляет обратную ссылку на объект
 * Контекст, связанный с Состоянием. Эта обратная ссылка может использоваться
 * Состояниями для передачи Контекста другому Состоянию.
 */
abstract class State {
    protected context: Context;

    public setContext(context: Context) {
        this.context = context;
    }

    public abstract handle1(): void;

    public abstract handle2(): void;
}

/**
 * Конкретные Состояния реализуют различные модели поведения, связанные с
 * состоянием Контекста.
 */
class ConcreteStateA extends State {
    public handle1(): void {
        console.log(&#39;ConcreteStateA handles request1.&#39;);
        console.log(&#39;ConcreteStateA wants to change the state of the context.&#39;);
        this.context.transitionTo(new ConcreteStateB());
    }

    public handle2(): void {
        console.log(&#39;ConcreteStateA handles request2.&#39;);
    }
}

class ConcreteStateB extends State {
    public handle1(): void {
        console.log(&#39;ConcreteStateB handles request1.&#39;);
    }

    public handle2(): void {
        console.log(&#39;ConcreteStateB handles request2.&#39;);
        console.log(&#39;ConcreteStateB wants to change the state of the context.&#39;);
        this.context.transitionTo(new ConcreteStateA());
    }
}

/**
 * Клиентский код.
 */
const context = new Context(new ConcreteStateA());
context.request1();
context.request2();</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** Результат выполнения

<figure class="code">
<pre class="code" lang="output"><code>Context: Transition to ConcreteStateA.
ConcreteStateA handles request1.
ConcreteStateA wants to change the state of the context.
Context: Transition to ConcreteStateB.
ConcreteStateB handles request2.
ConcreteStateB wants to change the state of the context.
Context: Transition to ConcreteStateA.</code></pre>
</figure>

::: next
#### Читаем дальше

[Стратегия на TypeScript []{.fa
.fa-arrow-right}](../../strategy/typescript/example.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Наблюдатель на
TypeScript](../../observer/typescript/example.html){.btn .btn-default
rel="prev"}
:::

## **Состояние** на других языках программирования

[![Состояние на
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Состояние на C#"){.prog-lang-link}
[![Состояние на
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Состояние на C++"){.prog-lang-link}
[![Состояние на
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Состояние на Go"){.prog-lang-link}
[![Состояние на
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Состояние на Java"){.prog-lang-link}
[![Состояние на
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Состояние на PHP"){.prog-lang-link}
[![Состояние на
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Состояние на Python"){.prog-lang-link}
[![Состояние на
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Состояние на Ruby"){.prog-lang-link}
[![Состояние на
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Состояние на Rust"){.prog-lang-link}
[![Состояние на
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Состояние на Swift"){.prog-lang-link}
:::::::::::::::::

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
:::::::::::::::::::::::
::::::::::::::::::::::::
