:::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::: {.main-content-container .center-content-container}
:::::::::: {.page .text}
::: breadcrumb
[](../../index.html){.home} / [Рефакторинг](../refactoring.html){.type}
/ [Запахи кода](../refactoring/smells.html){.type} /
[Замусориватели](../refactoring/smells/dispensables.html){.type}
:::

# Класс данных {#класс-данных .title}

::: aka
Также известен как: [Data Class]{style="display:inline-block"}
:::

### Симптомы и признаки

Классы данных --- это классы, которые содержат только поля и простейшие
методы для доступа к ним (геттеры и сеттеры). Это просто контейнеры для
данных, используемые другими классами. Эти классы не содержат никакой
дополнительной функциональности и не могут самостоятельно работать с
данными, которыми владеют.

<figure class="image">
<img
src="../../images/refactoring/content/smells/data-class-01949b.png?id=2ea1583b05a194a056d27ac559545318"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/data-class-01.png?id=2ea1583b05a194a056d27ac559545318 500w,/images/refactoring/content/smells/data-class-01-1.5x.png?id=e816a7b99f2cbef399ea43b1af66125b 750w,/images/refactoring/content/smells/data-class-01-2x.png?id=2beb8150d4ba31ca37d6515495ceff2d 1000w"
sizes="(max-width: 720px) 100vw, 500px" data-fetchpriority="high"
width="500" height="300" />
</figure>

### Причины появления

Это нормально, когда класс в начале своей жизни содержит всего лишь
несколько публичных полей (а может даже и парочку геттеров/сеттеров).
Тем не менее, настоящая сила объектов заключается в том, что они могут
хранить типы поведения или операции над собственными данными.

### Лечение

-   Если класс содержит публичные поля, примените [инкапсуляцию
    поля](../encapsulate-field.html) чтобы скрыть их из прямого доступа,
    разрешив доступ только через геттеры и сеттеры.

-   Примените [инкапсуляцию коллекции](../encapsulate-collection.html)
    для данных, которые хранятся в коллекциях (вроде массивов).

-   Осмотрите клиентский код, который использует этот класс. Возможно,
    там вы найдёте функциональность, которая смотрелась бы уместнее в
    самом классе данных. В этом случае используйте [перемещение
    метода](../move-method.html) и [извлечение
    метода](../extract-method.html) для переноса функциональности в
    класс данных.

-   После того как класс наполнился осмысленными методами, возможно,
    стоит подумать об уничтожении старых методов доступа к данным,
    которые дают слишком открытый доступ к данным класса. В этом вам
    поможет [удаление сеттера](../remove-setting-method.html) и
    [сокрытие метода](../hide-method.html).

<figure class="image">
<img
src="../../images/refactoring/content/smells/data-class-024cf2.png?id=db0eb15f9f229bafd8423b2cfd09f910"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/data-class-02.png?id=db0eb15f9f229bafd8423b2cfd09f910 500w,/images/refactoring/content/smells/data-class-02-1.5x.png?id=f2f0339a74ba638bfbd8d57317396f43 750w,/images/refactoring/content/smells/data-class-02-2x.png?id=fb9b6d670232d6effe790980e6b388ec 1000w"
sizes="(max-width: 720px) 100vw, 500px" loading="lazy" width="500"
height="300" />
</figure>

### Выигрыш

-   Улучшает понимание и организацию кода. Операции над определёнными
    данными теперь собраны в одном месте, их не надо искать по всему
    коду.

-   Может вскрыть факты дублирования клиентского кода.

::::: {.prom .banner-content .banner .banner-striped .banner-with-image data-id="Ref: Part of the ebook" creative-id="animated-ru" position="content_bottom"}
::: banner-image
Your browser does not support HTML video.
:::

::: banner-text
### Устали читать? {#устали-читать .title}

Сбегайте за подушкой, у нас тут контента на [7 часов]{.blue} чтения.

Или попробуйте наш интерактивный курс. Он гораздо более интересный, чем
банальный текст.

[ Узнать больше...](../refactoring/course.html){.btn .btn-secondary}
:::
:::::

::: next
#### Читаем дальше

[Мёртвый код []{.fa .fa-arrow-right}](dead-code.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Ленивый класс](lazy-class.html){.btn
.btn-default rel="prev"}
:::
::::::::::

:::::: {.prom .banner-sidebar .banner-removable .banner-removable-refactoring data-id="Ref: Sidebar" creative-id="standard-sidebar-ru" position="sidebar"}
::::: ban
::: {.image .product-image}
[Скидки!]{.banner-discount}
[![](../../images/refactoring/course/course-cover-rua60d.jpg?id=e9d6b4015f4c6c48cf06f7479874d8d7){width="300"
height="300" loading="lazy"}](../refactoring/course.html)
:::

::: {.banner-text .banner-text-ru}
Этот запах кода --- малая часть интерактивного **онлайн курса по
рефакторингу**.

[ Узнать больше...](../refactoring/course.html){.btn .btn-secondary
.btn-block}
:::
:::::
::::::
:::::::::::::::
::::::::::::::::
