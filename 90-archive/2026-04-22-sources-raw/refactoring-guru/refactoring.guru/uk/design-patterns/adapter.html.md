:::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [Патерни
проектування](../design-patterns.html){.type} / [Структурні
патерни](structural-patterns.html){.type}
:::

# Адаптер {#адаптер .title}

::: aka
Також відомий як:
[Wrapper, ]{style="display:inline-block"}[Обгортка, ]{style="display:inline-block"}[Adapter]{style="display:inline-block"}
:::

::: {.section .intent}
##  Суть патерна {#intent}

**Адаптер** --- це структурний патерн проектування, що дає змогу
об'єктам із несумісними інтерфейсами працювати разом.

<figure class="image">
<img
src="../../images/patterns/content/adapter/adapter-en6d0c.png?id=11ef6ae6177291834323e3f918c47cd2"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/adapter/adapter-en.png?id=11ef6ae6177291834323e3f918c47cd2 640w,/images/patterns/content/adapter/adapter-en-1.5x.png?id=ad30c3ed32ca2da3eb6b8d9116506764 960w,/images/patterns/content/adapter/adapter-en-2x.png?id=e0ab0f6103b0b7b0648a8fda592ffab8 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Патерн Адаптер" />
</figure>
:::

::: {.section .problem}
##  Проблема {#problem}

Уявіть, що ви пишете програму для торгівлі на біржі. Ваша програма
спочатку завантажує біржові котирування з декількох джерел в XML, а
потім малює гарні графіки.

У якийсь момент ви вирішуєте покращити програму, застосувавши сторонню
бібліотеку аналітики. Але от біда --- бібліотека підтримує тільки формат
даних JSON, несумісний із вашим додатком.

<figure class="image">
<img
src="../../images/patterns/diagrams/adapter/problem-en73b1.png?id=60d01f6c72ba85030cd52d5955caa3d8"
style="aspect-ratio: 2.41;"
srcset="/images/patterns/diagrams/adapter/problem-en.png?id=60d01f6c72ba85030cd52d5955caa3d8 530w,/images/patterns/diagrams/adapter/problem-en-1.5x.png?id=9d1ebfe5843bea44ad4025d4c648b34a 795w,/images/patterns/diagrams/adapter/problem-en-2x.png?id=f6f4bfbd2d6136a5ae4fb8c899e9854e 1060w"
sizes="(max-width: 720px) 100vw, 530px" loading="lazy" width="530"
alt="Структура програми до підключення сторонньої бібліотеки" />
<figcaption><p>Під’єднати сторонню бібліотеку неможливо через
несумісність форматів даних.</p></figcaption>
</figure>

Ви могли б переписати цю бібліотеку, щоб вона підтримувала формат XML,
але, по-перше, це може порушити роботу наявного коду, який уже залежить
від бібліотеки, по-друге, у вас може просто не бути доступу до її
вихідного коду.
:::

::: {.section .solution}
##  Рішення {#solution}

Ви можете створити *адаптер*. Це об'єкт-перекладач, який трансформує
інтерфейс або дані одного об'єкта таким чином, щоб він став зрозумілим
іншому об'єкту.

Адаптер загортає один з об'єктів так, що інший об'єкт навіть не підозрює
про існування першого. Наприклад, об'єкт, що працює в метричній системі
вимірювання, можна «обгорнути» адаптером, який буде конвертувати дані у
фути.

Адаптери можуть не тільки конвертувати дані з одного формату в інший,
але й допомагати об'єктам із різними інтерфейсами працювати разом. Це
виглядає так:

1.  Адаптер має інтерфейс, сумісний з одним із об'єктів.
2.  Тому цей об'єкт може вільно викликати методи адаптера.
3.  Адаптер отримує ці виклики та перенаправляє їх іншому об'єкту, але
    вже в тому форматі та послідовності, які є зрозумілими для цього
    об'єкта.

Іноді вдається створити навіть *двосторонній адаптер*, який може
працювати в обох напрямках.

