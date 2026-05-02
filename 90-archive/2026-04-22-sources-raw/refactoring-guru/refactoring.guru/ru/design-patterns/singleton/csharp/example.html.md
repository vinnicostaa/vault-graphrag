:::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Паттерны
проектирования](../../../design-patterns.html){.type} /
[Одиночка](../../singleton.html){.type} / [C#](../../csharp.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Одиночка](../../../../images/patterns/cards/singleton-minif559.png?id=914e1565dfdf15f240e766163bd303ec){srcset="/images/patterns/cards/singleton-mini-2x.png?id=4a7348083d7719d02709a5c9ff878b9f 2x"}
:::

# **Одиночка** на C# {#одиночка-на-c .pattern-example-title-block-title}
::::

::::::::::::::: pattern-example-body
::: pattern-example-brief
**Одиночка** --- это порождающий паттерн, который гарантирует
существование только одного объекта определённого класса, а также
позволяет достучаться до этого объекта из любого места программы.

Одиночка имеет такие же преимущества и недостатки, что и глобальные
переменные. Его невероятно удобно использовать, но он нарушает
модульность вашего кода.

Вы не сможете просто взять и использовать класс, зависящий от одиночки в
другой программе. Для этого придётся эмулировать присутствие одиночки и
там. Чаще всего эта проблема проявляется при написании юнит-тестов.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Подробней о паттерне Одиночка ](../../singleton.html){.btn .btn-lg
.btn-primary}
:::

:::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
Навигация
:::

