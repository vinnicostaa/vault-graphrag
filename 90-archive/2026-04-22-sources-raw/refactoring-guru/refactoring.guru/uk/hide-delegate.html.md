:::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Рефакторинг](refactoring.html){.type} /
[Прийоми рефакторингу](refactoring/techniques.html){.type} /
[Переміщення функцій між
об\'єктами](refactoring/techniques/moving-features-between-objects.html){.type}
:::

# Приховання делегування {#приховання-делегування .title}

::: aka
Також відомий як: [Hide Delegate]{style="display:inline-block"}
:::

::::: before-after-container
::: before
### Проблема

Клієнт отримує об'єкт B з поля або методу об'єкта А. Потім клієнт
викликає якийсь метод об'єкта B.
:::

::: after
### Рішення

Створіть новий метод в класі А, який би делегував виклик об'єкта B.
Таким чином, клієнт перестане знати про клас В і залежати від нього.
:::
:::::

:::::::::: examples
::::::::: {.before-after .before-after-container}
::::: before
::: ba
До
:::

::: ba-container
![Hide Delegate -
Before](../images/refactoring/diagrams/Hide%20Delegate%20-%20Before0412.png?id=f7de1016e76545f7c51af09463ce5f4c){width="415"}
:::
:::::

::::: after
::: ba
Після
:::

::: ba-container
![Hide Delegate -
After](../images/refactoring/diagrams/Hide%20Delegate%20-%20After921c.png?id=f51110f3e0d4423b3f9088e92fc3dce4){width="153"}
:::
:::::
:::::::::
::::::::::

### Причини рефакторингу

Спершу слід визначитися з назвами:

-   *Сервер* --- це об'єкт, до якого клієнт має безпосередній доступ.

-   *Делегат* --- це кінцевий об'єкт, який містить функціональність,
    потрібну клієнтові.

Ланцюжок викликів з'являється тоді, коли клієнт запитує з одного об'єкта
інший, потім вже другий об'єкт запитує ще один і так далі. Такі
послідовності викликів означають, що клієнт пов'язаний з навігацією по
структурі класів. Будь-які зміни проміжних зв'язків означають
необхідність модифікації клієнта.

### Переваги

-   Приховує делегування від клієнта. Чим менше клієнтський код знає
    подробиць про зв'язки між об'єктами, тим простіше згодом вноситиме
    зміни в програму.

### Недоліки

-   Якщо потрібно створити занадто багато делегуючих методів,
    *клас-сервер* ризикує перетворитися на зайву проміжну ланку і
    привести до запаху [посередник](smells/middle-man.html).

### Порядок рефакторингу

1.  Для кожного методу *класа-делегата*, що викликається клієнтом, треба
    створити метод в *класі-сервері*, який би делегував виклик
    *класу-делегату*.

2.  Змініть код клієнта так, щоб він викликав методи *класу-сервера*.

3.  Якщо після всіх змін клієнт більше не має потреби в
    *класі-делегаті*, можна прибрати метод доступу до *класу-делегата* з
    *класу-сервера* (той метод, який використовувався спочатку для
    отримання класу-делегата).

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

::::::::::: row
::::: {.col-xs-12 .col-sm-6 .col-lg-6}
### Анти-рефакторинг

:::: dl
::: dt
[Видалення посередника](remove-middle-man.html)
:::
::::
:::::

::::::: {.col-xs-12 .col-sm-6 .col-lg-6}
### Бореться з запахом

:::: dl
::: dt
[Ланцюжок викликів](smells/message-chains.html)
:::
::::

:::: dl
::: dt
[Недоречна близькість](smells/inappropriate-intimacy.html)
:::
::::
:::::::
:::::::::::

::: next
#### Читаємо далі

[Видалення посередника []{.fa
.fa-arrow-right}](remove-middle-man.html){.btn .btn-primary rel="next"}
:::

::: prev
#### Повернутись назад

[[]{.fa .fa-arrow-left} Вбудовування класу](inline-class.html){.btn
.btn-default rel="prev"}
:::
::::::::::::::::::::::::::::::

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
:::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::
