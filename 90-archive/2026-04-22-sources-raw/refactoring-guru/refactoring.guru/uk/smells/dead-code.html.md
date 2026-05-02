:::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::: {.main-content-container .center-content-container}
:::::::::: {.page .text}
::: breadcrumb
[](../../index.html){.home} / [Рефакторинг](../refactoring.html){.type}
/ [Запахи коду](../refactoring/smells.html){.type} / [Забрюднювачі
коду](../refactoring/smells/dispensables.html){.type}
:::

# Мертвий код {#мертвий-код .title}

::: aka
Також відомий як: [Dead Code]{style="display:inline-block"}
:::

### Симптоми і ознаки

Змінна, параметр, поле, метод або клас більше не використовуються
(найчастіше тому, що застаріли).

<figure class="image">
<img
src="../../images/refactoring/content/smells/dead-code-01fd46.png?id=418685bee5de933c472c48efcb5b67a0"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/dead-code-01.png?id=418685bee5de933c472c48efcb5b67a0 500w,/images/refactoring/content/smells/dead-code-01-1.5x.png?id=264efc49269071a3781dd96a1994c249 750w,/images/refactoring/content/smells/dead-code-01-2x.png?id=d9981d853e7e855cf0454fc32aab85a6 1000w"
sizes="(max-width: 720px) 100vw, 500px" data-fetchpriority="high"
width="500" height="300" />
</figure>

### Причини появи

Коли вимоги до програмного продукту змінилися, або були внесені якісь
коригування, але чистка старого коду не відбулася.

Мертвий код можна знайти в складному умовному коді, коли одна з гілок
ніколи не може бути виконана (через наявність помилки або іншого збігу
обставин).

### Лікування

Краще за все мертвий код виявляється за допомогою якісного [середовища
розробки
(IDE)](https://en.wikipedia.org/wiki/Integrated%20development%20environment).

-   Видаліть невживаний код і зайві файли.

-   Непотрібних класів можна позбутися за допомогою [вбудовування
    класу](../inline-class.html). Якщо в такого класа є підкласи чи
    суперклас, то підійте [згортання
    ієрархії](../collapse-hierarchy.html).

-   Для видалення непотрібних параметрів використайте [видалення
    параметру](../remove-parameter.html).

<figure class="image">
<img
src="../../images/refactoring/content/smells/dead-code-025e37.png?id=b368f23b7cc88340933b761cf2ad1954"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/dead-code-02.png?id=b368f23b7cc88340933b761cf2ad1954 500w,/images/refactoring/content/smells/dead-code-02-1.5x.png?id=c84149b6f42ea2e4ab59b653276627c6 750w,/images/refactoring/content/smells/dead-code-02-2x.png?id=bf78ebf643d890b41b60058d8e67d845 1000w"
sizes="(max-width: 720px) 100vw, 500px" loading="lazy" width="500"
height="300" />
</figure>

### Виграш

-   Зменшує розмір коду.

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

[Теоретична спільність []{.fa
.fa-arrow-right}](speculative-generality.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Повернутись назад

[[]{.fa .fa-arrow-left} Клас даних](data-class.html){.btn .btn-default
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
