::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [Патерни
проектування](../design-patterns.html){.type} / [Структурні
патерни](structural-patterns.html){.type}
:::

# Фасад {#фасад .title}

::: aka
Також відомий як: [Facade]{style="display:inline-block"}
:::

::: {.section .intent}
##  Суть патерна {#intent}

**Фасад** --- це структурний патерн проектування, який надає простий
інтерфейс до складної системи класів, бібліотеки або фреймворку.

<figure class="image">
<img
src="../../images/patterns/content/facade/facade721d.png?id=1f4be17305b6316fbd548edf1937ac3b"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/facade/facade.png?id=1f4be17305b6316fbd548edf1937ac3b 640w,/images/patterns/content/facade/facade-1.5x.png?id=a6af5003b243b59990842d24bdaae2df 960w,/images/patterns/content/facade/facade-2x.png?id=b69fce5943703f5f07c0ba38e3baaed0 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Патерн Фасад" />
</figure>
:::

::: {.section .problem}
##  Проблема {#problem}

Вашому коду доводиться працювати з великою кількістю об'єктів певної
складної бібліотеки чи фреймворка. Ви повинні самостійно ініціалізувати
ці об'єкти, стежити за правильним порядком залежностей тощо.

В результаті бізнес-логіка ваших класів тісно переплітається з деталями
реалізації сторонніх класів. Такий код досить складно розуміти та
підтримувати.
:::

::: {.section .solution}
##  Рішення {#solution}

Фасад --- це простий інтерфейс для роботи зі складною підсистемою, яка
містить безліч класів. Фасад може бути спрощеним відображенням системи,
що не має 100% тієї функціональності, якої можна було б досягти,
використовуючи складну підсистему безпосередньо. Разом з тим, він надає
саме ті «фічі», які потрібні клієнтові, і приховує все інше.

Фасад корисний у тому випадку, якщо ви використовуєте якусь складну
бібліотеку з безліччю рухомих частин, з яких вам потрібна тільки
частина.

Наприклад, програма, що заливає в соціальні мережі відео з кошенятками,
може використовувати професійну бібліотеку для стискання відео, але все,
що потрібно клієнтському коду цієї програми, --- це простий метод
`encode(filename, format)`. Створивши клас з таким методом, ви
реалізуєте свій перший фасад.
:::

::: {.section .analogy}
##  Аналогія з життя {#analogy}

<figure class="image">
<img
src="../../images/patterns/diagrams/facade/live-example-uk03f6.png?id=4a3ddecfb924b1d5cc0b28db9df808a2"
style="aspect-ratio: 2.58;"
srcset="/images/patterns/diagrams/facade/live-example-uk.png?id=4a3ddecfb924b1d5cc0b28db9df808a2 490w,/images/patterns/diagrams/facade/live-example-uk-1.5x.png?id=925977a329d2d7f30c23a7de8d87845f 735w,/images/patterns/diagrams/facade/live-example-uk-2x.png?id=411d9913013a69b6995bfcfe70bd1844 980w"
sizes="(max-width: 720px) 100vw, 490px" loading="lazy" width="490"
alt="Приклад замовлення через телефон" />
<figcaption><p>Приклад замовлення через телефон.</p></figcaption>
</figure>

Коли ви телефонуєте до магазину і робите замовлення, співробітник служби
підтримки є вашим фасадом до всіх служб і відділів магазину. Він надає
вам спрощений інтерфейс до системи створення замовлення, платіжної
системи та відділу доставки.
:::

::::: {.section .structure-container}
##  Структура {#structure}

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/facade/structure2883.png?id=258401362234ac77a2aaf1cde62339e7"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.47;"
srcset="/images/patterns/diagrams/facade/structure.png?id=258401362234ac77a2aaf1cde62339e7 560w,/images/patterns/diagrams/facade/structure-1.5x.png?id=98d134de0c368d382909ba162ec21767 840w,/images/patterns/diagrams/facade/structure-2x.png?id=528ca429555bce293b7c3bd90954e097 1120w"
sizes="(max-width: 720px) 100vw, 560px" loading="lazy" width="560"
alt="Структура класів патерна Фасад" /><img
src="../../images/patterns/diagrams/facade/structure-indexeddb1c.png?id=2da06d6b850701ea15cf72f9d2642fb8"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.58;"
srcset="/images/patterns/diagrams/facade/structure-indexed.png?id=2da06d6b850701ea15cf72f9d2642fb8 600w,/images/patterns/diagrams/facade/structure-indexed-1.5x.png?id=fad7d667f4d8d9a7d522748bcd37dfde 900w,/images/patterns/diagrams/facade/structure-indexed-2x.png?id=4d181bcf1df5d58c533e6c92461e9571 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Структура класів патерна Фасад" />
</figure>
:::

