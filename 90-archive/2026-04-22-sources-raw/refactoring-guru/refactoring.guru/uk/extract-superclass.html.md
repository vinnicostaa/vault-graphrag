:::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Рефакторинг](refactoring.html){.type} /
[Прийоми рефакторингу](refactoring/techniques.html){.type} / [Задачі
узагальнення
об\'єктів](refactoring/techniques/dealing-with-generalization.html){.type}
:::

# Відокремлення суперкласу {#відокремлення-суперкласу .title}

::: aka
Також відомий як: [Extract Superclass]{style="display:inline-block"}
:::

::::: before-after-container
::: before
### Проблема

У вас є два класи із схожими полями і методами.
:::

::: after
### Рішення

Створіть для них спільний суперклас і перенесіть туди схожі поля і
методи.
:::
:::::

:::::::::: examples
::::::::: {.before-after .before-after-container}
::::: before
::: ba
До
:::

::: ba-container
![Extract Superclass -
Before](../images/refactoring/diagrams/Extract%20Superclass%20-%20Beforef473.png?id=e8192774aeefddece5b3c1a7a868127d){width="209"}
:::
:::::

::::: after
::: ba
Після
:::

::: ba-container
![Extract Superclass -
After](../images/refactoring/diagrams/Extract%20Superclass%20-%20After12ec.png?id=1cff46d95be1632df8af715e14ea88c9){width="509"}
:::
:::::
:::::::::
::::::::::

### Причини рефакторингу

Одним з видів дублювання коду є наявність двох класів, що виконують
схожі завдання однаковим способом або схожі завдання різними способами.
Об'єкти надають вбудований механізм для спрощення такої ситуації за
допомогою наслідування. Проте часто спільність виявляється непоміченою
до тих пір, поки не будуть створені якісь класи, і тоді з'являється
необхідність створювати структуру наслідування пізніше.

### Переваги

-   Прибирає дублювання коду. Схожі поля і методи тепер «живуть» тільки
    в одному місці.

### Коли не слід застосовувати

-   Ви не можете застосувати цей рефакторинг до класів, які вже мають
    суперклас.

### Порядок рефакторингу

1.  Створіть абстрактний суперклас.

2.  Використайте [підйом поля](pull-up-field.html), [підйом
    методу](pull-up-method.html) і [підйом тіла
    конструктора](pull-up-constructor-body.html) для переміщення
    спільної функціональності в суперклас. Краще розпочинати з полів,
    оскільки окрім спільних полів, вам треба буде перенести ті з них,
    які використовуються в спільних методах.

3.  Варто пошукати місця в клієнтському коді, в яких можна замінити
    використання підкласів вашим спільним класом (наприклад, в
    оголошеннях типів).

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

::::::::: row
::::: {.col-xs-12 .col-sm-6 .col-lg-6}
### Схожі рефакторинги

:::: dl
::: dt
[Відокремлення інтерфейсу](extract-interface.html)
:::
::::
:::::

::::: {.col-xs-12 .col-sm-6 .col-lg-6}
### Бореться з запахом

:::: dl
::: dt
[Дублювання коду](smells/duplicate-code.html)
:::
::::
:::::
:::::::::

::: next
#### Читаємо далі

[Відокремлення інтерфейсу []{.fa
.fa-arrow-right}](extract-interface.html){.btn .btn-primary rel="next"}
:::

::: prev
#### Повернутись назад

[[]{.fa .fa-arrow-left} Витягання підкласу](extract-subclass.html){.btn
.btn-default rel="prev"}
:::
::::::::::::::::::::::::::::

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
:::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::
