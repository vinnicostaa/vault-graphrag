:::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [Патерни
проектування](../design-patterns.html){.type} / [Породжувальні
патерни](creational-patterns.html){.type}
:::

# Будівельник {#будівельник .title}

::: aka
Також відомий як: [Builder]{style="display:inline-block"}
:::

::: {.section .intent}
##  Суть патерна {#intent}

**Будівельник** --- це породжувальний патерн проектування, що дає змогу
створювати складні об'єкти крок за кроком. Будівельник дає можливість
використовувати один і той самий код будівництва для отримання різних
відображень об'єктів.

<figure class="image">
<img
src="../../images/patterns/content/builder/builder-uk29ac.png?id=6a76149a5bcedf5b17ec83285d9dd9a7"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/builder/builder-uk.png?id=6a76149a5bcedf5b17ec83285d9dd9a7 640w,/images/patterns/content/builder/builder-uk-1.5x.png?id=1c0926912282aa5e2fb9a03a918aab9e 960w,/images/patterns/content/builder/builder-uk-2x.png?id=fac1ebbc1c71096eac43080843f0afb7 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Патерн Будівельник" />
</figure>
:::

::: {.section .problem}
##  Проблема {#problem}

Уявіть складний об'єкт, що вимагає кропіткої покрокової ініціалізації
безлічі полів і вкладених об'єктів. Код ініціалізації таких об'єктів
зазвичай захований всередині монстроподібного конструктора з десятком
параметрів. Або ще гірше --- розпорошений по всьому клієнтському коду.

<figure class="image">
<img
src="../../images/patterns/diagrams/builder/problem16274.png?id=11e715c5c97811f848c48e0f399bb05e"
style="aspect-ratio: 1.71;"
srcset="/images/patterns/diagrams/builder/problem1.png?id=11e715c5c97811f848c48e0f399bb05e 600w,/images/patterns/diagrams/builder/problem1-1.5x.png?id=a46778018c4f0f4bbf2357940c1f5f40 900w,/images/patterns/diagrams/builder/problem1-2x.png?id=02ffbd7ad294600e42aa78989d441b4d 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Проблема з великою кількістю класів" />
<figcaption><p>Створивши купу підкласів для всіх конфігурацій об’єктів,
ви можете надміру ускладнити програму.</p></figcaption>
</figure>

Наприклад, подумаймо про те, як створити об'єкт `Будинок`. Щоб
побудувати стандартний будинок, потрібно: звести 4 стіни, встановити
двері, вставити пару вікон та постелити дах. Але що робити, якщо ви
хочете більший та світліший будинок, що має басейн, сад та інше добро?

Найпростіше рішення --- розширити клас `Будинок`, створивши підкласи для
всіх комбінацій параметрів будинку. Проблема такого підходу ---
величезна кількість класів, які вам доведеться створити. Кожен новий
параметр, на кшталт кольору шпалер чи матеріалу покрівлі, змусить вас
створювати все більше й більше класів для перерахування усіх можливих
варіантів.

Аби не плодити підкласи, можна підійти до вирішення питання з іншого
боку. Ви можете створити гігантський конструктор `Будинку`, що приймає
безліч параметрів для контролю над створюваним продуктом. Так, це
позбавить вас від підкласів, але призведе до появи іншої проблеми.

<figure class="image">
<img
src="../../images/patterns/diagrams/builder/problem27ab5.png?id=2e91039b6c7d2d2df6ee519983a3b036"
style="aspect-ratio: 1.71;"
srcset="/images/patterns/diagrams/builder/problem2.png?id=2e91039b6c7d2d2df6ee519983a3b036 600w,/images/patterns/diagrams/builder/problem2-1.5x.png?id=3d302cf2762fd04d70cbb91cb54e923c 900w,/images/patterns/diagrams/builder/problem2-2x.png?id=5e7975a91c0e4f4ba960f908cc9c2ea2 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Телескопічний конструктор" />
<figcaption><p>Конструктор з безліччю параметрів має свій недолік: не
всі параметри потрібні протягом більшої частини часу.</p></figcaption>
</figure>

