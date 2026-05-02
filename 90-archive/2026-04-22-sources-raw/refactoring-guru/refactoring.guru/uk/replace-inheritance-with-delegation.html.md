:::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Рефакторинг](refactoring.html){.type} /
[Прийоми рефакторингу](refactoring/techniques.html){.type} / [Задачі
узагальнення
об\'єктів](refactoring/techniques/dealing-with-generalization.html){.type}
:::

# Заміна наслідування делегуванням {#заміна-наслідування-делегуванням .title}

::: aka
Також відомий як: [Replace Inheritance with
Delegation]{style="display:inline-block"}
:::

::::: before-after-container
::: before
### Проблема

У вас є підклас, який використовує тільки частину методів суперкласу або
не хоче наслідувати його дані.
:::

::: after
### Рішення

Створіть поле і помістіть в нього об'єкт суперкласу, делегуйте виконання
методів об'єкта суперкласу, приберіть наслідування.
:::
:::::

:::::::::: examples
::::::::: {.before-after .before-after-container}
::::: before
::: ba
До
:::

::: ba-container
![Replace Inheritance with Delegation -
Before](../images/refactoring/diagrams/Replace%20Inheritance%20with%20Delegation%20-%20Before7f4b.png?id=ea9631c50eaae9a19a51f039b44e35bf){width="152"}
:::
:::::

::::: after
::: ba
Після
:::

::: ba-container
![Replace Inheritance with Delegation -
After](../images/refactoring/diagrams/Replace%20Inheritance%20with%20Delegation%20-%20Afterf4ed.png?id=7c6d23661792658feae42d051b2fd7ea){width="453"}
:::
:::::
:::::::::
::::::::::

### Причини рефакторингу

Заміна наслідування композицією може значно поліпшити дизайн класів,
якщо:

-   Ваш підклас порушує *принцип заміщення Барбари Лісков*. Іншими
    словами, наслідування виникло тільки заради об'єднання спільного
    коду, але не тому, що підклас є (is-a) розширенням суперкласу.

-   Підклас використовує тільки частину методів суперкласу. В цьому
    випадку, це тільки питання часу, поки хтось не викличе метод
    суперкласу, який він не повинен був викликати.

Суть рефакторингу зводиться до того, щоб розділити обидва класи, і
зробити суперклас помічником підкласу, а не його батьком. Замість того
щоб наслідувати усі методи суперкласу, підклас матиме тільки необхідні
методи, які делегуватимуть виконання методам об'єкта суперкласу.

### Переваги

-   Клас не містить зайвих методів, які дісталися йому в спадок від
    суперкласу.

-   У поле-делегат можна підставляти різні об'єкти, що мають різні
    реалізації функціональності. По суті, ви отримуєте реалізацію
    патерна проектування [Стратегія](design-patterns/strategy.html).

### Недоліки

-   Доводиться писати дуже багато простих делегуючих методів.

### Порядок рефакторингу

1.  Створіть поле в підкласі для утримання суперкласу. На першому етапі
    додайте в нього поточний об'єкт.

2.  Змініть методи підкласу так, щоб вони використовували об'єкт
    суперкласу, замість `this`.

3.  Для методів, які були успадковані з суперкласу і які викликаються в
    клієнтському коді, в підкласі треба створити прості делегуючі
    методи.

4.  Приберіть оголошення наслідування з підкласу.

5.  Змініть код ініціалізації поля-делегата новим об\' єктом суперкласу.

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
[Заміна делегування
наслідуванням](replace-delegation-with-inheritance.html)
:::
::::
:::::

::::: {.col-xs-12 .col-sm-6 .col-lg-6}
### Реалізує паттерн проектування

:::: dl
::: dt
[Стратегія](design-patterns/strategy.html)
:::
::::
:::::
:::::::::

::: next
#### Читаємо далі

[Заміна делегування наслідуванням []{.fa
.fa-arrow-right}](replace-delegation-with-inheritance.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Повернутись назад

[[]{.fa .fa-arrow-left} Створення шаблонного
методу](form-template-method.html){.btn .btn-default rel="prev"}
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
