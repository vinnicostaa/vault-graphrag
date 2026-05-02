::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [Патерни
проектування](../design-patterns.html){.type} / [Породжувальні
патерни](creational-patterns.html){.type}
:::

# Абстрактна фабрика {#абстрактна-фабрика .title}

::: aka
Також відомий як: [Abstract Factory]{style="display:inline-block"}
:::

::: {.section .intent}
##  Суть патерна {#intent}

**Абстрактна фабрика** --- це породжувальний патерн проектування, що дає
змогу створювати сімейства пов'язаних об'єктів, не прив'язуючись до
конкретних класів створюваних об'єктів.

<figure class="image">
<img
src="../../images/patterns/content/abstract-factory/abstract-factory-uk547d.png?id=ad52745fedfc0bf94a583caf88d16bf3"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/abstract-factory/abstract-factory-uk.png?id=ad52745fedfc0bf94a583caf88d16bf3 640w,/images/patterns/content/abstract-factory/abstract-factory-uk-1.5x.png?id=2f762d10504cacfc128ad5bd02498e6f 960w,/images/patterns/content/abstract-factory/abstract-factory-uk-2x.png?id=17f447252e823c3dbc9bc4704a6571df 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Патерн Абстрактна фабрика" />
</figure>
:::

::: {.section .problem}
##  Проблема {#problem}

Уявіть, що ви пишете симулятор меблевого магазину. Ваш код містить:

1.  Сімейство залежних продуктів. Скажімо, `Крісло` + `Диван` +
    `Столик`.

2.  Кілька варіацій цього сімейства. Наприклад, продукти `Крісло`,
    `Диван` та `Столик` представлені в трьох різних стилях: `Ар-деко`,
    `Вікторіанському` і `Модерн`.

<figure class="image">
<img
src="../../images/patterns/diagrams/abstract-factory/problem-uk84ca.png?id=72cda3235aef3e0087d367ae13d4d2b8"
style="aspect-ratio: 1.45;"
srcset="/images/patterns/diagrams/abstract-factory/problem-uk.png?id=72cda3235aef3e0087d367ae13d4d2b8 420w,/images/patterns/diagrams/abstract-factory/problem-uk-1.5x.png?id=2d690ce0803e053ed215b92041b61143 630w,/images/patterns/diagrams/abstract-factory/problem-uk-2x.png?id=aa90f7a63a0dc29b19a0b106fae0987e 840w"
sizes="(max-width: 720px) 100vw, 420px" loading="lazy" width="420"
alt="Таблиця відповідності сімейства продуктів до їхніх варіацій" />
<figcaption><p>Сімейства продуктів та їхніх варіацій.</p></figcaption>
</figure>

Вам потрібно створювати об'єкти продуктів у такий спосіб, щоб вони
завжди пасували до інших продуктів того самого сімейства. Це дуже
важливо, адже клієнти засмучуються, коли отримують меблі, що не можна
поєднати між собою.

<figure class="image">
<img
src="../../images/patterns/content/abstract-factory/abstract-factory-comic-1-uk6ee9.png?id=274727a931b3430c3fad2f142220fbf5"
style="aspect-ratio: 2;"
srcset="/images/patterns/content/abstract-factory/abstract-factory-comic-1-uk.png?id=274727a931b3430c3fad2f142220fbf5 600w,/images/patterns/content/abstract-factory/abstract-factory-comic-1-uk-1.5x.png?id=c2d5ed313fa131f341758dace80fde8d 900w,/images/patterns/content/abstract-factory/abstract-factory-comic-1-uk-2x.png?id=2a61892bddbb5e3a71d348b86dfe1de1 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600" />
<figcaption><p>Клієнти засмучуються, якщо отримують продукти, що
не поєднуються.</p></figcaption>
</figure>

Крім того, ви не хочете вносити зміни в існуючий код під час додавання в
програму нових продуктів або сімейств. Постачальники часто оновлюють
свої каталоги, але ви б не хотіли змінювати вже написаний код кожен раз
при надходженні нових моделей меблів.
:::

::: {.section .solution}
##  Рішення {#solution}

