::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [Паттерны
проектирования](../design-patterns.html){.type} / [Поведенческие
паттерны](behavioral-patterns.html){.type}
:::

# Цепочка обязанностей {#цепочка-обязанностей .title}

::: aka
Также известен как: [CoR, ]{style="display:inline-block"}[Chain of
Command, ]{style="display:inline-block"}[Chain of
Responsibility]{style="display:inline-block"}
:::

::: {.section .intent}
##  Суть паттерна {#intent}

**Цепочка обязанностей** --- это поведенческий паттерн проектирования,
который позволяет передавать запросы последовательно по цепочке
обработчиков. Каждый последующий обработчик решает, может ли он
обработать запрос сам и стоит ли передавать запрос дальше по цепи.

<figure class="image">
<img
src="../../images/patterns/content/chain-of-responsibility/chain-of-responsibilityf2d1.png?id=56c10d0dc712546cc283cfb3fb463458"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/chain-of-responsibility/chain-of-responsibility.png?id=56c10d0dc712546cc283cfb3fb463458 640w,/images/patterns/content/chain-of-responsibility/chain-of-responsibility-1.5x.png?id=770c03ad168fa797ec8ed4556ddf0b5c 960w,/images/patterns/content/chain-of-responsibility/chain-of-responsibility-2x.png?id=cc104b0a00a410f37fb39da80f392b88 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Паттерн Цепочка обязанностей" />
</figure>
:::

::: {.section .problem}
##  Проблема {#problem}

Представьте, что вы делаете систему приёма онлайн-заказов. Вы хотите
ограничить к ней доступ так, чтобы только авторизованные пользователи
могли создавать заказы. Кроме того, определённые пользователи, владеющие
правами администратора, должны иметь полный доступ к заказам.

Вы быстро сообразили, что эти проверки нужно выполнять последовательно.
Ведь пользователя можно попытаться «залогинить» в систему, если его
запрос содержит логин и пароль. Но если такая попытка не удалась, то
проверять расширенные права доступа попросту не имеет смысла.

<figure class="image">
<img
src="../../images/patterns/diagrams/chain-of-responsibility/problem1-rufceb.png?id=85981afa10f1e1f5bfa51bf8f18ff774"
style="aspect-ratio: 2.5;"
srcset="/images/patterns/diagrams/chain-of-responsibility/problem1-ru.png?id=85981afa10f1e1f5bfa51bf8f18ff774 600w,/images/patterns/diagrams/chain-of-responsibility/problem1-ru-1.5x.png?id=47a72988cf0767dd644b612fe230a3fb 900w,/images/patterns/diagrams/chain-of-responsibility/problem1-ru-2x.png?id=ceec9e9d1ba1597bb01c6d8328aaf995 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Проблема, которую решает Цепочка обязанностей" />
<figcaption><p>Запрос проходит ряд проверок перед доступом в
систему заказов.</p></figcaption>
</figure>

На протяжении следующих нескольких месяцев вам пришлось добавить ещё
несколько таких последовательных проверок.

-   Кто-то резонно заметил, что неплохо бы проверять данные,
    передаваемые в запросе перед тем, как вносить их в систему --- вдруг
    запрос содержит данные о покупке несуществующих продуктов.

-   Кто-то предложил блокировать массовые отправки формы с одним и тем
    же логином, чтобы предотвратить подбор паролей ботами.

-   Кто-то заметил, что форму заказа неплохо бы доставать из кеша, если
    она уже была однажды показана.

<figure class="image">
<img
src="../../images/patterns/diagrams/chain-of-responsibility/problem2-ru1ba6.png?id=400543b25c39681d2cbdb5b1bb8cf75b"
style="aspect-ratio: 1.65;"
srcset="/images/patterns/diagrams/chain-of-responsibility/problem2-ru.png?id=400543b25c39681d2cbdb5b1bb8cf75b 610w,/images/patterns/diagrams/chain-of-responsibility/problem2-ru-1.5x.png?id=7200893ab2fbd91da9607958f4cd8ff6 915w,/images/patterns/diagrams/chain-of-responsibility/problem2-ru-2x.png?id=69263ba7c95ed20e7da24775e1158ece 1220w"
sizes="(max-width: 720px) 100vw, 610px" loading="lazy" width="610"
alt="Со временем код проверок становится всё более запутанным" />
<figcaption><p>Со временем код проверок становится всё
более запутанным.</p></figcaption>
</figure>

