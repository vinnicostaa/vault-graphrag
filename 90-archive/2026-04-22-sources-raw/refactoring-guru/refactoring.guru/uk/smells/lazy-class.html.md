:::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::: {.main-content-container .center-content-container}
:::::::::: {.page .text}
::: breadcrumb
[](../../index.html){.home} / [Рефакторинг](../refactoring.html){.type}
/ [Запахи коду](../refactoring/smells.html){.type} / [Забрюднювачі
коду](../refactoring/smells/dispensables.html){.type}
:::

# Ледачий клас {#ледачий-клас .title}

::: aka
Також відомий як: [Lazy Class]{style="display:inline-block"}
:::

### Симптоми і ознаки

На розуміння і підтримку класів завжди потрібно витрачати час і гроші. А
тому, якщо клас не робить досить багато, щоби приділяти йому достатньо
уваги, він має бути знищений.

<figure class="image">
<img
src="../../images/refactoring/content/smells/lazy-class-0148c1.png?id=efec5911dfaaa3ba69d3eb4dab03fd3c"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/lazy-class-01.png?id=efec5911dfaaa3ba69d3eb4dab03fd3c 500w,/images/refactoring/content/smells/lazy-class-01-1.5x.png?id=4b9cf29aa6991dc8050debc5494a2a92 750w,/images/refactoring/content/smells/lazy-class-01-2x.png?id=3b5b07bc60eb98c883fc68c5e1a05aed 1000w"
sizes="(max-width: 720px) 100vw, 500px" data-fetchpriority="high"
width="500" height="300" />
</figure>

### Причини появи

Це може статися, якщо клас був задуманий як повнофункціональний, але в
результаті рефакторингу зменшився до непристойних розмірів.

Або клас створювався з розрахунку на деякі майбутні розробки, до яких
руки так і не дійшли.

### Лікування

-   Для компонентів, які мають дуже мало корисних елементів, необхідно
    провести [вбудовування класу](../inline-class.html).

-   За наявності підкласів з недостатніми функціями спробуйте [згортання
    ієрархії](../collapse-hierarchy.html).

<figure class="image">
<img
src="../../images/refactoring/content/smells/lazy-class-02adb6.png?id=393302f2bd27ba0197660caea274ae23"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/lazy-class-02.png?id=393302f2bd27ba0197660caea274ae23 500w,/images/refactoring/content/smells/lazy-class-02-1.5x.png?id=72b0880a8673312f80bd10755cdeae74 750w,/images/refactoring/content/smells/lazy-class-02-2x.png?id=d46dd63f159b40aa266ccbdbefb319bd 1000w"
sizes="(max-width: 720px) 100vw, 500px" loading="lazy" width="500"
height="300" />
</figure>

### Виграш

-   Зменшення розміру коду.

-   Спрощення підтримки.

### Не варто чіпати, якщо\...

-   Іноді *Ледачий клас* буває створений для того, щоб явно окреслити
    якісь наміри. В цьому випадку варто дотримуватися балансу
    зрозумілості коду і його простоти.

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

[Клас даних []{.fa .fa-arrow-right}](data-class.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Повернутись назад

[[]{.fa .fa-arrow-left} Дублювання коду](duplicate-code.html){.btn
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
