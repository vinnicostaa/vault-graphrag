::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [Паттерны
проектирования](../design-patterns.html){.type} / [Поведенческие
паттерны](behavioral-patterns.html){.type}
:::

# Наблюдатель {#наблюдатель .title}

::: aka
Также известен как:
[Издатель-Подписчик, ]{style="display:inline-block"}[Слушатель, ]{style="display:inline-block"}[Observer]{style="display:inline-block"}
:::

::: {.section .intent}
##  Суть паттерна {#intent}

**Наблюдатель** --- это поведенческий паттерн проектирования, который
создаёт механизм подписки, позволяющий одним объектам следить и
реагировать на события, происходящие в других объектах.

<figure class="image">
<img
src="../../images/patterns/content/observer/observerd508.png?id=6088e31e1b0d4a417506a66614dcf065"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/observer/observer.png?id=6088e31e1b0d4a417506a66614dcf065 640w,/images/patterns/content/observer/observer-1.5x.png?id=aa64f9f336354462b57bbff5c09d8269 960w,/images/patterns/content/observer/observer-2x.png?id=d5a83e115528e9fd633f04ad2650f1db 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Паттерн Наблюдатель" />
</figure>
:::

::: {.section .problem}
##  Проблема {#problem}

Представьте, что вы имеете два объекта: `Покупатель` и `Магазин`. В
магазин вот-вот должны завезти новый товар, который интересен
покупателю.

Покупатель может каждый день ходить в магазин, чтобы проверить наличие
товара. Но при этом он будет злиться, без толку тратя своё драгоценное
время.

<figure class="image">
<img
src="../../images/patterns/content/observer/observer-comic-1-ru6ac7.png?id=1eb0c12fe6cc1f4b5e0f5c61ab8340f3"
style="aspect-ratio: 2;"
srcset="/images/patterns/content/observer/observer-comic-1-ru.png?id=1eb0c12fe6cc1f4b5e0f5c61ab8340f3 600w,/images/patterns/content/observer/observer-comic-1-ru-1.5x.png?id=ddf40557556eeb9b45ac2cc510cf6693 900w,/images/patterns/content/observer/observer-comic-1-ru-2x.png?id=608c47f7f40a853486127037cfde85ff 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Постоянное посещение магазина или спам?" />
<figcaption><p>Постоянное посещение магазина или спам?</p></figcaption>
</figure>

С другой стороны, магазин может разослать спам каждому своему
покупателю. Многих это расстроит, так как товар специфический, и не всем
он нужен.

Получается конфликт: либо покупатель тратит время на периодические
проверки, либо магазин тратит ресурсы на бесполезные оповещения.
:::

::: {.section .solution}
##  Решение {#solution}

Давайте называть `Издателями` те объекты, которые содержат важное или
интересное для других состояние. Остальные объекты, которые хотят
отслеживать изменения этого состояния, назовём `Подписчиками`.

Паттерн Наблюдатель предлагает хранить внутри объекта издателя список
ссылок на объекты подписчиков, причём издатель не должен вести список
подписки самостоятельно. Он предоставит методы, с помощью которых
подписчики могли бы добавлять или убирать себя из списка.

<figure class="image">
<img
src="../../images/patterns/diagrams/observer/solution1-rua57e.png?id=b8f3a761d9fbb505d29706087f362a60"
style="aspect-ratio: 2.61;"
srcset="/images/patterns/diagrams/observer/solution1-ru.png?id=b8f3a761d9fbb505d29706087f362a60 470w,/images/patterns/diagrams/observer/solution1-ru-1.5x.png?id=227cf3bef318067dbe5527abf16723ba 705w,/images/patterns/diagrams/observer/solution1-ru-2x.png?id=3f0603674586f4cfb5718b883a34a863 940w"
sizes="(max-width: 720px) 100vw, 470px" loading="lazy" width="470"
alt="Подписка на события" />
<figcaption><p>Подписка на события.</p></figcaption>
</figure>

Теперь самое интересное. Когда в издателе будет происходить важное
событие, он будет проходиться по списку подписчиков и оповещать их об
этом, вызывая определённый метод объектов-подписчиков.

