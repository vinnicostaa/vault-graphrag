::::::::::::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Рефакторинг](refactoring.html){.type} /
[Приёмы](refactoring/techniques.html){.type} / [Составление
методов](refactoring/techniques/composing-methods.html){.type}
:::

# Встраивание переменной {#встраивание-переменной .title}

::: aka
Также известен как: [Inline Temp]{style="display:inline-block"}
:::

::::: before-after-container
::: before
### Проблема

У вас есть временная переменная, которой присваивается результат
простого выражения (и больше ничего).
:::

::: after
### Решение

Замените обращения к переменной этим выражением.
:::
:::::

:::::::::::::::::::::::::::: examples
::::::: {.before-after .before-after-container .java data-lang="java"}
:::: before
::: ba
До
:::

``` {.code lang="java"}
boolean hasDiscount(Order order) {
  double basePrice = order.basePrice();
  return basePrice > 1000;
}
```
::::

:::: after
::: ba
После
:::

``` {.code lang="java"}
boolean hasDiscount(Order order) {
  return order.basePrice() > 1000;
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
bool HasDiscount(Order order)
{
  double basePrice = order.BasePrice();
  return basePrice > 1000;
}
```
::::

:::: after
::: ba
После
:::

``` {.code lang="csharp"}
bool HasDiscount(Order order)
{
  return order.BasePrice() > 1000;
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
$basePrice = $anOrder->basePrice();
return $basePrice > 1000;
```
::::

:::: after
::: ba
После
:::

``` {.code lang="php"}
return $anOrder->basePrice() > 1000;
```
::::
:::::::

::::::: {.before-after .before-after-container .python data-lang="python"}
:::: before
::: ba
До
:::

``` {.code lang="python"}
def hasDiscount(order):
    basePrice = order.basePrice()
    return basePrice > 1000
```
::::

:::: after
::: ba
После
:::

``` {.code lang="python"}
def hasDiscount(order):
    return order.basePrice() > 1000
```
::::
:::::::

::::::: {.before-after .before-after-container .typescript data-lang="typescript"}
:::: before
::: ba
До
:::

``` {.code lang="typescript"}
hasDiscount(order: Order): boolean {
  let basePrice: number = order.basePrice();
  return basePrice > 1000;
}
```
::::

:::: after
::: ba
После
:::

``` {.code lang="typescript"}
hasDiscount(order: Order): boolean {
  return order.basePrice() > 1000;
}
```
::::
:::::::
::::::::::::::::::::::::::::

### Причины рефакторинга

Встраивание локальной переменной почти всегда используется как часть
[замены переменной вызовом метода](replace-temp-with-query.html) или для
облегчения [извлечения метода](extract-method.html).

### Достоинства

-   Сам по себе данный рефакторинг не несёт почти никакой пользы. Тем не
    менее, если переменной присваивается результат выполнения какого-то
    метода, у вас есть возможность немного улучшить читабельность
    программы, избавившись от лишней переменной.

### Недостатки

-   Иногда с виду бесполезные временные переменные служат для
    кеширования, то есть сохранения результата какой-то дорогостоящей
    операции, который будет в ходе работы использован несколько раз
    повторно. Перед тем как осуществлять рефакторинг, убедитесь, что в
    вашем случае это не так.

### Порядок рефакторинга

1.  Найдите все места, где используется переменная, и замените их
    выражением, которое ей присваивалось.

2.  Удалите объявление переменной и строку присваивания ей значения.

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

:::::::: row
::::::: {.col-xs-12 .col-sm-6 .col-lg-12}
### Помогает рефакторингу

:::: dl
::: dt
[Замена переменной вызовом метода](replace-temp-with-query.html)
:::
::::

:::: dl
::: dt
[Извлечение метода](extract-method.html)
:::
::::
:::::::
::::::::

::: next
#### Читаем дальше

[Замена переменной вызовом метода []{.fa
.fa-arrow-right}](replace-temp-with-query.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Извлечение
переменной](extract-variable.html){.btn .btn-default rel="prev"}
:::
:::::::::::::::::::::::::::::::::::::::::::::

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
::::::::::::::::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::::::::::::::::
