::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [Патерни
проектування](../design-patterns.html){.type} / [Породжувальні
патерни](creational-patterns.html){.type}
:::

# Одинак {#одинак .title}

::: aka
Також відомий як: [Singleton]{style="display:inline-block"}
:::

::: {.section .intent}
##  Суть патерна {#intent}

**Одинак** --- це породжувальний патерн проектування, який гарантує, що
клас має лише один екземпляр, та надає глобальну точку доступу до нього.

<figure class="image">
<img
src="../../images/patterns/content/singleton/singleton4263.png?id=108a0b9b5ea5c4426e0afa4504491d6f"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/singleton/singleton.png?id=108a0b9b5ea5c4426e0afa4504491d6f 640w,/images/patterns/content/singleton/singleton-1.5x.png?id=29490ad6cd1426c63cf5f88243a1701c 960w,/images/patterns/content/singleton/singleton-2x.png?id=accb2cc7594f7a491ce01dddf0d2f876 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Патерн Одинак" />
</figure>
:::

::: {.section .problem}
##  Проблема {#problem}

Одинак вирішує відразу дві проблеми (порушуючи *принцип єдиного
обов'язку* класу):

1.  **Гарантує наявність єдиного екземпляра класу**. Найчастіше за все
    це корисно для доступу до якогось спільного ресурсу, наприклад, бази
    даних.

    Уявіть собі, що ви створили об'єкт, а через деякий час намагаєтесь
    створити ще один. У цьому випадку хотілося б отримати старий об'єкт
    замість створення нового.

    Таку поведінку неможливо реалізувати за допомогою звичайного
    конструктора, оскільки конструктор класу **завжди** повертає новий
    об'єкт.

<figure class="image">
<img
src="../../images/patterns/content/singleton/singleton-comic-1-ukc60b.png?id=a6cc49e24c8b70dc73549d5e90e3c3df"
style="aspect-ratio: 2;"
srcset="/images/patterns/content/singleton/singleton-comic-1-uk.png?id=a6cc49e24c8b70dc73549d5e90e3c3df 600w,/images/patterns/content/singleton/singleton-comic-1-uk-1.5x.png?id=29bd2b808ea4294597dbf4551326561b 900w,/images/patterns/content/singleton/singleton-comic-1-uk-2x.png?id=d8d0838eb40bd7a6c6e85b4a1098a196 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Глобальний доступ до одного об&#39;єкта" />
<figcaption><p>Клієнти можуть не підозрювати, що працюють з одним і тим
самим об’єктом.</p></figcaption>
</figure>

2.  **Надає глобальну точку доступу**. Це не просто глобальна змінна,
    через яку можна дістатися до певного об'єкта. Глобальні змінні не
    захищені від запису, тому будь-який код може підмінити їхнє значення
    без вашого відома.

    Проте, є ще одна особливість. Було б непогано й зберігати в одному
    місці код, який вирішує проблему №1, і мати до нього простий та
    доступний інтерфейс.

Цікаво, що в наш час патерн став настільки відомим, що тепер люди
називають «одинаками» навіть ті класи, які вирішують лише одну з
проблем, перерахованих вище.
:::

::: {.section .solution}
##  Рішення {#solution}

Всі реалізації Одинака зводяться до того, аби приховати типовий
конструктор та створити публічний статичний метод, який і контролюватиме
життєвий цикл об'єкта-одинака.

Якщо у вас є доступ до класу одинака, отже, буде й доступ до цього
статичного методу. З якої точки коду ви б його не викликали, він завжди
віддаватиме один і той самий об'єкт.
:::

::: {.section .analogy}
##  Аналогія з життя {#analogy}

Уряд держави --- вдалий приклад Одинака. У державі може бути тільки один
офіційний уряд. Незалежно від того, хто конкретно засідає в уряді, він
має глобальну точку доступу «Уряд країни N».
:::

::::: {.section .structure-container}
##  Структура {#structure}

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/singleton/structure-uk4ff2.png?id=80776b11ac36250b03265d70efaabbeb"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.48;"
srcset="/images/patterns/diagrams/singleton/structure-uk.png?id=80776b11ac36250b03265d70efaabbeb 430w,/images/patterns/diagrams/singleton/structure-uk-1.5x.png?id=661776c6e3499287688728165b7e88e0 645w,/images/patterns/diagrams/singleton/structure-uk-2x.png?id=46e85e8745effa4bb906733337846913 860w"
sizes="(max-width: 720px) 100vw, 430px" loading="lazy" width="430"
alt="Структура класів патерна Одинак" /><img
src="../../images/patterns/diagrams/singleton/structure-uk-indexede4fb.png?id=8a0bdd1977d8d6f81dfd428316aedf09"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.48;"
srcset="/images/patterns/diagrams/singleton/structure-uk-indexed.png?id=8a0bdd1977d8d6f81dfd428316aedf09 430w,/images/patterns/diagrams/singleton/structure-uk-indexed-1.5x.png?id=089a1828f23aa07c295da4c96369c3a3 645w,/images/patterns/diagrams/singleton/structure-uk-indexed-2x.png?id=db41a140bbd2062f60f9148bf6a23091 860w"
sizes="(max-width: 720px) 100vw, 430px" loading="lazy" width="430"
alt="Структура класів патерна Одинак" />
</figure>
:::

1.  **Одинак** визначає статичний метод `getInstance`, який повертає
    один екземпляр свого класу.

    Конструктор Одинака повинен бути прихований від клієнтів. Виклик
    методу `getInstance` повинен стати єдиним способом отримати об'єкт
    цього класу.
::::
:::::

::: {.section .pseudocode}
##  Псевдокод {#pseudocode}

У цьому прикладі роль **Одинака** грає клас підключення до бази даних.

Цей клас не має публічного конструктора, тому єдиним способом отримання
його об'єкта є виклик методу `getInstance`. Цей метод збереже перший
створений об'єкт і повертатиме його в усіх наступних викликах.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// Клас одинака визначає статичний метод `getInstance`, котрий
// дозволяє клієнтам повторно використовувати одне і теж
// підключення до бази даних по всій програмі.
class Database is
    // Поле для зберігання об&#39;єкта-одинака має бути оголошено
    // статичним.
    private static field instance: Database

    // Конструктор одинака завжди повинен залишатися приватним,
    // аби клієнти не могли самостійно створювати екземпляри
    // цього класу через оператор `new`.
    private constructor Database() is
        // Тут може жити код ініціалізації підключення до
        // сервера баз даних.
        // ...

    // Головний статичний метод одинака служить альтернативою
    // конструктору і є точкою доступу до екземпляра цього
    // класу.
    public static method getInstance() is
        if (Database.instance == null) then
            acquireThreadLock() and then
                // Про всяк випадок, ще раз перевіримо, чи не
                // було створено об&#39;єкт в іншому потоці, поки
                // даний потік чекав на звільнення блокування.
                if (Database.instance == null) then
                    Database.instance = new Database()
        return Database.instance

    // І, нарешті, будь-який клас одинака повинен мати якусь
    // корисну функціональність, яку клієнти будуть запускати
    // через отриманий об&#39;єкт одинака.
    public method query(sql) is
        // Усі запити до бази даних проходитимуть через цей
        // метод. Тому є сенс помістити сюди якусь логіку
        // кешування.
        // ...

class Application is
    method main() is
        Database foo = Database.getInstance()
        foo.query(&quot;SELECT ...&quot;)
        // ...
        Database bar = Database.getInstance()
        bar.query(&quot;SELECT ...&quot;)
        // Змінна &quot;bar&quot; містить той самий об&#39;єкт, що і змінна
        // &quot;foo&quot;.</code></pre>
</figure>
:::

:::::::: {.section .applicability-container}
##  Застосування {#applicability}

::::::: applicability
::: applicability-problem
Коли в програмі повинен бути єдиний екземпляр якого-небудь класу,
доступний усім клієнтам (наприклад, спільний доступ до бази даних з
різних частин програми).
:::

::: applicability-solution
Одинак приховує від клієнтів всі способи створення нового об'єкта, окрім
спеціального методу. Цей метод або створює об'єкт, або віддає існуючий
об'єкт, якщо він вже був створений.
:::

::: applicability-problem
Коли ви хочете мати більше контролю над глобальними змінними.
:::

::: applicability-solution
На відміну від глобальних змінних, Одинак гарантує, що жоден інший код
не замінить створений екземпляр класу, тому ви завжди впевнені в
наявності лише одного об'єкта-одинака.

Тим не менше, будь-коли ви можете розширити це обмеження і дозволити
будь-яку кількість об'єктів-одинаків, змінивши код в одному місці (метод
`getInstance`).
:::
:::::::
::::::::

