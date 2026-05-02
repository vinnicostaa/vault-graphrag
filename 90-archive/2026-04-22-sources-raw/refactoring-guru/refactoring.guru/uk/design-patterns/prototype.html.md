:::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [Патерни
проектування](../design-patterns.html){.type} / [Породжувальні
патерни](creational-patterns.html){.type}
:::

# Прототип {#прототип .title}

::: aka
Також відомий як:
[Клон, ]{style="display:inline-block"}[Prototype]{style="display:inline-block"}
:::

::: {.section .intent}
##  Суть патерна {#intent}

**Прототип** --- це породжувальний патерн проектування, що дає змогу
копіювати об'єкти, не вдаючись у подробиці їхньої реалізації.

<figure class="image">
<img
src="../../images/patterns/content/prototype/prototype3cb9.png?id=e912b1ada20bbf7b2ffc09e93b9fab20"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/prototype/prototype.png?id=e912b1ada20bbf7b2ffc09e93b9fab20 640w,/images/patterns/content/prototype/prototype-1.5x.png?id=a0f76894fb657783b65b9ed899857468 960w,/images/patterns/content/prototype/prototype-2x.png?id=670789c80c8a114e25838ede2da4a881 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Патерн Прототип" />
</figure>
:::

::: {.section .problem}
##  Проблема {#problem}

У вас є об'єкт, який потрібно скопіювати. Як це зробити? Потрібно
створити порожній об'єкт того самого класу, а потім по черзі копіювати
значення всіх полів зі старого об'єкта до нового.

Чудово! Проте є нюанс. Не кожен об'єкт вдасться скопіювати у такий
спосіб, адже частина його стану може бути приватною, а значить ---
недоступною для решти коду програми.

<figure class="image">
<img
src="../../images/patterns/content/prototype/prototype-comic-1-ukf5e5.png?id=876deb7c8bc04b34dc9c8828b4e908fa"
style="aspect-ratio: 2;"
srcset="/images/patterns/content/prototype/prototype-comic-1-uk.png?id=876deb7c8bc04b34dc9c8828b4e908fa 600w,/images/patterns/content/prototype/prototype-comic-1-uk-1.5x.png?id=ffa2b7db7dd13584c1d30a8ea4bb8bbb 900w,/images/patterns/content/prototype/prototype-comic-1-uk-2x.png?id=1283883c3a7b03674a043ff87383010a 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Приклад невдалого копіювання ззовні" />
<figcaption><p>Копіювання «ззовні» <a
href="https://vimeo.com/120816425">не завжди</a> можливе
на практиці.</p></figcaption>
</figure>

Є й інша проблема. Код, що копіює, стане залежним від класів об'єктів,
які він копіює. Адже, щоб перебрати *усі* поля об'єкта, потрібно
прив'язатися до його класу. Тому ви не зможете копіювати об'єкти, знаючи
тільки їхні інтерфейси, але не їхні конкретні класи.
:::

::: {.section .solution}
##  Рішення {#solution}

Патерн Прототип доручає процес копіювання самим об'єктам, які треба
скопіювати. Він вводить загальний інтерфейс для всіх об'єктів, що
підтримують клонування. Це дозволяє копіювати об'єкти, не прив'язуючись
до їхніх конкретних класів. Зазвичай такий інтерфейс має всього один
метод --- `clone`.

Реалізація цього методу в різних класах дуже схожа. Метод створює новий
об'єкт поточного класу й копіює в нього значення всіх полів власного
об'єкта. Таким чином можна скопіювати навіть приватні поля, оскільки
більшість мов програмування дозволяє отримати доступ до приватних полів
будь-якого об'єкта поточного класу.

Об'єкт, який копіюють, називається *прототипом* (звідси і назва
патерна). Коли об'єкти програми містять сотні полів і тисячі можливих
конфігурацій, прототипи можуть слугувати своєрідною альтернативою
створенню підкласів.

