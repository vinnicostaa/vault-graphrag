::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [Патерни
проектування](../design-patterns.html){.type} / [Структурні
патерни](structural-patterns.html){.type}
:::

# Декоратор {#декоратор .title}

::: aka
Також відомий як:
[Wrapper, ]{style="display:inline-block"}[Обгортка, ]{style="display:inline-block"}[Decorator]{style="display:inline-block"}
:::

::: {.section .intent}
##  Суть патерна {#intent}

**Декоратор** --- це структурний патерн проектування, що дає змогу
динамічно додавати об'єктам нову функціональність, загортаючи їх у
корисні «обгортки».

<figure class="image">
<img
src="../../images/patterns/content/decorator/decoratore8f5.png?id=710c66670c7123e0928d3b3758aea79e"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/decorator/decorator.png?id=710c66670c7123e0928d3b3758aea79e 640w,/images/patterns/content/decorator/decorator-1.5x.png?id=72aafc603a289fc64e028e83e8aa843b 960w,/images/patterns/content/decorator/decorator-2x.png?id=736ab07b1d8920ab2c7a70c9cb1305cc 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Патерн Декоратор" />
</figure>
:::

::: {.section .problem}
##  Проблема {#problem}

Ви працюєте над бібліотекою сповіщень, яку можна підключати до
різноманітних програм, щоб отримувати сповіщення про важливі події.

Основою бібліотеки є клас `Notifier` з методом `send`, який приймає на
вхід рядок-повідомлення і надсилає його всім адміністраторам електронною
поштою. Стороння програма повинна створити й налаштувати цей об'єкт,
вказавши, кому надсилати сповіщення, та використовувати його щоразу,
коли щось відбувається.

<figure class="image">
<img
src="../../images/patterns/diagrams/decorator/problem1-ukaa56.png?id=616e3b8bd84edf3e850920d0fab558af"
style="aspect-ratio: 2.45;"
srcset="/images/patterns/diagrams/decorator/problem1-uk.png?id=616e3b8bd84edf3e850920d0fab558af 540w,/images/patterns/diagrams/decorator/problem1-uk-1.5x.png?id=952c7907abd50781b0c0a023523d3b06 810w,/images/patterns/diagrams/decorator/problem1-uk-2x.png?id=763f80682e1551750cea268e0c2c43b8 1080w"
sizes="(max-width: 720px) 100vw, 540px" loading="lazy" width="540"
alt="Структура бібліотеки до застосування Декоратора" />
<figcaption><p>Сторонні програми використовують головний
клас сповіщень.</p></figcaption>
</figure>

В якийсь момент стало зрозуміло, що користувачам не вистачає одних
тільки email-сповіщень. Деякі з них хотіли б отримувати сповіщення про
критичні проблеми через SMS. Інші хотіли б отримувати їх у вигляді
Facebook-повідомлень. Корпоративні користувачі хотіли би бачити
повідомлення у Slack.

<figure class="image">
<img
src="../../images/patterns/diagrams/decorator/problem2a2f8.png?id=ba5d5e106ea8c4848d60e230feca9135"
style="aspect-ratio: 2.59;"
srcset="/images/patterns/diagrams/decorator/problem2.png?id=ba5d5e106ea8c4848d60e230feca9135 440w,/images/patterns/diagrams/decorator/problem2-1.5x.png?id=4f15b6208533188dd09e7da7dd6b509a 660w,/images/patterns/diagrams/decorator/problem2-2x.png?id=28b2c8509b4e78c031d728424b876ebc 880w"
sizes="(max-width: 720px) 100vw, 440px" loading="lazy" width="440"
alt="Бібліотека після додавання інших типів сповіщення" />
<figcaption><p>Кожен тип сповіщення живе у
власному підкласі.</p></figcaption>
</figure>

Спершу ви додали кожен з типів сповіщень до програми, успадкувавши їх
від базового класу `Notifier`. Тепер користувачі могли вибрати один з
типів сповіщень, який і використовувався надалі.

Але потім хтось резонно запитав, чому не можна увімкнути кілька типів
сповіщень одночасно? Адже, якщо у вашому будинку раптом почалася пожежа,
ви б хотіли отримати сповіщення по всіх каналах, чи не так?

Ви зробили спробу реалізувати всі можливі комбінації підкласів
сповіщень, але після того, як додали перший десяток класів, стало
зрозуміло, що такий підхід неймовірно роздуває код програми.

