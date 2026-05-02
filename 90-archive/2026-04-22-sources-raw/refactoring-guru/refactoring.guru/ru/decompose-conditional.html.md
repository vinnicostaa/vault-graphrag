::::::::::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Рефакторинг](refactoring.html){.type} /
[Приёмы](refactoring/techniques.html){.type} / [Упрощение условных
выражений](refactoring/techniques/simplifying-conditional-expressions.html){.type}
:::

# Разбиение условного оператора {#разбиение-условного-оператора .title}

::: aka
Также известен как: [Decompose
Conditional]{style="display:inline-block"}
:::

::::: before-after-container
::: before
### Проблема

У вас есть сложный условный оператор (`if-then`/`else` или `switch`).
:::

::: after
### Решение

Выделите в отдельные методы все сложные части оператора: условие, `then`
и `else`.
:::
:::::

:::::::::::::::::::::::::::: examples
::::::: {.before-after .before-after-container .java data-lang="java"}
:::: before
::: ba
До
:::

``` {.code lang="java"}
if (date.before(SUMMER_START) || date.after(SUMMER_END)) {
  charge = quantity * winterRate + winterServiceCharge;
}
else {
  charge = quantity * summerRate;
}
```
::::

:::: after
::: ba
После
:::

``` {.code lang="java"}
if (isSummer(date)) {
  charge = summerCharge(quantity);
}
else {
  charge = winterCharge(quantity);
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
if (date < SUMMER_START || date > SUMMER_END) 
{
  charge = quantity * winterRate + winterServiceCharge;
}
else 
{
  charge = quantity * summerRate;
}
```
::::

:::: after
::: ba
После
:::

``` {.code lang="csharp"}
if (isSummer(date))
{
  charge = SummerCharge(quantity);
}
else 
{
  charge = WinterCharge(quantity);
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
if ($date->before(SUMMER_START) || $date->after(SUMMER_END)) {
  $charge = $quantity * $winterRate + $winterServiceCharge;
} else {
  $charge = $quantity * $summerRate;
}
```
::::

:::: after
::: ba
После
:::

``` {.code lang="php"}
if (isSummer($date)) {
  $charge = summerCharge($quantity);
} else {
  $charge = winterCharge($quantity);
}
```
::::
:::::::

::::::: {.before-after .before-after-container .python data-lang="python"}
:::: before
::: ba
До
:::

``` {.code lang="python"}
if date.before(SUMMER_START) or date.after(SUMMER_END):
    charge = quantity * winterRate + winterServiceCharge
else:
    charge = quantity * summerRate
```
::::

:::: after
::: ba
После
:::

``` {.code lang="python"}
if isSummer(date):
    charge = summerCharge(quantity)
else:
    charge = winterCharge(quantity)
```
::::
:::::::

::::::: {.before-after .before-after-container .typescript data-lang="typescript"}
:::: before
::: ba
До
:::

``` {.code lang="typescript"}
if (date.before(SUMMER_START) || date.after(SUMMER_END)) {
  charge = quantity * winterRate + winterServiceCharge;
}
else {
  charge = quantity * summerRate;
}
```
::::

:::: after
::: ba
После
:::

``` {.code lang="typescript"}
if (isSummer(date)) {
  charge = summerCharge(quantity);
}
else {
  charge = winterCharge(quantity);
}
```
::::
:::::::
::::::::::::::::::::::::::::

### Причины рефакторинга

Чем длиннее кусок кода, тем сложнее понять, что он делает. Все
усложняется ещё больше, когда код щедро приправлен условными
операторами:

-   пока вы разберётесь в том, что делает код в `then`, вы забываете,
    какое условие стояло в операторе;

-   пока вы разбираетесь с `else`, вы забываете, что делал код в `then`.

### Достоинства

-   Извлекая код условного оператора в методы с понятным названием, вы
    упрощаете жизнь тому, кто впоследствии будет этот код поддерживать
    (зачастую вам самим через месяц или два).

-   Кстати, этот рефакторинг применим и для коротких выражений в
    условиях оператора. Строка `isSalaryDay()` куда наглядней опишет то,
    что она делает, чем код сравнения дат.

### Порядок рефакторинга

1.  Выделите условие в отдельный метод с помощью [выделения
    метода](extract-method.html).

2.  Повторите выделение для `then` и `else` части оператора.

::::: {.prom .banner-content .banner .banner-striped .banner-with-image data-id="Ref: Part of the ebook" creative-id="animated-ru" position="content_bottom"}
::: banner-image
Your browser does not support HTML video.
:::

::: banner-text
### Устали читать? {#устали-читать .title}

Сбегайте за подушкой, у нас тут контента на [7 часов]{.blue} чтения.

Или попробуйте наш интерактивный курс. Он гораздо более интересный, чем
банальный текст.

[ Узнать больше...](refactoring/course.html){.btn .btn-secondary}
:::
:::::

:::::: row
::::: {.col-xs-12 .col-sm-6 .col-lg-12}
### Борется с запахом

:::: dl
::: dt
[Длинный метод](smells/long-method.html)
:::
::::
:::::
::::::

::: next
#### Читаем дальше

[Объединение условных операторов []{.fa
.fa-arrow-right}](consolidate-conditional-expression.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Упрощение условных
выражений](refactoring/techniques/simplifying-conditional-expressions.html){.btn
.btn-default rel="prev"}
:::
:::::::::::::::::::::::::::::::::::::::::::

:::::: {.prom .banner-sidebar .banner-removable .banner-removable-refactoring data-id="Ref: Sidebar" creative-id="standard-sidebar-ru" position="sidebar"}
::::: ban
::: {.image .product-image}
[Скидки!]{.banner-discount}
[![](../images/refactoring/course/course-cover-rua60d.jpg?id=e9d6b4015f4c6c48cf06f7479874d8d7){width="300"
height="300" loading="lazy"}](refactoring/course.html)
:::

::: {.banner-text .banner-text-ru}
Этот рефакторинг --- малая часть интерактивного **онлайн курса по
рефакторингу**.

[ Узнать больше...](refactoring/course.html){.btn .btn-secondary
.btn-block}
:::
:::::
::::::
::::::::::::::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::::::::::::::
