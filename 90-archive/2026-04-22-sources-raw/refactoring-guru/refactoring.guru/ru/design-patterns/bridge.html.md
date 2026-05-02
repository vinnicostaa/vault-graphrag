:::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [Паттерны
проектирования](../design-patterns.html){.type} / [Структурные
паттерны](structural-patterns.html){.type}
:::

# Мост {#мост .title}

::: aka
Также известен как: [Bridge]{style="display:inline-block"}
:::

::: {.section .intent}
##  Суть паттерна {#intent}

**Мост** --- это структурный паттерн проектирования, который разделяет
один или несколько классов на две отдельные иерархии --- абстракцию и
реализацию, позволяя изменять их независимо друг от друга.

<figure class="image">
<img
src="../../images/patterns/content/bridge/bridge3e59.png?id=bd543d4fb32e11647767301581a5ad54"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/bridge/bridge.png?id=bd543d4fb32e11647767301581a5ad54 640w,/images/patterns/content/bridge/bridge-1.5x.png?id=8ef07b78d61cc3830b7e2b783c76c775 960w,/images/patterns/content/bridge/bridge-2x.png?id=1e905ae5742e5cd10a7eb0e3175ef00d 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Паттерн Мост" />
</figure>
:::

::: {.section .problem}
##  Проблема {#problem}

*Абстракция?* *Реализация?!* Звучит пугающе! Чтобы понять, о чём идёт
речь, давайте разберём очень простой пример.

У вас есть класс геометрических `Фигур`, который имеет подклассы `Круг`
и `Квадрат`. Вы хотите расширить иерархию фигур по цвету, то есть иметь
`Красные` и `Синие` фигуры. Но чтобы всё это объединить, вам придётся
создать 4 комбинации подклассов, вроде `СиниеКруги` и `КрасныеКвадраты`.

<figure class="image">
<img
src="../../images/patterns/diagrams/bridge/problem-ru787f.png?id=77abf3e03813e7d31ff123bbc05645dc"
style="aspect-ratio: 1.41;"
srcset="/images/patterns/diagrams/bridge/problem-ru.png?id=77abf3e03813e7d31ff123bbc05645dc 480w,/images/patterns/diagrams/bridge/problem-ru-1.5x.png?id=382255585cf41f0df9ccecd83583985a 720w,/images/patterns/diagrams/bridge/problem-ru-2x.png?id=1f3e32742cd1fe74684e4e050250d73a 960w"
sizes="(max-width: 720px) 100vw, 480px" loading="lazy" width="480"
alt="Проблема паттерна Мост" />
<figcaption><p>Количество подклассов растёт в
геометрической прогрессии.</p></figcaption>
</figure>

При добавлении новых видов фигур и цветов количество комбинаций будет
расти в геометрической прогрессии. Например, чтобы ввести в программу
фигуры треугольников, придётся создать сразу два новых подкласса
треугольников под каждый цвет. После этого новый цвет потребует создания
уже трёх классов для всех видов фигур. Чем дальше, тем хуже.
:::

::: {.section .solution}
##  Решение {#solution}

Корень проблемы заключается в том, что мы пытаемся расширить классы
фигур сразу в двух независимых плоскостях --- по виду и по цвету. Именно
это приводит к разрастанию дерева классов.

Паттерн Мост предлагает заменить наследование агрегацией или
композицией. Для этого нужно выделить одну из таких «плоскостей» в
отдельную иерархию и ссылаться на объект этой иерархии, вместо хранения
его состояния и поведения внутри одного класса.

