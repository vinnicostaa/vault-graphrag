::::::::::::::::::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Рефакторинг](refactoring.html){.type} /
[Прийоми рефакторингу](refactoring/techniques.html){.type} / [Спрощення
викликів
методів](refactoring/techniques/simplifying-method-calls.html){.type}
:::

# Заміна параметра набором спеціалізованих методів {#заміна-параметра-набором-спеціалізованих-методів .title}

::: aka
Також відомий як: [Replace Parameter with Explicit
Methods]{style="display:inline-block"}
:::

::::: before-after-container
::: before
### Проблема

Метод було розбито на частини, кожна з яких виконується залежно від
значення якогось параметра.
:::

::: after
### Рішення

Витягніть окремі частини методу у власні методи і викликайте їх замість
оригінального методу.
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
Після
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
Після
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
Після
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
Після
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
Після
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

### Причини рефакторингу

Метод, що містить варіанти виконання, розрісся до грандіозних розмірів.
У кожному з таких ланцюжків виконується нетривіальний код. При цьому
нові варіанти додаються дуже рідко.

### Переваги

-   Покращує читабельність коду. Куди очевидніше, що робить метод
    `startEngine()`, ніж `setValue("engineEnabled", true)`.

### Коли не слід застосовувати

-   Не варто застосовувати заміну параметра явними методами, якщо метод
    рідко міняється, а нові варіації усередині нього не додаються.

### Порядок рефакторингу

1.  Для кожного варіанту виконання методу створіть свій власний метод.
    Запускайте ці методи залежно від значення параметра в основному
    методі.

2.  Знайдіть усі місця, де викликається оригінальний метод. Підставте
    туди виклик одного з нових методів залежно від параметра, що
    передається.

3.  Коли не залишиться жодного виклику оригінального методу, його можна
    буде видалити.

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

:::::::::::::: row
::::: {.col-xs-12 .col-sm-6 .col-lg-4}
### Анти-рефакторинг

:::: dl
::: dt
[Параметризація методу](parameterize-method.html)
:::
::::
:::::

::::: {.col-xs-12 .col-sm-6 .col-lg-4}
### Схожі рефакторинги

:::: dl
::: dt
[Заміна умовного оператора
поліморфізмом](replace-conditional-with-polymorphism.html)
:::
::::
:::::

::::::: {.col-xs-12 .col-sm-6 .col-lg-4}
### Бореться з запахом

:::: dl
::: dt
[Оператори switch](smells/switch-statements.html)
:::
::::

:::: dl
::: dt
[Довгий метод](smells/long-method.html)
:::
::::
:::::::
::::::::::::::

::: next
#### Читаємо далі

[Передача усього об\'єкта []{.fa
.fa-arrow-right}](preserve-whole-object.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Повернутись назад

[[]{.fa .fa-arrow-left} Параметризація
методу](parameterize-method.html){.btn .btn-default rel="prev"}
:::
:::::::::::::::::::::::::::::::::::::::::::::::::::

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
::::::::::::::::::::::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::::::::::::::::::::::
