:::::::::::::::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Рефакторинг](refactoring.html){.type} /
[Приёмы](refactoring/techniques.html){.type} / [Составление
методов](refactoring/techniques/composing-methods.html){.type}
:::

# Замена переменной вызовом метода {#замена-переменной-вызовом-метода .title}

::: aka
Также известен как: [Replace Temp with
Query]{style="display:inline-block"}
:::

::::: before-after-container
::: before
### Проблема

Вы помещаете результат какого-то выражения в локальную переменную, чтобы
использовать её далее в коде.
:::

::: after
### Решение

Выделите все выражение в отдельный метод и возвращайте результат из
него. Замените использование вашей переменной вызовом метода. Новый
метод может быть использован и в других методах.
:::
:::::

:::::::::::::::::::::::::::: examples
::::::: {.before-after .before-after-container .java data-lang="java"}
:::: before
::: ba
До
:::

``` {.code lang="java"}
double calculateTotal() {
  double basePrice = quantity * itemPrice;
  if (basePrice > 1000) {
    return basePrice * 0.95;
  }
  else {
    return basePrice * 0.98;
  }
}
```
::::

:::: after
::: ba
После
:::

``` {.code lang="java"}
double calculateTotal() {
  if (basePrice() > 1000) {
    return basePrice() * 0.95;
  }
  else {
    return basePrice() * 0.98;
  }
}
double basePrice() {
  return quantity * itemPrice;
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
double CalculateTotal() 
{
  double basePrice = quantity * itemPrice;
  
  if (basePrice > 1000) 
  {
    return basePrice * 0.95;
  }
  else 
  {
    return basePrice * 0.98;
  }
}
```
::::

:::: after
::: ba
После
:::

``` {.code lang="csharp"}
double CalculateTotal() 
{
  if (BasePrice() > 1000) 
  {
    return BasePrice() * 0.95;
  }
  else 
  {
    return BasePrice() * 0.98;
  }
}
double BasePrice() 
{
  return quantity * itemPrice;
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
$basePrice = $this->quantity * $this->itemPrice;
if ($basePrice > 1000) {
  return $basePrice * 0.95;
} else {
  return $basePrice * 0.98;
}
```
::::

:::: after
::: ba
После
:::

``` {.code lang="php"}
if ($this->basePrice() > 1000) {
  return $this->basePrice() * 0.95;
} else {
  return $this->basePrice() * 0.98;
}

...

function basePrice() {
  return $this->quantity * $this->itemPrice;
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
def calculateTotal():
    basePrice = quantity * itemPrice
    if basePrice > 1000:
        return basePrice * 0.95
    else:
        return basePrice * 0.98
```
::::

:::: after
::: ba
После
:::

``` {.code lang="python"}
def calculateTotal():
    if basePrice() > 1000:
        return basePrice() * 0.95
    else:
        return basePrice() * 0.98

def basePrice():
    return quantity * itemPrice
```
::::
:::::::

::::::: {.before-after .before-after-container .typescript data-lang="typescript"}
:::: before
::: ba
До
:::

``` {.code lang="typescript"}
 calculateTotal(): number {
  let basePrice = quantity * itemPrice;
  if (basePrice > 1000) {
    return basePrice * 0.95;
  }
  else {
    return basePrice * 0.98;
  }
}
```
::::

:::: after
::: ba
После
:::

``` {.code lang="typescript"}
calculateTotal(): number {
  if (basePrice() > 1000) {
    return basePrice() * 0.95;
  }
  else {
    return basePrice() * 0.98;
  }
}
basePrice(): number {
  return quantity * itemPrice;
}
```
::::
:::::::
::::::::::::::::::::::::::::

### Причины рефакторинга

Применение данного рефакторинга может быть подготовительным этапом для
применения [выделения метода](extract-method.html) для какой-то части
очень длинного метода.

Кроме того, иногда можно найти это же выражение и в других методах, что
заставляет задуматься о создании общего метода для его получения.

### Достоинства

-   Улучшает читабельность кода. Намного проще понять, что делает метод
    `getTax()` чем строка `orderPrice() * -2`.

-   Помогает убрать дублирование кода, если заменяемая строка
    используется более чем в одном методе.

### Полезные факты

#### Вопрос производительности

При использовании этого рефакторинга может возникнуть вопрос, не
скажется ли результат рефакторинга не лучшим образом на
производительности программы. Честный ответ --- да, результирующий код
может получить дополнительную нагрузку за счёт вызова нового метода.
Однако в наше время быстрых процессоров и хороших компиляторов такая
нагрузка вряд ли будет заметна. Зато взамен мы получаем лучшую
читабельность кода и возможность использовать новый метод в других
местах программы.

Тем не менее, если ваша временная переменная служит для кеширования
результата действительно трудоёмкого выражения, имеет смысл остановить
этот рефакторинг после выделения выражения в новый метод.

### Порядок рефакторинга

1.  Убедитесь, что переменной в пределах метода присваивается значение
    только один раз. Если это не так, используйте [расщепление
    переменной](split-temporary-variable.html) для того, чтобы
    гарантировать, что переменная будет использована только для хранения
    результата вашего выражения.

2.  Используйте [извлечение метода](extract-method.html) для того, чтобы
    переместить интересующее вас выражение в новый метод. Убедитесь, что
    этот метод только возвращает значение и не меняет состояние объекта.
    Если он как-то влияет на видимое состояние объекта, используйте
    [разделение запроса и
    модификатора](separate-query-from-modifier.html).

3.  Замените использование переменной вызовом вашего нового метода.

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
::::: {.col-xs-12 .col-sm-6 .col-lg-6}
### Родственные рефакторинги

:::: dl
::: dt
[Извлечение метода](extract-method.html)
:::
::::
:::::

::::::: {.col-xs-12 .col-sm-6 .col-lg-6}
### Борется с запахом

:::: dl
::: dt
[Длинный метод](smells/long-method.html)
:::
::::

:::: dl
::: dt
[Дублирование кода](smells/duplicate-code.html)
:::
::::
:::::::
:::::::::::

::: next
#### Читаем дальше

[Расщепление переменной []{.fa
.fa-arrow-right}](split-temporary-variable.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Встраивание переменной](inline-temp.html){.btn
.btn-default rel="prev"}
:::
::::::::::::::::::::::::::::::::::::::::::::::::

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
:::::::::::::::::::::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::::::::::::::::::::