<figure class="image">
<img
src="../../images/patterns/diagrams/adapter/solution-en716c.png?id=5f4f1b4575236a3853f274b690bd6656"
style="aspect-ratio: 1.51;"
srcset="/images/patterns/diagrams/adapter/solution-en.png?id=5f4f1b4575236a3853f274b690bd6656 530w,/images/patterns/diagrams/adapter/solution-en-1.5x.png?id=40da8d488b9776aca5011c6ca96c5da4 795w,/images/patterns/diagrams/adapter/solution-en-2x.png?id=5846ed9b88cad0220993f79bdfe817a8 1060w"
sizes="(max-width: 720px) 100vw, 530px" loading="lazy" width="530"
alt="Структура програми після застосування адаптера" />
<figcaption><p>Програма може працювати зі сторонньою бібліотекою
через адаптер.</p></figcaption>
</figure>

Таким чином, для програми біржових котирувань ви могли б створити клас
`XML_To_JSON_Adapter`, який би обгортав об'єкт того чи іншого класу
бібліотеки аналітики. Ваш код посилав би адаптеру запити у форматі XML,
а адаптер спочатку б транслював вхідні дані у формат JSON, а потім
передавав їх методам загорнутого об'єкта аналітики.
:::

::: {.section .analogy}
##  Аналогія з життя {#analogy}

<figure class="image">
<img
src="../../images/patterns/content/adapter/adapter-comic-1-uka932.png?id=12d290e27da1b159ac662de8719c80c2"
style="aspect-ratio: 1.78;"
srcset="/images/patterns/content/adapter/adapter-comic-1-uk.png?id=12d290e27da1b159ac662de8719c80c2 533w,/images/patterns/content/adapter/adapter-comic-1-uk-1.5x.png?id=eb0dafb5ea4b325293bff5cefcd71593 800w,/images/patterns/content/adapter/adapter-comic-1-uk-2x.png?id=bcd1d0b2238b98ad2c9e60f1ed3be5b0 1067w"
sizes="(max-width: 720px) 100vw, 533px" loading="lazy" width="533"
alt="Приклад патерна Адаптер" />
<figcaption><p>Вміст валіз до й після поїздки
за кордон.</p></figcaption>
</figure>

Під час вашої першої подорожі за кордон спроба зарядити ноутбук може
стати неприємним сюрпризом, тому що стандарти розеток у багатьох країнах
різняться. Ваша європейська зарядка стане непотрібом у США без
спеціального адаптера, що дозволяє під'єднуватися до розетки іншого
типу.
:::

:::::::: {.section .structure-container}
##  Структура {#structure}

::::::: structure
#### Адаптер об'єктів

Ця реалізація використовує агрегацію: об'єкт адаптера «загортає», тобто
містить посилання на службовий об'єкт. Такий підхід працює в усіх мовах
програмування.

:::: adapter-object
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/adapter/structure-object-adaptera83b.png?id=33dffbe3aece294162440c7ddd3d5d4f"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.81;"
srcset="/images/patterns/diagrams/adapter/structure-object-adapter.png?id=33dffbe3aece294162440c7ddd3d5d4f 580w,/images/patterns/diagrams/adapter/structure-object-adapter-1.5x.png?id=c1b8a87b74fc4ce5639212fe19ee06fe 870w,/images/patterns/diagrams/adapter/structure-object-adapter-2x.png?id=03e8052e168c962d6bc369bbb13b0945 1160w"
sizes="(max-width: 720px) 100vw, 580px" loading="lazy" width="580"
alt="Структура класів патерна Адаптер (адаптер об’єктів)" /><img
src="../../images/patterns/diagrams/adapter/structure-object-adapter-indexedd6ac.png?id=a20b311948b361a058097e5bcdbf067a"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.76;"
srcset="/images/patterns/diagrams/adapter/structure-object-adapter-indexed.png?id=a20b311948b361a058097e5bcdbf067a 600w,/images/patterns/diagrams/adapter/structure-object-adapter-indexed-1.5x.png?id=72a1c8fde064c4915379fb34a1a66881 900w,/images/patterns/diagrams/adapter/structure-object-adapter-indexed-2x.png?id=759771515f08d74d53cf4fe500f814a3 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Структура класів патерна Адаптер (адаптер об’єктів)" />
</figure>
:::
::::

