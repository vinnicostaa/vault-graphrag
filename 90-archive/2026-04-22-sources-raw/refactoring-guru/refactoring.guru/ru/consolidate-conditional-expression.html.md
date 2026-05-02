::::::::::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Рефакторинг](refactoring.html){.type} /
[Приёмы](refactoring/techniques.html){.type} / [Упрощение условных
выражений](refactoring/techniques/simplifying-conditional-expressions.html){.type}
:::

# Объединение условных операторов {#объединение-условных-операторов .title}

::: aka
Также известен как: [Consolidate Conditional
Expression]{style="display:inline-block"}
:::

::::: before-after-container
::: before
### Проблема

У вас есть несколько условных операторов, ведущих к одинаковому
результату или действию.
:::

::: after
### Решение

Объедините все условия в одном условном операторе.
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
После
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
После
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
После
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
После
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
После
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

### Причины рефакторинга

Код содержит множество чередующихся операторов, которые выполняют
одинаковые действия. Причина разделения операторов неочевидна.

Главная цель объединения операторов --- извлечь условие оператора в
отдельный метод, упростив его понимание.

### Достоинства

-   Убирает дублирование управляющего кода. Объединение множества
    условных операторов, ведущих к одной цели, помогает показать, что на
    самом деле вы делаете только одну сложную проверку, ведущую к одному
    общему действию.

-   Объединив все операторы в одном, вы позволяете выделить это сложное
    условие в новый метод с названием, отражающим суть этого выражения.

### Порядок рефакторинга

Перед тем как осуществлять рефакторинг, убедитесь, что в условиях
операторов нет «побочных эффектов», или, другими словами, они не
модифицируют что-то, а только возвращают значения. Побочные эффекты
могут быть и в коде, который выполняется внутри самого оператора.
Например, по результатам условия, что-то добавляется к переменной.

1.  Объедините множество условий в одном с помощью операторов `и` и
    `или`. Объединение операторов обычно следует такому правилу:

    -   Вложенные условия соединяются с помощью оператора `и`.

    -   Условия, следующие друг за другом, соединяются с помощью
        оператора `или`.

2.  [Извлеките метод](extract-method.html) из условия оператора и
    назовите его так, чтобы он отражал суть проверяемого выражения.

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

[Объединение дублирующихся фрагментов в условных операторах []{.fa
.fa-arrow-right}](consolidate-duplicate-conditional-fragments.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Разбиение условного
оператора](decompose-conditional.html){.btn .btn-default rel="prev"}
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
