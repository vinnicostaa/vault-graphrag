:::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Паттерны
проектирования](../../../design-patterns.html){.type} /
[Адаптер](../../adapter.html){.type} / [Ruby](../../ruby.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Адаптер](../../../../images/patterns/cards/adapter-minid866.png?id=b2ee4f681fb589be5a0685b94692aebb){srcset="/images/patterns/cards/adapter-mini-2x.png?id=8274d99afbbe9c63bfbfd0d68ceeffc7 2x"}
:::

# **Адаптер** на Ruby {#адаптер-на-ruby .pattern-example-title-block-title}
::::

::::::::::: pattern-example-body
::: pattern-example-brief
**Адаптер** --- это структурный паттерн, который позволяет подружить
несовместимые объекты.

Адаптер выступает прослойкой между двумя объектами, превращая вызовы
одного в вызовы понятные другому.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Подробней о паттерне Адаптер ](../../adapter.html){.btn .btn-lg
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
 [main](#example-0--main-rb)
:::

::: en-file
 [output](#example-0--output-txt)
:::
::::::::

**Сложность:** []{.fa-stars}

**Популярность:** []{.fa-stars}

**Применимость:** Паттерн можно часто встретить в Ruby-коде, особенно
там, где требуется конвертация разных типов данных или совместная работа
классов с разными интерфейсами.

**Признаки применения паттерна:** Адаптер получает конвертируемый объект
в конструкторе или через параметры своих методов. Методы Адаптера обычно
совместимы с интерфейсом одного объекта. Они делегируют вызовы
вложенному объекту, превратив перед этим параметры вызова в формат,
поддерживаемый вложенным объектом.
:::::::::::

[]{#example-0}

## Концептуальный пример {#example-0-title}

Этот пример показывает структуру паттерна **Адаптер**, а именно --- из
каких классов он состоит, какие роли эти классы выполняют и как они
взаимодействуют друг с другом.

#### []{#example-0--main-rb .anchor} **main.rb:** Пример структуры паттерна

<figure class="code">
<pre class="code" lang="ruby"><code># Целевой класс объявляет интерфейс, с которым может работать клиентский код.
class Target
  # @return [String]
  def request
    &#39;Target: The default target\&#39;s behavior.&#39;
  end
end

# Адаптируемый класс содержит некоторое полезное поведение, но его интерфейс
# несовместим с существующим клиентским кодом. Адаптируемый класс нуждается в
# некоторой доработке, прежде чем клиентский код сможет его использовать.
class Adaptee
  # @return [String]
  def specific_request
    &#39;.eetpadA eht fo roivaheb laicepS&#39;
  end
end

# Адаптер делает интерфейс Адаптируемого класса совместимым с целевым
# интерфейсом.
class Adapter &lt; Target
  # @param [Adaptee] adaptee
  def initialize(adaptee)
    @adaptee = adaptee
  end

  def request
    &quot;Adapter: (TRANSLATED) #{@adaptee.specific_request.reverse!}&quot;
  end
end

# Клиентский код поддерживает все классы, использующие интерфейс Target.
#
# @param [Target] target
def client_code(target)
  print target.request
end

puts &#39;Client: I can work just fine with the Target objects:&#39;
target = Target.new
client_code(target)
puts &quot;\n\n&quot;

adaptee = Adaptee.new
puts &#39;Client: The Adaptee class has a weird interface. See, I don\&#39;t understand it:&#39;
puts &quot;Adaptee: #{adaptee.specific_request}&quot;
puts &quot;\n&quot;

puts &#39;Client: But I can work with it via the Adapter:&#39;
adapter = Adapter.new(adaptee)
client_code(adapter)</code></pre>
</figure>

#### []{#example-0--output-txt .anchor} **output.txt:** Результат выполнения

<figure class="code">
<pre class="code" lang="output"><code>Client: I can work just fine with the Target objects:
Target: The default target&#39;s behavior.

Client: The Adaptee class has a weird interface. See, I don&#39;t understand it:
Adaptee: .eetpadA eht fo roivaheb laicepS

Client: But I can work with it via the Adapter:
Adapter: (TRANSLATED) Special behavior of the Adaptee.</code></pre>
</figure>

## **Адаптер** на других языках программирования

[![Адаптер на
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Адаптер на C#"){.prog-lang-link}
[![Адаптер на
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Адаптер на C++"){.prog-lang-link}
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
:::::::::::::::

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
:::::::::::::::::::::
::::::::::::::::::::::
