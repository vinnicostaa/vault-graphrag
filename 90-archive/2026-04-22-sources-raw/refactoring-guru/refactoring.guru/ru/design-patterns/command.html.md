::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [Паттерны
проектирования](../design-patterns.html){.type} / [Поведенческие
паттерны](behavioral-patterns.html){.type}
:::

# Команда {#команда .title}

::: aka
Также известен как:
[Действие, ]{style="display:inline-block"}[Транзакция, ]{style="display:inline-block"}[Action, ]{style="display:inline-block"}[Command]{style="display:inline-block"}
:::

::: {.section .intent}
##  Суть паттерна {#intent}

**Команда** --- это поведенческий паттерн проектирования, который
превращает запросы в объекты, позволяя передавать их как аргументы при
вызове методов, ставить запросы в очередь, логировать их, а также
поддерживать отмену операций.

<figure class="image">
<img
src="../../images/patterns/content/command/command-en99f0.png?id=80fbadc666cf3b9b1958c546d2746ca4"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/command/command-en.png?id=80fbadc666cf3b9b1958c546d2746ca4 640w,/images/patterns/content/command/command-en-1.5x.png?id=c3e1a91b500ce88f9b8fda6176762698 960w,/images/patterns/content/command/command-en-2x.png?id=6149af804cbbbd5cb18595c30b856d89 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Паттерн Команда" />
</figure>
:::

::: {.section .problem}
##  Проблема {#problem}

Представьте, что вы работаете над программой текстового редактора. Дело
как раз подошло к разработке панели управления. Вы создали класс
красивых `Кнопок` и хотите использовать его для всех кнопок приложения,
начиная от панели управления, заканчивая простыми кнопками в диалогах.

<figure class="image">
<img
src="../../images/patterns/diagrams/command/problem163e2.png?id=84189315a0e8d91da262792979005ab4"
style="aspect-ratio: 0.88;"
srcset="/images/patterns/diagrams/command/problem1.png?id=84189315a0e8d91da262792979005ab4 230w,/images/patterns/diagrams/command/problem1-1.5x.png?id=77bf0bc58e5c9c9b8369bc4f8dec457f 345w,/images/patterns/diagrams/command/problem1-2x.png?id=af4c4e9c1d1b4fa2c4104c5f6bb18719 460w"
sizes="(max-width: 720px) 100vw, 230px" loading="lazy" width="230"
alt="Проблема, которую решает Команда" />
<figcaption><p>Все кнопки приложения унаследованы от
одного класса.</p></figcaption>
</figure>

Все эти кнопки, хоть и выглядят схоже, но делают разные вещи. Поэтому
возникает вопрос: куда поместить код обработчиков кликов по этим
кнопкам? Самым простым решением было бы создать подклассы для каждой
кнопки и переопределить в них метод действия под разные задачи.

<figure class="image">
<img
src="../../images/patterns/diagrams/command/problem2f2f9.png?id=f0e33da1842b3a3ee3b4857de0b6ec93"
style="aspect-ratio: 2.11;"
srcset="/images/patterns/diagrams/command/problem2.png?id=f0e33da1842b3a3ee3b4857de0b6ec93 400w,/images/patterns/diagrams/command/problem2-1.5x.png?id=7ae15e2b07d5d076a878d99c3ed8ac32 600w,/images/patterns/diagrams/command/problem2-2x.png?id=5eea7d0f6247da011150bb63e423f405 800w"
sizes="(max-width: 720px) 100vw, 400px" loading="lazy" width="400"
alt="Множество подклассов кнопок" />
<figcaption><p>Множество подклассов кнопок.</p></figcaption>
</figure>

Но скоро стало понятно, что такой подход никуда не годится. Во-первых,
получается очень много подклассов. Во-вторых, код кнопок, относящийся к
графическому интерфейсу, начинает зависеть от классов бизнес-логики,
которая довольно часто меняется.

