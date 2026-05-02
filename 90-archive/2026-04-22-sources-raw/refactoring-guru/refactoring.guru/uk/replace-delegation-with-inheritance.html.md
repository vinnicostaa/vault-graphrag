:::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Рефакторинг](refactoring.html){.type} /
[Прийоми рефакторингу](refactoring/techniques.html){.type} / [Задачі
узагальнення
об\'єктів](refactoring/techniques/dealing-with-generalization.html){.type}
:::

# Заміна делегування наслідуванням {#заміна-делегування-наслідуванням .title}

::: aka
Також відомий як: [Replace Delegation with
Inheritance]{style="display:inline-block"}
:::

::::: before-after-container
::: before
### Проблема

Клас містить безліч простих делегуючих методів до усіх методів іншого
класу.
:::

::: after
### Рішення

Зробіть клас спадкоємцем делегата, після чого делегуючі методи втратять
сенс.
:::
:::::

:::::::::: examples
::::::::: {.before-after .before-after-container}
::::: before
::: ba
До
:::

::: ba-container
![Replace Delegation with Inheritance -
Before](../images/refactoring/diagrams/Replace%20Delegation%20with%20Inheritance%20-%20Before7d55.png?id=b408bcc35ce5a341e675818d6c1124a2){width="453"}
:::
:::::

::::: after
::: ba
Після
:::

::: ba-container
![Replace Delegation with Inheritance -
After](../images/refactoring/diagrams/Replace%20Delegation%20with%20Inheritance%20-%20After3a88.png?id=633a1c3ff0e87033e10408df57ac6118){width="152"}
:::
:::::
:::::::::
::::::::::

### Причини рефакторингу

Делегування є гнучкішим підходом, ніж наслідування, оскільки дозволяє
змінювати поведінку класу на льоту, заміняючи об'єкт, до якого
делегується виконання. Проте, застосування делегування перестає бути
вигідним, якщо ви делегуєте дії тільки одному класу, причому усім його
публічним методам.

Якщо в цьому випадку замінити делегування наслідуванням, ви позбавите
клас від безлічі делегуючих методів, а себе від необхідності створювати
їх для кожного нового методу класу-делегата.

### Переваги

-   Зменшує кількість коду. Вам більше не потрібні усі ці делегуючі
    методи.

### Коли не слід застосовувати

-   Не застосовуйте рефакторинг, якщо клас містить делегування тільки до
    частини публічних методів класу-делегата. Цим ви порушите *принцип
    заміщення Барбари Лісков*.

-   Цей рефакторинг може бути застосований тільки якщо клас ще не має
    батьків.

### Порядок рефакторингу

1.  Зробіть клас підкласом класу-делегата.

2.  У поле, що містить посилання на об'єкт-делегат, поставте поточний
    об'єкт.

3.  Один за іншим видаляйте методи з простим делегуванням. Якщо в них
    відрізнялися назви, використайте [перейменування
    методу](rename-method.html) щоби привести усі методи до однієї
    назви.

4.  Замініть усі звернення до поля-делегата зверненнями до поточного
    об'єкта.

5.  Видаліть поле-делегат.

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
[Заміна наслідування
делегуванням](replace-inheritance-with-delegation.html)
:::
::::
:::::

::::: {.col-xs-12 .col-sm-6 .col-lg-4}
### Схожі рефакторинги

:::: dl
::: dt
[Видалення посередника](remove-middle-man.html)
:::
::::
:::::

::::: {.col-xs-12 .col-sm-6 .col-lg-4}
### Бореться з запахом

:::: dl
::: dt
[Недоречна близькість](smells/inappropriate-intimacy.html)
:::
::::
:::::
::::::::::::

::: prev
#### Повернутись назад

[[]{.fa .fa-arrow-left} Заміна наслідування
делегуванням](replace-inheritance-with-delegation.html){.btn
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
