::::::::::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Рефакторинг](refactoring.html){.type} /
[Приёмы](refactoring/techniques.html){.type} / [Упрощение условных
выражений](refactoring/techniques/simplifying-conditional-expressions.html){.type}
:::

# Замена условного оператора полиморфизмом {#замена-условного-оператора-полиморфизмом .title}

::: aka
Также известен как: [Replace Conditional with
Polymorphism]{style="display:inline-block"}
:::

::::: before-after-container
::: before
### Проблема

У вас есть условный оператор, который, в зависимости от типа или свойств
объекта, выполняет различные действия.
:::

::: after
### Решение

Создайте подклассы, которым соответствуют ветки условного оператора. В
них создайте общий метод и переместите в него код из соответствующей
ветки условного оператора. Впоследствии замените условный оператор на
вызов этого метода. Таким образом, нужная реализация будет выбираться
через полиморфизм в зависимости от класса объекта.
:::
:::::

:::::::::::::::::::::::::::: examples
::::::: {.before-after .before-after-container .java data-lang="java"}
:::: before
::: ba
До
:::

``` {.code lang="java"}
class Bird {
  // ...
  double getSpeed() {
    switch (type) {
      case EUROPEAN:
        return getBaseSpeed();
      case AFRICAN:
        return getBaseSpeed() - getLoadFactor() * numberOfCoconuts;
      case NORWEGIAN_BLUE:
        return (isNailed) ? 0 : getBaseSpeed(voltage);
    }
    throw new RuntimeException("Should be unreachable");
  }
}
```
::::

:::: after
::: ba
После
:::

``` {.code lang="java"}
abstract class Bird {
  // ...
  abstract double getSpeed();
}

class European extends Bird {
  double getSpeed() {
    return getBaseSpeed();
  }
}
class African extends Bird {
  double getSpeed() {
    return getBaseSpeed() - getLoadFactor() * numberOfCoconuts;
  }
}
class NorwegianBlue extends Bird {
  double getSpeed() {
    return (isNailed) ? 0 : getBaseSpeed(voltage);
  }
}

// Somewhere in client code
speed = bird.getSpeed();
```
::::
:::::::

::::::: {.before-after .before-after-container .csharp data-lang="csharp"}
:::: before
::: ba
До
:::

``` {.code lang="csharp"}
public class Bird 
{
  // ...
  public double GetSpeed() 
  {
    switch (type) 
    {
      case EUROPEAN:
        return GetBaseSpeed();
      case AFRICAN:
        return GetBaseSpeed() - GetLoadFactor() * numberOfCoconuts;
      case NORWEGIAN_BLUE:
        return isNailed ? 0 : GetBaseSpeed(voltage);
      default:
        throw new Exception("Should be unreachable");
    }
  }
}
```
::::

:::: after
::: ba
После
:::

``` {.code lang="csharp"}
public abstract class Bird 
{
  // ...
  public abstract double GetSpeed();
}

class European: Bird 
{
  public override double GetSpeed() 
  {
    return GetBaseSpeed();
  }
}
class African: Bird 
{
  public override double GetSpeed() 
  {
    return GetBaseSpeed() - GetLoadFactor() * numberOfCoconuts;
  }
}
class NorwegianBlue: Bird
{
  public override double GetSpeed() 
  {
    return isNailed ? 0 : GetBaseSpeed(voltage);
  }
}

// Somewhere in client code
speed = bird.GetSpeed();
```
::::
:::::::

::::::: {.before-after .before-after-container .php data-lang="php"}
:::: before
::: ba
До
:::

``` {.code lang="php"}
class Bird {
  // ...
  public function getSpeed() {
    switch ($this->type) {
      case EUROPEAN:
        return $this->getBaseSpeed();
      case AFRICAN:
        return $this->getBaseSpeed() - $this->getLoadFactor() * $this->numberOfCoconuts;
      case NORWEGIAN_BLUE:
        return ($this->isNailed) ? 0 : $this->getBaseSpeed($this->voltage);
    }
    throw new Exception("Should be unreachable");
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
abstract class Bird {
  // ...
  abstract function getSpeed();
  // ...
}

class European extends Bird {
  public function getSpeed() {
    return $this->getBaseSpeed();
  }
}
class African extends Bird {
  public function getSpeed() {
    return $this->getBaseSpeed() - $this->getLoadFactor() * $this->numberOfCoconuts;
  }
}
class NorwegianBlue extends Bird {
  public function getSpeed() {
    return ($this->isNailed) ? 0 : $this->getBaseSpeed($this->voltage);
  }
}

// Somewhere in Client code.
$speed = $bird->getSpeed();
```
::::
:::::::

::::::: {.before-after .before-after-container .python data-lang="python"}
:::: before
::: ba
До
:::

``` {.code lang="python"}
class Bird:
    # ...
    def getSpeed(self):
        if self.type == EUROPEAN:
            return self.getBaseSpeed()
        elif self.type == AFRICAN:
            return self.getBaseSpeed() - self.getLoadFactor() * self.numberOfCoconuts
        elif self.type == NORWEGIAN_BLUE:
            return 0 if self.isNailed else self.getBaseSpeed(self.voltage)
        else:
            raise Exception("Should be unreachable")
```
::::

