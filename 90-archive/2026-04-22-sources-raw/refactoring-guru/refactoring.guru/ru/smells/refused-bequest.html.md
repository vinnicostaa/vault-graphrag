:::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::: {.main-content-container .center-content-container}
:::::::::: {.page .text}
::: breadcrumb
[](../../index.html){.home} / [Рефакторинг](../refactoring.html){.type}
/ [Запахи кода](../refactoring/smells.html){.type} / [Нарушители
объектно-ориентированного
дизайна](../refactoring/smells/oo-abusers.html){.type}
:::

# Отказ от наследства {#отказ-от-наследства .title}

::: aka
Также известен как: [Refused Bequest]{style="display:inline-block"}
:::

### Симптомы и признаки

Если подкласс использует лишь малую часть унаследованных методов и
свойств суперкласса, это является признаком неправильной иерархии. При
этом ненужные методы могут просто не использоваться либо быть
переопределёнными и выбрасывать исключения.

<figure class="image">
<img
src="../../images/refactoring/content/smells/refused-bequest-014f59.png?id=7a1d79e75a3836c22ec865d72c98664e"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/refused-bequest-01.png?id=7a1d79e75a3836c22ec865d72c98664e 500w,/images/refactoring/content/smells/refused-bequest-01-1.5x.png?id=90bf2279c14d74bc0e63b9bcc3b69808 750w,/images/refactoring/content/smells/refused-bequest-01-2x.png?id=d2e31b7b9fa3326118817b8e0c65e435 1000w"
sizes="(max-width: 720px) 100vw, 500px" data-fetchpriority="high"
width="500" height="300" />
</figure>

### Причины появления

Кто-то создал наследование между классами только из побуждений
повторного использования кода, находящегося в суперклассе. При этом
суперкласс и подкласс могут являться совершенно различными сущностями.

<figure class="image">
<img
src="../../images/refactoring/content/smells/refused-bequest-02c0d0.png?id=f9b0affd4bbf6fec22c05783fc75562e"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/refused-bequest-02.png?id=f9b0affd4bbf6fec22c05783fc75562e 500w,/images/refactoring/content/smells/refused-bequest-02-1.5x.png?id=28b34af7a9319c2d15c32e4a5890c835 750w,/images/refactoring/content/smells/refused-bequest-02-2x.png?id=33b42b7d51bca13f27e4933d24f82751 1000w"
sizes="(max-width: 720px) 100vw, 500px" loading="lazy" width="500"
height="300" />
</figure>

### Лечение

-   Если наследование не имеет смысла, и подкласс в действительности не
    является представителем суперкласса, следует избавиться от отношения
    наследования между этими классами, применив [замену наследования
    делегированием](../replace-inheritance-with-delegation.html).

-   Если наследование имеет смысл, нужно избавиться от лишних полей и
    методов в подклассе. Для этого необходимо извлечь из родительского
    класса все поля и методы, которые нужны подклассу, в новый
    суперкласс, и сделать оба класса его наследниками ([извлечение
    суперкласса](../extract-superclass.html)).

<figure class="image">
<img
src="../../images/refactoring/content/smells/refused-bequest-03ba1b.png?id=2a84293620fa1caf4329fca1f4a44e08"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/refused-bequest-03.png?id=2a84293620fa1caf4329fca1f4a44e08 500w,/images/refactoring/content/smells/refused-bequest-03-1.5x.png?id=9c9b198ba12e2bb4a4e73ea3df19de8b 750w,/images/refactoring/content/smells/refused-bequest-03-2x.png?id=6990ba0083e3de07881bd551928e3a79 1000w"
sizes="(max-width: 720px) 100vw, 500px" loading="lazy" width="500"
height="300" />
</figure>

### Выигрыш

-   Улучшает понимание и организацию кода. Теперь вы не будете тратить
    время на догадки о том, почему класс `Стул` унаследован от класса
    `Животное` [(несмотря на то, что оба имеют четыре ноги)]{.strike}.

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

[Альтернативные классы с разными интерфейсами []{.fa
.fa-arrow-right}](alternative-classes-with-different-interfaces.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Временное поле](temporary-field.html){.btn
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