Для початку, патерн Абстрактна фабрика пропонує виділити загальні
інтерфейси для окремих продуктів, що складають одне сімейство, і описати
в них спільну для цих продуктів поведінку. Так, наприклад, усі варіації
крісел отримають спільний інтерфейс `Крісло`, усі дивани реалізують
інтерфейс `Диван` тощо.

<figure class="image">
<img
src="../../images/patterns/diagrams/abstract-factory/solution1a1ac.png?id=71f2018d8bb443b9cce90d57110675e3"
style="aspect-ratio: 1.5;"
srcset="/images/patterns/diagrams/abstract-factory/solution1.png?id=71f2018d8bb443b9cce90d57110675e3 420w,/images/patterns/diagrams/abstract-factory/solution1-1.5x.png?id=4366e971c850514cde5d33cb7956de8b 630w,/images/patterns/diagrams/abstract-factory/solution1-2x.png?id=eacec286153e058db9255d26541c0529 840w"
sizes="(max-width: 720px) 100vw, 420px" loading="lazy" width="420"
alt="Схема ієрархії класів крісел." />
<figcaption><p>Всі варіації одного й того самого об’єкта мають жити в
одній ієрархії класів.</p></figcaption>
</figure>

Далі ви створюєте *абстрактну фабрику* --- загальний інтерфейс, який
містить методи створення всіх продуктів сімейства (наприклад,
`створитиКрісло`, `створитиДиван` і `створитиСтолик`). Ці операції
повинні повертати **абстрактні** типи продуктів, представлені
інтерфейсами, які ми виділили раніше --- `Крісла`, `Дивани` і `Столики`.

<figure class="image">
<img
src="../../images/patterns/diagrams/abstract-factory/solution24d8e.png?id=53975d6e4714c6f942633a879f7ac571"
style="aspect-ratio: 2;"
srcset="/images/patterns/diagrams/abstract-factory/solution2.png?id=53975d6e4714c6f942633a879f7ac571 640w,/images/patterns/diagrams/abstract-factory/solution2-1.5x.png?id=581a6cdc0dcd5511f1c30e509b1d4a7f 960w,/images/patterns/diagrams/abstract-factory/solution2-2x.png?id=b21557081f75ac7b4110427e89a10378 1280w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="640"
alt="Схема ієрархії класів фабрик." />
<figcaption><p>Конкретні фабрики відповідають певній варіації
сімейства продуктів.</p></figcaption>
</figure>

Як щодо варіацій продуктів? Для кожної варіації сімейства продуктів ми
повинні створити свою власну фабрику, реалізувавши абстрактний
інтерфейс. Фабрики створюють продукти однієї варіації. Наприклад,
`ФабрикаМодерн` буде повертати тільки `КріслаМодерн`,`ДиваниМодерн` і
`СтоликиМодерн`.

Клієнтський код повинен працювати як із фабриками, так і з продуктами
тільки через їхні загальні інтерфейси. Це дозволить подавати у ваші
класи будь-які типи фабрик і виробляти будь-які типи продуктів, без
необхідності вносити зміни в існуючий код.

<figure class="image">
<img
src="../../images/patterns/content/abstract-factory/abstract-factory-comic-2-uk280b.png?id=92dfa9838fdc86970eab762103fa1c23"
style="aspect-ratio: 2;"
srcset="/images/patterns/content/abstract-factory/abstract-factory-comic-2-uk.png?id=92dfa9838fdc86970eab762103fa1c23 600w,/images/patterns/content/abstract-factory/abstract-factory-comic-2-uk-1.5x.png?id=5923116c4bd261f99ec57c78e1a00d9a 900w,/images/patterns/content/abstract-factory/abstract-factory-comic-2-uk-2x.png?id=456a053a95972ccf1e409795d759e3dc 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600" />
<figcaption><p>Для клієнтського коду повинно бути не важливо, з якою
фабрикою працювати.</p></figcaption>
</figure>

Наприклад, клієнтський код просить фабрику зробити стілець. Він не знає,
якому типу відповідає ця фабрика. Він не знає, отримає вікторіанський
або модерновий стілець. Для нього важливо, щоб на цьому стільці можна
було сидіти та щоб цей стілець відмінно виглядав поруч із диваном тієї ж
фабрики.

Залишилося прояснити останній момент: хто ж створює об'єкти конкретних
фабрик, якщо клієнтський код працює лише із загальними інтерфейсами?
Зазвичай програма створює конкретний об'єкт фабрики під час запуску,
причому тип фабрики вибирається на підставі параметрів оточення або
конфігурації.
:::