<figure class="image">
<img
src="../../images/patterns/diagrams/decorator/problem34033.png?id=f3b3e7a107d870871f2c3167adcb7ccb"
style="aspect-ratio: 1.85;"
srcset="/images/patterns/diagrams/decorator/problem3.png?id=f3b3e7a107d870871f2c3167adcb7ccb 630w,/images/patterns/diagrams/decorator/problem3-1.5x.png?id=f4ef9d367df838c77121e9f84260b09c 945w,/images/patterns/diagrams/decorator/problem3-2x.png?id=abb7a87b521ce97d7661dd9c0b988cc3 1260w"
sizes="(max-width: 720px) 100vw, 630px" loading="lazy" width="630"
alt="Бібліотека після комбінування класів сповіщень" />
<figcaption><p>Комбінаторний вибух підкласів при поєднанні
типів сповіщень.</p></figcaption>
</figure>

Отже, потрібен інший спосіб комбінування поведінки об'єктів, який не
призводить до збільшення кількості підкласів.
:::

::: {.section .solution}
##  Рішення {#solution}

Спадкування --- це перше, що приходить в голову багатьом програмістам,
коли потрібно розширити яку-небудь чинну поведінку. Проте механізм
спадкування має кілька прикрих проблем.

-   Він **статичний**. Ви не можете змінити поведінку об'єкта, який вже
    існує. Для цього необхідно створити новий об'єкт, вибравши інший
    підклас.
-   Він **не дозволяє наслідувати поведінку декількох класів
    одночасно**. Тому доведеться створювати безліч підкласів-комбінацій,
    щоб досягти поєднання поведінки.

Одним зі способів, що дозволяє обійти ці проблеми, є заміна спадкування
*агрегацією* або *композицією* []{.pop-i}[*Композиція* --- це більш
суворий варіант агрегації, при якому компоненти не можуть існувати без
контейнера.]{.pop-content}. Це той випадок, коли один об'єкт *утримує*
інший і делегує йому роботу, замість того, щоб самому *успадкувати* його
поведінку. Саме на цьому принципі побудовано патерн Декоратор.

<figure class="image">
<img
src="../../images/patterns/diagrams/decorator/solution1-ukd729.png?id=78c162f77a191b3cba41b6c70fa51b35"
style="aspect-ratio: 3.44;"
srcset="/images/patterns/diagrams/decorator/solution1-uk.png?id=78c162f77a191b3cba41b6c70fa51b35 550w,/images/patterns/diagrams/decorator/solution1-uk-1.5x.png?id=eacec6a00be6292973ce555f8239f84b 825w,/images/patterns/diagrams/decorator/solution1-uk-2x.png?id=3ea164b3886fbed749037c623eeed34e 1100w"
sizes="(max-width: 720px) 100vw, 550px" loading="lazy" width="550"
alt="Спадкування проти Агрегації" />
<figcaption><p>Спадкування проти Агрегації</p></figcaption>
</figure>

Декоратор має альтернативну назву --- *обгортка*. Вона більш вдало
описує суть патерна: ви розміщуєте цільовий об'єкт у іншому
об'єкті-обгортці, який запускає базову поведінку об'єкта, а потім додає
до результату щось своє.

Обидва об'єкти мають загальний інтерфейс, тому для користувача немає
жодної різниці, з чим працювати --- з чистим чи загорнутим об'єктом. Ви
можете використовувати кілька різних обгорток одночасно --- результат
буде мати об'єднану поведінку всіх обгорток.

В нашому прикладі зі сповіщеннями залишимо в базовому класі просте
надсилання сповіщень електронною поштою, а розширені способи зробимо
декораторами.

<figure class="image">
<img
src="../../images/patterns/diagrams/decorator/solution2e4ab.png?id=cbee4a27080ce3a0bf773482613e1347"
style="aspect-ratio: 1.39;"
srcset="/images/patterns/diagrams/decorator/solution2.png?id=cbee4a27080ce3a0bf773482613e1347 640w,/images/patterns/diagrams/decorator/solution2-1.5x.png?id=d2fa0c0b9bf5cba0e38be7aff7e7b7a8 960w,/images/patterns/diagrams/decorator/solution2-2x.png?id=7775f76b94dbd5cd25f711ce81f59262 1280w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="640"
alt="Схема рішення з Декоратором" />
<figcaption><p>Розширені способи надсилання сповіщень
стають декораторами.</p></figcaption>
</figure>

Стороння програма, яка виступає клієнтом, під час початкового
налаштовування буде загортати об'єкт сповіщення в ті обгортки, які
відповідають бажаному способу сповіщення.