<figure class="image">
<img
src="../../images/patterns/diagrams/command/problem3-enf2d0.png?id=1fbd56ae7ba3e3dfac45cfa2de6db450"
style="aspect-ratio: 4.36;"
srcset="/images/patterns/diagrams/command/problem3-en.png?id=1fbd56ae7ba3e3dfac45cfa2de6db450 480w,/images/patterns/diagrams/command/problem3-en-1.5x.png?id=2da629d84c7cbabbca11562a3d7d2559 720w,/images/patterns/diagrams/command/problem3-en-2x.png?id=e06878f7cfbe4131980c68db2904878e 960w"
sizes="(max-width: 720px) 100vw, 480px" loading="lazy" width="480"
alt="Несколько классов дублируют одну и ту же функциональность" />
<figcaption><p>Несколько классов дублируют одну и ту
же функциональность.</p></figcaption>
</figure>

Но самое обидное ещё впереди. Ведь некоторые операции, например,
«сохранить», можно вызывать из нескольких мест: нажав кнопку на панели
управления, вызвав контекстное меню или просто нажав клавиши `Ctrl+S`.
Когда в программе были только кнопки, код сохранения имелся только в
подклассе `SaveButton`. Но теперь его придётся продублировать ещё в два
класса.
:::

::: {.section .solution}
##  Решение {#solution}

Хорошие программы обычно структурированы в виде слоёв. Самый
распространённый пример --- слои пользовательского интерфейса и
бизнес-логики. Первый всего лишь рисует красивую картинку для
пользователя. Но когда нужно сделать что-то важное, интерфейс «просит»
слой бизнес-логики заняться этим.

В реальности это выглядит так: один из объектов интерфейса напрямую
вызывает метод одного из объектов бизнес-логики, передавая в него
какие-то параметры.

<figure class="image">
<img
src="../../images/patterns/diagrams/command/solution1-rub55d.png?id=f431dd00a55caf6b85e9fb1c1cc5d859"
style="aspect-ratio: 2.24;"
srcset="/images/patterns/diagrams/command/solution1-ru.png?id=f431dd00a55caf6b85e9fb1c1cc5d859 470w,/images/patterns/diagrams/command/solution1-ru-1.5x.png?id=e76b4de7b8c711eb33ac94e8e5689b47 705w,/images/patterns/diagrams/command/solution1-ru-2x.png?id=d1987064fb2c02212cd6c407937bc295 940w"
sizes="(max-width: 720px) 100vw, 470px" loading="lazy" width="470"
alt="Прямой доступ из UI в бизнес-логику" />
<figcaption><p>Прямой доступ из UI в бизнес-логику.</p></figcaption>
</figure>

Паттерн Команда предлагает больше не отправлять такие вызовы напрямую.
Вместо этого каждый вызов, отличающийся от других, следует завернуть в
собственный класс с единственным методом, который и будет осуществлять
вызов. Такие объекты называют *командами*.

К объекту интерфейса можно будет привязать объект команды, который
знает, кому и в каком виде следует отправлять запросы. Когда объект
интерфейса будет готов передать запрос, он вызовет метод команды, а
та --- позаботится обо всём остальном.

<figure class="image">
<img
src="../../images/patterns/diagrams/command/solution2-rue70d.png?id=b198d652acf400381f9202f56a664fa6"
style="aspect-ratio: 2.62;"
srcset="/images/patterns/diagrams/command/solution2-ru.png?id=b198d652acf400381f9202f56a664fa6 550w,/images/patterns/diagrams/command/solution2-ru-1.5x.png?id=d0baa28eeeeb74d9e64e86abcdd17340 825w,/images/patterns/diagrams/command/solution2-ru-2x.png?id=f824bb61256b3afaa6fcab1f2647c436 1100w"
sizes="(max-width: 720px) 100vw, 550px" loading="lazy" width="550"
alt="Доступ из UI в бизнес-логику через команду" />
<figcaption><p>Доступ из UI в бизнес-логику
через команду.</p></figcaption>
</figure>

