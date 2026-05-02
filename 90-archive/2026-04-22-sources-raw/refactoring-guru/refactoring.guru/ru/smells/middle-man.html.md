:::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::: {.main-content-container .center-content-container}
:::::::::: {.page .text}
::: breadcrumb
[](../../index.html){.home} / [Рефакторинг](../refactoring.html){.type}
/ [Запахи кода](../refactoring/smells.html){.type} / [Опутыватели
связями](../refactoring/smells/couplers.html){.type}
:::

# Посредник {#посредник .title}

::: aka
Также известен как: [Middle Man]{style="display:inline-block"}
:::

### Симптомы и признаки

Если класс выполняет одно действие --- делегирует работу другому
классу --- стоит задуматься, зачем он вообще существует.

<figure class="image">
<img
src="../../images/refactoring/content/smells/middle-man-01d4ce.png?id=14c65845c4e0cf03e7e9e48108090c98"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/middle-man-01.png?id=14c65845c4e0cf03e7e9e48108090c98 500w,/images/refactoring/content/smells/middle-man-01-1.5x.png?id=9fd227d0b8d3e082fef36f02df56d907 750w,/images/refactoring/content/smells/middle-man-01-2x.png?id=a1a99f8b475b719d9f894aa613515761 1000w"
sizes="(max-width: 720px) 100vw, 500px" data-fetchpriority="high"
width="500" height="300" />
</figure>

### Причины появления

Данный запах может быть результатом фанатичной борьбы с [цепочками
вызовов](message-chains.html).

Кроме того, бывает так, что вся полезная нагрузка класса постепенно
перемещается в другие классы, в результате кроме делегирующих методов в
нем ничего не остается.

### Лечение

-   Если большую часть методов класс делегирует другому классу, нужно
    воспользоваться [удалением посредника](../remove-middle-man.html).

### Выигрыш

-   Уменьшение размера кода.

<figure class="image">
<img
src="../../images/refactoring/content/smells/middle-man-02455a.png?id=f507c0fd9a7bde8df8c22b9027d0a404"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/middle-man-02.png?id=f507c0fd9a7bde8df8c22b9027d0a404 500w,/images/refactoring/content/smells/middle-man-02-1.5x.png?id=4ff17bc08cd0dd6f84473f971873f154 750w,/images/refactoring/content/smells/middle-man-02-2x.png?id=41869f090e8263d46e708778fe64059c 1000w"
sizes="(max-width: 720px) 100vw, 500px" loading="lazy" width="500"
height="300" />
</figure>

### Не стоит трогать, если\...

Не удаляйте посредников, которые были созданы осознанно:

-   Посредник мог быть введён для избавления от нежелательной
    зависимости между классами.

-   Некоторые паттерны проектирования намеренно создают посредников
    (например, [Заместитель](../design-patterns/proxy.html) или
    [Декоратор](../design-patterns/decorator.html)).

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

[Остальные запахи []{.fa
.fa-arrow-right}](../refactoring/smells/other.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Цепочка вызовов](message-chains.html){.btn
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
