:::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [Паттерны
проектирования](../design-patterns.html){.type} / [Структурные
паттерны](structural-patterns.html){.type}
:::

# Легковес {#легковес .title}

::: aka
Также известен как:
[Приспособленец, ]{style="display:inline-block"}[Кэш, ]{style="display:inline-block"}[Flyweight]{style="display:inline-block"}
:::

::: {.section .intent}
##  Суть паттерна {#intent}

**Легковес** --- это структурный паттерн проектирования, который
позволяет вместить бóльшее количество объектов в отведённую оперативную
память. Легковес экономит память, разделяя общее состояние объектов
между собой, вместо хранения одинаковых данных в каждом объекте.

<figure class="image">
<img
src="../../images/patterns/content/flyweight/flyweight7be2.png?id=e34fbacb847dd609b5e68aaf252c4db0"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/flyweight/flyweight.png?id=e34fbacb847dd609b5e68aaf252c4db0 640w,/images/patterns/content/flyweight/flyweight-1.5x.png?id=b32df2297473b8a7577e1900e57325ac 960w,/images/patterns/content/flyweight/flyweight-2x.png?id=6a8f17d9550c75c3d648a605c4d31b45 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Паттерн Легковес" />
</figure>
:::

::: {.section .problem}
##  Проблема {#problem}

На досуге вы решили написать небольшую игру, в которой игроки
перемещаются по карте и стреляют друг в друга. Фишкой игры должна была
стать реалистичная система частиц. Пули, снаряды, осколки от взрывов ---
всё это должно красиво летать и радовать взгляд.

Игра отлично работала на вашем мощном компьютере. Однако ваш друг
сообщил, что игра начинает тормозить и вылетает через несколько минут
после запуска. Покопавшись в логах, вы обнаружили, что игра вылетает
из-за недостатка оперативной памяти. У вашего друга компьютер
значительно менее «прокачанный», поэтому проблема у него и проявляется
так быстро.

И действительно, каждая частица представлена собственным объектом,
имеющим множество данных. В определённый момент, когда побоище на экране
достигает кульминации, новые объекты частиц уже не вмещаются в
оперативную память компьютера, и программа вылетает.

<figure class="image">
<img
src="../../images/patterns/diagrams/flyweight/problem-ruf029.png?id=9b16755ee3803b2728fa6234b057ecaa"
style="aspect-ratio: 2.54;"
srcset="/images/patterns/diagrams/flyweight/problem-ru.png?id=9b16755ee3803b2728fa6234b057ecaa 660w,/images/patterns/diagrams/flyweight/problem-ru-1.5x.png?id=4d058e030a700e72cab391eba6c839bf 990w,/images/patterns/diagrams/flyweight/problem-ru-2x.png?id=2394105b6f4b0b461623d2fa7c09b6ff 1320w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="660"
alt="Проблема паттерна Легковес" />
</figure>
:::

::: {.section .solution}
##  Решение {#solution}

Если внимательно посмотреть на класс частиц, то можно заметить, что цвет
и спрайт занимают больше всего памяти. Более того, они хранятся в каждом
объекте, хотя фактически их значения одинаковы для большинства частиц.

<figure class="image">
<img
src="../../images/patterns/diagrams/flyweight/solution1-ru764f.png?id=4b5605699fc8c531720110438fac9c9f"
style="aspect-ratio: 2.06;"
srcset="/images/patterns/diagrams/flyweight/solution1-ru.png?id=4b5605699fc8c531720110438fac9c9f 640w,/images/patterns/diagrams/flyweight/solution1-ru-1.5x.png?id=8b65eacc4e78b2b8052ecba5db462f4f 960w,/images/patterns/diagrams/flyweight/solution1-ru-2x.png?id=79f38dbfec2259507c5e06d0354b4af3 1280w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="640"
alt="Решение паттерна Легковес" />
</figure>

Остальное состояние объектов --- координаты, вектор движения и
скорость --- отличаются для всех частиц. Таким образом, эти поля можно
рассматривать как контекст, в котором частица используется. А цвет и
спрайт --- это данные, не изменяющиеся во времени.

Неизменяемые данные объекта принято называть «внутренним состоянием».
Все остальные данные --- это «внешнее состояние».

Паттерн Легковес предлагает не хранить в классе внешнее состояние, а
передавать его в те или иные методы через параметры. Таким образом, одни
и те же объекты можно будет повторно использовать в различных
контекстах. Но главное --- понадобится гораздо меньше объектов, ведь
теперь они будут отличаться только внутренним состоянием, а оно имеет не
так много вариаций.

