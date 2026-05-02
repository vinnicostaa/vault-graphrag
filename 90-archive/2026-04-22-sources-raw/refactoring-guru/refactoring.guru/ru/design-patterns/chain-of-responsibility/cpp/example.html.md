:::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Паттерны
проектирования](../../../design-patterns.html){.type} / [Цепочка
обязанностей](../../chain-of-responsibility.html){.type} /
[C++](../../cpp.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Цепочка
обязанностей](../../../../images/patterns/cards/chain-of-responsibility-mini6d42.png?id=36d85eba8d14986f053123de17aac7a7){srcset="/images/patterns/cards/chain-of-responsibility-mini-2x.png?id=8c81f7069e51259b2443801b91135f7f 2x"}
:::

# **Цепочка обязанностей** на C++ {#цепочка-обязанностей-на-c .pattern-example-title-block-title}
::::

::::::::::: pattern-example-body
::: pattern-example-brief
**Цепочка обязанностей** --- это поведенческий паттерн, позволяющий
передавать запрос по цепочке потенциальных обработчиков, пока один из
них не обработает запрос.

Избавляет от жёсткой привязки отправителя запроса к его получателю,
позволяя выстраивать цепь из различных обработчиков динамически.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Подробней о паттерне Цепочка обязанностей
](../../chain-of-responsibility.html){.btn .btn-lg .btn-primary}
:::

