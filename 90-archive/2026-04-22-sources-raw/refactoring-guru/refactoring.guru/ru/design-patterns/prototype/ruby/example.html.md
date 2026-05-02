:::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Паттерны
проектирования](../../../design-patterns.html){.type} /
[Прототип](../../prototype.html){.type} / [Ruby](../../ruby.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Прототип](../../../../images/patterns/cards/prototype-mini6540.png?id=bc3046bb39ff36574c08d49839fd1c8e){srcset="/images/patterns/cards/prototype-mini-2x.png?id=b871f789a736e7efbb1cd082d2de6398 2x"}
:::

# **Прототип** на Ruby {#прототип-на-ruby .pattern-example-title-block-title}
::::

::::::::::: pattern-example-body
::: pattern-example-brief
**Прототип** --- это порождающий паттерн, который позволяет копировать
объекты любой сложности без привязки к их конкретным классам.

Все классы---Прототипы имеют общий интерфейс. Поэтому вы можете
копировать объекты, не обращая внимания на их конкретные типы и всегда
быть уверены, что получите точную копию. Клонирование совершается самим
объектом-прототипом, что позволяет ему скопировать значения всех полей,
даже приватных.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Подробней о паттерне Прототип ](../../prototype.html){.btn .btn-lg
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

**Применимость:** Паттерн Прототип реализован в базовой библиотеке Ruby
посредством методов `dup` или `clone`.

**Признаки применения паттерна:** Прототип легко определяется в коде по
наличию методов `clone`, `copy` и прочих.
:::::::::::

[]{#example-0}

## Концептуальный пример {#example-0-title}

Этот пример показывает структуру паттерна **Прототип**, а именно --- из
каких классов он состоит, какие роли эти классы выполняют и как они
взаимодействуют друг с другом.

#### []{#example-0--main-rb .anchor} **main.rb:** Пример структуры паттерна

<figure class="code">
<pre class="code" lang="ruby"><code># Пример класса, имеющего возможность клонирования. Мы посмотрим как происходит
# клонирование значений полей разных типов.
class Prototype
  attr_accessor :primitive, :component, :circular_reference

  def initialize
    @primitive = nil
    @component = nil
    @circular_reference = nil
  end

  # @return [Prototype]
  def clone
    @component = deep_copy(@component)

    # Клонирование объекта, который имеет вложенный объект с обратной ссылкой,
    # требует специального подхода. После завершения клонирования вложенный
    # объект должен указывать на клонированный объект, а не на исходный объект.
    @circular_reference = deep_copy(@circular_reference)
    @circular_reference.prototype = self
    deep_copy(self)
  end

  # Нередко метод deep_copy использует хак «маршалинг», чтобы создать глубокую
  # копию объекта. Однако это медленное и неэффективно, поэтому в реальных
  # приложениях используйте для этой задачи соответствующий пакет.
  #
  # @param [Object] object
  private def deep_copy(object)
    Marshal.load(Marshal.dump(object))
  end
end

class ComponentWithBackReference
  attr_accessor :prototype

  # @param [Prototype] prototype
  def initialize(prototype)
    @prototype = prototype
  end
end

# Клиентский код.
p1 = Prototype.new
p1.primitive = 245
p1.component = Time.now
p1.circular_reference = ComponentWithBackReference.new(p1)

p2 = p1.clone

if p1.primitive == p2.primitive
  puts &#39;Primitive field values have been carried over to a clone. Yay!&#39;
else
  puts &#39;Primitive field values have not been copied. Booo!&#39;
end

if p1.component.equal?(p2.component)
  puts &#39;Simple component has not been cloned. Booo!&#39;
else
  puts &#39;Simple component has been cloned. Yay!&#39;
end

if p1.circular_reference.equal?(p2.circular_reference)
  puts &#39;Component with back reference has not been cloned. Booo!&#39;
else
  puts &#39;Component with back reference has been cloned. Yay!&#39;
end

if p1.circular_reference.prototype.equal?(p2.circular_reference.prototype)
  print &#39;Component with back reference is linked to original object. Booo!&#39;
else
  print &#39;Component with back reference is linked to the clone. Yay!&#39;
end</code></pre>
</figure>

#### []{#example-0--output-txt .anchor} **output.txt:** Результат выполнения

<figure class="code">
<pre class="code" lang="output"><code>Primitive field values have been carried over to a clone. Yay!
Simple component has been cloned. Yay!
Component with back reference has been cloned. Yay!
Component with back reference is linked to the clone. Yay!</code></pre>
</figure>

## **Прототип** на других языках программирования

[![Прототип на
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Прототип на C#"){.prog-lang-link}
[![Прототип на
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Прототип на C++"){.prog-lang-link}
[![Прототип на
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Прототип на Go"){.prog-lang-link}
[![Прототип на
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Прототип на Java"){.prog-lang-link}
[![Прототип на
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Прототип на PHP"){.prog-lang-link}
[![Прототип на
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Прототип на Python"){.prog-lang-link}
[![Прототип на
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Прототип на Rust"){.prog-lang-link}
[![Прототип на
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Прототип на Swift"){.prog-lang-link}
[![Прототип на
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Прототип на TypeScript"){.prog-lang-link}
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
