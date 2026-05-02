::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [Паттерны
проектирования](../design-patterns.html){.type} / [Структурные
паттерны](structural-patterns.html){.type}
:::

# Фасад {#фасад .title}

::: aka
Также известен как: [Facade]{style="display:inline-block"}
:::

::: {.section .intent}
##  Суть паттерна {#intent}

**Фасад** --- это структурный паттерн проектирования, который
предоставляет простой интерфейс к сложной системе классов, библиотеке
или фреймворку.

<figure class="image">
<img
src="../../images/patterns/content/facade/facade721d.png?id=1f4be17305b6316fbd548edf1937ac3b"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/facade/facade.png?id=1f4be17305b6316fbd548edf1937ac3b 640w,/images/patterns/content/facade/facade-1.5x.png?id=a6af5003b243b59990842d24bdaae2df 960w,/images/patterns/content/facade/facade-2x.png?id=b69fce5943703f5f07c0ba38e3baaed0 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Паттерн Фасад" />
</figure>
:::

::: {.section .problem}
##  Проблема {#problem}

Вашему коду приходится работать с большим количеством объектов некой
сложной библиотеки или фреймворка. Вы должны самостоятельно
инициализировать эти объекты, следить за правильным порядком
зависимостей и так далее.

В результате бизнес-логика ваших классов тесно переплетается с деталями
реализации сторонних классов. Такой код довольно сложно понимать и
поддерживать.
:::

::: {.section .solution}
##  Решение {#solution}

Фасад --- это простой интерфейс для работы со сложной подсистемой,
содержащей множество классов. Фасад может иметь урезанный интерфейс, не
имеющий 100% функциональности, которой можно достичь, используя сложную
подсистему напрямую. Но он предоставляет именно те фичи, которые нужны
клиенту, и скрывает все остальные.

Фасад полезен, если вы используете какую-то сложную библиотеку со
множеством подвижных частей, но вам нужна только часть её возможностей.

К примеру, программа, заливающая видео котиков в социальные сети, может
использовать профессиональную библиотеку сжатия видео. Но все, что нужно
клиентскому коду этой программы --- простой метод
`encode(filename, format)`. Создав класс с таким методом, вы реализуете
свой первый фасад.
:::

::: {.section .analogy}
##  Аналогия из жизни {#analogy}

<figure class="image">
<img
src="../../images/patterns/diagrams/facade/live-example-rud41c.png?id=4bc5ff5a2d0004ca33c488d5a606d0ca"
style="aspect-ratio: 2.58;"
srcset="/images/patterns/diagrams/facade/live-example-ru.png?id=4bc5ff5a2d0004ca33c488d5a606d0ca 490w,/images/patterns/diagrams/facade/live-example-ru-1.5x.png?id=9cb7949135e5c11e5657d80ff85ea615 735w,/images/patterns/diagrams/facade/live-example-ru-2x.png?id=cdeb83ae732a2a09bf33005bcae09c1e 980w"
sizes="(max-width: 720px) 100vw, 490px" loading="lazy" width="490"
alt="Пример телефонного заказа" />
<figcaption><p>Пример телефонного заказа.</p></figcaption>
</figure>

Когда вы звоните в магазин и делаете заказ по телефону, сотрудник службы
поддержки является вашим фасадом ко всем службам и отделам магазина. Он
предоставляет вам упрощённый интерфейс к системе создания заказа,
платёжной системе и отделу доставки.
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
alt="Структура классов паттерна Фасад" /><img
src="../../images/patterns/diagrams/facade/structure-indexeddb1c.png?id=2da06d6b850701ea15cf72f9d2642fb8"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.58;"
srcset="/images/patterns/diagrams/facade/structure-indexed.png?id=2da06d6b850701ea15cf72f9d2642fb8 600w,/images/patterns/diagrams/facade/structure-indexed-1.5x.png?id=fad7d667f4d8d9a7d522748bcd37dfde 900w,/images/patterns/diagrams/facade/structure-indexed-2x.png?id=4d181bcf1df5d58c533e6c92461e9571 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Структура классов паттерна Фасад" />
</figure>
:::

1.  **Фасад** предоставляет быстрый доступ к определённой
    функциональности подсистемы. Он «знает», каким классам нужно
    переадресовать запрос, и какие данные для этого нужны.

2.  **Дополнительный фасад** можно ввести, чтобы не «захламлять»
    единственный фасад разнородной функциональностью. Он может
    использоваться как клиентом, так и другими фасадами.

