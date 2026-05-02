:::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::: {.main-content-container .center-content-container}
:::::::::: {.page .text}
::: breadcrumb
[](../../index.html){.home} / [Рефакторинг](../refactoring.html){.type}
/ [Запахи коду](../refactoring/smells.html){.type} /
[Роздувальщики](../refactoring/smells/bloaters.html){.type}
:::

# Занадто великий клас {#занадто-великий-клас .title}

::: aka
Також відомий як: [Large Class]{style="display:inline-block"}
:::

### Симптоми і ознаки

В класі безліч полів/методів/рядків коду.

<figure class="image">
<img
src="../../images/refactoring/content/smells/large-class-01eaa7.png?id=acac82f25cc90aaa413c2daefebf0e4b"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/large-class-01.png?id=acac82f25cc90aaa413c2daefebf0e4b 500w,/images/refactoring/content/smells/large-class-01-1.5x.png?id=ba2cf337d9a765204a1b313169f65cf8 750w,/images/refactoring/content/smells/large-class-01-2x.png?id=44aea94399b8bd6398a01b46b5bc7f29 1000w"
sizes="(max-width: 720px) 100vw, 500px" data-fetchpriority="high"
width="500" height="300" />
</figure>

### Причини появи

Як правило, спочатку класи не бувають занадто великими. Але з часом у
зв'язку з розвитком програми багато хто з них «роздувається».

Як і у випадку з довгими методами, найчастіше програмістові ментально
простіше додати фічу в існуючий клас, ніж створювати новий клас для цієї
фічі.

<figure class="image">
<img
src="../../images/refactoring/content/smells/large-class-027a0b.png?id=973b37334ae57489945a88b9327f81e3"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/large-class-02.png?id=973b37334ae57489945a88b9327f81e3 500w,/images/refactoring/content/smells/large-class-02-1.5x.png?id=4b4846ad906c16339481236196214bcb 750w,/images/refactoring/content/smells/large-class-02-2x.png?id=f51627abdfb96fad29cb114d00795fec 1000w"
sizes="(max-width: 720px) 100vw, 500px" loading="lazy" width="500"
height="300" />
</figure>

### Лікування

Коли клас реалізує занадто об'ємний функціонал, варто подумати про його
розділення:

-   [Відокремлення класу](../extract-class.html) допоможе, якщо частина
    поведінки великого класу може бути виділена в свій власний
    компонент.

-   [Відокремлення підкласу](../extract-subclass.html) допоможе, якщо
    частина поведінки великого класу може мати альтернативні реалізації
    або використовується лише в окремих випадках.

-   [Відокремлення інтерфейсу](../extract-interface.html) допоможе, якщо
    вам потрібно мати список операцій і поведінки, яку клієнт зможе
    використовувати.

-   В класах графічного інтерфейсу часто можна знайти дані і поведінки,
    які не відносяться до безпосереднього виводу інтерфейсу, а скоріше
    відповідають за його загальну логіку роботи. Такі дані і поведінки
    слід виділити в окремий клас предметної області, який би керував
    роботою графічного інтерфейсу. Також при цьому може виявитися
    необхідність зберігати копії деяких даних в двох місцях і
    забезпечувати їх узгодженість. [Дублювання видимих
    даних](../duplicate-observed-data.html) пропонує шлях, яким можна це
    здійснити.

<figure class="image">
<img
src="../../images/refactoring/content/smells/large-class-030a79.png?id=f0a0109f731dbc420ffe385cb658f0de"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/large-class-03.png?id=f0a0109f731dbc420ffe385cb658f0de 500w,/images/refactoring/content/smells/large-class-03-1.5x.png?id=8f7537669b0af86e9e77928004ad865f 750w,/images/refactoring/content/smells/large-class-03-2x.png?id=2e497ff65fc035f0d51f908361daee78 1000w"
sizes="(max-width: 720px) 100vw, 500px" loading="lazy" width="500"
height="300" />
</figure>

### Виграш

-   Рефакторинг таких класів позбавить розробників від необхідності
    запам'ятовувати надмірну кількість атрибутів класу.

-   У багатьох випадках розділення великих класів на частини дозволяє
    уникнути дублювання коду і функціональності.

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

[Одержимість елементарними типами []{.fa
.fa-arrow-right}](primitive-obsession.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Повернутись назад

[[]{.fa .fa-arrow-left} Довгий метод](long-method.html){.btn
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
