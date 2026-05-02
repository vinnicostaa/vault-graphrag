:::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Паттерны
проектирования](../../../design-patterns.html){.type} /
[Наблюдатель](../../observer.html){.type} / [C++](../../cpp.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Наблюдатель](../../../../images/patterns/cards/observer-minifa2e.png?id=fd2081ab1cff29c60b499bcf6a62786a){srcset="/images/patterns/cards/observer-mini-2x.png?id=f205b0655572ac8e4636691c0e0debfd 2x"}
:::

# **Наблюдатель** на C++ {#наблюдатель-на-c .pattern-example-title-block-title}
::::

::::::::::: pattern-example-body
::: pattern-example-brief
**Наблюдатель** --- это поведенческий паттерн, который позволяет
объектам оповещать другие объекты об изменениях своего состояния.

При этом наблюдатели могут свободно подписываться и отписываться от этих
оповещений.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Подробней о паттерне Наблюдатель ](../../observer.html){.btn .btn-lg
.btn-primary}
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

**Применимость:** Наблюдатель можно часто встретить в C++ коде, особенно
там, где применяется событийная модель отношений между компонентами.
Наблюдатель позволяет отдельным компонентам реагировать на события,
происходящие в других компонентах.

**Признаки применения паттерна:** Наблюдатель можно определить по
механизму подписки и методам оповещения, которые вызывают компоненты
программы.
:::::::::::

[]{#example-0}

## Концептуальный пример {#example-0-title}

Этот пример показывает структуру паттерна **Наблюдатель**, а именно ---
из каких классов он состоит, какие роли эти классы выполняют и как они
взаимодействуют друг с другом.

#### []{#example-0--main-cc .anchor} **main.cc:** Пример структуры паттерна

<figure class="code">
<pre class="code" lang="cpp"><code>/**
 * Паттерн Наблюдатель
 *
 * Назначение: Создаёт механизм подписки, позволяющий одним объектам следить и
 * реагировать на события, происходящие в других объектах.
 *
 * Обратите внимание, что существует множество различных терминов с похожими
 * значениями, связанных с этим паттерном. Просто помните, что Субъекта также
 * называют Издателем, а Наблюдателя часто называют Подписчиком и наоборот.
 * Также глаголы «наблюдать», «слушать» или «отслеживать» обычно означают одно и
 * то же.
 */

#include &lt;iostream&gt;
#include &lt;list&gt;
#include &lt;string&gt;

class IObserver {
 public:
  virtual ~IObserver(){};
  virtual void Update(const std::string &amp;message_from_subject) = 0;
};

class ISubject {
 public:
  virtual ~ISubject(){};
  virtual void Attach(IObserver *observer) = 0;
  virtual void Detach(IObserver *observer) = 0;
  virtual void Notify() = 0;
};

/**
 * Издатель владеет некоторым важным состоянием и оповещает наблюдателей о его
 * изменениях.
 */

class Subject : public ISubject {
 public:
  virtual ~Subject() {
    std::cout &lt;&lt; &quot;Goodbye, I was the Subject.\n&quot;;
  }

  /**
   * Методы управления подпиской.
   */
  void Attach(IObserver *observer) override {
    list_observer_.push_back(observer);
  }
  void Detach(IObserver *observer) override {
    list_observer_.remove(observer);
  }
  void Notify() override {
    std::list&lt;IObserver *&gt;::iterator iterator = list_observer_.begin();
    HowManyObserver();
    while (iterator != list_observer_.end()) {
      (*iterator)-&gt;Update(message_);
      ++iterator;
    }
  }

  void CreateMessage(std::string message = &quot;Empty&quot;) {
    this-&gt;message_ = message;
    Notify();
  }
  void HowManyObserver() {
    std::cout &lt;&lt; &quot;There are &quot; &lt;&lt; list_observer_.size() &lt;&lt; &quot; observers in the list.\n&quot;;
  }

  /**
   * Обычно логика подписки – только часть того, что делает Издатель. Издатели
   * часто содержат некоторую важную бизнес-логику, которая запускает метод
   * уведомления всякий раз, когда должно произойти что-то важное (или после
   * этого).
   */
  void SomeBusinessLogic() {
    this-&gt;message_ = &quot;change message message&quot;;
    Notify();
    std::cout &lt;&lt; &quot;I&#39;m about to do some thing important\n&quot;;
  }

 private:
  std::list&lt;IObserver *&gt; list_observer_;
  std::string message_;
};

class Observer : public IObserver {
 public:
  Observer(Subject &amp;subject) : subject_(subject) {
    this-&gt;subject_.Attach(this);
    std::cout &lt;&lt; &quot;Hi, I&#39;m the Observer \&quot;&quot; &lt;&lt; ++Observer::static_number_ &lt;&lt; &quot;\&quot;.\n&quot;;
    this-&gt;number_ = Observer::static_number_;
  }
  virtual ~Observer() {
    std::cout &lt;&lt; &quot;Goodbye, I was the Observer \&quot;&quot; &lt;&lt; this-&gt;number_ &lt;&lt; &quot;\&quot;.\n&quot;;
  }

  void Update(const std::string &amp;message_from_subject) override {
    message_from_subject_ = message_from_subject;
    PrintInfo();
  }
  void RemoveMeFromTheList() {
    subject_.Detach(this);
    std::cout &lt;&lt; &quot;Observer \&quot;&quot; &lt;&lt; number_ &lt;&lt; &quot;\&quot; removed from the list.\n&quot;;
  }
  void PrintInfo() {
    std::cout &lt;&lt; &quot;Observer \&quot;&quot; &lt;&lt; this-&gt;number_ &lt;&lt; &quot;\&quot;: a new message is available --&gt; &quot; &lt;&lt; this-&gt;message_from_subject_ &lt;&lt; &quot;\n&quot;;
  }

 private:
  std::string message_from_subject_;
  Subject &amp;subject_;
  static int static_number_;
  int number_;
};

int Observer::static_number_ = 0;

void ClientCode() {
  Subject *subject = new Subject;
  Observer *observer1 = new Observer(*subject);
  Observer *observer2 = new Observer(*subject);
  Observer *observer3 = new Observer(*subject);
  Observer *observer4;
  Observer *observer5;

  subject-&gt;CreateMessage(&quot;Hello World! :D&quot;);
  observer3-&gt;RemoveMeFromTheList();

  subject-&gt;CreateMessage(&quot;The weather is hot today! :p&quot;);
  observer4 = new Observer(*subject);

  observer2-&gt;RemoveMeFromTheList();
  observer5 = new Observer(*subject);

  subject-&gt;CreateMessage(&quot;My new car is great! ;)&quot;);
  observer5-&gt;RemoveMeFromTheList();

  observer4-&gt;RemoveMeFromTheList();
  observer1-&gt;RemoveMeFromTheList();

  delete observer5;
  delete observer4;
  delete observer3;
  delete observer2;
  delete observer1;
  delete subject;
}

