::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [Паттерны
проектирования](../design-patterns.html){.type} / [Порождающие
паттерны](creational-patterns.html){.type}
:::

# Абстрактная фабрика {#абстрактная-фабрика .title}

::: aka
Также известен как: [Abstract Factory]{style="display:inline-block"}
:::

::: {.section .intent}
##  Суть паттерна {#intent}

**Абстрактная фабрика** --- это порождающий паттерн проектирования,
который позволяет создавать семейства связанных объектов, не
привязываясь к конкретным классам создаваемых объектов.

<figure class="image">
<img
src="../../images/patterns/content/abstract-factory/abstract-factory-ru5f87.png?id=988d97ceead03bccaf4cdca6e8f4c8d2"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/abstract-factory/abstract-factory-ru.png?id=988d97ceead03bccaf4cdca6e8f4c8d2 640w,/images/patterns/content/abstract-factory/abstract-factory-ru-1.5x.png?id=2b0cc2fbdbf5c371cc2cad8f1eb1cefe 960w,/images/patterns/content/abstract-factory/abstract-factory-ru-2x.png?id=f37d10fac2924dcece1baed2d20c579e 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Паттерн Абстрактная фабрика" />
</figure>
:::

::: {.section .problem}
##  Проблема {#problem}

Представьте, что вы пишете симулятор мебельного магазина. Ваш код
содержит:

1.  Семейство зависимых продуктов. Скажем, `Кресло` + `Диван` +
    `Столик`.

2.  Несколько вариаций этого семейства. Например, продукты `Кресло`,
    `Диван` и `Столик` представлены в трёх разных стилях: `Ар-деко`,
    `Викторианском` и `Модерне`.

<figure class="image">
<img
src="../../images/patterns/diagrams/abstract-factory/problem-ru5aa2.png?id=b64f798b7fb74aebf3b5f900a81c9c3a"
style="aspect-ratio: 1.45;"
srcset="/images/patterns/diagrams/abstract-factory/problem-ru.png?id=b64f798b7fb74aebf3b5f900a81c9c3a 420w,/images/patterns/diagrams/abstract-factory/problem-ru-1.5x.png?id=edfb21c38fd14db814fc300074cf15e8 630w,/images/patterns/diagrams/abstract-factory/problem-ru-2x.png?id=08530964f9ff3a1c39a4d234fdef27b6 840w"
sizes="(max-width: 720px) 100vw, 420px" loading="lazy" width="420"
alt="Таблица соответствия семейства продуктам к их вариациям" />
<figcaption><p>Семейства продуктов и их вариации.</p></figcaption>
</figure>

Вам нужен такой способ создавать объекты продуктов, чтобы они сочетались
с другими продуктами того же семейства. Это важно, так как клиенты
расстраиваются, если получают несочетающуюся мебель.

<figure class="image">
<img
src="../../images/patterns/content/abstract-factory/abstract-factory-comic-1-rue230.png?id=bc5575fb1f4f810bef146df559f45da0"
style="aspect-ratio: 2;"
srcset="/images/patterns/content/abstract-factory/abstract-factory-comic-1-ru.png?id=bc5575fb1f4f810bef146df559f45da0 600w,/images/patterns/content/abstract-factory/abstract-factory-comic-1-ru-1.5x.png?id=744a57850cae64c6456085dc6b253a5a 900w,/images/patterns/content/abstract-factory/abstract-factory-comic-1-ru-2x.png?id=aac5ded07e9d5db54b24be93c3928428 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600" />
<figcaption><p>Клиенты расстраиваются, если получают
несочетающиеся продукты.</p></figcaption>
</figure>

Кроме того, вы не хотите вносить изменения в существующий код при
добавлении новых продуктов или семейcтв в программу. Поставщики часто
обновляют свои каталоги, и вы бы не хотели менять уже написанный код
каждый раз при получении новых моделей мебели.
:::

