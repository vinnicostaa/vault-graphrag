::::::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [Патерни
проектування](../design-patterns.html){.type} / [Структурні
патерни](structural-patterns.html){.type}
:::

# Замісник {#замісник .title}

::: aka
Також відомий як: [Proxy]{style="display:inline-block"}
:::

::: {.section .intent}
##  Суть патерна {#intent}

**Замісник** --- це структурний патерн проектування, що дає змогу
підставляти замість реальних об'єктів спеціальні об'єкти-замінники. Ці
об'єкти перехоплюють виклики до оригінального об'єкта, дозволяючи
зробити щось *до* чи *після* передачі виклику оригіналові.

<figure class="image">
<img
src="../../images/patterns/content/proxy/proxyf258.png?id=efece4647fb11e3f7539291796327666"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/proxy/proxy.png?id=efece4647fb11e3f7539291796327666 640w,/images/patterns/content/proxy/proxy-1.5x.png?id=80a11690fa49172c517470a834289b60 960w,/images/patterns/content/proxy/proxy-2x.png?id=fb3d14e21c210a758d4777f4d93dce09 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Патерн Замісник" />
</figure>
:::

::: {.section .problem}
##  Проблема {#problem}

Для чого взагалі контролювати доступ до об'єктів? Розглянемо такий
приклад: у вас є зовнішній ресурсоємний об'єкт, який потрібен не весь
час, а лише зрідка.

<figure class="image">
<img
src="../../images/patterns/diagrams/proxy/problem-uka522.png?id=34adcac67bb84430d067ebb1eb5c0fc0"
style="aspect-ratio: 3.19;"
srcset="/images/patterns/diagrams/proxy/problem-uk.png?id=34adcac67bb84430d067ebb1eb5c0fc0 510w,/images/patterns/diagrams/proxy/problem-uk-1.5x.png?id=8bb932c320a437ea8482301467549ca3 765w,/images/patterns/diagrams/proxy/problem-uk-2x.png?id=ace4f55dfbe4b8e33a67d90750de49d2 1020w"
sizes="(max-width: 720px) 100vw, 510px" loading="lazy" width="510"
alt="Проблема, яку вирішує Замісник" />
<figcaption><p>Запити до бази даних можуть бути
дуже повільними.</p></figcaption>
</figure>

Ми могли б створювати цей об'єкт не на самому початку програми, а тільки
тоді, коли він реально кому-небудь знадобиться. Кожен клієнт об'єкта
отримав би деякий код відкладеної ініціалізації. Це, ймовірно, призвело
б до дублювання великої кількості коду.

В ідеалі цей код хотілося б помістити безпосередньо до службового класу,
але це не завжди можливо. Наприклад, код класу може знаходитися в
закритій сторонній бібліотеці.
:::

::: {.section .solution}
##  Рішення {#solution}

Патерн Замісник пропонує створити новий клас-дублер, який має той самий
інтерфейс, що й оригінальний службовий об'єкт. При отриманні запиту від
клієнта об'єкт-замісник сам би створював примірник службового об'єкта та
переадресовував би йому всю реальну роботу.

<figure class="image">
<img
src="../../images/patterns/diagrams/proxy/solution-uk28be.png?id=9d16a02873494c38d1bf215b86195ada"
style="aspect-ratio: 3.19;"
srcset="/images/patterns/diagrams/proxy/solution-uk.png?id=9d16a02873494c38d1bf215b86195ada 510w,/images/patterns/diagrams/proxy/solution-uk-1.5x.png?id=22614e8d3217a4135392fa6f2d716e86 765w,/images/patterns/diagrams/proxy/solution-uk-2x.png?id=fbfaefd00f862d73fb1c1f44a7078561 1020w"
sizes="(max-width: 720px) 100vw, 510px" loading="lazy" width="510"
alt="Рішення з допомогою Замісника" />
<figcaption><p>Замісник «прикидається» базою даних, прискорюючи роботу
внаслідок ледачої ініціалізації і кешування запитів,
що повторюються.</p></figcaption>
</figure>

