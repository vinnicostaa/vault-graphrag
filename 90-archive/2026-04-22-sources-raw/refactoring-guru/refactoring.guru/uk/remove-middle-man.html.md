:::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Рефакторинг](refactoring.html){.type} /
[Прийоми рефакторингу](refactoring/techniques.html){.type} /
[Переміщення функцій між
об\'єктами](refactoring/techniques/moving-features-between-objects.html){.type}
:::

# Видалення посередника {#видалення-посередника .title}

::: aka
Також відомий як: [Remove Middle Man]{style="display:inline-block"}
:::

::::: before-after-container
::: before
### Проблема

Клас має занадто багато методів, які просто делегують роботу іншим
об'єктам.
:::

::: after
### Рішення

Видаліть ці методи і змусьте клієнта викликати кінцеві методи
безпосередньо.
:::
:::::

:::::::::: examples
::::::::: {.before-after .before-after-container}
::::: before
::: ba
До
:::

::: ba-container
![Remove Middle Man -
Before](../images/refactoring/diagrams/Remove%20Middle%20Man%20-%20Before921c.png?id=f51110f3e0d4423b3f9088e92fc3dce4){width="153"}
:::
:::::

::::: after
::: ba
Після
:::

::: ba-container
![Remove Middle Man -
After](../images/refactoring/diagrams/Remove%20Middle%20Man%20-%20After0412.png?id=f7de1016e76545f7c51af09463ce5f4c){width="415"}
:::
:::::
:::::::::
::::::::::

### Причини рефакторингу

У цьому рефакторингу ми використаємо назви з [приховання
делегування](hide-delegate.html), а саме:

-   *Сервер* --- це об'єкт, до якого клієнт має безпосередній доступ.

-   *Делегат* --- це кінцевий об'єкт, який містить функціональність,
    потрібну клієнтові.

Існує два різновиди проблем:

1.  *Клас-сервер* нічого не робить сам по собі, створюючи зайву
    складність. В цьому випадку варто замислитися, чи потрібний цей клас
    взагалі.

2.  Кожного разу, коли в *делегаті* з'являється нова фіча, для неї треба
    створювати делегуючий метод у *класі-сервері*. Це може бути
    невигідно при великій кількості змін.

### Порядок рефакторингу

1.  Створіть геттер для доступу до об'єкта *класу-делегата* з об'єкта
    *класу-сервера*.

2.  Замініть виклики делегуючих методів *класу-сервера* прямими
    викликами методів *класу-делегата*.

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
### Анти-рефакторинг

:::: dl
::: dt
[Приховання делегування](hide-delegate.html)
:::
::::
:::::

::::: {.col-xs-12 .col-sm-6 .col-lg-6}
### Бореться з запахом

:::: dl
::: dt
[Посередник](smells/middle-man.html)
:::
::::
:::::
:::::::::

::: next
#### Читаємо далі

[Введення зовнішнього методу []{.fa
.fa-arrow-right}](introduce-foreign-method.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Повернутись назад

[[]{.fa .fa-arrow-left} Приховання делегування](hide-delegate.html){.btn
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
