:::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [Паттерны
проектирования](../design-patterns.html){.type} / [Порождающие
паттерны](creational-patterns.html){.type}
:::

# Строитель {#строитель .title}

::: aka
Также известен как: [Builder]{style="display:inline-block"}
:::

::: {.section .intent}
##  Суть паттерна {#intent}

**Строитель** --- это порождающий паттерн проектирования, который
позволяет создавать сложные объекты пошагово. Строитель даёт возможность
использовать один и тот же код строительства для получения разных
представлений объектов.

<figure class="image">
<img
src="../../images/patterns/content/builder/builder-ru3fbc.png?id=99a3da99d02b7a60be71afb1f26e8e5c"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/builder/builder-ru.png?id=99a3da99d02b7a60be71afb1f26e8e5c 640w,/images/patterns/content/builder/builder-ru-1.5x.png?id=be941bf67cb1eea323e603eb63bcd680 960w,/images/patterns/content/builder/builder-ru-2x.png?id=5f47c9a77b255143d4f2c2cc2951a7e3 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Паттерн Строитель" />
</figure>
:::

::: {.section .problem}
##  Проблема {#problem}

Представьте сложный объект, требующий кропотливой пошаговой
инициализации множества полей и вложенных объектов. Код инициализации
таких объектов обычно спрятан внутри монструозного конструктора с
десятком параметров. Либо ещё хуже --- распылён по всему клиентскому
коду.

<figure class="image">
<img
src="../../images/patterns/diagrams/builder/problem16274.png?id=11e715c5c97811f848c48e0f399bb05e"
style="aspect-ratio: 1.71;"
srcset="/images/patterns/diagrams/builder/problem1.png?id=11e715c5c97811f848c48e0f399bb05e 600w,/images/patterns/diagrams/builder/problem1-1.5x.png?id=a46778018c4f0f4bbf2357940c1f5f40 900w,/images/patterns/diagrams/builder/problem1-2x.png?id=02ffbd7ad294600e42aa78989d441b4d 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Проблема со множеством классов" />
<figcaption><p>Создав кучу подклассов для всех конфигураций объектов, вы
можете излишне усложнить программу.</p></figcaption>
</figure>

Например, давайте подумаем о том, как создать объект `Дом`. Чтобы
построить стандартный дом, нужно поставить 4 стены, установить двери,
вставить пару окон и положить крышу. Но что, если вы хотите дом побольше
да посветлее, имеющий сад, бассейн и прочее добро?

Самое простое решение --- расширить класс `Дом`, создав подклассы для
всех комбинаций параметров дома. Проблема такого подхода --- это
громадное количество классов, которые вам придётся создать. Каждый новый
параметр, вроде цвета обоев или материала кровли, заставит вас создавать
всё больше и больше классов для перечисления всех возможных вариантов.

Чтобы не плодить подклассы, вы можете подойти к решению с другой
стороны. Вы можете создать гигантский конструктор `Дома`, принимающий
уйму параметров для контроля над создаваемым продуктом. Действительно,
это избавит вас от подклассов, но приведёт к другой проблеме.

<figure class="image">
<img
src="../../images/patterns/diagrams/builder/problem27ab5.png?id=2e91039b6c7d2d2df6ee519983a3b036"
style="aspect-ratio: 1.71;"
srcset="/images/patterns/diagrams/builder/problem2.png?id=2e91039b6c7d2d2df6ee519983a3b036 600w,/images/patterns/diagrams/builder/problem2-1.5x.png?id=3d302cf2762fd04d70cbb91cb54e923c 900w,/images/patterns/diagrams/builder/problem2-2x.png?id=5e7975a91c0e4f4ba960f908cc9c2ea2 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Телескопический конструктор" />
<figcaption><p>Конструктор со множеством параметров имеет свой
недостаток: не все параметры нужны большую
часть времени.</p></figcaption>
</figure>

Большая часть этих параметров будет простаивать, а вызовы конструктора
будут выглядеть монструозно из-за [длинного списка
параметров](../smells/long-parameter-list.html). К примеру, далеко не
каждый дом имеет бассейн, поэтому параметры, связанные с бассейнами,
будут простаивать бесполезно в 99% случаев.
:::

::: {.section .solution}
##  Решение {#solution}

Паттерн Строитель предлагает вынести конструирование объекта за пределы
его собственного класса, поручив это дело отдельным объектам, которые
следует называть *строителями*.