Більшість цих параметрів буде простоювати, а виклики конструктора будуть
виглядати монстроподібно через [довгий список
параметрів](../smells/long-parameter-list.html). Наприклад, басейн є
далеко не в кожному будинку, тому параметри, пов'язані з басейнами,
даремно простоюватимуть у 99% випадків.
:::

::: {.section .solution}
##  Рішення {#solution}

Патерн Будівельник пропонує винести конструювання об'єкта за межі його
власного класу, доручивши цю справу окремим об'єктам, які називаються
*будівельниками*.

<figure class="image">
<img
src="../../images/patterns/diagrams/builder/solution13e5f.png?id=8ce82137f8935998de802cae59e00e11"
style="aspect-ratio: 1.46;"
srcset="/images/patterns/diagrams/builder/solution1.png?id=8ce82137f8935998de802cae59e00e11 410w,/images/patterns/diagrams/builder/solution1-1.5x.png?id=8ab77eb73760a75c35940bae243d43f2 615w,/images/patterns/diagrams/builder/solution1-2x.png?id=a9c2ab02f0b2aca1a7512022194dd113 820w"
sizes="(max-width: 720px) 100vw, 410px" loading="lazy" width="410"
alt="Застосування патерна Будівельник" />
<figcaption><p>Будівельник дозволяє створювати складні об’єкти
покроково. Проміжний результат захищений від
стороннього втручання.</p></figcaption>
</figure>

Патерн пропонує розбити процес конструювання об'єкта на окремі кроки
(наприклад, `побудуватиСтіни`, `встановитиДвері` і т. д.) Щоб створити
об'єкт, вам потрібно по черзі викликати методи будівельника. До того ж
не потрібно викликати всі кроки, а лише ті, що необхідні для виробництва
об'єкта певної конфігурації.

Зазвичай один і той самий крок будівництва може відрізнятися для різних
варіацій виготовлених об'єктів. Наприклад, дерев'яний будинок потребує
будівництва стін з дерева, а кам'яний --- з каменю.

У цьому випадку ви можете створити кілька класів будівельників, які
по-різному виконуватимуть ті ж самі кроки. Використовуючи цих
будівельників в одному й тому самому будівельному процесі, ви зможете
отримувати на виході різні об'єкти.

<figure class="image">
<img
src="../../images/patterns/content/builder/builder-comic-1-uk0a08.png?id=eb3ecd599fb96d861b82c963f1840f33"
style="aspect-ratio: 2;"
srcset="/images/patterns/content/builder/builder-comic-1-uk.png?id=eb3ecd599fb96d861b82c963f1840f33 600w,/images/patterns/content/builder/builder-comic-1-uk-1.5x.png?id=6a10ac77802b74fff768798035971d8f 900w,/images/patterns/content/builder/builder-comic-1-uk-2x.png?id=61ce39b35f66129b860a4b762d8ab510 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600" />
<figcaption><p>Різні будівельники виконають одне і те саме
завдання по-різному.</p></figcaption>
</figure>

Наприклад, один будівельник робить стіни з дерева і скла, інший --- з
каменю і заліза, третій --- із золота та діамантів. Викликавши одні й ті
самі кроки будівництва, у першому випадку ви отримаєте звичайний
житловий будинок, у другому --- маленьку фортецю, а в третьому ---
розкішне житло. Зауважу, що код, який викликає кроки будівництва,
повинен працювати з будівельниками через загальний інтерфейс, щоб їх
можна було вільно замінювати один на інший.

#### Директор

Ви можете піти далі та виділити виклики методів будівельника в окремий
клас, що називається «Директором». У цьому випадку директор задаватиме
порядок кроків будівництва, а будівельник --- виконуватиме їх.

