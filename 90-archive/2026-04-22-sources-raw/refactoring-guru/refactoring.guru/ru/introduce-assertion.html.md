::::::::::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Рефакторинг](refactoring.html){.type} /
[Приёмы](refactoring/techniques.html){.type} / [Упрощение условных
выражений](refactoring/techniques/simplifying-conditional-expressions.html){.type}
:::

# Введение проверки утверждения {#введение-проверки-утверждения .title}

::: aka
Также известен как: [Introduce Assertion]{style="display:inline-block"}
:::

> Здесь и далее под проверками подразумеваются вызовы функции `assert`.

::::: before-after-container
::: before
### Проблема

Корректная работа участка кода предполагает наличие каких-то
определённых условий или значений.
:::

::: after
### Решение

Замените эти предположения конкретными проверками.
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
После
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
После
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
После
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
После
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
После
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

### Причины рефакторинга

Участок кода создает предположения о чем-то (например, о текущем
состоянии объекта, значении параметра или локальной переменной). Обычно
это предположение никогда не будет нарушено разве что при наличии
какой-то ошибки.

Сделайте эти предположения явными, добавив соответствующие проверки. Как
и подсказки типов в параметрах методов, эти проверки могут играть роль
живой документации кода.

Хорошим признаком того, что нужна проверка каких-то условий, являются
комментарии в коде. Если вы видите комментарии, описывающие условия, при
которых метод корректно работает, значит здесь неплохо было бы вставить
проверку-утверждение этих условий.

### Достоинства

-   Если какое-то предположение неверно и в результате код производит
    также неверный результат, лучше сразу остановить выполнение, пока
    это не привело к фатальным последствиям и порче данных. Кроме того,
    это означает, что вы упустили написание какого-то теста в наборе
    тестов вашей программы.

### Недостатки

-   Иногда вместо простой проверки лучше вставить исключение. При этом
    вы можете выбрать необходимый класс исключения и дать остальному
    коду возможность корректно его обработать.

-   В каких случаях исключение лучше простой проверки? Если к нему может
    привести деятельность пользователя или системы, и вы можете
    обработать это исключение. С другой стороны, обычные безымянные
    необрабатываемые исключения равносильны простым проверкам --- вы их
    не обрабатываете, они могут быть вызваны только как результат бага в
    программе, который никогда не должен был возникнуть.

### Порядок рефакторинга

Когда вы видите, что предполагается какое-то условие, добавьте проверку
этого условия, чтобы проверить его наверняка.

Добавление проверки не должно изменять поведение программы.

Не перебарщивайте с использованием проверок для **всего**, что только
можно, на выбранном участке кода. Нужно проверять лишь те условия, без
которых код перестанет корректно работать. Если ваш код нормально
работает, в том числе, в случае, когда проверка не проходит успешно,
такую проверку можно убрать.

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

:::::: row
::::: {.col-xs-12 .col-sm-6 .col-lg-12}
### Борется с запахом

:::: dl
::: dt
[Комментарии](smells/comments.html)
:::
::::
:::::
::::::

::: next
#### Читаем дальше

[Упрощение вызовов методов []{.fa
.fa-arrow-right}](refactoring/techniques/simplifying-method-calls.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Введение
Null-объекта](introduce-null-object.html){.btn .btn-default rel="prev"}
:::
:::::::::::::::::::::::::::::::::::::::::::

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
::::::::::::::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::::::::::::::