<figure class="image">
<img
src="../../images/patterns/content/prototype/prototype-comic-2-uk2edc.png?id=ccaf60940c3a7d8121e4e1fe8e14018b"
style="aspect-ratio: 1.14;"
srcset="/images/patterns/content/prototype/prototype-comic-2-uk.png?id=ccaf60940c3a7d8121e4e1fe8e14018b 343w,/images/patterns/content/prototype/prototype-comic-2-uk-1.5x.png?id=87f8bcd1755b7692d0fbe536ec99b136 515w,/images/patterns/content/prototype/prototype-comic-2-uk-2x.png?id=de654f6129e988289e0125677b384bd5 687w"
sizes="(max-width: 720px) 100vw, 343px" loading="lazy" width="343"
alt="Попередньо заготовлені прототипи" />
<figcaption><p>Попередньо заготовлені прототипи можуть стати
заміною підкласів.</p></figcaption>
</figure>

У цьому випадку всі можливі прототипи готуються і налаштовуються на
етапі ініціалізації програми. Потім, коли програмі буде потрібний новий
об'єкт, вона створить копію з попередньо заготовленого прототипа.
:::

::: {.section .analogy}
##  Аналогія з життя {#analogy}

У промисловому виробництві прототипи створюються перед виготовленням
основної партії продуктів для проведення різноманітних випробувань. При
цьому прототип не бере участі в подальшому виробництві, відіграючи
пасивну роль.

<figure class="image">
<img
src="../../images/patterns/content/prototype/prototype-comic-3-uk07fe.png?id=d4a38ea25b36305c88c50a406b5cd162"
style="aspect-ratio: 2;"
srcset="/images/patterns/content/prototype/prototype-comic-3-uk.png?id=d4a38ea25b36305c88c50a406b5cd162 600w,/images/patterns/content/prototype/prototype-comic-3-uk-1.5x.png?id=0aba7f9d595be55982ade220a72d165d 900w,/images/patterns/content/prototype/prototype-comic-3-uk-2x.png?id=ced5a51a56224dbddfd877c4495bb2f9 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Приклад поділу клітини" />
<figcaption><p>Приклад поділу клітини.</p></figcaption>
</figure>

Виробничий прототип не створює копію самого себе, тому більш наближений
до патерна приклад --- це поділ клітин. Після мітозного поділу клітин
утворюються дві абсолютно ідентичні клітини. Материнська клітина
відіграє роль прототипу, беручи активну участь у створенні нового
об'єкта.
:::

:::::::: {.section .structure-container}
##  Структура {#structure}

::::::: structure
#### Базова реалізація

:::: prototype-normal
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/prototype/structure87f6.png?id=088102c5e9785ff45debbbce86f4df81"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.25;"
srcset="/images/patterns/diagrams/prototype/structure.png?id=088102c5e9785ff45debbbce86f4df81 500w,/images/patterns/diagrams/prototype/structure-1.5x.png?id=beec6a1a5242268e10e39f9fdc0b894b 750w,/images/patterns/diagrams/prototype/structure-2x.png?id=ba75079f42f08028ae4cdbda0cfecc26 1000w"
sizes="(max-width: 720px) 100vw, 500px" loading="lazy" width="500"
alt="Структура класів патерна Прототип" /><img
src="../../images/patterns/diagrams/prototype/structure-indexed6499.png?id=0e1c809842f5c43aca0541a2eba1f844"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.27;"
srcset="/images/patterns/diagrams/prototype/structure-indexed.png?id=0e1c809842f5c43aca0541a2eba1f844 520w,/images/patterns/diagrams/prototype/structure-indexed-1.5x.png?id=05b072b5b83f5ff1024a7ba448ea9d59 780w,/images/patterns/diagrams/prototype/structure-indexed-2x.png?id=74584ac729c0f6b49d2a97a53cc266ff 1040w"
sizes="(max-width: 720px) 100vw, 520px" loading="lazy" width="520"
alt="Структура класів патерна Прототип" />
</figure>
:::
::::

1.  **Інтерфейс прототипів** описує операції клонування. Для більшості
    випадків --- це єдиний метод `clone`.

