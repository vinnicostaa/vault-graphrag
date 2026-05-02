:::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::: {.main-content-container .center-content-container}
:::::::::: {.page .text}
::: breadcrumb
[](../../index.html){.home} / [Рефакторинг](../refactoring.html){.type}
/ [Запахи кода](../refactoring/smells.html){.type} / [Утяжелители
изменений](../refactoring/smells/change-preventers.html){.type}
:::

# Стрельба дробью {#стрельба-дробью .title}

::: aka
Также известен как: [Shotgun Surgery]{style="display:inline-block"}
:::

> «Стрельба дробью» похожа на [Расходящиеся
> модификации](divergent-change.html), но является противоположностью
> этого запаха. «Расходящиеся модификации» имеют место, когда есть один
> класс, в котором производится много различных изменений, а «Стрельба
> дробью» --- это одно изменение, затрагивающее много классов.

### Симптомы и признаки

При выполнении любых модификаций приходится вносить множество мелких
изменений в большое число классов.

<figure class="image">
<img
src="../../images/refactoring/content/smells/shotgun-surgery-01e5e8.png?id=9cc1117a6d787364788e152a3adb6a53"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/shotgun-surgery-01.png?id=9cc1117a6d787364788e152a3adb6a53 500w,/images/refactoring/content/smells/shotgun-surgery-01-1.5x.png?id=38debad74fb67588357f07c149e76569 750w,/images/refactoring/content/smells/shotgun-surgery-01-2x.png?id=01431b43fcaee83fade53530a3dd91ab 1000w"
sizes="(max-width: 720px) 100vw, 500px" data-fetchpriority="high"
width="500" height="300" />
</figure>

### Причины появления

Одна обязанность была разделена среди множества классов. Это может
случиться после фанатичного исправления [Расходящихся
модификаций](divergent-change.html).

<figure class="image">
<img
src="../../images/refactoring/content/smells/shotgun-surgery-02b284.png?id=48f8a4a0f17d112e02ae73bacaed43fa"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/shotgun-surgery-02.png?id=48f8a4a0f17d112e02ae73bacaed43fa 500w,/images/refactoring/content/smells/shotgun-surgery-02-1.5x.png?id=fce8ce3dacc4177bbc18af12a39da6cd 750w,/images/refactoring/content/smells/shotgun-surgery-02-2x.png?id=a35426ca3f6e64857e66b2fdeb395870 1000w"
sizes="(max-width: 720px) 100vw, 500px" loading="lazy" width="500"
height="300" />
</figure>

### Лечение

-   Вынести все изменения в один класс позволят [перемещение
    метода](../move-method.html) и [перемещение
    поля](../move-field.html). Если для выполнения этого действия нет
    подходящего класса, то следует предварительно создать новый.

-   Если после вынесения кода в один класс в оригинальных классах мало
    что осталось, следует попытаться от них избавиться, воспользовавшись
    [встраиванием класса](../inline-class.html).

<figure class="image">
<img
src="../../images/refactoring/content/smells/shotgun-surgery-030760.png?id=cf013f14eb5cde98bd48595a1c9836a9"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/shotgun-surgery-03.png?id=cf013f14eb5cde98bd48595a1c9836a9 500w,/images/refactoring/content/smells/shotgun-surgery-03-1.5x.png?id=c29cddda01b23e0dd21d8b12f5937f84 750w,/images/refactoring/content/smells/shotgun-surgery-03-2x.png?id=259b00413f0f8be143ead703c74b7e38 1000w"
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

[Параллельные иерархии наследования []{.fa
.fa-arrow-right}](parallel-inheritance-hierarchies.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Расходящиеся
модификации](divergent-change.html){.btn .btn-default rel="prev"}
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
