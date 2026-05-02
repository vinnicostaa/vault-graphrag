:::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Рефакторинг](refactoring.html){.type} /
[Приёмы](refactoring/techniques.html){.type} / [Составление
методов](refactoring/techniques/composing-methods.html){.type}
:::

# Извлечение метода {#извлечение-метода .title}

::: aka
Также известен как: [Extract Method]{style="display:inline-block"}
:::

::::: before-after-container
::: before
### Проблема

У вас есть фрагмент кода, который можно сгруппировать.
:::

::: after
### Решение

Выделите участок кода в новый метод (или функцию) и вызовите этот метод
вместо старого кода.
:::
:::::

:::::::::::::::::::::::::::: examples
::::::: {.before-after .before-after-container .java data-lang="java"}
:::: before
::: ba
До
:::

``` {.code lang="java"}
void printOwing() {
  printBanner();

  // Print details.
  System.out.println("name: " + name);
  System.out.println("amount: " + getOutstanding());
}
```
::::

:::: after
::: ba
После
:::

``` {.code lang="java"}
void printOwing() {
  printBanner();
  printDetails(getOutstanding());
}

void printDetails(double outstanding) {
  System.out.println("name: " + name);
  System.out.println("amount: " + outstanding);
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
void PrintOwing() 
{
  this.PrintBanner();

  // Print details.
  Console.WriteLine("name: " + this.name);
  Console.WriteLine("amount: " + this.GetOutstanding());
}
```
::::

:::: after
::: ba
После
:::

``` {.code lang="csharp"}
void PrintOwing()
{
  this.PrintBanner();
  this.PrintDetails();
}

void PrintDetails()
{
  Console.WriteLine("name: " + this.name);
  Console.WriteLine("amount: " + this.GetOutstanding());
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
function printOwing() {
  $this->printBanner();

  // Print details.
  print("name:  " . $this->name);
  print("amount " . $this->getOutstanding());
}
```
::::

:::: after
::: ba
После
:::

``` {.code lang="php"}
function printOwing() {
  $this->printBanner();
  $this->printDetails($this->getOutstanding());
}

function printDetails($outstanding) {
  print("name:  " . $this->name);
  print("amount " . $outstanding);
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
def printOwing(self):
    self.printBanner()

    # print details
    print("name:", self.name)
    print("amount:", self.getOutstanding())
```
::::

:::: after
::: ba
После
:::

``` {.code lang="python"}
def printOwing(self):
    self.printBanner()
    self.printDetails(self.getOutstanding())

def printDetails(self, outstanding):
    print("name:", self.name)
    print("amount:", outstanding)
```
::::
:::::::

::::::: {.before-after .before-after-container .typescript data-lang="typescript"}
:::: before
::: ba
До
:::

``` {.code lang="typescript"}
printOwing(): void {
  printBanner();

  // Print details.
  console.log("name: " + name);
  console.log("amount: " + getOutstanding());
}
```
::::

:::: after
::: ba
После
:::

``` {.code lang="typescript"}
printOwing(): void {
  printBanner();
  printDetails(getOutstanding());
}

printDetails(outstanding: number): void {
  console.log("name: " + name);
  console.log("amount: " + outstanding);
}
```
::::
:::::::
::::::::::::::::::::::::::::

### Причины рефакторинга

Чем больше строк кода в методе, тем сложнее разобраться в том, что он
делает. Это основная проблема, которую решает этот рефакторинг.

Извлечение метода не только убивает множество запашков в коде, но и
является одним из этапов множества других рефакторингов.

### Достоинства

-   Улучшает читабельность кода. Постарайтесь дать новому методу
    название, которое бы отражало суть того, что он делает. Например,
    `createOrder()`, `renderCustomerInfo()` и т. д.

-   Убирает дублирование кода. Иногда код, вынесенный в метод, можно
    найти и в других местах программы. В таком случае, имеет смысл
    заменить найденные участки кода вызовом вашего нового метода.

-   Изолирует независимые части кода, уменьшая вероятность ошибок
    (например, по вине переопределения не той переменной).

### Порядок рефакторинга

1.  Создайте новый метод и назовите его так, чтобы название отражало
    суть того, что будет делать этот метод.

2.  Скопируйте беспокоящий вас фрагмент кода в новый метод. Удалите этот
    фрагмент из старого места и замените вызовом вашего нового метода.

    Найдите все переменные, которые использовались в этом фрагменте
    кода. Если они были объявлены внутри этого фрагмента и не
    используются вне его, просто оставьте их без изменений --- они
    станут локальными переменными нового метода.

3.  Если переменные объявлены перед интересующим вас участком кода,
    значит, их следует передать в параметры вашего нового метода, чтобы
    использовать значения, которые в них находились ранее. Иногда от
    таких переменных проще избавиться с помощью [замены переменных
    вызовом метода](replace-temp-with-query.html).

4.  Если вы видите, что локальная переменная как-то изменяется в вашем
    участке кода, это может означать, что её изменённое значение
    понадобится дальше в основном методе. Проверьте это. Если подозрение
    подтвердилось, значение этой переменной следует возвратить в
    основной метод, чтобы ничего не сломать.

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

::::::::::::::::::::::::::::::: row
::::: {.col-xs-12 .col-sm-6 .col-lg-3}
### Анти-рефакторинг

:::: dl
::: dt
[Встраивание метода](inline-method.html)
:::
::::
:::::

::::: {.col-xs-12 .col-sm-6 .col-lg-3}
### Родственные рефакторинги

:::: dl
::: dt
[Перемещение метода](move-method.html)
:::
::::
:::::

::::::::: {.col-xs-12 .col-sm-6 .col-lg-3}
### Помогает рефакторингу

:::: dl
::: dt
[Замена параметров объектом](introduce-parameter-object.html)
:::
::::

:::: dl
::: dt
[Создание шаблонного метода](form-template-method.html)
:::
::::

:::: dl
::: dt
[Параметризация метода](parameterize-method.html)
:::
::::
:::::::::

::::::::::::::::: {.col-xs-12 .col-sm-6 .col-lg-3}
### Борется с запахом

:::: dl
::: dt
[Дублирование кода](smells/duplicate-code.html)
:::
::::

:::: dl
::: dt
[Длинный метод](smells/long-method.html)
:::
::::

:::: dl
::: dt
[Завистливые функции](smells/feature-envy.html)
:::
::::

:::: dl
::: dt
[Операторы switch](smells/switch-statements.html)
:::
::::

:::: dl
::: dt
[Цепочка вызовов](smells/message-chains.html)
:::
::::

:::: dl
::: dt
[Комментарии](smells/comments.html)
:::
::::

:::: dl
::: dt
[Класс данных](smells/data-class.html)
:::
::::
:::::::::::::::::
:::::::::::::::::::::::::::::::

::: next
#### Читаем дальше

[Встраивание метода []{.fa .fa-arrow-right}](inline-method.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Составление
методов](refactoring/techniques/composing-methods.html){.btn
.btn-default rel="prev"}
:::
::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::

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
:::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::
