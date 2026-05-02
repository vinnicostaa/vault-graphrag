::::::::::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Рефакторинг](refactoring.html){.type} /
[Приёмы](refactoring/techniques.html){.type} / [Упрощение вызовов
методов](refactoring/techniques/simplifying-method-calls.html){.type}
:::

# Замена исключения проверкой условия {#замена-исключения-проверкой-условия .title}

::: aka
Также известен как: [Replace Exception with
Test]{style="display:inline-block"}
:::

::::: before-after-container
::: before
### Проблема

Вы выбрасываете исключение там, где можно было бы обойтись простой
проверкой условия.
:::

::: after
### Решение

Замените выбрасывание исключения проверкой этого условия.
:::
:::::

:::::::::::::::::::::::::::: examples
::::::: {.before-after .before-after-container .java data-lang="java"}
:::: before
::: ba
До
:::

``` {.code lang="java"}
double getValueForPeriod(int periodNumber) {
  try {
    return values[periodNumber];
  } catch (ArrayIndexOutOfBoundsException e) {
    return 0;
  }
}
```
::::

:::: after
::: ba
После
:::

``` {.code lang="java"}
double getValueForPeriod(int periodNumber) {
  if (periodNumber >= values.length) {
    return 0;
  }
  return values[periodNumber];
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
double GetValueForPeriod(int periodNumber) 
{
  try 
  {
    return values[periodNumber];
  } 
  catch (IndexOutOfRangeException e) 
  {
    return 0;
  }
}
```
::::

:::: after
::: ba
После
:::

``` {.code lang="csharp"}
double GetValueForPeriod(int periodNumber) 
{
  if (periodNumber >= values.Length) 
  {
    return 0;
  }
  return values[periodNumber];
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
function getValueForPeriod($periodNumber) {
  try {
    return $this->values[$periodNumber];
  } catch (ArrayIndexOutOfBoundsException $e) {
    return 0;
  }
}
```
::::

:::: after
::: ba
После
:::

``` {.code lang="php"}
function getValueForPeriod($periodNumber) {
  if ($periodNumber >= count($this->values)) {
    return 0;
  }
  return $this->values[$periodNumber];
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
def getValueForPeriod(periodNumber):
    try:
        return values[periodNumber]
    except IndexError:
        return 0
```
::::

:::: after
::: ba
После
:::

``` {.code lang="python"}
def getValueForPeriod(self, periodNumber):
    if periodNumber >= len(self.values):
        return 0
    return self.values[periodNumber]
```
::::
:::::::

::::::: {.before-after .before-after-container .typescript data-lang="typescript"}
:::: before
::: ba
До
:::

``` {.code lang="typescript"}
getValueForPeriod(periodNumber: number): number {
  try {
    return values[periodNumber];
  } catch (ArrayIndexOutOfBoundsException e) {
    return 0;
  }
}
```
::::

:::: after
::: ba
После
:::

``` {.code lang="typescript"}
getValueForPeriod(periodNumber: number): number {
  if (periodNumber >= values.length) {
    return 0;
  }
  return values[periodNumber];
}
```
::::
:::::::
::::::::::::::::::::::::::::

### Причины рефакторинга

Исключения должны использоваться для обработки внештатного поведения,
связанного с неожиданной ошибкой. Они не должны служить заменой
проверкам выполнения условий. Если исключения можно избежать, просто
проверив какое-то условие перед выполнением действия, то стоит так и
сделать. Исключения следует приберечь для настоящих ошибок.

Например, вы зашли на минное поле и там подорвались, вызвав исключение;
исключение успешно обработалось и вас вынесло за пределы минного поля.
Вместо этого можно бы было просто прочитать указатель перед минным полем
и обойти его другой дорогой.

### Достоинства

-   Простой условный оператор иногда может быть очевиднее блока
    обработки исключения.

### Порядок рефакторинга

1.  Создайте условный оператор для граничного случая и поместите его
    перед `try`/`catch` блоком.

2.  Переместите код из `catch`-секции внутрь этого условного оператора.

3.  В `catch`-секции поставьте код выбрасывания обычного безымянного
    исключения и запустите все тесты.

4.  Если никаких исключений не было выброшено во время тестов,
    избавьтесь от оператора `try`/`catch`.

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
### Родственные рефакторинги

:::: dl
::: dt
[Замена кода ошибки исключением](replace-error-code-with-exception.html)
:::
::::
:::::
::::::

::: next
#### Читаем дальше

[Решение задач обобщения []{.fa
.fa-arrow-right}](refactoring/techniques/dealing-with-generalization.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Замена кода ошибки
исключением](replace-error-code-with-exception.html){.btn .btn-default
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
