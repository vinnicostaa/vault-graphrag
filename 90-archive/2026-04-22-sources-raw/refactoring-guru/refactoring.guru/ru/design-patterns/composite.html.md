::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [Паттерны
проектирования](../design-patterns.html){.type} / [Структурные
паттерны](structural-patterns.html){.type}
:::

# Компоновщик {#компоновщик .title}

::: aka
Также известен как:
[Дерево, ]{style="display:inline-block"}[Composite]{style="display:inline-block"}
:::

::: {.section .intent}
##  Суть паттерна {#intent}

**Компоновщик** --- это структурный паттерн проектирования, который
позволяет сгруппировать множество объектов в древовидную структуру, а
затем работать с ней так, как будто это единичный объект.

<figure class="image">
<img
src="../../images/patterns/content/composite/compositec299.png?id=73bcf0d94db360b636cd745f710d19db"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/composite/composite.png?id=73bcf0d94db360b636cd745f710d19db 640w,/images/patterns/content/composite/composite-1.5x.png?id=098acddafae70b1ec1cb1b05e833c3ae 960w,/images/patterns/content/composite/composite-2x.png?id=8847e6f8e2cb892ed2229faba83bd1b7 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Паттерн Компоновщик" />
</figure>
:::

::: {.section .problem}
##  Проблема {#problem}

Паттерн Компоновщик имеет смысл только тогда, когда основная модель
вашей программы может быть структурирована в виде дерева.

Например, есть два объекта: `Продукт` и `Коробка`. `Коробка` может
содержать несколько `Продуктов` и других `Коробок` поменьше. Те, в свою
очередь, тоже содержат либо `Продукты`, либо `Коробки` и так далее.

Теперь предположим, ваши `Продукты` и `Коробки` могут быть частью
заказов. Каждый заказ может содержать как простые `Продукты` без
упаковки, так и составные `Коробки`. Ваша задача состоит в том, чтобы
узнать цену всего заказа.

<figure class="image">
<img
src="../../images/patterns/diagrams/composite/problem-rud8b0.png?id=d05b9da7e4776c853e783ecbacd09308"
style="aspect-ratio: 1;"
srcset="/images/patterns/diagrams/composite/problem-ru.png?id=d05b9da7e4776c853e783ecbacd09308 370w,/images/patterns/diagrams/composite/problem-ru-1.5x.png?id=c8b7f2d63b166019f023a4ddfa9439fe 555w,/images/patterns/diagrams/composite/problem-ru-2x.png?id=9079f9c1003bc7c43427a9e16b8422c5 740w"
sizes="(max-width: 720px) 100vw, 370px" loading="lazy" width="370"
alt="Структура сложного заказа" />
<figcaption><p>Заказ может состоять из различных продуктов, упакованных
в собственные коробки.</p></figcaption>
</figure>

Если решать задачу в лоб, то вам потребуется открыть все коробки заказа,
перебрать все продукты и посчитать их суммарную стоимость. Но это
слишком хлопотно, так как типы коробок и их содержимое могут быть вам
неизвестны. Кроме того, наперёд неизвестно и количество уровней
вложенности коробок, поэтому перебрать коробки простым циклом не выйдет.
:::

::: {.section .solution}
##  Решение {#solution}

Компоновщик предлагает рассматривать `Продукт` и `Коробку` через единый
интерфейс с общим методом получения стоимости.

`Продукт` просто вернёт свою цену. `Коробка` спросит цену каждого
предмета внутри себя и вернёт сумму результатов. Если одним из
внутренних предметов окажется коробка поменьше, она тоже будет
перебирать своё содержимое, и так далее, пока не будут посчитаны все
составные части.

<figure class="image">
<img
src="../../images/patterns/content/composite/composite-comic-1-rub2f1.png?id=f8f5303a2010179bdae1371615e295a4"
style="aspect-ratio: 2;"
srcset="/images/patterns/content/composite/composite-comic-1-ru.png?id=f8f5303a2010179bdae1371615e295a4 600w,/images/patterns/content/composite/composite-comic-1-ru-1.5x.png?id=17610b0de0a9f47bf420bf35117955bc 900w,/images/patterns/content/composite/composite-comic-1-ru-2x.png?id=35a58fe3be308d39d80c596f5ff5d21a 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Решение с Компоновщиком" />
<figcaption><p>Компоновщик рекурсивно запускает действие по всем
элементам дерева — от корня к листьям.</p></figcaption>
</figure>