Издателю безразлично, какой класс будет иметь тот или иной подписчик,
так как все они должны следовать общему интерфейсу и иметь единый метод
оповещения.

<figure class="image">
<img
src="../../images/patterns/diagrams/observer/solution2-ru7818.png?id=e9a5e872aff022763ab28c3320cadb3b"
style="aspect-ratio: 1.24;"
srcset="/images/patterns/diagrams/observer/solution2-ru.png?id=e9a5e872aff022763ab28c3320cadb3b 460w,/images/patterns/diagrams/observer/solution2-ru-1.5x.png?id=52e2e75af10e0685983ba86c2b1c211f 690w,/images/patterns/diagrams/observer/solution2-ru-2x.png?id=dd7295d8ca6b22e43fb53fe6d8565ca2 920w"
sizes="(max-width: 720px) 100vw, 460px" loading="lazy" width="460"
alt="Оповещения о событиях" />
<figcaption><p>Оповещения о событиях.</p></figcaption>
</figure>

Увидев, как складно всё работает, вы можете выделить общий интерфейс,
описывающий методы подписки и отписки, и для всех издателей. После этого
подписчики смогут работать с разными типами издателей, а также получать
оповещения от них через один и тот же метод.
:::

::: {.section .analogy}
##  Аналогия из жизни {#analogy}

<figure class="image">
<img
src="../../images/patterns/content/observer/observer-comic-2-ruc426.png?id=47e9df81619aace155f883a8b3d27765"
style="aspect-ratio: 2;"
srcset="/images/patterns/content/observer/observer-comic-2-ru.png?id=47e9df81619aace155f883a8b3d27765 600w,/images/patterns/content/observer/observer-comic-2-ru-1.5x.png?id=badd7850fe434d4cafbf2e96c2926b52 900w,/images/patterns/content/observer/observer-comic-2-ru-2x.png?id=75926144c7ac8af556c956a044121bed 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Подписка на газеты и их доставка." />
<figcaption><p>Подписка на газеты и их доставка.</p></figcaption>
</figure>

После того как вы оформили подписку на газету или журнал, вам больше не
нужно ездить в супермаркет и проверять, не вышел ли очередной номер.
Вместо этого издательство будет присылать новые номера по почте прямо к
вам домой сразу после их выхода.

Издательство ведёт список подписчиков и знает, кому какой журнал
высылать. Вы можете в любой момент отказаться от подписки, и журнал
перестанет вам приходить.
:::

::::: {.section .structure-container}
##  Структура {#structure}

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/observer/structurea121.png?id=365b7e2b8fbecc8948f34b9f8f16f33c"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.97;"
srcset="/images/patterns/diagrams/observer/structure.png?id=365b7e2b8fbecc8948f34b9f8f16f33c 610w,/images/patterns/diagrams/observer/structure-1.5x.png?id=7f23a48c9a2d789844f912d3a0f289e6 915w,/images/patterns/diagrams/observer/structure-2x.png?id=228af9bded4d6ee6daf43a0e23cca9ff 1220w"
sizes="(max-width: 720px) 100vw, 610px" loading="lazy" width="610"
alt="Структура классов паттерна Наблюдатель" /><img
src="../../images/patterns/diagrams/observer/structure-indexed8afd.png?id=2ca2c123503ede860740af2a22bc4b4d"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.88;"
srcset="/images/patterns/diagrams/observer/structure-indexed.png?id=2ca2c123503ede860740af2a22bc4b4d 620w,/images/patterns/diagrams/observer/structure-indexed-1.5x.png?id=20b9e7765e83ca75514c202968d9e9fa 930w,/images/patterns/diagrams/observer/structure-indexed-2x.png?id=910eec855bc41f05199e510494078926 1240w"
sizes="(max-width: 720px) 100vw, 620px" loading="lazy" width="620"
alt="Структура классов паттерна Наблюдатель" />
</figure>
:::

