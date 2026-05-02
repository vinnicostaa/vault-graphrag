:::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::: {.main-content-container .center-content-container}
:::::::::: {.page .text}
::: breadcrumb
[](../../index.html){.home} / [Рефакторинг](../refactoring.html){.type}
/ [Запахи кода](../refactoring/smells.html){.type} / [Утяжелители
изменений](../refactoring/smells/change-preventers.html){.type}
:::

# Параллельные иерархии наследования {#параллельные-иерархии-наследования .title}

::: aka
Также известен как: [Parallel Inheritance
Hierarchies]{style="display:inline-block"}
:::

### Симптомы и признаки

Всякий раз при создании подкласса какого-то класса приходится создавать
ещё один подкласс для другого класса.

<figure class="image">
<img
src="../../images/refactoring/content/smells/parallel-inheritance-hierarchies-0187af.png?id=9167875f5f0e80256edcc8fcaaed3563"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/parallel-inheritance-hierarchies-01.png?id=9167875f5f0e80256edcc8fcaaed3563 500w,/images/refactoring/content/smells/parallel-inheritance-hierarchies-01-1.5x.png?id=e2129d5eb8da79a491ed0c7d802e8d1c 750w,/images/refactoring/content/smells/parallel-inheritance-hierarchies-01-2x.png?id=975e6a0589795c59b47ed3aa122beead 1000w"
sizes="(max-width: 720px) 100vw, 500px" data-fetchpriority="high"
width="500" height="300" />
</figure>

### Причины появления

Пока иерархия была небольшая, всё было хорошо. Но с появлением новых
классов вносить изменения становилось всё сложнее и сложнее.

### Лечение

-   Вы можете попытаться устранить дублирования паралельных классов в
    два этапа. Во-первых, нужно заставить экземпляры одной иерархии
    ссылаться на экземпляры другой иерархии. Затем следует убрать
    иерархию в ссылающемся классе с помощью [перемещения
    метода](../move-method.html) и [перемещения
    поля](../move-field.html).

### Выигрыш

-   Уменьшает дублирования кода.

-   Может улучшить организацию кода.

<figure class="image">
<img
src="../../images/refactoring/content/smells/parallel-inheritance-hierarchies-02b2cf.png?id=4dca6795d3d087b23ad1027298d6f1dd"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/parallel-inheritance-hierarchies-02.png?id=4dca6795d3d087b23ad1027298d6f1dd 500w,/images/refactoring/content/smells/parallel-inheritance-hierarchies-02-1.5x.png?id=ef89bd3332ebb11a6c6974a9e8017779 750w,/images/refactoring/content/smells/parallel-inheritance-hierarchies-02-2x.png?id=b45e8dde4f4abbe2f0d329964c921960 1000w"
sizes="(max-width: 720px) 100vw, 500px" loading="lazy" width="500"
height="300" />
</figure>

### Не стоит трогать, если\...

-   Иногда наличие паралельной иерархии --- это необходимое зло, без
    которого устройство программы было бы еще хуже. Если вы обнаружите,
    что ваши попытки устранить дублирование приводят к еще большему
    ухудшению организации кода, то\... остановите рефакторинг, откатите
    все внесенные изменения, выпейте чаю и начните привыкать к этому
    коду.

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

[Замусориватели []{.fa
.fa-arrow-right}](../refactoring/smells/dispensables.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Стрельба дробью](shotgun-surgery.html){.btn
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