Для вас, клиента, главное, что теперь не нужно ничего знать о структуре
заказов. Вы вызываете метод получения цены, он возвращает цифру, а вы не
тонете в горах картона и скотча.
:::

::: {.section .analogy}
##  Аналогия из жизни {#analogy}

<figure class="image">
<img
src="../../images/patterns/diagrams/composite/live-examplee724.png?id=548a7cec45b493af66e8bfe524a137d3"
style="aspect-ratio: 1.22;"
srcset="/images/patterns/diagrams/composite/live-example.png?id=548a7cec45b493af66e8bfe524a137d3 280w,/images/patterns/diagrams/composite/live-example-1.5x.png?id=2129195f4103a5a4d3440a086ce3dc87 420w,/images/patterns/diagrams/composite/live-example-2x.png?id=b555458f20fc30425ae6dada5da492af 560w"
sizes="(max-width: 720px) 100vw, 280px" loading="lazy" width="280"
alt="Пример армейской структуры" />
<figcaption><p>Пример армейской структуры.</p></figcaption>
</figure>

Армии большинства государств могут быть представлены в виде перевёрнутых
деревьев. На нижнем уровне у вас есть солдаты, затем взводы, затем
полки, а затем целые армии. Приказы отдаются сверху и спускаются вниз по
структуре командования, пока не доходят до конкретного солдата.
:::

::::: {.section .structure-container}
##  Структура {#structure}

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/composite/structure-ru263a.png?id=43f15362885e26953c04791230ab72df"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 0.8;"
srcset="/images/patterns/diagrams/composite/structure-ru.png?id=43f15362885e26953c04791230ab72df 360w,/images/patterns/diagrams/composite/structure-ru-1.5x.png?id=f7f5514f6339c541ac345d7d58e90da3 540w,/images/patterns/diagrams/composite/structure-ru-2x.png?id=12e816a1e6b2b41a071b0b114b24f2b7 720w"
sizes="(max-width: 720px) 100vw, 360px" loading="lazy" width="360"
alt="Структура классов паттерна Компоновщик" /><img
src="../../images/patterns/diagrams/composite/structure-ru-indexed96d8.png?id=274e1678d04edece151bf2fdfd8c33f7"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 0.85;"
srcset="/images/patterns/diagrams/composite/structure-ru-indexed.png?id=274e1678d04edece151bf2fdfd8c33f7 400w,/images/patterns/diagrams/composite/structure-ru-indexed-1.5x.png?id=d30a2b0eb4f884e78dd6c7c222ab5702 600w,/images/patterns/diagrams/composite/structure-ru-indexed-2x.png?id=adc2dbef505122987d3899e10b299be7 800w"
sizes="(max-width: 720px) 100vw, 400px" loading="lazy" width="400"
alt="Структура классов паттерна Компоновщик" />
</figure>
:::

1.  **Компонент** определяет общий интерфейс для простых и составных
    компонентов дерева.

2.  **Лист** --- это простой компонент дерева, не имеющий ответвлений.

    Из-за того, что им некому больше передавать выполнение, классы
    листьев будут содержать большую часть полезного кода.

3.  **Контейнер** (или *композит*) --- это составной компонент дерева.
    Он содержит набор дочерних компонентов, но ничего не знает об их
    типах. Это могут быть как простые компоненты-листья, так и другие
    компоненты-контейнеры. Но это не является проблемой, если все
    дочерние компоненты следуют единому интерфейсу.

    Методы контейнера переадресуют основную работу своим дочерним
    компонентам, хотя и могут добавлять что-то своё к результату.

4.  **Клиент** работает с деревом через общий интерфейс компонентов.

    Благодаря этому, клиенту не важно, что перед ним находится ---
    простой или составной компонент дерева.
::::
:::::

::: {.section .pseudocode}
##  Псевдокод {#pseudocode}

В этом примере **Компоновщик** помогает реализовать вложенные
геометрические фигуры.

<figure class="image">
<img
src="../../images/patterns/diagrams/composite/example7ea7.png?id=98ba81d07c979038dd2ebf3c83a2e19f"
style="aspect-ratio: 0.84;"
srcset="/images/patterns/diagrams/composite/example.png?id=98ba81d07c979038dd2ebf3c83a2e19f 370w,/images/patterns/diagrams/composite/example-1.5x.png?id=ae008efd4146628cd7303925d4ce90be 555w,/images/patterns/diagrams/composite/example-2x.png?id=d21edef39d3792e8a4c6736727ac7305 740w"
sizes="(max-width: 720px) 100vw, 370px" loading="lazy" width="370"
alt="Структура классов примера паттерна Компоновщик" />
<figcaption><p>Пример редактора геометрических фигур.</p></figcaption>
</figure>

