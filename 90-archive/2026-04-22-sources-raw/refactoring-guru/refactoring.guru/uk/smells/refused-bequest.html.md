:::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::: {.main-content-container .center-content-container}
:::::::::: {.page .text}
::: breadcrumb
[](../../index.html){.home} / [Рефакторинг](../refactoring.html){.type}
/ [Запахи коду](../refactoring/smells.html){.type} / [Порушники
об\'єктно-орієнтованого
дизайну](../refactoring/smells/oo-abusers.html){.type}
:::

# Відмова від спадку {#відмова-від-спадку .title}

::: aka
Також відомий як: [Refused Bequest]{style="display:inline-block"}
:::

### Симптоми і ознаки

Якщо підклас використовує лише малу частину успадкованих методів і
властивостей суперкласа, це є ознакою неправильної ієрархії. При цьому
зайві методи можуть просто не використовуватися або бути перевизначеними
і викидати виключення.

<figure class="image">
<img
src="../../images/refactoring/content/smells/refused-bequest-014f59.png?id=7a1d79e75a3836c22ec865d72c98664e"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/refused-bequest-01.png?id=7a1d79e75a3836c22ec865d72c98664e 500w,/images/refactoring/content/smells/refused-bequest-01-1.5x.png?id=90bf2279c14d74bc0e63b9bcc3b69808 750w,/images/refactoring/content/smells/refused-bequest-01-2x.png?id=d2e31b7b9fa3326118817b8e0c65e435 1000w"
sizes="(max-width: 720px) 100vw, 500px" data-fetchpriority="high"
width="500" height="300" />
</figure>

### Причини появи

Хтось створив наслідування між класами тільки з мотивів повторного
використання коду, що знаходиться в суперкласі. При цьому суперклас і
підклас можуть бути абсолютно різними сутностями.

<figure class="image">
<img
src="../../images/refactoring/content/smells/refused-bequest-02c0d0.png?id=f9b0affd4bbf6fec22c05783fc75562e"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/refused-bequest-02.png?id=f9b0affd4bbf6fec22c05783fc75562e 500w,/images/refactoring/content/smells/refused-bequest-02-1.5x.png?id=28b34af7a9319c2d15c32e4a5890c835 750w,/images/refactoring/content/smells/refused-bequest-02-2x.png?id=33b42b7d51bca13f27e4933d24f82751 1000w"
sizes="(max-width: 720px) 100vw, 500px" loading="lazy" width="500"
height="300" />
</figure>

### Лікування

-   Якщо наслідування не має сенсу, і підклас насправді не є
    представником суперкласу, слід позбутися від відношення наслідування
    між цими класами, застосувавши [заміну наслідування
    делегуванням](../replace-inheritance-with-delegation.html).

-   Якщо наслідування має сенс, треба позбутися від зайвих полів і
    методів в підкласі. Для цього необхідно витягнути з батьківського
    класу всі поля і методи, які потрібні підкласу, в новий суперклас, і
    зробити обидва класи його спадкоємцями ([відокремлення
    суперкласу](../extract-superclass.html)).

<figure class="image">
<img
src="../../images/refactoring/content/smells/refused-bequest-03ba1b.png?id=2a84293620fa1caf4329fca1f4a44e08"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/refused-bequest-03.png?id=2a84293620fa1caf4329fca1f4a44e08 500w,/images/refactoring/content/smells/refused-bequest-03-1.5x.png?id=9c9b198ba12e2bb4a4e73ea3df19de8b 750w,/images/refactoring/content/smells/refused-bequest-03-2x.png?id=6990ba0083e3de07881bd551928e3a79 1000w"
sizes="(max-width: 720px) 100vw, 500px" loading="lazy" width="500"
height="300" />
</figure>

### Виграш

-   Покращує розуміння і організацію коду. Тепер ви не витрачатимете час
    на припущення, чому саме клас `Стілець` є підкласом класу `Тварина`
    [(незважаючи на те, що в обох є чотири ноги)]{.strike}.

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

[Альтернативні класи з різними інтерфейсами []{.fa
.fa-arrow-right}](alternative-classes-with-different-interfaces.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Повернутись назад

[[]{.fa .fa-arrow-left} Тимчасове поле](temporary-field.html){.btn
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