3.  **Сложная подсистема** состоит из множества разнообразных классов.
    Для того, чтобы заставить их что-то делать, нужно знать подробности
    устройства подсистемы, порядок инициализации объектов и так далее.

    Классы подсистемы не знают о существовании фасада и работают друг с
    другом напрямую.

4.  **Клиент** использует фасад вместо прямой работы с объектами сложной
    подсистемы.
::::
:::::

::: {.section .pseudocode}
##  Псевдокод {#pseudocode}

В этом примере **Фасад** упрощает работу со сложным фреймворком
видеоконвертации.

<figure class="image">
<img
src="../../images/patterns/diagrams/facade/example16bc.png?id=2249d134e3ff83819dfc19032f02eced"
style="aspect-ratio: 1.5;"
srcset="/images/patterns/diagrams/facade/example.png?id=2249d134e3ff83819dfc19032f02eced 570w,/images/patterns/diagrams/facade/example-1.5x.png?id=034ecbe0f3cc108f4ae49d05d1c77dbe 855w,/images/patterns/diagrams/facade/example-2x.png?id=f2c846d74041626c923ff3e8919b68a9 1140w"
sizes="(max-width: 720px) 100vw, 570px" loading="lazy" width="570"
alt="Структура классов примера паттерна Фасад" />
<figcaption><p>Пример изоляции множества зависимостей в
одном фасаде.</p></figcaption>
</figure>

Вместо непосредственной работы с дюжиной классов, фасад предоставляет
коду приложения единственный метод для конвертации видео, который сам
заботится о том, чтобы правильно сконфигурировать нужные объекты
фреймворка и получить требуемый результат.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// Классы сложного стороннего фреймворка конвертации видео. Мы
// не контролируем этот код, поэтому не можем его упростить.

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


// Вместо этого мы создаём Фасад — простой интерфейс для работы
// со сложным фреймворком. Фасад не имеет всей функциональности
// фреймворка, но зато скрывает его сложность от клиентов.
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

// Приложение не зависит от сложного фреймворка конвертации
// видео. Кстати, если вы вдруг решите сменить фреймворк, вам
// нужно будет переписать только класс фасада.
class Application is
    method main() is
        convertor = new VideoConverter()
        mp4 = convertor.convert(&quot;funny-cats-video.ogg&quot;, &quot;mp4&quot;)
        mp4.save()</code></pre>
</figure>
:::

:::::::: {.section .applicability-container}
##  Применимость {#applicability}

::::::: applicability
::: applicability-problem
Когда вам нужно представить простой или урезанный интерфейс к сложной
подсистеме.
:::

::: applicability-solution
Часто подсистемы усложняются по мере развития программы. Применение
большинства паттернов приводит к появлению меньших классов, но в бóльшем
количестве. Такую подсистему проще повторно использовать, настраивая её
каждый раз под конкретные нужды, но вместе с тем, применять подсистему
без настройки становится труднее. Фасад предлагает определённый вид
системы по умолчанию, устраивающий большинство клиентов.
:::

::: applicability-problem
Когда вы хотите разложить подсистему на отдельные слои.
:::

::: applicability-solution
Используйте фасады для определения точек входа на каждый уровень
подсистемы. Если подсистемы зависят друг от друга, то зависимость можно
упростить, разрешив подсистемам обмениваться информацией только через
фасады.

Например, возьмём ту же сложную систему видеоконвертации. Вы хотите
разбить её на слои работы с аудио и видео. Для каждой из этих частей
можно попытаться создать фасад и заставить классы аудио и видео
обработки общаться друг с другом через эти фасады, а не напрямую.
:::
:::::::
::::::::

::: {.section .checklist}
##  Шаги реализации {#checklist}

1.  Определите, можно ли создать более простой интерфейс, чем тот,
    который предоставляет сложная подсистема. Вы на правильном пути,
    если этот интерфейс избавит клиента от необходимости знать о
    подробностях подсистемы.

2.  Создайте класс фасада, реализующий этот интерфейс. Он должен
    переадресовывать вызовы клиента нужным объектам подсистемы. Фасад
    должен будет позаботиться о том, чтобы правильно инициализировать
    объекты подсистемы.

3.  Вы получите максимум пользы, если клиент будет работать только с
    фасадом. В этом случае изменения в подсистеме будут затрагивать
    только код фасада, а клиентский код останется рабочим.

4.  Если ответственность фасада [начинает
    размываться](../smells/large-class.html), подумайте о введении
    дополнительных фасадов.
:::