Класс `CompoundGraphic` может содержать любое количество подфигур,
включая такие же контейнеры, как он сам. Контейнер реализует те же
методы, что и простые фигуры. Но, вместо непосредственного действия, он
передаёт вызовы всем вложенным компонентам, используя рекурсию. Затем он
как бы «суммирует» результаты всех вложенных фигур.

Клиентский код работает со всеми фигурами через общий интерфейс фигур и
не знает, что перед ним --- простая фигура или составная. Это позволяет
клиентскому коду работать с деревьями объектов любой сложности, не
привязываясь к конкретным классам объектов, формирующих дерево.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// Общий интерфейс компонентов.
interface Graphic is
    method move(x, y)
    method draw()

// Простой компонент.
class Dot implements Graphic is
    field x, y

    constructor Dot(x, y) { ... }

    method move(x, y) is
        this.x += x, this.y += y

    method draw() is
        // Нарисовать точку в координате X, Y.

// Компоненты могут расширять другие компоненты.
class Circle extends Dot is
    field radius

    constructor Circle(x, y, radius) { ... }

    method draw() is
        // Нарисовать окружность в координате X, Y и радиусом R.

// Контейнер содержит операции добавления/удаления дочерних
// компонентов. Все стандартные операции интерфейса компонентов
// он делегирует каждому из дочерних компонентов.
class CompoundGraphic implements Graphic is
    field children: array of Graphic

    method add(child: Graphic) is
        // Добавить компонент в список дочерних.

    method remove(child: Graphic) is
        // Убрать компонент из списка дочерних.

    method move(x, y) is
        foreach (child in children) do
            child.move(x, y)

    method draw() is
        // 1. Для каждого дочернего компонента:
        //     - Отрисовать компонент.
        //     - Определить координаты максимальной границы.
        // 2. Нарисовать пунктирную границу вокруг всей области.


// Приложение работает единообразно как с единичными
// компонентами, так и с целыми группами компонентов.
class ImageEditor is
    field all: CompoundGraphic

    method load() is
        all = new CompoundGraphic()
        all.add(new Dot(1, 2))
        all.add(new Circle(5, 3, 10))
        // ...

    // Группировка выбранных компонентов в один сложный
    // компонент.
    method groupSelected(components: array of Graphic) is
        group = new CompoundGraphic()
        foreach (component in components) do
            group.add(component)
            all.remove(component)
        all.add(group)
        // Все компоненты будут отрисованы.
        all.draw()</code></pre>
</figure>
:::

:::::::: {.section .applicability-container}
##  Применимость {#applicability}

::::::: applicability
::: applicability-problem
Когда вам нужно представить древовидную структуру объектов.
:::

::: applicability-solution
Паттерн Компоновщик предлагает хранить в составных объектах ссылки на
другие простые или составные объекты. Те, в свою очередь, тоже могут
хранить свои вложенные объекты и так далее. В итоге вы можете строить
сложную древовидную структуру данных, используя всего две основные
разновидности объектов.
:::

::: applicability-problem
Когда клиенты должны единообразно трактовать простые и составные
объекты.
:::

::: applicability-solution
Благодаря тому, что простые и составные объекты реализуют общий
интерфейс, клиенту безразлично, с каким именно объектом ему предстоит
работать.
:::
:::::::
::::::::

::: {.section .checklist}
##  Шаги реализации {#checklist}

1.  Убедитесь, что вашу бизнес-логику можно представить как древовидную
    структуру. Попытайтесь разбить её на простые компоненты и
    контейнеры. Помните, что контейнеры могут содержать как простые
    компоненты, так и другие вложенные контейнеры.

2.  Создайте общий интерфейс компонентов, который объединит операции
    контейнеров и простых компонентов дерева. Интерфейс будет удачным,
    если вы сможете использовать его, чтобы взаимозаменять простые и
    составные компоненты без потери смысла.

3.  Создайте класс компонентов-листьев, не имеющих дальнейших
    ответвлений. Имейте в виду, что программа может содержать несколько
    таких классов.

