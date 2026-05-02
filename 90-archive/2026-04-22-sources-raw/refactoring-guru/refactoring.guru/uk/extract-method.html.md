:::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Рефакторинг](refactoring.html){.type} /
[Прийоми рефакторингу](refactoring/techniques.html){.type} / [Складання
методів](refactoring/techniques/composing-methods.html){.type}
:::

# Відокремлення методу {#відокремлення-методу .title}

::: aka
Також відомий як: [Extract Method]{style="display:inline-block"}
:::

::::: before-after-container
::: before
### Проблема

У вас є фрагмент коду, який можна згрупувати.
:::

::: after
### Рішення

Виділіть цей фрагмент в новий метод (чи функцію) і викличте його замість
старого коду.
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
Після
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
Після
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
Після
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
Після
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
Після
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

### Причини рефакторингу

Чим більше коду в методі, тим складніше розібратися в тому, що він
робить. Це основна проблема, яку вирішує даний рефакторинг.

Відокремлення методу не лише вбиває безліч запахів в коді, але й є одним
з етапів безлічі інших рефакторингів.

### Переваги

-   Покращує читабельність коду. Намагайтеся дати новому методу таку
    назву, яка б пояснювала суть того, що він робить. Наприклад,
    `createOrder()`, `renderCustomerInfo()` і так далі.

-   Прибирає дублювання коду. Іноді код, винесений в метод, можна знайти
    і в інших місцях програми. У такому разі є сенс замінити знайдені
    ділянки коду викликом вашого нового методу.

-   Ізолює незалежні частини коду, зменшуючи ймовірність помилок
    (наприклад, з вини перепризначення не тієї змінної).

### Порядок рефакторингу

1.  Створіть новий метод. Потурбуйтеся, щоб його назва відбивала суть
    того, що робитиме цей метод.

2.  Скопіюйте фрагмент коду, що вас цікавить, в новий метод. Видаліть
    цей фрагмент із старого місця і замініть викликом вашого нового
    методу.

    Знайдіть усі змінні, які використовувалися в цьому фрагменті коду.
    Якщо вони були оголошені всередині цього фрагменту і не
    використовуються поза ним, просто залиште їх без змін --- вони
    стануть локальними змінними нового методу.

3.  Якщо змінні оголошені перед ділянкою коду, що вас цікавить, значить,
    їх слід передати в параметри вашого нового методу, щоб використати
    значення, які в них знаходилися раніше. Іноді від таких змінних
    простіше позбутися за допомогою [заміни змінних викликом
    методу](replace-temp-with-query.html)

4.  Якщо ви бачите, що локальна змінна якось змінюється у вашій ділянці
    коду, це може означати, що її змінене значення знадобиться далі в
    основному методі. Перевірте це. Якщо підозра підтвердилася, значення
    цієї змінної слід повернути в основний метод, щоб нічого не зламати.

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

::::::::::::::::::::::::::::::: row
::::: {.col-xs-12 .col-sm-6 .col-lg-3}
### Анти-рефакторинг

:::: dl
::: dt
[Вбудовування методу](inline-method.html)
:::
::::
:::::

::::: {.col-xs-12 .col-sm-6 .col-lg-3}
### Схожі рефакторинги

:::: dl
::: dt
[Переміщення методу](move-method.html)
:::
::::
:::::

::::::::: {.col-xs-12 .col-sm-6 .col-lg-3}
### Домогає іншим рефакторингам

:::: dl
::: dt
[Заміна параметрів об\'єктом](introduce-parameter-object.html)
:::
::::

:::: dl
::: dt
[Створення шаблонного методу](form-template-method.html)
:::
::::

:::: dl
::: dt
[Параметризація методу](parameterize-method.html)
:::
::::
:::::::::

::::::::::::::::: {.col-xs-12 .col-sm-6 .col-lg-3}
### Бореться з запахом

:::: dl
::: dt
[Дублювання коду](smells/duplicate-code.html)
:::
::::

:::: dl
::: dt
[Довгий метод](smells/long-method.html)
:::
::::

:::: dl
::: dt
[Заздрісні функції](smells/feature-envy.html)
:::
::::

:::: dl
::: dt
[Оператори switch](smells/switch-statements.html)
:::
::::

:::: dl
::: dt
[Ланцюжок викликів](smells/message-chains.html)
:::
::::

:::: dl
::: dt
[Коментарі](smells/comments.html)
:::
::::

:::: dl
::: dt
[Клас даних](smells/data-class.html)
:::
::::
:::::::::::::::::
:::::::::::::::::::::::::::::::

::: next
#### Читаємо далі

[Вбудовування методу []{.fa .fa-arrow-right}](inline-method.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Повернутись назад

[[]{.fa .fa-arrow-left} Складання
методів](refactoring/techniques/composing-methods.html){.btn
.btn-default rel="prev"}
:::
::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::

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
:::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::
