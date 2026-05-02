:::::::::::::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Рефакторинг](refactoring.html){.type} /
[Прийоми рефакторингу](refactoring/techniques.html){.type} / [Складання
методів](refactoring/techniques/composing-methods.html){.type}
:::

# Видалення присвоювань параметрам {#видалення-присвоювань-параметрам .title}

::: aka
Також відомий як: [Remove Assignments to
Parameters]{style="display:inline-block"}
:::

::::: before-after-container
::: before
### Проблема

Параметру методу присвоюється якесь значення.
:::

::: after
### Рішення

Замість параметра скористайтеся новою локальною змінною.
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
Після
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
Після
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
Після
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
Після
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
Після
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

### Причини рефакторингу

Причини проведення цього рефакторингу такі ж, як і при [розщеплюванні
змінної](split-temporary-variable.html), але в даному випадку йдеться
про параметр, а не про локальну змінну.

По-перше, якщо параметр передається за посиланням, то після зміни його
значення всередині методу, воно передається аргументу, який подавався на
виклик цього методу. Дуже часто це відбувається випадково і призводить
до сумних наслідків. Навіть якщо у вашій мові програмування параметри
зазвичай передаються за значенням, а не за посиланням, сам код може
викликати складнощі в тих, хто звик вважати інакше.

По-друге, багаторазові присвоєння різних значень параметру призводять до
того, що вам стає складно зрозуміти, які саме дані повинні знаходитися в
параметрі в певний момент часу. Проблема посилюється, якщо ваш параметр
і те, що він повинен зберігати, описані в документації, але фактично
його значення може не співпадати з очікуваним усередині методу.

### Переваги

-   Кожен елемент програми повинен відповідати тільки за одну річ. Це
    значно спрощує підтримку коду в майбутньому, оскільки ви можете
    спокійно замінити цей елемент, не побоюючись побічних ефектів.

-   Цей рефакторинг допомагає в подальшому [виділити ділянки коду, що
    повторюються, в окремі методи](extract-method.html).

### Порядок рефакторингу

1.  Створіть локальну змінну і присвойте їй початкове значення вашого
    параметру.

2.  В усьому коді методу після цього рядка замініть використання
    параметра вашою локальною змінною.

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

::::::::: row
::::: {.col-xs-12 .col-sm-6 .col-lg-6}
### Схожі рефакторинги

:::: dl
::: dt
[Розщеплювання змінної](split-temporary-variable.html)
:::
::::
:::::

::::: {.col-xs-12 .col-sm-6 .col-lg-6}
### Домогає іншим рефакторингам

:::: dl
::: dt
[Відокремлення методу](extract-method.html)
:::
::::
:::::
:::::::::

::: next
#### Читаємо далі

[Заміна методу об\'єктом методів []{.fa
.fa-arrow-right}](replace-method-with-method-object.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Повернутись назад

[[]{.fa .fa-arrow-left} Розщеплювання
змінної](split-temporary-variable.html){.btn .btn-default rel="prev"}
:::
::::::::::::::::::::::::::::::::::::::::::::::

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
:::::::::::::::::::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::::::::::::::::::
