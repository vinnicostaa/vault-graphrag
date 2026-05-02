:::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Паттерны
проектирования](../../../design-patterns.html){.type} /
[Состояние](../../state.html){.type} / [C#](../../csharp.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Состояние](../../../../images/patterns/cards/state-mini63fb.png?id=f4018837e0641d1dade756b6678fd4ee){srcset="/images/patterns/cards/state-mini-2x.png?id=7e24398b27a43c7bd286fc0ea54d2a35 2x"}
:::

# **Состояние** на C# {#состояние-на-c .pattern-example-title-block-title}
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
 [Program](#example-0--Program-cs)
:::

::: en-file
 [Output](#example-0--Output-txt)
:::
::::::::

**Сложность:** []{.fa-stars}

**Популярность:** []{.fa-stars}

**Применимость:** Паттерн Состояние часто используют в C# для
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

#### []{#example-0--Program-cs .anchor} **Program.cs:** Пример структуры паттерна

<figure class="code">
<pre class="code" lang="csharp"><code>using System;

namespace RefactoringGuru.DesignPatterns.State.Conceptual
{
    // Контекст определяет интерфейс, представляющий интерес для клиентов. Он
    // также хранит ссылку на экземпляр подкласса Состояния, который отображает
    // текущее состояние Контекста.
    class Context
    {
        // Ссылка на текущее состояние Контекста.
        private State _state = null;

        public Context(State state)
        {
            this.TransitionTo(state);
        }

        // Контекст позволяет изменять объект Состояния во время выполнения.
        public void TransitionTo(State state)
        {
            Console.WriteLine($&quot;Context: Transition to {state.GetType().Name}.&quot;);
            this._state = state;
            this._state.SetContext(this);
        }

        // Контекст делегирует часть своего поведения текущему объекту
        // Состояния.
        public void Request1()
        {
            this._state.Handle1();
        }

        public void Request2()
        {
            this._state.Handle2();
        }
    }
    
    // Базовый класс Состояния объявляет методы, которые должны реализовать все
    // Конкретные Состояния, а также предоставляет обратную ссылку на объект
    // Контекст, связанный с Состоянием. Эта обратная ссылка может
    // использоваться Состояниями для передачи Контекста другому Состоянию.
    abstract class State
    {
        protected Context _context;

        public void SetContext(Context context)
        {
            this._context = context;
        }

        public abstract void Handle1();

        public abstract void Handle2();
    }

    // Конкретные Состояния реализуют различные модели поведения, связанные с
    // состоянием Контекста.
    class ConcreteStateA : State
    {
        public override void Handle1()
        {
            Console.WriteLine(&quot;ConcreteStateA handles request1.&quot;);
            Console.WriteLine(&quot;ConcreteStateA wants to change the state of the context.&quot;);
            this._context.TransitionTo(new ConcreteStateB());
        }

        public override void Handle2()
        {
            Console.WriteLine(&quot;ConcreteStateA handles request2.&quot;);
        }
    }

    class ConcreteStateB : State
    {
        public override void Handle1()
        {
            Console.Write(&quot;ConcreteStateB handles request1.&quot;);
        }

        public override void Handle2()
        {
            Console.WriteLine(&quot;ConcreteStateB handles request2.&quot;);
            Console.WriteLine(&quot;ConcreteStateB wants to change the state of the context.&quot;);
            this._context.TransitionTo(new ConcreteStateA());
        }
    }

    class Program
    {
        static void Main(string[] args)
        {
            // Клиентский код.
            var context = new Context(new ConcreteStateA());
            context.Request1();
            context.Request2();
        }
    }
}</code></pre>
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

[Стратегия на C# []{.fa
.fa-arrow-right}](../../strategy/csharp/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Наблюдатель на
C#](../../observer/csharp/example.html){.btn .btn-default rel="prev"}
:::

## **Состояние** на других языках программирования

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
[![Состояние на
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Состояние на TypeScript"){.prog-lang-link}
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
