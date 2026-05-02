:::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::: {.main-content-container .center-content-container}
:::::::::: {.page .text}
::: breadcrumb
[](../../index.html){.home} / [Рефакторинг](../refactoring.html){.type}
/ [Запахи коду](../refactoring/smells.html){.type} / [Заплутувальники
зв\'язками](../refactoring/smells/couplers.html){.type}
:::

# Заздрісні функції {#заздрісні-функції .title}

::: aka
Також відомий як: [Feature Envy]{style="display:inline-block"}
:::

### Симптоми і ознаки

Метод звертається до даних іншого об'єкта частіше, ніж до власних даних.

<figure class="image">
<img
src="../../images/refactoring/content/smells/feature-envy-010982.png?id=f520a24562e3f4b7848eca94792c329f"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/feature-envy-01.png?id=f520a24562e3f4b7848eca94792c329f 500w,/images/refactoring/content/smells/feature-envy-01-1.5x.png?id=8f02ea7aa51cd62bef6763bcecabec9b 750w,/images/refactoring/content/smells/feature-envy-01-2x.png?id=4329322703e5af5b3ef7faefd17c4750 1000w"
sizes="(max-width: 720px) 100vw, 500px" data-fetchpriority="high"
width="500" height="300" />
</figure>

### Причини появи

Цей запах може з'явитися після переміщення якихось полів в клас даних. В
цьому випадку операції з даними, можливо, також слід перемістити в цей
клас.

### Лікування

Слід дотримуватися такого правила: те, що змінюється разом, треба
зберігати в одному місці. Зазвичай дані і функції, які використовують ці
дані, також змінюються разом (хоча бувають виключення).

-   Якщо метод явно слід перенести в інше місце, застосуйте [переміщення
    методу](../move-method.html).

-   Якщо тільки частина методу звертається до даних іншого об'єкта,
    застосуйте [відокремлення методу](../extract-method.html) до цієї
    частини.

-   Якщо метод використовує функції декількох інших класів, треба
    спочатку визначити, в якому класі знаходиться найбільше даних, що
    використовуються. Потім слід перемістити метод в цей клас разом з
    іншими даними. Як альтернатива, за допомогою [відокремлення
    методу](../extract-method.html) метод розбивається на декілька
    частин, і вони розміщуються в різних частинах інших класів.

<figure class="image">
<img
src="../../images/refactoring/content/smells/feature-envy-025fc9.png?id=a90a3545498c7c22e605ceeb1f23d005"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/feature-envy-02.png?id=a90a3545498c7c22e605ceeb1f23d005 500w,/images/refactoring/content/smells/feature-envy-02-1.5x.png?id=56b1c3639436656391810a4122cc488c 750w,/images/refactoring/content/smells/feature-envy-02-2x.png?id=641faecaeb0d422232c0bcc6346352c5 1000w"
sizes="(max-width: 720px) 100vw, 500px" loading="lazy" width="500"
height="300" />
</figure>

### Виграш

-   Зменшення дублювання коду (якщо код обробки даних переїхав в одне
    загальне місце).

-   Поліпшення організації коду (оскільки методи роботи з даними
    знаходяться біля цих даних).

<figure class="image">
<img
src="../../images/refactoring/content/smells/feature-envy-03a90e.png?id=ea63eeab9eda1910348d0930c8592780"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/feature-envy-03.png?id=ea63eeab9eda1910348d0930c8592780 500w,/images/refactoring/content/smells/feature-envy-03-1.5x.png?id=9556986c5ae21dd7e2367d44d28e3258 750w,/images/refactoring/content/smells/feature-envy-03-2x.png?id=d8d24af45285db63e68560788e6240bc 1000w"
sizes="(max-width: 720px) 100vw, 500px" loading="lazy" width="500"
height="300" />
</figure>

### Не варто чіпати, якщо\...

-   Бувають випадки, коли поведінка навмисно відділяється від класу, що
    містить дані. Найчастіше це роблять для того, щоби мати можливість
    динамічно міняти цю поведінку (патерни
    [Стратегія](../design-patterns/strategy.html),
    [Відвідувач](../design-patterns/visitor.html) і так далі).

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

[Недоречна близькість []{.fa
.fa-arrow-right}](inappropriate-intimacy.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Повернутись назад

[[]{.fa .fa-arrow-left} Заплутувальники
зв\'язками](../refactoring/smells/couplers.html){.btn .btn-default
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