Але в чому ж його користь? Ви могли б помістити до класу замісника якусь
проміжну логіку, що виконувалася б до або після викликів цих самих
методів чинного об'єкта. А завдяки однаковому інтерфейсу об'єкт-замісник
можна передати до будь-якого коду, що очікує на сервісний об'єкт.
:::

::: {.section .analogy}
##  Аналогія з життя {#analogy}

<figure class="image">
<img
src="../../images/patterns/diagrams/proxy/live-example66da.png?id=a268c57fdaf073ee81cf4dfc7239eae2"
style="aspect-ratio: 2.57;"
srcset="/images/patterns/diagrams/proxy/live-example.png?id=a268c57fdaf073ee81cf4dfc7239eae2 540w,/images/patterns/diagrams/proxy/live-example-1.5x.png?id=4b8ee7f90cf76180004e9f5d37661ee2 810w,/images/patterns/diagrams/proxy/live-example-2x.png?id=8b8083fa1954d2d92ca84a5a5636ead7 1080w"
sizes="(max-width: 720px) 100vw, 540px" loading="lazy" width="540"
alt="Платіжна картка та готівка" />
<figcaption><p>Платіжною карткою можна розраховуватися так само, як
і готівкою.</p></figcaption>
</figure>

Платіжна картка --- це замісник пачки готівки. І чек, і готівка мають
спільний інтерфейс --- ними обома можна оплачувати товари. Вигода
покупця в тому, що не потрібно носити з собою «тонни» готівки. З іншого
боку власник магазину не змушений замовляти клопітку інкасацію коштів з
магазину, бо вони потрапляють безпосередньо на його банківський рахунок.
:::

::::: {.section .structure-container}
##  Структура {#structure}

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/proxy/structure7fcd.png?id=f2478a82a84e1a1e512a8414bf1abd1c"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1;"
srcset="/images/patterns/diagrams/proxy/structure.png?id=f2478a82a84e1a1e512a8414bf1abd1c 370w,/images/patterns/diagrams/proxy/structure-1.5x.png?id=0db7bf3c1193b2a1961894349f1e07ad 555w,/images/patterns/diagrams/proxy/structure-2x.png?id=3d54eeca9af4aa373e989a73463539b5 740w"
sizes="(max-width: 720px) 100vw, 370px" loading="lazy" width="370"
alt="Структура класів патерна Замісник" /><img
src="../../images/patterns/diagrams/proxy/structure-indexed2b2c.png?id=e56d420f31925b3d41348c5036d3cd64"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.05;"
srcset="/images/patterns/diagrams/proxy/structure-indexed.png?id=e56d420f31925b3d41348c5036d3cd64 410w,/images/patterns/diagrams/proxy/structure-indexed-1.5x.png?id=5b24af632e76d81d24eab0b7c0be1a66 615w,/images/patterns/diagrams/proxy/structure-indexed-2x.png?id=ddf2b3b4807e52330c456c59fc52d504 820w"
sizes="(max-width: 720px) 100vw, 410px" loading="lazy" width="410"
alt="Структура класів патерна Замісник" />
</figure>
:::

1.  **Інтерфейс сервісу** визначає загальний інтерфейс для сервісу й
    замісника. Завдяки цьому об'єкт замісника можна використовувати там,
    де очікується об'єкт сервісу.

2.  **Сервіс** містить корисну бізнес-логіку.

3.  **Замісник** зберігає посилання на об'єкт сервісу. Після того, як
    замісник закінчує свою роботу (наприклад, ініціалізацію, логування,
    захист або інше), він передає виклики вкладеному сервісу.

    Замісник може сам відповідати за створення й видалення об'єкта
    сервісу.

4.  **Клієнт** працює з об'єктами через інтерфейс сервісу. Завдяки цьому
    його можна «обдурити», підмінивши об'єкт сервісу об'єктом замісника.
::::
:::::

::: {.section .pseudocode}
##  Псевдокод {#pseudocode}

У цьому прикладі **Замісник** допомагає додати до програми механізм
ледачої ініціалізації та кешування результатів роботи бібліотеки
інтеграції з YouTube.

