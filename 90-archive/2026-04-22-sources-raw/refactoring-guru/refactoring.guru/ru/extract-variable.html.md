::::::::::::::::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Рефакторинг](refactoring.html){.type} /
[Приёмы](refactoring/techniques.html){.type} / [Составление
методов](refactoring/techniques/composing-methods.html){.type}
:::

# Извлечение переменной {#извлечение-переменной .title}

::: aka
Также известен как: [Extract Variable]{style="display:inline-block"}
:::

::::: before-after-container
::: before
### Проблема

У вас есть сложное для понимания выражение.
:::

::: after
### Решение

Поместите результат выражения или его части в отдельные переменные,
поясняющие суть выражения.
:::
:::::

:::::::::::::::::::::::::::: examples
::::::: {.before-after .before-after-container .java data-lang="java"}
:::: before
::: ba
До
:::

``` {.code lang="java"}
void renderBanner() {
  if ((platform.toUpperCase().indexOf("MAC") > -1) &&
       (browser.toUpperCase().indexOf("IE") > -1) &&
        wasInitialized() && resize > 0 )
  {
    // do something
  }
}
```
::::

:::: after
::: ba
После
:::

``` {.code lang="java"}
void renderBanner() {
  final boolean isMacOs = platform.toUpperCase().indexOf("MAC") > -1;
  final boolean isIE = browser.toUpperCase().indexOf("IE") > -1;
  final boolean wasResized = resize > 0;

  if (isMacOs && isIE && wasInitialized() && wasResized) {
    // do something
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
void RenderBanner() 
{
  if ((platform.ToUpper().IndexOf("MAC") > -1) &&
       (browser.ToUpper().IndexOf("IE") > -1) &&
        wasInitialized() && resize > 0 )
  {
    // do something
  }
}
```
::::

:::: after
::: ba
После
:::

``` {.code lang="csharp"}
void RenderBanner() 
{
  readonly bool isMacOs = platform.ToUpper().IndexOf("MAC") > -1;
  readonly bool isIE = browser.ToUpper().IndexOf("IE") > -1;
  readonly bool wasResized = resize > 0;

  if (isMacOs && isIE && wasInitialized() && wasResized) 
  {
    // do something
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
if (($platform->toUpperCase()->indexOf("MAC") > -1) &&
     ($browser->toUpperCase()->indexOf("IE") > -1) &&
      $this->wasInitialized() && $this->resize > 0)
{
  // do something
}
```
::::

:::: after
::: ba
После
:::

``` {.code lang="php"}
$isMacOs = $platform->toUpperCase()->indexOf("MAC") > -1;
$isIE = $browser->toUpperCase()->indexOf("IE")  > -1;
$wasResized = $this->resize > 0;

if ($isMacOs && $isIE && $this->wasInitialized() && $wasResized) {
  // do something
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
def renderBanner(self):
    if (self.platform.toUpperCase().indexOf("MAC") > -1) and \
       (self.browser.toUpperCase().indexOf("IE") > -1) and \
       self.wasInitialized() and (self.resize > 0):
        # do something
```
::::

:::: after
::: ba
После
:::

``` {.code lang="python"}
def renderBanner(self):
    isMacOs = self.platform.toUpperCase().indexOf("MAC") > -1
    isIE = self.browser.toUpperCase().indexOf("IE") > -1
    wasResized = self.resize > 0

    if isMacOs and isIE and self.wasInitialized() and wasResized:
        # do something
```
::::
:::::::

::::::: {.before-after .before-after-container .typescript data-lang="typescript"}
:::: before
::: ba
До
:::

``` {.code lang="typescript"}
renderBanner(): void {
  if ((platform.toUpperCase().indexOf("MAC") > -1) &&
       (browser.toUpperCase().indexOf("IE") > -1) &&
        wasInitialized() && resize > 0 )
  {
    // do something
  }
}
```
::::

:::: after
::: ba
После
:::

``` {.code lang="typescript"}
renderBanner(): void {
  const isMacOs = platform.toUpperCase().indexOf("MAC") > -1;
  const isIE = browser.toUpperCase().indexOf("IE") > -1;
  const wasResized = resize > 0;

  if (isMacOs && isIE && wasInitialized() && wasResized) {
    // do something
  }
}
```
::::
:::::::
::::::::::::::::::::::::::::

### Причины рефакторинга

Главная мотивация этого рефакторинга --- сделать более понятным сложное
выражение, разбив его на промежуточные части. Это может быть:

-   Условие оператора `if()` или части оператора `?:` в C-подобных
    языках.

-   Длинное арифметическое выражение без промежуточных результатов.

-   Длинное склеивание строк.

Выделение переменной может стать первым шагом к последующему [извлечению
метода](extract-method.html), если вы увидите, что выделенное выражение
используется и в других местах кода.

### Достоинства

-   Улучшает читабельность кода. Постарайтесь дать выделенным переменным
    хорошие названия, которые будут отражать точно суть выражения. Так
    вы сделаете код читабельным и сумеете избавиться от лишних
    комментариев. Например, `customerTaxValue`, `cityUnemploymentRate`,
    `clientSalutationString` и т. д.

### Недостатки

-   Появляются дополнительные переменные. Но этот минус компенсируется
    простотой чтения кода.

-   При рефакторинге выражений условных операторов помните о том, что
    программа обычно оптимизирует выполнение условных выражений и не
    выполняет дальнейшие проверки, если уже понятен финальный результат
    выражения. Например, если в выражении `if (a() || b()) ...` метод
    `a` вернет `true`, то программа не станет выполнять `b` --- что бы
    он ни вернул, результирующее выражение всё равно будет истинным.

    Однако, после извлечения частей этого выражения в переменные, оба
    метода будут вызываться постоянно, что может негативно сказаться на
    производительности, особенно если эти методы выполняют какую-то
    ресурсоёмкую работу.

### Порядок рефакторинга

1.  Вставьте новую строку перед интересующим вас выражением и объявите
    там новую переменную. Присвойте этой переменной часть сложного
    выражения.

2.  Замените часть вынесенного выражения новой переменной.

3.  Повторите это для всех сложных частей выражения.

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

:::::::::::: row
::::: {.col-xs-12 .col-sm-6 .col-lg-4}
### Анти-рефакторинг

:::: dl
::: dt
[Встраивание переменной](inline-temp.html)
:::
::::
:::::

::::: {.col-xs-12 .col-sm-6 .col-lg-4}
### Родственные рефакторинги

:::: dl
::: dt
[Извлечение метода](extract-method.html)
:::
::::
:::::

::::: {.col-xs-12 .col-sm-6 .col-lg-4}
### Борется с запахом

:::: dl
::: dt
[Комментарии](smells/comments.html)
:::
::::
:::::
::::::::::::

::: next
#### Читаем дальше

[Встраивание переменной []{.fa .fa-arrow-right}](inline-temp.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Встраивание метода](inline-method.html){.btn
.btn-default rel="prev"}
:::
:::::::::::::::::::::::::::::::::::::::::::::::::

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
::::::::::::::::::::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::::::::::::::::::::
