:::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::: {.main-content-container .center-content-container}
:::::::::: {.page .text}
::: breadcrumb
[](../../index.html){.home} / [Рефакторинг](../refactoring.html){.type}
/ [Запахи кода](../refactoring/smells.html){.type} / [Остальные
запахи](../refactoring/smells/other.html){.type}
:::

# Неполнота библиотечного класса {#неполнота-библиотечного-класса .title}

::: aka
Также известен как: [Incomplete Library
Class]{style="display:inline-block"}
:::

### Симптомы и признаки

[Библиотеки](https://ru.wikipedia.org/wiki/Библиотека_(программирование))
через некоторое время перестают удовлетворять требованиям пользователей.
Естественное решение --- внести изменения в библиотеку --- очень часто
оказывается недоступным, так как библиотека закрыта для записи.

<figure class="image">
<img
src="../../images/refactoring/content/smells/incomplete-library-class-0152c5.png?id=ca51f740f7fd39b7de1430b64cae9f8c"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/incomplete-library-class-01.png?id=ca51f740f7fd39b7de1430b64cae9f8c 500w,/images/refactoring/content/smells/incomplete-library-class-01-1.5x.png?id=e0efa4b241a785afba5f0e85672dbe4a 750w,/images/refactoring/content/smells/incomplete-library-class-01-2x.png?id=25c39ccf56423153b7c977c57943af54 1000w"
sizes="(max-width: 720px) 100vw, 500px" data-fetchpriority="high"
width="500" height="300" />
</figure>

### Причины появления

Автор библиотеки не предусмотрел возможности, которые вам нужны, либо
отказался их внедрять.

### Лечение

-   Если надо добавить пару методов в библиотечный класс, используется
    [введение внешнего метода](../introduce-foreign-method.html).

-   Если надо серьёзно поменять поведение класса, используется [введение
    локального расширения](../introduce-local-extension.html).

### Выигрыш

-   Уменьшает дублирование кода (вместо создания своей библиотеки с
    нуля, вы используете готовую библиотеку).

<figure class="image">
<img
src="../../images/refactoring/content/smells/incomplete-library-class-02b14e.png?id=05a8d9c631d43a3fb256196f366fd089"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/incomplete-library-class-02.png?id=05a8d9c631d43a3fb256196f366fd089 500w,/images/refactoring/content/smells/incomplete-library-class-02-1.5x.png?id=bb28ce39729bc28b39030ed6b0ff60b0 750w,/images/refactoring/content/smells/incomplete-library-class-02-2x.png?id=cb204d62084939b3d9e6f97d5d3662ee 1000w"
sizes="(max-width: 720px) 100vw, 500px" loading="lazy" width="500"
height="300" />
</figure>

### Не стоит трогать, если\...

-   Расширение библиотеки может стать причиной появления дополнительного
    объема работы. Это происходит в том случае, когда изменения в
    библиотеке затрагивают изменения в коде.

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

[Приёмы []{.fa .fa-arrow-right}](../refactoring/techniques.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Остальные
запахи](../refactoring/smells/other.html){.btn .btn-default rel="prev"}
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