<figure class="image">
<img
src="../../images/patterns/content/builder/builder-comic-2-uk34ed.png?id=6c326c42f218253292a7a5c1b15d47b9"
style="aspect-ratio: 1.14;"
srcset="/images/patterns/content/builder/builder-comic-2-uk.png?id=6c326c42f218253292a7a5c1b15d47b9 343w,/images/patterns/content/builder/builder-comic-2-uk-1.5x.png?id=5b2437b7b69f27ca70d7205620da1031 515w,/images/patterns/content/builder/builder-comic-2-uk-2x.png?id=2aedbe36c862bbaa788dd87cb6dfbe85 687w"
sizes="(max-width: 720px) 100vw, 343px" loading="lazy" width="343" />
<figcaption><p>Директор знає, які кроки повинен виконати
об’єкт-будівельник, щоб виготовити продукт.</p></figcaption>
</figure>

Окремий клас *директора* не є суворо обов'язковим. Ви можете викликати
методи будівельника і безпосередньо з клієнтського коду. Тим не менш,
директор корисний, якщо у вас є кілька способів конструювання продуктів,
що відрізняються порядком і наявними кроками конструювання. У цьому
випадку ви зможете об'єднати всю цю логіку в одному класі.

Така структура класів повністю приховає від клієнтського коду процес
конструювання об'єктів. Клієнту залишиться лише прив'язати бажаного
будівельника до директора, а потім отримати від будівельника готовий
результат.
:::

::::: {.section .structure-container}
##  Структура {#structure}

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/builder/structure05e4.png?id=fe9e23559923ea0657aa5fe75efef333"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 0.79;"
srcset="/images/patterns/diagrams/builder/structure.png?id=fe9e23559923ea0657aa5fe75efef333 460w,/images/patterns/diagrams/builder/structure-1.5x.png?id=91ea8cd3137b403542c23abf63938f00 690w,/images/patterns/diagrams/builder/structure-2x.png?id=dca1b1508e23c266cbedc80ffb84311a 920w"
sizes="(max-width: 720px) 100vw, 460px" loading="lazy" width="460"
alt="Структура класів патерна Будівельник" /><img
src="../../images/patterns/diagrams/builder/structure-indexedadac.png?id=44b3d763ce91dbada5d8394ef777437f"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 0.81;"
srcset="/images/patterns/diagrams/builder/structure-indexed.png?id=44b3d763ce91dbada5d8394ef777437f 470w,/images/patterns/diagrams/builder/structure-indexed-1.5x.png?id=d57a6ff9342ea31736ea98e5066e0748 705w,/images/patterns/diagrams/builder/structure-indexed-2x.png?id=153eed9a51784cbe00d0ca8b3d6b268d 940w"
sizes="(max-width: 720px) 100vw, 470px" loading="lazy" width="470"
alt="Структура класів патерна Будівельник" />
</figure>
:::

1.  **Інтерфейс будівельника** оголошує кроки конструювання продуктів,
    спільні для всіх видів будівельників.

2.  **Конкретні будівельники** реалізують кроки будівництва, кожен
    по-своєму. Конкретні будівельники можуть виготовляти різнорідні
    об'єкти, що не мають спільного інтерфейсу.

3.  **Продукт** --- об'єкт, що створюється. Продукти, зроблені різними
    будівельниками, не зобов'язані мати спільний інтерфейс.

4.  **Директор** визначає порядок виклику кроків будівельників,
    необхідних для виробництва продуктів тієї чи іншої конфігурації.

5.  Зазвичай **Клієнт** подає до конструктора директора вже готовий
    об'єкт-будівельник, а директор надалі використовує тільки його. Але
    можливим є також інший варіант, коли клієнт передає будівельника
    через параметр будівельного методу директора. У такому випадку можна
    щоразу використовувати різних будівельників для виробництва
    різноманітних відображень об'єктів.
::::
:::::

::: {.section .pseudocode}
##  Псевдокод {#pseudocode}

У цьому прикладі **Будівельник** використовується для покрокового
конструювання автомобілів та технічних посібників до них.

