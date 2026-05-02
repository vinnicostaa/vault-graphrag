::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [Паттерны
проектирования](../design-patterns.html){.type} / [Структурные
паттерны](structural-patterns.html){.type}
:::

# Декоратор {#декоратор .title}

::: aka
Также известен как:
[Wrapper, ]{style="display:inline-block"}[Обёртка, ]{style="display:inline-block"}[Decorator]{style="display:inline-block"}
:::

::: {.section .intent}
##  Суть паттерна {#intent}

**Декоратор** --- это структурный паттерн проектирования, который
позволяет динамически добавлять объектам новую функциональность,
оборачивая их в полезные «обёртки».

<figure class="image">
<img
src="../../images/patterns/content/decorator/decoratore8f5.png?id=710c66670c7123e0928d3b3758aea79e"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/decorator/decorator.png?id=710c66670c7123e0928d3b3758aea79e 640w,/images/patterns/content/decorator/decorator-1.5x.png?id=72aafc603a289fc64e028e83e8aa843b 960w,/images/patterns/content/decorator/decorator-2x.png?id=736ab07b1d8920ab2c7a70c9cb1305cc 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Паттерн Декоратор" />
</figure>
:::

::: {.section .problem}
##  Проблема {#problem}

Вы работаете над библиотекой оповещений, которую можно подключать к
разнообразным программам, чтобы получать уведомления о важных событиях.

Основой библиотеки является класс `Notifier` с методом `send`, который
принимает на вход строку-сообщение и высылает её всем администраторам по
электронной почте. Сторонняя программа должна создать и настроить этот
объект, указав кому отправлять оповещения, а затем использовать его
каждый раз, когда что-то случается.

<figure class="image">
<img
src="../../images/patterns/diagrams/decorator/problem1-rua682.png?id=65cf66903d80d5c2da3e63775018ae28"
style="aspect-ratio: 2.45;"
srcset="/images/patterns/diagrams/decorator/problem1-ru.png?id=65cf66903d80d5c2da3e63775018ae28 540w,/images/patterns/diagrams/decorator/problem1-ru-1.5x.png?id=5d581bdaf519ab4ac2bc564c5460b390 810w,/images/patterns/diagrams/decorator/problem1-ru-2x.png?id=ba735cc6c4faff485704a7c27f40cd7b 1080w"
sizes="(max-width: 720px) 100vw, 540px" loading="lazy" width="540"
alt="Структура библиотеки до применения Декоратора" />
<figcaption><p>Сторонние программы используют главный
класс оповещений.</p></figcaption>
</figure>

В какой-то момент стало понятно, что одних email-оповещений
пользователям мало. Некоторые из них хотели бы получать извещения о
критических проблемах через SMS. Другие хотели бы получать их в виде
сообщений Facebook. Корпоративные пользователи хотели бы видеть
сообщения в Slack.

<figure class="image">
<img
src="../../images/patterns/diagrams/decorator/problem2a2f8.png?id=ba5d5e106ea8c4848d60e230feca9135"
style="aspect-ratio: 2.59;"
srcset="/images/patterns/diagrams/decorator/problem2.png?id=ba5d5e106ea8c4848d60e230feca9135 440w,/images/patterns/diagrams/decorator/problem2-1.5x.png?id=4f15b6208533188dd09e7da7dd6b509a 660w,/images/patterns/diagrams/decorator/problem2-2x.png?id=28b2c8509b4e78c031d728424b876ebc 880w"
sizes="(max-width: 720px) 100vw, 440px" loading="lazy" width="440"
alt="Библиотека после добавления других типов оповещений" />
<figcaption><p>Каждый тип оповещения живёт в
собственном подклассе.</p></figcaption>
</figure>

Сначала вы добавили каждый из этих типов оповещений в программу,
унаследовав их от базового класса `Notifier`. Теперь пользователь
выбирал один из типов оповещений, который и использовался в дальнейшем.

Но затем кто-то резонно спросил, почему нельзя выбрать несколько типов
оповещений сразу? Ведь если вдруг в вашем доме начался пожар, вы бы
хотели получить оповещения по всем каналам, не так ли?

