:::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Патерни
проектування](../../../design-patterns.html){.type} /
[Адаптер](../../adapter.html){.type} / [C++](../../cpp.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Адаптер](../../../../images/patterns/cards/adapter-minid866.png?id=b2ee4f681fb589be5a0685b94692aebb){srcset="/images/patterns/cards/adapter-mini-2x.png?id=8274d99afbbe9c63bfbfd0d68ceeffc7 2x"}
:::

# **Адаптер** на C++ {#адаптер-на-c .pattern-example-title-block-title}
::::

:::::::::::::: pattern-example-body
::: pattern-example-brief
**Адаптер** --- це структурний патерн, який дозволяє подружити несумісні
об'єкти.

Адаптер виступає прошарком між двома об'єктами, перетворюючи виклики
одного у виклики, що зрозумілі іншому.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Детальніше про Адаптер ](../../adapter.html){.btn .btn-lg
.btn-primary}
:::

::::::::::: {.sidebar-navigation .with-scroll}
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

::: en-intro
 [Множинне успадкування](#example-1)
:::

::: en-file
 [main](#example-1--main-cc)
:::

::: en-file
 [Output](#example-1--Output-txt)
:::
:::::::::::

**Складність:** []{.fa-stars}

**Популярність:** []{.fa-stars}

**Застосування:** Патерн можна часто зустріти в C++ коді, особливо там,
де потрібна конвертація різних типів даних або спільна робота класів з
різними інтерфейсами.

**Ознаки застосування патерна:** Адаптер отримує конвертований об'єкт у
конструкторі або через параметри своїх методів. Методи Адаптера,
зазвичай, сумісні з інтерфейсом одного об'єкта. Вони делегують виклики
вкладеному об'єктові, перетворивши перед цим параметри виклику у формат,
підтримуваний вкладеним об'єктом.
::::::::::::::

::: {#nav-tab .nav .nav-tabs .nav-justified role="tablist"}
[Концептуальний приклад](#example-0){#example-0-tab .nav-item .nav-link
.active toggle="tab" role="tab" aria-controls="example-0"
aria-selected="true"} [Множинне успадкування](#example-1){#example-1-tab
.nav-item .nav-link toggle="tab" role="tab" aria-controls="example-1"
aria-selected="true"}
:::

::::: {#nav-tabContent .pattern-example-tab-content .tab-content}
::: {#example-0 .tab-pane .show .active role="tabpanel" aria-labelledby="example-0-tab" style="padding-top: 24px"}
## Концептуальний приклад {#example-0-title}

Цей приклад показує структуру патерна **Адаптер**, а саме --- з яких
класів він складається, які ролі ці класи виконують і як вони
взаємодіють один з одним.

#### []{#example-0--main-cc .anchor} **main.cc:** Приклад структури патерна

<figure class="code">
<pre class="code" lang="cpp"><code>/**
 * The Target defines the domain-specific interface used by the client code.
 */
class Target {
 public:
  virtual ~Target() = default;

  virtual std::string Request() const {
    return &quot;Target: The default target&#39;s behavior.&quot;;
  }
};

/**
 * The Adaptee contains some useful behavior, but its interface is incompatible
 * with the existing client code. The Adaptee needs some adaptation before the
 * client code can use it.
 */
class Adaptee {
 public:
  std::string SpecificRequest() const {
    return &quot;.eetpadA eht fo roivaheb laicepS&quot;;
  }
};

/**
 * The Adapter makes the Adaptee&#39;s interface compatible with the Target&#39;s
 * interface.
 */
class Adapter : public Target {
 private:
  Adaptee *adaptee_;

 public:
  Adapter(Adaptee *adaptee) : adaptee_(adaptee) {}
  std::string Request() const override {
    std::string to_reverse = this-&gt;adaptee_-&gt;SpecificRequest();
    std::reverse(to_reverse.begin(), to_reverse.end());
    return &quot;Adapter: (TRANSLATED) &quot; + to_reverse;
  }
};

/**
 * The client code supports all classes that follow the Target interface.
 */
void ClientCode(const Target *target) {
  std::cout &lt;&lt; target-&gt;Request();
}

int main() {
  std::cout &lt;&lt; &quot;Client: I can work just fine with the Target objects:\n&quot;;
  Target *target = new Target;
  ClientCode(target);
  std::cout &lt;&lt; &quot;\n\n&quot;;
  Adaptee *adaptee = new Adaptee;
  std::cout &lt;&lt; &quot;Client: The Adaptee class has a weird interface. See, I don&#39;t understand it:\n&quot;;
  std::cout &lt;&lt; &quot;Adaptee: &quot; &lt;&lt; adaptee-&gt;SpecificRequest();
  std::cout &lt;&lt; &quot;\n\n&quot;;
  std::cout &lt;&lt; &quot;Client: But I can work with it via the Adapter:\n&quot;;
  Adapter *adapter = new Adapter(adaptee);
  ClientCode(adapter);
  std::cout &lt;&lt; &quot;\n&quot;;

  delete target;
  delete adaptee;
  delete adapter;

  return 0;
}</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** Результат виконання

<figure class="code">
<pre class="code" lang="output"><code>Client: I can work just fine with the Target objects:
Target: The default target&#39;s behavior.

Client: The Adaptee class has a weird interface. See, I don&#39;t understand it:
Adaptee: .eetpadA eht fo roivaheb laicepS

Client: But I can work with it via the Adapter:
Adapter: (TRANSLATED) Special behavior of the Adaptee.</code></pre>
</figure>
:::

::: {#example-1 .tab-pane role="tabpanel" aria-labelledby="example-1-tab" style="padding-top: 24px"}
## Множинне успадкування {#example-1-title}

Патерн **Адаптер** можна реалізувати на C++ за допомогою множинного
успадкування.

#### []{#example-1--main-cc .anchor} **main.cc:** Множинне успадкування

<figure class="code">
<pre class="code" lang="cpp"><code>/**
 * The Target defines the domain-specific interface used by the client code.
 */
class Target {
 public:
  virtual ~Target() = default;
  virtual std::string Request() const {
    return &quot;Target: The default target&#39;s behavior.&quot;;
  }
};

/**
 * The Adaptee contains some useful behavior, but its interface is incompatible
 * with the existing client code. The Adaptee needs some adaptation before the
 * client code can use it.
 */
class Adaptee {
 public:
  std::string SpecificRequest() const {
    return &quot;.eetpadA eht fo roivaheb laicepS&quot;;
  }
};

/**
 * The Adapter makes the Adaptee&#39;s interface compatible with the Target&#39;s
 * interface using multiple inheritance.
 */
class Adapter : public Target, public Adaptee {
 public:
  Adapter() {}
  std::string Request() const override {
    std::string to_reverse = SpecificRequest();
    std::reverse(to_reverse.begin(), to_reverse.end());
    return &quot;Adapter: (TRANSLATED) &quot; + to_reverse;
  }
};

/**
 * The client code supports all classes that follow the Target interface.
 */
void ClientCode(const Target *target) {
  std::cout &lt;&lt; target-&gt;Request();
}

int main() {
  std::cout &lt;&lt; &quot;Client: I can work just fine with the Target objects:\n&quot;;
  Target *target = new Target;
  ClientCode(target);
  std::cout &lt;&lt; &quot;\n\n&quot;;
  Adaptee *adaptee = new Adaptee;
  std::cout &lt;&lt; &quot;Client: The Adaptee class has a weird interface. See, I don&#39;t understand it:\n&quot;;
  std::cout &lt;&lt; &quot;Adaptee: &quot; &lt;&lt; adaptee-&gt;SpecificRequest();
  std::cout &lt;&lt; &quot;\n\n&quot;;
  std::cout &lt;&lt; &quot;Client: But I can work with it via the Adapter:\n&quot;;
  Adapter *adapter = new Adapter;
  ClientCode(adapter);
  std::cout &lt;&lt; &quot;\n&quot;;

  delete target;
  delete adaptee;
  delete adapter;

  return 0;
}</code></pre>
</figure>

#### []{#example-1--Output-txt .anchor} **Output.txt:** Результат виконання

<figure class="code">
<pre class="code" lang="output"><code>Client: I can work just fine with the Target objects:
Target: The default target&#39;s behavior.

Client: The Adaptee class has a weird interface. See, I don&#39;t understand it:
Adaptee: .eetpadA eht fo roivaheb laicepS

Client: But I can work with it via the Adapter:
Adapter: (TRANSLATED) Special behavior of the Adaptee.</code></pre>
</figure>
:::
:::::

::: {#nav-tab .nav .nav-tabs .nav-tabs-bottom .nav-justified role="tablist"}
[Концептуальний приклад](#example-0){#example-0-tab-bottom .nav-item
.nav-link .active toggle="tab" role="tab" aria-controls="example-0"
aria-selected="true"} [Множинне
успадкування](#example-1){#example-1-tab-bottom .nav-item .nav-link
toggle="tab" role="tab" aria-controls="example-1" aria-selected="true"}
:::

::: next
#### Читаємо далі

[Міст на C++ []{.fa
.fa-arrow-right}](../../bridge/cpp/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Повернутись назад

[[]{.fa .fa-arrow-left} Одинак на
C++](../../singleton/cpp/example.html){.btn .btn-default rel="prev"}
:::

## **Адаптер** іншими мовами програмування

[![Адаптер на
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Адаптер на C#"){.prog-lang-link}
[![Адаптер на
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Адаптер на Go"){.prog-lang-link}
[![Адаптер на
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Адаптер на Java"){.prog-lang-link}
[![Адаптер на
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Адаптер на PHP"){.prog-lang-link}
[![Адаптер на
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Адаптер на Python"){.prog-lang-link}
[![Адаптер на
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Адаптер на Ruby"){.prog-lang-link}
[![Адаптер на
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Адаптер на Rust"){.prog-lang-link}
[![Адаптер на
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Адаптер на Swift"){.prog-lang-link}
[![Адаптер на
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Адаптер на TypeScript"){.prog-lang-link}
:::::::::::::::::::::::::

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
:::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::
