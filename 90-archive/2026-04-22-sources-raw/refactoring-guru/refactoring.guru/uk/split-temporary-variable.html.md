::::::::::::::::::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Рефакторинг](refactoring.html){.type} /
[Прийоми рефакторингу](refactoring/techniques.html){.type} / [Складання
методів](refactoring/techniques/composing-methods.html){.type}
:::

# Розщеплювання змінної {#розщеплювання-змінної .title}

::: aka
Також відомий як: [Split Temporary
Variable]{style="display:inline-block"}
:::

::::: before-after-container
::: before
### Проблема

У вас є локальна змінна, яка використовується для зберігання
різноманітних значень всередині методу (не рахуючи змінних циклів).
:::

::: after
### Рішення

Використайте різні змінні для різних значень. Кожна змінна повинна
відповідати тільки за одну певну річ.
:::
:::::

:::::::::::::::::::::::::::: examples
::::::: {.before-after .before-after-container .java data-lang="java"}
:::: before
::: ba
До
:::

``` {.code lang="java"}
double temp = 2 * (height + width);
System.out.println(temp);
temp = height * width;
System.out.println(temp);
```
::::

:::: after
::: ba
Після
:::

``` {.code lang="java"}
final double perimeter = 2 * (height + width);
System.out.println(perimeter);
final double area = height * width;
System.out.println(area);
```
::::
:::::::

::::::: {.before-after .before-after-container .csharp data-lang="csharp"}
:::: before
::: ba
До
:::

``` {.code lang="csharp"}
double temp = 2 * (height + width);
Console.WriteLine(temp);
temp = height * width;
Console.WriteLine(temp);
```
::::

:::: after
::: ba
Після
:::

``` {.code lang="csharp"}
readonly double perimeter = 2 * (height + width);
Console.WriteLine(perimeter);
readonly double area = height * width;
Console.WriteLine(area);
```
::::
:::::::

::::::: {.before-after .before-after-container .php data-lang="php"}
:::: before
::: ba
До
:::

``` {.code lang="php"}
$temp = 2 * ($this->height + $this->width);
echo $temp;
$temp = $this->height * $this->width;
echo $temp;
```
::::

:::: after
::: ba
Після
:::

``` {.code lang="php"}
$perimeter = 2 * ($this->height + $this->width);
echo $perimeter;
$area = $this->height * $this->width;
echo $area;
```
::::
:::::::

::::::: {.before-after .before-after-container .python data-lang="python"}
:::: before
::: ba
До
:::

``` {.code lang="python"}
temp = 2 * (height + width)
print(temp)
temp = height * width
print(temp)
```
::::

:::: after
::: ba
Після
:::

``` {.code lang="python"}
perimeter = 2 * (height + width)
print(perimeter)
area = height * width
print(area)
```
::::
:::::::

::::::: {.before-after .before-after-container .typescript data-lang="typescript"}
:::: before
::: ba
До
:::

``` {.code lang="typescript"}
let temp = 2 * (height + width);
console.log(temp);
temp = height * width;
console.log(temp);
```
::::

:::: after
::: ba
Після
:::

``` {.code lang="typescript"}
const perimeter = 2 * (height + width);
console.log(perimeter);
const area = height * width;
console.log(area);
```
::::
:::::::
::::::::::::::::::::::::::::

### Причини рефакторингу

Якщо ви «економите» змінні всередині функції і повторно використовуєте
їх для різноманітних непов'язаних між собою цілей, у вас обов'язково
почнуться проблеми в той момент, коли потрібно буде внести якісь
оновлення в код, що містить ці змінні. Вам доведеться перевірити
декілька раз усі випадки використання змінної, щоб впевнитись у
відсутності помилки в коді.

### Переваги

-   Кожен елемент програми повинен відповідати тільки за одну річ. Це
    значно спрощує підтримку коду в майбутньому, оскільки ви можете
    спокійно змінити цей елемент, не побоюючись побічних ефектів.

-   Покращується читабельність коду. Якщо змінна створювалася дуже
    давно, та ще і в поспіху, вона могла дістати елементарну назву, яка
    не пояснює суті значення, що зберігається, наприклад, `k`, `a2`,
    `value` і так далі. У вас є шанс виправити ситуацію, призначивши
    новим змінним хороші назви, що відбивають суть значень, що
    зберігаються. Наприклад, `customerTaxValue`, `cityUnemploymentRate`,
    `clientSalutationString` і так далі

-   Цей рефакторинг допомагає надалі [виділити ділянки коду, що
    повторюються, в окремі методи](extract-method.html).

### Порядок рефакторингу

1.  Знайдіть місце в коді, де змінна вперше заповнюється якимось
    значенням. У цьому місці перейменуйте цю змінну, причому нова назва
    повинна бути зрозумілою і відповідати її значенню.

2.  Підставте її нову назву замість старого в місцях, де
    використовувалося це значення змінної.

3.  Повторіть операцію для інших випадків, де змінній присвоюється нове
    значення.

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

:::::::::::::: row
::::: {.col-xs-12 .col-sm-6 .col-lg-4}
### Анти-рефакторинг

:::: dl
::: dt
[Вбудовування змінної](inline-temp.html)
:::
::::
:::::

::::::: {.col-xs-12 .col-sm-6 .col-lg-4}
### Схожі рефакторинги

:::: dl
::: dt
[Відокремлення змінної](extract-variable.html)
:::
::::

:::: dl
::: dt
[Видалення присвоювань
параметрам](remove-assignments-to-parameters.html)
:::
::::
:::::::

::::: {.col-xs-12 .col-sm-6 .col-lg-4}
### Домогає іншим рефакторингам

:::: dl
::: dt
[Відокремлення методу](extract-method.html)
:::
::::
:::::
::::::::::::::

::: next
#### Читаємо далі

[Видалення присвоювань параметрам []{.fa
.fa-arrow-right}](remove-assignments-to-parameters.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Повернутись назад

[[]{.fa .fa-arrow-left} Заміна змінної викликом
методу](replace-temp-with-query.html){.btn .btn-default rel="prev"}
:::
:::::::::::::::::::::::::::::::::::::::::::::::::::

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
::::::::::::::::::::::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::::::::::::::::::::::
