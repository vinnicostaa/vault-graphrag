:::::::::::::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Рефакторинг](refactoring.html){.type} /
[Прийоми рефакторингу](refactoring/techniques.html){.type} / [Задачі
узагальнення
об\'єктів](refactoring/techniques/dealing-with-generalization.html){.type}
:::

# Підйом тіла конструктора {#підйом-тіла-конструктора .title}

::: aka
Також відомий як: [Pull Up Constructor
Body]{style="display:inline-block"}
:::

::::: before-after-container
::: before
### Проблема

Підкласи мають конструктори з переважно однаковим кодом.
:::

::: after
### Рішення

Створіть конструктор в суперкласі і винесіть в нього спільний для
підкласів код. Викликайте конструктор суперкласу в конструкторах
підкласу.
:::
:::::

:::::::::::::::::::::::::::: examples
::::::: {.before-after .before-after-container .java data-lang="java"}
:::: before
::: ba
До
:::

``` {.code lang="java"}
class Manager extends Employee {
  public Manager(String name, String id, int grade) {
    this.name = name;
    this.id = id;
    this.grade = grade;
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
class Manager extends Employee {
  public Manager(String name, String id, int grade) {
    super(name, id);
    this.grade = grade;
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
public class Manager: Employee 
{
  public Manager(string name, string id, int grade) 
  {
    this.name = name;
    this.id = id;
    this.grade = grade;
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
public class Manager: Employee 
{
  public Manager(string name, string id, int grade): base(name, id)
  {
    this.grade = grade;
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
class Manager extends Employee {
  public function __construct($name, $id, $grade) {
    $this->name = $name;
    $this->id = $id;
    $this->grade = $grade;
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
class Manager extends Employee {
  public function __construct($name, $id, $grade) {
    parent::__construct($name, $id);
    $this->grade = $grade;
  }
  // ...
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
class Manager(Employee):
    def __init__(self, name, id, grade):
        self.name = name
        self.id = id
        self.grade = grade
    # ...
```
::::

:::: after
::: ba
Після
:::

``` {.code lang="python"}
class Manager(Employee):
    def __init__(self, name, id, grade):
        Employee.__init__(name, id)
        self.grade = grade
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
class Manager extends Employee {
  constructor(name: string, id: string, grade: number) {
    this.name = name;
    this.id = id;
    this.grade = grade;
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
class Manager extends Employee {
  constructor(name: string, id: string, grade: number) {
    super(name, id);
    this.grade = grade;
  }
  // ...
}
```
::::
:::::::
::::::::::::::::::::::::::::

### Причини рефакторингу

Чим цей рефакторинг відрізняється від [підйому
методу](pull-up-method.html)?

1.  У Java підкласи не можуть наслідувати конструктор, тому ви не можете
    просто застосувати [підйом методу](pull-up-method.html) до
    конструктора підкласу і видалити його після переміщення усього коду
    конструктора в суперклас. На додаток до створення конструктора в
    суперкласі треба буде мати конструктори в підкласах з простим
    делегуванням до конструктора суперкласу.

2.  В C++ і Java (у разі, якщо ви явно не викликали конструктор
    суперкласу) конструктор суперкласу автоматично викликається перед
    конструктором підкласу, що робить обов'язковим переміщення спільного
    коду тільки з початку конструкторів підкласів (оскільки ви не
    зможете викликати конструктор суперкласу в довільному місці
    конструктора підкласу).

3.  У більшості мов програмування конструктор підкласу може мати свій
    власний список параметрів, відмінний від параметрів суперкласу, тому
    ви повинні створити конструктор суперкласу тільки з тими
    параметрами, які йому дійсно потрібні.

### Порядок рефакторингу

1.  Створіть конструктор в суперкласі.

2.  Витягніть спільний код з початку конструктора кожного з підкласів в
    конструктор суперкласу. Перед цією дією варто спробувати помістити
    якомога більше спільного коду в початок конструктора.

3.  Розташуйте виклик конструктора суперкласу першим рядком в
    конструкторах підкласів.

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
[Підйом методу](pull-up-method.html)
:::
::::
:::::

::::: {.col-xs-12 .col-sm-6 .col-lg-6}
### Бореться з запахом

:::: dl
::: dt
[Дублювання коду](smells/duplicate-code.html)
:::
::::
:::::
:::::::::

::: next
#### Читаємо далі

[Спуск методу []{.fa .fa-arrow-right}](push-down-method.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Повернутись назад

[[]{.fa .fa-arrow-left} Підйом методу](pull-up-method.html){.btn
.btn-default rel="prev"}
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