:::::::: {.sidebar-navigation .with-scroll}
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
 [main](#example-0--main-cc)
:::

::: en-file
 [Output](#example-0--Output-txt)
:::
::::::::

**Сложность:** []{.fa-stars}

**Популярность:** []{.fa-stars}

**Применимость:** Паттерн встречается в C++ не так уж часто, так как для
его применения нужна цепь объектов, например, связанный список.

**Признаки применения паттерна:** Цепочку обязанностей можно определить
по спискам обработчиков или проверок, через которые пропускаются
запросы. Особенно если порядок следования обработчиков важен.
:::::::::::

[]{#example-0}

## Концептуальный пример {#example-0-title}

Этот пример показывает структуру паттерна **Цепочка обязанностей**, а
именно --- из каких классов он состоит, какие роли эти классы выполняют
и как они взаимодействуют друг с другом.

#### []{#example-0--main-cc .anchor} **main.cc:** Пример структуры паттерна

<figure class="code">
<pre class="code" lang="cpp"><code>/**
 * Интерфейс Обработчика объявляет метод построения цепочки обработчиков. Он
 * также объявляет метод для выполнения запроса.
 */
class Handler {
 public:
  virtual Handler *SetNext(Handler *handler) = 0;
  virtual std::string Handle(std::string request) = 0;
};
/**
 * Поведение цепочки по умолчанию может быть реализовано внутри базового класса
 * обработчика.
 */
class AbstractHandler : public Handler {
  /**
   * @var Handler
   */
 private:
  Handler *next_handler_;

 public:
  AbstractHandler() : next_handler_(nullptr) {
  }
  Handler *SetNext(Handler *handler) override {
    this-&gt;next_handler_ = handler;
    // Возврат обработчика отсюда позволит связать обработчики простым способом,
    // вот так:
    // $monkey-&gt;setNext($squirrel)-&gt;setNext($dog);
    return handler;
  }
  std::string Handle(std::string request) override {
    if (this-&gt;next_handler_) {
      return this-&gt;next_handler_-&gt;Handle(request);
    }

    return {};
  }
};
/**
 * Все Конкретные Обработчики либо обрабатывают запрос, либо передают его
 * следующему обработчику в цепочке.
 */
class MonkeyHandler : public AbstractHandler {
 public:
  std::string Handle(std::string request) override {
    if (request == &quot;Banana&quot;) {
      return &quot;Monkey: I&#39;ll eat the &quot; + request + &quot;.\n&quot;;
    } else {
      return AbstractHandler::Handle(request);
    }
  }
};
class SquirrelHandler : public AbstractHandler {
 public:
  std::string Handle(std::string request) override {
    if (request == &quot;Nut&quot;) {
      return &quot;Squirrel: I&#39;ll eat the &quot; + request + &quot;.\n&quot;;
    } else {
      return AbstractHandler::Handle(request);
    }
  }
};
class DogHandler : public AbstractHandler {
 public:
  std::string Handle(std::string request) override {
    if (request == &quot;MeatBall&quot;) {
      return &quot;Dog: I&#39;ll eat the &quot; + request + &quot;.\n&quot;;
    } else {
      return AbstractHandler::Handle(request);
    }
  }
};
/**
 * Обычно клиентский код приспособлен для работы с единственным обработчиком. В
 * большинстве случаев клиенту даже неизвестно, что этот обработчик является
 * частью цепочки.
 */
void ClientCode(Handler &amp;handler) {
  std::vector&lt;std::string&gt; food = {&quot;Nut&quot;, &quot;Banana&quot;, &quot;Cup of coffee&quot;};
  for (const std::string &amp;f : food) {
    std::cout &lt;&lt; &quot;Client: Who wants a &quot; &lt;&lt; f &lt;&lt; &quot;?\n&quot;;
    const std::string result = handler.Handle(f);
    if (!result.empty()) {
      std::cout &lt;&lt; &quot;  &quot; &lt;&lt; result;
    } else {
      std::cout &lt;&lt; &quot;  &quot; &lt;&lt; f &lt;&lt; &quot; was left untouched.\n&quot;;
    }
  }
}
/**
 * Другая часть клиентского кода создает саму цепочку.
 */
int main() {
  MonkeyHandler *monkey = new MonkeyHandler;
  SquirrelHandler *squirrel = new SquirrelHandler;
  DogHandler *dog = new DogHandler;
  monkey-&gt;SetNext(squirrel)-&gt;SetNext(dog);

  /**
   * Клиент должен иметь возможность отправлять запрос любому обработчику, а не
   * только первому в цепочке.
   */
  std::cout &lt;&lt; &quot;Chain: Monkey &gt; Squirrel &gt; Dog\n\n&quot;;
  ClientCode(*monkey);
  std::cout &lt;&lt; &quot;\n&quot;;
  std::cout &lt;&lt; &quot;Subchain: Squirrel &gt; Dog\n\n&quot;;
  ClientCode(*squirrel);

  delete monkey;
  delete squirrel;
  delete dog;

  return 0;
}</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** Результат выполнения

<figure class="code">
<pre class="code" lang="output"><code>Chain: Monkey &gt; Squirrel &gt; Dog

Client: Who wants a Nut?
  Squirrel: I&#39;ll eat the Nut.
Client: Who wants a Banana?
  Monkey: I&#39;ll eat the Banana.
Client: Who wants a Cup of coffee?
  Cup of coffee was left untouched.

Subchain: Squirrel &gt; Dog

Client: Who wants a Nut?
  Squirrel: I&#39;ll eat the Nut.
Client: Who wants a Banana?
  Banana was left untouched.
Client: Who wants a Cup of coffee?
  Cup of coffee was left untouched.</code></pre>
</figure>

::: next
#### Читаем дальше

[Команда на C++ []{.fa
.fa-arrow-right}](../../command/cpp/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Заместитель на
C++](../../proxy/cpp/example.html){.btn .btn-default rel="prev"}
:::

## **Цепочка обязанностей** на других языках программирования

[![Цепочка обязанностей на
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Цепочка обязанностей на C#"){.prog-lang-link}
[![Цепочка обязанностей на
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Цепочка обязанностей на Go"){.prog-lang-link}
[![Цепочка обязанностей на
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Цепочка обязанностей на Java"){.prog-lang-link}
[![Цепочка обязанностей на
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Цепочка обязанностей на PHP"){.prog-lang-link}
[![Цепочка обязанностей на
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Цепочка обязанностей на Python"){.prog-lang-link}
[![Цепочка обязанностей на
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Цепочка обязанностей на Ruby"){.prog-lang-link}
[![Цепочка обязанностей на
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Цепочка обязанностей на Rust"){.prog-lang-link}
[![Цепочка обязанностей на
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Цепочка обязанностей на Swift"){.prog-lang-link}
[![Цепочка обязанностей на
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Цепочка обязанностей на TypeScript"){.prog-lang-link}
:::::::::::::::::

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
:::::::::::::::::::::::
::::::::::::::::::::::::
