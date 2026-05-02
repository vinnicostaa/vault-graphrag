::::::::::::::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Рефакторинг](refactoring.html){.type} /
[Прийоми рефакторингу](refactoring/techniques.html){.type} /
[Переміщення функцій між
об\'єктами](refactoring/techniques/moving-features-between-objects.html){.type}
:::

# Введення зовнішнього методу {#введення-зовнішнього-методу .title}

::: aka
Також відомий як: [Introduce Foreign
Method]{style="display:inline-block"}
:::

::::: before-after-container
::: before
### Проблема

Службовий клас не містить методу, який вам потрібен, при цьому у вас
немає можливості додати метод в цей клас.
:::

::: after
### Рішення

Додайте метод в клієнтський клас і передавайте в нього об'єкт службового
класу в якості аргументу.
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
Після
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
Після
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
Після
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
Після
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
Після
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

### Причини рефакторингу

У вас є код, який використовує дані і методи певного класу. Ви робите
висновок, що цей код набагато краще буде виглядати і працювати всередині
нового методу в цьому класі. Проте можливості додати такий метод в клас
у вас немає (наприклад, тому що клас знаходиться в сторонній
бібліотеці).

Цей рефакторинг найвигідніше використовувати у випадках, коли ділянка
коду, який ви хочете перенести в метод, повторюється кілька разів в
різних місцях програми.

Оскільки ви передаєте об'єкт службового класу в параметри нового методу,
у вас є доступ до всіх його полів. Ви можете робити всередині цього
методу практично все, що вам може бути потрібно, начебто метод є
частиною службового класу.

### Переваги

-   Прибирає дублювання коду. Якщо ваша ділянка коду повторюється в
    декількох місцях, ви можете замінити її викликом методу. Це зручніше
    за дублювання навіть з урахуванням того, що зовнішній метод
    знаходиться не там, де б хотілось.

### Недоліки

-   Причини того, чому метод службового класу знаходиться в клієнтському
    класі не завжди очевидні для того фахівця, який підтримуватиме код
    після вас. Якщо цей метод може бути використаний і в інших класах,
    має сенс створити обгортку над службовим класом, і помістити метод
    туди. Те ж саме має сенс зробити, якщо таких службових методів
    декілька. У цьому допоможе рефакторинг [введення локального
    розширення](introduce-local-extension.html).

### Порядок рефакторингу

1.  Створіть новий метод в клієнтському класі.

2.  У цьому методі створіть параметр, в який передаватиметься об'єкт
    службового класу. Якщо цей об'єкт може бути отриманий з клієнтського
    класу, параметр можна не створювати.

3.  Витягніть ділянки коду, які вам потрібні, в цей метод і замініть їх
    викликами методу.

4.  Обов'язково залиште в коментарі до цього методу мітку *Foreight
    method* і заклик перенести цей метод в службовий клас, якщо така
    можливість з'явиться надалі. Це полегшить розуміння того, чому метод
    знаходиться в цьому класі для тих, хто підтримуватиме програмний
    продукт в майбутньому.

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

:::::::::: row
:::::: {.col-xs-12 .col-sm-6 .col-lg-6}
### Схожі рефакторинги

::::: dl
::: dt
[Введення локального розширення](introduce-local-extension.html)
:::

::: dd
Винести всі розширені методи в окремий клас-обгортку або нащадок
службового класу.
:::
:::::
::::::

::::: {.col-xs-12 .col-sm-6 .col-lg-6}
### Бореться з запахом

:::: dl
::: dt
[Неповнота бібліотечного класу](smells/incomplete-library-class.html)
:::
::::
:::::
::::::::::

::: next
#### Читаємо далі

[Введення локального розширення []{.fa
.fa-arrow-right}](introduce-local-extension.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Повернутись назад

[[]{.fa .fa-arrow-left} Видалення
посередника](remove-middle-man.html){.btn .btn-default rel="prev"}
:::
:::::::::::::::::::::::::::::::::::::::::::::::

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
::::::::::::::::::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::::::::::::::::::
