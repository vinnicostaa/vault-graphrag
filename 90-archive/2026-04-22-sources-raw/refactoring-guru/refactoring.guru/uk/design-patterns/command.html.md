::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [Патерни
проектування](../design-patterns.html){.type} / [Поведінкові
патерни](behavioral-patterns.html){.type}
:::

# Команда {#команда .title}

::: aka
Також відомий як:
[Дія, ]{style="display:inline-block"}[Транзакція, ]{style="display:inline-block"}[Action, ]{style="display:inline-block"}[Command]{style="display:inline-block"}
:::

::: {.section .intent}
##  Суть патерна {#intent}

**Команда** --- це поведінковий патерн проектування, який перетворює
запити на об'єкти, дозволяючи передавати їх як аргументи під час виклику
методів, ставити запити в чергу, логувати їх, а також підтримувати
скасування операцій.

<figure class="image">
<img
src="../../images/patterns/content/command/command-en99f0.png?id=80fbadc666cf3b9b1958c546d2746ca4"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/command/command-en.png?id=80fbadc666cf3b9b1958c546d2746ca4 640w,/images/patterns/content/command/command-en-1.5x.png?id=c3e1a91b500ce88f9b8fda6176762698 960w,/images/patterns/content/command/command-en-2x.png?id=6149af804cbbbd5cb18595c30b856d89 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Патерн Команда" />
</figure>
:::

::: {.section .problem}
##  Проблема {#problem}

Уявіть, що ви працюєте над програмою текстового редактора. Якраз
підійшов час розробки панелі керування. Ви створили клас гарних `Кнопок`
і хочете використовувати його для всіх кнопок програми, починаючи з
панелі керування та закінчуючи звичайними кнопками в діалогах.

<figure class="image">
<img
src="../../images/patterns/diagrams/command/problem163e2.png?id=84189315a0e8d91da262792979005ab4"
style="aspect-ratio: 0.88;"
srcset="/images/patterns/diagrams/command/problem1.png?id=84189315a0e8d91da262792979005ab4 230w,/images/patterns/diagrams/command/problem1-1.5x.png?id=77bf0bc58e5c9c9b8369bc4f8dec457f 345w,/images/patterns/diagrams/command/problem1-2x.png?id=af4c4e9c1d1b4fa2c4104c5f6bb18719 460w"
sizes="(max-width: 720px) 100vw, 230px" loading="lazy" width="230"
alt="Проблема, яку вирішує Команда" />
<figcaption><p>Всі кнопки програми успадковані від
одного класу.</p></figcaption>
</figure>

Усі ці кнопки, хоч і виглядають схоже, але виконують різні команди.
Виникає запитання: куди розмістити код обробників кліків по цих кнопках?
Найпростіше рішення --- це створити підкласи для кожної кнопки та
перевизначити в них методи дії для різних завдань.

<figure class="image">
<img
src="../../images/patterns/diagrams/command/problem2f2f9.png?id=f0e33da1842b3a3ee3b4857de0b6ec93"
style="aspect-ratio: 2.11;"
srcset="/images/patterns/diagrams/command/problem2.png?id=f0e33da1842b3a3ee3b4857de0b6ec93 400w,/images/patterns/diagrams/command/problem2-1.5x.png?id=7ae15e2b07d5d076a878d99c3ed8ac32 600w,/images/patterns/diagrams/command/problem2-2x.png?id=5eea7d0f6247da011150bb63e423f405 800w"
sizes="(max-width: 720px) 100vw, 400px" loading="lazy" width="400"
alt="Безліч підкласів кнопок" />
<figcaption><p>Безліч підкласів кнопок.</p></figcaption>
</figure>

Але скоро стало зрозуміло, що такий підхід нікуди не годиться. По-перше,
з'являється дуже багато підкласів. По-друге, код кнопок, який
відноситься до графічного інтерфейсу, починає залежати від класів
бізнес-логіки, яка досить часто змінюється.

