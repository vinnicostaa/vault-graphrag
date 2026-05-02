:::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Padrões de
Projeto](../../../design-patterns.html){.type} /
[Singleton](../../singleton.html){.type} / [PHP](../../php.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Singleton](../../../../images/patterns/cards/singleton-minif559.png?id=914e1565dfdf15f240e766163bd303ec){srcset="/images/patterns/cards/singleton-mini-2x.png?id=4a7348083d7719d02709a5c9ff878b9f 2x"}
:::

# **Singleton** em PHP {#singleton-em-php .pattern-example-title-block-title}
::::

:::::::::::::: pattern-example-body
::: pattern-example-brief
O **Singleton** é um padrão de projeto criacional, que garante que
apenas um objeto desse tipo exista e forneça um único ponto de acesso a
ele para qualquer outro código.

O Singleton tem quase os mesmos prós e contras que as variáveis globais.
Embora sejam super úteis, eles quebram a modularidade do seu código.

Você pode usar classes que dependem de singletons em algumas outras
situações. Você terá que levar a classe singleton também. Na maioria das
vezes, essa limitação surge durante a criação de testes de unidade.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Saiba mais sobre o Singleton ](../../singleton.html){.btn .btn-lg
.btn-primary}
:::

::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
Navegação
:::

::: en-intro
 [Introdução](#)
:::

::: en-intro
 [Exemplo conceitual](#example-0)
:::

::: en-file
 [index](#example-0--index-php)
:::

::: en-file
 [Output](#example-0--Output-txt)
:::

::: en-intro
 [Exemplo do mundo real](#example-1)
:::

::: en-file
 [index](#example-1--index-php)
:::

::: en-file
 [Output](#example-1--Output-txt)
:::
:::::::::::

**Complexidade:** []{.fa-stars}

**Popularidade:** []{.fa-stars}

**Exemplos de uso:** Muitos desenvolvedores consideram o padrão
Singleton um antipadrão. É por isso que seu uso está diminuindo no
código PHP.

**Identificação:** O Singleton pode ser reconhecido por um método de
criação estático, que retorna o mesmo objeto em cache.
::::::::::::::

::: {#nav-tab .nav .nav-tabs .nav-justified role="tablist"}
[Exemplo conceitual](#example-0){#example-0-tab .nav-item .nav-link
.active toggle="tab" role="tab" aria-controls="example-0"
aria-selected="true"} [Exemplo do mundo real](#example-1){#example-1-tab
.nav-item .nav-link toggle="tab" role="tab" aria-controls="example-1"
aria-selected="true"}
:::

::::: {#nav-tabContent .pattern-example-tab-content .tab-content}
::: {#example-0 .tab-pane .show .active role="tabpanel" aria-labelledby="example-0-tab" style="padding-top: 24px"}
## Exemplo conceitual {#example-0-title}

Este exemplo ilustra a estrutura do padrão de projeto **Singleton**. Ele
se concentra em responder a estas perguntas:

-   De quais classes ele consiste?
-   Quais papéis essas classes desempenham?
-   De que maneira os elementos do padrão estão relacionados?

Depois de aprender sobre a estrutura do padrão, será mais fácil entender
o exemplo a seguir, com base em um caso de uso PHP do mundo real.

#### []{#example-0--index-php .anchor} **index.php:** Exemplo conceitual

<figure class="code">
<pre class="code" lang="php"><code>&lt;?php

namespace RefactoringGuru\Singleton\Conceptual;

/**
 * The Singleton class defines the `GetInstance` method that serves as an
 * alternative to constructor and lets clients access the same instance of this
 * class over and over.
 */
class Singleton
{
    /**
     * The Singleton&#39;s instance is stored in a static field. This field is an
     * array, because we&#39;ll allow our Singleton to have subclasses. Each item in
     * this array will be an instance of a specific Singleton&#39;s subclass. You&#39;ll
     * see how this works in a moment.
     */
    private static $instances = [];

    /**
     * The Singleton&#39;s constructor should always be private to prevent direct
     * construction calls with the `new` operator.
     */
    protected function __construct()
    {
    }

    /**
     * Singletons should not be cloneable.
     */
    protected function __clone()
    {
    }

    /**
     * Singletons should not be restorable from strings.
     */
    public function __wakeup()
    {
        throw new \Exception(&quot;Cannot unserialize a singleton.&quot;);
    }

    /**
     * This is the static method that controls the access to the singleton
     * instance. On the first run, it creates a singleton object and places it
     * into the static field. On subsequent runs, it returns the client existing
     * object stored in the static field.
     *
     * This implementation lets you subclass the Singleton class while keeping
     * just one instance of each subclass around.
     */
    public static function getInstance(): Singleton
    {
        $cls = static::class;
        if (!isset(self::$instances[$cls])) {
            self::$instances[$cls] = new static();
        }

        return self::$instances[$cls];
    }

    /**
     * Finally, any singleton should define some business logic, which can be
     * executed on its instance.
     */
    public function someBusinessLogic()
    {
        // ...
    }
}

/**
 * The client code.
 */
function clientCode()
{
    $s1 = Singleton::getInstance();
    $s2 = Singleton::getInstance();
    if ($s1 === $s2) {
        echo &quot;Singleton works, both variables contain the same instance.&quot;;
    } else {
        echo &quot;Singleton failed, variables contain different instances.&quot;;
    }
}

clientCode();</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** Resultados da execução

<figure class="code">
<pre class="code" lang="output"><code>Singleton works, both variables contain the same instance.</code></pre>
</figure>
:::

::: {#example-1 .tab-pane role="tabpanel" aria-labelledby="example-1-tab" style="padding-top: 24px"}
## Exemplo do mundo real {#example-1-title}

O padrão **Singleton** é notório por limitar a reutilização de código e
complicar testes unitários. No entanto, ainda é muito útil em alguns
casos. Em particular, é útil quando você precisa controlar alguns
recursos compartilhados. Por exemplo, um objeto de log global que
precisa controlar o acesso a um arquivo de log. Outro bom exemplo: um
armazenamento de configuração de tempo de execução compartilhado.

#### []{#example-1--index-php .anchor} **index.php:** Exemplo do mundo real

<figure class="code">
<pre class="code" lang="php"><code>&lt;?php

namespace RefactoringGuru\Singleton\RealWorld;

/**
 * If you need to support several types of Singletons in your app, you can
 * define the basic features of the Singleton in a base class, while moving the
 * actual business logic (like logging) to subclasses.
 */
class Singleton
{
    /**
     * The actual singleton&#39;s instance almost always resides inside a static
     * field. In this case, the static field is an array, where each subclass of
     * the Singleton stores its own instance.
     */
    private static $instances = [];

    /**
     * Singleton&#39;s constructor should not be public. However, it can&#39;t be
     * private either if we want to allow subclassing.
     */
    protected function __construct()
    {
    }

    /**
     * Cloning and unserialization are not permitted for singletons.
     */
    protected function __clone()
    {
    }

    public function __wakeup()
    {
        throw new \Exception(&quot;Cannot unserialize singleton&quot;);
    }

    /**
     * The method you use to get the Singleton&#39;s instance.
     */
    public static function getInstance()
    {
        $subclass = static::class;
        if (!isset(self::$instances[$subclass])) {
            // Note that here we use the &quot;static&quot; keyword instead of the actual
            // class name. In this context, the &quot;static&quot; keyword means &quot;the name
            // of the current class&quot;. That detail is important because when the
            // method is called on the subclass, we want an instance of that
            // subclass to be created here.

            self::$instances[$subclass] = new static();
        }
        return self::$instances[$subclass];
    }
}

/**
 * The logging class is the most known and praised use of the Singleton pattern.
 * In most cases, you need a single logging object that writes to a single log
 * file (control over shared resource). You also need a convenient way to access
 * that instance from any context of your app (global access point).
 */
class Logger extends Singleton
{
    /**
     * A file pointer resource of the log file.
     */
    private $fileHandle;

    /**
     * Since the Singleton&#39;s constructor is called only once, just a single file
     * resource is opened at all times.
     *
     * Note, for the sake of simplicity, we open the console stream instead of
     * the actual file here.
     */
    protected function __construct()
    {
        $this-&gt;fileHandle = fopen(&#39;php://stdout&#39;, &#39;w&#39;);
    }

    /**
     * Write a log entry to the opened file resource.
     */
    public function writeLog(string $message): void
    {
        $date = date(&#39;Y-m-d&#39;);
        fwrite($this-&gt;fileHandle, &quot;$date: $message\n&quot;);
    }

    /**
     * Just a handy shortcut to reduce the amount of code needed to log messages
     * from the client code.
     */
    public static function log(string $message): void
    {
        $logger = static::getInstance();
        $logger-&gt;writeLog($message);
    }
}

/**
 * Applying the Singleton pattern to the configuration storage is also a common
 * practice. Often you need to access application configurations from a lot of
 * different places of the program. Singleton gives you that comfort.
 */
class Config extends Singleton
{
    private $hashmap = [];

    public function getValue(string $key): string
    {
        return $this-&gt;hashmap[$key];
    }

    public function setValue(string $key, string $value): void
    {
        $this-&gt;hashmap[$key] = $value;
    }
}

/**
 * The client code.
 */
Logger::log(&quot;Started!&quot;);

// Compare values of Logger singleton.
$l1 = Logger::getInstance();
$l2 = Logger::getInstance();
if ($l1 === $l2) {
    Logger::log(&quot;Logger has a single instance.&quot;);
} else {
    Logger::log(&quot;Loggers are different.&quot;);
}

// Check how Config singleton saves data...
$config1 = Config::getInstance();
$login = &quot;test_login&quot;;
$password = &quot;test_password&quot;;
$config1-&gt;setValue(&quot;login&quot;, $login);
$config1-&gt;setValue(&quot;password&quot;, $password);
// ...and restores it.
$config2 = Config::getInstance();
if (
    $login == $config2-&gt;getValue(&quot;login&quot;) &amp;&amp;
    $password == $config2-&gt;getValue(&quot;password&quot;)
) {
    Logger::log(&quot;Config singleton also works fine.&quot;);
}

Logger::log(&quot;Finished!&quot;);</code></pre>
</figure>

#### []{#example-1--Output-txt .anchor} **Output.txt:** Resultados da execução

<figure class="code">
<pre class="code" lang="output"><code>2018-06-04: Started!
2018-06-04: Logger has a single instance.
2018-06-04: Config singleton also works fine.
2018-06-04: Finished!</code></pre>
</figure>
:::
:::::

::: {#nav-tab .nav .nav-tabs .nav-tabs-bottom .nav-justified role="tablist"}
[Exemplo conceitual](#example-0){#example-0-tab-bottom .nav-item
.nav-link .active toggle="tab" role="tab" aria-controls="example-0"
aria-selected="true"} [Exemplo do mundo
real](#example-1){#example-1-tab-bottom .nav-item .nav-link toggle="tab"
role="tab" aria-controls="example-1" aria-selected="true"}
:::

::: next
#### Leia a seguir

[Adapter em PHP []{.fa
.fa-arrow-right}](../../adapter/php/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Voltar

[[]{.fa .fa-arrow-left} Prototype em
PHP](../../prototype/php/example.html){.btn .btn-default rel="prev"}
:::

## **Singleton** em outras linguagens

[![Singleton em
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Singleton em C#"){.prog-lang-link}
[![Singleton em
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Singleton em C++"){.prog-lang-link}
[![Singleton em
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Singleton em Go"){.prog-lang-link}
[![Singleton em
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Singleton em Java"){.prog-lang-link}
[![Singleton em
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Singleton em Python"){.prog-lang-link}
[![Singleton em
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Singleton em Ruby"){.prog-lang-link}
[![Singleton em
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Singleton em Rust"){.prog-lang-link}
[![Singleton em
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Singleton em Swift"){.prog-lang-link}
[![Singleton em
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Singleton em TypeScript"){.prog-lang-link}
:::::::::::::::::::::::::

::::::: {.prom .banner-sidebar .banner .banner-striped .banner-examples .banner-removable .banner-removable-patterns data-id="DP: Examples-IDE" creative-id="examples-sidebar-pt-br" position="sidebar"}
:::::: {.banner-inner style="font-size: 14px; line-height: 21px;"}
::: banner-examples-img
[![](../../../../images/patterns/banners/examples-ide91ca.png?id=3115b4b548fb96b75974e2de8f4f49bc){width="215"
height="158" loading="lazy"
srcset="/images/patterns/banners/examples-ide-2x.png?id=93c007a6d157b97c28eb213f59d716ae 2x"}](../../book.html)
:::

::: banner-examples-fade
[ Arquivo com exemplos de código](../../book.html){.btn .btn-secondary
.btn-block}
:::

::: {.banner-examples-text style="margin-top: 1rem;"}
Compre o eBook **Mergulho nos Padrões de Projeto** e tenha acesso ao
arquivo com dúzias de exemplos detalhados que podem ser abertos
diretamente em seu IDE.

[ Saiba mais...](../../book.html){.btn .btn-secondary .btn-block}
:::
::::::
:::::::
:::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::