::: {.section .solution}
##  Решение {#solution}

Для начала паттерн Абстрактная фабрика предлагает выделить общие
интерфейсы для отдельных продуктов, составляющих семейства. Так, все
вариации кресел получат общий интерфейс `Кресло`, все диваны реализуют
интерфейс `Диван` и так далее.

<figure class="image">
<img
src="../../images/patterns/diagrams/abstract-factory/solution1a1ac.png?id=71f2018d8bb443b9cce90d57110675e3"
style="aspect-ratio: 1.5;"
srcset="/images/patterns/diagrams/abstract-factory/solution1.png?id=71f2018d8bb443b9cce90d57110675e3 420w,/images/patterns/diagrams/abstract-factory/solution1-1.5x.png?id=4366e971c850514cde5d33cb7956de8b 630w,/images/patterns/diagrams/abstract-factory/solution1-2x.png?id=eacec286153e058db9255d26541c0529 840w"
sizes="(max-width: 720px) 100vw, 420px" loading="lazy" width="420"
alt="Схема иерархии классов кресел." />
<figcaption><p>Все вариации одного и того же объекта должны жить в одной
иерархии классов.</p></figcaption>
</figure>

Далее вы создаёте *абстрактную фабрику* --- общий интерфейс, который
содержит методы создания всех продуктов семейства (например,
`создатьКресло`, `создатьДиван` и `создатьСтолик`). Эти операции должны
возвращать **абстрактные** типы продуктов, представленные интерфейсами,
которые мы выделили ранее --- `Кресла`, `Диваны` и `Столики`.

<figure class="image">
<img
src="../../images/patterns/diagrams/abstract-factory/solution24d8e.png?id=53975d6e4714c6f942633a879f7ac571"
style="aspect-ratio: 2;"
srcset="/images/patterns/diagrams/abstract-factory/solution2.png?id=53975d6e4714c6f942633a879f7ac571 640w,/images/patterns/diagrams/abstract-factory/solution2-1.5x.png?id=581a6cdc0dcd5511f1c30e509b1d4a7f 960w,/images/patterns/diagrams/abstract-factory/solution2-2x.png?id=b21557081f75ac7b4110427e89a10378 1280w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="640"
alt="Схема иерархии классов фабрик." />
<figcaption><p>Конкретные фабрики соответствуют определённой вариации
семейства продуктов.</p></figcaption>
</figure>

Как насчёт вариаций продуктов? Для каждой вариации семейства продуктов
мы должны создать свою собственную фабрику, реализовав абстрактный
интерфейс. Фабрики создают продукты одной вариации. Например,
`ФабрикаМодерн` будет возвращать только `КреслаМодерн`,`ДиваныМодерн` и
`СтоликиМодерн`.

Клиентский код должен работать как с фабриками, так и с продуктами
только через их общие интерфейсы. Это позволит подавать в ваши классы
любой тип фабрики и производить любые продукты, ничего не ломая.

<figure class="image">
<img
src="../../images/patterns/content/abstract-factory/abstract-factory-comic-2-ru73ac.png?id=fc9fcaf97a9e9551339c1918ce171158"
style="aspect-ratio: 2;"
srcset="/images/patterns/content/abstract-factory/abstract-factory-comic-2-ru.png?id=fc9fcaf97a9e9551339c1918ce171158 600w,/images/patterns/content/abstract-factory/abstract-factory-comic-2-ru-1.5x.png?id=4b4a41adf9290c657be0a04282bc29e2 900w,/images/patterns/content/abstract-factory/abstract-factory-comic-2-ru-2x.png?id=926bbe3eb9db92b5f85d3fc7a0e696fe 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600" />
<figcaption><p>Для клиентского кода должно быть безразлично, с какой
фабрикой работать.</p></figcaption>
</figure>