4.  Создайте класс компонентов-контейнеров и добавьте в него массив для
    хранения ссылок на вложенные компоненты. Этот массив должен быть
    способен содержать как простые, так и составные компоненты, поэтому
    убедитесь, что он объявлен с типом интерфейса компонентов.

    Реализуйте в контейнере методы интерфейса компонентов, помня о том,
    что контейнеры должны делегировать основную работу своим дочерним
    компонентам.

5.  Добавьте операции добавления и удаления дочерних компонентов в класс
    контейнеров.

    Имейте в виду, что методы добавления/удаления дочерних компонентов
    можно поместить и в интерфейс компонентов. Да, это нарушит *принцип
    разделения интерфейса*, так как реализации методов будут пустыми в
    компонентах-листьях. Но зато все компоненты дерева станут
    действительно одинаковыми для клиента.
:::

:::::: {.section .pros-cons}
##  Преимущества и недостатки {#pros-cons}

::::: row
::: col-sm-6
-    Упрощает архитектуру клиента при работе со сложным деревом
    компонентов.
-    Облегчает добавление новых видов компонентов.
:::

::: col-sm-6
-    Создаёт слишком общий дизайн классов.
:::
:::::
::::::

::: {.section .relations}
##  Отношения с другими паттернами {#relations}

-   [Строитель](builder.html) позволяет пошагово сооружать дерево
    [Компоновщика](composite.html).

-   [Цепочку обязанностей](chain-of-responsibility.html) часто
    используют вместе с [Компоновщиком](composite.html). В этом случае
    запрос передаётся от дочерних компонентов к их родителям.

-   Вы можете обходить дерево [Компоновщика](composite.html), используя
    [Итератор](iterator.html).

-   Вы можете выполнить какое-то действие над всем деревом
    [Компоновщика](composite.html) при помощи
    [Посетителя](visitor.html).

-   [Компоновщик](composite.html) часто совмещают с
    [Легковесом](flyweight.html), чтобы реализовать общие ветки дерева и
    сэкономить при этом память.

-   [Компоновщик](composite.html) и [Декоратор](decorator.html) имеют
    похожие структуры классов из-за того, что оба построены на
    рекурсивной вложенности. Она позволяет связать в одну структуру
    бесконечное количество объектов.

    *Декоратор* оборачивает только один объект, а узел *Компоновщика*
    может иметь много детей. *Декоратор* добавляет вложенному объекту
    новую функциональность, а *Компоновщик* не добавляет ничего нового,
    но «суммирует» результаты всех своих детей.

    Но они могут и сотрудничать: *Компоновщик* может использовать
    *Декоратор*, чтобы переопределить функции отдельных частей дерева
    компонентов.

-   Архитектура, построенная на [Компоновщиках](composite.html) и
    [Декораторах](decorator.html), часто может быть улучшена за счёт
    внедрения [Прототипа](prototype.html). Он позволяет клонировать
    сложные структуры объектов, а не собирать их заново.
:::

::: {.section .implementations}
##  Примеры реализации паттерна {#implementations}

[![Компоновщик на
C#](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](composite/csharp/example.html "Компоновщик на C#"){.prog-lang-link}
[![Компоновщик на
C++](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](composite/cpp/example.html "Компоновщик на C++"){.prog-lang-link}
[![Компоновщик на
Go](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](composite/go/example.html "Компоновщик на Go"){.prog-lang-link}
[![Компоновщик на
Java](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](composite/java/example.html "Компоновщик на Java"){.prog-lang-link}
[![Компоновщик на
PHP](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](composite/php/example.html "Компоновщик на PHP"){.prog-lang-link}
[![Компоновщик на
Python](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](composite/python/example.html "Компоновщик на Python"){.prog-lang-link}
[![Компоновщик на
Ruby](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](composite/ruby/example.html "Компоновщик на Ruby"){.prog-lang-link}
[![Компоновщик на
Rust](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](composite/rust/example.html "Компоновщик на Rust"){.prog-lang-link}
[![Компоновщик на
Swift](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](composite/swift/example.html "Компоновщик на Swift"){.prog-lang-link}
[![Компоновщик на
TypeScript](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](composite/typescript/example.html "Компоновщик на TypeScript"){.prog-lang-link}
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

[Декоратор []{.fa .fa-arrow-right}](decorator.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Мост](bridge.html){.btn .btn-default rel="prev"}
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
