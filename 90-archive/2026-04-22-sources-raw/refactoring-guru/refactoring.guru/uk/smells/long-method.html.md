:::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::: {.main-content-container .center-content-container}
:::::::::: {.page .text}
::: breadcrumb
[](../../index.html){.home} / [Рефакторинг](../refactoring.html){.type}
/ [Запахи коду](../refactoring/smells.html){.type} /
[Роздувальщики](../refactoring/smells/bloaters.html){.type}
:::

# Довгий метод {#довгий-метод .title}

::: aka
Також відомий як: [Long Method]{style="display:inline-block"}
:::

### Симптоми і ознаки

Метод містить занадто багато коду. Довжина методу більше десяти рядків
повинна починати вас турбувати.

<figure class="image">
<img
src="../../images/refactoring/content/smells/long-method-012095.png?id=ba3b4a6d8ef25a8f676543cee5e1e019"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/long-method-01.png?id=ba3b4a6d8ef25a8f676543cee5e1e019 500w,/images/refactoring/content/smells/long-method-01-1.5x.png?id=0aba5fa284aac80c0c3c909fca9ff51c 750w,/images/refactoring/content/smells/long-method-01-2x.png?id=fd9479c2221526f284c9b14fb94f9169 1000w"
sizes="(max-width: 720px) 100vw, 500px" data-fetchpriority="high"
width="500" height="300" />
</figure>

### Причини появи

В метод весь час щось додається, але нічого не виноситься. Оскільки
писати код набагато простіше, ніж читати, цей запах довго залишається
непоміченим --- аж поки метод не перетвориться на справжнього монстра.

Варто пам'ятати, що людині ментально складніше створити новий метод, ніж
дописати щось у вже існуючий: «Мені потрібно додати всього два рядки, не
буду ж я створювати заради цього цілий метод».

Таким чином додається один рядок за іншим, внаслідок чого метод
перетворюється на велику тарілку спагеті.

### Лікування

Слід дотримуватися такого правила: якщо відчувається необхідність щось
прокоментувати всередині методу, цей код краще виділити в новий метод.
Навіть один рядок має сенс виділяти в метод, якщо він потребує
роз'яснень. До того ж, якщо метод має гарну назву, то не потрібно буде
дивитися в код заради розуміння, що ж він робить.

<figure class="image">
<img
src="../../images/refactoring/content/smells/long-method-027655.png?id=274350a92b305ae79848ab40b3bdb0cb"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/long-method-02.png?id=274350a92b305ae79848ab40b3bdb0cb 500w,/images/refactoring/content/smells/long-method-02-1.5x.png?id=dbb0cb724dd6beefe6474b86ae33913c 750w,/images/refactoring/content/smells/long-method-02-2x.png?id=beba19e840bf4763e85f006ef79cc89c 1000w"
sizes="(max-width: 720px) 100vw, 500px" loading="lazy" width="500"
height="300" />
</figure>

-   Для скорочення тіла методу досить застосувати [відокремлення
    методу](../extract-method.html).

-   Якщо локальні змінні і параметри перешкоджають виділенню методу,
    можна застосувати [заміну тимчасової змінної викликом
    методу](../replace-temp-with-query.html), [заміну параметрів
    об'єктом](../introduce-parameter-object.html) і [передачу всього
    об'єкта](../preserve-whole-object.html).

-   Якщо і вони не допомогли, можна спробувати виділити весь метод в
    окремий об'єкт за допомогою [заміни методу об'єктом
    методів](../replace-method-with-method-object.html).

-   Умовні оператори і цикли свідчать про можливість виділення коду в
    окремий метод. Для роботи з умовними виразами підійде [декомпозиція
    умовних операторів](../decompose-conditional.html). Для роботи з
    циклом --- [відокремлення методу](../extract-method.html).

### Виграш

-   З усіх видів об'єктного коду найкраще виживають класи з короткими
    методами. Чим довше ваш метод або функція, тим важче буде її
    зрозуміти і підтримувати.

-   Крім того, в довгих методах найчастіше зустрічаються «поклади»
    дублювання коду.

<figure class="image">
<img
src="../../images/refactoring/content/smells/long-method-03ca39.png?id=82ce2d388aa14bdae4e8f62b875f0259"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/long-method-03.png?id=82ce2d388aa14bdae4e8f62b875f0259 500w,/images/refactoring/content/smells/long-method-03-1.5x.png?id=fd71378acb56437e26c4b9b5654a3f28 750w,/images/refactoring/content/smells/long-method-03-2x.png?id=ba563da468cf42a704ff53da2e89b6d5 1000w"
sizes="(max-width: 720px) 100vw, 500px" loading="lazy" width="500"
height="300" />
</figure>

### Швидкодія

Багато хто хвилюється, що збільшення числа методів може погано
позначитися на швидкодії програми. В абсолютній більшості випадків це не
є реальною проблемою, а тому --- **досить про це хвилюватися!**

Маючи чистий і зрозумілий код, ви з більшою вірогідністю побачите
класний варіант реструктуризації коду програми і збільшення реальної
продуктивності.

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

[Занадто великий клас []{.fa .fa-arrow-right}](large-class.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Повернутись назад

[[]{.fa
.fa-arrow-left} Роздувальщики](../refactoring/smells/bloaters.html){.btn
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
