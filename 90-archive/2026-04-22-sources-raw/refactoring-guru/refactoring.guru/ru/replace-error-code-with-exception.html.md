::::::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Рефакторинг](refactoring.html){.type} /
[Приёмы](refactoring/techniques.html){.type} / [Упрощение вызовов
методов](refactoring/techniques/simplifying-method-calls.html){.type}
:::

# Замена кода ошибки исключением {#замена-кода-ошибки-исключением .title}

::: aka
Также известен как: [Replace Error Code with
Exception]{style="display:inline-block"}
:::

::::: before-after-container
::: before
### Проблема

Метод возвращает определенное значение, которое будет сигнализировать об
ошибке.
:::

::: after
### Решение

Вместо этого следует выбрасывать исключение.
:::
:::::

:::::::::::::::::::::::::::: examples
::::::: {.before-after .before-after-container .java data-lang="java"}
:::: before
::: ba
До
:::

``` {.code lang="java"}
int withdraw(int amount) {
  if (amount > _balance) {
    return -1;
  }
  else {
    balance -= amount;
    return 0;
  }
}
```
::::

:::: after
::: ba
После
:::

``` {.code lang="java"}
void withdraw(int amount) throws BalanceException {
  if (amount > _balance) {
    throw new BalanceException();
  }
  balance -= amount;
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
int Withdraw(int amount) 
{
  if (amount > _balance) 
  {
    return -1;
  }
  else 
  {
    balance -= amount;
    return 0;
  }
}
```
::::

:::: after
::: ba
После
:::

``` {.code lang="csharp"}
///<exception cref="BalanceException">Thrown when amount > _balance</exception>
void Withdraw(int amount)
{
  if (amount > _balance) 
  {
    throw new BalanceException();
  }
  balance -= amount;
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
function withdraw($amount) {
  if ($amount > $this->balance) {
    return -1;
  } else {
    $this->balance -= $amount;
    return 0;
  }
}
```
::::

:::: after
::: ba
После
:::

``` {.code lang="php"}
function withdraw($amount) {
  if ($amount > $this->balance) {
    throw new BalanceException;
  }
  $this->balance -= $amount;
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
def withdraw(self, amount):
    if amount > self.balance:
        return -1
    else:
        self.balance -= amount
    return 0
```
::::

:::: after
::: ba
После
:::

``` {.code lang="python"}
def withdraw(self, amount):
    if amount > self.balance:
        raise BalanceException()
    self.balance -= amount
```
::::
:::::::

::::::: {.before-after .before-after-container .typescript data-lang="typescript"}
:::: before
::: ba
До
:::

``` {.code lang="typescript"}
withdraw(amount: number): number {
  if (amount > _balance) {
    return -1;
  }
  else {
    balance -= amount;
    return 0;
  }
}
```
::::

:::: after
::: ba
После
:::

``` {.code lang="typescript"}
withdraw(amount: number): void {
  if (amount > _balance) {
    throw new Error();
  }
  balance -= amount;
}
```
::::
:::::::
::::::::::::::::::::::::::::

### Причины рефакторинга

Возвращение кодов ошибок --- давно устаревшая практика процедурного
программирования. В современном программировании для обработки ошибок
используются специальные классы, называемые исключениями. При
возникновении проблемы вы «выбрасываете» такое исключение и оно
впоследствии «ловится» одним из обработчиков исключений. При этом
запускается специальный код обработки внештатной ситуации, который
игнорируется в обычных условиях.

### Достоинства

-   Избавляет код от множества условных операторов проверки кодов
    ошибок. Обработчики исключений намного чётче разграничивают
    нормальный и нештатный путь исполнения программы.

-   Классы исключений могут реализовывать собственные методы, а значит
    содержать часть функциональности по обработке ошибок (например, для
    перевода сообщений об ошибках).

-   В отличие от исключений, коды ошибок не могут быть использованы в
    конструкторе, так как он должен возвращать только новый объект.

### Недостатки

-   Обработку исключений можно превратить в `goto`-подобный костыль. Не
    делайте так! Не используйте исключения для управления исполнением
    кода. Исключения следует выбрасывать только в целях сообщения об
    ошибке или критической ситуации.

### Порядок рефакторинга

Старайтесь выполнять шаги этого рефакторинга только для одного кода
ошибки за один раз. Так будет легче удержать в голове все важные
сведения и избежать ошибок.

1.  Найдите все вызовы метода, возвращающего код ошибки, и оберните его
    в `try`/`catch` блоки вместо проверки кода ошибки.

2.  Внутри метода вместо возвращения кода ошибки выбрасывайте
    исключение.

3.  Измените сигнатуру метода так, чтобы она содержала информацию о
    выбрасываемом исключении (секция `@throws`).

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

::: next
#### Читаем дальше

[Замена исключения проверкой условия []{.fa
.fa-arrow-right}](replace-exception-with-test.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Замена конструктора фабричным
методом](replace-constructor-with-factory-method.html){.btn .btn-default
rel="prev"}
:::
:::::::::::::::::::::::::::::::::::::::

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
::::::::::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::::::::::
