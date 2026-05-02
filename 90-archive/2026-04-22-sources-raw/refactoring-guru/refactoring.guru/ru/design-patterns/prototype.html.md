:::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [Паттерны
проектирования](../design-patterns.html){.type} / [Порождающие
паттерны](creational-patterns.html){.type}
:::

# Прототип {#прототип .title}

::: aka
Также известен как:
[Клон, ]{style="display:inline-block"}[Prototype]{style="display:inline-block"}
:::

::: {.section .intent}
##  Суть паттерна {#intent}

**Прототип** --- это порождающий паттерн проектирования, который
позволяет копировать объекты, не вдаваясь в подробности их реализации.

<figure class="image">
<img
src="../../images/patterns/content/prototype/prototype3cb9.png?id=e912b1ada20bbf7b2ffc09e93b9fab20"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/prototype/prototype.png?id=e912b1ada20bbf7b2ffc09e93b9fab20 640w,/images/patterns/content/prototype/prototype-1.5x.png?id=a0f76894fb657783b65b9ed899857468 960w,/images/patterns/content/prototype/prototype-2x.png?id=670789c80c8a114e25838ede2da4a881 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Паттерн Прототип" />
</figure>
:::

::: {.section .problem}
##  Проблема {#problem}

У вас есть объект, который нужно скопировать. Как это сделать? Нужно
создать пустой объект такого же класса, а затем поочерёдно скопировать
значения всех полей из старого объекта в новый.

Прекрасно! Но есть нюанс. Не каждый объект удастся скопировать таким
образом, ведь часть его состояния может быть приватной, а значит ---
недоступной для остального кода программы.

<figure class="image">
<img
src="../../images/patterns/content/prototype/prototype-comic-1-ru33ee.png?id=783e233fbde180187cb1d00a7b712814"
style="aspect-ratio: 2;"
srcset="/images/patterns/content/prototype/prototype-comic-1-ru.png?id=783e233fbde180187cb1d00a7b712814 600w,/images/patterns/content/prototype/prototype-comic-1-ru-1.5x.png?id=6feeacd026bbf8af5cec5f07ad4c3910 900w,/images/patterns/content/prototype/prototype-comic-1-ru-2x.png?id=9739f9263fea42a8e63613696fa8b8b5 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Пример неудачного копирования извне" />
<figcaption><p>Копирование «извне» <a
href="https://vimeo.com/120816425">не всегда</a> возможно
в реальности.</p></figcaption>
</figure>

Но есть и другая проблема. Копирующий код станет зависим от классов
копируемых объектов. Ведь, чтобы перебрать *все* поля объекта, нужно
привязаться к его классу. Из-за этого вы не сможете копировать объекты,
зная только их интерфейсы, а не конкретные классы.
:::

::: {.section .solution}
##  Решение {#solution}

Паттерн Прототип поручает создание копий самим копируемым объектам. Он
вводит общий интерфейс для всех объектов, поддерживающих клонирование.
Это позволяет копировать объекты, не привязываясь к их конкретным
классам. Обычно такой интерфейс имеет всего один метод `clone`.

Реализация этого метода в разных классах очень схожа. Метод создаёт
новый объект текущего класса и копирует в него значения всех полей
собственного объекта. Так получится скопировать даже приватные поля, так
как большинство языков программирования разрешает доступ к приватным
полям любого объекта текущего класса.

Объект, который копируют, называется *прототипом* (откуда и название
паттерна). Когда объекты программы содержат сотни полей и тысячи
возможных конфигураций, прототипы могут служить своеобразной
альтернативой созданию подклассов.

<figure class="image">
<img
src="../../images/patterns/content/prototype/prototype-comic-2-ru7c52.png?id=6df088efe826fcab5162c8d2944f7477"
style="aspect-ratio: 1.14;"
srcset="/images/patterns/content/prototype/prototype-comic-2-ru.png?id=6df088efe826fcab5162c8d2944f7477 343w,/images/patterns/content/prototype/prototype-comic-2-ru-1.5x.png?id=df6f6191a0e0979f2ebcfca22ca239b8 515w,/images/patterns/content/prototype/prototype-comic-2-ru-2x.png?id=6c9ed21fd081a5c65ce466289a510499 687w"
sizes="(max-width: 720px) 100vw, 343px" loading="lazy" width="343"
alt="Предварительно заготовленные прототипы" />
<figcaption><p>Предварительно заготовленные прототипы могут стать
заменой подклассам.</p></figcaption>
</figure>