<figure class="image">
<img
src="../../images/patterns/diagrams/builder/example-ukafbf.png?id=b21b5480cf64e05b7d3db50c24ed2f11"
style="aspect-ratio: 0.85;"
srcset="/images/patterns/diagrams/builder/example-uk.png?id=b21b5480cf64e05b7d3db50c24ed2f11 500w,/images/patterns/diagrams/builder/example-uk-1.5x.png?id=dd9d04a013865e85eb28e7a2ad93ea35 750w,/images/patterns/diagrams/builder/example-uk-2x.png?id=04a55ab57e514392fa744c3be3f52290 1000w"
sizes="(max-width: 720px) 100vw, 500px" loading="lazy" width="500"
alt="Структура класів прикладу патерна Будівельник" />
<figcaption><p>Приклад покрокового конструювання автомобілів та
інструкцій до них.</p></figcaption>
</figure>

Автомобіль --- це складний об'єкт, який можна налаштувати сотнею різних
способів. Замість того, щоб налаштовувати автомобіль через конструктор,
ми винесемо його збирання в окремий клас-будівельник, передбачивши
методи для конфігурації всіх частин автомобіля.

Клієнт може збирати автомобілі, працюючи з будівельником безпосередньо.
З іншого боку, він може доручити цю справу директору. Це об'єкт, який
знає, які кроки будівельника потрібно викликати, щоб отримати кілька
найпопулярніших конфігурацій автомобілів.

Проте, до кожного автомобіля ще потрібен посібник користувача, що
відповідає його конфігурації. Для цього ми створимо ще один клас
будівельника, який замість конструювання автомобіля друкуватиме сторінки
посібника до тієї деталі, яку ми вбудовуємо в продукт. Тепер,
пропустивши через одні й ті самі кроки обидва типи будівельників, ми
отримаємо автомобіль та відповідний до нього посібник користувача.

Очевидно, що паперовий посібник і металевий автомобіль --- це дві
абсолютно різні речі. З цієї причини ми повинні отримувати результат
безпосередньо від будівельників, а не від директора. Інакше нам довелося
б жорстко прив'язати директора до конкретних класів автомобілів і
посібників.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// Будівельник може створювати різні продукти, використовуючи
// один і той самий процес будівництва.
class Car is
    // Автомобілі можуть відрізнятися комплектацією: типом
    // двигуна, кількістю сидінь, можуть мати або не мати GPS і
    // систему навігації тощо. Крім того, автомобілі можуть бути
    // міськими, спортивними або позашляховиками.

class Manual is
    // Посібник користувача для даної конфігурації автомобіля.


// Інтерфейс будівельників оголошує всі можливі етапи та кроки
// конфігурації продукту.
interface Builder is
    method reset()
    method setSeats(...)
    method setEngine(...)
    method setTripComputer(...)
    method setGPS(...)

// Усі конкретні будівельники реалізують загальний інтерфейс по-
// своєму.
class CarBuilder implements Builder is
    private field car:Car
    method reset()
        // Помістити новий об&#39;єкт Car в полі &quot;car&quot;.
    method setSeats(...) is
        // Встановити вказану кількість сидінь.
    method setEngine(...) is
        // Встановити наданий двигун.
    method setTripComputer(...) is
        // Встановити надану систему навігації.
    method setGPS(...) is
        // Встановити або зняти GPS.
    method getResult(): Car is
        // Повернути поточний об&#39;єкт автомобіля.

// На відміну від інших породжувальних патернів, де продукти
// мають бути частиною одніє ієрархії класів або слідувати
// загальному інтерфейсу, будівельники можуть створювати
// абсолютно різні продукти, які не мають спільного предка.
class CarManualBuilder implements Builder is
    private field manual:Manual
    method reset()
        // Помістити новий об&#39;єкт Manual у полі &quot;manual&quot;.
    method setSeats(...) is
        // Описати кількість місць в автівці.
    method setEngine(...) is
        // Додати до посібника опис двигуна.
    method setTripComputer(...) is
        // Додати до посібника опис системи навігації.
    method setGPS(...) is
        // Додати до посібника інструкцію для GPS.
    method getResult(): Manual is
        // Повернути поточний об&#39;єкт посібника.


