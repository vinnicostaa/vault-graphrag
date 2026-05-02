::::::::::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Рефакторинг](refactoring.html){.type} /
[Приёмы](refactoring/techniques.html){.type} / [Упрощение условных
выражений](refactoring/techniques/simplifying-conditional-expressions.html){.type}
:::

# Объединение дублирующихся фрагментов в условных операторах {#объединение-дублирующихся-фрагментов-в-условных-операторах .title}

::: aka
Также известен как: [Consolidate Duplicate Conditional
Fragments]{style="display:inline-block"}
:::

::::: before-after-container
::: before
### Проблема

Одинаковый фрагмент кода находится во всех ветках условного оператора.
:::

::: after
### Решение

Вынесите его за рамки оператора.
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
После
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
После
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
После
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
После
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
После
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

### Причины рефакторинга

Дублирующий код находится внутри всех веток условного оператора.
Зачастую это является результатом эволюции кода внутри веток оператора,
тем более, если над кодом работало несколько человек.

### Достоинства

-   Убивает дублирование кода.

### Порядок рефакторинга

1.  Если дублирующие участки находятся вначале веток оператора, вынесите
    их перед условным оператором.

2.  Если такой код выполняется в конце веток, поместите его после
    условного оператора.

3.  Если дублирующий код расположен случайным образом внутри веток, вам
    нужно для начала попытаться передвинуть его в начало или в конец
    ветки, в зависимости от того, меняет ли он результат последующего
    кода.

4.  Дублирующий фрагмент кода более одной строки можно попытаться
    [извлечь в новый метод](extract-method.html), если в этом есть
    смысл.

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
[Дублирование кода](smells/duplicate-code.html)
:::
::::
:::::
::::::

::: next
#### Читаем дальше

[Удаление управляющего флага []{.fa
.fa-arrow-right}](remove-control-flag.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Объединение условных
операторов](consolidate-conditional-expression.html){.btn .btn-default
rel="prev"}
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