<figure class="image">
<img
src="../../images/patterns/diagrams/proxy/example4e80.png?id=ddb1402479562b9e2c776933cc861bed"
style="aspect-ratio: 1.23;"
srcset="/images/patterns/diagrams/proxy/example.png?id=ddb1402479562b9e2c776933cc861bed 490w,/images/patterns/diagrams/proxy/example-1.5x.png?id=9b7608e1ef46a52d4ca1d7d89986e5b0 735w,/images/patterns/diagrams/proxy/example-2x.png?id=40f22d12d183b9285a380e4a072efb16 980w"
sizes="(max-width: 720px) 100vw, 490px" loading="lazy" width="490"
alt="Структура класів прикладу патерна Замісник" />
<figcaption><p>Приклад кешування результатів роботи реального сервісу за
допомогою замісника.</p></figcaption>
</figure>

Оригінальний об'єкт починав завантаження з мережі, навіть якщо
користувач повторно запитував одне й те саме відео. Замісник завантажує
відео тільки один раз, використовуючи для цього службовий об'єкт, але в
інших випадках повертає закешований файл.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// Інтерфейс віддаленого сервісу.
interface ThirdPartyYouTubeLib is
    method listVideos()
    method getVideoInfo(id)
    method downloadVideo(id)

// Конкретна реалізація сервісу. Методи цього класу запитують у
// YouTube різну інформацію. Швидкість запиту залежить не лише
// від якості інтернет-каналу користувача, але й від стану
// самого YouTube. Тому, чим більше буде викликів до сервісу,
// тим менш відзивною стане програма.
class ThirdPartyYouTubeClass implements ThirdPartyYouTubeLib is
    method listVideos() is
        // Отримати список відеороликів за допомогою API
        // YouTube.

    method getVideoInfo(id) is
        // Отримати детальну інформацію про якийсь відеоролик.

    method downloadVideo(id) is
        // Завантажити відео з YouTube.

// З іншого боку, можна кешувати запити до YouTube і не
// повторювати їх деякий час, доки кеш не застаріє. Але внести
// цей код безпосередньо в сервісний клас неможливо, бо він
// знаходиться у сторонній бібліотеці. Тому ми помістимо логіку
// кешування в окремий клас-обгортку. Він буде делегувати запити
// сервісному об&#39;єкту, тільки якщо потрібно безпосередньо
// відіслати запит.
class CachedYouTubeClass implements ThirdPartyYouTubeLib is
    private field service: ThirdPartyYouTubeLib
    private field listCache, videoCache
    field needReset

    constructor CachedYouTubeClass(service: ThirdPartyYouTubeLib) is
        this.service = service

    method listVideos() is
        if (listCache == null || needReset)
            listCache = service.listVideos()
        return listCache

    method getVideoInfo(id) is
        if (videoCache == null || needReset)
            videoCache = service.getVideoInfo(id)
        return videoCache

    method downloadVideo(id) is
        if (!downloadExists(id) || needReset)
            service.downloadVideo(id)

// Клас GUI, який використовує сервісний об&#39;єкт. Замість
// реального сервісу, ми підсунемо йому об&#39;єкт-замісник. Клієнт
// нічого не помітить, так як замісник має той самий інтерфейс,
// що й сервіс.
class YouTubeManager is
    protected field service: ThirdPartyYouTubeLib

    constructor YouTubeManager(service: ThirdPartyYouTubeLib) is
        this.service = service

    method renderVideoPage(id) is
        info = service.getVideoInfo(id)
        // Відобразити сторінку відеоролика.

    method renderListPanel() is
        list = service.listVideos()
        // Відобразити список превью відеороликів.

    method reactOnUserInput() is
        renderVideoPage()
        renderListPanel()

// Конфігураційна частина програми створює та передає клієнтам
// об&#39;єкт замісника.
class Application is
    method init() is
        YouTubeService = new ThirdPartyYouTubeClass()
        YouTubeProxy = new CachedYouTubeClass(YouTubeService)
        manager = new YouTubeManager(YouTubeProxy)
        manager.reactOnUserInput()</code></pre>
</figure>
:::

:::::::::::::: {.section .applicability-container}
##  Застосування {#applicability}

