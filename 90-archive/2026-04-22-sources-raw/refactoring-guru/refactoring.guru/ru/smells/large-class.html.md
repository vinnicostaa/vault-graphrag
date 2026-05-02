:::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::: {.main-content-container .center-content-container}
:::::::::: {.page .text}
::: breadcrumb
[](../../index.html){.home} / [Рефакторинг](../refactoring.html){.type}
/ [Запахи кода](../refactoring/smells.html){.type} /
[Раздувальщики](../refactoring/smells/bloaters.html){.type}
:::

# Большой класс {#большой-класс .title}

::: aka
Также известен как: [Large Class]{style="display:inline-block"}
:::

### Симптомы и признаки

Класс содержит множество полей/методов/строк кода.

<figure class="image">
<img
src="../../images/refactoring/content/smells/large-class-01eaa7.png?id=acac82f25cc90aaa413c2daefebf0e4b"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/large-class-01.png?id=acac82f25cc90aaa413c2daefebf0e4b 500w,/images/refactoring/content/smells/large-class-01-1.5x.png?id=ba2cf337d9a765204a1b313169f65cf8 750w,/images/refactoring/content/smells/large-class-01-2x.png?id=44aea94399b8bd6398a01b46b5bc7f29 1000w"
sizes="(max-width: 720px) 100vw, 500px" data-fetchpriority="high"
width="500" height="300" />
</figure>

### Причины появления

Классы редко бывают большими изначально. Но со временем постепенно
многие из них «раздуваются» в связи с развитием программы.

Как и в случае с длинными методами, чаще всего программисту ментально
проще добавить фичу в существующий класс, чем создать новый класс для
этой фичи.

<figure class="image">
<img
src="../../images/refactoring/content/smells/large-class-027a0b.png?id=973b37334ae57489945a88b9327f81e3"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/large-class-02.png?id=973b37334ae57489945a88b9327f81e3 500w,/images/refactoring/content/smells/large-class-02-1.5x.png?id=4b4846ad906c16339481236196214bcb 750w,/images/refactoring/content/smells/large-class-02-2x.png?id=f51627abdfb96fad29cb114d00795fec 1000w"
sizes="(max-width: 720px) 100vw, 500px" loading="lazy" width="500"
height="300" />
</figure>

### Лечение

Когда класс реализует слишком обширный функционал, стоит подумать о его
разделении:

-   [Извлечение класса](../extract-class.html) поможет, если часть
    поведения большого класса может быть выделена в свой собственный
    компонент.

-   [Извлечение подкласса](../extract-subclass.html) поможет, если часть
    поведения большого класса может иметь альтернативные реализации либо
    используется в редких случаях.

-   [Извлечение интерфейса](../extract-interface.html) поможет, если
    нужно иметь список операций и поведений, которые клиент сможет
    использовать.

-   В классах графического интерфейса часто можно найти данные и
    поведения, которые не относятся к непосредственной отрисовке
    интерфейса, а скорее отвечают за общую логику работы. Такие данные и
    поведения следует выделить в отдельный класс предметной области,
    который бы управлял работой графического интерфейса. При этом может
    оказаться необходимым хранить копии некоторых данных в двух местах и
    обеспечить их согласованность. [Дублирование видимых
    данных](../duplicate-observed-data.html) предлагает путь, которым
    можно это осуществить.

<figure class="image">
<img
src="../../images/refactoring/content/smells/large-class-030a79.png?id=f0a0109f731dbc420ffe385cb658f0de"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/large-class-03.png?id=f0a0109f731dbc420ffe385cb658f0de 500w,/images/refactoring/content/smells/large-class-03-1.5x.png?id=8f7537669b0af86e9e77928004ad865f 750w,/images/refactoring/content/smells/large-class-03-2x.png?id=2e497ff65fc035f0d51f908361daee78 1000w"
sizes="(max-width: 720px) 100vw, 500px" loading="lazy" width="500"
height="300" />
</figure>

### Выигрыш

-   Рефакторинг таких классов избавит разработчиков от необходимости
    запоминать чрезмерное количество имеющихся у класса атрибутов.

-   Во многих случаях разделение больших классов на части позволяет
    избежать дублирования кода и функциональности.

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

[Одержимость элементарными типами []{.fa
.fa-arrow-right}](primitive-obsession.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Длинный метод](long-method.html){.btn
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
