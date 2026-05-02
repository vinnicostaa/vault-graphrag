:::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Паттерны
проектирования](../../../design-patterns.html){.type} /
[Стратегия](../../strategy.html){.type} / [C#](../../csharp.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Стратегия](../../../../images/patterns/cards/strategy-mini28f6.png?id=d38abee4fb6f2aed909d262bdadca936){srcset="/images/patterns/cards/strategy-mini-2x.png?id=f4e6608561f8e5d18be6927d4620ad29 2x"}
:::

# **Стратегия** на C# {#стратегия-на-c .pattern-example-title-block-title}
::::

::::::::::: pattern-example-body
::: pattern-example-brief
**Стратегия** --- это поведенческий паттерн, выносит набор алгоритмов в
собственные классы и делает их взаимозаменимыми.

Другие объекты содержат ссылку на объект-стратегию и делегируют ей
работу. Программа может подменить этот объект другим, если требуется
иной способ решения задачи.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Подробней о паттерне Стратегия ](../../strategy.html){.btn .btn-lg
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

**Применимость:** Стратегия часто используется в C#-коде, особенно там,
где нужно подменять алгоритм во время выполнения программы. Многие
примеры стратегии можно заменить простыми lambda-выражениями.

**Признаки применения паттерна:** Класс делегирует выполнение вложенному
объекту абстрактного типа или интерфейса.
:::::::::::

[]{#example-0}

## Концептуальный пример {#example-0-title}

Этот пример показывает структуру паттерна **Стратегия**, а именно --- из
каких классов он состоит, какие роли эти классы выполняют и как они
взаимодействуют друг с другом.

#### []{#example-0--Program-cs .anchor} **Program.cs:** Пример структуры паттерна

<figure class="code">
<pre class="code" lang="csharp"><code>using System;
using System.Collections.Generic;

namespace RefactoringGuru.DesignPatterns.Strategy.Conceptual
{
    // Контекст определяет интерфейс, представляющий интерес для клиентов.
    class Context
    {
        // Контекст хранит ссылку на один из объектов Стратегии. Контекст не
        // знает конкретного класса стратегии. Он должен работать со всеми
        // стратегиями через интерфейс Стратегии.
        private IStrategy _strategy;

        public Context()
        { }

        // Обычно Контекст принимает стратегию через конструктор, а также
        // предоставляет сеттер для её изменения во время выполнения.
        public Context(IStrategy strategy)
        {
            this._strategy = strategy;
        }

        // Обычно Контекст позволяет заменить объект Стратегии во время
        // выполнения.
        public void SetStrategy(IStrategy strategy)
        {
            this._strategy = strategy;
        }

        // Вместо того, чтобы самостоятельно реализовывать множественные версии
        // алгоритма, Контекст делегирует некоторую работу объекту Стратегии.
        public void DoSomeBusinessLogic()
        {
            Console.WriteLine(&quot;Context: Sorting data using the strategy (not sure how it&#39;ll do it)&quot;);
            var result = this._strategy.DoAlgorithm(new List&lt;string&gt; { &quot;a&quot;, &quot;b&quot;, &quot;c&quot;, &quot;d&quot;, &quot;e&quot; });

            string resultStr = string.Empty;
            foreach (var element in result as List&lt;string&gt;)
            {
                resultStr += element + &quot;,&quot;;
            }

            Console.WriteLine(resultStr);
        }
    }

    // Интерфейс Стратегии объявляет операции, общие для всех поддерживаемых
    // версий некоторого алгоритма.
    //
    // Контекст использует этот интерфейс для вызова алгоритма, определённого
    // Конкретными Стратегиями.
    public interface IStrategy
    {
        object DoAlgorithm(object data);
    }

    // Конкретные Стратегии реализуют алгоритм, следуя базовому интерфейсу
    // Стратегии. Этот интерфейс делает их взаимозаменяемыми в Контексте.
    class ConcreteStrategyA : IStrategy
    {
        public object DoAlgorithm(object data)
        {
            var list = data as List&lt;string&gt;;
            list.Sort();

            return list;
        }
    }

    class ConcreteStrategyB : IStrategy
    {
        public object DoAlgorithm(object data)
        {
            var list = data as List&lt;string&gt;;
            list.Sort();
            list.Reverse();

            return list;
        }
    }

    class Program
    {
        static void Main(string[] args)
        {
            // Клиентский код выбирает конкретную стратегию и передаёт её в
            // контекст. Клиент должен знать о различиях между стратегиями,
            // чтобы сделать правильный выбор.
            var context = new Context();

            Console.WriteLine(&quot;Client: Strategy is set to normal sorting.&quot;);
            context.SetStrategy(new ConcreteStrategyA());
            context.DoSomeBusinessLogic();
            
            Console.WriteLine();
            
            Console.WriteLine(&quot;Client: Strategy is set to reverse sorting.&quot;);
            context.SetStrategy(new ConcreteStrategyB());
            context.DoSomeBusinessLogic();
        }
    }
}</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** Результат выполнения

<figure class="code">
<pre class="code" lang="output"><code>Client: Strategy is set to normal sorting.
Context: Sorting data using the strategy (not sure how it&#39;ll do it)
a,b,c,d,e

Client: Strategy is set to reverse sorting.
Context: Sorting data using the strategy (not sure how it&#39;ll do it)
e,d,c,b,a</code></pre>
</figure>

::: next
#### Читаем дальше

[Шаблонный метод на C# []{.fa
.fa-arrow-right}](../../template-method/csharp/example.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Состояние на
C#](../../state/csharp/example.html){.btn .btn-default rel="prev"}
:::

## **Стратегия** на других языках программирования

[![Стратегия на
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Стратегия на C++"){.prog-lang-link}
[![Стратегия на
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Стратегия на Go"){.prog-lang-link}
[![Стратегия на
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Стратегия на Java"){.prog-lang-link}
[![Стратегия на
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Стратегия на PHP"){.prog-lang-link}
[![Стратегия на
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Стратегия на Python"){.prog-lang-link}
[![Стратегия на
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Стратегия на Ruby"){.prog-lang-link}
[![Стратегия на
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Стратегия на Rust"){.prog-lang-link}
[![Стратегия на
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Стратегия на Swift"){.prog-lang-link}
[![Стратегия на
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Стратегия на TypeScript"){.prog-lang-link}
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