// Директор знає, в якій послідовності потрібно змушувати
// працювати будівельника, щоб отримати ту чи іншу версію
// продукту. Зауважте, що директор працює з будівельником через
// загальний інтерфейс, завдяки чому він не знає тип продукту,
// який виготовляє будівельник.
class Director is
    method constructSportsCar(builder: Builder) is
        builder.reset()
        builder.setSeats(2)
        builder.setEngine(new SportEngine())
        builder.setTripComputer(true)
        builder.setGPS(true)


// Директор отримує об&#39;єкт конкретного будівельника від клієнта
// (програми). Програма сама знає, якого будівельника
// використати, аби отримати потрібний продукт.
class Application is
    method makeCar() is
        director = new Director()

        CarBuilder builder = new CarBuilder()
        director.constructSportsCar(builder)
        Car car = builder.getResult()

        CarManualBuilder builder = new CarManualBuilder()
        director.constructSportsCar(builder)

        // Готовий продукт повертає будівельник, оскільки
        // директор частіше за все не знає і не залежить від
        // конкретних класів будівельників та продуктів.
        Manual manual = builder.getResult()</code></pre>
</figure>
:::

:::::::::: {.section .applicability-container}
##  Застосування {#applicability}

::::::::: applicability
::: applicability-problem
Коли ви хочете позбутися від «телескопічного конструктора».
:::

::: applicability-solution
Припустімо, у вас є один конструктор з десятьма опціональними
параметрами. Його незручно викликати, тому ви створили ще десять
конструкторів з меншою кількістю параметрів. Все, що вони роблять, ---
це переадресовують виклик до базового конструктора, подаючи якісь типові
значення в параметри, які відсутні в них самих.