С каждой новой «фичей» код проверок, выглядящий как большой клубок
условных операторов, всё больше и больше раздувался. При изменении
одного правила приходилось трогать код всех проверок. А для того, чтобы
применить проверки к другим ресурсам, пришлось продублировать их код в
других классах.

Поддерживать такой код стало не только очень хлопотно, но и затратно. И
вот в один прекрасный день вы получаете задачу рефакторинга\...
:::

::: {.section .solution}
##  Решение {#solution}

Как и многие другие поведенческие паттерны, Цепочка обязанностей
базируется на том, чтобы превратить отдельные поведения в объекты. В
нашем случае каждая проверка переедет в отдельный класс с единственным
методом выполнения. Данные запроса, над которым происходит проверка,
будут передаваться в метод как аргументы.

А теперь по-настоящему важный этап. Паттерн предлагает связать объекты
обработчиков в одну цепь. Каждый из них будет иметь ссылку на следующий
обработчик в цепи. Таким образом, при получении запроса обработчик
сможет не только сам что-то с ним сделать, но и передать обработку
следующему объекту в цепочке.

Передавая запросы в первый обработчик цепочки, вы можете быть уверены,
что все объекты в цепи смогут его обработать. При этом длина цепочки не
имеет никакого значения.

И последний штрих. Обработчик не обязательно должен передавать запрос
дальше, причём эта особенность может быть использована по-разному.

В примере с фильтрацией доступа обработчики прерывают дальнейшие
проверки, если текущая проверка не прошла. Ведь нет смысла тратить
попусту ресурсы, если и так понятно, что с запросом что-то не так.

<figure class="image">
<img
src="../../images/patterns/diagrams/chain-of-responsibility/solution1-rudb57.png?id=c1c1477e534ab88833a6183a9c2a1193"
style="aspect-ratio: 4;"
srcset="/images/patterns/diagrams/chain-of-responsibility/solution1-ru.png?id=c1c1477e534ab88833a6183a9c2a1193 640w,/images/patterns/diagrams/chain-of-responsibility/solution1-ru-1.5x.png?id=938d4faddfa236f1b7239525899610aa 960w,/images/patterns/diagrams/chain-of-responsibility/solution1-ru-2x.png?id=4f94c55000334239f94f0e54925f19b8 1280w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="640"
alt="Обработчики следуют в цепочке один за другим" />
<figcaption><p>Обработчики следуют в цепочке один
за другим.</p></figcaption>
</figure>

Но есть и другой подход, при котором обработчики прерывают цепь только
когда они *могут* обработать запрос. В этом случае запрос движется по
цепи, пока не найдётся обработчик, который может его обработать. Очень
часто такой подход используется для передачи событий, создаваемых
классами графического интерфейса в результате взаимодействия с
пользователем.

Например, когда пользователь кликает по кнопке, программа выстраивает
цепочку из объекта этой кнопки, всех её родительских элементов и общего
окна приложения на конце. Событие клика передаётся по этой цепи до тех
пор, пока не найдётся объект, способный его обработать. Этот пример
примечателен ещё и тем, что цепочку всегда можно выделить из древовидной
структуры объектов, в которую обычно и свёрнуты элементы
пользовательского интерфейса.

<figure class="image">
<img
src="../../images/patterns/diagrams/chain-of-responsibility/solution2-ru4e42.png?id=cb0aeb4420e7ab9add900e13c82b072d"
style="aspect-ratio: 1.73;"
srcset="/images/patterns/diagrams/chain-of-responsibility/solution2-ru.png?id=cb0aeb4420e7ab9add900e13c82b072d 520w,/images/patterns/diagrams/chain-of-responsibility/solution2-ru-1.5x.png?id=372a4f65c40296b79ea0fe89ad051ba0 780w,/images/patterns/diagrams/chain-of-responsibility/solution2-ru-2x.png?id=ac4da36da79c8a8a006f331ee2746292 1040w"
sizes="(max-width: 720px) 100vw, 520px" loading="lazy" width="520"
alt="Цепочку можно выделить даже из дерева объектов" />
<figcaption><p>Цепочку можно выделить даже из
дерева объектов.</p></figcaption>
</figure>

