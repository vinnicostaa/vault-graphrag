:::::::::::::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Рефакторинг](refactoring.html){.type} /
[Приёмы](refactoring/techniques.html){.type} / [Решение задач
обобщения](refactoring/techniques/dealing-with-generalization.html){.type}
:::

# Подъём тела конструктора {#подъём-тела-конструктора .title}

::: aka
Также известен как: [Pull Up Constructor
Body]{style="display:inline-block"}
:::

::::: before-after-container
::: before
### Проблема

Подклассы имеют конструкторы с преимущественно одинаковым кодом.
:::

::: after
### Решение

Создайте конструктор в суперклассе и вынесите в него общий для
подклассов код. Вызывайте конструктор суперкласса в конструкторах
подкласса.
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
После
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
После
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
После
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
После
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
После
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

### Причины рефакторинга

Чем этот рефакторинг отличается от [подъёма
метода](pull-up-method.html)?

1.  В Java подклассы не могут наследовать конструктор, поэтому вы не
    можете просто применить [подъём метода](pull-up-method.html) к
    конструктору подкласса и удалить его после перемещения всего кода
    конструктора в суперкласс. Вдобавок к созданию конструктора в
    суперклассе нужно будет иметь конструкторы в подклассах с простым
    делегированием к конструктору суперкласса.

2.  В C++ и Java (в случае, если вы явно не вызвали конструктор
    суперкласса) конструктор суперкласса автоматически вызывается перед
    конструктором подкласса, что делает обязательным перемещение общего
    кода только из начала конструкторов подклассов (так как вы не
    сможете вызвать конструктор суперкласса в произвольном месте
    конструктора подкласса).

3.  В большинстве языков программирования конструктор подкласса может
    иметь свой собственный список параметров, отличный от параметров
    суперкласса, поэтому вы должны создать конструктор суперкласса
    только с теми параметрами, которые ему действительно нужны.

### Порядок рефакторинга

1.  Создайте конструктор в суперклассе.

2.  Извлеките общий код из начала конструктора каждого из подклассов в
    конструктор суперкласса. Перед этим действием стоит попробовать
    поместить как можно больше общего кода в начало конструктора.

3.  Поместите вызов конструктора суперкласса первой строкой в
    конструкторах подклассов.

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

::::::::: row
::::: {.col-xs-12 .col-sm-6 .col-lg-6}
### Родственные рефакторинги

:::: dl
::: dt
[Подъём метода](pull-up-method.html)
:::
::::
:::::

::::: {.col-xs-12 .col-sm-6 .col-lg-6}
### Борется с запахом

:::: dl
::: dt
[Дублирование кода](smells/duplicate-code.html)
:::
::::
:::::
:::::::::

::: next
#### Читаем дальше

[Спуск метода []{.fa .fa-arrow-right}](push-down-method.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Подъём метода](pull-up-method.html){.btn
.btn-default rel="prev"}
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