1.  **Фасад** надає швидкий доступ до певної функціональності
    підсистеми. Він «знає», яким класам потрібно переадресувати запит, і
    які дані для цього потрібні.

2.  **Додатковий фасад** можна ввести, щоб не захаращувати єдиний фасад
    різнорідною функціональністю. Він може використовуватися як
    клієнтом, так й іншими фасадами.

3.  **Складна підсистема** має безліч різноманітних класів. Для того,
    щоб примусити усіх їх щось робити, потрібно знати подробиці
    влаштування підсистеми, порядок ініціалізації об'єктів та інші
    деталі.

    Класи підсистеми не знають про існування фасаду і працюють один з
    одним безпосередньо.

4.  **Клієнт** використовує фасад замість безпосередньої роботи з
    об'єктами складної підсистеми.
::::
:::::

::: {.section .pseudocode}
##  Псевдокод {#pseudocode}

У цьому прикладі **Фасад** спрощує роботу зі складним фреймворком
конвертації відео.

<figure class="image">
<img
src="../../images/patterns/diagrams/facade/example16bc.png?id=2249d134e3ff83819dfc19032f02eced"
style="aspect-ratio: 1.5;"
srcset="/images/patterns/diagrams/facade/example.png?id=2249d134e3ff83819dfc19032f02eced 570w,/images/patterns/diagrams/facade/example-1.5x.png?id=034ecbe0f3cc108f4ae49d05d1c77dbe 855w,/images/patterns/diagrams/facade/example-2x.png?id=f2c846d74041626c923ff3e8919b68a9 1140w"
sizes="(max-width: 720px) 100vw, 570px" loading="lazy" width="570"
alt="Структура класів прикладу патерна Фасад" />
<figcaption><p>Приклад ізоляції множини залежностей в
одному фасаді.</p></figcaption>
</figure>

Замість безпосередньої роботи з дюжиною класів, фасад надає коду
програми єдиний метод для конвертації відео, який сам подбає про те, щоб
правильно налаштувати потрібні об'єкти фреймворку і отримати необхідний
результат.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// Класи складного стороннього фреймворку конвертації відео. Ми
// не контролюємо цей код, тому не можемо його спростити.

class VideoFile
// ...

class OggCompressionCodec
// ...

class MPEG4CompressionCodec
// ...

class CodecFactory
// ...

class BitrateReader
// ...

class AudioMixer
// ...


// Замість цього, ми створюємо Фасад — простий інтерфейс для
// роботи зі складним фреймворком. Фасад не має всієї
// функціональності фреймворку, але приховує його складність від
// клієнтів.
class VideoConverter is
    method convert(filename, format):File is
        file = new VideoFile(filename)
        sourceCodec = (new CodecFactory).extract(file)
        if (format == &quot;mp4&quot;)
            destinationCodec = new MPEG4CompressionCodec()
        else
            destinationCodec = new OggCompressionCodec()
        buffer = BitrateReader.read(filename, sourceCodec)
        result = BitrateReader.convert(buffer, destinationCodec)
        result = (new AudioMixer()).fix(result)
        return new File(result)

// Програма не залежить від складного фреймворку конвертації
// відео. До речі, якщо ви раптом вирішите змінити фреймворк,
// вам потрібно буде переписати тільки клас фасаду.
class Application is
    method main() is
        convertor = new VideoConverter()
        mp4 = convertor.convert(&quot;funny-cats-video.ogg&quot;, &quot;mp4&quot;)
        mp4.save()</code></pre>
</figure>
:::

:::::::: {.section .applicability-container}
##  Застосування {#applicability}

::::::: applicability
::: applicability-problem
Якщо вам потрібно надати простий або урізаний інтерфейс до складної
підсистеми.
:::

::: applicability-solution
Часто підсистеми ускладнюються в міру розвитку програми. Застосування
більшості патернів призводить до появи менших класів, але у великій
кількості. Таку підсистему простіше використовувати повторно,
налаштовуючи її кожен раз під конкретні потреби, але, разом з тим,
використовувати таку підсистему без налаштовування важче. Фасад пропонує
певний вид системи за замовчуванням, який влаштовує більшість клієнтів.
:::

::: applicability-problem
Якщо ви хочете розкласти підсистему на окремі рівні.
:::

::: applicability-solution
Використовуйте фасади для визначення точок входу на кожен рівень
підсистеми. Якщо підсистеми залежать одна від одної, тоді залежність
можна спростити, дозволивши підсистемам обмінюватися інформацією тільки
через фасади.

