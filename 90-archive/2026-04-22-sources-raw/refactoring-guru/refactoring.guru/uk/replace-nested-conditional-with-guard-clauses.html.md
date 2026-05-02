::::::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Рефакторинг](refactoring.html){.type} /
[Прийоми рефакторингу](refactoring/techniques.html){.type} / [Спрощення
умовних
виразів](refactoring/techniques/simplifying-conditional-expressions.html){.type}
:::

# Заміна вкладених умовних операторів граничним оператором {#заміна-вкладених-умовних-операторів-граничним-оператором .title}

::: aka
Також відомий як: [Replace Nested Conditional with Guard
Clauses]{style="display:inline-block"}
:::

::::: before-after-container
::: before
### Проблема

У вас є група вкладених умовних операторів, серед яких складно виділити
нормальний хід виконання коду.
:::

::: after
### Рішення

Виділіть усі перевірки спеціальних або граничних випадків виконання в
окремі умови і поставте їх перед основними перевірками. В ідеалі, ви
повинні отримати «плаский» список умовних операторів, що йдуть один за
іншим.
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
Після
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
Після
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
Після
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
Після
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
Після
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

### Причини рефакторингу

«Умовний оператор з пекла» досить просто відрізнити. Відступи кожного з
рівнів вкладеності формують в нім виразну стрілку, що вказує направо:

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

Розібратися в тому, що і як робить такий оператор досить складно,
оскільки «нормальний» хід виконання в ньому не очевидний. Такі оператори
з'являються еволюційним шляхом, коли кожна з умов додається в різні часи
без думки про необхідності оптимизаціі інших умов.

Щоб спростити такий оператор, треба виділити усі особливі випадки в
окремі умовні оператори, які у випадках настання граничних умов будуть
відразу закінчувати виконання і повертатимуть потрібне значення. По
суті, ваша мета --- зробити такий оператор пласким.

### Порядок рефакторингу

Намагайтеся позбутися від «побічних ефектів» в умовах операторів.
[Розділення запиту і модифікатора](separate-query-from-modifier.html)
може в цьому допомогти. Таке рішення знадобиться для подальших
перестановок умов.

1.  Виділіть граничні умови, які призводять до виклику виключення або
    негайного повернення значення з методу. Перемістіть ці умови в
    початок методу.

2.  Після того, як з перенесеннями покінчено, і усі тести стали
    проходити, перевірте, чи можна використати [ооб'єднання умовних
    операторів](consolidate-conditional-expression.html) для граничних
    умовних операторів, що ведуть до однакових виключень або значень,
    які повертаються.

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

[Заміна умовного оператора поліморфізмом []{.fa
.fa-arrow-right}](replace-conditional-with-polymorphism.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Повернутись назад

[[]{.fa .fa-arrow-left} Видалення керуючого
флагу](remove-control-flag.html){.btn .btn-default rel="prev"}
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
