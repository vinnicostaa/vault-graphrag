:::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Рефакторинг](refactoring.html){.type} /
[Прийоми рефакторингу](refactoring/techniques.html){.type} / [Задачі
узагальнення
об\'єктів](refactoring/techniques/dealing-with-generalization.html){.type}
:::

# Створення шаблонного методу {#створення-шаблонного-методу .title}

::: aka
Також відомий як: [Form Template Method]{style="display:inline-block"}
:::

::::: before-after-container
::: before
### Проблема

В підкласах реалізовані алгоритми, що містять схожі кроки і однаковий
порядок виконання цих кроків.
:::

::: after
### Рішення

Винесіть структуру алгоритму і однакові кроки в суперклас, а в підкласах
залиште реалізацію кроків, що відрізняються.
:::
:::::

:::::::::: examples
::::::::: {.before-after .before-after-container}
::::: before
::: ba
До
:::

::: ba-container
![Form Template Method -
Before](../images/refactoring/diagrams/Form%20Template%20Method%20-%20Beforee8c2.png?id=858de51c181ba956e5ecf63e3a88fda2){width="696"}
:::
:::::

::::: after
::: ba
Після
:::

::: ba-container
![Form Template Method -
After](../images/refactoring/diagrams/Form%20Template%20Method%20-%20After9b6d.png?id=7a2977c70d9fcd80c96307a877343828){width="640"}
:::
:::::
:::::::::
::::::::::

### Причини рефакторингу

Підкласи розвиваються паралельно. Íноді з ними працюють різні люди, що
призводить до дублювання коду і помилок, а також до ускладнення
підтримки, оскільки кожну зміну доводиться проводити в усіх підкласах.

### Переваги

-   Коли ми говоримо про дублювання коду, не завжди мається на увазі
    програмування методом копіювання-вставки. Часто дублювання виникає
    на абстрактнішому рівні. Наприклад, у вас є метод сортування чисел і
    метод сортування колекції об'єктів, при цьому, єдине, чим вони
    відрізняються це порівняння елементів. Створення шаблонного методу
    дозволяє впоратися з таким дублюванням, об'єднавши спільні кроки
    алгоритму в суперкласі і залишивши відмінності для підкласів.

-   Створення шаблонного методу реалізує *принцип
    відкритості/закритості*. Під час реалізації нової версії алгоритму
    вам треба буде усього лише створити новий підклас, не міняючи
    існуючий код.

### Порядок рефакторингу

1.  Розбийте алгоритми в підкласах на складові частини, описані в
    окремих методах. У цьому може допомогти [витягання
    методу](extract-method.html).

2.  Методи, що вийшли однаковимі для усіх підкласів, можете сміливо
    переміщати в суперклас, використовуючи [підйом
    методу](pull-up-method.html).

3.  Методи, що відрізняються, приведіть до єдиних назв за допомогою
    [перейменування методу](rename-method.html).

4.  Помістіть сигнатури методів, що відрізняються, в суперклас як
    абстрактні за допомогою [підйому методу](pull-up-method.html). Їх
    реалізації залиште в підкласах.

5.  І, нарешті, підніміть основний метод алгоритму в суперклас. Він
    тепер повинен працювати з методами-кроками, описаними в
    суперкласі --- реальними або абстрактними.

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
### Реалізує паттерн проектування

:::: dl
::: dt
[Шаблонний метод](design-patterns/template-method.html)
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

[Заміна наслідування делегуванням []{.fa
.fa-arrow-right}](replace-inheritance-with-delegation.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Повернутись назад

[[]{.fa .fa-arrow-left} Згортання
ієрархії](collapse-hierarchy.html){.btn .btn-default rel="prev"}
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
