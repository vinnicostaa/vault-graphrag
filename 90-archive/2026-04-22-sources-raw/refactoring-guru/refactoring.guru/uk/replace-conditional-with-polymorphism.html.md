::::::::::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Рефакторинг](refactoring.html){.type} /
[Прийоми рефакторингу](refactoring/techniques.html){.type} / [Спрощення
умовних
виразів](refactoring/techniques/simplifying-conditional-expressions.html){.type}
:::

# Заміна умовного оператора поліморфізмом {#заміна-умовного-оператора-поліморфізмом .title}

::: aka
Також відомий як: [Replace Conditional with
Polymorphism]{style="display:inline-block"}
:::

::::: before-after-container
::: before
### Проблема

У вас є умовний оператор, який, залежно від типу або властивостей
об'єкта, виконує різні дії.
:::

::: after
### Рішення

Створіть підкласи, яким відповідають гілки умовного оператора. У них
створіть спільний метод і перемістіть в нього код з відповідної гілки
умовного оператора. Згодом проведіть заміну умовного оператора на виклик
цього методу. Таким чином, потрібна реалізація вибиратиметься через
поліморфізм залежно від класу об'єкта.
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
Після
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
Після
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
Після
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
Після
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
Після
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

### Причини рефакторингу

Цей рефакторинг може допомогти, якщо у вас в коді є умовні оператори,
які виконують різну роботу, залежно від:

-   класу об'єкта або інтерфейсу, який він реалізує;

-   значення якогось з полів об'єкта;

-   результату виклику одного з методів об'єкта.

При цьому, якщо у вас з'явиться новий тип або властивість об'єкта, треба
буде шукати і додавати код в усі схожі умовні оператори. Таким чином,
користь від цього рефакторингу збільшується, якщо умовних операторів
більш, ніж один, і вони розкидані по усіх методах об'єкта.

### Переваги

-   Цей рефакторинг реалізує принцип *кажи, а не запитуй*: замість того,
    щоб запитувати об'єкт про його стан, а потім виконувати на підставі
    цього якісь дії, набагато простіше просто сказати йому, що треба
    робити, а як це робити він вирішить сам.

-   Вбиває дублювання коду. Ви позбавляєтеся від безлічі майже однакових
    умовних операторів.

-   Якщо вам потрібно буде додати новий варіант виконання, все, що
    доведеться зробити, це додати новий підклас, не чіпаючи існуючий код
    (*принцип відкритості/закритості*).

### Порядок рефакторингу

#### Підготовка до рефакторингу

Щоб виконати цей рефакторинг, вам слід мати готову ієрархію класів, в
яких міститиметься альтернативна поведінка. Якщо такої ієрархії ще
немає, треба створити її. У цьому можуть допомогти інші рефакторинги:

-   [Заміна кодування типу
    підкласами](replace-type-code-with-subclasses.html). При цьому для
    усіх значень якоїсь властивості об'єкта будуть створені свої
    підкласи. Це хоч і простий, але менш гнучкий спосіб, оскільки не
    можна буде створити підкласи для інших властивостей об'єкта.

-   [Заміна кодування типу
    станом/стратегією](replace-type-code-with-state-strategy.html). При
    цьому для певної властивості об'єкта буде виділений свій власний
    клас і з нього створені підкласи для кожного значення цієї
    властивості. Поточний клас утримуватиме посилання на об'єкти такого
    типу і делегуватиме їм виконання.

Подальші кроки цього рефакторингу вважають, що ви вже створили ієрархію.

#### Кроки рефакторингу

1.  Якщо умовний оператор знаходиться в методі, який виконує ще якісь
    дії, [витягніть його в новий метод](extract-method.html).

2.  Для кожного підкласу ієрархії потібно перевизначити метод, ща
    містить умовний оператор, і скопіювати туди код відповідної гілки
    оператора.

3.  Видаліть цю гілку з умовного оператора.

4.  Повторюйте заміну, поки умовний оператор не спорожніє. Потім
    видаліть умовний оператор і оголосите метод абстрактним.

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

:::::: row
::::: {.col-xs-12 .col-sm-6 .col-lg-12}
### Бореться з запахом

:::: dl
::: dt
[Оператори switch](smells/switch-statements.html)
:::
::::
:::::
::::::

::: next
#### Читаємо далі

[Введення Null-об\'єкта []{.fa
.fa-arrow-right}](introduce-null-object.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Повернутись назад

[[]{.fa .fa-arrow-left} Заміна вкладених умовних операторів граничним
оператором](replace-nested-conditional-with-guard-clauses.html){.btn
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