2.  **Конкретний прототип** реалізує операцію клонування самого себе.
    Крім звичайного копіювання значень усіх полів, тут можуть бути
    приховані різноманітні складнощі, про які клієнту не потрібно знати.
    Наприклад, клонування пов'язаних об'єктів, розплутування рекурсивних
    залежностей та інше.

3.  **Клієнт** створює копію об'єкта, звертаючись до нього через
    загальний інтерфейс прототипів.

#### Реалізація зі спільним сховищем прототипів

:::: prototype-pool
::: {.struct-image2 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/prototype/structure-prototype-cache9d19.png?id=609c2af5d14ed55dcbb218a00f98e7d5"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.15;"
srcset="/images/patterns/diagrams/prototype/structure-prototype-cache.png?id=609c2af5d14ed55dcbb218a00f98e7d5 550w,/images/patterns/diagrams/prototype/structure-prototype-cache-1.5x.png?id=8ca0b829185d49c5acbe19966a0659d4 825w,/images/patterns/diagrams/prototype/structure-prototype-cache-2x.png?id=a1e4514bbcc9b10968b856f19b407105 1100w"
sizes="(max-width: 720px) 100vw, 550px" loading="lazy" width="550"
alt="Варіант Прототипу зі спільним сховищем прототипів" /><img
src="../../images/patterns/diagrams/prototype/structure-prototype-cache-indexed1801.png?id=10a4a84a1a318f59dbc2b806fc936d04"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.12;"
srcset="/images/patterns/diagrams/prototype/structure-prototype-cache-indexed.png?id=10a4a84a1a318f59dbc2b806fc936d04 550w,/images/patterns/diagrams/prototype/structure-prototype-cache-indexed-1.5x.png?id=cb56c95533a4020368c48db9f9e2a37d 825w,/images/patterns/diagrams/prototype/structure-prototype-cache-indexed-2x.png?id=47b99eb7ae51158bdbb31deea4f5e98f 1100w"
sizes="(max-width: 720px) 100vw, 550px" loading="lazy" width="550"
alt="Варіант Прототипу зі спільним сховищем прототипів" />
</figure>
:::
::::

1.  **Сховище прототипів** полегшує доступ до часто використовуваних
    прототипів, зберігаючи попередньо створений набір еталонних, готових
    до копіювання об'єктів. Найпростіше сховище може бути побудовано за
    допомогою хеш-таблиці виду `ім'я-прототипу` → `прототип`. Для
    полегшення пошуку прототипи можна маркувати ще й за іншими
    критеріями, а не тільки за умовним іменем.
:::::::
::::::::

::: {.section .pseudocode}
##  Псевдокод {#pseudocode}

У цьому прикладі **Прототип** дозволяє робити точні копії об'єктів
геометричних фігур без прив'язки до їхніх класів.

<figure class="image">
<img
src="../../images/patterns/diagrams/prototype/examplee491.png?id=47bc6c1058cb100b81e675b5ca6bda6c"
style="aspect-ratio: 1.42;"
srcset="/images/patterns/diagrams/prototype/example.png?id=47bc6c1058cb100b81e675b5ca6bda6c 470w,/images/patterns/diagrams/prototype/example-1.5x.png?id=067f3a2415370fadf5f92aadaa12b5e2 705w,/images/patterns/diagrams/prototype/example-2x.png?id=80393e0df17ae8130e5ada832d494949 940w"
sizes="(max-width: 720px) 100vw, 470px" loading="lazy" width="470"
alt="Структура класів прикладу патерна Прототип" />
<figcaption><p>Приклад клонування ієрархії
геометричних фігур.</p></figcaption>
</figure>

Кожна фігура реалізує інтерфейс клонування і надає метод для відтворення
самої себе. Підкласи використовують батьківський метод клонування, а
потім копіюють власні поля до створеного об'єкта.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// Базовий прототип.
abstract class Shape is
    field X: int
    field Y: int
    field color: string

    // Звичайний конструктор.
    constructor Shape() is
        // ...

    // Конструктор прототипа.
    constructor Shape(source: Shape) is
        this()
        this.X = source.X
        this.Y = source.Y
        this.color = source.color

    // Результатом операції клонування завжди буде об&#39;єкт з
    // ієрархії класів Shape.
    abstract method clone(): Shape


// Конкретний прототип. Метод клонування створює новий об&#39;єкт
// поточного класу, передаючи до конструктора посилання на
// власний об&#39;єкт. Завдяки цьому, клонування виходить
// атомарним — доки не виконається конструктор, нового об&#39;єкта
// ще не існує. Але як тільки конструктор завершено, ми
// отримаємо завершений об&#39;єкт-клон, а не порожній об&#39;єкт, який
// потрібно ще заповнити.
class Rectangle extends Shape is
    field width: int
    field height: int

    constructor Rectangle(source: Rectangle) is
        // Виклик батьківського конструктора потрібен, щоб
        // скопіювати потенційні приватні поля, оголошені в
        // батьківському класі.
        super(source)
        this.width = source.width
        this.height = source.height

    method clone(): Shape is
        return new Rectangle(this)


class Circle extends Shape is
    field radius: int

    constructor Circle(source: Circle) is
        super(source)
        this.radius = source.radius

    method clone(): Shape is
        return new Circle(this)


// Десь у клієнтському програмному коді.
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
        // anotherCircle буде містити точну копію circle.

        Rectangle rectangle = new Rectangle()
        rectangle.width = 10
        rectangle.height = 20
        shapes.add(rectangle)

    method businessLogic() is
        // Неочевидний плюс Прототипу в тому, що ви можете
        // клонувати набір об&#39;єктів, не знаючи їхніх конкретних
        // класів.
        Array shapesCopy = new Array of Shapes.

        // Наприклад, ми не знаємо, які конкретно об&#39;єкти
        // знаходяться всередині масиву shapes так як його
        // оголошено з типом Shape. Але завдяки поліморфізму, ми
        // можемо клонувати усі об&#39;єкти «наосліп». Буде виконано
        // метод clone того класу, яким є цей об&#39;єкт.
        foreach (s in shapes) do
            shapesCopy.add(s.clone())

        // Змінна shapesCopy буде містити точні копії елементів
        // масиву shapes.</code></pre>
</figure>
:::

:::::::: {.section .applicability-container}
##  Застосування {#applicability}

::::::: applicability
::: applicability-problem
Коли ваш код не повинен залежати від класів об'єктів, призначених для
копіювання.
:::

::: applicability-solution
Таке часто буває, якщо ваш код працює з об'єктами, поданими ззовні через
який-небудь загальний інтерфейс. Ви не зможете прив'язатися до їхніх
класів, навіть якби захотіли, тому що конкретні класи об'єктів невідомі.

Патерн Прототип надає клієнту загальний інтерфейс для роботи з усіма
прототипами. Клієнту не потрібно залежати від усіх класів об'єктів,
призначених для копіювання, а тільки від інтерфейсу клонування.
:::

::: applicability-problem
Коли ви маєте безліч підкласів, які відрізняються початковими значеннями
полів. Хтось міг створити усі ці класи для того, щоб мати легкий спосіб
породжувати об'єкти певної конфігурації.
:::

::: applicability-solution
Патерн Прототип пропонує використовувати набір прототипів замість
створення підкласів для опису популярних конфігурацій об'єктів.

Таким чином, замість породження об'єктів з підкласів ви копіюватимете
існуючі об'єкти-прототипи, внутрішній стан яких вже налаштовано. Це
дозволить уникнути вибухоподібного зростання кількості класів програми й
зменшити її складність.
:::
:::::::
::::::::

::: {.section .checklist}
##  Кроки реалізації {#checklist}

1.  Створіть інтерфейс прототипів з єдиним методом `clone`. Якщо у вас
    вже є ієрархія продуктів, метод клонування можна оголосити в кожному
    з її класів.

2.  Додайте до класів майбутніх прототипів альтернативний конструктор,
    що приймає в якості аргументу об'єкт поточного класу. Спочатку цей
    конструктор повинен скопіювати значення всіх полів поданого об'єкта,
    оголошених в рамках поточного класу. Потім --- передати виконання
    батьківському конструктору, щоб той потурбувався про поля, оголошені
    в суперкласі.

    Якщо мова програмування, яку ви використовуєте, не підтримує
    перевантаження методів, тоді вам не вдасться створити декілька
    версій конструктора. В цьому випадку копіювання значень можна
    проводити в іншому методі, спеціально створеному для цих цілей.
    Конструктор є зручнішим, тому що дозволяє клонувати об'єкт за один
    виклик.

3.  Зазвичай метод клонування складається з одного рядка, а саме виклику
    оператора `new` з конструктором прототипу. Усі класи, що підтримують
    клонування, повинні явно визначити метод `clone` для того, щоб
    вказати власний клас з оператором `new`. Інакше результатом
    клонування стане об'єкт батьківського класу.

4.  На додачу можете створити центральне сховище прототипів. У ньому
    зручно зберігати варіації об'єктів, можливо, навіть одного класу,
    але по-різному налаштованих.

    Ви можете розмістити це сховище або у новому фабричному класі, або у
    фабричному методі базового класу прототипів. Такий фабричний метод,
    керуючись вхідними аргументами, повинен шукати відповідний екземпляр
    у сховищі прототипів, а потім викликати його метод клонування і
    повертати отриманий об'єкт.

    Нарешті, потрібно позбутися прямих викликів конструкторів об'єктів,
    замінивши їх викликами фабричного методу сховища прототипів.
:::

:::::: {.section .pros-cons}
##  Переваги та недоліки {#pros-cons}

::::: row
::: col-sm-6
-    Дозволяє клонувати об'єкти без прив'язки до їхніх конкретних
    класів.
-    Менша кількість повторювань коду ініціалізації об'єктів.
-    Прискорює створення об'єктів.
-    Альтернатива створенню підкласів під час конструювання складних
    об'єктів.
:::

::: col-sm-6
-    Складно клонувати складові об'єкти, що мають посилання на інші
    об'єкти.
:::
:::::
::::::

::: {.section .relations}
##  Відносини з іншими патернами {#relations}

-   Багато архітектур починаються із застосування [Фабричного
    методу](factory-method.html) (простішого та більш розширюваного за
    допомогою підкласів) та еволюціонують у бік [Абстрактної
    фабрики](abstract-factory.html), [Прототипу](prototype.html) або
    [Будівельника](builder.html) (гнучкіших, але й складніших).

-   Класи [Абстрактної фабрики](abstract-factory.html) найчастіше
    реалізуються за допомогою [Фабричного методу](factory-method.html),
    хоча вони можуть бути побудовані і на основі
    [Прототипу](prototype.html).

-   Якщо [Команду](command.html) потрібно копіювати перед вставкою в
    історію виконаних команд, вам може допомогти
    [Прототип](prototype.html).

-   Архітектура, побудована на [Компонувальниках](composite.html) та
    [Декораторах](decorator.html), часто може поліпшуватися за рахунок
    впровадження [Прототипу](prototype.html). Він дозволяє клонувати
    складні структури об'єктів, а не збирати їх заново.

-   [Прототип](prototype.html) не спирається на спадкування, але йому
    потрібна складна операція ініціалізації. [Фабричний
    метод](factory-method.html), навпаки, побудований на спадкуванні,
    але не вимагає складної ініціалізації.

-   [Знімок](memento.html) іноді можна замінити
    [Прототипом](prototype.html), якщо об'єкт, чий стан потрібно
    зберігати в історії, досить простий, не має посилань на зовнішні
    ресурси або їх можна легко відновити.

-   [Абстрактна фабрика](abstract-factory.html),
    [Будівельник](builder.html) та [Прототип](prototype.html) можуть
    реалізовуватися за допомогою [Одинака](singleton.html).
:::

::: {.section .implementations}
##  Приклади реалізації патерна {#implementations}

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

[Одинак []{.fa .fa-arrow-right}](singleton.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Повернутись назад

[[]{.fa .fa-arrow-left} Будівельник](builder.html){.btn .btn-default
rel="prev"}
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