<figure class="image">
<img
src="../../images/patterns/diagrams/builder/solution13e5f.png?id=8ce82137f8935998de802cae59e00e11"
style="aspect-ratio: 1.46;"
srcset="/images/patterns/diagrams/builder/solution1.png?id=8ce82137f8935998de802cae59e00e11 410w,/images/patterns/diagrams/builder/solution1-1.5x.png?id=8ab77eb73760a75c35940bae243d43f2 615w,/images/patterns/diagrams/builder/solution1-2x.png?id=a9c2ab02f0b2aca1a7512022194dd113 820w"
sizes="(max-width: 720px) 100vw, 410px" loading="lazy" width="410"
alt="Применение паттерна Строитель" />
<figcaption><p>Строитель позволяет создавать сложные объекты пошагово.
Промежуточный результат всегда остаётся защищён.</p></figcaption>
</figure>

Паттерн предлагает разбить процесс конструирования объекта на отдельные
шаги (например, `построитьСтены`, `вставитьДвери` и другие). Чтобы
создать объект, вам нужно поочерёдно вызывать методы строителя. Причём
не нужно запускать все шаги, а только те, что нужны для производства
объекта определённой конфигурации.

Зачастую один и тот же шаг строительства может отличаться для разных
вариаций производимых объектов. Например, деревянный дом потребует
строительства стен из дерева, а каменный --- из камня.

В этом случае вы можете создать несколько классов строителей,
выполняющих одни и те же шаги по-разному. Используя этих строителей в
одном и том же строительном процессе, вы сможете получать на выходе
различные объекты.

<figure class="image">
<img
src="../../images/patterns/content/builder/builder-comic-1-ru5648.png?id=a70912d7312bb1e34249f0b52cb36e71"
style="aspect-ratio: 2;"
srcset="/images/patterns/content/builder/builder-comic-1-ru.png?id=a70912d7312bb1e34249f0b52cb36e71 600w,/images/patterns/content/builder/builder-comic-1-ru-1.5x.png?id=edd1e3321811710aa2dfa48ecb26995a 900w,/images/patterns/content/builder/builder-comic-1-ru-2x.png?id=4556de7176fb3b77e36b380c5b34dbe5 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600" />
<figcaption><p>Разные строители выполнят одну и ту же
задачу по-разному.</p></figcaption>
</figure>

Например, один строитель делает стены из дерева и стекла, другой из
камня и железа, третий из золота и бриллиантов. Вызвав одни и те же шаги
строительства, в первом случае вы получите обычный жилой дом, во
втором --- маленькую крепость, а в третьем --- роскошное жилище. Замечу,
что код, который вызывает шаги строительства, должен работать со
строителями через общий интерфейс, чтобы их можно было свободно
взаимозаменять.

#### Директор

Вы можете пойти дальше и выделить вызовы методов строителя в отдельный
класс, называемый *директором*. В этом случае директор будет задавать
порядок шагов строительства, а строитель --- выполнять их.

<figure class="image">
<img
src="../../images/patterns/content/builder/builder-comic-2-ruf40f.png?id=f7247174f7496671611c31e106bcca25"
style="aspect-ratio: 1.14;"
srcset="/images/patterns/content/builder/builder-comic-2-ru.png?id=f7247174f7496671611c31e106bcca25 343w,/images/patterns/content/builder/builder-comic-2-ru-1.5x.png?id=976643f071be17fb2436d2fb5a49076a 515w,/images/patterns/content/builder/builder-comic-2-ru-2x.png?id=fd15d73ef1022da16d9542514232b6cb 687w"
sizes="(max-width: 720px) 100vw, 343px" loading="lazy" width="343" />
<figcaption><p>Директор знает, какие шаги должен выполнить
объект-строитель, чтобы произвести продукт.</p></figcaption>
</figure>

Отдельный класс директора не является строго обязательным. Вы можете
вызывать методы строителя и напрямую из клиентского кода. Тем не менее,
директор полезен, если у вас есть несколько способов конструирования
продуктов, отличающихся порядком и наличием шагов конструирования. В
этом случае вы сможете объединить всю эту логику в одном классе.

Такая структура классов полностью скроет от клиентского кода процесс
конструирования объектов. Клиенту останется только привязать желаемого
строителя к директору, а затем получить у строителя готовый результат.
:::

