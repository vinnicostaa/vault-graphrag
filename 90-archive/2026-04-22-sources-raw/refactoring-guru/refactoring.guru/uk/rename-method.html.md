:::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Рефакторинг](refactoring.html){.type} /
[Прийоми рефакторингу](refactoring/techniques.html){.type} / [Спрощення
викликів
методів](refactoring/techniques/simplifying-method-calls.html){.type}
:::

# Перейменування методу {#перейменування-методу .title}

::: aka
Також відомий як: [Rename Method]{style="display:inline-block"}
:::

::::: before-after-container
::: before
### Проблема

Назва методу не розкриває суть того, що він робить.
:::

::: after
### Рішення

Змініть назву методу.
:::
:::::

:::::::::: examples
::::::::: {.before-after .before-after-container}
::::: before
::: ba
До
:::

::: ba-container
![Rename Method -
Before](../images/refactoring/diagrams/Rename%20Method%20-%20Before175a.png?id=7943798ae9db6b5b232860eed6262462){width="171"}
:::
:::::

::::: after
::: ba
Після
:::

::: ba-container
![Rename Method -
After](../images/refactoring/diagrams/Rename%20Method%20-%20Aftere5be.png?id=62b4e6747951bbbacba3ede379fef200){width="171"}
:::
:::::
:::::::::
::::::::::

### Причини рефакторингу

Метод міг дістати невдалу назву із самого початку. Наприклад, хтось
створив метод похапцем, не надав належного значення хорошій назві.

З іншого боку, метод міг бути названий спочатку вдало, але зважаючи на
розширення його функціональності, ім'я методу перестало бути актуальним.

### Переваги

-   Покращує читабельність коду. Придумайте для нового методу таку
    назву, яка б відбивала суть того, що він робить. Наприклад,
    `createOrder()`, `renderCustomerInfo()` і так далі.

### Порядок рефакторингу

1.  Перевірте, чи не визначений метод в суперкласі або підкласі. Якщо
    так, треба буде повторити усі кроки і в цих класах.

2.  Наступний крок важливий, щоб зберегти працездатність програми під
    час рефакторингу. Отже, створіть новий метод з новими ім'ям.
    Скопіюйте туди код старого методу. Видаліть весь код в старому
    методі, а замість нього додайте виклик нового методу.

3.  Знайдіть усі звернення до старого методу і замініть їх зверненнями
    до нового.

4.  Видаліть старий метод. Цей крок неможливо виконати, якщо старий
    метод є частиною публічного інтерфейсу. В цьому випадку, старий
    метод треба помітити як застарілий (`deprecated`).

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

::::::::::::: row
::::::: {.col-xs-12 .col-sm-6 .col-lg-6}
### Схожі рефакторинги

:::: dl
::: dt
[Додавання параметра](add-parameter.html)
:::
::::

:::: dl
::: dt
[Видалення параметру](remove-parameter.html)
:::
::::
:::::::

::::::: {.col-xs-12 .col-sm-6 .col-lg-6}
### Бореться з запахом

:::: dl
::: dt
[Альтернативні класи з різними
інтерфейсами](smells/alternative-classes-with-different-interfaces.html)
:::
::::

:::: dl
::: dt
[Коментарі](smells/comments.html)
:::
::::
:::::::
:::::::::::::

::: next
#### Читаємо далі

[Додавання параметра []{.fa .fa-arrow-right}](add-parameter.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Повернутись назад

[[]{.fa .fa-arrow-left} Спрощення викликів
методів](refactoring/techniques/simplifying-method-calls.html){.btn
.btn-default rel="prev"}
:::
::::::::::::::::::::::::::::::::

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
:::::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::::