Классы команд можно объединить под общим интерфейсом c единственным
методом запуска. После этого одни и те же отправители смогут работать с
различными командами, не привязываясь к их классам. Даже больше: команды
можно будет взаимозаменять на лету, изменяя итоговое поведение
отправителей.

Параметры, с которыми должен быть вызван метод объекта получателя, можно
загодя сохранить в полях объекта-команды. Благодаря этому, объекты,
отправляющие запросы, могут не беспокоиться о том, чтобы собрать
необходимые для получателя данные. Более того, они теперь вообще не
знают, кто будет получателем запроса. Вся эта информация скрыта внутри
команды.

<figure class="image">
<img
src="../../images/patterns/diagrams/command/solution3-endc8e.png?id=c92f6874b95de46b041499d41234b00b"
style="aspect-ratio: 1.83;"
srcset="/images/patterns/diagrams/command/solution3-en.png?id=c92f6874b95de46b041499d41234b00b 440w,/images/patterns/diagrams/command/solution3-en-1.5x.png?id=580ba981ae73f0608513349ecf38ad90 660w,/images/patterns/diagrams/command/solution3-en-2x.png?id=c12bb9971d1ba4f8a3d3717bbced2859 880w"
sizes="(max-width: 720px) 100vw, 440px" loading="lazy" width="440"
alt="Классы UI делегируют работу командам" />
<figcaption><p>Классы UI делегируют работу командам.</p></figcaption>
</figure>

После применения Команды в нашем примере с текстовым редактором вам
больше не потребуется создавать уйму подклассов кнопок под разные
действия. Будет достаточно единственного класса с полем для хранения
объекта команды.

Используя общий интерфейс команд, объекты кнопок будут ссылаться на
объекты команд различных типов. При нажатии кнопки будут делегировать
работу связанным командам, а команды --- перенаправлять вызовы тем или
иным объектам бизнес-логики.

Так же можно поступить и с контекстным меню, и с горячими клавишами. Они
будут привязаны к тем же объектам команд, что и кнопки, избавляя классы
от дублирования.

Таким образом, команды станут гибкой прослойкой между пользовательским
интерфейсом и бизнес-логикой. И это лишь малая доля пользы, которую
может принести паттерн Команда!
:::

::: {.section .analogy}
##  Аналогия из жизни {#analogy}

<figure class="image">
<img
src="../../images/patterns/content/command/command-comic-16e6f.png?id=551df832f445080976f3116e0dc120c9"
style="aspect-ratio: 2;"
srcset="/images/patterns/content/command/command-comic-1.png?id=551df832f445080976f3116e0dc120c9 600w,/images/patterns/content/command/command-comic-1-1.5x.png?id=82dc5c38ce372ed4dfd4a37c04faeb72 900w,/images/patterns/content/command/command-comic-1-2x.png?id=47b3c00b2cfbda7157a1ed9da6ce2948 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Пример заказа в ресторане" />
<figcaption><p>Пример заказа в ресторане.</p></figcaption>
</figure>

Вы заходите в ресторан и садитесь у окна. К вам подходит вежливый
официант и принимает заказ, записывая все пожелания в блокнот.
Откланявшись, он уходит на кухню, где вырывает лист из блокнота и клеит
на стену. Далее лист оказывается в руках повара, который читает
содержание заказа и готовит заказанные блюда.

В этом примере вы являетесь *отправителем*, официант с блокнотом ---
*командой*, а повар --- *получателем*. Как и в паттерне, вы не
соприкасаетесь напрямую с поваром. Вместо этого вы отправляете заказ с
официантом, который самостоятельно «настраивает» повара на работу. С
другой стороны, повар не знает, кто конкретно послал ему заказ. Но это
ему безразлично, так как вся необходимая информация есть в листе заказа.
:::