::::: {.section .structure-container}
##  Структура {#structure}

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/builder/structure05e4.png?id=fe9e23559923ea0657aa5fe75efef333"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 0.79;"
srcset="/images/patterns/diagrams/builder/structure.png?id=fe9e23559923ea0657aa5fe75efef333 460w,/images/patterns/diagrams/builder/structure-1.5x.png?id=91ea8cd3137b403542c23abf63938f00 690w,/images/patterns/diagrams/builder/structure-2x.png?id=dca1b1508e23c266cbedc80ffb84311a 920w"
sizes="(max-width: 720px) 100vw, 460px" loading="lazy" width="460"
alt="Структура классов паттерна Строитель" /><img
src="../../images/patterns/diagrams/builder/structure-indexedadac.png?id=44b3d763ce91dbada5d8394ef777437f"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 0.81;"
srcset="/images/patterns/diagrams/builder/structure-indexed.png?id=44b3d763ce91dbada5d8394ef777437f 470w,/images/patterns/diagrams/builder/structure-indexed-1.5x.png?id=d57a6ff9342ea31736ea98e5066e0748 705w,/images/patterns/diagrams/builder/structure-indexed-2x.png?id=153eed9a51784cbe00d0ca8b3d6b268d 940w"
sizes="(max-width: 720px) 100vw, 470px" loading="lazy" width="470"
alt="Структура классов паттерна Строитель" />
</figure>
:::

1.  **Интерфейс строителя** объявляет шаги конструирования продуктов,
    общие для всех видов строителей.

2.  **Конкретные строители** реализуют строительные шаги, каждый
    по-своему. Конкретные строители могут производить разнородные
    объекты, не имеющие общего интерфейса.

3.  **Продукт** --- создаваемый объект. Продукты, сделанные разными
    строителями, не обязаны иметь общий интерфейс.

4.  **Директор** определяет порядок вызова строительных шагов для
    производства той или иной конфигурации продуктов.

5.  Обычно **Клиент** подаёт в конструктор директора уже готовый
    объект-строитель, и в дальнейшем данный директор использует только
    его. Но возможен и другой вариант, когда клиент передаёт строителя
    через параметр строительного метода директора. В этом случае можно
    каждый раз применять разных строителей для производства различных
    представлений объектов.
::::
:::::

::: {.section .pseudocode}
##  Псевдокод {#pseudocode}

В этом примере **Строитель** используется для пошагового конструирования
автомобилей, а также технических руководств к ним.

<figure class="image">
<img
src="../../images/patterns/diagrams/builder/example-ru827d.png?id=c3870f65681304a099137c34e8cde1c3"
style="aspect-ratio: 0.85;"
srcset="/images/patterns/diagrams/builder/example-ru.png?id=c3870f65681304a099137c34e8cde1c3 500w,/images/patterns/diagrams/builder/example-ru-1.5x.png?id=f1e562298092837bf442a7ffc9a0735d 750w,/images/patterns/diagrams/builder/example-ru-2x.png?id=fdc0495cab1a8735514aa2d473ba6de3 1000w"
sizes="(max-width: 720px) 100vw, 500px" loading="lazy" width="500"
alt="Структура классов примера паттерна Строитель" />
<figcaption><p>Пример пошагового конструирования автомобилей и
инструкций к ним.</p></figcaption>
</figure>

Автомобиль --- это сложный объект, который может быть сконфигурирован
сотней разных способов. Вместо того, чтобы настраивать автомобиль через
конструктор, мы вынесем его сборку в отдельный класс-строитель,
предусмотрев методы для конфигурации всех частей автомобиля.

Клиент может собирать автомобили, работая со строителем напрямую. Но, с
другой стороны, он может поручить это дело директору. Это объект,
который знает, какие шаги строителя нужно вызвать, чтобы получить
несколько самых популярных конфигураций автомобилей.

Но к каждому автомобилю нужно ещё и руководство, совпадающее с его
конфигурацией. Для этого мы создадим ещё один класс строителя, который
вместо конструирования автомобиля, будет печатать страницы руководства к
той детали, которую мы встраиваем в продукт. Теперь, пропустив оба типа
строителей через одни и те же шаги, мы получим автомобиль и подходящее к
нему руководство пользователя.

Очевидно, что бумажное руководство и железный автомобиль --- это две
разных вещи, не имеющих ничего общего. По этой причине мы должны
получать результат напрямую от строителей, а не от директора. Иначе нам
пришлось бы жёстко привязать директора к конкретным классам автомобилей
и руководств.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// Строитель может создавать различные продукты, используя один
// и тот же процесс строительства.
class Car is
    // Автомобили могут отличаться комплектацией: типом
    // двигателя, количеством сидений, могут иметь или не иметь
    // GPS и систему навигации и т. д. Кроме того, автомобили
    // могут быть городскими, спортивными или внедорожниками.