Например, клиентский код просит фабрику сделать стул. Он не знает,
какого типа была эта фабрика. Он не знает, получит викторианский или
модерновый стул. Для него важно, чтобы на стуле можно было сидеть и
чтобы этот стул отлично смотрелся с диваном той же фабрики.

Осталось прояснить последний момент: кто создаёт объекты конкретных
фабрик, если клиентский код работает только с интерфейсами фабрик?
Обычно программа создаёт конкретный объект фабрики при запуске, причём
тип фабрики выбирается, исходя из параметров окружения или конфигурации.
:::

::::: {.section .structure-container}
##  Структура {#structure}

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/abstract-factory/structureb809.png?id=a3112cdd98765406af94595a3c5e7762"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/diagrams/abstract-factory/structure.png?id=a3112cdd98765406af94595a3c5e7762 720w,/images/patterns/diagrams/abstract-factory/structure-1.5x.png?id=d2964e541620d9c087e693bd0e0a0836 1080w,/images/patterns/diagrams/abstract-factory/structure-2x.png?id=c4d3634ec2e74e02a0fe1a83ce9b50f6 1440w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="720"
alt="Структура классов паттерна Абстрактная фабрика" /><img
src="../../images/patterns/diagrams/abstract-factory/structure-indexed028d.png?id=6ae1c99cbd90cf58753c633624fb1a04"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.56;"
srcset="/images/patterns/diagrams/abstract-factory/structure-indexed.png?id=6ae1c99cbd90cf58753c633624fb1a04 700w,/images/patterns/diagrams/abstract-factory/structure-indexed-1.5x.png?id=473be2975c5716162e6969e6532210ac 1050w,/images/patterns/diagrams/abstract-factory/structure-indexed-2x.png?id=cb6d4e1e89826c42966dc7097374f889 1400w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="700"
alt="Структура классов паттерна Абстрактная фабрика" />
</figure>
:::

1.  **Абстрактные продукты** объявляют интерфейсы продуктов, которые
    связаны друг с другом по смыслу, но выполняют разные функции.

2.  **Конкретные продукты** --- большой набор классов, которые относятся
    к различным абстрактным продуктам (кресло/столик), но имеют одни и
    те же вариации (Викторианский/Модерн).

3.  **Абстрактная фабрика** объявляет методы создания различных
    абстрактных продуктов (кресло/столик).

4.  **Конкретные фабрики** относятся каждая к своей вариации продуктов
    (Викторианский/Модерн) и реализуют методы абстрактной фабрики,
    позволяя создавать все продукты определённой вариации.

5.  Несмотря на то, что конкретные фабрики порождают конкретные
    продукты, сигнатуры их методов должны возвращать соответствующие
    абстрактные продукты. Это позволит клиентскому коду, использующему
    фабрику, не привязываться к конкретным классам продуктов. Клиент
    сможет работать с любыми вариациями продуктов через абстрактные
    интерфейсы.
::::
:::::

::: {.section .pseudocode}
##  Псевдокод {#pseudocode}

В этом примере **Абстрактная фабрика** создаёт кросс-платформенные
элементы интерфейса и следит за тем, чтобы они соответствовали выбранной
операционной системе.

<figure class="image">
<img
src="../../images/patterns/diagrams/abstract-factory/exampled62f.png?id=5928a61d18bf00b047463471c599100a"
style="aspect-ratio: 1.42;"
srcset="/images/patterns/diagrams/abstract-factory/example.png?id=5928a61d18bf00b047463471c599100a 640w,/images/patterns/diagrams/abstract-factory/example-1.5x.png?id=baee3bac793ec97e0ec91c49d9382e7d 960w,/images/patterns/diagrams/abstract-factory/example-2x.png?id=eb5127b1d6486f6fad73be2d5e444449 1280w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="640"
alt="Структура классов примера паттерна Абстрактной фабрики" />
<figcaption><p>Пример кросс-платформенного графического
интерфейса пользователя.</p></figcaption>
</figure>

