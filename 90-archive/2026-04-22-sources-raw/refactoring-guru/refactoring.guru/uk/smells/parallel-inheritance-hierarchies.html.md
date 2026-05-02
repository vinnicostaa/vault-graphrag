:::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::: {.main-content-container .center-content-container}
:::::::::: {.page .text}
::: breadcrumb
[](../../index.html){.home} / [Рефакторинг](../refactoring.html){.type}
/ [Запахи коду](../refactoring/smells.html){.type} / [Ускладнювачі
змін](../refactoring/smells/change-preventers.html){.type}
:::

# Паралельні ієрархії наслідування {#паралельні-ієрархії-наслідування .title}

::: aka
Також відомий як: [Parallel Inheritance
Hierarchies]{style="display:inline-block"}
:::

### Симптоми і ознаки

Всякий раз при створенні підкласу якогось класу доводиться створювати ще
один підклас для іншого класу.

<figure class="image">
<img
src="../../images/refactoring/content/smells/parallel-inheritance-hierarchies-0187af.png?id=9167875f5f0e80256edcc8fcaaed3563"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/parallel-inheritance-hierarchies-01.png?id=9167875f5f0e80256edcc8fcaaed3563 500w,/images/refactoring/content/smells/parallel-inheritance-hierarchies-01-1.5x.png?id=e2129d5eb8da79a491ed0c7d802e8d1c 750w,/images/refactoring/content/smells/parallel-inheritance-hierarchies-01-2x.png?id=975e6a0589795c59b47ed3aa122beead 1000w"
sizes="(max-width: 720px) 100vw, 500px" data-fetchpriority="high"
width="500" height="300" />
</figure>

### Причини появи

Доки ієрархія була невелика, все було добре. Але з додаванням нових
класів вносити зміни ставало все складніше і складніше.

### Лікування

-   Ви можете спробувати усунути дублювання паралельних класів в два
    етапи. По-перше, потрібно змусити екземпляри однієї ієрархії
    посилатися на екземпляри іншої ієрархії. Потім слід прибрати
    ієрархію в класі, що посилається, за допомогою [переміщення
    методу](../move-method.html) і [переміщення
    поля](../move-field.html).

### Виграш

-   Зникає дублювання коду.

-   Може поліпшити організацію коду.

<figure class="image">
<img
src="../../images/refactoring/content/smells/parallel-inheritance-hierarchies-02b2cf.png?id=4dca6795d3d087b23ad1027298d6f1dd"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/parallel-inheritance-hierarchies-02.png?id=4dca6795d3d087b23ad1027298d6f1dd 500w,/images/refactoring/content/smells/parallel-inheritance-hierarchies-02-1.5x.png?id=ef89bd3332ebb11a6c6974a9e8017779 750w,/images/refactoring/content/smells/parallel-inheritance-hierarchies-02-2x.png?id=b45e8dde4f4abbe2f0d329964c921960 1000w"
sizes="(max-width: 720px) 100vw, 500px" loading="lazy" width="500"
height="300" />
</figure>

### Не варто чіпати, якщо\...

-   Іноді наявність паралельної ієрархії --- це необхідне зло, без якого
    структура коду програми була б ще гірше. Якщо ви виявите, що ваші
    спроби усунути дублювання призводять до ще більшого погіршення
    організації коду, то\... зупиніть рефакторинг, відкатіть всі внесені
    зміни, випийте чаю і почніть звикати до цього коду.

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

[Забрюднювачі коду []{.fa
.fa-arrow-right}](../refactoring/smells/dispensables.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Повернутись назад

[[]{.fa .fa-arrow-left} Стрільба дробом](shotgun-surgery.html){.btn
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
