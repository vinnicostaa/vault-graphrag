::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [Паттерны
проектирования](../design-patterns.html){.type} / [Порождающие
паттерны](creational-patterns.html){.type}
:::

# Фабричный метод {#фабричный-метод .title}

::: aka
Также известен как: [Виртуальный
конструктор, ]{style="display:inline-block"}[Factory
Method]{style="display:inline-block"}
:::

::: {.section .intent}
##  Суть паттерна {#intent}

**Фабричный метод** --- это порождающий паттерн проектирования, который
определяет общий интерфейс для создания объектов в суперклассе, позволяя
подклассам изменять тип создаваемых объектов.

<figure class="image">
<img
src="../../images/patterns/content/factory-method/factory-method-ru02b5.png?id=501c10632ffb19356386ea8676d3507c"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/factory-method/factory-method-ru.png?id=501c10632ffb19356386ea8676d3507c 640w,/images/patterns/content/factory-method/factory-method-ru-1.5x.png?id=0d41a2841a3a148ecd83f6f4159874cd 960w,/images/patterns/content/factory-method/factory-method-ru-2x.png?id=2669e862faec428d1b3d7b563e4845ee 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Паттерн Фабричный метод" />
</figure>
:::

::: {.section .problem}
##  Проблема {#problem}

Представьте, что вы создаёте программу управления грузовыми перевозками.
Сперва вы рассчитываете перевозить товары только на автомобилях. Поэтому
весь ваш код работает с объектами класса `Грузовик`.

В какой-то момент ваша программа становится настолько известной, что
морские перевозчики выстраиваются в очередь и просят добавить поддержку
морской логистики в программу.

<figure class="image">
<img
src="../../images/patterns/diagrams/factory-method/problem1-en51bc.png?id=de638561be0bbb1025ada6bfcb815def"
style="aspect-ratio: 2.4;"
srcset="/images/patterns/diagrams/factory-method/problem1-en.png?id=de638561be0bbb1025ada6bfcb815def 600w,/images/patterns/diagrams/factory-method/problem1-en-1.5x.png?id=a46ffaf7114d367522cabda6012404bf 900w,/images/patterns/diagrams/factory-method/problem1-en-2x.png?id=9a4959d9dde4edadf809be33d3da0c74 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Проблема с добавлением нового класса в программу" />
<figcaption><p>Добавить новый класс не так-то просто, если весь код уже
завязан на конкретные классы.</p></figcaption>
</figure>

Отличные новости, правда?! Но как насчёт кода? Большая часть
существующего кода жёстко привязана к классам `Грузовиков`. Чтобы
добавить в программу классы морских `Судов`, понадобится перелопатить
всю программу. Более того, если вы потом решите добавить в программу ещё
один вид транспорта, то всю эту работу придётся повторить.

В итоге вы получите ужасающий код, наполненный условными операторами,
которые выполняют то или иное действие, в зависимости от класса
транспорта.
:::

::: {.section .solution}
##  Решение {#solution}

Паттерн Фабричный метод предлагает создавать объекты не напрямую,
используя оператор `new`, а через вызов особого *фабричного* метода. Не
пугайтесь, объекты всё равно будут создаваться при помощи `new`, но
делать это будет фабричный метод.

<figure class="image">
<img
src="../../images/patterns/diagrams/factory-method/solution16ec2.png?id=fc756d2af296b5b4d482e548214d08ef"
style="aspect-ratio: 2.3;"
srcset="/images/patterns/diagrams/factory-method/solution1.png?id=fc756d2af296b5b4d482e548214d08ef 620w,/images/patterns/diagrams/factory-method/solution1-1.5x.png?id=22d3b6bb74e966d1cb3a4d8f380cefa3 930w,/images/patterns/diagrams/factory-method/solution1-2x.png?id=c482b3cd7a4d8dd73b4c8c12dfcaa03c 1240w"
sizes="(max-width: 720px) 100vw, 620px" loading="lazy" width="620"
alt="Структура классов-создателей" />
<figcaption><p>Подклассы могут изменять класс
создаваемых объектов.</p></figcaption>
</figure>

На первый взгляд, это может показаться бессмысленным: мы просто
переместили вызов конструктора из одного конца программы в другой. Но
теперь вы сможете переопределить фабричный метод в подклассе, чтобы
изменить тип создаваемого продукта.

