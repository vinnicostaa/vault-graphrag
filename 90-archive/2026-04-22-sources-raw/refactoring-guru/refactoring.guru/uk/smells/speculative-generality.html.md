:::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::: {.main-content-container .center-content-container}
:::::::::: {.page .text}
::: breadcrumb
[](../../index.html){.home} / [Рефакторинг](../refactoring.html){.type}
/ [Запахи коду](../refactoring/smells.html){.type} / [Забрюднювачі
коду](../refactoring/smells/dispensables.html){.type}
:::

# Теоретична спільність {#теоретична-спільність .title}

::: aka
Також відомий як: [Speculative Generality]{style="display:inline-block"}
:::

### Симптоми і ознаки

Клас, метод, поле або параметр не використовуються.

<figure class="image">
<img
src="../../images/refactoring/content/smells/speculative-generality-01d899.png?id=c804fce5c6c5c34b4d9389fcb2aa60aa"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/speculative-generality-01.png?id=c804fce5c6c5c34b4d9389fcb2aa60aa 500w,/images/refactoring/content/smells/speculative-generality-01-1.5x.png?id=475f4713f626920ce866b378e350abc8 750w,/images/refactoring/content/smells/speculative-generality-01-2x.png?id=7875dde405dfa433685fa9040da39b5e 1000w"
sizes="(max-width: 720px) 100vw, 500px" data-fetchpriority="high"
width="500" height="300" />
</figure>

### Причини появи

Іноді код створюється «про запас», щоби підтримувати якийсь можливий
майбутній функціонал, який так і не реалізується. В результаті цей код
стає важче розуміти і супроводжувати.

### Лікування

-   Для видалення незадіяних абстрактних класів використайте [згортання
    ієрархії](../collapse-hierarchy.html).

-   Зайве делегування функціональності іншому класу може бути видалене
    за допомогою [вбудовування класу](../inline-class.html).

-   Від невживаних методів можна позбутися за допомогою [вбудовування
    методу](../inline-method.html).

-   Методи з невживаними параметрами потрібно передати [видаленню
    параметрів](../remove-parameter.html).

-   Зайві поля можна просто видалити.

<figure class="image">
<img
src="../../images/refactoring/content/smells/speculative-generality-02bab3.png?id=e9d0e8a6170b6d0d0be9cca44175fe44"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/speculative-generality-02.png?id=e9d0e8a6170b6d0d0be9cca44175fe44 500w,/images/refactoring/content/smells/speculative-generality-02-1.5x.png?id=f9c73fb909b254c3451fe73b2dde4548 750w,/images/refactoring/content/smells/speculative-generality-02-2x.png?id=017235ad164cb220c99f21f201872d29 1000w"
sizes="(max-width: 720px) 100vw, 500px" loading="lazy" width="500"
height="300" />
</figure>

### Виграш

-   Зменшення розміру коду.

-   Спрощення підтримки.

### Не варто чіпати, якщо\...

-   У випадках, коли ви працюєте над фреймворком, створення
    функціональності, що не буде використовуватись самим фреймворком,
    цілком виправдане. Головне, щоб вона була потрібна користувачам
    фреймворка.

-   Перед видаленням елементів варто впевнитися, чи не використовуються
    вони в юніт-тестах. Таке буває, якщо тести потребують способу
    отримання якоїсь службової інформації класу або здійснення якихось
    спеціальних тестових дій.

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

[Заплутувальники зв\'язками []{.fa
.fa-arrow-right}](../refactoring/smells/couplers.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Повернутись назад

[[]{.fa .fa-arrow-left} Мертвий код](dead-code.html){.btn .btn-default
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
