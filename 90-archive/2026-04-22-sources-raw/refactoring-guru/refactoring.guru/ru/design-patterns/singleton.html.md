::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [Паттерны
проектирования](../design-patterns.html){.type} / [Порождающие
паттерны](creational-patterns.html){.type}
:::

# Одиночка {#одиночка .title}

::: aka
Также известен как: [Singleton]{style="display:inline-block"}
:::

::: {.section .intent}
##  Суть паттерна {#intent}

**Одиночка** --- это порождающий паттерн проектирования, который
гарантирует, что у класса есть только один экземпляр, и предоставляет к
нему глобальную точку доступа.

<figure class="image">
<img
src="../../images/patterns/content/singleton/singleton4263.png?id=108a0b9b5ea5c4426e0afa4504491d6f"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/singleton/singleton.png?id=108a0b9b5ea5c4426e0afa4504491d6f 640w,/images/patterns/content/singleton/singleton-1.5x.png?id=29490ad6cd1426c63cf5f88243a1701c 960w,/images/patterns/content/singleton/singleton-2x.png?id=accb2cc7594f7a491ce01dddf0d2f876 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Паттерн Одиночка" />
</figure>
:::

::: {.section .problem}
##  Проблема {#problem}

Одиночка решает сразу две проблемы, нарушая *принцип единственной
ответственности* класса.

1.  **Гарантирует наличие единственного экземпляра класса**. Чаще всего
    это полезно для доступа к какому-то общему ресурсу, например, базе
    данных.

    Представьте, что вы создали объект, а через некоторое время пробуете
    создать ещё один. В этом случае хотелось бы получить старый объект,
    вместо создания нового.

    Такое поведение невозможно реализовать с помощью обычного
    конструктора, так как конструктор класса **всегда** возвращает новый
    объект.

<figure class="image">
<img
src="../../images/patterns/content/singleton/singleton-comic-1-ru3e0b.png?id=c3a9bfe5411ebd5c12205fa32dd7d675"
style="aspect-ratio: 2;"
srcset="/images/patterns/content/singleton/singleton-comic-1-ru.png?id=c3a9bfe5411ebd5c12205fa32dd7d675 600w,/images/patterns/content/singleton/singleton-comic-1-ru-1.5x.png?id=6a5123c46d05bbf8de9fd0ed2a08321c 900w,/images/patterns/content/singleton/singleton-comic-1-ru-2x.png?id=6de2f2412cbd0e6ab2fc1a3429315633 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Глобальный доступ к одному объекту" />
<figcaption><p>Клиенты могут не подозревать, что работают с одним и тем
же объектом.</p></figcaption>
</figure>

2.  **Предоставляет глобальную точку доступа**. Это не просто глобальная
    переменная, через которую можно достучаться к определённому объекту.
    Глобальные переменные не защищены от записи, поэтому любой код может
    подменять их значения без вашего ведома.

    Но есть и другой нюанс. Неплохо бы хранить в одном месте и код,
    который решает проблему №1, а также иметь к нему простой и доступный
    интерфейс.

Интересно, что в наше время паттерн стал настолько известен, что теперь
люди называют «одиночками» даже те классы, которые решают лишь одну из
проблем, перечисленных выше.
:::

::: {.section .solution}
##  Решение {#solution}

Все реализации одиночки сводятся к тому, чтобы скрыть конструктор по
умолчанию и создать публичный статический метод, который и будет
контролировать жизненный цикл объекта-одиночки.

Если у вас есть доступ к классу одиночки, значит, будет доступ и к этому
статическому методу. Из какой точки кода вы бы его ни вызвали, он всегда
будет отдавать один и тот же объект.
:::

::: {.section .analogy}
##  Аналогия из жизни {#analogy}

Правительство государства --- хороший пример одиночки. В государстве
может быть только одно официальное правительство. Вне зависимости от
того, кто конкретно заседает в правительстве, оно имеет глобальную точку
доступа «Правительство страны N».
:::

