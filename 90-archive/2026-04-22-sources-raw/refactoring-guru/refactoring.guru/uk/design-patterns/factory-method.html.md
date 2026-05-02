::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [Патерни
проектування](../design-patterns.html){.type} / [Породжувальні
патерни](creational-patterns.html){.type}
:::

# Фабричний метод {#фабричний-метод .title}

::: aka
Також відомий як: [Віртуальний
конструктор, ]{style="display:inline-block"}[Factory
Method]{style="display:inline-block"}
:::

::: {.section .intent}
##  Суть патерна {#intent}

**Фабричний метод** --- це породжувальний патерн проектування, який
визначає загальний інтерфейс для створення об'єктів у суперкласі,
дозволяючи підкласам змінювати тип створюваних об'єктів.

<figure class="image">
<img
src="../../images/patterns/content/factory-method/factory-method-uk799d.png?id=71bda7a5380dafddbb39f31d44f05d18"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/factory-method/factory-method-uk.png?id=71bda7a5380dafddbb39f31d44f05d18 640w,/images/patterns/content/factory-method/factory-method-uk-1.5x.png?id=82105d6df2be873f30fd3b4bfc73147c 960w,/images/patterns/content/factory-method/factory-method-uk-2x.png?id=ba2251705324a303eb42e3d222202713 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Патерн Фабричний метод" />
</figure>
:::

::: {.section .problem}
##  Проблема {#problem}

Уявіть, що ви створюєте програму керування вантажними перевезеннями.
Спочатку ви плануєте перевезення товарів тільки вантажними автомобілями.
Тому весь ваш код працює з об'єктами класу `Вантажівка`.

Згодом ваша програма стає настільки відомою, що морські перевізники
шикуються в чергу і благають додати до програми підтримку морської
логістики.

<figure class="image">
<img
src="../../images/patterns/diagrams/factory-method/problem1-en51bc.png?id=de638561be0bbb1025ada6bfcb815def"
style="aspect-ratio: 2.4;"
srcset="/images/patterns/diagrams/factory-method/problem1-en.png?id=de638561be0bbb1025ada6bfcb815def 600w,/images/patterns/diagrams/factory-method/problem1-en-1.5x.png?id=a46ffaf7114d367522cabda6012404bf 900w,/images/patterns/diagrams/factory-method/problem1-en-2x.png?id=9a4959d9dde4edadf809be33d3da0c74 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Проблема з додаванням нового класу до програми" />
<figcaption><p>Додати новий клас не так просто, якщо весь код вже
залежить від конкретних класів.</p></figcaption>
</figure>

Чудові новини, чи не так?! Але як щодо коду? Велика частина існуючого
коду жорстко прив'язана до класів `Вантажівок`. Щоб додати до програми
класи морських `Суден`, знадобиться перелопачувати весь код. Якщо ж ви
вирішите додати до програми ще один вид транспорту, тоді всю цю роботу
доведеться повторити.

У підсумку ви отримаєте жахливий код, переповнений умовними операторами,
що виконують ту чи іншу дію в залежності від вибраного класу транспорту.
:::

::: {.section .solution}
##  Рішення {#solution}

Патерн Фабричний метод пропонує відмовитись від безпосереднього
створення об'єктів за допомогою оператора `new`, замінивши його викликом
особливого *фабричного* методу. Не лякайтеся, об'єкти все одно будуть
створюватися за допомогою `new`, але робити це буде фабричний метод.

<figure class="image">
<img
src="../../images/patterns/diagrams/factory-method/solution16ec2.png?id=fc756d2af296b5b4d482e548214d08ef"
style="aspect-ratio: 2.3;"
srcset="/images/patterns/diagrams/factory-method/solution1.png?id=fc756d2af296b5b4d482e548214d08ef 620w,/images/patterns/diagrams/factory-method/solution1-1.5x.png?id=22d3b6bb74e966d1cb3a4d8f380cefa3 930w,/images/patterns/diagrams/factory-method/solution1-2x.png?id=c482b3cd7a4d8dd73b4c8c12dfcaa03c 1240w"
sizes="(max-width: 720px) 100vw, 620px" loading="lazy" width="620"
alt="Структура класів-творців" />
<figcaption><p>Підкласи можуть змінювати клас
створюваних об’єктів.</p></figcaption>
</figure>

На перший погляд це може здатись безглуздим --- ми просто перемістили
виклик конструктора з одного кінця програми в інший. Проте тепер ви
зможете перевизначити фабричний метод у підкласі, щоб змінити тип
створюваного продукту.

