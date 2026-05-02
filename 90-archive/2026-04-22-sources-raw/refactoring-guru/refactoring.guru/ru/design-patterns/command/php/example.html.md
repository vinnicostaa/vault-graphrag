:::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Паттерны
проектирования](../../../design-patterns.html){.type} /
[Команда](../../command.html){.type} / [PHP](../../php.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Команда](../../../../images/patterns/cards/command-mini8482.png?id=b149eda017c0583c1e92343b83cfb1eb){srcset="/images/patterns/cards/command-mini-2x.png?id=e5f6332e057f6d352a209da963a8fc54 2x"}
:::

# **Команда** на PHP {#команда-на-php .pattern-example-title-block-title}
::::

:::::::::::::: pattern-example-body
::: pattern-example-brief
**Команда** --- это поведенческий паттерн, позволяющий заворачивать
запросы или простые операции в отдельные объекты.

Это позволяет откладывать выполнение команд, выстраивать их в очереди, а
также хранить историю и делать отмену.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Подробней о паттерне Команда ](../../command.html){.btn .btn-lg
.btn-primary}
:::

::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
Навигация
:::

::: en-intro
 [Интро](#)
:::

::: en-intro
 [Концептуальный пример](#example-0)
:::

::: en-file
 [index](#example-0--index-php)
:::

::: en-file
 [Output](#example-0--Output-txt)
:::

::: en-intro
 [Пример из реальной жизни](#example-1)
:::

::: en-file
 [index](#example-1--index-php)
:::

::: en-file
 [Output](#example-1--Output-txt)
:::
:::::::::::

**Сложность:** []{.fa-stars}

**Популярность:** []{.fa-stars}

**Применимость:** Паттерн можно часто встретить в PHP-коде, особенно
когда нужно откладывать выполнение команд, выстраивать их в очереди, а
также хранить историю и делать отмену.

**Признаки применения паттерна:** Классы команд построены вокруг одного
действия и имеют очень узкий контекст. Объекты команд часто подаются в
обработчики событий элементов GUI. Практически любая реализация отмены
использует принципа команд.
::::::::::::::

::: {#nav-tab .nav .nav-tabs .nav-justified role="tablist"}
[Концептуальный пример](#example-0){#example-0-tab .nav-item .nav-link
.active toggle="tab" role="tab" aria-controls="example-0"
aria-selected="true"} [Пример из реальной
жизни](#example-1){#example-1-tab .nav-item .nav-link toggle="tab"
role="tab" aria-controls="example-1" aria-selected="true"}
:::

::::: {#nav-tabContent .pattern-example-tab-content .tab-content}
::: {#example-0 .tab-pane .show .active role="tabpanel" aria-labelledby="example-0-tab" style="padding-top: 24px"}
## Концептуальный пример {#example-0-title}

Этот пример показывает структуру паттерна **Команда**, а именно --- из
каких классов он состоит, какие роли эти классы выполняют и как они
взаимодействуют друг с другом.

После ознакомления со структурой, вам будет легче воспринимать второй
пример, который рассматривает реальный случай использования паттерна в
мире PHP.

#### []{#example-0--index-php .anchor} **index.php:** Пример структуры паттерна

<figure class="code">
<pre class="code" lang="php"><code>&lt;?php

namespace RefactoringGuru\Command\Conceptual;

/**
 * Интерфейс Команды объявляет метод для выполнения команд.
 */
interface Command
{
    public function execute(): void;
}

/**
 * Некоторые команды способны выполнять простые операции самостоятельно.
 */
class SimpleCommand implements Command
{
    private $payload;

    public function __construct(string $payload)
    {
        $this-&gt;payload = $payload;
    }

    public function execute(): void
    {
        echo &quot;SimpleCommand: See, I can do simple things like printing (&quot; . $this-&gt;payload . &quot;)\n&quot;;
    }
}

/**
 * Но есть и команды, которые делегируют более сложные операции другим объектам,
 * называемым «получателями».
 */
class ComplexCommand implements Command
{
    /**
     * @var Receiver
     */
    private $receiver;

    /**
     * Данные о контексте, необходимые для запуска методов получателя.
     */
    private $a;

    private $b;

    /**
     * Сложные команды могут принимать один или несколько объектов-получателей
     * вместе с любыми данными о контексте через конструктор.
     */
    public function __construct(Receiver $receiver, string $a, string $b)
    {
        $this-&gt;receiver = $receiver;
        $this-&gt;a = $a;
        $this-&gt;b = $b;
    }

    /**
     * Команды могут делегировать выполнение любым методам получателя.
     */
    public function execute(): void
    {
        echo &quot;ComplexCommand: Complex stuff should be done by a receiver object.\n&quot;;
        $this-&gt;receiver-&gt;doSomething($this-&gt;a);
        $this-&gt;receiver-&gt;doSomethingElse($this-&gt;b);
    }
}

/**
 * Классы Получателей содержат некую важную бизнес-логику. Они умеют выполнять
 * все виды операций, связанных с выполнением запроса. Фактически, любой класс
 * может выступать Получателем.
 */
class Receiver
{
    public function doSomething(string $a): void
    {
        echo &quot;Receiver: Working on (&quot; . $a . &quot;.)\n&quot;;
    }

    public function doSomethingElse(string $b): void
    {
        echo &quot;Receiver: Also working on (&quot; . $b . &quot;.)\n&quot;;
    }
}

/**
 * Отправитель связан с одной или несколькими командами. Он отправляет запрос
 * команде.
 */
class Invoker
{
    /**
     * @var Command
     */
    private $onStart;

    /**
     * @var Command
     */
    private $onFinish;

    /**
     * Инициализация команд.
     */
    public function setOnStart(Command $command): void
    {
        $this-&gt;onStart = $command;
    }

    public function setOnFinish(Command $command): void
    {
        $this-&gt;onFinish = $command;
    }

    /**
     * Отправитель не зависит от классов конкретных команд и получателей.
     * Отправитель передаёт запрос получателю косвенно, выполняя команду.
     */
    public function doSomethingImportant(): void
    {
        echo &quot;Invoker: Does anybody want something done before I begin?\n&quot;;
        if ($this-&gt;onStart instanceof Command) {
            $this-&gt;onStart-&gt;execute();
        }

        echo &quot;Invoker: ...doing something really important...\n&quot;;

        echo &quot;Invoker: Does anybody want something done after I finish?\n&quot;;
        if ($this-&gt;onFinish instanceof Command) {
            $this-&gt;onFinish-&gt;execute();
        }
    }
}

/**
 * Клиентский код может параметризовать отправителя любыми командами.
 */
$invoker = new Invoker();
$invoker-&gt;setOnStart(new SimpleCommand(&quot;Say Hi!&quot;));
$receiver = new Receiver();
$invoker-&gt;setOnFinish(new ComplexCommand($receiver, &quot;Send email&quot;, &quot;Save report&quot;));

$invoker-&gt;doSomethingImportant();</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** Результат выполнения

<figure class="code">
<pre class="code" lang="output"><code>Invoker: Does anybody want something done before I begin?
SimpleCommand: See, I can do simple things like printing (Say Hi!)
Invoker: ...doing something really important...
Invoker: Does anybody want something done after I finish?
ComplexCommand: Complex stuff should be done by a receiver object.
Receiver: Working on (Send email.)
Receiver: Also working on (Save report.)</code></pre>
</figure>
:::

::: {#example-1 .tab-pane role="tabpanel" aria-labelledby="example-1-tab" style="padding-top: 24px"}
## Пример из реальной жизни {#example-1-title}

В этом примере паттерн **Команда** применяется для построения очереди из
вызовов скрейпинга (скачивания) отдельных страниц сайта IMDB и
выполнения их один за другим. Сама очередь хранится в базе данных,
которая помогает не терять команды между запусками скрипта.

#### []{#example-1--index-php .anchor} **index.php:** Пример из реальной жизни

<figure class="code">
<pre class="code" lang="php"><code>&lt;?php

namespace RefactoringGuru\Command\RealWorld;

/**
 * Интерфейс Команды объявляет основной метод выполнения, а также несколько
 * вспомогательных методов для получения метаданных команды.
 */
interface Command
{
    public function execute(): void;

    public function getId(): int;

    public function getStatus(): int;
}

/**
 * Базовая Команда скрейпинга устанавливает базовую инфраструктуру загрузки,
 * общую для всех конкретных команд скрейпинга.
 */
abstract class WebScrapingCommand implements Command
{
    public $id;

    public $status = 0;

    /**
     * @var string URL для скрейпинга.
     */
    public $url;

    public function __construct(string $url)
    {
        $this-&gt;url = $url;
    }

    public function getId(): int
    {
        return $this-&gt;id;
    }

    public function getStatus(): int
    {
        return $this-&gt;status;
    }

    public function getURL(): string
    {
        return $this-&gt;url;
    }

    /**
     * Поскольку методы выполнения для всех команд скрейпинга очень похожи, мы
     * можем предоставить реализацию по умолчанию, позволив подклассам
     * переопределить её при необходимости.
     *
     * Шш! Наблюдательный читатель может обнаружить здесь другой поведенческий
     * паттерн в действии.
     */
    public function execute(): void
    {
        $html = $this-&gt;download();
        $this-&gt;parse($html);
        $this-&gt;complete();
    }

    public function download(): string
    {
        $html = file_get_contents($this-&gt;getURL());
        echo &quot;WebScrapingCommand: Downloaded {$this-&gt;url}\n&quot;;

        return $html;
    }

    abstract public function parse(string $html): void;

    public function complete(): void
    {
        $this-&gt;status = 1;
        Queue::get()-&gt;completeCommand($this);
    }
}

/**
 * Конкретная Команда для извлечения списка жанров фильма.
 */
class IMDBGenresScrapingCommand extends WebScrapingCommand
{
    public function __construct()
    {
        $this-&gt;url = &quot;https://www.imdb.com/feature/genre/&quot;;
    }

    /**
     * Извлечение всех жанров и их поисковых URL со страницы:
     * https://www.imdb.com/feature/genre/
     */
    public function parse($html): void
    {
        preg_match_all(&quot;|href=\&quot;(https://www.imdb.com/search/title\?genres=.*?)\&quot;|&quot;, $html, $matches);
        echo &quot;IMDBGenresScrapingCommand: Discovered &quot; . count($matches[1]) . &quot; genres.\n&quot;;

        foreach ($matches[1] as $genre) {
            Queue::get()-&gt;add(new IMDBGenrePageScrapingCommand($genre));
        }
    }
}

/**
 * Конкретная Команда для извлечения списка фильмов определённого жанра.
 */
class IMDBGenrePageScrapingCommand extends WebScrapingCommand
{
    private $page;

    public function __construct(string $url, int $page = 1)
    {
        parent::__construct($url);
        $this-&gt;page = $page;
    }

    public function getURL(): string
    {
        return $this-&gt;url . &#39;?page=&#39; . $this-&gt;page;
    }

    /**
     * Извлечение всех фильмов со страницы вроде этой:
     * https://www.imdb.com/search/title?genres=sci-fi&amp;explore=title_type,genres
     */
    public function parse(string $html): void
    {
        preg_match_all(&quot;|href=\&quot;(/title/.*?/)\?ref_=adv_li_tt\&quot;|&quot;, $html, $matches);
        echo &quot;IMDBGenrePageScrapingCommand: Discovered &quot; . count($matches[1]) . &quot; movies.\n&quot;;

        foreach ($matches[1] as $moviePath) {
            $url = &quot;https://www.imdb.com&quot; . $moviePath;
            Queue::get()-&gt;add(new IMDBMovieScrapingCommand($url));
        }

        // Извлечение URL следующей страницы.
        if (preg_match(&quot;|Next &amp;#187;&lt;/a&gt;|&quot;, $html)) {
            Queue::get()-&gt;add(new IMDBGenrePageScrapingCommand($this-&gt;url, $this-&gt;page + 1));
        }
    }
}

/**
 * Конкретная Команда для извлечения подробных сведений о фильме.
 */
class IMDBMovieScrapingCommand extends WebScrapingCommand
{
    /**
     * Получить информацию о фильме с подобной страницы:
     * https://www.imdb.com/title/tt4154756/
     */
    public function parse(string $html): void
    {
        if (preg_match(&quot;|&lt;h1 itemprop=\&quot;name\&quot; class=\&quot;\&quot;&gt;(.*?)&lt;/h1&gt;|&quot;, $html, $matches)) {
            $title = $matches[1];
        }
        echo &quot;IMDBMovieScrapingCommand: Parsed movie $title.\n&quot;;
    }
}

/**
 * Класс Очередь действует как Отправитель. Он складывает объекты команд в стек
 * и выполняет их поочерёдно. Если выполнение скрипта внезапно завершится,
 * очередь и все её команды можно будет легко восстановить, и вам не придётся
 * повторять все выполненные команды.
 *
 * Обратите внимание, что это очень примитивная реализация очереди команд,
 * которая хранит команды в локальной базе данных SQLite. Существуют десятки
 * надёжных реализаций очереди, доступных для использования в реальных
 * приложениях.
 */
class Queue
{
    private $db;

    public function __construct()
    {
        $this-&gt;db = new \SQLite3(
            __DIR__ . &#39;/commands.sqlite&#39;,
            SQLITE3_OPEN_CREATE | SQLITE3_OPEN_READWRITE
        );

        $this-&gt;db-&gt;query(&#39;CREATE TABLE IF NOT EXISTS &quot;commands&quot; (
            &quot;id&quot; INTEGER PRIMARY KEY AUTO_INCREMENT NOT NULL,
            &quot;command&quot; TEXT,
            &quot;status&quot; INTEGER
        )&#39;);
    }

    public function isEmpty(): bool
    {
        $query = &#39;SELECT COUNT(&quot;id&quot;) FROM &quot;commands&quot; WHERE status = 0&#39;;

        return $this-&gt;db-&gt;querySingle($query) === 0;
    }

    public function add(Command $command): void
    {
        $query = &#39;INSERT INTO commands (command, status) VALUES (:command, :status)&#39;;
        $statement = $this-&gt;db-&gt;prepare($query);
        $statement-&gt;bindValue(&#39;:command&#39;, base64_encode(serialize($command)));
        $statement-&gt;bindValue(&#39;:status&#39;, $command-&gt;getStatus());
        $statement-&gt;execute();
    }

    public function getCommand(): Command
    {
        $query = &#39;SELECT * FROM &quot;commands&quot; WHERE &quot;status&quot; = 0 LIMIT 1&#39;;
        $record = $this-&gt;db-&gt;querySingle($query, true);
        $command = unserialize(base64_decode($record[&quot;command&quot;]));
        $command-&gt;id = $record[&#39;id&#39;];

        return $command;
    }

    public function completeCommand(Command $command): void
    {
        $query = &#39;UPDATE commands SET status = :status WHERE id = :id&#39;;
        $statement = $this-&gt;db-&gt;prepare($query);
        $statement-&gt;bindValue(&#39;:status&#39;, $command-&gt;getStatus());
        $statement-&gt;bindValue(&#39;:id&#39;, $command-&gt;getId());
        $statement-&gt;execute();
    }

    public function work(): void
    {
        while (!$this-&gt;isEmpty()) {
            $command = $this-&gt;getCommand();
            $command-&gt;execute();
        }
    }

    /**
     * Для удобства объект Очереди является Одиночкой.
     */
    public static function get(): Queue
    {
        static $instance;
        if (!$instance) {
            $instance = new Queue();
        }

        return $instance;
    }
}

/**
 * Клиентский код.
 */

$queue = Queue::get();

if ($queue-&gt;isEmpty()) {
    $queue-&gt;add(new IMDBGenresScrapingCommand());
}

$queue-&gt;work();</code></pre>
</figure>

#### []{#example-1--Output-txt .anchor} **Output.txt:** Результат выполнения

<figure class="code">
<pre class="code" lang="output"><code>WebScrapingCommand: Downloaded https://www.imdb.com/feature/genre/
IMDBGenresScrapingCommand: Discovered 14 genres.
WebScrapingCommand: Downloaded https://www.imdb.com/search/title?genres=comedy
IMDBGenrePageScrapingCommand: Discovered 50 movies.
WebScrapingCommand: Downloaded https://www.imdb.com/search/title?genres=sci-fi
IMDBGenrePageScrapingCommand: Discovered 50 movies.
WebScrapingCommand: Downloaded https://www.imdb.com/search/title?genres=horror
IMDBGenrePageScrapingCommand: Discovered 50 movies.
WebScrapingCommand: Downloaded https://www.imdb.com/search/title?genres=romance
IMDBGenrePageScrapingCommand: Discovered 50 movies.
...</code></pre>
</figure>
:::
:::::

::: {#nav-tab .nav .nav-tabs .nav-tabs-bottom .nav-justified role="tablist"}
[Концептуальный пример](#example-0){#example-0-tab-bottom .nav-item
.nav-link .active toggle="tab" role="tab" aria-controls="example-0"
aria-selected="true"} [Пример из реальной
жизни](#example-1){#example-1-tab-bottom .nav-item .nav-link
toggle="tab" role="tab" aria-controls="example-1" aria-selected="true"}
:::

::: next
#### Читаем дальше

[Итератор на PHP []{.fa
.fa-arrow-right}](../../iterator/php/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Цепочка обязанностей на
PHP](../../chain-of-responsibility/php/example.html){.btn .btn-default
rel="prev"}
:::

## **Команда** на других языках программирования

[![Команда на
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Команда на C#"){.prog-lang-link}
[![Команда на
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Команда на C++"){.prog-lang-link}
[![Команда на
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Команда на Go"){.prog-lang-link}
[![Команда на
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Команда на Java"){.prog-lang-link}
[![Команда на
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Команда на Python"){.prog-lang-link}
[![Команда на
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Команда на Ruby"){.prog-lang-link}
[![Команда на
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Команда на Rust"){.prog-lang-link}
[![Команда на
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Команда на Swift"){.prog-lang-link}
[![Команда на
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Команда на TypeScript"){.prog-lang-link}
:::::::::::::::::::::::::

::::::: {.prom .banner-sidebar .banner .banner-striped .banner-examples .banner-removable .banner-removable-patterns data-id="DP: Examples-IDE" creative-id="examples-sidebar-ru" position="sidebar"}
:::::: {.banner-inner style="font-size: 14px; line-height: 21px;"}
::: banner-examples-img
[![](../../../../images/patterns/banners/examples-ide91ca.png?id=3115b4b548fb96b75974e2de8f4f49bc){width="215"
height="158" loading="lazy"
srcset="/images/patterns/banners/examples-ide-2x.png?id=93c007a6d157b97c28eb213f59d716ae 2x"}](../../book.html)
:::

::: banner-examples-fade
[ Архив с примерами](../../book.html){.btn .btn-secondary .btn-block}
:::

::: {.banner-examples-text style="margin-top: 1rem;"}
Купи книгу **Погружение в Паттерны** и получи архив с десятками
детальных примеров, которые можно открывать прямо в IDE.

[ Узнать больше...](../../book.html){.btn .btn-secondary .btn-block}
:::
::::::
:::::::
:::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::