:::: after
::: ba
После
:::

``` {.code lang="python"}
class Bird:
    # ...
    def getSpeed(self):
        pass

class European(Bird):
    def getSpeed(self):
        return self.getBaseSpeed()
    
    
class African(Bird):
    def getSpeed(self):
        return self.getBaseSpeed() - self.getLoadFactor() * self.numberOfCoconuts


class NorwegianBlue(Bird):
    def getSpeed(self):
        return 0 if self.isNailed else self.getBaseSpeed(self.voltage)

# Somewhere in client code
speed = bird.getSpeed()
```
::::
:::::::

::::::: {.before-after .before-after-container .typescript data-lang="typescript"}
:::: before
::: ba
До
:::

``` {.code lang="typescript"}
class Bird {
  // ...
  getSpeed(): number {
    switch (type) {
      case EUROPEAN:
        return getBaseSpeed();
      case AFRICAN:
        return getBaseSpeed() - getLoadFactor() * numberOfCoconuts;
      case NORWEGIAN_BLUE:
        return (isNailed) ? 0 : getBaseSpeed(voltage);
    }
    throw new Error("Should be unreachable");
  }
}
```
::::

:::: after
::: ba
После
:::

``` {.code lang="typescript"}
abstract class Bird {
  // ...
  abstract getSpeed(): number;
}

class European extends Bird {
  getSpeed(): number {
    return getBaseSpeed();
  }
}
class African extends Bird {
  getSpeed(): number {
    return getBaseSpeed() - getLoadFactor() * numberOfCoconuts;
  }
}
class NorwegianBlue extends Bird {
  getSpeed(): number {
    return (isNailed) ? 0 : getBaseSpeed(voltage);
  }
}

// Somewhere in client code
let speed = bird.getSpeed();
```
::::
:::::::
::::::::::::::::::::::::::::

### Причины рефакторинга

Этот рефакторинг может помочь, если у вас в коде есть условные
операторы, которые выполняют различную работу, в зависимости от:

-   класса объекта или интерфейса, который он реализует;

-   значения какого-то из полей объекта;

-   результата вызова одного из методов объекта.

При этом если у вас появится новый тип или свойство объекта, нужно будет
искать и добавлять код во все схожие условные операторы. Таким образом,
польза от данного рефакторинга увеличивается, если условных операторов
больше одного, и они разбросаны по всем методам объекта.

### Достоинства

-   Этот рефакторинг реализует принцип *говори, а не спрашивай*: вместо
    того, чтобы спрашивать объект о его состоянии, а потом выполнять на
    основании этого какие-то действия, гораздо проще просто сказать ему,
    что нужно делать, а как это делать он решит сам.

-   Убивает дублирование кода. Вы избавляетесь от множества почти
    одинаковых условных операторов.

-   Если вам потребуется добавить новый вариант выполнения, все, что
    придётся сделать, это добавить новый подкласс, не трогая
    существующий код (*принцип открытости/закрытости*).

### Порядок рефакторинга

#### Подготовка к рефакторингу

Чтобы выполнить этот рефакторинг, вам следует иметь готовую иерархию
классов, в которых будут содержаться альтернативные поведения. Если
такой иерархии ещё нет, нужно создать её. В этом могут помочь другие
рефакторинги:

-   [Замена кодирования типа
    подклассами](replace-type-code-with-subclasses.html). При этом для
    всех значений какого-то свойства объекта будут созданы свои
    подклассы. Это хоть и простой, но менее гибкий способ, так как
    нельзя будет создать подклассы для других свойств объекта.

-   [Замена кодирования типа
    состоянием/стратегией](replace-type-code-with-state-strategy.html).
    При этом для определенного свойства объекта будет выделен свой класс
    и из него созданы подклассы для каждого значения этого свойства.
    Текущий класс будет содержать ссылки на объекты такого типа и
    делегировать им выполнение.

Последующие шаги этого рефакторинга подразумевают, что вы уже создали
иерархию.

#### Шаги рефакторинга

1.  Если условный оператор находится в методе, который выполняет ещё
    какие-то действия, [извлеките его в новый
    метод](extract-method.html).

2.  Для каждого подкласса иерархии, переопределите метод, содержащий
    условный оператор, и скопируйте туда код соответствующей ветки
    оператора.

3.  Удалите эту ветку из условного оператора.

4.  Повторяйте замену, пока условный оператор не опустеет. Затем удалите
    условный оператор и объявите метод абстрактным.

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

:::::: row
::::: {.col-xs-12 .col-sm-6 .col-lg-12}
### Борется с запахом

:::: dl
::: dt
[Операторы switch](smells/switch-statements.html)
:::
::::
:::::
::::::

::: next
#### Читаем дальше

[Введение Null-объекта []{.fa
.fa-arrow-right}](introduce-null-object.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Замена вложенных условных операторов граничным
оператором](replace-nested-conditional-with-guard-clauses.html){.btn
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
