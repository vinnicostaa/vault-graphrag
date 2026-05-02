::::::::::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Рефакторинг](refactoring.html){.type} /
[Прийоми рефакторингу](refactoring/techniques.html){.type} / [Спрощення
умовних
виразів](refactoring/techniques/simplifying-conditional-expressions.html){.type}
:::

# Введення перевірки твердження {#введення-перевірки-твердження .title}

::: aka
Також відомий як: [Introduce Assertion]{style="display:inline-block"}
:::

> Тут і надалі під перевірками маються на увазі виклики функці `assert`.

::::: before-after-container
::: before
### Проблема

Коректна робота ділянки коду припускає наявність якихось певних умов або
значень.
:::

::: after
### Рішення

Замініть ці припущення конкретними перевірками.
:::
:::::

:::::::::::::::::::::::::::: examples
::::::: {.before-after .before-after-container .java data-lang="java"}
:::: before
::: ba
До
:::

``` {.code lang="java"}
double getExpenseLimit() {
  // Should have either expense limit or
  // a primary project.
  return (expenseLimit != NULL_EXPENSE) ?
    expenseLimit :
    primaryProject.getMemberExpenseLimit();
}
```
::::

:::: after
::: ba
Після
:::

``` {.code lang="java"}
double getExpenseLimit() {
  Assert.isTrue(expenseLimit != NULL_EXPENSE || primaryProject != null);

  return (expenseLimit != NULL_EXPENSE) ?
    expenseLimit:
    primaryProject.getMemberExpenseLimit();
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
double GetExpenseLimit() 
{
  // Should have either expense limit or
  // a primary project.
  return (expenseLimit != NULL_EXPENSE) ?
    expenseLimit :
    primaryProject.GetMemberExpenseLimit();
}
```
::::

:::: after
::: ba
Після
:::

``` {.code lang="csharp"}
double GetExpenseLimit() 
{
  Assert.IsTrue(expenseLimit != NULL_EXPENSE || primaryProject != null);

  return (expenseLimit != NULL_EXPENSE) ?
    expenseLimit:
    primaryProject.GetMemberExpenseLimit();
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
function getExpenseLimit() {
  // Should have either expense limit or
  // a primary project.
  return ($this->expenseLimit !== NULL_EXPENSE) ?
    $this->expenseLimit:
    $this->primaryProject->getMemberExpenseLimit();
}
```
::::

:::: after
::: ba
Після
:::

``` {.code lang="php"}
function getExpenseLimit() {
  assert($this->expenseLimit !== NULL_EXPENSE || isset($this->primaryProject));

  return ($this->expenseLimit !== NULL_EXPENSE) ?
    $this->expenseLimit:
    $this->primaryProject->getMemberExpenseLimit();
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
def getExpenseLimit(self):
    # Should have either expense limit or
    # a primary project.
    return self.expenseLimit if self.expenseLimit != NULL_EXPENSE else \
        self.primaryProject.getMemberExpenseLimit()
```
::::

:::: after
::: ba
Після
:::

``` {.code lang="python"}
def getExpenseLimit(self):
    assert (self.expenseLimit != NULL_EXPENSE) or (self.primaryProject != None)

    return self.expenseLimit if (self.expenseLimit != NULL_EXPENSE) else \
        self.primaryProject.getMemberExpenseLimit()
```
::::
:::::::

::::::: {.before-after .before-after-container .typescript data-lang="typescript"}
:::: before
::: ba
До
:::

``` {.code lang="typescript"}
getExpenseLimit(): number {
  // Should have either expense limit or
  // a primary project.
  return (expenseLimit != NULL_EXPENSE) ?
    expenseLimit:
    primaryProject.getMemberExpenseLimit();
}
```
::::

:::: after
::: ba
Після
:::

``` {.code lang="typescript"}
getExpenseLimit(): number {
  // TypeScript and JS doesn't have built-in assertions, so we'll use
  // good-old console.error(). You can always extract this into a
  // designated assertion function.
  if (!(expenseLimit != NULL_EXPENSE ||
       (typeof primaryProject !== 'undefined' && primaryProject))) {
      console.error("Assertion failed: getExpenseLimit()");
  }

  return (expenseLimit != NULL_EXPENSE) ?
    expenseLimit:
    primaryProject.getMemberExpenseLimit();
}
```
::::
:::::::
::::::::::::::::::::::::::::

### Причини рефакторингу

Ділянка коду створює припущення про щось (наприклад, про поточний стан
об'єкта, значення параметра або локальної змінної). Зазвичай це
припущення ніколи не буде порушено хіба що за наявності якоїсь помилки.

Зробіть ці припущення явними, додавши відповідні перевірки. Як і
підказки типів в параметрах методів, ці перевірки можуть грати роль
живої документації коду.

Хорошою ознакою того, що потрібна перевірка якихось умов, будуть
коментарі в коді. Якщо ви бачите коментарі, що описують умови, при яких
метод коректно працюватиме, значить тут непогано було б вставити
перевірку-твердження цих умов.

### Переваги

-   Якщо якесь припущення невірне і в результаті код дає також невірний
    результат, краще відразу зупинити виконання, поки це не привело до
    фатальних наслідків і псування даних. Крім того, це означає, що ви
    забули написати якийсь тест в наборі тестів вашої програми.

### Недоліки

-   Іноді замість простої перевірки краще вставити виключення. При цьому
    ви можете вибрати необхідний клас виключення і дати іншому коду
    можливість коректно його обробити.

-   В яких випадках виключення краще простої перевірки? Якщо до нього
    може привести діяльність користувача або системи, і ви можете
    обробити це виключення. З іншого боку, звичайні безіменні
    необроблювані виключення роблять теж саме, що й перевірки --- ви їх
    також не обробляєте, вони можуть бути викликані тільки як результат
    бага в програмі, який ніколи не повинен був виникнути.

### Порядок рефакторингу

Коли ви розумієте, що передбачається якась умова, додайте перевірку цієї
умови, щоби перевірити її напевно.

Додавання перевірки не повинне змінювати поведінку програми.

Не переборщуйте з використанням перевірок для **всього**, що тільки
можна, на вибраній ділянці коду. Треба перевіряти лише ті умови, без
яких код перестане коректно працювати. Якщо ваш код нормально працює, у
тому числі, у разі, коли перевірка не проходить успішно, таку перевірку
можна прибрати.

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

:::::: row
::::: {.col-xs-12 .col-sm-6 .col-lg-12}
### Бореться з запахом

:::: dl
::: dt
[Коментарі](smells/comments.html)
:::
::::
:::::
::::::

::: next
#### Читаємо далі

[Спрощення викликів методів []{.fa
.fa-arrow-right}](refactoring/techniques/simplifying-method-calls.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Повернутись назад

[[]{.fa .fa-arrow-left} Введення
Null-об\'єкта](introduce-null-object.html){.btn .btn-default rel="prev"}
:::
:::::::::::::::::::::::::::::::::::::::::::

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
::::::::::::::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::::::::::::::