Чтобы эта система заработала, все возвращаемые объекты должны иметь
общий интерфейс. Подклассы смогут производить объекты различных классов,
следующих одному и тому же интерфейсу.

<figure class="image">
<img
src="../../images/patterns/diagrams/factory-method/solution2-ru7041.png?id=3964a2cf8aedfb5662bb8d992b927713"
style="aspect-ratio: 1.96;"
srcset="/images/patterns/diagrams/factory-method/solution2-ru.png?id=3964a2cf8aedfb5662bb8d992b927713 490w,/images/patterns/diagrams/factory-method/solution2-ru-1.5x.png?id=72496e90ca3d3aca222018e7d094415a 735w,/images/patterns/diagrams/factory-method/solution2-ru-2x.png?id=d0305f546f3cb246be74cc50a3ee761d 980w"
sizes="(max-width: 720px) 100vw, 490px" loading="lazy" width="490"
alt="Структура иерархии продуктов" />
<figcaption><p>Все объекты-продукты должны иметь
общий интерфейс.</p></figcaption>
</figure>

Например, классы `Грузовик` и `Судно` реализуют интерфейс `Транспорт` с
методом `доставить`. Каждый из этих классов реализует метод по-своему:
грузовики везут грузы по земле, а суда --- по морю. Фабричный метод в
классе `ДорожнойЛогистики` вернёт объект-грузовик, а класс
`МорскойЛогистики` --- объект-судно.

<figure class="image">
<img
src="../../images/patterns/diagrams/factory-method/solution3-en8f03.png?id=b6f53911fc0d56f9ef99501fc4aec059"
style="aspect-ratio: 1.83;"
srcset="/images/patterns/diagrams/factory-method/solution3-en.png?id=b6f53911fc0d56f9ef99501fc4aec059 640w,/images/patterns/diagrams/factory-method/solution3-en-1.5x.png?id=f66d5c45f61b619d513a12b7e0a755d5 960w,/images/patterns/diagrams/factory-method/solution3-en-2x.png?id=542c0ba89e91ac11ea79e94bc0229f70 1280w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="640"
alt="Программа после применения фабричного метода" />
<figcaption><p>Пока все продукты реализуют общий интерфейс, их объекты
можно взаимозаменять в клиентском коде.</p></figcaption>
</figure>

Для клиента фабричного метода нет разницы между этими объектами, так как
он будет трактовать их как некий абстрактный `Транспорт`. Для него будет
важно, чтобы объект имел метод `доставить`, а как конкретно он
работает --- не важно.
:::

::::: {.section .structure-container}
##  Структура {#structure}

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/factory-method/structure78c3.png?id=4cba0803f42517cfe8548c9bc7dc4c9b"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.74;"
srcset="/images/patterns/diagrams/factory-method/structure.png?id=4cba0803f42517cfe8548c9bc7dc4c9b 660w,/images/patterns/diagrams/factory-method/structure-1.5x.png?id=ece8300890afc14494770a6b6d370428 990w,/images/patterns/diagrams/factory-method/structure-2x.png?id=9ea3aa8a47f8be22e12e523c15b448fd 1320w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="660"
alt="Структура классов паттерна Фабричный метод" /><img
src="../../images/patterns/diagrams/factory-method/structure-indexed6136.png?id=4c603207859ca1f939b17b60a3a2e9e0"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.74;"
srcset="/images/patterns/diagrams/factory-method/structure-indexed.png?id=4c603207859ca1f939b17b60a3a2e9e0 660w,/images/patterns/diagrams/factory-method/structure-indexed-1.5x.png?id=626b6d7f577e1c265cce244678b2cf7f 990w,/images/patterns/diagrams/factory-method/structure-indexed-2x.png?id=c794e4f2d05013fb176464a1d1a5d7ab 1320w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="660"
alt="Структура классов паттерна Фабричный метод" />
</figure>
:::

1.  **Продукт** определяет общий интерфейс объектов, которые может
    произвести создатель и его подклассы.

2.  **Конкретные продукты** содержат код различных продуктов. Продукты
    будут отличаться реализацией, но интерфейс у них будет общий.