::::: {.section .structure-container}
##  Структура {#structure}

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/singleton/structure-ru9e19.png?id=da49955f21ea6fcf9ae4c915b128838b"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.48;"
srcset="/images/patterns/diagrams/singleton/structure-ru.png?id=da49955f21ea6fcf9ae4c915b128838b 430w,/images/patterns/diagrams/singleton/structure-ru-1.5x.png?id=20a419904de28d05470e116c416899ee 645w,/images/patterns/diagrams/singleton/structure-ru-2x.png?id=3f13b854c8b349271585d421df599a02 860w"
sizes="(max-width: 720px) 100vw, 430px" loading="lazy" width="430"
alt="Структура классов паттерна Одиночка" /><img
src="../../images/patterns/diagrams/singleton/structure-ru-indexed0e2b.png?id=c2dee6ae7067e5ce69256c52ce9375a0"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.48;"
srcset="/images/patterns/diagrams/singleton/structure-ru-indexed.png?id=c2dee6ae7067e5ce69256c52ce9375a0 430w,/images/patterns/diagrams/singleton/structure-ru-indexed-1.5x.png?id=8eafd0a71073c99caa79baa6027ee0e2 645w,/images/patterns/diagrams/singleton/structure-ru-indexed-2x.png?id=5ff51b0823b36d870c5c4dfb4f8f257f 860w"
sizes="(max-width: 720px) 100vw, 430px" loading="lazy" width="430"
alt="Структура классов паттерна Одиночка" />
</figure>
:::

1.  **Одиночка** определяет статический метод `getInstance`, который
    возвращает единственный экземпляр своего класса.

    Конструктор одиночки должен быть скрыт от клиентов. Вызов метода
    `getInstance` должен стать единственным способом получить объект
    этого класса.
::::
:::::

::: {.section .pseudocode}
##  Псевдокод {#pseudocode}

В этом примере роль **Одиночки** отыгрывает класс подключения к базе
данных.

Этот класс не имеет публичного конструктора, поэтому единственный способ
получить его объект --- это вызвать метод `getInstance`. Этот метод
сохранит первый созданный объект и будет возвращать его при всех
последующих вызовах.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// Класс одиночки определяет статический метод `getInstance`,
// который позволяет клиентам повторно использовать одно и то же
// подключение к базе данных по всей программе.
class Database is
    // Поле для хранения объекта-одиночки должно быть объявлено
    // статичным.
    private static field instance: Database

    // Конструктор одиночки всегда должен оставаться приватным,
    // чтобы клиенты не могли самостоятельно создавать
    // экземпляры этого класса через оператор `new`.
    private constructor Database() is
        // Здесь может жить код инициализации подключения к
        // серверу баз данных.
        // ...

    // Основной статический метод одиночки служит альтернативой
    // конструктору и является точкой доступа к экземпляру этого
    // класса.
    public static method getInstance() is
        if (Database.instance == null) then
            acquireThreadLock() and then
                // На всякий случай ещё раз проверим, не был ли
                // объект создан другим потоком, пока текущий
                // ждал освобождения блокировки.
                if (Database.instance == null) then
                    Database.instance = new Database()
        return Database.instance

    // Наконец, любой класс одиночки должен иметь какую-то
    // полезную функциональность, которую клиенты будут
    // запускать через полученный объект одиночки.
    public method query(sql) is
        // Все запросы к базе данных будут проходить через этот
        // метод. Поэтому имеет смысл поместить сюда какую-то
        // логику кеширования.
        // ...

class Application is
    method main() is
        Database foo = Database.getInstance()
        foo.query(&quot;SELECT ...&quot;)
        // ...
        Database bar = Database.getInstance()
        bar.query(&quot;SELECT ...&quot;)
        // Переменная &quot;bar&quot; содержит тот же объект, что и
        // переменная &quot;foo&quot;.</code></pre>
</figure>
:::

:::::::: {.section .applicability-container}
##  Применимость {#applicability}

::::::: applicability
::: applicability-problem
Когда в программе должен быть единственный экземпляр какого-то класса,
доступный всем клиентам (например, общий доступ к базе данных из разных
частей программы).
:::

