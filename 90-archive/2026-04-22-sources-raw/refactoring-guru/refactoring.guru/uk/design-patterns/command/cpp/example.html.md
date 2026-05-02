:::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Патерни
проектування](../../../design-patterns.html){.type} /
[Команда](../../command.html){.type} / [C++](../../cpp.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Команда](../../../../images/patterns/cards/command-mini8482.png?id=b149eda017c0583c1e92343b83cfb1eb){srcset="/images/patterns/cards/command-mini-2x.png?id=e5f6332e057f6d352a209da963a8fc54 2x"}
:::

# **Команда** на C++ {#команда-на-c .pattern-example-title-block-title}
::::

::::::::::: pattern-example-body
::: pattern-example-brief
**Команда** --- це поведінковий патерн, що дозволяє загортати запити або
прості операції в окремі об'єкти.

Це дозволяє відкладати виконання команд, вибудовувати їх у чергу, а
також зберігати історію і робити скасування.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Детальніше про Команда ](../../command.html){.btn .btn-lg
.btn-primary}
:::

:::::::: {.sidebar-navigation .with-scroll}
::: en-title
Навігація
:::

::: en-intro
 [Інтро](#)
:::

::: en-intro
 [Концептуальний приклад](#example-0)
:::

::: en-file
 [main](#example-0--main-cc)
:::

::: en-file
 [Output](#example-0--Output-txt)
:::
::::::::

**Складність:** []{.fa-stars}

**Популярність:** []{.fa-stars}

**Застосування:** Патерн часто зустрічається в C++ коді, особливо якщо
потрібно відкладати виконання команд, вибудовувати їх у чергу, а також
зберігати історію та робити скасування.

**Ознаки застосування патерна:** Класи команд побудовані навколо однієї
дії та мають дуже вузький контекст. Об'єкти команд часто подаються в
обробники подій елементів GUI. Практично будь-яка реалізація скасування
використовує принцип команд.
:::::::::::

[]{#example-0}

## Концептуальний приклад {#example-0-title}

Цей приклад показує структуру патерна **Команда**, а саме --- з яких
класів він складається, які ролі ці класи виконують і як вони
взаємодіють один з одним.

#### []{#example-0--main-cc .anchor} **main.cc:** Приклад структури патерна

<figure class="code">
<pre class="code" lang="cpp"><code>/**
 * The Command interface declares a method for executing a command.
 */
class Command {
 public:
  virtual ~Command() {
  }
  virtual void Execute() const = 0;
};
/**
 * Some commands can implement simple operations on their own.
 */
class SimpleCommand : public Command {
 private:
  std::string pay_load_;

 public:
  explicit SimpleCommand(std::string pay_load) : pay_load_(pay_load) {
  }
  void Execute() const override {
    std::cout &lt;&lt; &quot;SimpleCommand: See, I can do simple things like printing (&quot; &lt;&lt; this-&gt;pay_load_ &lt;&lt; &quot;)\n&quot;;
  }
};

/**
 * The Receiver classes contain some important business logic. They know how to
 * perform all kinds of operations, associated with carrying out a request. In
 * fact, any class may serve as a Receiver.
 */
class Receiver {
 public:
  void DoSomething(const std::string &amp;a) {
    std::cout &lt;&lt; &quot;Receiver: Working on (&quot; &lt;&lt; a &lt;&lt; &quot;.)\n&quot;;
  }
  void DoSomethingElse(const std::string &amp;b) {
    std::cout &lt;&lt; &quot;Receiver: Also working on (&quot; &lt;&lt; b &lt;&lt; &quot;.)\n&quot;;
  }
};

/**
 * However, some commands can delegate more complex operations to other objects,
 * called &quot;receivers.&quot;
 */
class ComplexCommand : public Command {
  /**
   * @var Receiver
   */
 private:
  Receiver *receiver_;
  /**
   * Context data, required for launching the receiver&#39;s methods.
   */
  std::string a_;
  std::string b_;
  /**
   * Complex commands can accept one or several receiver objects along with any
   * context data via the constructor.
   */
 public:
  ComplexCommand(Receiver *receiver, std::string a, std::string b) : receiver_(receiver), a_(a), b_(b) {
  }
  /**
   * Commands can delegate to any methods of a receiver.
   */
  void Execute() const override {
    std::cout &lt;&lt; &quot;ComplexCommand: Complex stuff should be done by a receiver object.\n&quot;;
    this-&gt;receiver_-&gt;DoSomething(this-&gt;a_);
    this-&gt;receiver_-&gt;DoSomethingElse(this-&gt;b_);
  }
};

/**
 * The Invoker is associated with one or several commands. It sends a request to
 * the command.
 */
class Invoker {
  /**
   * @var Command
   */
 private:
  Command *on_start_;
  /**
   * @var Command
   */
  Command *on_finish_;
  /**
   * Initialize commands.
   */
 public:
  ~Invoker() {
    delete on_start_;
    delete on_finish_;
  }

  void SetOnStart(Command *command) {
    this-&gt;on_start_ = command;
  }
  void SetOnFinish(Command *command) {
    this-&gt;on_finish_ = command;
  }
  /**
   * The Invoker does not depend on concrete command or receiver classes. The
   * Invoker passes a request to a receiver indirectly, by executing a command.
   */
  void DoSomethingImportant() {
    std::cout &lt;&lt; &quot;Invoker: Does anybody want something done before I begin?\n&quot;;
    if (this-&gt;on_start_) {
      this-&gt;on_start_-&gt;Execute();
    }
    std::cout &lt;&lt; &quot;Invoker: ...doing something really important...\n&quot;;
    std::cout &lt;&lt; &quot;Invoker: Does anybody want something done after I finish?\n&quot;;
    if (this-&gt;on_finish_) {
      this-&gt;on_finish_-&gt;Execute();
    }
  }
};
/**
 * The client code can parameterize an invoker with any commands.
 */

int main() {
  Invoker *invoker = new Invoker;
  invoker-&gt;SetOnStart(new SimpleCommand(&quot;Say Hi!&quot;));
  Receiver *receiver = new Receiver;
  invoker-&gt;SetOnFinish(new ComplexCommand(receiver, &quot;Send email&quot;, &quot;Save report&quot;));
  invoker-&gt;DoSomethingImportant();

  delete invoker;
  delete receiver;

  return 0;
}</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** Результат виконання

<figure class="code">
<pre class="code" lang="output"><code>Invoker: Does anybody want something done before I begin?
SimpleCommand: See, I can do simple things like printing (Say Hi!)
Invoker: ...doing something really important...
Invoker: Does anybody want something done after I finish?
ComplexCommand: Complex stuff should be done by a receiver object.
Receiver: Working on (Send email.)
Receiver: Also working on (Save report.)</code></pre>
</figure>

::: next
#### Читаємо далі

[Ітератор на C++ []{.fa
.fa-arrow-right}](../../iterator/cpp/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Повернутись назад

[[]{.fa .fa-arrow-left} Ланцюжок обов\'язків на
C++](../../chain-of-responsibility/cpp/example.html){.btn .btn-default
rel="prev"}
:::

## **Команда** іншими мовами програмування

[![Команда на
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Команда на C#"){.prog-lang-link}
[![Команда на
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Команда на Go"){.prog-lang-link}
[![Команда на
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Команда на Java"){.prog-lang-link}
[![Команда на
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Команда на PHP"){.prog-lang-link}
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
:::::::::::::::::

::::::: {.prom .banner-sidebar .banner .banner-striped .banner-examples .banner-removable .banner-removable-patterns data-id="DP: Examples-IDE" creative-id="examples-sidebar-uk" position="sidebar"}
:::::: {.banner-inner style="font-size: 14px; line-height: 21px;"}
::: banner-examples-img
[![](../../../../images/patterns/banners/examples-ide91ca.png?id=3115b4b548fb96b75974e2de8f4f49bc){width="215"
height="158" loading="lazy"
srcset="/images/patterns/banners/examples-ide-2x.png?id=93c007a6d157b97c28eb213f59d716ae 2x"}](../../book.html)
:::

::: banner-examples-fade
[ Архів з прикладами](../../book.html){.btn .btn-secondary .btn-block}
:::

::: {.banner-examples-text style="margin-top: 1rem;"}
Придбай книгу **Занурення в Патерни** та отримай архів з десятками
детальних прикладів, які можна відкривати прямо в IDE.

[ Дізнатися більше...](../../book.html){.btn .btn-secondary .btn-block}
:::
::::::
:::::::
:::::::::::::::::::::::
::::::::::::::::::::::::