<figure class="image">
<img
src="../../images/patterns/diagrams/command/problem3-enf2d0.png?id=1fbd56ae7ba3e3dfac45cfa2de6db450"
style="aspect-ratio: 4.36;"
srcset="/images/patterns/diagrams/command/problem3-en.png?id=1fbd56ae7ba3e3dfac45cfa2de6db450 480w,/images/patterns/diagrams/command/problem3-en-1.5x.png?id=2da629d84c7cbabbca11562a3d7d2559 720w,/images/patterns/diagrams/command/problem3-en-2x.png?id=e06878f7cfbe4131980c68db2904878e 960w"
sizes="(max-width: 720px) 100vw, 480px" loading="lazy" width="480"
alt="Кілька класів дублюють одну і ту саму функціональність" />
<figcaption><p>Кілька класів дублюють одну і ту
саму функціональність.</p></figcaption>
</figure>

Проте, найгірше ще попереду, адже деякі операції, на кшталт «зберегти»,
можна викликати з декількох місць: натиснувши кнопку на панелі
керування, викликавши контекстне меню або натиснувши клавіші `Ctrl+S`.
Коли в програмі були тільки кнопки, код збереження був тільки у підкласі
`SaveButton`. Але тепер його доведеться продублювати ще в два класи.
:::

::: {.section .solution}
##  Рішення {#solution}

Хороші програми зазвичай структурують у вигляді шарів. Найпоширеніший
приклад --- це шари користувацького інтерфейсу та бізнес-логіки. Перший
лише малює гарне зображення для користувача, але коли потрібно зробити
щось важливе, інтерфейс користувача «просить» шар бізнес-логіки
зайнятися цим.

У дійсності це виглядає так: один з об'єктів інтерфейсу користувача
викликає метод одного з об'єктів бізнес-логіки, передаючи до нього якісь
параметри.

<figure class="image">
<img
src="../../images/patterns/diagrams/command/solution1-uk5f9a.png?id=23df7da8869341442cf78df9c42fc4cb"
style="aspect-ratio: 2.24;"
srcset="/images/patterns/diagrams/command/solution1-uk.png?id=23df7da8869341442cf78df9c42fc4cb 470w,/images/patterns/diagrams/command/solution1-uk-1.5x.png?id=5bc03497dd6e5c18b19c7faa795702f2 705w,/images/patterns/diagrams/command/solution1-uk-2x.png?id=4aa4cd7c2bb239d617638a395cddbd6f 940w"
sizes="(max-width: 720px) 100vw, 470px" loading="lazy" width="470"
alt="Прямий доступ з UI до бізнес-логіки" />
<figcaption><p>Прямий доступ з UI до бізнес-логіки.</p></figcaption>
</figure>

Патерн Команда пропонує більше не надсилати такі виклики безпосередньо.
Замість цього кожен виклик, що відрізняється від інших, слід звернути у
власний клас з єдиним методом, який і здійснюватиме виклик. Такий об'єкт
зветься *командою*.

До об'єкта інтерфейсу можна буде прив'язати об'єкт команди, який знає,
кому і в якому вигляді слід відправляти запити. Коли об'єкт інтерфейсу
буде готовий передати запит, він викличе метод команди, а та --- подбає
про все інше.

<figure class="image">
<img
src="../../images/patterns/diagrams/command/solution2-ukad4a.png?id=924a88c7d615bc0c0ec40b608ca53459"
style="aspect-ratio: 2.62;"
srcset="/images/patterns/diagrams/command/solution2-uk.png?id=924a88c7d615bc0c0ec40b608ca53459 550w,/images/patterns/diagrams/command/solution2-uk-1.5x.png?id=0e1342d824b1f22bc58c46ada3eea38c 825w,/images/patterns/diagrams/command/solution2-uk-2x.png?id=19e07e02f213791f8ecb77dd39790863 1100w"
sizes="(max-width: 720px) 100vw, 550px" loading="lazy" width="550"
alt="Доступ з UI до бізнес-логіки через команду" />
<figcaption><p>Доступ з UI до бізнес-логіки
через команду.</p></figcaption>
</figure>

Класи команд можна об'єднати під загальним інтерфейсом, що має єдиний
метод запуску команди. Після цього одні й ті самі відправники зможуть
працювати з різними командами, не прив'язуючись до їхніх класів. Навіть
більше, команди можна буде взаємозаміняти «на льоту», змінюючи
підсумкову поведінку відправників.

Параметри, з якими повинен бути викликаний метод об'єкта одержувача,
можна заздалегідь зберегти в полях об'єкта-команди. Завдяки цьому,
об'єкти, які надсилають запити, можуть не турбуватися про те, щоб
зібрати необхідні дані для одержувача. Навіть більше, вони тепер взагалі
не знають, хто буде одержувачем запиту. Вся ця інформація прихована
всередині команди.

