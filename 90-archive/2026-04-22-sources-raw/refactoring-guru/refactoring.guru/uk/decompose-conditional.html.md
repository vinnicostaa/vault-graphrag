::::::::::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Рефакторинг](refactoring.html){.type} /
[Прийоми рефакторингу](refactoring/techniques.html){.type} / [Спрощення
умовних
виразів](refactoring/techniques/simplifying-conditional-expressions.html){.type}
:::

# Розподіл умовного оператора {#розподіл-умовного-оператора .title}

::: aka
Також відомий як: [Decompose Conditional]{style="display:inline-block"}
:::

::::: before-after-container
::: before
### Проблема

У вас є складний умовний оператор (`if-then`/`else` або `switch`).
:::

::: after
### Рішення

Виділіть в окремі методи усі складні частини оператора: умова, `then` і
`else`.
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
Після
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
Після
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
Після
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
Після
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
Після
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

### Причини рефакторингу

Чим довша частина коду, тим складніше зрозуміти, що вона робить. Все
ускладнюється ще більше, коли код щедро приправлений умовними
операторами:

-   доки ви зрозумієте, що робить код в `this`, ви можете забути, що за
    умова була в операторі;

-   доки ви розбираєтеся з `else`, ви забуваєте, що робив код в `this`.

### Переваги

-   Відокремлюючи код умовного оператора в методи із зрозумілою назвою,
    ви спрощуєте життя тому, хто згодом цей код буде підтримувати (дуже
    часто тією людиною будете ви самі через деякий час).

-   До речі, цей рефакторинг застосовується і для коротких виразів в
    умовах оператора. Рядок `isSalaryDay()` набагато наочніше опише те,
    що вона робить, ніж код порівняння дат.

### Порядок рефакторингу

1.  Виділіть умову в окремий метод за допомогою [відокремлення
    методу](extract-method.html).

2.  Повторіть видокремлення для частин оператора then і else.

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
[Довгий метод](smells/long-method.html)
:::
::::
:::::
::::::

::: next
#### Читаємо далі

[Об\'єднання умовних операторів []{.fa
.fa-arrow-right}](consolidate-conditional-expression.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Повернутись назад

[[]{.fa .fa-arrow-left} Спрощення умовних
виразів](refactoring/techniques/simplifying-conditional-expressions.html){.btn
.btn-default rel="prev"}
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