::: {.section .checklist}
##  Кроки реалізації {#checklist}

1.  Додайте до класу приватне статичне поле, котре міститиме одиночний
    об'єкт.

2.  Оголосіть статичний створюючий метод, що використовуватиметься для
    отримання Одинака.

3.  Додайте «ліниву ініціалізацію» (створення об'єкта під час першого
    виклику методу) до створюючого методу одинака.

4.  Зробіть конструктор класу приватним.

5.  У клієнтському коді замініть прямі виклики конструктора одинака на
    виклики його створюючого методу.
:::

:::::: {.section .pros-cons}
##  Переваги та недоліки {#pros-cons}

::::: row
::: col-sm-6
-    Гарантує наявність єдиного екземпляра класу.
-    Надає глобальну точку доступу до нього.
-    Реалізує відкладену ініціалізацію об'єкта-одинака.
:::

::: col-sm-6
-    Порушує *принцип єдиного обов'язку класу*.
-    Маскує поганий дизайн.
-    Проблеми багатопоточності.
-    Вимагає постійного створення Mock-об'єктів при юніт-тестуванні.
:::
:::::
::::::

::: {.section .relations}
##  Відносини з іншими патернами {#relations}

-   [Фасад](facade.html) можна зробити [Одинаком](singleton.html),
    оскільки зазвичай потрібен тільки один об'єкт-фасад.

