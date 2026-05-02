:::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Паттерны
проектирования](../../../design-patterns.html){.type} /
[Декоратор](../../decorator.html){.type} / [PHP](../../php.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Декоратор](../../../../images/patterns/cards/decorator-mini6adb.png?id=d30458908e315af195cb183bc52dbef9){srcset="/images/patterns/cards/decorator-mini-2x.png?id=3b58e540d7d28523080cad341ed9b2e9 2x"}
:::

# **Декоратор** на PHP {#декоратор-на-php .pattern-example-title-block-title}
::::

:::::::::::::: pattern-example-body
::: pattern-example-brief
**Декоратор** --- это структурный паттерн, который позволяет добавлять
объектам новые поведения на лету, помещая их в объекты-обёртки.

Декоратор позволяет оборачивать объекты бесчисленное количество раз
благодаря тому, что и обёртки, и реальные оборачиваемые объекты имеют
общий интерфейс.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Подробней о паттерне Декоратор ](../../decorator.html){.btn .btn-lg
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

**Применимость:** Паттерн можно часто встретить в PHP-коде, особенно в
коде, работающем с потоками данных.

**Признаки применения паттерна:** Декоратор можно распознать по
создающим методам, которые принимают в параметрах объекты того же
абстрактного типа или интерфейса, что и текущий класс.
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

Этот пример показывает структуру паттерна **Декоратор**, а именно --- из
каких классов он состоит, какие роли эти классы выполняют и как они
взаимодействуют друг с другом.

После ознакомления со структурой, вам будет легче воспринимать второй
пример, который рассматривает реальный случай использования паттерна в
мире PHP.

#### []{#example-0--index-php .anchor} **index.php:** Пример структуры паттерна

<figure class="code">
<pre class="code" lang="php"><code>&lt;?php

namespace RefactoringGuru\Decorator\Conceptual;

/**
 * Базовый интерфейс Компонента определяет поведение, которое изменяется
 * декораторами.
 */
interface Component
{
    public function operation(): string;
}

/**
 * Конкретные Компоненты предоставляют реализации поведения по умолчанию. Может
 * быть несколько вариаций этих классов.
 */
class ConcreteComponent implements Component
{
    public function operation(): string
    {
        return &quot;ConcreteComponent&quot;;
    }
}

/**
 * Базовый класс Декоратора следует тому же интерфейсу, что и другие компоненты.
 * Основная цель этого класса - определить интерфейс обёртки для всех конкретных
 * декораторов. Реализация кода обёртки по умолчанию может включать в себя поле
 * для хранения завёрнутого компонента и средства его инициализации.
 */
class Decorator implements Component
{
    /**
     * @var Component
     */
    protected $component;

    public function __construct(Component $component)
    {
        $this-&gt;component = $component;
    }

    /**
     * Декоратор делегирует всю работу обёрнутому компоненту.
     */
    public function operation(): string
    {
        return $this-&gt;component-&gt;operation();
    }
}

/**
 * Конкретные Декораторы вызывают обёрнутый объект и изменяют его результат
 * некоторым образом.
 */
class ConcreteDecoratorA extends Decorator
{
    /**
     * Декораторы могут вызывать родительскую реализацию операции, вместо того,
     * чтобы вызвать обёрнутый объект напрямую. Такой подход упрощает расширение
     * классов декораторов.
     */
    public function operation(): string
    {
        return &quot;ConcreteDecoratorA(&quot; . parent::operation() . &quot;)&quot;;
    }
}

/**
 * Декораторы могут выполнять своё поведение до или после вызова обёрнутого
 * объекта.
 */
class ConcreteDecoratorB extends Decorator
{
    public function operation(): string
    {
        return &quot;ConcreteDecoratorB(&quot; . parent::operation() . &quot;)&quot;;
    }
}

/**
 * Клиентский код работает со всеми объектами, используя интерфейс Компонента.
 * Таким образом, он остаётся независимым от конкретных классов компонентов, с
 * которыми работает.
 */
function clientCode(Component $component)
{
    // ...

    echo &quot;RESULT: &quot; . $component-&gt;operation();

    // ...
}

/**
 * Таким образом, клиентский код может поддерживать как простые компоненты...
 */
$simple = new ConcreteComponent();
echo &quot;Client: I&#39;ve got a simple component:\n&quot;;
clientCode($simple);
echo &quot;\n\n&quot;;

/**
 * ...так и декорированные.
 *
 * Обратите внимание, что декораторы могут обёртывать не только простые
 * компоненты, но и другие декораторы.
 */
$decorator1 = new ConcreteDecoratorA($simple);
$decorator2 = new ConcreteDecoratorB($decorator1);
echo &quot;Client: Now I&#39;ve got a decorated component:\n&quot;;
clientCode($decorator2);</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** Результат выполнения

<figure class="code">
<pre class="code" lang="output"><code>Client: I&#39;ve got a simple component:
RESULT: ConcreteComponent

Client: Now I&#39;ve got a decorated component:
RESULT: ConcreteDecoratorB(ConcreteDecoratorA(ConcreteComponent))</code></pre>
</figure>
:::

::: {#example-1 .tab-pane role="tabpanel" aria-labelledby="example-1-tab" style="padding-top: 24px"}
## Пример из реальной жизни {#example-1-title}

В этом примере паттерн **Декоратора** помогает создать сложные правила
фильтрации текста для приведения информации в порядок перед её
отображением на веб-странице. Разные типы информации, такие как
комментарии, сообщения на форуме или личные сообщения, требуют разных
наборов фильтров.

Например, вы хотели бы удалить весь HTML из комментариев и в тоже время
сохранить некоторые основные теги HTML в сообщениях на форуме. Кроме
того, вы можете пожелать разрешить публикацию в формате Markdown,
который должен быть обработан перед какой-либо фильтрацией HTML. Все эти
правила фильтрации могут быть представлены в виде отдельных классов
декораторов, которые могут быть сложены в стек по-разному, в зависимости
от характера содержимого.

#### []{#example-1--index-php .anchor} **index.php:** Пример из реальной жизни

<figure class="code">
<pre class="code" lang="php"><code>&lt;?php

namespace RefactoringGuru\Decorator\RealWorld;

/**
 * Интерфейс Компонента объявляет метод фильтрации, который должен быть
 * реализован всеми конкретными компонентами и декораторами.
 */
interface InputFormat
{
    public function formatText(string $text): string;
}

/**
 * Конкретный Компонент является основным элементом декорирования. Он содержит
 * исходный текст как есть, без какой-либо фильтрации или форматирования.
 */
class TextInput implements InputFormat
{
    public function formatText(string $text): string
    {
        return $text;
    }
}

/**
 * Базовый класс Декоратора не содержит реальной логики фильтрации или
 * форматирования. Его основная цель – реализовать базовую инфраструктуру
 * декорирования: поле для хранения обёрнутого компонента или другого декоратора
 * и базовый метод форматирования, который делегирует работу обёрнутому объекту.
 * Реальная работа по форматированию выполняется подклассами.
 */
class TextFormat implements InputFormat
{
    /**
     * @var InputFormat
     */
    protected $inputFormat;

    public function __construct(InputFormat $inputFormat)
    {
        $this-&gt;inputFormat = $inputFormat;
    }

    /**
     * Декоратор делегирует всю работу обёрнутому компоненту.
     */
    public function formatText(string $text): string
    {
        return $this-&gt;inputFormat-&gt;formatText($text);
    }
}

/**
 * Этот Конкретный Декоратор удаляет все теги HTML из данного текста.
 */
class PlainTextFilter extends TextFormat
{
    public function formatText(string $text): string
    {
        $text = parent::formatText($text);
        return strip_tags($text);
    }
}

/**
 * Этот Конкретный Декоратор удаляет только опасные теги и атрибуты HTML,
 * которые могут привести к XSS-уязвимости.
 */
class DangerousHTMLTagsFilter extends TextFormat
{
    private $dangerousTagPatterns = [
        &quot;|&lt;script.*?&gt;([\s\S]*)?&lt;/script&gt;|i&quot;, // ...
    ];

    private $dangerousAttributes = [
        &quot;onclick&quot;, &quot;onkeypress&quot;, // ...
    ];


    public function formatText(string $text): string
    {
        $text = parent::formatText($text);

        foreach ($this-&gt;dangerousTagPatterns as $pattern) {
            $text = preg_replace($pattern, &#39;&#39;, $text);
        }

        foreach ($this-&gt;dangerousAttributes as $attribute) {
            $text = preg_replace_callback(&#39;|&lt;(.*?)&gt;|&#39;, function ($matches) use ($attribute) {
                $result = preg_replace(&quot;|$attribute=|i&quot;, &#39;&#39;, $matches[1]);
                return &quot;&lt;&quot; . $result . &quot;&gt;&quot;;
            }, $text);
        }

        return $text;
    }
}

/**
 * Этот Конкретный Декоратор предоставляет элементарное преобразование Markdown
 * → HTML.
 */
class MarkdownFormat extends TextFormat
{
    public function formatText(string $text): string
    {
        $text = parent::formatText($text);

        // Форматирование элементов блока.
        $chunks = preg_split(&#39;|\n\n|&#39;, $text);
        foreach ($chunks as &amp;$chunk) {
            // Форматирование заголовков.
            if (preg_match(&#39;|^#+|&#39;, $chunk)) {
                $chunk = preg_replace_callback(&#39;|^(#+)(.*?)$|&#39;, function ($matches) {
                    $h = strlen($matches[1]);
                    return &quot;&lt;h$h&gt;&quot; . trim($matches[2]) . &quot;&lt;/h$h&gt;&quot;;
                }, $chunk);
            } // Форматирование параграфов.
            else {
                $chunk = &quot;&lt;p&gt;$chunk&lt;/p&gt;&quot;;
            }
        }
        $text = implode(&quot;\n\n&quot;, $chunks);

        // Форматирование встроенных элементов.
        $text = preg_replace(&quot;|__(.*?)__|&quot;, &#39;&lt;strong&gt;$1&lt;/strong&gt;&#39;, $text);
        $text = preg_replace(&quot;|\*\*(.*?)\*\*|&quot;, &#39;&lt;strong&gt;$1&lt;/strong&gt;&#39;, $text);
        $text = preg_replace(&quot;|_(.*?)_|&quot;, &#39;&lt;em&gt;$1&lt;/em&gt;&#39;, $text);
        $text = preg_replace(&quot;|\*(.*?)\*|&quot;, &#39;&lt;em&gt;$1&lt;/em&gt;&#39;, $text);

        return $text;
    }
}


/**
 * Клиентский код может быть частью реального веб-сайта, который отображает
 * создаваемый пользователями контент. Поскольку он работает с модулями
 * форматирования через интерфейс компонента, ему всё равно, получает ли он
 * простой объект компонента или обёрнутый.
 */
function displayCommentAsAWebsite(InputFormat $format, string $text)
{
    // ..

    echo $format-&gt;formatText($text);

    // ..
}

/**
 * Модули форматирования пользовательского ввода очень удобны при работе с
 * контентом, создаваемым пользователями. Отображение такого контента «как есть»
 * может быть очень опасным, особенно когда его могут создавать анонимные
 * пользователи (например, комментарии). Ваш сайт не только рискует получить
 * массу спам-ссылок, но также может быть подвергнут XSS-атакам.
 */
$dangerousComment = &lt;&lt;&lt;HERE
Hello! Nice blog post!
Please visit my &lt;a href=&#39;http://www.iwillhackyou.com&#39;&gt;homepage&lt;/a&gt;.
&lt;script src=&quot;http://www.iwillhackyou.com/script.js&quot;&gt;
  performXSSAttack();
&lt;/script&gt;
HERE;

/**
 * Наивное отображение комментариев (небезопасное).
 */
$naiveInput = new TextInput();
echo &quot;Website renders comments without filtering (unsafe):\n&quot;;
displayCommentAsAWebsite($naiveInput, $dangerousComment);
echo &quot;\n\n\n&quot;;

/**
 * Отфильтрованное отображение комментариев (безопасное).
 */
$filteredInput = new PlainTextFilter($naiveInput);
echo &quot;Website renders comments after stripping all tags (safe):\n&quot;;
displayCommentAsAWebsite($filteredInput, $dangerousComment);
echo &quot;\n\n\n&quot;;


/**
 * Декоратор позволяет складывать несколько входных форматов для получения
 * точного контроля над отображаемым содержимым.
 */
$dangerousForumPost = &lt;&lt;&lt;HERE
# Welcome

This is my first post on this **gorgeous** forum.

&lt;script src=&quot;http://www.iwillhackyou.com/script.js&quot;&gt;
  performXSSAttack();
&lt;/script&gt;
HERE;

/**
 * Наивное отображение сообщений (небезопасное, без форматирования).
 */
$naiveInput = new TextInput();
echo &quot;Website renders a forum post without filtering and formatting (unsafe, ugly):\n&quot;;
displayCommentAsAWebsite($naiveInput, $dangerousForumPost);
echo &quot;\n\n\n&quot;;

/**
 * Форматтер Markdown + фильтрация опасных тегов (безопасно, красиво).
 */
$text = new TextInput();
$markdown = new MarkdownFormat($text);
$filteredInput = new DangerousHTMLTagsFilter($markdown);
echo &quot;Website renders a forum post after translating markdown markup&quot; .
    &quot; and filtering some dangerous HTML tags and attributes (safe, pretty):\n&quot;;
displayCommentAsAWebsite($filteredInput, $dangerousForumPost);
echo &quot;\n\n\n&quot;;</code></pre>
</figure>

#### []{#example-1--Output-txt .anchor} **Output.txt:** Результат выполнения

<figure class="code">
<pre class="code" lang="output"><code>Website renders comments without filtering (unsafe):
Hello! Nice blog post!
Please visit my &lt;a href=&#39;http://www.iwillhackyou.com&#39;&gt;homepage&lt;/a&gt;.
&lt;script src=&quot;http://www.iwillhackyou.com/script.js&quot;&gt;
  performXSSAttack();
&lt;/script&gt;


Website renders comments after stripping all tags (safe):
Hello! Nice blog post!
Please visit my homepage.

  performXSSAttack();



Website renders a forum post without filtering and formatting (unsafe, ugly):
# Welcome

This is my first post on this **gorgeous** forum.

&lt;script src=&quot;http://www.iwillhackyou.com/script.js&quot;&gt;
  performXSSAttack();
&lt;/script&gt;


Website renders a forum post after translating markdown markupand filtering some dangerous HTML tags and attributes (safe, pretty):
&lt;h1&gt;Welcome&lt;/h1&gt;

&lt;p&gt;This is my first post on this &lt;strong&gt;gorgeous&lt;/strong&gt; forum.&lt;/p&gt;

&lt;p&gt;&lt;/p&gt;</code></pre>
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

[Фасад на PHP []{.fa
.fa-arrow-right}](../../facade/php/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Компоновщик на
PHP](../../composite/php/example.html){.btn .btn-default rel="prev"}
:::

## **Декоратор** на других языках программирования

[![Декоратор на
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Декоратор на C#"){.prog-lang-link}
[![Декоратор на
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Декоратор на C++"){.prog-lang-link}
[![Декоратор на
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Декоратор на Go"){.prog-lang-link}
[![Декоратор на
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Декоратор на Java"){.prog-lang-link}
[![Декоратор на
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Декоратор на Python"){.prog-lang-link}
[![Декоратор на
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Декоратор на Ruby"){.prog-lang-link}
[![Декоратор на
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Декоратор на Rust"){.prog-lang-link}
[![Декоратор на
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Декоратор на Swift"){.prog-lang-link}
[![Декоратор на
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Декоратор на TypeScript"){.prog-lang-link}
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
