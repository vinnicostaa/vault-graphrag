:::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::: {.main-content-container .center-content-container}
:::::::::: {.page .text}
::: breadcrumb
[](../../index.html){.home} / [Рефакторинг](../refactoring.html){.type}
/ [Запахи кода](../refactoring/smells.html){.type} / [Утяжелители
изменений](../refactoring/smells/change-preventers.html){.type}
:::

# Расходящиеся модификации {#расходящиеся-модификации .title}

::: aka
Также известен как: [Divergent Change]{style="display:inline-block"}
:::

> «Расходящиеся модификации» похожи на [Стрельбу
> дробью](shotgun-surgery.html), но на самом деле являются ее
> противоположностью. «Расходящиеся модификации» имеют место, когда есть
> один класс, в котором производится много разных изменений, а «Стрельба
> дробью» --- это одно изменение, затрагивающее одновременно много
> классов.

### Симптомы и признаки

При внесении изменений в класс приходится изменять большое число
различных методов. Например, для добавления нового вида товара вам нужно
изменить методы поиска, отображения и заказа товаров.

<figure class="image">
<img
src="../../images/refactoring/content/smells/divergent-change-010f98.png?id=d62e68e1778d67bf82ff74064c24de33"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/divergent-change-01.png?id=d62e68e1778d67bf82ff74064c24de33 500w,/images/refactoring/content/smells/divergent-change-01-1.5x.png?id=166d6b9318e5bcaa76ea673e0346d58e 750w,/images/refactoring/content/smells/divergent-change-01-2x.png?id=1c7d20737703941d1e3f7ad85e180578 1000w"
sizes="(max-width: 720px) 100vw, 500px" data-fetchpriority="high"
width="500" height="300" />
</figure>

### Причины появления

Часто появление расходящихся модификаций является следствием плохой
структурированности программы или программирования методом
копирования-вставки.

### Лечение

-   Разделите поведения класса, используя [извлечение
    класса](../extract-class.html).

-   Если различные классы имеют одно и то же поведение, возможно,
    следует объединить эти классы, используя наследование ([извлечение
    суперкласса](../extract-superclass.html) и [извлечение
    подкласса](../extract-subclass.html)).

<figure class="image">
<img
src="../../images/refactoring/content/smells/divergent-change-02aacb.png?id=21b6fd7cba36f123c09497cb8f5a5625"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/divergent-change-02.png?id=21b6fd7cba36f123c09497cb8f5a5625 500w,/images/refactoring/content/smells/divergent-change-02-1.5x.png?id=1c18d8c9f8a40887e60041d4d5027532 750w,/images/refactoring/content/smells/divergent-change-02-2x.png?id=581f6218d8a2393ece88419ad60831da 1000w"
sizes="(max-width: 720px) 100vw, 500px" loading="lazy" width="500"
height="300" />
</figure>

### Выигрыш

-   Улучшает организацию кода.

-   Уменьшает дублирование кода.

-   Упрощает поддержку.

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

[Стрельба дробью []{.fa .fa-arrow-right}](shotgun-surgery.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Утяжелители
изменений](../refactoring/smells/change-preventers.html){.btn
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