::::: {.section .structure-container}
##  Структура {#structure}

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/command/structure44ec.png?id=1cd7833638f4c43630f4a84017d31195"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.7;"
srcset="/images/patterns/diagrams/command/structure.png?id=1cd7833638f4c43630f4a84017d31195 630w,/images/patterns/diagrams/command/structure-1.5x.png?id=6e5b68277c7747b22266e408891d5841 945w,/images/patterns/diagrams/command/structure-2x.png?id=1dfaa84354ffe49ef7ad46ce069482d2 1260w"
sizes="(max-width: 720px) 100vw, 630px" loading="lazy" width="630"
alt="Структура классов паттерна Команда" /><img
src="../../images/patterns/diagrams/command/structure-indexed49b8.png?id=95529d7282dc7bc1c5bc443423b1cf4f"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.64;"
srcset="/images/patterns/diagrams/command/structure-indexed.png?id=95529d7282dc7bc1c5bc443423b1cf4f 640w,/images/patterns/diagrams/command/structure-indexed-1.5x.png?id=29d6c5c7a06da2747ed756c0ddad6169 960w,/images/patterns/diagrams/command/structure-indexed-2x.png?id=e4cc286a44768c7d060af33497da7df6 1280w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="640"
alt="Структура классов паттерна Команда" />
</figure>
:::

1.  **Отправитель** хранит ссылку на объект команды и обращается к нему,
    когда нужно выполнить какое-то действие. Отправитель работает с
    командами только через их общий интерфейс. Он не знает, какую
    конкретно команду использует, так как получает готовый объект
    команды от клиента.

2.  **Команда** описывает общий для всех конкретных команд интерфейс.
    Обычно здесь описан всего один метод для запуска команды.

3.  **Конкретные команды** реализуют различные запросы, следуя общему
    интерфейсу команд. Обычно команда не делает всю работу
    самостоятельно, а лишь передаёт вызов получателю, которым является
    один из объектов бизнес-логики.

    Параметры, с которыми команда обращается к получателю, следует
    хранить в виде полей. В большинстве случаев объекты команд можно
    сделать неизменяемыми, передавая в них все необходимые параметры
    только через конструктор.

4.  **Получатель** содержит бизнес-логику программы. В этой роли может
    выступать практически любой объект. Обычно команды перенаправляют
    вызовы получателям. Но иногда, чтобы упростить программу, вы можете
    избавиться от получателей, «слив» их код в классы команд.

5.  **Клиент** создаёт объекты конкретных команд, передавая в них все
    необходимые параметры, среди которых могут быть и ссылки на объекты
    получателей. После этого клиент связывает объекты отправителей с
    созданными командами.
::::
:::::

::: {.section .pseudocode}
##  Псевдокод {#pseudocode}

В этом примере паттерн **Команда** служит для ведения истории
выполненных операций, позволяя отменять их, если потребуется.

<figure class="image">
<img
src="../../images/patterns/diagrams/command/example31bf.png?id=1f42c8395fe54d0e409026b91881e2a0"
style="aspect-ratio: 1.07;"
srcset="/images/patterns/diagrams/command/example.png?id=1f42c8395fe54d0e409026b91881e2a0 640w,/images/patterns/diagrams/command/example-1.5x.png?id=435055a78a82c005eebb1bef869998ae 960w,/images/patterns/diagrams/command/example-2x.png?id=ed44dfd9b02eccf1ae7e2714d018ed36 1280w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="640"
alt="Структура классов примера паттерна Команда" />
<figcaption><p>Пример реализации отмены в
текстовом редакторе.</p></figcaption>
</figure>

Команды, которые меняют состояние редактора (например, команда вставки
текста из буфера обмена), сохраняют копию состояния редактора перед
выполнением действия. Копии выполненных команд помещаются в историю
команд, откуда они могут быть получены, если нужно будет сделать отмену
операции.

