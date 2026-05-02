:::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::: {.main-content-container .center-content-container}
:::::::::: {.page .text}
::: breadcrumb
[](../../index.html){.home} / [Рефакторинг](../refactoring.html){.type}
/ [Запахи коду](../refactoring/smells.html){.type} /
[Роздувальщики](../refactoring/smells/bloaters.html){.type}
:::

# Групи даних {#групи-даних .title}

::: aka
Також відомий як: [Data Clumps]{style="display:inline-block"}
:::

### Симптоми і ознаки

Іноді в різних частинах коду зустрічаються однакові групи змінних
(наприклад, параметри підключення до бази даних). Такі групи слід
перетворювати на самостійні класи.

<figure class="image">
<img
src="../../images/refactoring/content/smells/data-clumps-018c21.png?id=9d8a38ce942720cee728797eba695a2f"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/data-clumps-01.png?id=9d8a38ce942720cee728797eba695a2f 500w,/images/refactoring/content/smells/data-clumps-01-1.5x.png?id=148bb6bd87a02e66d83bc11bda3ae7e6 750w,/images/refactoring/content/smells/data-clumps-01-2x.png?id=64c7f4113e3c06f10dbec825833fa190 1000w"
sizes="(max-width: 720px) 100vw, 500px" data-fetchpriority="high"
width="500" height="300" />
</figure>

### Причини появи

Поява груп даних є наслідком поганої структурованості програми або
програмування методом копіювання-вставки.

Щоб виявити групу даних, достатньо видалити одне зі значень даних і
перевірити, чи збережуть сенс інші. Якщо ні, це вірна ознака того, що
група змінних напрошується на об'єднання їх в об'єкт.

### Лікування

-   Якщо дані, що повторюються, є полями якогось класу, використайте
    [відокремлення класу](../extract-class.html) для переміщення полів у
    власний клас.

-   Якщо ті ж групи даних передаються в параметрах методів, використайте
    [заміну параметрів об'єктом](../introduce-parameter-object.html) щоб
    виділити їх в спільний клас.

-   Якщо деякі з цих даних передаються в інші методи, подумайте про
    можливість передачі в метод усього об'єкта даних замість окремих
    полів (у цьому допоможе [передача всього
    об'єкта](../preserve-whole-object.html)).

-   Подивіться на код, який використовує ці поля. Можливо, має сенс
    перенести цей код в клас даних.

<figure class="image">
<img
src="../../images/refactoring/content/smells/data-clumps-02ec1e.png?id=cfb0a8fa64a983473dd31527e28ae158"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/data-clumps-02.png?id=cfb0a8fa64a983473dd31527e28ae158 500w,/images/refactoring/content/smells/data-clumps-02-1.5x.png?id=2dbf0e8e5960f6797d32abbbc1579d3f 750w,/images/refactoring/content/smells/data-clumps-02-2x.png?id=195f24da42dbd21508aa9ef46a57abba 1000w"
sizes="(max-width: 720px) 100vw, 500px" loading="lazy" width="500"
height="300" />
</figure>

### Виграш

-   Покращує розуміння і організацію коду. Операції над певними даними
    тепер зібрані в одному місці, їх не потрібно шукати за всім кодом.

-   Зменшує розмір коду.

<figure class="image">
<img
src="../../images/refactoring/content/smells/data-clumps-038987.png?id=c170bbdea77b7d4a26947ef328b70adc"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/data-clumps-03.png?id=c170bbdea77b7d4a26947ef328b70adc 500w,/images/refactoring/content/smells/data-clumps-03-1.5x.png?id=88875edb97b4047aa9bfb2fd319c0f19 750w,/images/refactoring/content/smells/data-clumps-03-2x.png?id=2b4a70e09a6236715a9bc4bd4663508b 1000w"
sizes="(max-width: 720px) 100vw, 500px" loading="lazy" width="500"
height="300" />
</figure>

### Не варто чіпати, якщо\...

-   Передача всього об'єкта в параметрах методу замість передачі його
    значень (елементарних типів) може створити небажану залежність між
    двома класами.

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

[Порушники об\'єктно-орієнтованого дизайну []{.fa
.fa-arrow-right}](../refactoring/smells/oo-abusers.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Повернутись назад

[[]{.fa .fa-arrow-left} Довгий список
параметрів](long-parameter-list.html){.btn .btn-default rel="prev"}
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
