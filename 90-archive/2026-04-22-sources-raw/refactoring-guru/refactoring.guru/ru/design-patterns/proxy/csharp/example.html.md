:::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Паттерны
проектирования](../../../design-patterns.html){.type} /
[Заместитель](../../proxy.html){.type} / [C#](../../csharp.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Заместитель](../../../../images/patterns/cards/proxy-minid8d7.png?id=25890b11e7dc5af29625ccd0678b63a8){srcset="/images/patterns/cards/proxy-mini-2x.png?id=8638fac9dc08c992852492f9cb29d9c6 2x"}
:::

# **Заместитель** на C# {#заместитель-на-c .pattern-example-title-block-title}
::::

::::::::::: pattern-example-body
::: pattern-example-brief
**Заместитель** --- это объект, который выступает прослойкой между
клиентом и реальным сервисным объектом. Заместитель получает вызовы от
клиента, выполняет свою функцию (контроль доступа, кеширование,
изменение запроса и прочее), а затем передаёт вызов сервисному объекту.

Заместитель имеет тот же интерфейс, что и реальный объект, поэтому для
клиента нет разницы --- работать через заместителя или напрямую.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Подробней о паттерне Заместитель ](../../proxy.html){.btn .btn-lg
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

**Применимость:** Паттерн Заместитель применяется в C# коде тогда, когда
надо заменить настоящий объект его суррогатом, причём незаметно для
клиентов настоящего объекта. Это позволит выполнить какие-то добавочные
поведения до или после основного поведения настоящего объекта.

**Признаки применения паттерна:** Класс заместителя чаще всего
делегирует всю настоящую работу своему реальному объекту. Заместители
часто сами следят за жизненным циклом своего реального объекта.
:::::::::::

[]{#example-0}

## Концептуальный пример {#example-0-title}

Этот пример показывает структуру паттерна **Заместитель**, а именно ---
из каких классов он состоит, какие роли эти классы выполняют и как они
взаимодействуют друг с другом.

#### []{#example-0--Program-cs .anchor} **Program.cs:** Пример структуры паттерна

<figure class="code">
<pre class="code" lang="csharp"><code>using System;

namespace RefactoringGuru.DesignPatterns.Proxy.Conceptual
{
    // Интерфейс Субъекта объявляет общие операции как для Реального Субъекта,
    // так и для Заместителя. Пока клиент работает с Реальным Субъектом,
    // используя этот интерфейс, вы сможете передать ему заместителя вместо
    // реального субъекта.
    public interface ISubject
    {
        void Request();
    }
    
    // Реальный Субъект содержит некоторую базовую бизнес-логику. Как правило,
    // Реальные Субъекты способны выполнять некоторую полезную работу, которая к
    // тому же может быть очень медленной или точной – например, коррекция
    // входных данных. Заместитель может решить эти задачи без каких-либо
    // изменений в коде Реального Субъекта.
    class RealSubject : ISubject
    {
        public void Request()
        {
            Console.WriteLine(&quot;RealSubject: Handling Request.&quot;);
        }
    }
    
    // Интерфейс Заместителя идентичен интерфейсу Реального Субъекта.
    class Proxy : ISubject
    {
        private RealSubject _realSubject;
        
        public Proxy(RealSubject realSubject)
        {
            this._realSubject = realSubject;
        }
        
        // Наиболее распространёнными областями применения паттерна Заместитель
        // являются ленивая загрузка, кэширование, контроль доступа, ведение
        // журнала и т.д. Заместитель может выполнить одну из этих задач, а
        // затем, в зависимости от результата, передать выполнение одноимённому
        // методу в связанном объекте класса Реального Субъект.
        public void Request()
        {
            if (this.CheckAccess())
            {
                this._realSubject.Request();

                this.LogAccess();
            }
        }
        
        public bool CheckAccess()
        {
            // Некоторые реальные проверки должны проходить здесь.
            Console.WriteLine(&quot;Proxy: Checking access prior to firing a real request.&quot;);

            return true;
        }
        
        public void LogAccess()
        {
            Console.WriteLine(&quot;Proxy: Logging the time of request.&quot;);
        }
    }
    
    public class Client
    {
        // Клиентский код должен работать со всеми объектами (как с реальными,
        // так и заместителями) через интерфейс Субъекта, чтобы поддерживать как
        // реальные субъекты, так и заместителей. В реальной жизни, однако,
        // клиенты в основном работают с реальными субъектами напрямую. В этом
        // случае, для более простой реализации паттерна, можно расширить
        // заместителя из класса реального субъекта.
        public void ClientCode(ISubject subject)
        {
            // ...
            
            subject.Request();
            
            // ...
        }
    }
    
    class Program
    {
        static void Main(string[] args)
        {
            Client client = new Client();
            
            Console.WriteLine(&quot;Client: Executing the client code with a real subject:&quot;);
            RealSubject realSubject = new RealSubject();
            client.ClientCode(realSubject);

            Console.WriteLine();

            Console.WriteLine(&quot;Client: Executing the same client code with a proxy:&quot;);
            Proxy proxy = new Proxy(realSubject);
            client.ClientCode(proxy);
        }
    }
}</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** Результат выполнения

<figure class="code">
<pre class="code" lang="output"><code>Client: Executing the client code with a real subject:
RealSubject: Handling Request.

Client: Executing the same client code with a proxy:
Proxy: Checking access prior to firing a real request.
RealSubject: Handling Request.
Proxy: Logging the time of request.</code></pre>
</figure>

::: next
#### Читаем дальше

[Цепочка обязанностей на C# []{.fa
.fa-arrow-right}](../../chain-of-responsibility/csharp/example.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Легковес на
C#](../../flyweight/csharp/example.html){.btn .btn-default rel="prev"}
:::

## **Заместитель** на других языках программирования

[![Заместитель на
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Заместитель на C++"){.prog-lang-link}
[![Заместитель на
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Заместитель на Go"){.prog-lang-link}
[![Заместитель на
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Заместитель на Java"){.prog-lang-link}
[![Заместитель на
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Заместитель на PHP"){.prog-lang-link}
[![Заместитель на
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Заместитель на Python"){.prog-lang-link}
[![Заместитель на
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Заместитель на Ruby"){.prog-lang-link}
[![Заместитель на
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Заместитель на Rust"){.prog-lang-link}
[![Заместитель на
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Заместитель на Swift"){.prog-lang-link}
[![Заместитель на
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Заместитель на TypeScript"){.prog-lang-link}
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
