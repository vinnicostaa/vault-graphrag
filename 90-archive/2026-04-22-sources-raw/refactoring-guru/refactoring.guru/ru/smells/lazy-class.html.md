:::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::: {.main-content-container .center-content-container}
:::::::::: {.page .text}
::: breadcrumb
[](../../index.html){.home} / [Рефакторинг](../refactoring.html){.type}
/ [Запахи кода](../refactoring/smells.html){.type} /
[Замусориватели](../refactoring/smells/dispensables.html){.type}
:::

# Ленивый класс {#ленивый-класс .title}

::: aka
Также известен как: [Lazy Class]{style="display:inline-block"}
:::

### Симптомы и признаки

На понимание и поддержку классов всегда требуются затраты времени и
денег. А потому, если класс не делает достаточно много, чтобы уделять
ему достаточно внимания, он должен быть уничтожен.

<figure class="image">
<img
src="../../images/refactoring/content/smells/lazy-class-0148c1.png?id=efec5911dfaaa3ba69d3eb4dab03fd3c"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/lazy-class-01.png?id=efec5911dfaaa3ba69d3eb4dab03fd3c 500w,/images/refactoring/content/smells/lazy-class-01-1.5x.png?id=4b9cf29aa6991dc8050debc5494a2a92 750w,/images/refactoring/content/smells/lazy-class-01-2x.png?id=3b5b07bc60eb98c883fc68c5e1a05aed 1000w"
sizes="(max-width: 720px) 100vw, 500px" data-fetchpriority="high"
width="500" height="300" />
</figure>

### Причины появления

Это может произойти, если класс был задуман как полнофункциональный, но
в результате рефакторинга ужался до неприличных размеров.

Либо класс создавался в расчёте на некие будущие разработки, до которых
руки так и не дошли.

### Лечение

-   Почти бесполезные компоненты должны быть подвергнуты [встраиванию
    класса](../inline-class.html).

-   При наличии подклассов с недостаточными функциями попробуйте
    [свёртывание иерархии](../collapse-hierarchy.html).

<figure class="image">
<img
src="../../images/refactoring/content/smells/lazy-class-02adb6.png?id=393302f2bd27ba0197660caea274ae23"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/lazy-class-02.png?id=393302f2bd27ba0197660caea274ae23 500w,/images/refactoring/content/smells/lazy-class-02-1.5x.png?id=72b0880a8673312f80bd10755cdeae74 750w,/images/refactoring/content/smells/lazy-class-02-2x.png?id=d46dd63f159b40aa266ccbdbefb319bd 1000w"
sizes="(max-width: 720px) 100vw, 500px" loading="lazy" width="500"
height="300" />
</figure>

### Выигрыш

-   Уменьшение размера кода.

-   Упрощение поддержки.

### Не стоит трогать, если\...

-   Иногда *Ленивый класс* бывает создан для того, чтобы явно очертить
    какие-то намерения. В этом случае, стоит соблюдать баланс понятности
    кода и его простоты.

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

[Класс данных []{.fa .fa-arrow-right}](data-class.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Дублирование кода](duplicate-code.html){.btn
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
