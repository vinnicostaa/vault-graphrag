::::::::::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Рефакторинг](refactoring.html){.type} /
[Прийоми рефакторингу](refactoring/techniques.html){.type} / [Спрощення
викликів
методів](refactoring/techniques/simplifying-method-calls.html){.type}
:::

# Заміна конструктора фабричним методом {#заміна-конструктора-фабричним-методом .title}

::: aka
Також відомий як: [Replace Constructor with Factory
Method]{style="display:inline-block"}
:::

::::: before-after-container
::: before
### Проблема

У вас є складний конструктор, що робить щось більше, ніж просте
встановлення значень для полів об'єкта.
:::

::: after
### Рішення

Створіть фабричний метод і замініть ним виклики конструктора.
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
Після
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
Після
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
Після
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
Після
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

### Причини рефакторингу

Найочевидніша причина застосування цього рефакторингу пов'язана з
[заміною кодування типу
підкласами](replace-type-code-with-subclasses.html).

У вас є код, в якому раніше створювався об'єкт, куди передавалося
значення кодованого типу. Після застосування рефакторингу з'явилося вже
декілька підкласів, з яких треба створювати об'єкти залежно від значення
кодованого типу. Змінити оригінальний конструктор так, щоб він повертав
об'єкти підкласів, неможливо, тому ми створюємо статичний фабричний
метод, який повертатиме об'єкти потрібних класів, після чого він замінює
собою усі виклики оригінального конструктора.

Фабричні методи можна використати і в інших ситуаціях, коли можливостей
конструкторів виявляється недостатньо. Вони важливі при [заміні значення
посиланням](change-value-to-reference.html). Їх можна також
застосовувати для завдання різноманітних режимів створення, параметрів і
типів, що виходять за рамки числа.

### Переваги

-   Фабричний метод не обов'язково повертає об'єкт того класу, в якому
    він був викликаний. Часто це можуть бути його підкласи, вибрані
    залежно від аргументів, що подаються в метод.

-   Фабричний метод може мати більш вдале ім'я, яке описує, що і яким
    чином він повертає, наприклад, `Troops::GetCrew(myTank)`.

-   Фабричний метод може повернути вже створений об'єкт на відміну від
    конструктора, який завжди створює новий екземпляр.

### Порядок рефакторингу

1.  Створіть фабричний метод. Помістіть його в тіло виклику поточного
    конструктора.

2.  Замініть усі виклики конструктора викликами фабричного методу.

3.  Оголосіть конструктор приватним.

4.  Обстежте код конструктора і спробуйте винести у фабричний метод той
    код, який не відноситься до безпосереднього конструювання об'єкта
    поточного класу.

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

::::::::::: row
::::::: {.col-xs-12 .col-sm-6 .col-lg-6}
### Домогає іншим рефакторингам

:::: dl
::: dt
[Заміна значення посиланням](change-value-to-reference.html)
:::
::::

:::: dl
::: dt
[Заміна кодування типу
підкласами](replace-type-code-with-subclasses.html)
:::
::::
:::::::

::::: {.col-xs-12 .col-sm-6 .col-lg-6}
### Реалізує паттерн проектування

:::: dl
::: dt
[Фабричний метод](design-patterns/factory-method.html)
:::
::::
:::::
:::::::::::

::: next
#### Читаємо далі

[Заміна коду помилки виключенням []{.fa
.fa-arrow-right}](replace-error-code-with-exception.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Повернутись назад

[[]{.fa .fa-arrow-left} Приховання методу](hide-method.html){.btn
.btn-default rel="prev"}
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
