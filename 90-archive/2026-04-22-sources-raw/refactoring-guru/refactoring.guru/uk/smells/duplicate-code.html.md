:::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::: {.main-content-container .center-content-container}
:::::::::: {.page .text}
::: breadcrumb
[](../../index.html){.home} / [Рефакторинг](../refactoring.html){.type}
/ [Запахи коду](../refactoring/smells.html){.type} / [Забрюднювачі
коду](../refactoring/smells/dispensables.html){.type}
:::

# Дублювання коду {#дублювання-коду .title}

::: aka
Також відомий як: [Duplicate Code]{style="display:inline-block"}
:::

### Симптоми і ознаки

Два фрагменти коду виглядають майже однаковими.

<figure class="image">
<img
src="../../images/refactoring/content/smells/duplicate-code-013568.png?id=16fe591195fa50073551852b3d44844e"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/duplicate-code-01.png?id=16fe591195fa50073551852b3d44844e 500w,/images/refactoring/content/smells/duplicate-code-01-1.5x.png?id=fc62042bc009d4890e7b1d37fe1d5817 750w,/images/refactoring/content/smells/duplicate-code-01-2x.png?id=9e462bc4bd52927cf45cfc7dbc5907af 1000w"
sizes="(max-width: 720px) 100vw, 500px" data-fetchpriority="high"
width="500" height="300" />
</figure>

### Причини появи

У більшості випадків дублювання виникає тоді, коли в проекті працюють
декілька авторів, причому над різними його частинами. Вони працюють над
схожими задачами, але не знають, що колега вже написав схожий код, який
можна використати замість написання свого.

Зустрічається і непряме дублювання, коли конкретні ділянки коду
відрізняються зовні, хоча і виконують одну і ту ж задачу. Таке
дублювання буває досить складно виявити і виправити.

В окремих випадках дублювання створюється навмисно. Найчастіше в
поспіху, коли строки здачі проекту горять. Програміст-початківець бачить
у вже написаному коді «майже такий, як треба» фрагмент, і не може
встояти від спокуси просто скопіювати його як є і вставити десь в іншому
місці (і так десяток разів).

В інших випадках програміст просто занадто ледачий, щоби позбавити код
від дублювання.

### Лікування

-   Одна і та ж ділянка коду присутня в двох методах одного і того ж
    класу: необхідно застосувати [відокремлення
    методу](../extract-method.html), після чого викликати код створеного
    методу з обох ділянок.

    <figure class="image">
    <img
    src="../../images/refactoring/content/smells/duplicate-code-020123.png?id=50d92af3defe2c2688f66cde102c9c09"
    style="aspect-ratio: 1.67;"
    srcset="/images/refactoring/content/smells/duplicate-code-02.png?id=50d92af3defe2c2688f66cde102c9c09 500w,/images/refactoring/content/smells/duplicate-code-02-1.5x.png?id=224c1f9b44e12cfc3ef7b39e6230d53c 750w,/images/refactoring/content/smells/duplicate-code-02-2x.png?id=5b9325ca1b0369ec3423808380fa9022 1000w"
    sizes="(max-width: 720px) 100vw, 500px" loading="lazy" width="500"
    height="300" />
    </figure>

-   Одна і та ж ділянка коду присутня в двох підкласах, що знаходяться
    на одному рівні:

    -   Необхідно застосувати [відокремлення
        методу](../extract-method.html) для обох класів з подальшим
        [підйомом поля](../pull-up-field.html) для полів, які
        використовуються в піднятому методі.

    -   Якщо спільний код знаходиться в конструкторі, слід використати
        [підйом тіла конструктора](../pull-up-constructor-body.html).

    -   Якщо ділянки коду схожі, але не співпадають повністю, треба
        користуватися [створення шаблонного
        методу](../form-template-method.html).

    -   Якщо обидва методи роблять одне і те ж, але за допомогою різних
        алгоритмів, можна вибрати чіткіший з цих алгоритмів і
        застосувати [заміщення алгоритму](../substitute-algorithm.html).