::: en-intro
 [Интро](#)
:::

::: en-intro
 [Наивный Одиночка (небезопасный в многопоточной среде)](#example-0)
:::

::: en-file
 [Program](#example-0--Program-cs)
:::

::: en-file
 [Output](#example-0--Output-txt)
:::

::: en-intro
 [Многопоточный Одиночка](#example-1)
:::

::: en-file
 [Program](#example-1--Program-cs)
:::

::: en-file
 [Output](#example-1--Output-txt)
:::

::: en-intro
 [Хотите ещё?](#example-2)
:::
::::::::::::

**Сложность:** []{.fa-stars}

**Популярность:** []{.fa-stars}

**Применимость:** Многие программисты считают Одиночку антипаттерном,
поэтому его всё реже и реже можно встретить в C#-коде.

**Признаки применения паттерна:** Одиночку можно определить по
статическому создающему методу, который возвращает один и тот же объект.
:::::::::::::::

::: {#nav-tab .nav .nav-tabs .nav-justified role="tablist"}
[Наивный Одиночка (небезопасный в многопоточной
среде)](#example-0){#example-0-tab .nav-item .nav-link .active
toggle="tab" role="tab" aria-controls="example-0" aria-selected="true"}
[Многопоточный Одиночка](#example-1){#example-1-tab .nav-item .nav-link
toggle="tab" role="tab" aria-controls="example-1" aria-selected="true"}
[Хотите ещё?](#example-2){#example-2-tab .nav-item .nav-link
toggle="tab" role="tab" aria-controls="example-2" aria-selected="true"}
:::

:::::: {#nav-tabContent .pattern-example-tab-content .tab-content}
::: {#example-0 .tab-pane .show .active role="tabpanel" aria-labelledby="example-0-tab" style="padding-top: 24px"}
## Наивный Одиночка (небезопасный в многопоточной среде) {#example-0-title}

Топорно реализовать Одиночку очень просто --- достаточно скрыть
конструктор и предоставить статический создающий метод.

Тот же класс ведёт себя неправильно в многопоточной среде. Несколько
потоков могут одновременно вызвать метод получения Одиночки и создать
сразу несколько экземпляров объекта.

#### []{#example-0--Program-cs .anchor} **Program.cs:** Пример структуры паттерна

<figure class="code">
<pre class="code" lang="csharp"><code>using System;

namespace RefactoringGuru.DesignPatterns.Singleton.Conceptual.NonThreadSafe
{
    // Класс Одиночка предоставляет метод `GetInstance`, который ведёт себя как
    // альтернативный конструктор и позволяет клиентам получать один и тот же
    // экземпляр класса при каждом вызове.

    // EN : The Singleton should always be a &#39;sealed&#39; class to prevent class
    // inheritance through external classes and also through nested classes.
    public sealed class Singleton
    {
        // Конструктор Одиночки всегда должен быть скрытым, чтобы предотвратить
        // создание объекта через оператор new.
        private Singleton() { }

        // Объект одиночки храниться в статичном поле класса. Существует
        // несколько способов инициализировать это поле, и все они имеют разные
        // достоинства и недостатки. В этом примере мы рассмотрим простейший из
        // них, недостатком которого является полная неспособность правильно
        // работать в многопоточной среде.
        private static Singleton _instance;

        // Это статический метод, управляющий доступом к экземпляру одиночки.
        // При первом запуске, он создаёт экземпляр одиночки и помещает его в
        // статическое поле. При последующих запусках, он возвращает клиенту
        // объект, хранящийся в статическом поле.
        public static Singleton GetInstance()
        {
            if (_instance == null)
            {
                _instance = new Singleton();
            }
            return _instance;
        }

        // Наконец, любой одиночка должен содержать некоторую бизнес-логику,
        // которая может быть выполнена на его экземпляре.
        public void someBusinessLogic()
        {
            // ...
        }
    }

    class Program
    {
        static void Main(string[] args)
        {
            // Клиентский код.
            Singleton s1 = Singleton.GetInstance();
            Singleton s2 = Singleton.GetInstance();

            if (s1 == s2)
            {
                Console.WriteLine(&quot;Singleton works, both variables contain the same instance.&quot;);
            }
            else
            {
                Console.WriteLine(&quot;Singleton failed, variables contain different instances.&quot;);
            }
        }
    }
}</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** Результат выполнения

<figure class="code">
<pre class="code" lang="output"><code>Singleton works, both variables contain the same instance.</code></pre>
</figure>
:::

::: {#example-1 .tab-pane role="tabpanel" aria-labelledby="example-1-tab" style="padding-top: 24px"}
## Многопоточный Одиночка {#example-1-title}

Чтобы исправить проблему, требуется синхронизировать потоки при создании
объекта-Одиночки.

#### []{#example-1--Program-cs .anchor} **Program.cs:** Пример структуры паттерна

<figure class="code">
<pre class="code" lang="csharp"><code>using System;
using System.Threading;

namespace Singleton
{
    // Эта реализация Одиночки называется &quot;блокировка с двойной проверкой&quot;
    // (double check lock). Она безопасна в многопоточной среде, а также
    // позволяет отложенную инициализацию объекта Одиночки.
    class Singleton
    {
        private Singleton() { }

        private static Singleton _instance;

        // У нас теперь есть объект-блокировка для синхронизации потоков во
        // время первого доступа к Одиночке.
        private static readonly object _lock = new object();

        public static Singleton GetInstance(string value)
        {
            // Это условие нужно для того, чтобы не стопорить потоки блокировкой
            // после того как объект-одиночка уже создан.
            if (_instance == null)
            {
                // Теперь представьте, что программа была только-только
                // запущена. Объекта-одиночки ещё никто не создавал, поэтому
                // несколько потоков вполне могли одновременно пройти через
                // предыдущее условие и достигнуть блокировки. Самый быстрый
                // поток поставит блокировку и двинется внутрь секции, пока
                // другие будут здесь его ожидать.
                lock (_lock)
                {
                    // Первый поток достигает этого условия и проходит внутрь,
                    // создавая объект-одиночку. Как только этот поток покинет
                    // секцию и освободит блокировку, следующий поток может
                    // снова установить блокировку и зайти внутрь. Однако теперь
                    // экземпляр одиночки уже будет создан и поток не сможет
                    // пройти через это условие, а значит новый объект не будет
                    // создан.
                    if (_instance == null)
                    {
                        _instance = new Singleton();
                        _instance.Value = value;
                    }
                }
            }
            return _instance;
        }

        // Мы используем это поле, чтобы доказать, что наш Одиночка
        // действительно работает.
        public string Value { get; set; }
    }

    class Program
    {
        static void Main(string[] args)
        {
            // Клиентский код.
            
            Console.WriteLine(
                &quot;{0}\n{1}\n\n{2}\n&quot;,
                &quot;If you see the same value, then singleton was reused (yay!)&quot;,
                &quot;If you see different values, then 2 singletons were created (booo!!)&quot;,
                &quot;RESULT:&quot;
            );
            
            Thread process1 = new Thread(() =&gt;
            {
                TestSingleton(&quot;FOO&quot;);
            });
            Thread process2 = new Thread(() =&gt;
            {
                TestSingleton(&quot;BAR&quot;);
            });
            
            process1.Start();
            process2.Start();
            
            process1.Join();
            process2.Join();
        }
        
        public static void TestSingleton(string value)
        {
            Singleton singleton = Singleton.GetInstance(value);
            Console.WriteLine(singleton.Value);
        } 
    }
}</code></pre>
</figure>

#### []{#example-1--Output-txt .anchor} **Output.txt:** Результат выполнения

<figure class="code">
<pre class="code" lang="output"><code>FOO
FOO</code></pre>
</figure>
:::

::: {#example-2 .tab-pane role="tabpanel" aria-labelledby="example-2-tab" style="padding-top: 24px"}
## Хотите ещё? {#example-2-title}

Существует ещё с полдюжины способов реализации Одиночки в C#. Если
интересно, можете ознакомиться с ними здесь:

[C# in Depth: Implementing
Singleton](https://csharpindepth.com/Articles/Singleton)
:::
::::::

::: {#nav-tab .nav .nav-tabs .nav-tabs-bottom .nav-justified role="tablist"}
[Наивный Одиночка (небезопасный в многопоточной
среде)](#example-0){#example-0-tab-bottom .nav-item .nav-link .active
toggle="tab" role="tab" aria-controls="example-0" aria-selected="true"}
[Многопоточный Одиночка](#example-1){#example-1-tab-bottom .nav-item
.nav-link toggle="tab" role="tab" aria-controls="example-1"
aria-selected="true"} [Хотите ещё?](#example-2){#example-2-tab-bottom
.nav-item .nav-link toggle="tab" role="tab" aria-controls="example-2"
aria-selected="true"}
:::

::: next
#### Читаем дальше

[Адаптер на C# []{.fa
.fa-arrow-right}](../../adapter/csharp/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Прототип на
C#](../../prototype/csharp/example.html){.btn .btn-default rel="prev"}
:::

## **Одиночка** на других языках программирования

[![Одиночка на
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Одиночка на C++"){.prog-lang-link}
[![Одиночка на
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Одиночка на Go"){.prog-lang-link}
[![Одиночка на
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Одиночка на Java"){.prog-lang-link}
[![Одиночка на
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Одиночка на PHP"){.prog-lang-link}
[![Одиночка на
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Одиночка на Python"){.prog-lang-link}
[![Одиночка на
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Одиночка на Ruby"){.prog-lang-link}
[![Одиночка на
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Одиночка на Rust"){.prog-lang-link}
[![Одиночка на
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Одиночка на Swift"){.prog-lang-link}
[![Одиночка на
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Одиночка на TypeScript"){.prog-lang-link}
:::::::::::::::::::::::::::

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
:::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::