Очень важно, чтобы все объекты цепочки имели общий интерфейс. Обычно
каждому конкретному обработчику достаточно знать только то, что
следующий объект в цепи имеет метод `выполнить`. Благодаря этому связи
между объектами цепочки будут более гибкими. Кроме того, вы сможете
формировать цепочки на лету из разнообразных объектов, не привязываясь к
конкретным классам.
:::

::: {.section .analogy}
##  Аналогия из жизни {#analogy}

<figure class="image">
<img
src="../../images/patterns/content/chain-of-responsibility/chain-of-responsibility-comic-1-ru7700.png?id=36f613d78baebc27d21d2a94deb3992f"
style="aspect-ratio: 2;"
srcset="/images/patterns/content/chain-of-responsibility/chain-of-responsibility-comic-1-ru.png?id=36f613d78baebc27d21d2a94deb3992f 600w,/images/patterns/content/chain-of-responsibility/chain-of-responsibility-comic-1-ru-1.5x.png?id=ed6517eb09cccdf5e6fa75fe427c6eb3 900w,/images/patterns/content/chain-of-responsibility/chain-of-responsibility-comic-1-ru-2x.png?id=d7f6e32b22e12f3a6a4f271838468ef8 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Пример общения с поддержкой" />
<figcaption><p>Пример общения с поддержкой.</p></figcaption>
</figure>

Вы купили новую видеокарту. Она автоматически определилась и заработала
под Windows, но в вашей любимой Ubuntu «завести» её не удалось. Со
слабой надеждой вы звоните в службу поддержки.

Первым вы слышите голос автоответчика, предлагающий выбор из десятка
стандартных решений. Ни один из вариантов не подходит, и робот соединяет
вас с живым оператором.

Увы, но рядовой оператор поддержки умеет общаться только заученными
фразами и давать шаблонные ответы. После очередного предложения
«выключить и включить компьютер» вы просите связать вас с настоящими
инженерами.

Оператор перебрасывает звонок дежурному инженеру, изнывающему от скуки в
своей каморке. Уж он-то знает, как вам помочь! Инженер рассказывает вам,
где скачать подходящие драйвера и как настроить их под Ubuntu. Запрос
удовлетворён. Вы кладёте трубку.
:::

::::: {.section .structure-container}
##  Структура {#structure}

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/chain-of-responsibility/structure03c9.png?id=848f0fc8dca57a44974d63f8181f5406"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 0.93;"
srcset="/images/patterns/diagrams/chain-of-responsibility/structure.png?id=848f0fc8dca57a44974d63f8181f5406 380w,/images/patterns/diagrams/chain-of-responsibility/structure-1.5x.png?id=49dfbae38f146af7319f80dce9ea7e2a 570w,/images/patterns/diagrams/chain-of-responsibility/structure-2x.png?id=bb837faaac88e7f2a16f751d0beaa201 760w"
sizes="(max-width: 720px) 100vw, 380px" loading="lazy" width="380"
alt="Структура классов паттерна Цепочка обязанностей" /><img
src="../../images/patterns/diagrams/chain-of-responsibility/structure-indexedfcb6.png?id=e13a5bf44f9ca47299223116af77cbef"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 0.88;"
srcset="/images/patterns/diagrams/chain-of-responsibility/structure-indexed.png?id=e13a5bf44f9ca47299223116af77cbef 380w,/images/patterns/diagrams/chain-of-responsibility/structure-indexed-1.5x.png?id=ca660e2cb697b512aadc07fdd3e109cd 570w,/images/patterns/diagrams/chain-of-responsibility/structure-indexed-2x.png?id=4f27e2c48e635f45a78472d707a8df3c 760w"
sizes="(max-width: 720px) 100vw, 380px" loading="lazy" width="380"
alt="Структура классов паттерна Цепочка обязанностей" />
</figure>
:::

1.  **Обработчик** определяет общий для всех конкретных обработчиков
    интерфейс. Обычно достаточно описать единственный метод обработки
    запросов, но иногда здесь может быть объявлен и метод выставления
    следующего обработчика.

2.  **Базовый обработчик** --- опциональный класс, который позволяет
    избавиться от дублирования одного и того же кода во всех конкретных
    обработчиках.

    Обычно этот класс имеет поле для хранения ссылки на следующий
    обработчик в цепочке. Клиент связывает обработчики в цепь, подавая
    ссылку на следующий обработчик через конструктор или сеттер поля.
    Также здесь можно реализовать базовый метод обработки, который бы
    просто перенаправлял запрос следующему обработчику, проверив его
    наличие.