-   Код, що дублюється, знаходиться в двох різних класах:

    -   Якщо ці класи не є частиною якоїсь ієрархії, слід використати
        [відокремлення суперкласу](../extract-superclass.html), щоб
        створити для цих класів один суперклас, що містить усю спільну
        функціональність.

    -   Якщо створення суперкласу небажане або неможливе, слід
        застосувати [відокремлення класу](../extract-class.html) в
        одному класі, а потім використати новий компонент в іншому.

-   Якщо присутня низка умовних операторів, які виконують один і той же
    код і відрізняються тільки умовами, слід об'єднати ці оператори в
    один із загальною умовою за допомогою [об'єднання умовних
    операторів](../consolidate-conditional-expression.html), а також
    застосувати [відокремлення методу](../extract-method.html), щоб
    винести цю умову в окремий метод із зрозумілою назвою.

-   Однаковий код виконується в усіх гілках умовного оператора:
    необхідно винести однаковий код за межі умовного оператора за
    допомогою [об'єднання фрагментів з дублюванням коду в умовних
    операторах](../consolidate-duplicate-conditional-fragments.html).

### Виграш

-   Об'єднання коду, що дублюєтся, дозволяє поліпшити структуру коду і
    зменшити його об'єм.

-   Це, у свою чергу, веде до спрощення і здешевлення підтримки коду в
    майбутньому.

<figure class="image">
<img
src="../../images/refactoring/content/smells/duplicate-code-038497.png?id=bd88b98ff5e5e1b5a4019cb0a50df9f5"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/duplicate-code-03.png?id=bd88b98ff5e5e1b5a4019cb0a50df9f5 500w,/images/refactoring/content/smells/duplicate-code-03-1.5x.png?id=6c3566b366c660b727083df7f491a38d 750w,/images/refactoring/content/smells/duplicate-code-03-2x.png?id=33df6a84eddb7c888f6757d4d80d5e20 1000w"
sizes="(max-width: 720px) 100vw, 500px" loading="lazy" width="500"
height="300" />
</figure>

### Не варто чіпати, якщо\...

-   В окремих випадках об'єднання двох однакових ділянок коду може
    зробити код менш очевидним і зрозумілим.

::::: {.prom .banner-content .banner .banner-striped .banner-with-image data-id="Ref: Part of the ebook" creative-id="animated-uk" position="content_bottom"}
::: banner-image
Your browser does not support HTML video.
:::

::: banner-text
### Замучились читати? {#замучились-читати .title}

Збігайте за подушкою, в нас тут контенту на [7 годин]{.blue} читання.

Або спробуйте наш новий інтерактивний курс. Він набагато цікавіший за
банальний тест.

[ Дізнатися більше...](../refactoring/course.html){.btn .btn-secondary}
:::
:::::

::: next
#### Читаємо далі

[Ледачий клас []{.fa .fa-arrow-right}](lazy-class.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Повернутись назад

[[]{.fa .fa-arrow-left} Коментарі](comments.html){.btn .btn-default
rel="prev"}
:::
::::::::::

:::::: {.prom .banner-sidebar .banner-removable .banner-removable-refactoring data-id="Ref: Sidebar" creative-id="standard-sidebar-uk" position="sidebar"}
::::: ban
::: {.image .product-image}
[Знижки!]{.banner-discount}
[![](../../images/refactoring/course/course-cover-uk4341.jpg?id=e53397ef67078e742fb132da37ca7cc8){width="300"
height="300" loading="lazy"}](../refactoring/course.html)
:::

::: {.banner-text .banner-text-uk}
Цей запах коду є частиною нашого інтерактивного **онлайн курсу з
рефакторингу**.

[ Подробиці...](../refactoring/course.html){.btn .btn-secondary
.btn-block}
:::
:::::
::::::
:::::::::::::::
::::::::::::::::