class Manual is
    // Руководство пользователя для данной конфигурации
    // автомобиля.


// Интерфейс строителя объявляет все возможные этапы и шаги
// конфигурации продукта.
interface Builder is
    method reset()
    method setSeats(...)
    method setEngine(...)
    method setTripComputer(...)
    method setGPS(...)

// Все конкретные строители реализуют общий интерфейс по-своему.
class CarBuilder implements Builder is
    private field car:Car
    method reset()
        // Поместить новый объект Car в поле &quot;car&quot;.
    method setSeats(...) is
        // Установить указанное количество сидений.
    method setEngine(...) is
        // Установить поданный двигатель.
    method setTripComputer(...) is
        // Установить поданную систему навигации.
    method setGPS(...) is
        // Установить или снять GPS.
    method getResult():Car is
        // Вернуть текущий объект автомобиля.

// В отличие от других порождающих паттернов, где продукты
// должны быть частью одной иерархии классов или следовать
// общему интерфейсу, строители могут создавать совершенно
// разные продукты, которые не имеют общего предка.
class CarManualBuilder implements Builder is
    private field manual:Manual
    method reset()
        // Поместить новый объект Manual в поле &quot;manual&quot;.
    method setSeats(...) is
        // Описать, сколько мест в машине.
    method setEngine(...) is
        // Добавить в руководство описание двигателя.
    method setTripComputer(...) is
        // Добавить в руководство описание системы навигации.
    method setGPS(...) is
        // Добавить в инструкцию инструкцию GPS.
    method getResult():Manual is
        // Вернуть текущий объект руководства.


// Директор знает, в какой последовательности нужно заставлять
// работать строителя, чтобы получить ту или иную версию
// продукта. Заметьте, что директор работает со строителем через
// общий интерфейс, благодаря чему он не знает тип продукта,
// который изготовляет строитель.
class Director is
    method constructSportsCar(builder: Builder) is
        builder.reset()
        builder.setSeats(2)
        builder.setEngine(new SportEngine())
        builder.setTripComputer(true)
        builder.setGPS(true)


// Директор получает объект конкретного строителя от клиента
// (приложения). Приложение само знает, какого строителя нужно
// использовать, чтобы получить определённый продукт.
class Application is
    method makeCar() is
        director = new Director()

        CarBuilder builder = new CarBuilder()
        director.constructSportsCar(builder)
        Car car = builder.getResult()

        CarManualBuilder builder = new CarManualBuilder()
        director.constructSportsCar(builder)

        // Готовый продукт возвращает строитель, так как
        // директор чаще всего не знает и не зависит от
        // конкретных классов строителей и продуктов.
        Manual manual = builder.getResult()</code></pre>
</figure>
:::

:::::::::: {.section .applicability-container}
##  Применимость {#applicability}

::::::::: applicability
::: applicability-problem
Когда вы хотите избавиться от «телескопического конструктора».
:::

::: applicability-solution
Допустим, у вас есть один конструктор с десятью опциональными
параметрами. Его неудобно вызывать, поэтому вы создали ещё десять
конструкторов с меньшим количеством параметров. Всё, что они делают ---
это переадресуют вызов к базовому конструктору, подавая какие-то
значения по умолчанию в параметры, которые пропущены в них самих.

