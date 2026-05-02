::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Рефакторинг](refactoring.html){.type} /
[Прийоми рефакторингу](refactoring/techniques.html){.type} / [Задачі
узагальнення
об\'єктів](refactoring/techniques/dealing-with-generalization.html){.type}
:::

# Підйом поля {#підйом-поля .title}

::: aka
Також відомий як: [Pull Up Field]{style="display:inline-block"}
:::

::::: before-after-container
::: before
### Проблема

Два класи мають одне і те ж поле.
:::

::: after
### Рішення

Перемістіть поле в суперклас, прибравши його з підкласів.
:::
:::::

:::::::::: examples
::::::::: {.before-after .before-after-container}
::::: before
::: ba
До
:::

::: ba-container
![Pull Up Field -
Before](../images/refactoring/diagrams/Pull%20Up%20Field%20-%20Before41a3.png?id=8174b96aa69ce1541271a14b97cc3e33){width="452"}
:::
:::::

::::: after
::: ba
Після
:::

::: ba-container
![Pull Up Field -
After](../images/refactoring/diagrams/Pull%20Up%20Field%20-%20After12cb.png?id=c97f930072aba9f00c03ba140797de19){width="452"}
:::
:::::
:::::::::
::::::::::

### Причини рефакторингу

Підкласи розвивалися незалежно один від одного. Це привело до створення
однакових (чи дуже схожих) полів і методів.

### Переваги

-   Вбиває дублювання полів в підкласах.

-   Полегшує подальше перенесення дублюючих методів з підкласів в
    суперклас, якщо вони є.

### Порядок рефакторингу

1.  Перевірте, чи обидва поля використовуються для однакових потреб в
    підкласах.

2.  Якщо поля мають різні назви, дайте їм спільне ім'я і замініть усі
    звернення до полів в існуючому коді.

3.  Створіть поле з таким же ім'ям в суперкласі. Зверніть увагу на те,
    що якщо поля були приватні (private), поле в суперкласі має бути
    захищеним (protected).

4.  Видаліть поля з підкласів.

5.  Можливо, має сенс використати [самоінкапсуляцію
    поля](self-encapsulate-field.html) для нового поля, щоби приховати
    його за методами доступу.

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

:::::::::::: row
::::: {.col-xs-12 .col-sm-6 .col-lg-4}
### Анти-рефакторинг

:::: dl
::: dt
[Спуск поля](push-down-field.html)
:::
::::
:::::

::::: {.col-xs-12 .col-sm-6 .col-lg-4}
### Схожі рефакторинги

:::: dl
::: dt
[Підйом методу](pull-up-method.html)
:::
::::
:::::

::::: {.col-xs-12 .col-sm-6 .col-lg-4}
### Бореться з запахом

:::: dl
::: dt
[Дублювання коду](smells/duplicate-code.html)
:::
::::
:::::
::::::::::::

::: next
#### Читаємо далі

[Підйом методу []{.fa .fa-arrow-right}](pull-up-method.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Повернутись назад

[[]{.fa .fa-arrow-left} Задачі узагальнення
об\'єктів](refactoring/techniques/dealing-with-generalization.html){.btn
.btn-default rel="prev"}
:::
:::::::::::::::::::::::::::::::

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
::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::