<figure class="image">
<img
src="../../images/patterns/diagrams/flyweight/solution3-rub330.png?id=8264a2a42f970643a4a1a62154f0dc30"
style="aspect-ratio: 0.76;"
srcset="/images/patterns/diagrams/flyweight/solution3-ru.png?id=8264a2a42f970643a4a1a62154f0dc30 424w,/images/patterns/diagrams/flyweight/solution3-ru-1.5x.png?id=bdf00b2fe6f8c0d652d2484a0abb1b2f 636w,/images/patterns/diagrams/flyweight/solution3-ru-2x.png?id=9bfaf275f5fcf3f74fbeb036e4c64c40 848w"
sizes="(max-width: 720px) 100vw, 424px" loading="lazy" width="424"
alt="Решение паттерна Легковес" />
</figure>

В нашем примере с частицами достаточно будет оставить всего три объекта
с отличающимися спрайтами и цветом --- для пуль, снарядов и осколков.
Несложно догадаться, что такие облегчённые объекты называют
*легковéсами* []{.pop-i}[Название пришло из бокса и означает весовую
категорию до 50 кг.]{.pop-content}.

#### Хранилище внешнего состояния

Но куда переедет внешнее состояние? Ведь кто-то должен его хранить. Чаще
всего, его перемещают в контейнер, который управлял объектами до
применения паттерна.

В нашем случае это был главный объект игры. Вы могли бы добавить в его
класс поля-массивы для хранения координат, векторов и скоростей частиц.
Кроме этого, понадобится ещё один массив для хранения ссылок на
объекты-легковесы, соответствующие той или иной частице.

<figure class="image">
<img
src="../../images/patterns/diagrams/flyweight/solution2-rue36e.png?id=41d98f4b85df2b887641ec3e9f495471"
style="aspect-ratio: 1.94;"
srcset="/images/patterns/diagrams/flyweight/solution2-ru.png?id=41d98f4b85df2b887641ec3e9f495471 640w,/images/patterns/diagrams/flyweight/solution2-ru-1.5x.png?id=036afdc7ce43c60bad7617182bda3cf0 960w,/images/patterns/diagrams/flyweight/solution2-ru-2x.png?id=7316d6a2f29997d4a25d23b56991607c 1280w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="640"
alt="Решение паттерна Легковес" />
</figure>

Но более элегантным решением было бы создать дополнительный
класс-контекст, который бы связывал внешнее состояние с тем или иным
легковесом. Это позволит обойтись только одним полем-массивом в классе
контейнера.

«Но погодите-ка, нам потребуется столько же этих объектов, сколько было
в самом начале!», --- скажете вы и будете правы! Но дело в том, что
объекты-контексты занимают намного меньше места, чем первоначальные.
Ведь самые тяжёлые поля остались в легковесах (простите за каламбур), и
сейчас мы будем ссылаться на эти объекты из контекстов, вместо того,
чтобы повторно хранить дублирующееся состояние.

#### Неизменяемость Легковесов

Так как объекты легковесов будут использованы в разных контекстах, вы
должны быть уверены в том, что их состояние невозможно изменить после
создания. Всё внутреннее состояние легковес должен получать через
параметры конструктора. Он не должен иметь сеттеров и публичных полей.

#### Фабрика Легковесов

Для удобства работы с легковесами и контекстами можно создать фабричный
метод, принимающий в параметрах всё внутреннее (а иногда и внешнее)
состояние желаемого объекта.

Главная польза от этого метода в том, чтобы искать уже созданные
легковесы с таким же внутренним состоянием, что и требуемое. Если
легковес находится, его можно повторно использовать. Если нет --- просто
создаём новый. Обычно этот метод добавляют в контейнер легковесов либо
создают отдельный класс-фабрику. Его даже можно сделать статическим и
поместить в класс легковесов.
:::

::::: {.section .structure-container}
##  Структура {#structure}

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/flyweight/structure7b00.png?id=c1e7e1748f957a4792822f902bc1d420"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.64;"
srcset="/images/patterns/diagrams/flyweight/structure.png?id=c1e7e1748f957a4792822f902bc1d420 640w,/images/patterns/diagrams/flyweight/structure-1.5x.png?id=34cf08f6c2b09dcd1c521c7512cc52b6 960w,/images/patterns/diagrams/flyweight/structure-2x.png?id=a7c8347421bde16435fc90a706f5dd34 1280w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="640"
alt="Паттерн Легковес (Приспособленец)" /><img
src="../../images/patterns/diagrams/flyweight/structure-indexed086a.png?id=aa490792baa26b04464dacbc1f4a19cd"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.54;"
srcset="/images/patterns/diagrams/flyweight/structure-indexed.png?id=aa490792baa26b04464dacbc1f4a19cd 630w,/images/patterns/diagrams/flyweight/structure-indexed-1.5x.png?id=b22a5eab6aacbbd03c66c34802b240ca 945w,/images/patterns/diagrams/flyweight/structure-indexed-2x.png?id=205e2f7d08b4ac0695f445a9db8989c4 1260w"
sizes="(max-width: 720px) 100vw, 630px" loading="lazy" width="630"
alt="Паттерн Легковес (Приспособленец)" />
</figure>
:::