Кросс-платформенная программа может показывать одни и те же элементы
интерфейса, выглядящие чуточку по-другому в различных операционных
системах. В такой программе важно, чтобы все создаваемые элементы всегда
соответствовали текущей операционной системе. Вы бы не хотели, чтобы
программа, запущенная на Windows, вдруг начала показывать чекбоксы в
стиле macOS.

Абстрактная фабрика объявляет список создающих методов, которые
клиентский код может использовать для получения тех или иных
разновидностей элементов интерфейса. Конкретные фабрики относятся к
различным операционным системам и создают элементы, совместимые с этой
системой.

В самом начале программа определяет, какая из фабрик соответствует
текущей операционке. Затем создаёт эту фабрику и отдаёт её клиентскому
коду. В дальнейшем клиент будет работать только с этой фабрикой, чтобы
исключить несовместимость возвращаемых продуктов.

Клиентский код не зависит от конкретных классов фабрик и элементов
интерфейса. Он общается с ними через абстрактные интерфейсы. Благодаря
этому клиент может работать с любой разновидностью фабрик и элементов
интерфейса.

Чтобы добавить в программу новую вариацию элементов (например, для
поддержки Linux), вам не нужно трогать клиентский код. Достаточно
создать ещё одну фабрику, производящую эти элементы.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// Этот паттерн предполагает, что у вас есть несколько семейств
// продуктов, находящихся в отдельных иерархиях классов
// (Button/Checkbox). Продукты одного семейства должны иметь
// общий интерфейс.
interface Button is
    method paint()

// Семейства продуктов имеют те же вариации (macOS/Windows).
class WinButton implements Button is
    method paint() is
        // Отрисовать кнопку в стиле Windows.

class MacButton implements Button is
    method paint() is
        // Отрисовать кнопку в стиле macOS.


interface Checkbox is
    method paint()

class WinCheckbox implements Checkbox is
    method paint() is
        // Отрисовать чекбокс в стиле Windows.

class MacCheckbox implements Checkbox is
    method paint() is
        // Отрисовать чекбокс в стиле macOS.


// Абстрактная фабрика знает обо всех абстрактных типах
// продуктов.
interface GUIFactory is
    method createButton():Button
    method createCheckbox():Checkbox


// Каждая конкретная фабрика знает и создаёт только продукты
// своей вариации.
class WinFactory implements GUIFactory is
    method createButton():Button is
        return new WinButton()
    method createCheckbox():Checkbox is
        return new WinCheckbox()

// Несмотря на то, что фабрики оперируют конкретными классами,
// их методы возвращают абстрактные типы продуктов. Благодаря
// этому фабрики можно взаимозаменять, не изменяя клиентский
// код.
class MacFactory implements GUIFactory is
    method createButton():Button is
        return new MacButton()
    method createCheckbox():Checkbox is
        return new MacCheckbox()


// Для кода, использующего фабрику, не важно, с какой конкретно
// фабрикой он работает. Все получатели продуктов работают с
// ними через общие интерфейсы.
class Application is
    private field factory: GUIFactory
    private field button: Button
    constructor Application(factory: GUIFactory) is
        this.factory = factory
    method createUI()
        this.button = factory.createButton()
    method paint()
        button.paint()


// Приложение выбирает тип конкретной фабрики и создаёт её
// динамически, исходя из конфигурации или окружения.
class ApplicationConfigurator is
    method main() is
        config = readApplicationConfigFile()

        if (config.OS == &quot;Windows&quot;) then
            factory = new WinFactory()
        else if (config.OS == &quot;Mac&quot;) then
            factory = new MacFactory()
        else
            throw new Exception(&quot;Error! Unknown operating system.&quot;)

        Application app = new Application(factory)</code></pre>
</figure>
:::

:::::::: {.section .applicability-container}
##  Применимость {#applicability}

::::::: applicability
::: applicability-problem
Когда бизнес-логика программы должна работать с разными видами связанных
друг с другом продуктов, не завися от конкретных классов продуктов.
:::