Щоб ця система запрацювала, всі об'єкти, що повертаються, повинні мати
спільний інтерфейс. Підкласи зможуть виготовляти об'єкти різних класів,
що відповідають одному і тому самому інтерфейсу.

<figure class="image">
<img
src="../../images/patterns/diagrams/factory-method/solution2-uk4ee3.png?id=e141da08c054abe54b122ab2414a0902"
style="aspect-ratio: 1.96;"
srcset="/images/patterns/diagrams/factory-method/solution2-uk.png?id=e141da08c054abe54b122ab2414a0902 490w,/images/patterns/diagrams/factory-method/solution2-uk-1.5x.png?id=6aff47c79739ab690262df04f2910251 735w,/images/patterns/diagrams/factory-method/solution2-uk-2x.png?id=fbba828f3d9a97d71417fa92921f3489 980w"
sizes="(max-width: 720px) 100vw, 490px" loading="lazy" width="490"
alt="Структура ієрархії продуктів" />
<figcaption><p>Всі об’єкти-продукти повинні мати
спільний інтерфейс.</p></figcaption>
</figure>

Наприклад, класи `Вантажівка` і `Судно` реалізують інтерфейс `Транспорт`
з методом `доставити`. Кожен з цих класів реалізує метод по-своєму:
вантажівки перевозять вантажі сушею, а судна --- морем. Фабричний метод
класу `ДорожноїЛогістики` поверне об'єкт-вантажівку, а класу
`МорськоїЛогістики` --- об'єкт-судно.

<figure class="image">
<img
src="../../images/patterns/diagrams/factory-method/solution3-en8f03.png?id=b6f53911fc0d56f9ef99501fc4aec059"
style="aspect-ratio: 1.83;"
srcset="/images/patterns/diagrams/factory-method/solution3-en.png?id=b6f53911fc0d56f9ef99501fc4aec059 640w,/images/patterns/diagrams/factory-method/solution3-en-1.5x.png?id=f66d5c45f61b619d513a12b7e0a755d5 960w,/images/patterns/diagrams/factory-method/solution3-en-2x.png?id=542c0ba89e91ac11ea79e94bc0229f70 1280w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="640"
alt="Програма після застосування фабричного методу" />
<figcaption><p>Допоки всі продукти реалізують спільний інтерфейс, їхні
об’єкти можна змінювати один на інший у
клієнтському коді.</p></figcaption>
</figure>

Клієнт фабричного методу не відчує різниці між цими об'єктами, адже він
трактуватиме їх як якийсь абстрактний `Транспорт`. Для нього буде
важливим, щоб об'єкт мав метод `доставити`, а не те, як конкретно він
працює.
:::

::::: {.section .structure-container}
##  Структура {#structure}

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/factory-method/structure78c3.png?id=4cba0803f42517cfe8548c9bc7dc4c9b"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.74;"
srcset="/images/patterns/diagrams/factory-method/structure.png?id=4cba0803f42517cfe8548c9bc7dc4c9b 660w,/images/patterns/diagrams/factory-method/structure-1.5x.png?id=ece8300890afc14494770a6b6d370428 990w,/images/patterns/diagrams/factory-method/structure-2x.png?id=9ea3aa8a47f8be22e12e523c15b448fd 1320w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="660"
alt="Структура класів патерна Фабричний метод" /><img
src="../../images/patterns/diagrams/factory-method/structure-indexed6136.png?id=4c603207859ca1f939b17b60a3a2e9e0"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.74;"
srcset="/images/patterns/diagrams/factory-method/structure-indexed.png?id=4c603207859ca1f939b17b60a3a2e9e0 660w,/images/patterns/diagrams/factory-method/structure-indexed-1.5x.png?id=626b6d7f577e1c265cce244678b2cf7f 990w,/images/patterns/diagrams/factory-method/structure-indexed-2x.png?id=c794e4f2d05013fb176464a1d1a5d7ab 1320w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="660"
alt="Структура класів патерна Фабричний метод" />
</figure>
:::

1.  **Продукт** визначає загальний інтерфейс об'єктів, які може
    створювати творець та його підкласи.

2.  **Конкретні продукти** містять код різних продуктів. Продукти
    відрізнятимуться реалізацією, але інтерфейс у них буде спільним.

