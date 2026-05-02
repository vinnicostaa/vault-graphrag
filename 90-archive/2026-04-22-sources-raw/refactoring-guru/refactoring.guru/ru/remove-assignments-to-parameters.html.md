:::::::::::::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Рефакторинг](refactoring.html){.type} /
[Приёмы](refactoring/techniques.html){.type} / [Составление
методов](refactoring/techniques/composing-methods.html){.type}
:::

# Удаление присваиваний параметрам {#удаление-присваиваний-параметрам .title}

::: aka
Также известен как: [Remove Assignments to
Parameters]{style="display:inline-block"}
:::

::::: before-after-container
::: before
### Проблема

Параметру метода присваивается какое-то значение.
:::

::: after
### Решение

Вместо параметра воспользуйтесь новой локальной переменной.
:::
:::::

:::::::::::::::::::::::::::: examples
::::::: {.before-after .before-after-container .java data-lang="java"}
:::: before
::: ba
До
:::

``` {.code lang="java"}
int discount(int inputVal, int quantity) {
  if (quantity > 50) {
    inputVal -= 2;
  }
  // ...
}
```
::::

:::: after
::: ba
После
:::

``` {.code lang="java"}
int discount(int inputVal, int quantity) {
  int result = inputVal;
  if (quantity > 50) {
    result -= 2;
  }
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
int Discount(int inputVal, int quantity) 
{
  if (quantity > 50) 
  {
    inputVal -= 2;
  }
  // ...
}
```
::::

:::: after
::: ba
После
:::

``` {.code lang="csharp"}
int Discount(int inputVal, int quantity) 
{
  int result = inputVal;
  
  if (quantity > 50) 
  {
    result -= 2;
  }
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
function discount($inputVal, $quantity) {
  if ($quantity > 50) {
    $inputVal -= 2;
  }
  ...
```
::::

:::: after
::: ba
После
:::

``` {.code lang="php"}
function discount($inputVal, $quantity) {
  $result = $inputVal;
  if ($quantity > 50) {
    $result -= 2;
  }
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
def discount(inputVal, quantity):
    if quantity > 50:
        inputVal -= 2
    # ...
```
::::

:::: after
::: ba
После
:::

``` {.code lang="python"}
def discount(inputVal, quantity):
    result = inputVal
    if quantity > 50:
        result -= 2
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
discount(inputVal: number, quantity: number): number {
  if (quantity > 50) {
    inputVal -= 2;
  }
  // ...
}
```
::::

:::: after
::: ba
После
:::

``` {.code lang="typescript"}
discount(inputVal: number, quantity: number): number {
  let result = inputVal;
  if (quantity > 50) {
    result -= 2;
  }
  // ...
}
```
::::
:::::::
::::::::::::::::::::::::::::

### Причины рефакторинга

Причины проведения этого рефакторинга такие же, как и при [расщеплении
переменной](split-temporary-variable.html), но в данном случае речь идёт
о параметре, а не о локальной переменной.

Во-первых, если параметр передаётся по ссылке, то после изменения его
значения внутри метода, оно передается аргументу, который подавался на
вызов этого метода. Очень часто это происходит случайно и приводит к
печальным последствиям. Даже если в вашем языке программирования
параметры обычно передаются по значению, а не по ссылке, сам код может
вызвать замешательство у тех, кто привык считать иначе.

Во-вторых, множественные присваивания разных значений параметру приводят
к тому, что вам становится сложно понять, какие именно данные должны
находиться в параметре в определенный момент времени. Проблема
усугубляется, если ваш параметр и то, что он должен хранить, описаны в
документации, но фактически, его значение может не совпадать с ожидаемым
внутри метода.

### Достоинства

-   Каждый элемент программы должен отвечать только за одну вещь. Это
    сильно упрощает поддержку кода в будущем, так как вы можете спокойно
    заменить этот элемент, не опасаясь побочных эффектов.

-   Этот рефакторинг помогает в дальнейшем [выделить повторяющиеся
    участки кода в отдельные методы](extract-method.html).

### Порядок рефакторинга

1.  Создайте локальную переменную и присвойте ей начальное значение
    вашего параметра.

2.  Во всем коде метода после этой строки замените использование
    параметра вашей локальной переменной.

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

::::::::: row
::::: {.col-xs-12 .col-sm-6 .col-lg-6}
### Родственные рефакторинги

:::: dl
::: dt
[Расщепление переменной](split-temporary-variable.html)
:::
::::
:::::

::::: {.col-xs-12 .col-sm-6 .col-lg-6}
### Помогает рефакторингу

:::: dl
::: dt
[Извлечение метода](extract-method.html)
:::
::::
:::::
:::::::::

::: next
#### Читаем дальше

[Замена метода объектом методов []{.fa
.fa-arrow-right}](replace-method-with-method-object.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Расщепление
переменной](split-temporary-variable.html){.btn .btn-default rel="prev"}
:::
::::::::::::::::::::::::::::::::::::::::::::::

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
:::::::::::::::::::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::::::::::::::::::