::: applicability-solution
Абстрактная фабрика скрывает от клиентского кода подробности того, как и
какие конкретно объекты будут созданы. Но при этом клиентский код может
работать со всеми типами создаваемых продуктов, поскольку их общий
интерфейс был заранее определён.
:::

::: applicability-problem
Когда в программе уже используется [Фабричный
метод](factory-method.html), но очередные изменения предполагают
введение новых типов продуктов.
:::

::: applicability-solution
В хорошей программе каждый *класс отвечает только за одну вещь*. Если
класс имеет слишком много фабричных методов, они способны затуманить его
основную функцию. Поэтому имеет смысл вынести всю логику создания
продуктов в отдельную иерархию классов, применив абстрактную фабрику.
:::
:::::::
::::::::

::: {.section .checklist}
##  Шаги реализации {#checklist}

1.  Создайте таблицу соотношений типов продуктов к вариациям семейств
    продуктов.

2.  Сведите все вариации продуктов к общим интерфейсам.

3.  Определите интерфейс абстрактной фабрики. Он должен иметь фабричные
    методы для создания каждого из типов продуктов.

4.  Создайте классы конкретных фабрик, реализовав интерфейс абстрактной
    фабрики. Этих классов должно быть столько же, сколько и вариаций
    семейств продуктов.

5.  Измените код инициализации программы так, чтобы она создавала
    определённую фабрику и передавала её в клиентский код.

6.  Замените в клиентском коде участки создания продуктов через
    конструктор вызовами соответствующих методов фабрики.
:::

:::::: {.section .pros-cons}
##  Преимущества и недостатки {#pros-cons}

::::: row
::: col-sm-6
-    Гарантирует сочетаемость создаваемых продуктов.
-    Избавляет клиентский код от привязки к конкретным классам
    продуктов.
-    Выделяет код производства продуктов в одно место, упрощая поддержку
    кода.
-    Упрощает добавление новых продуктов в программу.
-    Реализует *принцип открытости/закрытости*.
:::

::: col-sm-6
-    Усложняет код программы из-за введения множества дополнительных
    классов.
-    Требует наличия всех типов продуктов в каждой вариации.
:::
:::::
::::::

::: {.section .relations}
##  Отношения с другими паттернами {#relations}

-   Многие архитектуры начинаются с применения [Фабричного
    метода](factory-method.html) (более простого и расширяемого через
    подклассы) и эволюционируют в сторону [Абстрактной
    фабрики](abstract-factory.html), [Прототипа](prototype.html) или
    [Строителя](builder.html) (более гибких, но и более сложных).

-   [Строитель](builder.html) концентрируется на построении сложных
    объектов шаг за шагом. [Абстрактная фабрика](abstract-factory.html)
    специализируется на создании семейств связанных продуктов.
    *Строитель* возвращает продукт только после выполнения всех шагов, а
    *Абстрактная фабрика* возвращает продукт сразу же.

-   Классы [Абстрактной фабрики](abstract-factory.html) чаще всего
    реализуются с помощью [Фабричного метода](factory-method.html), хотя
    они могут быть построены и на основе [Прототипа](prototype.html).

-   [Абстрактная фабрика](abstract-factory.html) может быть использована
    вместо [Фасада](facade.html) для того, чтобы скрыть
    платформо-зависимые классы.

-   [Абстрактная фабрика](abstract-factory.html) может работать
    совместно с [Мостом](bridge.html). Это особенно полезно, если у вас
    есть абстракции, которые могут работать только с некоторыми из
    реализаций. В этом случае фабрика будет определять типы создаваемых
    абстракций и реализаций.

-   [Абстрактная фабрика](abstract-factory.html),
    [Строитель](builder.html) и [Прототип](prototype.html) могут быть
    реализованы при помощи [Одиночки](singleton.html).
:::

