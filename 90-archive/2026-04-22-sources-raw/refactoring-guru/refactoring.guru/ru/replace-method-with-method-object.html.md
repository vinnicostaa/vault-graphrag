::::::::::::::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Рефакторинг](refactoring.html){.type} /
[Приёмы](refactoring/techniques.html){.type} / [Составление
методов](refactoring/techniques/composing-methods.html){.type}
:::

# Замена метода объектом методов {#замена-метода-объектом-методов .title}

::: aka
Также известен как: [Replace Method with Method
Object]{style="display:inline-block"}
:::

::::: before-after-container
::: before
### Проблема

У вас есть длинный метод, в котором локальные переменные так сильно
переплетены, что это делает невозможным применение «извлечения метода».
:::

::: after
### Решение

Преобразуйте метод в отдельный класс так, чтобы локальные переменные
стали полями этого класса. После этого можно без труда разделить метод
на части.
:::
:::::

:::::::::::::::::::::::::::: examples
::::::: {.before-after .before-after-container .java data-lang="java"}
:::: before
::: ba
До
:::

``` {.code lang="java"}
class Order {
  // ...
  public double price() {
    double primaryBasePrice;
    double secondaryBasePrice;
    double tertiaryBasePrice;
    // Perform long computation.
  }
}
```
::::

:::: after
::: ba
После
:::

``` {.code lang="java"}
class Order {
  // ...
  public double price() {
    return new PriceCalculator(this).compute();
  }
}

class PriceCalculator {
  private double primaryBasePrice;
  private double secondaryBasePrice;
  private double tertiaryBasePrice;
  
  public PriceCalculator(Order order) {
    // Copy relevant information from the
    // order object.
  }
  
  public double compute() {
    // Perform long computation.
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
public class Order 
{
  // ...
  public double Price() 
  {
    double primaryBasePrice;
    double secondaryBasePrice;
    double tertiaryBasePrice;
    // Perform long computation.
  }
}
```
::::

:::: after
::: ba
После
:::

``` {.code lang="csharp"}
public class Order 
{
  // ...
  public double Price() 
  {
    return new PriceCalculator(this).Compute();
  }
}

public class PriceCalculator 
{
  private double primaryBasePrice;
  private double secondaryBasePrice;
  private double tertiaryBasePrice;
  
  public PriceCalculator(Order order) 
  {
    // Copy relevant information from the
    // order object.
  }
  
  public double Compute() 
  {
    // Perform long computation.
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
class Order {
  // ...
  public function price() {
    $primaryBasePrice = 10;
    $secondaryBasePrice = 20;
    $tertiaryBasePrice = 30;
    // Perform long computation.
  }
}
```
::::

:::: after
::: ba
После
:::

``` {.code lang="php"}
class Order {
  // ...
  public function price() {
    return (new PriceCalculator($this))->compute();
  }
}

class PriceCalculator {
  private $primaryBasePrice;
  private $secondaryBasePrice;
  private $tertiaryBasePrice;
  
  public function __construct(Order $order) {
      // Copy relevant information from the
      // order object.
  }
  
  public function compute() {
    // Perform long computation.
  }
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
class Order:
    # ...
    def price(self):
        primaryBasePrice = 0
        secondaryBasePrice = 0
        tertiaryBasePrice = 0
        # Perform long computation.
```
::::

:::: after
::: ba
После
:::

``` {.code lang="python"}
class Order:
    # ...
    def price(self):
        return PriceCalculator(self).compute()


class PriceCalculator:
    def __init__(self, order):
        self._primaryBasePrice = 0
        self._secondaryBasePrice = 0
        self._tertiaryBasePrice = 0
        # Copy relevant information from the
        # order object.

    def compute(self):
        # Perform long computation.
```
::::
:::::::

::::::: {.before-after .before-after-container .typescript data-lang="typescript"}
:::: before
::: ba
До
:::

``` {.code lang="typescript"}
class Order {
  // ...
  price(): number {
    let primaryBasePrice;
    let secondaryBasePrice;
    let tertiaryBasePrice;
    // Perform long computation.
  }
}
```
::::

:::: after
::: ba
После
:::

``` {.code lang="typescript"}
class Order {
  // ...
  price(): number {
    return new PriceCalculator(this).compute();
  }
}

class PriceCalculator {
  private _primaryBasePrice: number;
  private _secondaryBasePrice: number;
  private _tertiaryBasePrice: number;
  
  constructor(order: Order) {
    // Copy relevant information from the
    // order object.
  }
  
  compute(): number {
    // Perform long computation.
  }
}
```
::::
:::::::
::::::::::::::::::::::::::::

### Причины рефакторинга

Метод слишком длинный, и вы не можете его разделить из-за хитросплетения
локальных переменных, которые сложно изолировать друг от друга.

Первым шагом к решению проблемы будет выделение всего этого метода в
отдельный класс и превращение его локальных переменных в поля.

Во-первых, это позволит вам изолировать проблему в пределах этого
класса, а во-вторых, расчистит дорогу для разделения большого метода на
методы поменьше, которые, к тому же, не подходили бы к смыслу
оригинального класса.

### Достоинства

-   Изоляция длинного метода в собственном классе позволяет остановить
    бесконтрольный рост метода. Кроме того, даёт возможность разделить
    его на подметоды в рамках своего класса, не засоряя служебными
    методами оригинальный класс.

### Недостатки

-   Создаётся ещё один класс, повышая общую сложность программы.

### Порядок рефакторинга

1.  Создайте новый класс. Дайте ему название, основываясь на
    предназначении метода, который рефакторите.

2.  В новом классе создайте приватное поле для хранения ссылки на
    экземпляр класса, в котором раньше находился метод. Эту ссылку потом
    можно будет использовать, чтобы брать из оригинального объекта
    нужные данные, если потребуется.

3.  Создайте отдельное приватное поле для каждой локальной переменной
    метода.

4.  Создайте конструктор, который принимает в параметрах значения всех
    локальных переменных метода, а также инициализирует соответствующие
    приватные поля.

5.  Объявите основной метод и скопируйте в него код оригинального
    метода, заменив локальные переменные приватным полями.

6.  Замените тело оригинального метода в исходном классе созданием
    объекта-метода и вызовом его основного метода.

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
[Замена простого поля объектом](replace-data-value-with-object.html)
:::

::: dd
Делает то же, но с полем.
:::
:::::
::::::

::::: {.col-xs-12 .col-sm-6 .col-lg-6}
### Борется с запахом

:::: dl
::: dt
[Длинный метод](smells/long-method.html)
:::
::::
:::::
::::::::::

::: next
#### Читаем дальше

[Замена алгоритма []{.fa
.fa-arrow-right}](substitute-algorithm.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Удаление присваиваний
параметрам](remove-assignments-to-parameters.html){.btn .btn-default
rel="prev"}
:::
:::::::::::::::::::::::::::::::::::::::::::::::

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
::::::::::::::::::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::::::::::::::::::