3.  **Создатель** объявляет фабричный метод, который должен возвращать
    новые объекты продуктов. Важно, чтобы тип результата совпадал с
    общим интерфейсом продуктов.

    Зачастую фабричный метод объявляют абстрактным, чтобы заставить все
    подклассы реализовать его по-своему. Но он может возвращать и некий
    стандартный продукт.

    Несмотря на название, важно понимать, что создание продуктов **не
    является** единственной функцией создателя. Обычно он содержит и
    другой полезный код работы с продуктом. Аналогия: большая софтверная
    компания может иметь центр подготовки программистов, но основная
    задача компании --- создавать программные продукты, а не готовить
    программистов.

4.  **Конкретные создатели** по-своему реализуют фабричный метод,
    производя те или иные конкретные продукты.

    Фабричный метод не обязан всё время создавать новые объекты. Его
    можно переписать так, чтобы возвращать существующие объекты из
    какого-то хранилища или кэша.
::::
:::::

::: {.section .pseudocode}
##  Псевдокод {#pseudocode}

В этом примере **Фабричный метод** помогает создавать
кросс-платформенные элементы интерфейса, не привязывая основной код
программы к конкретным классам элементов.

<figure class="image">
<img
src="../../images/patterns/diagrams/factory-method/examplec59f.png?id=67db9a5cb817913444efcb1c067c9835"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/diagrams/factory-method/example.png?id=67db9a5cb817913444efcb1c067c9835 640w,/images/patterns/diagrams/factory-method/example-1.5x.png?id=921d97276e5e2fd8e64609c9cfa021e7 960w,/images/patterns/diagrams/factory-method/example-2x.png?id=a2470830778e318263155000dbdc5870 1280w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="640"
alt="Структура классов примера паттерна Фабричного метода" />
<figcaption><p>Пример кросс-платформенного диалога.</p></figcaption>
</figure>

Фабричный метод объявлен в классе диалогов. Его подклассы относятся к
различным операционным системам. Благодаря фабричному методу, вам не
нужно переписывать логику диалогов под каждую систему. Подклассы могут
наследовать почти весь код из базового диалога, изменяя типы кнопок и
других элементов, из которых базовый код строит окна графического
пользовательского интерфейса.

Базовый класс диалогов работает с кнопками через их общий программный
интерфейс. Поэтому, какую вариацию кнопок ни вернул бы фабричный метод,
диалог останется рабочим. Базовый класс не зависит от конкретных классов
кнопок, оставляя подклассам решение о том, какой тип кнопок создавать.

Такой подход можно применить и для создания других элементов интерфейса.
Хотя каждый новый тип элементов будет приближать вас к [Абстрактной
фабрике](abstract-factory.html).

<figure class="code">
<pre class="code" lang="pseudocode"><code>// Паттерн Фабричный метод применим тогда, когда в программе
// есть иерархия классов продуктов.
interface Button is
    method render()
    method onClick(f)

class WindowsButton implements Button is
    method render(a, b) is
        // Отрисовать кнопку в стиле Windows.
    method onClick(f) is
        // Навесить на кнопку обработчик событий Windows.

class HTMLButton implements Button is
    method render(a, b) is
        // Вернуть HTML-код кнопки.
    method onClick(f) is
        // Навесить на кнопку обработчик события браузера.


// Базовый класс фабрики. Заметьте, что &quot;фабрика&quot; — это всего
// лишь дополнительная роль для класса. Скорее всего, он уже
// имеет какую-то бизнес-логику, в которой требуется создание
// разнообразных продуктов.
class Dialog is
    method render() is
        // Чтобы использовать фабричный метод, вы должны
        // убедиться в том, что эта бизнес-логика не зависит от
        // конкретных классов продуктов. Button — это общий
        // интерфейс кнопок, поэтому все хорошо.
        Button okButton = createButton()
        okButton.onClick(closeDialog)
        okButton.render()

    // Мы выносим весь код создания продуктов в особый метод,
    // который назвают &quot;фабричным&quot;.
    abstract method createButton():Button


// Конкретные фабрики переопределяют фабричный метод и
// возвращают из него собственные продукты.
class WindowsDialog extends Dialog is
    method createButton():Button is
        return new WindowsButton()

class WebDialog extends Dialog is
    method createButton():Button is
        return new HTMLButton()