::: applicability-solution
Одиночка скрывает от клиентов все способы создания нового объекта, кроме
специального метода. Этот метод либо создаёт объект, либо отдаёт
существующий объект, если он уже был создан.
:::

::: applicability-problem
Когда вам хочется иметь больше контроля над глобальными переменными.
:::

::: applicability-solution
В отличие от глобальных переменных, Одиночка гарантирует, что никакой
другой код не заменит созданный экземпляр класса, поэтому вы всегда
уверены в наличии лишь одного объекта-одиночки.

Тем не менее, в любой момент вы можете расширить это ограничение и
позволить любое количество объектов-одиночек, поменяв код в одном месте
(метод `getInstance`).
:::
:::::::
::::::::

::: {.section .checklist}
##  Шаги реализации {#checklist}

1.  Добавьте в класс приватное статическое поле, которое будет содержать
    одиночный объект.

2.  Объявите статический создающий метод, который будет использоваться
    для получения одиночки.

3.  Добавьте «ленивую инициализацию» (создание объекта при первом вызове
    метода) в создающий метод одиночки.

4.  Сделайте конструктор класса приватным.

5.  В клиентском коде замените вызовы конструктора одиночка вызовами его
    создающего метода.
:::

:::::: {.section .pros-cons}
##  Преимущества и недостатки {#pros-cons}

::::: row
::: col-sm-6
-    Гарантирует наличие единственного экземпляра класса.
-    Предоставляет к нему глобальную точку доступа.
-    Реализует отложенную инициализацию объекта-одиночки.
:::

::: col-sm-6
-    Нарушает *принцип единственной ответственности класса*.
-    Маскирует плохой дизайн.
-    Проблемы мультипоточности.
-    Требует постоянного создания Mock-объектов при юнит-тестировании.
:::
:::::
::::::

::: {.section .relations}
##  Отношения с другими паттернами {#relations}

-   [Фасад](facade.html) можно сделать [Одиночкой](singleton.html), так
    как обычно нужен только один объект-фасад.

-   Паттерн [Легковес](flyweight.html) может напоминать
    [Одиночку](singleton.html), если для конкретной задачи у вас
    получилось свести количество объектов к одному. Но помните, что
    между паттернами есть два кардинальных отличия:

    1.  В отличие от *Одиночки*, вы можете иметь множество
        объектов-легковесов.
    2.  Объекты-легковесы должны быть неизменяемыми, тогда как
        объект-одиночка допускает изменение своего состояния.

-   [Абстрактная фабрика](abstract-factory.html),
    [Строитель](builder.html) и [Прототип](prototype.html) могут быть
    реализованы при помощи [Одиночки](singleton.html).
:::

::: {.section .implementations}
##  Примеры реализации паттерна {#implementations}

[![Одиночка на
C#](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](singleton/csharp/example.html "Одиночка на C#"){.prog-lang-link}
[![Одиночка на
C++](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](singleton/cpp/example.html "Одиночка на C++"){.prog-lang-link}
[![Одиночка на
Go](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](singleton/go/example.html "Одиночка на Go"){.prog-lang-link}
[![Одиночка на
Java](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](singleton/java/example.html "Одиночка на Java"){.prog-lang-link}
[![Одиночка на
PHP](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](singleton/php/example.html "Одиночка на PHP"){.prog-lang-link}
[![Одиночка на
Python](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](singleton/python/example.html "Одиночка на Python"){.prog-lang-link}
[![Одиночка на
Ruby](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](singleton/ruby/example.html "Одиночка на Ruby"){.prog-lang-link}
[![Одиночка на
Rust](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](singleton/rust/example.html "Одиночка на Rust"){.prog-lang-link}
[![Одиночка на
Swift](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](singleton/swift/example.html "Одиночка на Swift"){.prog-lang-link}
[![Одиночка на
TypeScript](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](singleton/typescript/example.html "Одиночка на TypeScript"){.prog-lang-link}
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

[Структурные паттерны []{.fa
.fa-arrow-right}](structural-patterns.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Прототип](prototype.html){.btn .btn-default
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