<figure class="image">
<img
src="../../images/patterns/diagrams/command/solution3-endc8e.png?id=c92f6874b95de46b041499d41234b00b"
style="aspect-ratio: 1.83;"
srcset="/images/patterns/diagrams/command/solution3-en.png?id=c92f6874b95de46b041499d41234b00b 440w,/images/patterns/diagrams/command/solution3-en-1.5x.png?id=580ba981ae73f0608513349ecf38ad90 660w,/images/patterns/diagrams/command/solution3-en-2x.png?id=c12bb9971d1ba4f8a3d3717bbced2859 880w"
sizes="(max-width: 720px) 100vw, 440px" loading="lazy" width="440"
alt="Класи UI делегують роботу командам" />
<figcaption><p>Класи UI делегують роботу командам.</p></figcaption>
</figure>

Після застосування Команди в нашому прикладі з текстовим редактором вам
більше не потрібно буде створювати безліч підкласів кнопок для різних
дій. Буде достатньо одного класу з полем для зберігання об'єкта команди.

Використовуючи загальний інтерфейс команд, об'єкти кнопок посилатимуться
на об'єкти команд різних типів. При натисканні кнопки делегуватимуть
роботу командам, а команди --- перенаправляти виклики тим чи іншим
об'єктам бізнес-логіки.

Так само можна вчинити і з контекстним меню, і з гарячими клавішами.
Вони будуть прив'язані до тих самих об'єктів команд, що і кнопки,
позбавляючи класи від дублювання.

Таким чином, команди стануть гнучким прошарком між користувацьким
інтерфейсом та бізнес-логікою. І це лише невелика частина тієї користі,
яку може принести патерн Команда!
:::

::: {.section .analogy}
##  Аналогія з життя {#analogy}

<figure class="image">
<img
src="../../images/patterns/content/command/command-comic-16e6f.png?id=551df832f445080976f3116e0dc120c9"
style="aspect-ratio: 2;"
srcset="/images/patterns/content/command/command-comic-1.png?id=551df832f445080976f3116e0dc120c9 600w,/images/patterns/content/command/command-comic-1-1.5x.png?id=82dc5c38ce372ed4dfd4a37c04faeb72 900w,/images/patterns/content/command/command-comic-1-2x.png?id=47b3c00b2cfbda7157a1ed9da6ce2948 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Приклад замовлення в ресторані" />
<figcaption><p>Приклад замовлення в ресторані.</p></figcaption>
</figure>

Ви заходите в ресторан і сідаєте біля вікна. До вас підходить ввічливий
офіціант і приймає замовлення, записуючи всі побажання в блокнот.

Закінчивши, він поспішає на кухню, вириває аркуш з блокнота та клеїть
його на стіну. Далі лист опиняється в руках кухаря, який читає
замовлення і готує описану страву.

У цьому прикладі ви є *відправником*, офіціант з блокнотом ---
*командою*, а кухар --- *отримувачем*. Як і в самому патерні, ви не
стикаєтесь з кухарем безпосередньо. Замість цього ви відправляєте
замовлення офіціантом, який самостійно «налаштовує» кухаря на роботу. З
іншого боку, кухар не знає, хто конкретно надіслав йому замовлення. Але
йому це байдуже, бо вся необхідна інформація є в листі замовлення.
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
alt="Структура класів патерна Команда" /><img
src="../../images/patterns/diagrams/command/structure-indexed49b8.png?id=95529d7282dc7bc1c5bc443423b1cf4f"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.64;"
srcset="/images/patterns/diagrams/command/structure-indexed.png?id=95529d7282dc7bc1c5bc443423b1cf4f 640w,/images/patterns/diagrams/command/structure-indexed-1.5x.png?id=29d6c5c7a06da2747ed756c0ddad6169 960w,/images/patterns/diagrams/command/structure-indexed-2x.png?id=e4cc286a44768c7d060af33497da7df6 1280w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="640"
alt="Структура класів патерна Команда" />
</figure>
:::

1.  **Відправник** зберігає посилання на об'єкт команди та звертається
    до нього, коли потрібно виконати якусь дію. Відправник працює з
    командами тільки через їхній загальний інтерфейс. Він не знає, яку
    конкретно команду використовує, оскільки отримує готовий об'єкт
    команди від клієнта.

