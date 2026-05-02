::::::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Рефакторинг](refactoring.html){.type} /
[Прийоми рефакторингу](refactoring/techniques.html){.type} / [Спрощення
викликів
методів](refactoring/techniques/simplifying-method-calls.html){.type}
:::

# Заміна коду помилки виключенням {#заміна-коду-помилки-виключенням .title}

::: aka
Також відомий як: [Replace Error Code with
Exception]{style="display:inline-block"}
:::

::::: before-after-container
::: before
### Проблема

Метод повертає певне значення, яке сигналізуватиме про помилку.
:::

::: after
### Рішення

Замість цього слід викидати виключення.
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
Після
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
Після
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
Після
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
Після
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
Після
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

### Причини рефакторингу

Повернення кодів помилок --- давно застаріла практика процедурного
програмування. У сучасному програмуванні для обробки помилок
використовуються спеціальні класи, що називаються виключеннями. При
виникненні проблеми ви «викидаєте» таке виключення і воно згодом
\"ловиться\" одним з обробників виключень. При цьому запускається
спеціальний код обробки позаштатної ситуації, який ігнорується в
звичайних умовах.

### Переваги

-   Позбавляє код від безлічі умовних операторів перевірки кодів
    помилок. Обробники виключень набагато чіткіше розмежовують
    нормальний і нештатний шлях виконання програми.

-   Класи виключень можуть реалізовувати власні методи, а значить
    містити частину функціональності по обробці помилок (наприклад, для
    перекладу повідомлень про помилки).

-   На відміну від виключень, коди помилок не можуть бути використані в
    конструкторі, оскільки він повинен повертати тільки новий об'єкт.

### Недоліки

-   Обробку виключень можна перетворити на `goto`-подібний костиль. Не
    робіть так! Не використовуйте виключення для управління виконанням
    коду. Виключення слід викидати тільки з метою повідомлення про
    помилку або критичну ситуацію.

### Порядок рефакторингу

Намагайтеся виконувати кроки цього рефакторингу тільки для одного коду
помилки за один раз. Так буде легше утримати в голові усі важливі
відомості і уникнути помилок.

1.  Знайдіть усі виклики методу, що повертає код помилки, і оберніть
    його в `try`/`catch` блоки замість перевірки коду помилки.

2.  Усередині методу замість повернення коду помилки викидайте
    виключення.

3.  Змініть сигнатуру методу так, щоб вона містила інформацію про
    виключення, що викидається (секція `@throws`).

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

::: next
#### Читаємо далі

[Заміна виключення перевіркою умови []{.fa
.fa-arrow-right}](replace-exception-with-test.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Повернутись назад

[[]{.fa .fa-arrow-left} Заміна конструктора фабричним
методом](replace-constructor-with-factory-method.html){.btn .btn-default
rel="prev"}
:::
:::::::::::::::::::::::::::::::::::::::

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
::::::::::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::::::::::