<figure class="image">
<img
src="../../images/patterns/diagrams/decorator/solution3-enf2ff.png?id=b7e2e2036435265350ba0c6796162ab5"
style="aspect-ratio: 1.03;"
srcset="/images/patterns/diagrams/decorator/solution3-en.png?id=b7e2e2036435265350ba0c6796162ab5 300w,/images/patterns/diagrams/decorator/solution3-en-1.5x.png?id=3c9b7a4aec3a8d2f68d64bffb79fa4f7 450w,/images/patterns/diagrams/decorator/solution3-en-2x.png?id=9a4ef2b4267685a83d0233d78775497b 600w"
sizes="(max-width: 720px) 100vw, 300px" loading="lazy" width="300"
alt="Програма може створювати складні стеки декораторів" />
<figcaption><p>Програма може збирати складові об’єкти
з декораторів.</p></figcaption>
</figure>

Остання обгортка у списку буде саме тим об'єктом, з яким клієнт
працюватиме увесь інший час. Для решти клієнтського коду нічого не
зміниться, адже всі обгортки мають такий самий інтерфейс, що і базовий
клас сповіщень.

Так само можна змінювати не тільки спосіб доставки сповіщень, але й
форматування, список адресатів і так далі. До того ж клієнт зможе
«дозагорнути» об'єкт у будь-які інші обгортки, якщо йому цього
захочеться.
:::

::: {.section .analogy}
##  Аналогія з життя {#analogy}

<figure class="image">
<img
src="../../images/patterns/content/decorator/decorator-comic-1cbff.png?id=80d95baacbfb91f5bcdbdc7814b0c64d"
style="aspect-ratio: 2;"
srcset="/images/patterns/content/decorator/decorator-comic-1.png?id=80d95baacbfb91f5bcdbdc7814b0c64d 600w,/images/patterns/content/decorator/decorator-comic-1-1.5x.png?id=490ef9754d7554c2046b69f6f213c6da 900w,/images/patterns/content/decorator/decorator-comic-1-2x.png?id=ba869f621b6e0ea173fdc2b535fc7eed 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Приклад патерна Декоратор" />
<figcaption><p>Одяг можна одягати кількома шарами, отримуючи
комбінований ефект.</p></figcaption>
</figure>

Будь-який одяг --- це аналог Декоратора. Застосовуючи Декоратор, ви не
змінюєте початковий клас і не створюєте дочірніх класів. Так само з
одягом: вдягаючи светра, ви не перестаєте бути собою, але отримуєте нову
властивість --- захист від холоду. Ви можете піти далі й одягти зверху
ще один декоратор --- плащ, щоб захиститися від дощу.
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
alt="Структура класів патерна Декоратор" /><img
src="../../images/patterns/diagrams/decorator/structure-indexeda93c.png?id=09401b230a58f2249e4c9a1195d485a0"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 0.98;"
srcset="/images/patterns/diagrams/decorator/structure-indexed.png?id=09401b230a58f2249e4c9a1195d485a0 500w,/images/patterns/diagrams/decorator/structure-indexed-1.5x.png?id=dccc54182965078ccb4cfdeee41acbe5 750w,/images/patterns/diagrams/decorator/structure-indexed-2x.png?id=2733e7d0e322bfb2f150ccf8a878dbf6 1000w"
sizes="(max-width: 720px) 100vw, 500px" loading="lazy" width="500"
alt="Структура класів патерна Декоратор" />
</figure>
:::

1.  **Компонент** задає загальний інтерфейс обгорток та об'єктів, що
    загортаються.

2.  **Конкретний компонент** визначає клас об'єктів, що загортаються.
    Він містить якусь базову поведінку, яку потім змінюють декоратори.

3.  **Базовий декоратор** зберігає посилання на вкладений
    об'єкт-компонент. Це може бути як конкретний компонент, так і один з
    конкретних декораторів. Базовий декоратор делегує всі свої операції
    вкладеному об'єкту. Додаткова поведінка житиме в конкретних
    декораторах.

4.  **Конкретні декоратори** --- це різні варіації декораторів, що
    містять додаткову поведінку. Вона виконується до або після виклику
    аналогічної поведінки загорнутого об'єкта.

5.  **Клієнт** може обертати прості компоненти й декоратори в інші
    декоратори, працюючи з усіма об'єктами через загальний інтерфейс
    компонентів.
::::
:::::

::: {.section .pseudocode}
##  Псевдокод {#pseudocode}

У цьому прикладі **Декоратор** захищає фінансові дані додатковими
рівнями безпеки прозоро для коду, який їх використовує.