3.  **Творець** оголошує фабричний метод, який має повертати нові
    об'єкти продуктів. Важливо, щоб тип результату цього методу
    співпадав із загальним інтерфейсом продуктів.

    Зазвичай, фабричний метод оголошують абстрактним, щоб змусити всі
    підкласи реалізувати його по-своєму. Однак він може також повертати
    продукт за замовчуванням.

    Незважаючи на назву, важливо розуміти, що створення продуктів **не
    є** єдиною і головною функцією творця. Зазвичай він містить ще й
    інший корисний код для роботи з продуктом. Аналогія: у великій
    софтверній компанії може бути центр підготовки програмістів, але все
    ж таки основним завданням компанії залишається написання коду, а не
    навчання програмістів.

4.  **Конкретні творці** по-своєму реалізують фабричний метод,
    виробляючи ті чи інші конкретні продукти.

    Фабричний метод не зобов'язаний створювати нові об'єкти увесь час.
    Його можна переписати так, аби повертати з якогось сховища або кешу
    вже існуючі об'єкти.
::::
:::::

::: {.section .pseudocode}
##  Псевдокод {#pseudocode}

У цьому прикладі **Фабричний метод** допомагає створювати
крос-платформові елементи інтерфейсу, не прив'язуючи основний код
програми до конкретних класів кожного елементу.

<figure class="image">
<img
src="../../images/patterns/diagrams/factory-method/examplec59f.png?id=67db9a5cb817913444efcb1c067c9835"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/diagrams/factory-method/example.png?id=67db9a5cb817913444efcb1c067c9835 640w,/images/patterns/diagrams/factory-method/example-1.5x.png?id=921d97276e5e2fd8e64609c9cfa021e7 960w,/images/patterns/diagrams/factory-method/example-2x.png?id=a2470830778e318263155000dbdc5870 1280w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="640"
alt="Структура класів прикладу патерна Фабричного методу" />
<figcaption><p>Приклад крос-платформового діалогу.</p></figcaption>
</figure>

Фабричний метод оголошений у класі діалогів. Його підкласи належать до
різних операційних систем. Завдяки фабричному методу, вам не потрібно
переписувати логіку діалогів під кожну систему. Підкласи можуть
успадкувати майже увесь код базового діалогу, змінюючи типи кнопок та
інших елементів, з яких базовий код будує вікна графічного
користувацього інтерфейсу.

Базовий клас діалогів працює з кнопками через їхній загальний програмний
інтерфейс. Незалежно від того, яку варіацію кнопок повернув фабричний
метод, діалог залишиться робочим. Базовий клас не залежить від
конкретних класів кнопок, залишаючи підкласам прийняття рішення про тип
кнопок, які необхідно створити.

Такий підхід можна застосувати і для створення інших елементів
інтерфейсу. Хоча кожен новий тип елементів наближатиме вас до
[Абстрактної фабрики](abstract-factory.html).

<figure class="code">
<pre class="code" lang="pseudocode"><code>// Патерн Фабричний метод має сенс лише тоді, коли в програмі є
// ієрархія класів продуктів.
interface Button is
    method render()
    method onClick(f)

class WindowsButton implements Button is
    method render(a, b) is
        // Відобразити кнопку в стилі Windows.
    method onClick(f) is
        // Навісити на кнопку обробник подій Windows.

class HTMLButton implements Button is
    method render(a, b) is
        // Повернути HTML-код кнопки.
    method onClick(f) is
        // Навісити на кнопку обробник події браузера.


// Базовий клас фабрики. Зауважте, що &quot;фабрика&quot; — це всього лише
// додаткова роль для цього класу. Скоріше за все, він вже має
// якусь бізнес-логіку, яка потребує створення продуктів.
class Dialog is
    method render() is
        // Щоб використати фабричний метод, ви маєте
        // пересвідчитися, що ця бізнес-логіка не залежить від
        // конкретних класів продуктів. Button — це загальний
        // інтейрфейс кнопок, тому все гаразд.
        Button okButton = createButton()
        okButton.onClick(closeDialog)
        okButton.render()

    // Ми виносимо весь код створення продуктів до особливого
    // методу, який називають &quot;фабричним&quot;.
    abstract method createButton():Button


// Конкретні фабрики перевизначають фабричний метод і повертають
// з нього власні продукти.
class WindowsDialog extends Dialog is
    method createButton():Button is
        return new WindowsButton()

class WebDialog extends Dialog is
    method createButton():Button is
        return new HTMLButton()


