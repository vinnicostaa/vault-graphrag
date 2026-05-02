:::::::::::::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Рефакторинг](refactoring.html){.type} /
[Приёмы](refactoring/techniques.html){.type} / [Организация
данных](refactoring/techniques/organizing-data.html){.type}
:::

# Самоинкапсуляция поля {#самоинкапсуляция-поля .title}

::: aka
Также известен как: [Self Encapsulate
Field]{style="display:inline-block"}
:::

> Самоинкапсуляция отличается от обычной [инкапсуляции
> поля](encapsulate-field.html) тем, что рефакторинг производится над
> приватным полем.

::::: before-after-container
::: before
### Проблема

Вы используете прямой доступ к приватным полями внутри класса.
:::

::: after
### Решение

Создайте геттер и сеттер для поля, и пользуйтесь для доступа к полю
только ими.
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
После
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
После
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
После
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
После
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

### Причины рефакторинга

Бывает так, что вам перестаёт хватать гибкости с прямым доступом к
приватному полю внутри класса. Вы хотите иметь возможность
инициализировать значение поля при первом запросе или производить
какие-то операции над новыми значениями поля в момент присваивания, либо
делать все это разными способами в подклассах.

### Достоинства

-   *Непрямой доступ к полям* --- это когда работа с полем происходит
    через методы доступа (геттеры и сеттеры). Этот подход отличается
    гораздо большей гибкостью, чем *прямой доступ к полям*.

    -   Во-первых, вы можете осуществлять сложные операции при получении
        или установке данных в поле. *Ленивая инициализация*, *валидация
        значений в поле* --- все это легко реализуемо внутри геттеров и
        сеттеров поля.

    -   Во-вторых, что ещё важнее, вы можете переопределять геттеры и
        сеттеры в подклассах.

-   Вы можете вообще не реализовывать сеттер для поля. Значение поля
    будет задаваться только в конструкторе, делая это поле неизменяемым
    для всего периода жизни объекта.

### Недостатки

-   Когда используется *прямой доступ к полям*, код выглядит проще и
    нагляднее, хотя и теряет в гибкости.

### Порядок рефакторинга

1.  Создайте геттер (и опциональный сеттер) для поля. Они должны быть
    защищёнными (`protected`) либо публичными (`public`).

2.  Найдите все прямые обращения к полю и замените их вызовами геттера и
    сеттера.

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
:::::: {.col-xs-12 .col-sm-6 .col-lg-6}
### Родственные рефакторинги

::::: dl
::: dt
[Инкапсуляция поля](encapsulate-field.html)
:::

::: dd
Скрываем публичные поля, создаём для них методы доступа.
:::
:::::
::::::

::::::::: {.col-xs-12 .col-sm-6 .col-lg-6}
### Помогает рефакторингу

:::: dl
::: dt
[Дублирование видимых данных](duplicate-observed-data.html)
:::
::::

:::: dl
::: dt
[Замена кодирования типа
подклассами](replace-type-code-with-subclasses.html)
:::
::::

:::: dl
::: dt
[Замена кодирования типа
состоянием/стратегией](replace-type-code-with-state-strategy.html)
:::
::::
:::::::::
::::::::::::::

::: next
#### Читаем дальше

[Замена простого поля объектом []{.fa
.fa-arrow-right}](replace-data-value-with-object.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Организация
данных](refactoring/techniques/organizing-data.html){.btn .btn-default
rel="prev"}
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