Вы попытались реализовать все возможные комбинации подклассов
оповещений. Но после того как вы добавили первый десяток классов, стало
ясно, что такой подход невероятно раздувает код программы.

<figure class="image">
<img
src="../../images/patterns/diagrams/decorator/problem34033.png?id=f3b3e7a107d870871f2c3167adcb7ccb"
style="aspect-ratio: 1.85;"
srcset="/images/patterns/diagrams/decorator/problem3.png?id=f3b3e7a107d870871f2c3167adcb7ccb 630w,/images/patterns/diagrams/decorator/problem3-1.5x.png?id=f4ef9d367df838c77121e9f84260b09c 945w,/images/patterns/diagrams/decorator/problem3-2x.png?id=abb7a87b521ce97d7661dd9c0b988cc3 1260w"
sizes="(max-width: 720px) 100vw, 630px" loading="lazy" width="630"
alt="Библиотека после комбинирования оповещений" />
<figcaption><p>Комбинаторный взрыв подклассов при совмещении
типов оповещений.</p></figcaption>
</figure>

Итак, нужен какой-то другой способ комбинирования поведения объектов,
который не приводит к взрыву количества подклассов.
:::

::: {.section .solution}
##  Решение {#solution}

Наследование --- это первое, что приходит в голову многим программистам,
когда нужно расширить какое-то существующее поведение. Но механизм
наследования имеет несколько досадных проблем.

-   Он **статичен**. Вы не можете изменить поведение существующего
    объекта. Для этого вам надо создать новый объект, выбрав другой
    подкласс.
-   Он **не разрешает наследовать поведение нескольких классов
    одновременно**. Из-за этого вам приходится создавать множество
    подклассов-комбинаций для получения совмещённого поведения.

Одним из способов обойти эти проблемы является замена наследования
*агрегацией* либо *композицией* []{.pop-i}[*Композиция* --- это более
строгий вариант агрегации, при котором компоненты не могут жить без
контейнера.]{.pop-content}. Это когда один объект *содержит* ссылку на
другой и делегирует ему работу, вместо того чтобы самому *наследовать*
его поведение. Как раз на этом принципе построен паттерн Декоратор.

<figure class="image">
<img
src="../../images/patterns/diagrams/decorator/solution1-rubb38.png?id=aa536fe5e9f55a8c817eaa43bb609a97"
style="aspect-ratio: 3.44;"
srcset="/images/patterns/diagrams/decorator/solution1-ru.png?id=aa536fe5e9f55a8c817eaa43bb609a97 550w,/images/patterns/diagrams/decorator/solution1-ru-1.5x.png?id=2769c3e2e693b09c089bb66686e4dc21 825w,/images/patterns/diagrams/decorator/solution1-ru-2x.png?id=50b4c774a08a0c9f603733d1c7526bc2 1100w"
sizes="(max-width: 720px) 100vw, 550px" loading="lazy" width="550"
alt="Наследование против Агрегации" />
<figcaption><p>Наследование против Агрегации.</p></figcaption>
</figure>

Декоратор имеет альтернативное название --- *обёртка*. Оно более точно
описывает суть паттерна: вы помещаете целевой объект в другой
объект-обёртку, который запускает базовое поведение объекта, а затем
добавляет к результату что-то своё.

Оба объекта имеют общий интерфейс, поэтому для пользователя нет никакой
разницы, с каким объектом работать --- чистым или обёрнутым. Вы можете
использовать несколько разных обёрток одновременно --- результат будет
иметь объединённое поведение всех обёрток сразу.

В примере с оповещениями мы оставим в базовом классе простую отправку по
электронной почте, а расширенные способы отправки сделаем декораторами.