1.  **Издатель** владеет внутренним состоянием, изменение которого
    интересно отслеживать подписчикам. Издатель содержит механизм
    подписки: список подписчиков и методы подписки/отписки.

2.  Когда внутреннее состояние издателя меняется, он оповещает своих
    подписчиков. Для этого издатель проходит по списку подписчиков и
    вызывает их метод оповещения, заданный в общем интерфейсе
    подписчиков.

3.  **Подписчик** определяет интерфейс, которым пользуется издатель для
    отправки оповещения. В большинстве случаев для этого достаточно
    единственного метода.

4.  **Конкретные подписчики** выполняют что-то в ответ на оповещение,
    пришедшее от издателя. Эти классы должны следовать общему интерфейсу
    подписчиков, чтобы издатель не зависел от конкретных классов
    подписчиков.

5.  По приходу оповещения подписчику нужно получить обновлённое
    состояние издателя. Издатель может передать это состояние через
    параметры метода оповещения. Более гибкий вариант --- передавать
    через параметры весь объект издателя, чтобы подписчик мог сам
    получить требуемые данные. Как вариант, подписчик может постоянно
    хранить ссылку на объект издателя, переданный ему в конструкторе.

6.  **Клиент** создаёт объекты издателей и подписчиков, а затем
    регистрирует подписчиков на обновления в издателях.
::::
:::::

::: {.section .pseudocode}
##  Псевдокод {#pseudocode}

В этом примере **Наблюдатель** позволяет объекту текстового редактора
оповещать другие объекты об изменениях своего состояния.

<figure class="image">
<img
src="../../images/patterns/diagrams/observer/example04a2.png?id=6d0603ab5a00e4463b81d9639cd746a2"
style="aspect-ratio: 1.19;"
srcset="/images/patterns/diagrams/observer/example.png?id=6d0603ab5a00e4463b81d9639cd746a2 510w,/images/patterns/diagrams/observer/example-1.5x.png?id=67adccd1030a38dd263bfa09867f8dbe 765w,/images/patterns/diagrams/observer/example-2x.png?id=e2838e1562325e485fc7c2828a8ca445 1020w"
sizes="(max-width: 720px) 100vw, 510px" loading="lazy" width="510"
alt="Структура классов примера паттерна Наблюдатель" />
<figcaption><p>Пример оповещения объектов о событиях в
других объектах.</p></figcaption>
</figure>

Список подписчиков составляется динамически, объекты могут как
подписываться на определённые события, так и отписываться от них прямо
во время выполнения программы.

В этой реализации редактор не ведёт список подписчиков самостоятельно, а
делегирует это вложенному объекту. Это даёт возможность использовать
механизм подписки не только в классе редактора, но и в других классах
программы.

Для добавления в программу новых подписчиков не нужно менять классы
издателей, пока они работают с подписчиками через общий интерфейс.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// Базовый класс-издатель. Содержит код управления подписчиками
// и их оповещения.
class EventManager is
    private field listeners: hash map of event types and listeners

    method subscribe(eventType, listener) is
        listeners.add(eventType, listener)

    method unsubscribe(eventType, listener) is
        listeners.remove(eventType, listener)

    method notify(eventType, data) is
        foreach (listener in listeners.of(eventType)) do
            listener.update(data)

// Конкретный класс-издатель, содержащий интересную для других
// компонентов бизнес-логику. Мы могли бы сделать его прямым
// потомком EventManager, но в реальной жизни это не всегда
// возможно (например, если у класса уже есть родитель). Поэтому
// здесь мы подключаем механизм подписки при помощи композиции.
class Editor is
    public field events: EventManager
    private field file: File

    constructor Editor() is
        events = new EventManager()

    // Методы бизнес-логики, которые оповещают подписчиков об
    // изменениях.
    method openFile(path) is
        this.file = new File(path)
        events.notify(&quot;open&quot;, file.name)

    method saveFile() is
        file.write()
        events.notify(&quot;save&quot;, file.name)
    // ...


// Общий интерфейс подписчиков. Во многих языках, поддерживающих
// функциональные типы, можно обойтись без этого интерфейса и
// конкретных классов, заменив объекты подписчиков функциями.
interface EventListener is
    method update(filename)