Наприклад, візьмемо ту ж саму складну систему конвертації відео. Ви
хочете розбити її на окремі шари для роботи з аудіо й відео. Можна
спробувати створити фасад для кожної з цих частин і примусити класи
аудіо та відео обробки спілкуватися один з одним через ці фасади, а не
безпосередньо.
:::
:::::::
::::::::

::: {.section .checklist}
##  Кроки реалізації {#checklist}

1.  Визначте, чи можна створити більш простий інтерфейс, ніж той, який
    надає складна підсистема. Ви на правильному шляху, якщо цей
    інтерфейс позбавить клієнта від необхідності знати подробиці
    підсистеми.

2.  Створіть клас фасаду, що реалізує цей інтерфейс. Він повинен
    переадресовувати виклики клієнта потрібним об'єктам підсистеми.
    Фасад повинен буде подбати про те, щоб правильно ініціалізувати
    об'єкти підсистеми.

3.  Ви отримаєте максимум користі, якщо клієнт працюватиме тільки з
    фасадом. В такому випадку зміни в підсистемі стосуватимуться тільки
    коду фасаду, а клієнтський код залишиться робочим.

4.  Якщо відповідальність фасаду стає розмитою, подумайте про введення
    додаткових фасадів.
:::

:::::: {.section .pros-cons}
##  Переваги та недоліки {#pros-cons}

::::: row
::: col-sm-6
-    Ізолює клієнтів від компонентів складної підсистеми.
:::

::: col-sm-6
-    Фасад ризикує стати [божественим
    об'єктом](https://en.wikipedia.org/wiki/God_object), прив'язаним до
    всіх класів програми.
:::
:::::
::::::

::: {.section .relations}
##  Відносини з іншими патернами {#relations}

-   [Фасад](facade.html) задає новий інтерфейс, тоді як
    [Адаптер](adapter.html) повторно використовує старий. *Адаптер*
    обгортає тільки один клас, а *Фасад* обгортає цілу підсистему. Крім
    того, *Адаптер* дозволяє двом існуючим інтерфейсам працювати
    спільно, замість того, щоб визначити повністю новий.

-   [Абстрактна фабрика](abstract-factory.html) може бути використана
    замість [Фасаду](facade.html) для того, щоб приховати
    платформо-залежні класи.

-   [Легковаговик](flyweight.html) показує, як створювати багато дрібних
    об'єктів, а [Фасад](facade.html) показує, як створити один об'єкт,
    який відображає цілу підсистему.

-   [Посередник](mediator.html) та [Фасад](facade.html) схожі тим, що
    намагаються організувати роботу багатьох існуючих класів.

    -   *Фасад* створює спрощений інтерфейс підсистеми, не вносячи в неї
        жодної додаткової функціональності. Сама підсистема не знає про
        існування *Фасаду*. Класи підсистеми спілкуються один з одним
        безпосередньо.
    -   *Посередник* централізує спілкування між компонентами системи.
        Компоненти системи знають тільки про існування *Посередника*, у
        них немає прямого доступу до інших компонентів.

-   [Фасад](facade.html) можна зробити [Одинаком](singleton.html),
    оскільки зазвичай потрібен тільки один об'єкт-фасад.

-   [Фасад](facade.html) схожий на [Замісник](proxy.html) тим, що
    замінює складну підсистему та може сам її ініціалізувати. Але, на
    відміну від *Фасаду*, *Замісник* має такий самий інтерфейс, що і
    його службовий об'єкт, завдяки чому їх можна взаємозаміняти.
:::

::: {.section .implementations}
##  Приклади реалізації патерна {#implementations}

[![Фасад на
C#](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](facade/csharp/example.html "Фасад на C#"){.prog-lang-link}
[![Фасад на
C++](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](facade/cpp/example.html "Фасад на C++"){.prog-lang-link}
[![Фасад на
Go](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](facade/go/example.html "Фасад на Go"){.prog-lang-link}
[![Фасад на
Java](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](facade/java/example.html "Фасад на Java"){.prog-lang-link}
[![Фасад на
PHP](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](facade/php/example.html "Фасад на PHP"){.prog-lang-link}
[![Фасад на
Python](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](facade/python/example.html "Фасад на Python"){.prog-lang-link}
[![Фасад на
Ruby](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](facade/ruby/example.html "Фасад на Ruby"){.prog-lang-link}
[![Фасад на
Rust](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](facade/rust/example.html "Фасад на Rust"){.prog-lang-link}
[![Фасад на
Swift](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](facade/swift/example.html "Фасад на Swift"){.prog-lang-link}
[![Фасад на
TypeScript](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](facade/typescript/example.html "Фасад на TypeScript"){.prog-lang-link}
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

[Легковаговик []{.fa .fa-arrow-right}](flyweight.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Повернутись назад

[[]{.fa .fa-arrow-left} Декоратор](decorator.html){.btn .btn-default
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