::::: {.section .structure-container}
##  Структура {#structure}

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/abstract-factory/structureb809.png?id=a3112cdd98765406af94595a3c5e7762"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/diagrams/abstract-factory/structure.png?id=a3112cdd98765406af94595a3c5e7762 720w,/images/patterns/diagrams/abstract-factory/structure-1.5x.png?id=d2964e541620d9c087e693bd0e0a0836 1080w,/images/patterns/diagrams/abstract-factory/structure-2x.png?id=c4d3634ec2e74e02a0fe1a83ce9b50f6 1440w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="720"
alt="Структура класів патерна Абстрактна фабрика" /><img
src="../../images/patterns/diagrams/abstract-factory/structure-indexed028d.png?id=6ae1c99cbd90cf58753c633624fb1a04"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.56;"
srcset="/images/patterns/diagrams/abstract-factory/structure-indexed.png?id=6ae1c99cbd90cf58753c633624fb1a04 700w,/images/patterns/diagrams/abstract-factory/structure-indexed-1.5x.png?id=473be2975c5716162e6969e6532210ac 1050w,/images/patterns/diagrams/abstract-factory/structure-indexed-2x.png?id=cb6d4e1e89826c42966dc7097374f889 1400w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="700"
alt="Структура класів патерна Абстрактна фабрика" />
</figure>
:::

1.  **Абстрактні продукти** оголошують інтерфейси продуктів, що
    пов'язані один з одним за змістом, але виконують різні функції.

2.  **Конкретні продукти** --- великий набір класів, що належать до
    різних абстрактних продуктів (крісло/столик), але мають одні й ті
    самі варіації (Вікторіанський/Модерн).

3.  **Абстрактна фабрика** оголошує методи створення різних абстрактних
    продуктів (крісло/столик).

4.  **Конкретні фабрики** кожна належить до своєї варіації продуктів
    (Вікторіанський/Модерн) і реалізує методи абстрактної фабрики, даючи
    змогу створювати всі продукти певної варіації.

5.  Незважаючи на те, що конкретні фабрики породжують конкретні
    продукти, сигнатури їхніх методів мусять повертати відповідні
    абстрактні продукти. Це дозволить клієнтського коду, що використовує
    фабрику, не прив'язуватися до конкретних класів продуктів. Клієнт
    зможе працювати з будь-якими варіаціями продуктів через абстрактні
    інтерфейси.
::::
:::::

::: {.section .pseudocode}
##  Псевдокод {#pseudocode}

У цьому прикладі **Абстрактна фабрика** створює крос-платформові
елементи інтерфейсу і стежить за тим, щоб вони відповідали обраній
операційній системі.

<figure class="image">
<img
src="../../images/patterns/diagrams/abstract-factory/exampled62f.png?id=5928a61d18bf00b047463471c599100a"
style="aspect-ratio: 1.42;"
srcset="/images/patterns/diagrams/abstract-factory/example.png?id=5928a61d18bf00b047463471c599100a 640w,/images/patterns/diagrams/abstract-factory/example-1.5x.png?id=baee3bac793ec97e0ec91c49d9382e7d 960w,/images/patterns/diagrams/abstract-factory/example-2x.png?id=eb5127b1d6486f6fad73be2d5e444449 1280w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="640"
alt="Структура класів прикладу патерна Абстрактної фабрики" />
<figcaption><p>Приклад крос-платформового графічного
інтерфейсу користувача.</p></figcaption>
</figure>

Крос-платформова програма може відображати одні й ті самі елементи
інтерфейсу по-різному, в залежності від обраної операційної системи.
Важливо, щоб у такій програмі всі створювані елементи завжди відповідали
поточній операційній системі. Ви ж не хотіли б, аби програма, запущена
на Windows, раптом почала показувати чек-бокси в стилі macOS?

Абстрактна фабрика оголошує список створюючих методів, які клієнтський
код може використовувати для отримання тих чи інших різновидів елементів
інтерфейсу. Конкретні фабрики відносяться до різних операційних систем і
створюють елементи, сумісні з цією системою.