3.  **Конкретные обработчики** содержат код обработки запросов. При
    получении запроса каждый обработчик решает, может ли он обработать
    запрос, а также стоит ли передать его следующему объекту.

    В большинстве случаев обработчики могут работать сами по себе и быть
    неизменяемыми, получив все нужные детали через параметры
    конструктора.

4.  **Клиент** может либо сформировать цепочку обработчиков единожды,
    либо перестраивать её динамически, в зависимости от логики
    программы. Клиент может отправлять запросы любому из объектов
    цепочки, не обязательно первому из них.
::::
:::::

::: {.section .pseudocode}
##  Псевдокод {#pseudocode}

В этом примере **Цепочка обязанностей** отвечает за показ контекстной
помощи для активных элементов пользовательского интерфейса.

<figure class="image">
<img
src="../../images/patterns/diagrams/chain-of-responsibility/example-ru4e05.png?id=9e9b6643933273864826aadedb8e23f8"
style="aspect-ratio: 1.09;"
srcset="/images/patterns/diagrams/chain-of-responsibility/example-ru.png?id=9e9b6643933273864826aadedb8e23f8 610w,/images/patterns/diagrams/chain-of-responsibility/example-ru-1.5x.png?id=846d06de5594b645da909025c9937b2b 915w,/images/patterns/diagrams/chain-of-responsibility/example-ru-2x.png?id=78d62a61aa19796018c791b444d50686 1220w"
sizes="(max-width: 720px) 100vw, 610px" loading="lazy" width="610"
alt="Структура классов примера паттерна Цепочка обязанностей" />
<figcaption><p>Графический интерфейс построен с помощью компоновщика,
где у каждого элемента есть ссылка на свой элемент-контейнер. Цепочку
можно выстроить, пройдясь по всем контейнерам, в которые
вложен элемент.</p></figcaption>
</figure>

Графический интерфейс приложения обычно структурирован в виде дерева.
Класс `Диалог`, отображающий всё окно приложения --- это корень дерева.
Диалог содержит `Панели`, которые, в свою очередь, могут содержать либо
другие вложенные панели, либо простые элементы, вроде `Кнопок`.

Простые элементы могут показывать небольшие подсказки, если для них
указан текст помощи. Но есть и более сложные компоненты, для которых
этот способ демонстрации помощи слишком прост. Они определяют
собственный способ отображения контекстной помощи.

<figure class="image">
<img
src="../../images/patterns/diagrams/chain-of-responsibility/example2-ru557a.png?id=e7efb8d48e671549f58007043efed006"
style="aspect-ratio: 0.81;"
srcset="/images/patterns/diagrams/chain-of-responsibility/example2-ru.png?id=e7efb8d48e671549f58007043efed006 250w,/images/patterns/diagrams/chain-of-responsibility/example2-ru-1.5x.png?id=255291f043d2728762a18756a55bad4f 375w,/images/patterns/diagrams/chain-of-responsibility/example2-ru-2x.png?id=c532b84577037132b60c68507bedd05f 500w"
sizes="(max-width: 720px) 100vw, 250px" loading="lazy" width="250"
alt="Структура классов примера паттерна Цепочка обязанностей" />
<figcaption><p>Пример вызова контекстной помощи в цепочке
объектов UI.</p></figcaption>
</figure>

Когда пользователь наводит указатель мыши на элемент и жмёт клавишу
`F1`, приложение шлёт этому элементу запрос на показ помощи. Если он не
содержит никакой справочной информации, запрос путешествует далее по
списку контейнера элемента, пока не находится тот, который способен
отобразить помощь.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// Интерфейс обработчиков.
interface ComponentWithContextualHelp is
    method showHelp()


// Базовый класс простых компонентов.
abstract class Component implements ComponentWithContextualHelp is
    field tooltipText: string

    // Контейнер, содержащий компонент, служит в качестве
    // следующего звена цепочки.
    protected field container: Container

    // Базовое поведение компонента заключается в том, чтобы
    // показать всплывающую подсказку, если для неё задан текст.
    // В обратном случае — перенаправить запрос своему
    // контейнеру, если тот существует.
    method showHelp() is
        if (tooltipText != null)
            // Показать подсказку.
        else
            container.showHelp()


