:::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Рефакторинг](refactoring.html){.type} /
[Прийоми рефакторингу](refactoring/techniques.html){.type} / [Спрощення
викликів
методів](refactoring/techniques/simplifying-method-calls.html){.type}
:::

# Видалення параметру {#видалення-параметру .title}

::: aka
Також відомий як: [Remove Parameter]{style="display:inline-block"}
:::

::::: before-after-container
::: before
### Проблема

Параметр не використовується в тілі методу.
:::

::: after
### Рішення

Видаліть невживаний параметр.
:::
:::::

:::::::::: examples
::::::::: {.before-after .before-after-container}
::::: before
::: ba
До
:::

::: ba-container
![Remove Parameter -
Before](../images/refactoring/diagrams/Remove%20Parameter%20-%20Before1916.png?id=c49cf53fd9ebf39daa1ec27bb897a5cf){width="171"}
:::
:::::

::::: after
::: ba
Після
:::

::: ba-container
![Remove Parameter -
After](../images/refactoring/diagrams/Remove%20Parameter%20-%20After4dba.png?id=93177c2a159087d73ebef60cc8c158a3){width="171"}
:::
:::::
:::::::::
::::::::::

### Причини рефакторингу

Кожен параметр у виклику методу примушує людину, що пише код методу,
роздумувати про те, яка інформація може опинитися в цьому параметрі. І
якщо параметр потім зовсім не використовується в тілі методу, значить,
весь цей розумовий процес йде даремно.

Крім того, додаткові параметри --- це ще і зайвий код до виконання.

Іноді ми додаємо параметри на майбутнє, передчуваючи в майбутньому зміни
в методі, для яких знадобиться цей параметр. Проте, досвід показує, що
краще додати параметр тоді, коли він дійсно знадобиться, адже очікувані
зміни можуть так і не настати.

### Переваги

-   Метод повинен містити тільки дійсно необхідні параметри.

### Коли не слід застосовувати

-   Не варто починати цей рефакторинг, якщо метод має альтернативні
    реалізації в підкласах або в суперкласі, і ваш параметр
    використовується в цих реалізаціях.

### Порядок рефакторингу

1.  Перевірте, чи не визначений метод в суперкласі або підкласі. Якщо
    так, чи використовується там параметр? Якщо в якійсь з реалізацій
    параметр використовується, утримайтеся від рефакторингу.

2.  Наступний крок потрібен, щоб зберегти працездатність програми під
    час рефакторингу. Створіть новий метод, скопіювавши старий, і
    видаліть з нього необхідний параметр. Замініть код старого методу
    викликом нового методу.

3.  Знайдіть усі звернення до старого методу і замініть їх зверненнями
    до нового методу.

4.  Видаліть старий метод. Цей крок неможливо виконати, якщо старий
    метод є частиною публічного інтерфейсу. В цьому випадку, старий
    метод треба помітити як застарілий (`deprecated`)

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

::::::::::::::: row
::::: {.col-xs-12 .col-sm-6 .col-lg-3}
### Анти-рефакторинг

:::: dl
::: dt
[Додавання параметра](add-parameter.html)
:::
::::
:::::

::::: {.col-xs-12 .col-sm-6 .col-lg-3}
### Схожі рефакторинги

:::: dl
::: dt
[Перейменування методу](rename-method.html)
:::
::::
:::::

::::: {.col-xs-12 .col-sm-6 .col-lg-3}
### Домогає іншим рефакторингам

:::: dl
::: dt
[Заміна параметра викликом
методу](replace-parameter-with-method-call.html)
:::
::::
:::::

::::: {.col-xs-12 .col-sm-6 .col-lg-3}
### Бореться з запахом

:::: dl
::: dt
[Теоретична спільність](smells/speculative-generality.html)
:::
::::
:::::
:::::::::::::::

::: next
#### Читаємо далі

[Розділення запиту і модифікатора []{.fa
.fa-arrow-right}](separate-query-from-modifier.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Повернутись назад

[[]{.fa .fa-arrow-left} Додавання параметра](add-parameter.html){.btn
.btn-default rel="prev"}
:::
::::::::::::::::::::::::::::::::::

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
:::::::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::::::
