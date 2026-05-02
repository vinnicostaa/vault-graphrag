:::::::::::::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Рефакторинг](refactoring.html){.type} /
[Прийоми рефакторингу](refactoring/techniques.html){.type} /
[Організація даних](refactoring/techniques/organizing-data.html){.type}
:::

# Самоінкапсуляція поля {#самоінкапсуляція-поля .title}

::: aka
Також відомий як: [Self Encapsulate Field]{style="display:inline-block"}
:::

> Самоінкапсуляція відрізняється від звичайної [інкапсуляції
> поля](encapsulate-field.html) тим, що рефакторинг робиться над
> приватним полем.

::::: before-after-container
::: before
### Проблема

Ви використовуєте прямий доступ до приватних полів усередині класу.
:::

::: after
### Рішення

Створіть геттер і сеттер для поля, і користуйтеся для доступу до поля
тільки ними.
:::
:::::

::::::::::::::::::::::: examples
::::::: {.before-after .before-after-container .java data-lang="java"}
:::: before
::: ba
До
:::

``` {.code lang="java"}
class Range {
  private int low, high;
  boolean includes(int arg) {
    return arg >= low && arg <= high;
  }
}
```
::::

:::: after
::: ba
Після
:::

``` {.code lang="java"}
class Range {
  private int low, high;
  boolean includes(int arg) {
    return arg >= getLow() && arg <= getHigh();
  }
  int getLow() {
    return low;
  }
  int getHigh() {
    return high;
  }
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
class Range 
{
  private int low, high;
  
  bool Includes(int arg) 
  {
    return arg >= low && arg <= high;
  }
}
```
::::

:::: after
::: ba
Після
:::

``` {.code lang="csharp"}
class Range 
{
  private int low, high;
  
  int Low {
    get { return low; }
  }
  int High {
    get { return high; }
  }
  
  bool Includes(int arg) 
  {
    return arg >= Low && arg <= High;
  }
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
private $low;
private $high;

function includes($arg) {
  return $arg >= $this->low && $arg <= $this->high;
}
```
::::

:::: after
::: ba
Після
:::

``` {.code lang="php"}
private $low;
private $high;

function includes($arg) {
  return $arg >= $this->getLow() && $arg <= $this->getHigh();
}
function getLow() {
  return $this->low;
}
function getHigh() {
  return $this->high;
}
```
::::
:::::::

::::::: {.before-after .before-after-container .typescript data-lang="typescript"}
:::: before
::: ba
До
:::

``` {.code lang="typescript"}
class Range {
  private low: number
  private high: number;
  includes(arg: number): boolean {
    return arg >= low && arg <= high;
  }
}
```
::::

:::: after
::: ba
Після
:::

``` {.code lang="typescript"}
class Range {
  private low: number
  private high: number;
  includes(arg: number): boolean {
    return arg >= getLow() && arg <= getHigh();
  }
  getLow(): number {
    return low;
  }
  getHigh(): number {
    return high;
  }
}
```
::::
:::::::
:::::::::::::::::::::::

### Причини рефакторингу

Буває так, що вам перестає вистачати гнучкості з прямим доступом до
приватного поля всередині класу. Ви хочете мати можливість
ініціалізувати значення поля при першому запиті або робити якісь
операції над новими значеннями поля в момент присвоєння, або робити все
це різними способами в підкласах.

### Переваги

-   *Непрямий доступ до полів* --- це коли робота з полем відбувається
    через методи доступу (геттери і сеттери). Цей підхід відрізняється
    набагато більшою гнучкістю, ніж *прямий доступ до полів*.

    -   По-перше, ви можете здійснювати складні операції при отриманні
        або установці даних в полі. *Лінива ініціалізація*, *валідація
        значень в полі* --- усе це легко реалізується всередині геттерів
        і сеттерів поля.

    -   По-друге, що ще важливіше, ви можете перевизначати геттери і
        сеттери в підкласах.

-   Ви можете взагалі не реалізовувати сеттер для поля. Значення поля
    задаватиметься тільки в конструкторі, роблячи це поле незмінним для
    всього періоду життя об'єкта.

### Недоліки

-   Коли використовується *прямий доступ до полів*, код виглядає
    простіше і наочніше, хоча і втрачає в гнучкості.

### Порядок рефакторингу

1.  Створіть геттер (і, опціонально, сеттер) для поля. Вони мають бути
    захищеними (`protected`) або публічними (`public`).

2.  Знайдіть усі прямі звернення до поля і замініть їх викликами геттера
    і сеттера.

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
:::::: {.col-xs-12 .col-sm-6 .col-lg-6}
### Схожі рефакторинги

::::: dl
::: dt
[Інкапсуляція поля](encapsulate-field.html)
:::

::: dd
Ховаєм публічні поля, створюєм для них методи доступу.
:::
:::::
::::::

::::::::: {.col-xs-12 .col-sm-6 .col-lg-6}
### Домогає іншим рефакторингам

:::: dl
::: dt
[Дублювання видимих даних](duplicate-observed-data.html)
:::
::::

:::: dl
::: dt
[Заміна кодування типу
підкласами](replace-type-code-with-subclasses.html)
:::
::::

:::: dl
::: dt
[Заміна кодування типу
станом/стратегією](replace-type-code-with-state-strategy.html)
:::
::::
:::::::::
::::::::::::::

::: next
#### Читаємо далі

[Заміна простого поля об\'єктом []{.fa
.fa-arrow-right}](replace-data-value-with-object.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Повернутись назад

[[]{.fa .fa-arrow-left} Організація
даних](refactoring/techniques/organizing-data.html){.btn .btn-default
rel="prev"}
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
