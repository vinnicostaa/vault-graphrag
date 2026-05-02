:::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Патерни
проектування](../../../design-patterns.html){.type} /
[Легковаговик](../../flyweight.html){.type} /
[Ruby](../../ruby.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Легковаговик](../../../../images/patterns/cards/flyweight-minie3d8.png?id=422ca8d2f90614dce810a8812c626698){srcset="/images/patterns/cards/flyweight-mini-2x.png?id=857ca676fff84317bb0dab76abfce08e 2x"}
:::

# **Легковаговик** на Ruby {#легковаговик-на-ruby .pattern-example-title-block-title}
::::

::::::::::: pattern-example-body
::: pattern-example-brief
**Легковаговик** --- це структурний патерн, який економить пам'ять
завдяки розподілу спільного стану, винесеного в один об'єкт, між
безліччю об'єктів.

Легковаговик дозволяє економити пам'ять, записуючи в кеш однакові дані,
що використовуються різними об'єктами.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Детальніше про Легковаговик ](../../flyweight.html){.btn .btn-lg
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
 [main](#example-0--main-rb)
:::

::: en-file
 [output](#example-0--output-txt)
:::
::::::::

**Складність:** []{.fa-stars}

**Популярність:** []{.fa-stars}

**Застосування:** Сенс використання Легковаговика --- це економія
пам'яті. Тому, якщо в програмі немає такої проблеми, ви навряд чи
знайдете там приклади Легковаговика.

**Ознаки застосування патерна:** Легковаговик можна визначити за
створюваними методами класу, які повертають закешовані об'єкти, замість
створення нових.
:::::::::::

[]{#example-0}

## Концептуальний приклад {#example-0-title}

Цей приклад показує структуру патерна **Легковаговик**, а саме --- з
яких класів він складається, які ролі ці класи виконують і як вони
взаємодіють один з одним.

#### []{#example-0--main-rb .anchor} **main.rb:** Приклад структури патерна

<figure class="code">
<pre class="code" lang="ruby"><code>require &#39;json&#39;

# The Flyweight stores a common portion of the state (also called intrinsic
# state) that belongs to multiple real business entities. The Flyweight accepts
# the rest of the state (extrinsic state, unique for each entity) via its method
# parameters.
class Flyweight
  # @param [String] shared_state
  def initialize(shared_state)
    @shared_state = shared_state
  end

  # @param [String] unique_state
  def operation(unique_state)
    s = @shared_state.to_json
    u = unique_state.to_json
    print &quot;Flyweight: Displaying shared (#{s}) and unique (#{u}) state.&quot;
  end
end

# The Flyweight Factory creates and manages the Flyweight objects. It ensures
# that flyweights are shared correctly. When the client requests a flyweight,
# the factory either returns an existing instance or creates a new one, if it
# doesn&#39;t exist yet.
class FlyweightFactory
  # @param [Hash] initial_flyweights
  def initialize(initial_flyweights)
    @flyweights = {}
    initial_flyweights.each do |state|
      @flyweights[get_key(state)] = Flyweight.new(state)
    end
  end

  # Returns a Flyweight&#39;s string hash for a given state.
  def get_key(state)
    state.sort.join(&#39;_&#39;)
  end

  # Returns an existing Flyweight with a given state or creates a new one.
  def get_flyweight(shared_state)
    key = get_key(shared_state)

    if !@flyweights.key?(key)
      puts &quot;FlyweightFactory: Can&#39;t find a flyweight, creating new one.&quot;
      @flyweights[key] = Flyweight.new(shared_state)
    else
      puts &#39;FlyweightFactory: Reusing existing flyweight.&#39;
    end

    @flyweights[key]
  end

  def list_flyweights
    puts &quot;FlyweightFactory: I have #{@flyweights.size} flyweights:&quot;
    print @flyweights.keys.join(&quot;\n&quot;)
  end
end

# @param [FlyweightFactory] factory
# @param [String] plates
# @param [String] owner
# @param [String] brand
# @param [String] model
# @param [String] color
def add_car_to_police_database(factory, plates, owner, brand, model, color)
  puts &quot;\n\nClient: Adding a car to database.&quot;
  flyweight = factory.get_flyweight([brand, model, color])
  # The client code either stores or calculates extrinsic state and passes it to
  # the flyweight&#39;s methods.
  flyweight.operation([plates, owner])
end

# The client code usually creates a bunch of pre-populated flyweights in the
# initialization stage of the application.

factory = FlyweightFactory.new([
                                 %w[Chevrolet Camaro2018 pink],
                                 [&#39;Mercedes Benz&#39;, &#39;C300&#39;, &#39;black&#39;],
                                 [&#39;Mercedes Benz&#39;, &#39;C500&#39;, &#39;red&#39;],
                                 %w[BMW M5 red],
                                 %w[BMW X6 white]
                               ])

factory.list_flyweights

add_car_to_police_database(factory, &#39;CL234IR&#39;, &#39;James Doe&#39;, &#39;BMW&#39;, &#39;M5&#39;, &#39;red&#39;)

add_car_to_police_database(factory, &#39;CL234IR&#39;, &#39;James Doe&#39;, &#39;BMW&#39;, &#39;X1&#39;, &#39;red&#39;)

puts &quot;\n\n&quot;

factory.list_flyweights</code></pre>
</figure>

#### []{#example-0--output-txt .anchor} **output.txt:** Результат виконання

<figure class="code">
<pre class="code" lang="output"><code>FlyweightFactory: I have 5 flyweights:
Camaro2018_Chevrolet_pink
C300_Mercedes Benz_black
C500_Mercedes Benz_red
BMW_M5_red
BMW_X6_white

Client: Adding a car to database.
FlyweightFactory: Reusing existing flyweight.
Flyweight: Displaying shared ([&quot;BMW&quot;,&quot;M5&quot;,&quot;red&quot;]) and unique ([&quot;CL234IR&quot;,&quot;James Doe&quot;]) state.

Client: Adding a car to database.
FlyweightFactory: Can&#39;t find a flyweight, creating new one.
Flyweight: Displaying shared ([&quot;BMW&quot;,&quot;X1&quot;,&quot;red&quot;]) and unique ([&quot;CL234IR&quot;,&quot;James Doe&quot;]) state.

FlyweightFactory: I have 6 flyweights:
Camaro2018_Chevrolet_pink
C300_Mercedes Benz_black
C500_Mercedes Benz_red
BMW_M5_red
BMW_X6_white
BMW_X1_red</code></pre>
</figure>

## **Легковаговик** іншими мовами програмування

[![Легковаговик на
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Легковаговик на C#"){.prog-lang-link}
[![Легковаговик на
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Легковаговик на C++"){.prog-lang-link}
[![Легковаговик на
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Легковаговик на Go"){.prog-lang-link}
[![Легковаговик на
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Легковаговик на Java"){.prog-lang-link}
[![Легковаговик на
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Легковаговик на PHP"){.prog-lang-link}
[![Легковаговик на
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Легковаговик на Python"){.prog-lang-link}
[![Легковаговик на
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Легковаговик на Rust"){.prog-lang-link}
[![Легковаговик на
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Легковаговик на Swift"){.prog-lang-link}
[![Легковаговик на
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Легковаговик на TypeScript"){.prog-lang-link}
:::::::::::::::

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
:::::::::::::::::::::
::::::::::::::::::::::