<figure class="image">
<img
src="../../images/patterns/diagrams/decorator/example42cb.png?id=eec9dc488f00c85f50e764323baa723e"
style="aspect-ratio: 0.98;"
srcset="/images/patterns/diagrams/decorator/example.png?id=eec9dc488f00c85f50e764323baa723e 540w,/images/patterns/diagrams/decorator/example-1.5x.png?id=40ccaba4f5a8e6a2ebac697f04b9f10a 810w,/images/patterns/diagrams/decorator/example-2x.png?id=4891323a27d5601a174eec366187d833 1080w"
sizes="(max-width: 720px) 100vw, 540px" loading="lazy" width="540"
alt="Структура класів прикладу патерна Декоратор" />
<figcaption><p>Приклад шифрування й компресії даних за
допомогою обгорток.</p></figcaption>
</figure>

Програма обгортає клас даних у шифруючу та стискаючу обгортку, які при
читанні видають оригінальні дані, а при записі --- зашифровані та
стислі.

Декоратори, як і сам клас даних, мають спільний інтерфейс. Тому
клієнтському коду не важливо, з чим працювати --- зі звичайним об'єктом
даних чи з загорнутим.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// Загальний інтерфейс компонентів.
interface DataSource is
    method writeData(data)
    method readData():data

// Один з конкретних компонентів реалізує базову
// функціональність.
class FileDataSource implements DataSource is
    constructor FileDataSource(filename) { ... }

    method writeData(data) is
        // Записати дані до файлу.

    method readData():data is
        // Прочитати дані з файлу.

// Базовий клас усіх декораторів містить код обгортування.
class DataSourceDecorator implements DataSource is
    protected field wrappee: DataSource

    constructor DataSourceDecorator(source: DataSource) is
        wrappee = source

    method writeData(data) is
        wrappee.writeData(data)

    method readData():data is
        return wrappee.readData()

// Конкретні декоратори додають щось своє до базової поведінки
// обгорнутого компонента.
class EncryptionDecorator extends DataSourceDecorator is
    method writeData(data) is
        // 1. Зашифрувати подані дані.
        // 2. Передати зашифровані дані до методу writeData
        // обгорнутого об&#39;єкта (wrappee).

    method readData():data is
        // 1. Отримати дані з методу readData обгорнутого
        // об&#39;єкта (wrappee).
        // 2. Розшифрувати їх, якщо вони зашифровані.
        // 3. Повернути результат.

// Декорувати можна не тільки базові компоненти, але й вже
// обгорнуті об&#39;єкти.
class CompressionDecorator extends DataSourceDecorator is
    method writeData(data) is
        // 1. Запакувати подані дані.
        // 2. Передати запаковані дані до методу writeData
        // обгорнутого об&#39;єкта (wrappee).

    method readData():data is
        // 1. Отримати дані з методу readData обгорнутого
        // об&#39;єкта (wrappee).
        // 2. Розпакувати їх, якщо вони запаковані.
        // 3. Повернути результат.


// Варіант 1. Простий приклад збирання та використання
// декораторів.
class Application is
    method dumbUsageExample() is
        source = new FileDataSource(&quot;somefile.dat&quot;)
        source.writeData(salaryRecords)
        // До файлу було занесено чисті дані.

        source = new CompressionDecorator(source)
        source.writeData(salaryRecords)
        // До файлу було занесено стислі дані.

        source = new EncryptionDecorator(source)
        // Зараз у source знаходиться зв&#39;язка з трьох об&#39;єктів:
        // Encryption &gt; Compression &gt; FileDataSource

        source.writeData(salaryRecords)
        // До файлу було занесено стислі та зашифровані дані.


// Варіант 2. Клієнтський код, який використовує зовнішнє
// джерело даних. Клас SalaryManager нічого не знає про те, як
// саме буде зчитано та записано дані. Він отримує вже готове
// джерело даних.
class SalaryManager is
    field source: DataSource

    constructor SalaryManager(source: DataSource) { ... }

    method load() is
        return source.readData()

    method save() is
        source.writeData(salaryRecords)
    // ...Інші корисні методи...


// Програма може різним шляхом збирати об&#39;єкти, які декоруються
// залежно від умов використання.
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
##  Застосування {#applicability}

::::::: applicability
::: applicability-problem
Якщо вам потрібно додавати об'єктам нові обов'язки «на льоту», непомітно
для коду, який їх використовує.
:::

::: applicability-solution
Об'єкти вкладаються в обгортки, які мають додаткові поведінки. Обгортки
і самі об'єкти мають однаковий інтерфейс, тому клієнтам не важливо, з
чим працювати --- зі звичайним об'єктом чи з загорнутим.
:::

