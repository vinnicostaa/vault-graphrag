::::::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Рефакторинг](refactoring.html){.type} /
[Прийоми рефакторингу](refactoring/techniques.html){.type} /
[Організація даних](refactoring/techniques/organizing-data.html){.type}
:::

# Заміна магічного числа символьною константою {#заміна-магічного-числа-символьною-константою .title}

::: aka
Також відомий як: [Replace Magic Number with Symbolic
Constant]{style="display:inline-block"}
:::

::::: before-after-container
::: before
### Проблема

В коді використовується число, яке несе якийсь певний сенс.
:::

::: after
### Рішення

Замініть це число константою з такою назвою, що пояснює сенс цього
числа.
:::
:::::

:::::::::::::::::::::::::::: examples
::::::: {.before-after .before-after-container .java data-lang="java"}
:::: before
::: ba
До
:::

``` {.code lang="java"}
double potentialEnergy(double mass, double height) {
  return mass * height * 9.81;
}
```
::::

:::: after
::: ba
Після
:::

``` {.code lang="java"}
static final double GRAVITATIONAL_CONSTANT = 9.81;

double potentialEnergy(double mass, double height) {
  return mass * height * GRAVITATIONAL_CONSTANT;
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
double PotentialEnergy(double mass, double height) 
{
  return mass * height * 9.81;
}
```
::::

:::: after
::: ba
Після
:::

``` {.code lang="csharp"}
const double GRAVITATIONAL_CONSTANT = 9.81;

double PotentialEnergy(double mass, double height) 
{
  return mass * height * GRAVITATIONAL_CONSTANT;
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
function potentialEnergy($mass, $height) {
  return $mass * $height * 9.81;
}
```
::::

:::: after
::: ba
Після
:::

``` {.code lang="php"}
define("GRAVITATIONAL_CONSTANT", 9.81);

function potentialEnergy($mass, $height) {
  return $mass * $height * GRAVITATIONAL_CONSTANT;
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
def potentialEnergy(mass, height):
    return mass * height * 9.81
```
::::

:::: after
::: ba
Після
:::

``` {.code lang="python"}
GRAVITATIONAL_CONSTANT = 9.81

def potentialEnergy(mass, height):
    return mass * height * GRAVITATIONAL_CONSTANT
```
::::
:::::::

::::::: {.before-after .before-after-container .typescript data-lang="typescript"}
:::: before
::: ba
До
:::

``` {.code lang="typescript"}
potentialEnergy(mass: number, height: number): number {
  return mass * height * 9.81;
}
```
::::

:::: after
::: ba
Після
:::

``` {.code lang="typescript"}
static const GRAVITATIONAL_CONSTANT = 9.81;

potentialEnergy(mass: number, height: number): number {
  return mass * height * GRAVITATIONAL_CONSTANT;
}
```
::::
:::::::
::::::::::::::::::::::::::::

### Причини рефакторингу

Магічні числа --- це числові значення, що зустрічаються в коді, але при
цьому не очевидно, що вони означають. Цей антипатерн ускладнює розуміння
програми, а також її рефакторинг.

Додаткові складнощі виникають, коли треба поміняти певне магічне число.
Це не можна зробити автозаміною, оскільки одне і те ж число може
використовуватися для різних цілей, тобто вам треба буде перевіряти
кожну ділянку коду, де використовується це число.

### Переваги

-   Символьна константа може служити живою документацією, пояснюючи сенс
    значення, яке в ній зберігається.

-   Значення константи набагато простіше замінити, ніж шукати потрібне
    число по всьому коду, ризикуючи при цьому замінити таке ж число, яке
    використовувалося для інших цілей.

-   Прибирає дублювання використання числа або рядка по всьому коду. Це
    особливо актуально, якщо значення є складним і довгим (наприклад,
    `-14159`, `0xcafebabe`).

### Корисні факти

#### Не всі числа є магічними

Якщо призначення чисел очевидні, їх не потрібно замінювати константами,
класичний приклад:

<figure class="code">
<pre class="code"><code>for (i = 0; i &lt; сount; i++) { ... }</code></pre>
</figure>

#### Альтернативи

1.  Іноді магічне число можна замінити викликом методу. Наприклад, якщо
    у вас є магічне число, що означає кількість елементів колекції, вам
    не обов'язково використовувати його для перевірок останнього
    елементу колекції. Замість цього можна використати вбудований метод
    отримання довжини колекції.

2.  Магічні числа можуть бути використані для реалізації кодування типу.
    Наприклад, у вас є два типи користувачів, і щоби позначити їх, у вас
    є числове поле в класі, в якому для адміністраторів зберігається
    число `1`, а для простих користувачів --- число `2`.

    В цьому випадку має сенс використати один з рефакторингів
    позбавлення від кодування типу:

    -   [заміна кодування типу
        класом](replace-type-code-with-class.html)

    -   [заміна кодування типу
        підкласами](replace-type-code-with-subclasses.html)

    -   [заміна кодування типу
        станом/стратегією](replace-type-code-with-state-strategy.html)

### Порядок рефакторингу

1.  Оголосіть константу і присвойте їй значення магічного числа.

2.  Знайдіть усі згадки магічного числа.

3.  Для всіх знайдених чисел перевірте, чи узгоджується це магічне число
    з призначенням константи. Якщо так, замініть його вашою константою.
    Ця перевірка важлива, оскільки одне і теж число може означати
    абсолютно різні речі (в цьому випадку, вони мають бути замінені
    різними константами).

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

[Інкапсуляція поля []{.fa .fa-arrow-right}](encapsulate-field.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Повернутись назад

[[]{.fa .fa-arrow-left} Заміна двонаправленого зв\'язку
одностороннім](change-bidirectional-association-to-unidirectional.html){.btn
.btn-default rel="prev"}
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
