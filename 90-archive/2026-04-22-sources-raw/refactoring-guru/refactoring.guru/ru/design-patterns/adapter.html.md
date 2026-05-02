:::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [Паттерны
проектирования](../design-patterns.html){.type} / [Структурные
паттерны](structural-patterns.html){.type}
:::

# Адаптер {#адаптер .title}

::: aka
Также известен как:
[Wrapper, ]{style="display:inline-block"}[Обёртка, ]{style="display:inline-block"}[Adapter]{style="display:inline-block"}
:::

::: {.section .intent}
##  Суть паттерна {#intent}

**Адаптер** --- это структурный паттерн проектирования, который
позволяет объектам с несовместимыми интерфейсами работать вместе.

<figure class="image">
<img
src="../../images/patterns/content/adapter/adapter-en6d0c.png?id=11ef6ae6177291834323e3f918c47cd2"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/adapter/adapter-en.png?id=11ef6ae6177291834323e3f918c47cd2 640w,/images/patterns/content/adapter/adapter-en-1.5x.png?id=ad30c3ed32ca2da3eb6b8d9116506764 960w,/images/patterns/content/adapter/adapter-en-2x.png?id=e0ab0f6103b0b7b0648a8fda592ffab8 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Паттерн Адаптер" />
</figure>
:::

::: {.section .problem}
##  Проблема {#problem}

Представьте, что вы делаете приложение для торговли на бирже. Ваше
приложение скачивает биржевые котировки из нескольких источников в XML,
а затем рисует красивые графики.

В какой-то момент вы решаете улучшить приложение, применив стороннюю
библиотеку аналитики. Но вот беда --- библиотека поддерживает только
формат данных JSON, несовместимый с вашим приложением.

<figure class="image">
<img
src="../../images/patterns/diagrams/adapter/problem-en73b1.png?id=60d01f6c72ba85030cd52d5955caa3d8"
style="aspect-ratio: 2.41;"
srcset="/images/patterns/diagrams/adapter/problem-en.png?id=60d01f6c72ba85030cd52d5955caa3d8 530w,/images/patterns/diagrams/adapter/problem-en-1.5x.png?id=9d1ebfe5843bea44ad4025d4c648b34a 795w,/images/patterns/diagrams/adapter/problem-en-2x.png?id=f6f4bfbd2d6136a5ae4fb8c899e9854e 1060w"
sizes="(max-width: 720px) 100vw, 530px" loading="lazy" width="530"
alt="Структура программы до подключения сторонней библиотеки" />
<figcaption><p>Подключить стороннюю библиотеку не выйдет из-за
несовместимых форматов данных.</p></figcaption>
</figure>

Вы смогли бы переписать библиотеку, чтобы та поддерживала формат XML.
Но, во-первых, это может нарушить работу существующего кода, который уже
зависит от библиотеки. А во-вторых, у вас может просто не быть доступа к
её исходному коду.
:::

::: {.section .solution}
##  Решение {#solution}

Вы можете создать *адаптер*. Это объект-переводчик, который
трансформирует интерфейс или данные одного объекта в такой вид, чтобы он
стал понятен другому объекту.

При этом адаптер оборачивает один из объектов, так что другой объект
даже не знает о наличии первого. Например, вы можете обернуть объект,
работающий в метрах, адаптером, который бы конвертировал данные в футы.

Адаптеры могут не только переводить данные из одного формата в другой,
но и помогать объектам с разными интерфейсами работать сообща. Это
работает так:

1.  Адаптер имеет интерфейс, который совместим с одним из объектов.
2.  Поэтому этот объект может свободно вызывать методы адаптера.
3.  Адаптер получает эти вызовы и перенаправляет их второму объекту, но
    уже в том формате и последовательности, которые понятны второму
    объекту.

Иногда возможно создать даже *двухсторонний адаптер*, который работал бы
в обе стороны.

<figure class="image">
<img
src="../../images/patterns/diagrams/adapter/solution-en716c.png?id=5f4f1b4575236a3853f274b690bd6656"
style="aspect-ratio: 1.51;"
srcset="/images/patterns/diagrams/adapter/solution-en.png?id=5f4f1b4575236a3853f274b690bd6656 530w,/images/patterns/diagrams/adapter/solution-en-1.5x.png?id=40da8d488b9776aca5011c6ca96c5da4 795w,/images/patterns/diagrams/adapter/solution-en-2x.png?id=5846ed9b88cad0220993f79bdfe817a8 1060w"
sizes="(max-width: 720px) 100vw, 530px" loading="lazy" width="530"
alt="Структура программы после применения адаптера" />
<figcaption><p>Программа может работать со сторонней библиотекой
через адаптер.</p></figcaption>
</figure>