class Application is
    field dialog: Dialog

    // Приложение создаёт определённую фабрику в зависимости от
    // конфигурации или окружения.
    method initialize() is
        config = readApplicationConfigFile()

        if (config.OS == &quot;Windows&quot;) then
            dialog = new WindowsDialog()
        else if (config.OS == &quot;Web&quot;) then
            dialog = new WebDialog()
        else
            throw new Exception(&quot;Error! Unknown operating system.&quot;)

    // Если весь остальной клиентский код работает с фабриками и
    // продуктами только через общий интерфейс, то для него
    // будет не важно, какая фабрика была создана изначально.
    method main() is
        this.initialize()
        dialog.render()</code></pre>
</figure>
:::

:::::::::: {.section .applicability-container}
##  Применимость {#applicability}

::::::::: applicability
::: applicability-problem
Когда заранее неизвестны типы и зависимости объектов, с которыми должен
работать ваш код.
:::

::: applicability-solution
Фабричный метод отделяет код производства продуктов от остального кода,
который эти продукты использует.

Благодаря этому, код производства можно расширять, не трогая основной.
Так, чтобы добавить поддержку нового продукта, вам нужно создать новый
подкласс и определить в нём фабричный метод, возвращая оттуда экземпляр
нового продукта.
:::

::: applicability-problem
Когда вы хотите дать возможность пользователям расширять части вашего
фреймворка или библиотеки.
:::

::: applicability-solution
Пользователи могут расширять классы вашего фреймворка через
наследование. Но как сделать так, чтобы фреймворк создавал объекты из
этих новых классов, а не из стандартных?

Решением будет дать пользователям возможность расширять не только
желаемые компоненты, но и классы, которые создают эти компоненты. А для
этого создающие классы должны иметь конкретные создающие методы, которые
можно определить.

Например, вы используете готовый UI-фреймворк для своего приложения. Но
вот беда --- требуется иметь круглые кнопки, вместо стандартных
прямоугольных. Вы создаёте класс `RoundButton`. Но как сказать главному
классу фреймворка `UIFramework`, чтобы он теперь создавал круглые
кнопки, вместо стандартных?

Для этого вы создаёте подкласс `UIWithRoundButtons` из базового класса
фреймворка, переопределяете в нём метод создания кнопки (а-ля
`createButton`) и вписываете туда создание своего класса кнопок. Затем
используете `UIWithRoundButtons` вместо стандартного `UIFramework`.
:::

::: applicability-problem
Когда вы хотите экономить системные ресурсы, повторно используя уже
созданные объекты, вместо порождения новых.
:::

::: applicability-solution
Такая проблема обычно возникает при работе с тяжёлыми ресурсоёмкими
объектами, такими, как подключение к базе данных, файловой системе
и т. д.

Представьте, сколько действий вам нужно совершить, чтобы повторно
использовать существующие объекты:

1.  Сначала вам следует создать общее хранилище, чтобы хранить в нём все
    создаваемые объекты.
2.  При запросе нового объекта нужно будет заглянуть в хранилище и
    проверить, есть ли там неиспользуемый объект.
3.  А затем вернуть его клиентскому коду.
4.  Но если свободных объектов нет --- создать новый, не забыв добавить
    его в хранилище.

Весь этот код нужно куда-то поместить, чтобы не засорять клиентский код.

Самым удобным местом был бы конструктор объекта, ведь все эти проверки
нужны только при создании объектов. Но, увы, конструктор всегда создаёт
**новые** объекты, он не может вернуть существующий экземпляр.

Значит, нужен другой метод, который бы отдавал как существующие, так и
новые объекты. Им и станет фабричный метод.
:::
:::::::::
::::::::::

::: {.section .checklist}
##  Шаги реализации {#checklist}

1.  Приведите все создаваемые продукты к общему интерфейсу.

2.  В классе, который производит продукты, создайте пустой фабричный
    метод. В качестве возвращаемого типа укажите общий интерфейс
    продукта.

3.  Затем пройдитесь по коду класса и найдите все участки, создающие
    продукты. Поочерёдно замените эти участки вызовами фабричного
    метода, перенося в него код создания различных продуктов.

    В фабричный метод, возможно, придётся добавить несколько параметров,
    контролирующих, какой из продуктов нужно создать.

    На этом этапе фабричный метод, скорее всего, будет выглядеть
    удручающе. В нём будет жить большой условный оператор, выбирающий
    класс создаваемого продукта. Но не волнуйтесь, мы вот-вот исправим
    это.

