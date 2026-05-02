::::::::::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Рефакторинг](refactoring.html){.type} /
[Прийоми рефакторингу](refactoring/techniques.html){.type} / [Спрощення
викликів
методів](refactoring/techniques/simplifying-method-calls.html){.type}
:::

# Заміна виключення перевіркою умови {#заміна-виключення-перевіркою-умови .title}

::: aka
Також відомий як: [Replace Exception with
Test]{style="display:inline-block"}
:::

::::: before-after-container
::: before
### Проблема

Ви викидаєте виключення там, де можна було б обійтися простою перевіркою
умови.
:::

::: after
### Рішення

Замініть викидання виключення перевіркою цієї умови.
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
Після
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
Після
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
Після
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
Після
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
Після
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

### Причини рефакторингу

Виключення повинні використовуватися для обробки позаштатної поведінки,
пов'язаної з несподіваною помилкою. Вони не повинні служити заміною
перевіркам виконання умов. Якщо виключення можна уникнути, просто
перевіривши якусь умову перед виконанням дії, то варто так і зробити.
Виключення слід приберегти для справжніх помилок.

Наприклад, ви зайшли на мінне поле і там підірвалися, викликавши
виключення; виключення успішно обробилося і вас винесло за межі мінного
поля. Замість цього можна б було просто прочитати знак перед мінним
полем і обійти його іншою дорогою.

### Переваги

-   Простий умовний оператор іноді може бути очевиднішим за блок обробки
    виключення.

### Порядок рефакторингу

1.  Створіть умовний оператор для граничного випадку і розташуйте його
    перед `try`/`catch` блоком.

2.  Перемістіть код з `catch`-секції всередину цього умовного оператора.

3.  У `catch`-секції розташцуйте код викидання звичайного безіменного
    виключення і запустіть усі тести.

4.  Якщо ніяких виключень не було викинуто під час тестів, позбавтеся
    від оператора `try`/`catch`.

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
### Схожі рефакторинги

:::: dl
::: dt
[Заміна коду помилки
виключенням](replace-error-code-with-exception.html)
:::
::::
:::::
::::::

::: next
#### Читаємо далі

[Задачі узагальнення об\'єктів []{.fa
.fa-arrow-right}](refactoring/techniques/dealing-with-generalization.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Повернутись назад

[[]{.fa .fa-arrow-left} Заміна коду помилки
виключенням](replace-error-code-with-exception.html){.btn .btn-default
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