<figure class="image">
<img
src="../../images/patterns/diagrams/bridge/solution-ru858e.png?id=885cafb59a249da6870829dc87f72cd8"
style="aspect-ratio: 2.3;"
srcset="/images/patterns/diagrams/bridge/solution-ru.png?id=885cafb59a249da6870829dc87f72cd8 460w,/images/patterns/diagrams/bridge/solution-ru-1.5x.png?id=65ef50e383ebfacf90df938793a62795 690w,/images/patterns/diagrams/bridge/solution-ru-2x.png?id=ba2c4b2d990f8b8844eaf343a7c5e3e8 920w"
sizes="(max-width: 720px) 100vw, 460px" loading="lazy" width="460"
alt="Решение паттерна Мост" />
<figcaption><p>Размножение подклассов можно остановить, разбив классы на
несколько иерархий.</p></figcaption>
</figure>

Таким образом, мы можем сделать `Цвет` отдельным классом с подклассами
`Красный` и `Синий`. Класс `Фигур` получит ссылку на объект `Цвета` и
сможет делегировать ему работу, если потребуется. Такая связь и станет
мостом между `Фигурами` и `Цветом`. При добавлении новых классов цветов
не потребуется трогать классы фигур и наоборот.

#### Абстракция и Реализация

Эти термины были введены в книге GoF []{.pop-i}[Gang of Four / «Банда
четырёх». Авторы книги *Design Patterns: Elements of Reusable
Object-Oriented Software*
[https://refactoring.guru/ru/gof-book](https://www.ozon.ru/context/detail/id/2457392/).]{.pop-content}
при описании Моста. На мой взгляд, они выглядят слишком академичными,
делая описание паттерна сложнее, чем он есть на самом деле. Помня о
примере с фигурами и цветами, давайте все же разберёмся, что имели в
виду авторы паттерна.

Итак, *абстракция* (или *интерфейс*) --- это образный слой управления
чем-либо. Он не делает работу самостоятельно, а делегирует её слою
*реализации* (иногда называемому *платформой*).

> Только не путайте эти термины с *интерфейсами* или *абстрактными
> классами* из вашего языка программирования, это не одно и то же.

Если говорить о реальных программах, то абстракцией может выступать
графический интерфейс программы (GUI), а реализацией --- низкоуровневый
код операционной системы (API), к которому графический интерфейс
обращается по реакции на действия пользователя.

Вы можете развивать программу в двух разных направлениях:

-   иметь несколько видов GUI (например, для простых пользователей и
    администраторов);
-   поддерживать много видов API (например, работать под Windows, Linux
    и macOS).

Такая программа может выглядеть как один большой клубок кода, в котором
намешаны условные операторы слоёв GUI и API.

<figure class="image">
<img
src="../../images/patterns/content/bridge/bridge-3-ru8a59.png?id=1ab86269116f3f1fbad1ebd8913f0ead"
style="aspect-ratio: 2;"
srcset="/images/patterns/content/bridge/bridge-3-ru.png?id=1ab86269116f3f1fbad1ebd8913f0ead 600w,/images/patterns/content/bridge/bridge-3-ru-1.5x.png?id=6872e4330a703620e4e111c13a64a977 900w,/images/patterns/content/bridge/bridge-3-ru-2x.png?id=9f25a2cb4cde5f58664bc0fb22d8bdae 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Защита от изменений" />
<figcaption><p>Когда изменения «осаждают» проект, вам легче отбиваться,
если разделить монолитный код на части.</p></figcaption>
</figure>

Вы можете попытаться структурировать этот хаос, создав для каждой
вариации интерфейса-платформы свои подклассы. Но такой подход приведёт к
росту классов комбинаций, и с каждой новой платформой их будет всё
больше.

Мы можем решить эту проблему, применив Мост. Паттерн предлагает
распутать этот код, разделив его на две части:

-   Абстракцию: слой графического интерфейса приложения.
-   Реализацию: слой взаимодействия с операционной системой.

<figure class="image">
<img
src="../../images/patterns/content/bridge/bridge-2-ruf062.png?id=90985b90567f2d63590e6c5aa4d289a5"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/bridge/bridge-2-ru.png?id=90985b90567f2d63590e6c5aa4d289a5 640w,/images/patterns/content/bridge/bridge-2-ru-1.5x.png?id=fb212972909fea2c07bfdd1264713b13 960w,/images/patterns/content/bridge/bridge-2-ru-2x.png?id=5595acd103743a61ca92bc2d039cece1 1280w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="640"
alt="Вариант кросс-платформенной архитектуры" />
<figcaption><p>Один из вариантов
кросс-платформенной архитектуры.</p></figcaption>
</figure>

Абстракция будет делегировать работу одному из объектов реализаций.
Причём, реализации можно будет взаимозаменять, но только при условии,
что все они будут следовать общему интерфейсу.

Таким образом, вы сможете изменять графический интерфейс приложения, не
трогая низкоуровневый код работы с операционной системой. И наоборот, вы
сможете добавлять поддержку новых операционных систем, создавая
подклассы реализации, без необходимости менять классы графического
интерфейса.
:::

::::: {.section .structure-container}
##  Структура {#structure}

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/bridge/structure-ruda1f.png?id=2ba4707200888c2b64a6a71d4ac1a6cc"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.47;"
srcset="/images/patterns/diagrams/bridge/structure-ru.png?id=2ba4707200888c2b64a6a71d4ac1a6cc 560w,/images/patterns/diagrams/bridge/structure-ru-1.5x.png?id=bd3392d8841d418814270a7c540251c7 840w,/images/patterns/diagrams/bridge/structure-ru-2x.png?id=0df864aa4e3daf3a9122996c29a946da 1120w"
sizes="(max-width: 720px) 100vw, 560px" loading="lazy" width="560"
alt="Структура классов паттерна Мост" /><img
src="../../images/patterns/diagrams/bridge/structure-ru-indexed3efb.png?id=d1001bcbc7cccf327c6a0fcfa0a7c7c9"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.44;"
srcset="/images/patterns/diagrams/bridge/structure-ru-indexed.png?id=d1001bcbc7cccf327c6a0fcfa0a7c7c9 560w,/images/patterns/diagrams/bridge/structure-ru-indexed-1.5x.png?id=1904c8b1896609e1e1ca35e4d1ea85a4 840w,/images/patterns/diagrams/bridge/structure-ru-indexed-2x.png?id=32de8caad2c44665e5eaf3e800ce0797 1120w"
sizes="(max-width: 720px) 100vw, 560px" loading="lazy" width="560"
alt="Структура классов паттерна Мост" />
</figure>
:::

1.  **Абстракция** содержит управляющую логику. Код абстракции
    делегирует реальную работу связанному объекту реализации.

2.  **Реализация** задаёт общий интерфейс для всех реализаций. Все
    методы, которые здесь описаны, будут доступны из класса абстракции и
    его подклассов.

    Интерфейсы абстракции и реализации могут как совпадать, так и быть
    совершенно разными. Но обычно в реализации живут базовые операции,
    на которых строятся сложные операции абстракции.

3.  **Конкретные реализации** содержат платформо-зависимый код.

4.  **Расширенные абстракции** содержат различные вариации управляющей
    логики. Как и родитель, работает с реализациями только через общий
    интерфейс реализации.

5.  **Клиент** работает только с объектами абстракции. Не считая
    начального связывания абстракции с одной из реализаций, клиентский
    код не имеет прямого доступа к объектам реализации.
::::
:::::

::: {.section .pseudocode}
##  Псевдокод {#pseudocode}

В этом примере **Мост** разделяет монолитный код приборов и пультов на
две части: приборы (выступают реализацией) и пульты управления ими
(выступают абстракцией).

<figure class="image">
<img
src="../../images/patterns/diagrams/bridge/example-rud9dd.png?id=400eb33d8efffdd05b25450d8e9aef30"
style="aspect-ratio: 1.33;"
srcset="/images/patterns/diagrams/bridge/example-ru.png?id=400eb33d8efffdd05b25450d8e9aef30 640w,/images/patterns/diagrams/bridge/example-ru-1.5x.png?id=70bb734a002336c670dd4ed49a9677e3 960w,/images/patterns/diagrams/bridge/example-ru-2x.png?id=8c9afb5ed67d7ba1db89ab734affd07d 1280w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="640"
alt="Структура классов примера паттерна Мост" />
<figcaption><p>Пример разделения двух иерархий классов — приборов и
пультов управления.</p></figcaption>
</figure>

Класс пульта имеет ссылку на объект прибора, которым он управляет.
Пульты работают с приборами через общий интерфейс. Это даёт возможность
связать пульты с различными приборами.

Сами пульты можно развивать независимо от приборов. Для этого достаточно
создать новый подкласс абстракции. Вы можете создать как простой пульт с
двумя кнопками, так и более сложный пульт с тач-интерфейсом.

Клиентскому коду остаётся выбрать версию абстракции и реализации, с
которым он хочет работать, и связать их между собой.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// Класс пультов имеет ссылку на устройство, которым управляет.
// Методы этого класса делегируют работу методам связанного
// устройства.
class Remote is
    protected field device: Device
    constructor Remote(device: Device) is
        this.device = device
    method togglePower() is
        if (device.isEnabled()) then
            device.disable()
        else
            device.enable()
    method volumeDown() is
        device.setVolume(device.getVolume() - 10)
    method volumeUp() is
        device.setVolume(device.getVolume() + 10)
    method channelDown() is
        device.setChannel(device.getChannel() - 1)
    method channelUp() is
        device.setChannel(device.getChannel() + 1)


// Вы можете расширять класс пультов, не трогая код устройств.
class AdvancedRemote extends Remote is
    method mute() is
        device.setVolume(0)


// Все устройства имеют общий интерфейс. Поэтому с ними может
// работать любой пульт.
interface Device is
    method isEnabled()
    method enable()
    method disable()
    method getVolume()
    method setVolume(percent)
    method getChannel()
    method setChannel(channel)


// Но каждое устройство имеет особую реализацию.
class Tv implements Device is
    // ...

class Radio implements Device is
    // ...


// Где-то в клиентском коде.
tv = new Tv()
remote = new Remote(tv)
remote.togglePower()

radio = new Radio()
remote = new AdvancedRemote(radio)</code></pre>
</figure>
:::

:::::::::: {.section .applicability-container}
##  Применимость {#applicability}

::::::::: applicability
::: applicability-problem
Когда вы хотите разделить монолитный класс, который содержит несколько
различных реализаций какой-то функциональности (например, если класс
может работать с разными системами баз данных).
:::

::: applicability-solution
Чем больше класс, тем тяжелее разобраться в его коде, и тем больше это
затягивает разработку. Кроме того, изменения, вносимые в одну из
реализаций, приводят к редактированию всего класса, что может привести к
внесению случайных ошибок в код.

Мост позволяет разделить монолитный класс на несколько отдельных
иерархий. После этого вы можете менять их код независимо друг от друга.
Это упрощает работу над кодом и уменьшает вероятность внесения ошибок.
:::

::: applicability-problem
Когда класс нужно расширять в двух независимых плоскостях.
:::

::: applicability-solution
Мост предлагает выделить одну из таких плоскостей в отдельную иерархию
классов, храня ссылку на один из её объектов в первоначальном классе.
:::

::: applicability-problem
Когда вы хотите, чтобы реализацию можно было бы изменять во время
выполнения программы.
:::

::: applicability-solution
Мост позволяет заменять реализацию даже во время выполнения программы,
так как конкретная реализация не «вшита» в класс абстракции.

*Кстати, из-за этого пункта Мост часто путают со
[Стратегией](strategy.html). Обратите внимание, что у Моста этот пункт
стоит на последнем месте по значимости, поскольку его главная задача ---
структурная.*
:::
:::::::::
::::::::::

::: {.section .checklist}
##  Шаги реализации {#checklist}

1.  Определите, существует ли в ваших классах два непересекающихся
    измерения. Это может быть функциональность/платформа,
    предметная-область/инфраструктура, фронт-энд/бэк-энд или
    интерфейс/реализация.

2.  Продумайте, какие операции будут нужны клиентам, и опишите их в
    базовом классе *абстракции*.

3.  Определите поведения, доступные на всех платформах, и выделите из
    них ту часть, которая нужна абстракции. На основании этого опишите
    общий интерфейс *реализации*.

4.  Для каждой платформы создайте свой класс конкретной реализации. Все
    они должны следовать общему интерфейсу, который мы выделили перед
    этим.

5.  Добавьте в класс абстракции ссылку на объект реализации. Реализуйте
    методы абстракции, делегируя основную работу связанному объекту
    реализации.

6.  Если у вас есть несколько вариаций абстракции, создайте для каждой
    из них свой подкласс.

7.  Клиент должен подать объект реализации в конструктор абстракции,
    чтобы связать их воедино. После этого он может свободно использовать
    объект абстракции, забыв о реализации.
:::

:::::: {.section .pros-cons}
##  Преимущества и недостатки {#pros-cons}

::::: row
::: col-sm-6
-    Позволяет строить платформо-независимые программы.
-    Скрывает лишние или опасные детали реализации от клиентского кода.
-    Реализует *принцип открытости/закрытости*.
:::

::: col-sm-6
-    Усложняет код программы из-за введения дополнительных классов.
:::
:::::
::::::

::: {.section .relations}
##  Отношения с другими паттернами {#relations}

-   [Мост](bridge.html) проектируют загодя, чтобы развивать большие
    части приложения отдельно друг от друга. [Адаптер](adapter.html)
    применяется постфактум, чтобы заставить несовместимые классы
    работать вместе.

-   [Мост](bridge.html), [Стратегия](strategy.html) и
    [Состояние](state.html) (а также слегка и [Адаптер](adapter.html))
    имеют схожие структуры классов --- все они построены на принципе
    «композиции», то есть делегирования работы другим объектам. Тем не
    менее, они отличаются тем, что решают разные проблемы. Помните, что
    паттерны --- это не только рецепт построения кода определённым
    образом, но и описание проблем, которые привели к данному решению.

-   [Абстрактная фабрика](abstract-factory.html) может работать
    совместно с [Мостом](bridge.html). Это особенно полезно, если у вас
    есть абстракции, которые могут работать только с некоторыми из
    реализаций. В этом случае фабрика будет определять типы создаваемых
    абстракций и реализаций.

-   Паттерн [Строитель](builder.html) может быть построен в виде
    [Моста](bridge.html): *директор* будет играть роль абстракции, а
    *строители* --- реализации.
:::

::: {.section .implementations}
##  Примеры реализации паттерна {#implementations}

[![Мост на
C#](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](bridge/csharp/example.html "Мост на C#"){.prog-lang-link}
[![Мост на
C++](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](bridge/cpp/example.html "Мост на C++"){.prog-lang-link}
[![Мост на
Go](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](bridge/go/example.html "Мост на Go"){.prog-lang-link}
[![Мост на
Java](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](bridge/java/example.html "Мост на Java"){.prog-lang-link}
[![Мост на
PHP](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](bridge/php/example.html "Мост на PHP"){.prog-lang-link}
[![Мост на
Python](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](bridge/python/example.html "Мост на Python"){.prog-lang-link}
[![Мост на
Ruby](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](bridge/ruby/example.html "Мост на Ruby"){.prog-lang-link}
[![Мост на
Rust](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](bridge/rust/example.html "Мост на Rust"){.prog-lang-link}
[![Мост на
Swift](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](bridge/swift/example.html "Мост на Swift"){.prog-lang-link}
[![Мост на
TypeScript](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](bridge/typescript/example.html "Мост на TypeScript"){.prog-lang-link}
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

[Компоновщик []{.fa .fa-arrow-right}](composite.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Адаптер](adapter.html){.btn .btn-default
rel="prev"}
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