1.  **Клієнт** --- це клас, який містить існуючу бізнес-логіку програми.

2.  **Клієнтський інтерфейс** описує протокол, через який клієнт може
    працювати з іншими класами.

3.  **Сервіс** --- це який-небудь корисний клас, зазвичай сторонній.
    Клієнт не може використовувати цей клас безпосередньо, оскільки
    сервіс має незрозумілий йому інтерфейс.

4.  **Адаптер** --- це клас, який може одночасно працювати і з клієнтом,
    і з сервісом. Він реалізує клієнтський інтерфейс і містить посилання
    на об'єкт сервісу. Адаптер отримує виклики від клієнта через методи
    клієнтського інтерфейсу, а потім конвертує їх у виклики методів
    загорнутого об'єкта в потрібному форматі.

5.  Працюючи з адаптером через інтерфейс, клієнт не прив'язується до
    конкретного класу адаптера. Завдяки цьому ви можете додавати до
    програми нові види адаптерів, незалежно від клієнтського коду. Це
    може стати в нагоді, якщо інтерфейс сервісу раптом зміниться,
    наприклад, після виходу нової версії сторонньої бібліотеки.

#### Адаптер класів

Ця реалізація базується на спадкуванні: адаптер успадковує обидва
інтерфейси одночасно. Такий підхід можливий тільки в мовах, які
підтримують множинне спадкування, наприклад у C++.

