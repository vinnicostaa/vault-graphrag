:::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::: {.main-content-container .center-content-container}
:::::::::: {.page .text}
::: breadcrumb
[](../../index.html){.home} / [Рефакторинг](../refactoring.html){.type}
/ [Запахи коду](../refactoring/smells.html){.type} /
[Роздувальщики](../refactoring/smells/bloaters.html){.type}
:::

# Одержимість елементарними типами {#одержимість-елементарними-типами .title}

::: aka
Також відомий як: [Primitive Obsession]{style="display:inline-block"}
:::

### Симптоми і ознаки

-   Використання елементарних типів замість маленьких об'єктів для
    невеликих завдань (наприклад, валюта, діапазони, спеціальні рядки
    для телефонних номерів і тому подібне).

-   Використання констант для кодування якоїсь інформації (наприклад,
    константа `USER_ADMIN_ROLE = 1` для позначення користувачів з роллю
    адміністратора).

-   Використання рядкових констант як назв полів в масивах.

<figure class="image">
<img
src="../../images/refactoring/content/smells/primitive-obsession-0177ec.png?id=73e1c5b08ea68a7ad7a66832644e6696"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/primitive-obsession-01.png?id=73e1c5b08ea68a7ad7a66832644e6696 500w,/images/refactoring/content/smells/primitive-obsession-01-1.5x.png?id=128163a863560b18aa19640c18468bfb 750w,/images/refactoring/content/smells/primitive-obsession-01-2x.png?id=e13be298e4b8d9d4a987972dfc777f4b 1000w"
sizes="(max-width: 720px) 100vw, 500px" data-fetchpriority="high"
width="500" height="300" />
</figure>

### Причини появи

Як і більшість інших запахів, цей розпочинається з маленької слабкості.
Програмістові знадобилося поле для зберігання якихось даних. Він
подумав, що створити поле елементарного типу куди простіше, ніж новий
клас. Це й було зроблено. Потім знадобилося інше поле, і воно також було
додане схожим чином. А потім не встигли ви озирнутися, як клас вже
збільшився до грандіозних розмірів.

Примітивні типи досить часто використовуються для «симуляції» типів. Це
коли замість окремого типу даних у вас є декілька чисел або рядків, які
складаються в список допустимих значень для якоїсь сутності. Найчастіше
цим конкретним числам і рядкам дають зрозумілі імена за допомогою
констант, що і стає причиною їх широкого поширення.

Ще одним поганим методом використання примітивних типів є «симуляція»
полів. При цьому клас містить великий масив різноманітних даних, а в
ролі індексів масиву для отримання цих даних використовуються строкові
константи, задані в класі.

### Лікування

-   Якщо ви маєте безліч різноманітних полів примітивних типів, можливо,
    деякі з них можна логічно згрупувати і перенести в свій власний
    клас. Ще краще, якщо в цей клас ви зможете перенести і поведінку,
    пов'язану з цими даними. Впоратися з цією проблемою допоможе [заміна
    значення даних об'єктом](../replace-data-value-with-object.html).

    <figure class="image">
    <img
    src="../../images/refactoring/content/smells/primitive-obsession-02edeb.png?id=69dfd06eec8d6053e9d88c03ecc78044"
    style="aspect-ratio: 1.67;"
    srcset="/images/refactoring/content/smells/primitive-obsession-02.png?id=69dfd06eec8d6053e9d88c03ecc78044 500w,/images/refactoring/content/smells/primitive-obsession-02-1.5x.png?id=6531a2995361be154aa01d81aa9340a0 750w,/images/refactoring/content/smells/primitive-obsession-02-2x.png?id=255bbb1e0b340ce3b62a5898e8edc75a 1000w"
    sizes="(max-width: 720px) 100vw, 500px" loading="lazy" width="500"
    height="300" />
    </figure>

-   Якщо значення цих полів використовуються в параметрах методів,
    використайте [заміну параметрів
    об'єктом](../introduce-parameter-object.html) або [передачу всього
    об'єкта](../preserve-whole-object.html).

-   У випадках, коли в змінних закодовані якісь складні дані,
    використайте [заміну кодування типу
    класом](../replace-type-code-with-class.html), [заміну кодування
    типу підкласами](../replace-type-code-with-subclasses.html) або
    [заміну кодування типу
    станом/стратегією](../replace-type-code-with-state-strategy.html).

-   Якщо серед змінних є масиви, використайте [заміну масиву
    об'єктом](../replace-array-with-object.html).

<figure class="image">
<img
src="../../images/refactoring/content/smells/primitive-obsession-03b918.png?id=ab0a8c7b6188265bb9890424e679af2f"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/primitive-obsession-03.png?id=ab0a8c7b6188265bb9890424e679af2f 500w,/images/refactoring/content/smells/primitive-obsession-03-1.5x.png?id=c3dd77a95422b4727db752c6634cf57f 750w,/images/refactoring/content/smells/primitive-obsession-03-2x.png?id=bec1080bcf2be14eeda69b0090d5d3fb 1000w"
sizes="(max-width: 720px) 100vw, 500px" loading="lazy" width="500"
height="300" />
</figure>

### Виграш

-   Підвищує гнучкість коду завдяки використанню об'єктів замість
    примітивних типів.

-   Покращує розуміння і організацію коду. Операції над певними даними
    тепер зібрані в одному місці, і їх не потрібно шукати по всьому
    коду. Тепер не треба здогадуватися, навіщо створені всі ці дивні
    константи і чому поля містяться в масиві.

-   Може розкрити факти дублювання коду.

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

[Довгий список параметрів []{.fa
.fa-arrow-right}](long-parameter-list.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Повернутись назад

[[]{.fa .fa-arrow-left} Занадто великий клас](large-class.html){.btn
.btn-default rel="prev"}
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
