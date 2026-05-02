::::::::::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Рефакторинг](refactoring.html){.type} /
[Прийоми рефакторингу](refactoring/techniques.html){.type} / [Спрощення
умовних
виразів](refactoring/techniques/simplifying-conditional-expressions.html){.type}
:::

# Об\'єднання умовних операторів {#обєднання-умовних-операторів .title}

::: aka
Також відомий як: [Consolidate Conditional
Expression]{style="display:inline-block"}
:::

::::: before-after-container
::: before
### Проблема

У вас є декілька умовних операторів, що ведуть до однакового результату
або дії.
:::

::: after
### Рішення

Об'єднайте всі умови в одному умовному операторі.
:::
:::::

:::::::::::::::::::::::::::: examples
::::::: {.before-after .before-after-container .java data-lang="java"}
:::: before
::: ba
До
:::

``` {.code lang="java"}
double disabilityAmount() {
  if (seniority < 2) {
    return 0;
  }
  if (monthsDisabled > 12) {
    return 0;
  }
  if (isPartTime) {
    return 0;
  }
  // Compute the disability amount.
  // ...
}
```
::::

:::: after
::: ba
Після
:::

``` {.code lang="java"}
double disabilityAmount() {
  if (isNotEligibleForDisability()) {
    return 0;
  }
  // Compute the disability amount.
  // ...
}
```
::::
:::::::

::::::: {.before-after .before-after-container .csharp data-lang="csharp"}
:::: before
::: ba
До
:::

``` {.code lang="csharp"}
double DisabilityAmount() 
{
  if (seniority < 2) 
  {
    return 0;
  }
  if (monthsDisabled > 12) 
  {
    return 0;
  }
  if (isPartTime) 
  {
    return 0;
  }
  // Compute the disability amount.
  // ...
}
```
::::

:::: after
::: ba
Після
:::

``` {.code lang="csharp"}
double DisabilityAmount()
{
  if (IsNotEligibleForDisability())
  {
    return 0;
  }
  // Compute the disability amount.
  // ...
}
```
::::
:::::::

::::::: {.before-after .before-after-container .php data-lang="php"}
:::: before
::: ba
До
:::

``` {.code lang="php"}
function disabilityAmount() {
  if ($this->seniority < 2) {
    return 0;
  }
  if ($this->monthsDisabled > 12) {
    return 0;
  }
  if ($this->isPartTime) {
    return 0;
  }
  // compute the disability amount
  ...
```
::::

:::: after
::: ba
Після
:::

``` {.code lang="php"}
function disabilityAmount() {
  if ($this->isNotEligibleForDisability()) {
    return 0;
  }
  // compute the disability amount
  ...
```
::::
:::::::

::::::: {.before-after .before-after-container .python data-lang="python"}
:::: before
::: ba
До
:::

``` {.code lang="python"}
def disabilityAmount():
    if seniority < 2:
        return 0
    if monthsDisabled > 12:
        return 0
    if isPartTime:
        return 0
    # Compute the disability amount.
    # ...
```
::::

:::: after
::: ba
Після
:::

``` {.code lang="python"}
def disabilityAmount():
    if isNotEligibleForDisability():
        return 0
    # Compute the disability amount.
    # ...
```
::::
:::::::

::::::: {.before-after .before-after-container .typescript data-lang="typescript"}
:::: before
::: ba
До
:::

``` {.code lang="typescript"}
disabilityAmount(): number {
  if (seniority < 2) {
    return 0;
  }
  if (monthsDisabled > 12) {
    return 0;
  }
  if (isPartTime) {
    return 0;
  }
  // Compute the disability amount.
  // ...
}
```
::::

:::: after
::: ba
Після
:::

``` {.code lang="typescript"}
disabilityAmount(): number {
  if (isNotEligibleForDisability()) {
    return 0;
  }
  // Compute the disability amount.
  // ...
}
```
::::
:::::::
::::::::::::::::::::::::::::

### Причини рефакторингу

Код містить безліч операторів, що чергуються і виконують однакові дії.
Причина розділення операторів неочевидна.

Головна мета об'єднання операторів --- відокремити умову оператора в
окремий метод, спростивши його розуміння.

### Переваги

-   Прибирає дублювання управляючого коду. Об'єднання безлічі умовних
    операторів, що ведуть до однієї мети, допомагає показати, що
    насправді ви робите тільки одну складну перевірку, що веде до однієї
    спільної дії.

-   Об'єднавши усі оператори в одному, ви зможете виділити цю складну
    умову в новий метод з назвою, що відображає суть цього виразу.

### Порядок рефакторингу

Перш ніж здійснювати рефакторинг, переконайтеся, що в умовах операторів
немає «побічних ефектів», або, іншими словами, ці умови не модифікують
щось, а тільки повертають значення. Побічні ефекти можуть бути і в коді,
який виконується усередині самого оператора. Наприклад, за результатами
умови, щось додається до змінної.

1.  Об'єднайте декілька умов в одному за допомогою операторів `і` та
    `або`. Об'єднання операторів зазвичай робиться за таким правилом:

    -   Вкладені умови з'єднуються за допомогою оператора `і`.

    -   Умови, які йдуть одна за одною, з'єднуються за допомогою
        оператора `або`.

2.  [Відокремте метод](extract-method.html) від умови оператора і
    назвіть його так, щоб він відображав суть виразу, який перевірявся.

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

:::::: row
::::: {.col-xs-12 .col-sm-6 .col-lg-12}
### Бореться з запахом

:::: dl
::: dt
[Дублювання коду](smells/duplicate-code.html)
:::
::::
:::::
::::::

::: next
#### Читаємо далі

[Об\'єднання фрагментів, що дублюються, в умовних операторах []{.fa
.fa-arrow-right}](consolidate-duplicate-conditional-fragments.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Повернутись назад

[[]{.fa .fa-arrow-left} Розподіл умовного
оператора](decompose-conditional.html){.btn .btn-default rel="prev"}
:::
:::::::::::::::::::::::::::::::::::::::::::

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
::::::::::::::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::::::::::::::