-   Патерн [Легковаговик](flyweight.html) може нагадувати
    [Одинака](singleton.html), якщо для конкретного завдання ви змогли
    зменшити кількість об'єктів до одного. Але пам'ятайте, що між
    патернами є дві суттєві відмінності:

    1.  На відміну від *Одинака*, ви можете мати безліч
        об'єктів-легковаговиків.
    2.  Об'єкти-легковаговики повинні бути незмінними, тоді як
        об'єкт-одинак допускає зміну свого стану.

-   [Абстрактна фабрика](abstract-factory.html),
    [Будівельник](builder.html) та [Прототип](prototype.html) можуть
    реалізовуватися за допомогою [Одинака](singleton.html).
:::

::: {.section .implementations}
##  Приклади реалізації патерна {#implementations}

[![Одинак на
C#](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](singleton/csharp/example.html "Одинак на C#"){.prog-lang-link}
[![Одинак на
C++](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](singleton/cpp/example.html "Одинак на C++"){.prog-lang-link}
[![Одинак на
Go](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](singleton/go/example.html "Одинак на Go"){.prog-lang-link}
[![Одинак на
Java](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](singleton/java/example.html "Одинак на Java"){.prog-lang-link}
[![Одинак на
PHP](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](singleton/php/example.html "Одинак на PHP"){.prog-lang-link}
[![Одинак на
Python](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](singleton/python/example.html "Одинак на Python"){.prog-lang-link}
[![Одинак на
Ruby](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](singleton/ruby/example.html "Одинак на Ruby"){.prog-lang-link}
[![Одинак на
Rust](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](singleton/rust/example.html "Одинак на Rust"){.prog-lang-link}
[![Одинак на
Swift](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](singleton/swift/example.html "Одинак на Swift"){.prog-lang-link}
[![Одинак на
TypeScript](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](singleton/typescript/example.html "Одинак на TypeScript"){.prog-lang-link}
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

[Структурні патерни []{.fa
.fa-arrow-right}](structural-patterns.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Повернутись назад

[[]{.fa .fa-arrow-left} Прототип](prototype.html){.btn .btn-default
rel="prev"}
:::
::::::::::::::::::::::::::::::::

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
::::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::::
