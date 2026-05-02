:::::::::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Рефакторинг](refactoring.html){.type} /
[Приёмы](refactoring/techniques.html){.type} / [Организация
данных](refactoring/techniques/organizing-data.html){.type}
:::

# Инкапсуляция поля {#инкапсуляция-поля .title}

::: aka
Также известен как: [Encapsulate Field]{style="display:inline-block"}
:::

::::: before-after-container
::: before
### Проблема

У вас есть публичное поле.
:::

::: after
### Решение

Сделайте поле приватным и создайте для него методы доступа.
:::
:::::

::::::::::::::::::::::: examples
::::::: {.before-after .before-after-container .java data-lang="java"}
:::: before
::: ba
До
:::

``` {.code lang="java"}
class Person {
  public String name;
}
```
::::

:::: after
::: ba
После
:::

``` {.code lang="java"}
class Person {
  private String name;

  public String getName() {
    return name;
  }
  public void setName(String arg) {
    name = arg;
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
class Person 
{
  public string name;
}
```
::::

:::: after
::: ba
После
:::

``` {.code lang="csharp"}
class Person 
{
  private string name;

  public string Name
  {
    get { return name; }
    set { name = value; }
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
public $name;
```
::::

:::: after
::: ba
После
:::

``` {.code lang="php"}
private $name;

public getName() {
  return $this->name;
}

public setName($arg) {
  $this->name = $arg;
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
class Person {
  name: string;
}
```
::::

:::: after
::: ba
После
:::

``` {.code lang="typescript"}
class Person {
  private _name: string;

  get name() {
    return this._name;
  }
  setName(arg: string): void {
    this._name = arg;
  }
}
```
::::
:::::::
:::::::::::::::::::::::

### Причины рефакторинга

Одним из столпов объектного программирования является *Инкапсуляция* или
возможность сокрытия данных объекта. Иначе все данные объектов были бы
публичными, и другие объекты могли бы получать и модифицировать данные
вашего объекта без его ведома. При этом отделяются данные от поведений,
связанных с этими данными, ухудшается модульность частей программы и
усложняется её поддержка.

### Достоинства

-   Если данные и поведения какого-то компонента тесно связанны между
    собой и находятся в одном месте в коде, вам гораздо проще
    поддерживать и развивать этот компонент.

-   Кроме того, вы можете производить какие-то сложные операции,
    связанные с доступом к полям объекта.

### Когда нельзя применить

-   Встречаются случаи, когда инкапсуляция полей нежелательна из
    соображений, связанных с повышением производительности. Эти случаи
    очень редки, но иногда этот момент бывает очень важным.

    Например, у вас есть графический редактор, в котором есть объекты,
    имеющие координаты `x` и `y`. Эти поля вряд ли будут меняться в
    будущем. К тому же, в программе участвует очень много различных
    объектов, в которых присутствуют эти поля. Поэтому обращение
    напрямую к полям координат экономит значительную часть процессорного
    времени, которое иначе затрачивалось бы на вызовы методов доступа.

    Как иллюстрация этого исключения, существует класс
    [Point](http://docs.oracle.com/javase/7/docs/api/java/awt/Point.html)
    в Java, все поля которого являются публичными.

### Порядок рефакторинга

1.  Создайте геттер и сеттер для поля.

2.  Найдите все обращения к полю. Замените получение значения из поля
    геттером, а установку новых значений в поле --- сеттером.

3.  После того как все обращения к полям заменены, сделайте поле
    приватным.

#### Последующие шаги

«Инкапсуляция поля» является всего лишь первым шагом к сближению данных
и поведений над этими данными. После того как вы создали простые методы
доступа к полям, стоит ещё раз проверить места, где эти методы
вызываются. Вполне возможно, код из этих участков уместнее смотрелся бы
в самих методах доступа.

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

:::::::::: row
:::::: {.col-xs-12 .col-sm-6 .col-lg-6}
### Родственные рефакторинги

::::: dl
::: dt
[Самоинкапсуляция поля](self-encapsulate-field.html)
:::

::: dd
Вместо прямого доступа к приватным полям, создаём методы доступа к этим
полям.
:::
:::::
::::::

::::: {.col-xs-12 .col-sm-6 .col-lg-6}
### Борется с запахом

:::: dl
::: dt
[Класс данных](smells/data-class.html)
:::
::::
:::::
::::::::::

::: next
#### Читаем дальше

[Инкапсуляция коллекции []{.fa
.fa-arrow-right}](encapsulate-collection.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Замена магического числа символьной
константой](replace-magic-number-with-symbolic-constant.html){.btn
.btn-default rel="prev"}
:::
::::::::::::::::::::::::::::::::::::::::::

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
:::::::::::::::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::::::::::::::
