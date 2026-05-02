::::::::::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Рефакторинг](refactoring.html){.type} /
[Приёмы](refactoring/techniques.html){.type} / [Упрощение вызовов
методов](refactoring/techniques/simplifying-method-calls.html){.type}
:::

# Замена конструктора фабричным методом {#замена-конструктора-фабричным-методом .title}

::: aka
Также известен как: [Replace Constructor with Factory
Method]{style="display:inline-block"}
:::

::::: before-after-container
::: before
### Проблема

У вас есть сложный конструктор, делающий нечто большее, чем простая
установка значений полей объекта.
:::

::: after
### Решение

Создайте фабричный метод и замените им вызовы конструктора.
:::
:::::

::::::::::::::::::::::: examples
::::::: {.before-after .before-after-container .java data-lang="java"}
:::: before
::: ba
До
:::

``` {.code lang="java"}
class Employee {
  Employee(int type) {
    this.type = type;
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
class Employee {
  static Employee create(int type) {
    employee = new Employee(type);
    // do some heavy lifting.
    return employee;
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
public class Employee 
{
  public Employee(int type) 
  {
    this.type = type;
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
public class Employee
{
  public static Employee Create(int type)
  {
    employee = new Employee(type);
    // Do some heavy lifting.
    return employee;
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
class Employee {
  // ...
  public function __construct($type) {
   $this->type = $type;
  }
  // ...
}
```
::::

:::: after
::: ba
После
:::

``` {.code lang="php"}
class Employee {
  // ...
  static public function create($type) {
    $employee = new Employee($type);
    // do some heavy lifting.
    return $employee;
  }
  // ...
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
class Employee {
  constructor(type: number) {
    this.type = type;
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
class Employee {
  static create(type: number): Employee {
    let employee = new Employee(type);
    // Do some heavy lifting.
    return employee;
  }
  // ...
}
```
::::
:::::::
:::::::::::::::::::::::

### Причины рефакторинга

Самая очевидная причина применения этого рефакторинга связана с [заменой
кодирования типа подклассами](replace-type-code-with-subclasses.html).

У вас есть код, в котором раньше создавался объект, куда передавалось
значение кодированного типа. После применения рефакторинга появилось уже
несколько подклассов, из которых нужно создавать объекты в зависимости
от значения кодированного типа. Изменить оригинальный конструктор так,
чтобы он возвращал объекты подклассов, невозможно, поэтому мы создаём
статический фабричный метод, который будет возвращать объекты нужных
классов, после чего он заменяет собой все вызовы оригинального
конструктора.

Фабричные методы можно использовать и в других ситуациях, когда
возможностей конструкторов оказывается недостаточно. Они важны при
[замене значения ссылкой](change-value-to-reference.html). Их можно
также применять для задания различных режимов создания, выходящих за
рамки числа и типов параметров.

### Достоинства

-   Фабричный метод не обязательно возвращает объект того класса, в
    котором он был вызван. Зачастую это могут быть его подклассы,
    выбираемые в зависимости от подаваемых в метод аргументов.

-   Фабричный метод может иметь более удачное имя, описывающее, что и
    каким образом он возвращает, например, `Troops::GetCrew(myTank)`.

-   Фабричный метод может вернуть уже созданный объект в отличие от
    конструктора, который всегда создает новый экземпляр.

### Порядок рефакторинга

1.  Создайте фабричный метод. Поместите в него вызов текущего
    конструктора.

2.  Замените все вызовы конструктора вызовами фабричного метода.

3.  Объявите конструктор приватным.

4.  Обследуйте код конструктора и попытайтесь вынести в фабричный метод
    тот код, который не относится к непосредственному конструированию
    объекта текущего класса.

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

::::::::::: row
::::::: {.col-xs-12 .col-sm-6 .col-lg-6}
### Помогает рефакторингу

:::: dl
::: dt
[Замена значения ссылкой](change-value-to-reference.html)
:::
::::

:::: dl
::: dt
[Замена кодирования типа
подклассами](replace-type-code-with-subclasses.html)
:::
::::
:::::::

::::: {.col-xs-12 .col-sm-6 .col-lg-6}
### Реализует паттерн проектирования

:::: dl
::: dt
[Фабричный метод](design-patterns/factory-method.html)
:::
::::
:::::
:::::::::::

::: next
#### Читаем дальше

[Замена кода ошибки исключением []{.fa
.fa-arrow-right}](replace-error-code-with-exception.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Сокрытие метода](hide-method.html){.btn
.btn-default rel="prev"}
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