<figure class="image">
<img
src="../../images/patterns/diagrams/decorator/solution2e4ab.png?id=cbee4a27080ce3a0bf773482613e1347"
style="aspect-ratio: 1.39;"
srcset="/images/patterns/diagrams/decorator/solution2.png?id=cbee4a27080ce3a0bf773482613e1347 640w,/images/patterns/diagrams/decorator/solution2-1.5x.png?id=d2fa0c0b9bf5cba0e38be7aff7e7b7a8 960w,/images/patterns/diagrams/decorator/solution2-2x.png?id=7775f76b94dbd5cd25f711ce81f59262 1280w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="640"
alt="Схема решения Декоратором" />
<figcaption><p>Расширенные способы оповещения
становятся декораторами.</p></figcaption>
</figure>

Сторонняя программа, выступающая клиентом, во время первичной настройки
будет заворачивать объект оповещений в те обёртки, которые соответствуют
желаемому способу оповещения.

<figure class="image">
<img
src="../../images/patterns/diagrams/decorator/solution3-enf2ff.png?id=b7e2e2036435265350ba0c6796162ab5"
style="aspect-ratio: 1.03;"
srcset="/images/patterns/diagrams/decorator/solution3-en.png?id=b7e2e2036435265350ba0c6796162ab5 300w,/images/patterns/diagrams/decorator/solution3-en-1.5x.png?id=3c9b7a4aec3a8d2f68d64bffb79fa4f7 450w,/images/patterns/diagrams/decorator/solution3-en-2x.png?id=9a4ef2b4267685a83d0233d78775497b 600w"
sizes="(max-width: 720px) 100vw, 300px" loading="lazy" width="300"
alt="Программа может составлять сложные стеки декораторов" />
<figcaption><p>Программа может составлять составные объекты
из декораторов.</p></figcaption>
</figure>

Последняя обёртка в списке и будет тем объектом, с которым клиент будет
работать в остальное время. Для остального клиентского кода, по сути,
ничего не изменится, ведь все обёртки имеют точно такой же интерфейс,
что и базовый класс оповещений.

Таким же образом можно изменять не только способ доставки оповещений, но
и форматирование, список адресатов и так далее. К тому же клиент может
«дообернуть» объект любыми другими обёртками, когда ему захочется.
:::

::: {.section .analogy}
##  Аналогия из жизни {#analogy}

<figure class="image">
<img
src="../../images/patterns/content/decorator/decorator-comic-1cbff.png?id=80d95baacbfb91f5bcdbdc7814b0c64d"
style="aspect-ratio: 2;"
srcset="/images/patterns/content/decorator/decorator-comic-1.png?id=80d95baacbfb91f5bcdbdc7814b0c64d 600w,/images/patterns/content/decorator/decorator-comic-1-1.5x.png?id=490ef9754d7554c2046b69f6f213c6da 900w,/images/patterns/content/decorator/decorator-comic-1-2x.png?id=ba869f621b6e0ea173fdc2b535fc7eed 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Пример паттерна Декоратор" />
<figcaption><p>Одежду можно надевать слоями, получая
комбинированный эффект.</p></figcaption>
</figure>

Любая одежда --- это аналог Декоратора. Применяя Декоратор, вы не
меняете первоначальный класс и не создаёте дочерних классов. Так и с
одеждой --- надевая свитер, вы не перестаёте быть собой, но получаете
новое свойство --- защиту от холода. Вы можете пойти дальше и надеть
сверху ещё один декоратор --- плащ, чтобы защититься и от дождя.
:::

::::: {.section .structure-container}
##  Структура {#structure}

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/decorator/structure128f.png?id=8c95d894aecce5315cc1b12093a7ea0c"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 0.92;"
srcset="/images/patterns/diagrams/decorator/structure.png?id=8c95d894aecce5315cc1b12093a7ea0c 480w,/images/patterns/diagrams/decorator/structure-1.5x.png?id=4b2cd91d4483d9e3bba725f0e45d281d 720w,/images/patterns/diagrams/decorator/structure-2x.png?id=3cfa1f10417a4ef0c12580bc4a63b80d 960w"
sizes="(max-width: 720px) 100vw, 480px" loading="lazy" width="480"
alt="Структура классов паттерна Декоратор" /><img
src="../../images/patterns/diagrams/decorator/structure-indexeda93c.png?id=09401b230a58f2249e4c9a1195d485a0"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 0.98;"
srcset="/images/patterns/diagrams/decorator/structure-indexed.png?id=09401b230a58f2249e4c9a1195d485a0 500w,/images/patterns/diagrams/decorator/structure-indexed-1.5x.png?id=dccc54182965078ccb4cfdeee41acbe5 750w,/images/patterns/diagrams/decorator/structure-indexed-2x.png?id=2733e7d0e322bfb2f150ccf8a878dbf6 1000w"
sizes="(max-width: 720px) 100vw, 500px" loading="lazy" width="500"
alt="Структура классов паттерна Декоратор" />
</figure>
:::