::::::::::::: applicability
::: applicability-problem
Лінива ініціалізація (віртуальний проксі). Коли у вас є важкий об'єкт,
який завантажує дані з файлової системи або бази даних.
:::

::: applicability-solution
Замість того, щоб завантажувати дані відразу після старту програми,
можна заощадити ресурси й створити об'єкт тоді, коли він дійсно
знадобиться.
:::

::: applicability-problem
Захист доступу (захищаючий проксі). Коли в програмі є різні типи
користувачів, і вам хочеться захистити об'єкт від неавторизованого
доступу. Наприклад, якщо ваші об'єкти --- це важлива частина операційної
системи, а користувачі --- сторонні програми (корисні чи шкідливі).
:::

::: applicability-solution
Проксі може перевіряти доступ під час кожного виклику та передавати
виконання службовому об'єкту, якщо доступ дозволено.
:::

::: applicability-problem
Локальний запуск сервісу (віддалений проксі). Коли справжній сервісний
об'єкт знаходиться на віддаленому сервері.
:::

::: applicability-solution
У цьому випадку замісник транслює запити клієнта у виклики через мережу
по протоколу, який є зрозумілим віддаленому сервісу.
:::

::: applicability-problem
Логування запитів (логуючий проксі). Коли потрібно зберігати історію
звернень до сервісного об'єкта.
:::

::: applicability-solution
Замісник може зберігати історію звернення клієнта до сервісного об'єкта.
:::

::: applicability-problem
Кешування об'єктів («розумне» посилання). Коли потрібно кешувати
результати запитів клієнтів і керувати їхнім життєвим циклом.
:::

::: applicability-solution
Замісник може підраховувати кількість посилань на сервісний об'єкт, які
були віддані клієнту та залишаються активними. Коли всі посилання
звільняться, можна буде звільнити і сам сервісний об'єкт (наприклад,
закрити підключення до бази даних).

Крім того, Замісник може відстежувати, чи клієнт не змінював сервісний
об'єкт. Це дозволить повторно використовувати об'єкти й суттєво
заощаджувати ресурси, особливо якщо мова йде про великі «ненажерливі»
сервіси.
:::
:::::::::::::
::::::::::::::

::: {.section .checklist}
##  Кроки реалізації {#checklist}

1.  Визначте інтерфейс, який би зробив замісника та оригінальний об'єкт
    взаємозамінними.

2.  Створіть клас замісника. Він повинен містити посилання на сервісний
    об'єкт. Частіше за все сервісний об'єкт створюється самим
    замісником. У рідкісних випадках замісник отримує готовий сервісний
    об'єкт від клієнта через конструктор.

3.  Реалізуйте методи замісника в залежності від його призначення. У
    більшості випадків, виконавши якусь корисну роботу, методи замісника
    повинні передати запит сервісному об'єкту.

4.  Подумайте про введення фабрики, яка б вирішувала, який з об'єктів
    створювати: замісника або реальний сервісний об'єкт. Проте, з іншого
    боку, ця логіка може бути вкладена до створюючого методу самого
    замісника.

5.  Подумайте, чи не реалізувати вам ліниву ініціалізацію сервісного
    об'єкта при першому зверненні клієнта до методів замісника.
:::

:::::: {.section .pros-cons}
##  Переваги та недоліки {#pros-cons}

::::: row
::: col-sm-6
-    Дозволяє контролювати сервісний об'єкт непомітно для клієнта.
-    Може працювати, навіть якщо сервісний об'єкт ще не створено.
-    Може контролювати життєвий цикл службового об'єкта.
:::

::: col-sm-6
-    Ускладнює код програми внаслідок введення додаткових класів.
-    Збільшує час отримання відклику від сервісу.
:::
:::::
::::::

::: {.section .relations}
##  Відносини з іншими патернами {#relations}

-   З [Адаптером](adapter.html) ви отримуєте доступ до існуючого об'єкта
    через інший інтерфейс. Використовуючи [Замісник](proxy.html),
    інтерфейс залишається незмінним. Використовуючи
    [Декоратор](decorator.html), ви отримуєте доступ до об'єкта через
    розширений інтерфейс.

