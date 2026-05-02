:::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../index.html){.home} / [Design
Patterns](../../../design-patterns.html){.type} /
[Command](../../command.html){.type} / [PHP](../../php.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Command](../../../images/patterns/cards/command-mini8482.png?id=b149eda017c0583c1e92343b83cfb1eb){srcset="/images/patterns/cards/command-mini-2x.png?id=e5f6332e057f6d352a209da963a8fc54 2x"}
:::

# **Command** in PHP {#command-in-php .pattern-example-title-block-title}
::::

:::::::::::::: pattern-example-body
::: pattern-example-brief
**Command** is behavioral design pattern that converts requests or
simple operations into objects.

The conversion allows deferred or remote execution of commands, storing
command history, etc.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Learn more about Command ](../../command.html){.btn .btn-lg
.btn-primary}
:::

::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
Navigation
:::

::: en-intro
 [Intro](#)
:::

::: en-intro
 [Conceptual Example](#example-0)
:::

::: en-file
 [index](#example-0--index-php)
:::

::: en-file
 [Output](#example-0--Output-txt)
:::

::: en-intro
 [Real World Example](#example-1)
:::

::: en-file
 [index](#example-1--index-php)
:::

::: en-file
 [Output](#example-1--Output-txt)
:::
:::::::::::

**Complexity:** []{.fa-stars}

**Popularity:** []{.fa-stars}

**Usage examples:** The Command pattern is pretty common in PHP code.
It's used for queueing tasks, tracking the history of executed tasks and
performing the "undo".

**Identification:** The Command pattern is recognizable by behavioral
methods in an abstract/interface type (sender) which invokes a method in
an implementation of a different abstract/interface type (receiver)
which has been encapsulated by the command implementation during its
creation. Command classes are usually limited to specific actions.
::::::::::::::

::: {#nav-tab .nav .nav-tabs .nav-justified role="tablist"}
[Conceptual Example](#example-0){#example-0-tab .nav-item .nav-link
.active toggle="tab" role="tab" aria-controls="example-0"
aria-selected="true"} [Real World Example](#example-1){#example-1-tab
.nav-item .nav-link toggle="tab" role="tab" aria-controls="example-1"
aria-selected="true"}
:::

::::: {#nav-tabContent .pattern-example-tab-content .tab-content}
::: {#example-0 .tab-pane .show .active role="tabpanel" aria-labelledby="example-0-tab" style="padding-top: 24px"}
## Conceptual Example {#example-0-title}

This example illustrates the structure of the **Command** design pattern
and focuses on the following questions:

-   What classes does it consist of?
-   What roles do these classes play?
-   In what way the elements of the pattern are related?

After learning about the pattern's structure it'll be easier for you to
grasp the following example, based on a real-world PHP use case.

#### []{#example-0--index-php .anchor} **index.php:** Conceptual example

<figure class="code">
<pre class="code" lang="php"><code>&lt;?php

namespace RefactoringGuru\Command\Conceptual;

/**
 * The Command interface declares a method for executing a command.
 */
interface Command
{
    public function execute(): void;
}

/**
 * Some commands can implement simple operations on their own.
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
 * However, some commands can delegate more complex operations to other objects,
 * called &quot;receivers.&quot;
 */
class ComplexCommand implements Command
{
    /**
     * @var Receiver
     */
    private $receiver;

    /**
     * Context data, required for launching the receiver&#39;s methods.
     */
    private $a;

    private $b;

    /**
     * Complex commands can accept one or several receiver objects along with
     * any context data via the constructor.
     */
    public function __construct(Receiver $receiver, string $a, string $b)
    {
        $this-&gt;receiver = $receiver;
        $this-&gt;a = $a;
        $this-&gt;b = $b;
    }

    /**
     * Commands can delegate to any methods of a receiver.
     */
    public function execute(): void
    {
        echo &quot;ComplexCommand: Complex stuff should be done by a receiver object.\n&quot;;
        $this-&gt;receiver-&gt;doSomething($this-&gt;a);
        $this-&gt;receiver-&gt;doSomethingElse($this-&gt;b);
    }
}

/**
 * The Receiver classes contain some important business logic. They know how to
 * perform all kinds of operations, associated with carrying out a request. In
 * fact, any class may serve as a Receiver.
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
 * The Invoker is associated with one or several commands. It sends a request to
 * the command.
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
     * Initialize commands.
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
     * The Invoker does not depend on concrete command or receiver classes. The
     * Invoker passes a request to a receiver indirectly, by executing a
     * command.
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
 * The client code can parameterize an invoker with any commands.
 */
$invoker = new Invoker();
$invoker-&gt;setOnStart(new SimpleCommand(&quot;Say Hi!&quot;));
$receiver = new Receiver();
$invoker-&gt;setOnFinish(new ComplexCommand($receiver, &quot;Send email&quot;, &quot;Save report&quot;));

$invoker-&gt;doSomethingImportant();</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** Execution result

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
## Real World Example {#example-1-title}

In this example, the **Command** pattern is used to queue web scraping
calls to the IMDB website and execute them one by one. The queue itself
is kept in a database that helps to preserve commands between script
launches.

#### []{#example-1--index-php .anchor} **index.php:** Real world example

<figure class="code">
<pre class="code" lang="php"><code>&lt;?php

namespace RefactoringGuru\Command\RealWorld;

/**
 * The Command interface declares the main execution method as well as several
 * helper methods for retrieving a command&#39;s metadata.
 */
interface Command
{
    public function execute(): void;

    public function getId(): int;

    public function getStatus(): int;
}

/**
 * The base web scraping Command defines the basic downloading infrastructure,
 * common to all concrete web scraping commands.
 */
abstract class WebScrapingCommand implements Command
{
    public $id;

    public $status = 0;

    /**
     * @var string URL for scraping.
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
     * Since the execution methods for all web scraping commands are very
     * similar, we can provide a default implementation and let subclasses
     * override them if needed.
     *
     * Psst! An observant reader may spot another behavioral pattern in action
     * here.
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
 * The Concrete Command for scraping the list of movie genres.
 */
class IMDBGenresScrapingCommand extends WebScrapingCommand
{
    public function __construct()
    {
        $this-&gt;url = &quot;https://www.imdb.com/feature/genre/&quot;;
    }

    /**
     * Extract all genres and their search URLs from the page:
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
 * The Concrete Command for scraping the list of movies in a specific genre.
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
     * Extract all movies from a page like this:
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

        // Parse the next page URL.
        if (preg_match(&quot;|Next &amp;#187;&lt;/a&gt;|&quot;, $html)) {
            Queue::get()-&gt;add(new IMDBGenrePageScrapingCommand($this-&gt;url, $this-&gt;page + 1));
        }
    }
}

/**
 * The Concrete Command for scraping the movie details.
 */
class IMDBMovieScrapingCommand extends WebScrapingCommand
{
    /**
     * Get the movie info from a page like this:
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
 * The Queue class acts as an Invoker. It stacks the command objects and
 * executes them one by one. If the script execution is suddenly terminated, the
 * queue and all its commands can easily be restored, and you won&#39;t need to
 * repeat all of the executed commands.
 *
 * Note that this is a very primitive implementation of the command queue, which
 * stores commands in a local SQLite database. There are dozens of robust queue
 * solution available for use in real apps.
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
     * For our convenience, the Queue object is a Singleton.
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
 * The client code.
 */

$queue = Queue::get();

if ($queue-&gt;isEmpty()) {
    $queue-&gt;add(new IMDBGenresScrapingCommand());
}

$queue-&gt;work();</code></pre>
</figure>

#### []{#example-1--Output-txt .anchor} **Output.txt:** Execution result

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
[Conceptual Example](#example-0){#example-0-tab-bottom .nav-item
.nav-link .active toggle="tab" role="tab" aria-controls="example-0"
aria-selected="true"} [Real World
Example](#example-1){#example-1-tab-bottom .nav-item .nav-link
toggle="tab" role="tab" aria-controls="example-1" aria-selected="true"}
:::

::: next
#### Read next

[Iterator in PHP []{.fa
.fa-arrow-right}](../../iterator/php/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Return

[[]{.fa .fa-arrow-left} Chain of Responsibility in
PHP](../../chain-of-responsibility/php/example.html){.btn .btn-default
rel="prev"}
:::

## **Command** in Other Languages

[![Command in
C#](../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Command in C#"){.prog-lang-link}
[![Command in
C++](../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Command in C++"){.prog-lang-link}
[![Command in
Go](../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Command in Go"){.prog-lang-link}
[![Command in
Java](../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Command in Java"){.prog-lang-link}
[![Command in
Python](../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Command in Python"){.prog-lang-link}
[![Command in
Ruby](../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Command in Ruby"){.prog-lang-link}
[![Command in
Rust](../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Command in Rust"){.prog-lang-link}
[![Command in
Swift](../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Command in Swift"){.prog-lang-link}
[![Command in
TypeScript](../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Command in TypeScript"){.prog-lang-link}
:::::::::::::::::::::::::

::::::: {.prom .banner-sidebar .banner .banner-striped .banner-examples .banner-removable .banner-removable-patterns data-id="DP: Examples-IDE" creative-id="examples-sidebar-en" position="sidebar"}
:::::: {.banner-inner style="font-size: 14px; line-height: 21px;"}
::: banner-examples-img
[![](../../../images/patterns/banners/examples-ide91ca.png?id=3115b4b548fb96b75974e2de8f4f49bc){width="215"
height="158" loading="lazy"
srcset="/images/patterns/banners/examples-ide-2x.png?id=93c007a6d157b97c28eb213f59d716ae 2x"}](../../book.html)
:::

::: banner-examples-fade
[ Archive with examples](../../book.html){.btn .btn-secondary
.btn-block}
:::

::: {.banner-examples-text style="margin-top: 1rem;"}
Buy the eBook **Dive Into Design Patterns** and get the access to
archive with dozens of detailed examples that can be opened right in
your IDE.

[ Learn more...](../../book.html){.btn .btn-secondary .btn-block}
:::
::::::
:::::::
:::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::
