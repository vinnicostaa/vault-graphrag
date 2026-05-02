:::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::: {.main-content-container .center-content-container}
:::::::::: {.page .text}
::: breadcrumb
[](../../index.html){.home} / [Рефакторинг](../refactoring.html){.type}
/ [Запахи коду](../refactoring/smells.html){.type} / [Ускладнювачі
змін](../refactoring/smells/change-preventers.html){.type}
:::

# Стрільба дробом {#стрільба-дробом .title}

::: aka
Також відомий як: [Shotgun Surgery]{style="display:inline-block"}
:::

> «Стрільба дробом» схожа на [Розбіжні
> модифікації](divergent-change.html), але насправді є протилежністю
> цього запаху. «Розбіжні модифікації» мають місце, коли є один клас, в
> якому робиться багато різних змін, а «Стрільба дробом» --- це одна
> зміна, що зачіпає одночасно багато класів.

### Симптоми і ознаки

При виконанні будь-яких модифікацій доводиться вносити безліч дрібних
змін у купу класів.

<figure class="image">
<img
src="../../images/refactoring/content/smells/shotgun-surgery-01e5e8.png?id=9cc1117a6d787364788e152a3adb6a53"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/shotgun-surgery-01.png?id=9cc1117a6d787364788e152a3adb6a53 500w,/images/refactoring/content/smells/shotgun-surgery-01-1.5x.png?id=38debad74fb67588357f07c149e76569 750w,/images/refactoring/content/smells/shotgun-surgery-01-2x.png?id=01431b43fcaee83fade53530a3dd91ab 1000w"
sizes="(max-width: 720px) 100vw, 500px" data-fetchpriority="high"
width="500" height="300" />
</figure>

### Причини появи

Один обов'язок був розділений серед безлічі класів. Це може статися
після фанатичного виправлення [Розбіжних
модифікацій](divergent-change.html).

<figure class="image">
<img
src="../../images/refactoring/content/smells/shotgun-surgery-02b284.png?id=48f8a4a0f17d112e02ae73bacaed43fa"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/shotgun-surgery-02.png?id=48f8a4a0f17d112e02ae73bacaed43fa 500w,/images/refactoring/content/smells/shotgun-surgery-02-1.5x.png?id=fce8ce3dacc4177bbc18af12a39da6cd 750w,/images/refactoring/content/smells/shotgun-surgery-02-2x.png?id=a35426ca3f6e64857e66b2fdeb395870 1000w"
sizes="(max-width: 720px) 100vw, 500px" loading="lazy" width="500"
height="300" />
</figure>

### Лікування

-   Винести всі зміни в один клас дозволять [переміщення
    методу](../move-method.html) і [переміщення
    поля](../move-field.html). Якщо для виконання цієї дії немає
    відповідного класу, то слід заздалегідь створити новий.

-   Якщо після винесення коду в один клас в оригінальних класах мало що
    залишилося, слід спробувати від них позбутися, скориставшись
    [вбудовуванням класу](../inline-class.html).

<figure class="image">
<img
src="../../images/refactoring/content/smells/shotgun-surgery-030760.png?id=cf013f14eb5cde98bd48595a1c9836a9"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/shotgun-surgery-03.png?id=cf013f14eb5cde98bd48595a1c9836a9 500w,/images/refactoring/content/smells/shotgun-surgery-03-1.5x.png?id=c29cddda01b23e0dd21d8b12f5937f84 750w,/images/refactoring/content/smells/shotgun-surgery-03-2x.png?id=259b00413f0f8be143ead703c74b7e38 1000w"
sizes="(max-width: 720px) 100vw, 500px" loading="lazy" width="500"
height="300" />
</figure>

### Виграш

-   Покращує організацію коду.

-   Зменшує дублювання коду.

-   Спрощує підтримку.

::::: {.prom .banner-content .banner .banner-striped .banner-with-image data-id="Ref: Part of the ebook" creative-id="animated-uk" position="content_bottom"}
::: banner-image
Your browser does not support HTML video.
:::

::: banner-text
### Замучились читати? {#замучились-читати .title}

Збігайте за подушкою, в нас тут контенту на [7 годин]{.blue} читання.

Або спробуйте наш новий інтерактивний курс. Він набагато цікавіший за
банальний тест.

[ Дізнатися більше...](../refactoring/course.html){.btn .btn-secondary}
:::
:::::

::: next
#### Читаємо далі

[Паралельні ієрархії наслідування []{.fa
.fa-arrow-right}](parallel-inheritance-hierarchies.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Повернутись назад

[[]{.fa .fa-arrow-left} Розбіжні
модифікації](divergent-change.html){.btn .btn-default rel="prev"}
:::
::::::::::

:::::: {.prom .banner-sidebar .banner-removable .banner-removable-refactoring data-id="Ref: Sidebar" creative-id="standard-sidebar-uk" position="sidebar"}
::::: ban
::: {.image .product-image}
[Знижки!]{.banner-discount}
[![](../../images/refactoring/course/course-cover-uk4341.jpg?id=e53397ef67078e742fb132da37ca7cc8){width="300"
height="300" loading="lazy"}](../refactoring/course.html)
:::

::: {.banner-text .banner-text-uk}
Цей запах коду є частиною нашого інтерактивного **онлайн курсу з
рефакторингу**.

[ Подробиці...](../refactoring/course.html){.btn .btn-secondary
.btn-block}
:::
:::::
::::::
:::::::::::::::
::::::::::::::::