1.  **Компонент** задаёт общий интерфейс обёрток и оборачиваемых
    объектов.

2.  **Конкретный компонент** определяет класс оборачиваемых объектов. Он
    содержит какое-то базовое поведение, которое потом изменяют
    декораторы.

3.  **Базовый декоратор** хранит ссылку на вложенный объект-компонент.
    Им может быть как конкретный компонент, так и один из конкретных
    декораторов. Базовый декоратор делегирует все свои операции
    вложенному объекту. Дополнительное поведение будет жить в конкретных
    декораторах.

4.  **Конкретные декораторы** --- это различные вариации декораторов,
    которые содержат добавочное поведение. Оно выполняется до или после
    вызова аналогичного поведения обёрнутого объекта.

5.  **Клиент** может оборачивать простые компоненты и декораторы в
    другие декораторы, работая со всеми объектами через общий интерфейс
    компонентов.
::::
:::::

::: {.section .pseudocode}
##  Псевдокод {#pseudocode}

В этом примере **Декоратор** защищает финансовые данные дополнительными
уровнями безопасности прозрачно для кода, который их использует.

<figure class="image">
<img
src="../../images/patterns/diagrams/decorator/example42cb.png?id=eec9dc488f00c85f50e764323baa723e"
style="aspect-ratio: 0.98;"
srcset="/images/patterns/diagrams/decorator/example.png?id=eec9dc488f00c85f50e764323baa723e 540w,/images/patterns/diagrams/decorator/example-1.5x.png?id=40ccaba4f5a8e6a2ebac697f04b9f10a 810w,/images/patterns/diagrams/decorator/example-2x.png?id=4891323a27d5601a174eec366187d833 1080w"
sizes="(max-width: 720px) 100vw, 540px" loading="lazy" width="540"
alt="Структура классов примера паттерна Декоратор" />
<figcaption><p>Пример шифрования и компрессии данных с
помощью обёрток.</p></figcaption>
</figure>

Приложение оборачивает класс данных в шифрующую и сжимающую обёртки,
которые при чтении выдают оригинальные данные, а при записи ---
зашифрованные и сжатые.

Декораторы, как и сам класс данных, имеют общий интерфейс. Поэтому
клиентскому коду не важно, с чем работать --- c «чистым» объектом данных
или с «обёрнутым».

<figure class="code">
<pre class="code" lang="pseudocode"><code>// Общий интерфейс компонентов.
interface DataSource is
    method writeData(data)
    method readData():data

// Один из конкретных компонентов реализует базовую
// функциональность.
class FileDataSource implements DataSource is
    constructor FileDataSource(filename) { ... }

    method writeData(data) is
        // Записать данные в файл.

    method readData():data is
        // Прочитать данные из файла.

// Родитель всех декораторов содержит код обёртывания.
class DataSourceDecorator implements DataSource is
    protected field wrappee: DataSource

    constructor DataSourceDecorator(source: DataSource) is
        wrappee = source

    method writeData(data) is
        wrappee.writeData(data)

    method readData():data is
        return wrappee.readData()

// Конкретные декораторы добавляют что-то своё к базовому
// поведению обёрнутого компонента.
class EncryptionDecorator extends DataSourceDecorator is
    method writeData(data) is
        // 1. Зашифровать поданные данные.
        // 2. Передать зашифрованные данные в метод writeData
        // обёрнутого объекта (wrappee).

    method readData():data is
        // 1. Получить данные из метода readData обёрнутого
        // объекта (wrappee).
        // 2. Расшифровать их, если они зашифрованы.
        // 3. Вернуть результат.