class Application is
    field dialog: Dialog

    // Програма створює певну фабрику в залежності від
    // конфігурації або оточення.
    method initialize() is
        config = readApplicationConfigFile()

        if (config.OS == &quot;Windows&quot;) then
            dialog = new WindowsDialog()
        else if (config.OS == &quot;Web&quot;) then
            dialog = new WebDialog()
        else
            throw new Exception(&quot;Error! Unknown operating system.&quot;)

    // Якщо весь інший клієнтський код працює з фабриками та
    // продуктами тільки через загальний інтерфейс, то для нього
    // байдуже, якого типу фабрику було створено на початку.
    method main() is
        this.initialize()
        dialog.render()</code></pre>
</figure>
:::

:::::::::: {.section .applicability-container}
##  Застосування {#applicability}

::::::::: applicability
::: applicability-problem
Коли типи і залежності об'єктів, з якими повинен працювати ваш код,
невідомі заздалегідь.
:::

::: applicability-solution
Фабричний метод відокремлює код виробництва продуктів від решти коду,
який використовує ці продукти.

Завдяки цьому код виробництва можна розширювати, не зачіпаючи основний
код. Щоб додати підтримку нового продукту, вам потрібно створити новий
підклас та визначити в ньому фабричний метод, повертаючи звідти
екземпляр нового продукту.
:::

::: applicability-problem
Коли ви хочете надати користувачам можливість розширювати частини вашого
фреймворку чи бібліотеки.
:::

::: applicability-solution
Користувачі можуть розширювати класи вашого фреймворку через
успадкування. Але як же зробити так, аби фреймворк створював об'єкти цих
класів, а не стандартних?

Рішення полягає у тому, щоб надати користувачам можливість розширювати
не лише бажані компоненти, але й класи, які їх створюють. Тому ці класи
повинні мати конкретні створюючі методи, які можна буде перевизначити.

Наприклад, ви використовуєте готовий UI-фреймворк для свого додатку.
Але --- от халепа --- вам необхідно мати круглі кнопки, а не стандартні
прямокутні. Ви створюєте клас `RoundButton`. Але як сказати головному
класу фреймворку `UIFramework`, щоб він почав тепер створювати круглі
кнопки замість стандартних прямокутних?

Для цього з базового класу фреймворку ви створюєте підклас
`UIWithRoundButtons`, перевизначаєте в ньому метод створення кнопки
(а-ля, `createButton`) і вписуєте туди створення свого класу кнопок.
Потім використовуєте `UIWithRoundButtons` замість стандартного
`UIFramework`.
:::

::: applicability-problem
Коли ви хочете зекономити системні ресурси, повторно використовуючи вже
створені об'єкти, замість породження нових.
:::

::: applicability-solution
Така проблема зазвичай виникає під час роботи з «важкими», вимогливими
до ресурсів об'єктами, такими, як підключення до бази даних, файлової
системи й подібними.

Уявіть, скільки дій вам потрібно зробити, аби повторно використовувати
вже існуючі об'єкти:

1.  Спочатку слід створити загальне сховище, щоб зберігати в ньому всі
    створювані об'єкти.
2.  При запиті нового об'єкта потрібно буде подивитись у сховище та
    перевірити, чи є там невикористаний об'єкт.
3.  Потім повернути його клієнтському коду.
4.  Але якщо ж вільних об'єктів немає, створити новий, не забувши додати
    його до сховища.

Увесь цей код потрібно десь розмістити, щоб не засмічувати клієнтський
код.

Найзручнішим місцем був би конструктор об'єкта, адже всі ці перевірки
потрібні тільки під час створення об'єктів, але, на жаль, конструктор
завжди створює **нові** об'єкти, тому він не може повернути існуючий
екземпляр.

Отже, має бути інший метод, який би віддавав як існуючі, так і нові
об'єкти. Ним і стане фабричний метод.
:::
:::::::::
::::::::::

::: {.section .checklist}
##  Кроки реалізації {#checklist}

1.  Приведіть усі створювані продукти до загального інтерфейсу.

2.  Створіть порожній фабричний метод у класі, який виробляє продукти. В
    якості типу, що повертається, вкажіть загальний інтерфейс продукту.

3.  Пройдіться по коду класу й знайдіть усі ділянки, що створюють
    продукти. По черзі замініть ці ділянки викликами фабричного методу,
    переносячи в нього код створення різних продуктів.

    Можливо, доведеться додати до фабричного методу декілька параметрів,
    що контролюють, який з продуктів потрібно створити.

    Імовірніше за все, фабричний метод виглядатиме гнітюче на цьому
    етапі. В ньому житиме великий умовний оператор, який вибирає клас
    створюваного продукту. Але не хвилюйтеся, ми ось-ось все це
    виправимо.