Програма на самому початку визначає фабрику, що відповідає поточній
операційній системі. Потім створює цю фабрику та віддає її клієнтському
коду. У подальшому, щоб виключити несумісність продуктів, що
повертаються, клієнт працюватиме тільки з цією фабрикою.

Клієнтський код не залежить від конкретних класів фабрик чи елементів
інтерфейсу. Він спілкується з ними через загальні інтерфейси, не
залежачи від конкретних класів фабрик чи елементів користувацького
інтерфейсу.

Таким чином, щоб додати до програми нову варіацію елементів інтерфейсу
(наприклад, для підтримки Linux), вам не потрібно змінювати клієнтський
код. Достатньо створити ще одну фабрику, що виготовляє ці елементи.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// Цей патерн передбачає, що ви маєте кілька сімейств продуктів,
// які знаходяться в окремих ієрархіях класів (Button/Checkbox).
// Продукти одного сімейства повинні мати спільний інтерфейс.
interface Button is
    method paint()

// Cімейства продуктів мають однакові варіації (macOS/Windows).
class WinButton implements Button is
    method paint() is
        // Відобразити кнопку в стилі Windows.

class MacButton implements Button is
    method paint() is
        // Відобразити кнопку в стилі macOS.


interface Checkbox is
    method paint()

class WinCheckbox implements Checkbox is
    method paint() is
        // Відобразити чекбокс в стилі Windows.

class MacCheckbox implements Checkbox is
    method paint() is
        // Відобразити чекбокс в стилі macOS.


// Абстрактна фабрика знає про всі абстрактні типи продуктів.
interface GUIFactory is
    method createButton():Button
    method createCheckbox():Checkbox


// Кожна конкретна фабрика знає лише про продукти своєї варіації
// і створює лише їх.
class WinFactory implements GUIFactory is
    method createButton():Button is
        return new WinButton()
    method createCheckbox():Checkbox is
        return new WinCheckbox()

// Незважаючи на те, що фабрики оперують конкретними класами,
// їхні методи повертають абстрактні типи продуктів. Завдяки
// цьому фабрики можна заміняти одну на іншу, не змінюючи
// клієнтського коду.
class MacFactory implements GUIFactory is
    method createButton():Button is
        return new MacButton()
    method createCheckbox():Checkbox is
        return new MacCheckbox()


// Для коду, який використовує фабрику, не важливо, з якою
// конкретно фабрикою він працює. Всі отримувачі продуктів
// працюють з ними через загальні інтерфейси.
class Application is
    private field factory: GUIFactory
    private field button: Button
    constructor Application(factory: GUIFactory) is
        this.factory = factory
    method createUI()
        this.button = factory.createButton()
    method paint()
        button.paint()


// Програма вибирає тип конкретної фабрики й створює її
// динамічно, виходячи з конфігурації або оточення.
class ApplicationConfigurator is
    method main() is
        config = readApplicationConfigFile()

        if (config.OS == &quot;Windows&quot;) then
            factory = new WinFactory()
        else if (config.OS == &quot;Mac&quot;) then
            factory = new MacFactory()
        else
            throw new Exception(&quot;Error! Unknown operating system.&quot;)

        Application app = new Application(factory)</code></pre>
</figure>
:::

:::::::: {.section .applicability-container}
##  Застосування {#applicability}

::::::: applicability
::: applicability-problem
Коли бізнес-логіка програми повинна працювати з різними видами
пов'язаних один з одним продуктів, незалежно від конкретних класів
продуктів.
:::

::: applicability-solution
Абстрактна фабрика приховує від клієнтського коду подробиці того, як і
які конкретно об'єкти будуть створені. Внаслідок цього, клієнтський код
може працювати з усіма типами створюваних продуктів, так як їхній
загальний інтерфейс був визначений заздалегідь.
:::

::: applicability-problem
Коли в програмі вже використовується [Фабричний
метод](factory-method.html), але чергові зміни передбачають введення
нових типів продуктів.
:::

::: applicability-solution
У будь-якій добротній програмі кожен *клас має відповідати лише за одну
річ*. Якщо клас має занадто багато фабричних методів, вони здатні
затуманити його основну функцію. Тому є сенс у тому, щоб винести усю
логіку створення продуктів в окрему ієрархію класів, застосувавши
абстрактну фабрику.
:::
:::::::
::::::::