:::: adapter-class
::: {.struct-image2 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/adapter/structure-class-adapterf043.png?id=e1c60240508146ed3b98ac562cc8e510"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.72;"
srcset="/images/patterns/diagrams/adapter/structure-class-adapter.png?id=e1c60240508146ed3b98ac562cc8e510 550w,/images/patterns/diagrams/adapter/structure-class-adapter-1.5x.png?id=299a79bdfa10ac53213ba02408255430 825w,/images/patterns/diagrams/adapter/structure-class-adapter-2x.png?id=ddca3e3e4d972b7c58207daba8d24866 1100w"
sizes="(max-width: 720px) 100vw, 550px" loading="lazy" width="550"
alt="Структура класів патерна Адаптер (адаптер класів)" /><img
src="../../images/patterns/diagrams/adapter/structure-class-adapter-indexedd30e.png?id=250b5c485a7dfba7c16b89a9201538fb"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.72;"
srcset="/images/patterns/diagrams/adapter/structure-class-adapter-indexed.png?id=250b5c485a7dfba7c16b89a9201538fb 550w,/images/patterns/diagrams/adapter/structure-class-adapter-indexed-1.5x.png?id=b56d736f8076d34b1896de0a2b22a219 825w,/images/patterns/diagrams/adapter/structure-class-adapter-indexed-2x.png?id=9ae1182ef2a34d2ea65f4b4f94a4019e 1100w"
sizes="(max-width: 720px) 100vw, 550px" loading="lazy" width="550"
alt="Структура класів патерна Адаптер (адаптер класів)" />
</figure>
:::
::::

1.  **Адаптер класів** не потребує вкладеного об'єкта, тому що він може
    одночасно успадкувати й частину існуючого класу, й частину класу
    сервісу.
:::::::
::::::::

::: {.section .pseudocode}
##  Псевдокод {#pseudocode}

У цьому жартівливому прикладі **Адаптер** перетворює один інтерфейс на
інший, дозволяючи поєднувати квадратні кілочки та круглі отвори.

<figure class="image">
<img
src="../../images/patterns/diagrams/adapter/examplef4c9.png?id=9d2b6857ce256f2c669383ce4df3d0aa"
style="aspect-ratio: 1.68;"
srcset="/images/patterns/diagrams/adapter/example.png?id=9d2b6857ce256f2c669383ce4df3d0aa 640w,/images/patterns/diagrams/adapter/example-1.5x.png?id=76e68b9cea3b371e465e81587e57e5d9 960w,/images/patterns/diagrams/adapter/example-2x.png?id=0ac62d1bc151e8ce6abad8e8502756cf 1280w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="640"
alt="Структура класів прикладу патерна Адаптер" />
<figcaption><p>Приклад адаптації квадратних кілочків та
круглих отворів.</p></figcaption>
</figure>

Адаптер обчислює найменший радіус кола, у яке можна вписати квадратний
кілочок, і подає його як круглий кілочок із цим радіусом.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// Класи з сумісними інтерфейсами: КруглийОтвір та
// КруглийКілочок.
class RoundHole is
    constructor RoundHole(radius) { ... }

    method getRadius() is
        // Повернути радіус отвору.

    method fits(peg: RoundPeg) is
        return this.getRadius() &gt;= peg.getRadius()

class RoundPeg is
    constructor RoundPeg(radius) { ... }

    method getRadius() is
        // Повернути радіус круглого кілочка.


// Застарілий несумісний клас: КвадратнийКілочок.
class SquarePeg is
    constructor SquarePeg(width) { ... }

    method getWidth() is
        // Повернути ширину квадратного кілочка.


// Адаптер дозволяє використовувати квадратні кілочки й круглі
// отвори разом.
class SquarePegAdapter extends RoundPeg is
    private field peg: SquarePeg

    constructor SquarePegAdapter(peg: SquarePeg) is
        this.peg = peg

    method getRadius() is
        // Обчислити половину діагоналі квадратного кілочка за
        // теоремою Піфагора.
        return peg.getWidth() * Math.sqrt(2) / 2


// Десь у клієнтському програмному коді.
hole = new RoundHole(5)
rpeg = new RoundPeg(5)
hole.fits(rpeg) // TRUE

small_sqpeg = new SquarePeg(5)
large_sqpeg = new SquarePeg(10)
hole.fits(small_sqpeg) // Помилка компіляції, несумісні типи.

small_sqpeg_adapter = new SquarePegAdapter(small_sqpeg)
large_sqpeg_adapter = new SquarePegAdapter(large_sqpeg)
hole.fits(small_sqpeg_adapter) // TRUE
hole.fits(large_sqpeg_adapter) // FALSE</code></pre>
</figure>
:::

:::::::: {.section .applicability-container}
##  Застосування {#applicability}

::::::: applicability
::: applicability-problem
Якщо ви хочете використати сторонній клас, але його інтерфейс не
відповідає решті кодів програми.
:::

::: applicability-solution
Адаптер дозволяє створити об'єкт-прокладку, який перетворюватиме виклики
програми у формат, зрозумілий сторонньому класу.
:::

::: applicability-problem
Якщо вам потрібно використати декілька існуючих підкласів, але в них не
вистачає якої-небудь спільної функціональності, а розширити суперклас ви
не можете.
:::

::: applicability-solution
Ви могли б створити ще один рівень підкласів та додати до них забраклу
функціональність. Але при цьому доведеться дублювати один і той самий
код в обох гілках підкласів.

Більш елегантним рішенням було б розмістити відсутню функціональність в
адаптері й пристосувати його для роботи із суперкласом. Такий адаптер
зможе працювати з усіма підкласами ієрархії. Це рішення сильно
нагадуватиме патерн [Декоратор](decorator.html).
:::
:::::::
::::::::

::: {.section .checklist}
##  Кроки реалізації {#checklist}

1.  Переконайтеся, що у вас є два класи з незручними інтерфейсами:

    -   корисний *сервіс* --- службовий клас, який ви не можете
        змінювати (він або сторонній, або від нього залежить інший код);
    -   один або декілька *клієнтів* --- існуючих класів програми, які
        не можуть використовувати сервіс через несумісний із ним
        інтерфейс.

2.  Опишіть клієнтський інтерфейс, через який класи програм могли б
    використовувати клас сервісу.

3.  Створіть клас адаптера, реалізувавши цей інтерфейс.

4.  Розмістіть в адаптері поле, що міститиме посилання на об'єкт
    сервісу. Зазвичай це поле заповнюють об'єктом, переданим у
    конструктор адаптера. Але цей об'єкт можна передавати й
    безпосередньо до методів адаптера.

5.  Реалізуйте всі методи клієнтського інтерфейсу в адаптері. Адаптер
    повинен делегувати основну роботу сервісу.

6.  Програма повинна використовувати адаптер тільки через клієнтський
    інтерфейс. Це дозволить легко змінювати та додавати адаптери в
    майбутньому.
:::

:::::: {.section .pros-cons}
##  Переваги та недоліки {#pros-cons}

::::: row
::: col-sm-6
-    Відокремлює та приховує від клієнта подробиці перетворення різних
    інтерфейсів.
:::

::: col-sm-6
-    Ускладнює код програми внаслідок введення додаткових класів.
:::
:::::
::::::

::: {.section .relations}
##  Відносини з іншими патернами {#relations}

-   [Міст](bridge.html) проектують заздалегідь, щоб розвивати великі
    частини програми окремо одну від одної. [Адаптер](adapter.html)
    застосовується постфактум, щоб змусити несумісні класи працювати
    разом.

-   [Адаптер](adapter.html) надає зовсім інший інтерфейс для доступу до
    існуючого об'єкта. З іншого боку, з [Декоратором](decorator.html)
    інтерфейс або залишається тим самим, або розширюється. Крім того
    *Декоратор* підтримує рекурсивну вкладуваність, на відміну від
    *Адаптеру*.

-   З [Адаптером](adapter.html) ви отримуєте доступ до існуючого об'єкта
    через інший інтерфейс. Використовуючи [Замісник](proxy.html),
    інтерфейс залишається незмінним. Використовуючи
    [Декоратор](decorator.html), ви отримуєте доступ до об'єкта через
    розширений інтерфейс.

-   [Фасад](facade.html) задає новий інтерфейс, тоді як
    [Адаптер](adapter.html) повторно використовує старий. *Адаптер*
    обгортає тільки один клас, а *Фасад* обгортає цілу підсистему. Крім
    того, *Адаптер* дозволяє двом існуючим інтерфейсам працювати
    спільно, замість того, щоб визначити повністю новий.

-   [Міст](bridge.html), [Стратегія](strategy.html) та
    [Стан](state.html) (а також трохи і [Адаптер](adapter.html)) мають
    схожі структури класів --- усі вони побудовані за принципом
    «композиції», тобто делегування роботи іншим об'єктам. Проте вони
    відрізняються тим, що вирішують різні проблеми. Пам'ятайте, що
    патерни --- це не тільки рецепт побудови коду певним чином, але й
    описування проблем, які призвели до такого рішення.
:::

::: {.section .implementations}
##  Приклади реалізації патерна {#implementations}

[![Адаптер на
C#](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](adapter/csharp/example.html "Адаптер на C#"){.prog-lang-link}
[![Адаптер на
C++](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](adapter/cpp/example.html "Адаптер на C++"){.prog-lang-link}
[![Адаптер на
Go](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](adapter/go/example.html "Адаптер на Go"){.prog-lang-link}
[![Адаптер на
Java](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](adapter/java/example.html "Адаптер на Java"){.prog-lang-link}
[![Адаптер на
PHP](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](adapter/php/example.html "Адаптер на PHP"){.prog-lang-link}
[![Адаптер на
Python](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](adapter/python/example.html "Адаптер на Python"){.prog-lang-link}
[![Адаптер на
Ruby](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](adapter/ruby/example.html "Адаптер на Ruby"){.prog-lang-link}
[![Адаптер на
Rust](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](adapter/rust/example.html "Адаптер на Rust"){.prog-lang-link}
[![Адаптер на
Swift](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](adapter/swift/example.html "Адаптер на Swift"){.prog-lang-link}
[![Адаптер на
TypeScript](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](adapter/typescript/example.html "Адаптер на TypeScript"){.prog-lang-link}
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

[Міст []{.fa .fa-arrow-right}](bridge.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Повернутись назад

[[]{.fa .fa-arrow-left} Структурні
патерни](structural-patterns.html){.btn .btn-default rel="prev"}
:::
:::::::::::::::::::::::::::::::::::

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
:::::::::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::::::::
