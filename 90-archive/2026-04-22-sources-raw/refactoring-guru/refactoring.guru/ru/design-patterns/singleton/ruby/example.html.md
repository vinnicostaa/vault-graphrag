:::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Паттерны
проектирования](../../../design-patterns.html){.type} /
[Одиночка](../../singleton.html){.type} / [Ruby](../../ruby.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Одиночка](../../../../images/patterns/cards/singleton-minif559.png?id=914e1565dfdf15f240e766163bd303ec){srcset="/images/patterns/cards/singleton-mini-2x.png?id=4a7348083d7719d02709a5c9ff878b9f 2x"}
:::

# **Одиночка** на Ruby {#одиночка-на-ruby .pattern-example-title-block-title}
::::

:::::::::::::: pattern-example-body
::: pattern-example-brief
**Одиночка** --- это порождающий паттерн, который гарантирует
существование только одного объекта определённого класса, а также
позволяет достучаться до этого объекта из любого места программы.

Одиночка имеет такие же преимущества и недостатки, что и глобальные
переменные. Его невероятно удобно использовать, но он нарушает
модульность вашего кода.

Вы не сможете просто взять и использовать класс, зависящий от одиночки в
другой программе. Для этого придётся эмулировать присутствие одиночки и
там. Чаще всего эта проблема проявляется при написании юнит-тестов.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Подробней о паттерне Одиночка ](../../singleton.html){.btn .btn-lg
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
 [Наивный Одиночка (небезопасный в многопоточной среде)](#example-0)
:::

::: en-file
 [main](#example-0--main-rb)
:::

::: en-file
 [output](#example-0--output-txt)
:::

::: en-intro
 [Многопоточный Одиночка](#example-1)
:::

::: en-file
 [main](#example-1--main-rb)
:::

::: en-file
 [output](#example-1--output-txt)
:::
:::::::::::

**Сложность:** []{.fa-stars}

**Популярность:** []{.fa-stars}

**Применимость:** Многие программисты считают Одиночку антипаттерном,
поэтому его всё реже и реже можно встретить в Ruby-коде.

**Признаки применения паттерна:** Одиночку можно определить по
статическому создающему методу, который возвращает один и тот же объект.
::::::::::::::

::: {#nav-tab .nav .nav-tabs .nav-justified role="tablist"}
[Наивный Одиночка (небезопасный в многопоточной
среде)](#example-0){#example-0-tab .nav-item .nav-link .active
toggle="tab" role="tab" aria-controls="example-0" aria-selected="true"}
[Многопоточный Одиночка](#example-1){#example-1-tab .nav-item .nav-link
toggle="tab" role="tab" aria-controls="example-1" aria-selected="true"}
:::

::::: {#nav-tabContent .pattern-example-tab-content .tab-content}
::: {#example-0 .tab-pane .show .active role="tabpanel" aria-labelledby="example-0-tab" style="padding-top: 24px"}
## Наивный Одиночка (небезопасный в многопоточной среде) {#example-0-title}

Топорно реализовать Одиночку очень просто --- достаточно скрыть
конструктор и предоставить статический создающий метод.

Тот же класс ведёт себя неправильно в многопоточной среде. Несколько
потоков могут одновременно вызвать метод получения Одиночки и создать
сразу несколько экземпляров объекта.

#### []{#example-0--main-rb .anchor} **main.rb:** Пример структуры паттерна

<figure class="code">
<pre class="code" lang="ruby"><code># Класс Одиночка предоставляет метод instance, который позволяет клиентам
# получить доступ к уникальному экземпляру одиночки.
class Singleton
  @instance = new

  private_class_method :new

  # Статический метод, управляющий доступом к экземпляру одиночки.
  #
  # Эта реализация позволяет вам расширять класс Одиночки, сохраняя повсюду
  # только один экземпляр каждого подкласса.
  def self.instance
    @instance
  end

  # Наконец, любой одиночка должен содержать некоторую бизнес-логику, которая
  # может быть выполнена на его экземпляре.
  def some_business_logic
    # ...
  end
end

# Клиентский код.

s1 = Singleton.instance
s2 = Singleton.instance

if s1.equal?(s2)
  print &#39;Singleton works, both variables contain the same instance.&#39;
else
  print &#39;Singleton failed, variables contain different instances.&#39;
end</code></pre>
</figure>

#### []{#example-0--output-txt .anchor} **output.txt:** Результат выполнения

<figure class="code">
<pre class="code" lang="output"><code>Singleton works, both variables contain the same instance.</code></pre>
</figure>
:::

::: {#example-1 .tab-pane role="tabpanel" aria-labelledby="example-1-tab" style="padding-top: 24px"}
## Многопоточный Одиночка {#example-1-title}

Чтобы исправить проблему, требуется синхронизировать потоки при создании
объекта-Одиночки.

#### []{#example-1--main-rb .anchor} **main.rb:** Пример структуры паттерна

<figure class="code">
<pre class="code" lang="ruby"><code># Класс Одиночка предоставляет метод instance, который позволяет клиентам
# получить доступ к уникальному экземпляру одиночки.
class Singleton
  attr_reader :value

  @instance_mutex = Mutex.new

  private_class_method :new

  def initialize(value)
    @value = value
  end

  # Статический метод, управляющий доступом к экземпляру одиночки.
  #
  # Эта реализация позволяет вам расширять класс Одиночки, сохраняя повсюду
  # только один экземпляр каждого подкласса.
  def self.instance(value)
    return @instance if @instance

    @instance_mutex.synchronize do
      @instance ||= new(value)
    end

    @instance
  end

  # Наконец, любой одиночка должен содержать некоторую бизнес-логику, которая
  # может быть выполнена на его экземпляре.
  def some_business_logic
    # ...
  end
end

# @param [String] value
def test_singleton(value)
  singleton = Singleton.instance(value)
  puts singleton.value
end

# Клиентский код.

puts &quot;If you see the same value, then singleton was reused (yay!)\n&quot;\
     &quot;If you see different values, then 2 singletons were created (booo!!)\n\n&quot;\
     &quot;RESULT:\n\n&quot;

process1 = Thread.new { test_singleton(&#39;FOO&#39;) }
process2 = Thread.new { test_singleton(&#39;BAR&#39;) }
process1.join
process2.join</code></pre>
</figure>

#### []{#example-1--output-txt .anchor} **output.txt:** Результат выполнения

<figure class="code">
<pre class="code" lang="output"><code>If you see the same value, then singleton was reused (yay!)
If you see different values, then 2 singletons were created (booo!!)

RESULT:

FOO
FOO</code></pre>
</figure>
:::
:::::

::: {#nav-tab .nav .nav-tabs .nav-tabs-bottom .nav-justified role="tablist"}
[Наивный Одиночка (небезопасный в многопоточной
среде)](#example-0){#example-0-tab-bottom .nav-item .nav-link .active
toggle="tab" role="tab" aria-controls="example-0" aria-selected="true"}
[Многопоточный Одиночка](#example-1){#example-1-tab-bottom .nav-item
.nav-link toggle="tab" role="tab" aria-controls="example-1"
aria-selected="true"}
:::

## **Одиночка** на других языках программирования

[![Одиночка на
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Одиночка на C#"){.prog-lang-link}
[![Одиночка на
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Одиночка на C++"){.prog-lang-link}
[![Одиночка на
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Одиночка на Go"){.prog-lang-link}
[![Одиночка на
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Одиночка на Java"){.prog-lang-link}
[![Одиночка на
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Одиночка на PHP"){.prog-lang-link}
[![Одиночка на
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Одиночка на Python"){.prog-lang-link}
[![Одиночка на
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Одиночка на Rust"){.prog-lang-link}
[![Одиночка на
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Одиночка на Swift"){.prog-lang-link}
[![Одиночка на
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Одиночка на TypeScript"){.prog-lang-link}
:::::::::::::::::::::::

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
:::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::