<figure class="code">
<pre class="code" lang="java"><code>class Pizza {
    Pizza(int size) { ... }
    Pizza(int size, boolean cheese) { ... }
    Pizza(int size, boolean cheese, boolean pepperoni) { ... }
    // ...</code></pre>
<figcaption><p>Такого монстра можно создать только в языках, имеющих
механизм перегрузки методов, например, C# или Java.</p></figcaption>
</figure>

Паттерн Строитель позволяет собирать объекты пошагово, вызывая только те
шаги, которые вам нужны. А значит, больше не нужно пытаться «запихнуть»
в конструктор все возможные опции продукта.
:::

::: applicability-problem
Когда ваш код должен создавать разные представления какого-то объекта.
Например, деревянные и железобетонные дома.
:::

::: applicability-solution
Строитель можно применить, если создание нескольких представлений
объекта состоит из одинаковых этапов, которые отличаются в деталях.

Интерфейс строителей определит все возможные этапы конструирования.
Каждому представлению будет соответствовать собственный класс-строитель.
А порядок этапов строительства будет задавать класс-директор.
:::

::: applicability-problem
Когда вам нужно собирать сложные составные объекты, например, деревья
[Компоновщика](composite.html).
:::

::: applicability-solution
Строитель конструирует объекты пошагово, а не за один проход. Более
того, шаги строительства можно выполнять рекурсивно. А без этого не
построить древовидную структуру, вроде Компоновщика.

Заметьте, что Строитель не позволяет посторонним объектам иметь доступ к
конструируемому объекту, пока тот не будет полностью готов. Это
предохраняет клиентский код от получения незаконченных «битых» объектов.
:::
:::::::::
::::::::::

::: {.section .checklist}
##  Шаги реализации {#checklist}

1.  Убедитесь в том, что создание разных представлений объекта можно
    свести к общим шагам.

2.  Опишите эти шаги в общем интерфейсе строителей.

3.  Для каждого из представлений объекта-продукта создайте по одному
    классу-строителю и реализуйте их методы строительства.

    Не забудьте про метод получения результата. Обычно конкретные
    строители определяют собственные методы получения результата
    строительства. Вы не можете описать эти методы в интерфейсе
    строителей, поскольку продукты не обязательно должны иметь общий
    базовый класс или интерфейс. Но вы всегда сможете добавить метод
    получения результата в общий интерфейс, если ваши строители
    производят однородные продукты с общим предком.

4.  Подумайте о создании класса директора. Его методы будут создавать
    различные конфигурации продуктов, вызывая разные шаги одного и того
    же строителя.

5.  Клиентский код должен будет создавать и объекты строителей, и объект
    директора. Перед началом строительства клиент должен связать
    определённого строителя с директором. Это можно сделать либо через
    конструктор, либо через сеттер, либо подав строителя напрямую в
    строительный метод директора.

6.  Результат строительства можно вернуть из директора, но только если
    метод возврата продукта удалось поместить в общий интерфейс
    строителей. Иначе вы жёстко привяжете директора к конкретным классам
    строителей.
:::

:::::: {.section .pros-cons}
##  Преимущества и недостатки {#pros-cons}

::::: row
::: col-sm-6
-    Позволяет создавать продукты пошагово.
-    Позволяет использовать один и тот же код для создания различных
    продуктов.
-    Изолирует сложный код сборки продукта от его основной
    бизнес-логики.
:::

::: col-sm-6
-    Усложняет код программы из-за введения дополнительных классов.
-    Клиент будет привязан к конкретным классам строителей, так как в
    интерфейсе директора может не быть метода получения результата.
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

-   [Строитель](builder.html) позволяет пошагово сооружать дерево
    [Компоновщика](composite.html).

-   Паттерн [Строитель](builder.html) может быть построен в виде
    [Моста](bridge.html): *директор* будет играть роль абстракции, а
    *строители* --- реализации.

-   [Абстрактная фабрика](abstract-factory.html),
    [Строитель](builder.html) и [Прототип](prototype.html) могут быть
    реализованы при помощи [Одиночки](singleton.html).
:::

::: {.section .implementations}
##  Примеры реализации паттерна {#implementations}

[![Строитель на
C#](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](builder/csharp/example.html "Строитель на C#"){.prog-lang-link}
[![Строитель на
C++](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](builder/cpp/example.html "Строитель на C++"){.prog-lang-link}
[![Строитель на
Go](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](builder/go/example.html "Строитель на Go"){.prog-lang-link}
[![Строитель на
Java](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](builder/java/example.html "Строитель на Java"){.prog-lang-link}
[![Строитель на
PHP](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](builder/php/example.html "Строитель на PHP"){.prog-lang-link}
[![Строитель на
Python](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](builder/python/example.html "Строитель на Python"){.prog-lang-link}
[![Строитель на
Ruby](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](builder/ruby/example.html "Строитель на Ruby"){.prog-lang-link}
[![Строитель на
Rust](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](builder/rust/example.html "Строитель на Rust"){.prog-lang-link}
[![Строитель на
Swift](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](builder/swift/example.html "Строитель на Swift"){.prog-lang-link}
[![Строитель на
TypeScript](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](builder/typescript/example.html "Строитель на TypeScript"){.prog-lang-link}
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

[Прототип []{.fa .fa-arrow-right}](prototype.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Сравнение фабрик](factory-comparison.html){.btn
.btn-default rel="prev"}
:::
:::::::::::::::::::::::::::::::::

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
:::::::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::::::