::: {.section .checklist}
##  Кроки реалізації {#checklist}

1.  Створіть таблицю співвідношень типів продуктів до варіацій сімейств
    продуктів.

2.  Зведіть усі варіації продуктів до загальних інтерфейсів.

3.  Визначте інтерфейс абстрактної фабрики. Він повинен мати фабричні
    методи для створення кожного типу продуктів.

4.  Створіть класи конкретних фабрик, реалізувавши інтерфейс абстрактної
    фабрики. Цих класів має бути стільки ж, скільки й варіацій сімейств
    продуктів.

5.  Змініть код ініціалізації програми так, щоб вона створювала певну
    фабрику й передавала її до клієнтського коду.

6.  Замініть у клієнтському коді ділянки створення продуктів через
    конструктор на виклики відповідних методів фабрики.
:::

:::::: {.section .pros-cons}
##  Переваги та недоліки {#pros-cons}

::::: row
::: col-sm-6
-    Гарантує поєднання створюваних продуктів.
-    Звільняє клієнтський код від прив'язки до конкретних класів
    продукту.
-    Виділяє код виробництва продуктів в одне місце, спрощуючи підтримку
    коду.
-    Спрощує додавання нових продуктів до програми.
-    Реалізує *принцип відкритості/закритості*.
:::

::: col-sm-6
-    Ускладнює код програми внаслідок введення великої кількості
    додаткових класів.
-    Вимагає наявності всіх типів продукту в кожній варіації.
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

-   Класи [Абстрактної фабрики](abstract-factory.html) найчастіше
    реалізуються за допомогою [Фабричного методу](factory-method.html),
    хоча вони можуть бути побудовані і на основі
    [Прототипу](prototype.html).

-   [Абстрактна фабрика](abstract-factory.html) може бути використана
    замість [Фасаду](facade.html) для того, щоб приховати
    платформо-залежні класи.

-   [Абстрактна фабрика](abstract-factory.html) може працювати спільно з
    [Мостом](bridge.html). Це особливо корисно, якщо у вас є абстракції,
    які можуть працювати тільки з деякими реалізаціями. В цьому випадку
    фабрика визначатиме типи створюваних абстракцій та реалізацій.

-   [Абстрактна фабрика](abstract-factory.html),
    [Будівельник](builder.html) та [Прототип](prototype.html) можуть
    реалізовуватися за допомогою [Одинака](singleton.html).
:::

::: {.section .implementations}
##  Приклади реалізації патерна {#implementations}

[![Абстрактна фабрика на
C#](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](abstract-factory/csharp/example.html "Абстрактна фабрика на C#"){.prog-lang-link}
[![Абстрактна фабрика на
C++](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](abstract-factory/cpp/example.html "Абстрактна фабрика на C++"){.prog-lang-link}
[![Абстрактна фабрика на
Go](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](abstract-factory/go/example.html "Абстрактна фабрика на Go"){.prog-lang-link}
[![Абстрактна фабрика на
Java](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](abstract-factory/java/example.html "Абстрактна фабрика на Java"){.prog-lang-link}
[![Абстрактна фабрика на
PHP](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](abstract-factory/php/example.html "Абстрактна фабрика на PHP"){.prog-lang-link}
[![Абстрактна фабрика на
Python](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](abstract-factory/python/example.html "Абстрактна фабрика на Python"){.prog-lang-link}
[![Абстрактна фабрика на
Ruby](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](abstract-factory/ruby/example.html "Абстрактна фабрика на Ruby"){.prog-lang-link}
[![Абстрактна фабрика на
Rust](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](abstract-factory/rust/example.html "Абстрактна фабрика на Rust"){.prog-lang-link}
[![Абстрактна фабрика на
Swift](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](abstract-factory/swift/example.html "Абстрактна фабрика на Swift"){.prog-lang-link}
[![Абстрактна фабрика на
TypeScript](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](abstract-factory/typescript/example.html "Абстрактна фабрика на TypeScript"){.prog-lang-link}
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

[Порівняння фабрик []{.fa
.fa-arrow-right}](factory-comparison.html){.btn .btn-primary rel="next"}
:::

::: prev
#### Повернутись назад

[[]{.fa .fa-arrow-left} Фабричний метод](factory-method.html){.btn
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