В этом случае все возможные прототипы заготавливаются и настраиваются на
этапе инициализации программы. Потом, когда программе нужен новый
объект, она создаёт копию из приготовленного прототипа.
:::

::: {.section .analogy}
##  Аналогия из жизни {#analogy}

В промышленном производстве прототипы создаются перед основной партией
продуктов для проведения всевозможных испытаний. При этом прототип не
участвует в последующем производстве, отыгрывая пассивную роль.

<figure class="image">
<img
src="../../images/patterns/content/prototype/prototype-comic-3-ru8caf.png?id=f63e3202142bb53f33b796a13b797255"
style="aspect-ratio: 2;"
srcset="/images/patterns/content/prototype/prototype-comic-3-ru.png?id=f63e3202142bb53f33b796a13b797255 600w,/images/patterns/content/prototype/prototype-comic-3-ru-1.5x.png?id=176b37d01c7716ebc9c42e9d49adf4d1 900w,/images/patterns/content/prototype/prototype-comic-3-ru-2x.png?id=88a54ccbaf6aa6cb23133c46ada418f4 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Пример деления клетки" />
<figcaption><p>Пример деления клетки.</p></figcaption>
</figure>

Прототип на производстве не делает копию самого себя, поэтому более
близкий пример паттерна --- деление клеток. После митозного деления
клеток образуются две совершенно идентичные клетки. Оригинальная клетка
отыгрывает роль прототипа, принимая активное участие в создании нового
объекта.
:::

:::::::: {.section .structure-container}
##  Структура {#structure}

::::::: structure
#### Базовая реализация

:::: prototype-normal
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/prototype/structure87f6.png?id=088102c5e9785ff45debbbce86f4df81"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.25;"
srcset="/images/patterns/diagrams/prototype/structure.png?id=088102c5e9785ff45debbbce86f4df81 500w,/images/patterns/diagrams/prototype/structure-1.5x.png?id=beec6a1a5242268e10e39f9fdc0b894b 750w,/images/patterns/diagrams/prototype/structure-2x.png?id=ba75079f42f08028ae4cdbda0cfecc26 1000w"
sizes="(max-width: 720px) 100vw, 500px" loading="lazy" width="500"
alt="Структура классов паттерна Прототип" /><img
src="../../images/patterns/diagrams/prototype/structure-indexed6499.png?id=0e1c809842f5c43aca0541a2eba1f844"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.27;"
srcset="/images/patterns/diagrams/prototype/structure-indexed.png?id=0e1c809842f5c43aca0541a2eba1f844 520w,/images/patterns/diagrams/prototype/structure-indexed-1.5x.png?id=05b072b5b83f5ff1024a7ba448ea9d59 780w,/images/patterns/diagrams/prototype/structure-indexed-2x.png?id=74584ac729c0f6b49d2a97a53cc266ff 1040w"
sizes="(max-width: 720px) 100vw, 520px" loading="lazy" width="520"
alt="Структура классов паттерна Прототип" />
</figure>
:::
::::

1.  **Интерфейс прототипов** описывает операции клонирования. В
    большинстве случаев --- это единственный метод `clone`.

2.  **Конкретный прототип** реализует операцию клонирования самого себя.
    Помимо банального копирования значений всех полей, здесь могут быть
    спрятаны различные сложности, о которых не нужно знать клиенту.
    Например, клонирование связанных объектов, распутывание рекурсивных
    зависимостей и прочее.

3.  **Клиент** создаёт копию объекта, обращаясь к нему через общий
    интерфейс прототипов.

#### Реализация с общим хранилищем прототипов