// Набор конкретных подписчиков. Они реализуют добавочную
// функциональность, реагируя на извещения от издателя.
class LoggingListener implements EventListener is
    private field log: File
    private field message: string

    constructor LoggingListener(log_filename, message) is
        this.log = new File(log_filename)
        this.message = message

    method update(filename) is
        log.write(replace(&#39;%s&#39;,filename,message))

class EmailAlertsListener implements EventListener is
    private field email: string
    private field message: string

    constructor EmailAlertsListener(email, message) is
        this.email = email
        this.message = message

    method update(filename) is
        system.email(email, replace(&#39;%s&#39;,filename,message))


// Приложение может сконфигурировать издателей и подписчиков как
// угодно, в зависимости от целей и окружения.
class Application is
    method config() is
        editor = new Editor()

        logger = new LoggingListener(
            &quot;/path/to/log.txt&quot;,
            &quot;Someone has opened file: %s&quot;);
        editor.events.subscribe(&quot;open&quot;, logger)

        emailAlerts = new EmailAlertsListener(
            &quot;admin@example.com&quot;,
            &quot;Someone has changed the file: %s&quot;)
        editor.events.subscribe(&quot;save&quot;, emailAlerts)</code></pre>
</figure>
:::

:::::::: {.section .applicability-container}
##  Применимость {#applicability}

::::::: applicability
::: applicability-problem
Когда после изменения состояния одного объекта требуется что-то сделать
в других, но вы не знаете наперёд, какие именно объекты должны
отреагировать.
:::

::: applicability-solution
Описанная проблема может возникнуть при разработке библиотек
пользовательского интерфейса, когда вам надо дать возможность сторонним
классам реагировать на клики по кнопкам.

Паттерн Наблюдатель позволяет любому объекту с интерфейсом подписчика
зарегистрироваться на получение оповещений о событиях, происходящих в
объектах-издателях.
:::

::: applicability-problem
Когда одни объекты должны наблюдать за другими, но только в определённых
случаях.
:::

::: applicability-solution
Издатели ведут динамические списки. Все наблюдатели могут подписываться
или отписываться от получения оповещений прямо во время выполнения
программы.
:::
:::::::
::::::::

::: {.section .checklist}
##  Шаги реализации {#checklist}

1.  Разбейте вашу функциональность на две части: независимое ядро и
    опциональные зависимые части. Независимое ядро станет издателем.
    Зависимые части станут подписчиками.

2.  Создайте интерфейс подписчиков. Обычно в нём достаточно определить
    единственный метод оповещения.

3.  Создайте интерфейс издателей и опишите в нём операции управления
    подпиской. Помните, что издатель должен работать только с общим
    интерфейсом подписчиков.

4.  Вам нужно решить, куда поместить код ведения подписки, ведь он
    обычно бывает одинаков для всех типов издателей. Самый очевидный
    способ --- вынести этот код в промежуточный абстрактный класс, от
    которого будут наследоваться все издатели.

    Но если вы интегрируете паттерн в существующие классы, то создать
    новый базовый класс может быть затруднительно. В этом случае вы
    можете поместить логику подписки во вспомогательный объект и
    делегировать ему работу из издателей.

5.  Создайте классы конкретных издателей. Реализуйте их так, чтобы после
    каждого изменения состояния они отправляли оповещения всем своим
    подписчикам.

6.  Реализуйте метод оповещения в конкретных подписчиках. Не забудьте
    предусмотреть параметры, через которые издатель мог бы отправлять
    какие-то данные, связанные с происшедшим событием.

    Возможен и другой вариант, когда подписчик, получив оповещение, сам
    возьмёт из объекта издателя нужные данные. Но в этом случае вы
    будете вынуждены привязать класс подписчика к конкретному классу
    издателя.

7.  Клиент должен создавать необходимое количество объектов подписчиков
    и подписывать их у издателей.
:::

:::::: {.section .pros-cons}
##  Преимущества и недостатки {#pros-cons}

::::: row
::: col-sm-6
-    Издатели не зависят от конкретных классов подписчиков и наоборот.
-    Вы можете подписывать и отписывать получателей на лету.
-    Реализует *принцип открытости/закрытости*.
:::

::: col-sm-6
-    Подписчики оповещаются в случайном порядке.
:::
:::::
::::::

::: {.section .relations}
##  Отношения с другими паттернами {#relations}

-   [Цепочка обязанностей](chain-of-responsibility.html),
    [Команда](command.html), [Посредник](mediator.html) и
    [Наблюдатель](observer.html) показывают различные способы работы
    отправителей запросов с их получателями:

    -   *Цепочка обязанностей* передаёт запрос последовательно через
        цепочку потенциальных получателей, ожидая, что какой-то из них
        обработает запрос.
    -   *Команда* устанавливает косвенную одностороннюю связь от
        отправителей к получателям.
    -   *Посредник* убирает прямую связь между отправителями и
        получателями, заставляя их общаться опосредованно, через себя.
    -   *Наблюдатель* передаёт запрос одновременно всем заинтересованным
        получателям, но позволяет им динамически подписываться или
        отписываться от таких оповещений.

-   Разница между [Посредником](mediator.html) и
    [Наблюдателем](observer.html) не всегда очевидна. Чаще всего они
    выступают как конкуренты, но иногда могут работать вместе.

    Цель *Посредника* --- убрать обоюдные зависимости между компонентами
    системы. Вместо этого они становятся зависимыми от самого
    посредника. С другой стороны, цель *Наблюдателя* --- обеспечить
    динамическую одностороннюю связь, в которой одни объекты косвенно
    зависят от других.

    Довольно популярна реализация *Посредника* при помощи *Наблюдателя*.
    При этом объект посредника будет выступать издателем, а все
    остальные компоненты станут подписчиками и смогут динамически
    следить за событиями, происходящими в посреднике. В этом случае
    трудно понять, чем же отличаются оба паттерна.

    Но *Посредник* имеет и другие реализации, когда отдельные компоненты
    жёстко привязаны к объекту посредника. Такой код вряд ли будет
    напоминать *Наблюдателя*, но всё же останется *Посредником*.

    Напротив, в случае реализации посредника с помощью *Наблюдателя*
    представим такую программу, в которой каждый компонент системы
    становится издателем. Компоненты могут подписываться друг на друга,
    в то же время не привязываясь к конкретным классам. Программа будет
    состоять из целой сети *Наблюдателей*, не имея центрального
    объекта-*Посредника*.
:::

::: {.section .implementations}
##  Примеры реализации паттерна {#implementations}

[![Наблюдатель на
C#](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](observer/csharp/example.html "Наблюдатель на C#"){.prog-lang-link}
[![Наблюдатель на
C++](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](observer/cpp/example.html "Наблюдатель на C++"){.prog-lang-link}
[![Наблюдатель на
Go](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](observer/go/example.html "Наблюдатель на Go"){.prog-lang-link}
[![Наблюдатель на
Java](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](observer/java/example.html "Наблюдатель на Java"){.prog-lang-link}
[![Наблюдатель на
PHP](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](observer/php/example.html "Наблюдатель на PHP"){.prog-lang-link}
[![Наблюдатель на
Python](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](observer/python/example.html "Наблюдатель на Python"){.prog-lang-link}
[![Наблюдатель на
Ruby](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](observer/ruby/example.html "Наблюдатель на Ruby"){.prog-lang-link}
[![Наблюдатель на
Rust](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](observer/rust/example.html "Наблюдатель на Rust"){.prog-lang-link}
[![Наблюдатель на
Swift](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](observer/swift/example.html "Наблюдатель на Swift"){.prog-lang-link}
[![Наблюдатель на
TypeScript](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](observer/typescript/example.html "Наблюдатель на TypeScript"){.prog-lang-link}
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

[Состояние []{.fa .fa-arrow-right}](state.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Снимок](memento.html){.btn .btn-default
rel="prev"}
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