Классы элементов интерфейса, истории команд и прочие не зависят от
конкретных классов команд, так как работают с ними через общий
интерфейс. Это позволяет добавлять в приложение новые команды, не
изменяя существующий код.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// Абстрактная команда задаёт общий интерфейс для конкретных
// классов команд и содержит базовое поведение отмены операции.
abstract class Command is
    protected field app: Application
    protected field editor: Editor
    protected field backup: text

    constructor Command(app: Application, editor: Editor) is
        this.app = app
        this.editor = editor

    // Сохраняем состояние редактора.
    method saveBackup() is
        backup = editor.text

    // Восстанавливаем состояние редактора.
    method undo() is
        editor.text = backup

    // Главный метод команды остаётся абстрактным, чтобы каждая
    // конкретная команда определила его по-своему. Метод должен
    // возвратить true или false в зависимости о того, изменила
    // ли команда состояние редактора, а значит, нужно ли её
    // сохранить в истории.
    abstract method execute()


// Конкретные команды.
class CopyCommand extends Command is
    // Команда копирования не записывается в историю, так как
    // она не меняет состояние редактора.
    method execute() is
        app.clipboard = editor.getSelection()
        return false

class CutCommand extends Command is
    // Команды, меняющие состояние редактора, сохраняют
    // состояние редактора перед своим действием и сигнализируют
    // об изменении, возвращая true.
    method execute() is
        saveBackup()
        app.clipboard = editor.getSelection()
        editor.deleteSelection()
        return true

class PasteCommand extends Command is
    method execute() is
        saveBackup()
        editor.replaceSelection(app.clipboard)
        return true

// Отмена — это тоже команда.
class UndoCommand extends Command is
    method execute() is
        app.undo()
        return false


// Глобальная история команд — это стек.
class CommandHistory is
    private field history: array of Command

    // Последний зашедший...
    method push(c: Command) is
        // Добавить команду в конец массива-истории.

    // ...выходит первым.
    method pop():Command is
        // Достать последнюю команду из массива-истории.


// Класс редактора содержит непосредственные операции над
// текстом. Он отыгрывает роль получателя — команды делегируют
// ему свои действия.
class Editor is
    field text: string

    method getSelection() is
        // Вернуть выбранный текст.

    method deleteSelection() is
        // Удалить выбранный текст.

    method replaceSelection(text) is
        // Вставить текст из буфера обмена в текущей позиции.


// Класс приложения настраивает объекты для совместной работы.
// Он выступает в роли отправителя — создаёт команды, чтобы
// выполнить какие-то действия.
class Application is
    field clipboard: string
    field editors: array of Editors
    field activeEditor: Editor
    field history: CommandHistory

    // Код, привязывающий команды к элементам интерфейса, может
    // выглядеть примерно так.
    method createUI() is
        // ...
        copy = function() {executeCommand(
            new CopyCommand(this, activeEditor)) }
        copyButton.setCommand(copy)
        shortcuts.onKeyPress(&quot;Ctrl+C&quot;, copy)

        cut = function() { executeCommand(
            new CutCommand(this, activeEditor)) }
        cutButton.setCommand(cut)
        shortcuts.onKeyPress(&quot;Ctrl+X&quot;, cut)

        paste = function() { executeCommand(
            new PasteCommand(this, activeEditor)) }
        pasteButton.setCommand(paste)
        shortcuts.onKeyPress(&quot;Ctrl+V&quot;, paste)

        undo = function() { executeCommand(
            new UndoCommand(this, activeEditor)) }
        undoButton.setCommand(undo)
        shortcuts.onKeyPress(&quot;Ctrl+Z&quot;, undo)

    // Запускаем команду и проверяем, надо ли добавить её в
    // историю.
    method executeCommand(command) is
        if (command.execute())
            history.push(command)

    // Берём последнюю команду из истории и заставляем её все
    // отменить. Мы не знаем конкретный тип команды, но это и не
    // важно, так как каждая команда знает, как отменить своё
    // действие.
    method undo() is
        command = history.pop()
        if (command != null)
            command.undo()</code></pre>
