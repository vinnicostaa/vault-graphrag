::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Рефакторинг](refactoring.html){.type} /
[Прийоми рефакторингу](refactoring/techniques.html){.type} / [Спрощення
умовних
виразів](refactoring/techniques/simplifying-conditional-expressions.html){.type}
:::

# Видалення керуючого флагу {#видалення-керуючого-флагу .title}

::: aka
Також відомий як: [Remove Control Flag]{style="display:inline-block"}
:::

::::: before-after-container
::: before
### Проблема

У вас є булева змінна, яка грає роль керуючого флагу для декількох
булевих виразів.
:::

::: after
### Рішення

Використайте `break`, `continue` і `return` замість цієї змінної.
:::
:::::

### Причини рефакторингу

Керуючи флаги прийшли до нас з тих «бородатих» днів, коли гарним стилем
програмування вважалося мати у функції одну вхідну точку (рядок
оголошення функції) і одну вихідну точку (у самому кінці функції).

У сучасних мовах програмування цей підхід застарів, оскільки в нас
з'явилися спеціальні оператори для керування ходом програми в циклах і
інших складних конструкціях:

-   `break`: зупиняє виконання циклу;

-   `continue`: зупиняє виконання поточного кола циклу і переходить до
    перевірки умови циклу і наступної ітерації;

-   `return`: зупиняє виконання усієї функції і повертає її результат,
    якщо він поданий в цьому операторові.

### Переваги

-   Код з управляючим флагом дуже часто виходить значно заплутанішим,
    ніж при використанні операторів керування виконанням.

### Порядок рефакторингу

1.  Знайдіть, де керюючому флагу программа надає якесь значення, завдяки
    якому виконується виход з циклу або поточної ітерації.

2.  Замініть його на `break`, якщо це вихід з циклу або `continue`, якщо
    це вихід з ітерації.

3.  Приберіть весь інший код і перевірки, пов'язані з керуючим флагом.

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

::: next
#### Читаємо далі

[Заміна вкладених умовних операторів граничним оператором []{.fa
.fa-arrow-right}](replace-nested-conditional-with-guard-clauses.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Повернутись назад

[[]{.fa .fa-arrow-left} Об\'єднання фрагментів, що дублюються, в умовних
операторах](consolidate-duplicate-conditional-fragments.html){.btn
.btn-default rel="prev"}
:::
:::::::::::::

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
::::::::::::::::::
:::::::::::::::::::
