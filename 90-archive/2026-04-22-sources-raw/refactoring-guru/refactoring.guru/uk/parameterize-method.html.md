::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Рефакторинг](refactoring.html){.type} /
[Прийоми рефакторингу](refactoring/techniques.html){.type} / [Спрощення
викликів
методів](refactoring/techniques/simplifying-method-calls.html){.type}
:::

# Параметризація методу {#параметризація-методу .title}

::: aka
Також відомий як: [Parameterize Method]{style="display:inline-block"}
:::

::::: before-after-container
::: before
### Проблема

Декілька методів виконують схожі дії, які відрізняються тільки якимись
внутрішніми значеннями, числами або операціями.
:::

::: after
### Рішення

Об'єднайте усі ці методи в один з параметром, в який передаватиметься
значення, що відрізняється.
:::
:::::

:::::::::: examples
::::::::: {.before-after .before-after-container}
::::: before
::: ba
До
:::

::: ba-container
![Parameterize Method -
Before](../images/refactoring/diagrams/Parameterize%20Method%20-%20Beforeb413.png?id=28369c5a763d56c4aa68ff0a9d2d6beb){width="171"}
:::
:::::

::::: after
::: ba
Після
:::

::: ba-container
![Parameterize Method -
After](../images/refactoring/diagrams/Parameterize%20Method%20-%20After948e.png?id=fd1602ffde98a8fea3ad50cb63e14553){width="171"}
:::
:::::
:::::::::
::::::::::

### Причини рефакторингу

Якщо у вас є схожі методи, скоріш за все, в них знайдеться дублюючий код
з усіма витікаючими недоліками.

Крім того, якщо вам треба буде додати ще одну варіацію функціональності,
вам доведеться створювати ще один метод. Замість цього можна б було
запустити існуючий метод з іншим параметром.

### Недоліки

-   Іноді при проведенні рефакторингу можна перестаратися, в результаті
    чого у вас з'явиться довгий і складний загальний метод замість
    декількох простих специфічних.

-   Крім того, будьте обережні, виділяючи в параметр перемикач якоїсь
    функціональності. Надалі це може привести до створення великого
    умовного оператора, який потрібно буде лікувати за допомогою [заміни
    параметра набором спеціалізованих
    методів](replace-parameter-with-explicit-methods.html).

### Порядок рефакторингу

1.  Створіть новий метод з параметром і помістіть в нього спільний для
    усіх методів код, застосовуючи [відокремлення
    методу](extract-method.html).

2.  Значення, що відрізняється, замініть на параметр в коді нового
    методу.

3.  Для кожного старого методу знайдіть місця, де він викликається, і
    поміняйте його виклики на виклики нового методу з параметром. Після
    чого старий метод можна видалити.

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

:::::::::::::: row
::::: {.col-xs-12 .col-sm-6 .col-lg-4}
### Анти-рефакторинг

:::: dl
::: dt
[Заміна параметра набором спеціалізованих
методів](replace-parameter-with-explicit-methods.html)
:::
::::
:::::

::::::: {.col-xs-12 .col-sm-6 .col-lg-4}
### Схожі рефакторинги

:::: dl
::: dt
[Відокремлення методу](extract-method.html)
:::
::::

:::: dl
::: dt
[Створення шаблонного методу](form-template-method.html)
:::
::::
:::::::

::::: {.col-xs-12 .col-sm-6 .col-lg-4}
### Бореться з запахом

:::: dl
::: dt
[Дублювання коду](smells/duplicate-code.html)
:::
::::
:::::
::::::::::::::

::: next
#### Читаємо далі

[Заміна параметра набором спеціалізованих методів []{.fa
.fa-arrow-right}](replace-parameter-with-explicit-methods.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Повернутись назад

[[]{.fa .fa-arrow-left} Розділення запиту і
модифікатора](separate-query-from-modifier.html){.btn .btn-default
rel="prev"}
:::
:::::::::::::::::::::::::::::::::

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
::::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::::