Таким образом, в приложении биржевых котировок вы могли бы создать класс
`XML_To_JSON_Adapter`, который бы оборачивал объект того или иного
класса библиотеки аналитики. Ваш код посылал бы адаптеру запросы в
формате XML, а адаптер сначала транслировал входящие данные в формат
JSON, а затем передавал бы их методам обёрнутого объекта аналитики.
:::

::: {.section .analogy}
##  Аналогия из жизни {#analogy}

<figure class="image">
<img
src="../../images/patterns/content/adapter/adapter-comic-1-ru7324.png?id=85783cd786e264ded1022c1053872ceb"
style="aspect-ratio: 1.78;"
srcset="/images/patterns/content/adapter/adapter-comic-1-ru.png?id=85783cd786e264ded1022c1053872ceb 533w,/images/patterns/content/adapter/adapter-comic-1-ru-1.5x.png?id=d528299d569a79bc62c2497bb14b7e2a 800w,/images/patterns/content/adapter/adapter-comic-1-ru-2x.png?id=45ae8a27d870cd430125c14f7446277d 1067w"
sizes="(max-width: 720px) 100vw, 533px" loading="lazy" width="533"
alt="Пример паттерна Адаптер" />
<figcaption><p>Содержимое чемоданов до и после поездки
за границу.</p></figcaption>
</figure>

Когда вы в первый раз летите за границу, вас может ждать сюрприз при
попытке зарядить ноутбук. Стандарты розеток в разных странах отличаются.
Ваша европейская зарядка будет бесполезна в США без специального
адаптера, позволяющего подключиться к розетке другого типа.
:::

:::::::: {.section .structure-container}
##  Структура {#structure}

::::::: structure
#### Адаптер объектов

Эта реализация использует агрегацию: объект адаптера «оборачивает», то
есть содержит ссылку на служебный объект. Такой подход работает во всех
языках программирования.

:::: adapter-object
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/adapter/structure-object-adaptera83b.png?id=33dffbe3aece294162440c7ddd3d5d4f"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.81;"
srcset="/images/patterns/diagrams/adapter/structure-object-adapter.png?id=33dffbe3aece294162440c7ddd3d5d4f 580w,/images/patterns/diagrams/adapter/structure-object-adapter-1.5x.png?id=c1b8a87b74fc4ce5639212fe19ee06fe 870w,/images/patterns/diagrams/adapter/structure-object-adapter-2x.png?id=03e8052e168c962d6bc369bbb13b0945 1160w"
sizes="(max-width: 720px) 100vw, 580px" loading="lazy" width="580"
alt="Структура классов паттерна Адаптер (адаптер объектов)" /><img
src="../../images/patterns/diagrams/adapter/structure-object-adapter-indexedd6ac.png?id=a20b311948b361a058097e5bcdbf067a"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.76;"
srcset="/images/patterns/diagrams/adapter/structure-object-adapter-indexed.png?id=a20b311948b361a058097e5bcdbf067a 600w,/images/patterns/diagrams/adapter/structure-object-adapter-indexed-1.5x.png?id=72a1c8fde064c4915379fb34a1a66881 900w,/images/patterns/diagrams/adapter/structure-object-adapter-indexed-2x.png?id=759771515f08d74d53cf4fe500f814a3 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Структура классов паттерна Адаптер (адаптер объектов)" />
</figure>
:::
::::

1.  **Клиент** --- это класс, который содержит существующую
    бизнес-логику программы.

2.  **Клиентский интерфейс** описывает протокол, через который клиент
    может работать с другими классами.

3.  **Сервис** --- это какой-то полезный класс, обычно сторонний. Клиент
    не может использовать этот класс напрямую, так как сервис имеет
    непонятный ему интерфейс.

4.  **Адаптер** --- это класс, который может одновременно работать и с
    клиентом, и с сервисом. Он реализует клиентский интерфейс и содержит
    ссылку на объект сервиса. Адаптер получает вызовы от клиента через
    методы клиентского интерфейса, а затем переводит их в вызовы методов
    обёрнутого объекта в правильном формате.

5.  Работая с адаптером через интерфейс, клиент не привязывается к
    конкретному классу адаптера. Благодаря этому, вы можете добавлять в
    программу новые виды адаптеров, независимо от клиентского кода. Это
    может пригодиться, если интерфейс сервиса вдруг изменится, например,
    после выхода новой версии сторонней библиотеки.

#### Адаптер классов

Эта реализация базируется на наследовании: адаптер наследует оба
интерфейса одновременно. Такой подход возможен только в языках,
поддерживающих множественное наследование, например, C++.