// Декорировать можно не только базовые компоненты, но и уже
// обёрнутые объекты.
class CompressionDecorator extends DataSourceDecorator is
    method writeData(data) is
        // 1. Запаковать поданные данные.
        // 2. Передать запакованные данные в метод writeData
        // обёрнутого объекта (wrappee).

    method readData():data is
        // 1. Получить данные из метода readData обёрнутого
        // объекта (wrappee).
        // 2. Распаковать их, если они запакованы.
        // 3. Вернуть результат.


// Вариант 1. Простой пример сборки и использования декораторов.
class Application is
    method dumbUsageExample() is
        source = new FileDataSource(&quot;somefile.dat&quot;)
        source.writeData(salaryRecords)
        // В файл были записаны чистые данные.

        source = new CompressionDecorator(source)
        source.writeData(salaryRecords)
        // В файл были записаны сжатые данные.

        source = new EncryptionDecorator(source)
        // Сейчас в source находится связка из трёх объектов:
        // Encryption &gt; Compression &gt; FileDataSource

        source.writeData(salaryRecords)
        // В файл были записаны сжатые и зашифрованные данные.


// Вариант 2. Клиентский код, использующий внешний источник
// данных. Класс SalaryManager ничего не знает о том, как именно
// будут считаны и записаны данные. Он получает уже готовый
// источник данных.
class SalaryManager is
    field source: DataSource

    constructor SalaryManager(source: DataSource) { ... }

    method load() is
        return source.readData()

    method save() is
        source.writeData(salaryRecords)
    // ...Остальные полезные методы...


// Приложение может по-разному собирать декорируемые объекты, в
// зависимости от условий использования.
class ApplicationConfigurator is
    method configurationExample() is
        source = new FileDataSource(&quot;salary.dat&quot;)
        if (enabledEncryption)
            source = new EncryptionDecorator(source)
        if (enabledCompression)
            source = new CompressionDecorator(source)

        logger = new SalaryManager(source)
        salary = logger.load()
    // ...</code></pre>
</figure>
:::

:::::::: {.section .applicability-container}
##  Применимость {#applicability}

::::::: applicability
::: applicability-problem
Когда вам нужно добавлять обязанности объектам на лету, незаметно для
кода, который их использует.
:::

::: applicability-solution
Объекты помещают в обёртки, имеющие дополнительные поведения. Обёртки и
сами объекты имеют одинаковый интерфейс, поэтому клиентам без разницы, с
чем работать --- с обычным объектом данных или с обёрнутым.
:::

::: applicability-problem
Когда нельзя расширить обязанности объекта с помощью наследования.
:::

::: applicability-solution
Во многих языках программирования есть ключевое слово `final`, которое
может заблокировать наследование класса. Расширить такие классы можно
только с помощью Декоратора.
:::
:::::::
::::::::

::: {.section .checklist}
##  Шаги реализации {#checklist}

1.  Убедитесь, что в вашей задаче есть один основной компонент и
    несколько опциональных дополнений или надстроек над ним.

2.  Создайте интерфейс компонента, который описывал бы общие методы как
    для основного компонента, так и для его дополнений.

3.  Создайте класс конкретного компонента и поместите в него основную
    бизнес-логику.

4.  Создайте базовый класс декораторов. Он должен иметь поле для
    хранения ссылки на вложенный объект-компонент. Все методы базового
    декоратора должны делегировать действие вложенному объекту.

5.  И конкретный компонент, и базовый декоратор должны следовать одному
    и тому же интерфейсу компонента.

6.  Теперь создайте классы конкретных декораторов, наследуя их от
    базового декоратора. Конкретный декоратор должен выполнять свою
    добавочную функцию, а затем (или перед этим) вызывать эту же
    операцию обёрнутого объекта.

7.  Клиент берёт на себя ответственность за конфигурацию и порядок
    обёртывания объектов.
:::

