:::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::: {.main-content-container .center-content-container}
:::::::::: {.page .text}
::: breadcrumb
[](../../index.html){.home} / [Рефакторинг](../refactoring.html){.type}
/ [Запахи коду](../refactoring/smells.html){.type} / [Порушники
об\'єктно-орієнтованого
дизайну](../refactoring/smells/oo-abusers.html){.type}
:::

# Альтернативні класи з різними інтерфейсами {#альтернативні-класи-з-різними-інтерфейсами .title}

::: aka
Також відомий як: [Alternative Classes with Different
Interfaces]{style="display:inline-block"}
:::

### Симптоми і ознаки

Два класи виконують однакові функції, але мають різні назви методів.

<figure class="image">
<img
src="../../images/refactoring/content/smells/alternative-classes-with-different-interfaces-011fe4.png?id=e5fccb2e5390e0a62b5c9f56029bd361"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/alternative-classes-with-different-interfaces-01.png?id=e5fccb2e5390e0a62b5c9f56029bd361 500w,/images/refactoring/content/smells/alternative-classes-with-different-interfaces-01-1.5x.png?id=41cee7e73c74ea76fd0e5a65157aef1d 750w,/images/refactoring/content/smells/alternative-classes-with-different-interfaces-01-2x.png?id=00f0e52d679514e0c16e836e7cee5c24 1000w"
sizes="(max-width: 720px) 100vw, 500px" data-fetchpriority="high"
width="500" height="300" />
</figure>

### Причини появи

Програміст, який створив один з класів, скоріше за все, не знав, що в
програмі вже існує аналогічний за функціоналом клас.

### Лікування

Потрібно привести інтерфейс класів до спільного знаменника:

-   [Перейменуйте методи](../rename-method.html) таким чином, щоб вони
    стали однаковими для всіх альтернативних класів.

-   Скористайтеся [переміщенням методу](../move-method.html),
    [додаванням параметру](../add-parameter.html) і [параметризацією
    методу](../parameterize-method.html) для того, щоб сигнатура і
    реалізація методів стали однаковими.

-   Якщо ідентичною є тільки частина функціональності класів, спробуйте
    [відокремити цю частину в спільний
    суперклас](../extract-superclass.html). При цьому існуючі класи
    стають підкласами.

-   Після того, як ви визначилися з варіантом «лікування» і здійснили
    його, подумайте, можливо, один з класів тепер можна видалити.

### Виграш

-   Ви позбавляєтесь від зайвого дублювання коду, і, таким чином,
    зменшуєте його розмір.

-   Підвищується читабельність коду, поліпшується його розуміння. Вам
    більше не доведеться гадати, навіщо створювався другий клас, який
    виконує точно такі ж функції, як і перший.

<figure class="image">
<img
src="../../images/refactoring/content/smells/alternative-classes-with-different-interfaces-02ece4.png?id=669874e082965799a70076a120288c6a"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/alternative-classes-with-different-interfaces-02.png?id=669874e082965799a70076a120288c6a 500w,/images/refactoring/content/smells/alternative-classes-with-different-interfaces-02-1.5x.png?id=720363b1a9f666cc0d41345ccf4e4de3 750w,/images/refactoring/content/smells/alternative-classes-with-different-interfaces-02-2x.png?id=db011d16b1dcea2e68d252eb435e63ef 1000w"
sizes="(max-width: 720px) 100vw, 500px" loading="lazy" width="500"
height="300" />
</figure>

### Не варто чіпати, якщо\...

-   Іноді об'єднання класів виявляється неможливим або настільки
    складним, що сенсу займатися цією роботою немає. Один із
    прикладів --- альтернативні класи знаходяться в двох різних
    бібліотеках, кожна з яких має свою версію класу.

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

[Ускладнювачі змін []{.fa
.fa-arrow-right}](../refactoring/smells/change-preventers.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Повернутись назад

[[]{.fa .fa-arrow-left} Відмова від спадку](refused-bequest.html){.btn
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