1.  Вы всегда должны помнить о том, что Легковес применяется в
    программе, имеющей громадное количество одинаковых объектов. Этих
    объектов должно быть так много, чтобы они не помещались в доступную
    оперативную память без ухищрений. Паттерн разделяет данные этих
    объектов на две части --- легковесы и контексты.

2.  **Легковес** содержит состояние, которое повторялось во множестве
    первоначальных объектов. Один и тот же легковес можно использовать в
    связке со множеством контекстов. Состояние, которое хранится здесь,
    называется *внутренним*, а то, которое он получает извне ---
    *внешним*.

3.  **Контекст** содержит «внешнюю» часть состояния, уникальную для
    каждого объекта. Контекст связан с одним из объектов-легковесов,
    хранящих оставшееся состояние.

4.  Поведение оригинального объекта чаще всего оставляют в Легковесе,
    передавая значения контекста через параметры методов. Тем не менее,
    поведение можно поместить и в контекст, используя легковес как
    объект данных.

5.  **Клиент** вычисляет или хранит контекст, то есть внешнее состояние
    легковесов. Для клиента легковесы выглядят как шаблонные объекты,
    которые можно настроить во время использования, передав контекст
    через параметры.

6.  **Фабрика легковесов** управляет созданием и повторным
    использованием легковесов. Фабрика получает запросы, в которых
    указано желаемое состояние легковеса. Если легковес с таким
    состоянием уже создан, фабрика сразу его возвращает, а если нет ---
    создаёт новый объект.
::::
:::::

::: {.section .pseudocode}
##  Псевдокод {#pseudocode}

В этом примере **Легковес** помогает сэкономить оперативную память при
отрисовке на экране миллионов объектов-деревьев.

<figure class="image">
<img
src="../../images/patterns/diagrams/flyweight/example5053.png?id=0818d078c1a79f373e96397f37b7ee06"
style="aspect-ratio: 1.54;"
srcset="/images/patterns/diagrams/flyweight/example.png?id=0818d078c1a79f373e96397f37b7ee06 540w,/images/patterns/diagrams/flyweight/example-1.5x.png?id=449fa5906e277c134870c07e33dd4b62 810w,/images/patterns/diagrams/flyweight/example-2x.png?id=9423640fe3688a64201389b6e7aa1f48 1080w"
sizes="(max-width: 720px) 100vw, 540px" loading="lazy" width="540"
alt="Структура классов примера паттерна Легковес" />
</figure>

Легковес выделяет повторяющуюся часть состояния из основного класса
`Tree` и помещает его в дополнительный класс `TreeType`.

Теперь, вместо хранения повторяющихся данных во всех объектах, отдельные
деревья будут ссылаться на несколько общих объектов, хранящих эти
данные. Клиент работает с деревьями через фабрику деревьев, которая
скрывает от него сложность кеширования общих данных деревьев.

Таким образом, программа будет использовать намного меньше оперативной
памяти, что позволит отрисовать больше деревьев на экране на том же
железе.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// Этот класс-легковес содержит часть полей, которые описывают
// деревья. Эти поля не уникальны для каждого дерева, в отличие,
// например, от координат: несколько деревьев могут иметь ту же
// текстуру.
//
// Поэтому мы переносим повторяющиеся данные в один-единственный
// объект и ссылаемся на него из множества отдельных деревьев.
class TreeType is
    field name
    field color
    field texture
    constructor TreeType(name, color, texture) { ... }
    method draw(canvas, x, y) is
        // 1. Создать картинку данного типа, цвета и текстуры.
        // 2. Нарисовать картинку на холсте в позиции X, Y.

// Фабрика легковесов решает, когда нужно создать новый
// легковес, а когда можно обойтись существующим.
class TreeFactory is
    static field treeTypes: collection of tree types
    static method getTreeType(name, color, texture) is
        type = treeTypes.find(name, color, texture)
        if (type == null)
            type = new TreeType(name, color, texture)
            treeTypes.add(type)
        return type

// Контекстный объект, из которого мы выделили легковес
// TreeType. В программе могут быть тысячи объектов Tree, так
// как накладные расходы на их хранение совсем небольшие — в
// памяти нужно держать всего три целых числа (две координаты и
// ссылка).
class Tree is
    field x,y
    field type: TreeType
    constructor Tree(x, y, type) { ... }
    method draw(canvas) is
        type.draw(canvas, this.x, this.y)