:::::: {.section .pros-cons}
##  Преимущества и недостатки {#pros-cons}

::::: row
::: col-sm-6
-    Изолирует клиентов от компонентов сложной подсистемы.
:::

::: col-sm-6
-    Фасад рискует стать [божественным
    объектом](https://ru.wikipedia.org/wiki/%D0%91%D0%BE%D0%B6%D0%B5%D1%81%D1%82%D0%B2%D0%B5%D0%BD%D0%BD%D1%8B%D0%B9_%D0%BE%D0%B1%D1%8A%D0%B5%D0%BA%D1%82),
    привязанным ко всем классам программы.
:::
:::::
::::::

::: {.section .relations}
##  Отношения с другими паттернами {#relations}

-   [Фасад](facade.html) задаёт новый интерфейс, тогда как
    [Адаптер](adapter.html) повторно использует старый. *Адаптер*
    оборачивает только один класс, а *Фасад* оборачивает целую
    подсистему. Кроме того, *Адаптер* позволяет двум существующим
    интерфейсам работать сообща, вместо того, чтобы задать полностью
    новый.

-   [Абстрактная фабрика](abstract-factory.html) может быть использована
    вместо [Фасада](facade.html) для того, чтобы скрыть
    платформо-зависимые классы.

-   [Легковес](flyweight.html) показывает, как создавать много мелких
    объектов, а [Фасад](facade.html) показывает, как создать один
    объект, который отображает целую подсистему.

-   [Посредник](mediator.html) и [Фасад](facade.html) похожи тем, что
    пытаются организовать работу множества существующих классов.

    -   *Фасад* создаёт упрощённый интерфейс к подсистеме, не внося в
        неё никакой добавочной функциональности. Сама подсистема не
        знает о существовании *Фасада*. Классы подсистемы общаются друг
        с другом напрямую.
    -   *Посредник* централизует общение между компонентами системы.
        Компоненты системы знают только о существовании *Посредника*, у
        них нет прямого доступа к другим компонентам.

-   [Фасад](facade.html) можно сделать [Одиночкой](singleton.html), так
    как обычно нужен только один объект-фасад.

-   [Фасад](facade.html) похож на [Заместитель](proxy.html) тем, что
    замещает сложную подсистему и может сам её инициализировать. Но в
    отличие от *Фасада*, *Заместитель* имеет тот же интерфейс, что его
    служебный объект, благодаря чему их можно взаимозаменять.
:::

::: {.section .implementations}
##  Примеры реализации паттерна {#implementations}

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
::::: {.prom .banner-content .banner-bg .banner-striped .banner-with-image data-id="DP: 1: Ne vtykay" creative-id="standard-ru" position="content_bottom"}
::: banner-image
[![](../../images/patterns/banners/patterns-book-banner-1-b158a.png?id=0877cab2f0102d98cd07b50af0e5beea){width="200"
height="200" loading="lazy"
srcset="/images/patterns/banners/patterns-book-banner-1-b-2x.png?id=5572aa55e5b09e59780aca9e0ea8e44b 2x"}](book.html)
:::

::: banner-text
### Не втыкай в транспорте {#не-втыкай-в-транспорте .title}

Лучше почитай нашу книгу о паттернах проектирования.

Теперь это удобно делать даже во время поездок в общественном
транспорте.

[ Узнать больше...](book.html){.btn .btn-secondary}
:::
:::::
::::::

::: next
#### Читаем дальше

[Легковес []{.fa .fa-arrow-right}](flyweight.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Декоратор](decorator.html){.btn .btn-default
rel="prev"}
:::
::::::::::::::::::::::::::::::::

::::::: {.prom .banner-sidebar .banner-removable .banner-removable-patterns data-id="DP: Sidebar" creative-id="standard-sidebar-ru" position="sidebar"}
:::::: banner-inner
:::: image3d-book-right
::: {.image3d-cover style="background: #0b3752;"}
[![](../../images/patterns/book/web-cover-ruee5e.png?id=ea25feb28a6deb8aa4af6a633904a568){width="250"
height="375" loading="lazy"
srcset="/images/patterns/book/web-cover-ru-2x.png?id=95101b307f6dba409f28a417746ff3c1 2x"}](book.html)
:::
::::

::: {style="margin-top: 1rem"}
Эта статья является частью нашей электронной книги **Погружение в
Паттерны Проектирования**.

[ Узнать больше...](book.html){.btn .btn-secondary .btn-block}
:::
::::::
:::::::
::::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::::