2.  **Команда** описує інтерфейс, спільний для всіх конкретних команд.
    Зазвичай тут описується лише один метод запуску команди.

3.  **Конкретні команди** реалізують різні запити, дотримуючись
    загального інтерфейсу команд. Як правило, команда не робить всю
    роботу самостійно, а лише передає виклик одержувачу, яким виступає
    один з об'єктів бізнес-логіки.

    Параметри, з якими команда звертається до одержувача, необхідно
    зберігати у вигляді полів. У більшості випадків об'єкти команд можна
    зробити незмінними, передаючи у них всі необхідні параметри тільки
    через конструктор.

4.  **Одержувач** містить бізнес-логіку програми. У цій ролі може
    виступати практично будь-який об'єкт. Зазвичай, команди
    перенаправляють виклики одержувачам, але іноді, щоб спростити
    програму, ви можете позбутися від одержувачів, «зливши» їхній код у
    класи команд.

5.  **Клієнт** створює об'єкти конкретних команд, передаючи до них усі
    необхідні параметри, серед яких можуть бути і посилання на об'єкти
    одержувачів. Після цього клієнт зв'язує об'єкти відправників зі
    створеними командами.
::::
:::::

::: {.section .pseudocode}
##  Псевдокод {#pseudocode}

У цьому прикладі патерн **Команда** використовується для ведення історії
виконаних операцій, дозволяючи скасовувати їх за потреби.

<figure class="image">
<img
src="../../images/patterns/diagrams/command/example31bf.png?id=1f42c8395fe54d0e409026b91881e2a0"
style="aspect-ratio: 1.07;"
srcset="/images/patterns/diagrams/command/example.png?id=1f42c8395fe54d0e409026b91881e2a0 640w,/images/patterns/diagrams/command/example-1.5x.png?id=435055a78a82c005eebb1bef869998ae 960w,/images/patterns/diagrams/command/example-2x.png?id=ed44dfd9b02eccf1ae7e2714d018ed36 1280w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="640"
alt="Структура класів прикладу патерна Команда" />
<figcaption><p>Приклад реалізації скасування у
текстовому редакторі.</p></figcaption>
</figure>

Команди, які змінюють стан редактора (наприклад, команда вставки тексту
з буфера обміну), зберігають копію стану редактора перед виконанням дії.
Копії виконаних команд розміщуються в історії команд, звідки вони можуть
бути доставлені, якщо потрібно буде скасувати виконану операцію.

Класи елементів інтерфейсу, історії команд та інші не залежать від
конкретних класів команд, оскільки працюють з ними через загальний
інтерфейс. Це дозволяє додавати до програми нові команди, не змінюючи
наявний код.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// Абстрактна команда задає загальний інтерфейс для конкретних
// класів команд, а також містить реалізацію базової поведінки
// скасування операції.
abstract class Command is
    protected field app: Application
    protected field editor: Editor
    protected field backup: text

    constructor Command(app: Application, editor: Editor) is
        this.app = app
        this.editor = editor

    // Зберігаємо стан редактора.
    method saveBackup() is
        backup = editor.text

    // Відновлюємо стан редактора.
    method undo() is
        editor.text = backup

    // Головний метод команди залишається абстрактним, щоб кожна
    // конкретна команда визначила його по-своєму. Метод повинен
    // повернути true або false, залежно від того, чи змінила
    // команда стан редактора, а отже, чи потрібно її зберігати
    // в історії.
    abstract method execute()


// Конкретні команди.
class CopyCommand extends Command is
    // Команда копіювання не записується до історії, бо вона не
    // змінює стан редактора.
    method execute() is
        app.clipboard = editor.getSelection()
        return false

class CutCommand extends Command is
    // Команди, що змінюють стан редактора, зберігають стан
    // редактора перед своєю дією і сигналізують про зміну,
    // повертаючи true.
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

// Відміна — це також команда.
class UndoCommand extends Command is
    method execute() is
        app.undo()
        return false


// Глобальна історія команд — це стек.
class CommandHistory is
    private field history: array of Command

    // Той, що зайшов останнім...
    method push(c: Command) is
        // Додати команду в кінець масиву-історії.

    // ...виходить першим.
    method pop():Command is
        // Дістати останню команду з масиву-історії.


