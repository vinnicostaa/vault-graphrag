::::::::::::::::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Рефакторинг](refactoring.html){.type} /
[Прийоми рефакторингу](refactoring/techniques.html){.type} / [Складання
методів](refactoring/techniques/composing-methods.html){.type}
:::

# Відокремлення змінної {#відокремлення-змінної .title}

::: aka
Також відомий як: [Extract Variable]{style="display:inline-block"}
:::

::::: before-after-container
::: before
### Проблема

У вас є складний для розуміння вираз.
:::

::: after
### Рішення

Помістіть результат виразу або його частини в окремі змінні, що
пояснюють суть виразу.
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
Після
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
Після
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
Після
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
Після
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
Після
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

### Причини рефакторингу

Головна мотивація цього рефакторингу --- зробити зрозумілішим складний
вираз, розбивши його на проміжні частини. Це може бути:

-   Умова оператора `if ()` або частини оператора `?:` у C-подібних
    мовах.

-   Довгий арифметичний вираз без проміжних результатів.

-   Довге склеювання рядків.

Виділення змінної може стати першим кроком до подальшого [відокремлення
методу](extract-method.html), якщо ви побачите, що виділений вираз
використовується і в інших місцях коду.

### Переваги

-   Покращує читабельність коду. Намагайтеся дати виділеним змінним
    хороші назви, які точно відображатимуть суть виразу. Так ви зробите
    код читабельним і зможете позбутися від зайвих коментарів.
    Наприклад, `customerTaxValue`, `cityUnemploymentRate`,
    `clientSalutationString` і так далі.

### Недоліки

-   З'являються додаткові змінні. Але цей мінус компенсується простотою
    читання коду.

-   Під час рефакторингу виразів умовних операторів пам'ятайте про те,
    що програма зазвичай оптимізує виконання цих виразів і не виконує
    подальші перевірки, якщо вже можна передбачити фінальний результат.
    Наприклад, якщо у виразі `if (a () || b ()) ...` метод `a` поверне
    `true`, то програма не стане виконувати `b`, позаяк що б він не
    повернув, результат все одно буде істинним.

    Одначе, після вилучення частин цього виразу в змінні, обидва методи
    будуть викликатися постійно, що може негативно позначитися на
    швидкодії, особливо якщо ці методи виконують якусь ресурсоємну
    роботу.

### Порядок рефакторингу

1.  Додайте новий рядок перед виразом, що вас цікавить, і оголосіть там
    нову змінну. Присвойте цій змінній частину складного виразу.

2.  Замініть частину винесеного виразу новою змінною.

3.  Повторіть це для всіх складних частин виразу.

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

:::::::::::: row
::::: {.col-xs-12 .col-sm-6 .col-lg-4}
### Анти-рефакторинг

:::: dl
::: dt
[Вбудовування змінної](inline-temp.html)
:::
::::
:::::

::::: {.col-xs-12 .col-sm-6 .col-lg-4}
### Схожі рефакторинги

:::: dl
::: dt
[Відокремлення методу](extract-method.html)
:::
::::
:::::

::::: {.col-xs-12 .col-sm-6 .col-lg-4}
### Бореться з запахом

:::: dl
::: dt
[Коментарі](smells/comments.html)
:::
::::
:::::
::::::::::::

::: next
#### Читаємо далі

[Вбудовування змінної []{.fa .fa-arrow-right}](inline-temp.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Повернутись назад

[[]{.fa .fa-arrow-left} Вбудовування методу](inline-method.html){.btn
.btn-default rel="prev"}
:::
:::::::::::::::::::::::::::::::::::::::::::::::::

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
::::::::::::::::::::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::::::::::::::::::::
