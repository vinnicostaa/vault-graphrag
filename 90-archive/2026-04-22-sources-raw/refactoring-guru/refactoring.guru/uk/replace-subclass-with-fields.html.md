::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Рефакторинг](refactoring.html){.type} /
[Прийоми рефакторингу](refactoring/techniques.html){.type} /
[Організація даних](refactoring/techniques/organizing-data.html){.type}
:::

# Заміна підкласу полями {#заміна-підкласу-полями .title}

::: aka
Також відомий як: [Replace Subclass with
Fields]{style="display:inline-block"}
:::

::::: before-after-container
::: before
### Проблема

У вас є підкласи, які відрізняються тільки методами, що повертають
дані-константи.
:::

::: after
### Рішення

Замініть методи полями в батьківському класі і видаліть підкласи.
:::
:::::

:::::::::: examples
::::::::: {.before-after .before-after-container}
::::: before
::: ba
До
:::

::: ba-container
![Replace Subclass with Fields -
Before](../images/refactoring/diagrams/Replace%20Subclass%20with%20Fields%20-%20Beforec050.png?id=ea6525cc6b55e1a03fdb35def943c675){width="415"}
:::
:::::

::::: after
::: ba
Після
:::

::: ba-container
![Replace Subclass with Fields -
After](../images/refactoring/diagrams/Replace%20Subclass%20with%20Fields%20-%20After22d1.png?id=bd1b29687aa333b3adbeb2bfb3e78614){width="152"}
:::
:::::
:::::::::
::::::::::

### Причини рефакторингу

Буває так, що вам треба розгорнути дію рефакторингу позбавлення від
кодування типу.

В одному з подібних випадків ієрархія підкласів може відрізнятися тільки
значеннями, які повертають певні методи. Причому ці значення не є
результатом обчислення, а жорстко прописані або в самих методах, або в
полях, які повертають методи. Щоб спростити архітектуру класів, така
ієрархія може бути згорнута в один клас, що містить одне або декілька
полів з потрібними значеннями залежно від ситуації.

Потреба в цих змінах могла виникнути після переміщення великої кількості
функціональності з ієрархії класів кудись в інше місце. Після цього
поточна ієрархія втратила свою цінність, і підкласи стали створювати
тільки надмірну складність.

### Переваги

-   Спрощує архітектуру системи. Створення підкласів --- зайве рішення,
    якщо все, що треба зробити, це повертати різні значення в декількох
    методах.

### Порядок рефакторингу

1.  Застосуйте до підкласів [заміну конструктора фабричним
    методом](replace-constructor-with-factory-method.html).

2.  Якщо якийсь код посилається на підкласи, замініть його використанням
    суперкласу.

3.  Оголосіть в суперкласі поля для зберігання значень кожного з методів
    підкласів, що повертають константні значення.

4.  Створіть захищений конструктор суперкласу для ініціалізації нових
    полів.

5.  Створіть або модифікуйте наявні конструктори підкласів, щоб вони
    викликали новий конструктор батьківського класу і передавали в нього
    відповідні значення.

6.  Реалізуйте кожен константний метод у батьківському класі так, щоб
    він повертав значення відповідного поля, а потім видаліть метод з
    підкласу.

7.  Якщо конструктор підкласу має якусь додаткову функціональність,
    застосуйте [вбудовування методу](inline-method.html) для
    вбудовування конструктора у фабричний метод суперкласу.

8.  Видаліть підклас.

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
### Анти-рефакторинг

:::: dl
::: dt
[Заміна кодування типу
підкласами](replace-type-code-with-subclasses.html)
:::
::::
:::::
::::::

::: next
#### Читаємо далі

[Спрощення умовних виразів []{.fa
.fa-arrow-right}](refactoring/techniques/simplifying-conditional-expressions.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Повернутись назад

[[]{.fa .fa-arrow-left} Заміна кодування типу
станом/стратегією](replace-type-code-with-state-strategy.html){.btn
.btn-default rel="prev"}
:::
:::::::::::::::::::::::::

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
::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::