int main() {
  ClientCode();
  return 0;
}</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** Результат выполнения

<figure class="code">
<pre class="code" lang="output"><code>Hi, I&#39;m the Observer &quot;1&quot;.
Hi, I&#39;m the Observer &quot;2&quot;.
Hi, I&#39;m the Observer &quot;3&quot;.
There are 3 observers in the list.
Observer &quot;1&quot;: a new message is available --&gt; Hello World! :D
Observer &quot;2&quot;: a new message is available --&gt; Hello World! :D
Observer &quot;3&quot;: a new message is available --&gt; Hello World! :D
Observer &quot;3&quot; removed from the list.
There are 2 observers in the list.
Observer &quot;1&quot;: a new message is available --&gt; The weather is hot today! :p
Observer &quot;2&quot;: a new message is available --&gt; The weather is hot today! :p
Hi, I&#39;m the Observer &quot;4&quot;.
Observer &quot;2&quot; removed from the list.
Hi, I&#39;m the Observer &quot;5&quot;.
There are 3 observers in the list.
Observer &quot;1&quot;: a new message is available --&gt; My new car is great! ;)
Observer &quot;4&quot;: a new message is available --&gt; My new car is great! ;)
Observer &quot;5&quot;: a new message is available --&gt; My new car is great! ;)
Observer &quot;5&quot; removed from the list.
Observer &quot;4&quot; removed from the list.
Observer &quot;1&quot; removed from the list.
Goodbye, I was the Observer &quot;5&quot;.
Goodbye, I was the Observer &quot;4&quot;.
Goodbye, I was the Observer &quot;3&quot;.
Goodbye, I was the Observer &quot;2&quot;.
Goodbye, I was the Observer &quot;1&quot;.
Goodbye, I was the Subject.</code></pre>
</figure>

::: next
#### Читаем дальше

[Состояние на C++ []{.fa
.fa-arrow-right}](../../state/cpp/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Снимок на
C++](../../memento/cpp/example.html){.btn .btn-default rel="prev"}
:::

## **Наблюдатель** на других языках программирования

[![Наблюдатель на
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Наблюдатель на C#"){.prog-lang-link}
[![Наблюдатель на
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Наблюдатель на Go"){.prog-lang-link}
[![Наблюдатель на
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Наблюдатель на Java"){.prog-lang-link}
[![Наблюдатель на
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Наблюдатель на PHP"){.prog-lang-link}
[![Наблюдатель на
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Наблюдатель на Python"){.prog-lang-link}
[![Наблюдатель на
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Наблюдатель на Ruby"){.prog-lang-link}
[![Наблюдатель на
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Наблюдатель на Rust"){.prog-lang-link}
[![Наблюдатель на
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Наблюдатель на Swift"){.prog-lang-link}
[![Наблюдатель на
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Наблюдатель на TypeScript"){.prog-lang-link}
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