</figure>
:::

:::::::::: {.section .applicability-container}
##  Применимость {#applicability}

::::::::: applicability
::: applicability-problem
Когда вы хотите параметризовать объекты выполняемым действием.
:::

::: applicability-solution
Команда превращает операции в объекты. А объекты можно передавать,
хранить и взаимозаменять внутри других объектов.

Скажем, вы разрабатываете библиотеку графического меню и хотите, чтобы
пользователи могли использовать меню в разных приложениях, не меняя
каждый раз код ваших классов. Применив паттерн, пользователям не
придётся изменять классы меню, вместо этого они будут конфигурировать
объекты меню различными командами.
:::

::: applicability-problem
Когда вы хотите ставить операции в очередь, выполнять их по расписанию
или передавать по сети.
:::

::: applicability-solution
Как и любые другие объекты, команды можно сериализовать, то есть
превратить в строку, чтобы потом сохранить в файл или базу данных. Затем
в любой удобный момент её можно достать обратно, снова превратить в
объект команды и выполнить. Таким же образом команды можно передавать по
сети, логировать или выполнять на удалённом сервере.
:::

::: applicability-problem
Когда вам нужна операция отмены.
:::

::: applicability-solution
Главная вещь, которая вам нужна, чтобы иметь возможность отмены
операций, --- это хранение истории. Среди многих способов, которыми
можно это сделать, паттерн Команда является, пожалуй, самым популярным.

История команд выглядит как стек, в который попадают все выполненные
объекты команд. Каждая команда перед выполнением операции сохраняет
текущее состояние объекта, с которым она будет работать. После
выполнения операции копия команды попадает в стек истории, все ещё неся
в себе сохранённое состояние объекта. Если потребуется отмена, программа
возьмёт последнюю команду из истории и возобновит сохранённое в ней
состояние.

Этот способ имеет две особенности. Во-первых, точное состояние объектов
не так-то просто сохранить, ведь часть его может быть приватным. Но с
этим может помочь справиться паттерн [Снимок](memento.html).

Во-вторых, копии состояния могут занимать довольно много оперативной
памяти. Поэтому иногда можно прибегнуть к альтернативной реализации,
когда вместо восстановления старого состояния команда выполняет обратное
действие. Недостаток этого способа в сложности (а иногда и
невозможности) реализации обратного действия.
:::
:::::::::
::::::::::

::: {.section .checklist}
##  Шаги реализации {#checklist}

1.  Создайте общий интерфейс команд и определите в нём метод запуска.

2.  Один за другим создайте классы конкретных команд. В каждом классе
    должно быть поле для хранения ссылки на один или несколько
    объектов-получателей, которым команда будет перенаправлять основную
    работу.

    Кроме этого, команда должна иметь поля для хранения параметров,
    которые нужны при вызове методов получателя. Значения всех этих
    полей команда должна получать через конструктор.

    И, наконец, реализуйте основной метод команды, вызывая в нём те или
    иные методы получателя.

3.  Добавьте в классы отправителей поля для хранения команд. Обычно
    объекты-отправители принимают готовые объекты команд извне --- через
    конструктор либо через сеттер поля команды.

4.  Измените основной код отправителей так, чтобы они делегировали
    выполнение действия команде.

5.  Порядок инициализации объектов должен выглядеть так:

    -   Создаём объекты получателей.
    -   Создаём объекты команд, связав их с получателями.
    -   Создаём объекты отправителей, связав их с командами.
:::

:::::: {.section .pros-cons}
##  Преимущества и недостатки {#pros-cons}

::::: row
::: col-sm-6
-    Убирает прямую зависимость между объектами, вызывающими операции, и
    объектами, которые их непосредственно выполняют.
-    Позволяет реализовать простую отмену и повтор операций.
-    Позволяет реализовать отложенный запуск операций.
-    Позволяет собирать сложные команды из простых.
-    Реализует *принцип открытости/закрытости*.
:::

