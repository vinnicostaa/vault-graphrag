:::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Паттерны
проектирования](../../../design-patterns.html){.type} /
[Легковес](../../flyweight.html){.type} / [C#](../../csharp.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Легковес](../../../../images/patterns/cards/flyweight-minie3d8.png?id=422ca8d2f90614dce810a8812c626698){srcset="/images/patterns/cards/flyweight-mini-2x.png?id=857ca676fff84317bb0dab76abfce08e 2x"}
:::

# **Легковес** на C# {#легковес-на-c .pattern-example-title-block-title}
::::

::::::::::: pattern-example-body
::: pattern-example-brief
**Легковес** --- это структурный паттерн, который экономит память,
благодаря разделению общего состояния, вынесенного в один объект, между
множеством объектов.

Легковес позволяет экономить память, кешируя одинаковые данные,
используемые в разных объектах.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Подробней о паттерне Легковес ](../../flyweight.html){.btn .btn-lg
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

**Применимость:** Весь смысл использования Легковеса --- в экономии
памяти. Поэтому, если в приложении нет такой проблемы, то вы вряд ли
найдёте там примеры Легковеса.

**Признаки применения паттерна:** Легковес можно определить по создающим
методам класса, которые возвращают закешированные объекты, вместо
создания новых.
:::::::::::

[]{#example-0}

## Концептуальный пример {#example-0-title}

Этот пример показывает структуру паттерна **Легковес**, а именно --- из
каких классов он состоит, какие роли эти классы выполняют и как они
взаимодействуют друг с другом.

#### []{#example-0--Program-cs .anchor} **Program.cs:** Пример структуры паттерна

<figure class="code">
<pre class="code" lang="csharp"><code>using System;
using System.Collections.Generic;
using System.Linq;
using System.Text.Json;
// Используем библиотеку Json.NET, загрузить можно через NuGet Package Manager

namespace RefactoringGuru.DesignPatterns.Flyweight.Conceptual
{
    // Легковес хранит общую часть состояния (также называемую внутренним
    // состоянием), которая принадлежит нескольким реальным бизнес-объектам.
    // Легковес принимает оставшуюся часть состояния (внешнее состояние,
    // уникальное для каждого объекта) через его параметры метода.
    public class Flyweight
    {
        private Car _sharedState;

        public Flyweight(Car car)
        {
            this._sharedState = car;
        }

        public void Operation(Car uniqueState)
        {
            string s = JsonSerializer.Serialize(this._sharedState);
            string u = JsonSerializer.Serialize(uniqueState);
            Console.WriteLine($&quot;Flyweight: Displaying shared {s} and unique {u} state.&quot;);
        }
    }

    // Фабрика Легковесов создает объекты-Легковесы и управляет ими. Она
    // обеспечивает правильное разделение легковесов. Когда клиент запрашивает
    // легковес, фабрика либо возвращает существующий экземпляр, либо создает
    // новый, если он ещё не существует.
    public class FlyweightFactory
    {
        private List&lt;Tuple&lt;Flyweight, string&gt;&gt; flyweights = new List&lt;Tuple&lt;Flyweight, string&gt;&gt;();

        public FlyweightFactory(params Car[] args)
        {
            foreach (var elem in args)
            {
                flyweights.Add(new Tuple&lt;Flyweight, string&gt;(new Flyweight(elem), this.getKey(elem)));
            }
        }

        // Возвращает хеш строки Легковеса для данного состояния.
        public string getKey(Car key)
        {
            List&lt;string&gt; elements = new List&lt;string&gt;();

            elements.Add(key.Model);
            elements.Add(key.Color);
            elements.Add(key.Company);

            if (key.Owner != null &amp;&amp; key.Number != null)
            {
                elements.Add(key.Number);
                elements.Add(key.Owner);
            }

            elements.Sort();

            return string.Join(&quot;_&quot;, elements);
        }

        // Возвращает существующий Легковес с заданным состоянием или создает
        // новый.
        public Flyweight GetFlyweight(Car sharedState)
        {
            string key = this.getKey(sharedState);

            if (flyweights.Where(t =&gt; t.Item2 == key).Count() == 0)
            {
                Console.WriteLine(&quot;FlyweightFactory: Can&#39;t find a flyweight, creating new one.&quot;);
                this.flyweights.Add(new Tuple&lt;Flyweight, string&gt;(new Flyweight(sharedState), key));
            }
            else
            {
                Console.WriteLine(&quot;FlyweightFactory: Reusing existing flyweight.&quot;);
            }
            return this.flyweights.Where(t =&gt; t.Item2 == key).FirstOrDefault().Item1;
        }

        public void listFlyweights()
        {
            var count = flyweights.Count;
            Console.WriteLine($&quot;\nFlyweightFactory: I have {count} flyweights:&quot;);
            foreach (var flyweight in flyweights)
            {
                Console.WriteLine(flyweight.Item2);
            }
        }
    }

    public class Car
    {
        public string Owner { get; set; }

        public string Number { get; set; }

        public string Company { get; set; }

        public string Model { get; set; }

        public string Color { get; set; }
    }

    class Program
    {
        static void Main(string[] args)
        {
            // Клиентский код обычно создает кучу предварительно заполненных
            // легковесов на этапе инициализации приложения.
            var factory = new FlyweightFactory(
                new Car { Company = &quot;Chevrolet&quot;, Model = &quot;Camaro2018&quot;, Color = &quot;pink&quot; },
                new Car { Company = &quot;Mercedes Benz&quot;, Model = &quot;C300&quot;, Color = &quot;black&quot; },
                new Car { Company = &quot;Mercedes Benz&quot;, Model = &quot;C500&quot;, Color = &quot;red&quot; },
                new Car { Company = &quot;BMW&quot;, Model = &quot;M5&quot;, Color = &quot;red&quot; },
                new Car { Company = &quot;BMW&quot;, Model = &quot;X6&quot;, Color = &quot;white&quot; }
            );
            factory.listFlyweights();

            addCarToPoliceDatabase(factory, new Car {
                Number = &quot;CL234IR&quot;,
                Owner = &quot;James Doe&quot;,
                Company = &quot;BMW&quot;,
                Model = &quot;M5&quot;,
                Color = &quot;red&quot;
            });

            addCarToPoliceDatabase(factory, new Car {
                Number = &quot;CL234IR&quot;,
                Owner = &quot;James Doe&quot;,
                Company = &quot;BMW&quot;,
                Model = &quot;X1&quot;,
                Color = &quot;red&quot;
            });

            factory.listFlyweights();
        }

        public static void addCarToPoliceDatabase(FlyweightFactory factory, Car car)
        {
            Console.WriteLine(&quot;\nClient: Adding a car to database.&quot;);

            var flyweight = factory.GetFlyweight(new Car {
                Color = car.Color,
                Model = car.Model,
                Company = car.Company
            });

            // Клиентский код либо сохраняет, либо вычисляет внешнее состояние и
            // передает его методам легковеса.
            flyweight.Operation(car);
        }
    }
}</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** Результат выполнения

<figure class="code">
<pre class="code" lang="output"><code>FlyweightFactory: I have 5 flyweights:
Camaro2018_Chevrolet_pink
black_C300_Mercedes Benz
C500_Mercedes Benz_red
BMW_M5_red
BMW_white_X6

Client: Adding a car to database.
FlyweightFactory: Reusing existing flyweight.
Flyweight: Displaying shared {&quot;Owner&quot;:null,&quot;Number&quot;:null,&quot;Company&quot;:&quot;BMW&quot;,&quot;Model&quot;:&quot;M5&quot;,&quot;Color&quot;:&quot;red&quot;} and unique {&quot;Owner&quot;:&quot;James Doe&quot;,&quot;Number&quot;:&quot;CL234IR&quot;,&quot;Company&quot;:&quot;BMW&quot;,&quot;Model&quot;:&quot;M5&quot;,&quot;Color&quot;:&quot;red&quot;} state.

Client: Adding a car to database.
FlyweightFactory: Can&#39;t find a flyweight, creating new one.
Flyweight: Displaying shared {&quot;Owner&quot;:null,&quot;Number&quot;:null,&quot;Company&quot;:&quot;BMW&quot;,&quot;Model&quot;:&quot;X1&quot;,&quot;Color&quot;:&quot;red&quot;} and unique {&quot;Owner&quot;:&quot;James Doe&quot;,&quot;Number&quot;:&quot;CL234IR&quot;,&quot;Company&quot;:&quot;BMW&quot;,&quot;Model&quot;:&quot;X1&quot;,&quot;Color&quot;:&quot;red&quot;} state.

FlyweightFactory: I have 6 flyweights:
Camaro2018_Chevrolet_pink
black_C300_Mercedes Benz
C500_Mercedes Benz_red
BMW_M5_red
BMW_white_X6
BMW_red_X1</code></pre>
</figure>

::: next
#### Читаем дальше

[Заместитель на C# []{.fa
.fa-arrow-right}](../../proxy/csharp/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Фасад на
C#](../../facade/csharp/example.html){.btn .btn-default rel="prev"}
:::

## **Легковес** на других языках программирования

[![Легковес на
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Легковес на C++"){.prog-lang-link}
[![Легковес на
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Легковес на Go"){.prog-lang-link}
[![Легковес на
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Легковес на Java"){.prog-lang-link}
[![Легковес на
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Легковес на PHP"){.prog-lang-link}
[![Легковес на
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Легковес на Python"){.prog-lang-link}
[![Легковес на
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Легковес на Ruby"){.prog-lang-link}
[![Легковес на
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Легковес на Rust"){.prog-lang-link}
[![Легковес на
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Легковес на Swift"){.prog-lang-link}
[![Легковес на
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Легковес на TypeScript"){.prog-lang-link}
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