// Классы Tree и Forest являются клиентами Легковеса. При
// желании их можно слить в один класс, если вам не нужно
// расширять класс деревьев далее.
class Forest is
    field trees: collection of Trees

    method plantTree(x, y, name, color, texture) is
        type = TreeFactory.getTreeType(name, color, texture)
        tree = new Tree(x, y, type)
        trees.add(tree)

    method draw(canvas) is
        foreach (tree in trees) do
            tree.draw(canvas)</code></pre>
</figure>
:::

:::::: {.section .applicability-container}
##  Применимость {#applicability}

::::: applicability
::: applicability-problem
Когда не хватает оперативной памяти для поддержки всех нужных объектов.
:::

::: applicability-solution
Эффективность паттерна **Легковес** во многом зависит от того, как и где
он используется. Применяйте этот паттерн, когда выполнены все
перечисленные условия:

-   в приложении используется большое число объектов;
-   из-за этого высоки расходы оперативной памяти;
-   большую часть состояния объектов можно вынести за пределы их
    классов;
-   большие группы объектов можно заменить относительно небольшим
    количеством разделяемых объектов, поскольку внешнее состояние
    вынесено.
:::
:::::
::::::

::: {.section .checklist}
##  Шаги реализации {#checklist}

1.  Разделите поля класса, который станет легковесом, на две части:

    -   внутреннее состояние: значения этих полей одинаковы для большого
        числа объектов;
    -   внешнее состояние (контекст): значения полей уникальны для
        каждого объекта.

2.  Оставьте поля внутреннего состояния в классе, но убедитесь, что их
    значения неизменяемы. Эти поля должны инициализироваться только
    через конструктор.

3.  Превратите поля внешнего состояния в параметры методов, где эти поля
    использовались. Затем удалите поля из класса.

4.  Создайте фабрику, которая будет кешировать и повторно отдавать уже
    созданные объекты. Клиент должен запрашивать из этой фабрики
    легковеса с определённым внутренним состоянием, а не создавать его
    напрямую.

5.  Клиент должен хранить или вычислять значения внешнего состояния
    (контекст) и передавать его в методы объекта легковеса.
:::

:::::: {.section .pros-cons}
##  Преимущества и недостатки {#pros-cons}

::::: row
::: col-sm-6
-    Экономит оперативную память.
:::

::: col-sm-6
-    Расходует процессорное время на поиск/вычисление контекста.
-    Усложняет код программы из-за введения множества дополнительных
    классов.
:::
:::::
::::::

::: {.section .relations}
##  Отношения с другими паттернами {#relations}

-   [Компоновщик](composite.html) часто совмещают с
    [Легковесом](flyweight.html), чтобы реализовать общие ветки дерева и
    сэкономить при этом память.

-   [Легковес](flyweight.html) показывает, как создавать много мелких
    объектов, а [Фасад](facade.html) показывает, как создать один
    объект, который отображает целую подсистему.

-   Паттерн [Легковес](flyweight.html) может напоминать
    [Одиночку](singleton.html), если для конкретной задачи у вас
    получилось свести количество объектов к одному. Но помните, что
    между паттернами есть два кардинальных отличия:

    1.  В отличие от *Одиночки*, вы можете иметь множество
        объектов-легковесов.
    2.  Объекты-легковесы должны быть неизменяемыми, тогда как
        объект-одиночка допускает изменение своего состояния.
:::

::: {.section .implementations}
##  Примеры реализации паттерна {#implementations}

[![Легковес на
C#](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](flyweight/csharp/example.html "Легковес на C#"){.prog-lang-link}
[![Легковес на
C++](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](flyweight/cpp/example.html "Легковес на C++"){.prog-lang-link}
[![Легковес на
Go](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](flyweight/go/example.html "Легковес на Go"){.prog-lang-link}
[![Легковес на
Java](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](flyweight/java/example.html "Легковес на Java"){.prog-lang-link}
[![Легковес на
PHP](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](flyweight/php/example.html "Легковес на PHP"){.prog-lang-link}
[![Легковес на
Python](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](flyweight/python/example.html "Легковес на Python"){.prog-lang-link}
[![Легковес на
Ruby](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](flyweight/ruby/example.html "Легковес на Ruby"){.prog-lang-link}
[![Легковес на
Rust](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](flyweight/rust/example.html "Легковес на Rust"){.prog-lang-link}
[![Легковес на
Swift](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](flyweight/swift/example.html "Легковес на Swift"){.prog-lang-link}
[![Легковес на
TypeScript](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](flyweight/typescript/example.html "Легковес на TypeScript"){.prog-lang-link}
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

[Заместитель []{.fa .fa-arrow-right}](proxy.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Фасад](facade.html){.btn .btn-default
rel="prev"}
:::
:::::::::::::::::::::::::::::

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
:::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::