::: col-sm-6
-    Усложняет код программы из-за введения множества дополнительных
    классов.
:::
:::::
::::::

::: {.section .relations}
##  Отношения с другими паттернами {#relations}

-   [Цепочка обязанностей](chain-of-responsibility.html),
    [Команда](command.html), [Посредник](mediator.html) и
    [Наблюдатель](observer.html) показывают различные способы работы
    отправителей запросов с их получателями:

    -   *Цепочка обязанностей* передаёт запрос последовательно через
        цепочку потенциальных получателей, ожидая, что какой-то из них
        обработает запрос.
    -   *Команда* устанавливает косвенную одностороннюю связь от
        отправителей к получателям.
    -   *Посредник* убирает прямую связь между отправителями и
        получателями, заставляя их общаться опосредованно, через себя.
    -   *Наблюдатель* передаёт запрос одновременно всем заинтересованным
        получателям, но позволяет им динамически подписываться или
        отписываться от таких оповещений.

-   Обработчики в [Цепочке обязанностей](chain-of-responsibility.html)
    могут быть выполнены в виде [Команд](command.html). В этом случае
    множество разных операций может быть выполнено над одним и тем же
    контекстом, коим является запрос.

    Но есть и другой подход, в котором сам запрос является *Командой*,
    посланной по цепочке объектов. В этом случае одна и та же операция
    может быть выполнена над множеством разных контекстов,
    представленных в виде цепочки.

-   [Команду](command.html) и [Снимок](memento.html) можно использовать
    сообща для реализации отмены операций. В этом случае объекты команд
    будут отвечать за выполнение действия над объектом, а снимки будут
    хранить резервную копию состояния этого объекта, сделанную перед
    самым запуском команды.

-   [Команда](command.html) и [Стратегия](strategy.html) похожи по духу,
    но отличаются масштабом и применением:

    -   *Команду* используют, чтобы превратить любые разнородные
        действия в объекты. Параметры операции превращаются в поля
        объекта. Этот объект теперь можно логировать, хранить в истории
        для отмены, передавать во внешние сервисы и так далее.
    -   С другой стороны, *Стратегия* описывает разные способы
        произвести одно и то же действие, позволяя взаимозаменять эти
        способы в каком-то объекте контекста.

-   Если [Команду](command.html) нужно копировать перед вставкой в
    историю выполненных команд, вам может помочь
    [Прототип](prototype.html).

-   [Посетитель](visitor.html) можно рассматривать как расширенный
    аналог [Команды](command.html), который способен работать сразу с
    несколькими видами получателей.
:::

::: {.section .implementations}
##  Примеры реализации паттерна {#implementations}

[![Команда на
C#](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](command/csharp/example.html "Команда на C#"){.prog-lang-link}
[![Команда на
C++](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](command/cpp/example.html "Команда на C++"){.prog-lang-link}
[![Команда на
Go](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](command/go/example.html "Команда на Go"){.prog-lang-link}
[![Команда на
Java](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](command/java/example.html "Команда на Java"){.prog-lang-link}
[![Команда на
PHP](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](command/php/example.html "Команда на PHP"){.prog-lang-link}
[![Команда на
Python](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](command/python/example.html "Команда на Python"){.prog-lang-link}
[![Команда на
Ruby](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](command/ruby/example.html "Команда на Ruby"){.prog-lang-link}
[![Команда на
Rust](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](command/rust/example.html "Команда на Rust"){.prog-lang-link}
[![Команда на
Swift](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](command/swift/example.html "Команда на Swift"){.prog-lang-link}
[![Команда на
TypeScript](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](command/typescript/example.html "Команда на TypeScript"){.prog-lang-link}
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

[Итератор []{.fa .fa-arrow-right}](iterator.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Цепочка
обязанностей](chain-of-responsibility.html){.btn .btn-default
rel="prev"}
:::
::::::::::::::::::::::::::::::::::

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
::::::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::::::
