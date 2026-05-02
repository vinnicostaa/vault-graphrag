:::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::: {.main-content-container .center-content-container}
:::::::::: {.page .text}
::: breadcrumb
[](../../index.html){.home} / [Рефакторинг](../refactoring.html){.type}
/ [Запахи коду](../refactoring/smells.html){.type} / [Порушники
об\'єктно-орієнтованого
дизайну](../refactoring/smells/oo-abusers.html){.type}
:::

# Тимчасове поле {#тимчасове-поле .title}

::: aka
Також відомий як: [Temporary Field]{style="display:inline-block"}
:::

### Симптоми і ознаки

Тимчасові поля --- це поля, які потрібні об'єкту лише час від часу.
Тільки тоді вони заповнюються якимись значеннями, залишаючись порожніми
решту часу.

<figure class="image">
<img
src="../../images/refactoring/content/smells/temporary-field-01e767.png?id=5e30a8144171693ee4894091762b9742"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/temporary-field-01.png?id=5e30a8144171693ee4894091762b9742 500w,/images/refactoring/content/smells/temporary-field-01-1.5x.png?id=5079cd815f19d0a8938bc64c40f763ab 750w,/images/refactoring/content/smells/temporary-field-01-2x.png?id=1cf05ea67f1e3bd1c1f634af7408f67c 1000w"
sizes="(max-width: 720px) 100vw, 500px" data-fetchpriority="high"
width="500" height="300" />
</figure>

### Причини появи

Найчастіше тимчасові поля створюються для використання в алгоритмі, який
вимагає великого числа вхідних даних. Так, замість створення великого
числа параметрів в такому методі програміст вирішує створити для цих
даних поля в класі. Ці поля використовуються тільки в цьому алгоритмі, а
решту часу простоюють.

Такий код дуже важко зрозуміти. Ви очікуєте побачити дані в полях
об'єкта, а вони чомусь порожні майже весь час.

<figure class="image">
<img
src="../../images/refactoring/content/smells/temporary-field-02bc27.png?id=2cf98e02ffb0c1d36a98d48ba7bc5c45"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/temporary-field-02.png?id=2cf98e02ffb0c1d36a98d48ba7bc5c45 500w,/images/refactoring/content/smells/temporary-field-02-1.5x.png?id=d29dd17bc354ec725ad3092a67cecfd8 750w,/images/refactoring/content/smells/temporary-field-02-2x.png?id=6c8e4384d9029cd3ec84cd330d5871eb 1000w"
sizes="(max-width: 720px) 100vw, 500px" loading="lazy" width="500"
height="300" />
</figure>

### Лікування

-   Тимчасові поля і весь код, що працює з ними, можна перемістити в
    свій власний клас за допомогою [відокремлення
    класу](../extract-class.html). По суті, ви таким чином створюєте
    [об'єкт-метод](../replace-method-with-method-object.html).

-   [Введіть Null-об'єкт](../introduce-null-object.html) і вбудуйте його
    замість коду перевірки наявності значень в тимчасових полях.

<figure class="image">
<img
src="../../images/refactoring/content/smells/temporary-field-032ad0.png?id=cf0e1c1e2a19745d23ca9e1d917d45cc"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/temporary-field-03.png?id=cf0e1c1e2a19745d23ca9e1d917d45cc 500w,/images/refactoring/content/smells/temporary-field-03-1.5x.png?id=09f1df24d41f5f39b6b298b8a71701b7 750w,/images/refactoring/content/smells/temporary-field-03-2x.png?id=c633fd664958307bf296b09de72959dd 1000w"
sizes="(max-width: 720px) 100vw, 500px" loading="lazy" width="500"
height="300" />
</figure>

### Виграш

-   Покращує зрозумілість і організацію коду.

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

[Відмова від спадку []{.fa .fa-arrow-right}](refused-bequest.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Повернутись назад

[[]{.fa .fa-arrow-left} Оператори switch](switch-statements.html){.btn
.btn-default rel="prev"}
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
