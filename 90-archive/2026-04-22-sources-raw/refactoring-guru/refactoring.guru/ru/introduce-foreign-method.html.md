::::::::::::::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Рефакторинг](refactoring.html){.type} /
[Приёмы](refactoring/techniques.html){.type} / [Перемещение функций
между
объектами](refactoring/techniques/moving-features-between-objects.html){.type}
:::

# Введение внешнего метода {#введение-внешнего-метода .title}

::: aka
Также известен как: [Introduce Foreign
Method]{style="display:inline-block"}
:::

::::: before-after-container
::: before
### Проблема

Служебный класс не содержит метода, который вам нужен, при этом у вас
нет возможности добавить метод в этот класс.
:::

::: after
### Решение

Добавьте метод в клиентский класс и передавайте в него объект служебного
класса в качестве аргумента.
:::
:::::

:::::::::::::::::::::::::::: examples
::::::: {.before-after .before-after-container .java data-lang="java"}
:::: before
::: ba
До
:::

``` {.code lang="java"}
class Report {
  // ...
  void sendReport() {
    Date nextDay = new Date(previousEnd.getYear(),
      previousEnd.getMonth(), previousEnd.getDate() + 1);
    // ...
  }
}
```
::::

:::: after
::: ba
После
:::

``` {.code lang="java"}
class Report {
  // ...
  void sendReport() {
    Date newStart = nextDay(previousEnd);
    // ...
  }
  private static Date nextDay(Date arg) {
    return new Date(arg.getYear(), arg.getMonth(), arg.getDate() + 1);
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
class Report 
{
  // ...
  void SendReport() 
  {
    DateTime nextDay = previousEnd.AddDays(1);
    // ...
  }
}
```
::::

:::: after
::: ba
После
:::

``` {.code lang="csharp"}
class Report 
{
  // ...
  void SendReport() 
  {
    DateTime nextDay = NextDay(previousEnd);
    // ...
  }
  private static DateTime NextDay(DateTime date) 
  {
    return date.AddDays(1);
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
class Report {
  // ...
  public function sendReport() {
    $previousDate = clone $this->previousDate;
    $paymentDate = $previousDate->modify("+7 days");
    // ...
  }
}
```
::::

:::: after
::: ba
После
:::

``` {.code lang="php"}
class Report {
  // ...
  public function sendReport() {
    $paymentDate = self::nextWeek($this->previousDate);
    // ...
  }
  /**
   * Foreign method. Should be in Date.
   */
  private static function nextWeek(DateTime $arg) {
    $previousDate = clone $arg;
    return $previousDate->modify("+7 days");
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
class Report:
    # ...
    def sendReport(self):
        nextDay = Date(self.previousEnd.getYear(),
            self.previousEnd.getMonth(), self.previousEnd.getDate() + 1)
        # ...
```
::::

:::: after
::: ba
После
:::

``` {.code lang="python"}
class Report:
    # ...
    def sendReport(self):
        newStart = self._nextDay(self.previousEnd)
        # ...
        
    def _nextDay(self, arg):
        return Date(arg.getYear(), arg.getMonth(), arg.getDate() + 1)
```
::::
:::::::

::::::: {.before-after .before-after-container .typescript data-lang="typescript"}
:::: before
::: ba
До
:::

``` {.code lang="typescript"}
class Report {
  // ...
  sendReport(): void {
    let nextDay: Date = new Date(previousEnd.getYear(),
      previousEnd.getMonth(), previousEnd.getDate() + 1);
    // ...
  }
}
```
::::

:::: after
::: ba
После
:::

``` {.code lang="typescript"}
class Report {
  // ...
  sendReport() {
    let newStart: Date = nextDay(previousEnd);
    // ...
  }
  private static nextDay(arg: Date): Date {
    return new Date(arg.getFullYear(), arg.getMonth(), arg.getDate() + 1);
  }
}
```
::::
:::::::
::::::::::::::::::::::::::::

### Причины рефакторинга

У вас есть код, который использует данные и методы определённого класса.
Вы приходите к выводу, что этот код намного лучше будет смотреться и
работать внутри нового метода в этом классе. Однако возможность добавить
такой метод в класс у вас отсутствует (например, потому что класс
находится в сторонней библиотеке).

Данный рефакторинг особенно выгоден в случаях, когда участок кода,
который вы хотите перенести в метод, повторяется несколько раз в
различных местах программы.

Так как вы передаёте объект служебного класса в параметры нового метода,
у вас есть доступ ко всем его полям. Вы можете делать внутри этого
метода практически все, что вам может потребоваться, как если бы метод
был частью служебного класса.

### Достоинства

-   Убирает дублирование кода. Если ваш участок кода повторяется в
    нескольких местах, вы можете заменить их вызовом метода. Это удобнее
    дублирования даже с учетом того, что внешний метод находится не там,
    где хотелось бы.

### Недостатки

-   Причины того, почему метод служебного класса находится в клиентском
    классе, не всегда очевидны для того специалиста, который будет
    поддерживать код после вас. Если данный метод может быть использован
    и в других классах, имеет смысл создать обёртку над служебным
    классом, и поместить метод туда. То же самое имеет смысл сделать,
    если таких служебных методов несколько. В этом поможет рефакторинг
    [введение локального расширения](introduce-local-extension.html).

### Порядок рефакторинга

1.  Создайте новый метод в клиентском классе.

2.  В этом методе создайте параметр, в который будет передаваться объект
    служебного класса. Если этот объект может быть получен из
    клиентского класса, параметр можно не создавать.

3.  Извлеките волнующие вас участки кода в этот метод и замените их
    вызовами метода.

4.  Обязательно оставьте в комментарии к этому методу метку *Foreign
    method* и призыв поместить этот метод в служебный класс, если такая
    возможность появится в дальнейшем. Это облегчит понимание того,
    почему этот метод находится в данном классе для тех, кто будет
    поддерживать программный продукт в будущем.

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
[Введение локального расширения](introduce-local-extension.html)
:::

::: dd
Вынести все расширенные методы в отдельный класс обёртку/наследник
служебного класса.
:::
:::::
::::::

::::: {.col-xs-12 .col-sm-6 .col-lg-6}
### Борется с запахом

:::: dl
::: dt
[Неполнота библиотечного класса](smells/incomplete-library-class.html)
:::
::::
:::::
::::::::::

::: next
#### Читаем дальше

[Введение локального расширения []{.fa
.fa-arrow-right}](introduce-local-extension.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Удаление
посредника](remove-middle-man.html){.btn .btn-default rel="prev"}
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
