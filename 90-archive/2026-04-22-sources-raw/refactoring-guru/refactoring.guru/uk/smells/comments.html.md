:::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::: {.main-content-container .center-content-container}
:::::::::: {.page .text}
::: breadcrumb
[](../../index.html){.home} / [Рефакторинг](../refactoring.html){.type}
/ [Запахи коду](../refactoring/smells.html){.type} / [Забрюднювачі
коду](../refactoring/smells/dispensables.html){.type}
:::

# Коментарі {#коментарі .title}

::: aka
Також відомий як: [Comments]{style="display:inline-block"}
:::

### Симптоми і ознаки

Метод містить безліч пояснювальних коментарів.

<figure class="image">
<img
src="../../images/refactoring/content/smells/comments-01cb86.png?id=584958123f3b902e0ad0895d879509b9"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/comments-01.png?id=584958123f3b902e0ad0895d879509b9 500w,/images/refactoring/content/smells/comments-01-1.5x.png?id=c6c70d9f1f12d481d8cabfc726a7fbf7 750w,/images/refactoring/content/smells/comments-01-2x.png?id=15fe22a84b974b19a752ad169ae999ae 1000w"
sizes="(max-width: 720px) 100vw, 500px" data-fetchpriority="high"
width="500" height="300" />
</figure>

### Причини появи

Здебільшого, автор керується найдобрішими намірами при створенні
коментарів. Він сам розуміє, що його код недостатньо очевидний і
зрозумілий. У таких випадках коментарі грають роль «дезодоранту», тобто
намагаються заглушити «поганий запах» недостатньо опрацьованого коду.

> Найкращий коментар --- це хороша назва методу або класу.

Якщо ви відчуваєте, що фрагмент коду буде незрозумілим без коментаря,
спробуйте змінити структуру коду таким чином, щоби будь-які коментарі
стали зайвими.

### Лікування

-   Якщо коментар призначений для того, щоби пояснити складний вираз,
    можливо, цей вираз краще розбити на більш зрозумілі підвирази за
    допомогою [відокремлення змінної](../extract-variable.html).

-   Якщо коментар пояснює цілий блок коду, можливо, цей блок можна
    витягнути в окремий метод за допомогою [відокремлення
    методу](../extract-method.html). Назву нового методу вам, скоріш за
    все, підкаже сам коментар.

-   Якщо метод вже виділений, але для пояснення його дії, як і раніше,
    потрібен коментар, дайте методу нову назву, що не вимагає коментаря.
    Використайте для цього [перейменування
    методу](../rename-method.html).

-   Якщо потрібно описати якість правила, що стосуються корректної
    роботи метода, спробуйте рефакторинг [введення
    твердження](../introduce-assertion.html).

### Виграш

-   Код стає очевиднішим і зрозумілішим.

<figure class="image">
<img
src="../../images/refactoring/content/smells/comments-02b499.png?id=266f82bb7081957d409ae690c2c66483"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/comments-02.png?id=266f82bb7081957d409ae690c2c66483 500w,/images/refactoring/content/smells/comments-02-1.5x.png?id=6451f6937bda39e8f101367df8dec883 750w,/images/refactoring/content/smells/comments-02-2x.png?id=57a83d2b705347aa0d0a6d197a1f9d3c 1000w"
sizes="(max-width: 720px) 100vw, 500px" loading="lazy" width="500"
height="300" />
</figure>

### Не варто чіпати, якщо\...

Іноді коментарі бувають корисними:

-   Ті, що пояснюють **чому** щось виконується саме таким чином.

-   Ті, що пояснюють складні алгоритми (коли всі інші засоби спрощення
    алгоритму вже були випробувані).

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

[Дублювання коду []{.fa .fa-arrow-right}](duplicate-code.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Повернутись назад

[[]{.fa .fa-arrow-left} Забрюднювачі
коду](../refactoring/smells/dispensables.html){.btn .btn-default
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