:::: prototype-pool
::: {.struct-image2 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/prototype/structure-prototype-cache9d19.png?id=609c2af5d14ed55dcbb218a00f98e7d5"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.15;"
srcset="/images/patterns/diagrams/prototype/structure-prototype-cache.png?id=609c2af5d14ed55dcbb218a00f98e7d5 550w,/images/patterns/diagrams/prototype/structure-prototype-cache-1.5x.png?id=8ca0b829185d49c5acbe19966a0659d4 825w,/images/patterns/diagrams/prototype/structure-prototype-cache-2x.png?id=a1e4514bbcc9b10968b856f19b407105 1100w"
sizes="(max-width: 720px) 100vw, 550px" loading="lazy" width="550"
alt="Вариант Прототипа с общим хранилищем прототипов" /><img
src="../../images/patterns/diagrams/prototype/structure-prototype-cache-indexed1801.png?id=10a4a84a1a318f59dbc2b806fc936d04"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.12;"
srcset="/images/patterns/diagrams/prototype/structure-prototype-cache-indexed.png?id=10a4a84a1a318f59dbc2b806fc936d04 550w,/images/patterns/diagrams/prototype/structure-prototype-cache-indexed-1.5x.png?id=cb56c95533a4020368c48db9f9e2a37d 825w,/images/patterns/diagrams/prototype/structure-prototype-cache-indexed-2x.png?id=47b99eb7ae51158bdbb31deea4f5e98f 1100w"
sizes="(max-width: 720px) 100vw, 550px" loading="lazy" width="550"
alt="Вариант Прототипа с общим хранилищем прототипов" />
</figure>
:::
::::

1.  **Хранилище прототипов** облегчает доступ к часто используемым
    прототипам, храня набор предварительно созданных эталонных, готовых
    к копированию объектов. Простейшее хранилище может быть построено с
    помощью хеш-таблицы вида `имя-прототипа → прототип`. Но для удобства
    поиска прототипы можно маркировать и другими критериями, а не только
    условным именем.
:::::::
::::::::

::: {.section .pseudocode}
##  Псевдокод {#pseudocode}

В этом примере **Прототип** позволяет производить точные копии объектов
геометрических фигур, не привязываясь к их классам.

<figure class="image">
<img
src="../../images/patterns/diagrams/prototype/examplee491.png?id=47bc6c1058cb100b81e675b5ca6bda6c"
style="aspect-ratio: 1.42;"
srcset="/images/patterns/diagrams/prototype/example.png?id=47bc6c1058cb100b81e675b5ca6bda6c 470w,/images/patterns/diagrams/prototype/example-1.5x.png?id=067f3a2415370fadf5f92aadaa12b5e2 705w,/images/patterns/diagrams/prototype/example-2x.png?id=80393e0df17ae8130e5ada832d494949 940w"
sizes="(max-width: 720px) 100vw, 470px" loading="lazy" width="470"
alt="Структура классов примера паттерна Прототип" />
<figcaption><p>Пример клонирования иерархии
геометрических фигур.</p></figcaption>
</figure>

Все фигуры реализуют интерфейс клонирования и предоставляют метод для
воспроизводства самой себя. Подклассы используют метод клонирования
родителя, а затем копируют собственные поля в получившийся объект.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// Базовый прототип.
abstract class Shape is
    field X: int
    field Y: int
    field color: string

    // Обычный конструктор.
    constructor Shape() is
        // ...

    // Конструктор прототипа.
    constructor Shape(source: Shape) is
        this()
        this.X = source.X
        this.Y = source.Y
        this.color = source.color

    // Результатом операции клонирования всегда будет объект из
    // иерархии классов Shape.
    abstract method clone():Shape


// Конкретный прототип. Метод клонирования создаёт новый объект
// текущего класса, передавая в его конструктор ссылку на
// собственный объект. Благодаря этому операция клонирования
// получается атомарной — пока не выполнится конструктор, нового
// объекта ещё не существует. Но как только конструктор завершит
// работу, мы получим полностью готовый объект-клон, а не пустой
// объект, который нужно ещё заполнить.
class Rectangle extends Shape is
    field width: int
    field height: int

    constructor Rectangle(source: Rectangle) is
        // Вызов родительского конструктора нужен, чтобы
        // скопировать потенциальные приватные поля, объявленные
        // в родительском классе.
        super(source)
        this.width = source.width
        this.height = source.height

    method clone():Shape is
        return new Rectangle(this)


class Circle extends Shape is
    field radius: int

    constructor Circle(source: Circle) is
        super(source)
        this.radius = source.radius

    method clone():Shape is
        return new Circle(this)


// Где-то в клиентском коде.
class Application is
    field shapes: array of Shape

    constructor Application() is
        Circle circle = new Circle()
        circle.X = 10
        circle.Y = 10
        circle.radius = 20
        shapes.add(circle)

        Circle anotherCircle = circle.clone()
        shapes.add(anotherCircle)
        // anotherCircle будет содержать точную копию circle.

        Rectangle rectangle = new Rectangle()
        rectangle.width = 10
        rectangle.height = 20
        shapes.add(rectangle)

    method businessLogic() is
        // Плюс Прототипа в том, что вы можете клонировать набор
        // объектов, не зная их конкретные классы.
        Array shapesCopy = new Array of Shapes.

        // Например, мы не знаем, какие конкретно объекты
        // находятся внутри массива shapes, так как он объявлен
        // с типом Shape. Но благодаря полиморфизму, мы можем
        // клонировать все объекты «вслепую». Будет выполнен
        // метод clone того класса, которым является этот
        // объект.
        foreach (s in shapes) do
            shapesCopy.add(s.clone())

        // Переменная shapesCopy будет содержать точные копии
        // элементов массива shapes.</code></pre>
</figure>
:::

:::::::: {.section .applicability-container}
##  Применимость {#applicability}

::::::: applicability
::: applicability-problem
Когда ваш код не должен зависеть от классов копируемых объектов.
:::

::: applicability-solution
Такое часто бывает, если ваш код работает с объектами, поданными извне
через какой-то общий интерфейс. Вы не можете привязаться к их классам,
даже если бы хотели, поскольку их конкретные классы неизвестны.

Паттерн прототип предоставляет клиенту общий интерфейс для работы со
всеми прототипами. Клиенту не нужно зависеть от всех классов копируемых
объектов, а только от интерфейса клонирования.
:::

::: applicability-problem
Когда вы имеете уйму подклассов, которые отличаются начальными
значениями полей. Кто-то мог создать все эти классы, чтобы иметь
возможность легко порождать объекты с определённой конфигурацией.
:::

::: applicability-solution
Паттерн прототип предлагает использовать набор прототипов, вместо
создания подклассов для описания популярных конфигураций объектов.

Таким образом, вместо порождения объектов из подклассов, вы будете
копировать существующие объекты-прототипы, в которых уже настроено
внутреннее состояние. Это позволит избежать взрывного роста количества
классов в программе и уменьшить её сложность.
:::
:::::::
::::::::

::: {.section .checklist}
##  Шаги реализации {#checklist}

1.  Создайте интерфейс прототипов с единственным методом `clone`. Если у
    вас уже есть иерархия продуктов, метод клонирования можно объявить
    непосредственно в каждом из её классов.

2.  Добавьте в классы будущих прототипов альтернативный конструктор,
    принимающий в качестве аргумента объект текущего класса. Этот
    конструктор должен скопировать из поданного объекта значения всех
    полей, объявленных в рамках текущего класса, а затем передать
    выполнение родительскому конструктору, чтобы тот позаботился о
    полях, объявленных в суперклассе.

    Если ваш язык программирования не поддерживает перегрузку методов,
    то вам не удастся создать несколько версий конструктора. В этом
    случае копирование значений можно проводить и в другом методе,
    специально созданном для этих целей. Конструктор удобнее тем, что
    позволяет клонировать объект за один вызов.

3.  Метод клонирования обычно состоит всего из одной строки: вызова
    оператора `new` с конструктором прототипа. Все классы,
    поддерживающие клонирование, должны явно определить метод `clone`,
    чтобы использовать собственный класс с оператором `new`. В обратном
    случае результатом клонирования станет объект родительского класса.

4.  Опционально, создайте центральное хранилище прототипов. В нём удобно
    хранить вариации объектов, возможно, даже одного класса, но
    по-разному настроенных.

    Вы можете разместить это хранилище либо в новом фабричном классе,
    либо в фабричном методе базового класса прототипов. Такой фабричный
    метод должен на основании входящих аргументов искать в хранилище
    прототипов подходящий экземпляр, а затем вызывать его метод
    клонирования и возвращать полученный объект.

    Наконец, нужно избавиться от прямых вызовов конструкторов объектов,
    заменив их вызовами фабричного метода хранилища прототипов.
:::

:::::: {.section .pros-cons}
##  Преимущества и недостатки {#pros-cons}

::::: row
::: col-sm-6
-    Позволяет клонировать объекты, не привязываясь к их конкретным
    классам.
-    Меньше повторяющегося кода инициализации объектов.
-    Ускоряет создание объектов.
-    Альтернатива созданию подклассов для конструирования сложных
    объектов.
:::

::: col-sm-6
-    Сложно клонировать составные объекты, имеющие ссылки на другие
    объекты.
:::
:::::
::::::

::: {.section .relations}
##  Отношения с другими паттернами {#relations}

-   Многие архитектуры начинаются с применения [Фабричного
    метода](factory-method.html) (более простого и расширяемого через
    подклассы) и эволюционируют в сторону [Абстрактной
    фабрики](abstract-factory.html), [Прототипа](prototype.html) или
    [Строителя](builder.html) (более гибких, но и более сложных).

-   Классы [Абстрактной фабрики](abstract-factory.html) чаще всего
    реализуются с помощью [Фабричного метода](factory-method.html), хотя
    они могут быть построены и на основе [Прототипа](prototype.html).

-   Если [Команду](command.html) нужно копировать перед вставкой в
    историю выполненных команд, вам может помочь
    [Прототип](prototype.html).

-   Архитектура, построенная на [Компоновщиках](composite.html) и
    [Декораторах](decorator.html), часто может быть улучшена за счёт
    внедрения [Прототипа](prototype.html). Он позволяет клонировать
    сложные структуры объектов, а не собирать их заново.

-   [Прототип](prototype.html) не опирается на наследование, но ему
    нужна сложная операция инициализации. [Фабричный
    метод](factory-method.html), наоборот, построен на наследовании, но
    не требует сложной инициализации.

-   [Снимок](memento.html) иногда можно заменить
    [Прототипом](prototype.html), если объект, состояние которого
    требуется сохранять в истории, довольно простой, не имеет активных
    ссылок на внешние ресурсы либо их можно легко восстановить.

-   [Абстрактная фабрика](abstract-factory.html),
    [Строитель](builder.html) и [Прототип](prototype.html) могут быть
    реализованы при помощи [Одиночки](singleton.html).
:::

::: {.section .implementations}
##  Примеры реализации паттерна {#implementations}

[![Прототип на
C#](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](prototype/csharp/example.html "Прототип на C#"){.prog-lang-link}
[![Прототип на
C++](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](prototype/cpp/example.html "Прототип на C++"){.prog-lang-link}
[![Прототип на
Go](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](prototype/go/example.html "Прототип на Go"){.prog-lang-link}
[![Прототип на
Java](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](prototype/java/example.html "Прототип на Java"){.prog-lang-link}
[![Прототип на
PHP](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](prototype/php/example.html "Прототип на PHP"){.prog-lang-link}
[![Прототип на
Python](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](prototype/python/example.html "Прототип на Python"){.prog-lang-link}
[![Прототип на
Ruby](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](prototype/ruby/example.html "Прототип на Ruby"){.prog-lang-link}
[![Прототип на
Rust](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](prototype/rust/example.html "Прототип на Rust"){.prog-lang-link}
[![Прототип на
Swift](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](prototype/swift/example.html "Прототип на Swift"){.prog-lang-link}
[![Прототип на
TypeScript](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](prototype/typescript/example.html "Прототип на TypeScript"){.prog-lang-link}
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

[Одиночка []{.fa .fa-arrow-right}](singleton.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Строитель](builder.html){.btn .btn-default
rel="prev"}
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
