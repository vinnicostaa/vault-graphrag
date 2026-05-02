:::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Паттерны
проектирования](../../../design-patterns.html){.type} /
[Компоновщик](../../composite.html){.type} /
[C#](../../csharp.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Компоновщик](../../../../images/patterns/cards/composite-minib543.png?id=a369d98d18b417f255d04568fd0131b8){srcset="/images/patterns/cards/composite-mini-2x.png?id=3f7f811fefeb0b64f6774746eb42af09 2x"}
:::

# **Компоновщик** на C# {#компоновщик-на-c .pattern-example-title-block-title}
::::

::::::::::: pattern-example-body
::: pattern-example-brief
**Компоновщик** --- это структурный паттерн, который позволяет создавать
дерево объектов и работать с ним так же, как и с единичным объектом.

Компоновщик давно стал синонимом всех задач, связанных с построением
дерева объектов. Все операции компоновщика основаны на рекурсии и
«суммировании» результатов на ветвях дерева.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Подробней о паттерне Компоновщик ](../../composite.html){.btn .btn-lg
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

**Применимость:** Паттерн Компоновщик встречается в любых задачах,
которые связаны с построением дерева. Самый простой пример --- составные
элементы GUI, которые тоже можно рассматривать как дерево.

**Признаки применения паттерна:** Если из объектов строится древовидная
структура, и со всеми объектами дерева, как и с самим деревом работают
через общий интерфейс.
:::::::::::

[]{#example-0}

## Концептуальный пример {#example-0-title}

Этот пример показывает структуру паттерна **Компоновщик**, а именно ---
из каких классов он состоит, какие роли эти классы выполняют и как они
взаимодействуют друг с другом.

#### []{#example-0--Program-cs .anchor} **Program.cs:** Пример структуры паттерна

<figure class="code">
<pre class="code" lang="csharp"><code>using System;
using System.Collections.Generic;

namespace RefactoringGuru.DesignPatterns.Composite.Conceptual
{
    // Базовый класс Компонент объявляет общие операции как для простых, так и
    // для сложных объектов структуры.
    abstract class Component
    {
        public Component() { }

        // Базовый Компонент может сам реализовать некоторое поведение по
        // умолчанию или поручить это конкретным классам, объявив метод,
        // содержащий поведение абстрактным.
        public abstract string Operation();

        // В некоторых случаях целесообразно определить операции управления
        // потомками прямо в базовом классе Компонент. Таким образом, вам не
        // нужно будет предоставлять  конкретные классы компонентов клиентскому
        // коду, даже во время сборки дерева объектов. Недостаток такого подхода
        // в том, что эти методы будут пустыми для компонентов уровня листа.
        public virtual void Add(Component component)
        {
            throw new NotImplementedException();
        }

        public virtual void Remove(Component component)
        {
            throw new NotImplementedException();
        }

        // Вы можете предоставить метод, который позволит клиентскому коду
        // понять, может ли компонент иметь вложенные объекты.
        public virtual bool IsComposite()
        {
            return true;
        }
    }

    // Класс Лист представляет собой конечные объекты структуры. Лист не может
    // иметь вложенных компонентов.
    //
    // Обычно объекты Листьев выполняют фактическую работу, тогда как объекты
    // Контейнера лишь делегируют работу своим подкомпонентам.
    class Leaf : Component
    {
        public override string Operation()
        {
            return &quot;Leaf&quot;;
        }

        public override bool IsComposite()
        {
            return false;
        }
    }

    // Класс Контейнер содержит сложные компоненты, которые могут иметь
    // вложенные компоненты. Обычно объекты Контейнеры делегируют фактическую
    // работу своим детям, а затем «суммируют» результат.
    class Composite : Component
    {
        protected List&lt;Component&gt; _children = new List&lt;Component&gt;();
        
        public override void Add(Component component)
        {
            this._children.Add(component);
        }

        public override void Remove(Component component)
        {
            this._children.Remove(component);
        }

        // Контейнер выполняет свою основную логику особым образом. Он проходит
        // рекурсивно через всех своих детей, собирая и суммируя их результаты.
        // Поскольку потомки контейнера передают эти вызовы своим потомкам и так
        // далее,  в результате обходится всё дерево объектов.
        public override string Operation()
        {
            int i = 0;
            string result = &quot;Branch(&quot;;

            foreach (Component component in this._children)
            {
                result += component.Operation();
                if (i != this._children.Count - 1)
                {
                    result += &quot;+&quot;;
                }
                i++;
            }
            
            return result + &quot;)&quot;;
        }
    }

    class Client
    {
        // Клиентский код работает со всеми компонентами через базовый
        // интерфейс.
        public void ClientCode(Component leaf)
        {
            Console.WriteLine($&quot;RESULT: {leaf.Operation()}\n&quot;);
        }

        // Благодаря тому, что операции управления потомками объявлены в базовом
        // классе Компонента, клиентский код может работать как с простыми, так
        // и со сложными компонентами, вне зависимости от их конкретных классов.
        public void ClientCode2(Component component1, Component component2)
        {
            if (component1.IsComposite())
            {
                component1.Add(component2);
            }
            
            Console.WriteLine($&quot;RESULT: {component1.Operation()}&quot;);
        }
    }
    
    class Program
    {
        static void Main(string[] args)
        {
            Client client = new Client();

            // Таким образом, клиентский код может поддерживать простые
            // компоненты-листья...
            Leaf leaf = new Leaf();
            Console.WriteLine(&quot;Client: I get a simple component:&quot;);
            client.ClientCode(leaf);

            // ...а также сложные контейнеры.
            Composite tree = new Composite();
            Composite branch1 = new Composite();
            branch1.Add(new Leaf());
            branch1.Add(new Leaf());
            Composite branch2 = new Composite();
            branch2.Add(new Leaf());
            tree.Add(branch1);
            tree.Add(branch2);
            Console.WriteLine(&quot;Client: Now I&#39;ve got a composite tree:&quot;);
            client.ClientCode(tree);

            Console.Write(&quot;Client: I don&#39;t need to check the components classes even when managing the tree:\n&quot;);
            client.ClientCode2(tree, leaf);
        }
    }
}</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** Результат выполнения

<figure class="code">
<pre class="code" lang="output"><code>Client: I get a simple component:
RESULT: Leaf

Client: Now I&#39;ve got a composite tree:
RESULT: Branch(Branch(Leaf+Leaf)+Branch(Leaf))

Client: I don&#39;t need to check the components classes even when managing the tree:
RESULT: Branch(Branch(Leaf+Leaf)+Branch(Leaf)+Leaf)</code></pre>
</figure>

::: next
#### Читаем дальше

[Декоратор на C# []{.fa
.fa-arrow-right}](../../decorator/csharp/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Мост на
C#](../../bridge/csharp/example.html){.btn .btn-default rel="prev"}
:::

## **Компоновщик** на других языках программирования

[![Компоновщик на
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Компоновщик на C++"){.prog-lang-link}
[![Компоновщик на
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Компоновщик на Go"){.prog-lang-link}
[![Компоновщик на
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Компоновщик на Java"){.prog-lang-link}
[![Компоновщик на
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Компоновщик на PHP"){.prog-lang-link}
[![Компоновщик на
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Компоновщик на Python"){.prog-lang-link}
[![Компоновщик на
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Компоновщик на Ruby"){.prog-lang-link}
[![Компоновщик на
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Компоновщик на Rust"){.prog-lang-link}
[![Компоновщик на
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Компоновщик на Swift"){.prog-lang-link}
[![Компоновщик на
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Компоновщик на TypeScript"){.prog-lang-link}
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