:::::: {.section .pros-cons}
##  Преимущества и недостатки {#pros-cons}

::::: row
::: col-sm-6
-    Большая гибкость, чем у наследования.
-    Позволяет добавлять обязанности на лету.
-    Можно добавлять несколько новых обязанностей сразу.
-    Позволяет иметь несколько мелких объектов вместо одного объекта на
    все случаи жизни.
:::

::: col-sm-6
-    Трудно конфигурировать многократно обёрнутые объекты.
-    Обилие крошечных классов.
:::
:::::
::::::

::: {.section .relations}
##  Отношения с другими паттернами {#relations}

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

-   [Цепочка обязанностей](chain-of-responsibility.html) и
    [Декоратор](decorator.html) имеют очень похожие структуры. Оба
    паттерна базируются на принципе рекурсивного выполнения операции
    через серию связанных объектов. Но есть и несколько важных отличий.

    Обработчики в *Цепочке обязанностей* могут выполнять произвольные
    действия, независимые друг от друга, а также в любой момент
    прерывать дальнейшую передачу по цепочке. С другой стороны
    *Декораторы* расширяют какое-то определённое действие, не ломая
    интерфейс базовой операции и не прерывая выполнение остальных
    декораторов.

-   [Компоновщик](composite.html) и [Декоратор](decorator.html) имеют
    похожие структуры классов из-за того, что оба построены на
    рекурсивной вложенности. Она позволяет связать в одну структуру
    бесконечное количество объектов.

    *Декоратор* оборачивает только один объект, а узел *Компоновщика*
    может иметь много детей. *Декоратор* добавляет вложенному объекту
    новую функциональность, а *Компоновщик* не добавляет ничего нового,
    но «суммирует» результаты всех своих детей.

    Но они могут и сотрудничать: *Компоновщик* может использовать
    *Декоратор*, чтобы переопределить функции отдельных частей дерева
    компонентов.

-   Архитектура, построенная на [Компоновщиках](composite.html) и
    [Декораторах](decorator.html), часто может быть улучшена за счёт
    внедрения [Прототипа](prototype.html). Он позволяет клонировать
    сложные структуры объектов, а не собирать их заново.

-   [Стратегия](strategy.html) меняет поведение объекта «изнутри», а
    [Декоратор](decorator.html) изменяет его «снаружи».

-   [Декоратор](decorator.html) и [Заместитель](proxy.html) имеют схожие
    структуры, но разные назначения. Они похожи тем, что оба построены
    на принципе композиции и делегируют работу другим объектам. Паттерны
    отличаются тем, что *Заместитель* сам управляет жизнью сервисного
    объекта, а обёртывание *Декораторов* контролируется клиентом.
:::

::: {.section .implementations}
##  Примеры реализации паттерна {#implementations}

[![Декоратор на
C#](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](decorator/csharp/example.html "Декоратор на C#"){.prog-lang-link}
[![Декоратор на
C++](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](decorator/cpp/example.html "Декоратор на C++"){.prog-lang-link}
[![Декоратор на
Go](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](decorator/go/example.html "Декоратор на Go"){.prog-lang-link}
[![Декоратор на
Java](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](decorator/java/example.html "Декоратор на Java"){.prog-lang-link}
[![Декоратор на
PHP](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](decorator/php/example.html "Декоратор на PHP"){.prog-lang-link}
[![Декоратор на
Python](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](decorator/python/example.html "Декоратор на Python"){.prog-lang-link}
[![Декоратор на
Ruby](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](decorator/ruby/example.html "Декоратор на Ruby"){.prog-lang-link}
[![Декоратор на
Rust](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](decorator/rust/example.html "Декоратор на Rust"){.prog-lang-link}
[![Декоратор на
Swift](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](decorator/swift/example.html "Декоратор на Swift"){.prog-lang-link}
[![Декоратор на
TypeScript](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](decorator/typescript/example.html "Декоратор на TypeScript"){.prog-lang-link}
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

[Фасад []{.fa .fa-arrow-right}](facade.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Компоновщик](composite.html){.btn .btn-default
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