<figure class="code">
<pre class="code" lang="java"><code>class Pizza {
    Pizza(int size) { ... }
    Pizza(int size, boolean cheese) { ... }
    Pizza(int size, boolean cheese, boolean pepperoni) { ... }
    // ...</code></pre>
<figcaption><p>Такого монстра можна створити тільки в мовах, що мають
механізм перевантаження методів, наприклад, C#
або Java.</p></figcaption>
</figure>

Патерн Будівельник дозволяє збирати об'єкти покроково, викликаючи тільки
ті кроки, які вам потрібні. Отже, більше не потрібно намагатися
«запхати» до конструктора всі можливі опції продукту.
:::

::: applicability-problem
Коли ваш код повинен створювати різні уявлення якогось об'єкта.
Наприклад, дерев'яні та залізобетонні будинки.
:::

::: applicability-solution
Будівельник можна застосувати, якщо створення кількох відображень
об'єкта складається з однакових етапів, які відрізняються деталями.

Інтерфейс будівельників визначить всі можливі етапи конструювання.
Кожному відображенню відповідатиме власний клас-будівельник. Порядок
етапів будівництва визначатиме клас-директор.
:::

::: applicability-problem
Коли вам потрібно збирати складні об'єкти, наприклад, дерева
[Компонувальника](composite.html).
:::

::: applicability-solution
Будівельник конструює об'єкти покроково, а не за один прохід. Більш
того, кроки будівництва можна виконувати рекурсивно. А без цього не
побудувати деревоподібну структуру на зразок
[Компонувальника](composite.html).

Зауважте, що Будівельник не дозволяє стороннім об'єктам отримувати
доступ до об'єкта, що конструюється, доки той не буде повністю готовий.
Це захищає клієнтський код від отримання незавершених «битих» об'єктів.
:::
:::::::::
::::::::::

::: {.section .checklist}
##  Кроки реалізації {#checklist}

1.  Переконайтеся в тому, що створення різних відображень об'єкта можна
    звести до загальних кроків.

2.  Опишіть ці кроки в загальному інтерфейсі будівельників.

3.  Для кожного з відображень об'єкта-продукту створіть по одному
    класу-будівельнику й реалізуйте їхні методи будівництва.

    Не забудьте про метод отримання результату. Зазвичай конкретні
    будівельники визначають власні методи отримання результату
    будівництва. Ви не можете описати ці методи в інтерфейсі
    будівельників, оскільки продукти не обов'язково повинні мати
    загальний базовий клас або інтерфейс. Але ви завжди можете додати
    метод отримання результату до загального інтерфейсу, якщо ваші
    будівельники виготовляють однорідні продукти, які мають спільного
    предка.

4.  Подумайте про створення класу директора. Його методи створюватимуть
    різні конфігурації продуктів, викликаючи різні кроки одного і того
    самого будівельника.

5.  Клієнтський код повинен буде створювати й об'єкти будівельників, й
    об'єкт директора. Перед початком будівництва клієнт повинен зв'язати
    певного будівельника з директором. Це можна зробити або через
    конструктор, або через сетер, або подавши будівельника безпосередньо
    до будівельного методу директора.

6.  Результат будівництва можна повернути з директора, але тільки якщо
    метод повернення продукту вдалося розмістити в загальному інтерфейсі
    будівельників. Інакше ви жорстко прив'яжете директора до конкретних
    класів будівельників.
:::

:::::: {.section .pros-cons}
##  Переваги та недоліки {#pros-cons}

::::: row
::: col-sm-6
-    Дозволяє створювати продукти покроково.
-    Дозволяє використовувати один і той самий код для створення
    різноманітних продуктів.
-    Ізолює складний код конструювання продукту від його головної
    бізнес-логіки.
:::

::: col-sm-6
-    Ускладнює код програми за рахунок додаткових класів.
-    Клієнт буде прив'язаний до конкретних класів будівельників, тому що
    в інтерфейсі будівельника може не бути методу отримання результату.
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

-   [Будівельник](builder.html) концентрується на будівництві складних
    об'єктів крок за кроком. [Абстрактна фабрика](abstract-factory.html)
    спеціалізується на створенні сімейств пов'язаних продуктів.
    *Будівельник* повертає продукт тільки після виконання всіх кроків, а
    *Абстрактна фабрика* повертає продукт одразу.

-   [Будівельник](builder.html) дозволяє покроково конструювати дерево
    [Компонувальника](composite.html).

-   Патерн [Будівельник](builder.html) може бути побудований у вигляді
    [Мосту](bridge.html): *директор* гратиме роль абстракції, а
    *будівельники* --- реалізації.

-   [Абстрактна фабрика](abstract-factory.html),
    [Будівельник](builder.html) та [Прототип](prototype.html) можуть
    реалізовуватися за допомогою [Одинака](singleton.html).
:::

::: {.section .implementations}
##  Приклади реалізації патерна {#implementations}

[![Будівельник на
C#](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](builder/csharp/example.html "Будівельник на C#"){.prog-lang-link}
[![Будівельник на
C++](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](builder/cpp/example.html "Будівельник на C++"){.prog-lang-link}
[![Будівельник на
Go](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](builder/go/example.html "Будівельник на Go"){.prog-lang-link}
[![Будівельник на
Java](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](builder/java/example.html "Будівельник на Java"){.prog-lang-link}
[![Будівельник на
PHP](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](builder/php/example.html "Будівельник на PHP"){.prog-lang-link}
[![Будівельник на
Python](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](builder/python/example.html "Будівельник на Python"){.prog-lang-link}
[![Будівельник на
Ruby](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](builder/ruby/example.html "Будівельник на Ruby"){.prog-lang-link}
[![Будівельник на
Rust](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](builder/rust/example.html "Будівельник на Rust"){.prog-lang-link}
[![Будівельник на
Swift](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](builder/swift/example.html "Будівельник на Swift"){.prog-lang-link}
[![Будівельник на
TypeScript](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](builder/typescript/example.html "Будівельник на TypeScript"){.prog-lang-link}
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

[Прототип []{.fa .fa-arrow-right}](prototype.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Повернутись назад

[[]{.fa .fa-arrow-left} Порівняння фабрик](factory-comparison.html){.btn
.btn-default rel="prev"}
:::
:::::::::::::::::::::::::::::::::

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
:::::::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::::::
