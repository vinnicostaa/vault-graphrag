:::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::: {.main-content-container .center-content-container}
:::::::::: {.page .text}
::: breadcrumb
[](../../index.html){.home} / [Рефакторинг](../refactoring.html){.type}
/ [Запахи коду](../refactoring/smells.html){.type} / [Порушники
об\'єктно-орієнтованого
дизайну](../refactoring/smells/oo-abusers.html){.type}
:::

# Оператори switch {#оператори-switch .title}

::: aka
Також відомий як: [Switch Statements]{style="display:inline-block"}
:::

### Симптоми і ознаки

У вас є складний оператор `switch` або послідовність `if `-ів.

<figure class="image">
<img
src="../../images/refactoring/content/smells/switch-statements-013ae1.png?id=1c5a538d451c3d2b2e43d0d4f53b994b"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/switch-statements-01.png?id=1c5a538d451c3d2b2e43d0d4f53b994b 500w,/images/refactoring/content/smells/switch-statements-01-1.5x.png?id=77cd896e1d6bcf499db5eb5acdda01e4 750w,/images/refactoring/content/smells/switch-statements-01-2x.png?id=da7f9cad4c4bb8bf5d6910aba08680be 1000w"
sizes="(max-width: 720px) 100vw, 500px" data-fetchpriority="high"
width="500" height="300" />
</figure>

### Причини появи

Однією з очевидних ознак об'єктно-орієнтованого коду служить порівняно
рідкісне використання операторів типу `switch` або `case`. Часто один і
той же блок `switch` виявляється розкиданим по різних місцях програми.
При додаванні в нього нового варіанту доводиться шукати всі ці блоки
`switch` і модифікувати їх.

Як правило, помітивши блок `switch`, слід подумати про поліморфізм.

### Лікування

-   Щоб ізолювати `switch` і помістити його в потрібний клас може
    знадобитися [відокремлення методу](../extract-method.html) і
    [переміщення методу](../move-method.html).

-   Якщо `switch` перемикається за допомогою коду типу, наприклад,
    перемикається режим виконання програми, то слід використати [заміну
    кодування типу
    підкласами](../replace-type-code-with-subclasses.html) або [заміну
    кодування типу
    станом/стратегією](../replace-type-code-with-state-strategy.html).

-   Після налаштування структури наслідування слід використати [заміну
    умовного оператора
    поліморфізмом](../replace-conditional-with-polymorphism.html).

-   Якщо варіантів в операторі не дуже багато і всі вони призводять до
    виклику одного й того ж методу з різними параметрами, то введення
    поліморфізму буде надмірним. В цьому випадку варто замислитися про
    розбиття цього методу на декілька різних, які виконуватимуть кожен
    свої функції, для чого треба застосувати [заміну параметра набором
    спеціалізованих
    методів](../replace-parameter-with-explicit-methods.html).

-   Якщо одним з варіантів умовного оператора є `null`, використайте
    [введення Null-об'єкта](../introduce-null-object.html).

### Виграш

-   Покращує організацію коду.

<figure class="image">
<img
src="../../images/refactoring/content/smells/switch-statements-029112.png?id=29ec9ad9c6d760d2b940a68a1d23c4be"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/switch-statements-02.png?id=29ec9ad9c6d760d2b940a68a1d23c4be 500w,/images/refactoring/content/smells/switch-statements-02-1.5x.png?id=ba92702333e88eed74ee9866f9ac9d0b 750w,/images/refactoring/content/smells/switch-statements-02-2x.png?id=b7c9163ebe366c956f1aa64c4b3312e4 1000w"
sizes="(max-width: 720px) 100vw, 500px" loading="lazy" width="500"
height="300" />
</figure>

### Не варто чіпати, якщо\...

-   Коли оператор `switch` виконує прості дії, немає ніякого сенсу щось
    міняти в коді.

-   Часто буває, що оператор `switch` використовується у фабричних
    патернах проектування ([Фабричний
    метод](../design-patterns/factory-method.html), [Абстрактна
    фабрика](../design-patterns/abstract-factory.html)), для вибору
    створюваного класу.

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

[Тимчасове поле []{.fa .fa-arrow-right}](temporary-field.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Повернутись назад

[[]{.fa .fa-arrow-left} Порушники об\'єктно-орієнтованого
дизайну](../refactoring/smells/oo-abusers.html){.btn .btn-default
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
