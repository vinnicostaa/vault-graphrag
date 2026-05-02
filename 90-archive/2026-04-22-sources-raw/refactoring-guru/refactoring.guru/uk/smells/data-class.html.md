:::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::: {.main-content-container .center-content-container}
:::::::::: {.page .text}
::: breadcrumb
[](../../index.html){.home} / [Рефакторинг](../refactoring.html){.type}
/ [Запахи коду](../refactoring/smells.html){.type} / [Забрюднювачі
коду](../refactoring/smells/dispensables.html){.type}
:::

# Клас даних {#клас-даних .title}

::: aka
Також відомий як: [Data Class]{style="display:inline-block"}
:::

### Симптоми і ознаки

Класи даних --- це класи, які містять тільки поля і прості методи для
доступу до них (геттери і сеттери). Це просто контейнери для даних, які
використовуються іншими класами. Ці класи не містять ніякої додаткової
функціональності і не можуть самостійно працювати з даними, якими
володіють.

<figure class="image">
<img
src="../../images/refactoring/content/smells/data-class-01949b.png?id=2ea1583b05a194a056d27ac559545318"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/data-class-01.png?id=2ea1583b05a194a056d27ac559545318 500w,/images/refactoring/content/smells/data-class-01-1.5x.png?id=e816a7b99f2cbef399ea43b1af66125b 750w,/images/refactoring/content/smells/data-class-01-2x.png?id=2beb8150d4ba31ca37d6515495ceff2d 1000w"
sizes="(max-width: 720px) 100vw, 500px" data-fetchpriority="high"
width="500" height="300" />
</figure>

### Причини появи

Це нормально, коли клас на початку свого життя містить усього лише
декілька публічних полів (а може навіть і парочку геттерів/сеттерів).
Проте, справжня сила об'єктів полягає в тому, що вони можуть зберігати
типи поведінки або операції над власними даними.

### Лікування

-   Якщо клас містить публічні поля, застосуйте [інкапсуляцію
    поля](../encapsulate-field.html) щоби приховати їх з прямого
    доступу, дозволивши доступ тільки через геттери і сеттери.

-   Застосуйте [інкапсуляцію колекції](../encapsulate-collection.html)
    для даних, які зберігаються в колекціях (на зразок масивів).

-   Огляньте клієнтський код, який використовує цей клас. Можливо, там
    ви знайдете функціональність, яка виглядала б доречніше в самому
    класі даних. В цьому випадку використайте [переміщення
    методу](../move-method.html) і [відокремлення
    методу](../extract-method.html) для перенесення функціональності в
    клас даних.

-   Після того, як клас наповнився осмисленими методами, можливо, варто
    подумати про знищення старих методів доступу до даних, які дають
    занадто відкритий доступ до даних класу. В цьому вам допоможе
    [видалення сеттера](../remove-setting-method.html) і [приховання
    методу](../hide-method.html).

<figure class="image">
<img
src="../../images/refactoring/content/smells/data-class-024cf2.png?id=db0eb15f9f229bafd8423b2cfd09f910"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/data-class-02.png?id=db0eb15f9f229bafd8423b2cfd09f910 500w,/images/refactoring/content/smells/data-class-02-1.5x.png?id=f2f0339a74ba638bfbd8d57317396f43 750w,/images/refactoring/content/smells/data-class-02-2x.png?id=fb9b6d670232d6effe790980e6b388ec 1000w"
sizes="(max-width: 720px) 100vw, 500px" loading="lazy" width="500"
height="300" />
</figure>

### Виграш

-   Покращує розуміння і організацію коду. Операції над певними даними
    тепер зібрані в одному місці, їх не потрібно шукати по всьому коду.

-   Може розкрити факти дублювання клієнтського коду.

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

[Мертвий код []{.fa .fa-arrow-right}](dead-code.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Повернутись назад

[[]{.fa .fa-arrow-left} Ледачий клас](lazy-class.html){.btn .btn-default
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
