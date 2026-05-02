:::::::::::::::::::::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Рефакторинг](refactoring.html){.type} /
[Прийоми рефакторингу](refactoring/techniques.html){.type} / [Спрощення
викликів
методів](refactoring/techniques/simplifying-method-calls.html){.type}
:::

# Передача усього об\'єкта {#передача-усього-обєкта .title}

::: aka
Також відомий як: [Preserve Whole Object]{style="display:inline-block"}
:::

::::: before-after-container
::: before
### Проблема

Ви отримуєте декілька значень від об'єкта, а потім передаєте їх в метод
як параметри.
:::

::: after
### Рішення

Замість цього передавайте весь об'єкт.
:::
:::::

:::::::::::::::::::::::::::: examples
::::::: {.before-after .before-after-container .java data-lang="java"}
:::: before
::: ba
До
:::

``` {.code lang="java"}
int low = daysTempRange.getLow();
int high = daysTempRange.getHigh();
boolean withinPlan = plan.withinRange(low, high);
```
::::

:::: after
::: ba
Після
:::

``` {.code lang="java"}
boolean withinPlan = plan.withinRange(daysTempRange);
```
::::
:::::::

::::::: {.before-after .before-after-container .csharp data-lang="csharp"}
:::: before
::: ba
До
:::

``` {.code lang="csharp"}
int low = daysTempRange.GetLow();
int high = daysTempRange.GetHigh();
bool withinPlan = plan.WithinRange(low, high);
```
::::

:::: after
::: ba
Після
:::

``` {.code lang="csharp"}
bool withinPlan = plan.WithinRange(daysTempRange);
```
::::
:::::::

::::::: {.before-after .before-after-container .php data-lang="php"}
:::: before
::: ba
До
:::

``` {.code lang="php"}
$low = $daysTempRange->getLow();
$high = $daysTempRange->getHigh();
$withinPlan = $plan->withinRange($low, $high);
```
::::

:::: after
::: ba
Після
:::

``` {.code lang="php"}
$withinPlan = $plan->withinRange($daysTempRange);
```
::::
:::::::

::::::: {.before-after .before-after-container .python data-lang="python"}
:::: before
::: ba
До
:::

``` {.code lang="python"}
low = daysTempRange.getLow()
high = daysTempRange.getHigh()
withinPlan = plan.withinRange(low, high)
```
::::

:::: after
::: ba
Після
:::

``` {.code lang="python"}
withinPlan = plan.withinRange(daysTempRange)
```
::::
:::::::

::::::: {.before-after .before-after-container .typescript data-lang="typescript"}
:::: before
::: ba
До
:::

``` {.code lang="typescript"}
let low = daysTempRange.getLow();
let high = daysTempRange.getHigh();
let withinPlan = plan.withinRange(low, high);
```
::::

:::: after
::: ba
Після
:::

``` {.code lang="typescript"}
let withinPlan = plan.withinRange(daysTempRange);
```
::::
:::::::
::::::::::::::::::::::::::::

### Причини рефакторингу

Проблема полягає в тому, що перед викликом вашого методу треба кожного
разу викликати методи майбутнього об'єкта-параметра. Якщо метод виклику
цих методів або кількість даних, що отримуються для методу, зміниться,
то зміни доведеться шукати і вносити в безлічі місць програми.

Замість цього код отримання усіх потрібних даних може зберігатися одному
місці --- в самому методі.

### Переваги

-   Замість пачки різноманітних параметрів ви бачите один об'єкт із
    зрозумілою назвою.

-   Якщо методу знадобляться ще якісь дані з об'єкта, не треба буде
    переписувати усі місця, де викликається цей метод, а тільки нутрощі
    самого методу.

### Недоліки

-   В деяких випадках після такого перетворення метод втрачає в
    універсальності, оскільки він міг отримувати дані з безлічі різних
    джерел, а в результаті рефакторингу ми обмежуємо круг його
    застосування тільки для об'єктів з певним інтерфейсом.

### Порядок рефакторингу

1.  Створіть параметр в методі для об'єкта, з якого можна отримати
    потрібні значення.

2.  Тепер починайте по одному видаляти старі параметри з методу,
    замінюючи їх в коді викликами відповідних методів об'єкта-параметра.
    Тестуйте програму після кожної заміни параметра.

3.  Видаліть код отримання значень з об'єкта-параметра, який стояв перед
    викликом методу.

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

::::::::::::::::: row
::::::: {.col-xs-12 .col-sm-6 .col-lg-6}
### Схожі рефакторинги

:::: dl
::: dt
[Заміна параметрів об\'єктом](introduce-parameter-object.html)
:::
::::

:::: dl
::: dt
[Заміна параметра викликом
методу](replace-parameter-with-method-call.html)
:::
::::
:::::::

::::::::::: {.col-xs-12 .col-sm-6 .col-lg-6}
### Бореться з запахом

:::: dl
::: dt
[Одержимість елементарними типами](smells/primitive-obsession.html)
:::
::::

:::: dl
::: dt
[Довгий список параметрів](smells/long-parameter-list.html)
:::
::::

:::: dl
::: dt
[Довгий метод](smells/long-method.html)
:::
::::

:::: dl
::: dt
[Групи даних](smells/data-clumps.html)
:::
::::
:::::::::::
:::::::::::::::::

::: next
#### Читаємо далі

[Заміна параметра викликом методу []{.fa
.fa-arrow-right}](replace-parameter-with-method-call.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Повернутись назад

[[]{.fa .fa-arrow-left} Заміна параметра набором спеціалізованих
методів](replace-parameter-with-explicit-methods.html){.btn .btn-default
rel="prev"}
:::
::::::::::::::::::::::::::::::::::::::::::::::::::::::

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
:::::::::::::::::::::::::::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::