// Клас редактора містить безпосередні операції над текстом. Він
// відіграє роль одержувача — команди делегують йому свої дії.
class Editor is
    field text: string

    method getSelection() is
        // Повернути вибраний текст.

    method deleteSelection() is
        // Видалити вибраний текст.

    method replaceSelection(text) is
        // Вкласти текст з буфера обміну в поточній позиції.


// Клас програми налаштовує об&#39;єкти для спільної роботи. Він
// виступає у ролі відправника — створює команди, щоб виконати
// якісь дії.
class Application is
    field clipboard: string
    field editors: array of Editors
    field activeEditor: Editor
    field history: CommandHistory

    // Код, що прив&#39;язує команди до елементів інтерфейсу, може
    // виглядати приблизно так.
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

    // Запускаємо команду й перевіряємо, чи потрібно додати її
    // до історії.
    method executeCommand(command) is
        if (command.execute())
            history.push(command)

    // Беремо останню команду з історії та змушуємо її все
    // скасувати. Ми не знаємо конкретний тип команди, але це і
    // не важливо, оскільки кожна команда знає, як скасувати
    // свою дію.
    method undo() is
        command = history.pop()
        if (command != null)
            command.undo()</code></pre>
</figure>
:::

:::::::::: {.section .applicability-container}
##  Застосування {#applicability}

::::::::: applicability
::: applicability-problem
Якщо ви хочете параметризувати об'єкти виконуваною дією.
:::

::: applicability-solution
Команда перетворює операції на об'єкти, а об'єкти, у свою чергу, можна
передавати, зберігати та взаємозаміняти всередині інших об'єктів.

Скажімо, ви розробляєте бібліотеки графічного меню і хочете, щоб
користувачі могли використовувати меню в різних програмах, не змінюючи
кожного разу код ваших класів. Застосувавши патерн, користувачам не
доведеться змінювати класи меню, замість цього вони будуть конфігурувати
об'єкти меню різними командами.
:::

::: applicability-problem
Якщо ви хочете поставити операції в чергу, виконувати їх за розкладом
або передавати мережею.
:::

::: applicability-solution
Як і будь-які інші об'єкти, команди можна серіалізувати, тобто
перетворити на рядок, щоб потім зберегти у файл або базу даних. Потім в
будь-який зручний момент його можна дістати назад, знову перетворити на
об'єкт команди та виконати. Так само команди можна передавати мережею,
логувати або виконувати на віддаленому сервері.
:::

::: applicability-problem
Якщо вам потрібна операція скасування.
:::

::: applicability-solution
Головна річ, яка потрібна для того, щоб мати можливість скасовувати
операції --- це зберігання історії. Серед багатьох способів реалізації
цієї можливості патерн Команда є, мабуть, найпопулярнішим.

Історія команд виглядає як стек, до якого потрапляють усі виконані
об'єкти команд. Кожна команда перед виконанням операції зберігає
поточний стан об'єкта, з яким вона працюватиме. Після виконання операції
копія команди потрапляє до стеку історії, продовжуючи нести у собі
збережений стан об'єкта. Якщо знадобиться скасування, програма візьме
останню команду з історії та відновить збережений у ній стан.

Цей спосіб має дві особливості. По-перше, точний стан об'єктів не дуже
просто зберегти, адже його частина може бути приватною. Вирішити це
можна за допомогою патерна [Знімок](memento.html).

По-друге, копії стану можуть займати досить багато оперативної пам'яті.
Тому іноді можна вдатися до альтернативної реалізації, тобто замість
відновлення старого стану, команда виконає зворотню дію. Недолік цього
способу у складності (іноді неможливості) реалізації зворотньої дії.
:::
:::::::::
::::::::::

::: {.section .checklist}
##  Кроки реалізації {#checklist}

1.  Створіть загальний інтерфейс команд і визначте в ньому метод
    запуску.

2.  Один за одним створіть класи конкретних команд. У кожному класі має
    бути поле для зберігання посилання на один або декілька
    об'єктів-одержувачів, яким команда перенаправлятиме основну роботу.

    Крім цього, команда повинна мати поля для зберігання параметрів,
    потрібних під час виклику методів одержувача. Значення всіх цих
    полів команда повинна отримувати через конструктор.

    І, нарешті, реалізуйте основний метод команди, викликаючи в ньому ті
    чи інші методи одержувача.

