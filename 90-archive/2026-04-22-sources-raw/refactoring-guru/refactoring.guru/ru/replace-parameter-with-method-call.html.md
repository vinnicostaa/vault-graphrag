::::::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Рефакторинг](refactoring.html){.type} /
[Приёмы](refactoring/techniques.html){.type} / [Упрощение вызовов
методов](refactoring/techniques/simplifying-method-calls.html){.type}
:::

# Замена параметра вызовом метода {#замена-параметра-вызовом-метода .title}

::: aka
Также известен как: [Replace Parameter with Method
Call]{style="display:inline-block"}
:::

::::: before-after-container
::: before
### Проблема

Вызываем метод и передаем его результаты как параметры другого метода.
При этом значение параметров могли бы быть получены и внутри вызываемого
метода.
:::

::: after
### Решение

Вместо передачи значения через параметры метода, попробуйте переместить
код получения значения внутрь самого метода.
:::
:::::

:::::::::::::::::::::::::::: examples
::::::: {.before-after .before-after-container .java data-lang="java"}
:::: before
::: ba
До
:::

``` {.code lang="java"}
int basePrice = quantity * itemPrice;
double seasonDiscount = this.getSeasonalDiscount();
double fees = this.getFees();
double finalPrice = discountedPrice(basePrice, seasonDiscount, fees);
```
::::

:::: after
::: ba
После
:::

``` {.code lang="java"}
int basePrice = quantity * itemPrice;
double finalPrice = discountedPrice(basePrice);
```
::::
:::::::

::::::: {.before-after .before-after-container .csharp data-lang="csharp"}
:::: before
::: ba
До
:::

``` {.code lang="csharp"}
int basePrice = quantity * itemPrice;
double seasonDiscount = this.GetSeasonalDiscount();
double fees = this.GetFees();
double finalPrice = DiscountedPrice(basePrice, seasonDiscount, fees);
```
::::

:::: after
::: ba
После
:::

``` {.code lang="csharp"}
int basePrice = quantity * itemPrice;
double finalPrice = DiscountedPrice(basePrice);
```
::::
:::::::

::::::: {.before-after .before-after-container .php data-lang="php"}
:::: before
::: ba
До
:::

``` {.code lang="php"}
$basePrice = $this->quantity * $this->itemPrice;
$seasonDiscount = $this->getSeasonalDiscount();
$fees = $this->getFees();
$finalPrice = $this->discountedPrice($basePrice, $seasonDiscount, $fees);
```
::::

:::: after
::: ba
После
:::

``` {.code lang="php"}
$basePrice = $this->quantity * $this->itemPrice;
$finalPrice = $this->discountedPrice($basePrice);
```
::::
:::::::

::::::: {.before-after .before-after-container .python data-lang="python"}
:::: before
::: ba
До
:::

``` {.code lang="python"}
basePrice = quantity * itemPrice
seasonalDiscount = self.getSeasonalDiscount()
fees = self.getFees()
finalPrice = discountedPrice(basePrice, seasonalDiscount, fees)
```
::::

:::: after
::: ba
После
:::

``` {.code lang="python"}
basePrice = quantity * itemPrice
finalPrice = discountedPrice(basePrice)
```
::::
:::::::

::::::: {.before-after .before-after-container .typescript data-lang="typescript"}
:::: before
::: ba
До
:::

``` {.code lang="typescript"}
let basePrice = quantity * itemPrice;
const seasonDiscount = this.getSeasonalDiscount();
const fees = this.getFees();
const finalPrice = discountedPrice(basePrice, seasonDiscount, fees);
```
::::

:::: after
::: ba
После
:::

``` {.code lang="typescript"}
let basePrice = quantity * itemPrice;
let finalPrice = discountedPrice(basePrice);
```
::::
:::::::
::::::::::::::::::::::::::::

### Причины рефакторинга

В длинном списке параметров зачастую крайне сложно ориентироваться.
Кроме того, вызовы таких методов часто превращаются в целую вереницу
вычислений значений, которые будут передаваться в метод. Вот почему если
значения параметра может быть вычислено при помощи вызова какого-то
метода, это следует сделать внутри самого метода, а от параметра
избавиться.

### Достоинства

-   Избавляемся от лишних параметров, упрощая вызовы методов. Эти
    параметры зачастую создаются как задел на будущее (которое может так
    и не наступить).

### Недостатки

-   Параметр может понадобиться завтра для каких-то других целей и метод
    придётся переписать.

### Порядок рефакторинга

1.  Убедитесь, что код получения значения не использует параметров из
    текущего метода. Это важно, так как параметры текущего метода будут
    недоступны внутри другого метода, из-за чего перенос станет
    невозможен.

2.  Если код получения значения сложнее, чем один вызов какого-то метода
    или функции, примените [извлечение метода](extract-method.html),
    чтобы выделить этот код в новый метод и сделать вызов простым.

3.  В коде главного метода замените все обращения к заменяемому
    параметру вызовами метода получения значения.

4.  Используйте [удаление параметра](remove-parameter.html), чтобы
    удалить неиспользуемый теперь параметр.

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

::: next
#### Читаем дальше

[Замена параметров объектом []{.fa
.fa-arrow-right}](introduce-parameter-object.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Передача всего
объекта](preserve-whole-object.html){.btn .btn-default rel="prev"}
:::
:::::::::::::::::::::::::::::::::::::::

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
::::::::::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::::::::::
