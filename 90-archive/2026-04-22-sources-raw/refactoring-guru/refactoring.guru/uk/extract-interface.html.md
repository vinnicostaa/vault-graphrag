::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Рефакторинг](refactoring.html){.type} /
[Прийоми рефакторингу](refactoring/techniques.html){.type} / [Задачі
узагальнення
об\'єктів](refactoring/techniques/dealing-with-generalization.html){.type}
:::

# Відокремлення інтерфейсу {#відокремлення-інтерфейсу .title}

::: aka
Також відомий як: [Extract Interface]{style="display:inline-block"}
:::

::::: before-after-container
::: before
### Проблема

Декілька клієнтів користуються однією і тією ж частиною інтерфейсу
класу. Або в двох класах частина інтерфейсу виявилася спільною.
:::

::: after
### Рішення

Виділіть цю спільну частину в свій власний інтерфейс.
:::
:::::

:::::::::: examples
::::::::: {.before-after .before-after-container}
::::: before
::: ba
До
:::

::: ba-container
![Extract Interface -
Before](../images/refactoring/diagrams/Extract%20Interface%20-%20Before543f.png?id=fc2bba34908c16e2b86c5c9cc8c424b1){width="209"}
:::
:::::

::::: after
::: ba
Після
:::

::: ba-container
![Extract Interface -
After](../images/refactoring/diagrams/Extract%20Interface%20-%20After2e06.png?id=2692cc5b979184941c55080bc89226b4){width="209"}
:::
:::::
:::::::::
::::::::::

### Причини рефакторингу

1.  Інтерфейси бувають корисні, коли один клас може відігрувати
    різноманітні ролі в залежності від ситуації. Використайте
    [відокремлення інтерфейсу](extract-interface.html) щоб явно
    позначити кожну з ролей.

2.  Ще одна слушна нагода виникає, коли потрібно описати операції, які
    клас виконує на своєму сервері. Якщо в майбутньому передбачається
    дозволити використання серверів декількох видів, усі вони повинні
    реалізовувати цей інтерфейс.

### Корисні факти

Є деяка схожість між [відокремленням
суперкласу](extract-superclass.html) і [відокремленням
інтерфейсу](extract-interface.html).

Відокремлення інтерфейсу дозволяє виділяти тільки спільні інтерфейси,
але не спільний код. Іншими словами, якщо в класах знаходиться
[дублюючий код](smells/duplicate-code.html), то, застосувавши
відокремлення інтерфейсу, ви ніяк не позбавитеся від цього дублювання.

Проте, цю проблему можна зменшити, застосувавши [відокремлення
класу](extract-class.html) для розміщення поведінки, що містить
дублювання, в окремий компонент і делегування йому усієї роботи. У
випадку, якщо об'єм спільної поведінки виявиться досить великим, завжди
можна застосувати [відокремлення суперкласу](extract-superclass.html).
Звичайно, це навіть простіше, але пам'ятайте, що при цьому ви отримуєте
тільки один батьківський клас.

### Порядок рефакторингу

1.  Створіть порожній інтерфейс.

2.  Оголосіть спільні операції в інтерфейсі.

3.  Оголосіть потрібні класи, які будуть реалізовуюти цей інтерфейс.

4.  Змініть оголошення типів в клієнтському коді так, щоб вони
    використовували новий інтерфейс.

::::: {.prom .banner-content .banner .banner-striped .banner-with-image data-id="Ref: Part of the ebook" creative-id="animated-uk" position="content_bottom"}
::: banner-image
Your browser does not support HTML video.
:::

::: banner-text
### Замучились читати? {#замучились-читати .title}

Збігайте за подушкою, в нас тут контенту на [7 годин]{.blue} читання.

Або спробуйте наш новий інтерактивний курс. Він набагато цікавіший за
банальний тест.

[ Дізнатися більше...](refactoring/course.html){.btn .btn-secondary}
:::
:::::

:::::: row
::::: {.col-xs-12 .col-sm-6 .col-lg-12}
### Схожі рефакторинги

:::: dl
::: dt
[Відокремлення суперкласу](extract-superclass.html)
:::
::::
:::::
::::::

::: next
#### Читаємо далі

[Згортання ієрархії []{.fa
.fa-arrow-right}](collapse-hierarchy.html){.btn .btn-primary rel="next"}
:::

::: prev
#### Повернутись назад

[[]{.fa .fa-arrow-left} Відокремлення
суперкласу](extract-superclass.html){.btn .btn-default rel="prev"}
:::
:::::::::::::::::::::::::

:::::: {.prom .banner-sidebar .banner-removable .banner-removable-refactoring data-id="Ref: Sidebar" creative-id="standard-sidebar-uk" position="sidebar"}
::::: ban
::: {.image .product-image}
[Знижки!]{.banner-discount}
[![](../images/refactoring/course/course-cover-uk4341.jpg?id=e53397ef67078e742fb132da37ca7cc8){width="300"
height="300" loading="lazy"}](refactoring/course.html)
:::

::: {.banner-text .banner-text-uk}
Цей рефакторинг є частиною нашого інтерактивного **онлайн курсу з
рефакторингу**.

[ Подробиці...](refactoring/course.html){.btn .btn-secondary .btn-block}
:::
:::::
::::::
::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::