-   [Фасад](facade.html) схожий на [Замісник](proxy.html) тим, що
    замінює складну підсистему та може сам її ініціалізувати. Але, на
    відміну від *Фасаду*, *Замісник* має такий самий інтерфейс, що і
    його службовий об'єкт, завдяки чому їх можна взаємозаміняти.

-   [Декоратор](decorator.html) та [Замісник](proxy.html) мають схожі
    структури, але різні призначення. Вони схожі тим, що обидва
    побудовані на композиції та делегуванні роботи іншому об'єкту.
    Патерни відрізняються тим, що *Замісник* сам керує життям сервісного
    об'єкта, а обгортання *Декораторів* контролюється клієнтом.
:::

::: {.section .implementations}
##  Приклади реалізації патерна {#implementations}

[![Замісник на
C#](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](proxy/csharp/example.html "Замісник на C#"){.prog-lang-link}
[![Замісник на
C++](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](proxy/cpp/example.html "Замісник на C++"){.prog-lang-link}
[![Замісник на
Go](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](proxy/go/example.html "Замісник на Go"){.prog-lang-link}
[![Замісник на
Java](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](proxy/java/example.html "Замісник на Java"){.prog-lang-link}
[![Замісник на
PHP](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](proxy/php/example.html "Замісник на PHP"){.prog-lang-link}
[![Замісник на
Python](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](proxy/python/example.html "Замісник на Python"){.prog-lang-link}
[![Замісник на
Ruby](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](proxy/ruby/example.html "Замісник на Ruby"){.prog-lang-link}
[![Замісник на
Rust](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](proxy/rust/example.html "Замісник на Rust"){.prog-lang-link}
[![Замісник на
Swift](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](proxy/swift/example.html "Замісник на Swift"){.prog-lang-link}
[![Замісник на
TypeScript](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](proxy/typescript/example.html "Замісник на TypeScript"){.prog-lang-link}
:::

:::::: {#book-promo .banner-set}
::::: {.prom .banner-content .banner-bg .banner-striped .banner-with-image data-id="DP: 1: Ne vtykay" creative-id="standard-uk" position="content_bottom"}
::: banner-image
[![](../../images/patterns/banners/patterns-book-banner-1-b158a.png?id=0877cab2f0102d98cd07b50af0e5beea){width="200"
height="200" loading="lazy"
srcset="/images/patterns/banners/patterns-book-banner-1-b-2x.png?id=5572aa55e5b09e59780aca9e0ea8e44b 2x"}](book.html)
:::

::: banner-text
### Не нудьгуй в транспорті {#не-нудьгуй-в-транспорті .title}

Краще почитай нашу книжку про патерни проектування.

Тепер це зручно робити навіть під час поїздок в громадському транспорті.

[ Дізнатися більше...](book.html){.btn .btn-secondary}
:::
:::::
::::::

::: next
#### Читаємо далі

[Поведінкові патерни []{.fa
.fa-arrow-right}](behavioral-patterns.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Повернутись назад

[[]{.fa .fa-arrow-left} Легковаговик](flyweight.html){.btn .btn-default
rel="prev"}
:::
::::::::::::::::::::::::::::::::::::::

::::::: {.prom .banner-sidebar .banner-removable .banner-removable-patterns data-id="DP: Sidebar" creative-id="standard-sidebar-uk" position="sidebar"}
:::::: banner-inner
:::: image3d-book-right
::: {.image3d-cover style="background: #0b3752;"}
[![](../../images/patterns/book/web-cover-ukc0ed.png?id=03e01a1a94fb4b34fed20e260af002ec){width="250"
height="375" loading="lazy"
srcset="/images/patterns/book/web-cover-uk-2x.png?id=beb38c652838bad2e9e7205bd0d7a293 2x"}](book.html)
:::
::::

::: {style="margin-top: 1rem"}
Ця стаття є частиною нашої електронної книги **Занурення в Патерни
Проектування**.

[ Дізнатися більше...](book.html){.btn .btn-secondary .btn-block}
:::
::::::
:::::::
::::::::::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::::::::::