3.  Додайте до класів відправників поля для зберігання команд. Зазвичай
    об'єкти-відправники приймають готові об'єкти команд ззовні --- через
    конструктор або через сетер поля команди.

4.  Змініть основний код відправників так, щоб вони делегували виконання
    дії команді.

5.  Порядок ініціалізації об'єктів повинен виглядати так:

    -   Створюємо об'єкти одержувачів.
    -   Створюємо об'єкти команд, зв'язавши їх з одержувачами.
    -   Створюємо об'єкти відправників, зв'язавши їх з командами.
:::

:::::: {.section .pros-cons}
##  Переваги та недоліки {#pros-cons}

::::: row
::: col-sm-6
-    Прибирає пряму залежність між об'єктами, що викликають операції, та
    об'єктами, які їх безпосередньо виконують.
-    Дозволяє реалізувати просте скасування і повтор операцій.
-    Дозволяє реалізувати відкладений запуск операцій.
-    Дозволяє збирати складні команди з простих.
-    Реалізує *принцип відкритості/закритості*.
:::

::: col-sm-6
-    Ускладнює код програми внаслідок введення великої кількості
    додаткових класів.
:::
:::::
::::::

::: {.section .relations}
##  Відносини з іншими патернами {#relations}

-   [Ланцюжок обов'язків](chain-of-responsibility.html),
    [Команда](command.html) [Посередник](mediator.html) та
    [Спостерігач](observer.html) показують різні способи роботи тих, хто
    надсилає запити, та тих, хто їх отримує:

    -   *Ланцюжок обов'язків* передає запит послідовно через ланцюжок
        потенційних отримувачів, очікуючи, що один з них обробить запит.
    -   *Команда* встановлює непрямий односторонній зв'язок від
        відправників до одержувачів.
    -   *Посередник* прибирає прямий зв'язок між відправниками та
        одержувачами, змушуючи їх спілкуватися опосередковано, через
        себе.
    -   *Спостерігач* передає запит одночасно всім зацікавленим
        одержувачам, але дозволяє їм динамічно підписуватися або
        відписуватися від таких повідомлень.

-   Обробники в [Ланцюжкові обов'язків](chain-of-responsibility.html)
    можуть бути виконані у вигляді [Команд](command.html). В цьому
    випадку роль запиту відіграє контекст команд, який послідовно
    подається до кожної команди у ланцюгу.

    Але є й інший підхід, в якому сам запит є *Командою*, надісланою
    ланцюжком об'єктів. У цьому випадку одна і та сама операція може
    бути застосована до багатьох різних контекстів, представлених у
    вигляді ланцюжка.

-   [Команду](command.html) та [Знімок](memento.html) можна
    використовувати спільно для реалізації скасування операцій. У цьому
    випадку об'єкти команд відповідатимуть за виконання дії над
    об'єктом, а знімки зберігатимуть резервну копію стану цього об'єкта,
    зроблену перед запуском команди.

-   [Команда](command.html) та [Стратегія](strategy.html) схожі за
    принципом, але відрізняються масштабом та застосуванням:

    -   *Команду* використовують для перетворення будь-яких різнорідних
        дій на об'єкти. Параметри операції перетворюються на поля
        об'єкта. Цей об'єкт тепер можна логувати, зберігати в історії
        для скасування, передавати у зовнішні сервіси тощо.
    -   З іншого боку, *Стратегія* описує різні способи того, як зробити
        одну і ту саму дію, дозволяючи замінювати ці способи в якомусь
        об'єкті контексту прямо під час виконання програми.

-   Якщо [Команду](command.html) потрібно копіювати перед вставкою в
    історію виконаних команд, вам може допомогти
    [Прототип](prototype.html).

-   [Відвідувач](visitor.html) можна розглядати як розширений аналог
    [Команди](command.html), що здатен працювати відразу з декількома
    видами одержувачів.
:::

::: {.section .implementations}
##  Приклади реалізації патерна {#implementations}

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

[Ітератор []{.fa .fa-arrow-right}](iterator.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Повернутись назад

[[]{.fa .fa-arrow-left} Ланцюжок
обов\'язків](chain-of-responsibility.html){.btn .btn-default rel="prev"}
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