4.  Для кожного типу продуктів заведіть підклас і перевизначте в ньому
    фабричний метод. З суперкласу перемістіть туди код створення
    відповідного продукту.

5.  Якщо створюваних продуктів занадто багато для існуючих підкласів
    творця, ви можете подумати про введення параметрів до фабричного
    методу, аби повертати різні продукти в межах одного підкласу.

    Наприклад, у вас є клас `Пошта` з підкласами `АвіаПошта` і
    `НаземнаПошта`, а також класи продуктів `Літак`, `Вантажівка` й
    `Потяг`. `Авіа` відповідає `Літакам`, але для `НаземноїПошти` є
    відразу два продукти. Ви могли б створити новий підклас пошти й для
    потягів, але проблему можна вирішити по-іншому. Клієнтський код може
    передавати до фабричного методу `НаземноїПошти` аргумент, що
    контролює, який з продуктів буде створено.

6.  Якщо після цих всіх переміщень фабричний метод став порожнім, можете
    зробити його абстрактним. Якщо ж у ньому щось залишилося --- не
    страшно, це буде його типовою реалізацією (за замовчуванням).
:::

:::::: {.section .pros-cons}
##  Переваги та недоліки {#pros-cons}

::::: row
::: col-sm-6
-    Позбавляє клас від прив'язки до конкретних класів продуктів.
-    Виділяє код виробництва продуктів в одне місце, спрощуючи підтримку
    коду.
-    Спрощує додавання нових продуктів до програми.
-    Реалізує *принцип відкритості/закритості*.
:::

::: col-sm-6
-    Може призвести до створення великих [паралельних ієрархій
    класів](../smells/parallel-inheritance-hierarchies.html), адже для
    кожного класу продукту потрібно створити власний підклас творця.
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

-   [Фабричний метод](factory-method.html) можна використовувати разом з
    [Ітератором](iterator.html), щоб підкласи колекцій могли створювати
    необхідні їм ітератори.

-   [Прототип](prototype.html) не спирається на спадкування, але йому
    потрібна складна операція ініціалізації. [Фабричний
    метод](factory-method.html), навпаки, побудований на спадкуванні,
    але не вимагає складної ініціалізації.

-   [Фабричний метод](factory-method.html) можна розглядати як окремий
    випадок [Шаблонного методу](template-method.html). Крім того,
    *Фабричний метод* нерідко буває частиною великого класу з
    *Шаблонними методами*.
:::

::: {.section .implementations}
##  Приклади реалізації патерна {#implementations}

[![Фабричний метод на
C#](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](factory-method/csharp/example.html "Фабричний метод на C#"){.prog-lang-link}
[![Фабричний метод на
C++](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](factory-method/cpp/example.html "Фабричний метод на C++"){.prog-lang-link}
[![Фабричний метод на
Go](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](factory-method/go/example.html "Фабричний метод на Go"){.prog-lang-link}
[![Фабричний метод на
Java](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](factory-method/java/example.html "Фабричний метод на Java"){.prog-lang-link}
[![Фабричний метод на
PHP](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](factory-method/php/example.html "Фабричний метод на PHP"){.prog-lang-link}
[![Фабричний метод на
Python](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](factory-method/python/example.html "Фабричний метод на Python"){.prog-lang-link}
[![Фабричний метод на
Ruby](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](factory-method/ruby/example.html "Фабричний метод на Ruby"){.prog-lang-link}
[![Фабричний метод на
Rust](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](factory-method/rust/example.html "Фабричний метод на Rust"){.prog-lang-link}
[![Фабричний метод на
Swift](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](factory-method/swift/example.html "Фабричний метод на Swift"){.prog-lang-link}
[![Фабричний метод на
TypeScript](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](factory-method/typescript/example.html "Фабричний метод на TypeScript"){.prog-lang-link}
:::

::: {.section .extras}
##  Додаткові матеріали {#extras}

-   Якщо ви вже чули про *Фабрику*, *Фабричний метод* чи *Абстрактну
    фабрику*, але вам все одно важко їх розрізняти, прочитайте нашу
    статтю [Порівняння фабрик](factory-comparison.html).
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

[Абстрактна фабрика []{.fa .fa-arrow-right}](abstract-factory.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Повернутись назад

[[]{.fa .fa-arrow-left} Породжувальні
патерни](creational-patterns.html){.btn .btn-default rel="prev"}
:::
::::::::::::::::::::::::::::::::::

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
::::::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::::::