::: applicability-problem
Якщо не можна розширити обов'язки об'єкта за допомогою спадкування.
:::

::: applicability-solution
У багатьох мовах програмування є ключове слово `final`, яке може
заблокувати спадкування класу. Розширити такі класи можна тільки за
допомогою Декоратора.
:::
:::::::
::::::::

::: {.section .checklist}
##  Кроки реалізації {#checklist}

1.  Переконайтеся, що у вашому завданні присутні основний компонент і
    декілька опціональних доповнень-надбудов над ним.

2.  Створіть інтерфейс компонента, який описував би загальні методи як
    для основного компонента, так і для його доповнень.

3.  Створіть клас конкретного компонента й помістіть в нього основну
    бізнес-логіку.

4.  Створіть базовий клас декораторів. Він повинен мати поле для
    зберігання посилань на вкладений об'єкт-компонент. Усі методи
    базового декоратора повинні делегувати роботу вкладеному об'єкту.

5.  Конкретний компонент, як і базовий декоратор, повинні дотримуватися
    одного і того самого інтерфейсу компонента.

6.  Створіть класи конкретних декораторів, успадковуючи їх від базового
    декоратора. Конкретний декоратор повинен виконувати свою додаткову
    функціональність, а потім (або перед цим) викликати цю ж операцію
    загорнутого об'єкта.

7.  Клієнт бере на себе відповідальність за конфігурацію і порядок
    загортання об'єктів.
:::

:::::: {.section .pros-cons}
##  Переваги та недоліки {#pros-cons}

::::: row
::: col-sm-6
-    Більша гнучкість, ніж у спадкування.
-    Дозволяє додавати обов'язки «на льоту».
-    Можна додавати кілька нових обов'язків одразу.
-    Дозволяє мати кілька дрібних об'єктів, замість одного об'єкта «на
    всі випадки життя».
:::

::: col-sm-6
-    Важко конфігурувати об'єкти, які загорнуто в декілька обгорток
    одночасно.
-    Велика кількість крихітних класів.
:::
:::::
::::::

::: {.section .relations}
##  Відносини з іншими патернами {#relations}

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

-   [Ланцюжок обов'язків](chain-of-responsibility.html) та
    [Декоратор](decorator.html) мають дуже схожі структури. Обидва
    патерни базуються на принципі рекурсивного виконання операції через
    серію пов'язаних об'єктів. Але є декілька важливих відмінностей.

    Обробники в *Ланцюжку обов'язків* можуть виконувати довільні дії,
    незалежні одна від одної, а також у будь-який момент переривати
    подальшу передачу ланцюжком. З іншого боку, *Декоратори* розширюють
    певну дію, не ламаючи інтерфейс базової операції і не перериваючи
    виконання інших декораторів.

-   [Компонувальник](composite.html) та [Декоратор](decorator.html)
    мають схожі структури класів, бо обидва побудовані на рекурсивній
    вкладеності. Вона дозволяє зв'язати в одну структуру нескінченну
    кількість об'єктів.

    *Декоратор* обгортає тільки один об'єкт, а вузол *Компонувальника*
    може мати багато дітей. *Декоратор* додає вкладеному об'єкту нової
    функціональності, а *Компонувальник* не додає нічого нового, але
    «підсумовує» результати всіх своїх дітей.

    Але вони можуть і співпрацювати: *Компонувальник* може
    використовувати *Декоратор*, щоб перевизначити функції окремих
    частин дерева компонентів.

-   Архітектура, побудована на [Компонувальниках](composite.html) та
    [Декораторах](decorator.html), часто може поліпшуватися за рахунок
    впровадження [Прототипу](prototype.html). Він дозволяє клонувати
    складні структури об'єктів, а не збирати їх заново.

-   [Стратегія](strategy.html) змінює поведінку об'єкта «зсередини», а
    [Декоратор](decorator.html) змінює його «ззовні».

-   [Декоратор](decorator.html) та [Замісник](proxy.html) мають схожі
    структури, але різні призначення. Вони схожі тим, що обидва
    побудовані на композиції та делегуванні роботи іншому об'єкту.
    Патерни відрізняються тим, що *Замісник* сам керує життям сервісного
    об'єкта, а обгортання *Декораторів* контролюється клієнтом.
:::

::: {.section .implementations}
##  Приклади реалізації патерна {#implementations}

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

[Фасад []{.fa .fa-arrow-right}](facade.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Повернутись назад

[[]{.fa .fa-arrow-left} Компонувальник](composite.html){.btn
.btn-default rel="prev"}
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
