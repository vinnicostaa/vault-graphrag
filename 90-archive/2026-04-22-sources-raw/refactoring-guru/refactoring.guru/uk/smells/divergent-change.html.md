:::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::: {.main-content-container .center-content-container}
:::::::::: {.page .text}
::: breadcrumb
[](../../index.html){.home} / [Рефакторинг](../refactoring.html){.type}
/ [Запахи коду](../refactoring/smells.html){.type} / [Ускладнювачі
змін](../refactoring/smells/change-preventers.html){.type}
:::

# Розбіжні модифікації {#розбіжні-модифікації .title}

::: aka
Також відомий як: [Divergent Change]{style="display:inline-block"}
:::

> «Розбіжні модифікації» схожі на [Стрільбу
> дробом](shotgun-surgery.html), але насправді є протилежністю цього
> запаху. «Розбіжні модифікації» мають місце, коли є один клас, в якому
> робиться багато різних змін, а «Стрільба дробом» --- це одна зміна, що
> зачіпає одночасно багато класів.

### Симптоми і ознаки

При внесенні змін до класу доводиться змінювати багато різноманітних
методів. Наприклад, для додавання нового виду товару вам буде потрібно
змінити методи пошуку, відображення і замовлення товарів.

<figure class="image">
<img
src="../../images/refactoring/content/smells/divergent-change-010f98.png?id=d62e68e1778d67bf82ff74064c24de33"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/divergent-change-01.png?id=d62e68e1778d67bf82ff74064c24de33 500w,/images/refactoring/content/smells/divergent-change-01-1.5x.png?id=166d6b9318e5bcaa76ea673e0346d58e 750w,/images/refactoring/content/smells/divergent-change-01-2x.png?id=1c7d20737703941d1e3f7ad85e180578 1000w"
sizes="(max-width: 720px) 100vw, 500px" data-fetchpriority="high"
width="500" height="300" />
</figure>

### Причини появи

Найчастіше поява розбіжних модифікацій є наслідком поганої
структурованості програми або програмування методом копіювання-вставки.

### Лікування

-   Поведінку класу можна розділити на декілька частин за допомогою
    [відокремлення класу](../extract-class.html).

-   Якщо різні класи мають однакову поведінку, можливо, слід об'єднати
    ці класи, використовуючи наслідування ([відокремлення
    суперкласу](../extract-superclass.html) і [відокремлення
    підкласу](../extract-subclass.html)).

<figure class="image">
<img
src="../../images/refactoring/content/smells/divergent-change-02aacb.png?id=21b6fd7cba36f123c09497cb8f5a5625"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/divergent-change-02.png?id=21b6fd7cba36f123c09497cb8f5a5625 500w,/images/refactoring/content/smells/divergent-change-02-1.5x.png?id=1c18d8c9f8a40887e60041d4d5027532 750w,/images/refactoring/content/smells/divergent-change-02-2x.png?id=581f6218d8a2393ece88419ad60831da 1000w"
sizes="(max-width: 720px) 100vw, 500px" loading="lazy" width="500"
height="300" />
</figure>

### Виграш

-   Покращує організацію коду.

-   Зменшує дублювання коду.

-   Спрощує його підтримку.

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

[Стрільба дробом []{.fa .fa-arrow-right}](shotgun-surgery.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Повернутись назад

[[]{.fa .fa-arrow-left} Ускладнювачі
змін](../refactoring/smells/change-preventers.html){.btn .btn-default
rel="prev"}
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
