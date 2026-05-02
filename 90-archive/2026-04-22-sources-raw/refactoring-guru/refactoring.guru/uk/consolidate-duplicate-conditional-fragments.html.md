::::::::::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Рефакторинг](refactoring.html){.type} /
[Прийоми рефакторингу](refactoring/techniques.html){.type} / [Спрощення
умовних
виразів](refactoring/techniques/simplifying-conditional-expressions.html){.type}
:::

# Об\'єднання фрагментів, що дублюються, в умовних операторах {#обєднання-фрагментів-що-дублюються-в-умовних-операторах .title}

::: aka
Також відомий як: [Consolidate Duplicate Conditional
Fragments]{style="display:inline-block"}
:::

::::: before-after-container
::: before
### Проблема

Однаковий фрагмент коду знаходиться в усіх гілках умовного оператора.
:::

::: after
### Рішення

Винесіть його за рамки оператора.
:::
:::::

:::::::::::::::::::::::::::: examples
::::::: {.before-after .before-after-container .java data-lang="java"}
:::: before
::: ba
До
:::

``` {.code lang="java"}
if (isSpecialDeal()) {
  total = price * 0.95;
  send();
}
else {
  total = price * 0.98;
  send();
}
```
::::

:::: after
::: ba
Після
:::

``` {.code lang="java"}
if (isSpecialDeal()) {
  total = price * 0.95;
}
else {
  total = price * 0.98;
}
send();
```
::::
:::::::

::::::: {.before-after .before-after-container .csharp data-lang="csharp"}
:::: before
::: ba
До
:::

``` {.code lang="csharp"}
if (IsSpecialDeal()) 
{
  total = price * 0.95;
  Send();
}
else 
{
  total = price * 0.98;
  Send();
}
```
::::

:::: after
::: ba
Після
:::

``` {.code lang="csharp"}
if (IsSpecialDeal())
{
  total = price * 0.95;
}
else
{
  total = price * 0.98;
}
Send();
```
::::
:::::::

::::::: {.before-after .before-after-container .php data-lang="php"}
:::: before
::: ba
До
:::

``` {.code lang="php"}
if (isSpecialDeal()) {
  $total = $price * 0.95;
  send();
} else {
  $total = $price * 0.98;
  send();
}
```
::::

:::: after
::: ba
Після
:::

``` {.code lang="php"}
if (isSpecialDeal()) {
  $total = $price * 0.95;
} else {
  $total = $price * 0.98;
}
send();
```
::::
:::::::

::::::: {.before-after .before-after-container .python data-lang="python"}
:::: before
::: ba
До
:::

``` {.code lang="python"}
if isSpecialDeal():
    total = price * 0.95
    send()
else:
    total = price * 0.98
    send()
```
::::

:::: after
::: ba
Після
:::

``` {.code lang="python"}
if isSpecialDeal():
    total = price * 0.95
else:
    total = price * 0.98
send()
```
::::
:::::::

::::::: {.before-after .before-after-container .typescript data-lang="typescript"}
:::: before
::: ba
До
:::

``` {.code lang="typescript"}
if (isSpecialDeal()) {
  total = price * 0.95;
  send();
}
else {
  total = price * 0.98;
  send();
}
```
::::

:::: after
::: ba
Після
:::

``` {.code lang="typescript"}
if (isSpecialDeal()) {
  total = price * 0.95;
}
else {
  total = price * 0.98;
}
send();
```
::::
:::::::
::::::::::::::::::::::::::::

### Причини рефакторингу

Дублюючий код знаходиться усередині усіх гілок умовного оператора.
Найчастіше це є результатом еволюції коду усередині гілок оператора,
особливо, якщо над кодом працювало декілька чоловік.

### Переваги

-   Вбиває дублювання коду.

### Порядок рефакторингу

1.  Якщо дублюючі ділянки знаходяться на початку гілок оператора,
    винесіть їх перед умовним оператором.

2.  Якщо такий код виконується в кінці гілок, помістить його після
    умовного оператора.

3.  Якщо дублюючий код розташован випадковим чином усередині гілок, вам
    треба спробувати пересунути його в початок або в кінець гілки,
    залежно від того, чи міняє він результат подальшого коду.

4.  Дублюючий фрагмент коду більший за один рядок можна спробувати
    [відокремити в новий метод](extract-method.html), якщо в цьому є
    сенс.

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

[Видалення керуючого флагу []{.fa
.fa-arrow-right}](remove-control-flag.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Повернутись назад

[[]{.fa .fa-arrow-left} Об\'єднання умовних
операторів](consolidate-conditional-expression.html){.btn .btn-default
rel="prev"}
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
