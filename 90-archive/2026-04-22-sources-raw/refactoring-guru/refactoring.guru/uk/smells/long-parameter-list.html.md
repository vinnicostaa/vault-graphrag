:::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::: {.main-content-container .center-content-container}
:::::::::: {.page .text}
::: breadcrumb
[](../../index.html){.home} / [Рефакторинг](../refactoring.html){.type}
/ [Запахи коду](../refactoring/smells.html){.type} /
[Роздувальщики](../refactoring/smells/bloaters.html){.type}
:::

# Довгий список параметрів {#довгий-список-параметрів .title}

::: aka
Також відомий як: [Long Parameter List]{style="display:inline-block"}
:::

### Симптоми і ознаки

Кількість параметрів методу більше трьох-чотирьох.

<figure class="image">
<img
src="../../images/refactoring/content/smells/long-parameter-list-015e37.png?id=06fad4adaf485cfaa569e66c20f268eb"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/long-parameter-list-01.png?id=06fad4adaf485cfaa569e66c20f268eb 500w,/images/refactoring/content/smells/long-parameter-list-01-1.5x.png?id=7e78b50f369819e051b3aa450a183659 750w,/images/refactoring/content/smells/long-parameter-list-01-2x.png?id=d964f68180e89b6312726c7a5719e35d 1000w"
sizes="(max-width: 720px) 100vw, 500px" data-fetchpriority="high"
width="500" height="300" />
</figure>

### Причини появи

Довгий список параметрів може з'явитися після об'єднання декількох
варіантів алгоритмів в одному методі. В цьому випадку може бути
створений довгий список параметрів, контролюючих те, яка з варіацій буде
виконана і як.

Поява довгого списку параметрів також може бути пов'язана із спробою
програміста зменшити зв'язаність між класами. Наприклад, код створення
конкретних об'єктів, потрібних в методі, перенесли з самого методу в код
виклику цього методу, а створені об'єкти вирішили передавати в метод як
параметри. Таким чином, оригінальний клас перестав знати про зв'язки між
об'єктами, і зв'язність зменшилася.

Але якщо з'являється потреба в декількох таких об'єктах, для кожного з
них знадобиться свій власний параметр, що призводить до розростання
списку параметрів.

У довгих списках параметрів важко орієнтуватися, вони стають
суперечливими і складними у використанні. Замість довгого списку
параметрів метод може використовувати дані свого власного об'єкта. Якщо
всіх необхідних даних в поточному об'єкті немає, в якості параметра
методу можна передати інший об'єкт, який отримає відсутні дані.

### Лікування

-   Якщо дані, які передаються в метод, можна отримати шляхом виклику
    методу іншого об'єкта, застосовуємо [заміну параметра викликом
    методу](../replace-parameter-with-method-call.html). Цей об'єкт може
    бути розташованим у полі власного класу або переданий як параметр
    методу.

-   Замість того щоби передавати групу даних, отриманих з іншого об'єкта
    в якості параметрів, можна передати в метод сам об'єкт,
    використовуючи [передачу всього
    об'єкта](../preserve-whole-object.html).

-   Якщо є декілька незв'язаних елементів даних, іноді їх можна
    об'єднати в один об'єкт-параметр, застосувавши для цього [заміну
    параметрів об'єктом](../introduce-parameter-object.html).

<figure class="image">
<img
src="../../images/refactoring/content/smells/long-parameter-list-020f28.png?id=7571291fcaea939ed137400cbe0f3c02"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/long-parameter-list-02.png?id=7571291fcaea939ed137400cbe0f3c02 500w,/images/refactoring/content/smells/long-parameter-list-02-1.5x.png?id=f933042e101fee9bb5424c308c62daeb 750w,/images/refactoring/content/smells/long-parameter-list-02-2x.png?id=90514c8d76f4eae7b76439778b82c778 1000w"
sizes="(max-width: 720px) 100vw, 500px" loading="lazy" width="500"
height="300" />
</figure>

### Виграш

-   Підвищує читабельність коду, зменшує його розмір.

-   В процесі рефакторингу ви зможете виявити дублювання коду, яке
    раніше було непомітним.

### Не варто чіпати, якщо\...

-   Не варто позбавлятися від параметрів, якщо при цьому з'являється
    небажана зв'язаність між класами.

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

[Групи даних []{.fa .fa-arrow-right}](data-clumps.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Повернутись назад

[[]{.fa .fa-arrow-left} Одержимість елементарними
типами](primitive-obsession.html){.btn .btn-default rel="prev"}
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
