:::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::: {.main-content-container .center-content-container}
:::::::::: {.page .text}
::: breadcrumb
[](../../index.html){.home} / [Рефакторинг](../refactoring.html){.type}
/ [Запахи коду](../refactoring/smells.html){.type} / [Інші
запахи](../refactoring/smells/other.html){.type}
:::

# Неповнота бібліотечного класу {#неповнота-бібліотечного-класу .title}

::: aka
Також відомий як: [Incomplete Library
Class]{style="display:inline-block"}
:::

### Симптоми і ознаки

[Бібліотеки](https://uk.wikipedia.org/wiki/Бібліотека_програм) через
деякий час перестають задовольняти вимогам користувачів. Найбільш
природним рішенням в цьому випадку є внесення змін у бібліотеку, але
дуже часто воно виявляється неможливим, оскільки бібліотека закрита для
запису.

<figure class="image">
<img
src="../../images/refactoring/content/smells/incomplete-library-class-0152c5.png?id=ca51f740f7fd39b7de1430b64cae9f8c"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/incomplete-library-class-01.png?id=ca51f740f7fd39b7de1430b64cae9f8c 500w,/images/refactoring/content/smells/incomplete-library-class-01-1.5x.png?id=e0efa4b241a785afba5f0e85672dbe4a 750w,/images/refactoring/content/smells/incomplete-library-class-01-2x.png?id=25c39ccf56423153b7c977c57943af54 1000w"
sizes="(max-width: 720px) 100vw, 500px" data-fetchpriority="high"
width="500" height="300" />
</figure>

### Причини появи

Автор бібліотеки не передбачив можливості, які вам потрібні, або
відмовився їх впроваджувати.

### Лікування

-   Якщо потрібно додати пару методів у бібліотечний клас,
    використовується [впровадження зовнішнього
    методу](../introduce-foreign-method.html).

-   Якщо потрібно серйозно змінити поведінку класу, використовується
    [впровадження локального
    розширення](../introduce-local-extension.html).

### Виграш

-   Зменшує дублювання коду (замість створення своєї бібліотеки з нуля,
    ви продовжуєте працювати з готовою бібліотекою).

<figure class="image">
<img
src="../../images/refactoring/content/smells/incomplete-library-class-02b14e.png?id=05a8d9c631d43a3fb256196f366fd089"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/incomplete-library-class-02.png?id=05a8d9c631d43a3fb256196f366fd089 500w,/images/refactoring/content/smells/incomplete-library-class-02-1.5x.png?id=bb28ce39729bc28b39030ed6b0ff60b0 750w,/images/refactoring/content/smells/incomplete-library-class-02-2x.png?id=cb204d62084939b3d9e6f97d5d3662ee 1000w"
sizes="(max-width: 720px) 100vw, 500px" loading="lazy" width="500"
height="300" />
</figure>

### Не варто чіпати, якщо\...

-   Розширення бібліотеки може стати причиною появи додаткового обсягу
    роботи. Це відбувається в тому випадку, коли зміни в бібліотеці
    пов'язані зі змінами в коді.

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

[Прийоми рефакторингу []{.fa
.fa-arrow-right}](../refactoring/techniques.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Повернутись назад

[[]{.fa .fa-arrow-left} Інші
запахи](../refactoring/smells/other.html){.btn .btn-default rel="prev"}
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