// Контейнеры могут включать в себя как простые компоненты, так
// и другие контейнеры. Здесь формируются связи цепочки. Класс
// контейнера унаследует метод showHelp от своего родителя —
// базового компонента.
abstract class Container extends Component is
    protected field children: array of Component

    method add(child) is
        children.add(child)
        child.container = this


// Большинство примитивных компонентов устроит базовое поведение
// показа помощи через подсказку, которое они унаследуют из
// класса Component.
class Button extends Component is
    // ...

// Но сложные компоненты могут переопределять метод показа
// помощи по-своему. Но и в этом случае они всегда могут
// вернуться к базовой реализации, вызвав метод родителя.
class Panel extends Container is
    field modalHelpText: string

    method showHelp() is
        if (modalHelpText != null)
            // Показать модальное окно с помощью.
        else
            super.showHelp()

// ...то же, что и выше...
class Dialog extends Container is
    field wikiPageURL: string

    method showHelp() is
        if (wikiPageURL != null)
            // Открыть страницу Wiki в браузере.
        else
            super.showHelp()


// Клиентский код.
class Application is
    // Каждое приложение конфигурирует цепочку по-своему.
    method createUI() is
        dialog = new Dialog(&quot;Budget Reports&quot;)
        dialog.wikiPageURL = &quot;http://...&quot;
        panel = new Panel(0, 0, 400, 800)
        panel.modalHelpText = &quot;This panel does...&quot;
        ok = new Button(250, 760, 50, 20, &quot;OK&quot;)
        ok.tooltipText = &quot;This is an OK button that...&quot;
        cancel = new Button(320, 760, 50, 20, &quot;Cancel&quot;)
        // ...
        panel.add(ok)
        panel.add(cancel)
        dialog.add(panel)

    // Представьте, что здесь произойдёт.
    method onF1KeyPress() is
        component = this.getComponentAtMouseCoords()
        component.showHelp()</code></pre>
</figure>
:::

:::::::::: {.section .applicability-container}
##  Применимость {#applicability}

::::::::: applicability
::: applicability-problem
Когда программа должна обрабатывать разнообразные запросы несколькими
способами, но заранее неизвестно, какие конкретно запросы будут
приходить и какие обработчики для них понадобятся.
:::

::: applicability-solution
С помощью Цепочки обязанностей вы можете связать потенциальных
обработчиков в одну цепь и при получении запроса поочерёдно спрашивать
каждого из них, не хочет ли он обработать запрос.
:::

::: applicability-problem
Когда важно, чтобы обработчики выполнялись один за другим в строгом
порядке.
:::

::: applicability-solution
Цепочка обязанностей позволяет запускать обработчиков последовательно
один за другим в том порядке, в котором они находятся в цепочке.
:::

::: applicability-problem
Когда набор объектов, способных обработать запрос, должен задаваться
динамически.
:::

::: applicability-solution
В любой момент вы можете вмешаться в существующую цепочку и
переназначить связи так, чтобы убрать или добавить новое звено.
:::
:::::::::
::::::::::

::: {.section .checklist}
##  Шаги реализации {#checklist}

1.  Создайте интерфейс обработчика и опишите в нём основной метод
    обработки.

    Продумайте, в каком виде клиент должен передавать данные запроса в
    обработчик. Самый гибкий способ --- превратить данные запроса в
    объект и передавать его целиком через параметры метода обработчика.

2.  Имеет смысл создать абстрактный базовый класс обработчиков, чтобы не
    дублировать реализацию метода получения следующего обработчика во
    всех конкретных обработчиках.

    Добавьте в базовый обработчик поле для хранения ссылки на следующий
    объект цепочки. Устанавливайте начальное значение этого поля через
    конструктор. Это сделает объекты обработчиков неизменяемыми. Но если
    программа предполагает динамическую перестройку цепочек, можете
    добавить и сеттер для поля.

    Реализуйте базовый метод обработки так, чтобы он перенаправлял
    запрос следующему объекту, проверив его наличие. Это позволит
    полностью скрыть поле-ссылку от подклассов, дав им возможность
    передавать запросы дальше по цепи, обращаясь к родительской
    реализации метода.

3.  Один за другим создайте классы конкретных обработчиков и реализуйте
    в них методы обработки запросов. При получении запроса каждый
    обработчик должен решить:

    -   Может ли он обработать запрос или нет?
    -   Следует ли передать запрос следующему обработчику или нет?