:::: adapter-class
::: {.struct-image2 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/adapter/structure-class-adapterf043.png?id=e1c60240508146ed3b98ac562cc8e510"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.72;"
srcset="/images/patterns/diagrams/adapter/structure-class-adapter.png?id=e1c60240508146ed3b98ac562cc8e510 550w,/images/patterns/diagrams/adapter/structure-class-adapter-1.5x.png?id=299a79bdfa10ac53213ba02408255430 825w,/images/patterns/diagrams/adapter/structure-class-adapter-2x.png?id=ddca3e3e4d972b7c58207daba8d24866 1100w"
sizes="(max-width: 720px) 100vw, 550px" loading="lazy" width="550"
alt="Структура классов паттерна Адаптер (адаптер классов)" /><img
src="../../images/patterns/diagrams/adapter/structure-class-adapter-indexedd30e.png?id=250b5c485a7dfba7c16b89a9201538fb"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.72;"
srcset="/images/patterns/diagrams/adapter/structure-class-adapter-indexed.png?id=250b5c485a7dfba7c16b89a9201538fb 550w,/images/patterns/diagrams/adapter/structure-class-adapter-indexed-1.5x.png?id=b56d736f8076d34b1896de0a2b22a219 825w,/images/patterns/diagrams/adapter/structure-class-adapter-indexed-2x.png?id=9ae1182ef2a34d2ea65f4b4f94a4019e 1100w"
sizes="(max-width: 720px) 100vw, 550px" loading="lazy" width="550"
alt="Структура классов паттерна Адаптер (адаптер классов)" />
</figure>
:::
::::

1.  **Адаптер классов** не нуждается во вложенном объекте, так как он
    может одновременно наследовать и часть существующего класса, и часть
    сервиса.
:::::::
::::::::

::: {.section .pseudocode}
##  Псевдокод {#pseudocode}

В этом шуточном примере **Адаптер** преобразует один интерфейс в другой,
позволяя совместить квадратные колышки и круглые отверстия.

<figure class="image">
<img
src="../../images/patterns/diagrams/adapter/examplef4c9.png?id=9d2b6857ce256f2c669383ce4df3d0aa"
style="aspect-ratio: 1.68;"
srcset="/images/patterns/diagrams/adapter/example.png?id=9d2b6857ce256f2c669383ce4df3d0aa 640w,/images/patterns/diagrams/adapter/example-1.5x.png?id=76e68b9cea3b371e465e81587e57e5d9 960w,/images/patterns/diagrams/adapter/example-2x.png?id=0ac62d1bc151e8ce6abad8e8502756cf 1280w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="640"
alt="Структура классов примера паттерна Адаптер" />
<figcaption><p>Пример адаптации квадратных колышков и
круглых отверстий.</p></figcaption>
</figure>

Адаптер вычисляет наименьший радиус окружности, в которую можно вписать
квадратный колышек, и представляет его как круглый колышек с этим
радиусом.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// Классы с совместимыми интерфейсами: КруглоеОтверстие и
// КруглыйКолышек.
class RoundHole is
    constructor RoundHole(radius) { ... }

    method getRadius() is
        // Вернуть радиус отверстия.

    method fits(peg: RoundPeg) is
        return this.getRadius() &gt;= peg.getRadius()

class RoundPeg is
    constructor RoundPeg(radius) { ... }

    method getRadius() is
        // Вернуть радиус круглого колышка.


// Устаревший, несовместимый класс: КвадратныйКолышек.
class SquarePeg is
    constructor SquarePeg(width) { ... }

    method getWidth() is
        // Вернуть ширину квадратного колышка.


// Адаптер позволяет использовать квадратные колышки и круглые
// отверстия вместе.
class SquarePegAdapter extends RoundPeg is
    private field peg: SquarePeg

    constructor SquarePegAdapter(peg: SquarePeg) is
        this.peg = peg

    method getRadius() is
        // Вычислить половину диагонали квадратного колышка по
        // теореме Пифагора.
        return peg.getWidth() * Math.sqrt(2) / 2


// Где-то в клиентском коде.
hole = new RoundHole(5)
rpeg = new RoundPeg(5)
hole.fits(rpeg) // TRUE

small_sqpeg = new SquarePeg(5)
large_sqpeg = new SquarePeg(10)
hole.fits(small_sqpeg) // Ошибка компиляции, несовместимые типы

small_sqpeg_adapter = new SquarePegAdapter(small_sqpeg)
large_sqpeg_adapter = new SquarePegAdapter(large_sqpeg)
hole.fits(small_sqpeg_adapter) // TRUE
hole.fits(large_sqpeg_adapter) // FALSE</code></pre>
</figure>
:::

:::::::: {.section .applicability-container}
##  Применимость {#applicability}

::::::: applicability
::: applicability-problem
Когда вы хотите использовать сторонний класс, но его интерфейс не
соответствует остальному коду приложения.
:::

::: applicability-solution
Адаптер позволяет создать объект-прокладку, который будет превращать
вызовы приложения в формат, понятный стороннему классу.
:::

::: applicability-problem
Когда вам нужно использовать несколько существующих подклассов, но в них
не хватает какой-то общей функциональности, причём расширить суперкласс
вы не можете.
:::

::: applicability-solution
Вы могли бы создать ещё один уровень подклассов и добавить в них
недостающую функциональность. Но при этом придётся дублировать один и
тот же код в обеих ветках подклассов.

Более элегантным решением было бы поместить недостающую функциональность
в адаптер и приспособить его для работы с суперклассом. Такой адаптер
сможет работать со всеми подклассами иерархии. Это решение будет сильно
напоминать паттерн [Декоратор](decorator.html).
:::
:::::::
::::::::

::: {.section .checklist}
##  Шаги реализации {#checklist}

1.  Убедитесь, что у вас есть два класса с несовместимыми интерфейсами:

    -   полезный *сервис* --- служебный класс, который вы не можете
        изменять (он либо сторонний, либо от него зависит другой код);
    -   один или несколько *клиентов* --- существующих классов
        приложения, несовместимых с сервисом из-за неудобного или
        несовпадающего интерфейса.

2.  Опишите клиентский интерфейс, через который классы приложения смогли
    бы использовать класс сервиса.

3.  Создайте класс адаптера, реализовав этот интерфейс.

4.  Поместите в адаптер поле, которое будет хранить ссылку на объект
    сервиса. Обычно это поле заполняют объектом, переданным в
    конструктор адаптера. В случае простой адаптации этот объект можно
    передавать через параметры методов адаптера.

5.  Реализуйте все методы клиентского интерфейса в адаптере. Адаптер
    должен делегировать основную работу сервису.

6.  Приложение должно использовать адаптер только через клиентский
    интерфейс. Это позволит легко изменять и добавлять адаптеры в
    будущем.
:::

:::::: {.section .pros-cons}
##  Преимущества и недостатки {#pros-cons}

::::: row
::: col-sm-6
-    Отделяет и скрывает от клиента подробности преобразования различных
    интерфейсов.
:::

::: col-sm-6
-    Усложняет код программы из-за введения дополнительных классов.
:::
:::::
::::::

::: {.section .relations}
##  Отношения с другими паттернами {#relations}

-   [Мост](bridge.html) проектируют загодя, чтобы развивать большие
    части приложения отдельно друг от друга. [Адаптер](adapter.html)
    применяется постфактум, чтобы заставить несовместимые классы
    работать вместе.

-   [Адаптер](adapter.html) предоставляет совершенно другой интерфейс
    для доступа к существующему объекту. С другой стороны, при
    использовании паттерна [Декоратор](decorator.html) интерфейс либо
    остается прежним, либо расширяется. Причём *Декоратор* поддерживает
    рекурсивную вложенность, чего не скажешь об *Адаптере*.

-   С [Адаптером](adapter.html) вы получаете доступ к существующему
    объекту через другой интерфейс. Используя [Заместитель](proxy.html),
    интерфейс остается неизменным. Используя
    [Декоратор](decorator.html), вы получаете доступ к объекту через
    расширенный интерфейс.

-   [Фасад](facade.html) задаёт новый интерфейс, тогда как
    [Адаптер](adapter.html) повторно использует старый. *Адаптер*
    оборачивает только один класс, а *Фасад* оборачивает целую
    подсистему. Кроме того, *Адаптер* позволяет двум существующим
    интерфейсам работать сообща, вместо того, чтобы задать полностью
    новый.

-   [Мост](bridge.html), [Стратегия](strategy.html) и
    [Состояние](state.html) (а также слегка и [Адаптер](adapter.html))
    имеют схожие структуры классов --- все они построены на принципе
    «композиции», то есть делегирования работы другим объектам. Тем не
    менее, они отличаются тем, что решают разные проблемы. Помните, что
    паттерны --- это не только рецепт построения кода определённым
    образом, но и описание проблем, которые привели к данному решению.
:::

::: {.section .implementations}
##  Примеры реализации паттерна {#implementations}

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

[Мост []{.fa .fa-arrow-right}](bridge.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Структурные
паттерны](structural-patterns.html){.btn .btn-default rel="prev"}
:::
:::::::::::::::::::::::::::::::::::

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
:::::::::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::::::::