::: {.section .implementations}
##  Примеры реализации паттерна {#implementations}

[![Абстрактная фабрика на
C#](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](abstract-factory/csharp/example.html "Абстрактная фабрика на C#"){.prog-lang-link}
[![Абстрактная фабрика на
C++](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](abstract-factory/cpp/example.html "Абстрактная фабрика на C++"){.prog-lang-link}
[![Абстрактная фабрика на
Go](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](abstract-factory/go/example.html "Абстрактная фабрика на Go"){.prog-lang-link}
[![Абстрактная фабрика на
Java](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](abstract-factory/java/example.html "Абстрактная фабрика на Java"){.prog-lang-link}
[![Абстрактная фабрика на
PHP](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](abstract-factory/php/example.html "Абстрактная фабрика на PHP"){.prog-lang-link}
[![Абстрактная фабрика на
Python](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](abstract-factory/python/example.html "Абстрактная фабрика на Python"){.prog-lang-link}
[![Абстрактная фабрика на
Ruby](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](abstract-factory/ruby/example.html "Абстрактная фабрика на Ruby"){.prog-lang-link}
[![Абстрактная фабрика на
Rust](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](abstract-factory/rust/example.html "Абстрактная фабрика на Rust"){.prog-lang-link}
[![Абстрактная фабрика на
Swift](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](abstract-factory/swift/example.html "Абстрактная фабрика на Swift"){.prog-lang-link}
[![Абстрактная фабрика на
TypeScript](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](abstract-factory/typescript/example.html "Абстрактная фабрика на TypeScript"){.prog-lang-link}
:::

::: {.section .extras}
##  Дополнительные материалы {#extras}

-   Если вы уже слышали о *Фабрике*, *Фабричном методе* или *Абстрактной
    фабрике*, но с трудом их различаете --- почитайте нашу статью
    [Сравнение фабрик](factory-comparison.html).
:::

:::::: {#book-promo .banner-set}
::::: {.prom .banner-content .banner-bg .banner-striped .banner-with-image data-id="DP: 1: Ne vtykay" creative-id="standard-ru" position="content_bottom"}
::: banner-image
[![](../../images/patterns/banners/patterns-book-banner-1-b158a.png?id=0877cab2f0102d98cd07b50af0e5beea){width="200"
height="200" loading="lazy"
srcset="/images/patterns/banners/patterns-book-banner-1-b-2x.png?id=5572aa55e5b09e59780aca9e0ea8e44b 2x"}](book.html)
:::

::: banner-text
### Не втыкай в транспорте {#не-втыкай-в-транспорте .title}

Лучше почитай нашу книгу о паттернах проектирования.

Теперь это удобно делать даже во время поездок в общественном
транспорте.

[ Узнать больше...](book.html){.btn .btn-secondary}
:::
:::::
::::::

::: next
#### Читаем дальше

[Сравнение фабрик []{.fa .fa-arrow-right}](factory-comparison.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Фабричный метод](factory-method.html){.btn
.btn-default rel="prev"}
:::
::::::::::::::::::::::::::::::::

::::::: {.prom .banner-sidebar .banner-removable .banner-removable-patterns data-id="DP: Sidebar" creative-id="standard-sidebar-ru" position="sidebar"}
:::::: banner-inner
:::: image3d-book-right
::: {.image3d-cover style="background: #0b3752;"}
[![](../../images/patterns/book/web-cover-ruee5e.png?id=ea25feb28a6deb8aa4af6a633904a568){width="250"
height="375" loading="lazy"
srcset="/images/patterns/book/web-cover-ru-2x.png?id=95101b307f6dba409f28a417746ff3c1 2x"}](book.html)
:::
::::

::: {style="margin-top: 1rem"}
Эта статья является частью нашей электронной книги **Погружение в
Паттерны Проектирования**.

[ Узнать больше...](book.html){.btn .btn-secondary .btn-block}
:::
::::::
:::::::
::::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::::