4.  Клиент может собирать цепочку обработчиков самостоятельно, опираясь
    на свою бизнес-логику, либо получать уже готовые цепочки извне. В
    последнем случае цепочки собираются фабричными объектами, опираясь
    на конфигурацию приложения или параметры окружения.

5.  Клиент может посылать запросы любому обработчику в цепи, а не только
    первому. Запрос будет передаваться по цепочке до тех пор, пока
    какой-то обработчик не откажется передавать его дальше, либо когда
    будет достигнут конец цепи.

6.  Клиент должен знать о динамической природе цепочки и быть готов к
    таким случаям:

    -   Цепочка может состоять из единственного объекта.
    -   Запросы могут не достигать конца цепи.
    -   Запросы могут достигать конца, оставаясь необработанными.
:::

:::::: {.section .pros-cons}
##  Преимущества и недостатки {#pros-cons}

::::: row
::: col-sm-6
-    Уменьшает зависимость между клиентом и обработчиками.
-    Реализует *принцип единственной обязанности*.
-    Реализует *принцип открытости/закрытости*.
:::

::: col-sm-6
-    Запрос может остаться никем не обработанным.
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

-   [Цепочку обязанностей](chain-of-responsibility.html) часто
    используют вместе с [Компоновщиком](composite.html). В этом случае
    запрос передаётся от дочерних компонентов к их родителям.

-   Обработчики в [Цепочке обязанностей](chain-of-responsibility.html)
    могут быть выполнены в виде [Команд](command.html). В этом случае
    множество разных операций может быть выполнено над одним и тем же
    контекстом, коим является запрос.

    Но есть и другой подход, в котором сам запрос является *Командой*,
    посланной по цепочке объектов. В этом случае одна и та же операция
    может быть выполнена над множеством разных контекстов,
    представленных в виде цепочки.

-   [Цепочка обязанностей](chain-of-responsibility.html) и
    [Декоратор](decorator.html) имеют очень похожие структуры. Оба
    паттерна базируются на принципе рекурсивного выполнения операции
    через серию связанных объектов. Но есть и несколько важных отличий.

    Обработчики в *Цепочке обязанностей* могут выполнять произвольные
    действия, независимые друг от друга, а также в любой момент
    прерывать дальнейшую передачу по цепочке. С другой стороны
    *Декораторы* расширяют какое-то определённое действие, не ломая
    интерфейс базовой операции и не прерывая выполнение остальных
    декораторов.
:::

::: {.section .implementations}
##  Примеры реализации паттерна {#implementations}

[![Цепочка обязанностей на
C#](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](chain-of-responsibility/csharp/example.html "Цепочка обязанностей на C#"){.prog-lang-link}
[![Цепочка обязанностей на
C++](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](chain-of-responsibility/cpp/example.html "Цепочка обязанностей на C++"){.prog-lang-link}
[![Цепочка обязанностей на
Go](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](chain-of-responsibility/go/example.html "Цепочка обязанностей на Go"){.prog-lang-link}
[![Цепочка обязанностей на
Java](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](chain-of-responsibility/java/example.html "Цепочка обязанностей на Java"){.prog-lang-link}
[![Цепочка обязанностей на
PHP](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](chain-of-responsibility/php/example.html "Цепочка обязанностей на PHP"){.prog-lang-link}
[![Цепочка обязанностей на
Python](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](chain-of-responsibility/python/example.html "Цепочка обязанностей на Python"){.prog-lang-link}
[![Цепочка обязанностей на
Ruby](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](chain-of-responsibility/ruby/example.html "Цепочка обязанностей на Ruby"){.prog-lang-link}
[![Цепочка обязанностей на
Rust](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](chain-of-responsibility/rust/example.html "Цепочка обязанностей на Rust"){.prog-lang-link}
[![Цепочка обязанностей на
Swift](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](chain-of-responsibility/swift/example.html "Цепочка обязанностей на Swift"){.prog-lang-link}
[![Цепочка обязанностей на
TypeScript](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](chain-of-responsibility/typescript/example.html "Цепочка обязанностей на TypeScript"){.prog-lang-link}
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

[Команда []{.fa .fa-arrow-right}](command.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Поведенческие
паттерны](behavioral-patterns.html){.btn .btn-default rel="prev"}
:::
::::::::::::::::::::::::::::::::::

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
::::::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::::::