4.  Для каждого типа продуктов заведите подкласс и переопределите в нём
    фабричный метод. Переместите туда код создания соответствующего
    продукта из суперкласса.

5.  Если создаваемых продуктов слишком много для существующих подклассов
    создателя, вы можете подумать о введении параметров в фабричный
    метод, которые позволят возвращать различные продукты в пределах
    одного подкласса.

    Например, у вас есть класс `Почта` с подклассами `АвиаПочта` и
    `НаземнаяПочта`, а также классы продуктов `Самолёт`, `Грузовик` и
    `Поезд`. `Авиа` соответствует `Самолётам`, но для `НаземнойПочты`
    есть сразу два продукта. Вы могли бы создать новый подкласс почты
    для поездов, но проблему можно решить и по-другому. Клиентский код
    может передавать в фабричный метод `НаземнойПочты` аргумент,
    контролирующий тип создаваемого продукта.

6.  Если после всех перемещений фабричный метод стал пустым, можете
    сделать его абстрактным. Если в нём что-то осталось --- не беда, это
    будет его реализацией по умолчанию.
:::

:::::: {.section .pros-cons}
##  Преимущества и недостатки {#pros-cons}

::::: row
::: col-sm-6
-    Избавляет класс от привязки к конкретным классам продуктов.
-    Выделяет код производства продуктов в одно место, упрощая поддержку
    кода.
-    Упрощает добавление новых продуктов в программу.
-    Реализует *принцип открытости/закрытости*.
:::

::: col-sm-6
-    Может привести к созданию больших [параллельных иерархий
    классов](../smells/parallel-inheritance-hierarchies.html), так как
    для каждого класса продукта надо создать свой подкласс создателя.
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

-   Классы [Абстрактной фабрики](abstract-factory.html) чаще всего
    реализуются с помощью [Фабричного метода](factory-method.html), хотя
    они могут быть построены и на основе [Прототипа](prototype.html).

-   [Фабричный метод](factory-method.html) можно использовать вместе с
    [Итератором](iterator.html), чтобы подклассы коллекций могли
    создавать подходящие им итераторы.

-   [Прототип](prototype.html) не опирается на наследование, но ему
    нужна сложная операция инициализации. [Фабричный
    метод](factory-method.html), наоборот, построен на наследовании, но
    не требует сложной инициализации.

-   [Фабричный метод](factory-method.html) можно рассматривать как
    частный случай [Шаблонного метода](template-method.html). Кроме
    того, *Фабричный метод* нередко бывает частью большого класса с
    *Шаблонными методами*.
:::

::: {.section .implementations}
##  Примеры реализации паттерна {#implementations}

[![Фабричный метод на
C#](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](factory-method/csharp/example.html "Фабричный метод на C#"){.prog-lang-link}
[![Фабричный метод на
C++](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](factory-method/cpp/example.html "Фабричный метод на C++"){.prog-lang-link}
[![Фабричный метод на
Go](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](factory-method/go/example.html "Фабричный метод на Go"){.prog-lang-link}
[![Фабричный метод на
Java](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](factory-method/java/example.html "Фабричный метод на Java"){.prog-lang-link}
[![Фабричный метод на
PHP](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](factory-method/php/example.html "Фабричный метод на PHP"){.prog-lang-link}
[![Фабричный метод на
Python](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](factory-method/python/example.html "Фабричный метод на Python"){.prog-lang-link}
[![Фабричный метод на
Ruby](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](factory-method/ruby/example.html "Фабричный метод на Ruby"){.prog-lang-link}
[![Фабричный метод на
Rust](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](factory-method/rust/example.html "Фабричный метод на Rust"){.prog-lang-link}
[![Фабричный метод на
Swift](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](factory-method/swift/example.html "Фабричный метод на Swift"){.prog-lang-link}
[![Фабричный метод на
TypeScript](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](factory-method/typescript/example.html "Фабричный метод на TypeScript"){.prog-lang-link}
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

[Абстрактная фабрика []{.fa
.fa-arrow-right}](abstract-factory.html){.btn .btn-primary rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Порождающие
паттерны](creational-patterns.html){.btn .btn-default rel="prev"}
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
