::::::::::::::::::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Рефакторинг](refactoring.html){.type} /
[Приёмы](refactoring/techniques.html){.type} / [Упрощение вызовов
методов](refactoring/techniques/simplifying-method-calls.html){.type}
:::

# Замена параметра набором специализированных методов {#замена-параметра-набором-специализированных-методов .title}

::: aka
Также известен как: [Replace Parameter with Explicit
Methods]{style="display:inline-block"}
:::

::::: before-after-container
::: before
### Проблема

Метод разбит на части, каждая из которых выполняется в зависимости от
значения какого-то параметра.
:::

::: after
### Решение

Извлеките отдельные части метода в собственные методы и вызывайте их
вместо оригинального метода.
:::
:::::

:::::::::::::::::::::::::::: examples
::::::: {.before-after .before-after-container .java data-lang="java"}
:::: before
::: ba
До
:::

``` {.code lang="java"}
void setValue(String name, int value) {
  if (name.equals("height")) {
    height = value;
    return;
  }
  if (name.equals("width")) {
    width = value;
    return;
  }
  Assert.shouldNeverReachHere();
}
```
::::

:::: after
::: ba
После
:::

``` {.code lang="java"}
void setHeight(int arg) {
  height = arg;
}
void setWidth(int arg) {
  width = arg;
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
void SetValue(string name, int value) 
{
  if (name.Equals("height")) 
  {
    height = value;
    return;
  }
  if (name.Equals("width")) 
  {
    width = value;
    return;
  }
  Assert.Fail();
}
```
::::

:::: after
::: ba
После
:::

``` {.code lang="csharp"}
void SetHeight(int arg) 
{
  height = arg;
}
void SetWidth(int arg) 
{
  width = arg;
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
function setValue($name, $value) {
  if ($name === "height") {
    $this->height = $value;
    return;
  }
  if ($name === "width") {
    $this->width = $value;
    return;
  }
  assert("Should never reach here");
}
```
::::

:::: after
::: ba
После
:::

``` {.code lang="php"}
function setHeight($arg) {
  $this->height = $arg;
}
function setWidth($arg) {
  $this->width = $arg;
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
def output(self, name):
    if name == "banner"
        # Print the banner.
        # ...
    if name == "info"
        # Print the info.
        # ...
```
::::

:::: after
::: ba
После
:::

``` {.code lang="python"}
def outputBanner(self):
    # Print the banner.
    # ...

def outputInfo(self):
    # Print the info.
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
 setValue(name: string, value: number): void {
  if (name.equals("height")) {
    height = value;
    return;
  }
  if (name.equals("width")) {
    width = value;
    return;
  }
  
}
```
::::

:::: after
::: ba
После
:::

``` {.code lang="typescript"}
setHeight(arg: number): void {
  height = arg;
}
setWidth(arg: number): number {
  width = arg;
}
```
::::
:::::::
::::::::::::::::::::::::::::

### Причины рефакторинга

Метод, содержащий варианты выполнения, разросся до грандиозных размеров.
В каждой из таких цепочек выполняется нетривиальный код. При этом новые
варианты добавляются очень редко.

### Достоинства

-   Улучшает читабельность кода. Куда очевидней, что делает метод
    `startEngine()` чем `setValue("engineEnabled", true)`.

### Когда нельзя применить

-   Не стоит применять замену параметра явными методами, если метод
    редко меняется, а новые вариации внутри него не добавляются.

### Порядок рефакторинга

1.  Для каждого варианта исполнения метода создайте свой метод.
    Запускайте эти методы в зависимости от значения параметра в основном
    методе.

2.  Найдите все места, где вызывается оригинальный метод. Подставьте
    туда вызов одного из новых методов в зависимости от передающегося
    параметра.

3.  Когда не останется ни одного вызова оригинального метода, его можно
    будет удалить.

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

:::::::::::::: row
::::: {.col-xs-12 .col-sm-6 .col-lg-4}
### Анти-рефакторинг

:::: dl
::: dt
[Параметризация метода](parameterize-method.html)
:::
::::
:::::

::::: {.col-xs-12 .col-sm-6 .col-lg-4}
### Родственные рефакторинги

:::: dl
::: dt
[Замена условного оператора
полиморфизмом](replace-conditional-with-polymorphism.html)
:::
::::
:::::

::::::: {.col-xs-12 .col-sm-6 .col-lg-4}
### Борется с запахом

:::: dl
::: dt
[Операторы switch](smells/switch-statements.html)
:::
::::

:::: dl
::: dt
[Длинный метод](smells/long-method.html)
:::
::::
:::::::
::::::::::::::

::: next
#### Читаем дальше

[Передача всего объекта []{.fa
.fa-arrow-right}](preserve-whole-object.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Параметризация
метода](parameterize-method.html){.btn .btn-default rel="prev"}
:::
:::::::::::::::::::::::::::::::::::::::::::::::::::

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
::::::::::::::::::::::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::::::::::::::::::::::
