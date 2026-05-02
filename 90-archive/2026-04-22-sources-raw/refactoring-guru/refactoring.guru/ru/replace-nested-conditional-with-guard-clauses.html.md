::::::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Рефакторинг](refactoring.html){.type} /
[Приёмы](refactoring/techniques.html){.type} / [Упрощение условных
выражений](refactoring/techniques/simplifying-conditional-expressions.html){.type}
:::

# Замена вложенных условных операторов граничным оператором {#замена-вложенных-условных-операторов-граничным-оператором .title}

::: aka
Также известен как: [Replace Nested Conditional with Guard
Clauses]{style="display:inline-block"}
:::

::::: before-after-container
::: before
### Проблема

У вас есть группа вложенных условных операторов, среди которых сложно
выделить нормальный ход выполнения кода.
:::

::: after
### Решение

Выделите все проверки специальных или граничных случаев выполнения в
отдельные условия и поместите их перед основными проверками. В идеале,
вы должны получить «плоский» список условных операторов, идущих один за
другим.
:::
:::::

:::::::::::::::::::::::::::: examples
::::::: {.before-after .before-after-container .java data-lang="java"}
:::: before
::: ba
До
:::

``` {.code lang="java"}
public double getPayAmount() {
  double result;
  if (isDead){
    result = deadAmount();
  }
  else {
    if (isSeparated){
      result = separatedAmount();
    }
    else {
      if (isRetired){
        result = retiredAmount();
      }
      else{
        result = normalPayAmount();
      }
    }
  }
  return result;
}
```
::::

:::: after
::: ba
После
:::

``` {.code lang="java"}
public double getPayAmount() {
  if (isDead){
    return deadAmount();
  }
  if (isSeparated){
    return separatedAmount();
  }
  if (isRetired){
    return retiredAmount();
  }
  return normalPayAmount();
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
public double GetPayAmount()
{
  double result;
  
  if (isDead)
  {
    result = DeadAmount();
  }
  else 
  {
    if (isSeparated)
    {
      result = SeparatedAmount();
    }
    else 
    {
      if (isRetired)
      {
        result = RetiredAmount();
      }
      else
      {
        result = NormalPayAmount();
      }
    }
  }
  
  return result;
}
```
::::

:::: after
::: ba
После
:::

``` {.code lang="csharp"}
public double GetPayAmount() 
{
  if (isDead)
  {
    return DeadAmount();
  }
  if (isSeparated)
  {
    return SeparatedAmount();
  }
  if (isRetired)
  {
    return RetiredAmount();
  }
  return NormalPayAmount();
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
function getPayAmount() {
  if ($this->isDead) {
    $result = $this->deadAmount();
  } else {
    if ($this->isSeparated) {
      $result = $this->separatedAmount();
    } else {
      if ($this->isRetired) {
        $result = $this->retiredAmount();
      } else {
        $result = $this->normalPayAmount();
      }
    }
  }
  return $result;
}
```
::::

:::: after
::: ba
После
:::

``` {.code lang="php"}
function getPayAmount() {
  if ($this->isDead) {
    return $this->deadAmount();
  }
  if ($this->isSeparated) {
    return $this->separatedAmount();
  }
  if ($this->isRetired) {
    return $this->retiredAmount();
  }
  return $this->normalPayAmount();
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
def getPayAmount(self):
    if self.isDead:
        result = deadAmount()
    else:
        if self.isSeparated:
            result = separatedAmount()
        else:
            if self.isRetired:
                result = retiredAmount()
            else:
                result = normalPayAmount()
    return result
```
::::

:::: after
::: ba
После
:::

``` {.code lang="python"}
def getPayAmount(self):
    if self.isDead:
        return deadAmount()
    if self.isSeparated:
        return separatedAmount()
    if self.isRetired:
        return retiredAmount()
    return normalPayAmount()
```
::::
:::::::

::::::: {.before-after .before-after-container .typescript data-lang="typescript"}
:::: before
::: ba
До
:::

``` {.code lang="typescript"}
getPayAmount(): number {
  let result: number;
  if (isDead){
    result = deadAmount();
  }
  else {
    if (isSeparated){
      result = separatedAmount();
    }
    else {
      if (isRetired){
        result = retiredAmount();
      }
      else{
        result = normalPayAmount();
      }
    }
  }
  return result;
}
```
::::

:::: after
::: ba
После
:::

``` {.code lang="typescript"}
getPayAmount(): number {
  if (isDead){
    return deadAmount();
  }
  if (isSeparated){
    return separatedAmount();
  }
  if (isRetired){
    return retiredAmount();
  }
  return normalPayAmount();
}
```
::::
:::::::
::::::::::::::::::::::::::::

### Причины рефакторинга

«Условный оператор из ада» довольно просто отличить. Отступы каждого из
уровней вложенности формируют в нем отчётливую стрелку, указывающую
вправо:

``` code
if () {
    if () {
        do {
            if () {
                if () {
                    if () {
                        ...
                    }
                }
                ...
            }
            ...
        }
        while ();
        ...
    }
    else {
        ...
    }
}
```

Разобраться в том, что и как делает такой оператор довольно сложно, так
как «нормальный» ход выполнения в нем не очевиден. Такие операторы
появляются эволюционным путём, когда каждое из условий добавляется в
разные промежутки времени без мыслей об оптимизации остальных условий.

Чтобы упростить такой оператор, нужно выделить все особые случаи в
отдельные условные операторы, которые бы при наступлении граничных
условий, сразу заканчивали выполнение и возвращали нужное значение. По
сути, ваша цель --- сделать такой оператор плоским.

### Порядок рефакторинга

Постарайтесь избавиться от «побочных эффектов» в условиях операторов.
[Разделение запроса и модификатора](separate-query-from-modifier.html)
может в этом помочь. Такое решение понадобится для дальнейших
перестановок условий.

1.  Выделите граничные условия, которые приводят к вызову исключения или
    немедленному возвращению значения из метода. Переместите эти условия
    в начало метода.

2.  После того как с переносами покончено, и все тесты стали проходить,
    проверьте, можно ли использовать [объединение условных
    операторов](consolidate-conditional-expression.html) для граничных
    условных операторов, ведущих к одинаковым исключениям или
    возвращаемым значениям.

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

[Замена условного оператора полиморфизмом []{.fa
.fa-arrow-right}](replace-conditional-with-polymorphism.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Удаление управляющего
флага](remove-control-flag.html){.btn .btn-default rel="prev"}
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
